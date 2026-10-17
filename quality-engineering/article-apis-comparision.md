# API Communication Styles

REST and WebSocket are two popular API communication approaches, but there are several others used in modern software development.

## 1. REST API

- Most common API architecture
- HTTP-based
- Request/response model
- Easy to understand and implement

### Example

```http
GET /users/1
```

### Best For

- CRUD operations
- Web applications
- Mobile backends
- Public APIs

---

## 2. WebSocket API

- Persistent connection
- Real-time bidirectional communication
- Lower overhead after connection establishment

### Example

```javascript
const socket = new WebSocket("wss://example.com");
socket.send("Hello Server");
```

### Best For

- Chat applications
- Online gaming
- Live dashboards
- Collaboration tools

---

## 3. GraphQL

Developed by Meta (Facebook).

Instead of multiple REST endpoints, clients request exactly the data they need.

### Example

```graphql
query {
  user(id: 1) {
    name
    email
  }
}
```

### Advantages

- Avoids over-fetching
- Avoids under-fetching
- Flexible for frontend applications

### Used By

- GitHub
- Shopify

---

## 4. gRPC

Created by Google.

Uses:

- Protocol Buffers (Protobuf)
- HTTP/2

### Example

```proto
service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}
```

### Advantages

- Extremely fast
- Compact payloads
- Strongly typed

### Best For

- Microservices
- Cloud-native applications
- Internal service communication

---

## 5. SOAP

An older enterprise API protocol.

Uses XML-based messages.

### Example

```xml
<GetUserRequest>
   <UserId>1</UserId>
</GetUserRequest>
```

### Advantages

- Strong standards
- Built-in security specifications
- Transaction support

### Common Use Cases

- Banking
- Government systems
- Enterprise software

---

## 6. Server-Sent Events (SSE)

Allows servers to push updates to clients over HTTP.

### Example

```javascript
const source = new EventSource('/events');
```

### Advantages

- Simpler than WebSockets
- Excellent for one-way streaming

### Best For

- Notifications
- News feeds
- Live score updates

---

## 7. Webhooks

An event-driven integration mechanism.

Instead of applications continuously checking for updates, a server sends data when an event occurs.

### Example Flow

```text
Stripe ---> Your Application
```

### Common Events

- Payment completed
- Order created
- GitHub push event

### Best For

- Third-party integrations
- Event-driven workflows
- Automation

---

# Quick Comparison

| Type | Real-Time | Direction | Best For |
|--------|----------|-----------|----------|
| REST | No | Request → Response | CRUD operations |
| WebSocket | Yes | Two-way | Chat, gaming |
| GraphQL | No | Request → Response | Flexible data fetching |
| gRPC | Yes/No | RPC | High-performance microservices |
| SOAP | No | Request → Response | Enterprise apps |
| SSE | Yes | Server → Client | Live notifications |
| Webhooks | Event-based | Server → Server | Integrations |

---

# Modern Architecture Example

Many applications combine multiple API styles:

- REST or GraphQL for standard operations
- WebSockets for real-time updates
- Webhooks for external integrations
- gRPC for internal microservice communication

## Rule of Thumb

- Use **REST** for standard business operations.
- Use **GraphQL** when clients need flexible data fetching.
- Use **WebSockets** for real-time interactions.
- Use **SSE** for one-way server updates.
- Use **Webhooks** for event-driven integrations.
- Use **gRPC** for high-performance service-to-service communication.
- Use **SOAP** when working with legacy enterprise systems.