### Materialized Views

- Query that gets executed only at very specific times, but the results are saved and can be referenced without rerunning the query.
- Can be used when there's a very expensive query.

Example Query

For each week, show the number of likes that posts and comments received. Use the post and comment created_at date, not when the like is received.

```
CREATE MATERIALIZED VIEW weekly_likes AS (
SELECT 
  date_trunc('week', COALESCE(posts.created_at, comments.created_at)) AS week,
  COUNT(posts.id) AS num_likes_for_posts,
  COUNT(comments.id) AS num_likes_for_comments
FROM likes
LEFT JOIN posts ON posts.id = likes.id
LEFT JOIN comments ON comments.id = likes.comment_id
GROUP BY week
ORDER BY week
) WITH DATA;
```
- With Data means we are telling postgres to hold on to the results.

`SELECT * FROM weekly_likes;` -> would read from materialized view.

- Downside is modifying any underlying tables, as postgres wont update materialized views.

`REFRESH MATERIALIZED VIEW weekly_likes; ` -> to update the DATA holding the result of materialized views.



