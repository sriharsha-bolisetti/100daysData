#### Spring Saving Entities

- Saving an entity can be performed with the `crudRepository.save()`, It persists or merges the given entity by using the underlying JPA EntityManager.
- If the entity has not yet been persisted, Spring Data JPA saves the entity with a call to the `entityManager.persist()` method, else it calls `entityManager.merge()` method.

##### Entity State-Detection Strategies

Spring Data JPA offers the following strategies to detect whether an entity is new or not

1. Version-Property and Id-Property inspection.

- By default Spring Data JPA inspects first if there is a Verion-Property of non-primitive type. If there is, the entity is considered new if the value of that property is `null`. Without such a Version property Spring Data JPA inspects the identifier property of the given entity, if the identifier property is null, then the entity is assumed to be new. Otherwise, it is assumed to be not new.

2. Implementing Persistable: If an entity implements Persistable, Spring Data JPA delegates the new detection to the isNew(…) method of the entity
3. Implementing EntityInformation

#### JPQL

- Object Oriented query language which is used to perform database operations on persistent entities.
- Instead of database table, JPQL uses entity object model to operate the SQL queries.

##### Creating Queries in JPQL

1. Query createQuery(String name)
2. Query createNamedQuery(String name)

### Spring/Hibernate Optimistic Locking

#### How to Retry The Transaction After a Versioned (@Version) OptmisticLockException

- Optimistic Locking is a concurrency control technique that doesn't use locks. It's extremely useful for preventing `lost updates`

##### Versioned Optimistic Locking Exception

- Most commonly, Optimistic Locking is implemented by adding a field annotated with `@Version` to the entity, this is known as `Versioned Optimistic Locking` and it relies on a numerical value that is automatically managed (incremented by 1 when data is modified) by the JPA Persistence Provider.
- In a rude experimentation, based on this value, the JPA persistence provider can check if the data manipulated by the current transaction has been changed by a concurrent transaction. Therefore, it would be prone to a `lost update`.

##### Versionless Optimistic Locking Exception

- Basically, Versionless Optimistic Locking relies on the WHERE clause added to the UPDATE statement.
- This clause checks `if the data that should be updated has been changed` since it was fetched in the current persistence context.
- Versionless Optimistic Locking works as long as the current Persistence Context is open, which avoids detaching entities.
  The preferable way to go with Versionless Optimistic Locking is as follows:

```
@Entity
@DynamicUpdate
@OptimisticLocking(type = OptimisticLockType.DIRTY)
public class Inventory implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    private Long id;
    private String title;
    private int quantity;
    // getters and setters omitted for brevity
}
```

- Setting `OptimisticLockType.DIRTY` instructs Hibernate to automatically add the modified columns, eg., the column corresponding to the `quantity` property to the `UPDATE WHERE` clause.
- The `@DynamicUpdate` annotation is required in this case and in the case of `OptimisticLockType.ALL` - all properties of the entity will be used to verify the entity version.
- You can exclude a certain field from versioning, e.g., children collection changes should not trigger a parent version update, at the field level via the `@OptimisticLock(excluded = true)` annotation.

#### How to Handle Versioned Optimistic Locking and Detached Entities

- Versioned Optimistic Locking works with detached entities, while Hibernate ORM Versionless Optimistic Locking doesn't work.
- Illustration by example
- Transaction A fetches an inventory entity with version 0.
- Transaction B fetches and updates same version 1 inventory entity, with the update this is version 1.
- Meanwhile, Transaction C tries to update inventory entity fetched in Tranaction A.

After first transaction A is done, the transaction commits and the Persistence Context is closed.
The returned Inventory becomes a `detached entity`.

After second transaction B modified the quantity of the inventory, the version is updated to 2 and Persistence Context is closed.

The third Transaction C, say with method thirdTransactionMergesAndUpdates() and pass as an argument the `detached entity` fetched in Transaction A. `Spring inspects the entity and concludes that this should be merged. Therefore, behind the scenes (behind the save() call), it calls the EntityManager#merge().`

- Further, the JPA provider fetches from the database (via SELECT) a persistent object `equivalent to the detached entity` (since there is no such object) and `copies the detached entity to the persisted one`

```
SELECT
  inventory0_.id AS id1_0_0_,
  inventory0_.quantity AS quantity2_0_0_,
  inventory0_.title AS title3_0_0_,
  inventory0_.version AS version4_0_0_
FROM inventory inventory0_
WHERE inventory0_.id = ?
Binding:[1] Extracted:[9, A People's History, 1]
```

`At the merge operation, the detached entity doesn't become managed. The detached entity is copied into a managed entity (available in the Persistence Context)`

- At this point, Hibernate concludes that the `version` of the fetched entity and the `version` of the detached entity `doesn't match`.
- This will lead to an Optimistic Locking exception reported by Spring Boot as `ObjectOptimisticLockingFailureExceptin`

```
Do not attempt to retry the transaction that uses merge(). Each retry will just fetch from the database the entity whose version doesn't match the version of the detached entity, resulting in an Optimistic Locking exception.
```

#### How to Use the Optimistic Locking Mechanism and Detached Entities in long HTTP Conversations

- The following scenario is a common case in web applications and is known as a `long conversation`.
- In other words, a bunch of requests that are logically related shape a stateful long conversation that contains the client thinking periods as well.
- Mainly, this read -> modify -> write flow is perceived as a logical or application-level transaction that may span over multiple physical transactions.
- Application-level transactions should be suitable for ACID properties as well. In other words, you must control concurrency (e.g., via Optimistic Locking mechanism, which is suitable for both application-level and physical transactions) and have application-level repeatable reads. This way, `lost updates` are prevented.

```
In long conversations, only the last physical transaction is writeable to propogate to the database (flush and commit).
If an application-level transaction has intermediate physical writable transactions, then it cannot sustain atomicity of the application-level transaction.
In other words, in the context of the application-level transaction, while a physical transaction may commit, a subsequent one may roll back.
```
- Detached entities are commonly used in long conversations that span across several requests over the stateless HTTP protocol.
