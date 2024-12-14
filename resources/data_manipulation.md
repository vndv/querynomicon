# Управление данными в базе

## Создание новых таблиц из старых

```sql
create table new_work (
    person_id integer not null,
    job_id integer not null,
    foreign key (person_id) references person (ident),
    foreign key (job_id) references job (ident)
);

insert into new_work
select
    person.ident as person_id,
    job.ident as job_id
from person 
inner join work on person.name = work.person
inner join job on job.name = work.job;

select * from new_work;
```
```
| person_id | job_id |
|-----------|--------|
| 1         | 1      |
| 1         | 2      |
| 2         | 2      |
```
1. `new_work` - это новая таблица созданная на основе старой
2. Каждый столбец ссылается на запись в какой-либо другой таблице.

## Удаление таблиц

```sql
drop table work;

alter table new_work rename to work;

create table job (
    ident integer primary key autoincrement,
    name text not null,
    billable real not null
);

create table person (
    ident integer primary key autoincrement,
    name text not null
);

create table IF NOT EXISTS "work" (
    person_id integer not null,
    job_id integer not null,
    foreign key(person_id) references person(ident),
    foreign key(job_id) references job(ident)
);
```
1. Удалите старую таблицу и переименуйте новую, чтобы она заняла ее место.
2. Пожалуйста, сначала сделайте резервную копию ваших данных
3. Не забывайте про условие `IF NOT EXISTS`

### Упражнение

1. Реорганизуйте базу данных пингвинов:
    1. Сделайте копию файла penguins.db, чтобы ваши изменения не повлияли на оригинал.
    2. Напишите скрипт SQL, который реорганизует данные в три таблицы: по одной для каждого острова.
    3. Почему такая организация данных — плохая идея?

2. Такие инструменты, как [Sqitch](https://sqitch.org), могут управлять изменениями в схемах и данных базы данных, чтобы их можно было сохранить в системе контроля версий и откатить в случае неудачи. Переведите изменения, внесенные приведенными выше скриптами, в Sqitch. Примечание: это упражнение может занять час или больше.

### Системные таблицы

```sql
select * from sqlite_sequence;
```
```
| name   | seq |
|--------|-----|
| person | 3   |
```

1. Порядковые номера не сбрасываются при удалении строк.
2. Частично для того, чтобы их можно было использовать в качестве первичных ключей.

#### Упражнение

Можете ли вы изменить значения, хранящиеся в sqlite_sequence? В частности, можете ли вы сбросить значения, чтобы снова генерировались те же порядковые номера?

### Изменение таблиц

```sql
-- добавляем колонку к таблице  
alter table job
add ident integer not null default -1; -- не пустая, знечение по умолчанию -1 

-- обновляем данные в таблице
update job
set ident = 1
where name = 'Калибровка';

update job
set ident = 2
where name = 'Очистка';

-- проверяем результат
select * from job;
```
```
| name       | billable | ident |
|------------|----------|-------|
| Калибровка | 1.5      | 1     |
| Очистка    | 0.5      | 2     |
```

1. Добавить столбец постфактум.
2. Поскольку оно не может быть нулевым, мы должны указать значение по умолчанию.
3. Хочется сделать его первичным ключом, но SQLite не позволяет этого сделать постфактум.
4. Затем используйте обновление для изменения существующих записей.
5. Можно изменить любое количество записей одновременно
6. Так что будьте осторожны с условием `where`

## Индексы

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/fbe5b2d4189e05d804bd58a4d422fec1/))

Индекс - объект базы данных, создаваемый с целью повышения производительности поиска данных.
Таблицы в базе данных могут иметь большое количество строк, которые хранятся в произвольном порядке, и их поиск по заданному критерию путём последовательного просмотра таблицы строка за строкой может занимать много времени.
Индекс формируется из значений одного или нескольких столбцов таблицы и указателей на соответствующие строки таблицы, таким образом, позволяет быстрее находить строки, удовлетворяющие критерию поиска.
Ускорение работы с использованием индексов достигается в первую очередь за счёт того, что индекс имеет структуру, оптимизированную под поиск — например, сбалансированного дерева.

