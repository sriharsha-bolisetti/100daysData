### Tips for DynamoDB Data Modelling

- Seperate application attributes from your indexing attributes.
- Implement your data model at the very boundary of your application i.e. all interaction with DynamoDB should be handled in `data` module that is at the boundary of your application. 
- There's a lot of work needed to reshape your application object into the format needed by DynamoDB, and it's not something that the core of your application should care about.
- Write that DynamoDB logic once at the endge of your application, and operate on application objects the rest of the time.
- Don't reuse attributes across multiple indexes
- Add a `Type` attribute to every item.
- DynamoDB data model will include multiple types of items in a single table. You'll use different patterns for the primary key of each entity to distinguish between entity types.
- It can be difficulat to easily distinguish between this with a glance or when doing a filter expression.
- A helpful tip is to include a `Type` attribute on every item written to table. This attribute will be a simple string declaring the type of entity - User, Order, SensorReading, etc.
- Adding type can also be helpful when you may need to modify existing items to decorate them with new indexing attributes.
- Final reason to add this `Type` attribute is for analytics reasons, when data from dynamoDB is exported to an external system, data can be re-normalized by moving your different entity types to their own tables.
- Having a `Type` attribute makes it easier to write the transformation query and find the right items to move around.
- Write scripts to help debug access patterns.

## DynamoDB Data Modelling Strategies
### One-To-Many Relationships

#### Denormalization by using a complex attribute

- The first way we'll use denormalization with DynamoDB is by having an attribute that uses a complex data type, like a list or a mpa.
- This violates the first tenet of database normalization: to get into first normal form, each attribute value must be atomic.
-  There are two factors to consider whether to handle a one-to-many relationship by denormalizing with a complex attribute.
1. Do you have any access patterns based on the values in the complex attribute?
- All data access in DynamoDB is done via primary keys and secondary indexes. You cannot use a complex attribute like a list or a map in a primary key. Thus, `you won't be able to make queries based on the values in a complex attribute`.
2. Is the amount of data in the complex attribute unbounded?
- A single DynamoDB item cannot exceed 400KB of data. If the amount of data that is contained in your complex attribute is potentially unbounded, it won't be a good fit for denormalizing and keeping together on a single item.

#### Denormalization by duplicating data

- In this strategy, Second Normal form principles are violated by duplicating data across multiple items.
- In general to get to second normal form, data should not be duplicated across multiple records. If data is duplicated, it should be pulled out into a seperate table. Each record that uses that data should refer to it via a foriegn key reference.

Two main questions need to asked about this strategy -
1. Is the duplicated information immutable?
2. If the data does change, how often does it change and how many items include the duplicated information?

#### Composite Primary Key + The Query API action

- Item Collections are all the items in a table or secondary index that share the same partition key.
- When using the Query API action, you can fetch multiple items within a single item collection.
- This can include items of different types, which can give you `join-like` behavior with much better performance characteristics.
Eg. Because we will include different types of items in the same table, we won't have meaningful attribute names for the attributes in our primary key. Rather, we'll use generic attribute names, like `PK` and `SK` for our primary key.

We have two types of items in our table - Organization and Users. Patterns for the PK and SK values are as follows.

| Entity | PK | SK |Organizations |
---------------------------------- 
Organizations | ORG#<OrgName> | METADATA#<OrgName>|
Users | ORG#<OrgName> | USER#<UserName> |


This primary key design makes it easy to solve four access patterns.
1. Retrieve an organization. 
2. Retrieve an organization and all Users within the organization.
3. Retrieve only the Users within an Organization. 
4. Retrieve a specific User. 

The second pattern above - Retrieve an organization and all users within the organization is most interesting for one-to-many relationships. Here we're emulating a join operation in SQL by locating the parent object (the Organization) in the same item collection as the related objects (the Users). We are pre-joining our data by arranging them together at write time.

#### Secondary Index + the Query API action

- Similar pattern for one-to-many relationships is to use a global secondary index and the Query API to fetch multiple items in a single request.
- This patterns is almost the same as the previous pattern, but it uses a secondary index rather than the primary keys on the main table.
- This may need to be used instead of previous pattern because the primary keys in the table are reserved for another purpose.
(I don't think i fully understood who GSI work in DynamoDB)

#### Composite Sort Keys with Hierarchial data

- What if data has more than two level or hierarchy? We don't want to keep adding secondary indexes to enable arbitrary levels of fetching throughout the hierarchy.
- Common example is around location-based data. eg. Tracking all the locations of Starbucks around the world. The requirement is to be able to filter Starbucks locations on arbitrary geographic levels - by country, by state, by city, or by zip code.

- This problem can be solved by using composite sort key.
- Composite Sort Key means smashing a bunch of properties together in our sort key to allow for different search granularity.

Partition key can be country where the Starbucks is located. For the sort key, we include the State, City and ZipCode, with each level seperated by a `#`. With this pattern, we can search at four levels of granularity using just our primary key.

1. Find all locations in a given country.
- Use a query with a key condition expression of `PK = <Country>` where Country is the country you want to search.
2. Find all locations in a given country and state.
- Use a Query with a condition expression of `PK= <Country> AND begins_with (SK, '<State>#')`.
3. Find all locations in a given country, state, and city.
- Use a Query with a condition expression of `PK = <Country> AND begins_with(SK, '<State>#<City>')`
4. Find all locations in a given country, state, city and zip code.
- Use a Query with a condition expression of `PK = <Country> AND begins_with(SK, '<State>#<City>#<ZipCode>')`

Composite sort key pattern won't work for all scenarios but it can be great in the right situation. This works best when

1. YOu have many levels of hierarchy and have access patterns for different level within the hierarchy.
2. When searching at a particular level in the hierarchy, you want all the subitems in that level rather than just the items in that level.
