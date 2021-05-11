### Data Modeling Examples

#### Session Store


- Uniqueness requirement on session token.
- If you want to guarantee uniqueness on a value in DynamoDB, you have to build that value into your primary key. Hence, `SessionToken` is the primary key.
- When inserting items into the table, you want to use a condition expression that ensures there is not an existing item with the same token.
- Time-based deletion of sessions -> DynamoDB TTL can be chosen for this.
- Manual Deletion of a user's tokens 
- to handle manual deletion, need to provide a way to lookup session tokens by a username.