Посмотрим как выполняется запрос к базе данных. Воспользуемся для этого командой `explain query plan`:

```sql
explain query plan
select *
from penguins
where island = 'Biscoe';
```
Для выполнения такого запроса без индекса СУБД должна проверить значение в поле `island` в каждой строке таблицы (этот механизм известен как «полный перебор» или «полное сканирование таблицы» (`full scan`)).

```
| id | parent | notused | detail              |
|----|--------|---------|---------------------|
| 2  | 0      | 0       | SCAN TABLE penguins |
```

### Создание индекса

```sql
create index island_ix on penguins(island);

explain query plan
select *
from penguins
where island = 'Biscoe';
```
```
| id | parent | notused | detail                                                 |
|----|--------|---------|--------------------------------------------------------|
| 3  | 0      | 0       | SEARCH TABLE penguins USING INDEX island_ix (island=?) |
```

Можно убедиться что план выполнения запроса изменился. Вместо полного перебора `SCAN TABLE` - поиск по индексу `SEARCH TABLE penguins USING INDEX`,  что в случае больших таблиц даёт существенный пророст призводитеьности.

### Ограниченная применимость индекса

```sql
explain query plan
select *
from penguins
where island != 'Biscoe';
```
```
| id | parent | notused | detail              |
|----|--------|---------|---------------------|
| 2  | 0      | 0       | SCAN TABLE penguins |
```

Этот запрос должен нам найти всех пингвинов, кроме тех что проживают на острове 'Biscoe', однако не смотря на то что по столбцу `island` есть индекс, СУБД всё равно будет использовать полный перебор таблицы.
Использование неравенства в условии поиска исключает для СУБД возможность использования поиска по B-дереву.
Если запросы такого типа критичны для вас, то задача может быть решена созданием специальных (функциональных) индексов. Так как не все СУБД поддерживают такой функционал мы не будем углубятся здесь в эту тему.

### Удаление индекса

```sql
drop index island_ix;
```

#### Упражнения

1. Создайте индекс в таблице `penguins` над столбцами `species` и `sex`. 
2. Какой запрос позволит вам воспользоваться преимуществами этого индекса?
3. Удалите созданный вами индекс.

### Закрепление

- Индекс — это вспомогательная структура данных, обеспечивающая более быстрый доступ к записям.
- Занимает место на диске и замедляет операции вставки (`insert`) и изменения данных (`update`), взамен индекс может значительно ускорять операции поиска данных в таблице (`select`).
- Следует следить за использованием индексов и удалять ненужные.
- Индексы могут создаваться и удаляться в любое время.

## Генерация последовательностей

```sql
select value from generate_series(1, 5);
```
```
| value |
|-------|
| 1     |
| 2     |
| 3     |
| 4     |
| 5     |
```

### Генерация последовательностей на основе данных

```sql
create table temp (
    num integer not null
);
insert into temp values (1), (5);
select value from generate_series (
    (select min(num) from temp),
    (select max(num) from temp)
);
```
```
| value |
|-------|
| 1     |
| 2     |
| 3     |
| 4     |
| 5     |
```
- Должны быть круглые скобки вокруг минимального и максимального значений, чтобы SQLite работал нормально.

### Генерация последовательности дат

```sql
select date((select julianday(min(started)) from experiment) + value) as some_day
from (
    select value from generate_series(
        (select 0),
        (select julianday(max(started)) - julianday(min(started)) from experiment)
    )
)
limit 5;
```
```
|  some_day  |
|------------|
| 2023-01-29 |
| 2023-01-30 |
| 2023-01-31 |
| 2023-02-01 |
| 2023-02-02 |
```

