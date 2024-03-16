# Репликация
1.На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
```
create table test (id serial primary key,
name varchar(255),
value integer
);
```
```
create table test2 (id serial primary key,
name varchar(255),
value integer
);
```
2.Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
поменял ```wal_level = logical``` в конфиге посгреса на первой машине
создаем публикацию 
```
create PUBLICATION test_pub for table test;
```
Задать пароль
```\password ```
на второй машине создаем подписку 
```
CREATE SUBSCRIPTION otus_sub
CONNECTION 'host=51.250.99.23 user=postgres password=123 dbname=otus'
PUBLICATION test_pub WITH (copy_data = true);
```
предварительно поменяв postgresql.conf and pg_hba.conf
3.На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.
```
create table test (id serial primary key,
name varchar(255),
value integer
);
```
```
create table test2 (id serial primary key,
name varchar(255),
value integer
);
```
4.Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
```ALTER SYSTEM SET wal_level = logical;```
Рестартуем кластер
```sudo pg_ctlcluster 14 main restart```
создаем публикацию 
```CREATE PUBLICATION test_pub FOR TABLE test2;```
Задать пароль
```\password ```
на первой машине машине создаем подписку 
```CREATE SUBSCRIPTION otus_sub
CONNECTION 'host=51.250.108.250 user=postgres password=123 dbname=otus' 
PUBLICATION test_pub WITH (copy_data = true);
```
предварительно поменяв postgresql.conf and pg_hba.conf

5. 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
поменял postgresql.conf and pg_hba.conf
проверил ```show wal_level;``` на всех машинах на 1 и 2 logical на 3 replica 
```
CREATE SUBSCRIPTION otus_sub2
CONNECTION 'host=51.250.108.250 user=postgres password=123 dbname=otus'
PUBLICATION test_pub WITH (copy_data = true);

CREATE SUBSCRIPTION otus_sub3
CONNECTION 'host=51.250.99.23 user=postgres password=123 dbname=otus'
PUBLICATION test_pub WITH (copy_data = true);
```
