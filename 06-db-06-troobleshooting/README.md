# Домашнее задание к занятию "6.6. Troubleshooting"

## Задача 1

Перед выполнением задания ознакомьтесь с документацией по [администрированию MongoDB](https://docs.mongodb.com/manual/administration/).

Пользователь (разработчик) написал в канал поддержки, что у него уже 3 минуты происходит CRUD операция в MongoDB и её 
нужно прервать. 

Вы как инженер поддержки решили произвести данную операцию:
- напишите список операций, которые вы будете производить для остановки запроса пользователя
- предложите вариант решения проблемы с долгими (зависающими) запросами в MongoDB

#### Ответ
- Для начала выявлю проблемный запрос с помощью команды db.currentOp({"secs_running":{$gte: 5}})
```bash
test> db.currentOp({"secs_running":{$gte: 5}})
{
  inprog: [
    {
      type: 'op',
      host: 'c8ac4eaa7003:27017',
      desc: 'conn6',
      connectionId: 6,
      client: '127.0.0.1:49448',
      appName: 'mongosh 1.6.0',
      clientMetadata: {
        driver: { name: 'nodejs|mongosh', version: '4.10.0' },
        os: {
          type: 'Linux',
          name: 'linux',
          architecture: 'x64',
          version: '5.15.0-47-generic'
        },
        platform: 'Node.js v16.17.0, LE (unified)',
        version: '4.10.0|1.6.0',
        application: { name: 'mongosh 1.6.0' }
      },
      active: true,
      currentOpTime: '2022-10-10T13:02:10.689+00:00',
      threaded: true,
      opid: 12902,
      secs_running: Long("9"),
      microsecs_running: Long("9064259"),
      op: 'command',
      ns: 'admin.$cmd',
      command: {
        hello: true,
        maxAwaitTimeMS: 10000,
        topologyVersion: {
          processId: ObjectId("634413b9e8ee099508ed0e5a"),
          counter: Long("0")
        },
        '$db': 'admin'
      },
      numYields: 0,
      waitingForLatch: {
        timestamp: ISODate("2022-10-10T13:02:01.725Z"),
        captureName: 'AnonymousLatch'
      },
      locks: {},
      waitingForLock: false,
      lockStats: {},
      waitingForFlowControl: false,
      flowControlStats: {}
    }
  ],
  ok: 1
}
test>
```
После чего завершу зависший процесс с помощью функции killOp:
```bash
db.killOp(<opId>)
```

- предложите вариант решения проблемы с долгими (зависающими) запросами в MongoDB
Использую профилирование.
Сначала посмотрю текущий уровень профилирования:
```bash
test> db.getProfilingStatus()
{ was: 0, slowms: 100, sampleRate: 1, ok: 1 }
```
затем найду все медленные запросы, подходящие под профиль (у меня - отсутствуют):
```bash
admin> db.system.profile.find({op: {$ne:'command'}}).pretty()

admin>
```
при необходимости -  создам индексы
```bash
admin> db.users.getIndexes()
[ { v: 2, key: { _id: 1 }, name: '_id_' } ]
admin> db.users.createIndex({title:1})
title_1
admin> db.users.getIndexes()
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { title: 1 }, name: 'title_1' }
]
admin>
```


## Задача 2

Перед выполнением задания познакомьтесь с документацией по [Redis latency troobleshooting](https://redis.io/topics/latency).

Вы запустили инстанс Redis для использования совместно с сервисом, который использует механизм TTL. 
Причем отношение количества записанных key-value значений к количеству истёкших значений есть величина постоянная и
увеличивается пропорционально количеству реплик сервиса. 

При масштабировании сервиса до N реплик вы увидели, что:
- сначала рост отношения записанных значений к истекшим
- Redis блокирует операции записи

Как вы думаете, в чем может быть проблема?

#### Ответ
Отсутствует доступная оперативная память
 
## Задача 3

Перед выполнением задания познакомьтесь с документацией по [Common Mysql errors](https://dev.mysql.com/doc/refman/8.0/en/common-errors.html).

Вы подняли базу данных MySQL для использования в гис-системе. При росте количества записей, в таблицах базы,
пользователи начали жаловаться на ошибки вида:
```python
InterfaceError: (InterfaceError) 2013: Lost connection to MySQL server during query u'SELECT..... '
```

Как вы думаете, почему это начало происходить и как локализовать проблему?

Какие пути решения данной проблемы вы можете предложить?

#### Ответ
Данные события начали происходить в связи с увеличением нагрузки.
пути решения:
- Увеличить значение параметров : connect_timeout, interactive_timeout, wait_timeout
- Добавить ресурсов на машине
- Создать индексы для оптимизации  и ускорения запросов (определить по плану запросов)

## Задача 4

Перед выполнением задания ознакомтесь со статьей [Common PostgreSQL errors](https://www.percona.com/blog/2020/06/05/10-common-postgresql-errors/) из блога Percona.

Вы решили перевести гис-систему из задачи 3 на PostgreSQL, так как прочитали в документации, что эта СУБД работает с 
большим объемом данных лучше, чем MySQL.

После запуска пользователи начали жаловаться, что СУБД время от времени становится недоступной. В dmesg вы видите, что:

`postmaster invoked oom-killer`

Как вы думаете, что происходит?

Как бы вы решили данную проблему?

#### Ответ

Судя по всем, отсутствует доступная оперативная память, из за чего операционная система завершает процессы, утилизирующие память, чтобы предотвратить падение всей системы. Механизм называется OOM Killer.
Требуется тонко настроить использование памяти postgress'ом с помощью следующих параметров:

max_connections
shared_buffer
work_mem
effective_cache_size
maintenance_work_mem

Следующая формула поможет определится с верными параметрами
```
Actual max RAM = shared_buffers + (temp_buffers + work_mem) * max_connections
```
Также, как советуют на хабре (перевод статьи Ibrar Ahmed), чтобы не приходилось использовать OOM-Killer для завершения PostgreSQL, установите для vm.overcommit_memory значение 2. Это не гарантирует, что OOM-Killer не придется вмешиваться, но снизит вероятность принудительного завершения процесса PostgreSQL.
Также, можно ограничить nr_overcommit_hugepages
```
echo "$(getconf _PHYS_PAGES) * 0.45 / 512" | bc > /proc/sys/vm/nr_overcommit_hugepages
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
