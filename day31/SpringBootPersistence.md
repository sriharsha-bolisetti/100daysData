- Unidirectional `@ManyToOne` association is quite efficient and it can be used whenever a bidirectional `@OneToMany` association is not needed. 
###### How to map ManyToMany Association
- Bidirectional `@ManyToOne` association can be navigated from both sides, therefore, both sides can be parents.
- Since both are parents, `none of them will hold a foreign key`. In this association, there are two foreign keys that are stored in a separate table, known as `junction or join table`. This junction table is `hidden` and it plays the child-side role.
Below steps on how to map 
1. Choose the owner of the relationship.
2. Always use set not list
3. Keep both sides of the association in sync
4. Avoid `cascadetype.all` and `cascadetype.remove` 
    - In most cases, cascading removals are bad ideas.
    - For example, removing an `Author` entity should not trigger a `Book` removal because the `Book` can be referenced by other authors as well. Hence avoid `cascadetype.all` and `cascadetype.remove` and rely on explicit `cascadetype.persist` and `cascadetype.merge`.
    - `orphanRemoval` option is defined on `@OneToOne` and `@OneToMany` relationship annotations, but on neigher of the `@ManyToOne` or `@ManyToMany` annotations.
5. Setting up the join table
6. Use lazy fetching on both sides of the association.
7. Override `equals()` and `hashcode()`.
8. Pay attention to how `toString()` is overriden.

###### mappedBy vs @JoinColumn
- The annotation `@JoinColumn` indicates that this entity is the `owner` of the relationship, i.e. the corresponding table has a column with a foreign key to the referenced table, Whereas attribute `mappedBy` indicates that the entity in this side is the `inverse of the relationship`, and the owner resides in the `other` entity. This also means that you can access the other table from the class which you've annotated with `mappedBy`.

Another way to think about this is
- `@JoinColumn` -> the purpose of `@JoinColumn` is to create a join column if one does not already exist. If it does, then this annotation can be used to `name` the join column.
- `mappedBy` -> Purpose of the `mappedBy` parameter is to instruct JPA: Do NOT create another join table as the relationship is already being `mapped by the opposite entity` of this relationship.
