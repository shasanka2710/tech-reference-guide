# 🏗️ System Design Quick Reference

## Reference Files

| File | Description |
|---|---|
| [microservices-design-patterns.md](./microservices-design-patterns.md) | All microservices design patterns — decomposition, communication, data, resilience, security, observability, deployment |
| [resilience-patterns.md](./resilience-patterns.md) | Deep-dive on resilience patterns with implementation examples |

## Core Principles

```
Scalability:   handle growing load without performance degradation
Reliability:   correct functioning even with partial failures
Availability:  percentage of time system is operational
Maintainability: ease of modification and operation
```

## Availability Nines

| Availability | Downtime per year | Downtime per month |
|---|---|---|
| 99%       | 3.65 days  | 7.2 hours  |
| 99.9%     | 8.7 hours  | 43.8 min   |
| 99.99%    | 52 min     | 4.4 min    |
| 99.999%   | 5.2 min    | 26 sec     |
| 99.9999%  | 31 sec     | 2.6 sec    |

## Scalability

### Vertical vs Horizontal

```
Vertical (Scale Up):
- Add more CPU/RAM to existing server
- Simpler, no code changes
- Hard limit (max machine size)
- Single point of failure

Horizontal (Scale Out):
- Add more servers
- No upper bound
- Requires load balancing, stateless design
- More complex
```

### Load Balancing

```
Algorithms:
- Round Robin:         requests distributed sequentially
- Weighted Round Robin: based on server capacity
- Least Connections:   send to server with fewest active connections
- IP Hash:             same client always to same server (sticky sessions)
- Random:              random selection

Layers:
- Layer 4 (Transport): based on IP/TCP — fast, less context
- Layer 7 (Application): based on HTTP headers, URL, cookies — more intelligent
```

## Caching

```
Cache-aside (Lazy Loading):
  1. App checks cache
  2. Cache miss -> query DB
  3. Store in cache, return result
  + Only requested data cached
  - Cache miss is slow (3 trips)

Write-through:
  1. Write to cache AND DB synchronously
  + No stale data
  - Write latency, cached data may never be read

Write-behind (Write-back):
  1. Write to cache only
  2. Async flush to DB
  + Low write latency
  - Risk of data loss if cache fails

Read-through:
  - Cache sits in front of DB
  - Cache manages its own population
  + Simpler application code

Cache eviction policies:
- LRU (Least Recently Used)
- LFU (Least Frequently Used)
- FIFO (First In First Out)
- TTL (Time To Live)
```

### CDN (Content Delivery Network)

```
- Distribute static content closer to users
- Cache assets at edge locations globally
- Reduces origin server load
- Types: Pull (on first request) vs Push (upload in advance)
```

## Database Design

### SQL vs NoSQL

```
SQL:
- ACID transactions
- Complex joins
- Structured data, fixed schema
- Best for: financial data, ERP, analytics

NoSQL:
- Horizontal scaling
- Flexible schema
- High throughput at scale
- Types: Document, Key-Value, Wide-Column, Graph
- Best for: social media, catalogs, sessions, IoT
```

### Database Scaling

```
Read Replicas:
- Offload read traffic from primary
- Asynchronous replication
- Use for: analytics, reporting, read-heavy workloads

Sharding (Horizontal Partitioning):
- Split data across multiple DB instances
- Shard key determines which shard
- Challenges: cross-shard queries, rebalancing, hot spots

Vertical Partitioning:
- Split table columns across separate tables/DBs
- Move infrequently accessed columns to separate store

Denormalization:
- Add redundant data to avoid expensive joins
- Trade write complexity for read speed
```

### CAP Theorem

```
For a distributed system, you can only guarantee 2 of 3:
- Consistency (C):   all nodes see the same data at same time
- Availability (A):  every request gets a response
- Partition Tolerance (P): system works despite network partitions

Network partitions are unavoidable, so choose:
- CP: consistent but may reject requests (HBase, MongoDB in strong mode, ZooKeeper)
- AP: available but may return stale data (Cassandra, CouchDB, DynamoDB)
```

### ACID vs BASE

```
ACID (SQL):
- Atomicity:   all or nothing
- Consistency: data always valid
- Isolation:   transactions don't interfere
- Durability:  committed data persists

BASE (NoSQL):
- Basically Available: system is always available
- Soft state:          state can change without input (eventual consistency)
- Eventually consistent: all nodes will eventually agree
```

## Communication Patterns

### Synchronous vs Asynchronous

