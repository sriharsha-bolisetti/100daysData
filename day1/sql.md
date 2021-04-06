#### Views
```
SELECT username, COUNT(*)
FROM users
JOIN (
    SELECT user_id FROM photo_tags
    UNION ALL
    SELECT user_id FROM caption_tags
) AS tags ON tags.user_id = users.id
GROUP BY username
ORDER BY COUNT(*) DESC;

```

- We've had to find the union several times
- There's been no benefit to keeping these records in seperate tables
- Guess we had a bad design!
- Two possibities to fix up.

### Solution 1.

- Merge the two tables, delete the orginal ones.

```
CREATE TABLE tags(
    id SERIAL PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    x INTEGER,
    y INTEGER
)
```

Copy all the rows from photo_tags and caption_tags

```
INSERT INTO tags(created_at, updated_at, user_id, post_id, x, y)
SELECT created_at, updated_at, user_id, post_id, x, y
FROM photo_tags;
```


```
INSERT INTO tags(created_at, updated_at, user_id, post_id)
SELECT created_at, updated_at, user_id, post_id
FROM caption_tags;
```

- This might not work all the time
- Can't copy over the ID's of photo_tags and caption_tags since they must be `unique`. -- breaks referential integrity!
- If we delete original tables, we break any existing queries that refer to them! -- costly rewrities.

### Solution 2 

- Create a `view`.
- Create a fake table of sorts that has rows from other tables.
- These can be exact rows as they exist on another table, or a computed value.
- Can reference the view in any place where we'd normally reference a table.
- View doesn't actually create a new table or move any data around.
- Doesn't have to be used for a union! Can compute absolutely any values.


Similar to CTE.

CTE's can be referred to only in the query they are attached to.
Example of CTE

```
WITH tags AS (
    SELECT user_id FROM photo_tags
    UNION ALL
    SELECT user_id FROM caption_tags
)
SELECT username, COUNT(*)
FROM USERS
JOIN tags ON tags.user_id = users.id
GROUP BY username
ORDER BY COUNT(*) DESC;
```


View is taking working table out of the CTE and creating it ahead of the time.

```
CREATE VIEW AS (
    SELECT user_id FROM photo_tags
    UNION ALL
    SELECT user_id FROM caption_tags
)
```

Few more examples

```
CREATE VIEWS recent_posts AS (
    SELECT * 
    FROM posts
    ORDER BY created_at DESC
    LIMIT 10
)
```

```
CREATE OR REPLACE VIEW recent_posts AS (
     SELECT * 
    FROM posts
    ORDER BY created_at DESC
    LIMIT 15
)
```

DROP VIEW recent_posts;
