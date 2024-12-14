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

## Подзапросы

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


### Сравнение отдельных значений с агрегатами

```sql
select body_mass_g
from penguins
where
    body_mass_g > (
        select avg(body_mass_g)
        from penguins
    )
limit 5;
```
```
| body_mass_g |
|-------------|
| 4675.0      |
| 4250.0      |
| 4400.0      |
| 4500.0      |
| 4650.0      |
```

1. Получить среднюю массу тела в подзапросе
2. Сравните каждую строку с данными в подзапросе
3. Требуется два сканирования данных, но избежать этого невозможно.
  - За исключением расчета промежуточной суммы каждый раз, когда в таблицу добавляется пингвин.
4. `Null` значения не включаются в среднее или окончательные результаты.

#### Упражнения 
Используйте подзапрос, чтобы найти количество пингвинов, которые весят столько же, сколько самый легкий пингвин.

### Сравнение отдельных значений с агрегатами внутри групп

```sql
select
    penguins.species,
    penguins.body_mass_g,
    round(averaged.avg_mass_g, 1) as avg_mass_g
from penguins inner join (
    select
        species,
        avg(body_mass_g) as avg_mass_g
    from penguins
    group by species
) as averaged
    on penguins.species = averaged.species
where penguins.body_mass_g > averaged.avg_mass_g
limit 5;
```
```
| species | body_mass_g | avg_mass_g |
|---------|-------------|------------|
| Adelie  | 3750.0      | 3700.7     |
| Adelie  | 3800.0      | 3700.7     |
| Adelie  | 4675.0      | 3700.7     |
| Adelie  | 4250.0      | 3700.7     |
| Adelie  | 3800.0      | 3700.7     |
```

1. Сначала запускается подзапрос для создания временной таблицы, усредненной по средней массе по видам.
2. Затем внутреннее соеденение с таблицей `penguins`
3. Фильтр для поиска пингвинов, вес которых превышает средний вес представителей своего вида.

#### Упражнение 
Используйте подзапрос, чтобы найти количество пингвинов, которые весят столько же, сколько самый легкий пингвин того же пола и вида.


### Вхождение и Коррелированные подзапросы

```sql
select
    name,
    building
from department
where
    exists (
        select 1
        from staff
        where dept = department.ident
    )
order by name;
```
```
|       name        |     building     |
|-------------------|------------------|
| Genetics          | Chesson          |
| Histology         | Fashet Extension |
| Molecular Biology | Chesson          |
```

1. Эндокринология отсутствует в списке.
2. `select 1` в равной степени может быть выбрано `true` или любое другое значение
3. Коррелированный подзапрос зависит от значения из внешнего запроса.
 - Эквивалент вложенного цикла

 ### Невхождение

```sql
 select
    name,
    building
from department
where
    not exists (
        select 1
        from staff
        where dept = department.ident
    )
order by name;
```
```
|     name      | building |
|---------------|----------|
| Endocrinology | TGVH     |
```

#### Упражнение

Можете ли вы переписать предыдущий запрос, используя исключение? Если да, то легче ли понять ваш новый запрос? Если запрос нельзя переписать, почему бы и нет?

### Как избежать коррелирующих подзапросов

```sql
select distinct
    department.name as name,
    department.building as building
from department inner join staff
    on department.ident = staff.dept
order by name;
```
```
|       name        |     building     |
|-------------------|------------------|
| Genetics          | Chesson          |
| Histology         | Fashet Extension |
| Molecular Biology | Chesson          |
```

1. Соединение может быть быстрее, а может и нет чем коррелированный подзапрос.


## Общие табличные выражения  (Common Table Expression, CTE)

```sql
with grouped as (
    select
        species,
        avg(body_mass_g) as avg_mass_g
    from penguins
    group by species
)

select
    penguins.species,
    penguins.body_mass_g,
    round(grouped.avg_mass_g, 1) as avg_mass_g
from penguins inner join grouped
where penguins.body_mass_g > grouped.avg_mass_g
limit 5;
```
```
| species | body_mass_g | avg_mass_g |
|---------|-------------|------------|
| Adelie  | 3750.0      | 3700.7     |
| Adelie  | 3800.0      | 3700.7     |
| Adelie  | 4675.0      | 3700.7     |
| Adelie  | 4250.0      | 3700.7     |
| Adelie  | 3800.0      | 3700.7     |
```

