# System Design – Session 02: Databases & Caching

## 1. SQL vs NoSQL

### What is SQL?

SQL databases are **relational databases** that store data in structured tables with relationships between entities.

Examples:
- PostgreSQL
- MySQL
- Oracle
- SQL Server

Example:

```text
Users
+----+----------+-------------------+
| id | name     | email             |
+----+----------+-------------------+
| 1  | Rahul    | rahul@email.com    |
| 2  | Shubham  | shubham@email.com  |
+----+----------+-------------------+

Orders
+-----+---------+--------+
| id  | user_id | amount |
+-----+---------+--------+
| 101 | 1       | 5000   |
| 102 | 2       | 3000   |
+-----+---------+--------+
```

`user_id` creates a relationship between Users and Orders.

### What is NoSQL?

NoSQL is a broad category of databases designed for different data models and distributed workloads.

Common types:

```text
NoSQL
│
├── Document
│   └── MongoDB
│
├── Key-Value
│   └── DynamoDB / Redis
│
├── Wide Column
│   └── Cassandra
│
└── Graph
    └── Neo4j
```

Example document:

```json
{
  "user_id": 123,
  "name": "Shubham",
  "orders": [
    {
      "order_id": 101,
      "amount": 5000
    }
  ]
}
```

### SQL vs NoSQL: How to decide

Don't use this rule:

```text
SQL = small systems
NoSQL = large systems
```

That's incorrect. Instead, ask:

1. What is the data model?
2. What are the access patterns?
3. Do we need strong transactions?
4. How complex are the queries?
5. What consistency do we need?
6. How much horizontal scaling is required?
7. What is the read/write workload?

**SQL is often a good choice when:**
- Data has strong relationships.
- Transactions are important.
- Complex joins are required.
- Strong consistency is important.
- The schema is relatively structured.
- Reporting and aggregation queries matter.

**NoSQL is often a good choice when:**
- Massive horizontal scaling is required.
- Very high read/write throughput is needed.
- Data structure is flexible.
- Access patterns are predictable.
- Distributed storage is important.
- Complex relational queries aren't the primary requirement.

### Interview example: Banking

Suppose:

```text
Account A → ₹50,000
Account B → ₹20,000
```

We transfer ₹10,000:

```text
Account A → ₹40,000
Account B → ₹30,000
```

We don't want the amount deducted from A without being credited to B. The operation requires strong transactional guarantees.

**Interview answer:**

> I would initially choose a relational SQL database because customers, accounts, and transactions have strong relationships, money transfers require transactional guarantees, and the system may require complex reporting and aggregation queries.

**Important nuance:** Don't say that NoSQL cannot support transactions. Some NoSQL databases support transactions. A stronger argument is that SQL provides a mature relational model and transactional semantics that naturally fit this workload.

---

## 2. Database Indexing

### What is an index?

An index is a **data structure that helps the database find rows faster without scanning the entire table**.

Think of a book index. Without it, you may have to scan every page. With it, you can jump toward the relevant page.

### Example

Suppose the `users` table has 100 million rows.

```sql
SELECT *
FROM users
WHERE email = 'abc@gmail.com';
```

**Without an index:**

```text
Database
   ↓
Scan many rows
   ↓
Find matching email
```

**With an index:**

```text
Email Index
     ↓
abc@gmail.com
     ↓
Row location
```

The database can find the required row much faster.

### Index trade-offs

**Benefit:** Faster reads for queries that can use the index.

**Costs:**
- More storage
- More memory/cache usage
- Slower writes
- Index maintenance overhead

When inserting or updating rows, the database may need to update relevant indexes too. Therefore, don't blindly index every column. Index according to query patterns and verify with query plans.

### Composite indexes

Sometimes queries filter by multiple columns:

```sql
SELECT *
FROM orders
WHERE user_id = 123
  AND status = 'pending';
```

A composite index might be:

```text
(user_id, status)
```

For many B-tree indexes, **column order matters**. This is related to the leftmost-prefix principle.

