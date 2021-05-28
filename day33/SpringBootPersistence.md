#### Entity State Transitions

##### Transient 
- A new entity that is totally unknown to the database - at flush time, Hibernate will issue an `Insert` statement for it.

##### Managed (or Persistent)
- The entity has a corresponding row in the database and is currently loaded in the Persistence Context.
- In read-write mode, at flush time, Hibername will run the Dirty Checking mechanism, and if it detects modifications, it will issue the proper `Update` statements on your behalf.

##### Detached
- The entity was in the Persistence Context but the Persistence Context was closed or the entity was cleared/evicted. 
- Any modifications of a detached entity are not propogated automatically to the database.

##### Removed
- The entity was in the Persistence Context and it is marked for deletion.
- At flush time, Hibernate will issue the proper `Delete` statement to delete the corresponding row from the database.

#### Understanding the Flushing Mechanism
- `Flushing` is the mechanism of synchronizing the in-memory persistent state with the database.
- During a transaction's lifespan, flushing can occur multiple times and can be done manually or automatically.
- If Persistence Context is closed (or cleared) and you forget to flush the pending modifications, these modifications are lost. They will not be available in the database.
- Persistence Context acts as a `transactional write-behind cache`.
- Since the entities state modifications are buffered, Hibernate can postpone the Persistence Context flush until the last moment.
- At flush time, Hibernate translates the buffered entities state transitions into DML statements, which are meant to synchronize the in-memory persistent state with the database.
- The operation of copying the entity state into an `Insert` or `Update` statement is called `Dehyration` in Hibernate.
###### Strict Flush order of actions
- Until the flush time, Hibernate buffers the entities modifications in the Persistence Context - more precisely in `ActionQueue`. 
- At flush time, Hibernate empowers a very strict order of processing these actions, as follows:
1. OrphanRemovalAction.
2. EntityInsertAction and EntityIdentityInsertAction
3. EntityUpdateAction
4. CollectionRemoveAction
5. CollectionUpdateAction
6. CollectionRecreateAction
7. EntityDeleteAction

- The DML corresponding to the entities actions start with `Insert` statements, continues with `Update` statements, and finishes with `Delete` statements.
- Since this strict order governs the data access layer actions and it was chosen to minimize the chances of `constraint violations`, you have to pay attention to how you write and interpret your data access layer code.
- Coding without repect to this strict order may lead to unexpected behaviors and exceptions, e.g. ConstraintViolationException.
- Whenever you are in such situations, take your time and try to find the correct fix.
- Almost always, the wrong fix is to disturb this order by forcing flushes by explicitly calling the `flush` methods.
###### Flush Before Executing A Data Query Language(DQL) - Select Query
- JPA empowers a `flush-before-query` strategy in order to maintain the `read-your-own-writes` consistency.
- Unless Second Level Cache is employed, DQL Select queries are executed against the database, and a query must see the in-memory changes as well. Therefore a prior flush is required.
- If this flush is not triggered before the query execution, the query is prone to not read your own writes, which may lead to data inconsistencies.
- But, once the flush is triggered, `the flushed changes become visible to the query and to the current database transaction`.
- They will become `visible to other transactions and Persistence Contexts only after this transaction commits`. 
###### Flush Before Transaction Commits
- After the current transaction commits, the Persistence Context is cleared and closed.
- Once the Persistence Context is cleared and closed, (except when using the Extended Persistence Context) and the entities in-memory changes are lost. To prevent this behavior, JPA `triggers the flush right before the transaction commits`. This way entities in-memory changes are propogated to the databae and these changes become `durable`.

###### Automatic FlushModes
- Flushing before a query execution and before transaction commits is accomplished by JPA (and Hibernate ORM) via automatic flush modes.
- The automatic flush mode of the Persistence Context should work as follows:
- Before running a DQL Select, JPA JPQL, or Hibernate-specific HQL query.
- Before running a native DQL select query.
- Before the transaction commits.

JPA defines two flush modes.
1. Auto
- This is the default flush mode that triggers a flush before every query execution (including native queries) and before the transaction commits.
- This flush mode follows the JPA specification, which says that `AUTO should guarantee that all pending changes are visible by any executing query`.
- This is the `default flush mode` if we bootstrap Hibernate as the JPA provider in a Spring Boot application.

2. Commit
- Triggers a flush only before the transaction commits.

Hibernate ORM defines four flush modes
1. Auto
- The main target behind the Hibernate-specific optimized flushing mechanism is to perform a flush operation only if the current executed query needs a pending DML - Insert, Update, or Delete statement.
- So, while a flush is triggered before the transaction commits, it may skip the overhead of premature flushes before the query execution.
- This may premature flushes are delayed as much as possible.
- Ideally, it will remain a single flush, the one triggered before the transaction commits, and the flush will propagate all the needed DML statements.
- This mode also has a major shortcoming - It doesn't work for native queries. Even if necessary, it will not trigger a Persistence Context flush when a native query is executed.
- Hibernate cannot parse native queries because it understands limited SQL syntax. Therefore, it cannot determine the referenced tables that may require a flush.
- This behavior may lead to `read-your-writes` inconsistencies. 
- This Auto flush mode is used automatically `only if you are bootstrapping Hibernate natively`. 
- Using Spring Boot andspring-boot-starter-data-jpaas the starter for using Spring Data JPA with Hibernate will rely on JPA’sAUTOflush mode, not on the Hibernate-specificAUTOflush mode.

2. Always
-  This is like the JPAAUTO mode. It triggers a flush before every query execution (including native queries).

- As a rule of thumb, do not trade data consistency for application efficiency. In other words, don’t permit data inconsistencies just to avoid several premature flushes.

3. COMMIT : This is like the JPA COMMIT mode.
4. MANUAL : This flush mode disables automatic flushes and requires explicit calls of dedicatedflush methods .

- Setting `@Transactional (readOnly=true)` sets the flush mode to `Manual`.
- Even if you set the `Commit` flush mode epxlicitly in application.properties, the presence of `readOnly=true` (which is a signal that the annotated method doesn't contain database write operations) has instructed Spring Boot to switch to the Hibernate-specific, `Manual` flush mode. This means that setting has no effect in thsi context. There will be no automatic flush before the transaction commit.
- Even if you set theMANUAL flush mode explicitly inapplication.properties , the presence of@Transactional (which is a signal that the annotated method contains database write operations) has instructed Spring Boot to switch to the JPA,AUTO flush mode. This means that the setting has no effect in this context. Pay attention to this aspect, since you may think that there will be no automatic flushing since you disabled it viaMANUAL .
