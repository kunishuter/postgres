# Логический уровень PostgreSQL 
1. создание базы ```CREATE DATABASE testdb;```, подключение к ней ```\с testdb;```
2. создание схемы ```CREATE SCHEMA testnm;```
3. создание таблицы ```CREATE TABLE t1(i int);```
4. добавление значение ```INSERT INTO testL VALUES (1)```
5. создание роли ``` create role RO;```
6. подключение к базе ```GRANT CONNECT ON DATABASE testdb to RO```
7. право на использование схемы ```GRANT USAGE ON SCHEMA testnm TO RO;```
8. дайте новой роли право на select для всех таблиц схемы testnm ```GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO RO;```
9. создайте пользователя testread с паролем test123 ```CREATE USER testread WITH PASSWORD 'test123';```
10. дайте роль readonly пользователю testread ```GRANT RO TO testread;```
11. зайдите под пользователем testread в базу данных testdb ``` sudo -u postgres psql -U testread -h 127.0.0.1 -W -d testdb``` ответ: ```select * from t1; ERROR:  relation "t1" does not exist LINE 1: select * from t1;```
12. проблема в том, что наша таблица создана в схеме public, а туда права не давали
13. вернитесь в базу данных testdb под пользователем postgres``` sudo -u postgres psql -U testread -h 127.0.0.1 -W -d testdb``` ответ: ```select * from t1; ERROR:  relation "t1" does not exist LINE 1: select * from t1;```
14. удаление таблицы ```drop table t1;```
15. создание таблицы в схеме ```CREATE TABLE testnm.t1(c1 integer);```, добавление данных:```INSERT INTO testnm.t1 values(1);```
16. зайдите под пользователем testread в базу данных testdb: ```\c testdb testread``` ответ: ```connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "testread"```
17. снова, что то с правами, потому что это новая таблица
18. попробовал полечить редактирование привелегий: ```ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly;``` дал права, которые давали выше под юзером постгреса
19. как сделать так чтобы такое больше не повторялось? дать еще раз права
20. ```select * from testnm.t1;``` получал ошибку ```ERROR:  permission denied for table t1```, дал прав из под постгреса  ```grant select on testnm.t1 to RO;``` все получилось
21. 37 пункт был не совсем понятен, смотрел шпаргалку
22. теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2); сделав: ``` REVOKE CREATE on SCHEMA public FROM public;``` ```REVOKE ALL on DATABASE testdb FROM public; ``` мы больше не можем по умолчанию создавать объекты в схеме public для любой базы поэтому сейчас ловим ```ERROR:  permission denied for schema public```
