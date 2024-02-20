# Физический уровень PostgreSQL
1. Установка postgres на локальной машине ```sudo apt update``` ```sudo apt install postgresql postgresql-contrib``` ```sudo systemctl start postgresql.service```
2. Подключение к базе ```sudo -i -u postgres psql```
3. Создание таблиц
```postgres=# create table test(c1 text);```
```postgres=# insert into test values('1');```
4. остоновка postgres'a ```sudo systemctl stop postgresql@14-main```
5. Создание диска через UI yc
6. Присоединение диска через UI yc
7. Инициализация диска по инструкции:  https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux.Также при настройке помогла инструкция: https://cloud.ru/ru/docs/evs/ug/topics/guides__initialize-disk-linux-parted.html, также настроил автомаунтинг диска по инструкции выше
8. После перезагрузки проблем не возникло.
9. ```chown -R postgres:postgres /mnt/sdc/ ```
10. ``` mv /var/lib/postgresql/14 /mnt/sdc/```
11. после попытки запустить кластер он не поднимался по причине переноса даты, поменял /var/lib/postgresql/14/main на /mnt/sdc/14/main/ в postgresql.conf(сделал даемон релоуд и рестарт службы)
12. зайдите через через psql и проверьте содержимое ранее созданной таблицы - данные остались
    
задание со звездочкой *: 
1. создание новой ВМ
2. Установка postgres на локальной машине ```sudo apt update``` ```sudo apt install postgresql postgresql-contrib``` ```sudo systemctl start postgresql.service``
3. удаление ```sudo rm -rf /var/lib/postgresql/*```
4. Отключение диска от старой машины и присоединение диска к новой машине через UI yc
5. не разобрался, как дальше себя диск ведет, предположение, что ремаунтинг, потом закидываешь файлы в папку откуда удалили и запускаешь кластер
