#### How to Optimize Unidirectional/Bidirectional @OneToOne via @MapsId
- The `@MapsId` annotation can be applied to `@ManyToOne` and unidirectional `@OneToOne` associations.
- Via this annotation, the book table's primary key can also be a foreign key referencing the author's table primary key.

`@MapsId` to the child entity as shown below -

```
@Entity
public class Book implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    private Long id;
    private String title;
    private String isbn;
    
    @MapsId
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    privar Author author;
```
- Check out the identifier of the Book entity. There is no need for it to be generated (@GeneratedValue is not present) `since this identifier is exactly the identifier of the author association`. The Book identifier is set by Hibernate on your behalf.
- The @JoinColumn annotation is used to customize the name of the primary key column.
- The parent entity is quite simple because there is no need to have a bidirectional `@OneToOne`. The code for Author is as follows
```
@Entity
public class Author implements Serializable{
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String genre;
    private int age;
}
```
Now you can persist a `Book` via a service-method as follows
```
@Transactional
public void newBookOfAuthor(){
    Author author = authorRepository.findById(1L).orElseThrow();
    Book book = new Book();
    book.setTitle("A History of Ancient Prague");
    book.setIsbn("001-JN");
    //this will set the id of the book as the id of the author
    book.setAuthor(author);

    bookRepository.save(book);

}
```
- Calling `newbookOfAuthor()` reveals the following `INSERT` statements.
```
INSERT INTO book (isbn, title, author_id)
  VALUES (?, ?, ?)
Binding:[001-JN, A History of Ancient Prague, 1]
```
- Notice the `author_id` was set to the author identifier.
- `This means that the parent and the child tables share the same primary key`

- Further the developer can fetch the Book via the Author identifier, since the identifier is shared between Author and Book, the developer can rely on author.getId() to specify the Book identifier.

```
@Transactional(readOnly = true)
public Book fetchBookByAuthorId(){
    Author author = authorRepository.findById(1L).orelseThrow();
    return bookRepository.findById(author.getId()).orElseThrow();
}
```
###### Advantages of using @MapsId, as follows:
- If Book is present in the Second Level Cache it will be fetched accordingly, no extra database round trip is needed. This is the main drawback of a regular unidirectional @OneToOne.
- Fetching the Author doesn't automatically trigger an unnecessary additional query for fetching the Book as well. This is the main drawback of a regular bidirectional @OneToOne.
- Sharing the primary key reduces memory footprint - no need to index both the primary key and the foreign key.