For an index on `(user_id, status)`, it is naturally useful for queries filtering by:
- `user_id`
- `user_id` and `status`

It may not be useful for a query filtering only by `status`. Actual behavior depends on the database engine, index type, query, and optimizer.

---

## 3. Database Replication

Replication means **keeping copies of database data on multiple database nodes**.

```text
             Primary
                |
        ┌───────┴───────┐
        ↓               ↓
   Read Replica 1   Read Replica 2
```

A common setup sends writes to the primary and distributes some reads across replicas.

### Why replication?

1. **High availability:** a replica may be promoted if the primary fails.
2. **Read scaling:** read workload can be spread across replicas.
3. **Disaster recovery:** additional copies can help recover from some failures.

Replication does not automatically guarantee zero downtime or zero data loss. The outcome depends on replication mode, failover design, and recovery objectives.

### Replication lag

Replication may not be instantaneous.

```text
Primary
Balance = ₹10,000
       |
       | replication delay
       ↓
Replica
Balance = ₹8,000
```

For a short period, a replica may contain older data. This is called **replication lag**.

If a user writes data and immediately reads from a lagging replica, they may see stale data. This is a **read-after-write consistency problem**.

---

## 4. Sharding

Replication means **copying** data. Sharding means **splitting** data across nodes.

Suppose the system has one billion users:

```text
                    1 Billion Users
                          |
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Shard 1         Shard 2         Shard 3
```

Each shard stores only a portion of the dataset.

### Why sharding?

Sharding can help when one database node cannot handle:
- Dataset size
- Write throughput
- Read workload
- Storage requirements

It enables horizontal database scaling, but adds operational and application complexity.

### Shard key

The shard key determines where data is stored.

Example:

```text
user_id
```

A system might use a hash-based approach:

```text
hash(user_id) % number_of_shards
```

Example:

```text
user_id = 12345

hash(12345) % 4
       ↓
Shard 2
```

This is illustrative pseudocode; actual partitioning rules depend on the database.

### Hot-shard problem

Suppose data is sharded by country and most users are from India:

```text
Shard 1 → USA       5%
Shard 2 → Europe   10%
Shard 3 → India    80%  ← overloaded
Shard 4 → Others    5%
```

One shard may receive a disproportionate share of traffic or data. This is called a **hot shard**.

Shard-key selection should consider even distribution, query patterns, write volume, and how easily the system can rebalance data.

### Replication and sharding together

Real systems often combine both:

```text
                 Database Cluster
                        |
          ┌─────────────┴─────────────┐
          ↓                           ↓
       Shard 1                     Shard 2
          |                           |
     ┌────┴────┐                 ┌────┴────┐
     ↓         ↓                 ↓         ↓
  Primary   Replica            Primary   Replica
```

Remember:

```text
Sharding    → Split data
Replication → Copy data
```

They solve different problems and can be used together.

---

## 5. Caching

A cache stores frequently accessed data so it can be retrieved faster and the database receives fewer requests.

**Without cache:**

```text
Client
  ↓
Application
  ↓
Database
```

**With cache:**

```text
Client
  ↓
Application
  ↓
Cache
  ↓
Database (on cache miss)
```

Common technologies include Redis and Memcached.

### Example

Suppose `GET /products/123` is requested 100,000 times.

Without caching, many requests may reach the database. With caching, cache hits can be served without a database query.

Benefits:
- Lower latency
- Lower database load
- Higher throughput

Caching introduces trade-offs, especially stale data, invalidation complexity, and extra infrastructure.

---

## 6. Cache-Aside

Cache-aside is a common caching strategy.

### Read flow

```text
Application
    |
    ↓
 Check Cache
    |
    ├── HIT ──→ Return data
    |
    └── MISS
          ↓
       Database
          ↓
       Store in Cache
          ↓
       Return data
```

Example:

1. Request product 123.
2. Application checks the cache.
3. If the cache contains the product, return it.
4. If not, read it from the database.
5. Store the result in the cache.
6. Return the result.

