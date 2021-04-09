#### Primary key

- Simple Primary Key
Consists of single element called a `partition key`.

- Composite Primary Key

Consists of two elements called - `partition key` and a `sort key`.


- Type of primary key to choose will depend on access patterns.
- Simple primary key allows you to fetch only a single item at a time.
- Works well for one-to-one operations where you are only operating on individual items.

Composite primary keys will enable a `fetch many` access pattern.
- With composite Primary key, you can use the Query API to grab all items with the same partition key.
- Composite primary keys are great for handling relations between items in you data and for retrieving multiple items at once.

#### Secondary Indexes

- When creating a secondary index, you will need to specify the key schema of your index.
- Key Schema is similar to the primary key of your table

Two kinds of secondary indexes in DynamoDB

1. Local Secondary Indexes.
2. Global Secondary Indexes.

- Local Secondary Index uses the same partition key as your table's primary key but a different sort key.
- This is a good fit when you are filtereing your data by the same top-level property but have access patterns to filter your dataset further.
- In contrast, with a Global Secondary Index, you can choose any attributes you want for your partition key and your sort key.
- GSI needs additional throughput for the secondary index.
- The read and write throughput for the index is seperate from the core table's throughput. This is not the case for local secondary indexes, which uses the throughput from the core table.
- Global Secondary Indexes only support eventual consistency.
- Local secondary indexes allow you to opt for strongly-consistent reads if needed. Although, it consumes more throughput.
- GSI can be created after the table exists.

#### Item Collections

- Item collection refers to a group of items that share the same `partition key` in either the base table or a secondary table.
- Item collections are useful for patitioning. 
- DynamoDB partitions your data across a number of nodes in a way that allows for consistent performance as you scale.
- However, all the items with the same partition key will be kept on same storage node.
- Item collections are useful for data modeling.

#### DynamoDB Streams

- With DynamoDB streams, you can create a stream of data that includes a record of each change to an item in your table.
- Whenever an item is written, updated or deleted, a record containing the details of that record will be written to your DynamoDB stream.
- This stream can be processed with AWS Lambda or other compute infrastructure.

#### Time to Live

- TTLs allow you to have DynamoDB automatically delete items on a per-item basis.
- Great option for storing short-term data in DynamoDB as you can use TTL to clean up your database rather than handling it manually via a scheduled job.

#### Partitions

- Partitions are the core storage units underlying DynamoDB
- Data is sharded across multiple server instances in DynamoDB.
- When a request comes into DynamoDB, the request router looks at the partition key in the request and applies a hash function to it.
- Result of the hash function indicates where that data will be stored.
- DynamoDB can add additional storage nodes infinitely as data scales up.

