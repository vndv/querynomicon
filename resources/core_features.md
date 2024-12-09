# Основные возможности SQL

## Агрегация

([онлайн демо](https://sqlize.online/sql/sqlite3_data/d208c0b539b4be11345c3a2e6ed8f9d2/))

```sql
select
    sum(body_mass_g) as total_mass
from penguins;
```
```
| total_mass |
|------------|
| 1437000    |
```

- Агрегация объединяет множество значений для получения одного
- Сумма — это функция агрегирования
- Объединяет соответствующие значения из нескольких строк

### Общие функции агрегирования

([онлайн демо](https://sqlize.online/sql/sqlite3_data/bcee1003096fc086d60954f2f6ad6a85/))

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

([онлайн демо](https://sqlize.online/sql/sqlite3_data/783908785044a5c9066acf9632e4ccc8/))

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

([онлайн демо](https://sqlize.online/sql/sqlite3_data/d1a1b06d5abbd587add37331d645b00c/))

```sql
select 
    avg(body_mass_g) as average_mass_g
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

([онлайн демо](https://sqlize.online/sql/sqlite3_data/23ec4ad50459f657b5ee0cfe5b3ce0af/))

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
| [null] | 4005.55555555556 |
| FEMALE | 3862.27272727273 |
| MALE   | 4545.68452380952 |
```

- Все строки в каждой группе имеют одинаковое значение пола, поэтому агрегировать не нужно.

### Произвольный выбор при агрегировании

([онлайн демо](https://sqlize.online/sql/sqlite3_data/6b20be9d42a236102d074f219291470b/))

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
| [null] | [null]      |
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

([онлайн демо](https://sqlize.online/sql/sqlite3_data/d2a2ad0c36bc566830a0e7a5e0a9065a/))

```sql
select
    sex,
    avg(body_mass_g) as average_mass_g
from penguins
group by sex
having average_mass_g > 4000.0;
```
```
| sex    |  average_mass_g  |
|--------|------------------|
| [null] | 4005.55555555556 |
| MALE   | 4545.68452380952 |
```

- Используйте `having` для фильтрации агрегированных значений вместо `where`.
- Команда `having` пишется сразу после `group by`.

#### Упражнения

1. Найдите острова со средней длиной плавника проживающих на них пингвинов меньше 200 милиметров.


### Читаемый вывод

([онлайн демо](https://sqlize.online/sql/sqlite3_data/0533124bddd2a3ce1007cb55123ace29/))

```sql
select
    sex,
    round(avg(body_mass_g), 1) as average_mass_g
from penguins
group by sex
having average_mass_g > 4000.0;
```
```
| sex    | average_mass_g |
|--------|----------------|
| [null] | 4005.6         |
| MALE   | 4545.7         |
```

- Используйте `round(значение, количество десятичных знаков)` для округления числа.

###  Фильтрация агрегируемых данных

([онлайн демо](https://sqlize.online/sql/sqlite3_data/b57970c1f981d1af69e177f418e20e30/))

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
```
|  sex   | average_mass_g |
|--------|----------------|
| [null] | 3362.5         |
| FEMALE | 3417.3         |
| MALE   | 3729.6         |
```

- Условие фильтрации применяется к входным данным агрегирующей функции.
> [!WARNING]
> Не все СУБД поддерживают такой синтаксис!


#### Упражнения

1. Напишите запрос, который использует фильтр для одновременного расчета средней массы тела тяжелых пингвинов (весом более 4500 граммов) и легких пингвинов (весом менее 3500 граммов). Можно ли это сделать, используя «where» вместо «filter»?

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

([онлайн демо](https://sqlize.online/sql/sqlite3_data/22173e8fc5b77d40add8716b72690474/))

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

- `create table имя таблицы`, за которым следует список столбцов в скобках
- Каждый столбец – это имя, тип данных и необязательная дополнительная информация.
- Например, `not null` предотвращает добавление нулей.
- Указание схемы в SQLite не является стандартом
- SQLite добавил несколько вещей
  - create if not exists
  - ключевые слова в верхнем регистре (SQL нечувствителен к регистру)


### Вставка данных

```sql
insert into job values
('calibrate', 1.5),
('clean', 0.5);
insert into work values
('mik', 'calibrate'),
('mik', 'clean'),
('mik', 'complain'),
('po', 'clean'),
('po', 'complain'),
('tay', 'complain');
```

|   name    | billable |
|-----------|----------|
| calibrate | 1.5      |
| clean     | 0.5      |
| person |    job    |
|--------|-----------|
| mik    | calibrate |
| mik    | clean     |
| mik    | complain  |
| po     | clean     |
| po     | complain  |
| tay    | complain  |

### Выполните следующие действия

- Скачайте примеры
- Зархивируйте файлы (UNZIP)
- Для создания баз выподните следующие команды
  - .read create_work_job.sql
  - .read populate_work_job.sql


#### Упражнение

Используя базу данных в памяти, определите таблицу под названием «заметки» с двумя текстовыми столбцами «автор» и «заметка», а затем добавьте три или четыре строки. Используйте запрос, чтобы проверить, что заметки сохранены и что вы можете (например) выбрать их по имени автора.

Что произойдет, если вы попытаетесь вставить в заметки слишком много или слишком мало значений? Что произойдет, если в поле примечания вместо строки вставить число?

### Обновление записей

```sql
update work
set person = 'tae'
where person = 'tay';
```

| person |    job    |
|--------|-----------|
| mik    | calibrate |
| mik    | clean     |
| mik    | complain  |
| po     | clean     |
| po     | complain  |
| tae    | complain  |

> [!WARNING]

> При обновлении записей в таблице всегда указывайте условие `where` в протоимвном случае вы обновите все записи в таблице


### Удаление записей

```sql
delete from work
where person = 'tae';
```

```sql
select * from work;
```

| person |    job    |
|--------|-----------|
| mik    | calibrate |
| mik    | clean     |
| mik    | complain  |
| po     | clean     |
| po     | complain  |

> [!WARNING]

> И снова, при удалении записей в таблице всегда указывайте условие `where`

#### Упражнение

Что произойдет, если вы попытаетесь удалить несуществующие строки (например, все записи в работе, относящиеся к Джуне)?


### Восстановление базы данных

```sql
create table backup (
    person text not null,
    job text not null
);
```

```sql
insert into backup
select
    person,
    job
from work
where person = 'tae';
```

```sql
delete from work
where person = 'tae';
```

```sql
select * from backup;
```
| person |   job    |
|--------|----------|
| tae    | complain |

- Далее мы рассмотрим еще одну стратегию основанную на [tombstone](glossary.md#надгробный-камень-tombstone)

#### Упражнение

Сохранение и восстановление данных в виде текста:

1. Воссоздайте таблицу примечаний в базе данных в памяти, а затем используйте команды SQLite .output и .dump, чтобы сохранить базу данных в файл с именем Notes.sql. Проверьте содержимое этого файла: как хранились ваши данные?

2. Запустите новый сеанс SQLite и загрузите Notes.sql с помощью команды .read. Проверьте базу данных с помощью .schema и выберите *: все ли так, как вы ожидали?

Сохранение и восстановление данных в двоичном формате:

1. Снова создайте таблицу примечаний в базе данных в памяти и используйте команду SQLite .backup, чтобы сохранить ее в файл с именем Notes.db. Проверьте этот файл с помощью od -c Notes.db или текстового редактора, который может обрабатывать двоичные данные: как хранились ваши данные?

2. Запустите новый сеанс SQLite и загрузите Notes.db с помощью команды .restore. Проверьте базу данных с помощью .schema и выберите *: все ли так, как вы ожидали?1.


### Проверка знаний

<img src="./assets/core_datamod_concept_map.svg" alt="Описание" style="max-width:100%; height:auto;">


## Объединение информации

```sql
select *
from work cross join job;
```

| person |    job    |   name    | billable |
|--------|-----------|-----------|----------|
| mik    | calibrate | calibrate | 1.5      |
| mik    | calibrate | clean     | 0.5      |
| mik    | clean     | calibrate | 1.5      |
| mik    | clean     | clean     | 0.5      |
| mik    | complain  | calibrate | 1.5      |
| mik    | complain  | clean     | 0.5      |
| po     | clean     | calibrate | 1.5      |
| po     | clean     | clean     | 0.5      |
| po     | complain  | calibrate | 1.5      |
| po     | complain  | clean     | 0.5      |
| tay    | complain  | calibrate | 1.5      |
| tay    | complain  | clean     | 0.5      |

- `JOIN` объединяет информацию из двух таблиц.

- `cross join` создает перекрестное (или декартово) произведение
- Все комбинации строк из каждой таблицы
Результат не особенно полезен: значения задания и имени не совпадают.
То есть в объединенных данных есть записи, части которых не связаны друг с другом.

### Внутренний джоин

```sql
select *
from work inner join job
    on work.job = job.name;
```

| person |    job    |   name    | billable |
|--------|-----------|-----------|----------|
| mik    | calibrate | calibrate | 1.5      |
| mik    | clean     | clean     | 0.5      |
| po     | clean     | clean     | 0.5      |

- Используйте обозначение table.column для указания столбцов.
- Столбец может иметь то же имя, что и таблица.
- Используйте `on` чтобы указать условие соединения таблиц.

#### Упражнение

Повторно запустите запрос, показанный выше, используя где job = name вместо полной записи table.name. Сокращенная форма легче или труднее читаться и с большей или меньшей вероятностью приведет к ошибкам?

### Агрегирование объединенных данных

```sql
select
    work.person,
    sum(job.billable) as pay
from work inner join job
    on work.job = job.name
group by work.person;
```

| person | pay |
|--------|-----|
| mik    | 2.0 |
| po     | 0.5 |


Объединяет идеи, которые мы видели раньше
Но Тэй отсутствует в таблице
В таблице должностей нет записей с именем Тэй
Поэтому нет записей, которые нужно группировать и суммировать.

### Левый джоин

```sql
select *
from work left join job
    on work.job = job.name;
```

| person |    job    |   name    | billable |
|--------|-----------|-----------|----------|
| mik    | calibrate | calibrate | 1.5      |
| mik    | clean     | clean     | 0.5      |
| mik    | complain  |           |          |
| po     | clean     | clean     | 0.5      |
| po     | complain  |           |          |
| tay    | complain  |           |          |


- Левое внешнее соединение сохраняет все строки из левой таблицы.
- Заполняет недостающие значения из правой таблицы нулевым значением

### Агрегированое левое соединений

```sql
select
    work.person,
    sum(job.billable) as pay
from work left join job
    on work.job = job.name
group by work.person;
```

| person | pay |
|--------|-----|
| mik    | 2.0 |
| po     | 0.5 |
| tay    |     |

- Так лучше, но нам бы хотелось видеть 0, а не пробел.

### Объединение значений

```sql
select
    work.person,
    coalesce(sum(job.billable), 0.0) as pay
from work left join job
    on work.job = job.name
group by work.person;
```

| person | pay |
|--------|-----|
| mik    | 2.0 |
| po     | 0.5 |
| tay    | 0.0 |

- Coalesce(val1, val2, …) возвращает первое ненулевое значение

### Полное внешнее соединение

- Полное внешнее соединение — это объединение левого внешнего соединения и правого внешнего соединения.
- Почти то же самое, что и перекрестное соединение, но учтите:

```sql
create table size (
    s text not null
);
insert into size values ('light'), ('heavy');

create table weight (
    w text not null
);
```

```sql
select * from size full outer join weight;
```

|   s   | w |
|-------|---|
| light |   |
| heavy |   |

- Полное внешнее соединение приведет к пустому результату

#### Упражнение

Найдите наименьшее время, которое каждый человек потратил на любую работу. Ваши выходные данные должны показать, что mik и po потратили на какую-то работу по 0,5 часа каждый. Можете ли вы найти способ показать название задания, используя SQL, который вы видели до сих пор?

### Проверка знаний

<img src="./assets/core_join_concept_map.svg" alt="Описание" style="max-width:100%; height:auto;">