### Write flow

A common pattern is:

```text
Application
    ↓
Update Database
    ↓
Invalidate Cache
```

The next read rebuilds the cache from the database. Other designs update the cached value directly, depending on consistency needs.

---

## 7. Write-Through Cache

In a write-through design, the application writes through the cache layer, and the backing database is updated as part of the write operation.

```text
Application
     ↓
   Cache
     ↓
  Database
```

**Advantage:** the cache is less likely to remain stale after a successful coordinated write.

**Disadvantage:** writes may take longer because the backing database must also be updated. Exact guarantees depend on the implementation.

---

## 8. Write-Back Cache

In a write-back design, the application writes to the cache first, and the database is updated later, asynchronously.

```text
Application
      ↓
    Cache
      ↓
Later / asynchronously
      ↓
   Database
```

**Advantage:** very fast writes.

**Risk:** if the cache fails before data is safely persisted, data may be lost. Write-back therefore requires careful durability and recovery design.

---

## 9. Cache Strategy Comparison

| Strategy | Write path | Main advantage | Main risk |
|---|---|---|---|
| Cache-aside | App → DB; cache handled separately | Simple and flexible | Stale cache or cache-miss load |
| Write-through | App → cache → DB | Cache updated as part of writes | Write latency |
| Write-back | App → cache; DB updated later | Very fast writes | Data-loss risk if durability is inadequate |

---

## 10. TTL (Time To Live)

TTL means **Time To Live**. It defines how long a cached entry should remain valid before it expires.

Example:

```text
Product 123
TTL = 10 minutes
```

After ten minutes, the entry expires. The next request may cause a cache miss, a database read, and a cache refresh.

### Why use TTL?

Without expiration or invalidation, cached data can remain stale:

```text
Database
Price = ₹50,000

Cache
Price = ₹45,000
```

TTL limits how long an entry remains in the cache without being refreshed, but it does not guarantee that data stays fresh until expiry.

**Important:** TTL is not a complete cache-invalidation strategy. Some data must be invalidated or refreshed immediately after a change.

---

## 11. Cache Invalidation

Cache invalidation means removing or updating cached data when it is no longer valid.

Common strategies:

### Delete on update

```text
Update Database
      ↓
Delete cache entry
```

The next read repopulates the cache.

### Update cache

```text
Update Database
      ↓
Update cache entry
```

This can reduce stale reads, but coordinating both updates correctly is important.

### TTL

Allow the entry to expire automatically. This is simple but may permit stale reads until expiration.

### Event-driven invalidation

```text
Database update / domain event
             ↓
       Event consumer
             ↓
      Invalidate cache
```

Events can make invalidation asynchronous, so a short stale-data window may still exist.

---

## 12. Cache Stampede (Thundering Herd)

A cache stampede occurs when a popular cached item expires and many requests simultaneously try to rebuild it.

Example:

```text
Popular product
100,000 requests/sec
       ↓
Cache expires
       ↓
Many requests miss
       ↓
Database receives a traffic spike
```

This can overload the database.

### Prevention techniques

**1. Locking or request coalescing:** allow one request to rebuild the value while others wait or reuse its result.

**2. Jittered TTL:** vary expiration times so many entries don't expire simultaneously.

**3. Background refresh:** refresh popular entries before they expire.

**4. Stale-while-revalidate:** temporarily serve an older value while refreshing it, when the data's correctness requirements permit this.

---

## 13. CDN (Content Delivery Network)

A CDN stores cacheable content at geographically distributed edge locations, bringing content closer to users.

**Without a CDN:**

```text
User in India
     |
     ↓
Origin server in US
```

**With a CDN:**

```text
User in India
     |
     ↓
Nearby CDN edge
     |
     ↓
Origin only when needed
```

If the content is cached at the edge, it can be served without contacting the origin server.

### Common CDN content

- Images
- CSS
- JavaScript
- Videos
- Fonts
- Downloads
- Cacheable static HTML

