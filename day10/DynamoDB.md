### Baby steps to Data modelling in DynamoDB

#### JOINS
- No joins in dynamoDB.
- Doing joins in application code for DynamoDB is anti-pattern.

#### Normalization in Relational Database

##### First Normal Form
- Each column value is atomic.
- Don't include multiple values in a single attribute.

##### Second Normal Form

- No partial dependencies.
- All non-key attributes must depend on the entirety of primary key.

##### Third Normal Form

- No transitive dependencies.
- All non-key attributes depend on only the primary key.

##### Why should you normalize?
- Reduce duplication of data by only writing it once and having other records point to it.
- Helps maintain data integrity over time. As if you have a piece of data duplicated across multiple records in a table, you'll need to make sure you update every record in that table in the event of a change. However, with normalized record, you update the record in one place and all records that refer to it will get the benefits of the update.

#### Why denormalize?

- Storage is cheap and plentiful. Now, need to optimize for compute by denormalizing data and storing it together in the format needed by a read operation rather than re-aggregating data at query time.
- Maintaining data integrity is still a factor. This is to be considered as data is modelled. Data integrity moves to become an application concern when data is denormalized.

#### Multiple Entity types per table

- Lack of joins mean you can have different types of entities in a single table, unlike in relational database.

#### Filtering

- Data access is one large filtering problem.
- With DynamoDB, filtering is built directly into your data model. The primary keys of the table and secondary indexes determine how data can be retrieved. Rather, than arbitrarily filtering on any attribute in your record, precise surgical requests to fetch the exact data are needed.

### Steps for modeling with DynamoDB

#### 1. Create an Entity-Relationship diagram.
#### 2. Define you access patterns
#### 3. Model your primary key structure
- Consider what your client will know at read time. else, costly additional queries are needed to be made to figure out primary key.
- Common anti-pattern is using a `createdAt` timestamp into primary key, this will help to ensure uniqueness, however, would this value be available when attempting to retrieve/update the item.
- Use Primary Key prefixes to distinguish between entity types.
Eg. If a table has Customers and CustomerOrders, pattern for customers can be CUSTOMER#<CustomerId> for the partition key and METADATA#<CustomerID> on the sort key, similarly pattern for Order would be ORDER#<OrderId> for the partition key and METADATA#<OrderId> for the sort key.

#### 4. Handle additional access patterns with secondary indexes and streams.
