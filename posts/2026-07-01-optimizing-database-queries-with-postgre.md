```json
{
  "title": "Optimizing Database Queries with PostgreSQL Indexes: A Post-Mortem of a 3000ms Query that Brought Down Our API",
  "seo_title": "Optimizing Database Queries with PostgreSQL Indexes | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize database queries using PostgreSQL indexes and avoid API downtime with this real-world example and step-by-step guide",
  "excerpt": "In this post, I'll share a personal experience of troubleshooting a slow database query that brought down our API, and how I used PostgreSQL indexes to optimize it. You'll learn how to identify slow queries, create effective indexes, and monitor their performance. By the end of this post, you'll be equipped with the knowledge to optimize your own database queries and prevent similar issues.",
  "tags": ["PostgreSQL", "database optimization", "indexing"]
}
```

I still remember the day our API went down due to a single slow database query. It was a typical Monday morning, and our team was busy preparing for a big product launch. Suddenly, our monitoring tools started alerting us about increased latency and error rates. After some frantic debugging, we identified the culprit: a 3000ms query that was blocking all other requests. In this post, I'll share my experience of troubleshooting that query and how I used PostgreSQL indexes to optimize it.

## The Problem
The slow query was a complex JOIN operation between three tables: `orders`, `customers`, and `products`. It was used to retrieve a list of orders for a given customer, along with their corresponding product details. The query looked like this:
```sql
SELECT o.*, c.name, p.description
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE c.id = $1
ORDER BY o.created_at DESC;
```
At first glance, the query seemed innocent enough. However, as the number of orders and customers grew, the query started to take longer and longer to execute. We tried increasing the database resources, but it didn't seem to make a difference. It wasn't until we dug deeper into the query execution plan that we realized the root cause of the problem.

## Understanding the Query Execution Plan
To understand why the query was slow, we needed to analyze its execution plan. We used the `EXPLAIN` command in PostgreSQL to get a detailed breakdown of the query execution plan:
```sql
EXPLAIN (ANALYZE) SELECT o.*, c.name, p.description
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE c.id = $1
ORDER BY o.created_at DESC;
```
The output showed that the query was using a sequential scan on the `orders` table, which was the most expensive operation. The query was also using a nested loop join to combine the `orders` and `customers` tables, which was causing a lot of overhead.

## Creating Effective Indexes
To optimize the query, we needed to create effective indexes on the columns used in the `WHERE` and `JOIN` clauses. We created the following indexes:
```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_orders_product_id ON orders (product_id);
CREATE INDEX idx_customers_id ON customers (id);
CREATE INDEX idx_products_id ON products (id);
```
We also created a composite index on the `orders` table to cover the `customer_id` and `created_at` columns:
```sql
CREATE INDEX idx_orders_customer_id_created_at ON orders (customer_id, created_at);
```
With these indexes in place, we re-ran the query and observed a significant reduction in execution time. The query was now taking around 100ms to execute, which was a huge improvement from the original 3000ms.

## Monitoring Index Performance
To ensure that our indexes were performing well, we used PostgreSQL's built-in monitoring tools to track their usage and performance. We used the `pg_stat_user_indexes` view to get statistics on index usage:
```sql
SELECT indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE schemaname = 'public';
```
This view provided us with information on the number of scans, tuple reads, and tuple fetches for each index. We used this data to identify which indexes were being used the most and which ones were not providing any benefits.

## Takeaway
In conclusion, optimizing database queries with PostgreSQL indexes can be a game-changer for improving performance and preventing downtime. By understanding the query execution plan, creating effective indexes, and monitoring their performance, we can significantly reduce the execution time of slow queries. In our case, creating indexes on the `orders` table reduced the query execution time from 3000ms to 100ms, which had a huge impact on our API's overall performance. As developers, it's essential to be proactive about optimizing our database queries and indexes to ensure that our applications can scale and perform well under heavy loads. By following these best practices, we can avoid similar issues and provide a better experience for our users.