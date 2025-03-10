> Список терминов: [Базовый случай (base-case)](/resources/glossary.md?id=Базовый-случай-base-case), [Бесконечная рекурсия (infinite recursion)](/resources/glossary.md?id=Бесконечная-рекурсия-infinite-recursion), [Большой двоичный объект (binary large object BLOB)](/resources/glossary.md?id=Большой-двоичный-объект-binary-large-object-blob), [Временная таблица (temporary table)](/resources/glossary.md?id=Временная-таблица-temporary-table), [Javascript Object Notation (JSON)](/resources/glossary.md?id=javascript-object-notation-json), [Материализованное представление (materialized view)](/resources/glossary.md?id=Материализованное-представление-materialized-view), [Представление (view)](/resources/glossary.md?id=Представление-view), [Путь (path expression)](/resources/glossary.md?id=Путь-path-expression), [Рекурсивное табличное выражение (recursive cte)](/resources/glossary.md?id=Рекурсивное-табличное-выражение-recursive-cte), [Рекурсивный случай (recursive case)](/resources/glossary.md?id=Рекурсивный-случай-recursive-case), [Триггер (trigger)](/resources/glossary.md?id=Триггер-trigger), [UPSERT](/resources/glossary.md?id=UPSERT)


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

## JSON
JSON (JavaScript Object Notation) стал популярным форматом для хранения и обмена данными, и его поддержка в реляционных базах данных значительно расширяет их возможности. С помощью JSON можно хранить полуструктурированные данные, которые не соответствуют традиционной табличной структуре. Это позволяет гибко работать с данными, которые могут часто меняться или иметь сложную структуру, например, конфигурации, логи или данные от внешних API.


### Ещё одна база данных

Для примеров в этом разделе мы будем использовать базу данных, которая отслеживает использование лабораторного оборудования. Вот её схема:

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

### Сохраняем JSON

SQLite позволяет хранить JSON в текстовом формате. Это удобно, но может быть неэффективно при поиске или фильтрации данных. Другие СУБД предоставояют специальные типы данных, такие как `json` или `jsonb`, которые предоставляют более эффективные методы доступа к данным.

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/20fe4b92bd91e1579d1fbc9d4b414a33/))

```sql
-- выгружаем все данные из таблицы machine
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

### Извлекаем данные из JSON

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/792567c791ee59f606c582dd3ceb9330/))


```sql
select
    name,
    json_extract(details, '$.acquired') as acquired,
    json_extract(details, '$.refurbished') as refurbished
from machine;
```
```
|----------------|------------|-------------|
| name           | acquired   | refurbished |
|----------------|------------|-------------|
| WY401          | 2023-05-01 | [null]      |
| Inphormex      | 2021-07-15 | 2023-10-22  |
| AutoPlate 9000 | [null]     | [null]      |
```

- Функция `json_extract` извлекает данные из JSON-поля.
- Первый аргумент — это JSON-поле, второй — путь к данным в JSON.
- Путь к данным в JSON начинается с `$`, за которым следует путь к данным в JSON-структуре.
- Если данные по указанному пути отсутствуют, возвращается `null`.
- Другие СУБД могут предоставлять другие функции для работы с JSON.

#### Упражнение

1. Напишите запрос, который выбирает год из поля `refurbished` данных JSON, связанных с планшетным ридером Inphormex.

###  Доступ к элементам массива в JSON

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/17bf208dae8a17afede50071b668c7b0/))

```sql
select
    ident,
    json_array_length(json_extract(log, '$')) as length,
    json_extract(log, '$[0]') as first
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

### Распаковка JSON массива

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/7bca29af605c38ce823485a723255ff1/))

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

### Получение последнего элемента массива

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/0aed5d4a1f6529b3159e8c67ecee4ede/))

```sql
select
    ident,
    json_extract(log, '$[#-1].machine') as final
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

### Изменение JSON

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/7abbd976b79871d57a67cdc4279c27e3/))

```sql
select
    ident,
    name,
    json_set(details, '$.sold', json_quote('2024-01-25')) as updated
from machine;
```
```
|-------|----------------|--------------------------------------------------------------------------|
| ident | name           | updated                                                                  |
|-------|----------------|--------------------------------------------------------------------------|
| 1     | WY401          | {"acquired":"2023-05-01","sold":"2024-01-25"}                            |
| 2     | Inphormex      | {"acquired":"2021-07-15","refurbished":"2023-10-22","sold":"2024-01-25"} |
| 3     | AutoPlate 9000 | {"note":"needs software update","sold":"2024-01-25"}                     |
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
    ('Сброс', -0.5);

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
| Антон  | 1   |
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
| Антон  | 1   |
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
| Антон  | 2   |
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

## Создание триггеров

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/b113b9038a7bbe90e5a05be8a31158e0/))

```sql
-- создаем таблицу учёта рабочего времени
create table job (
    person text not null,
    reported real not null check (reported >= 0.0)
);

-- создаем таблицу с общим временем работы
create table total (
    person text unique not null,
    hours real
);

-- создаем триггер для обновления таблицы `total`
create trigger total_trigger
before insert on job
begin
    insert into total values
        (new.person, new.reported)
    on conflict(person) do update set hours = hours + new.reported;
end;
```

