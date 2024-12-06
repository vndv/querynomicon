# Основные возможности 

*Terms defined: [administration command](https://lessonomicon.github.io/querynomicon/glossary.html#admin_command), [aggregation](https://lessonomicon.github.io/querynomicon/glossary.html#aggregation), [aggregation function](https://lessonomicon.github.io/querynomicon/glossary.html#aggregation_func), [cross join](https://lessonomicon.github.io/querynomicon/glossary.html#cross_join), [exclusive or](https://lessonomicon.github.io/querynomicon/glossary.html#exclusive_or), [filter](https://lessonomicon.github.io/querynomicon/glossary.html#filter), [full outer join](https://lessonomicon.github.io/querynomicon/glossary.html#full_outer_join), [group](https://lessonomicon.github.io/querynomicon/glossary.html#group), [in-memory database](https://lessonomicon.github.io/querynomicon/glossary.html#in_memory_db), [inclusive or](https://lessonomicon.github.io/querynomicon/glossary.html#inclusive_or), [join](https://lessonomicon.github.io/querynomicon/glossary.html#join), [join condition](https://lessonomicon.github.io/querynomicon/glossary.html#join_condition), [left outer join](https://lessonomicon.github.io/querynomicon/glossary.html#left_outer_join), [null](https://lessonomicon.github.io/querynomicon/glossary.html#null), [query](https://lessonomicon.github.io/querynomicon/glossary.html#query), [right outer join](https://lessonomicon.github.io/querynomicon/glossary.html#right_outer_join), [ternary logic](https://lessonomicon.github.io/querynomicon/glossary.html#ternary_logic), [tombstone](https://lessonomicon.github.io/querynomicon/glossary.html#tombstone)*


## Выбор констант

```SELECT 1```

```В данном случае 1 будет константой```

выбирать - это ключевое слово
Обычно используется для выбора данных из таблицы...
...но если все, что нам нужно, это постоянное значение, нам не нужно его указывать
Требуется разделитель точка с запятой

## Получение всех запросов из таблицы

```sql select * from little_penguins;```

> Gentoo|Biscoe|51.3|14.2|218.0|5300.0|MALE
> Adelie|Dream|35.7|18.0|202.0|3550.0|FEMALE
> Adelie|Torgersen|36.6|17.8|185.0|3700.0|FEMALE
> Chinstrap|Dream|55.8|19.8|207.0|4000.0|MALE
> Adelie|Dream|38.1|18.6|190.0|3700.0|FEMALE
> Adelie|Dream|36.2|17.3|187.0|3300.0|FEMALE
> Adelie|Dream|39.5|17.8|188.0|3300.0|FEMALE
> Gentoo|Biscoe|42.6|13.7|213.0|4950.0|FEMALE
> Gentoo|Biscoe|52.1|17.0|230.0|5550.0|MALE
> Adelie|Torgersen|36.7|18.8|187.0|3800.0|FEMALE

Результат выполнения можно посмотреть на [SQlize online](https://sqlize.online/sql/sqlite3_data/970e5de956dad37e24ca99f030d5ac13/)

Фактический запрос
Используйте * для обозначения «все столбцы».
Используйте from tablename для указания таблицы
Выходной формат будет не всегда понятнен для вас.


##  Административные команды

> .headers on

> .mode markdown

> select * from little_penguins;


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

.mode markdown и .headers делают вывод более читабельным.
Эти административные команды SQLite начинаются с  точки . и не являются частью стандарта SQL
Специальные команды PostgreSQL начинаются с косой черты\
Каждая команда должна располагаться на отдельной строке.
Используйте .help для получения полного списка.
И, как упоминалось ранее, используйте .quit для выхода.

## Выбор колонок

```sql
select
    species,
    island,
    sex
from little_penguins;
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

Укажите имена столбцов, разделенные запятыми.
В любом порядке
Дубликаты разрешены
Разрывы строк приветствуются для удобства чтения, вам все вернется.

## Cортировка результатов запроса

```sql
select
    species,
    sex,
    island
from little_penguins
order by island asc, sex desc;

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

порядок должен следовать из (который должен следовать первым за выбором)
asc — по возрастанию, desc — по убыванию
По умолчанию — по возрастанию, но укажите, пожалуйста.


### Упражнения

Напишите SQL-запрос для выбора столбцов пола и массы тела из Little_penguins в указанном порядке, отсортированных таким образом, чтобы сначала отображалась наибольшая масса тела.

- Полный набор данных содержит 344 строки.

## Ограничение результата запроса

```sql
select
    species,
    sex,
    island
from penguins
order by species, sex, island
limit 10;
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

- Комментарии начинаются с -- и продолжаются до конца строки.
- предел N указывает максимальное количество строк, возвращаемых запросом

## Постраничный выовд (пагинация)

```sql
select
    species,
    sex,
    island
from penguins
order by species, sex, island
limit 10 offset 3;
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

- смещение N должно соответствовать пределу
- Указывает количество строк, которые нужно пропустить с начала выделения.
- Таким образом, этот запрос пропускает первые 3 и показывает следующие 10.


## Удаление дупликатов из результата запроса

```sql
select distinct
    species,
    sex,
    island
from penguins;
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


- отдельное ключевое слово должно появиться сразу после выбора
- SQL должен был читаться как английский
- Показывает различные комбинации
- Пробелы в столбце «Пол» указывают на недостающие данные.
Мы поговорим об этом немного позже

### Упражнения 

1. Напишите SQL-запрос для выбора островов и видов из строк с 50 по 60 включительно таблицы пингвинов. Ваш результат должен состоять из 11 строк.

2. Измените свой запрос, чтобы выбрать различные комбинации островов и видов из одних и тех же строк, и сравните результат с тем, что вы получили в части 1.


## Фильтрация результатов запроса

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe';
```

| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Adelie  | MALE   | Biscoe |
| Gentoo  | FEMALE | Biscoe |
| Gentoo  | MALE   | Biscoe |
| Gentoo  |        | Biscoe |

- где условие фильтрует строки, полученные в результате выбора
Условие оценивается независимо для каждой строки
- В результатах отображаются только строки, прошедшие проверку.
Используйте одинарные кавычки для «текстовых данных» и двойные кавычки для «странных имен столбцов».
SQLite будет принимать текстовые данные в двойных кавычках, но SQLFluff будет жаловаться


### Упражнения
1. Напишите запрос для выбора массы тела пингвинов менее 3000,0 граммов.

2. Напишите еще один запрос, чтобы выбрать вид и пол пингвинов весом менее 3000,0 граммов. Это показывает, что отображаемые столбцы и столбцы, используемые при фильтрации, независимы друг от друга.


## Фильтрация с более сложными условиями

```sql
select distinct
    species,
    sex,
    island
from penguins
where island = 'Biscoe' and sex != 'MALE';
```

| species |  sex   | island |
|---------|--------|--------|
| Adelie  | FEMALE | Biscoe |
| Gentoo  | FEMALE | Biscoe |

- оба подусловия должны быть истинными
- или: одна или обе части должны быть истинными
- Обратите внимание, что строка для пингвинов Генту на острове Биско с неизвестным (пустым) полом не прошла проверку.
- Мы поговорим об этом немного позже

### Упражнения
1. Используйте оператор not, чтобы выбрать пингвинов, которые не являются Gentoos.
2. В SQL оператор or является включающим оператором or: он выполняется успешно, если одно или оба условия истинны. SQL не предоставляет специального оператора для исключающего или, который верен, если истинно одно из условий, но не оба, но того же эффекта можно достичь, используя and, or и not. Напишите запрос, чтобы выбрать пингвинов-самок или пингвинов на острове Торгерсен, но не обоих сразу.
