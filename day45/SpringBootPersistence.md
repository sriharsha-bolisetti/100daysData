#### How to Increment the Version of the Locked Entity Even if this Entity Was Not Modified
- Consider several editors that prepare a book for printing. They load each chapter and apply specific modifications. 
- Each of them should be allowed to save their modifications only if, in the meanwhile, the other one didn't save any modifications.
- In such cases, the chapter should be reloaded before considering modifications.
- In other words, the modifications should be applied sequentially.
- The chapter is mapped by the root entity `Chapter`, and the modification by the `Modification` entity.
- Between Modification (child-side) and Chapter (parent-side), there is a unidirectional lazy @ManyToOne association
##### Optimistic Force Increment
- For this requirement, `@Version` and the `OPTIMISTIC_FORCE_INCREMENT` strategies can be relied upon.
- Their powers combined can help increment the version of the locked entity (Chapter) even if this entity was not modified.
- In other words, each modification is forcibly propagated to the Parent entity (Chapter) Optimistic Locking version.
- So, the Optimistic Locking version should be added to the root entity, Chapter:
```
@Entity
pubic class Chapter implements Serializable {
   private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String content;
    @Version
    private short version;   
}

@Entity
public class Modification implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String description;
    private String modification;

    @ManyToOne(fetch = FetchType.LAZY)
    private Chapter chapter;
}
```
- The editor loads the chapter by ID, using the `LockModeType.OPTIMISTIC_FORCE_INCREMENT` lock strategy.
- For this, we have to override the `ChapterRepository.findById()` method to add the locking mode, as shown
```
@Repository
public interface ChapterRepository extends JpaRepository<Chapter, Long>{
    @Override
    @Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
    public Optional<Chapter> findById(Long id);
}
```

```
The OPTIMISTIC_FORCE_INCREMENT lock strategy is useful for cooridnating child-side state changes in a sequential manner by propagating these changes to the parent-side Optimistic Locking verison. You can orchestrate the sequence of state changes of a single child or of more children.
```
##### PESSIMISTIC_FORCE_INCREMENT
- While OPTIMISTIC_FORCE_INCREMENT increments the version at the end of the current transaction, PESSIMISTIC_FORCE_INCREMENT increments the version immediately.
- The entity version update is guaranteed to succeed immediately after acquiring the row-level lock.
- The increments take place beffore the entity is returned to the data access layer.
- If the entity was previously loaded without being locked and the PESSIMISTIC_FORCE_INCREMENT version update fails, the currently running transction can be rolled back right away.
- This time, we use a `@Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT)`

- To acquire an exclusive lock, Hibernate will rely on the underlying Dialect lock clause. 
- In Postgresql, the PostgreSQL95Dialect dialect acquires row-level locks via `FOR UPDATE NOWAIT`
- A transaction that increments the entity version will block other transactions to acquire a `PESSIMISTIC_FORCE_INCREMENT` lock until it releases the row-level physical lock (by commit or rollback). 
- In this context, always rely on `NOWAIT` or on explicit short timeouts to avoid deadlocks
- The database can detect and fix deadlocks (by killing one of the transactions), but it can do so only after timeout.
- A long timeout means a busy connection for a long time, and so a performance penalty. Moreover, locking too much data may affect scalability.

#### How PESSIMISTIC_READ/WRITE Works
- When we talk about PESSIMISTIC_READ and PESSIMISTIC_WRITE, we talk about shared and exclusive locks.
- Shared locks and read locks allow `multiple processes to read at the same time and disallow writes`.
- Exclusive or write locks `disallow reads and writes as long as write operation is in progress`.
- The purpose of shared/read lock is to prevent other processes from acquiring an exclusive/write lock.
- You can acquire a shared lock in Spring Boot at the query-level via PESSIMISTIC_READ while you can acquire an exclusive lock via PESSSIMISTIC_WRITE 
