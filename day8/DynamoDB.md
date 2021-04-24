# DynamoDB Expressions

- Expressions are sorta like mini-SQL statements.

Five types of expressions in DynamoDB.
- Key Condition Expressions -> used in query API 
- Filter Expressions
- Projection Expressions
- Condition Expressions
- Update Expressions

#### Key Condition Expressions
```
result = dynamodb.query(
      TableName='MovieRoles',
      KeyConditionExpression="#a = :a",
      ExpressionAttributeNames={
          "#a": "Actor"
      },
      ExpressionAttributeValues={
          ":a": { "S": "Natalie Portman" }
} )
```
- Partition Key of our MovieRoles table is `Actor`. Once we've declared the attribute name in `ExpressionAttributeValues`, our `KeyConditionExpression` expresses our desire to retrieve all items with Natalie Portam as the actor.
- You can also use conditions on the sort key in your key condition expression.
- This is useful for finding a specific subset of your data.
- All elements with a given partition key are sorted accroding to a sort key.

#### Filter Expressions

- This expression is also for a read-based operation.
- A filter expression is available for both Query and Scan operations.
- Filter expression will filter items that matched your key condition expression but not filter condition.
- Filter expression can be applied on `any` attribute in the table, not just those in the primary key.
- Filter expressions are useful for a few limited contexts
    1. Reducing response payload size.
    2. Easier Application Filtering.
    3. Better Validation around time-to-live TTL expiry. 

#### Projection Expressions

- Read-based expression.
- Similar to Filter expression, while filter expression works on an item-by-item basis, the projection expression works on an `attribute-by-attribute` basis.
- Projection expression can be used to access nested properties, such as in a list or map attribute.
- If you have a large object and you know exact property you want, you can really cut down on the bandwidth by specifying only the elements you need.

#### Conditional Expressions

- This is for write operations.
- Condition expression are available on every operation where you will alter an item - PutItem, UpdateItem, DeleteItem, and their batch and transactional equivalents.
- They allow you to assert specific statements about the status of the item before performing the write operation.
- If the condition expression evaluates to false, the operation will be canceled. 
- Without the existence of condition expressions, you would need to add costly additional requests to fetch an item before manipulating it, and you would need to consider how to handle race conditions if another request tried to manipulate your item at the same time.
- Condition expressions can be used to check across multiple entities. 
- You can use DynamoDB transactions, TransactWriteItem API allows you to use up to 10 items in a single request.
- These can be a combination of different write operations - PutItem, UpdateItem or DeleteItem - or they can be ConditionChecks, which simply assert a condition about a particular item.
- If you keep two operations in our TransactWriteItems request, first a ConditionCheck on Admins item for this organization to assert that the requesting user is an admin in the account, second, there is a Delete operation to remove the billing record for this organization. These operations will `succeed` or `fail` together.
- If the condition check fails because the user is not an administrator, the billing record won't be deleted.

#### Update Expressions
- This manupulates an item rather than reading or asserting the truth about existing properties.
- When using UpdateItemAPI you will only alter the properties you specify.
- If the item already exists in the table, the attributes that you don't specify will remain the smae as before the update operation.
- If this behavior is not desired PutItemAPI should be used to completely overwrite the item with the only the properties you give it.
- For update expression, you need to state the changes you want to make.
- SET -> to add or overwrite an attribute
- REMOVE 
- ADD
- DELETE

The four verbs can be used in combination in one update statement.

- If you have multiple operations for a single verb, you only state the verb once and use commas to seperate the clauses

``` 
UpdateExpression="SET Name= :name, UpdatedAt = :updatedAt"
```

- If multiple verbs in one update expression is desired
```
UpdateExpression="SET Name = "name, UpdatedAt = :updatedAt REMOVE InProgress"
````
