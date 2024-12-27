> Список терминов: [Атомарный (atomic)](/resources/glossary.md?id=Атомарный-atomic), [Базовый случай (base-case)](/resources/glossary.md?id=Базовый-случай-base-case), [Бесконечная рекурсия (infinite recursion)](/resources/glossary.md?id=Бесконечная-рекурсия-infinite-recursion), [Большой двоичный объект (binary large object BLOB)](/resources/glossary.md?id=Большой-двоичный-объект-binary-large-object-blob), [Временная таблица (temporary table)](/resources/glossary.md?id=Временная-таблица-temporary-table), [Денормализация (denormalization)](/resources/glossary.md?id=Денормализация-denormalization), [Долговечнось (durable)](/resources/glossary.md?id=Долговечнось-durable), [Значения разделенные запятыми (csv)](/resources/glossary.md?id=Значения-разделенные-запятыми-csv), [Изолированный (isolated)](/resources/glossary.md?id=Изолированный-isolated), [Javascript Object Notation (JSON)](/resources/glossary.md?id=javascript-object-notation-json), [Материализованное представление (materialized view)](/resources/glossary.md?id=Материализованное-представление-materialized-view), [Нормальная форма (normal form)](/resources/glossary.md?id=Нормальная-форма-normal-form), [Представление (view)](/resources/glossary.md?id=Представление-view), [Путь (path expression)](/resources/glossary.md?id=Путь-path-expression), [Рекурсивное табличное выражение (recursive cte)](/resources/glossary.md?id=Рекурсивное-табличное-выражение-recursive-cte), [Рекурсивный случай (recursive case)](/resources/glossary.md?id=Рекурсивный-случай-recursive-case), [Триггер (trigger)](/resources/glossary.md?id=Триггер-trigger), [UPSERT](/resources/glossary.md?id=UPSERT)


## Большой двоичный объект (binary large object BLOB)
```sql
-- создаём таблицу
create table images (
    name text not null,
    content blob
);

-- вставляем данные считывая содержимое файлов
insert into images (name, content) values
    ('biohazard', readfile('img/biohazard.png')),
    ('crush', readfile('img/crush.png')),
    ('fire', readfile('img/fire.png')),
    ('radioactive', readfile('img/radioactive.png')),
    ('tripping', readfile('img/tripping.png'));

-- проверяем результат
select
    name,
    length(content)
from images;
```
размер данных в поле `content` равен размеру исходного файла в байтах.
```
|    name     | length(content) |
|-------------|-----------------|
| biohazard   | 19629           |
| crush       | 15967           |
| fire        | 18699           |
| radioactive | 16661           |
| tripping    | 17208           |
```
BLOB - Большой двоичный объект - позволяет хранить в базе большие объёмы данных в двоичном формате (нпример излбражения, видео и дюбые другие данные).
Байты на входе, байты на выходе, что записали то и получили ...
Если вы считаете это странным, посмотрите на [Fossil](https://fossil-scm.org/)

#### Упражнения 

1. Измените запрос, показанный выше, чтобы выбрать значение поля `content`, а не его длину. Насколько понятен вывод? Делает ли использование функции SQLite hex() его более читабельным?

## Ещё одна база данных
```shell
sqlite3 db/lab_log.db
.schema
```
```sql
CREATE TABLE sqlite_sequence(name,seq);

CREATE TABLE person(
       ident            integer primary key autoincrement,
       details          text not null
);

CREATE TABLE machine(
       ident            integer primary key autoincrement,
       name             text not null,
       details          text not null
);

CREATE TABLE usage(
       ident            integer primary key autoincrement,
       log              text not null
);
```

## Сохраняем JSON

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/20fe4b92bd91e1579d1fbc9d4b414a33/))

```sql
select * from machine;
```
```
| ident |      name      |                         details                         |
|-------|----------------|---------------------------------------------------------|
| 1     | WY401          | {"acquired": "2023-05-01"}                              |
| 2     | Inphormex      | {"acquired": "2021-07-15", "refurbished": "2023-10-22"} |
| 3     | AutoPlate 9000 | {"note": "needs software update"}                       |
```

- Хранение разнородных данных в виде JSON-форматированного текста (со строками в двойных кавычках)
- База данных анализирует текст при каждом запросе, поэтому это может влиять на производительность
- В качестве альтернативы можно хранить как BLOB (jsonb)
  - Нельзя просматривать напрямую
  - Но более эффективно

