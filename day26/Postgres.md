##### Pagination In SQL
- Proper way to implement pagination is to use index lookups.
- If you have multiple columns in your ordering clause, you can do that with the `row()` construct.

```
select lap, drivers.code, position,
        milliseconds * interval '1ms' as laptime
    from laptimes
        join drivers using(driverid)
    where raceid = 972
order by lap, position
fetch first 3 rows only;
```
- To get the next page of results
```
select lap, drivers.code, position,
        milliseconds * interval '1ms' as laptime
    from laptimes
        join drivers using(driverid)
    where raceid = 972
        and row(lap, position) > (1, 3)
order by lap, position
fetch first 3 rows only;
```
##### JOINs
- Inner Join of A and B gives the result of A `intersect` B - inner part of venn diagram intersection.
- Outer Join will give the results of A intersect B in addition to one of the following 
1. all of A (left join)
2. all of B (right join)
3. all of A and all of B (full join).

##### Aggregates
- Aggregates work by computing a digest value for several input rows at a time.
- With aggregates, we can return a summary containing many fewer rows tha passed the where filter.
###### Group By
- Group by clause introduces aggregates in SQL, and allows implementing much the same thing as map/reduce in other systems: map your data into different groups and in each group reduce the data set to a single value.

```
select extract('year' from date_trunc('decade', date)) as decade,
    count(*)
from races
group by decade
order by decade;
```

Let's attempt to compute difference between each decade

```
with races_per_decade
as (
    select extract('year' from date_trunc('decade', date)) as decade,
    count(*) as nbraces
from races
group by decade
order by decade    
)
select decade, nbraces as nbraces, 
    case
        when lag(nbraces, 1)
            over(order by decade) is null
        then ''

        when nbraces - lag(nbraces, 1) over(order by decade) < 0
        then format('-%3s', lag(nbraces,1) over (order by decade) - nbraces)
        else format('+%3s', nbraces - lag(nbraces,1) over(order by decade))
    end as evolution
from races_per_decade;

```
- `bool_and` aggregate starts with `true` and remains `true` only if every row it sees evaluates to `true`
- With this aggregate, it's possible to search for all drivers who failed to finish any single race they participated in over their whole career:
```
with counts as
(
    select driverid, forename, surname,
        count(*) as races,
        bool_and(position is null) as never_finished
    from drivers
        join results using(driverid)
        join races using(raceid)
    group by driverid
)
    select driverid, forename, surname, races
        from counts
        where never_finished
    order by races desc;
```
- If we want to find out if some seasons were less lucky than others on that basis and search for drivers who didn't finish a single race they participated into, per season:

```
with counts as
(
    select date_trunc('year', date) as year,
        count(*) filter(where position is null) as outs,
        bool_and(position is null) as never_finished
    from drivers
        join results using(driverid)
        join races using(raceid)
    group by date_trunc('year', date), driverid
)
    select extract(year from year) as season,
        sum(outs) as "#times any driver didn't finish a race"
    from counts
    where never_finished
    group by season
    order by sum(outs) desc
    limit 5;

```

- In this query, we see the aggregate `filter(where...)` syntax that allows us to update our computation only for those rows that pass the filter.
- Here, we choose to count all race results where the position is null, which means the driver didn't make it to the finish line for some reason.