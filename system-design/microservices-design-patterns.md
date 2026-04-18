# 🧩 Microservices Design Patterns

A principal-engineer-level reference covering every major design pattern category in microservices architecture — from decomposition through deployment. Each entry includes the core concept, when to apply it, trade-offs, real-world failure scenarios, and the tooling ecosystem.

---

## Table of Contents

1. [Decomposition Patterns](#1-decomposition-patterns)
   - [Decompose by Business Capability](#11-decompose-by-business-capability)
   - [Decompose by Subdomain (DDD)](#12-decompose-by-subdomain-ddd)
   - [Strangler Fig](#13-strangler-fig)
2. [Communication Patterns](#2-communication-patterns)
   - [API Gateway](#21-api-gateway)
   - [Backend for Frontend (BFF)](#22-backend-for-frontend-bff)
   - [Service Mesh](#23-service-mesh)
   - [Synchronous vs Asynchronous Communication](#24-synchronous-vs-asynchronous-communication)
   - [Choreography vs Orchestration](#25-choreography-vs-orchestration)
3. [Data Management Patterns](#3-data-management-patterns)
   - [Database per Service](#31-database-per-service)
   - [Shared Database (Anti-Pattern)](#32-shared-database-anti-pattern)
   - [Saga Pattern](#33-saga-pattern)
   - [Outbox Pattern](#34-outbox-pattern)
   - [CQRS](#35-cqrs-command-query-responsibility-segregation)
   - [Event Sourcing](#36-event-sourcing)
   - [API Composition](#37-api-composition)
4. [Resilience Patterns](#4-resilience-patterns)
   - [Circuit Breaker](#41-circuit-breaker)
   - [Bulkhead](#42-bulkhead)
   - [Retry with Exponential Backoff & Jitter](#43-retry-with-exponential-backoff--jitter)
   - [Timeout](#44-timeout)
   - [Fallback & Graceful Degradation](#45-fallback--graceful-degradation)
   - [Backpressure](#46-backpressure)
5. [Security Patterns](#5-security-patterns)
   - [API Gateway Authentication](#51-api-gateway-authentication)
   - [Service-to-Service Authentication (mTLS)](#52-service-to-service-authentication-mtls)
   - [Zero Trust Architecture](#53-zero-trust-architecture)
6. [Observability Patterns](#6-observability-patterns)
   - [Log Aggregation](#61-log-aggregation)
   - [Distributed Tracing](#62-distributed-tracing)
   - [Health Check API](#63-health-check-api)
   - [Metrics Aggregation](#64-metrics-aggregation)
7. [Deployment Patterns](#7-deployment-patterns)
   - [Service Discovery](#71-service-discovery)
   - [Sidecar](#72-sidecar)
   - [Blue-Green Deployment](#73-blue-green-deployment)
   - [Canary Release](#74-canary-release)
   - [Feature Toggles](#75-feature-toggles)
8. [Anti-Patterns](#8-anti-patterns)

---

## 1. Decomposition Patterns

How you split a system into services is the most consequential architectural decision. Wrong decomposition creates a distributed monolith — all the operational complexity of microservices with none of the independence benefits.

---

### 1.1 Decompose by Business Capability

#### Concept

A business capability is something the organization **does** to generate value (e.g., "Order Management", "Inventory Tracking", "Customer Billing"). Each capability becomes a service with its own team, codebase, data store, and deployment pipeline.

```
Business Capabilities → Services (one-to-one or one-to-many)

Organization:
  ├── Order Management     → Order Service
  ├── Inventory            → Inventory Service
  ├── Customer             → Customer Service
  ├── Billing              → Billing Service
  └── Notifications        → Notification Service
```

#### When to Apply

- Greenfield systems or major re-architecture
- When organization structure already maps to clear business functions (Conway's Law alignment)

#### Trade-offs

```
✅ Stable boundaries — business capabilities rarely change
✅ Team autonomy — each team owns full end-to-end capability
✅ Enables independent release cadence per capability
❌ Requires understanding the full business domain upfront
❌ Cross-cutting concerns (auth, logging) still need a strategy
```

#### Pitfall

A payment team and an order team both having "payment status" logic leads to a distributed monolith where a change in either breaks both — because decomposition was done by **data entity** (Orders, Payments) rather than capability.

---

### 1.2 Decompose by Subdomain (DDD)

#### Concept

Domain-Driven Design (DDD) identifies bounded contexts — logical boundaries where a specific domain model applies consistently. Each bounded context maps to one or more microservices.

```
Core Domain:       The competitive differentiator (e.g., Recommendation Engine)
Supporting Domain: Necessary but not unique (e.g., User Profiles)
Generic Domain:    Commodity, buy-don't-build (e.g., Email, Auth)

Bounded Contexts:
  Order Context  | Customer Context | Catalog Context | Payment Context
  ─────────────────────────────────────────────────────────────────────
  "Order"        | "Order history"  | "Product"       | "Transaction"
  means something| means something  | means something | means something
  different in   | different in     | different in    | different in
  each context   | each context     | each context    | each context
```

#### Context Map Relationships

```
Customer ──(Partnership)──→ Order
Order    ──(Customer/Supplier)──→ Inventory
Order    ──(Anti-Corruption Layer)──→ Legacy ERP
```

#### When to Apply

- Complex domains with rich behavior (not just CRUD)
- Migrating a legacy monolith to microservices
- Multiple teams with different vocabularies for the same entity

#### Trade-offs

```
✅ Explicit model boundaries prevent anemic domain models
✅ Teams use ubiquitous language aligned with business
✅ Anti-Corruption Layer isolates legacy integration cleanly
❌ Requires deep domain modeling expertise upfront
❌ Context boundaries discovered iteratively — early wrong cuts are expensive
```

---

### 1.3 Strangler Fig

#### Concept

Gradually replace a monolith by building new functionality as microservices and routing traffic away from the monolith piece by piece until the monolith can be retired. Named after a fig tree that grows around a host tree and eventually replaces it.

```
Phase 1:  All traffic → Monolith
Phase 2:  Facade/Proxy intercepts traffic
          New Feature A → Microservice A
          Everything else → Monolith
Phase 3:  Feature B, C → Microservices B, C
          Remaining → Monolith (shrinking)
Phase N:  Monolith retired; facade removed or becomes API gateway
```

#### When to Apply

- Migrating from a legacy monolith incrementally
- Cannot afford a "big bang" rewrite (risk, cost, downtime)
- Need to keep delivering new features during migration

#### Trade-offs

```
✅ Zero-downtime migration; monolith and microservices coexist
✅ Validate microservice approach before full commitment
✅ Reversible — can route back to monolith if microservice fails
❌ Proxy/facade layer adds operational complexity
❌ Data synchronization between monolith and new services is hard
❌ Migration can drag on indefinitely (team loses discipline)
```

#### Key Decisions at Each Phase

```
1. Identify seam: which feature can be extracted without tight DB coupling?
2. Build facade/routing layer (API Gateway, Nginx, custom proxy)
3. Extract, test, validate in production shadow mode
4. Cut over traffic (feature flag or percentage routing)
5. Remove dead code from monolith
```

---

## 2. Communication Patterns

---

### 2.1 API Gateway

#### Concept

A single entry point that handles cross-cutting concerns before routing requests to downstream services.

```
Client
  │
  ▼
┌─────────────────────────────────────────────────┐
│                   API Gateway                    │
│  Auth  │  Rate Limit  │  Routing  │  Transform  │
└──────────────────────┬──────────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Order Svc    Payment Svc   User Svc
```

#### Responsibilities

```
Routing:          /orders/* → Order Service; /users/* → User Service
Authentication:   Validate JWT, API keys — services trust the gateway
Rate Limiting:    Per client, per endpoint, global
SSL Termination:  TLS handled at gateway; internal traffic can be plain HTTP
Request Shaping:  Transform request/response (protocol translation, aggregation)
Load Balancing:   Distribute across service instances
Observability:    Centralized logging, tracing, metrics
```

#### Trade-offs

```
✅ Single entry point simplifies client integration
✅ Cross-cutting concerns in one place (no duplication per service)
✅ Services decoupled from client protocol specifics
❌ Gateway is a potential single point of failure (must be HA)
❌ Can become a bottleneck if logic leaks in (business logic never belongs here)
❌ Multiple teams sharing the gateway configuration creates coupling
```

#### Pitfall — The "Fat Gateway"

Teams start adding business logic to the gateway (e.g., "if user is premium, call /orders/premium instead of /orders"). Business logic in the gateway makes independent deployments impossible. The gateway evolves into a distributed monolith chokepoint.

**Rule:** Gateway does routing, auth, rate limiting, observability. Never business logic.

#### Tools

Kong, AWS API Gateway, Nginx, Envoy, Spring Cloud Gateway, Apigee, Traefik

---

### 2.2 Backend for Frontend (BFF)

#### Concept

Instead of one generic API Gateway for all clients, create a dedicated backend aggregation layer per client type (mobile, web, third-party). Each BFF is owned by the frontend team and optimized for its client's data shape and protocol.

```
Mobile App ──→ Mobile BFF  ──┐
Web App    ──→ Web BFF     ──┼──→ Order Svc / User Svc / Catalog Svc
3rd Party  ──→ Partner BFF ──┘
```

#### When to Apply

- Different clients need significantly different data shapes or aggregations
- Mobile needs compact payloads; web needs richer data
- Frontend teams are slowed by a shared gateway bottleneck

#### Trade-offs

```
✅ Frontend teams control their own aggregation and deployment
✅ Optimized payloads per client (no over/under-fetching)
✅ Isolated failure surface per client type
❌ Code duplication across BFFs for shared concerns (auth, logging)
❌ Risk of N BFFs with divergent behavior if undisciplined
❌ Adds another network hop and service to maintain
```

---

### 2.3 Service Mesh

#### Concept

Infrastructure layer that handles all service-to-service communication — traffic management, mutual TLS, observability, retries — via a sidecar proxy injected into each service pod, without any changes to application code.

```
Service A Pod                Service B Pod
┌────────────────────┐       ┌────────────────────┐
│  App Container     │       │  App Container     │
│  +                 │──────▶│  +                 │
│  Envoy Sidecar     │       │  Envoy Sidecar     │
└────────────────────┘       └────────────────────┘
          │                            │
          └──────── Control Plane ─────┘
               (Istio / Linkerd / Consul)
```

#### What the Mesh Provides

```
Traffic Management:  Weighted routing, canary splits, circuit breaking
mTLS:               Automatic cert rotation, encrypted service-to-service traffic
Observability:      Distributed tracing, per-service metrics, access logs — free
Retries/Timeouts:   Configured via mesh policy, not application code
Authorization:      RBAC policies: "Service A may only call Service B on path /read"
```

#### Trade-offs

```
✅ Zero application code changes for resilience, security, observability
✅ Consistent policy enforcement across all services
✅ Fine-grained traffic control (A/B, canary, fault injection for testing)
❌ Significant operational complexity; Istio misconfiguration is hard to debug
❌ Sidecar adds latency (~1–2 ms per hop) and memory overhead per pod
❌ Control plane is a critical dependency — its failure can impact all service comms
```

#### When to Apply

- Organizations with 20+ services where per-service resilience code is unmaintainable
- Strong security requirement for zero-trust service-to-service auth
- Not suitable for small teams — operational burden is high

#### Tools

Istio (most feature-rich), Linkerd (simpler, lower overhead), Consul Connect, AWS App Mesh

---

### 2.4 Synchronous vs Asynchronous Communication

#### Decision Framework

```
Use Synchronous (REST/gRPC) when:
  - Client needs an immediate response to proceed
  - Simple request-response semantics
  - Strong consistency required within the interaction
  - Example: "Check if an item is in stock before confirming an order"

Use Asynchronous (Message Queue/Event) when:
  - Caller does not need an immediate response
  - Decoupling producer and consumer lifecycles matters
  - Operation may be long-running
  - Example: "Send a confirmation email after order placed"
```

#### gRPC vs REST

```
Feature          REST (HTTP/1.1+JSON)        gRPC (HTTP/2+Protobuf)
──────────────   ─────────────────────       ──────────────────────
Payload          JSON (text, verbose)         Binary Protobuf (~5-10x smaller)
Schema           Optional (OpenAPI)           Mandatory (.proto contract)
Streaming        Limited (SSE, chunked)       Native bidirectional streaming
Typing           Runtime validation           Compile-time typed contracts
Browser Support  Native                       Requires grpc-web proxy
Best for         Public APIs, polyglot        Internal high-throughput RPCs
```

---

### 2.5 Choreography vs Orchestration

#### Choreography (Event-Driven)

Each service listens for events and decides independently what action to take next. No central coordinator.

```
Order Placed Event
   │
   ├──▶ Inventory Service  → reserves stock → Stock Reserved Event
   │                                               │
   ├──▶ Payment Service    ◀──────────────────────┘ → Payment Processed Event
   │                                               │
   └──▶ Notification Svc  ◀──────────────────────┘ → sends email
```

```
✅ Loose coupling — services don't know about each other
✅ Easy to add new consumers without changing producers
✅ Scales well; each service independently
❌ Business flow is implicit (distributed across services)
❌ Hard to debug; tracing an end-to-end flow requires tooling
❌ Risk of cyclical event chains causing infinite loops
```

#### Orchestration (Process Manager / Saga Orchestrator)

A central coordinator explicitly calls each service and decides the next step based on outcomes.

```
Order Orchestrator
   │── calls ──▶ Inventory Service → reserve stock
   │── calls ──▶ Payment Service   → charge customer
   │── calls ──▶ Shipping Service  → create shipment
   │── calls ──▶ Notification Svc  → send confirmation
```

```
✅ Business flow is explicit and visible in one place
✅ Easier to reason about and debug
✅ Compensating transactions are straightforward to implement
❌ Orchestrator becomes a coupling point
❌ Harder to scale the orchestrator if it handles many flows
❌ Teams working on the same orchestrator create coordination overhead
```

#### Decision Heuristic

```
Use Choreography:  event-driven, simple reactions, no complex rollback
Use Orchestration: complex multi-step workflows, distributed transactions,
                   compliance audit trail required
Use Hybrid:        choreography at boundaries, orchestration within a bounded context
```

---

## 3. Data Management Patterns

---

### 3.1 Database per Service

#### Concept

Each microservice owns its data store exclusively. No other service may access it directly; all access goes through the service's API.

```
Order Service  ──▶ Orders DB (PostgreSQL)
User Service   ──▶ Users DB  (MySQL)
Catalog Service──▶ Catalog DB (MongoDB)
Cart Service   ──▶ Cart DB   (Redis)
```

#### Why It Matters

- Services can evolve schema independently without cross-team coordination
- Teams can choose the right database technology for their domain
- Failure of one data store doesn't cascade to all services
- Clear ownership — no implicit shared state

#### Trade-offs

```
✅ True service autonomy and independent deployability
✅ Technology polyglot — each service picks the best fit
✅ Isolated failure domains
❌ Cross-service joins require API composition or event-driven sync
❌ Distributed transactions become complex (see Saga)
❌ Higher operational overhead (more databases to manage, monitor, back up)
```

---

### 3.2 Shared Database (Anti-Pattern)

Multiple services read and write to the same database schema.

```
Why teams do it: "Just easier to query"

What actually happens:
  - Schema change in User table breaks Order Service silently
  - Services couple at the data layer, not at the API layer
  - "One service DB" deploys block all teams until all services are updated
  - Impossible to migrate one service to a different DB technology
```

**Use shared database only as a temporary migration step (Strangler Fig Phase 1), with an explicit plan to separate.**

---

### 3.3 Saga Pattern

#### Concept

Manages a multi-step distributed transaction across services using a sequence of local transactions and compensating transactions for rollback. Replaces ACID distributed transactions (2PC) which don't scale.

#### Orchestration-based Saga

```
Order Saga Orchestrator
  1. Create Order (PENDING)       → Order Service
  2. Reserve Inventory            → Inventory Service
     ✓ success → Step 3
     ✗ failure → Compensate: Cancel Order
  3. Process Payment              → Payment Service
     ✓ success → Step 4
     ✗ failure → Compensate: Release Inventory, Cancel Order
  4. Create Shipment              → Shipping Service
  5. Order CONFIRMED
```

#### Choreography-based Saga

```
OrderPlaced event
  → Inventory reserves stock → InventoryReserved event
    → Payment charges card  → PaymentProcessed event
      → Shipping creates label → ShipmentCreated event
        → Order marked CONFIRMED

On failure at any step, a compensating event chain runs in reverse.
```

#### Compensating Transactions

```
Business action            Compensating action
─────────────────────────────────────────────
Create order               Cancel order
Reserve inventory          Release inventory reservation
Charge payment             Refund payment
Create shipment            Cancel shipment
```

#### Trade-offs

```
✅ Avoids 2PC — scales horizontally
✅ Each step uses a local ACID transaction
✅ Supports long-running business processes
❌ Compensating transactions must be explicitly designed for every step
❌ No rollback isolation — intermediate states are visible
❌ Eventual consistency; temporarily inconsistent states exist
❌ Complex to implement and test correctly
```

---

### 3.4 Outbox Pattern

#### Concept

Solves the dual-write problem: atomically persisting state changes AND publishing events without distributed transactions.

**Problem:**

```
// Naive (broken) approach — two independent writes:
db.save(order)           // ✓ succeeds
messageQueue.publish(event)  // ✗ crashes → event never sent
// Result: order saved but downstream never notified
```

**Solution:**

```
Step 1: In ONE local DB transaction:
  INSERT INTO orders (...)       -- business record
  INSERT INTO outbox (event_payload, status=PENDING)  -- event record

Step 2: Outbox Relay process (background):
  SELECT * FROM outbox WHERE status=PENDING
  → Publish to message broker
  → UPDATE outbox SET status=PUBLISHED

Guarantees: at-least-once delivery
```

#### Variants

```
Polling Publisher:  background job queries outbox table at interval
Transaction Log Tailing: (CDC) reads DB transaction log directly
  Tools: Debezium (Kafka Connect), AWS DMS
```

#### Trade-offs

```
✅ Exactly-one write → atomicity between state change and event emission
✅ No 2PC, no distributed transaction coordinator
✅ Works with any relational DB + any message broker
❌ Consumers must handle duplicate messages (idempotent consumer required)
❌ Outbox table grows; requires cleanup/archival strategy
❌ Adds infrastructure (relay process or CDC pipeline)
```

---

### 3.5 CQRS (Command Query Responsibility Segregation)

#### Concept

Separate the write model (Command side) from the read model (Query side). They can use different data stores, schemas, and scale independently.

```
                    ┌──── Command Model ────┐
Client Write ──────▶│ Domain Logic + Write DB│──▶ Event Published
                    └───────────────────────┘
                                                        │
                                              ┌─────────▼──────────┐
                                              │  Read Model Updater │
                                              └─────────┬──────────┘
                    ┌──── Query Model ───────┐          │
Client Read ───────▶│ Denormalized Read DB   │◀─────────┘
                    └───────────────────────┘
```

#### When to Apply

```
✅ Read-heavy systems where read and write scaling requirements differ
✅ Complex domain logic on write side; simple projection on read side
✅ Multiple read representations of the same data needed
✅ Event Sourcing (natural pairing)
❌ Overkill for simple CRUD services
❌ Adds eventual consistency — reads may lag behind writes
❌ Increases system complexity and operational surface area
```

#### Example

```
E-commerce Order System:
  Write side: PostgreSQL (Order aggregate, enforces business rules)
  Read sides:
    - Elasticsearch (full-text order search)
    - Redis (customer's last 5 orders for display)
    - BI Warehouse (aggregated order analytics)
```

---

### 3.6 Event Sourcing

#### Concept

Store the **log of events** that caused each state change, not the current state itself. The current state is derived (projected) by replaying events from the beginning or from a snapshot.

```
Traditional:  orders table row → { status: "SHIPPED" }

Event Sourcing:
  events table:
    1. OrderCreated    { orderId: 42, items: [...] }
    2. PaymentReceived { orderId: 42, amount: 99.99 }
    3. OrderShipped    { orderId: 42, trackingId: "XYZ" }

Current state = replay of events 1+2+3
```

#### Snapshot Optimization

```
Replaying 10,000 events per request is impractical.
Periodically save a snapshot of current state at event N.
On load: restore snapshot + replay only events after N.
```

#### Trade-offs

```
✅ Complete audit trail by design — regulatory compliance built-in
✅ Temporal queries: "What was the state of this order at 3pm yesterday?"
✅ Event log is the integration point for CQRS projections
✅ Rebuild any read model from scratch by replaying events
❌ Eventual consistency for read projections
❌ Schema evolution of past events is complex (event versioning strategy needed)
❌ High implementation complexity — not suited for simple CRUD domains
❌ Storage grows without bound (mitigated by snapshots and archival)
```

#### When to Apply

- Financial systems requiring full audit trail (banking, trading, billing)
- Systems where temporal queries are needed (insurance, medical records)
- Naturally paired with CQRS and complex domain-driven models

---

### 3.7 API Composition

#### Concept

For queries that span multiple services (replacing cross-service DB joins), the API Composer collects data from multiple services in parallel and joins them in memory.

```
GET /order-details/{orderId}

API Composer:
  parallel:
    order = OrderService.getOrder(orderId)
    customer = CustomerService.getCustomer(order.customerId)
    shipment = ShippingService.getShipment(orderId)
  return merge(order, customer, shipment)
```

#### Trade-offs

```
✅ Works with Database-per-Service model
✅ Simple to implement for non-critical reads
❌ In-memory join is expensive at scale (N+1 problem)
❌ Availability: if any service is down, the composed response fails
   (mitigate with fallbacks or partial responses)
❌ No transactions — data from different services may be momentarily inconsistent
```

**For complex read scenarios, CQRS with a dedicated read model outperforms API Composition.**

---

## 4. Resilience Patterns

> See also: [resilience-patterns.md](./resilience-patterns.md) for detailed coverage of each pattern with implementation examples.

---

### 4.1 Circuit Breaker

```
States:
  CLOSED    → normal; requests pass through
  OPEN      → failing; requests fail fast, no downstream call
  HALF-OPEN → probe; limited requests test if service recovered

Thresholds to configure:
  - Failure rate threshold (e.g., >50% failures in 10s window → OPEN)
  - Slow call rate threshold (e.g., >50% calls >2s → OPEN)
  - Wait duration in OPEN state (e.g., 30s before HALF-OPEN)
  - Permitted calls in HALF-OPEN (e.g., 5 probe calls)
```

**Prevents:** Thread pool exhaustion cascading from a slow downstream service.

**Tools:** Resilience4j, Istio/Envoy (mesh-level), Polly (.NET)

---

### 4.2 Bulkhead

```
Isolate thread pools / connection pools per downstream dependency.

Without bulkhead:
  Payment calls + Order calls + Inventory calls share one 200-thread pool
  → Slow Payment service exhausts all 200 threads
  → Orders and Inventory calls also fail (collateral damage)

With bulkhead:
  Payment pool:   50 threads
  Order pool:     100 threads
  Inventory pool: 50 threads
  → Payment pool saturates → only Payment calls rejected
  → Orders and Inventory unaffected
```

**Also applies to:** Kubernetes resource limits per service, connection pool limits per DB.

---

### 4.3 Retry with Exponential Backoff & Jitter

```
Retry only transient failures (5xx, network timeout)
Never retry non-transient failures (4xx client errors, business validation)

Backoff formula:
  wait = min(cap, base * 2^attempt) + random_jitter(0..jitter_range)

Example:
  attempt 1: wait 1s  + 0–500ms
  attempt 2: wait 2s  + 0–500ms
  attempt 3: wait 4s  + 0–500ms
  attempt 4: give up → fallback / dead letter

Jitter prevents thundering herd: all clients retrying simultaneously
after a restart of a downstream service.
```

---

### 4.4 Timeout

```
Always set a timeout. Never allow unbounded waits.

Configure at every layer:
  Connection timeout:  time to establish TCP connection    (e.g., 2s)
  Read timeout:        time to receive first byte          (e.g., 5s)
  Write timeout:       time to finish sending the request  (e.g., 5s)
  Total request timeout: end-to-end budget                 (e.g., 10s)

Cascade-safe timeout budget:
  API Gateway timeout > BFF timeout > Service-to-service timeout
  E.g., 30s > 20s > 10s
  (Outer timeout must be larger than inner to allow graceful handling)
```

---

### 4.5 Fallback & Graceful Degradation

```
When a dependency fails, return a degraded but useful response:
  - Cached stale data
  - Default/empty response
  - Feature disabled (hide the component in the UI)
  - Redirect to a simpler code path

Example:
  Recommendation service down →
    fallback: return top-10 globally popular products (cached, stale-ok)
  
  NOT acceptable fallback:
    return HTTP 500 to the user because recommendations aren't critical
```

---

### 4.6 Backpressure

```
Producer generates data faster than consumer can process.

Signals (consumer → producer):
  - TCP flow control (built-in)
  - Reactive Streams: cancel / request(N) signals
  - Queue depth monitoring → pause producer
  - HTTP 429 Too Many Requests

Strategies:
  Drop:    discard excess messages (acceptable for metrics, telemetry)
  Buffer:  queue bounded by memory/disk limit
  Block:   slow down producer to match consumer rate
  Shed:    reject new work with immediate error response

Unbounded queues without backpressure = out-of-memory crash under load.
```

---

## 5. Security Patterns

---

### 5.1 API Gateway Authentication

```
External requests → API Gateway validates token → passes claims to services

Token types:
  JWT (JSON Web Token):
    - Stateless; service validates signature without network call
    - Contains claims: userId, roles, scopes, expiry
    - Expiry must be short (15m–1h); refresh tokens for long sessions

  Opaque Token:
    - Random string; must call auth server to validate (introspection)
    - Revocable immediately
    - Higher latency per request

Gateway passes identity downstream via:
  X-User-ID: 12345
  X-User-Roles: admin,billing
  Services trust these headers ONLY from the gateway (never from external)
```

#### Pitfall

Services that accept identity headers from any caller (not just the gateway) allow any client to impersonate any user by forging headers.

---

### 5.2 Service-to-Service Authentication (mTLS)

```
Problem: Service A calls Service B — how does B know the caller is truly A?

mTLS (Mutual TLS):
  - Both client and server present certificates
  - Certificate identifies the service (SPIFFE/SPIRE identity format)
  - Automatic rotation (service mesh handles cert lifecycle)

Without mTLS, a compromised service inside the cluster can call
any other service impersonating legitimate callers.

SPIFFE ID format: spiffe://cluster.local/ns/prod/sa/order-service
```

#### Implementation Approaches

```
Service Mesh (Istio/Linkerd):  automatic mTLS — zero code changes
Manual:                        manage certs in code (error-prone, avoid)
```

---

### 5.3 Zero Trust Architecture

```
Principle: "Never trust, always verify" — no implicit trust based on network location

Pillars:
  1. Identity: every service has a cryptographic identity (SPIFFE)
  2. Authorization: explicit RBAC/ABAC policies per service-to-service call
  3. Encryption: all traffic encrypted in transit (mTLS everywhere)
  4. Least Privilege: services only get permissions they need
  5. Observability: all access logged and audited

Kubernetes implementation:
  - NetworkPolicy: restrict pod-to-pod traffic at network layer
  - Service mesh AuthorizationPolicy: restrict which services can call which endpoints
  - Secrets management: Vault / AWS Secrets Manager (no secrets in env vars or config maps)
```

---

## 6. Observability Patterns

The three pillars of observability are **Logs**, **Metrics**, and **Traces**. In microservices, none of them is sufficient alone.

---

### 6.1 Log Aggregation

```
Problem: 50 services × 10 instances each = 500 log streams
         kubectl logs is not a debugging strategy at scale

Pattern:
  Service → Structured Log (JSON) → Log Agent (Fluentd/Filebeat)
          → Log Aggregator (Elasticsearch / Loki / Splunk)
          → Dashboards + Alerts (Kibana / Grafana)

Structured log fields (always include):
  timestamp, service_name, version, trace_id, span_id,
  correlation_id, user_id, request_id, level, message
```

#### Correlation ID

```
API Gateway generates X-Correlation-ID on entry.
Every service propagates it in all outbound calls and logs it.
Result: all log lines for one user request share the same correlation ID.
grep correlationId=abc-123 across all service logs = full request trace.
```

---

### 6.2 Distributed Tracing

```
One user request spans multiple services.
Distributed tracing reconstructs the full call graph with timing.

Concepts:
  Trace:  complete end-to-end journey of one request (has a TraceID)
  Span:   one unit of work within a trace (one service call)
          contains: service name, operation, start time, duration, status, tags
  Parent-Child: spans form a tree showing call hierarchy

Propagation (W3C Trace Context standard):
  traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01

What to look for:
  - Which span is the bottleneck (longest duration)?
  - Which service introduced an error?
  - How does p99 latency compare to p50 across all spans?
```

#### Tools

Jaeger, Zipkin, AWS X-Ray, Datadog APM, Honeycomb (best for high-cardinality queries)

**Instrumentation:** OpenTelemetry (language-agnostic standard SDK — use this)

---

### 6.3 Health Check API

```
Every service exposes:

GET /health/live   → liveness: is the process alive? (not deadlocked)
  Response: 200 OK  or  500 (restart the container)

GET /health/ready  → readiness: is the service ready to accept traffic?
  Response: 200 OK  or  503 (remove from load balancer rotation)
  Checks: DB connection alive, cache accessible, dependencies healthy

GET /health/startup → startup: has the service finished initializing?
  (Kubernetes startupProbe — prevents premature liveness checks)
```

#### What Not to Do

```
❌ Returning 200 always (fake health check — useless)
❌ Making external service calls in liveness check
   (external service down → your container restarts in an infinite loop)
❌ No health check at all (load balancer sends traffic to dead instances)
```

---

### 6.4 Metrics Aggregation

```
RED Method (for services):
  Rate:     requests per second
  Errors:   error rate (4xx, 5xx)
  Duration: latency distribution (p50, p95, p99)

USE Method (for resources):
  Utilization: % time resource is busy (CPU, DB connections)
  Saturation:  queue depth, wait time
  Errors:      error count

Key per-service metrics to always capture:
  - http_requests_total{method, path, status_code}
  - http_request_duration_seconds{quantiles}
  - db_pool_connections_active / db_pool_connections_waiting
  - outbound_call_duration_seconds{target_service}
  - circuit_breaker_state{service}  ← custom
  - queue_depth{queue_name}
```

#### Tools

```
Instrumentation:  Micrometer (JVM), OpenTelemetry, Prometheus client libs
Collection:       Prometheus (pull-based scraping)
Storage:          Prometheus TSDB, Thanos (long-term), Cortex, VictoriaMetrics
Dashboards:       Grafana
Alerting:         Alertmanager, PagerDuty, OpsGenie
```

---

## 7. Deployment Patterns

---

### 7.1 Service Discovery

#### Client-Side Discovery

```
Client queries Service Registry → gets list of instances → load balances itself

Service Registry: Eureka, Consul, etcd, ZooKeeper

Flow:
  Service starts → registers with registry (IP, port, health URL)
  Service scales → new instance self-registers
  Service stops  → deregisters (or TTL expires)
  Client         → queries registry, caches instance list, does load balancing

✅ Simple infrastructure — any LB algorithm the client wants
❌ Client must implement discovery + load balancing logic (every language/framework)
```

#### Server-Side Discovery (Kubernetes native)

```
Client → DNS (service name) → Kubernetes Service → kube-proxy → Pod

kube-proxy watches etcd for endpoint changes and updates iptables/IPVS rules.
Client needs no discovery logic — just call http://order-service:8080

✅ Zero client-side logic; consistent across all languages
✅ Platform-native in Kubernetes
❌ Slightly less flexible for custom LB algorithms (mitigated by service mesh)
```

---

### 7.2 Sidecar

#### Concept

Deploy a secondary container in the same pod as the main service container. The sidecar handles cross-cutting concerns without changing application code.

```
Application Pod:
  ┌─────────────────────────────────┐
  │  Main App Container             │
  │  + Sidecar: Envoy Proxy         │  ← traffic in/out
  │  + Sidecar: Log Shipper         │  ← reads log files, ships to aggregator
  │  + Sidecar: Config Reloader     │  ← watches config, hot-reloads
  │  (shared network namespace,     │
  │   shared /tmp volume)           │
  └─────────────────────────────────┘
```

#### Trade-offs

```
✅ Separation of concerns without library dependencies in app code
✅ Language-agnostic — sidecar works regardless of app's language
✅ Independently deployable and upgradable from the main app
❌ Increased pod startup time and resource consumption
❌ Complex debugging when sidecar and app interact unexpectedly
```

---

### 7.3 Blue-Green Deployment

```
Maintain two identical production environments:
  Blue:  current live version
  Green: new version being deployed and tested

Traffic: 100% → Blue (initially)

Steps:
  1. Deploy new version to Green
  2. Run smoke tests against Green
  3. Switch load balancer: 100% traffic → Green
  4. Monitor Green for issues
  5. If healthy: decommission Blue (or keep as rollback)
  6. If broken: switch back 100% → Blue (seconds to rollback)
```

```
✅ Zero-downtime deployment
✅ Instant rollback (switch back to Blue)
✅ Test production environment before taking traffic
❌ Requires 2x infrastructure during transition
❌ DB schema migrations must be backward-compatible (both versions share DB)
❌ Long-running transactions during cutover need draining strategy
```

---

### 7.4 Canary Release

```
Gradually shift traffic to the new version to limit blast radius.

  Version V1: 95% traffic
  Version V2: 5% traffic  (canary — like the canary in the coal mine)

Progression:
  5% → 10% → 25% → 50% → 100%
  Wait + monitor RED metrics at each step before proceeding.
  Auto-rollback if error rate or latency spikes.

Header-based canary (targeted testing):
  X-Canary: true → route to V2
  All other requests → V1
```

```
✅ Real production traffic validates the new version with limited user impact
✅ Metrics-driven promotion/rollback (automated with Argo Rollouts, Flagger)
✅ Works with feature flags for even finer control
❌ Both versions run simultaneously (backward compatibility required)
❌ Requires traffic splitting at load balancer or service mesh level
❌ Debugging requires filtering metrics/logs by version
```

#### Tools

Argo Rollouts, Flagger (Flux), Spinnaker, Istio VirtualService weights

---

### 7.5 Feature Toggles

```
Decouple deployment from release. Merge code to main, deploy often,
enable features independently via configuration.

Toggle types:
  Release Toggle:     hide incomplete features until ready
  Ops Toggle:         kill switch for features under high load
  Experiment Toggle:  A/B testing (different users see different behavior)
  Permission Toggle:  feature available only to specific users/tenants

Toggle evaluation:
  User 12345 + Feature "new-checkout-flow" + Rules:
    → Is user in beta group? → YES → return true
    → Is feature globally enabled? → NO → return false
```

```
✅ Trunk-based development — no long-lived feature branches
✅ Instant rollback without a redeploy
✅ Progressive rollout to specific users/segments
❌ Toggle debt: old, unused toggles accumulate → code complexity
❌ Combinatorial explosion: 10 toggles = 1024 possible states
❌ Feature toggle configuration is now a production dependency (must be HA)

Rule: every toggle has an owner and a deletion date.
```

#### Tools

LaunchDarkly, Unleash (open source), Flagsmith, AWS AppConfig, Spring Cloud Config

---

## 8. Anti-Patterns

Knowing what NOT to do is as important as knowing the patterns.

---

### 8.1 Distributed Monolith

```
Symptom: multiple "microservices" that must be deployed together
         because they share DB schema, hard-coded service URLs,
         or synchronous call chains where all must succeed.

Root cause: decomposition was done along technical layers
            (UI service, Business Logic service, Data service)
            instead of business capabilities.

Fix: re-decompose by capability; enforce Database-per-Service;
     eliminate synchronous chains with events or sagas.
```

---

### 8.2 Chatty Services

```
Symptom: one user action triggers 10–20 synchronous service-to-service calls
         Fan-out of synchronous calls kills p99 latency.

Example:  "Load user profile page" → 12 sequential service calls
          Each call: 10ms → total: 120ms minimum, p99 likely 500ms+

Fix:
  - Parallelize independent calls (API Composition)
  - Use CQRS read models that pre-join data
  - Coarser-grained APIs: reduce chattiness at the interface level
  - Event-driven pre-computation of composite views
```

---

### 8.3 Synchronous Call Chains (Death by a Thousand Timeouts)

```
A → B → C → D → E  (all synchronous)

If E has 1% error rate:
  Effective availability of A = 0.99^4 ≈ 96%
  (Each hop multiplies error probability)

Fix:
  - Break chains with async messaging
  - Apply timeouts at every hop (budget propagation)
  - Return partial responses where possible
  - Circuit breakers at every outbound call
```

---

### 8.4 No Idempotency

```
Problem: message queues deliver at-least-once. Retries happen.
         Non-idempotent consumers process the same message twice.

Example: "Charge $100" executed twice = $200 charged.

Fix:
  - Consumers check deduplication ID before processing
  - Use idempotency keys (UUID per logical operation)
  - Outbox + exactly-once consumer tracking table
  
  Idempotency table:
    INSERT INTO processed_events (event_id) VALUES (?) ON CONFLICT DO NOTHING
    → if 0 rows inserted, already processed → skip
```

---

### 8.5 Ignoring Backward Compatibility

```
Breaking changes:
  - Renaming/removing a field in an API response
  - Changing message schema in a Kafka topic
  - Altering a DB column a shared library reads

Rules:
  API versioning: v1 lives until all consumers migrated
  Expand-contract migration:
    Phase 1 (expand):   add new field alongside old field
    Phase 2 (migrate):  consumers update to read new field
    Phase 3 (contract): remove old field once all consumers migrated

Event schema evolution: use Avro/Protobuf with a Schema Registry
  - Backward compatible: new schema can read old messages
  - Forward compatible: old schema can read new messages
  - Full compatible: both directions
```

---

### 8.6 Skipping Observability

```
Pattern: "We'll add monitoring later"

Reality: first production incident at 3am with 50 services and no
         tracing, no structured logs, no dashboards — undebugable.

Non-negotiable baseline for every service before it goes to production:
  □ Structured JSON logs with correlation ID and trace ID
  □ /health/live and /health/ready endpoints
  □ RED metrics exported (Prometheus or equivalent)
  □ Distributed tracing instrumented (OpenTelemetry)
  □ Runbook documenting known failure modes and recovery steps
```

---

## Quick Decision Reference

```
Question                                    Pattern
──────────────────────────────────────────────────────────────────────────
How do I split my monolith?                 Strangler Fig
How do I define service boundaries?         DDD Bounded Contexts
Multiple clients need different APIs?        BFF
Single entry point for all clients?         API Gateway
Service needs to call 10 others?            API Composition or CQRS read model
Cross-service transaction?                  Saga + Outbox
State + full audit trail?                   Event Sourcing + CQRS
Read scales differently than write?         CQRS
Service calls keep failing?                 Circuit Breaker + Retry + Timeout
One slow service taking down others?        Bulkhead
How do I deploy without downtime?           Blue-Green or Canary
How do I test a new version safely?         Canary + Feature Toggles
How do I secure internal traffic?           mTLS via Service Mesh
How do I debug across 50 services?          Distributed Tracing (OpenTelemetry)
Services discovering each other?            Service Discovery (Consul / k8s DNS)
Cross-cutting concerns without code change? Sidecar / Service Mesh
Atomic DB write + event publish?            Outbox Pattern
```