## Выбираем данные из JSON

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/c6c045474024c57f394187190bd08bbf/))


```sql
select
    details->'$.acquired' as single_arrow,
    details->>'$.acquired' as double_arrow
from machine;
```
```
| single_arrow | double_arrow |
|--------------|--------------|
| "2023-05-01" | 2023-05-01   |
| "2021-07-15" | 2021-07-15   |
```

- Одинарная стрелка `->` возвращает JSON-представление результата.
- Двойная стрелка `->>` возвращает текст SQL, целое число, действительное число или ноль.
- Левая сторона — столбец, правая сторона — выражение пути
- Начинается с $ - root (что означает корень)
- Поля разделены `.`

#### Упражнение

1. Напишите запрос, который выбирает год из поля `refurbished` данных JSON, связанных с планшетным ридером Inphormex.

##  Доступ к элементам массива в JSON

```sql
select
    ident,
    json_array_length(log->'$') as length,
    log->'$[0]' as first
from usage;
```
```
| ident | length |                            first                             |
|-------|--------|--------------------------------------------------------------|
| 1     | 4      | {"machine":"Inphormex","person":["Gabrielle","Dub\u00e9"]}   |
| 2     | 5      | {"machine":"Inphormex","person":["Marianne","Richer"]}       |
| 3     | 2      | {"machine":"sterilizer","person":["Josette","Villeneuve"]}   |
| 4     | 1      | {"machine":"sterilizer","person":["Maude","Goulet"]}         |
| 5     | 2      | {"machine":"AutoPlate 9000","person":["Brigitte","Michaud"]} |
| 6     | 1      | {"machine":"sterilizer","person":["Marianne","Richer"]}      |
| 7     | 3      | {"machine":"WY401","person":["Maude","Goulet"]}              |
| 8     | 1      | {"machine":"AutoPlate 9000"}                                 |
```

- SQLite и другие СУБД имеют множество функций обработки JSON.
- `json_array_length` выдает количество элементов в выбранном массиве.
- Нумерация элементов массива начинаются с 0.
- Символы за пределами 7-битного ASCII представлены как экранированные символы Unicode.

## Распаковка JSON массива

```sql
select
    ident,
    json_each.key as key,
    json_each.value as value
from usage, json_each(usage.log)
limit 10;
```
```
| ident | key |                            value                             |
|-------|-----|--------------------------------------------------------------|
| 1     | 0   | {"machine":"Inphormex","person":["Gabrielle","Dub\u00e9"]}   |
| 1     | 1   | {"machine":"Inphormex","person":["Gabrielle","Dub\u00e9"]}   |
| 1     | 2   | {"machine":"WY401","person":["Gabrielle","Dub\u00e9"]}       |
| 1     | 3   | {"machine":"Inphormex","person":["Gabrielle","Dub\u00e9"]}   |
| 2     | 0   | {"machine":"Inphormex","person":["Marianne","Richer"]}       |
| 2     | 1   | {"machine":"AutoPlate 9000","person":["Marianne","Richer"]}  |
| 2     | 2   | {"machine":"sterilizer","person":["Marianne","Richer"]}      |
| 2     | 3   | {"machine":"AutoPlate 9000","person":["Monique","Marcotte"]} |
| 2     | 4   | {"machine":"sterilizer","person":["Marianne","Richer"]}      |
| 3     | 0   | {"machine":"sterilizer","person":["Josette","Villeneuve"]}   |
```

- `json_each` — еще одна табличная функция (возвращает таблицу данных).
- Используйте `json_each.name` для получения свойств распакованного массива.

#### Упражнение
1. Напишите запрос, который подсчитывает, сколько раз каждый человек появляется в первой записи журнала, связанной с любой единицей оборудования.

## Получение последнего элемента массива

```sql
select
    ident,
    log->'$[#-1].machine' as final
from usage
limit 5;
```
```
| ident |    final     |
|-------|--------------|
| 1     | "Inphormex"  |
| 2     | "sterilizer" |
| 3     | "Inphormex"  |
| 4     | "sterilizer" |
| 5     | "sterilizer" |
```

## Изменение JSON

