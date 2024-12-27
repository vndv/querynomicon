# R

## Подключение библиотек

```shell
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'

## The following objects are masked from 'package:stats':
## 
##     filter, lag

## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union

## [1] "little_penguins" "penguins"
```

- Используйте `dplyr` для создания запросов.
- Подключения к базе данных координируются пакетом DBI (DataBase Interface).

#### Что у нас есть?

- Получение списка таблиц с помощью dbListTables.

```
connection <- DBI::dbConnect(RSQLite::SQLite(), 'data/penguins.db')
DBI::dbListTables(connection)
```
```
## [1] "little_penguins" "penguins"
```

## Загрузка таблиц как датафрэйм

```
penguins <- dplyr::tbl(connection, 'penguins')
penguins
```
```
## # Source:   table<`penguins`> [?? x 7]
## # Database: sqlite 3.45.2 [/Users/tut/data/penguins.db]
##    species island    bill_length_mm bill_depth_mm flipper_length_mm body_mass_g
##    <chr>   <chr>              <dbl>         <dbl>             <dbl>       <dbl>
##  1 Adelie  Torgersen           39.1          18.7               181        3750
##  2 Adelie  Torgersen           39.5          17.4               186        3800
##  3 Adelie  Torgersen           40.3          18                 195        3250
##  4 Adelie  Torgersen           NA            NA                  NA          NA
##  5 Adelie  Torgersen           36.7          19.3               193        3450
##  6 Adelie  Torgersen           39.3          20.6               190        3650
##  7 Adelie  Torgersen           38.9          17.8               181        3625
##  8 Adelie  Torgersen           39.2          19.6               195        4675
##  9 Adelie  Torgersen           34.1          18.1               193        3475
## 10 Adelie  Torgersen           42            20.2               190        4250
## # ℹ more rows
## # ℹ 1 more variable: sex <chr>
```

- Получить таблицу с помощью функции tbl.
- Требуется соединение и имя таблицы.

#### Как мы сюда попали?

```
penguins |> 
  dplyr::show_query()
```
```
## <SQL>
## SELECT *
## FROM `penguins`
```

- Функции `dplyr` генерируют операторы SELECT.
- `show_query()` запускает команду SQL EXPLAIN, чтобы отобразить SQL-запрос.

## Ленивое выполнение (Lazy Evaluation)

```
penguins |> 
  filter(species == "Adelie")
```
```
## # Source:   SQL [?? x 7]
## # Database: sqlite 3.45.2 [/Users/tut/data/penguins.db]
##    species island    bill_length_mm bill_depth_mm flipper_length_mm body_mass_g
##    <chr>   <chr>              <dbl>         <dbl>             <dbl>       <dbl>
##  1 Adelie  Torgersen           39.1          18.7               181        3750
##  2 Adelie  Torgersen           39.5          17.4               186        3800
##  3 Adelie  Torgersen           40.3          18                 195        3250
##  4 Adelie  Torgersen           NA            NA                  NA          NA
##  5 Adelie  Torgersen           36.7          19.3               193        3450
##  6 Adelie  Torgersen           39.3          20.6               190        3650
##  7 Adelie  Torgersen           38.9          17.8               181        3625
##  8 Adelie  Torgersen           39.2          19.6               195        4675
##  9 Adelie  Torgersen           34.1          18.1               193        3475
## 10 Adelie  Torgersen           42            20.2               190        4250
## # ℹ more rows
## # ℹ 1 more variable: sex <chr>
```

- `filter` генерирует запрос, используя `WHERE` <условие>.
- вызов dbplyr ленив (`lazy`).
- SQL-запрос оценивается только тогда, когда он отправляется в базу данных.
- Выполнение вызова `dplyr` возвращает только предварительный просмотр результата.

```
penguins |> 
  filter(species == "Adelie") |> 
  collect()
```
```
## # A tibble: 152 × 7
##    species island    bill_length_mm bill_depth_mm flipper_length_mm body_mass_g
##    <chr>   <chr>              <dbl>         <dbl>             <dbl>       <dbl>
##  1 Adelie  Torgersen           39.1          18.7               181        3750
##  2 Adelie  Torgersen           39.5          17.4               186        3800
##  3 Adelie  Torgersen           40.3          18                 195        3250
##  4 Adelie  Torgersen           NA            NA                  NA          NA
##  5 Adelie  Torgersen           36.7          19.3               193        3450
##  6 Adelie  Torgersen           39.3          20.6               190        3650
##  7 Adelie  Torgersen           38.9          17.8               181        3625
##  8 Adelie  Torgersen           39.2          19.6               195        4675
##  9 Adelie  Torgersen           34.1          18.1               193        3475
## 10 Adelie  Torgersen           42            20.2               190        4250
## # ℹ 142 more rows
## # ℹ 1 more variable: sex <chr>
```

