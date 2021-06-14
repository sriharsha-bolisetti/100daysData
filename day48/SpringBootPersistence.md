#### How to Fetch DTO via Spring Projections
- Fetching data from the database results in a copy of that data in memory, usually referred to as the `result set` or JDBC result set.
- This zone of memory that holds the fetched result set is known and referred to as the `Persistence Context` or the `First Level Cache` or simply the Cache. 
- By default, Hibernate operates in read-write mode. This means that the fetched result set is stored in the Persistence Context as `Object[]` or more precisely Hibernate-specific `EntityEntry` instances and is known in Hibernate terminology as the `hydrated state`, and as entities built from this hydrated state.
- The hydrated state serves the Dirty Checking mechanism, at flush time, Hibernate compares the entitites against the Hydrated state to discover the potential changes/modifications and triggers UPDATE statements on your behalf, the Versionless Optimistic Locking mechanism, for building the WHERE clause, and the second level cache, the cached entries are built from the diassemmbled hydrated state
- In other words, after the fetching operation, the fetched result set lives outside the database, in memeory. The application accesses/managers this data via entities (so, via Java objects), and to facilitate this context, Hibernate applies several specific techniques that transform the fetched `raw` data (JDBC result set) into the hydrated state (this process is known as `hydration`) and further the mangeable representation (`entities`).
- This is a good reason for NOT fetching data as entities in read-wirte mode if there is no plan to modify them.
- In such a scenario, the read-write data will consume memory and CPU resources for nothing. This adds serious performance penalties to the application. 
- Alternatively, if you need read-only entities then switch to read-only mode. This will instruct Hibernate to discard the hydrated state from the memory. Moreover, there will be no automatic flush time and no Dirty Checking. Only entities remain in the Persistence Context. As a consequence, this will save memory and CPU resources.
- Read-only entities still mean that you plan to modify them at some point in the near future as well (e.g., you don't plan to modify them in the current Persistence Context, but `they will be modified in the detached state and merged later in another Persistence Context`). This is a good reason for `NOT fetching data as entities in read-only mode if you never plan to modify them`. However, as an exception, you can consider read-only entities as an alternative to DTOs that mirror the entity.
- As a rule of thumb, `if all you need is read-only data that it will not be modified then use Data Transfer Object (DTO) to represent read-only data as Java objects.` 
- Most of the time, DTOs contain only a subset of entity attributes and this way you avoid fetching more data than needed. Consider limiting the number of fetched rows via LIMIT or its counterparts.
- A spring projection may debut with a Java interface that contains getters only for the columns that should be fetched from the database.
- This type of spring projection is known as an `inteface-based closed projection`
- Behind the scenes, Spring generates a proxy instance of the projection interface for each entity object. Further, the calls to the proxy are automatically forwarded to that object.

#### Enrich Spriong Projections with Virtual Properties that are/aren't part of entities

- Spring projections can be enriched with `virtual` properties that are or are not part of the Domain model.
- Commonly, when they are not part of the Domain model, they are computed at runtime via SpEL expressions.
- An interface-based Spring projection that contains methods with unmatched names in the Domain model and with returns computed at run time is referenced as an `interface-based open projection`