#### Consistency

Two consistency options available for DyanmoDB.
- Strong Consistency.
- Eventual Consistency.

##### Consistency Consideration 1
- When reading data from your base table, you can choose desired consistency level.
- By default, DynamoDB will make an eventually-consistent read.
- However, you can opt into a strongly-consistent read by passing `ConsistentRead=True` in the API call.
- An eventually-consistent read consumes half the write capacity of a strongly consistent read and is a good choice for many applications.

##### Consistency Consideration 2

- Consistency should be thought of when choosing secondary index type.
- A local secondary index will allow you to make strongly-consistent reads against it.
- A global secondary index will only allow you to make eventually-consistent reads. 

### DynamoDB Limits
#### Item size limits
- 400KB of data.

#### Query and Scan Request Size Limits
- Query and Scan will read a maximum of 1MB of data from your table.
- This 1MB limit is applied `before` any filter expressions are considered.
- If you have a request that will address more than 1MB of data, you will need to paginate through the results by making follow-up requests to DynamoDB.

#### Partition Throughput Limits

- A single partition can have a maximum of 3000 Read Capacity Units or 1000 Write Capacity Units.
- Capacity Units are on a per-second basis, and these limits apply to a single partition, not the table as a whole.

#### Item Collection Limits

- If you have a local secondary index, a single item collection cannot be larger than 10GB.
- If you have a data model that has many items with the same partition key, this could bite you bad time because your writes will suddenly get rejected once you run out of partition space.
- The partition size limit is not a problem for a global secondary indexes.
- If the items in a global secondary index for a partition key exceed 10GB in total storage, they will be split across different partitions under the hood.
- This will happen transparantly.


## DynamoDB API Action Types

#### Item Based Actions

- Item-based actions are used whenever you are operating on a specific item in your DynamoDB table.

1. GetItem
2. PutItem
3. UpdateItem
4. DeleteItem

Three rules around item-based actions.
1. Full primary key must be specified in the request.
2. All actions to alter data - writes, updates and deletes must use an item-based action.
3. All items are to be performed on main table, not on secondary index.

Combination of first two rules -> You cannot do something as "Update the attribute X for all items with a partition key of Y", you would need to specify the full key of each of the items you'd like to update.

There are two sub-categories of single item API actions
- Batch Actions.
- Transation Actions.

There is a subtle difference between the batch API actions and the transactional API actions. In a batch API request, your reads or writes can succeed or fail independently. The failure of one write won't affect the other writes in the batch.

With the transactional API actions on the other hand, all of you reads or writes will succeed or fail together.

#### Query

- Query API lets you retrieve multiple items with the same partition key.
- You can use query api to easily fetch all related objects in a one-to-many relationship or a many-to-many relationship.

#### Scan

- The Scan API is the bluntest tool in the DynamoDB toolbox. 
- A Scan will grab everything in a table. If you have a large table, this will be infeasible in a single request, so it will paginate.
