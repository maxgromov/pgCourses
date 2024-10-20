## Задание
1. Создать таблицу accounts(id integer, amount numeric);
2. Добавить несколько записей и подключившись через 2 терминала добиться
ситуации взаимоблокировки (deadlock).
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

### 1.Создаем таблицу и добавляем данные
```sql
CREATE TABLE accounts (
    id INTEGER PRIMARY KEY,
    amount NUMERIC
);

-- Добавление нескольких записей
INSERT INTO accounts (id, amount) VALUES (1, 100);
INSERT INTO accounts (id, amount) VALUES (2, 200);
```

### 2.Делаем обновление строк в 2 параллельных транзакциях, пытаясь обновить уже заблокированные строки из другой транзакции
Делаем 2 терминала с ручным управлением транзакциями, стартуем транзакции без коммита и отката.

**Терминал 1**
```sql
-- шаг 1
UPDATE accounts SET amount = amount + 20 WHERE id = 2;
-- шаг 2
UPDATE accounts SET amount = amount + 20 WHERE id = 1;
```

**Терминал 2**
```sql
-- шаг 1
UPDATE accounts SET amount = amount + 20 WHERE id = 1;
-- шаг 2
UPDATE accounts SET amount = amount + 20 WHERE id = 2;
```

В этот момент возникнет взаимоблокировка (deadlock), так как обе транзакции пытаются получить доступ к строкам, заблокированным другой транзакцией.

### 3. Итоговый результат:

<img width="571" alt="image" src="https://github.com/user-attachments/assets/95823d24-7107-4bfb-a101-972aee230481">

*[40P01] ERROR: deadlock detected Detail: Process 3628 waits for ShareLock on transaction 1086; blocked by process 3627. 
Process 3627 waits for ShareLock on transaction 1087; blocked by process 3628. 
Hint: See server log for query details. Where: while updating tuple (0,1) in relation "accounts"*
