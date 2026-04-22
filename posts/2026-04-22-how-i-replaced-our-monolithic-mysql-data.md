```json
{
  "title": "How I Replaced Our Monolithic MySQL Database with a Distributed PostgreSQL Cluster Using patroni and pgpool for a 5000-User SaaS Application",
  "seo_title": "Migrating to Distributed PostgreSQL | Dev Notes by Devgupta",
  "seo_description": "Learn how to replace a monolithic MySQL database with a distributed PostgreSQL cluster for a 5000-user SaaS application using patroni and pgpool",
  "excerpt": "In this post, I'll share my experience of migrating our SaaS application's database from a monolithic MySQL setup to a distributed PostgreSQL cluster using patroni and pgpool. I'll cover the challenges I faced, the solutions I found, and the benefits we've seen since the migration. You'll learn how to design and implement a highly available and scalable database cluster for your own application.",
  "tags": ["postgresql", "mysql", "patroni", "pgpool", "database migration"]
}
`

I still remember the day our SaaS application's database crashed due to a sudden surge in traffic. We were using a monolithic MySQL database, and it just couldn't handle the load. Our 5000 users were left frustrated, and our team was under pressure to find a solution. That's when I decided to explore alternative database solutions that could provide high availability, scalability, and performance. After researching and evaluating different options, I chose to migrate our database to a distributed PostgreSQL cluster using patroni and pgpool.

## Why PostgreSQL?
I chose PostgreSQL for several reasons. Firstly, it's an open-source database management system that's highly extensible and customizable. Secondly, it supports a wide range of data types, including JSON, arrays, and user-defined types, which makes it ideal for storing complex data structures. Lastly, PostgreSQL has a strong focus on reliability, data integrity, and concurrency, which are essential for a high-traffic SaaS application like ours.

## Introducing patroni and pgpool
To build a distributed PostgreSQL cluster, I needed a tool that could manage the cluster's topology, handle failovers, and provide a single entry point for client connections. That's where patroni and pgpool come in. Patroni is a PostgreSQL cluster manager that automates the process of setting up and managing a highly available PostgreSQL cluster. It provides features like automatic failover, replication, and cluster topology management. Pgpool is a PostgreSQL clustering tool that provides a single entry point for client connections, load balancing, and connection pooling.

## Designing the Cluster
Before I started building the cluster, I needed to design its topology. I decided to use a combination of patroni and pgpool to create a distributed cluster with three PostgreSQL nodes: one primary node and two standby nodes. The primary node would handle all write operations, while the standby nodes would handle read operations and provide failover capability in case the primary node fails.

Here's an example of how I configured patroni to manage the cluster:
```python
# patroni.yaml
scope: my_cluster
namespace: /db/
name: my_postgres

# PostgreSQL configuration
postgresql:
  listen: 5432
  connect_address: 'localhost:5432'
  data_dir: /var/lib/postgresql/data
  pgpass: /tmp/pgpass
  parameters:
    wal_level: hot_standby
    archive_mode: on
    archive_command: 'cp %p /var/lib/postgresql/archive/%f'

# Patroni configuration
patroni:
  scope: my_cluster
  namespace: /db/
  name: my_postgres
  restapi:
    listen: 8008
    connect_address: 'localhost:8008'
  zookeeper:
    hosts: 'localhost:2181'
    retry: 5
    timeout: 10
```
This configuration tells patroni to manage a PostgreSQL cluster with a primary node and two standby nodes. It also specifies the PostgreSQL configuration, including the listen address, data directory, and replication parameters.

## Building the Cluster
Once I had designed the cluster topology and configured patroni, I started building the cluster. I created three PostgreSQL nodes: one primary node and two standby nodes. I then installed patroni and pgpool on each node and configured them according to my design.

Here's an example of how I configured pgpool to provide a single entry point for client connections:
```sql
-- pgpool.conf
listen_addresses = 'localhost'
port = 5432
backend_hostname0 = 'primary_node'
backend_port0 = 5432
backend_weight0 = 1
backend_hostname1 = 'standby_node1'
backend_port1 = 5432
backend_weight1 = 1
backend_hostname2 = 'standby_node2'
backend_port2 = 5432
backend_weight2 = 1
```
This configuration tells pgpool to listen on port 5432 and distribute incoming connections to the primary node and two standby nodes.

## Migrating the Application
Once the cluster was built, I needed to migrate our SaaS application to use the new distributed PostgreSQL cluster. I updated the application's database configuration to point to the pgpool node, which provides a single entry point for client connections.

I also had to update the application's database queries to use the new PostgreSQL database. This involved changing the database driver, updating the query syntax, and modifying the data types to match the new database schema.

## Testing and Verification
After migrating the application, I thoroughly tested and verified the new database cluster. I simulated various failure scenarios, including node failures and network partitions, to ensure that the cluster could recover automatically and maintain high availability.

I also monitored the cluster's performance and latency to ensure that it could handle the expected load. I used tools like pgbench and PostgreSQL's built-in monitoring tools to measure the cluster's performance and identify any bottlenecks.

## Takeaway
Migrating our SaaS application's database from a monolithic MySQL setup to a distributed PostgreSQL cluster using patroni and pgpool was a challenging but rewarding experience. I learned a lot about database design, cluster management, and high availability. The new cluster has provided significant improvements in performance, scalability, and reliability, and has allowed us to handle a growing user base with confidence. If you're facing similar challenges with your database, I highly recommend exploring distributed database solutions like PostgreSQL, patroni, and pgpool. With careful planning, design, and implementation, you can build a highly available and scalable database cluster that meets your application's needs and provides a great user experience.