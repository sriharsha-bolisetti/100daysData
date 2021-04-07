### DynamoDB

- NoSQL database 
- Concept of denormalized data model.
- NoSQL databazses are designed to eliminate the need for complex joins between tables.
- At the core of all NoSQL databases there is a collection of disparate objects, tied together by indexed common attributes, and queried with conditional select statements to produce result sets.

- DynamoDB has support for two similar data models.
- One you can use it as a key-value store. 
- To handle more complex access patterns, DynamoDB can also be used a `wide-column store`. 
- A wide-column store is like a super-charged version of a hash table where the value for each record in your hash table is a B-Tree.
- DynamoDB accelerator DAX which is a fully managed in-memory cache for DynamoDB table.
-  DynamoDB uses AWS IAM for authentication and authorization of database requests rather than a username and password model that is common with other database systems.
- Change data capture with DynamoDB streams. 
- With DynamoDB streams, you can get a transactional log of each write transaction in your DynamoDB table.
- You can programatically process this log, which opens up a huge number of use cases.
