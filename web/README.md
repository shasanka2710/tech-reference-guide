# 🌐 Web Technologies Quick Reference

## HTTP

### HTTP Methods

| Method  | Purpose                        | Idempotent | Safe |
|---------|--------------------------------|-----------|------|
| GET     | Retrieve resource              | Yes       | Yes  |
| HEAD    | Same as GET but no body        | Yes       | Yes  |
| POST    | Create resource                | No        | No   |
| PUT     | Replace resource entirely      | Yes       | No   |
| PATCH   | Partial update                 | No*       | No   |
| DELETE  | Remove resource                | Yes       | No   |
| OPTIONS | Describe supported methods     | Yes       | Yes  |

### HTTP Status Codes

```
1xx Informational
  100 Continue
  101 Switching Protocols

2xx Success
  200 OK
  201 Created
  202 Accepted
  204 No Content
  206 Partial Content

3xx Redirection
  301 Moved Permanently
  302 Found (temporary redirect)
  304 Not Modified (use cache)
  307 Temporary Redirect (preserve method)
  308 Permanent Redirect (preserve method)

4xx Client Error
  400 Bad Request
  401 Unauthorized (not authenticated)
  403 Forbidden (authenticated but not authorized)
  404 Not Found
  405 Method Not Allowed
  408 Request Timeout
  409 Conflict
  410 Gone
  422 Unprocessable Entity (validation error)
  429 Too Many Requests (rate limited)

5xx Server Error
  500 Internal Server Error
  502 Bad Gateway
  503 Service Unavailable
  504 Gateway Timeout
```

### Common HTTP Headers

```
Request headers:
  Authorization: Bearer <token>
  Content-Type:  application/json
  Accept:        application/json
  Accept-Encoding: gzip, deflate, br
  User-Agent:    Mozilla/5.0 ...
  Cookie:        session_id=abc123
  Cache-Control: no-cache
  If-None-Match: "etag-value"
  Origin:        https://example.com
  X-Request-ID:  uuid

Response headers:
  Content-Type:         application/json; charset=utf-8
  Content-Length:       1234
  Cache-Control:        max-age=3600, public
  ETag:                 "abc123"
  Last-Modified:        Wed, 21 Oct 2024 07:28:00 GMT
  Set-Cookie:           session_id=abc; HttpOnly; Secure; SameSite=Strict
  Location:             /api/users/123   (on 201 Created / 301)
  WWW-Authenticate:     Bearer
  X-RateLimit-Limit:    100
  X-RateLimit-Remaining: 99
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  Access-Control-Allow-Origin: https://example.com
```

## REST API Design

### URL Conventions

```
Resources (nouns, plural):
  GET    /users              list all users
  POST   /users              create user
  GET    /users/{id}         get user by id
  PUT    /users/{id}         replace user
  PATCH  /users/{id}         partial update user
  DELETE /users/{id}         delete user

Nested resources:
  GET    /users/{id}/orders         user's orders
  POST   /users/{id}/orders         create order for user
  GET    /users/{id}/orders/{oid}   specific order

Query parameters (filtering, sorting, pagination):
  GET /users?role=admin&status=active
  GET /users?sort=name&order=asc
  GET /users?page=2&limit=20
  GET /users?fields=id,name,email      (sparse fieldsets)
  GET /users?search=alice              (search)
```

### Pagination Strategies

```
Offset-based (simple):
  GET /users?page=3&limit=20
  Response: { data: [...], total: 100, page: 3, limit: 20 }
  - Simple to implement
  - Inconsistent if data changes during pagination

Cursor-based (efficient for large datasets):
  GET /users?cursor=abc123&limit=20
  Response: { data: [...], nextCursor: "xyz456", hasMore: true }
  - Consistent even with inserts/deletes
  - Cannot jump to arbitrary page

Keyset pagination:
  GET /users?after_id=100&limit=20
  - Uses last ID from previous page as anchor
```

