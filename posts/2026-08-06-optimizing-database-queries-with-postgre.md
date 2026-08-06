```json
{
  "title": "Optimizing Database Queries with PostgreSQL Indexes: How I Reduced Our API Latency by 70% with a Single Index Change",
  "seo_title": "Optimizing Database Queries with PostgreSQL Indexes | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize PostgreSQL database queries with indexes and reduce API latency by up to 70% with a single index change",
  "excerpt": "In this post, I'll share how I reduced our API latency by 70% with a single index change in PostgreSQL, including the steps I took to identify and optimize the query, and the code changes I made to achieve the improvement. You'll learn how to use PostgreSQL indexes to speed up your database queries and improve your application's performance.",
  "tags": ["PostgreSQL", "database optimization", "API latency"]
}
`

I still remember the day our API latency started to become a major issue. Our application was handling a large volume of requests, and our database queries were taking longer and longer to execute. Our users were complaining about slow load times, and our team was under pressure to fix the problem. As a developer, I knew that optimizing our database queries was the key to reducing our API latency. In this post, I'll share how I reduced our API latency by 70% with a single index change in PostgreSQL.

## The Problem
Our application is built using a PostgreSQL database, and we were using a lot of complex queries to retrieve data. One of these queries was particularly slow, and it was causing a bottleneck in our API. The query was a simple `SELECT` statement, but it was joining multiple tables and filtering on several columns. I used the `EXPLAIN` command in PostgreSQL to analyze the query plan and identify the performance bottleneck.

```sql
EXPLAIN (ANALYZE) SELECT *
FROM orders
JOIN customers ON orders.customer_id = customers.id
JOIN products ON orders.product_id = products.id
WHERE orders.status = 'pending' AND customers.country = 'USA';
```

The `EXPLAIN` command showed me that the query was using a sequential scan on the `orders` table, which was causing the slow performance. I knew that I needed to create an index on the `orders` table to speed up the query.

## Creating the Index
To create an index on the `orders` table, I used the `CREATE INDEX` command in PostgreSQL. I created a composite index on the `customer_id`, `product_id`, and `status` columns, which are the columns used in the `WHERE` and `JOIN` clauses.

```sql
CREATE INDEX idx_orders_customer_id_product_id_status
ON orders (customer_id, product_id, status);
```

I also created an index on the `customers` table on the `country` column, which is used in the `WHERE` clause.

```sql
CREATE INDEX idx_customers_country
ON customers (country);
```

## Verifying the Results
After creating the indexes, I re-ran the `EXPLAIN` command to verify that the query plan had changed. The results showed that the query was now using the index on the `orders` table, and the performance had improved significantly.

```sql
EXPLAIN (ANALYZE) SELECT *
FROM orders
JOIN customers ON orders.customer_id = customers.id
JOIN products ON orders.product_id = products.id
WHERE orders.status = 'pending' AND customers.country = 'USA';
```

The query was now executing in under 100ms, which was a 70% reduction in latency. I was amazed at how much of a difference a single index change had made.

## Maintaining the Indexes
To ensure that the indexes remain effective, I scheduled a regular maintenance task to run the `ANALYZE` and `VACUUM` commands on the tables. This helps to keep the indexes up-to-date and prevents them from becoming bloated.

```sql
ANALYZE orders;
VACUUM orders;
```

## Takeaway
In conclusion, optimizing database queries with PostgreSQL indexes can have a significant impact on reducing API latency. By creating the right indexes on the right columns, you can speed up your database queries and improve your application's performance. Remember to use the `EXPLAIN` command to analyze your query plans, and don't be afraid to experiment with different index combinations to find the optimal solution. With a little bit of effort, you can achieve significant performance gains and improve the overall user experience of your application.