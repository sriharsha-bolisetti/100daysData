#### Why Spring Ignores @Transactional
Two main reasons why @Transactional can be ignored
1. If `@Transactional` was added to a private, protected, or package-protected method.
2. `@Transactional` was added to a method defined in the same class as where it is invoked.
```
@Transactional works only on public methods, and the method should be added in a class different from where it is invoked.
```
#### How to Set and Check that Transaction Timeout and Rollback at Expiration work as expected

- Relying on the timeout element of @Transactional is a very handy way to set a transaction timeout at the method-level or class-level. 
- You can explicitly set a global timeout as well, via the `spring.transaction.default-timeout` property in `application-properties`.
```spring.transaction.default-timeout=10```

#### Why and How to Use @Transactional in a Repository Interface
- Generally speaking, the speed of a database is given by the `transaction throughput`, expressed as number of transactions per second.
- This means that databases were built to accommodate a lot of short transactions rather than long-running transactions.
- First of step of defining the query-methods - you define a domain class specific repository interface. The interface must extend Repository and be typed to the domain class and an ID type.

1. Does Query-methods listed in an interface repository run by default in a transactional context?
- Non-transactional-context refers to context with no explicit transaction boundaries, `not` to context with no physical database transaction.
- All database statements are triggered in the context of a physical database transaction.
- Spring doesn't supply a default transactional-context for user-defined query-methods. On the other hand, the built-in query-methods like save, findById, delete etc don't have this issue. They are inherited from extended built-in repository interfaces and come with default transactional-contexts.

However, lets consider following example

```
public void callFindByIdMethodAndUpdate() {    
    Author author = authorRepository.findById(1L).orElseThrow();
    author.setGenre("History");    
    authorRepository.save(author);
    }
```
- The code requires two separate physical transactions (two database round trips) to accommodate the SELECT triggered via findById(), and the SELECT and UPDATE triggered via save().
- The Persistence Context used by findById() is closed after method execution.
- Therefore, the save() method needs another Persistence Context.
- In order to update the author, Hibernate needs to merge the detached author. Basically, it loads the author in this Persistence Context via a prior SELECT.
- Obviously, these two SELECT statements may return different result sets if a concurrent transaction performs modifications on the concerned data, but this can be eliminated via Versioned Optimistic Locking to prevent lost updates.
- Spring has automatically supplied transactional-contexts for the findById() and save() methods, but it doesn't supply a transactional-context for the callFindByIdMethodAndUpdate() service method.
- Among the drawbacks, `the service-method` (in the example in book) doesn't take advantage of ACID properties as a unit-of-work, needs two physical transactions and database round trips and triggers three SQL statements instead of two.
- To provide an explicit transactional-context, you can add `@Transactional` at the service-method level. This way the SQL statements that run in the boundaries of this transactional-context will take advantage of ACID properties as a `unit of work`.
```
It is recommended to use @Transactional(readOnly = true) for query-methods as well, which you can easily achieve adding that annotation to your repository interface.
Make sure to add a plain @Transactional to the manipulating methods you might have declared or re-decorated in that interface.
```
- `So in short, I should use @Transactional on add/edit/delete queries and @Transaction(readOnly = true) on SELECT queries on all my DAO-methods?”`
```
Exactly. The easiest way to do so is by using @Transactional(readOnly = true) on the interface as it usually contains mostly finder methods and override this setting for each modifying query-method with a plain @Transactional. 
```

```
@Repository@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long> {    
    @Query("SELECT a FROM Author a WHERE a.name = ?1")    
    public Author fetchByName(String name);
    @Transactional    
    @Modifying    
    @Query("DELETE FROM Author a WHERE a.genre <> ?1")    
    public int deleteByNeGenre(String genre);}
```
In this we ensure that all query-methods are running in a read-only transactional-context by annotating the repository interface with `@Transactional(readOnly = true)`
- For Query methods that can modify data we switch to a transactional context that allows data modifications by adding @Transactional without a readOnly flag. 
- This is what exactly Spring Data does for internal for its built-in query methods.

```
@Transactional
public void longRunningServiceMethod() {
    System.out.println("Service-method start ...");
    System.out.println("Sleeping before triggering SQL
                      to simulate a long running code ...");
    Thread.sleep(40000);
    Author author = authorRepository.fetchByName("Joana Nimar");
    authorRepository.deleteByNeGenre(author.getGenre());
    System.out.println("Service-method done ...");
}
```
- Here, each query-method  (fetchByName() and deleteByNeGenre())  participates in the existing transaction opened when you call the `longRunningServiceMethod()` service-method.
- No reason to get confused to think @Transactional annotations from the repository interface will start new transactions or will consume new database connections.
- `Spring will automatically invite the called query-methods to participate in the existing transaction.`
- Spring applies the transaction propagation rules that are specific to the default transaction propagation mechanism, `Propagation.REQUIRED`.
- If you can, upgrade to Hibernate 5.2.10+ and perform the settings from Item 60 to delay the connection acquisition. Then, in most of these cases, you can use @Transactional only at the service-level and not in repository interfaces. But this means that you are still prone to forget adding @Transactional(readOnl=true) to service-methods that contain read-only database operations 