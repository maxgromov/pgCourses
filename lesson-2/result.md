
```
-- создаю новую таблицу
create table if not exists public.test_table
(
    user_id serial,
    amount  bigint
);
```


```
-- наполняю таблицу данными
insert into test_table(amount) values (100);
insert into test_table(amount) values (400);
```


```
-- проверяю текущий уровень изоляции: default = read committed
show transaction isolation level;
```

```
-- использую мануальное управление коммитом транзакции
-- добавляю новую запись в таблицу но не произвожу коммит
-- ожидаемый результат: во второй сессии если прочитать все строки в таблице останется также 2 строки
-- новая строка не доабвилась тк мы не закоммитили транзкцию, а в уровне изоляции read committed нельзя прочитать незакоммиченые данные
insert into test_table(amount) values (100);
-- после того как сделаем коммит первой транзакции, при повторном перезапуске второй транзакции без коммита мы увидим 3ю строку
-- потому что read committed позволяет видеть все зафиксированные изменения на момент выполнения запроса
```

```
-- меняем уровень изоляции на repeatable read
-- чекам что уровень изоляции поменялся
show transaction isolation level;

-- добавляем новую запись
-- во второй сессии не видно новой записи, тк уровень изоляции repeatable read запрещает фантомные чтения,
-- то есть мы видим состояние таблицы на момент начала вторйо транзакции (а там новая строка добавлена но еще не закоммичена)
insert into test_table(amount) values (1122);
--  после того как мы завершим 1 транзакцию коммитом, но не закоммитим 2 транзакцию сколько бы мы не запускали ее по новому
--  она будет видеть снимок данных на момент старта транзакции, то есть без новой добавленной строки, даже после коммита в 1 транзакции
--  мы увидим новую строку только после коммита/роллбэка 2 транзации и начала новой
```

