#### SQL Phenomena
- SQL Phenomena represent a set of data integrity anomalies that may occur when developer tries to squeeze peformance from transaction concurrency by relaxing the `SERIALIZABLE` isolation level in favor of another transaction isolation level.
```
There is always a trade-off between choosing the transaction isolation level and the performance of transaction concurrency.
```
##### Dirty Writes
- A dirty write is a `lost update`.
- In a `dirty write` case, a transaction overwrites another concurrent transaction which means that both transactions are allowed to affect the same row at the same moment.
- `Making business decisions in such a context is very risky`. The good news is that, by default, all database systems prevent `dirty writes`.(Even at the Read Uncommitted isolated level).

##### Dirty Reads
- A `dirty read` is commonly associated with the Read Uncommitted isolation level.
- In a dirty read case, a transaction reads the uncommitted modifications of another concurrent transaction that rolls back in the end.
- `Making business decisions based on uncommitted values can be very frustrating and can affect data integrity`. As a quick solution, you can simply use a higher isolation level. As a rule of thumb, always check the default isolation level of your database system. Most probably, the default will not be Read Uncommitted, but check it anyways since you must be aware of it.

##### Non-Repeatable Reads
- A non-repeatable read is commonly associated with Read Committed isolation level.
- A transaction reads some record while a concurrent transaction writes to the same record and commits.
- Later, the first transaction reads the same record again and gets a different value (the value that reflects the second transaction's change).
- Non-repeatable reads become problematic when the current transaction makes a business decision based on the first read value.
- One solution is to set isolation level as Repeatable Read or Serializable, both of the prevent this anomaly by default.
- Or, you can keep Read Committed, but acquire shared locks via `select for share` in an explicit way.
- Databases that use MVCC prevent `non-repeatable reads` by checking the row version to see if it was modified by a transaction that is concurrent to the current one. If it has been modified, the current transaction can be aborted.
- Hibernate guarantees session-level repeatable reads. This means that fetched entities via direct fetching or entity queries are cached in the Persistence Context. Subsequent fetches of the same entities are done from the Persistence Context. Nevertheless, this will not work for conversations that span over several HTTP requests. In such cases, a solution will rely on the Extended Persistence Context or, the recommended way, on detached entities, in web application, the detached entities can be stored in an HTTP session. You can also need an application level concurrency control strategy such as `Optimistic Locking` to prevent `lost updates`.

##### Phantom Reads
- A phantom read is commonly associated with the Repeatable Read isolation level. A transaction reads a range of records, eg. based on a condition.
- Meanwhile, a concurrent transaction inserts a new record in the same range of records and commits. eg. inserts a new record that passes the condition.
- Later, the first transaction reads the range again and it sees the new record.
- This anomaly can be prevented via the SERIALIZABLE isolation level or via `MVCC consistent snapshots`.

##### Read Skews
- A read skew is an anamaly that involves at least two tables eg. car and engine.
- A transaction reads from the first table (eg. reads from a record from the car table). 
- Further, a concurrent transaction updates the two tables in sync (eg. updates the car fetched by the first transaction and its corresponding engine).
- After both tables are updated, the first transaction reads from the second table (eg. reads the engine corresponding to the car fetched earlier).
- The first transaction sees an older version of the car record, without being aware of the update, and the latest version of the associated engine.
- You can prevent a `read skew` by acquiring shared locks on every read or by MVCC implementation of Repeatable Read isolation level.

##### Write Skews
- A write skews is an anomaly that involves at least two tables. 
- Both tables should be updated in sync, but a write skew allows two concurrent transactions to break this constraint.
- You can prevent write skews by acquiring shared locks on every read or by MVCC implementation of the Repeatable Read isloation level.

##### Lost Updates
- A lost update is a popular anomaly that can seriously affect data integrity. 
- A transaction reads a record and uses this information to make a business decisions. eg. decisions that may lead to modification of that record. 
- Without being aware that, in the meantime, a concurrent transaction has modified that record and committed.
- When the first transaction commits, it is totally unaware of the lost update.
- This causes data integrity issues.
- This anomaly affects Read Committed isolation level and can be avoided by setting the Repeatable Read or Serializable isolation level.
- For the Repeatable Read isolation level without MVCC, the database uses shared locks to reject other transactions attempts to modify an already fetched record.
- In the presence of MVCC, a concurrent transaction (Transaction B) can perform changes to a record already fetched by a previous transaction (Transaction A). 
- When the previous transaction (A) attempts to commit its change, the database compares the current value of the record version (which was modified by the concurrent transaction commit (Transaction B)) to the version that it is pushed via Transaction A.
- If there is a mismatch, which means the Transaction A has stale data, then transaction A is rolled back by the application.
- When a record is modified, Hibernate can automatically attach the record version to the corresponding SQL via the application-level Optimistic Locking mechanism.
- With long conversations that span over several HTTP requests, besides the application-level Optmistic Locking mechanism, you have to keep the old entity snapshots via Extended Persistence Context or, the recommended way, via detached entities (in web applications, they can be stored in the HTTP session).