Benefits include lower latency, reduced origin traffic, and improved content-delivery scalability.

Cache rules and authorization must be configured carefully. Private or user-specific content must not accidentally be shared across users.

---

## 14. CDN vs Redis

### Redis

Commonly used for:
- Application data
- Sessions
- Frequently accessed database records
- Counters
- Distributed coordination primitives

Example:

```text
API → Redis → Database
```

### CDN

Commonly used for:
- Images
- Videos
- CSS and JavaScript
- Downloads
- Other cacheable content delivered to users

Example:

```text
User → CDN edge → Origin
```

Mental model:

```text
Redis → Make application data faster.
CDN   → Bring cacheable content closer to users.
```

---

## 15. Big-Picture Architecture

```text
                         Users
                           |
                           ↓
                          CDN
                           |
                           ↓
                     Load Balancer
                           |
                           ↓
                     Application
                      /        \
                     /          \
                  Cache        Database
                    |             |
                  Redis       ┌───┴────┐
                              ↓        ↓
                           Primary   Replica
                              |
                         Sharding
```

Each component addresses a different concern:

| Component | Primary purpose |
|---|---|
| CDN | Geographic content delivery |
| Cache | Reduce latency and database load |
| Index | Speed up supported database queries |
| Replication | Maintain copies; can support availability and read scaling |
| Sharding | Split data across nodes for horizontal scaling |

This diagram is conceptual. Real deployments can place components differently and may shard and replicate at the same time.

---

## 16. Choosing the Right Scaling Technique

When a database struggles, don't immediately say, “Use NoSQL.”

First identify the bottleneck:

```text
                    Database problem
                          |
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
       Slow reads     Too many reads    Too much data
          |               |                |
          ↓               ↓                ↓
        Index           Cache           Sharding
```

If the problem is availability or distributing read load, replication may help.

These are starting points, not universal rules. Measure the bottleneck and validate the solution with query plans, metrics, and workload tests.

---

## 17. Interview Cheat Sheet

| Concept | Remember |
|---|---|
| SQL | Relational model, transactions, complex queries |
| NoSQL | Multiple data models; often useful for specific distributed access patterns |
| Index | Faster supported reads, with storage and write costs |
| Replication | Copy data |
| Sharding | Split data |
| Cache | Faster access, lower database load |
| Cache-aside | App checks cache; reads DB on miss |
| Write-through | Update cache and database in the write path |
| Write-back | Cache first; database later |
| TTL | Automatic cache expiration |
| Invalidation | Remove or update stale cache |
| Stampede | Many requests hit the database after a cache miss/expiry |
| CDN | Cacheable content closer to users |

### Four-word mental model

> **Index = Find**  
> **Cache = Remember**  
> **Replication = Copy**  
> **Sharding = Split**

This is a useful starting mental model for system-design interviews.

---

## 18. Current Learning Progress

### Session 02: Databases & Caching

- [x] SQL vs NoSQL
- [x] Database indexing
- [x] Replication
- [x] Sharding
- [x] Caching
- [x] Cache-aside
- [x] Write-through
- [x] Write-back
- [x] TTL
- [x] Cache invalidation
- [x] Cache stampede
- [x] CDN

### Topics still to deepen

- [ ] Redis vs Memcached
- [ ] Advanced indexing and query plans
- [ ] Advanced sharding strategies and rebalancing
- [ ] Cache consistency patterns
- [ ] Active-recall questions and design exercises

## 19. Active Recall Checkpoint

We paused at this question:

> You're building a social-media platform with 500 million users, billions of posts, very high write volume, flexible post metadata, massive horizontal-scaling requirements, and tolerance for temporarily stale data. Complex relational queries are not a major requirement.
>
> Would you choose SQL or NoSQL as the primary database for the posts/feed workload? Give 3–4 requirements that influenced your decision.

The goal is to reason from workload and access patterns, not to assume NoSQL is automatically better just because the system is large.
