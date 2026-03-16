# ☁️ AWS Quick Reference

## Core Service Categories

| Category | Key Services |
|----------|-------------|
| Compute | EC2, Lambda, ECS, EKS, Fargate, Lightsail |
| Storage | S3, EBS, EFS, Glacier, Storage Gateway |
| Database | RDS, Aurora, DynamoDB, ElastiCache, Redshift |
| Networking | VPC, Route 53, CloudFront, API Gateway, ELB |
| Security | IAM, KMS, WAF, Shield, Secrets Manager, Cognito |
| Messaging | SQS, SNS, EventBridge, Kinesis |
| DevOps | CodePipeline, CodeBuild, CodeDeploy, CodeCommit |
| Monitoring | CloudWatch, X-Ray, CloudTrail |
| AI/ML | SageMaker, Rekognition, Comprehend, Transcribe |

---

## Compute

### EC2 (Elastic Compute Cloud)

```
Instance Families:
- General Purpose:    t3, m6i, m7g
- Compute Optimized:  c6i, c7g
- Memory Optimized:   r6i, x2idn
- Storage Optimized:  i3, d3
- GPU:                p4, g5
- Arm (Graviton):     t4g, m7g, c7g

Purchasing options:
- On-Demand:   pay per second, no commitment
- Reserved:    1-3 year commitment, up to 72% discount
- Spot:        spare capacity, up to 90% discount, can be interrupted
- Savings Plan: flexible commitment, up to 66% discount
```

### Lambda

```
- Serverless function execution
- Max timeout: 15 minutes
- Max memory:  10 GB
- Ephemeral storage: /tmp up to 10 GB
- Concurrency: 1000 concurrent executions (soft limit, per region)
- Triggers: API Gateway, S3, SQS, SNS, DynamoDB streams, EventBridge, etc.
- Runtime: Node.js, Python, Java, Go, Ruby, .NET, custom runtime

Pricing: requests + duration (GB-seconds)
Free tier: 1M requests/month, 400,000 GB-seconds/month
```

---

## Storage

### S3 (Simple Storage Service)

```
Storage classes:
- Standard:            frequently accessed, 3+ AZs, 99.99% availability
- Intelligent-Tiering: auto moves objects between tiers
- Standard-IA:         infrequently accessed, 3+ AZs
- One Zone-IA:         infrequently accessed, single AZ
- Glacier Instant:     archive, millisecond retrieval
- Glacier Flexible:    archive, minutes-hours retrieval
- Glacier Deep Archive: lowest cost, 12-48 hour retrieval

Key features:
- Max object size: 5 TB (multipart upload required > 100 MB)
- Versioning, lifecycle policies, replication (CRR/SRR)
- Static website hosting
- Presigned URLs for temporary access
- Event notifications (to Lambda, SQS, SNS)
- S3 Select / Glacier Select (query within objects)
```

---

## Database

### RDS & Aurora

```
RDS engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
Aurora: AWS-optimized MySQL & PostgreSQL compatible, 5x faster

Aurora features:
- Storage auto-scales up to 128 TB
- Up to 15 read replicas
- Aurora Serverless: on-demand auto-scaling
- Global Database: < 1 second replication across regions
- Multi-AZ: synchronous standby for failover
```

### DynamoDB

```
- Fully managed NoSQL, key-value and document
- Single-digit millisecond performance at any scale
- Auto-scaling or provisioned capacity

Key concepts:
- Partition key (hash key): distributes data
- Sort key (range key): optional, enables range queries
- GSI (Global Secondary Index): different partition + sort key
- LSI (Local Secondary Index): same partition, different sort key
- DynamoDB Streams: captures item-level changes (use with Lambda)
- DAX: in-memory cache for DynamoDB (microsecond latency)
- TTL: automatically delete expired items

Capacity modes:
- On-demand: pay per request, no capacity planning
- Provisioned: specify RCU/WCU, cheaper at predictable workloads
  - 1 RCU = 1 strongly consistent read of item up to 4 KB/s
  - 1 WCU = 1 write of item up to 1 KB/s
```

### ElastiCache

```
- Redis:     rich data structures, persistence, pub/sub, Lua scripts
- Memcached: pure caching, multi-threaded, simpler

Use cases:
- Session storage
- Database query caching
- Leaderboards, real-time analytics
- Pub/Sub messaging
```

---

## Networking

### VPC (Virtual Private Cloud)

