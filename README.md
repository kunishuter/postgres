# Настройка autovacuum с учетом особеностей производительности
1.Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB - создавал руками через yc

2.Установить на него PostgreSQL 15 с дефолтными настройками, использовал команду ```sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15```
3. Создать БД для тестов: выполнить pgbench -i postgres - создание базы ```create database test;```

```
pgbench -i postgres -U postgres
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.20 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 0.09 s, vacuum 0.04 s, primary keys 0.06 s)
```
4. Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres:
```
pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 264.5 tps, lat 30.025 ms stddev 34.481, 0 failed
progress: 12.0 s, 470.8 tps, lat 17.040 ms stddev 12.569, 0 failed
progress: 18.0 s, 499.2 tps, lat 16.021 ms stddev 16.938, 0 failed
progress: 24.0 s, 475.2 tps, lat 16.841 ms stddev 13.304, 0 failed
progress: 30.0 s, 672.5 tps, lat 11.909 ms stddev 9.162, 0 failed
progress: 36.0 s, 375.0 tps, lat 21.309 ms stddev 34.800, 0 failed
progress: 42.0 s, 653.5 tps, lat 12.225 ms stddev 10.780, 0 failed
progress: 48.0 s, 479.7 tps, lat 16.673 ms stddev 15.943, 0 failed
progress: 54.0 s, 763.3 tps, lat 10.498 ms stddev 7.834, 0 failed
progress: 60.0 s, 462.7 tps, lat 17.298 ms stddev 15.679, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 30706
number of failed transactions: 0 (0.000%)
latency average = 15.630 ms
latency stddev = 17.642 ms
initial connection time = 14.928 ms
tps = 511.730436 (without initial connection time)
```
5. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
   
6. Протестировать заново:
```
pgbench -i postgres -U postgres
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.22 s (drop tables 0.01 s, create tables 0.01 s, client-side generate 0.10 s, vacuum 0.04 s, primary keys 0.06 s).
```

```
pgbench -c8 -P 6 -T 60 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 553.5 tps, lat 14.384 ms stddev 12.068, 0 failed
progress: 12.0 s, 356.0 tps, lat 22.495 ms stddev 17.523, 0 failed
progress: 18.0 s, 493.3 tps, lat 16.193 ms stddev 23.152, 0 failed
progress: 24.0 s, 781.0 tps, lat 10.094 ms stddev 7.333, 0 failed
progress: 30.0 s, 535.5 tps, lat 15.159 ms stddev 27.024, 0 failed
progress: 36.0 s, 394.0 tps, lat 20.274 ms stddev 18.252, 0 failed
progress: 42.0 s, 488.0 tps, lat 16.410 ms stddev 12.408, 0 failed
progress: 48.0 s, 300.8 tps, lat 26.639 ms stddev 42.392, 0 failed
progress: 54.0 s, 405.0 tps, lat 19.737 ms stddev 14.059, 0 failed
progress: 60.0 s, 435.8 tps, lat 18.313 ms stddev 18.677, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 28466
number of failed transactions: 0 (0.000%)
latency average = 16.860 ms
latency stddev = 20.309 ms
initial connection time = 15.808 ms
tps = 474.429483 (without initial connection time)

```
7. Что изменилось и почему?
Не могу сказать, что заметил сильные изменения, второй запрос отработал чуть быстрее(511.730436 -> 474.429483), первый отработал дольше(0.20 -> 0.22)

8. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
Создание таблицы:
```
CREATE TABLE student(
  id serial,
  fio char(100)
) WITH (autovacuum_enabled = off);
```
Заполнение:
```
INSERT INTO student(fio) SELECT 'noname' FROM generate_series(1,1000000);
```
9.Посмотреть размер файла с таблицей ``` SELECT pg_size_pretty(pg_total_relation_size('student')); ``` 135 MB

10. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```update student set fio = 'name';``` x5

11. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

После первой проверки было 1000000 мертвых строчек, автовакуум приходил по времени 2024-02-27 20:06:00.424816+00, но время некорректное по какой то причине, (системное время 23:08), обновил буквально через пару секунд, мертвых уже не было(при этом время не изменилось).

12. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```update student set fio = 'name+!';``` x5
13. Посмотреть размер файла с таблицей
``` SELECT pg_size_pretty(pg_total_relation_size('student')); ``` 432 MB
14. Отключить Автовакуум на конкретной таблице ```ALTER TABLE student SET (autovacuum_enabled = off); ```
15. 10 раз обновить все строчки и добавить к каждой строчке любой символ
```update student set fio = 'name+!';``` x5
16. Посмотреть размер файла с таблицей:
``` SELECT pg_size_pretty(pg_total_relation_size('student')); ``` 1502 MB
16. Объясните полученный результат
c выключенным вакууомом мертвые строчки не удаляются в этом причина размера файла с таблицей 

Задание со *:
Процедура
```
CREATE OR REPLACE PROCEDURE update_table_10_times()
LANGUAGE plpgsql
AS $$
DECLARE
    i INT := 1;
BEGIN
    WHILE i <= 10 LOOP
        RAISE NOTICE 'Шаг цикла: %', i; -- Вывод номера шага цикла
        UPDATE student
        SET fio = fio || 'name' || i; -- Новое значение для столбца fio
        i := i + 1;
    END LOOP;
END;
$$;
```
```
CALL update_table_10_times();
```
вывод консоли с выполнением процедуры
```
postgres=# CALL update_table_10_times();
NOTICE:  Шаг цикла: 1
NOTICE:  Шаг цикла: 2
NOTICE:  Шаг цикла: 3
NOTICE:  Шаг цикла: 4
NOTICE:  Шаг цикла: 5
NOTICE:  Шаг цикла: 6
NOTICE:  Шаг цикла: 7
NOTICE:  Шаг цикла: 8
NOTICE:  Шаг цикла: 9
NOTICE:  Шаг цикла: 10
CALL
```

