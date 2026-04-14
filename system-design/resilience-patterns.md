# 🛡️ Resilience Patterns for Microservices

Resilience is the ability of a system to recover from failures and continue operating. In a microservices architecture, where dozens of services communicate over a network, failures are not exceptional — they are inevitable. Applying the right resilience patterns prevents a single failure from cascading into a system-wide outage.

---

## Table of Contents

1. [Circuit Breaker](#1-circuit-breaker)
2. [Retry with Exponential Backoff](#2-retry-with-exponential-backoff)
3. [Bulkhead](#3-bulkhead)
4. [Timeout](#4-timeout)
5. [Rate Limiter / Throttling](#5-rate-limiter--throttling)
6. [Fallback (Graceful Degradation)](#6-fallback-graceful-degradation)
7. [Health Check & Readiness Probe](#7-health-check--readiness-probe)
8. [Idempotency](#8-idempotency)
9. [Dead Letter Queue (DLQ)](#9-dead-letter-queue-dlq)
10. [Saga Pattern (Distributed Resilience)](#10-saga-pattern-distributed-resilience)
11. [Backpressure](#11-backpressure)
12. [Service Mesh & Sidecar Proxy](#12-service-mesh--sidecar-proxy)

---

## 1. Circuit Breaker

### Description

Wraps a remote call with a state machine that monitors for failures. When failures exceed a threshold, the circuit "opens" and subsequent calls fail immediately without hitting the downstream service. After a cooldown period, a limited number of requests are allowed through (half-open state) to check if the service has recovered.

```
States:
  CLOSED   → normal operation; calls pass through
  OPEN     → service is failing; calls fail fast (no downstream hit)
  HALF-OPEN → probe state; limited calls allowed to test recovery
```

### How It Helps

- Prevents cascading failures across service chains
- Fails fast instead of blocking threads waiting for a timeout
- Gives failing downstream services time to recover without being flooded
- Reduces resource exhaustion (thread pools, connections) on the caller side

### Pitfalls if Not Applied

**Enterprise Example — Amazon (2021-style):**  
Without a circuit breaker, a slow payment gateway service causes all checkout threads in the Order service to block waiting for responses. Thread pools exhaust, the Order service becomes unresponsive, which causes the API Gateway to queue requests, which exhausts its memory — a full cascade failure. The entire e-commerce platform goes down because of one slow downstream dependency.

**Common mistakes:**
- Threshold too low → circuit opens on transient blips, causing unnecessary failures
- Threshold too high → circuit never opens, allowing cascade to happen anyway
- Not monitoring circuit state → ops teams are blind to upstream degradation
- Sharing one circuit breaker across different endpoints of the same service → one bad endpoint masks a healthy one

### Real-World Tools

`Resilience4j` (Java), `Hystrix` (deprecated, Netflix), `Polly` (.NET), Istio/Envoy (service mesh level)

---

## 2. Retry with Exponential Backoff

### Description

Automatically retries a failed operation a limited number of times, with an increasing wait interval between attempts (exponential backoff), often combined with jitter (randomized delay) to avoid synchronized retry storms.

```
Attempt 1: immediate
Attempt 2: wait 1s  (+ random jitter 0–500ms)
Attempt 3: wait 2s  (+ random jitter)
Attempt 4: wait 4s  (+ random jitter)
→ give up after N attempts
```

### How It Helps

- Handles transient faults: network blips, brief unavailability, GC pauses
- Exponential backoff reduces load on a struggling service
- Jitter prevents the "thundering herd" problem where all clients retry simultaneously

### Pitfalls if Not Applied

**Enterprise Example — Stripe:**  
A payment processing service retries a charge immediately and repeatedly on network timeout. If the original request actually succeeded but the response was lost, the customer is charged twice. Without idempotency keys AND bounded retries, uncontrolled retries produce duplicate charges, data corruption, and customer complaints at scale.

**Enterprise Example — Cloud batch job:**  
A data pipeline retries on every failure without backoff. When a downstream database is overloaded, 500 worker threads all retry every 100ms, generating 5,000 RPS against a database that was already struggling at 200 RPS — making recovery impossible.

**Common mistakes:**
- Retrying non-idempotent operations (POST, payment debits) without idempotency keys
- Infinite or unbounded retries
- No jitter → thundering herd on recovery
- Retrying on non-transient errors (4xx client errors) → wasted retries, hides bugs

---

## 3. Bulkhead

### Description

Isolates resources (thread pools, connection pools, semaphores) for different services or use cases so that failures in one part of the system cannot consume all shared resources and starve other parts.

Named after ship bulkheads that compartmentalize the hull — a breach in one compartment does not sink the ship.

```
Without Bulkhead:          With Bulkhead:
┌─────────────────┐        ┌──────────┬──────────┬──────────┐
│  Shared Thread  │        │ Pool A   │ Pool B   │ Pool C   │
│  Pool (100)     │        │ (30)     │ (40)     │ (30)     │
│                 │        │ Service A│ Service B│ Service C│
│ Service A (100) │        │          │          │          │
│ Service B ( 0 ) │        │  FAIL    │  OK      │  OK      │
└─────────────────┘        └──────────┴──────────┴──────────┘
   B starved by A           A fails, B and C unaffected
```

### How It Helps

- Prevents one slow/failing downstream service from consuming all threads
- Protects high-priority traffic from being starved by low-priority traffic
- Limits the blast radius of a failure to one isolated resource pool

### Pitfalls if Not Applied

**Enterprise Example — Netflix:**  
Netflix's API layer calls dozens of microservices (recommendations, user profiles, playback rights). Without bulkheads, a spike in slow recommendation calls consumes all 200 shared HTTP threads. Profile lookups and playback authorization queue up behind them. Users cannot log in or play any video — caused by the recommendations service, which is non-critical.

**Common mistakes:**
- Setting pool sizes too small → over-isolation kills legitimate throughput
- Setting pool sizes too large → no real isolation benefit
- Not tuning pool sizes per SLA tier (critical vs. non-critical traffic)
- Applying bulkheads only at the thread level but not at the connection pool level

---

## 4. Timeout

### Description

Sets a maximum duration for an operation (network call, database query, inter-service request). If the operation does not complete within the limit, it is aborted and treated as a failure.

```
Client ─── request ──→ Service
       ←── (no response after N ms)
       ←── TimeoutException (fail fast)
```

### How It Helps

- Prevents threads from blocking indefinitely on a hung downstream service
- Bounds worst-case latency for end users
- Works hand-in-hand with Circuit Breaker (timeouts feed failure counts)

### Pitfalls if Not Applied

**Enterprise Example — Uber:**  
The driver matching service calls a mapping service to compute ETAs. The mapping service hangs due to a database lock. Without a timeout, all driver-matching threads block indefinitely. Within seconds, the driver matching service's thread pool is exhausted. Riders requesting rides see infinite loading spinners. The entire matching platform stalls from one locked database query.

**Enterprise Example — GitHub:**  
An internal git operation times out after 30s at the application layer but the database query behind it has no timeout. The database continues running the expensive query for 10 minutes, consuming I/O and blocking other queries, even though the application already gave up and returned an error.

**Common mistakes:**
- Setting timeouts too high → threads still block too long, cascade still happens slowly
- Setting timeouts too low → high false-positive failure rate for slow-but-valid operations
- Not setting timeouts at ALL layers (HTTP client, connection pool, database query)
- Not propagating deadlines across service calls (a 500ms client deadline must flow through to all downstream calls)

---

## 5. Rate Limiter / Throttling

### Description

Limits the number of requests a service accepts from a caller (or in total) over a time window. Excess requests are rejected with a `429 Too Many Requests` response or queued.

```
Algorithms:
  Token Bucket:    smooth bursts; allows burst up to bucket size
  Leaky Bucket:    enforces strict rate; queues excess
  Sliding Window:  precise tracking per rolling time window
  Fixed Window:    simple; edge effects at window boundary
```

### How It Helps

- Protects a service from being overwhelmed by a single misbehaving client
- Prevents runaway retry storms from flooding a recovering service
- Enforces fair usage across tenants in multi-tenant systems
- Gives operators a safety valve during traffic spikes

### Pitfalls if Not Applied

**Enterprise Example — Twilio:**  
A customer's application has a bug that causes it to send 50,000 SMS API calls per second instead of 50. Without rate limiting, Twilio's SMS dispatch workers are flooded, impacting all other customers on the platform. A single misbehaving tenant causes a multi-tenant outage.

**Enterprise Example — AWS API Gateway:**  
An internal service polling a configuration API in a tight loop at 10,000 RPS brings down the configuration service for all microservices in the cluster — causing config-refresh failures and stale configuration propagation across the entire platform.

**Common mistakes:**
- Rate limiting only at the edge (API gateway) but not at the service-to-service level
- Not returning `Retry-After` headers — clients don't know when to retry
- Global rate limits without per-client limits → one noisy tenant starves others
- Rate limiting without monitoring → silent throttling hides bugs in callers

---

## 6. Fallback (Graceful Degradation)

### Description

When a service call fails (after retries / circuit opens), instead of propagating an error, return a pre-defined fallback response — a cached value, a default, or a simplified response — so the user experience degrades gracefully rather than failing completely.

```
Primary Call → FAILS
     ↓
Fallback:
  - Return last known cached value
  - Return a sensible default
  - Serve a static/simplified response
  - Return empty/partial data with a flag
```

### How It Helps

- Keeps the user-facing experience functional even during partial outages
- Decouples non-critical feature availability from core flows
- Reduces the perceived impact of backend failures on end users

### Pitfalls if Not Applied

**Enterprise Example — Netflix:**  
The personalized recommendations service is down. Without a fallback, the Netflix home page returns a 500 error and users see a blank screen. With a fallback, the home page returns a static list of trending content. The experience degrades gracefully — users can still browse and watch, just without personalized recommendations.

**Enterprise Example — Banking app:**  
A credit score enrichment service is unavailable during loan application processing. Without a fallback, the entire loan application flow fails with a 500 error. With a fallback, the application proceeds and the underwriter is flagged to manually verify the credit score. Revenue is protected.

**Common mistakes:**
- Returning stale fallback data that is no longer safe (e.g., stale pricing)
- Falling back silently without alerting ops → root cause is never investigated
- Using fallback as a substitute for fixing the real failure
- Falling back on ALL errors including business logic errors (4xx) that should surface to the user

---

## 7. Health Check & Readiness Probe

### Description

Services expose `/health` (liveness) and `/ready` (readiness) endpoints. Orchestrators (Kubernetes) and load balancers poll these to decide whether to route traffic to an instance.

```
Liveness Probe:   Is the process alive? (restart if not)
Readiness Probe:  Is the service ready to accept traffic?
                  (remove from LB pool if not)
Startup Probe:    Has the service finished initializing?
```

### How It Helps

- Kubernetes automatically restarts crashed or deadlocked pods
- Load balancers stop routing traffic to instances that are up but not ready (warming caches, DB reconnecting)
- Startup probes prevent premature traffic routing to slow-starting services
- Enables zero-downtime rolling deployments

### Pitfalls if Not Applied

**Enterprise Example — Kubernetes cluster:**  
A Java microservice takes 45 seconds to warm up (loading caches, establishing DB connection pools). Without a startup/readiness probe, Kubernetes marks the pod as ready immediately on process start. Traffic hits the pod before it is ready, causing a flood of 503 errors during every deployment — users experience errors on every release.

**Enterprise Example — Payment service:**  
A service's liveness probe checks only the HTTP port (process is alive) but not its database connection. The database connection pool silently exhausts. The pod passes liveness checks and stays in the load balancer pool, but all requests fail because the DB connection pool is empty. The platform serves errors for 20 minutes before ops manually investigates.

**Common mistakes:**
- Liveness probe that checks upstream dependencies → a failing upstream causes cascading pod restarts across the cluster
- Readiness probe that is too shallow (only checks if HTTP port is open)
- Not setting appropriate `initialDelaySeconds` → race condition on startup
- No probes at all → crashed pods receive traffic, failed deployments go undetected

---

## 8. Idempotency

### Description

Designing operations so that performing them multiple times has the same effect as performing them once. Achieved via idempotency keys (client-generated unique IDs per request) that the server uses to deduplicate.

```
Client sends:  POST /payments  { idempotencyKey: "abc-123", amount: 100 }
Server:
  - First call:  process payment, store result against "abc-123"
  - Second call: detect "abc-123" seen before, return stored result (no re-processing)
```

### How It Helps

- Enables safe retries on non-idempotent operations (payments, order creation)
- Protects against duplicate processing due to network retries, at-least-once message delivery
- Essential for event-driven architectures where message brokers may redeliver events

### Pitfalls if Not Applied

**Enterprise Example — Stripe/PayPal:**  
A payment API call times out on the client side. The client retries the charge without an idempotency key. The first request already succeeded server-side, so the customer is charged twice. At scale with millions of transactions, double-charges occur regularly, generating chargebacks, regulatory risk, and customer trust damage.

**Enterprise Example — Kafka consumer:**  
An order fulfilment consumer processes a "ship order" message. The consumer crashes after sending the shipping request but before committing the Kafka offset. On restart, Kafka redelivers the message. Without idempotency, the order ships twice.

**Common mistakes:**
- Idempotency keys with too short a TTL → a retry after key expiry re-processes
- Idempotency key stored in a non-durable store (in-memory) → lost on restart, retries duplicate
- Not propagating idempotency keys to downstream service calls within a transaction

---

## 9. Dead Letter Queue (DLQ)

### Description

A message queue where messages that cannot be successfully processed (after N retries) are moved, rather than being dropped or blocking the main queue.

```
Main Queue ─→ Consumer (fails 3x) ─→ DLQ
                                         ↑
                               ops team investigates,
                               replays after fix
```

### How It Helps

- Prevents a single poison pill message from blocking the entire queue
- Preserves failed messages for diagnosis and replay
- Decouples error handling from the happy path
- Enables partial failure recovery without data loss

### Pitfalls if Not Applied

**Enterprise Example — Financial messaging (SWIFT/FIX):**  
An order management system receives a malformed trade instruction message. Without a DLQ, the consumer retries indefinitely, blocking all subsequent trade messages. The trading desk's entire order flow halts until an engineer manually deletes the message. Millions of dollars in trades are delayed.

**Enterprise Example — E-commerce order events:**  
An "order placed" event has a schema incompatibility due to a producer deployment. Without a DLQ, the consumer keeps failing and redelivering, generating millions of retry attempts that overload the broker and cause consumer lag to grow across all topics.

**Common mistakes:**
- DLQ exists but nobody monitors or alerts on it → messages silently pile up for days
- No replay mechanism → DLQ becomes a graveyard, failed messages are never recovered
- DLQ retry loop: DLQ consumer also fails and messages loop back to DLQ infinitely
- Not including enough metadata (stack trace, attempt count, original timestamp) in DLQ messages to diagnose the root cause

---

## 10. Saga Pattern (Distributed Resilience)

### Description

Manages distributed transactions across multiple microservices by breaking them into a sequence of local transactions. Each step either succeeds and triggers the next step, or fails and triggers compensating (rollback) transactions.

```
Choreography (event-driven):
  Order Service → emits OrderCreated
    → Payment Service processes payment → emits PaymentProcessed
      → Inventory Service reserves items → emits ItemsReserved
        → Shipping Service creates shipment

  On failure at any step → emit failure event → preceding services run compensation
```

### How It Helps

- Achieves data consistency across service boundaries without distributed locks or 2PC
- Each service stays autonomous with its own local transaction
- Failures are handled explicitly via compensating actions rather than silent data inconsistency

### Pitfalls if Not Applied

**Enterprise Example — Airline booking:**  
Without the saga pattern, a flight booking system uses a distributed transaction across the reservations DB, payment DB, and loyalty points DB. A network partition during 2-phase commit leaves all three databases in an uncertain lock state. The system deadlocks — no bookings can be made or released until a DBA manually intervenes.

**Enterprise Example — Ride-sharing:**  
A ride booking tries to charge the rider and assign the driver atomically. Without a saga, a partial failure leaves the rider charged but no driver assigned. The company refunds manually at scale — expensive and operationally unsustainable.

**Common mistakes:**
- No compensation logic → partial failures leave data in an inconsistent state permanently
- Compensation is not idempotent → double-compensation creates more inconsistency
- Long saga chains → higher probability of failure midway and complex rollback
- No observability into saga state → impossible to diagnose stuck or partially-completed sagas

---

## 11. Backpressure

### Description

A mechanism for a downstream service to signal to an upstream producer that it is overwhelmed and the producer should slow down or stop sending. Prevents unbounded queue growth and memory exhaustion.

```
Producer ──→ Queue (growing) ──→ Consumer (slow)
                ↑
        Consumer signals: "slow down"
                ↓
        Producer reduces send rate or blocks
```

### How It Helps

- Prevents unbounded memory growth (queue fills up, OOM crashes)
- Allows the system to find a natural throughput equilibrium
- Propagates capacity constraints upstream instead of silently dropping data
- Fundamental to reactive systems (Reactive Streams / Project Reactor / RxJava)

### Pitfalls if Not Applied

**Enterprise Example — Log aggregation pipeline:**  
A log ingestion service (Logstash/Fluentd) consumes logs from thousands of microservices. A downstream Elasticsearch cluster slows due to indexing backlog. Without backpressure, the ingestion queue grows unboundedly. Within minutes, the ingestion service runs out of heap memory, crashes, and log data from all services is lost — destroying audit trails and incident investigation capability.

**Enterprise Example — Kafka consumer:**  
A consumer reads Kafka messages faster than it can process them and buffers them in memory. When the downstream DB is slow, the in-memory buffer grows until the consumer pod OOMs. Unprocessed messages are lost (if not properly offset-managed), causing data gaps in analytics pipelines.

**Common mistakes:**
- Using unbounded in-memory queues → OOM under load
- Dropping messages silently instead of propagating pressure upstream
- Not monitoring queue depth — unbounded growth goes unnoticed until crash
- Ignoring reactive streams contracts (not respecting demand signals in Project Reactor/RxJava)

---

## 12. Service Mesh & Sidecar Proxy

### Description

Offloads resilience concerns (retries, timeouts, circuit breaking, mutual TLS, observability) from the application code into a sidecar proxy (e.g., Envoy) deployed alongside each service instance. A control plane (e.g., Istio) manages policies centrally.

```
Service A Pod:                Service B Pod:
┌──────────────────┐          ┌──────────────────┐
│ App Container    │          │ App Container    │
│                  │──────→   │                  │
│ Envoy Sidecar    │          │ Envoy Sidecar    │
└──────────────────┘          └──────────────────┘
        ↕                              ↕
        └───── Istio Control Plane ────┘
               (policies, certs, observability)
```

### How It Helps

- Uniform resilience policies applied consistently across all services — no per-team implementation variance
- Language-agnostic — works for Java, Go, Python, Node.js services equally
- Centralized observability: distributed tracing, metrics, logs out of the box
- mTLS between services enforced at the mesh level without application changes

### Pitfalls if Not Applied

**Enterprise Example — Large bank (polyglot microservices):**  
50 teams each implement circuit breakers and retries differently across Java, .NET, and Go services. Some teams use aggressive retries, others have no retries. Failure modes are unpredictable and inconsistent. An audit finds that 30% of services have no timeout configuration at all. Incident response is complicated because each team's resilience behavior differs.

**Enterprise Example — Kubernetes platform without mTLS:**  
Without a service mesh enforcing mTLS, a compromised pod can make unauthenticated service-to-service calls to internal APIs. Data from the user profile service is exfiltrated because there is no mutual authentication between services — only perimeter-level security.

**Common mistakes:**
- Service mesh adds latency (sidecar hop) — not profiling the overhead before adopting
- Overly complex Istio policies that are difficult to debug → operators avoid using them
- Mixing in-app resilience libraries AND service mesh retries → double retries amplify load
- Not setting resource limits on sidecar proxies → Envoy itself can become a resource contention point

---

## Summary Matrix

| Pattern | Primary Failure Addressed | Key Benefit | Critical Pitfall |
|---|---|---|---|
| Circuit Breaker | Cascading failures | Fail fast, give downstream time to recover | No monitoring of circuit state |
| Retry + Backoff | Transient faults | Transparent recovery from blips | Retrying non-idempotent ops without keys |
| Bulkhead | Resource exhaustion | Isolate blast radius | Pool sizing too large (no real isolation) |
| Timeout | Hung calls, thread starvation | Bound worst-case latency | Not setting at all layers |
| Rate Limiter | Overload, abuse | Protect service capacity | Rate limiting only at edge |
| Fallback | Partial unavailability | Graceful UX degradation | Silent fallback hides root cause |
| Health Check | Routing to broken instances | Fast detection and recovery | Probes that trigger cascading restarts |
| Idempotency | Duplicate processing | Safe retries on any operation | Short TTL or non-durable key storage |
| Dead Letter Queue | Poison pill messages | No data loss, diagnosability | No monitoring or replay process |
| Saga Pattern | Distributed transaction failure | Autonomous consistency | No compensation logic |
| Backpressure | Queue overflow, OOM | Natural throughput equilibrium | Unbounded in-memory buffers |
| Service Mesh | Policy inconsistency, observability gaps | Uniform cross-cutting resilience | Double retries (app + mesh) |

---

## Applying Patterns Together

Resilience patterns work best as layers:

```
Incoming Request
       ↓
  Rate Limiter        (protect from overload)
       ↓
  Timeout             (bound call duration)
       ↓
  Retry + Backoff     (handle transient faults)
       ↓
  Circuit Breaker     (stop calling failing services)
       ↓
  Bulkhead            (isolate resource pools)
       ↓
  Fallback            (degrade gracefully on failure)
       ↓
  Idempotency         (safe at-least-once processing)
       ↓
  DLQ                 (capture unprocessable messages)
       ↓
  Health Checks       (route only to healthy instances)
       ↓
  Service Mesh        (enforce all of the above consistently)
```

> **Rule of thumb:** Start with Timeout + Circuit Breaker + Retry/Backoff for every synchronous call. Add Bulkhead for critical service isolation. Add DLQ + Idempotency for every async/event-driven flow. Layer in the rest as your system and team maturity grows.
