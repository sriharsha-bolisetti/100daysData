#### How @Transactional(readOnly=true) Really Works

- Via `@Transactional`, you explicitly demarcate the database transaction boundaries and ensure that one database connection will be used for the whole database transaction duration.
- All SQL statements will use this single isolation connection and all of them will run in the scope of the same Persistence Context.
- Generally speaking, JPA doesn't impose transactions for read operations, it does it only for write operations by throwing a meaningful exception, but this means that:-
1. You allow the auto-commit mode to control the behavior of data access - this behavior may differ depending on the JDBC driver, the database, and the conneciton pool implementations and settings.
2. Usually if `auto-commit` is set to `true`, then each SQL statement will have to executed in a seperate physical database transaction, which may imply a different connection per statement, e.g. in an environment that doesn't support `connection-per-thread`, a method with two SELECT statements requires two physical database transactions and two seperate database connections. Each SQL statement is automatically committed right after it is executed.
3. Explicitly setting the transaction isolation level may lead to unexpected behaviors.
4. Setting auto-commit to true makes sense only if you execute a single read-only SQL statement, but it doesn't lead to any significant benefit. Therefore, even in such cases, it's better to rely on explicit transactions.
- As a rule of thumb, use explicit (declarative) transactions even for read-only statements (e.g., select) to define the proper transactional-contexts. 
A non-transactional-context refers to a context with no explicit transaction boundaries, not to a context with no physical database transaction. All database statements are executed in the context of a physical database transaction. By omitting the explicit transaction boundaries, (transactional-context, begin/commit/rollback), you expose the application to at least the following drawbacks with implications in performance
5. By default, Hibernate will turn off `autocommit` mode anyway and a JDBC transaction is opened. The SQL statement runs inside this JDBC transaction and, afterwards, Hibernate closes the connection. But it doesn't close the transaction, which remains uncommitted (remains in the pending state), and this alows the database vendor implementation or the connection pool to take action. The JDBC specification doesn't impose a certain behavior for pending transactions, for example MYSQL rolls back the transaction while Oracle commits it. `You should not take this risk because as a rule of thumb, you always must ensure that the transaction ends by committing it or rolling it back`.
6. In the case of many small transactions, very common in applications with many concurrent requests, starting and ending a physical database transaction for every SQL statement implies a performance overhead.
7. A method running a non-transactional-context is prone to be altered by developers in order to write data. Having a transactional-context via @Transactional(readOnly=true) at the class/method-level acts as a flag for the team memebers signaling that no writes should be added to this method and prevent writes if this flag is ignored.
8. You cannot benefit from Spring optimizations for the underlying data access layer, e.g. flush mode is set to manual, so dirty checking is skipped.
9. You cannot benefit from database-specific optimizations for read-only transactions.
10. You don't follow the read-only Spring built-in query-methods that rea by default annotated with @Transactiona(readOnly=true).
11. No ACID support for a group of read-only SQL statements.

Being aware of these drawbacks (this list is not exhaustive) should help you to decide wisely between non-transactional contexts and classical database ACID transactions for read-only statements.


- Loading an entity in the Persistence Context is accomplished by Hibernate via what is called the `hydrated state or loaded state`. 
The hydration is the process of materializing the fetched database result set into an Object[]. The entity is materialized in the Persistence Context.
What's happening next depends on the read mode
###### Read-write mode: 
- In this mode, both the entity and its hydrated state are available in the Persistence Context.
- They are available during the Persistence Context lifespan until the Persistence Context is closed or until the entity is detached.
- The `hydrated state` is needed by the Dirty checking mechanism, the Versionless Optimistic Locking mechanism, and the Second Level Cache.
- The Dirty Checking mechanism takes advantage of the hydrated state at flush time. It simply compares the `current entity's sate with the corresponding hydrated state` and if they are not the same, then Hibernate triggers the proper UPDATE statements.
- The Versionless Optimistic Locking mechanism takes advantage of the hydrated state to build the WHERE clause filtering predicates.
- The Second level cache represents the cache entries via the disassembled hydrated state.
- In read-write mode, the entities have the MANAGED status.

###### Read-Only mode
- In this mode, the hydrated state is discarded from memory and only the entities are kept in the Persistence Context, these are read-only entities.
- Obviously, this means that the `automatic Dirty Checking and Versionless Optimistic Locking mechanisms are disabled`. 
- In read-only mode, `the entities have the READ_ONLY status`.
- There is no automatic flush because Spring Boot sets flush mode to MANUAL.
