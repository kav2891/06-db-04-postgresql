# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1


### Ответ:

- Команды : 
```
root@ubuntu:/# docker pull postgres:13
13: Pulling from library/postgres
5eb5b503b376: Already exists
daa0467a6c48: Already exists
7cf625de49ef: Already exists
bb8afcc973b2: Already exists
c74bf40d29ee: Already exists
2ceaf201bb22: Already exists
1255f255c0eb: Already exists
d27501cd0cca: Already exists
46d098a50510: Pull complete
415988e25784: Pull complete
f37265417882: Pull complete
4ad62bb529b5: Pull complete
781e3e6e8401: Pull complete
Digest: sha256:2e2e213c0fe4d1229373bebf114f255f68d86fba9e9f30e9bcc08e92cdf7b5b4
Status: Downloaded newer image for postgres:13
docker.io/library/postgres:13
root@ubuntu:/# docker volume create vol1_pg
vol1_pg
root@ubuntu:/# docker run --rm --name pg1-docker -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -d -p 5432:5432 -v vol1_pg:/var/lib/postgresql/data postgres:13
9462747f2e11f0d1a62674084a770c3246f8f1899184239d4872cfb2be0b1b29
root@ubuntu:/# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
9462747f2e11   postgres:13   "docker-entrypoint.s…"   20 seconds ago   Up 18 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg1-docker
root@ubuntu:/# docker exec -it pg1-docker bash

root@9462747f2e11:/# psql -h localhost -p 5432 -U postgres -W
Password:
psql (13.5 (Debian 13.5-1.pgdg110+1))
Type "help" for help.

postgres=#

```
- вывода списка БД
```
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)

```
- подключения к БД - \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
```
postgres-# \c postgres
Password: 
вывод списка таблиц postgres-# \dt в таблицах пусто , использовал параметр S - для системных объектов
postgres-# \dtS
  postgres=# \dtS
                    List of relations
   Schema   |          Name           | Type  |  Owner
------------+-------------------------+-------+----------
 pg_catalog | pg_aggregate            | table | postgres
 pg_catalog | pg_am                   | table | postgres
 pg_catalog | pg_amop                 | table | postgres
 pg_catalog | pg_amproc               | table | postgres
 pg_catalog | pg_attrdef              | table | postgres
 pg_catalog | pg_attribute            | table | postgres
 pg_catalog | pg_auth_members         | table | postgres
 
 ...
 
(62 rows)

```
- вывод описания содержимого таблиц
```
postgres-# \d[S+] NAME
postgres-# \dS+ pg_index
                                      Table "pg_catalog.pg_index"
     Column     |     Type     | Collation | Nullable | Default | Storage  | Stats target | Description 
----------------+--------------+-----------+----------+---------+----------+--------------+-------------
 indexrelid     | oid          |           | not null |         | plain    |              | 
 indrelid       | oid          |           | not null |         | plain    |              | 
 indnatts       | smallint     |           | not null |         | plain    |              | 
 indnkeyatts    | smallint     |           | not null |         | plain    |              | 
 indisunique    | boolean      |           | not null |         | plain    |              | 
 indisprimary   | boolean      |           | not null |         | plain    |              | 
 indisexclusion | boolean      |           | not null |         | plain    |              | 
 indimmediate   | boolean      |           | not null |         | plain    |              | 
 indisclustered | boolean      |           | not null |         | plain    |              | 
 indisvalid     | boolean      |           | not null |         | plain    |              | 
 indcheckxmin   | boolean      |           | not null |         | plain    |              | 
 indisready     | boolean      |           | not null |         | plain    |              | 
 indislive      | boolean      |           | not null |         | plain    |              | 
.....
```
- выход из psql
```
postgres=# \q
root@9462747f2e11:/#

```

## Задача 2

### Ответ:
```
postgres=# CREATE DATABASE test_database;
CREATE DATABASE

root@9462747f2e11:/var/lib/postgresql/data# psql -U postgres -f ./pg_backup.sql test_database

postgres=# \c test_database
Password: 
You are now connected to database "test_database" as user "postgres".
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

test_database=# select avg_width from pg_stats where tablename='orders';
 avg_width 
-----------
         4
        16
         4
(3 rows)
```

## Задача 3


### Ответ:
- преобразовать существующую таблицу в партиционированную, поэтому пересоздадим таблицу
```
test_database=# alter table orders rename to orders_simple;
ALTER TABLE
test_database=# create table orders (id integer, title varchar(80), price integer) partition by range(price);
CREATE TABLE
test_database=# create table orders_less499 partition of orders for values from (0) to (499);
CREATE TABLE
test_database=# create table orders_more499 partition of orders for values from (499) to (999999999);
CREATE TABLE
test_database=# insert into orders (id, title, price) select * from orders_simple;
INSERT 0 8
test_database=# 
```
- При изначальном проектировании таблиц можно было сделать ее секционированной, тогда не пришлось бы переименовывать исходную таблицу и переносить данные в новую.

## Задача 4


### Ответ:
```
root@9462747f2e11:/var/lib/postgresql/data# pg_dump -U postgres -d test_database >test_database_dump.sql

```

```
ALTER TABLE ONLY test_database ADD UNIQUE (title);

ALTER TABLE test_database ADD UNIQUE (title);
ALTER INDEX title
    ATTACH PARTITION title;

```
