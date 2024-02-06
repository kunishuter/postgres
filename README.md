# Установка и настройка PostgreSQL в контейнере Docker

1. Создание ключа:
```ssh-keygen -t ed25519```

2. При создании машины в яндекс облаке указываем наш публичный ключ.
Подключаемся к машине ```ssh <имя_пользователя>@<публичный_IP-адрес_ВМ>```

3. Установка докера:
```sudo apt install docker.io```
Включаем службу докера:
```sudo systemctl enable docker```
Проверяем, что докер успешно запущен:
```sudo docker run hello-world```

4. Устанавливаем PostgreSQL 15:
Стягиваем образ:
```docker pull postgres:15.0```
Поднимаем контейнер:
```docker run --name postgres -e POSTGRES_PASSWORD=user -d -p 5432:5432 -v /var/lib/postgresql/data:/var/lib/postgresql/data postgres```
Возникшие проблемы:
- *«Error starting userland proxy: listen tcp4 0.0.0.0:5432: bind: address already in use»*
лечил так:
```sudo ss -lptn 'sport = :5432'```
```sudo kill 2592```(2592 - номер процесса)
И снова поднимаем контейнер.

- *"psql: could not connect to server: Connection refused" Error when connecting to remote database*
лечит так:

```cd /etc/postgresql/9.x/main/```

open file named postgresql.conf
```sudo vi postgresql.conf```

add this line to that file
```listen_addresses = '*'```

then open file named pg_hba.conf
```sudo vi pg_hba.conf```

and add this line to that file
```host  all  all 0.0.0.0/0 md5```
It allows access to all databases for all users with an encrypted password

restart your server
```sudo /etc/init.d/postgresql restart```
И снова поднимаем контейнер.

5. Развернуть контейнер с клиентом postgres
Подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
```sudo docker run -it --rm --network host postgres psql -h localhost -U postgres -p user --port 5432```
Наполнение:
```create table persons(id serial, first_name text, second_name text);```
```insert into persons(first_name, second_name) values('ivan', 'ivanov');```

6. Подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера:
![image](https://github.com/kunishuter/postgres/assets/88041566/c3d56f3f-0c78-4604-b1e2-61a35b27b44e)
для подключения взял публичный айпи машины и порт postgres'a через dbeaver'a подключился без проблем.

7. удалить контейнер с сервером
создать его заново
подключится снова из контейнера с клиентом к контейнеру с сервером
проверить, что данные остались на месте
- *Данные пропали, есть предположение,что мапинг некорректный*
- *Проделал те же действия, но без мапинга томов - результат тот же*
- *В доке нашел информацию о дефолтном пути томов:
This optional variable can be used to define another location - like a subdirectory - for the database files. The default is /var/lib/postgresql/data. If the data volume you're using is a filesystem mountpoint (like with GCE persistent disks), or remote folder that cannot be chowned to the postgres user (like some NFS mounts), or contains folders/files (e.g. lost+found), Postgres initdb requires a subdirectory to be created within the mountpoint to contain the data.*
- *Раскатил командой:*
 ```docker run --name postgres -e POSTGRES_PASSWORD=user -d -p 5432:5432 -v /var/lib/postgresql/data:/var/lib/postgresql/data postgres```
*Проделал все тоже самое и на этот раз, данные после удаление контейнера и перераскатки - остались.*




