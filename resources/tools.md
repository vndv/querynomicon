# Инструменты

> Список терминов: [1-to-1 relation](glossary.md#отношение-один-к-одному-1-to-1-relation), [1-to-many relation](glossary.md#отношение-один-ко-многим-1-to-many-relation), [alias](glossary.md#псевдоним-alias), [autoincrement](glossary.md#автоинкремент-autoincrement), [B-tree](glossary.md#бинарное-дерево-b-tree), [common table expression (CTE)](glossary.md#общее-табличное-выражение-common-table-expression-cte), [correlated subquery](glossary.md#коррелированный-подзапрос-correlated-subquery), [data migration](glossary.md#мигрция-данных-data-migration), [entity-relationship diagram](glossary.md#диаграмма-сущность-связь-entity-relationship-diagram), [expression](glossary.md#выражение-expression), [foreign key](glossary.md#внешний-ключ-foreign-key), [index](glossary.md#индекс-index), [join table](glossary.md#соединение-join), [many-to-many relation](glossary.md#отношение-многие-ко-многим--many-to-many-relation), [primary key](glossary.md#первичный-ключ-primary-key), [statement](glossary.md#оператор-statement), [subquery](glossary.md#подзапрос-subquery), [table-valued function](glossary.md#табличная-функция-table-valued-function), [vectorization](glossary.md#векторизация-vectorization), [window function](glossary.md#оконная-функция-window-function)


### Неправильное отрицание

- Кто не калибрует?

```sql
select distinct person
from work
where job != 'Калибровка';
```
```
| person  |
|---------|
| Иван    |
| Евгений |
| Слава   |
```

- Но Иван калибрует!
- Проблема в том, что есть запись о том что Иван занимается очисткой.
- А поскольку `'Очистка' != 'Калибровка'`, эта строка включается в результаты.
- Нам нужен другой подход…

### Указать вхождение

```sql
select *
from work
where person not in ('Иван', 'Слава');
```
```
| person | job        |
|--------|------------|
| Иван   | Очистка    |
| Иван   | Сортировка |
```

- Оператор вхождения `in` и не вхождения `not in` делают именно то, что нам нужно

### Подзапросы

```sql
select distinct person
from work
where person not in (
    select distinct person
    from work
    where job = 'Калибровка'
);
```
```
| person  |
|---------|
| Евгений |
| Слава   |
```

1. Используйте подзапрос, чтобы выбрать людей, которые выполняют калибровку
2. Затем выберите всех людей, которых нет в этом наборе.
3. Поначалу кажется странным, но подзапросы полезны и в других отношениях.

### Первичный ключ (primary key)

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/fdc77f8d8b8c45b4275a88250491f395/))

1. Можно использовать любое поле (или комбинацию полей) в таблице в качестве первичного ключа, если значения уникальны для каждой записи.
2. Уникально идентифицирует конкретную запись в конкретной таблице.

```sql
create table lab_equipment (
    size  decimal not null,
    color text not null,
    num   integer not null,
    primary key (size, color)
);

insert into lab_equipment values
    (1.5, 'blue', 2),
    (1.5, 'green', 1),
    (2.5, 'blue', 1);

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
insert into lab_equipment values
    (1.5, 'green', 2);
```

```
**UNIQUE constraint failed: lab_equipment.size, lab_equipment.color**
```

#### Упражнения

1. Есть ли у таблицы `penguins` первичный ключ? Если да, то что это? А как насчет таблиц `work` и `job`?


### Автоинкремент и первичные ключи

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/44784ebcf722e3a8de6c89f58db7a88a/))

```sql
create table person (
    ident integer primary key autoincrement,
    name text not null
);

insert into person values
    (null, 'Иван'),
    (null, 'Евгений'),
    (null, 'Слава');

select * from person;
```
```
| ident | name    |
|-------|---------|
| 1     | Иван    |
| 2     | Евгений |
| 3     | Слава   |
```
```sql
insert into person values (1, 'prevented');
```
```
**UNIQUE constraint failed: person.ident**
```
1. Автоинкремент автоматически увеличивается для каждой вставленной записи
2. Обычно это поле используется в качестве первичного ключа, уникального для каждой записи.
3. Если Слава снова сменит имя, нам останется изменить только один факт в базе данных.
4. Недостаток: ручные запросы труднее читать (кто такой человек 17?)

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
alter table job
add ident integer not null default -1;

update job
set ident = 1
where name = 'Калибровка';

update job
set ident = 2
where name = 'Очистка';

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

