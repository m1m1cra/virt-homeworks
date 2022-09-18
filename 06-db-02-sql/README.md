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
  Column  |     Type     | Collation | Nullable |               Default               | Storage  | Stats target | Description
----------+--------------+-----------+----------+-------------------------------------+----------+--------------+-------------
 id       | integer      |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |
 lastname | character(1) |           |          |                                     | extended |              |
 country  | character(1) |           |          |                                     | extended |              |
 booking  | integer      |           |          |                                     | plain    |              |
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "clients_country_key" UNIQUE CONSTRAINT, btree (country)
    "indtbcl" UNIQUE, btree (country)
Foreign-key constraints:
    "clients_booking_fkey" FOREIGN KEY (booking) REFERENCES orders(id)
Access method: heap




test_db=# \d+ orders
                                                   Table "public.orders"
 Column |     Type     | Collation | Nullable |              Default               | Storage  | Stats target | Description
--------+--------------+-----------+----------+------------------------------------+----------+--------------+-------------
 id     | integer      |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |
 name   | character(1) |           |          |                                    | extended |              |
 price  | integer      |           |          |                                    | plain    |              |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_booking_fkey" FOREIGN KEY (booking) REFERENCES orders(id)
Access method: heap
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- 
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

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

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