### REST API Versioning

```
URL versioning (most common):
  /api/v1/users
  /api/v2/users

Header versioning:
  Accept: application/vnd.myapi.v2+json

Query parameter:
  /api/users?version=2
```

### Response Structure

```json
// Success
{
  "data": {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// Error
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// Paginated list
{
  "data": [...],
  "pagination": {
    "total": 100,
    "page": 2,
    "limit": 20,
    "totalPages": 5
  }
}
```

## Authentication & Authorization

```
Session-based:
  1. User logs in, server creates session
  2. Session ID stored in cookie
  3. Server looks up session on each request
  - Server-side state, harder to scale

JWT (JSON Web Token):
  Header.Payload.Signature
  - Stateless (server doesn't store)
  - Self-contained (claims in token)
  - Can be invalidated only via blocklist or short expiry

OAuth 2.0 Flows:
  Authorization Code:  web/mobile apps (most common)
  Client Credentials:  service-to-service (no user)
  Implicit:            deprecated, don't use
  Device Code:         TVs, CLI tools

PKCE (Proof Key for Code Exchange):
  - Extension to Authorization Code flow
  - Protects against authorization code interception
  - Required for public clients (SPAs, mobile)
```

## GraphQL

```graphql
# Schema definition
type User {
  id:      ID!
  name:    String!
  email:   String!
  posts:   [Post!]!
}

type Post {
  id:      ID!
  title:   String!
  body:    String
  author:  User!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

type Subscription {
  newPost: Post!
}

input CreateUserInput {
  name:  String!
  email: String!
}
```

```graphql
# Query
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    posts {
      title
    }
  }
}

# Mutation
mutation CreateUser {
  createUser(input: { name: "Alice", email: "alice@example.com" }) {
    id
    name
  }
}

# Fragment (reusable selection sets)
fragment UserFields on User {
  id
  name
  email
}
```

## WebSockets

```js
// Client
const ws = new WebSocket("wss://example.com/ws");

ws.onopen    = () => ws.send(JSON.stringify({ type: "subscribe", channel: "chat" }));
ws.onmessage = (event) => console.log(JSON.parse(event.data));
ws.onclose   = (event) => console.log("Closed:", event.code, event.reason);
ws.onerror   = (error) => console.error("Error:", error);
ws.close();  // clean close

// Server (Node.js with 'ws' library)
const { WebSocketServer } = require("ws");
const wss = new WebSocketServer({ port: 8080 });

wss.on("connection", (ws) => {
    ws.on("message", (data) => {
        const msg = JSON.parse(data);
        // Broadcast to all clients
        wss.clients.forEach((client) => {
            if (client.readyState === WebSocket.OPEN) {
                client.send(JSON.stringify(msg));
            }
        });
    });
    ws.on("close", () => console.log("Client disconnected"));
});
```

## CORS (Cross-Origin Resource Sharing)

```
Origin = scheme + host + port

Simple request (no preflight):
  - Methods: GET, POST, HEAD
  - Headers: Accept, Content-Type (text/plain, application/x-www-form-urlencoded, multipart/form-data)
  - Response must include: Access-Control-Allow-Origin

Preflight (OPTIONS request):
  - Triggered by: non-simple methods (PUT, DELETE, PATCH) or custom headers
  - Browser sends OPTIONS first, checks permissions, then actual request

Server response headers:
  Access-Control-Allow-Origin:  https://example.com   (or * for all)
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, Authorization
  Access-Control-Allow-Credentials: true  (allows cookies/auth)
  Access-Control-Max-Age:       86400     (preflight cache)
```

## Security Headers

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy:   default-src 'self'; script-src 'self' cdn.example.com
X-Frame-Options:           DENY
X-Content-Type-Options:    nosniff
Referrer-Policy:           strict-origin-when-cross-origin
Permissions-Policy:        camera=(), microphone=(), geolocation=()
```