- Триггер — это автоматически выполняемый код, который реагирует на изменения в базе данных.
- Триггеры могут выполняться до или после операции.
- Триггеры могут выполняться на вставку, обновление или удаление.
- Триггеры могут быть использованы для проверки данных, обновления связанных таблиц и многого другого.
- Триггеры могут быть отключены, включены или удалены.
- Триггеры могут быть созданы в любой момент, в любой таблице и в любой базе данных.
- Триггеры могут быть созданы в любой СУБД, но синтаксис может отличаться.
- Внутри триггера можнообращаться к старой и новой версии записи как old.column и new.column.

## Срабатывание триггера
```sql
-- вносим данные
insert into job values
    ('Олег', 1.5),
    ('Сергей', 0.5),
    ('Олег', 1.0);

-- проверяем результат
select * from job;

select * from total;
```
```
| person | reported |
|--------|----------|
| Олег   | 1.5      |
| Сергей | 0.5      |
| Олег   | 1.0      |

| person | hours |
|--------|-------|
| Олег   | 2.5   |
| Сергей | 0.5   |
```
- данные в таблице `total` обновились автоматически.

## Триггер не срабатывает

```sql
-- вносим данные
insert into job values
    ('Олег', 1.0),
    ('Сергей', -1.0);

-- проверяем результат
select * from job;

select * from total;
```
```
CHECK constraint failed: reported >= 0.0 (19)

| person | hours |
|--------|-------|
| Олег   | 0.0   |
| Сергей | 0.0   |
```
- данные не внесены в таблицу `total` из-за нарушения ограничения.

#### Упражнение

1. Использование базы данных пингвинов:
- создайте таблицу с именем `species` со столбцами `name` и `count` и
- определите триггер, который увеличивает счетчик, связанный с каждым видом, каждый раз, когда в таблицу penguins добавляется новый пингвин.
- Правильно ли ведет себя ваше решение, когда несколько пингвинов добавляются одним оператором вставки?

## Представление графов

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/914fe4ac2fb3b316223c79608bcf4f71/))

```sql
-- создаем таблицу родословной
create table lineage (
    parent text not null,
    child text not null
);
-- вносим данные
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

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/393c66e0a9580d88a89f9ef8ed4474d1/))

```sql
with recursive descendent as (
    select
        'Clemente' as person,
        0 as generations
    union all
    select
        lineage.child as person,
        descendent.generations + 1 as generations
    from descendent 
    inner join lineage
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

- Используйте рекурсивное CTE для создания временной таблицы (потомка).
- Базовый случай задает эту таблицу.
- Рекурсивный случай опирается на значения, уже находящиеся в этой таблице, и внешние таблицы.
- Используйте `union all` для объединения строк
  *Можно использовать `union`, но это имеет более низкую производительность (необходимо проверять уникальность каждый раз)*
- Рекурсия останавливается, когда рекурсивный случай выдает пустой набор строк (нечего нового добавлять).
- Затем выберите нужные значения из CTE

#### Упражнение

1. Измените рекурсивный запрос, показанный выше, чтобы использовать `union` вместо `union all`. 
2. Влияет ли это на результат? Почему или почему нет?

## База данных отслеживания контактов

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/7ce208fe11ac9be3fbbb4105304bc5c6/))

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

**Диаграмма контактов (диаграмма с прямоугольниками и линиями, показывающая, кто с кем контактировал)**
<img 
    src="./assets/advanced_recursive_contacts.svg" 
    alt="Диаграмма контактов (диаграмма с прямоугольниками и линиями, показывающая, кто с кем контактировал)" 
    style="max-width:100%; height:auto;">

## Временные таблицы

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/67d427000f1c354777f8701e548a0896/))

```sql
-- создаем временную таблицу
create temporary table bi_contact (
    left text,
    right text
);

-- вносим данные во временную таблицу на основе данных из таблицы `contact`
insert into bi_contact
select
    left, right from contact
    union all
    select right, left from contact
;
-- проверяем результат (данные дублированы)
select count(*) original_count from contact;

select count(*) num_bi_contacts from bi_contact;
```
```
| original_count |
|----------------|
| 8              |

| num_bi_contacts |
|-----------------|
| 16              |
```

- Создайте временную таблицу вместо использования длинной цепочки CTE.
- Временная таблица существует только до тех пор, пока длится сеанс (не сохраняется на диске).
- Дублируйте информацию вместо написания более сложного запроса.

## Обновление идентификаторов групп

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/9359987da54942c6acf20c431520ea1f/))

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

- Используйте `union` вместо `union all`, чтобы предотвратить бесконечную рекурсию.

#### Упражнение

1. Измените запрос выше, чтобы использовать `union all` вместо `union` для запуска бесконечной рекурсии. 
2. Как можно изменить запрос так, чтобы он останавливался на определенной глубине, чтобы можно было проследить его вывод?

**Концептуальная карта: общие табличные выражения (CTE)**
<img 
    src="./assets/advanced_cte_concept_map.svg" 
    alt="Концептуальная карта: общие табличные выражения (CTE)" 
    style="max-width:100%; height:auto;">

