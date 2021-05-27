#### Cascading
- Entity relationships often depend on the existence of another entity. For example, the Person-Address relationship, without the Person, the Address entity doesn't have any meaning of its own. When we delete the Person entity, our Address entity should also get deleted.
- Cascading is the way to achieve this. When we perform some action on the target entity, the same action will be applied to associated entity.

#### Persistence Unit
- Persistence Unit is a box holding all the needed information to create an `EntityManagerFactory` instance.
- Among this information are details about the data source(JDBC url, user, password, sql dialect, etc.), the list of entities that will be managed, and other specific properties.
- Persistence unit transaction type can be `resource-local` (single data source) or `JTA` (multiple data sources). In Java EE, you can specify all these details in an XML file named `persistence.xml`. 
- In Spring Boot, you can do it via `application.properties` and Spring Boot will handle the creation of persistence unit on your behalf.
- In the same application, you can have multiple persistence units and identify them each by name, therefore you can connect to different databases from same application.

#### Entity Manager Factory

- Factor capable of creating on-demand EntityManager instances.
- You provide the needed information via the persistence unit, and this information is used to create an `EntitymanagerFactory` that exposes a method named `createEntityManager()`, which returns a new application-managed `EntityManager` instance at each invocation.
- Programmatically, you can open an `EntityManagerFactory` via injection `@PersistenceUnit` or via `Persistence#createEntityManagerFactory()`
- You can check the status of an `EntityManagerFactory` via the `isOpen()` method, and can close it via the `close()` method.
- If you close an `EntityManagerFactory`, all its entity managers are considered to be in closed state.

#### Entity Manager
- Fetching data from the database results in a `copy of that data in memory` - usually referred to as the `JDBC result set` or simply `result set or data snapshot`.
- This zone of memory that holds the fetched data is known and referred to as `Persistence Context` or the `First level cache` or simple the `Cache`.
- After a fetching operation, the fetched `result set` lives outside the database, `in memory`. In applications, you access/manager this `result set` via `entities` (so, via Java objects), and for facilitating this context, Hibernate JPA applies specific techniques that `transform the fetched result set into an array of raw data (Object[] - hydrated/loaded state)` and to the manageable representation referenced as `managed entities`.
- Planning to modify the fetched entity objects is a way to exploit the fact that, besides being a cache for entities, the Persistence Context acts as an `entity state transitions buffer` and as a transactional `write-behind` cache as well.
- At flush time, Hibernate is responsible for translating the buffered entities state transitions into `DML` statements that are meant to `optimally synchronize the in-memory persistent state with the database`.
- One single active Persistence Context should be allocated to the currently active database transaction. 
- During the current database transaction lifespan, you can manipulate entities via the `EntityManager` methods (verbs/actions) and Hibernate JPA will buffer the entities state transitions.
- `Actions` such as finding objects, persisting or merging them, and removing them from the database and so on are specific to the `EntityManger`. 
- In a more simplistic day-to-day chat, there is nothing wrong with saying that the `EntityManager` instance is the current Persistence Context.
- After you modify the in-memory Persistence Context via entity state management methods like persist(), remove() etc, you expect to see these modifications reflected in the database. This action is known as a `flush` and it can be automatically or manually triggered multiple times during a `transaction lifespan`.
- At the flush time we synchronize the Persistence Context to the underlying database.
- Once the current transaction completes, all objects that were in the Persistence Context are `detached`.
- Detaching all entities takes place when the `EntityManager` is cleared via the `clear()` method or closed via the `close()` method.
- Certain entities can be detached by calling a dedicated method named `detach()`, this means that further modifications to these objects will not be reflected in the database. Propagating subsequent changes is possible only after `merging` or `reattaching` the objects in the context of an active transaction.
- This is the default behavior of a `transactional scoped Entitymanager` (a.k.a., transactional Persistence Context). There is also the `EntityManager` that spans across multiple transactions (extended-scope), known as the `Extended Persistence Context`.
- `JPA merging` is an operation that loads a new entity object from the database and updates it by copying onto it the internal state of detached entity.
- But, before loading a new entity, JPA merging takes into account the current Persistence Context managed entities.
- In other words, if the current Persistence Context already manages the needed entity, there is no need to load a new one from the database. It simply takes advantage of session-level repeatable reads.
- This operation will `overwrite` changes that you have performed on this managed entity during the current session as the attribute values of the detached entity are copied to the managed entity. 
- `merge()` method returns the newly updated and managed instance.
- `Hibernate reattaching` is a Hibernate-specific operation. Mainly, this operation takes place on the detached entity itself.
- It's purpose is to transition the passed entity object from a `detached` to a `managed` state.
- Trying to reattach a `transient` entity or an already loaded entity in the current Persistence Context will throw an exception.
- This operation bypasses the `Diry Checking` mechanism and it is materialized in an UPDATE triggered at Persistence Context flush time.
- Since Hibernate didn't read the latest state of the entity from the underlying database, it cannot perform Dirty Checking.
- In other words, the `UPDATE` is triggered all the time, `even if the entity and the database are in sync`. This can be fixed by annotating the entity with `@SelectBeforeUpdate`, this annotation instructs Hibernate to trigger a `SELECT` to fetch the entity and perform Dirty checking on it before it generates an `UPDATE` statement.
- NOTE: Calling `update(obj)` with `@SelectBeforeUpdate` will select only `obj`, however calling `merge(obj)` will select `obj` and all associations with `CascadeType.Merge`. This means `JPA merging is the proper choice for graphs of entities`.

Rule of thumb is 
- Use JPA merge() if you want to copy the detached entity state.
- Use Hibernate specific update() for batch processing.
- JPA persist() for persisting an entity.
- Upon detaching, Objects leave the Persistence Context and continue to live outside of it. This means that JPA wont manage them anymore. They don't get special treatment and are just usual Java objects. 
- Programmatically, you can open anEntityManager via injection (@PersistenceContext ) or viaEntityManagerFactory#createEntityManager() . You can check the status of anEntityManager via theisOpen() method , you can clear it via theclear() method , and you can close it via theclose() method .