1. Используйте общее табличное выражение (CTE), чтобы сделать запросы более понятными.
2. Вложенные подзапросы быстро становятся трудными для понимания.
3. База данных решает, как оптимизировать запрос


## Оконные функции

```sql
with ym_num as (
    select
        strftime('%Y-%m', started) as ym,
        count(*) as num
    from experiment
    group by ym
)

select
    ym,
    num,
    sum(num) over (order by ym) as num_done,
    (sum(num) over (order by ym) * 1.00) / (select sum(num) from ym_num) as completed_progress, 
    cume_dist() over (order by ym) as linear_progress
from ym_num
order by ym;
```
```
|   ym    | num | num_done | completed_progress |  linear_progress   |
|---------|-----|----------|--------------------|--------------------|
| 2023-01 | 2   | 2        | 0.04               | 0.0769230769230769 |
| 2023-02 | 5   | 7        | 0.14               | 0.153846153846154  |
| 2023-03 | 5   | 12       | 0.24               | 0.230769230769231  |
| 2023-04 | 1   | 13       | 0.26               | 0.307692307692308  |
| 2023-05 | 6   | 19       | 0.38               | 0.384615384615385  |
| 2023-06 | 5   | 24       | 0.48               | 0.461538461538462  |
| 2023-07 | 3   | 27       | 0.54               | 0.538461538461538  |
| 2023-08 | 2   | 29       | 0.58               | 0.615384615384615  |
| 2023-09 | 4   | 33       | 0.66               | 0.692307692307692  |
| 2023-10 | 6   | 39       | 0.78               | 0.769230769230769  |
| 2023-12 | 4   | 43       | 0.86               | 0.846153846153846  |
| 2024-01 | 5   | 48       | 0.96               | 0.923076923076923  |
| 2024-02 | 2   | 50       | 1.0                | 1.0                |
```

1. `sum()` выполняет промежуточный итог
2. `cume_dist()` — это доля просмотренных на данный момент строк.
3. Итак, столбец num_done — это количество проведенных экспериментов…
4. Completed_progress — это доля выполненных экспериментов…
5. а Linear_progress — это доля пройденного времени.

### Опережение и отставание (LEAD and LAG)

```sql
with ym_num as (
    select
        strftime('%Y-%m', started) as ym,
        count(*) as num
    from experiment
    group by ym
)

select
    ym,
    lag(num) over (order by ym) as prev_num,
    num,
    lead(num) over (order by ym) as next_num
from ym_num
order by ym;
```
```
|   ym    | prev_num | num | next_num |
|---------|----------|-----|----------|
| 2023-01 |          | 2   | 5        |
| 2023-02 | 2        | 5   | 5        |
| 2023-03 | 5        | 5   | 1        |
| 2023-04 | 5        | 1   | 6        |
| 2023-05 | 1        | 6   | 5        |
| 2023-06 | 6        | 5   | 3        |
| 2023-07 | 5        | 3   | 2        |
| 2023-08 | 3        | 2   | 4        |
| 2023-09 | 2        | 4   | 6        |
| 2023-10 | 4        | 6   | 4        |
| 2023-12 | 6        | 4   | 5        |
| 2024-01 | 4        | 5   | 2        |
| 2024-02 | 5        | 2   |          |
```

1. Используйте strftime для извлечения года и месяца
2. Обработка даты/времени не является сильной стороной SQLite.
3. Используйте оконные функции `lead` и `lag` для смещения значений.
4. Недоступные значения вверху или внизу равны `null`.

### Границ (Boundaries)

1. Документация по оконным функциям SQLite описывает три типа фреймов и пять видов границ фреймов.
2. Каждый тип нужно использовать по ситуации

### Партицированые Окна (Partitioned Windows)

```sql
with y_m_num as (
    select
        strftime('%Y', started) as year,
        strftime('%m', started) as month,
        count(*) as num
    from experiment
    group by year, month
)

select
    year,
    month,
    num,
    sum(num) over (partition by year order by month) as num_done
from y_m_num
order by year, month;
```
```
| year | month | num | num_done |
|------|-------|-----|----------|
| 2023 | 01    | 2   | 2        |
| 2023 | 02    | 5   | 7        |
| 2023 | 03    | 5   | 12       |
| 2023 | 04    | 1   | 13       |
| 2023 | 05    | 6   | 19       |
| 2023 | 06    | 5   | 24       |
| 2023 | 07    | 3   | 27       |
| 2023 | 08    | 2   | 29       |
| 2023 | 09    | 4   | 33       |
| 2023 | 10    | 6   | 39       |
| 2023 | 12    | 4   | 43       |
| 2024 | 01    | 5   | 5        |
| 2024 | 02    | 2   | 7        |
```

