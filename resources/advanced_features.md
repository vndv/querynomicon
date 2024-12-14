> Список терминов: [Атомарный (atomic)](/resources/glossary.md?id=Атомарный-atomic), [Базовый случай (base-case)](/resources/glossary.md?id=Базовый-случай-base-case), [Бесконечная рекурсия (infinite recursion)](/resources/glossary.md?id=Бесконечная-рекурсия-infinite-recursion), [Большой двоичный объект (binary large object BLOB)](/resources/glossary.md?id=Большой-двоичный-объект-binary-large-object-blob), [Временная таблица (temporary table)](/resources/glossary.md?id=Временная-таблица-temporary-table), [Денормализация (denormalization)](/resources/glossary.md?id=Денормализация-denormalization), [Долговечнось (durable)](/resources/glossary.md?id=Долговечнось-durable), [Значения разделенные запятыми (csv)](/resources/glossary.md?id=Значения-разделенные-запятыми-csv), [Изолированный (isolated)](/resources/glossary.md?id=Изолированный-isolated), [Javascript Object Notation (JSON)](/resources/glossary.md?id=javascript-object-notation-json), [Материализованное представление (materialized view)](/resources/glossary.md?id=Материализованное-представление-materialized-view), [Нормальная форма (normal form)](/resources/glossary.md?id=Нормальная-форма-normal-form), [Представление (view)](/resources/glossary.md?id=Представление-view), [Путь (path expression)](/resources/glossary.md?id=Путь-path-expression), [Рекурсивное табличное выражение (recursive cte)](/resources/glossary.md?id=Рекурсивное-табличное-выражение-recursive-cte), [Рекурсивный случай (recursive case)](/resources/glossary.md?id=Рекурсивный-случай-recursive-case), [Триггер (trigger)](/resources/glossary.md?id=Триггер-trigger), [UPSERT](/resources/glossary.md?id=UPSERT)


Большой двоичный объект (binary large object BLOB)
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