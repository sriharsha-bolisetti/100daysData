### Single Table Design

- Solution to avoid join drawbacks -> Pre-Join your data into the item collections.
- If you need to retrieve multiple heterogeneous items in a single request, you organize those items so that they are in the same item collection.
- Main benefit of single-table design is the performance improvement by making a single request to retrieve all needed items.
- Downside is inflexibility of adding new access patterns.
- DynamoDB cannot be used with GraphQL.