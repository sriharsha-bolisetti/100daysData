
Hyper-Ephermal Compute

- DynamoDB is a perfect fit for these hyper-ephermal applications.
- All access is over HTTP and uses AWS IAM for authentication.
- AWS is able to handle surges in traffic by using a shared Request Router across instances, which authenticates and validates your request before sending it to the proper shard for processing.

Other usecases

- Most OLTP applications.
- Caching.

DynamoDB vs MongoDB

- MongoDB uses a document oriented data model as compared to the wide column key-value storage of DynamoDB.
- MongoDB's document-oriented model provides significantly more flexibility in querying your data and altering your access patterns after the fact.
- Several index types in MongoDB aren't present in DynamoDB, such a text indexing for searching, geospatial indexing for locaiton-based queries, or multi-key indexes for searching within arrays.
- These indexes give additional power but they come at a cost, esp. as data scales.


DynamoDB Core Concepts

#### Table

- DynamoDB table differs from a relational database table in a few ways.
- A relational database table includes only a single type of entity.
- In contrast, you often include multiple entity types in the same DynamoDB table. This is to avoid the join operation, which is expensive as database scales.
- Relational database table enforces schema, however at the database level, DynamoDB  is schemaless.
- The fact dynamoDB is schemaless does not mean that your data should nto have a schema. Rather, record schema is enforced in application code, rather than in database.

#### Item

- An item is single record in dynamodb table.
- It's comparable to a row in the relational database or a document in MongoDB.

#### Attributes

- A DynamoDB item is made up of attributes, which are typed data values holding information about the element.
- When you write an item to DynamoDB, each attribute is given a specific type.
- There are ten different types in DynamoDB.

1. Scalars -> represent single and simple values. 5 scalar types.
2. Complex -> Flexible kind of attribute. 2 Complex types -> Lists, Maps.
3. Sets -> Powerful compound type that represents multiple, unique values.

The type of attribute affects which operations you can perform on that attribute in subsequent operations. 

#### Primary Key

- When creating a table in DynamoDB, you must declare a primary key for table.
- Primary key can be simple, consiting of a single value or composite, consisting of two values.
- Each item in table must include primary key, if you attempt to write an item without the primary key, it will be rejected.

#### Secondary Indexes

- Primary Keys drive key access patterns in DynamoDB.
- The way you configure primary keys may allow for one read or write access pattern but may prevent you from handling a second access pattern.
- To help with this problem, DynamoDB has the concept of secondary indexes.
- Secondary indexes allow you to reshape your data into another format for querying, so you can add additional access patterns to your data.
- When you create a secondary index on your table, you specify the primary keys for your secondary index, just like when you're creating a table. AWS will copy all items from your primary table into the secondary index in the reshaped form. You can then make queries against the secondary index.
