# Основные возможности 

*Список терминов: [Административная команда (administration command)](/resources/glossary.md?id=Административная-команда-administration-command), [Агрегация (aggregation)](/resources/glossary.md?id=Агрегация-aggregation), [Агрегатная функция (aggregation function)](/resources/glossary.md?id=Агрегатная-функция-aggregation-function), [Перекрестное соединение (cross join)](/resources/glossary.md?id=Перекрестное-соединение-cross-join), [Исключающее ИЛИ (exclusive or)](/resources/glossary.md?id=Исключающее-ИЛИ-exclusive-or), [Фильтрация (filter)](/resources/glossary.md?id=Фильтрация-filter), [Полное внешнее соединение (full outer join)](/resources/glossary.md?id=Полное-внешнее-соединение-full-outer-join), [Группа (group)](/resources/glossary.md?id=Группа-group), [База данных в памяти (in-memory database)](/resources/glossary.md?id=База-данных-в-памяти-in-memory-database), [Включающее ИЛИ (inclusive or)](/resources/glossary.md?id=Включающее-ИЛИ-inclusive-or), [Соединение (join)](/resources/glossary.md?id=Соединение-join), [Условие соединения (join condition)](/resources/glossary.md?id=Условие-соединения-join-condition), [Левое внешнее соединение (left outer join)](/resources/glossary.md?id=Левое-внешнее-соединение-left-outer-join), [Нуль (null)](/resources/glossary.md?id=Нуль-null), [Запрос](/resources/glossary.md?id=Запрос-query), [Правое внешнее соединение (right outer join)](/resources/glossary.md?id=Правое-внешнее-соединение-right-outer-join), [Трехзначная логика (ternary logic)](/resources/glossary.md?id=Трехзначная-логика-ternary-logic), [Надгробный камень (tombstone)](/resources/glossary.md?id=Надгробный-камень-tombstone)*

**SELECT** (выбрать) - это ключевое слово, которое обычно используется для получения данных из базы в языке SQL

## Выбор констант

SELECT обычно используется для выбора данных из таблицы...
...но если все, что нам нужно, это постоянное значение, нам не нужно указывать указывать имя таблицы

В большинстве диалектов SQL требуется разделитель точка с запятой в конце запроса;

```select 1;```

```Результатом этого запроса будет константа - 1```

## Получение всех данных из таблицы

```sql select * from little_penguins;```

```
Gentoo|Biscoe|51.3|14.2|218.0|5300.0|MALE
Adelie|Dream|35.7|18.0|202.0|3550.0|FEMALE
Adelie|Torgersen|36.6|17.8|185.0|3700.0|FEMALE
Chinstrap|Dream|55.8|19.8|207.0|4000.0|MALE
Adelie|Dream|38.1|18.6|190.0|3700.0|FEMALE
Adelie|Dream|36.2|17.3|187.0|3300.0|FEMALE
Adelie|Dream|39.5|17.8|188.0|3300.0|FEMALE
Gentoo|Biscoe|42.6|13.7|213.0|4950.0|FEMALE
Gentoo|Biscoe|52.1|17.0|230.0|5550.0|MALE
Adelie|Torgersen|36.7|18.8|187.0|3800.0|FEMALE
```

