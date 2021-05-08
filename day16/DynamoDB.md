### Strategies for Sorting

- Everything in DynamoDB flows through primary keys. It's true for relationships, true for filtering and true for `sorting`.
- If a specific ordering when retrieving multiple items in DynamoDB is needed, following two rules must be followed.
1. Use Composite primary key.
2. Ordering must be done with the sort key of a particular item collection.

- Sort keys of type number are sorted according to values of the number.
- Sort keys of type string or binary are sorted in order of UTF-8 bytes.
- As sorting UTF-8 Bytes is lexicographical which means uppercase letters come before lowercase letters, it's better to `standardize` all sort keys in all uppercase or all lowercase values.
- Choose a sortable timestamp - either epoch or iso8601 timestamps but not a diplay friendly format like "May 11, 1999", as this is not sortable in DynamoDB.
- If unique sortable IDs are desired, KSUID implementation can be considered. 
- A KSUID is a K-Sortable Unique Identifier, it's a unique identifier that is prefixed with a timestamp but also contains enough randomness to make collisions very unlikely.
