#### Application Transactions
- A logical transaction is an application-level unit of work that may span over multiple physical transactions. Holding the database connection open throughout several user requests, including user think time, is definitely an anti-pattern.
- A database server can accommodate a limited number of physical connections, and often those are reused by using connection pooling. Holding limited resources for long periods of time hinders scalability. So database transactions must be short to ensure that both database locks and the pooled connections are released as soon as possible.
- Web applications entail a read-modify-write conversational pattern. A web-conversation consists of multiple user requests all operations being logically connected to the same application-level transaction. 
- All these operations should be encapsulated in a single `unit-of-work`. We, therefore, need an `application-level transaction that's also ACID compliant`, because other concurrent users might modify the same entities, long after shared locks had been released.
- The database transaction ACID properties can only prevent this phenomenon within the boundaries of a single physical transaction. `Pushing transaction boundaries into the application layer requires application-level ACID guarantees`.
- To prevent lost updates, we must have application-level repeatable reads along with a concurrency control mechanisms.
##### Long Conversations
###### Using Detached Objects
- One option to achieving Long Conversation is to `bind the persistence context to the life-cycle of the intermediate physical transaction`.
- Upon persistence context closing all entities become `detachecd`.
- For a detached entity to become managed, can be merged with their persistent object equivalent. If there's no currently loaded persistence object, Hibernate will load one from the database. `The detached entity will not become managed`

trouble:
```
What if the loaded data doesn’t match what we have previously loaded?
What if the entity has changed since we first loaded it?
```
- `Overwriting new data with an older snapshot leads to lost updates. So the concurrency control mechanism is not an option when dealing with long conversations.`

- Even if we have application-level repeatable reads other can still modify the same entities. Within the context of single database transaction, `row-level locks can block concurrent modifications but this is not feasible for logical transactions`. The only option is to allow others to modify any rows
##### Optimistic locking to the rescue
- Optimistic locking is a generic-purpose concurrency control technique, and it works for both physical and application-level transactions. Using JPA is only a matter of adding a `@Version` field to our domain models:
- If you Hibernate session has already loaded a given entity then any successive entity query is going to return the same object reference, disregarding the current loaded database snapshot.
- While SQL query projections always load the latest database state, entity query results are managed by the first level cache, ensuring session-level repeatable reads.
- While native SQL remains the de facto relational data reading technique, Hibernate excels `in writing data`.
- Loading entities make sense if you plan on propogating changes back to the databae. you don't need to load entities for displaying read-only views, a SQL projection being a much better alternative in this case.
