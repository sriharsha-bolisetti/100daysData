#### How to Use Direct Fetching
- Direct Fetching or fetching by ID is the preferable way to fetch an entity `when its identifier is known and its lazy associations will not be navigated` in the current Persistence Context.
- By default, direct fetching will load the entity according to the default or specified FetchType. 
- JPA @OneToMany and @ManyToMany associations are considered `LAZY`
- @OneToOne and @ManyToOne associations are considered `EAGER`
- Fetching an entity by ID that has an EAGER association will load that association in the Persistence Context even if is not needed, and this causes performance penalities.
- On the other hand, fetching an entity that has a LAZY association and accessing this association in the current persistence context will cause extra queries for loading it as well - also leading to performance penalities.
- Best approach is to keep all the associations LAZY and rely on manual fetching strategy to load these associations. Rely on direct fetching only if you don't plan to access the LAZY associations in the current persistence context.
- JPA persistence provider fetches the entity with the given ID via findById(), find() and get() by searching it in this order:
1. Current Persistence Context
2. Second Level Cache.
3. Database.

##### Direct Fetching and Session Level Repeatable Reads
- Why does Hibernate check the Persistence Context first to find the entity with the given ID?
- Answer is that Hibernate guarantees `session-level repeatable reads`
- This means that the entity fetched the first time is cached in the Persistence Context. Subsequent fetches of the same entity (via direct fetching or explicit entity query) are done from the Persistence Context.
- Session-level repeatable reads prevent `lost updates` in concurrent writes cases.
- From a performance perspective, it is advisable to use findById(), find(), or get() instead of an explicit JPQL/SQL to fetch an entity by ID. That way, if the entity is present in the current Persistence Context, there is no SELECT triggered against the database and no data snapshot to be ignored.

#### Why Use a Read-Only Entities whenever you plan to propogate changes to the databae in a future persistence context
- Consider the Author entity that shapes an author profile via several properties as id, name, age and genre.
- The scenario requires you to load an Author profile, edit the profile and save it back to the database.
- You don't do this in a single transaction (Persistence Context). You do it in two different transactions as below.

- Do not confuse read-only entities with DTO (projections). A read-only entity is meant to be modified only so the modifications will be propogated to the database in a future Persistence Context.
- A DTO (projection) is never loaded in the Persistence Context and is sutable for data that `will never be modified`.
- `After Fetching and returning the entity it becomes detached, further we can modify it and merge it`.
- The Persistence Context is bound to the lifecycle of a single physical database transaction and a single logical @Transactional. If you choose to go with the Extended Persistence Context, then the implementation is governed by other rules. Nevertheless, using Extended Persistence Context in Spring is quite challenging. If you aren't completely sure you understand it, it's better to avoid it.