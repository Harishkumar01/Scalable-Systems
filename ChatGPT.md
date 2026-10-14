# How OpenAI scaled PostgreSQL to support ChatGPT (As of Jan 2026)

PostgreSQL is one of the OpenAIs core data systems

Architecture involves having primary database (PostgreSQL) and also using shared databases for read heavy queries but the primary database options are maximized and the full specifications are leveraged.

The solution implemented to maximize the performance are as follows,
## Reducing load on Primary
  Read queries can be offloaded to replicas but few read queries are still a part of write queries, In such cases read queries are made sure they are efficient.
## Query Optimization
  Complex join queries are avoided, if needed they are offloaded to the application layer
## Single point of failure mitigation
  To avoid single point of failure, the primary DB is maintained in High availability zones with a hot standby and continuously synchronized replica that can take over serving traffic.
## Workload Isolation
  Low and High priority tasks are run in isolated environments, so if one task becomes CPU intensive it does not affect other tasks
## Connection Pooling
  Use PgBouncer as proxy to pool database connections
## Caching
  When cache miss happens and if a multiple request comes for same key value then every request tries to fetch the value from DB which causes overload. Instead cache locking is implemented where one out of all these requests is reponsible for fetching that data and adding it in cache.
## Scaling read replicas
  Creating replicas involves streaming WAL for each replica. 
  Cascading replication was introduced which allows a standby server to receive the replication connections and stream the WAL acting as relay
## Rate limit
  Implemented across multiple layers to handle sudden spike in traffic
## Schema management
  Even slight schema changes can trigger full database rewrite. So only light weight schema changes are done with caution

This is how the OpenAI handles serving customers with primary DB, write-heavy workloads were moving to shared systems such as Cosmos DB; PostgreSQL sharding was not a near-term priority.
  
<img width="482" height="291" alt="image" src="https://github.com/user-attachments/assets/eb9c8e9f-a3ec-41e2-b6b2-34abc6810ed6" />

