#### How to Fetch associations via JPA Entity Graphs
- Entity graphs are introduced to help improve the performance of loading entities by solving lazy loading exceptions and N+1 issues.
- The developer specifies the entity's related associations and basic fields that should be loaded in `single SELECT statement`.
- Multiple entity graphs for the same entity can be defined and can chain any number of entities, and even use sub-graphs to create complex fetch plans.
- Entity graphs are global and reusable across the entities (Domain model).
- To override the current `FetchType` semantics, there are two properties you can set
1. Fetch Graph - The attributes present in `attributeNodes` are treated as `FetchType.EAGER`. The remaining attributes are treated as `FetchType.LAZY`, regardless of the default/explicit `FetchType`.
2. Load Graph - The attributes present in attributeNodes are treated as FetchType.EAGER. The remaining attributes are treated according to their specified or default FetchType.
##### Defining an Entity Graph via @NamedEntityGraph
- The @NamedEntityGraph annotation occurs at entity-level. Via its elements, the developer can specify a unique name for this entity graph (via the name element) and the attributes to include when fetching the entity graph.

Eg. Let's put the entity graph in code in the `Author` entity.
```
@Entity
@NamedEntityGraph(
    name = "author-books-graph",
    attributeNodes = {
        @NamedAttributeNode("books")
    } 
)
public class Author implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String genre;
    private int age;
    @OneToMany(cascade = CascadeType.ALL,
             mappedBy = "author", orphanRemoval = true)
      private List<Book> books = new ArrayList<>();
    // getters and setters omitted for brevity
}

```
- The `AuthorRepository` is the place where the entity graph should be specified.
- Spring Data provides support for entity graphs via `@EntityGraph` annotation.
- Overriding a Query method
- The code to use the entity graph (author-books-graph) to find all Authors, including the associated Book is as follows
- EntityGraphType.Fetch is the default and indicates a fetch graph, EntityGraphType.Load can be specified for a load graph.
```
@Repository
@Transactional(readOnly = true)
public interface AuthorRepository extends JpaRepository<Author, Long>{
    @Override
    @EntityGraph(value = "author-books-graph", type = EntityGraph.EntityGraphType.FETCH)
    public List<Author> findAll();
}
```
Calling the findAll() method will result in the following SQL SELECT statement:
```
SELECT
  author0_.id AS id1_0_0_,
  books1_.id AS id1_1_1_,
  author0_.age AS age2_0_0_,
  author0_.genre AS genre3_0_0_,
  author0_.name AS name4_0_0_,
  books1_.author_id AS author_i4_1_1_,
  books1_.isbn AS isbn2_1_1_,
  books1_.title AS title3_1_1_,
  books1_.author_id AS author_i4_1_0__,
  books1_.id AS id1_1_0__
FROM author author0_
LEFT OUTER JOIN book books1_
  ON author0_.id = books1_.author_id
```
Notice that the `generated query took into account the entity graph specified via @EntityGraph`.

##### Ad hoc entity graphs
- An ad hoc entity graph can be defined via the attributePaths element of the @EntityGraph annotation. 
- The entity's related associations and basic fields that should be loaded in a single SELECT are specified as a list separated by comma of type,`@EntityGraph(attributePaths = {"attr1", "attr2", ...}.`
#### JPA - @Embedded, @EmbeddedId, @MapsId

- Composite primary keys are keys that use more than one column to identify a row in the table uniquely.
- We represent a composite primary key in Spring Data by using the `@Embdeddable` annotation on a class.
- This key is then embedded in the table's corresponding entity class as the composite primary key by using the `@EmbeddedId` annotation on a field of the `@Embeddable` type.
- Eg. Consider a book table, where a book record has a composite primary key consiting on author and name. Sometimes, we might want to find books by a part of the primary key. For example, a user might want to search for books only by a particular author.
- This example application will consist of an `@Embeddable` BookId and `@Entity` Book with `@EmbeddedId` BookId.
```
@Embeddable
public class BookId implements Serializable {

    private String author;
    private String name;

    // standard getters and setters
}
```
- Our Book entity has `@EmbeddedId` BookId and other fields related to book.
- BookId tells JPA that the Book entity has a composite key.
```
@Entity
public class Book{
    @EmbeddedId
    private BookId id;
    private String genre;
    private Integer price;
}

```
- JPA Repository and Method Naming
```
@Repository
public interface BookRepository extends JpaRepository<Book, BookId>{
    List<Book> findByIdName(String name);
    List<Book> findByIdAuthor(String author);
}
```
We use a part of the id variable's field names to derive our Spring Data query methods.
- Here JPA interprets the partial primary key query as
```
findByIdName -> directive "findBy" field "id.name"
findByIdAuthor -> directive "findBy" field "id.author"
```