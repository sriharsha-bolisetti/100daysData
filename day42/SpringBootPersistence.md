#### Concurrency Control in Postgresql
- Data consistency is maintained by using a multiversion model - Multiversion Concurrency Control, MVCC.
- This means that each SQL statement sees a snapshot of data (a database version) as it was some time ago, regardless of the underlying data.
- This prevents statements from viewing inconsistent data produced by concurrent transactions performing update on the same data rows, providing `transaction isolation` for each database session.
- Main advantage of using MVCC model of concurrency control rather than locking is that in MVCC locks acquired for querying data do not conflict with locks acquired for writing data, so `reading never blocks writing and writing never blocks reading`
- Postgresql maintains this guarantee even when providing the strictest level of transaction isolation through the use of an innovative `Serializable Snapshot Isolation` level.

##### Read Committed Isolation level

- `Read Committed` is the default isolation level in Postgresql.
- When a transaction uses this isolation level, a SELECt query (without a FOR UPDATE/SHARE clause) sees only data committed before the query began.
- It never sees either uncommitted data or changes committed during query execution by concurrent transactions.
- In effect, a SELECT query sees a snapshot of the database as of the instant the query begins to run. However, SELECT does see the effects of previous updates executed within its own transcation, even though they are not yet committed.
- Also note that `two successive SELECT commands can see different data, even though they are within a single transaction, if other transcation commit changes after the first SELECT starts and before the second SELECT starts`.
- UPDATE, DELETE, SELECT FOR UPDATE and SELECT FOR SHARE commands behave the same as SELECT in terms of searching for target rows: they will only find target rows that were committed as of the command start time.
- However such a target row might have already been updated or deleted or locked by another concurrent transaction by the time it is found. 
- In this case, `the would-be updater will wait for the first updating transaction to commit or roll back (if it is still in progress).`
- If the first updater rolls back, then its effects are negated and the second updater can proceed with updating the originally found row.
- If the first updater commits, the second updater will ignore the row if the first updater deleted it, otherwise it will attempt to apply its operation to the updated version of the row.
- The `search condition of the command (the WHERE clause) is re-evaluated` to see if the updated version of the row still matches the search condition. If so, the second updater proceeds with its operation using the updated version of the row.
- `INSERT` with an `ON CONFLICT DO UPDATE` clause behaves similarly. In Read Committed mode, each row proposed for insertion will either insert or update. Unless there are unrelated errors, one of those two outcomes is guaranteed. If a conflict originates in another transaction whose effects are not yet visible to the INSERT, the `Update clause will affect that row`, even though possibly no version of that row is conventionally visible to the command.
- `INSERT` with an `ON CONFLICT DO NOTHING` clause may have insertion not proceed for a row due to the outcome of another transaction whose effects are not visible to the INSERT snapshot. 
- Because of above rules, it is possible for an updating command to see an inconsistent snapshot: it can see the effects of concurrent updating commands on the same rows it is trying to update, but it does not see effects of those commands on other rows in the database. This behavior makes Read Committed mode unsuitable for commands that involve complex search conditions, however it is just right for simpler cases.

- Complex usage can produce undesirable results in Read Committed mode. For example, consider a DELETE command operating on data that is being both added and removed from its restriction criteria by another command, e.g., assume `website` is a two-row table with `website.hits` equaling 9 and 10:
```
BEGIN;
UPDATE website SET hits = hits + 1;
-- run from another session: DELETE FROM website WHERE hits = 10;
COMMIT;
```
- The DELETE will have no effect even though there is a `website.hits = 10` row before and after the udpate. 
- This occurs because the pre-update row value is skipped, and when the UPDATE completes DELETE obtains a lock, the new row value is no longer 10 but 11, which no longer matches the criteria.
- Because the Read Committed mode starts each command with new snapshot that includes all transactions committed up to the instant, subsequent commands in the same transaction will see the effects of the committed concurrent transaction in any case. The point `at issue above is whether or not a single command sees an absolutely consistent view of the database`.
