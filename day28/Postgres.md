- Any and all aggregate function can be used against a `window frame` rather than a `grouping clause`
```
select surname,
    position as pos,
    row_number()
        over(order by fastestlapspeed::numeric)
            as fast,
        ntile(3) over w as "group",
        lag(code, 1) over w as "prev",
        lead(code, 1) over w as "next"
    from results
        join drivers using(driverid)
    where raceid = 890
window w as (order by position)
order by position;
```
- We are reusing the same `window definition` several times hence giving it a name to simplify the query.
- For each driver, we are fetching his position in the results, his position in terms of `fastest lap speed`, a `group` number if we divide the drivers into a set of four groups - `ntile` function.

- Real magic of what are called `window functions` is actually the frame of data they can see when using the `OVER ()` clause. This frame is specified thanks to `PARTITION BY` and `ORDER BY` clauses.
- Windowing clauses are always considered last in the query, meaning after the `where` clause. 
- In any frame, you can only see the rows that have been `selected for the output`.
- Use `window functions` whenever you want to compute values for each row of the result set and those computations depend on other rows within the same result set.
- Classic example is a marketing analysis of weekly results - you typically put boht each day's gross sales and the variation with the same day in comparision to the previous week.

-----
- `window` describes the set of rows on which the window function operates. A window function returns values from the rows in a window.
```
select product_name, price, group_name,
    AVG(price) over (partition by group_name)
from products
    inner join 
        product_groups using (group_id);
```
- In the above, `avg()` function works as a `window function` that operates on a set of rows specified by the `OVER` clause. Each set of rows is called a window.
- Window function always performs the calcuation on the result set after the join, where, group by and having clause and before the final `order by` clause in the evaluation order.

- first_value() -> function returns a value evaluated against the first row within its parition
- last_value() -> last row within its partition 
```
SELECT
	product_name,
	group_name,
	price,
	FIRST_VALUE (price) OVER (
		PARTITION BY group_name
		ORDER BY
			price
	) AS lowest_price_per_group
FROM
	products
INNER JOIN product_groups USING (group_id);```

```
SELECT
	product_name,
	group_name,
	price,
	LAST_VALUE (price) OVER (
		PARTITION BY group_name
		ORDER BY
			price RANGE BETWEEN UNBOUNDED PRECEDING
		AND UNBOUNDED FOLLOWING
	) AS highest_price_per_group
FROM
	products
INNER JOIN product_groups USING (group_id);
```
- In the query above, the frame clause is `range between unbounded preceding and unbounded following` because by default the frame clause is `range between unbounded preceding and current row`
