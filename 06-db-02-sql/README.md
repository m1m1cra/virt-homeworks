# Домашнее задание к занятию "6.2. SQL"
## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

#### Ответ
```bash
root@bhdevops:/home/avdeevan/pg# cat docker-compose.yml
version: "3.9"
services:
  postgres:
    image: postgres:12
    environment:
      POSTGRES_DB: "public"          
      POSTGRES_USER: "alexey"        #строго в рамках теста, на проде сделаю через .env
      POSTGRES_PASSWORD: "password"  #строго в рамках теста, на проде сделаю через .env
      PGDATA: "./data"
    volumes:
      - ./data:/data
      - ./backup:/backup
    ports:
      - "5433:5432"
```

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

#### Ответ
- итоговый список БД после выполнения пунктов выше,
```bash
test_db=# SELECT datname FROM pg_database;
  datname
-----------
 postgres
 public
 template1
 template0
 test_db
(5 rows)

test_db=#
```
- описание таблиц (describe)
```bash
test_db=# \d+ clients
                                                    Table "public.clients"
  Column  |     Type      | Collation | Nullable |               Default               | Storage  | Stats target | Description
----------+---------------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id       | integer       |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |
 lastname | character(30) |           |          |                                     | extended |              |
 country  | character(30) |           |          |                                     | extended |              |
 booking  | integer       |           |          |                                     | plain    |              |
Indexes:
    "intbcl" btree (country)
Foreign-key constraints:
    "clients_booking_fkey" FOREIGN KEY (booking) REFERENCES orders(id)
Access method: heap


test_db=# \d+ orders
                                                   Table "public.orders"
 Column |     Type     | Collation | Nullable |              Default               | Storage  | Stats target | Description
--------+--------------+-----------+----------+------------------------------------+----------+--------------+-------------
 id     | integer      |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |
 name   | character(30) |           |          |                                    | extended |              |
 price  | integer      |           |          |                                    | plain    |              |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_booking_fkey" FOREIGN KEY (booking) REFERENCES orders(id)
Access method: heap
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
```bash
SELECT * FROM information_schema.table_privileges
where (grantee = 'test-simple-user') or (grantee = 'test-admin-user')
ORDER BY grantee;
```

- список пользователей с правами над таблицами test_db
```bash
test_db=# SELECT * FROM information_schema.table_privileges where (grantee = 'test-simple-user') or (grantee = 'test-admin-user') ORDER BY grantee;
 grantor |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy
---------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 alexey  | test-admin-user  | test_db       | public       | clients    | INSERT         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | clients    | SELECT         | NO           | YES
 alexey  | test-admin-user  | test_db       | public       | clients    | UPDATE         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | clients    | DELETE         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | clients    | TRUNCATE       | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | clients    | REFERENCES     | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | clients    | TRIGGER        | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | INSERT         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | SELECT         | NO           | YES
 alexey  | test-admin-user  | test_db       | public       | orders     | UPDATE         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | DELETE         | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | TRUNCATE       | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | REFERENCES     | NO           | NO
 alexey  | test-admin-user  | test_db       | public       | orders     | TRIGGER        | NO           | NO
 alexey  | test-simple-user | test_db       | public       | orders     | INSERT         | NO           | NO
 alexey  | test-simple-user | test_db       | public       | clients    | INSERT         | NO           | NO
 alexey  | test-simple-user | test_db       | public       | clients    | SELECT         | NO           | YES
 alexey  | test-simple-user | test_db       | public       | clients    | UPDATE         | NO           | NO
 alexey  | test-simple-user | test_db       | public       | clients    | DELETE         | NO           | NO
 alexey  | test-simple-user | test_db       | public       | orders     | SELECT         | NO           | YES
 alexey  | test-simple-user | test_db       | public       | orders     | UPDATE         | NO           | NO
 alexey  | test-simple-user | test_db       | public       | orders     | DELETE         | NO           | NO
(22 rows)
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.
#### Ответ:
```bash
test_db=# select count(*) from clients;
 count
-------
     5
(1 row)

test_db=# select count(*) from orders;
 count
-------
     5
(1 row)

test_db=#
```
Также, вывожу все записи таблиц:
```bash
test_db=# select * from orders;
 id |                        name                        | price
----+----------------------------------------------------+-------
  1 | Шоколад                                            |    10
  2 | Принтер                                            |  3000
  3 | Книга                                              |   500
  4 | Монитор                                            |  7000
  5 | Гитара                                             |  4000
(5 rows)

test_db=# select * from clients;
 id |            lastname            |            country             | booking
----+--------------------------------+--------------------------------+---------
 21 | Иванов Иван Иванович           | USA                            |
 22 | Петров Петр Петрович           | Canada                         |
 23 | Иоганн Себастьян Бах           | Japan                          |
 24 | Ронни Джеймс Дио               | Russia                         |
 25 | Ritchie Blackmore              | Russia                         |
(5 rows)

test_db=#
```


## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.

#### Ответ
- Приведите SQL-запросы для выполнения данных операций.
```sql
update clients 
set booking = 3
where lastname = 'Иванов Иван Иванович'

update clients 
set booking = 5
where lastname = 'Иоганн Себастьян Бах'

update clients 
set booking = 4
where lastname = 'Петров Петр Петрович'
```
- Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
Если я правильно понял - вывести всех пользователей, которые хоть что-то купили - тогда вывод такой:
```sql
test_db=# select * from clients c
where c.booking is not null;
 id |            lastname            |            country             | booking
----+--------------------------------+--------------------------------+---------
 23 | Иоганн Себастьян Бах           | Japan                          |       5
 22 | Петров Петр Петрович           | Canada                         |       4
 21 | Иванов Иван Иванович           | USA                            |       3
(3 rows)
```
Также, прилагаю запрос и вывод через INNER JOIN с добавлением совершенных заказов:
```sql
test_db=# select c.lastname, c.country, o.name
from clients c
inner join orders o on o.id = c.booking;
            lastname            |            country             |                        name
--------------------------------+--------------------------------+----------------------------------------------------
 Иоганн Себастьян Бах           | Japan                          | Гитара
 Петров Петр Петрович           | Canada                         | Монитор
 Иванов Иван Иванович           | USA                            | Книга
(3 rows)

test_db=#
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.
#### Ответ
```sql
test_db=# explain  select * from clients c
where c.booking is not null;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on clients c  (cost=0.00..1.00 rows=1 width=256)
   Filter: (booking IS NOT NULL)
(2 rows)

test_db=# explain  analyze select * from clients c
where c.booking is not null;
```
Данная команда позволяет проанализировать скорость выполнения запроса, понять, что необходимо поменять в целях оптимизации
Seq Scan — последовательное, блок за блоком, чтение данных таблицы clients.
cost - абстрактная величина, демонстрирующая затратность операции, 1 позиция - время на вывод 1 строки, вторая - всех строк.
rows — приблизительное количество возвращаемых строк при выполнении операции Seq Scan. Это значение возвращает планировщик.
width — средний размер одной строки в байтах.

Это вывод планировщика, можем посмотреть реальные данные:
```sql
test_db=# explain  (analyze) select * from clients c
where c.booking is not null;
                                             QUERY PLAN
-----------------------------------------------------------------------------------------------------
 Seq Scan on clients c  (cost=0.00..1.00 rows=1 width=256) (actual time=0.028..0.030 rows=3 loops=1)
   Filter: (booking IS NOT NULL)
   Rows Removed by Filter: 2
 Planning Time: 0.319 ms
 Execution Time: 0.056 ms
(5 rows)
```

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
