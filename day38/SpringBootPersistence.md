#### Auto Commit Mode
- Auto-commit mode means that when a statement is completed, the method `commit` is called on that statement automatically. 
- Auto-commit in effect makes every SQL statement a transaction.
- The commit occurs when the statement completes or the next statement is executed, whichever comes first.
- A SQL statement executed in autocommit mode cannot be rolled back.
- The alternative to autocommit mode means that the SQL client application itself is responsible for ending transactions explicitly via the commit or rollback SQL commands.
- Non-autocommit mode enables grouping of multiple data manipulation SQL commands into a single atomic transaction.
- Most DBMS force autocommit for every DDL statement, even in non-autocommit mode. In this case, before each DDL statement, previous DML statements in transaction are autocommitted. Each DDL statement is executed in its own new autocommit transaction.
- Many DBMS enable the autocommit mode on every new connection by default. This autocommit is useful for ad hoc execution of SQL.
- Imagine that you connect to your database with a SQL console and that you run few queries, and maybe even update and delete rows. The interactive data access is adhoc, most of the time you don't have a plan or a sequence of statements that you consider a unit of work.
- The default autocommit mode on the database connection is perfect for this kind of data access - after all, you don't want to type `begin a transaction` and `end a transaction` for `every sql statement` you write and execute.
- You're working effectively nontransactionally, because there are no atomicity or isolation guarantees for you session with the SQL console. `The only guarantee is that a single SQL statement is atomic`.

 ```An application, by definition, always executes a planned sequence of statements. It seems reasonable that you always create a transaction boundaries to group your statements into units that are atomic. There, the autocommit mode has no place in an applicaiton.``` 

#### How to Delay Connection Acquisition Until It's Really Needed

- In the case of `resource-local`, Hibernate will acquire the database connection of a JDBC transaction right after the transaction starts i.e. in Spring, a method annotated with `@Transactional` acquires the database connection immediately after it is called.
- In resource-local, a database connection is acquired immediately because Hibernate needs to check the JDBC Connection auto-commit status. If this is true then Hibernate will disable it.
- Practically, the database connection is useless to the application until the first JDBC statement of the current transaction is triggered; holding the database connection unused for this time induces a performance penalty that can have a big impact if there are many or/and time-consuming tasks before the first JDBC statement.
- In order to prevent this performance penalty, you can inform Hibernate that you disabled auto-commit, so no check is needed. To do this, follow the below two steps
1. Turn off auto-commit. Eg. check you pool connection for a method of type `setAutoCommit (boolean commit)` and set it to `false`.
2. Set to `true` the Hibernate-specific property: `hibernate.connection.provider_disables_auto-commit`.
- By default, Spring Boot relies on HikariCP, and you can turn off auto-commit in `application.properties` via the `spring.datasource.hikari.auto-commit` property. So, the below two settings need to be added to `application.properties`
```
spring.datasource.hikari.auto-commit=false
spring.jpa.properties.hibernate.connection.provider_disables_autocommit=true
```
- As a rule of thumb, for `resource-local` JPA transactions, it is always good practice to configure the connection pool to disable the auto-commit and set `hibernate.connection.provider_disables_autocommit` to `true`. 
