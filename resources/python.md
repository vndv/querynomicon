# Python

> Список терминов: [Курсор (cursor)](/resources/glossary.md?id=Курсор-cursor), [Объектно-реляционная модель (Object-Relational Mapper, ORM)](/resources/glossary.md?id=Объектно-реляционная-модель-object-relational-mapper-orm), [Стандартный идентификатор ресурса (Uniform Resource Identifier - URI)](/resources/glossary.md?id=Объектно-реляционная-модель-object-relational-mapper-orm)

## Запросы из Python

```python
import sqlite3
import sys

db_path = sys.argv[1]
connection = sqlite3.connect(db_path)
cursor = connection.execute("select count(*) from penguins;")
rows = cursor.fetchall()
print(rows)
```
```
[(344,)]
```

- sqlite3 является частью стандартной библиотеки Python.
- Создайте соединение с файлом базы данных
- Получите курсор, выполнив запрос к базе данных
- Чаще всего создается курсор и используется для выполнения запросов.
- Получить все строки одновременно в виде списка кортежей

## Инкрементная выборка

```python
import sqlite3
import sys

db_path = sys.argv[1]
connection = sqlite3.connect(db_path)
cursor = connection.cursor()
cursor = cursor.execute("select species, island from penguins limit 5;")
while row := cursor.fetchone():
    print(row)
```
```
('Adelie', 'Torgersen')
('Adelie', 'Torgersen')
('Adelie', 'Torgersen')
('Adelie', 'Torgersen')
('Adelie', 'Torgersen')
```

- cursor.fetchone возвращает None, если данных больше нет
- Существует также fetchmany(N) для выборки (до) определенного количества строк.

## Вставка, удаление


```python
import sqlite3

connection = sqlite3.connect(":memory:")
cursor = connection.cursor()
cursor.execute("create table example(num integer);")

cursor.execute("insert into example values (10), (20);")
print("after insertion", cursor.execute("select * from example;").fetchall())

cursor.execute("delete from example where num < 15;")
print("after deletion", cursor.execute("select * from example;").fetchall())
```
```
after insertion [(10,), (20,)]
after deletion [(20,)]
```

- Каждое выполнение — это отдельная транзакция.

## Интерполяция значений

```python
import sqlite3

connection = sqlite3.connect(":memory:")
cursor = connection.cursor()
cursor.execute("create table example(num integer);")

cursor.executemany("insert into example values (?);", [(10,), (20,)])
print("after insertion", cursor.execute("select * from example;").fetchall())
```
```
after insertion [(10,), (20,)]
```

<img src="./assets/xkcd_327_exploits_of_a_mom.png" alt="XKCD Exploits of a Mom" style="max-width:100%; height:auto;">


#### Упражнение

Напишите Python скрипт, который принимает значения острова, вида, пола и других значений в качестве аргументов командной строки и вставляет запись в базу данных пингвинов.


## Выполнение скрипта

```python
import sqlite3

SETUP = """\
drop table if exists example;
create table example(num integer);
insert into example values (10), (20);
"""

connection = sqlite3.connect(":memory:")
cursor = connection.cursor()
cursor.executescript(SETUP)
print("after insertion", cursor.execute("select * from example;").fetchall())
```
```
after insertion [(10,), (20,)]
```

- 
Но что, если что-то пойдет не так?


## Исключения SQLite в Python

```python
import sqlite3

SETUP = """\
create table example(num integer check(num > 0));
insert into example values (10);
insert into example values (-1);
insert into example values (20);
"""

connection = sqlite3.connect(":memory:")
cursor = connection.cursor()
try:
    cursor.executescript(SETUP)
except sqlite3.Error as exc:
    print(f"SQLite exception: {exc}")
print("after execution", cursor.execute("select * from example;").fetchall())
```
```
SQLite exception: CHECK constraint failed: num > 0
after execution [(10,)]
```


## Python в SQLite

```python
import sqlite3

SETUP = """\
create table example(num integer);
insert into example values (-10), (10), (20), (30);
"""


def clip(value):
    if value < 0:
        return 0
    if value > 20:
        return 20
    return value


connection = sqlite3.connect(":memory:")
connection.create_function("clip", 1, clip)
cursor = connection.cursor()
cursor.executescript(SETUP)
for row in cursor.execute("select num, clip(num) from example;").fetchall():
    print(row)
```
```
(-10, 0)
(10, 10)
(20, 20)
(30, 20)
```

- SQLite делает `callback` в Python для выполнения функции
- Другие базы данных могут запускать Python в процессе сервера базы данных.
- Будте осторожны


## Обработка дат и времени

```python
from datetime import date
import sqlite3


# Преобразование даты в строку в формате ISO при записи в базу данных.
def _adapt_date_iso(val):
    return val.isoformat()


sqlite3.register_adapter(date, _adapt_date_iso)


# Преобразование строки в формате ISO в дату при чтении из базы данных.
def _convert_date(val):
    return date.fromisoformat(val.decode())


sqlite3.register_converter("date", _convert_date)

SETUP = """\
create table events(
    happened date not null,
    description text not null
);
"""

connection = sqlite3.connect(":memory:", detect_types=sqlite3.PARSE_DECLTYPES)
cursor = connection.cursor()
cursor.execute(SETUP)

cursor.executemany(
    "insert into events values (?, ?);",
    [(date(2024, 1, 10), "started tutorial"), (date(2024, 1, 29), "finished tutorial")],
)

for row in cursor.execute("select * from events;").fetchall():
    print(row)
```
```
(datetime.date(2024, 1, 10), 'started tutorial')
(datetime.date(2024, 1, 29), 'finished tutorial')
```