```
Components:
- Subnets:           public (has route to IGW) or private
- Internet Gateway:  enables internet access for public subnets
- NAT Gateway:       allows private subnets to initiate outbound internet
- Route Table:       rules for network traffic routing
- Security Group:    stateful firewall at instance level
- NACL:             stateless firewall at subnet level
- VPC Peering:       connect two VPCs (non-transitive)
- Transit Gateway:   hub-and-spoke for many VPCs/VPNs
- VPC Endpoints:     private connection to AWS services (no internet)

CIDR block: /16 (65,536 IPs) to /28 (16 IPs)
Reserved IPs per subnet: first 4 + last 1 (5 total)

Security Group vs NACL:
- Security Group: stateful, instance-level, allow rules only
- NACL:          stateless, subnet-level, allow + deny rules
```

### Route 53

```
Record types:
- A:     hostname -> IPv4
- AAAA:  hostname -> IPv6
- CNAME: hostname -> hostname (not for apex/root domain)
- Alias: hostname -> AWS resource (free, works at apex)
- MX, TXT, NS, SOA

Routing policies:
- Simple:          single resource
- Weighted:        % traffic to multiple resources (A/B testing)
- Latency-based:   lowest latency region
- Failover:        active-passive (health check required)
- Geolocation:     based on user geographic location
- Geoproximity:    based on resource location (Traffic Flow)
- Multi-value:     multiple healthy records
```

### Load Balancers

```
ALB (Application Load Balancer): Layer 7 (HTTP/HTTPS)
  - Path-based, host-based routing
  - WebSocket support
  - Target groups: EC2, ECS, Lambda, IP

NLB (Network Load Balancer): Layer 4 (TCP/UDP/TLS)
  - Millions of requests per second
  - Static IP / Elastic IP
  - Low latency

CLB (Classic): legacy, avoid for new workloads

GLB (Gateway Load Balancer): Layer 3, virtual appliances
```

---

## IAM (Identity & Access Management)

```
Components:
- Users:    individual people or services
- Groups:   collection of users with shared permissions
- Roles:    assumed by AWS services or users (temporary credentials)
- Policies: JSON documents defining permissions

Policy types:
- Identity-based:  attached to users/groups/roles
- Resource-based:  attached to resources (S3 buckets, KMS keys)
- Permission boundaries: max permissions an entity can have
- SCP (Service Control Policies): org-level guardrails

Best practices:
- Enable MFA for all users, especially root
- Use least privilege principle
- Use roles for EC2/Lambda (no access keys)
- Rotate access keys regularly
- Use IAM Access Analyzer to review permissions

Policy evaluation logic:
  1. Explicit Deny -> Deny
  2. Explicit Allow -> Allow
  3. Implicit Deny (default) -> Deny
```

---

## Messaging

### SQS (Simple Queue Service)

```
- Standard Queue: unlimited throughput, at-least-once, best-effort ordering
- FIFO Queue:     exactly-once, strict ordering, 300 TPS (or 3000 with batching)

Key settings:
- Visibility Timeout:    time message is hidden after being read (default 30s)
- Message Retention:     1 min to 14 days (default 4 days)
- Dead Letter Queue:     for messages that fail processing N times
- Long Polling:          wait up to 20s for messages (reduce empty responses)
- Delay Queue:           delay delivery up to 15 minutes
```

### SNS (Simple Notification Service)

```
- Pub/Sub messaging
- Publishers send to Topics
- Subscribers: SQS, Lambda, HTTP/S, Email, SMS, mobile push

Fan-out pattern: SNS -> multiple SQS queues for parallel processing
SNS FIFO: strict ordering, deduplication (can only fan out to SQS FIFO)
```

---

## Monitoring

### CloudWatch

```
Metrics:
- Default metrics: EC2 (CPU, network, disk), RDS, Lambda, etc.
- Custom metrics: push application-level metrics
- High-resolution metrics: 1-second granularity

Alarms:
- OK, ALARM, INSUFFICIENT_DATA states
- Actions: SNS, Auto Scaling, EC2 action

Logs:
- Log groups -> Log streams
- Log Insights: query logs with SQL-like syntax
- Metric filters: extract metrics from log data
- Subscriptions: stream logs to Lambda, Kinesis, OpenSearch

Events / EventBridge:
- Schedule-based rules (cron)
- Event pattern-based rules
```

---

## High Availability & Disaster Recovery

```
RPO (Recovery Point Objective): max acceptable data loss (time)
RTO (Recovery Time Objective): max acceptable downtime

Strategies (cheapest to most expensive):
1. Backup & Restore: hours RTO/RPO
2. Pilot Light:      minimal resources running, scale up on disaster
3. Warm Standby:     scaled-down version running in DR region
4. Multi-Site Active-Active: full capacity in 2+ regions, near-zero RTO/RPO

Multi-AZ vs Multi-Region:
- Multi-AZ:     protects against AZ failure, synchronous replication
- Multi-Region: protects against region failure, asynchronous replication
```