- `collect` извлекает полный результат запроса
- Обратите внимание, что теперь в таблице 152 строки.

## Выбор столбцов

```
penguins |> 
  select(species, island, contains('bill'))
```
```
## # Source:   SQL [?? x 4]
## # Database: sqlite 3.45.2 [C:\Users\tonin\Documents\Courses\r-sql\data\penguins.db]
##    species island    bill_length_mm bill_depth_mm
##    <chr>   <chr>              <dbl>         <dbl>
##  1 Adelie  Torgersen           39.1          18.7
##  2 Adelie  Torgersen           39.5          17.4
##  3 Adelie  Torgersen           40.3          18  
##  4 Adelie  Torgersen           NA            NA  
##  5 Adelie  Torgersen           36.7          19.3
##  6 Adelie  Torgersen           39.3          20.6
##  7 Adelie  Torgersen           38.9          17.8
##  8 Adelie  Torgersen           39.2          19.6
##  9 Adelie  Torgersen           34.1          18.1
## 10 Adelie  Torgersen           42            20.2
## # ℹ more rows
```

- Выбор данны в R изменяет оператор SQL `SELECT`.
- Работают помощники выбора из `dplyr` (например, contains).

## Сортировка

```
penguins |> 
  select(species, body_mass_g) |> 
  arrange(body_mass_g)
```
```
## # Source:     SQL [?? x 2]
## # Database:   sqlite 3.45.2 [C:\Users\tonin\Documents\Courses\r-sql\data\penguins.db]
## # Ordered by: body_mass_g
##    species   body_mass_g
##    <chr>           <dbl>
##  1 Adelie             NA
##  2 Gentoo             NA
##  3 Chinstrap        2700
##  4 Adelie           2850
##  5 Adelie           2850
##  6 Adelie           2900
##  7 Adelie           2900
##  8 Adelie           2900
##  9 Chinstrap        2900
## 10 Adelie           2925
## # ℹ more rows
```

- С `arrange` вы делаете сортировку данных по возрастанию.

#### Упражнение

Найдите самого легкого пингвина на Острове Мечты. Укажите только вид, остров и массу тела.

#### Решение

```
penguins |> 
  select(species, island, body_mass_g) |> 
  filter(island == 'Dream') |> 
  arrange(desc(body_mass_g))
```
```
## # Source:     SQL [?? x 3]
## # Database:   sqlite 3.45.2 [C:\Users\tonin\Documents\Courses\r-sql\data\penguins.db]
## # Ordered by: desc(body_mass_g)
##    species   island body_mass_g
##    <chr>     <chr>        <dbl>
##  1 Chinstrap Dream         4800
##  2 Adelie    Dream         4650
##  3 Adelie    Dream         4600
##  4 Chinstrap Dream         4550
##  5 Chinstrap Dream         4500
##  6 Adelie    Dream         4475
##  7 Adelie    Dream         4450
##  8 Chinstrap Dream         4450
##  9 Adelie    Dream         4400
## 10 Chinstrap Dream         4400
## # ℹ more rows
```

## Транчформация колонок

```
penguins |> 
  mutate(weight_kg = body_mass_g/1000)
```
```
## # Source:   SQL [?? x 8]
## # Database: sqlite 3.45.2 [C:\Users\tonin\Documents\Courses\r-sql\data\penguins.db]
##    species island    bill_length_mm bill_depth_mm flipper_length_mm body_mass_g
##    <chr>   <chr>              <dbl>         <dbl>             <dbl>       <dbl>
##  1 Adelie  Torgersen           39.1          18.7               181        3750
##  2 Adelie  Torgersen           39.5          17.4               186        3800
##  3 Adelie  Torgersen           40.3          18                 195        3250
##  4 Adelie  Torgersen           NA            NA                  NA          NA
##  5 Adelie  Torgersen           36.7          19.3               193        3450
##  6 Adelie  Torgersen           39.3          20.6               190        3650
##  7 Adelie  Torgersen           38.9          17.8               181        3625
##  8 Adelie  Torgersen           39.2          19.6               195        4675
##  9 Adelie  Torgersen           34.1          18.1               193        3475
## 10 Adelie  Torgersen           42            20.2               190        4250
## # ℹ more rows
## # ℹ 2 more variables: sex <chr>, weight_kg <dbl>
```

