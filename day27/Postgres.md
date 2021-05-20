##### Restrict Select Groups: Having

```
\set season 'date ''1978-01-01'''

select status, count(*) 
    from results
        join races using(raceid)
        join status using(statusid)
    where date >= :season
    and    date < :season + interval '1 year'
    and position is null
group by status
having count(*) >= 10
order by count(*) desc;
```
- `having` filter the result set to only those groups that meet the `having` filtering condition, much as the `where` clause works for individual rows selected for the result set.

##### Grouping Sets

- A restriction with classic aggregates is that you can only run them through a single group definition at a time.
- In some cases, you want to be able to compute aggregates for several groups in parallel. For those cases, SQL provides the `grouping sets` feature.
```
\set season 'date ''1978-01-01'''

select drivers.surname as driver,
    constructors.name as constructor,
    sum(points) as points

from results
    join races using(raceid)
    join drivers using(driverid)
    join constructors using(constructorid)

where date >= :season
    and date < :season + interval '1 year'

group by grouping sets ((drivers.surname),
                        (constructors.name))
        having sum(points) > 20
order by constructors.name is not null,
        drivers.surname is not null,
        points desc;

```
- not clear to me -> shouldn't constructor group present before driver group? why is it other way around?

##### Common Table Expressions: With

```
with accidents as
(
    select extract(year from races.date) as season,
        count(*) as participants,
        count(*) filter(where status = 'Accident') as accidents
    from results
        join status using(statusid)
        join races using(raceid)
    group by season
)

select season,
    round(100 * accidents/participants, 2) as pct,
    repeat(text '■', ceil(100 * accidents/participants):: int) as bar
    from accidents
    where season between 1974 and 1990
order by season;
```
- Common Table Expression is the full name of the `with` clause that you see in the effect in the query.
- It allows us to run a subquery as a prologue, and then refer to its result set like any other relation in the `from` clause of the main query.
```
with points as 
(
    select year as season, driverid, constructorid,
            sum(points) as points
        from results join races using(raceid)
    group by grouping sets((year, driverid), (year, constructorid))
    having sum(points) > 0
    order by season, points desc
),
tops as
(
    select season,
        max(points) filter(where driverid is null) as ctops,
        max(points) filter(where constructorid is null) as dtops
    from points
    group by season
    order by season, dtops, ctops
),
champs as
(
    select tops.season,
        champ_driver.driverid,
        champ_driver.points,
        champ_constructor.constructorid,
        champ_constructor.points
    
    from tops
        join points as champ_driver
            on champ_driver.season = tops.season
            and champ_driver.constructorid is null
            and champ_driver.points = tops.dtops
        join points as champ_constructor
            on champ_constructor.season = tops.season
            and champ_constructor.driverid is null and champ_constructor.points = tops.ctops
)
select season,
    format('%s %s', drivers.forename, drivers.surname) as "Driver's Champion",
    constructors.name   as "Constructor's champion" from champs
            join drivers using(driverid)
            join constructors using(constructorid) order by season;
```
##### Union

- Query to list the drivers who received no points in race 972 despite having gotten some points in the previous race 971.
(
    select driverid,
        format('%s %s', drivers.forename, drivers.surname) as name
    from results
        join drivers using(driverid)
    where raceid = 972
        and points = 0
)
except
(
    select driverid,
        format('%s %s', drivers.forename, drivers.surname) as name
    from results
        join drivers using(driverid)
    where raceid = 971
        and points = 0
);
- `except` operator helps us to compute a difference in between two result sets.
#### Nulls

- `null` in sql means `I don't know what this is` rather than `no value here`.
- Say you have in A (lef􏰄 hand) something (hidden) that you don’t know what it is and in B (right hand) something (hidden) that you don’t know what it is. You’re asked if A and B are the same thing. Well, you can’t know that, can you?
- So in SQL `null = null`.
- The default value for any column, unless you specify something else is always `null`.
- It's only a default value though, not a constraint on your data model, so your applicaiton may insert a `null` value in a column with a `non null` default.
- null is three-value logic.

#### Window Functions

- The idea behind `window functions` is to allow you to process several values of the result set at a time - you see through the window some `peer` rows and you are able to compute a single output value from them, much like when using an `aggregate` function.
- First step we are going through here is understanding what data the function has access to.
- For each input row, you have access to a `frame of data`.
- `frame_clause` defines a subset of rows in the current partition to which the window function is applied.
- Other frames are possible to define when using the clause `partition by`.
- `partition by` allows defining as `peer rows` those rows that share a common property with the `current row`, and the property is defined as a `partition`.
- Eg.
Query to fetch the list of competing drivers in their position order (winner first), and also their ranking compared to other drivers from the same constructor in the race.
```
select surname,
    constructors.name,
    position,
    format('%s / %s', 
        row_number()
            over(partition by constructorid 
                order by position nulls last), 
                count(*) over (partition by constructorid)
    )as "pos same constr"
from results
    join drivers using(driverid)
    join constructors using(constructorid)
where raceid = 890
order by position;
```
- Window functions only happens after the `where` clause, so you only get to see rows from the `available result set of the query`.

Tomorrow review -> https://www.postgresqltutorial.com/postgresql-window-function/

