### Many to Many Relationships in DynamoDB

- Many-to-many relationships are tricky because you often want to query both sides of the relationship.
- In Relational database, many-to-many relationships are handled by using a linking table which serves as an intermediary between your tow objects.
- Each object table will have a one-to-many relationship with the linking table, and you can traverse these relationships to find all related records for a particular entity.

##### Shallow Duplication

Illustration by example -

- Imagine an application with classes and students. A student is enrolled in many classes, and a class has many students.
- One of the access patterns is to fetch a class and all of the students in the class. 
- However, fetching information about a class doesn't require a detailed information about each student in the class, only a subset of information such as name or an Id is needed.
- To handle this situation, class items can be modelled sucht that information about the students enrolled in the class would be duplicated into the item.

Shallow duplication strategy works when below properties are true.
1. There is a limited number of related entities in the duplicated relationship.
- DynamoDB items have a 400KB limit. This strategy won't work if there is a high or unlimited number of related entities that are needed to be maintained.
2. The duplicated information is immutable.
- This strategy works if the information that is duplicated does not change. If the information is frequently changing, a lot of time needed to spent looking for all the references to the data and updating accordingly. This will result in additional write capacity needs and potential data integrity issues.

##### Adjacency List

- With this strategy, you model each top-level entity as an item in your table. The relationship between entities are also modelled as item in the table. Then, items are organized such that both top-level entity and information about the relationship is fetched in a single request.

Illustration by example

Movies:
- PK: `MOVIE#<MovieName>`
- SK: `MOVIE#<MovieName>`

Actors:
- PK: `ACTOR#<ActorName>`
- SK: `ACTOR#<ActorName>`

Roles:
- PK: `MOVIE#<MovieName>`
- SK: `ACTOR#<ActorName>`

Movie and Actor items are top-level items, and the Role item represents the many-to-many relationship between Movies and Actors.

- With this base table configuration, notice that Movie and Role items are in the same item collection.
- This allows us to fetch a movie and actor roles that played in the movie with a single request by making a Query API call that uses `PK = MOVIE#<MovieName>` in the key condition expression.
- We can then add a global secondary index that flips the composite key elements.
- In the secondary index, the partition key is `SK` and the sort key is `PK`. 

Now Actor item is in the same item collection as the actor's role items, allowing us to fetch an actor and all roles in a single request.

- This pattern works best when the information about the relationshipo between the two is immutable. 
- In the example, `PK` and `SK` were flipped for secondary index, this might be known as `inverted index`. 
- However, if other items are present in the table, `PK` and `SK` may not be flipped as this may not enable the access patterns needed. Two new attributes can be created `GSI1PK` and `GSI1SK` that have the flipped values from `PK` and `SK` for the items in many-to-many relationship. 

#### Materialized Graph

- A graph is made up of `nodes` and `edges`. A node is an object or concept, such as a place, person or a thing.
- Various `nodes` are connected via `edges`, which indicate `relationships` between nodes.
- To use a materialized graph in DynamoDB, create your nodes as an item collection.
- Then you can use a secondary index to reshuffle those items and group them according to particular relationships.
- The materialized graph pattern can be useful for highly-connected data that has a variety of relationships. You can quickly find a particular type of entity and all the entities that relate to it.

#### Normalization and Multiple Requests
Illustration by Example

Users
- PK - `USER#<Username>`
- SK - `USER#<Username>`

Following
- PK - `USER#<Username>`
- SK - `FOLLOWING#<Username>`

When user, say Harsha wants to view all the users he's following, the Twitter backend will do a two-step process -

1. Use the Query API call to fetch the User item and the initial Following items in Harsha's item collection to find information about the user and the first few uses he's following.
2. Use the BatchGetItem API call to fetch the detailed User items for each Following item that was found for the user. This will provide the authoritative information about the followed user, such as the display name and profile.

This isn't idea as we're making multiple rquests to DynamoDB. However, there's no better way to handle it. If you have highly-mutable many-to-many relationships in DynamoDB you'll likely need to make multiple requests at read time.


### What is Global Secondary Indexes?

- GSI allows you to query attributes of a row that are not the partition key that you originally selected when you created your table.
- Define a GSI index -> involves selecting a new partition key.
- Projected attributes -> you can just select subset of attributes you care about
- Creating a GSI clones your primary table using your new partion key, but keeps these two tables in sync.
- GSI table plays by the same rules as a normal Dynamo table.
- GSI Partition key requires uniform data distribution.
- Define RCU/WCU capacity separately on the index.
- Throttling can be enabled on just GSI table.
- Writes to the main table result in a write to the GSI, effectively double the cost of writing.
- Can only have max 20 GSI.
- Writes to the main table are `eventually replicated` on the GSI. (No Guaranteed SLA).
- Keep your WCU capacity on your GSI tables >= the WCU capacity on your main table.
- Separate metrics on your GSI - monitor them accordingly.

### Local Secondary Index 

- DynamoDB can only have `one` Sort Key. 
- LSI can essentially act as second sort key.
- Can `only` be defined on `table creation time`.
- Only limited to 5 LSI.
- No extra cost.