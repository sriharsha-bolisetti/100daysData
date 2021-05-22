### Relations

- Relation is a set of data all having a common set of properties, that is to say a `set of elements all from the same composite data type`.
- We can think of sql set as a bag rather than a set, because `duplicates` are allowed in SQL relations.
- Data types are defined by the `create type` statement or by the more common `create table` statement.
```
create table relation(id integer, f1 text, f2 date, f3 point); CREATE TABLE
```
- Here we created a table named `relation`.
- In the background, postgresql created a type with the same name that you can manipulate or reference.
- `select` statement here is returning tuples of the composite type `relation`.
- Where you use a subquery in your main query, either in the form of a `common table expression` or directly inlined in your `from` clause, you are effectively defining a `relation data type`.
- At query run time, this relation is filled with a dataset, thus you have a full-blown relation to use.
- Relational algegra is thereby a formalism of what you can do with such things.
- The result of a join in between two relations is a relation, and that relation can in-turn participates into other `join` operations.
- The result of a `from` clause is a relation, with which the query planner is executing the rest of your query: the `where` clause to restrict the relation dataset to what's interesting for the query, and other clauses, up until the `window functions` and the `select` projects are computed so that we can finally construct the `result set` i.e. a `relation`.
#### SQL Join Types

- Joins are the basic operations you do with relations.
- The natuer of a join is to `build a new relation from a pair of existing ones`.
- Most basic join is `cross join` or cartesian product.
- Other kind of join associate data between the two relations that participate in the operation.
- The association is specified in the `join condition` and is usually processed based on some equality operator but is not limited to that.

### Relational Theory
`date '2020-05-22'` -> This input syntax is named as `decorated literal`.
- We decorate the literal with its data type so that Postgresql doesn't have to guess what it is.

### Postgres Data Types

1. Boolean
- `=` doesn't work as you think it would.
- Use `is` to test against literal `true`, `false` or `null` rather than `=`
- Remember to use the `is distinct from` and `is not distinct from` operators when you need them.
- Booleans can be aggregated thanks to `bool_and` and `bool_or`.
- Booleans have three possible values `true`, `false` and `null`.
- Behavior with `null` is entirely ad-hoc.

2. Character and Text
- `text` and `varchar` are the same thing as far as Postgresql is concerned.
- `character varying` is an alias for `varchar`.
- `varchar(15)` means telling Postgresql to manage a `text` column with a `check` constraint of `15 characters`.
- In addition to `like`, and `ilike` patterns and to the SQL standard `similar to` operators, Postgresql embeds support for a full-blown regular expression matching engine.
- Main operator implementing regexp is `~`, and then you find the derivatives for `not matching` and `match either case`. In total, we have four operators `~`, `!~`, `~*` and `!~*`.

#### Server Encoding and Client Encoding

- The SQL_ASCII encoding is a trap you need to avoid falling into. To know which encoding your database is using, run the psql command \l:
- Use UTF-8 encoding and you’ll have a much simpler life. It must be noted that Unicode encoding makes comparing and sorting text a rather costly operation. 
#### Bytea and Bitstring
- Postgresql can store and process raw binary values, which is sometimes useful.
- Binary columns are limited to about `1GB` in size.
- While it's possible to store large binary data that way, Postgresql doesn't implement chunk API and will systematically fetch the whole content when the column is included in your queries output. That means loading the content from disk to memory, pushing it through the network and handling it as a while in-memory on the client-side, so it's not always the best solution around.
- When storing binary content in Postgresql it is then automatically part of your online backups and recovery solution, and the online backups are `transactional`.
- So if you need to have binary content with transactional properties, `bytea` might be a good option.

