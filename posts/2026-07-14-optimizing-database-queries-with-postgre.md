```json
{
  "title": "Optimizing Database Queries with PostgreSQL Indexes and Partitioning after Migrating from MongoDB to a Relational Database",
  "seo_title": "PostgreSQL Optimization after MongoDB Migration | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize PostgreSQL database queries using indexes and partitioning after migrating from MongoDB, improving query performance and scalability",
  "excerpt": "In this post, I'll share my experience of optimizing database queries with PostgreSQL indexes and partitioning after migrating from MongoDB to a relational database. You'll learn how to identify bottlenecks, create effective indexes, and implement partitioning to improve query performance and scalability.",
  "tags": ["PostgreSQL", "MongoDB", "Database Optimization", "Indexes", "Partitioning"]
}
```

I still remember the day our team decided to migrate our application's database from MongoDB to PostgreSQL. We were excited about the benefits of using a relational database, such as improved data consistency and support for complex transactions. However, we soon realized that our database queries were not optimized for the new database system, resulting in significant performance issues. In this post, I'll share my experience of optimizing database queries with PostgreSQL indexes and partitioning, and how it improved the performance and scalability of our application.

## Identifying Bottlenecks
After migrating to PostgreSQL, we started to notice that some of our database queries were taking an unusually long time to execute. To identify the bottlenecks, we used PostgreSQL's built-in tool, `EXPLAIN`, to analyze the query execution plans. This tool provides detailed information about the query execution, including the time spent on each operation, the number of rows scanned, and the indexes used.

For example, let's consider a simple query that retrieves a list of users with a specific email domain:
```sql
SELECT * FROM users WHERE email LIKE '%@example.com';
```
Running `EXPLAIN` on this query revealed that the database was performing a sequential scan on the entire `users` table, resulting in a significant delay:
```sql
EXPLAIN SELECT * FROM users WHERE email LIKE '%@example.com';
                          QUERY PLAN
---------------------------------------------------------------
 Seq Scan on users  (cost=0.00..10.70 rows=1 width=444)
   Filter: (email ~~ '%@example.com'::text)
```
This plan told us that the database was not using any indexes to filter the results, resulting in a slow query execution.

## Creating Effective Indexes
To improve the query performance, we created an index on the `email` column using the `CREATE INDEX` statement:
```sql
CREATE INDEX idx_users_email ON users (email);
```
This index allowed the database to use a more efficient index scan instead of a sequential scan, reducing the query execution time significantly:
```sql
EXPLAIN SELECT * FROM users WHERE email LIKE '%@example.com';
                          QUERY PLAN
---------------------------------------------------------------
 Bitmap Heap Scan on users  (cost=4.65..8.87 rows=1 width=444)
   Filter: (email ~~ '%@example.com'::text)
   ->  Bitmap Index Scan on idx_users_email  (cost=0.00..4.65 rows=1 width=0)
         Index Cond: (email ~~ '%@example.com'::text)
```
By creating an index on the `email` column, we were able to speed up the query execution by an order of magnitude.

## Implementing Partitioning
Another technique we used to optimize our database queries was partitioning. Partitioning involves dividing a large table into smaller, more manageable pieces based on a specific criteria, such as date or range. This allows the database to focus on a smaller subset of data when executing a query, reducing the overall query execution time.

For example, let's consider a table that stores log data for our application:
```sql
CREATE TABLE logs (
  id SERIAL PRIMARY KEY,
  log_date DATE NOT NULL,
  log_level VARCHAR(10) NOT NULL,
  message TEXT NOT NULL
);
```
To partition this table by date, we can use the `CREATE TABLE` statement with the `PARTITION BY` clause:
```sql
CREATE TABLE logs_part (
  id SERIAL PRIMARY KEY,
  log_date DATE NOT NULL,
  log_level VARCHAR(10) NOT NULL,
  message TEXT NOT NULL
) PARTITION BY RANGE (log_date);
```
We can then create individual partitions for each date range:
```sql
CREATE TABLE logs_part_2022 PARTITION OF logs_part
  FOR VALUES FROM ('2022-01-01') TO ('2022-12-31');

CREATE TABLE logs_part_2023 PARTITION OF logs_part
  FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');
```
By partitioning the `logs` table by date, we can speed up queries that filter by date range, such as:
```sql
SELECT * FROM logs WHERE log_date BETWEEN '2022-01-01' AND '2022-12-31';
```
The database can now focus on the specific partition that contains the relevant data, reducing the query execution time.

## Takeaway
In conclusion, optimizing database queries with PostgreSQL indexes and partitioning was a crucial step in improving the performance and scalability of our application after migrating from MongoDB. By identifying bottlenecks using `EXPLAIN`, creating effective indexes, and implementing partitioning, we were able to significantly reduce query execution times and improve the overall user experience. As a developer, it's essential to understand the strengths and weaknesses of your database system and use the right tools and techniques to optimize your database queries. By doing so, you can unlock the full potential of your database and build faster, more scalable applications.