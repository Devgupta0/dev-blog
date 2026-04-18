```json
{
  "title": "Debugging a 3AM Elasticsearch Cluster Failure that was Caused by a Misconfigured JVM Heap Size and how we Implemented a Custom Monitoring Solution using Prometheus and Grafana to Prevent it from Happening Again",
  "seo_title": "Elasticsearch Cluster Failure | Dev Notes by Devgupta",
  "seo_description": "Learn how to debug Elasticsearch cluster failure caused by misconfigured JVM heap size and implement custom monitoring solution using Prometheus and Grafana",
  "excerpt": "In this post, I'll share my experience of debugging a 3AM Elasticsearch cluster failure, caused by a misconfigured JVM heap size, and how we implemented a custom monitoring solution using Prometheus and Grafana to prevent it from happening again. You'll learn how to identify and fix similar issues in your own Elasticsearch cluster. We'll also cover the implementation of a custom monitoring solution to ensure the health and performance of your cluster.",
  "tags": ["Elasticsearch", "JVM Heap Size", "Prometheus", "Grafana", "Monitoring"]
}
```

I still remember the night I received a frantic call from our operations team at 3AM, informing me that our Elasticsearch cluster had gone down. As the lead developer responsible for the cluster, I knew I had to act fast to resolve the issue and get our services back online. After quickly getting dressed and rushing to my desk, I started investigating the cause of the failure. The first thing I checked was the Elasticsearch logs, which revealed a familiar error message: `java.lang.OutOfMemoryError: Heap space`. It was clear that the JVM heap size was not sufficient to handle the load on the cluster.

## Identifying the Root Cause
To understand what was happening, I needed to dive deeper into the metrics of our cluster. I started by checking the JVM heap usage using the Elasticsearch API. I used the `cluster/nodes/stats` endpoint to fetch the node statistics, which included the JVM heap usage. The response looked something like this:
```json
{
  "_nodes": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "cluster_name": "my-cluster",
  "nodes": {
    "node1": {
      "jvm": {
        "heap_used": "12gb",
        "heap_used_percent": 90
      }
    },
    "node2": {
      "jvm": {
        "heap_used": "10gb",
        "heap_used_percent": 80
      }
    },
    "node3": {
      "jvm": {
        "heap_used": "11gb",
        "heap_used_percent": 85
      }
    }
  }
}
```
As you can see, the heap usage was extremely high, with two nodes using over 80% of their allocated heap space. It was clear that the JVM heap size was not sufficient to handle the load on the cluster.

## Fixing the Issue
To fix the issue, I needed to increase the JVM heap size for each node in the cluster. I updated the `jvm.options` file for each node to include the following configuration:
```properties
-Xms16g
-Xmx16g
```
This set the initial and maximum heap size to 16GB for each node. After updating the configuration, I restarted each node in the cluster to apply the changes.

## Implementing a Custom Monitoring Solution
To prevent similar issues from happening in the future, I decided to implement a custom monitoring solution using Prometheus and Grafana. Prometheus is a popular monitoring system that provides a time-series database to store metrics, while Grafana is a visualization tool that allows you to create custom dashboards to display your metrics.

First, I installed the Prometheus agent on each node in the cluster. This agent collects metrics from the node and sends them to the Prometheus server. I then configured the Prometheus server to scrape the metrics from each node.

Next, I installed Grafana and created a custom dashboard to display the JVM heap usage metrics. I used the `prometheus` data source to fetch the metrics from the Prometheus server. The dashboard included a graph that displayed the heap usage for each node in the cluster, as well as a table that showed the current heap usage and the threshold values.

Here's an example of how I configured the Prometheus agent to collect JVM metrics:
```yml
scrape_configs:
  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['node1:9200', 'node2:9200', 'node3:9200']
    metrics_path: /_cluster/nodes/stats
```
And here's an example of how I configured the Grafana dashboard to display the JVM heap usage metrics:
```json
{
  "rows": [
    {
      "title": "JVM Heap Usage",
      "panels": [
        {
          "id": 1,
          "title": "Heap Usage",
          "type": "graph",
          "span": 6,
          "query": "increase(jvm_heap_used_bytes[1h])",
          "legend": {
            "show": true
          }
        }
      ]
    }
  ]
}
```
With the custom monitoring solution in place, I could now easily monitor the JVM heap usage for each node in the cluster and receive alerts when the usage exceeded a certain threshold.

## Takeaway
In this post, I shared my experience of debugging a 3AM Elasticsearch cluster failure caused by a misconfigured JVM heap size. I also explained how I implemented a custom monitoring solution using Prometheus and Grafana to prevent similar issues from happening in the future. By monitoring the JVM heap usage and receiving alerts when the usage exceeds a certain threshold, I can now ensure the health and performance of my Elasticsearch cluster. If you're experiencing similar issues with your own Elasticsearch cluster, I hope this post has provided you with valuable insights and a solution to prevent future failures. Remember to always monitor your cluster's metrics and adjust your configuration as needed to ensure optimal performance.