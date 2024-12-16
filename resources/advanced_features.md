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