```sql
select
    ident,
    name,
    json_set(details, '$.sold', json_quote('2024-01-25')) as updated
from machine;
```
```
| ident |      name      |                           updated                            |
|-------|----------------|--------------------------------------------------------------|
| 1     | WY401          | {"acquired":"2023-05-01","sold":"2024-01-25"}                |
| 2     | Inphormex      | {"acquired":"2021-07-15","refurbished":"2023-10-22","sold":" |
|       |                | 2024-01-25"}                                                 |
| 3     | AutoPlate 9000 | {"note":"needs software update","sold":"2024-01-25"}         |
```

- `json_set` обновляет копию JSON в памяти, а не запись базы данных
- Пожалуйста, используйте `json_quote` вместо того, чтобы пытаться форматировать JSON с помощью строковых операций.

#### Упражнение

1. В рамках очистки базы данных лабораторных журналов (таблица `usage`) измените данные в поле `log` так, чтобы заменить названия машин в записях на соответствующие идентификаторы этих машин из таблицы `machine`.

## Обновление бызы данных Penguins

```sql
select
    species,
    count(*) as num
from penguins
group by species;
```
```
|  species  | num |
|-----------|-----|
| Adelie    | 152 |
| Chinstrap | 68  |
| Gentoo    | 124 |
```

- Мы восстановим полную базу данных после каждого примера.

## Tombstones

```sql
-- добавим колонку для хранения признака 
alter table penguins
add active integer not null default 1;

-- щтметим неактивных
update penguins
set active = iif(species = 'Adelie', 0, 1);

-- посчитаем активных
select
    species,
    count(*) as num
from penguins
where active
group by species;
```
```
|  species  | num |
|-----------|-----|
| Chinstrap | 68  |
| Gentoo    | 124 |
```

- Используйте `tombstone` для обозначения (не)активных записей
- Теперь каждый запрос должен включать его в условие.

## Импорт CSV файлов

- SQLite и большинство других СУБД имеют инструменты для импорта и экспорта CSV.
- Процесс импорта в SQLite:
    1. Определение таблицы
    2. Импорт данных
    3. Преобразование пустых строк в значения NULL (при нелбходимости).
    4. Преобразование типов из текста в любые (не показано ниже).

```sql
drop table if exists penguins;

.mode csv penguins
.import misc/penguins.csv penguins

-- конвертируем пустые строки в null
update penguins set species = null where species = '';
update penguins set island = null where island = '';
update penguins set bill_length_mm = null where bill_length_mm = '';
update penguins set bill_depth_mm = null where bill_depth_mm = '';
update penguins set flipper_length_mm = null where flipper_length_mm = '';
update penguins set body_mass_g = null where body_mass_g = '';
update penguins set sex = null where sex = '';
```

#### Упражнение

1. Каковы будут типы данных столбцов в таблице `penguins`, созданной с помощью импорта CSV, показанного выше? 
2. Как можно исправить те, которые требуют исправления?

## Представления (views)

```sql
create view if not exists
active_penguins (
    species,
    island,
    bill_length_mm,
    bill_depth_mm,
    flipper_length_mm,
    body_mass_g,
    sex
) as
select
    species,
    island,
    bill_length_mm,
    bill_depth_mm,
    flipper_length_mm,
    body_mass_g,
    sex
from penguins
where active;

select
    species,
    count(*) as num
from active_penguins;
group by species;
```
```
|  species  | num |
|-----------|-----|
| Chinstrap | 68  |
| Gentoo    | 124 |
```

- Представление — это сохраненный запрос, который могут вызывать другие запросы.
- Представление вычисляется каждый раз при использовании.
- Похоже на CTE, но:
    - Может совместно использоваться запросами.
    - Представления в SQL появились первыми. CTE относительно новый инструмент.
    - Некоторые СУБД предлагают материализованные представления - временные таблицы с обновлением по требованию.

#### Упражнение

1. Создайте представление в базе данных журнала лаборатории под названием `busy` с двумя столбцами: `machine_id` и `total_log_length`. В первом столбце пердставьте числовой идентификатор каждой машины; во втором - общее количество записей журнала для этой машины.

### Проверка знаний

<img src="./assets/advanced_temp_concept_map.svg" alt="Проверка знаний" style="max-width:100%; height:auto;">


## Напоминание о часах

```sql
-- создаём таблицу
create table job (
    name text not null,
    billable real not null
);
-- вносим данные
insert into job values
    ('Калибровка', 1.5),
    ('Очистка', 0.5);
-- проверяем результат
select * from job;
```
```
| name       | billable |
|------------|----------|
| Калибровка | 1.5      |
| Очистка    | 0.5      |
```

