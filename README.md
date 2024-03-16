# Бэкапы
1.Создаем ВМ/докер c ПГ.
```
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-14 
```
2.Создаем БД, схему и в ней таблицу.
```
create database otus;
```
3.Заполним таблицы автосгенерированными 100 записями.
```
create table students as select generate_series(1, 10) as id, md5(random()::text)::char(10) as fio;
create table students1 as select generate_series(1, 10) as id, md5(random()::text)::char(10) as fio;
```
4. Под линукс пользователем Postgres создадим каталог для бэкапов
   дал права на весь путь до нужной мне директории
   зашел под 
5.Сделаем логический бэкап используя утилиту COPY
```
\copy students to '/home/user/backup/backup_copy.sql';
```
6.Восстановим в 2 таблицу данные из бэкапа.
```
\copy students1 from '/home/user/backup/backup_copy.sql';
```
7.Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц
```
sudo -u postgres pg_dump -t students -t students1 --create -Fc > /home/user/backup/backup_dump.gz
```
8.Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
```
sudo -u postgres createdb test1 && sudo -u postgres pg_restore -d test1 -t students1 -j 2 /home/user/backup/backu
p_dump.gz
```
```
user@postgres:/tmp$ sudo -u postgres psql
psql (14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Type "help" for help.

postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# \dt
           List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | students1 | table | postgres
(1 row)
```