Результат выполнения можно посмотреть на [SQlize online](https://sqlize.online/sql/sqlite3_data/970e5de956dad37e24ca99f030d5ac13/)

Рассмотрим этот запрос:
- SELECT (выбрать) - ключевое слово для получения любых данных 
- Используем * для обозначения «все столбцы».
- Используем `from tablename` где tablename - имя таблицы из которой извлекаем данные.
  
Выходной формат будет не всегда понятнен для вас.


##  Административные команды базы данных SQLite

- .headers on - показывать заголовки столбцов таблицы
- .mode markdown - использовать разметку отображения

.mode markdown и .headers делают вывод более читабельным. После примнения этих команд вывод данных примет следующий вид.

```sql
select * from little_penguins;
```
```
|  species  |  island   | bill_length_mm | bill_depth_mm | flipper_length_mm | body_mass_g |  sex   |
|-----------|-----------|----------------|---------------|-------------------|-------------|--------|
| Gentoo    | Biscoe    | 51.3           | 14.2          | 218.0             | 5300.0      | MALE   |
| Adelie    | Dream     | 35.7           | 18.0          | 202.0             | 3550.0      | FEMALE |
| Adelie    | Torgersen | 36.6           | 17.8          | 185.0             | 3700.0      | FEMALE |
| Chinstrap | Dream     | 55.8           | 19.8          | 207.0             | 4000.0      | MALE   |
| Adelie    | Dream     | 38.1           | 18.6          | 190.0             | 3700.0      | FEMALE |
| Adelie    | Dream     | 36.2           | 17.3          | 187.0             | 3300.0      | FEMALE |
| Adelie    | Dream     | 39.5           | 17.8          | 188.0             | 3300.0      | FEMALE |
| Gentoo    | Biscoe    | 42.6           | 13.7          | 213.0             | 4950.0      | FEMALE |
| Gentoo    | Biscoe    | 52.1           | 17.0          | 230.0             | 5550.0      | MALE   |
| Adelie    | Torgersen | 36.7           | 18.8          | 187.0             | 3800.0      | FEMALE |
```

Административные команды SQLite начинаются с  точки . и не являются частью стандарта SQL. Специальные команды PostgreSQL начинаются с косой черты \
Каждая команда должна располагаться на отдельной строке.
Используйте .help для получения полного списка.
И, как упоминалось ранее, используйте .quit для выхода.

## Выбор колонок 

([онлайн демо](https://sqlize.online/sql/sqlite3_data/9484db9d3625b78b2ad204aa48596150/))

```sql
select
    species,
    island,
    sex
from little_penguins;
```
```
|  species  |  island   |  sex   |
|-----------|-----------|--------|
| Gentoo    | Biscoe    | MALE   |
| Adelie    | Dream     | FEMALE |
| Adelie    | Torgersen | FEMALE |
| Chinstrap | Dream     | MALE   |
| Adelie    | Dream     | FEMALE |
| Adelie    | Dream     | FEMALE |
| Adelie    | Dream     | FEMALE |
| Gentoo    | Biscoe    | FEMALE |
| Gentoo    | Biscoe    | MALE   |
| Adelie    | Torgersen | FEMALE |
```

Для выбора определённых колонок укажите их имена разделенные запятыми.
В любом порядке!
Дубликаты разрешены!
Разрывы строк приветствуются для удобства чтения, вам все вернется.

## Cортировка результатов запроса

([онлайн демо](https://sqlize.online/sql/sqlite3_data/528cad2f89530cb59775b1d8ae00b4f2/))

```sql
select
    species,
    sex,
    island
from little_penguins
order by island asc, sex desc;
```
```
|  species  |  sex   |  island   |
|-----------|--------|-----------|
| Gentoo    | MALE   | Biscoe    |
| Gentoo    | MALE   | Biscoe    |
| Gentoo    | FEMALE | Biscoe    |
| Chinstrap | MALE   | Dream     |
| Adelie    | FEMALE | Dream     |
| Adelie    | FEMALE | Dream     |
| Adelie    | FEMALE | Dream     |
| Adelie    | FEMALE | Dream     |
| Adelie    | FEMALE | Torgersen |
| Adelie    | FEMALE | Torgersen |
```

`order by` должен следовать после `from` (который должен следовать первым за `select`)
- `asc` — сортировка по возрастанию,
- `desc` — по убыванию.
По умолчанию — выполняется сортировка по возрастанию, но лучше укажите, пожалуйста.

#### Упражнения

- Напишите SQL-запрос для выбора столбцов `sex` - пол и `body_mass_g` - масса тела из таблицы `little_penguins`, отсортированных таким образом, чтобы сначала отображалась наибольшая масса тела. [Выполнить задание онлайн](https://sqltest.online/ru/question/sqlite/sort-penguins)


## Ограничение результата запроса

([онлайн демо](https://sqlize.online/sql/sqlite3_data/7b9ec307409f3d84a132e00c0bff3908/))

```sql
select
    species,
    sex,
    island
from penguins
order by species, sex, island
limit 10;
```
```
| species |  sex   |  island   |
|---------|--------|-----------|
| Adelie  |        | Dream     |
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
```

- Оператор `limit N` ограничивает количество строк возвращаемых запросом. 
- Число N указывает максимальное количество строк. Если результат содержит меньше строк - будут выведены все.
- Ограничение следует использовать только после сортировки. В противном случае результат непредсказуем!

#### Упражнения

1. Напишите запрос для выбора вида (`species`) и острова обитания (`island`) трёх самых лёгких пингвинов из таблицы `penguins`. [Выполнить задание онлайн](https://sqltest.online/ru/question/sqlite/light-weight-penguins)

## Постраничный вывод (пагинация)

([онлайн демо](https://sqlize.online/sql/sqlite3_data/f223fa3e3a3eea42d51edd2e6c9ff915/))

```sql
select
    species,
    sex,
    island
from penguins
order by species, sex, island
limit 10 offset 3;
```
```
| species |  sex   |  island   |
|---------|--------|-----------|
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  |        | Torgersen |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
| Adelie  | FEMALE | Biscoe    |
```

- Оператор `offset N` должен следовать после `limit M`
- Число N указывает количество строк, которые нужно пропустить с начала результата запроса.
- Таким образом, запрос `limit 10 offset 3` пропускает первые 3 строки и показывает следующие 10.

## Удаление дубликатов из результата запроса

([онлайн демо](https://sqlize.online/sql/sqlite3_data/31e1f865c5de14af77eca070cd985d63/))

```sql
select distinct
    species,
    sex,
    island
from penguins;
```
```
|  species  |  sex   |  island   |
|-----------|--------|-----------|
| Adelie    | MALE   | Torgersen |
| Adelie    | FEMALE | Torgersen |
| Adelie    |        | Torgersen |
| Adelie    | FEMALE | Biscoe    |
| Adelie    | MALE   | Biscoe    |
| Adelie    | FEMALE | Dream     |
| Adelie    | MALE   | Dream     |
| Adelie    |        | Dream     |
| Chinstrap | FEMALE | Dream     |
| Chinstrap | MALE   | Dream     |
| Gentoo    | FEMALE | Biscoe    |
| Gentoo    | MALE   | Biscoe    |
| Gentoo    |        | Biscoe    |
```

- Ключевое слово `distinct` пишется сразу после `select` (потому что SQL должен был читаться как английский)
- Показывает только уникальные строки в результате запроса. Дипликаты игнорируются и не выводятся.
- Пробелы в столбце `sex` указывают на недостающие данные. Мы поговорим об этом немного позже

#### Упражнения 

1. Напишите SQL-запрос для выбора островов и видов из строк с 50 по 60 включительно таблицы пингвинов. Ваш результат должен состоять из 11 строк.
2. Измените свой запрос, чтобы выбрать различные комбинации островов и видов из одних и тех же строк, и сравните результат с тем, что вы получили в части 1.


## Фильтрация результатов запроса

([онлайн демо](https://sqlize.online/sql/sqlite3_data/ffbe246d8112c8c998b0d4b5cecee941/))

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe';
```
```
| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Adelie  | MALE   | Biscoe |
| Gentoo  | FEMALE | Biscoe |
| Gentoo  | MALE   | Biscoe |
| Gentoo  |        | Biscoe |
```

- Команда `where` фильтрует строки, полученные в результате выбора.
- Условие проверяется независимо для каждой строки
- В результатах отображаются только строки, прошедшие проверку.
*Используйте одинарные кавычки для 'текстовых данных' и двойные кавычки для "странных имен столбцов".
SQLite будет принимать текстовые данные в двойных кавычках, но SQLFluff будет жаловатьсяю*

#### Упражнения

1. Напишите запрос для выбора пингвинов с массой тела менее 3000 граммов.
2. Напишите еще один запрос, чтобы выбрать вид и пол пингвинов весом менее 3000 граммов. Это показывает, что отображаемые столбцы и столбцы, используемые при фильтрации, независимы друг от друга.

## Фильтрация с более сложными условиями

([онлайн демо](https://sqlize.online/sql/sqlite3_data/c99103b164cab302cf03bc976f9a6beb/))

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe' and sex != 'MALE';
```
```
| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Gentoo  | FEMALE | Biscoe |
```
- Можно комбинировать условия ипользуя союзы `and` и `or`
- `and` - требует чтобы оба подусловия были истинными.
- `or`  - достаточно что одно из условий верно.
- Сначала выполняется проверка  `and`, затем `or`. В сложных случаях используйте скобки.
- Обратите внимание, что строка для пингвинов Генту на острове Биско с неизвестным (пустым) полом не прошла проверку.
- Мы поговорим об этом немного позже

#### Упражнения

1. Используйте оператор not, чтобы выбрать пингвинов, которые не являются Gentoos.
2. В SQL оператор or является включающим оператором or: он выполняется успешно, если одно или оба условия истинны. SQL не предоставляет специального оператора для исключающего или, который верен, если истинно одно из условий, но не оба, но того же эффекта можно достичь, используя and, or и not. Напишите запрос, чтобы выбрать пингвинов-самок или пингвинов на острове Торгерсен, но не обоих сразу.

## Выполнение расчетов

([онлайн демо](https://sqlize.online/sql/sqlite3_data/5263669b94cf065f18343dcc33248739/))

```sql
select
    flipper_length_mm / 10.0,
    body_mass_g / 1000.0
from penguins
limit 3;
```
```
| flipper_length_mm / 10.0 | body_mass_g / 1000.0 |
|--------------------------|----------------------|
| 18.1                     | 3.75                 |
| 18.6                     | 3.8                  |
| 19.5                     | 3.25                 |
```

- SQL может выполнять различные арифметические операции с отдельными значениями.
- Расчет производится для каждой строки отдельно.
- Название столбца (`flipper_length_mm / 10.0`, `body_mass_g / 1000.0`) показывает выполненный расчет. 

## Псевдонимы колонок

([онлайн демо](https://sqlize.online/sql/sqlite3_data/05bde8ecc4459418c669e0637deb290d/))

```sql
select
    flipper_length_mm / 10.0 as flipper_cm,
    body_mass_g / 1000.0 as weight_kg,
    island as where_found
from penguins
limit 3;
```
```
| flipper_cm | weight_kg | where_found |
|------------|-----------|-------------|
| 18.1       | 3.75      | Torgersen   |
| 18.6       | 3.8       | Torgersen   |
| 19.5       | 3.25      | Torgersen   |
```

- Используйте `выражение as имя` для назначения псевдонима.
- Используйте осмысленные псевдонимы.
- Также можно давать псевдонимы столбцам без изменения.

#### Упражнения

Напишите один запрос, который вычисляет и возвращает:
1. Столбец под названием `what_where`, в котором виды и острова каждого пингвина разделены одним пробелом.
2. Столбец `bill_ratio`, в котором указано соотношение длины клюва к его глубине.

*Вы можете использовать || оператор для объединения текста для решения части 1 или посмотрите документацию по функции [SQLite format()](https://sqlite.org/printf.html).*

### Прверка знаний


<img src="./assets/concep.png" alt="Описание" style="max-width:100%; height:auto;">


## Вычисление с отсутствующими значениями

([онлайн демо](https://sqlize.online/sql/sqlite3_data/05bde8ecc4459418c669e0637deb290d/))

```sql
select
    flipper_length_mm / 10.0 as flipper_cm,
    body_mass_g / 1000.0 as weight_kg,
    island as where_found
from penguins
limit 5;
```
```
| flipper_cm | weight_kg | where_found |
|------------|-----------|-------------|
| 18.1       | 3.75      | Torgersen   |
| 18.6       | 3.8       | Torgersen   |
| 19.5       | 3.25      | Torgersen   |
|            |           | Torgersen   |
| 19.3       | 3.45      | Torgersen   |
```

- SQL использует специальное значение null для представления отсутствующих данных.
- null это не 0 или пустая строка, а «Я не знаю».
- Длина ласт и масса тела ни одного из первых пяти пингвинов неизвестны.
- «Я не знаю», разделенное на 10 или 1000, — это «Я не знаю».

#### Упражнения
1. Используйте команду SQLite .nullvalue, чтобы изменить печатное представление значения null на строку null, а затем повторно запустите предыдущий запрос. Когда будет легче понять отображение null как null? Когда это может ввести в заблуждение?

### Сравнение с `null`

([онлайн демо](https://sqlize.online/sql/sqlite3_data/0c62d806cfb999a431d3c78b488d965d/))

- это повторили ранее

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe';
```
```
| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Adelie  | MALE   | Biscoe |
| Gentoo  | FEMALE | Biscoe |
| Gentoo  | MALE   | Biscoe |
| Gentoo  |        | Biscoe |
```

Если мы спросим о самках пингвинов, строка с отсутствующим полом получаем следкющее.

```sql 
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe' and sex = 'FEMALE';
```
```
| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Gentoo  | FEMALE | Biscoe |
```

([онлайн демо](https://sqlize.online/sql/sqlite3_data/b77d9e619645f3562fe18b0bec573ed6/))

- Но если мы спросим о пингвинах, не являющихся самками, их мы тоже получим

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe' and sex != 'FEMALE';
```
```
| species | sex  | island |
|---------|------|--------|
| Adelie  | MALE | Biscoe |
| Gentoo  | MALE | Biscoe |
```

### Тернарная логика

([онлайн демо](https://sqlize.online/sql/sqlite3_data/f42a97eea4aabe84ac06069c2def564e/))

```sql
select null = null;
```
```
| null = null |
|-------------|
|             |
```

Если мы не знаем левого и правого значений, мы не знаем, равны они или нет.
Итак, результат нулевой
Получите тот же ответ для null != null
Тернарная логика

 <img src="./assets/logic.png" alt="Описание" style="max-width:100%; height:auto;">


### Безопасная обработка `null` значений

([онлайн демо](https://sqlize.online/sql/sqlite3_data/9fcfbdccb6ba09efcdfba07940429a40/))

```sql
 select
    species,
    sex,
    island
from penguins
where sex is null;
```
```
| species | sex |  island   |
|---------|-----|-----------|
| Adelie  |     | Torgersen |
| Adelie  |     | Torgersen |
| Adelie  |     | Torgersen |
| Adelie  |     | Torgersen |
| Adelie  |     | Torgersen |
| Adelie  |     | Dream     |
| Gentoo  |     | Biscoe    |
| Gentoo  |     | Biscoe    |
| Gentoo  |     | Biscoe    |
| Gentoo  |     | Biscoe    |
| Gentoo  |     | Biscoe    |
```

- Используйте `is null` и `is not null` для безопасной обработки null.
- Другие части SQL обрабатывают нули особым образом.

#### Упражнения

1. Напишите запрос, чтобы найти пингвинов, масса тела которых известна, но пол неизвестен.
2. Напишите еще один запрос, чтобы найти пингвинов, пол которых известен, но масса тела неизвестна.

### Закрепление

 <img src="./assets/check.png" alt="Описание" style="max-width:100%; height:auto;">

## Агрегация

```sql
select sum(body_mass_g) as total_mass
from penguins;
```
```
| total_mass |
|------------|
| 1437000.0  |
```

- Агрегация объединяет множество значений для получения одного
- Сумма — это функция агрегирования
- Объединяет соответствующие значения из нескольких строк

### Общие функции агрегирования

```sql
select
    max(bill_length_mm) as longest_bill,
    min(flipper_length_mm) as shortest_flipper,
    avg(bill_length_mm) / avg(bill_depth_mm) as weird_ratio
from penguins;
```
```
| longest_bill | shortest_flipper |   weird_ratio    |
|--------------|------------------|------------------|
| 59.6         | 172.0            | 2.56087082530644 |
```

- На самом деле это не должно работать: невозможно вычислить максимальное или среднее значение, если какие-либо значения равны нулю.
- SQL делает полезную вещь вместо правильной

#### Упражнения 

1. Какова средняя масса тела пингвинов весом более 3000 граммов?

### Подсчет

```sql
select
    count(*) as count_star,
    count(sex) as count_specific,
    count(distinct sex) as count_distinct
from penguins;
```
```
| count_star | count_specific | count_distinct |
|------------|----------------|----------------|
| 344        | 333            | 2              |
```

- `count(*)` - считает строки.
- `count(column)` - подсчитывает непустые (не `null`) записи в столбце.
- `count(distinct column)` - подсчитывает уникальные непустые (не `null`) записи.

#### Упражнения

1. Сколько разных масс тела содержится в наборе данных по пингвинам?

### Группировка

```sql
select avg(body_mass_g) as average_mass_g
from penguins
group by sex;
```
```
|  average_mass_g  |
|------------------|
| 4005.55555555556 |
| 3862.27272727273 |
| 4545.68452380952 |
```

- Поместите строки в группы на основе различных комбинаций значений в столбцах, указанных с помощью group by
- Затем выполните агрегацию отдельно для каждой группы.
- Но что есть что?


### Поведение неагрегированных столбцов

```sql
select
    sex,
    avg(body_mass_g) as average_mass_g
from penguins
group by sex;
```
```
|  sex   |  average_mass_g  |
|--------|------------------|
|        | 4005.55555555556 |
| FEMALE | 3862.27272727273 |
| MALE   | 4545.68452380952 |
```

- Все строки в каждой группе имеют одинаковое значение пола, поэтому агрегировать не нужно.

### Произвольный выбор при агрегировании

```sql
select
    sex,
    body_mass_g                   
from penguins
group by sex;
```
```
|  sex   | body_mass_g |
|--------|-------------|
|        |             |
| FEMALE | 3800.0      |
| MALE   | 3750.0      |
```

- Если мы не укажем, как агрегировать столбец, SQLite выбирает любое произвольное значение из группы.
- Все пингвины в каждой группе имеют один и тот же пол, потому что мы сгруппировали его по этому признаку и получили правильный ответ.
- Значения массы тела имеются в данных, но непредсказуемы.
- Распространенная ошибка
- Другие СУБД этого не делают. Например, PostgreSQL жалуется, что столбец необходимо использовать в функции агрегации.


#### Упражнения

1. Объясните, почему в результатах предыдущего запроса перед строками для самок и самцов пингвинов имеется пустая строка.
2. Напишите запрос, который показывает каждую уникальную массу тела в наборе данных о пингвинах и количество пингвинов, которые весят столько же.

### Фильтрация агрегированных значений

```sql
select
    sex,
    avg(body_mass_g) as average_mass_g
from penguins
group by sex
having average_mass_g > 4000.0;
```
```
| sex  |  average_mass_g  |
|------|------------------|
|      | 4005.55555555556 |
| MALE | 4545.68452380952 |
```

- Используйте `having` для фильтрации агрегированных значений вместо `where`.
- Команда `having` пишется сразу после `group by`.

#### Упражнения

1. Найдите острова со средней длиной плавника проживающих на них пингвинов меньше 200 милиметров.


### Читаемый вывод

```sql
select
    sex,
    round(avg(body_mass_g), 1) as average_mass_g
from penguins
group by sex
having average_mass_g > 4000.0;
```

| sex  | average_mass_g |
|------|----------------|
|      | 4005.6         |
| MALE | 4545.7         |

- Используйте round(value, десятичные дроби) для округления числа.

### Фильтрация аггрегированых входных данных

```sql
select
    sex,
    round(
        avg(body_mass_g) filter (where body_mass_g < 4000.0),
        1
    ) as average_mass_g
from penguins
group by sex;
```

|  sex   | average_mass_g |
|--------|----------------|
|        | 3362.5         |
| FEMALE | 3417.3         |
| MALE   | 3729.6         |

- условие фильтрации применяется к входным данным аггрегирующей функции


#### Упражнения

- Напишите запрос, который использует фильтр для одновременного расчета средней массы тела тяжелых пингвинов (весом более 4500 граммов) и легких пингвинов (весом менее 3500 граммов). Можно ли это сделать, используя «where» вместо «filter»?

### Прверка знаний


<img src="./assets/core_aggregate_concept_map.svg" alt="Описание" style="max-width:100%; height:auto;">

## Создание базы данных в памяти

```shell
sqlite3 :memory: 
```

- «Подключиться» к базе данных в памяти
- Изменения не сохраняются на диск
- Очень полезно для тестирования.

### Создание таблиц

```sql
create table job (
    name text not null,
    billable real not null
);
create table work (
    person text not null,
    job text not null
);
```

- Создать имя таблицы, за которым следует список столбцов в скобках
- Каждый столбец – это имя, тип данных и необязательная дополнительная информация.
- Например, not null предотвращает добавление нулей.
- Указание схемы в SQLite не является стандартом
- SQLite добавил несколько вещей
  - create if not exists
  - ключевые слова в верхнем регистре (SQL нечувствителен к регистру)