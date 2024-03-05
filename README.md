# Механизм блокировок
1. Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
   - ```alter system set log_lock_waits to 'on';```
   - ```alter system set deadlock_timeout to '200ms';```
   - ```select pg_reload_conf();```
   - Создание таблицы:```CREATE TABLE accounts(acc_no integer PRIMARY KEY, amount numeric);``` из этой же сессии блокирую таблицу ```begin;lock table accounts;```, из второй сессии выполняю ту же команду, она зависла т.к. блокировка будет висеть до тех пор пока удерживается 1 сессия. Дождался времени, чтобы сработала блокировка.
в логах по пути ```/var/log/postgresql/postgresql-14-main.log```, видим что отработало ```postgres@postgres LOG:  process 5882 still waiting for AccessExclusiveLock on relation 16384 of database 13761 after 200.137 ms```

2. Смоделируйте ситуацию обновления одной и той же строки тремя командами ```UPDATE``` в разных сеансах. Изучите возникшие блокировки в представлении ```pg_locks``` и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
- создание представления
```
CREATE VIEW locks_v AS
SELECT pid,
       locktype,
       CASE locktype
         WHEN 'relation' THEN relation::regclass::text
         WHEN 'transactionid' THEN transactionid::text
         WHEN 'tuple' THEN relation::regclass::text||':'||tuple::text
       END AS lockid,
       mode,
       granted
FROM pg_locks
WHERE locktype in ('relation','transactionid','tuple')
AND (locktype != 'relation' OR relation = 'accounts'::regclass);
```

- Теперь начнем первую транзакцию и обновим строку.

Session #1
```
BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
SELECT * FROM locks_v WHERE pid = 6206; 
```
locktype: relation,transactionid - удерживает блокировку таблицы и собственного номера

Session #2
```
sudo -u postgres psql
\c locks
```
Начинаем вторую транзакцию и пытаемся обновить ту же строку.
```
BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
```
locktype: relation,transactionid - удерживает блокировку таблицы и собственного номера, также появилась блокировка turple - блокировка, конкретной строки
Session #3

```
sudo -u postgres psql
\c locks
BEGIN;
SELECT txid_current(), pg_backend_pid();
UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
```
тут произошла блокировка версии строки и повисла сразу же.

3. Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

Даже, если мы возьмем задание 2, в журнале мы уже увидим какие транзакции были заблокированы и какими транзакциями они были заблокированы, также мы можем определить последовательность действий, соответствено, если бы действия из задания 2 делали не мы, то по журналу мы смогли бы разобраться в проблеме, понять какая первая транзакция является блокирующей. Также сможем по конкретной сессии посмотреть блокировки.

4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
Да могут, тк обе транзакции пытаются удержать эксклюзивную блокировку на всей таблице, чтобы выполнить апдейт. Когда одна транзакция начинает апдейт таблицы, она запрашивает эксклюзивную блокировку. Если в это же время другая транзакция также пытается выполнить апдейт, она также будет запрашивать эксклюзивную блокировку на всю таблицу. В результате возникает ситуация взаимной блокировки (deadlock), когда обе транзакции ожидают освобождения блокировки, удерживаемой другой транзакцией.
