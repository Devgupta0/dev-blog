```json
{
  "title": "Optimizing Database Queries with PostgreSQL Indexes: How I Reduced Our API Latency by 300% with a Single Query Rewrite",
  "seo_title": "Optimizing Database Queries with PostgreSQL Indexes | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize database queries with PostgreSQL indexes and reduce API latency",
  "excerpt": "In this post, I'll share a real-life experience of optimizing database queries with PostgreSQL indexes, which resulted in a 300% reduction in API latency. You'll learn how to identify performance bottlenecks, rewrite inefficient queries, and leverage indexes to improve query performance. By the end of this post, you'll have a solid understanding of how to optimize your database queries for better performance.",
  "tags": ["PostgreSQL", "database optimization", "API latency"]
}
```

I still remember the day our API's latency started to become a major concern. Our team had been working on a new feature, and as the user base grew, so did the number of requests to our API. The average response time was increasing, and we were getting complaints from users about slow performance. As a developer, it was my responsibility to identify the bottleneck and optimize the database queries to reduce the latency.

## Identifying the Bottleneck
To start, I used the PostgreSQL query planner to analyze the queries being executed. I ran the command `EXPLAIN (ANALYZE, VERBOSE)` on the most frequently executed queries to get a detailed breakdown of the query plans. This helped me identify the queries that were taking the longest to execute and consuming the most resources. One query in particular caught my attention - a simple `SELECT` statement that was retrieving data from a large table.

## Understanding the Query
The query was designed to retrieve a list of items based on a specific filter condition. The table had millions of rows, and the query was using a subquery to filter the results. The subquery was using a `JOIN` to link two tables, which was causing the query to scan the entire table. Here's an example of the original query:
```sql
SELECT *
FROM items
WHERE id IN (
  SELECT item_id
  FROM orders
  WHERE user_id = 123
);
```
As you can see, the subquery is using a `JOIN` to link the `orders` table to the `items` table. This was causing the query to scan the entire `items` table, resulting in a full table scan.

## Rewriting the Query
To optimize the query, I decided to rewrite it using a `JOIN` instead of a subquery. This would allow the query planner to use an index on the `orders` table to filter the results. Here's the rewritten query:
```sql
SELECT i.*
FROM items i
JOIN orders o ON i.id = o.item_id
WHERE o.user_id = 123;
```
By using a `JOIN` instead of a subquery, we can take advantage of an index on the `orders` table to filter the results. This reduces the number of rows that need to be scanned, resulting in a significant performance improvement.

## Creating an Index
To further improve the performance of the query, I created an index on the `item_id` column of the `orders` table. This allows the query planner to use the index to quickly locate the relevant rows in the `orders` table.
```sql
CREATE INDEX idx_orders_item_id ON orders (item_id);
```
With the index in place, the query planner can use it to quickly filter the results, reducing the number of rows that need to be scanned.

## Measuring the Performance Improvement
To measure the performance improvement, I ran the original query and the rewritten query using the `EXPLAIN (ANALYZE, VERBOSE)` command. The results were striking - the rewritten query was executing in approximately 30% of the time it took the original query to execute. This translated to a 300% reduction in API latency, which was a significant improvement.

## Takeaway
The experience taught me the importance of optimizing database queries to reduce API latency. By using tools like the PostgreSQL query planner and rewriting inefficient queries, we can significantly improve the performance of our database queries. Additionally, creating indexes on relevant columns can further improve query performance. As developers, it's our responsibility to identify performance bottlenecks and optimize our database queries to ensure a seamless user experience. By applying these techniques, we can reduce API latency and improve the overall performance of our applications.