## Добавляем ограничения

```sql
-- создаём таблицу с ограничением
create table job (
    name text not null,
    billable real not null,
    check (billable > 0.0)
);

-- вносим данные
insert into job values ('Калибровка', 1.5);
insert into job values ('Очистка', -0.5);

-- проверяем результат
select * from job;
```

```
CHECK constraint failed: billable > 0.0

| name       | billable |
|------------|----------|
| Калибровка | 1.5      |
```
- получаем ошибку
- строка не прошедшая проверку не внесена в таблицу

- Функция `check` добавляет ограничение в таблицу:
    - Должна выдавать результат типа `boolean`.
    - Выполняется каждый раз, когда значения добавляются или изменяются.
    - Но изменения, внесенные до ошибки, вступают в силу.

#### Упражнение

1. Перепишите определение таблицы `penguins`, добавив следующие ограничения:
    - `body_mass_g` должен быть неотрицательным.
    - `island` должен быть одним из "Biscoe", "Dream" или "Torgersen". (Подсказка: оператор `in` здесь будет полезен.)

## ACID (Atomicity, Consistency, Isolation, Durability)

ACID — это набор свойств транзакций базы данных, гарантирующих надежность и целостность данных при операциях с ними. Аббревиатура ACID расшифровывается как:

- Atomicity (Атомарность): гарантирует, что транзакция будет выполнена целиком или не будет выполнена вовсе. Недопустимы частичные изменения данных. Если в процессе выполнения транзакции происходит ошибка, все изменения, которые успели произойти, отменяются, и база данных возвращается в исходное состояние. Это похоже на неделимую операцию: либо всё, либо ничего.
- Consistency (Согласованность/Непротиворечивость): гарантирует, что транзакция переводит базу данных из одного согласованного состояния в другое. Согласованное состояние означает, что данные в базе соответствуют всем заданным правилам и ограничениям (например, ограничениям целостности, уникальности и т. д.). Если транзакция нарушает какое-либо из этих правил, она отменяется.
- Isolation (Изолированность): гарантирует, что параллельные транзакции не влияют друг на друга. Каждая транзакция выполняется так, как будто она является единственной, работающей с базой данных в данный момент. Это предотвращает возникновение ошибок, связанных с одновременным доступом и изменением данных разными транзакциями. Существуют различные уровни изоляции, определяющие степень защиты от таких ошибок.
- Durability (Стойкость/Надежность): гарантирует, что результаты успешно завершенной транзакции сохраняются в базе данных и не будут потеряны в случае сбоя системы (например, отключения питания). Обычно это достигается путем записи изменений на энергонезависимый носитель (например, жесткий диск).

Простыми словами: ACID гарантирует, что любая операция с базой данных будет выполнена надежно и безопасно. Если вы, например, переводите деньги с одного счета на другой, ACID гарантирует, что деньги будут списаны с первого счета и зачислены на второй, и что эта операция не будет прервана посередине из-за какой-либо ошибки, а так же что результаты этой операции будут сохранены и не потеряются.

## Транзакции

```sql
create table job (
    name text not null,
    billable real not null,
    check (billable > 0.0)
);

insert into job values ('Калибровка', 1.5);

begin transaction;
insert into job values ('Очистка', 0.5);
rollback;

select * from job;
```
```
| name       | billable |
|------------|----------|
| Калибровка | 1.5      |
```

- Операторы вне транзакции выполняются и фиксируются в базе немедленно.
- Операторы внутри транзакции не вступают в силу до тех пор, пока транзакция не будет полностью и успешно завершена или отменена.
- В транзакции может быть любое количество операторов.
- Но в SQLite нельзя вкладывать транзакции. (Другие СУБД поддерживают это.)

## Откат в результате ограничения

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/72a1ca04957cfe43afd251b552587723/))

```sql
create table job (
    name text not null,
    billable real not null,
    check (billable > 0.0) on conflict rollback
);

insert into job values
    ('Калибровка', 1.5);

insert into job values
    ('Очистка', 0.5),
    ('Сброс', -0.5);

select * from job;
```
```
CHECK constraint failed: billable > 0.0 (19)

| name       | billable |
|------------|----------|
| Калибровка | 1.5      |
```

Второй insert откатился сразу после возникновения ошибки, но первый insert вступил в силу.

