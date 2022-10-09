# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib` 
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

#### Ответ
- текст Dockerfile манифеста
```bash
FROM elasticsearch:7.17.6
ADD elasticsearch.yml /usr/share/elasticsearch/config
RUN mkdir /var/lib/logs && chown elasticsearch:elasticsearch /var/lib/logs && mkdir /var/lib/data && chown elasticsearch:elasticsearch /var/lib/data
```
- Также, прикладываю elasticsearch.yml
```bash
cluster.name: "docker-cluster"
network.host: 0.0.0.0
node.name: netology_test
path.data: /var/lib/data
path.logs: /var/lib/logs
```
- ссылку на образ в репозитории dockerhub
```html
https://hub.docker.com/repository/docker/m1cra/elasticsearch
```

- ответ `elasticsearch` на запрос пути `/` в json виде
```bash
{
  "name" : "netology_test",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "i11vTwRKRdWisRncjja1oQ",
  "version" : {
    "number" : "7.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "f65e9d338dc1d07b642e14a27f338990148ee5b6",
    "build_date" : "2022-08-23T11:08:48.893373482Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled` 
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

#### Ответ
- добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:
```bash
curl -X PUT localhost:9200/ind-1 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
curl -X PUT localhost:9200/ind-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 2,  "number_of_replicas": 1 }}'
curl -X PUT localhost:9200/ind-3 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 4,  "number_of_replicas": 2 }}'
```
- Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.
```bash
root@bhdevops:/home/avdeevan/elastic# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases H4swznAITWqhc3KRR1DDyQ   1   0         40            0     38.4mb         38.4mb
green  open   ind-1            DVwsPkbcS3SC2hjDyucqcQ   1   0          0            0       226b           226b
yellow open   ind-3            euiHBahSSJStKU7AOyqj4g   4   2          0            0       904b           904b
yellow open   ind-2            3lQahOxFSo2t9zVyo74BvA   2   1          0            0       452b           452b
root@bhdevops:/home/avdeevan/elastic# curl -X GET 'http://localhost:9200/_cluster/health/ind-1?pretty' && \ 
curl -X GET 'http://localhost:9200/_cluster/health/ind-2?pretty' && \ 
curl -X GET 'http://localhost:9200/_cluster/health/ind-3?pretty'
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
{
  "cluster_name" : "docker-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
{
  "cluster_name" : "docker-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
root@bhdevops:/home/avdeevan/elastic#
```
- Получите состояние кластера `elasticsearch`, используя API.
```bash
root@bhdevops:/home/avdeevan/elastic# curl -XGET localhost:9200/_cluster/health/?pretty=true
{
  "cluster_name" : "docker-cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
```
- Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Потому что указанное число реплик отсутствует 

- Удалите все индексы.
```bash
root@bhdevops:/home/avdeevan/elastic# curl -X DELETE 'http://localhost:9200/ind-1?pretty' && curl -X DELETE 'http://localhost:9200/ind-2?pretty' && curl -X DELETE 'http://localhost:9200/ind-3?pretty'
{
  "acknowledged" : true
}
{
  "acknowledged" : true
}
{
  "acknowledged" : true
}
root@bhdevops:/home/avdeevan/elastic#
```

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

#### Ответ
Добавил в файл elasticsearch.yml строку path.repo: /usr/share/elasticsearch/snapshots, после чего вызвал api
```bash
root@3941133a3792:/usr/share/elasticsearch# curl -XPOST localhost:9200/_snapshot/netology_backup?pretty -H 'Content-Type: application/json' -d'{"type": "fs", "settings": { "location":"/usr/share/elasticsearch/snapshots" }}'
{
  "acknowledged" : true
}
```
- Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.
```bash
root@3941133a3792:/usr/share/elasticsearch# curl -X PUT localhost:9200/test -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}root@3941133a3792:/usr/share/elasticsearch#
root@3941133a3792:/usr/share/elasticsearch# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases 8mULwHuiQCCNXcB8fzvEAQ   1   0         40            0     38.4mb         38.4mb
green  open   test             Ui0M2z6mRv6OZYH1ie9dfA   1   0          0            0       226b           226b
---
```
- [Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) состояния кластера `elasticsearch`.
**Приведите в ответе** список файлов в директории со `snapshot`ами.
```bash
root@3941133a3792:/usr/share/elasticsearch# curl -X PUT localhost:9200/_snapshot/netology_backup/elasticsearch?wait_for_completion=true
{"snapshot":{"snapshot":"elasticsearch","uuid":"4DFehc_vQYmuDm9PKuc6qQ","repository":"netology_backup","version_id":7170699,"version":"7.17.6","indices":[".ds-ilm-history-5-2022.10.09-000001",".geoip_databases",".ds-.logs-deprecation.elasticsearch-default-2022.10.09-000001","test","ind-1"],"data_streams":["ilm-history-5",".logs-deprecation.elasticsearch-default"],"include_global_state":true,"state":"SUCCESS","start_time":"2022-10-09T19:54:30.011Z","start_time_in_millis":1665345270011,"end_time":"2022-10-09T19:54:31.411Z","end_time_in_millis":1665345271411,"duration_in_millis":1400,"failures":[],"shards":{"total":5,"failed":0,"successful":5},"feature_states":[{"feature_name":"geoip","indices":[".geoip_databases"]}]}}root@3941133a3792:/usr/share/elasticsearch#
root@3941133a3792:/usr/share/elasticsearch# ls -l /usr/share/elasticsearch/snapshots/
total 48
-rw-rw-r-- 1 elasticsearch root  1671 Oct  9 19:54 index-0
-rw-rw-r-- 1 elasticsearch root     8 Oct  9 19:54 index.latest
drwxrwxr-x 7 elasticsearch root  4096 Oct  9 19:54 indices
-rw-rw-r-- 1 elasticsearch root 29221 Oct  9 19:54 meta-4DFehc_vQYmuDm9PKuc6qQ.dat
-rw-rw-r-- 1 elasticsearch root   734 Oct  9 19:54 snap-4DFehc_vQYmuDm9PKuc6qQ.dat
root@3941133a3792:/usr/share/elasticsearch#
```
- Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.
```bash
root@3941133a3792:/usr/share/elasticsearch# curl -X DELETE 'http://localhost:9200/test?pretty'
{
  "acknowledged" : true
}
root@3941133a3792:/usr/share/elasticsearch# curl -X PUT localhost:9200/test-2 -H 'Content-Type: application/json' -d'{ "settings": { "number_of_shards": 1,  "number_of_replicas": 0 }}'
{"acknowledged":true,"shards_acknowledged":true,"index":"test-2"}root@3941133a3792:/usr/share/elasticsearch#
root@3941133a3792:/usr/share/elasticsearch# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2           lgn1rbOwQhKGyGoeaSOg9w   1   0          0            0       226b           226b
green  open   .geoip_databases 8mULwHuiQCCNXcB8fzvEAQ   1   0         40            0     38.4mb         38.4mb
```
-[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 
- **Приведите в ответе** запрос к API восстановления и итоговый список индексов.
```bash
curl -X POST localhost:9200/_snapshot/netology_backup/elasticsearch/_restore?pretty -H 'Content-Type: application/json' -d'{"include_global_state":true}'
{
  "accepted" : true
}
root@3941133a3792:/usr/share/elasticsearch/snapshots# curl -X GET 'http://localhost:9200/_cat/indices?v'
health status index                     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases          8mULwHuiQCCNXcB8fzvEAQ   1   0         40            0     38.4mb         38.4mb
green  open   test                      7vb9RTExSjGWjztJYDPg7w   1   0          0            0       226b           226b
```


### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
