
`lag` - function to access a row which comes before the current row at a specific physical offset.

- LAG(expression [, offset [, default_value]])
OVER(
    [PARTITION BY partition_expression, ...]
    ORDER BY sort_expression [ASC | DESC], ...
)

- expression is evaluated against the row that comes before the current row at a specified offset.
- offset is a positive integer that specifies the number of rows which comes before the current row from which to access data.
- Partition By - Clause divides rows into partitions to which the `LAG()` function is applied.
- By default, the function will treat the whole result set as a single partition if you omit the Partition By clause
- Order By clause specifies the order of the rows in each partition to which the `LAG()` function is applied.

Eg.

```
WITH cte AS (
    SELECT year, SUM(amount) amount
    FROM sales
    GROUP BY year
    ORDER BY year
)

SELECT year, amount, 
LAG(amount,1) OVER (ORDER BY year) previous_year_sales

FROM 

cte;

```

- In the above sql snippet, CTE returns net sales summarized by year.
- Then, the outer query uses the `LAG()` function to return the sales of the previous year for each row. 

- Below snippet uses two common table expressions to return the sales variance between current and previous years:

```
WITH cte AS (
    SELECT year, SUM(amount) amount
    FROM sales
    GROUP BY year
    ORDER BY year
), cte2 AS (
    SELECT year, amount, 
    LAG(amount,1) OVER (ORDER BY year) previous_year_sales
    FROM 
    cte
)
SELECT year, amount, previous_year_sales, (previous_year_sales - amount ) variance
FROM cte2;
```

- LAG() function over partition
```
SELECT year, amount, group_id,
LAG(amount,1) OVER (PARTITION BY group_id ORDER BY year) previous_year_sales
FROM sales;
```

- I didn't understand the above query completely.
```
 year | amount  | group_id | previous_year_sales
------+---------+----------+---------------------
 2018 | 1474.00 |        1 |
 2019 | 1915.00 |        1 |             1474.00
 2020 | 1646.00 |        1 |             1915.00
 2018 | 1787.00 |        2 |
 2019 | 1911.00 |        2 |             1787.00
 2020 | 1975.00 |        2 |             1911.00
 2018 | 1760.00 |        3 |
 2019 | 1118.00 |        3 |             1760.00
 2020 | 1516.00 |        3 |             1118.00
 ```

 while i understand why the table output is the way it is, why should it be sorted??

 ----------------------------------

 ```
with computed_data as
(
    select cast(date as date) as date,
    to_char(date, 'Dy') as day,
    coalesce(dollars, 0) as dollars,
    lag(dollars,1)
    over(partition by extract('isodow' from date) order by date)
    as last_week_dollars
    
    from

        generate_series(date :'start' - interval '1 week', date :'start' + interval '1 month' - interval '1 day', '1 day') as calendar(date)
        left join factbook using(date)
    )
     select date, day,
     to_char(coalesce(dollars, 0), 'L99G999G999G999') as dollars,
     case when dollars is not null and dollars <> 0
     then round( 100.0 * (dollars - last_week_dollars)/2, 2)
     end
     as "WOW %"
     from computed_data
     where date >= date :'start'
     order by date;

 ```

 - what is `WoW %`??