## Откат в запросах

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/7e288b6fec3ad8b9239cb36c19bb5f5b/))

```sql
create table job (
    name text not null,
    billable real not null,
    check (billable > 0.0)
);

insert or rollback into job values
    ('Калибровка', 1.5);

insert or rollback into job values
    ('Очистка', 0.5),
    ('reset', -0.5);

select * from job;
```
```
CHECK constraint failed: billable > 0.0 (19)

| name       | billable |
|------------|----------|
| Калибровка | 1.5      |
```

- Ограничения включаются в определение таблицы.
- Действия включаются в оператор.

## Upsert

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/495404bbccf6adc8bbb3b466393cf99b/))

```sql
-- создаем таблицу
create table jobs_done (
    person text unique,
    num integer default 0
);
-- вносим данные
insert into jobs_done values
    ('Антон', 1);
-- проверяем результат
select * from jobs_done;
```
```
| person | num |
|--------|-----|
| zia    | 1   |
```
```sql
-- вносим данные
insert into jobs_done values
    ('Антон', 1);
-- получаем ошибку
-- проверяем результат
select * from jobs_done;
```
```
UNIQUE constraint failed: jobs_done.person (19)

| person | num |
|--------|-----|
| zia    | 1   |
```
```sql
-- вносим данные используя upsert
insert into jobs_done values
    ('Антон', 1)
on conflict(person) do update set num = num + 1;

-- ошибка не возникла
-- проверяем результат (данные обновились)
select * from jobs_done;
```

```
| person | num |
|--------|-----|
| zia    | 2   |
```

- upsert расшифровывается как "обновить или вставить" (update or insert).
- Создает запись, если ее не существует.
- Обновляет, если существует.
- Не является стандартом SQL, но широко используется.

#### Упражнение
1. Используя базу данных журнала лаборатории, создайте таблицу с именем `staff` с тремя столбцами: `personal`, `family` и `dept`. Столбец `personal` должен быть уникальным. Затем используйте оператор upsert, чтобы добавить или изменить людей в таблице `staff`, как показано:
    ```
    personal    family  dept    age
    Pranay      Khanna  mb	    41
    Riaan       Dua     gen	    23
    Parth	    Johel   gen	    27
    ``` 

## Нормализация

First normal form (1NF): every field of every record contains one indivisible value.
Second normal form (2NF) and third normal form (3NF): every value in a record that isn't a key depends solely on the key, not on other values.
Denormalization: explicitly store values that could be calculated on the fly to simplify queries and/or make processing faster

## Создание триггеров

```sql
-- Track hours of lab work.
create table job (
    person text not null,
    reported real not null check (reported >= 0.0)
);

-- Explicitly store per-person total rather than using sum().
create table total (
    person text unique not null,
    hours real
);

-- Initialize totals.
insert into total values
    ('gene', 0.0),
    ('august', 0.0);

-- Define a trigger.
create trigger total_trigger
before insert on job
begin
    -- Check that the person exists.
    select case
        when not exists (select 1 from total where person = new.person)
        then raise(rollback, 'Unknown person ')
    end;
    -- Update their total hours (or fail if non-negative constraint violated).
    update total
    set hours = hours + new.reported
    where total.person = new.person;
end;
```

A trigger automatically runs before or after a specified operation
Can have side effects (e.g., update some other table)
And/or implement checks (e.g., make sure other records exist)
Add processing overhead…
…but data is either cheap or correct, never both
Inside trigger, refer to old and new versions of record as old.column and new.column

## Триггер не срабатывает

```sql
insert into job values
    ('gene', 1.5),
    ('august', 0.5),
    ('gene', 1.0);
```
```
| person | reported |
|--------|----------|
| gene   | 1.5      |
| august | 0.5      |
| gene   | 1.0      |

| person | hours |
|--------|-------|
| gene   | 2.5   |
| august | 0.5   |
```

## Срабатывание триггера

```sql
insert into job values
    ('gene', 1.0),
    ('august', -1.0);
```
```
Runtime error near line 6: CHECK constraint failed: reported >= 0.0 (19)

| person | hours |
|--------|-------|
| gene   | 0.0   |
| august | 0.0   |
```

#### Упражнение
1. Using the penguins database:
    - create a table called species with columns name and count; and
    - define a trigger that increments the count associated with each species each time a new penguin is added to the penguins table.
    - Does your solution behave correctly when several penguins are added by a single insert statement?
