1. Установка wsl на windpws
   ```wsl --install```
   
2. Установка postgres на локальной машине
   ```sudo apt update```
   ```sudo apt install postgresql postgresql-contrib```
   ```sudo systemctl start postgresql.service```
   
3. Подключение к базе
   ```sudo -i -u postgres psql```
   
4. Создание базы
   ```CREATE DATABASE postgres;```
   
5. Создание таблицы и ее наполнение
   ```create table persons(id serial, first_name text, second_name text);```
   ```insert into persons(first_name, second_name) values('ivan', 'ivanov');```
   ```insert into persons(first_name, second_name) values('petr', 'petrov');```
   ```commit;```
   
6. Посмотреть текущий уровень изоляции:
   ```show transaction isolation level;```
   
7. В первой сессии добавить новую запись:
  ```insert into persons(first_name, second_name) values('sergey', 'sergeev');```
  сделать select from persons во второй сессии
  видите ли вы новую запись и если да то почему?
  - *Read Committed — уровень изоляции транзакции, выбираемый в Postgres Pro по умолчанию. В транзакции, работающей на этом уровне, запрос SELECT (без предложения FOR UPDATE/SHARE) видит только те данные, которые были зафиксированы до начала запроса, поэтому мы не видим добавленные в первой сессии данные.*

8. завершить первую транзакцию - commit;
сделать ```select * from persons``` во второй сессии
видите ли вы новую запись и если да то почему?
- *Теперь данные отображаются потому, что они были закоммичены.*

9. Начать новые но уже repeatable read транзации - ```set transaction isolation level repeatable read;```(предварительно начать транзакцию)

10. В первой сессии добавить новую запись ```insert into persons(first_name, second_name) values('sveta', 'svetova');```
сделать ```select * from``` persons во второй сессии
видите ли вы новую запись и если да то почему?
- *В режиме Repeatable Read видны только те данные, которые были зафиксированы до начала транзакции, но не видны незафиксированные данные и изменения, произведённые другими транзакциями в процессе выполнения данной транзакции, по этой причине данные не отображаются*

11. Завершить вторую транзакцию
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
- *Изменения были внесены после начала транзакции по этой причине мы не видим отображение данных.*
