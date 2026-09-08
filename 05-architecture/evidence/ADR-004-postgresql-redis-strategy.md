# ADR-004 — PostgreSQL + Redis Data Strategy

| Field | Value |
|-------|-------|
| **ID** | ADR-004 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento requires a data storage strategy that supports:

- **Transactional integrity**: Payment processing, seat reservations must be ACID-compliant
- **Complex relationships**: Events, venues, seats, tickets, users with referential integrity
- **JSON flexibility**: Event layouts, user preferences, notification templates
- **Real-time performance**: Seat status updates, WebSocket session management
- **Audit trails**: Complete history of seat changes, payment transactions
- **Concurrent access**: Multiple users reserving seats simultaneously
- **Search capabilities**: Event filtering, user management
- **Caching requirements**: Session storage, frequently accessed data
- **Backup and recovery**: Financial data protection and compliance

The team has PostgreSQL experience and needs a solution that balances ACID compliance with performance.

---

## Evaluated alternatives

### Alternative A — PostgreSQL + Redis (CHOSEN)
**PostgreSQL as primary database, Redis for caching and sessions**
- **Pros:**
  - ACID compliance for financial transactions and reservations
  - Excellent JSON/JSONB support for flexible schemas (event layouts)
  - Strong referential integrity and foreign key constraints
  - Advanced indexing (B-tree, GIN, GiST) for complex queries
  - Team expertise and familiarity
  - Robust backup and point-in-time recovery
  - Excellent Spring Data JPA integration
  - PostgreSQL 17 performance improvements
  - Redis provides high-performance caching and session storage
  - Redis supports WebSocket session management and pub/sub
- **Cons:**
  - Two systems to maintain and monitor
  - Network calls for cache operations
  - Redis adds complexity for small team

### Alternative B — PostgreSQL Only
**Single database approach with PostgreSQL extensions**
- **Pros:**
  - Single system to maintain and backup
  - No network latency for cached data
  - PostgreSQL can handle moderate caching via connection pooling
  - Simpler deployment and monitoring
- **Cons:**
  - WebSocket session state would need database storage (less efficient)
  - No distributed caching capabilities
  - Higher load on primary database for frequent reads
  - Session storage in database affects performance

### Alternative C — MySQL + Redis
**MySQL as primary database with Redis caching**
- **Pros:**
  - Wide adoption and extensive documentation
  - Good JSON support in recent versions
  - Redis caching benefits
- **Cons:**
  - Team lacks MySQL expertise
  - Weaker JSON handling compared to PostgreSQL
  - Less advanced indexing capabilities
  - Migration effort from team's PostgreSQL knowledge

### Alternative D — MongoDB + Redis
**Document database with caching layer**
- **Pros:**
  - Natural fit for JSON-heavy data (event layouts)
  - Flexible schema evolution
  - Good performance for read-heavy workloads
- **Cons:**
  - Team has no MongoDB experience
  - Weaker ACID guarantees for multi-document transactions
  - Less suitable for complex relational data (users, payments)
  - Limited SQL tooling and reporting capabilities

### Alternative E — Microservice Databases
**Separate databases per bounded context**
- **Pros:**
  - Perfect service isolation
  - Technology diversity per service needs
  - Independent scaling
- **Cons:**
  - Complex distributed transactions
  - Data consistency challenges
  - Multiple database systems to maintain
  - Premature for 2-person team

---

## Decision

**Alternative A:** PostgreSQL 17 as primary database + Redis 7.x for caching and sessions

**PostgreSQL configuration:**
- **Primary database**: `tuevento_db` with all business data
- **Connection pooling**: HikariCP with optimized settings
- **JSON storage**: JSONB for event layouts, user preferences, notification templates
- **Indexing strategy**: GIN indexes for JSONB queries, B-tree for standard lookups
- **Audit logging**: Trigger-based audit tables for critical operations
- **Backup strategy**: Daily automated backups with point-in-time recovery

**Redis configuration:**
- **Session storage**: Spring Session with Redis for JWT refresh tokens
- **WebSocket sessions**: Redis pub/sub for real-time seat updates
- **Application cache**: Frequently accessed data (event details, user profiles)
- **Temporary data**: Seat reservations with TTL (15-minute expiration)

**Schema design principles:**
- **Single database**: All modules share `tuevento_db` with clear schema separation
- **Foreign key constraints**: Enforce referential integrity between modules
- **Audit tables**: Separate audit schema for compliance and debugging
- **JSON fields**: Use JSONB for semi-structured data (layouts, preferences)

---

## Consequences

**Positive:**
- **ACID compliance** ensures data integrity for payments and reservations
- **Team expertise** leverages existing PostgreSQL knowledge for faster development
- **JSON flexibility** supports complex event layouts and user preferences
- **Performance** Redis caching reduces database load and improves response times
- **Real-time support** Redis pub/sub enables efficient WebSocket scaling
- **Backup/Recovery** PostgreSQL provides enterprise-grade data protection
- **Query capabilities** Advanced PostgreSQL features support complex business queries
- **Spring integration** Excellent Spring Boot support for both technologies

**Negative:**
- **Complexity** Two systems require separate monitoring and maintenance
- **Network latency** Cache operations add network round-trips
- **Memory usage** Redis requires dedicated memory allocation
- **Consistency** Cache invalidation needs careful coordination

**Performance optimizations:**
- **Connection pooling** HikariCP with proper sizing for concurrent access
- **Query optimization** Use EXPLAIN ANALYZE for complex queries, add appropriate indexes
- **Cache strategy** Write-through caching for frequently read data
- **Redis clustering** Prepare for Redis Sentinel/Cluster when scaling requirements grow

**Migration and deployment:**
- **Development** Docker Compose with PostgreSQL and Redis containers
- **Production** Managed PostgreSQL (cloud) + Redis instance with backup strategies
- **Monitoring** PostgreSQL slow query log, Redis monitoring via Spring Actuator

**Schema evolution:**
- **Database migrations** Flyway for versioned schema changes
- **Backward compatibility** Careful planning for schema changes affecting JSON fields
- **Module separation** Clear table prefixes and schemas for each bounded context

---

## References

- Database schema design → `06-data/models.md`
- Real-time seat management → `ADR-007-websocket-sockjs.md`
- JWT session management → `ADR-005-jwt-refresh-token-auth.md`
- Deployment configuration → `TuEvento/README.md#getting-started`
- Module data models → `02-domain/evidence/entities-and-rules.md`