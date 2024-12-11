# Управление данными в базе

## Создание новых таблиц из старых

```sql
create table new_work (
    person_id integer not null,
    job_id integer not null,
    foreign key (person_id) references person (ident),
    foreign key (job_id) references job (ident)
);

insert into new_work
select
    person.ident as person_id,
    job.ident as job_id
from person 
inner join work on person.name = work.person
inner join job on job.name = work.job;

select * from new_work;
```
```
| person_id | job_id |
|-----------|--------|
| 1         | 1      |
| 1         | 2      |
| 2         | 2      |
```
1. `new_work` - это новая таблица созданная на основе старой
2. Каждый столбец ссылается на запись в какой-либо другой таблице.

## Удаление таблиц

```sql
drop table work;

alter table new_work rename to work;

create table job (
    ident integer primary key autoincrement,
    name text not null,
    billable real not null
);

create table person (
    ident integer primary key autoincrement,
    name text not null
);

create table IF NOT EXISTS "work" (
    person_id integer not null,
    job_id integer not null,
    foreign key(person_id) references person(ident),
    foreign key(job_id) references job(ident)
);
```
1. Удалите старую таблицу и переименуйте новую, чтобы она заняла ее место.
2. Пожалуйста, сначала сделайте резервную копию ваших данных
3. Не забывайте про условие `IF NOT EXISTS`

### Упражнение

1. Реорганизуйте базу данных пингвинов:
    1. Сделайте копию файла penguins.db, чтобы ваши изменения не повлияли на оригинал.
    2. Напишите скрипт SQL, который реорганизует данные в три таблицы: по одной для каждого острова.
    3. Почему такая организация данных — плохая идея?

2. Такие инструменты, как [Sqitch](https://sqitch.org), могут управлять изменениями в схемах и данных базы данных, чтобы их можно было сохранить в системе контроля версий и откатить в случае неудачи. Переведите изменения, внесенные приведенными выше скриптами, в Sqitch. Примечание: это упражнение может занять час или больше.

## Индексы

## Внешние ключи