```json
{
  "title": "Optimizing Database Queries with pg_stat_statements: A Postmortem of Our Largest Production Bottleneck",
  "seo_title": "Optimizing Database Queries with pg_stat_statements | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize database queries using pg_stat_statements and resolve production bottlenecks with this technical postmortem analysis",
  "excerpt": "In this post, I'll share a personal story of how I identified and resolved our largest production bottleneck using pg_stat_statements. You'll learn how to analyze database query performance, identify bottlenecks, and optimize queries for better performance. By the end of this post, you'll have a clear understanding of how to use pg_stat_statements to optimize your database queries and improve overall system performance.",
  "tags": ["pg_stat_statements", "database optimization", "postgres"]
}
```

I still remember the day our production database started to show signs of strain. Our application, which had been performing smoothly for months, suddenly started to experience intermittent slowdowns and timeouts. As the developer on call, I was tasked with identifying and resolving the issue. After hours of digging through logs and metrics, I finally stumbled upon the culprit: a poorly performing database query that was causing a bottleneck in our system.

## Introduction to pg_stat_statements
As I dug deeper into the issue, I discovered that our PostgreSQL database had a built-in extension called `pg_stat_statements` that could help me analyze query performance. `pg_stat_statements` is a powerful tool that provides detailed statistics about query execution times, including the number of calls, total time spent, and average time per call. By analyzing these statistics, I could identify which queries were causing the bottleneck and optimize them for better performance.

To get started with `pg_stat_statements`, I had to enable it on our production database. This involved adding the following line to our `postgresql.conf` file:
```sql
shared_preload_libraries = 'pg_stat_statements'
```
I then restarted the database server to apply the changes. Once `pg_stat_statements` was enabled, I could view query statistics using the following SQL query:
```sql
SELECT query, calls, total_time, rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_time DESC;
```
This query shows the top queries by total execution time, along with other relevant metrics such as the number of calls, rows returned, and hit percentage.

## Analyzing Query Performance
As I analyzed the query statistics, I noticed that one particular query was dominating the top spot in terms of total execution time. The query was a complex join operation that involved multiple tables and subqueries. I could see that it was being called hundreds of times per minute, with an average execution time of over 100ms.

To optimize this query, I decided to use the `EXPLAIN` command to analyze its execution plan. The `EXPLAIN` command shows the step-by-step process that the database uses to execute a query, including the types of indexes used and the number of rows scanned.
```sql
EXPLAIN (ANALYZE, VERBOSE) SELECT * FROM orders
JOIN customers ON orders.customer_id = customers.id
JOIN products ON orders.product_id = products.id
WHERE orders.total_amount > 100;
```
The execution plan showed that the query was using a sequential scan on the `orders` table, which was causing the high execution time. I realized that adding an index on the `customer_id` and `product_id` columns could significantly improve the query performance.

## Optimizing the Query
I added the following indexes to the `orders` table:
```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_orders_product_id ON orders (product_id);
```
I then re-ran the `EXPLAIN` command to see the updated execution plan. The new plan showed that the query was now using the indexes to perform an index scan, which reduced the execution time by over 90%.

## Monitoring and Verification
After optimizing the query, I monitored the database performance closely to ensure that the changes had the desired effect. I used `pg_stat_statements` to track the query performance and verified that the execution time had decreased significantly.

To ensure that the optimized query continued to perform well over time, I set up a monitoring script that would alert me if the query execution time exceeded a certain threshold. This allowed me to catch any potential issues before they became major problems.

## Takeaway
In this post, I shared my personal experience of using `pg_stat_statements` to identify and optimize a poorly performing database query. By analyzing query statistics and execution plans, I was able to identify the bottleneck and optimize the query for better performance. The key takeaways from this experience are:

* `pg_stat_statements` is a powerful tool for analyzing query performance and identifying bottlenecks.
* Indexing can significantly improve query performance, especially for complex join operations.
* Monitoring and verification are crucial steps in ensuring that optimized queries continue to perform well over time.

By following these best practices and using `pg_stat_statements` to analyze query performance, you can optimize your database queries and improve the overall performance of your system.