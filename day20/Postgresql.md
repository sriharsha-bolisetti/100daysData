- Database model has to be statically declared so that we know the `type` of every bit of data involved at the time the query is carried out.
- A query result set defines a `relation`, of a `type determined when parsing the query`.

Data Type Formatting Functions

- `to_char` -> Certain patterns that are recognized and replaced with appropriately-formatted data based on the given value.
- Input template string, template patterns identify the values to be supplied by the input data string
- Postgresql implements a protocol level facility to send the static SQL query text separately from its dynamic arguments.
- SQL injection happens when the database server is mistakenly led to consider a `dynamic argument of a query as part of the query text`.
Server Side Prepared Statements

- Parse
- Bind 
- Execute

- When postgresql parses the query it doesn't contain parameters.
- SQL Injection happens when the SQL parser is fooled into believing that a parameter string is in fact a SQL query, and then the SQL engine goes on and executes that SQL statement.
- When the SQL query string lives in your application code, and the user-supplied parameters are sent separately on the network, there's no way a SQL parsing engine might get confused.

- `coalesce` -> returns first of the arguments that is not null
- `generate_series()` -> Generates a series of values, from start to stop with a step size of `step`, this is inclusive like BETWEEN operator.
- `cast(calendar.entry as date)` expression transforms the generated calendar.entry which is the result of generate_series function call into date data type.
- Cast is needed as `generate_series()` function returns a set of timestamp entries and we don't care about the time parts of it.
##### Window functions 
- Performs calculation across a set of table rows that are somehow related to the current row. 
- Similar to aggregate function, but unlike aggregate functions, use of a window function does not cause rows to become grouped into a single output row - the rows retain their separate identities.

```
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
```

First three output columns come directly from the table `empsalary`, and there is one output row for each row in the table.
The fourth column represents an average taken across all the table rows that have same depname value as the current row.

- A window function call always contains an `OVER` clause directly following the window function's name and arguments. This is what syntactically distinguishes it from a regula,r function or aggregate function.
- The `OVER` clause determines exactly how the rows of the query are split up for processing by the window function.


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

        generate_series(date: 'start' - interval '1 week', date: 'start' + interval '1 month' - interval '1 day') as calendar(date)
        left join factbook using(date)
)

```
-- This needs to be digged into more - i didn't understand anything!