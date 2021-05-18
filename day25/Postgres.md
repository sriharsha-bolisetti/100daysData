- Order by clause can also refer to query aliases and computed values. 

```
select  drivers.code, drivers.surname,
    position,
    laps, status
    from results
        join drivers using(driverid)
        join status using(statusid)
    where raceid = 972
order by position nulls last,
        laps desc,
        case when status = 'Power Unit'
            then 1
            else 2
        end;
```

A more complex expression, using `CASE` conditional in order to control the reace's results over the status information. Here we are ordering the results by position then number of laps and then by status with a special rule: the Power Unit failure condition is considered first, and then the other ones.

###### kNN ordering and GiST indexes
- Another use case for `order by` is to implement `k nearest neighbours`. 
- Query to find out the ten nearest circuits to Paris, France, which is at longitude 2.349014 and latitude 48.864716. That's a kNN search with k= 10;
```
select name, location, country
    from circuits
order by point(lng, lat) <-> point(2.349014, 48.864716)
limit 10;
```
- `point` datatype is a very useful PostgreSQL addition.
-  In the query above, the points have been computed from raw data in the database. 
- For a proper PostgreSQL experience, we can have a location column of point type in our circuits table and index it using GiST:

```
begin;

alter table f1db.circuits add column position point;
update f1db.circuits set position = point(lng, lat);
create index on f1db.circuits using gist(position);

commit;

```

Now the previous query can be written using the new column. 

```
explain (costs off, buffers, analyze)
    select name, location, country
        from circuits
order by position <-> point(2.349014, 48.864716)
    limit 10;
```

- The distance operator `<->` is defined only for geometric data types in PostgreSQL.

###### Top-N sorts: Limit

- `rank()` -> function assin a rank for every row within a partition of a result set.
- For each partition, the rank of the first row is 1.
- `rank()` function adds the number of tied rows to the tied rank to calculate the rank of the next row, so the ranks may not be sequential. In addition, rows with same values will get the same rank.
```
RANK() OVER (
    [PARTITION BY partition_expression, ... ]
    ORDER BY sort_expression [ASC | DESC], ...
)
```
- `Partition By` clause distributes rows of the result set into paritions to which `Rank()` is applied.
- Then, the `Order By` clause specifies the order of rows in each partition to which funciton is applied.

- https://www.postgresqltutorial.com/postgresql-rank-function/ helpful guide to understand rank() function.
####### Postgresql cross join
- A cross join clause allows you to produce a cartesian product of rows in two or more tables.
- Cross Join clause does not have a join predicate.
- If T1 has n rows and T2 has m rows, the result set will have nxm rows. For example, the T1 has 1,000 rows and T2 has 1,000 rows, the result set will have 1,000 x 1,000 = 1,000,000 rows.
- ```SELECT select_list
FROM T1
CROSS JOIN T2;
```
- Inner Join clause with a condition that always evaluates to true to simulate the cross join
```
select *
from t1
inner join t2 on true```
- FROM T1 CROSS JOIN T2 is equivalent to FROM T1 INNER JOIN T2 ON TRUE (see below). It is also equivalent to FROM T1, T2.
- Lateral join is more like a correlated subquery, in that expressions to the right of lateral join are evaluated once for each row left of it - just like correlated subquery - while a plain subquery is evaluated only once.

```
with decades as
(
    select extract('year' from date_trunc('decade', date)) as decade
        from races
    group by decade
)
select decade, 
    rank() over(partition by decade order by wins desc) as rank, 
    forename, surname, wins

    from decades
        left join lateral
        (
            select code, forename, surname, count(*) as wins
                from drivers
                 join results
                    on results.driverid = drivers.driverid
                    and results.position = 1
                join races using(raceid)
            where extract('year' from date_trunc('decade', races.date)) = decades.decade
            group by decades.decade, drivers.driverid
            order by wins desc
                limit 2
        ) as winners on true
 order by decade asc, wins desc;
 ```

- Query extracts the decade first, in a common table expression introduced with the `with` keyword.
- This CTE is then reused as a data source in the `from` clause.
- The `from`lause is about relations, which might be hosting a dynamically computed dataset, as is the case in the above example.
- Once we have our list of decades from the dataset, we can fetch for each decade the list of the top three winners for each decade from the results table. 
- The way done above is using a `lateral` join.
- Lateral Join allows you to write a subquery that runs in a loop over a data set.
- Here we loop over the decades and for each decade our lateral subquery finds the top three winners.
- Focussing now on the winners subquery, we want to count how many times a driver made it to the first position in a race.
- As we are only interested in winning results, the query pushes that restriction in the `join condition` of the `left join results` part.
- The subquery should only count victories that happened in the current decade from our loop, and that's implemented in the `where` clause, because that's how `lateral` subqueries work.
- Another interesting implication of using a `left join lateral subquery` is how the join clause is then written: `on true`.
- That's because we inject the join condition right into the subquery as a where clause.
- This trick allows us to only see the results from the current decade in the subquery, which then uses a `limit` clause on top of the `order by wins desc` to report the top three with the most wins.