- `mutate` также изменяет `select`.
  - Именование новой переменной работает как оператор AS.
  - Это необязательно, но желательно.
- Если условие `select` не опредлене запрос вернет все колонки.

## Агрегация

```
penguins |> 
  group_by(species) |> 
  summarise(avg_body_mas = mean(body_mass_g))
```
```
## Warning: Missing values are always removed in SQL aggregation functions.
## Use `na.rm = TRUE` to silence this warning
## This warning is displayed once every 8 hours.

## # Source:   SQL [3 x 2]
## # Database: sqlite 3.45.2 [C:\Users\tonin\Documents\Courses\r-sql\data\penguins.db]
##   species   avg_body_mas
##   <chr>            <dbl>
## 1 Adelie           3701.
## 2 Chinstrap        3733.
## 3 Gentoo           5076.
```

 - `summarise` и `group by` работают вместе.
 - `dplyr` по умолчанию использует SQL для обработки пропущенных значений, если вы не используете аргумент `na.rm` в функциях агрегирования.

 #### Упражнение

 - Рассчитайте соотношение длины клюва к глубине клюва для каждого пингвина.
 - Используя предыдущий результат, рассчитайте среднее соотношение для каждого вида.

 ## Создание таблиц

```
 library(DBI)

another_connection <- dbConnect(RSQLite::SQLite(), ':memory:')

table1 <- tibble(
  person = c('mik', 'mik', 'mik', 'po', 'po', 'tay'),
  job = c('calibrate', 'clean', 'complain', 'clean', 'complain', 'complain')
)

dbWriteTable(another_connection, "work", table1, row.names = FALSE, overwrite = TRUE)

work <- tbl(another_connection, "work")
```

- Запишите, перезапишите или добавьте фрейм данных в таблицу базы данных с помощью `dbWriteTable`.
- `:memory:` — это «путь», который создает базу данных в памяти.

#### Упражнение


Создайте таблицу со значениями, указанными ниже.

|name	   |billable|
|----------|--------|
|calibrate |	1.5 |
|clean	   |    0.5 |


#### Решение
```
table2 <- tibble(
  name = c('calibrate', 'clean'),
  billable = c(1.5, 0.5)
)

dbWriteTable(another_connection, "job", table2, row.names = FALSE, overwrite = TRUE)

job <- tbl(another_connection, "job")
```


## Отрицание сделано неправильно

- Кто не калибрует?

```
work |> 
  filter(job != 'calibrate') |> 
  distinct(person)
```
```
## # Source:   SQL [3 x 1]
## # Database: sqlite 3.45.2 [:memory:]
##   person
##   <chr> 
## 1 mik   
## 2 po    
## 3 tay
```

- Как и в случае с чистым SQL, результат неправильный (Мик выполняет калибровку).
- Но использовать подзапросы с `dplyr` не так просто.

## SQL внутри R (Literal SQL)

```
work |> 
  filter(!person %in% sql(
    # subquery
    "(SELECT DISTINCT person FROM work
    where job = 'calibrate')"
  )) |> 
  distinct(person) 
```

- Вы можете использовать SQL внутри R.
- Это возвращает объект SQL.
- Полезно, когда кода R недостаточно для написания запроса.

## Объединение таблиц

```
does_calibrate <- work |> 
  filter(job == 'calibrate')

work |> 
  anti_join(does_calibrate, by = join_by(person)) |> 
  distinct(person) |> 
  collect() 
```
```
## # A tibble: 2 × 1
##   person
##   <chr> 
## 1 po    
## 2 tay
```

- Объединение таблиц с помощью `dplyr` работает так же, как и в SQL.
- Аргумент `by` работает так же как `on` в SQL.
- В этом случае мы делаем объединение с `person`.
- Используйте `left_join`, `right_join`, `internal_join` или `full_join`.

#### Упражнение

Подсчитайте, сколько часов отработал каждый человек, суммируя все работы.

#### Решение

```
work |> 
  left_join(job, by = join_by(job == name)) |> 
  group_by(person) |> 
  summarise(total_hours = sum(billable))
```
```
## # Source:   SQL [3 x 2]
## # Database: sqlite 3.45.2 [:memory:]
##   person total_hours
##   <chr>        <dbl>
## 1 mik            2  
## 2 po             0.5
## 3 tay           NA\
```


