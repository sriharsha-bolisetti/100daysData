#### Why Set is Better than List in @ManyToMany
- Given that Hibernate deals with `@ManyToMany` relationships as two unidirectional `@OneToMany` associations. 
- The owner-side and the child-side (the junction table) represents one unidirectional `@OneToMany`. 
- On the other hand, the non-owner-side and the child-side (the junction table) represents another unidirectional `@OneToMany` association.
- Each association relies on a foriegn key stored in the junction table.
- Entity removal or reordering results in `deleting all junction entries from junction table and reinserts them` to reflect the current Persistence Context content.
- When using the `@ManyToMany` annotation always use a Set, not List. 
- Order of the result set can achieved by following two ways
1. Use `@OrderBy` to ask the database to order the fetched data by the given columns and Hibernate to preserve this order.
2. Use `@OrderColumn` to premanently order this via an extra column.
- Adding `@OrderBy` without an explicity column will result in ordering the entities ascending by their primary keys.
- Hibernate preserves the order by a `LinkedHashSet`.
- Using @OrderBy with HashSet will preserve the order of the loaded/fetched Set, but this is not consistent across the transient state. If this is an issue, to get consistency across the transient state as well, consider explicitly using LinkedHashSet instead of HashSet. So, for full consistency, use:
```
@ManyToMany(mappedBy = "books")
@OrderBy("name DESC")
private Set<Author> authors = new LinkedHashSet<>();
```
#### Why and When to Avoid Removing Child Entities with CascadeType.Remove and orphanRemoval=True
- In the example of Author and Book entities, 
Calling the `removeBook()` method in the presence of `orphanRemoval=true` will result in automatically removing the book via a `DELETE` statement.
Calling the same method in presence of `orphanRemoval=false` will trigger an `UPDATE` statement. Since disconnecting a Book is not a remove operation, the presence of `CascadeType.REMOVE` doesn't matter. 
- Hence `orphanRemoval=true` is useful for cleaning up entities, removing dangling references that should not exist without a reference from an owner entity.
- These are not very efficient settings if they must affect a significant number of entities. 
- Deleting an author will cascalde the deletion to the associated books. This is the effect of `CascadeType.ALL`, which includes the `CascadeType.Remove`. But, before deleting the associated books, they are loaded in the Persistence context via a SELECT. If they are already in the Persistence Context then they are not loaded. If the books are not present in the Persistence Context then `CascadeType.Remove` will not take effect.
- For each book there is a separate DELETE statement. The more books there are to delete, the more DELETE statements you have and the larger the performance penalty.
- If your application triggers sporadic deletes, you can rely on CascadeType.REMOVE and/or orphanRemoval=true.
- Via this approach, you benefit from the automatic Optimistic Locking mechanism for parents and children.
Deletion of authors and the associated books can be performed via `bulk` operations. This way, you can optimize and control the number of DELETE statements. These operations are very fast, but they have three main shortcomings.
1. They ignore the automatic Optimistic Locking mechanism.
2. The Persistence Context is not synchronized to reflect the modifications performed by the `bulk` operations which may lead to an outdated context.
3. They don't take advantage of cascading removals or orphanRemoval.
-  however manage the Persistence Context synchronization issues via flushAutomatically = true and clearAutomatically = true. 
- In order to avoid outdated entities in the Persistence Context, do not forget to `flush the EntityManager before the query is executed (flushAutomatically = true) and clear it after the query is executed`(clearAutomatically = true).
- Issues may arise when you interleave `bulk` operations with managed entity operations.
