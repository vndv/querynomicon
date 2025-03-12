# Инструменты

> Список терминов: [1-to-1 relation](glossary.md#отношение-один-к-одному-1-to-1-relation), [1-to-many relation](glossary.md#отношение-один-ко-многим-1-to-many-relation), [alias](glossary.md#псевдоним-alias), [autoincrement](glossary.md#автоинкремент-autoincrement), [B-tree](glossary.md#бинарное-дерево-b-tree), [common table expression (CTE)](glossary.md#общее-табличное-выражение-common-table-expression-cte), [correlated subquery](glossary.md#коррелированный-подзапрос-correlated-subquery), [data migration](glossary.md#мигрция-данных-data-migration), [entity-relationship diagram](glossary.md#диаграмма-сущность-связь-entity-relationship-diagram), [expression](glossary.md#выражение-expression), [foreign key](glossary.md#внешний-ключ-foreign-key), [index](glossary.md#индекс-index), [join table](glossary.md#соединение-join), [many-to-many relation](glossary.md#отношение-многие-ко-многим--many-to-many-relation), [primary key](glossary.md#первичный-ключ-primary-key), [statement](glossary.md#оператор-statement), [subquery](glossary.md#подзапрос-subquery), [table-valued function](glossary.md#табличная-функция-table-valued-function), [vectorization](glossary.md#векторизация-vectorization), [window function](glossary.md#оконная-функция-window-function), [Базовый случай (base-case)](/resources/glossary.md?id=Базовый-случай-base-case), [Бесконечная рекурсия (infinite recursion)](/resources/glossary.md?id=Бесконечная-рекурсия-infinite-recursion), [Временная таблица (temporary table)](/resources/glossary.md?id=Временная-таблица-temporary-table), [Материализованное представление (materialized view)](/resources/glossary.md?id=Материализованное-представление-materialized-view), [Представление (view)](/resources/glossary.md?id=Представление-view), [Рекурсивное табличное выражение (recursive cte)](/resources/glossary.md?id=Рекурсивное-табличное-выражение-recursive-cte), [Рекурсивный случай (recursive case)](/resources/glossary.md?id=Рекурсивный-случай-recursive-case)


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

- Оператор вхождения `in` и не вхождения `not in` делают именно то, что нам нужно.

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

1. Используйте подзапрос, чтобы выбрать людей, которые выполняют калибровку.
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
from penguins 
inner join (
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
2. Затем внутреннее соеденение с таблицей `penguins`.
3. Фильтр для поиска пингвинов, вес которых превышает средний вес представителей своего вида.

#### Упражнение 
1. Используйте подзапрос, чтобы найти количество пингвинов, которые весят столько же, сколько самый легкий пингвин того же пола и вида.


### Вхождение и коррелированные подзапросы

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/4ccdd62c3c256be8daae798a33ae1efc/))

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

- Эндокринология отсутствует в списке.
- Вместо `select 1` можно использовать `select true` или любое другое значение.
- Коррелированный подзапрос зависит от значения из внешнего запроса.
- Коррелированный подзапрос это эквивалент вложенного цикла - выполняется столько раз сколько строк вернул основной запрос.

 ### Невхождение

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/842a325283a5ed339786b07f5811cdae/))

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

1. Можете ли вы переписать предыдущий запрос, используя исключение? Если да, то легче ли понять ваш новый запрос? Если запрос нельзя переписать, почему бы и нет?

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

- Соединение может быть быстрее, а может и нет чем коррелированный подзапрос. Истинна определяется практикой.

## Производные таблицы

### Временные таблицы

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

### Представления (views)

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


### Общие табличные выражения  (Common Table Expression, CTE)

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

- Используйте общее табличное выражение (CTE), чтобы сделать запросы более понятными.
- Вложенные подзапросы быстро становятся трудными для понимания.
- База данных решает, как оптимизировать запрос.

### Проверка знаний

<img src="./assets/advanced_temp_concept_map.svg" alt="Проверка знаний" style="max-width:100%; height:auto;">

### Рекурсивные запросы

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

## Оконные функции

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/7a70450164ab33995277a98693f55f0b/))

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

### Границы (Boundaries)

1. Документация по оконным функциям SQLite описывает три типа окон и пять видов границ окон.
2. Каждый тип нужно использовать по ситуации

### Партицированые окна (Partitioned Windows)

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

1. Создайте запрос, который:
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

1. Как изменится результат предыдущего запроса, если убрать проверку на нулевую массу тела? Почему результат без этой проверки вводит в заблуждение?

Что дает каждое из приведенных ниже выражений? Как вы думаете, какие из них на самом деле пытаются разделить на ноль?

1. `iif(0, 123, 1/0)`
2. `iif(1, 123, 1/0)`
3. `iif(0, 1/0, 123)`
4. `iif(1, 1/0, 123)`

### Множественный выбор с CASE WHEN

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

- Сравнивает условия и выбирает первое подходящее.
- Результат `CASE` равен `null`, если ни одно условие не истинно.
- Используйте `else` для случая по-умолчанию.

#### Упражнения

Измените приведенный выше запрос так, чтобы выходные данные были «пингвин маленький» и «пингвин большой», объединив строку «пингвин есть» со всем регистром, а не с отдельными ответвлениями. (Это упражнение показывает, что `CASE/WHEN` является выражением, а не утверждением.)

### Функции работы со строками