1. Партиции создаются по группам
2. Таким образом, эксперименты начинаются с начала каждого года.

#### Упражнение 

Создайте запрос, который:
1. Находит уникальный вес пингвинов в базе данных `penguins`;
2. Отсортируйте их 
3. Найдите разницу между каждым последующим отдельным весом
4. Подсчитайте, сколько раз появляется каждое уникальное различие.

### Объяснение плана запроса

```sql
explain query plan
with ym_num as (
    select
        strftime('%Y-%m', started) as ym,
        count(*) as num
    from experiment
    group by ym
)
select
    ym,
    num,
    sum(num) over (order by ym) as num_done,
    cume_dist() over (order by ym) as progress
from ym_num
order by ym;
```
```
QUERY PLAN
|--CO-ROUTINE (subquery-3)
|  |--CO-ROUTINE (subquery-4)
|  |  |--CO-ROUTINE ym_num
|  |  |  |--SCAN experiment
|  |  |  `--USE TEMP B-TREE FOR GROUP BY
|  |  |--SCAN ym_num
|  |  `--USE TEMP B-TREE FOR ORDER BY
|  `--SCAN (subquery-4)
`--SCAN (subquery-3)
```

- Становится полезным… в конце концов



### Объяснение другого плана запроса

```sql
explain query plan
select
    species,
    avg(body_mass_g)
from penguins
group by species;
```
```
QUERY PLAN
|--SCAN penguins
`--USE TEMP B-TREE FOR GROUP BY
```

1. SQLite планирует сканировать каждую строку таблицы
2. Он создаст временную структуру данных B-дерева для группировки строк.

#### Упражнение 
Используйте подзапрос, чтобы найти количество пингвинов, которые весят столько же, сколько самый легкий пингвин того же пола и вида.

### Перечисление строк

- В каждой таблице есть специальный столбец, называемый `rowid`.

```sql
select
    rowid,
    species,
    island
from penguins
limit 5;
```
```
| rowid | species |  island   |
|-------|---------|-----------|
| 1     | Adelie  | Torgersen |
| 2     | Adelie  | Torgersen |
| 3     | Adelie  | Torgersen |
| 4     | Adelie  | Torgersen |
| 5     | Adelie  | Torgersen |
```

1. `rowid` постоянен в течение сеанса
2. То есть, если мы удалим первые 5 строк, у нас теперь будут идентификаторы строк 6…N.
3. Не полагайтесь на идентификатор строки, в частности, не используйте его в качестве ключа.

#### Упражнение

Чтобы узнать, как ведут себя идентификаторы строк

1. Предположим, вы создаете новую таблицу, добавляете три строки, удаляете эти строки и снова добавляете те же значения. Ожидаете ли вы, что идентификаторы последних строк будут 1–3 или 4–6?
2. Используя базу данных в памяти, выполните действия, описанные в части 1. Был ли результат ожидаемым?

### Условия

```sql
with sized_penguins as (
    select
        species,
        iif(
            body_mass_g < 3500,
            'small',
            'large'
        ) as size
    from penguins
    where body_mass_g is not null
)

select
    species,
    size,
    count(*) as num
from sized_penguins
group by species, size
order by species, num;
```
```
|  species  | size  | num |
|-----------|-------|-----|
| Adelie    | small | 54  |
| Adelie    | large | 97  |
| Chinstrap | small | 17  |
| Chinstrap | large | 51  |
| Gentoo    | large | 123 |
```

1. iif(condition, true_result, false_result)
  - **Примечание**: iif с двумя i
2. Может показаться странным думать об if/else как о функции, но это часто встречается в векторизованных вычислениях.

#### Упражнение


Как изменится результат предыдущего запроса, если убрать проверку на нулевую массу тела? Почему результат без этой проверки вводит в заблуждение?

Что дает каждое из приведенных ниже выражений? Как вы думаете, какие из них на самом деле пытаются разделить на ноль?

1. `iif(0, 123, 1/0)`
2. `iif(1, 123, 1/0)`
3. `iif(0, 1/0, 123)`
4. `iif(1, 1/0, 123)`

### Работа с CASE WHEN

- Что, если нам нужны маленькие, средние и большие пингвины?
- Условие `CASE` может вкладываться в `iif`, но быстро становится нечитаемым

```sql
with sized_penguins as (
    select
        species,
        case
            when body_mass_g < 3500 then 'small'
            when body_mass_g < 5000 then 'medium'
            else 'large'
        end as size
    from penguins
    where body_mass_g is not null
)

