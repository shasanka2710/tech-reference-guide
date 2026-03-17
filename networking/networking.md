# 🌐 Networking Fundamentals — Principal Engineer Reference

> **Audience:** Engineers with development backgrounds stepping into or deepening principal-level system knowledge.  
> **Goal:** Practical, interview-ready, and architecture-friendly networking reference.

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| 1 | [OSI & TCP/IP Models](#1--osi--tcpip-models) |
| 2 | [IP Addressing & Subnetting](#2--ip-addressing--subnetting) |
| 3 | [DNS — Domain Name System](#3--dns--domain-name-system) |
| 4 | [TCP vs UDP](#4--tcp-vs-udp) |
| 5 | [HTTP / HTTPS & TLS](#5--http--https--tls) |
| 6 | [Common Network Protocols](#6--common-network-protocols) |
| 7 | [Sockets & Connections](#7--sockets--connections) |
| 8 | [Load Balancing](#8--load-balancing) |
| 9 | [Proxies & API Gateways](#9--proxies--api-gateways) |
| 10 | [CDN — Content Delivery Network](#10--cdn--content-delivery-network) |
| 11 | [Network Security](#11--network-security) |
| 12 | [WebSockets & Real-Time Protocols](#12--websockets--real-time-protocols) |
| 13 | [Microservices Networking](#13--microservices-networking) |
| 14 | [Cloud Networking](#14--cloud-networking) |
| 15 | [Network Performance & Observability](#15--network-performance--observability) |
| 16 | [Diagnostics Toolkit](#16--diagnostics-toolkit) |
| 17 | [Quick-Reference Cheatsheets](#17--quick-reference-cheatsheets) |

---

## 1 · OSI & TCP/IP Models

### OSI Model — 7 Layers

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 7 │ Application  │ HTTP, FTP, SMTP, DNS, gRPC           │
│  Layer 6 │ Presentation │ TLS/SSL, JSON, XML, encoding          │
│  Layer 5 │ Session      │ Session establishment, RPC            │
│  Layer 4 │ Transport    │ TCP, UDP — ports & reliability        │
│  Layer 3 │ Network      │ IP, ICMP, routing                     │
│  Layer 2 │ Data Link    │ Ethernet, MAC addresses, ARP          │
│  Layer 1 │ Physical     │ Cables, Wi-Fi signals, bits           │
└─────────────────────────────────────────────────────────────────┘
```

### TCP/IP Model — 4 Layers (practical stack)

| TCP/IP Layer | Maps to OSI | Examples |
|---|---|---|
| **Application** | L5 + L6 + L7 | HTTP, DNS, SSH, SMTP |
| **Transport** | L4 | TCP, UDP |
| **Internet** | L3 | IP, ICMP, BGP |
| **Network Access** | L1 + L2 | Ethernet, Wi-Fi, ARP |

### How Data Flows (Encapsulation)

```
Sender:
  Application data
    → [Transport] wrap in TCP/UDP segment (adds port)
      → [Network]  wrap in IP packet (adds IP)
        → [Data Link] wrap in Ethernet frame (adds MAC)
          → [Physical] transmit bits

Receiver: strips headers in reverse order
```

### Key Takeaways for Developers

- **L4 (Transport)**: When you open a socket, you're working here.
- **L3 (Network)**: IP routing; this is how packets traverse the internet.
- **L7 (Application)**: Everything your app code builds on top of.
- **Why it matters**: Diagnosing latency, packet loss, or connection resets requires knowing which layer is responsible.

---

## 2 · IP Addressing & Subnetting

### IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---|---|---|
| Size | 32-bit (4 bytes) | 128-bit (16 bytes) |
| Format | `192.168.1.1` | `2001:0db8:85a3::8a2e:0370:7334` |
| Addresses | ~4.3 billion | ~340 undecillion |
| Notation | Dotted decimal | Colon-hex |
| Header size | 20 bytes | 40 bytes (fixed) |
| NAT needed? | Yes (shortage) | No |
| Broadcast | Yes | Replaced by multicast |

### IPv4 Address Classes (Legacy)

| Class | Range | Default Subnet | Use |
|---|---|---|---|
| A | 1.0.0.0 – 126.255.255.255 | /8 | Large networks |
| B | 128.0.0.0 – 191.255.255.255 | /16 | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | /24 | Small networks |
| D | 224.0.0.0 – 239.255.255.255 | — | Multicast |
| E | 240.0.0.0 – 255.255.255.255 | — | Reserved |

### Private (RFC 1918) Address Ranges

```
10.0.0.0/8          →  10.0.0.0   – 10.255.255.255  (16M hosts)
172.16.0.0/12       →  172.16.0.0 – 172.31.255.255  (1M hosts)
192.168.0.0/16      →  192.168.0.0 – 192.168.255.255 (65K hosts)
127.0.0.0/8         →  Loopback (localhost)
169.254.0.0/16      →  Link-local (APIPA — no DHCP response)
```

### CIDR — Classless Inter-Domain Routing

```
Notation:  <network_address>/<prefix_length>
Example:   192.168.1.0/24

Prefix /24  → subnet mask 255.255.255.0
           → 2^(32-24) = 256 addresses, 254 usable hosts

Quick CIDR table:
  /8  → 16,777,214 hosts
  /16 →     65,534 hosts
  /24 →        254 hosts
  /28 →         14 hosts  (common for small cloud subnets)
  /30 →          2 hosts  (point-to-point links)
  /32 →          1 host   (single IP, e.g., host route)
```

### Subnetting Mental Model

```
Network:   192.168.10.0/24
Subnet mask: 255.255.255.0

  Network address:   192.168.10.0    ← first address (not usable)
  Broadcast address: 192.168.10.255  ← last address (not usable)
  Usable hosts:      192.168.10.1 – 192.168.10.254  (254 hosts)

Splitting /24 into /26 subnets (4 subnets of 64 addresses each):
  192.168.10.0/26   → .0   – .63
  192.168.10.64/26  → .64  – .127
  192.168.10.128/26 → .128 – .191
  192.168.10.192/26 → .192 – .255
```

### Special Addresses

| Address | Meaning |
|---|---|
| `0.0.0.0` | Unspecified / "all interfaces" (bind) |
| `127.0.0.1` | Loopback (localhost) |
| `255.255.255.255` | Limited broadcast |
| `::1` | IPv6 loopback |
| `::` | IPv6 unspecified |

---

## 3 · DNS — Domain Name System

### Resolution Flow

```
Browser → OS Cache → /etc/hosts → Local Resolver (ISP/8.8.8.8)
                                        │
                               ┌────────▼─────────┐
                               │  Root Nameserver  │  → knows TLD servers
                               └────────┬──────────┘
                               ┌────────▼──────────┐
                               │  TLD Nameserver   │  e.g., .com, .org
                               └────────┬──────────┘
                               ┌────────▼──────────┐
                               │ Authoritative NS  │  → returns final IP
                               └───────────────────┘
```

### DNS Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | IPv4 address | `api.example.com → 93.184.216.34` |
| **AAAA** | IPv6 address | `api.example.com → 2606:2800::1` |
| **CNAME** | Alias to another hostname | `www → example.com` |
| **MX** | Mail exchange server | `example.com → mail.example.com` |
| **TXT** | Arbitrary text (SPF, DKIM, verification) | `"v=spf1 include:..."` |
| **NS** | Authoritative nameserver | `example.com → ns1.dns.com` |
| **PTR** | Reverse DNS (IP → hostname) | `34.216.184.93.in-addr.arpa` |
| **SRV** | Service location (host + port) | `_http._tcp.example.com` |
| **SOA** | Start of authority (zone metadata) | Serial, refresh, TTL info |
| **CAA** | Certificate Authority Authorization | Restricts which CAs can issue certs |

### TTL — Time to Live

```
TTL controls how long a resolver caches a record.

Low TTL  (30–300s)  → Fast propagation, more DNS queries, good for migrations
High TTL (3600+s)   → Fewer queries, slower propagation, good for stable records

Before a migration: lower TTL to 60s at least 48h in advance.
```

### Recursive vs Iterative Resolution

| Type | Who does the work? |
|---|---|
| **Recursive** | Resolver does all the querying on client's behalf |
| **Iterative** | Client asks each server in turn |

### DNS Security

| Mechanism | Purpose |
|---|---|
| **DNSSEC** | Cryptographic signing of DNS records (prevents spoofing) |
| **DoH** (DNS over HTTPS) | Encrypts DNS queries inside HTTPS |
| **DoT** (DNS over TLS) | Encrypts DNS queries via TLS on port 853 |
| **Split-Horizon DNS** | Different answers for internal vs external queries |

---

## 4 · TCP vs UDP

### Side-by-Side Comparison

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, ordering, retransmission | Best-effort, no guarantee |
| Error checking | Checksum + acknowledgement | Checksum only |
| Flow control | Yes (sliding window) | No |
| Congestion control | Yes (AIMD, CUBIC, BBR) | No |
| Header size | 20–60 bytes | 8 bytes |
| Speed | Slower (overhead) | Faster (low overhead) |
| Use cases | HTTP, SSH, FTP, databases | DNS, video streaming, gaming, VoIP |

### TCP 3-Way Handshake

```
Client                    Server
  │──── SYN (seq=x) ─────▶│
  │◀─── SYN-ACK (seq=y, ──│
  │      ack=x+1)          │
  │──── ACK (ack=y+1) ────▶│
  │                        │
  │  ←── Data flows ──▶    │
```

### TCP 4-Way Teardown (FIN)

```
Client                    Server
  │──── FIN ──────────────▶│   (Client: done sending)
  │◀─── ACK ───────────────│
  │◀─── FIN ───────────────│   (Server: also done)
  │──── ACK ──────────────▶│
  │   (TIME_WAIT 2×MSL)    │
```

### TCP States (important for debugging)

| State | Meaning |
|---|---|
| `LISTEN` | Server waiting for connections |
| `SYN_SENT` | Client sent SYN, waiting |
| `ESTABLISHED` | Active connection |
| `CLOSE_WAIT` | Remote end closed; local app needs to close |
| `TIME_WAIT` | Waiting 2×MSL before fully closing (prevents stale packets) |
| `FIN_WAIT_1/2` | Initiated close, waiting for ack/FIN |

```bash
# View all TCP connections and states
ss -tan          # (modern, preferred)
netstat -an      # (classic)
```

### TCP Flow Control — Sliding Window

```
Sender is allowed to have up to <window size> bytes unacknowledged.
Receiver advertises its buffer size → sender adjusts.
Zero Window → sender pauses until receiver signals it's ready.
```

### TCP Congestion Control Algorithms

| Algorithm | Behavior | Used in |
|---|---|---|
| **CUBIC** | Cubic growth function | Linux (default) |
| **BBR** | Bandwidth-Delay Product based | Google, modern Linux |
| **RENO** | Additive increase, multiplicative decrease | Classic |
| **QUIC** | UDP-based, built-in TLS + multiplexing | HTTP/3 |

---

## 5 · HTTP / HTTPS & TLS

### HTTP Versions Comparison

| Version | Transport | Key Feature |
|---|---|---|
| **HTTP/1.0** | TCP | New connection per request |
| **HTTP/1.1** | TCP | Persistent connections, pipelining |
| **HTTP/2** | TCP + TLS | Multiplexing, header compression (HPACK), server push |
| **HTTP/3** | QUIC (UDP) | 0-RTT, no head-of-line blocking at transport layer |

### HTTP Request Structure

```
GET /api/users?page=1 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGc...
Accept: application/json
Content-Type: application/json
Connection: keep-alive

{ "filter": "active" }      ← request body (for POST/PUT/PATCH)
```

### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 128
Cache-Control: max-age=3600
X-Request-ID: abc-123

{ "users": [...] }          ← response body
```

### HTTP Methods

| Method | Idempotent | Safe | Use |
|---|---|---|---|
| `GET` | ✅ | ✅ | Read resource |
| `HEAD` | ✅ | ✅ | Read headers only |
| `OPTIONS` | ✅ | ✅ | CORS preflight, capability check |
| `POST` | ❌ | ❌ | Create resource / trigger action |
| `PUT` | ✅ | ❌ | Replace resource (full) |
| `PATCH` | ❌ | ❌ | Partial update |
| `DELETE` | ✅ | ❌ | Delete resource |

> **Idempotent**: Multiple identical requests have same effect as one.  
> **Safe**: No side effects on the server.

### HTTP Status Codes

| Range | Category | Key Codes |
|---|---|---|
| 1xx | Informational | `100 Continue`, `101 Switching Protocols` |
| 2xx | Success | `200 OK`, `201 Created`, `204 No Content` |
| 3xx | Redirection | `301 Moved Permanently`, `302 Found`, `304 Not Modified` |
| 4xx | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `408 Timeout`, `429 Too Many Requests` |
| 5xx | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` |

### Key HTTP Headers

```
Request headers:
  Authorization: Bearer <token>     # Auth credentials
  Accept: application/json          # Expected response format
  Content-Type: application/json    # Request body format
  If-None-Match: "etag-value"       # Conditional GET (caching)
  If-Modified-Since: <date>         # Conditional GET (caching)
  X-Request-ID: <uuid>              # Tracing
  X-Forwarded-For: <client-ip>      # Original IP behind proxy

Response headers:
  Cache-Control: max-age=3600, public  # Caching directives
  ETag: "abc123"                       # Resource version for caching
  Location: /api/users/42              # After 201/redirect
  Retry-After: 60                      # After 429/503
  Strict-Transport-Security: max-age=31536000; includeSubDomains  # HSTS
  Content-Security-Policy: default-src 'self'  # CSP
  X-Content-Type-Options: nosniff      # Security
  Access-Control-Allow-Origin: *       # CORS
```

### TLS Handshake (TLS 1.3)

```
Client                              Server
  │──── ClientHello ───────────────▶│  (TLS version, cipher suites, key share)
  │◀─── ServerHello + Certificate ──│  (chosen cipher, cert, key share)
  │◀─── {EncryptedExtensions}  ─────│
  │◀─── {Finished} ─────────────────│
  │──── {Finished} ────────────────▶│
  │                                  │
  │  ←── Application Data (encrypted) ──▶

TLS 1.3 advantages over 1.2:
  - 1-RTT handshake (vs 2-RTT)
  - 0-RTT resumption (with session tickets — risk: replay attacks)
  - Removed weak ciphers (RC4, 3DES, SHA-1)
  - Forward secrecy mandatory
```

### Certificates & PKI

```
CA (Certificate Authority)
  └── Intermediate CA
        └── Server Certificate
              ├── Subject: api.example.com
              ├── Public Key
              ├── Validity period
              ├── SANs (Subject Alternative Names)
              └── Signature by Intermediate CA

Chain of trust: browser trusts root CA → validates intermediate → validates server cert.

Certificate types:
  DV (Domain Validated)  — cheap, automated (Let's Encrypt)
  OV (Organization Validated) — identity verified
  EV (Extended Validation) — highest trust, legal entity verified
  Wildcard: *.example.com
  SAN: multiple hostnames in one cert
```

---

## 6 · Common Network Protocols

### Protocol Reference Table

| Protocol | Port(s) | Transport | Description |
|---|---|---|---|
| **HTTP** | 80 | TCP | Web traffic (unencrypted) |
| **HTTPS** | 443 | TCP | Web traffic over TLS |
| **HTTP/3** | 443 | UDP (QUIC) | Next-gen HTTP |
| **DNS** | 53 | UDP / TCP | Name resolution |
| **SSH** | 22 | TCP | Secure shell, tunnelling |
| **FTP** | 20 (data), 21 (control) | TCP | File transfer (unencrypted) |
| **SFTP** | 22 | TCP | Secure file transfer (over SSH) |
| **SMTP** | 25, 587, 465 | TCP | Email sending |
| **IMAP** | 143, 993 | TCP | Email retrieval (sync) |
| **POP3** | 110, 995 | TCP | Email retrieval (download) |
| **NTP** | 123 | UDP | Network time synchronisation |
| **DHCP** | 67 (server), 68 (client) | UDP | Dynamic IP assignment |
| **SNMP** | 161, 162 | UDP | Network device monitoring |
| **gRPC** | typically 443 | TCP (HTTP/2) | High-performance RPC |
| **AMQP** | 5672, 5671 | TCP | Message queuing (RabbitMQ) |
| **MQTT** | 1883, 8883 | TCP | IoT messaging |
| **Redis** | 6379 | TCP | In-memory data store |
| **PostgreSQL** | 5432 | TCP | Relational database |
| **MySQL** | 3306 | TCP | Relational database |
| **MongoDB** | 27017 | TCP | Document database |
| **Kafka** | 9092 | TCP | Distributed streaming |

### ICMP — Internet Control Message Protocol

```
Used for:
  ping   → Echo Request (type 8) / Echo Reply (type 0)
  traceroute → TTL exceeded (type 11), destination unreachable (type 3)

Important: ICMP operates at Layer 3. It is NOT TCP or UDP.
Firewalls often block ICMP — absence of ping response ≠ host is down.
```

### ARP — Address Resolution Protocol

```
Problem: I have an IP address, but I need the MAC address to send a frame.
Solution: Broadcast "Who has 192.168.1.5?"
          That host responds with its MAC address.
          Result is cached in the ARP table.

ip neigh show      # Linux — view ARP table
arp -a             # Classic command
```

---

## 7 · Sockets & Connections

### Socket Fundamentals

```python
# Socket = (source IP, source port, destination IP, destination port, protocol)
# This 5-tuple uniquely identifies a connection.

# TCP server (Python)
import socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(("0.0.0.0", 8080))
server.listen(128)      # backlog: pending connection queue size
conn, addr = server.accept()

# UDP socket
udp = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp.bind(("0.0.0.0", 9090))
data, addr = udp.recvfrom(4096)
```

### Port Ranges

| Range | Name | Use |
|---|---|---|
| 0 – 1023 | Well-Known Ports | System services (HTTP=80, SSH=22) |
| 1024 – 49151 | Registered Ports | Application ports (PostgreSQL=5432) |
| 49152 – 65535 | Ephemeral / Dynamic | OS-assigned client-side ports |

### Connection Pooling

```
Problem: TCP handshake + TLS handshake on every request is expensive.

Solution: Maintain a pool of pre-established connections.

Database connection pool (e.g., HikariCP, pgBouncer, go-sql):
  - min idle connections = 5
  - max pool size       = 20
  - connection timeout  = 30s
  - idle timeout        = 600s

HTTP connection pool (keep-alive):
  Connection: keep-alive            # Request header
  Keep-Alive: timeout=5, max=100    # Response header

Too large a pool → resource exhaustion on DB side.
Too small a pool → connection queue builds up, latency spikes.
```

### SO_REUSEPORT vs SO_REUSEADDR

| Option | Effect |
|---|---|
| `SO_REUSEADDR` | Allows binding to a port in TIME_WAIT state |
| `SO_REUSEPORT` | Allows multiple sockets on the same address:port (kernel distributes) |

---

## 8 · Load Balancing

### Types of Load Balancers

| Layer | Type | How it works | Examples |
|---|---|---|---|
| **L4** | Transport LB | Routes based on IP + TCP/UDP port, no payload inspection | AWS NLB, HAProxy (TCP) |
| **L7** | Application LB | Inspects HTTP headers, URL, cookies; content-aware routing | AWS ALB, Nginx, Envoy |
| **DNS** | Global LB | Returns different IPs based on geography/health | Route 53, Cloudflare |
| **Anycast** | Network LB | Same IP announced from multiple PoPs; BGP routes to nearest | CDN edge, DDoS mitigation |

### Load Balancing Algorithms

| Algorithm | How it works | Best for |
|---|---|---|
| **Round Robin** | Requests distributed in order | Homogeneous servers |
| **Weighted Round Robin** | Proportion based on weight | Heterogeneous capacity |
| **Least Connections** | Route to server with fewest active connections | Variable-duration requests |
| **IP Hash** | Hash client IP → same server | Session affinity (stateful) |
| **Consistent Hashing** | Hash ring; minimal disruption on scale | Caches, distributed stores |
| **Random** | Random selection | Simple, high-performance |
| **Resource Based** | Route based on CPU/memory metrics | Dynamic environments |

### Sticky Sessions (Session Affinity)

```
Problem: Stateful app needs same client to reach same server.

Solutions:
  1. Cookie-based affinity: LB injects a cookie (e.g., AWSALB) → routes same client.
  2. IP-hash: Same client IP always hits same backend.
  3. Better: Move state out of server (Redis, DB) → become stateless → no affinity needed.
```

### Health Checks

```yaml
# Nginx upstream health check
upstream api_servers {
  server 10.0.1.10:8080;
  server 10.0.1.11:8080;

  # Active health check (Nginx Plus)
  health_check interval=5s fails=3 passes=2 uri=/health;
}

# Passive: mark server down after N consecutive connection failures
# Active: LB probes backend on schedule
```

---

## 9 · Proxies & API Gateways

### Forward vs Reverse Proxy

```
Forward Proxy:                         Reverse Proxy:
  Client → [Forward Proxy] → Internet    Internet → [Reverse Proxy] → Servers
  
  Client is hidden from server.          Server is hidden from client.
  Use: corporate filtering, anonymity.   Use: load balancing, TLS termination, caching.
```

### Reverse Proxy Capabilities

| Feature | Description |
|---|---|
| **TLS termination** | Decrypt HTTPS at proxy; communicate in HTTP internally |
| **Load balancing** | Distribute to backend pool |
| **Caching** | Cache static / repeated responses |
| **Compression** | gzip / Brotli compression |
| **Rate limiting** | Throttle by IP or API key |
| **Request rewriting** | Modify path, headers before forwarding |
| **Authentication** | Validate tokens before reaching service |

### API Gateway vs Reverse Proxy

| Capability | Reverse Proxy | API Gateway |
|---|---|---|
| Routing | ✅ | ✅ |
| TLS termination | ✅ | ✅ |
| Auth (JWT, OAuth2) | ❌ / Limited | ✅ |
| Rate limiting | ✅ | ✅ |
| Request/response transformation | Limited | ✅ |
| API versioning | ❌ | ✅ |
| Developer portal / API catalog | ❌ | ✅ |
| Service discovery | ❌ | ✅ |
| Examples | Nginx, HAProxy, Envoy | Kong, AWS API GW, Apigee |

### Service Mesh (Sidecar Pattern)

```
Traditional:                    Service Mesh:
  Service A ──▶ Service B         Service A + Sidecar ──▶ Service B + Sidecar
                                      (Envoy)                   (Envoy)
                                          └──── Control Plane (Istio/Linkerd)

Sidecar handles:
  - mTLS between services
  - Retries, circuit breaking, timeouts
  - Distributed tracing
  - Traffic shaping (canary, A/B)
  - Observability metrics
```

---

## 10 · CDN — Content Delivery Network

### How a CDN Works

```
Without CDN:                        With CDN:
  User (Tokyo)                        User (Tokyo)
    └──▶ Origin server (US-East)        └──▶ CDN Edge PoP (Tokyo) ← cache HIT
          ~200ms latency                      └──▶ Origin (US-East) ← cache MISS only
                                                    ~200ms (once)
```

### CDN Architecture

```
Origin Server
    │
    ├──▶ PoP (Point of Presence) — New York
    ├──▶ PoP — London
    ├──▶ PoP — Singapore
    └──▶ PoP — Tokyo
         (Each PoP has edge cache + anycast routing)
```

### What CDNs Serve

| Content Type | Cacheable? | TTL Guidance |
|---|---|---|
| Static assets (JS, CSS, images) | ✅ | Long (1 year + cache busting) |
| API responses (GET, public data) | ✅ | Short (seconds to minutes) |
| HTML pages (SSG) | ✅ | Medium, with purge on deploy |
| Authenticated responses | ❌ | No (vary by user) |
| POST / mutation requests | ❌ | No |

### Cache-Control Directives

```
Cache-Control: public, max-age=86400      # CDN + browser can cache for 1 day
Cache-Control: private, max-age=3600      # Browser only (not CDN)
Cache-Control: no-cache                   # Must revalidate before using cache
Cache-Control: no-store                   # Never cache (sensitive data)
Cache-Control: s-maxage=600               # CDN TTL override (ignores max-age)
Cache-Control: stale-while-revalidate=60  # Serve stale while refreshing in background
```

### CDN Security Features

- **DDoS mitigation**: Absorb volumetric attacks at edge
- **WAF (Web Application Firewall)**: Block OWASP Top 10 at edge
- **Bot protection**: Challenge / block automated traffic
- **TLS at edge**: Certificate managed by CDN provider
- **IP allowlist/blocklist**: Geo-blocking

### Popular CDN Providers

| Provider | Notes |
|---|---|
| **Cloudflare** | CDN + DNS + DDoS + WAF, free tier |
| **AWS CloudFront** | Integrates with S3, ALB, Lambda@Edge |
| **Akamai** | Enterprise, largest PoP network |
| **Fastly** | Programmable edge (VCL/Wasm), real-time purge |
| **Azure CDN** | Integrates with Azure services |
| **Google Cloud CDN** | Integrates with Cloud Load Balancing |

---

## 11 · Network Security

### Common Attack Vectors

| Attack | Description | Mitigation |
|---|---|---|
| **DDoS** | Flood server with traffic to exhaust resources | CDN, rate limiting, anycast, scrubbing |
| **SYN Flood** | Exhaust TCP connection table with half-open connections | SYN cookies, rate limits, firewall |
| **Man-in-the-Middle** | Intercept communication between two parties | TLS, certificate pinning, HSTS |
| **DNS Spoofing / Cache Poisoning** | Inject false DNS records | DNSSEC, DoH/DoT |
| **BGP Hijacking** | Announce false routes to attract traffic | RPKI, BGPsec, monitoring |
| **ARP Spoofing** | Send fake ARP replies to redirect LAN traffic | Dynamic ARP Inspection, static ARP |
| **SQL/Command Injection via network** | Craft malicious payloads in requests | Input validation, WAF, parameterized queries |
| **Port Scanning** | Discover open services | Firewall, IDS, minimal port exposure |
| **SSL Stripping** | Downgrade HTTPS to HTTP | HSTS, HSTS preload list |

### Firewall Types

| Type | Operates at | Inspects | Example |
|---|---|---|---|
| **Packet Filter** | L3/L4 | IP, port, protocol | iptables, AWS Security Groups |
| **Stateful** | L3/L4 | Connection state + packets | Most modern firewalls |
| **Application (NGFW)** | L7 | Application payload, SSL | Palo Alto, Fortinet |
| **WAF** | L7 | HTTP requests (OWASP Top 10) | ModSecurity, AWS WAF, Cloudflare |

### Network Segmentation

```
Internet
    │
   [WAF / DDoS protection]
    │
   [DMZ — Public Subnet]
    ├── Load Balancer
    │
   [App Subnet — Private]
    ├── App Server 1
    ├── App Server 2
    │
   [Data Subnet — Private, isolated]
    ├── Database Primary
    └── Database Replica
    
Security Groups / NACLs control traffic between each tier.
```

### Zero Trust Networking

```
Principle: "Never trust, always verify"
  ✖ Don't trust based on network location (inside corporate network ≠ trusted)
  ✔ Authenticate every request (device identity + user identity)
  ✔ Least privilege access
  ✔ Micro-segmentation — no implicit east-west trust

Tools: BeyondCorp, Okta, Tailscale, Cloudflare Access, WireGuard
```

### VPN Types

| Type | Use Case | Protocol |
|---|---|---|
| **Site-to-Site** | Connect two networks (office ↔ cloud) | IPSec, OpenVPN |
| **Client-to-Site (Remote Access)** | Individual user → corporate network | OpenVPN, WireGuard, Cisco AnyConnect |
| **Split Tunnelling** | Only specific traffic goes through VPN | Config option |
| **WireGuard** | Modern, minimal, fast VPN | WireGuard (UDP) |

### mTLS — Mutual TLS

```
Standard TLS:   Server proves identity to client.
mTLS:           Both server AND client present certificates.

Use case: Service-to-service authentication in microservices / zero-trust.

Flow:
  Client ──▶ (presents client cert) ──▶ Server
  Server validates client cert against trusted CA.
  Client validates server cert.
  Encrypted channel established.
```

---

## 12 · WebSockets & Real-Time Protocols

### Comparison of Real-Time Techniques

| Technique | Protocol | Direction | Use Case | Overhead |
|---|---|---|---|---|
| **Short Polling** | HTTP | Client→Server | Simple, low-frequency updates | High (wasted requests) |
| **Long Polling** | HTTP | Client→Server | Moderate frequency | Medium |
| **SSE** (Server-Sent Events) | HTTP | Server→Client | Live feeds, notifications | Low |
| **WebSocket** | WS/WSS | Bidirectional | Chat, collaboration, gaming | Very low |
| **WebRTC** | DTLS/SRTP | Peer-to-Peer | Video/audio calls, P2P data | — |
| **gRPC Streaming** | HTTP/2 | Bidirectional | Internal services, streaming RPC | Low |

### WebSocket Handshake

```http
# 1. HTTP Upgrade Request
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# 2. Server Response (101 Switching Protocols)
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

# 3. Bidirectional binary frames now flow over the same TCP connection
```

```javascript
// WebSocket client (browser)
const ws = new WebSocket("wss://api.example.com/ws");

ws.onopen    = () => ws.send(JSON.stringify({ type: "subscribe", channel: "prices" }));
ws.onmessage = (e) => console.log(JSON.parse(e.data));
ws.onerror   = (e) => console.error("WS error", e);
ws.onclose   = (e) => console.log("Closed", e.code, e.reason);
```

### Server-Sent Events (SSE)

```javascript
// Server (Node.js / Express)
app.get("/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  const send = (data) => res.write(`data: ${JSON.stringify(data)}\n\n`);
  const timer = setInterval(() => send({ ts: Date.now() }), 1000);
  req.on("close", () => clearInterval(timer));
});

// Client (browser)
const es = new EventSource("/events");
es.onmessage = (e) => console.log(JSON.parse(e.data));
```

---

## 13 · Microservices Networking

### Service Discovery

```
Problem: In dynamic environments (k8s, ECS), service IPs change constantly.

Client-Side Discovery:
  Service A → queries Service Registry → gets IP list → load-balances itself
  (e.g., Eureka + Ribbon in Spring Cloud)

Server-Side Discovery:
  Service A → Load Balancer → queries registry → forwards request
  (e.g., AWS ALB + ECS Service Discovery, Kubernetes Service)

DNS-Based Discovery (most common in k8s):
  Service A calls "payment-service" → resolves via kube-dns → ClusterIP → Pod(s)
```

### Kubernetes Networking Model

```
Key rules:
  1. Every Pod gets its own IP (routable within cluster).
  2. Pods can communicate without NAT.
  3. Agents (nodes) can communicate with pods without NAT.

Kubernetes networking objects:
  Service (ClusterIP)   → stable virtual IP, kube-proxy routes to pods
  Service (NodePort)    → exposes on each node's IP at a port (30000-32767)
  Service (LoadBalancer)→ provisions cloud LB (AWS ELB, etc.)
  Service (Headless)    → returns pod IPs directly via DNS (no ClusterIP)
  Ingress               → L7 HTTP routing, TLS termination (Nginx, ALB, Traefik)
  NetworkPolicy         → firewall rules between pods (labels-based)
```

### Network Policies (Kubernetes)

```yaml
# Allow only app=frontend pods to reach app=backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### Circuit Breaker Pattern

```
States:
  CLOSED  → requests flow normally
  OPEN    → requests short-circuit immediately (fail fast)
  HALF-OPEN → probe requests allowed; success → CLOSED, failure → OPEN

Thresholds (example):
  failure rate > 50% in 10s window → OPEN
  wait 30s → HALF-OPEN
  3 success → CLOSED

Libraries: Resilience4j (Java), Hystrix (deprecated), Polly (.NET), go-circuit
```

### Retry & Timeout Strategy

```
Timeouts:
  Connection timeout: how long to wait to establish connection (e.g., 3s)
  Read timeout:       how long to wait for response data  (e.g., 30s)
  Write timeout:      how long to wait to send request    (e.g., 10s)

Retry with exponential backoff + jitter:
  attempt 1: wait 1s  ± jitter
  attempt 2: wait 2s  ± jitter
  attempt 3: wait 4s  ± jitter
  Max retries: 3–5 (avoid retry storms)

Only retry on:
  ✅ Network errors (ECONNRESET, ETIMEDOUT)
  ✅ 503 Service Unavailable
  ✅ 429 Too Many Requests (with Retry-After header)
  ❌ 400 Bad Request
  ❌ 404 Not Found
  ❌ Non-idempotent POST (risk of duplicate side effects)
```

---

## 14 · Cloud Networking

### VPC — Virtual Private Cloud

```
VPC: Your isolated virtual network in the cloud.

┌─────────────────────────────────────────────┐
│  VPC: 10.0.0.0/16                           │
│                                             │
│  ┌──────────────┐  ┌──────────────────────┐ │
│  │ Public Subnet│  │   Private Subnet     │ │
│  │ 10.0.1.0/24  │  │   10.0.2.0/24        │ │
│  │ (Internet GW)│  │   (NAT GW / no IGW)  │ │
│  │  Load Balancer│  │   App Servers        │ │
│  └──────────────┘  │   Databases          │ │
│                    └──────────────────────┘ │
└─────────────────────────────────────────────┘
```

### AWS Networking Components

| Component | Purpose |
|---|---|
| **VPC** | Isolated virtual network |
| **Subnet** | Sub-division of VPC (public or private) |
| **Internet Gateway (IGW)** | Allows VPC to communicate with internet |
| **NAT Gateway** | Allows private subnet → internet (outbound only) |
| **Route Table** | Rules for directing traffic (destination → target) |
| **Security Group** | Stateful instance-level firewall (allow rules only) |
| **NACL** | Stateless subnet-level firewall (allow + deny rules) |
| **VPC Peering** | Connect two VPCs (no transitive routing) |
| **Transit Gateway** | Hub-and-spoke VPC interconnect (transitive routing) |
| **VPC Endpoints** | Private connection to AWS services (no internet) |
| **PrivateLink** | Expose service privately across VPCs |
| **Direct Connect** | Dedicated physical link to AWS (not over internet) |
| **Route 53** | DNS + health checks + routing policies |
| **CloudFront** | CDN at AWS edge |
| **ELB (ALB/NLB)** | Managed load balancers |

### Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Applies to | EC2 instance / ENI | Subnet |
| State | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
| Rules | Allow only | Allow + Deny |
| Evaluation | All rules evaluated | Rules evaluated in order (lowest number first) |
| Default | Deny all inbound, allow all outbound | Allow all inbound + outbound |

### Egress & Ingress Control

```
Public subnet:  Route table → 0.0.0.0/0 → Internet Gateway (IGW)
Private subnet: Route table → 0.0.0.0/0 → NAT Gateway (outbound only)

NAT Gateway: Allows private resources to reach internet (updates, downloads).
             Internet cannot initiate connections back in (NAT is one-way).
```

---

## 15 · Network Performance & Observability

### Key Metrics

| Metric | Definition | Good | Bad |
|---|---|---|---|
| **Latency** | Time for a packet to travel from A to B | < 1ms LAN, < 100ms internet | > 200ms+ noticeable |
| **Throughput** | Data transferred per unit time (Mbps / Gbps) | Depends on link | Consistently below capacity |
| **Bandwidth** | Maximum capacity of the link | — | Throughput ≈ Bandwidth = saturated |
| **Packet Loss** | % of packets dropped | < 0.1% | > 1% impacts TCP severely |
| **Jitter** | Variation in latency | < 10ms for voice/video | > 30ms causes audio issues |
| **RTT** | Round-trip time | — | Baseline for diagnosis |
| **Connection Rate** | New connections per second | — | Server resource consideration |
| **Error Rate** | 5xx, TCP resets per minute | < 0.1% | > 1% requires investigation |

### Latency Budget (End-to-End)

```
Total budget: 500ms (example SLA)

  DNS lookup:           ~20ms  (cached: ~1ms)
  TCP handshake:        ~50ms  (1 RTT to server)
  TLS handshake:        ~50ms  (TLS 1.3: 1 RTT; 1.2: 2 RTT)
  Server processing:   ~200ms  (application logic, DB query)
  Network transit:      ~50ms  (geographic distance)
  Response transfer:    ~30ms  (payload size / bandwidth)
  Browser rendering:   ~100ms
                      ─────────
  Total:               ~500ms
```

### Observability Tools

```bash
# 1. ping — basic reachability + RTT
ping -c 5 google.com

# 2. traceroute — path + per-hop latency
traceroute google.com      # Linux
tracert google.com         # Windows

# 3. mtr — continuous traceroute with stats
mtr google.com

# 4. dig — DNS resolution details
dig api.example.com
dig +trace api.example.com          # Full resolution path
dig api.example.com @8.8.8.8        # Use specific resolver

# 5. curl — HTTP debugging
curl -v https://api.example.com/health
curl -w "@curl-format.txt" -o /dev/null -s https://api.example.com/

# curl timing format:
# time_namelookup:  %{time_namelookup}\n
# time_connect:     %{time_connect}\n
# time_appconnect:  %{time_appconnect}\n
# time_starttransfer: %{time_starttransfer}\n
# time_total:       %{time_total}\n

# 6. ss / netstat — socket states
ss -s             # socket statistics summary
ss -tan           # all TCP connections with state
ss -lnp           # listening sockets with process

# 7. tcpdump — packet capture
tcpdump -i eth0 port 443 -w capture.pcap
tcpdump -i any host 10.0.1.5

# 8. Wireshark — GUI packet analysis (open .pcap)

# 9. iperf3 — throughput testing
iperf3 -s                   # server
iperf3 -c <server-ip> -t 30 # client: 30s TCP test

# 10. nmap — port scanning
nmap -sV -p 1-1024 target.host
```

### TCP Tuning (Linux)

```bash
# View current TCP settings
sysctl net.ipv4.tcp_keepalive_time
sysctl net.core.somaxconn

# Common performance tunings (add to /etc/sysctl.conf):
net.core.somaxconn = 65535            # max listen() backlog
net.ipv4.tcp_max_syn_backlog = 65535  # SYN queue size
net.ipv4.tcp_tw_reuse = 1            # reuse TIME_WAIT sockets
net.ipv4.ip_local_port_range = 1024 65535  # ephemeral port range
net.core.rmem_max = 134217728        # TCP receive buffer max
net.core.wmem_max = 134217728        # TCP send buffer max
net.ipv4.tcp_congestion_control = bbr  # use BBR congestion control
```

---

## 16 · Diagnostics Toolkit

### Systematic Troubleshooting Flow

```
Symptom: "Service is unreachable / slow"

Step 1: Ping — can I reach the host at all?
  ✖ Fail → firewall, routing, host down
  ✔ Pass → continue

Step 2: DNS — does the hostname resolve correctly?
  dig hostname                 # check A record
  dig +trace hostname          # check full resolution chain

Step 3: Port reachability — is the service listening?
  nc -zv host 443             # check port open
  telnet host 443             # alternative

Step 4: HTTP response — is the service responding correctly?
  curl -v https://host/health
  → Check: status code, response time, headers

Step 5: Trace the path — where is latency introduced?
  mtr host                    # see per-hop loss & latency

Step 6: Look at logs & metrics
  → Application error logs
  → Network flow logs (VPC Flow Logs, etc.)
  → APM traces

Step 7: Packet capture if needed
  tcpdump -i eth0 host target
```

### Common Failure Patterns

| Symptom | Likely Cause | Investigation |
|---|---|---|
| `Connection refused` | Service not listening / port closed | `ss -lnp`, check app logs |
| `Connection timed out` | Firewall dropping packets / host unreachable | `traceroute`, check Security Groups/NACLs |
| `Connection reset` | Server closed unexpectedly / LB timeout | Check server logs, LB idle timeout |
| `NXDOMAIN` | DNS name does not exist | `dig hostname`, check DNS records |
| `SSL handshake failed` | Cert expired, wrong cert, cipher mismatch | `curl -v`, `openssl s_client -connect host:443` |
| High latency | Geographic distance, overloaded server, large payload | `mtr`, APM traces, profiling |
| Packet loss > 0% | Network congestion, faulty hardware | `mtr`, check link utilisation |
| 502 Bad Gateway | Backend down or not responding in time | Check app health, LB logs |
| 503 Service Unavailable | All backends unhealthy / traffic spike | Scale out, circuit breaker |
| 504 Gateway Timeout | Backend took too long | Check timeout config, slow queries |

### OpenSSL Debugging

```bash
# Inspect TLS certificate
openssl s_client -connect api.example.com:443

# Check cert expiry
echo | openssl s_client -connect api.example.com:443 2>/dev/null \
  | openssl x509 -noout -dates

# Test specific TLS version
openssl s_client -connect api.example.com:443 -tls1_3

# Verify cert chain
openssl verify -CAfile ca-bundle.crt server.crt
```

---

## 17 · Quick-Reference Cheatsheets

### Ports Cheatsheet

```
20/21  FTP          22   SSH         23   Telnet (insecure)
25     SMTP         53   DNS         67/68 DHCP
80     HTTP         110  POP3        123  NTP
143    IMAP         161  SNMP        389  LDAP
443    HTTPS        465  SMTPS       587  SMTP (submission)
636    LDAPS        993  IMAPS       995  POP3S
1433   MSSQL        1521 Oracle      3306 MySQL
5432   PostgreSQL   5672 RabbitMQ    6379 Redis
8080   HTTP-alt     8443 HTTPS-alt   9092 Kafka
27017  MongoDB
```

### HTTP Status Codes Cheatsheet

```
200 OK                201 Created           204 No Content
301 Moved Permanently 302 Found             304 Not Modified
400 Bad Request       401 Unauthorized      403 Forbidden
404 Not Found         405 Method Not Allowed 408 Request Timeout
409 Conflict          410 Gone              422 Unprocessable Entity
429 Too Many Requests 500 Internal Server Error
502 Bad Gateway       503 Service Unavailable 504 Gateway Timeout
```

### CIDR Quick Reference

```
/8   → 16,777,214 hosts    /9  → 8,388,606    /10 → 4,194,302
/16  →     65,534 hosts    /17 →    32,766     /18 →   16,382
/20  →      4,094 hosts    /22 →     1,022     /24 →     254
/25  →        126 hosts    /26 →        62     /27 →      30
/28  →         14 hosts    /29 →         6     /30 →       2
/32  →          1 host  (single host route)
```

### DNS Record Cheatsheet

```
A      → IPv4 address
AAAA   → IPv6 address
CNAME  → Alias (canonical name) — cannot coexist with other records at apex
ALIAS  → Like CNAME but allowed at apex (Route 53 ALIAS, Cloudflare CNAME flatten)
MX     → Mail exchange (+ priority)
TXT    → Text (SPF: v=spf1, DKIM, DMARC, domain verification)
NS     → Name server
SOA    → Start of Authority
PTR    → Reverse DNS (in-addr.arpa)
SRV    → Service (_service._proto.name TTL class SRV priority weight port target)
CAA    → Certificate Authority Authorization
```

### curl Timing Reference

```bash
# Create timing format file
cat > /tmp/curl-fmt.txt << 'EOF'
    namelookup:  %{time_namelookup}s
       connect:  %{time_connect}s
    appconnect:  %{time_appconnect}s
   pretransfer:  %{time_pretransfer}s
  starttransfer: %{time_starttransfer}s
               ──────────────────────
          total: %{time_total}s
EOF

curl -w "@/tmp/curl-fmt.txt" -o /dev/null -s https://api.example.com/
```

### OSI Layers Memory Aid

```
"Please Do Not Throw Sausage Pizza Away"
  Physical → Data Link → Network → Transport → Session → Presentation → Application
  
Or bottom-up: "All People Seem To Need Data Processing"
```

---

*Last updated: 2026-03 | Part of the [Tech Reference Guide](../README.md)*
