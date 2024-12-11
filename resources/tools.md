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