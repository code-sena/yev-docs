# ADR-007 — WebSocket Real-time Communication

| Field | Value |
|-------|-------|
| **ID** | ADR-007 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento requires real-time seat reservation updates to prevent conflicts when multiple users select seats simultaneously. The system needs:

- **Immediate feedback**: Seat status changes visible to all users within 100ms
- **Cross-browser compatibility**: Support for older browsers and network restrictions  
- **Mobile support**: Reliable connections on mobile networks with intermittent connectivity
- **Scalability**: Handle multiple concurrent events with hundreds of users per event
- **Connection management**: Graceful handling of connection drops and reconnections
- **Security**: Authenticated WebSocket connections with proper authorization
- **Fallback support**: Degraded functionality when WebSocket unavailable

The core use case is seat selection where users must see real-time updates of seat availability to avoid booking conflicts.

---

## Evaluated alternatives

### Alternative A — WebSocket with SockJS + STOMP (CHOSEN)
**Spring WebSocket with SockJS fallback and STOMP messaging protocol**
- **Pros:**
  - Cross-browser compatibility with automatic fallback (WebSocket → SSE → polling)
  - Spring WebSocket provides seamless integration with Spring Security
  - STOMP protocol supports topic subscriptions (per-event channels)
  - SockJS handles connection recovery and network issues automatically
  - Message acknowledgment and reliability features
  - Built-in support for authentication and session management
  - Redis pub/sub integration for horizontal scaling
- **Cons:**
  - More complex than raw WebSockets
  - Additional protocol overhead with STOMP framing
  - SockJS fallbacks have higher latency than pure WebSocket

### Alternative B — Native WebSocket API
**Raw WebSocket implementation without frameworks**
- **Pros:**
  - Lowest latency and overhead
  - Full control over connection and message handling
  - No additional dependencies
- **Cons:**
  - No fallback for incompatible browsers or networks
  - Manual connection recovery and heartbeat implementation
  - Complex integration with Spring Security
  - Mobile network reliability issues
  - Custom protocol design and testing required

### Alternative C — Server-Sent Events (SSE)
**One-way server-to-client communication with EventSource**
- **Pros:**
  - Simpler than WebSocket, HTTP-based
  - Good browser support and automatic reconnection
  - Works through proxies and firewalls
- **Cons:**
  - One-way communication only (server → client)
  - Requires separate endpoint for client actions
  - Less efficient for bidirectional communication
  - Limited message types compared to WebSocket

### Alternative D — HTTP Polling
**Regular AJAX requests to check for seat updates**
- **Pros:**
  - Simple implementation with standard HTTP
  - No connection management complexity
  - Works in any network environment
- **Cons:**
  - High latency (polling interval vs real-time)
  - Increased server load and bandwidth usage
  - Poor user experience for real-time interactions
  - Battery drain on mobile devices

### Alternative E — Long Polling
**HTTP requests held open until updates are available**
- **Pros:**
  - Lower latency than regular polling
  - HTTP-based, works through firewalls
  - Automatic fallback behavior
- **Cons:**
  - Connection timeout handling complexity
  - Higher server resource usage (held connections)
  - Less efficient than WebSocket for frequent updates
  - Proxy and load balancer complications

---

## Decision

**Alternative A:** WebSocket with SockJS + STOMP protocol

**Technical implementation:**
- **Primary**: WebSocket for modern browsers with good connectivity
- **Fallback 1**: Server-Sent Events for browsers without WebSocket support
- **Fallback 2**: XHR streaming for older browsers
- **Fallback 3**: XHR polling as last resort

**Message protocol:**
```javascript
// Subscribe to event seat updates
CONNECT /ws/seats
SUBSCRIBE /topic/events/{eventId}/seats

// Seat status update message
{
  "type": "SEAT_STATUS_CHANGED",
  "eventId": "uuid",
  "seatId": "uuid", 
  "status": "RESERVED|AVAILABLE|SOLD",
  "userId": "uuid",
  "timestamp": "2024-09-08T10:30:00Z",
  "expiresAt": "2024-09-08T10:45:00Z" // for reservations
}

// Heartbeat for connection monitoring
{
  "type": "HEARTBEAT",
  "timestamp": "2024-09-08T10:30:00Z"
}
```

**Security model:**
- **Authentication**: JWT token validation for WebSocket handshake
- **Authorization**: Users can only subscribe to events they have access to
- **Rate limiting**: Maximum message rate per connection to prevent abuse
- **Connection limits**: Maximum concurrent connections per user

**Scaling strategy:**
- **Redis pub/sub**: Distribute messages across multiple application instances
- **Event-specific topics**: Users subscribe only to relevant event updates
- **Connection management**: Track active connections per event in Redis
- **Load balancing**: Sticky sessions for WebSocket connections

---

## Consequences

**Positive:**
- **Real-time experience**: Sub-100ms seat status updates provide immediate feedback
- **Cross-browser support**: SockJS ensures functionality across all target browsers
- **Mobile reliability**: Automatic reconnection handles mobile network issues
- **Scalable architecture**: Redis pub/sub enables horizontal scaling
- **Security integration**: Seamless authentication with existing JWT system  
- **Developer experience**: Spring WebSocket provides familiar programming model
- **Graceful degradation**: Fallback mechanisms ensure basic functionality always works

**Negative:**
- **Complexity**: More complex than simple HTTP polling solutions
- **Resource usage**: WebSocket connections consume server memory
- **Network requirements**: Some corporate firewalls may block WebSocket connections
- **Testing challenges**: Real-time features require specialized testing approaches

**Performance characteristics:**
- **Latency**: < 100ms for seat status updates under normal conditions
- **Throughput**: Support 500+ concurrent connections per application instance
- **Reliability**: 99.9% message delivery with SockJS fallbacks
- **Bandwidth**: ~50% reduction compared to polling equivalent

**Implementation phases:**
- **Phase 1**: Basic WebSocket connection with seat status broadcasts
- **Phase 2**: Authentication and per-event topic subscriptions  
- **Phase 3**: Redis pub/sub for multi-instance deployment
- **Phase 4**: Advanced features (connection monitoring, rate limiting)

**Monitoring and diagnostics:**
- **Connection metrics**: Active connections, connection duration, failure rates
- **Message metrics**: Messages per second, delivery success rates
- **Performance monitoring**: Message latency, memory usage per connection
- **Error tracking**: Connection failures, authentication failures, message delivery issues

**Fallback behavior:**
- **WebSocket unavailable**: Automatic SockJS fallback to SSE or polling
- **Authentication failure**: Connection rejected with clear error message
- **Server overload**: Graceful connection limiting with user notification
- **Network issues**: Client-side reconnection with exponential backoff

---

## References

- Seat management real-time requirements → `04-requirements/evidence/functional.md#FR-024-026`
- WebSocket security configuration → `modules/seat/interfaces/websocket/`
- Redis pub/sub setup → `ADR-004-postgresql-redis-strategy.md`
- Authentication integration → `ADR-005-jwt-refresh-token-auth.md`
- Load testing strategy → `12-testing/performance-tests.md`