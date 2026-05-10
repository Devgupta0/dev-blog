```json
{
  "title": "Optimizing Database Queries with PostgreSQL Indexes and Query Rewriting Saved Our API from a 500ms Latency Bottleneck",
  "seo_title": "Optimizing Database Queries | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize database queries with PostgreSQL indexes and query rewriting to reduce API latency",
  "excerpt": "In this post, I'll share my experience with optimizing database queries to reduce API latency. I'll cover how PostgreSQL indexes and query rewriting can help, and provide a step-by-step guide on how to implement these optimizations. By the end of this post, you'll know how to identify and fix common query performance issues.",
  "tags": ["PostgreSQL", "database optimization", "API latency"]
}
```

I still remember the day our API's latency suddenly spiked to 500ms. It was a typical Monday morning, and our team was busy preparing for a big product launch. Our API was handling a large volume of requests, and everything seemed fine until our monitoring tools started sending out alerts about high latency. We quickly realized that the issue was with our database queries, which were taking too long to execute. After some investigation, we found that the root cause was a poorly optimized query that was causing a bottleneck.

## Identifying the Bottleneck
To identify the bottleneck, we used PostgreSQL's built-in query analysis tools, such as `EXPLAIN` and `EXPLAIN ANALYZE`. These tools provide detailed information about the query execution plan, including the time spent on each step. By analyzing the query plan, we found that the query was using a sequential scan, which was causing the high latency. A sequential scan is a type of scan where the database reads the entire table from disk, row by row, to find the required data. This can be very slow, especially for large tables.

## Creating Indexes
To optimize the query, we created an index on the column used in the `WHERE` clause. Indexes are data structures that allow the database to quickly locate specific data. By creating an index, we could reduce the time spent on the sequential scan. Here's an example of how we created the index:
```sql
CREATE INDEX idx_column_name ON table_name (column_name);
```
In our case, the column name was `user_id`, and the table name was `orders`. So, the index creation query looked like this:
```sql
CREATE INDEX idx_user_id ON orders (user_id);
```
After creating the index, we re-ran the query and checked the execution plan using `EXPLAIN ANALYZE`. The results showed that the query was now using an index scan, which was much faster than the sequential scan.

## Query Rewriting
However, even with the index in place, the query was still taking around 200ms to execute. We realized that the query was doing a lot of unnecessary work, which was causing the high latency. The query was joining two tables, `orders` and `users`, on the `user_id` column. The `users` table was very large, and the join was causing the database to do a lot of extra work. To optimize the query, we rewrote it to use a subquery instead of a join. Here's an example of the original query:
```sql
SELECT * FROM orders
JOIN users ON orders.user_id = users.id
WHERE orders.user_id = 123;
```
And here's the rewritten query:
```sql
SELECT * FROM orders
WHERE user_id = 123 AND user_id IN (SELECT id FROM users);
```
By using a subquery, we could reduce the amount of data that needed to be joined, which improved the query performance.

## Testing and Verification
After rewriting the query, we tested it thoroughly to ensure that it was working correctly. We used a combination of automated tests and manual testing to verify that the query was returning the correct results. We also monitored the query performance using our monitoring tools to ensure that the latency had decreased.

## Takeaway
In this post, we learned how to optimize database queries using PostgreSQL indexes and query rewriting. We saw how creating an index on a column can reduce the time spent on sequential scans, and how rewriting a query can reduce unnecessary work and improve performance. By applying these techniques, we were able to reduce our API's latency from 500ms to around 50ms. The key takeaway is that database query optimization is a crucial step in improving API performance, and by using the right techniques, we can significantly reduce latency and improve user experience.