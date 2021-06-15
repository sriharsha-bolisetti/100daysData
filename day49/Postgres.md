#### Explicit Locking
- Postgresql provides various lock modes to control concurrent access to data in tables.
- These modes can be used for application-controlled locking in situations where MVCC does not give the desired behavior.
- Also, most postgresql commands automatically acquire locks of appropriate modes to ensure that referenced tables are not dropped or modified in incompatible ways while the command executes. For example, TRUNCATE cannot safely be executed concurrently with other operations on the same table, so it obtains an `ACCESS EXCLUSIVE` lock on the table to enforce that.

#### Table-Level Locks
- `access share` -> select command acquires a lock of this mode on referenced tables. In general, any query that only `reads` a table and does not modify it will acquire this lock mode.
- `row share` -> `select for update` and `select for share` commands acquire a lock of this mode on the target table (in addition to `access share` locks on any other tables that are referenced but not selected `for update/for share`)
- `row exclusive` -> The commands `update`, `delete` and `insert` acquire this lock mode on the target table (in addition to `access share` locks on any other referenced tables). In general, this lock mode will be acquired by any command that `modifies data` in a table.
- `share update exclusive` -> acquired by `vaccum`, `analyze`, `create index concurrently`, `reindex concurrently`, `create statistics` and certain `alter index` and `alter table` variants.
- `share` -> acquired by `create index` (without concurrently)
- `share row exclusive` -> This mode protects a table against concurrent data changes, and is self-exclusive so that only one session can hold it any time. Acquired by `create trigger` and some forms of `alter table`
- `exclusive` -> This mode allows only concurrent `access share` locks, i.e., only reads form the table can proceed in parallel with a transaction holding this lock mode.
- `access exclusive` -> This mode gurantees that the holder is the only transaction accessing the table in any way. Acquired by the DROP TABLE, TRUNCATE, REINDEX, CLUSTER, VACUUM FULL, and REFRESH MATERIALIZED VIEW (without CONCURRENTLY) commands. Many forms of ALTER INDEX and ALTER TABLE also acquire a lock at this level. This is also the default lock mode for LOCK TABLE statements that do not specify a mode explicitly.
- Only an `access exclusive` blocks a select.
- The only real difference between one lock mode and another is the set of lock modes with which each conflicts. Two transactions cannot hold locks of conflicting modes on the same table at the same time. However, a transaction never conflicts with itself. For example, it might acquire `access  exclusive` lock and later acquire `access share` lock on the same table.
- Non-conflicting lock modes can be held concurrently by many transactions.

#### Row-Level Locks
- Row-level locks do not affect data querying, they block only `writers` and `lockers` to the same row.
- Row-level locks are released at transaction end or during savepoint rollback, just like table-level locks.
##### Row-level lock modes
- `for update` -> this causes the rows retrieved by select statement to be locked as though for update. This prevents them from being locked, modified or deleted by other transactions until the current transaction ends. 
- `for no key update`
- `for share` 
- `for key share`