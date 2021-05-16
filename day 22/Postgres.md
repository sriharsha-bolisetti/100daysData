- Transaction should be atomic and isolated.
- RDBMS is tasked with maintaining a dataset that is consistent with business logic all the times.
- SQL language is statically typed: every query defines a new relation that must be fully understood by the system before executing it. That's why sometimes `cast` expressions are needed in your queries.
- Postgresql approach to implementing SQL was invented in the 80s with the stated goal of enabling extensibility. SQL operators and functions are defined in a catalog and looked up at run-time.
- Functions and operators in Postgresql support `polymorphism` and almost every part of system can be extended.
- Think of POstgresql not as `storage` layer but rather as a `concurrent data access service`.

- The main aspects to consider in terms of where to maintain the business logic are `correctness` and the `efficiency` aspects of your code architecture and organisation.

#### Correctness

- Read uncommitted -> Postgresql accepts this setting and actually implements `read committed` here.
- Read Committed -> deafult setting, allows your transaction to see other transactions changes as soon as they are committed.
- It means that if you run the following query twice in your transaction but someone else added or removed objects from the stock, you will have different counts at different points in your transaction.

```
SELECT count(*) FROM stock;
```

- Repeatable read -> In this isolation level, your transaction keeps the same `snapshot` of the whole database for its entire duration, from BEGIN to COMMIT. It is very useful to have that for online backups
- Serializable -> This level guarantees that a one-transaction-at-a-time ordering of what happens on the server exists with the exact same result as what you are obtaining with concurrent activity.
- When using Postgresql every query always runs within a single consistent snapshot. The isolation level impacts reusing a snapshot from one query to the next.

##### Stored Procedures

- Its possible to create server-side functions.
- ????? lateral join technique???? wth is this???
- Lateral JOIN
- Subqueries appearing in FROM can be preceded by keyword LATERAL. This allows them to reference columns provided by preceding FROM items.
- Without LATERAL each subquery is evaluated independently and so cannot cross-reference any other FROM item.
- Another better def. LATERAL keyword allows us to access columns after the FROM statement and reference these columns earlier in the query.
- If its within a JOIN tree, it can also refer to any items that are on the left-hand side of a JOIN that is on the right-hand side of.

Example -

Without Lateral

```
select
    (pledged / fx_rate) as pledged_usd,
    (pledged / fx_rate) / backers_count as avg_pledge_usd,
    (goal / fx_rate) - (pledged / fx_rate) as amt_from_goal,
    (deadline - launched_at) / 86400.00 as duration,
    ((goal / fx_rate) - (pledged / fx_rate)) / ((deadline - launched_at) / 86400.00) as usd_needed_daily
from kickstarter_data;
```

After Lateral

```
select

    pledged_usd,
    avg_pledge_usd,
    amt_from_goal,
    duration,
    (usd_from_goal / duration ) as usd_needed_daily

from kickstarter_data
    lateral ( select pledged/ fx_rate as pledged_usd )pu
    lateral ( select pledged_usd / backers_count as avg_pledge_usd ) apu
    lateral ( select goal/fx_rate as goal_usd ) gu
    lateral ( select goal_usd - pledged_usd ) gu
    lateral ( select (deadline - launched_at)/86400.00 as duration) dr;

```

With lateral joins, calculation can be defined once and reference these calculations in other parts of query.

-

```
select album, duration
from artist, lateral get_all_albums(artistid)
where artist.name = 'Red Hot Chili Peppers';
```

```
with four_albums as
(
    select artistid
    from album
    group by artistid
    having count(*) = 4
)
  select artist.name, album, duration
    from four_albums
        join artist using(artistid),
        lateral get_all_albums(artistid)
order by artistid, duration desc;

```