1. SQLite представляет даты в виде строк ГГГГ-ММ-ДД, юлианских дней, миллисекунд Unix 
2. Юлианские дни — дробное количество дней, прошедших с 24 ноября 4714 г. до н. э.
3. `julianday` и `date` конвертируются туда и обратно
4. Другие базы данных имеют свои собственные функции обработки дат.

### Подсчет начатых экспериментов за день без пропусков

```sql
with
-- complete sequence of days with 0 as placeholder for number of experiments
all_days as (
    select
        date((select julianday(min(started)) from experiment) + value) as some_day,
        0 as zeroes
    from (
        select value from generate_series(
            (select 0),
            (select count(*) - 1 from experiment)
        )
    )
),

-- sequence of actual days with actual number of experiments started
actual_days as (
    select
        started,
        count(started) as num_exp
    from experiment
    group by started
)

-- combined by joining on day and taking actual number (if available) or zero
select
    all_days.some_day as day,
    coalesce(actual_days.num_exp, all_days.zeroes) as num_exp
from
    all_days left join actual_days on all_days.some_day = actual_days.started
limit 5;
```
```
|    day     | num_exp |
|------------|---------|
| 2023-01-29 | 1       |
| 2023-01-30 | 1       |
| 2023-01-31 | 0       |
| 2023-02-01 | 0       |
| 2023-02-02 | 1       |
```

#### Упражнение

Что вернет выражение  `date('now', 'start of month', '+1 month', '-1 day')`? (Вам может оказаться полезной документация по функциям даты и времени SQLite.)

## Первичный ключ (primary key)

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/fdc77f8d8b8c45b4275a88250491f395/))

1. Можно использовать любое поле (или комбинацию полей) в таблице в качестве первичного ключа, если значения уникальны для каждой записи.
2. Уникально идентифицирует конкретную запись в конкретной таблице.

```sql
-- создаём таблицу
create table lab_equipment (
    -- определяем поля
    size  decimal not null,
    color text not null,
    num   integer not null,
    -- создаём первичный ключ на основе полей size и color
    primary key (size, color)
);

-- вставляем записи в таблицу
insert into lab_equipment values
    (1.5, 'blue', 2),
    (1.5, 'green', 1),
    (2.5, 'blue', 1);

-- проверяем результат
select * from lab_equipment;
```
```
| size | color | num |
|------|-------|-----|
| 1.5  | blue  | 2   |
| 1.5  | green | 1   |
| 2.5  | blue  | 1   |
```
```sql
-- пробуем вставить запись со значением первичного ключа уже существующего в таблице
insert into lab_equipment values
    (1.5, 'green', 2);
```

Закономерно получаем ошибку!

```
UNIQUE constraint failed: lab_equipment.size, lab_equipment.color
```

#### Упражнения

1. Есть ли у таблицы `penguins` первичный ключ? Если да, то что это? А как насчет таблиц `work` и `job`?


### Автоинкремент и первичные ключи

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/44784ebcf722e3a8de6c89f58db7a88a/))

```sql
-- создаём таблицу
create table person (
    ident integer primary key autoincrement, -- определяем поле как первичный ключ и автоинкремент
    name text not null
);

-- вставляем записи в таблицу
insert into person values
    (null, 'Иван'),
    (null, 'Евгений'),
    (null, 'Слава');

-- проверяем результат
select * from person;
```
Поле ident автоматически получает инкрементальные значения
```
| ident | name    |
|-------|---------|
| 1     | Иван    |
| 2     | Евгений |
| 3     | Слава   |
```
```sql
-- пробуем вставить запись с уже существующим значением инкремента
insert into person values (1, 'prevented');
```
И снова получаем ошибку!
```
UNIQUE constraint failed: person.ident
```
1. Автоинкремент автоматически увеличивается для каждой вставленной записи
2. Обычно это поле используется в качестве первичного ключа, уникального для каждой записи.
3. Если Слава снова сменит имя, нам останется изменить только один факт в базе данных.
4. Недостаток: ручные запросы труднее читать (кто такой человек 17?)

## Внешние ключи