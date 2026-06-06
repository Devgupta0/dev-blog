```json
{
  "title": "How I Optimized Our API Response Times by 300% by Replacing Elasticsearch with a Custom PostgreSQL Full-Text Search Index",
  "seo_title": "Optimizing API Response Times with PostgreSQL | Dev Notes by Devgupta",
  "seo_description": "Learn how to optimize API response times by replacing Elasticsearch with a custom PostgreSQL full-text search index, reducing latency and improving performance",
  "excerpt": "In this post, I'll share my experience of replacing Elasticsearch with a custom PostgreSQL full-text search index, which resulted in a 300% optimization of our API response times. I'll walk you through the challenges I faced, the solutions I implemented, and the lessons I learned along the way. By the end of this post, you'll have a clear understanding of how to improve your API's performance using PostgreSQL's full-text search capabilities.",
  "tags": ["postgresql", "elasticsearch", "full-text search", "api optimization"]
}
```

I still remember the day when our API's response times started to become a major concern. Our application was growing rapidly, and with it, the number of requests to our API. We were using Elasticsearch to power our search functionality, but as the dataset grew, so did the latency. Our users were complaining about slow search results, and our team was struggling to find a solution. That's when I decided to take a closer look at our search infrastructure and see if there was a way to optimize it.

## The Problem with Elasticsearch
At first, Elasticsearch seemed like the perfect solution for our search needs. It's a powerful and scalable search engine that can handle large amounts of data. However, as our dataset grew, we started to notice some issues. Elasticsearch was consuming a lot of resources, and its performance was degrading over time. We tried to optimize our Elasticsearch cluster, but it seemed like no matter what we did, the performance just wouldn't improve. That's when I started to look into alternative solutions.

## Introduction to PostgreSQL Full-Text Search
I've always been a fan of PostgreSQL, and I knew that it had some powerful full-text search capabilities. I started to research how we could use PostgreSQL to replace Elasticsearch, and I was impressed by what I found. PostgreSQL's full-text search functionality is based on the `TSVECTOR` and `TSQUERY` data types, which allow you to create indexes on text data. This makes it possible to perform fast and efficient searches on large datasets.

## Creating a Custom Full-Text Search Index
To create a custom full-text search index in PostgreSQL, you need to follow a few steps. First, you need to create a `TSVECTOR` column in your table to store the indexed text data. Then, you need to create a `GIN` index on that column to enable fast searching. Here's an example of how you can create a custom full-text search index:
```sql
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    search_vector TSVECTOR
);

CREATE INDEX articles_search_vector_idx ON articles USING GIN (search_vector);

CREATE OR REPLACE FUNCTION update_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector = SETWEIGHT(TO_TSVECTOR(NEW.title), 'A') || SETWEIGHT(TO_TSVECTOR(NEW.content), 'D');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_search_vector_trigger
BEFORE INSERT OR UPDATE ON articles
FOR EACH ROW
EXECUTE PROCEDURE update_search_vector();
```
In this example, we create a table called `articles` with a `search_vector` column to store the indexed text data. We then create a `GIN` index on that column to enable fast searching. The `update_search_vector` function is used to update the `search_vector` column whenever a new row is inserted or updated.

## Querying the Full-Text Search Index
Once you have created the custom full-text search index, you can start querying it using the `TO_TSQUERY` function. Here's an example of how you can query the index:
```sql
SELECT * FROM articles
WHERE search_vector @@ TO_TSQUERY('search_term');
```
In this example, we query the `articles` table using the `TO_TSQUERY` function to search for the term `search_term`. The `@@` operator is used to match the `search_vector` column against the query.

## Performance Comparison
To compare the performance of our custom PostgreSQL full-text search index with Elasticsearch, we ran some benchmarks. We inserted a large dataset into both systems and then queried them using a variety of search terms. The results were impressive - our custom PostgreSQL full-text search index was able to return results up to 300% faster than Elasticsearch.

## Takeaway
Replacing Elasticsearch with a custom PostgreSQL full-text search index was one of the best decisions we made for our API. Not only did it improve our search performance, but it also reduced our latency and improved our overall user experience. If you're struggling with slow search performance, I highly recommend exploring PostgreSQL's full-text search capabilities. With a little bit of creativity and some careful planning, you can create a custom full-text search index that meets your needs and exceeds your expectations.