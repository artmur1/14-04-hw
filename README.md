# Домашнее задание к занятию 4. "`PostgreSQL`" - `Мурчин Артем`

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
- подключения к БД,
- вывода списка таблиц,
- вывода описания содержимого таблиц,
- выхода из psql.

### Решение 1

    \l+ - показать список БД
    \conninfo - подключения к БД,
    \dt+ - показать список таблиц в БД
    \dS+ - вывода описания содержимого таблиц,
    \q - выход из psql

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

### Решение 2

Столбец title имеет наибольшее среднее значение размера элементов в байтах: avg_width=16. У остальных столбцов avg_width=4.

    select * from pg_stats where tablename = 'orders' and attname = 'title';

![alt text](https://github.com/artmur1/14-04-hw/blob/main/14-04-hw-2-1.png)

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

### Решение 3

SQL-транзакция для разбиение таблицы orders на 2

    BEGIN;
    CREATE TABLE orders_1 AS SELECT * FROM orders;
    DELETE FROM orders_1 WHERE true;
    CREATE TABLE orders_2 AS SELECT * FROM orders;
    DELETE FROM orders_2 WHERE true;
    INSERT INTO orders_1  SELECT * FROM orders where price > 499;
    INSERT INTO orders_2  SELECT * FROM orders where price <= 499;
    COMMIT;

Можно создать разделы, в которые в зависимости от цены будут записываться данные. В первый раздел запишем price<=499, во второй с price>499 до price<=999 и тд. Для создания разделов мы можем использовать встроенную поддержку секционирования в PostgreSQL.

## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

### Решение 4

    pg_dump -d test_database -U user -h 192.168.1.124 -p 25432 > /tmp/test_database.dump

Чтобы добавить уникальность значения столбца `title` для таблиц `test_database` можно использовать Unique constraint.

    CREATE TABLE public.orders (
        id integer NOT NULL,
        title character varying(80) NOT NULL UNIQUE,
        price integer DEFAULT 0
    );

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