select
    species,
    size,
    count(*) as num
from sized_penguins
group by species, size
order by species, num;
```
```
|  species  |  size  | num |
|-----------|--------|-----|
| Adelie    | small  | 54  |
| Adelie    | medium | 97  |
| Chinstrap | small  | 17  |
| Chinstrap | medium | 51  |
| Gentoo    | medium | 56  |
| Gentoo    | large  | 67  |
```

1. Сравнивает условия и выбирает первое подходящее
2. Результат `CASE` равен нулю, если ни одно условие не истинно
3. Используйте `else` в случае поумолчанию

#### Упражнения

Измените приведенный выше запрос так, чтобы выходные данные были «пингвин маленький» и «пингвин большой», объединив строку «пингвин есть» со всем регистром, а не с отдельными ответвлениями. (Это упражнение показывает, что `CASE/WHEN` является выражением, а не утверждением.)

### Проверка диапазона

```sql
with sized_penguins as (
    select
        species,
        case
            when body_mass_g between 3500 and 5000 then 'normal'
            else 'abnormal'
        end as size
    from penguins
    where body_mass_g is not null
)

select
    species,
    size,
    count(*) as num
from sized_penguins
group by species, size
order by species, num;
```
```
|  species  |   size   | num |
|-----------|----------|-----|
| Adelie    | abnormal | 54  |
| Adelie    | normal   | 97  |
| Chinstrap | abnormal | 17  |
| Chinstrap | normal   | 51  |
| Gentoo    | abnormal | 61  |
| Gentoo    | normal   | 62  |
```

1. `BETWEEN` может облегчить чтение запросов
2. Будьте внимательны с когда указываете диапазон в `BETWEEN`

#### Упражнения

Выражение val между «A» и «Z» истинно, если val равно «M» (верхний регистр), но ложно, если val равно «m» (нижний регистр). Перепишите выражение, используя встроенные скалярные функции SQLite, чтобы оно было истинным в обоих случаях.



|**name**	|**purpose**|
|-----------|-----------|
|substr	    |Get substring given starting point and length
|trim	    |Remove characters from beginning and end of string
|ltrim	    |Remove characters from beginning of string
|rtrim	    |Remove characters from end of string
|length	    |Length of string
|replace    |Replace occurrences of substring with another string
|upper	    |Return upper-case version of string
|lower	    |Return lower-case version of string
|instr	    |Find location of first occurrence of substring (returns 0 if not found)


## Еще одна база данных

1. Диаграмма сущность-связь (диаграмма ER) показывает связи между таблицами.
2. Как и все, что связано с базами данных, существует множество вариаций связи таблиц.


<img src="./assets/tools_assays_tables.svg" alt="Описание" style="max-width:100%; height:auto;">

```sql
select * from staff;
```
```
| ident | personal |  family   | dept | age |
|-------|----------|-----------|------|-----|
| 1     | Kartik   | Gupta     |      | 46  |
| 2     | Divit    | Dhaliwal  | hist | 34  |
| 3     | Indrans  | Sridhar   | mb   | 47  |
| 4     | Pranay   | Khanna    | mb   | 51  |
| 5     | Riaan    | Dua       |      | 23  |
| 6     | Vedika   | Rout      | hist | 45  |
| 7     | Abram    | Chokshi   | gen  | 23  |
| 8     | Romil    | Kapoor    | hist | 38  |
| 9     | Ishaan   | Ramaswamy | mb   | 35  |
| 10    | Nitya    | Lal       | gen  | 52  |
```

#### Упражнение

Нарисуйте табличную диаграмму и диаграмму ER, чтобы представить следующую базу данных:

1. person есть `id` и `full_name`
2. course есть `id` и `name`
3. section есть `course_id`, `start_date`, и `end_date`
4. instructor есть `person_id` и `section_id`
5. student есть `person_id`, `section_id`, и `status`

### Сопоставление с образцом

```sql
select
    personal,
    family