```
Synchronous (REST, gRPC):
+ Simple, immediate response
+ Easy debugging
- Caller blocks waiting for response
- Tight coupling

Asynchronous (message queues):
+ Decoupled services
+ Better throughput, handle bursts
+ Resilient to failures
- Eventual consistency
- Harder to debug
- Duplicate message handling needed
```

### API Design

```
REST:
- Resources identified by URLs
- HTTP verbs (GET, POST, PUT, PATCH, DELETE)
- Stateless
- Responses: JSON / XML
- Idempotent: GET, PUT, DELETE
- Non-idempotent: POST

gRPC:
- Protocol Buffers (binary, smaller)
- Bidirectional streaming
- Strongly typed contracts
- HTTP/2

GraphQL:
- Single endpoint
- Client specifies exact data shape
- Avoids over-fetching and under-fetching
- Good for complex, nested data

WebSockets:
- Full-duplex persistent connection
- Real-time bidirectional communication
- Use for: chat, live feeds, gaming, collaboration
```

## Rate Limiting

```
Algorithms:
- Token Bucket:     tokens replenished at fixed rate; allow bursts
- Leaky Bucket:     requests processed at fixed rate; smooths bursts
- Fixed Window:     count requests in fixed time window (edge case at boundary)
- Sliding Window Log:   track timestamps; accurate but memory-intensive
- Sliding Window Counter: hybrid; efficient and accurate

Implementation:
- In-memory (per instance): fast but no global limit
- Redis: shared global counter across all instances
```

## Consistent Hashing

```
Problem: adding/removing servers in hash ring causes mass reassignment
Solution: each server occupies multiple points on a virtual ring
          -> only k/n keys reassigned when adding server (k=keys, n=servers)

Used in: distributed caches, distributed databases, load balancers
Examples: Cassandra, DynamoDB, Chord DHT
```

## Common System Design Patterns

### Fan-out

```
One event triggers multiple downstream actions
Example: User posts tweet -> fan out to followers' home timelines
Options:
- Fan-out on write (push): write to all followers at write time
  + Fast reads
  - Slow writes, celebrity problem
- Fan-out on read (pull): compute timeline at read time
  + Fast writes
  - Slow reads
- Hybrid: push for normal users, pull for celebrities
```

### Saga Pattern (Distributed Transactions)

```
- Break transaction into sequence of local transactions
- Each step publishes event or sends message to trigger next step
- On failure: compensating transactions rollback completed steps

Choreography: services react to events (decentralized)
Orchestration: central coordinator directs steps
```

### CQRS (Command Query Responsibility Segregation)

```
- Separate read (query) and write (command) models
- Write model: optimized for consistency, domain logic
- Read model:  denormalized, optimized for queries

Benefits: scale reads and writes independently
Drawback: eventual consistency between write and read models
```

### Event Sourcing

```
- Store sequence of events instead of current state
- State is derived by replaying events
- Complete audit log, easy temporal queries
- Complex implementation, storage grows over time
```

## Back-of-the-Envelope Estimation

```
Powers of 2:
  2^10 = 1 KB (thousand)
  2^20 = 1 MB (million)
  2^30 = 1 GB (billion)
  2^40 = 1 TB (trillion)

Latency numbers (approximate):
  L1 cache:           0.5 ns
  L2 cache:           7 ns
  Main memory:        100 ns
  SSD read:           150 µs
  HDD seek:           10 ms
  Network RTT (same datacenter): 0.5 ms
  Network RTT (cross-region):    150 ms

Traffic estimation:
  1M DAU * 10 requests/day = 10M requests/day
  10M / 86,400 sec ≈ 115 RPS
  Peak = 2-3x average

Storage estimation:
  1M users * 1 KB metadata = 1 GB
  1B tweets * 280 chars = 280 GB text/day
```

## Design Interview Framework

```
1. Clarify requirements (5 min)
   - Functional requirements (what the system does)
   - Non-functional requirements (scale, latency, availability)
   - Constraints (team size, timeline, existing infra)

2. Capacity estimation (5 min)
   - Users (DAU, MAU)
   - Read/write ratio
   - QPS, peak QPS
   - Storage (data size, retention, growth)

3. High-level design (10-15 min)
   - Major components and data flow
   - API design
   - Data model

4. Deep dive (15-20 min)
   - Scale bottlenecks and solutions
   - Trade-offs
   - Edge cases

5. Wrap up (5 min)
   - Summary
   - Monitoring and alerting
   - Future improvements
```