| **функция**	| **назначение** |
|---------------|----------------|
| `substr`	    | Получить подстроку, задав начальную позицию и длину
| `trim`	    | Удалить символы с начала и конца строки
| `ltrim`	    | Удалить символы с начала строки
| `rtrim`	    | Удалить символы с конца строки
| `length`	    | Длина строки
| `replace`     | Заменить вхождения подстроки другой строкой
| `upper`	    | Вернуть строку в верхнем регистре
| `lower`	    | Вернуть строку в нижнем регистре
| `instr`	    | Найти позицию первого вхождения подстроки (возвращает 0, если не найдено)

- Полный список функций можно найти в [документации](https://www.sqlite.org/lang_corefunc.html)


## Еще одна база данных

- Диаграмма сущность-связь (диаграмма ER) показывает связи между таблицами.
- Как и все, что связано с базами данных, существует множество вариаций связи таблиц.

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

1. Нарисуйте табличную диаграмму и диаграмму ER, чтобы представить базу данных со следующими таблицами:
    1. person есть `id` и `full_name`
    2. course есть `id` и `name`
    3. section есть `course_id`, `start_date`, и `end_date`
    4. instructor есть `person_id` и `section_id`
    5. student есть `person_id`, `section_id`, и `status`


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

- `union all` объединяет результаты запросов, сохраняя все записи, включая дубликаты.
- `union` объединяет результаты, возвращая только уникальные записи.

#### Упражнение

1. Напишите запрос, результат которого включает две строки для каждого пингвина Адели в базе данных пингвинов. Как вы можете проверить, что ваш запрос работает правильно?

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

1. Используйте пересечение, чтобы найти всех пингвинов Адели весом более 4000 граммов. Как вы можете проверить, что ваш запрос работает правильно?
2. Используйте план запроса(`explain`) , чтобы сравнить только что написанный запрос на основе пересечения с запросом, в котором используется ключевое слово `where`. Какой запрос выглядит более эффективным? Почему вы этому верите?

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
    age 
from staff
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

1. Используйте «исключение», чтобы найти всех пингвинов Gentoo, кроме самцов. Как вы можете проверить, что ваш запрос работает правильно?

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

1. Напишите запрос, который:
    - использует CTE для создания 1000 случайных чисел от 0 до 10 включительно;
    - использует второй CTE для расчета среднего значения
    - использует третий CTE и встроенные математические функции SQLite для расчета их стандартного отклонения.


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
 
### Генерация уникальных пар (используем самосоединение)

```sql
with person as (
    -- получаем данные
    select
        ident,
        personal || ' ' || family as name
    from staff
)
select
    left_person.name,
    right_person.name
from person as left_person 
inner join person as right_person -- используем самосоединение
    on left_person.ident < right_person.ident -- за исключением идентичных пар
order by -- сортируем
    left_person.name asc,
    right_person.name asc
;
```

([онлайн sql песочница](https://sqlize.online/sql/sqlite3_data/abacb55ac4dd1d852dc1efa0405a0487/))

```
| name             | name             |
|------------------|------------------|
| Abram Chokshi    | Ishaan Ramaswamy |
| Abram Chokshi    | Nitya Lal        |
| Abram Chokshi    | Romil Kapoor     |
| Divit Dhaliwal   | Abram Chokshi    |
| ...              | ...              |
| Vedika Rout      | Abram Chokshi    |
| Vedika Rout      | Ishaan Ramaswamy |
| Vedika Rout      | Nitya Lal        |
| Vedika Rout      | Romil Kapoor     |
```

- условие `left.ident < right.ident` обеспечивает различные пары без дубликатов.

### Фильтрация пар

Найдём пары сотрудников работающие над общим экспериментом. 

```sql
with
person as (
    -- получаем список сотрудников
    select
        ident,
        personal || ' ' || family as name
    from staff
),
together as (
    -- пары сотрудников работающие над общим экспериментом
    select
        left_perf.staff as left_staff,
        right_perf.staff as right_staff
    from performed as left_perf 
    inner join performed as right_perf
        on left_perf.experiment = right_perf.experiment
    where left_staff < right_staff
)
select
    left_person.name as person_1,
    right_person.name as person_2
from person as left_person 
inner join person as right_person 
inner join together
    on left_person.ident = left_staff and right_person.ident = right_staff;
```

([выполнить sql онлайн](https://sqlize.online/sql/sqlite3_data/ecdd4df71c7daa33133c385b39921160/))

```
|    person_1     |     person_2     |
|-----------------|------------------|
| Kartik Gupta    | Vedika Rout      |
| Pranay Khanna   | Vedika Rout      |
| Indrans Sridhar | Romil Kapoor     |
| Abram Chokshi   | Ishaan Ramaswamy |
| Pranay Khanna   | Vedika Rout      |
| Kartik Gupta    | Abram Chokshi    |
| Abram Chokshi   | Romil Kapoor     |
| Kartik Gupta    | Divit Dhaliwal   |
| Divit Dhaliwal  | Abram Chokshi    |
| Pranay Khanna   | Ishaan Ramaswamy |
| Indrans Sridhar | Romil Kapoor     |
| Kartik Gupta    | Ishaan Ramaswamy |
| Kartik Gupta    | Nitya Lal        |
| Kartik Gupta    | Abram Chokshi    |
| Pranay Khanna   | Romil Kapoor     |
```