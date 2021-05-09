### Strategies for Migration

#### Adding new attributes to an existing entity

- Adding application attributes is the easiest type of migration because you don't really think of application attributes while modeling data in DynamoDB.
- You are primarily concerned with the attributes that are used for indexing within DynamoDB.
- The schemaless nature of DynamoDB makes it simple to add new attributes in your application without doing large-scale migrations.

#### Adding a new entity type without relations
- Illustration by example -
- Imagine new entity type - Projects.
- A Project belongs to an organization, and an organization can have multiple projects.
- Thus there is a one-to-many relationship between organizations and projects.
- When reviewing access pattern we have a `Get Projects for Organization` access pattern. But we don't need to fetch the parent Organization item as part of it. Because of this, we can model the projects into a new item collection altogether.
- As we don't have to make any changes to `existing` items to handle this new entity and its access patterns, it's purely additive change.

#### Adding a new entity type into an existing item collection
- Let's take a similar to the last one but we have access pattern of `Fetch parent entity and its related entities`.
- This is an additive change that don't require making changes to our existing item collection.

#### Adding a new entity type into an new item collection.
- Continuing example of social application with Psts and Likes.
- Needed to add comments such that Users can comment on a Post to give encouragement or argue about something.
- With the new Comment entity, we have a relational access pattern where we want to fetch a Post and the most recent Comments for that Post.
- The post item collection is already being used on the base table. To handle this access pattern, we will need to create a `new item collection in a global secondary index`.
- Add following attributes to the Post item.
- GSI1PK: `POST#<PostId>`
- GSI1SK: `POST#<PostId>`

Create a comment item with following attributes
- PK: `COMMENT#<CommentId>`
- SK: `COMMENT#<CommentId>`
- GSI1PK: `POST#<PostId>`
- GSI1SK: `COMMENT#<Timestamp>`

### Additional Strategies

#### Ensuring an attribute is unique.
- You can use DynamoDB transactions to include write operations on multiple items in a single request and the entire request will succeed or fail together.

#### Handling Sequential IDs

- DynamoDB doesn't have a built-in way to handle sequential identifiers for new items, but you can make it work through a combination of few building blocks.
- Example -> Let's use a project tracking software like jira. Here, you can create projects, each project has multiple issues.
- Issues are given a sequential number to identify them.
- When a new issue is created within a project, two step process can be followed.
1. Run an UpdateItem operation on the Project item to increment the `IssueCount` attribute by 1. Set the `ReturnValues` parameter to `UPDATED_NEW`, which will return the current value of all updated attributes in the operation. 2. End of this operation, we will know what number should be used for our new issue. We will create our new issue item with the new issue number.

#### Reference Counts
- Use dynamodb transactions for keeping a count of these related items as we go. Keep the below two in transaction
1. Ensure the related item doesn't already exist.
2. Increase the reference count on the parent item.

