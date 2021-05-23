- `@Repository` annotation is useful for translating the unchecked SQL specific exception to Spring exceptions. This way, we have to handle only `DataAccessException (and its subclasses)`
- Spring Data repositories are already backed by a Spring proxy. In other words, using `@Repository` doesn't make any difference.

#### Effectivey shape the @OneToMany Association

- Always Cascade from parent side to child side.
-  So we add the cascade type in the Author entity:
`@OneToMany(cascade = CascadeType.ALL)`
- `CascadeType.ALL` is that the persistence will propogate all entity manager operations - Persist, Remove, Refresh, Merge, Detach to the relating entities.
- `mappedBy` attribute characterizes a bidirectional association and must be set on the parent-side.
- and Add @ManyToOne on the child side referenced by mappedBy.
- Via mappedBy, the bidirectional @OneToMany association signals that it mirrors the @ManyToOne child-side mapping
- `@OneToMany(cascade = CascadeType.ALL,
              mappedBy = "author")`
- Setting `orphanRemoval` on the parent-side guarantees the removal of children without references.
- `Keep both sides of association in sync`.
- If you don't strive to keep both sides of the association in sync, then the entity state transitions may lead to unexpected behaviors.
- `Override Equals() and Hashcode()` - By properly overriding these methods, application obtain the same results across all entity state transitions.
- `If toString() needs to be overridden, then be sure to involve only the basic attributes fetched when the entity is loaded from the database.`
- Use `@JoinColumn` to specify the join column name - Join column defined by the owner entity stores the id value and has a foreign key to Author entity.

#### Avoid Unidirectional @OneToMany Association
- Author and Book entities are involved in a unidirectional @OneToMany association mapped, as follows:
`@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
private List<Book> books = new ArrayList<>();`
- The Missing `@ManyToOne` association leads to a seperate junction table(author_books) meant to manage the parent-child association.
- Adding `@OrderColumn` annotation, the unidirectional `@OneToMany` association becomes ordered. This annotation instructs Hiberate to materialize the element index (index of every collection element) into a seperate database column of the junction table so that the collection is sorted using an `Order By` clause.
- In this case, the index of every collection element is going to be stored in the `books_order` column of the junction table.
```
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@OrderColumn(name = "books_order")
private List<Book> books = new ArrayList<>();
```
- Adding @OrderColumn can bring some benefits for removal operations. Nevertheless, the closer an element to be removed is to the head of the fetched list, the more UPDATE statements are needed. This causes performance penalties. Even in the best-case scenario (removing an element from the tail of the collection), this approach is not better than bidirectional @OneToMany association.
- Adding `@JoinColumn` instructs Hibernate that the `@OneToMany` association is capable of controlling the child table foriegn key. This means the junction table is eliminated and the number of tables is reduced from three to two.
- Adding @JoinColumn can provide benefits over the regular unidirectional @OneToMany, but is not better than a bidirectional @OneToMany association. The additional UPDATE statements still cause a performance degradation.
- Adding @JoinColumn and @OrderColumn at the same time is still not better than bidirectional @OneToMany. Moreover, using Set instead of List or bidirectional @OneToMany with @JoinColumn (e.g., @ManyToOne @JoinColumn(name = "author_id", updatable = false, insertable = false)) still performs worse than a bidirectional @OneToMany association.
- As a rule of thumb, a unidirectional @OneToMany association is less efficient than a bidirectional @OneToMany or unidirectional @ManyToOne associations.