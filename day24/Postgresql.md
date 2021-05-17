#### Parts of SQL

- DML
- DDL -> Data Definition Language, it covers `create`, `alter` and `drop` statements, which are used to define on-disk data structures where to hold the data, and also their constraints and indexes - the things we refer to with the terms of SQL objects.
- TCL -> Transaction Control Language and includes `begin` and `commit` statements, and also `rollback`, `start transaction` and `set transaction` commands. It also includes less well-known `savepoint`, `release savepoint` and `rollback to savepoint` commands, let's not forget about the two phase commit protocol with `prepare commit`, `commit prepared` and `rollback prepared` commands.
- DCL -> starts for `data control language` and is covered with the statements `grant` and `revoke`.
- Postgresql maintainence commands such as `vaccum`, `analyze`, `cluster`.
- More commands such as `prepare`, `execute`, `listen`, `explain`, `notify`, `lock`, `set`.

#### Query Part of SQL

- instead of using `select * from races fetch first 1 rows only;`, you can use `table races limit 1;`.
- Processing functions
```
select date::date,
    extract('isodow' from date ) as dow,
    to_char(date, 'dy') as day,
    extract('isoyear' from date) as "iso year",
    extract('week' from date) as week,
    extract('day' from (date + interval '2 month - 1 day')) as feb,
    extract('year' from date) as year,
    extract('day' from (date + interval '2 month - 1 day')) = 29 as leap

from
    generate_series(date '2000-01-01', date '2010-01-01', interval '1 year') as t(date);
```

- Left join semantics are to keep the whole result set of the table lexically on the left of the operator, and to fill-in the columns for the table on the right of the `left join` operator when some data is found that matches the `join condition`, otherwise using NULL as the column's value.
```
\set beginning '2017-04-01'
\set months 3

select date, name, drivers.surname as winner
    from races
        left join results
             on results.raceid = races.raceid
             and results.position = 1
        left join drivers using(driverid)
    where date >= date :'beginning' and   date < date :'beginning' + :months * interval '1 month';

```
`results.position = 1` restriction has been moved directly into the join condition, rather than being kept in the where clause. Should the condition be in the `where` clause, it would `filter out races from which don't have a result yet, and we are interested in those`.

- Another way to write this query would be using an explicit subquery to build an intermediate results table containing only the winners and then join against that:
```
select date, name, drivers.surname as winner
    from races
        left join
            (
                select raceid, driverid
                from results
                where position = 1
            )
            as winners using(raceid)
        left join drivers using(driverid)
    where date >= date :'beginning'
        and date < date :'beginning' + :months * interval '1 month';
```

- `anti-join` pattern can be implemented with help of `not exists` feature.
- `anti-join` meant to keep only the rows that fail a test.
- Let's say if we want to list the drivers that were unlucky enough to not finish a single race in which they participated, then we can filter out those who did finish. We know that a driver finished because their `position` is filled in the `results` table: it is not null.

```
\set season 'date ''1978-01-01'''
select forename, surname,
    constructors.name as constructor, 
    count(*) as races,
    count(distinct status) as reasons
    
    from drivers
        join results using(driverid)
        join races using(raceid)
        join status using(statusid)
        join constructors using(constructorid)
    
    where date >= :season
        and date < :season + interval '1 year' 
        and not exists
        (
            select 1
            from results r
            where position is not null
            and r.driverid = drivers.driverid and r.resultid = results.resultid
        )
group by constructors.name, driverid
order by count(*) desc;
```

- What is `select 1` doing here?
Remember that a where clause is a filter. The `not exists` clause is filtering based on rows that are returned by the subquery. To pass the filter, just return anything, Postgresql will not even look at what is selected in subquery, it will only take into account the fact that a row was returned.
- I def. need to look at this again, aggregations are throwing me off.