- sqlite3.PARSE_DECLTYPES сообщает библиотеке sqlite3 использовать преобразования на основе объявленных типов столбцов.


#### Упражнение

Напишите адаптер Python, который усекает реальные значения до двух десятичных знаков при их записи в базу данных.

## SQL в Jupyter notebooks

```shell
pip install jupysql
```

- А потом внутри блокнота:

```shell
%load_ext sql
```

- Подключите базу

```shell
%sql sqlite:///db/penguins.db
```
```
Connecting to 'sqlite:///data/penguins.db'
```

- Подключаемся к базе данных
  - sqlite:// с двумя косыми чертами — это протокол
  - /data/penguins.db (одна косая черта) — локальный путь.
- Одиночный знак процента %sql представляет однострочную команду
- Используйте двойной знак процента %%sql, чтобы указать, что остальная часть ячейки представляет собой SQL.

```python
%%sql
select species, count(*) as num
from penguins
group by species;
```
```
Running query in 'sqlite:///data/penguins.db'
```

|species|	num|
|-------|------|
|Adelie |	152|
|Chinstrap|	68|
|Gentoo |	124|


## Pandas и SQL

```shell
pip install pandas
```
```python
import pandas as pd
import sqlite3
import sys

db_path = sys.argv[1]
connection = sqlite3.connect(db_path)
query = "select species, count(*) as num from penguins group by species;"
df = pd.read_sql(query, connection)
print(df)
```
```
     species  num
0     Adelie  152
1  Chinstrap   68
2     Gentoo  124
```

- Будьте осторожны с преобразованием типов данных при использовании Pandas.

#### Упражнение

Напишите скрипт Python для командной строки, который использует Pandas для воссоздания базы данных пингвинов.

## Polars и SQL

```shell
pip install polars pyarrow adbc-driver-sqlite
```
```python
import polars as pl
import sys

db_path = sys.argv[1]
uri = "sqlite:///{db_path}"
query = "select species, count(*) as num from penguins group by species;"
df = pl.read_database_uri(query, uri, engine="adbc")
print(df)
```
```
shape: (3, 2)
┌───────────┬─────┐
│ species   ┆ num │
│ ---       ┆ --- │
│ str       ┆ i64 │
╞═══════════╪═════╡
│ Adelie    ┆ 152 │
│ Chinstrap ┆ 68  │
│ Gentoo    ┆ 124 │
└───────────┴─────┘
```

- Единый идентификатор ресурса (URI) указывает базу данных.
- Используйте движок ADBC ​​вместо ConnectorX по умолчанию с Polars.

#### Упражнение

Напишите скрипт Python для командной строки, который использует Polars для воссоздания базы данных пингвинов.

## Объектно-реляционные модели (Object-Relational Mappers)

```python
from sqlmodel import Field, Session, SQLModel, create_engine, select
import sys


class Department(SQLModel, table=True):
    ident: str = Field(default=None, primary_key=True)
    name: str
    building: str


db_uri = f"sqlite:///{sys.argv[1]}"
engine = create_engine(db_uri)
with Session(engine) as session:
    statement = select(Department)
    for result in session.exec(statement).all():
        print(result)
```
```
building='Chesson' name='Genetics' ident='gen'
building='Fashet Extension' name='Histology' ident='hist'
building='Chesson' name='Molecular Biology' ident='mb'
building='TGVH' name='Endocrinology' ident='end'
```

- Объектно-реляционные модели (ORM) преобразует столбцы таблицы в свойства объекта и наоборот.
- SQLModel использует подсказки типов `(type hints)` Python.


#### Упражнение

Напишите скрипт Python для командной строки, который использует SQLModel для воссоздания базы данных Penguins.


## Отношения с ORM

```python
class Staff(SQLModel, table=True):
    ident: str = Field(default=None, primary_key=True)
    personal: str
    family: str
    dept: Optional[str] = Field(default=None, foreign_key="department.ident")
    age: int


db_uri = f"sqlite:///{sys.argv[1]}"
engine = create_engine(db_uri)
SQLModel.metadata.create_all(engine)
with Session(engine) as session:
    statement = select(Department, Staff).where(Staff.dept == Department.ident)
    for dept, staff in session.exec(statement):
        print(f"{dept.name}: {staff.personal} {staff.family}")
```
```
Histology: Divit Dhaliwal
Molecular Biology: Indrans Sridhar
Molecular Biology: Pranay Khanna
Histology: Vedika Rout
Genetics: Abram Chokshi
Histology: Romil Kapoor
Molecular Biology: Ishaan Ramaswamy
Genetics: Nitya Lal
```

- Сделайте внешние ключи явными в определениях классов.
- SQLModel автоматически выполняет соединение (join)
  - Два сотрудника без отдела не включены в результат.

