### Filtering

- Filtering in DynamoDB is almost exclusively focused on your primary key.
##### Filtering with the sort key
- Second most common filtering strategy is to add conditions on the sort key.
- This works as long as composite primary key is used.
- Assembling different collections of items.
- Filter using Composite Sort Key
- Composite sort key is when you combine multiple data values in a sort key that allow you to filter on both values.
- Example -> imagine an online e-commerce store where customers can make orders. Order can be in one of four statuses: placed, shipped, delivered or cancelled.
- In UI, application can give users a way to generate reports on past orders including looking for orders in a specific time period that have a particular status.
- For customers that have placed a lot of orders, this could be an expensive operation to retrieve all orders and filter out the ones that don't match.
- Rather than wasting a bunch of read capacity, we can use a composite sort key to handle this complex pattern.
- Combine a OrderStatus attribute and the OrderDate attribute. The value for this OrderStatusDate attribute is equal to those two attributes seperated by a #
- Create a secondary index using that attribute as a sort key.
- CustomerId is still used as primary key for this secondary index.
- Query API can be utilized to discover what's needed. 
- To find all CANCELLED orders for customer 2b5a41c0 between July 1, 2019 and September 30, 2019. A query such as below can be written

```
result = dynamodb.query(
    TableName='CustomerOrders', 
    IndexName="OrderStatusDateGSI",
    KeyConditionExpression="#c = :c AND #osd BETWEEN :start and :end",
    ExpressionAttributeNames={
        "#c": "CustomerId",
        "#ot": "OrderStatusDate"
    },
    ExpressionAttributeValues={
        ":c": { "S" : "2b5a41c0"},
        ":start": { "S" : "CANCELLED#2019-07-01"},
        ":end" : { "S" : "CANCELLED#2019-10-01"},
    }
) 
```
- Much efficient than looking through all of the order items to find proper ones.

Composite Sort Key Pattern works well when the following statements are true:
1. You always want to filter on two or more attributes in a particular access pattern.
2. One of the attributes is an enum-like value.

In this example, we wanted to allow users to filter on `OrderStatus` plus `OrderDate`, which mesn the first property is true, second the OrderStatus attribute has a limited set of potential values.

##### Sparse Indexes
-  When creating secondary index, you will define a key schema for the index.
- When you write an item into base table, DynamoDB will copy that item into your secondary index `if` it has the elements of the key schema for your secondary index.
- Crucially, if an item doesn't have those elements, `it won't be copied into the secondary index`.
- This is the important concept behind a sparse index.
- A sparse index is one that intentionally excludes certain items from your table to help satisfy a query.

This shows up in two situations.

1. Using sparse indexes to provide a global filter on an item type.
2. Projecting a single type of entity into a secondary index.

- Filter expression is applied `after` items are read, meaning you pay for all the items that get filterd out and are subjected to 1MB results limit before the filter is evaluated.
- Because of this, you cannot count on filter expressions to save a bad model.
- Filter expressions are, at best, a way to slightly improve the performance of a data model that already works well.

Filter expressions can be used in following situations.
1. Reducing response payload size.
2. Easier Application filtering.
3. Better validation around time-to-live.

Client side filtering can be used in following situations.

1. When filtering is difficult to model in the database.
2. When dataset isn't that big.