1. Использование базы данных пингвинов:
- создайте таблицу с именем `species` со столбцами `name` и `count` и
- определите триггер, который увеличивает счетчик, связанный с каждым видом, каждый раз, когда в таблицу penguins добавляется новый пингвин.
- Правильно ли ведет себя ваше решение, когда несколько пингвинов добавляются одним оператором вставки?

## Представление графов

```sql
create table lineage (
    parent text not null,
    child text not null
);

insert into lineage values
    ('Arturo', 'Clemente'),
    ('Darío', 'Clemente'),
    ('Clemente', 'Homero'),
    ('Clemente', 'Ivonne'),
    ('Ivonne', 'Lourdes'),
    ('Soledad', 'Lourdes'),
    ('Lourdes', 'Santiago');

select * from lineage;
```
```
|  parent  |  child   |
|----------|----------|
| Arturo   | Clemente |
| Darío    | Clemente |
| Clemente | Homero   |
| Clemente | Ivonne   |
| Ivonne   | Lourdes  |
| Soledad  | Lourdes  |
| Lourdes  | Santiago |
```

-  Диаграмма родословной
<img src="./assets/advanced_recursive_lineage.svg" alt="box and arrow diagram showing who is descended from whom in the lineage database" style="max-width:100%; height:auto;">


#### Упражнение
1. Напишите запрос, который использует самосоединение (self join) для поиска внуков каждого человека.

## Рекурсивные запросы

```sql
with recursive descendent as (
    select
        'Clemente' as person,
        0 as generations
    union all
    select
        lineage.child as person,
        descendent.generations + 1 as generations
    from descendent inner join lineage
        on descendent.person = lineage.parent
)
select
    person,
    generations
from descendent;
```
```
|  person  | generations |
|----------|-------------|
| Clemente | 0           |
| Homero   | 1           |
| Ivonne   | 1           |
| Lourdes  | 2           |
| Santiago | 3           |
```

Use a recursive CTE to create a temporary table (descendent)
Base case seeds this table
Recursive case relies on value(s) already in that table and external table(s)
union all to combine rows
Can use union but that has lower performance (must check uniqueness each time)
Stops when the recursive case yields an empty row set (nothing new to add)
Then select the desired values from the CTE
Используйте рекурсивное CTE для создания временной таблицы (потомка)
Базовый случай задает эту таблицу
Рекурсивный случай опирается на значения, уже находящиеся в этой таблице, и внешние таблицы
union all для объединения строк
Можно использовать union, но это имеет более низкую производительность (необходимо проверять уникальность каждый раз)
Останавливается, когда рекурсивный случай выдает пустой набор строк (нечего нового добавлять)
Затем выберите нужные значения из CTE

#### Упражнение

1. Измените рекурсивный запрос, показанный выше, чтобы использовать `union` вместо `union all`. 
2. Влияет ли это на результат? Почему или почему нет?

## База данных отслеживания контактов

```sql
select * from person;
```
```
| ident |         name          |
|-------|-----------------------|
| 1     | Juana Baeza           |
| 2     | Agustín Rodríquez     |
| 3     | Ariadna Caraballo     |
| 4     | Micaela Laboy         |
| 5     | Verónica Altamirano   |
| 6     | Reina Rivero          |
| 7     | Elias Merino          |
| 8     | Minerva Guerrero      |
| 9     | Mauro Balderas        |
| 10    | Pilar Alarcón         |
| 11    | Daniela Menéndez      |
| 12    | Marco Antonio Barrera |
| 13    | Cristal Soliz         |
| 14    | Bernardo Narváez      |
| 15    | Óscar Barrios         |
```
```sql
select * from contact;
```
```
|       left        |         right         |
|-------------------|-----------------------|
| Agustín Rodríquez | Ariadna Caraballo     |
| Agustín Rodríquez | Verónica Altamirano   |
| Juana Baeza       | Verónica Altamirano   |
| Juana Baeza       | Micaela Laboy         |
| Pilar Alarcón     | Reina Rivero          |
| Cristal Soliz     | Marco Antonio Barrera |
| Cristal Soliz     | Daniela Menéndez      |
| Daniela Menéndez  | Marco Antonio Barrera |
```

- Contact Diagram (box and line diagram showing who has had contact with whom)
<img src="./assets/advanced_recursive_contacts.svg" alt="box and line diagram showing who has had contact with whom" style="max-width:100%; height:auto;">

