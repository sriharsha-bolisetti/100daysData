#### DynamoDB API

##### Expression Names and Values

- KeyConditionExpression
Eg. `  KeyConditionExpression: '#actor = :actor AND #movie BETWEEN :a AND :m'`

Some placeholders start with a `#`, like `#actor` and `#movie`, other placeholders start with a `:`, like `:actor`, `:a`, and `:m`.

-  The ones that start with colons are `expression attribute values`.
- They are used to represent the value of the attribute you are evaluating in your request.
- Avoid ODMs
