### E-Commerce exampe

-- very rough notes --

- Query API can only fetch items with the same partition key, so we need to make sure our Order items have the same partition key as the Customer items.
- Also, we want to retrieve our Orders by the time they were placed, starting with the most recent.

PK:- `CUSTOMER#<Username>`
SK:- `#ORDER#<OrderId>`