## Двунаправленные контакты

```sql
create temporary table bi_contact (
    left text,
    right text
);

insert into bi_contact
select
    left, right from contact
    union all
    select right, left from contact
;
```
```
| original_count |
|----------------|
| 8              |

| num_contact |
|-------------|
| 16          |
```

Create a temporary table rather than using a long chain of CTEs
Only lasts as long as the session (not saved to disk)
Duplicate information rather than writing more complicated query
- Создайте временную таблицу вместо использования длинной цепочки CTE
- Действует только до тех пор, пока длится сеанс (не сохраняется на диске)
- Дублируйте информацию вместо написания более сложного запроса

## Обновление идентификаторов групп

```sql
select
    left.name as left_name,
    left.ident as left_ident,
    right.name as right_name,
    right.ident as right_ident,
    min(left.ident, right.ident) as new_ident
from
    (person as left join bi_contact on left.name = bi_contact.left)
    join person as right on bi_contact.right = right.name;
```
```
|       left_name       | left_ident |      right_name       | right_ident | new_ident |
|-----------------------|------------|-----------------------|-------------|-----------|
| Juana Baeza           | 1          | Micaela Laboy         | 4           | 1         |
| Juana Baeza           | 1          | Verónica Altamirano   | 5           | 1         |
| Agustín Rodríquez     | 2          | Ariadna Caraballo     | 3           | 2         |
| Agustín Rodríquez     | 2          | Verónica Altamirano   | 5           | 2         |
| Ariadna Caraballo     | 3          | Agustín Rodríquez     | 2           | 2         |
| Micaela Laboy         | 4          | Juana Baeza           | 1           | 1         |
| Verónica Altamirano   | 5          | Agustín Rodríquez     | 2           | 2         |
| Verónica Altamirano   | 5          | Juana Baeza           | 1           | 1         |
| Reina Rivero          | 6          | Pilar Alarcón         | 10          | 6         |
| Pilar Alarcón         | 10         | Reina Rivero          | 6           | 6         |
| Daniela Menéndez      | 11         | Cristal Soliz         | 13          | 11        |
| Daniela Menéndez      | 11         | Marco Antonio Barrera | 12          | 11        |
| Marco Antonio Barrera | 12         | Cristal Soliz         | 13          | 12        |
| Marco Antonio Barrera | 12         | Daniela Menéndez      | 11          | 11        |
| Cristal Soliz         | 13         | Daniela Menéndez      | 11          | 11        |
| Cristal Soliz         | 13         | Marco Antonio Barrera | 12          | 12        |
```

- `new_ident` - минимум собственного идентификатора и идентификаторов на один шаг дальше
- Не содержит людей без контактов

## Рекурсивная маркировка

```sql
with recursive labeled as (
    select
        person.name as name,
        person.ident as label
    from
        person
    union -- not 'union all'
    select
        person.name as name,
        labeled.label as label
    from
        (person join bi_contact on person.name = bi_contact.left)
        join labeled on bi_contact.right = labeled.name
    where labeled.label < person.ident
)
select name, min(label) as group_id
from labeled
group by name
order by label, name;
```
```
|         name          | group_id |
|-----------------------|----------|
| Agustín Rodríquez     | 1        |
| Ariadna Caraballo     | 1        |
| Juana Baeza           | 1        |
| Micaela Laboy         | 1        |
| Verónica Altamirano   | 1        |
| Pilar Alarcón         | 6        |
| Reina Rivero          | 6        |
| Elias Merino          | 7        |
| Minerva Guerrero      | 8        |
| Mauro Balderas        | 9        |
| Cristal Soliz         | 11       |
| Daniela Menéndez      | 11       |
| Marco Antonio Barrera | 11       |
| Bernardo Narváez      | 14       |
| Óscar Barrios         | 15       |
```

Используйте `union` вместо `union all`, чтобы предотвратить бесконечную рекурсию.

#### Упражнение

1. Измените запрос выше, чтобы использовать `union all` вместо `union` для запуска бесконечной рекурсии. 
2. Как можно изменить запрос так, чтобы он останавливался на определенной глубине, чтобы можно было проследить его вывод?

Check Understanding
<img src="./assets/advanced_cte_concept_map.svg" alt="box and arrow diagram showing concepts related to common table expressions in SQL" style="max-width:100%; height:auto;">

Концептуальная карта: общие табличные выражения (CTE)