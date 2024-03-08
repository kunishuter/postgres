# Нагрузочное тестирование и тюнинг PostgreSQL
Сделал со стандартным конфигом 
```sudo su postgres```
инициализация ```pgbench -i postgres```
запуск бенчмарка ```pgbench -c 50 -j 2 -P 10 -T 60 postgres```
Ответ:
```
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 684.9 tps, lat 72.223 ms stddev 55.196, 0 failed
progress: 20.0 s, 652.5 tps, lat 75.713 ms stddev 57.315, 0 failed
progress: 30.0 s, 653.8 tps, lat 77.310 ms stddev 65.536, 0 failed
progress: 40.0 s, 680.3 tps, lat 73.556 ms stddev 56.544, 0 failed
progress: 50.0 s, 592.9 tps, lat 83.332 ms stddev 70.802, 0 failed
progress: 60.0 s, 632.8 tps, lat 79.898 ms stddev 69.078, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 39022
number of failed transactions: 0 (0.000%)
latency average = 76.867 ms
latency stddev = 62.575 ms
initial connection time = 59.388 ms
tps = 649.829052 (without initial connection time)
```
Сгенерировал конфиг опираясь на настройки ```https://pgtune.sainth.de/ ```
Подложил в ```/etc/postgresql/15/main/conf.d ``` 
рестартанул кластер ```sudo pg_ctlcluster 15 main restart```
проверил что конфиг переставился ```select * from pg_settings where name = 'work_mem' ``` тут также можно  посмотреть файл конфигурации который используется ```sourcefile      | /etc/postgresql/15/main/conf.d/newconf.conf``` наш конфиг подтянулся
```sudo su postgres```
инициализация ```pgbench -i postgres```
запуск бенчмарка ```pgbench -c 50 -j 2 -P 10 -T 60 postgres```
Ответ:
```
postgres@postgres:/etc/postgresql/15/main/conf.d$ pgbench -c 50 -j 2 -P 10 -T 60 postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 678.4 tps, lat 72.781 ms stddev 55.421, 0 failed
progress: 20.0 s, 583.0 tps, lat 85.705 ms stddev 81.814, 0 failed
progress: 30.0 s, 710.2 tps, lat 70.488 ms stddev 50.670, 0 failed
progress: 40.0 s, 689.8 tps, lat 72.484 ms stddev 53.833, 0 failed
progress: 50.0 s, 574.6 tps, lat 86.734 ms stddev 84.176, 0 failed
progress: 60.0 s, 685.8 tps, lat 73.133 ms stddev 55.789, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 39268
number of failed transactions: 0 (0.000%)
latency average = 76.379 ms
latency stddev = 64.215 ms
initial connection time = 63.467 ms
tps = 654.036387 (without initial connection time)
```
Заметна небольшая разница в tps, в моем случае можно сказать, что по дефолту при установки посгресса под мою машину настройки подходят неплохо, под другие параметры машины уже необходимо еще тюнить
