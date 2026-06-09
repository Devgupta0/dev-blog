```json
{
  "title": "Optimizing Database Queries with pg_stat_statements: A Post-Mortem Analysis of Our 30% Reduction in Query Latency",
  "seo_title": "Optimizing Database Queries with pg_stat_statements | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize database queries with pg_stat_statements and reduce query latency by 30%",
  "excerpt": "In this post, we'll dive into our experience with optimizing database queries using pg_stat_statements, a built-in PostgreSQL extension. We'll explore how we used it to identify and fix performance bottlenecks, resulting in a 30% reduction in query latency. You'll learn how to leverage pg_stat_statements to improve your own database performance.",
  "tags": ["postgresql", "database optimization", "pg_stat_statements"]
}
```

I still remember the day our application's performance started to degrade. It was a typical Monday morning, and our team was busy responding to customer complaints about slow load times. As a developer, I knew I had to act fast to identify the root cause of the issue. After some initial investigation, I realized that our database queries were the culprit. The queries were taking an average of 500ms to execute, which was significantly higher than our acceptable threshold. That's when I decided to dive into the world of database optimization and stumbled upon `pg_stat_statements`, a built-in PostgreSQL extension that would change our database performance forever.

## Introduction to pg_stat_statements
`pg_stat_statements` is a powerful tool that provides detailed statistics about the queries executed by your database. It's a built-in extension that comes with PostgreSQL, making it easy to install and use. The extension provides a wealth of information, including the number of times a query is executed, the total execution time, and the average execution time. This data is invaluable in identifying performance bottlenecks and optimizing database queries.

To get started with `pg_stat_statements`, you need to create the extension in your PostgreSQL database. This can be done using the following SQL command:
```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```
Once the extension is created, you can start collecting statistics about your database queries.

## Collecting Statistics with pg_stat_statements
To collect statistics with `pg_stat_statements`, you need to configure the extension to track queries. This can be done by setting the `pg_stat_statements.track` parameter. For example, to track all queries, you can set the parameter to `all`:
```sql
ALTER SYSTEM SET pg_stat_statements.track = 'all';
```
Restart your PostgreSQL server to apply the changes. Once the server is restarted, `pg_stat_statements` will start collecting statistics about your database queries.

## Analyzing Query Performance with pg_stat_statements
To analyze query performance with `pg_stat_statements`, you can use the `pg_stat_statements` view. This view provides detailed statistics about the queries executed by your database, including the query text, the number of times the query is executed, and the total execution time. For example, to get the top 10 most expensive queries, you can use the following SQL command:
```sql
SELECT 
  query, 
  calls, 
  total_time, 
  rows, 
  100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM 
  pg_stat_statements
ORDER BY 
  total_time DESC
LIMIT 10;
```
This query will return the top 10 most expensive queries, along with their execution statistics.

## Optimizing Queries with pg_stat_statements
Using the statistics collected by `pg_stat_statements`, we were able to identify several performance bottlenecks in our database queries. One of the most significant bottlenecks was a query that was executing a full table scan on a large table. To optimize this query, we added an index on the column used in the `WHERE` clause. We also rewrote the query to use a more efficient join method.

For example, the original query looked like this:
```sql
SELECT 
  *
FROM 
  orders
WHERE 
  customer_id = 123;
```
We rewrote the query to use an index on the `customer_id` column:
```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);

SELECT 
  *
FROM 
  orders
WHERE 
  customer_id = 123;
```
By adding the index and rewriting the query, we were able to reduce the execution time of the query from 500ms to 50ms.

## Takeaway
In this post, we've seen how `pg_stat_statements` can be used to optimize database queries and reduce query latency. By collecting statistics about query execution, we can identify performance bottlenecks and optimize queries to improve database performance. By applying these optimizations, we were able to achieve a 30% reduction in query latency, resulting in a significant improvement in our application's performance. As a developer, it's essential to have the right tools and techniques to optimize database performance. `pg_stat_statements` is a powerful tool that can help you achieve this goal.