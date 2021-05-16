- Best way to implement a Top-N query is via `lateral` join.
```
select genre.name as genre,
    case when length(ss.name) > 15
         then substring(ss.name from 1 for 15 ) || '...'
         else ss.name
    end as track,
    artist.name as artist
from genre
    left join lateral (
        select track.name, track.albumid, count(playlistid) 
        from track
            left join playlisttrack using (trackid)
        where track.genreid = genre.genreid
        group by track.trackid
        order by count desc
        limit :n
    )
    ss(name, albumid, count) on true
    join album using (albumid)
    join artist using (artistid)
order by genre.name, ss.count desc;

```
- When using Postgresql, some indexes are necessary to ensure data consistency.
- Constraints such as UNIQUE PRIMARY KEY or EXCLUDE USING are only possible to implement in Postgresql with a backing index.
- Need to know wth is a backing index???
- When an index is used as an implementation detail to ensure data consistency, then the indexing strategy is indeed a data modeling activity.

##### Indexing for Constraints

- When using Postgresql some SQL modeling constraints can only be handled with the help of a backing index.
- That is the case for the primary key and unique constraints, and also for the exclusion constraints created with the Postgresql special syntax `EXCLUDE USING`.
- Postgres provides a rich set of tools for developers to manage concurrent access to data.
- Internally, data consistency is maintained by using a multiversion model. This means that each SQL statement sees a snapshot of data as it was some time ago, regardless of the current state of the underlying data. 
- This prevents statements from viewing inconsistent data produced by concurrent transactions performing updates on the same data rows, providing transaction isolation for each database session. 
- MVCC, by eschewing the locking methodologies of a traditional database systems, minimizes lock contention in order to allow for reasonable performance in multiuser environments.
- So if we think about how to implement the `unique` constraint, we soon realize that to be correct the implementation must prevent two concurrent statements from inserting duplicates.
- Below two transactions t1 and t2 happening in parallel
```
t1> insert into test(id) values(1);
t2> insert into test(id) values(1);
```

- The way the internals of Postgresql solve this problem is by relying on its index data structure in a non-MVCC compliant way, and this capability is not visible to SQL level users.
- So when you declare a unique constraint, a primary key constraint or an exclusion constraint Postgresql creates index for you.

###### some index notes

- GIN or generalized inverted index
GIN is designed for handling cases where the items to be indexed are composite values, and the queries to be handled by the index need to search for element values that appear `within the composite items`.
For example, the items could be documents, and the queries could be searches for documents containing specific words.
An inverted index contains a separate entry for each component value.
Such an index can efficiently handle queries that test for the presence of specific component values.
GIN access method is the foundation for the Postgresql Full Text Search support.