from staff
where personal like '%ya%';
```
```
| personal | family |
|----------|--------|
| Nitya    | Lal    |
```

1. `like` это исходное средство сопоставления шаблонов SQL
2. `%` соответствует нулю или более символам в начале или конце строки
3. По умолчанию нечувствителен к регистру
4. `glob` поддерживает подстановочные знаки в стиле Unix

#### Упражнение

Перепишите показанный выше запрос на сопоставление шаблонов, используя `glob`.


### Выбор первой и последней строк

```sql
select * from (
    select * from (select * from experiment order by started asc limit 5)
    union all
    select * from (select * from experiment order by started desc limit 5)
)
order by started asc;
```
```
| ident |    kind     |  started   |   ended    |
|-------|-------------|------------|------------|
| 17    | trial       | 2023-01-29 | 2023-01-30 |
| 35    | calibration | 2023-01-30 | 2023-01-30 |
| 36    | trial       | 2023-02-02 | 2023-02-03 |
| 25    | trial       | 2023-02-12 | 2023-02-14 |
| 2     | calibration | 2023-02-14 | 2023-02-14 |
| 40    | calibration | 2024-01-21 | 2024-01-21 |
| 12    | trial       | 2024-01-26 | 2024-01-28 |
| 44    | trial       | 2024-01-27 | 2024-01-29 |
| 34    | trial       | 2024-02-01 | 2024-02-02 |
| 14    | calibration | 2024-02-03 | 2024-02-03 |
```

1. `union all` объединяет строки
2. Сохраняет дубликаты: `union` само по себе сохраняет только уникальные записи.

#### Упражнение

Напишите запрос, результат которого включает две строки для каждого пингвина Адели в базе данных пингвинов. Как вы можете проверить, что ваш запрос работает правильно?

### Пересечение

```sql
select
    personal,
    family,
    dept,
    age
from staff
where dept = 'mb'
intersect
select
    personal,
    family,
    dept,
    age from staff
where age < 50;
```
```
| personal |  family   | dept | age |
|----------|-----------|------|-----|
| Indrans  | Sridhar   | mb   | 47  |
| Ishaan   | Ramaswamy | mb   | 35  |
```

1. Используемые строки должны иметь одинаковую структуру.
2. Пересечение обычно используется при получении значений из разных источников.
3. В приведенном выше запросе было бы понятнее использовать, `where`

#### Упражнение

Используйте пересечение, чтобы найти всех пингвинов Адели весом более 4000 граммов. Как вы можете проверить, что ваш запрос работает правильно?

Используйте план запроса(`explain`) , чтобы сравнить только что написанный запрос на основе пересечения с запросом, в котором используется ключевое слово `where`. Какой запрос выглядит более эффективным? Почему вы этому верите?

### Исключение

```sql
select
    personal,
    family,
    dept,
    age
from staff
where dept = 'mb'
except
    select
        personal,
        family,
        dept,
        age from staff
    where age < 50;
```
```
| personal | family | dept | age |
|----------|--------|------|-----|
| Pranay   | Khanna | mb   | 51  |
```

1. Опять же, таблицы должны иметь одинаковую структуру.
2. Было бы понятнее написать запрос с `where`
3. SQL работает с множествами, а не с таблицами, за исключением тех случаев, когда это не так.

#### Упражнение

Используйте «исключение», чтобы найти всех пингвинов Gentoo, кроме самцов. Как вы можете проверить, что ваш запрос работает правильно?

### Случайные числа

```sql
with decorated as (
    select random() as rand,
    personal || ' ' || family as name
    from staff
)

select
    rand,
    abs(rand) % 10 as selector,
    name
from decorated
where selector < 5;
```
```
|         rand         | selector |      name       |
|----------------------|----------|-----------------|
| -5088363674211922423 | 0        | Divit Dhaliwal  |
| 6557666280550701355  | 1        | Indrans Sridhar |
| -2149788664940846734 | 3        | Pranay Khanna   |
| -3941247926715736890 | 8        | Riaan Dua       |
| -3101076015498625604 | 5        | Vedika Rout     |
| -7884339441528700576 | 4        | Abram Chokshi   |
| -2718521057113461678 | 4        | Romil Kapoor    |
```

1. Невозможно заполнить генератор случайных чисел SQLite другим способом.
2. А это значит, что нет возможности воспроизвести его псевдослучайные последовательности
3. Это означает, что вы никогда не должны использовать его
4. Как вы собираетесь отлаживать то, что не можете перезапустить?

#### Упражнение

Напишите запрос, который:
 - использует CTE для создания 1000 случайных чисел от 0 до 10 включительно;
 - использует второй CTE для расчета среднего значения
 - использует третий CTE и встроенные математические функции SQLite для расчета их стандартного отклонения.

 