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

## Объяснение планов запросов

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