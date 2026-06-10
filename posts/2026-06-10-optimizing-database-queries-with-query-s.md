```json
{
  "title": "Optimizing Database Queries with Query Store in PostgreSQL After a Mysterious 30% CPU Spike on Our Production Servers",
  "seo_title": "Optimizing Database Queries | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize PostgreSQL database queries using Query Store after a sudden 30% CPU spike on production servers",
  "excerpt": "In this post, I'll share my experience with a mysterious 30% CPU spike on our production servers, and how I used PostgreSQL's Query Store to identify and optimize the problematic database queries. You'll learn how to leverage Query Store to improve your database performance and prevent similar issues. I'll also provide a step-by-step guide on how to set up and use Query Store to optimize your database queries.",
  "tags": ["PostgreSQL", "Database Optimization", "Query Store"]
}
```

I still remember the day when our production servers suddenly started consuming 30% more CPU than usual. It was a typical Monday morning, and our team was busy with the weekly planning and meetings. Our monitoring system started alerting us about the increased CPU usage, and we quickly jumped into action to investigate the issue. After initial analysis, we realized that the increased CPU usage was coming from our PostgreSQL database servers. We were using PostgreSQL as our primary database for our web application, and any performance issue with the database could directly impact our users.

## Initial Investigation
We started by checking the database logs, system metrics, and application logs to see if there were any errors or anomalies that could explain the increased CPU usage. However, we couldn't find any obvious issues. The database was still serving requests, and our application was functioning normally, but the CPU usage was consistently higher than usual. We decided to dig deeper and use PostgreSQL's built-in tools to analyze the database performance.

## Introduction to Query Store
That's when I remembered about Query Store, a relatively new feature in PostgreSQL that allows you to store and analyze query execution metrics. Query Store provides a centralized repository for query metrics, making it easier to identify and optimize problematic queries. I had heard about Query Store during a conference, but I hadn't had a chance to try it out until now.

## Setting Up Query Store
To set up Query Store, you need to enable it on your PostgreSQL server. You can do this by setting the `pg_qs.query_store_enabled` parameter to `on` in your `postgresql.conf` file. You'll also need to create a schema and tables to store the query metrics. You can use the following SQL script to create the necessary tables:
```sql
CREATE SCHEMA IF NOT EXISTS pg_qs;
SET search_path TO pg_qs;

CREATE TABLE IF NOT EXISTS query_store (
    id SERIAL PRIMARY KEY,
    query_id INTEGER,
    query_text TEXT,
    execution_time DOUBLE PRECISION,
    calls INTEGER,
    rows INTEGER,
    shared_blks_hit INTEGER,
    shared_blks_read INTEGER,
    shared_blks_written INTEGER,
    local_blks_hit INTEGER,
    local_blks_read INTEGER,
    local_blks_written INTEGER,
    temp_blks_read INTEGER,
    temp_blks_written INTEGER,
    blk_read_time DOUBLE PRECISION,
    blk_write_time DOUBLE PRECISION
);

CREATE INDEX IF NOT EXISTS query_store_query_id_idx ON query_store (query_id);
```
Once you've set up Query Store, you can start collecting query metrics by running the following command:
```sql
SELECT pg_qs.start_query_collection();
```
This will start collecting query metrics for all queries executed on your database.

## Analyzing Query Metrics
After collecting query metrics for a few hours, we started analyzing the data to identify the problematic queries. We used the following SQL query to get the top 10 queries by execution time:
```sql
SELECT 
    query_text, 
    execution_time, 
    calls, 
    rows, 
    shared_blks_hit, 
    shared_blks_read, 
    shared_blks_written
FROM 
    pg_qs.query_store
ORDER BY 
    execution_time DESC
LIMIT 10;
```
This query showed us the top 10 queries that were consuming the most CPU time. We were surprised to see that a simple query that was supposed to return a small amount of data was actually consuming a lot of CPU time due to a missing index.

## Optimizing Problematic Queries
We started optimizing the problematic queries by adding missing indexes, rewriting queries to use more efficient join algorithms, and optimizing database configuration parameters. We also used Query Store to monitor the performance of the optimized queries and make further adjustments as needed.

## Takeaway
In conclusion, Query Store is a powerful tool for optimizing database queries in PostgreSQL. By leveraging Query Store, we were able to identify and optimize the problematic queries that were causing the 30% CPU spike on our production servers. I learned that Query Store is not just a debugging tool, but also a proactive monitoring tool that can help you identify potential issues before they become critical. If you're using PostgreSQL as your primary database, I highly recommend setting up Query Store and using it to monitor and optimize your database queries. With Query Store, you can ensure that your database is running at optimal performance, and you can prevent similar issues in the future.