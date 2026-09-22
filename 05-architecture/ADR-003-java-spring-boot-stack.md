# ADR-003 — Java 23 + Spring Boot Technology Stack

| Field | Value |
|-------|-------|
| **ID** | ADR-003 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento requires a robust backend technology stack that can handle:

- **Real-time features**: WebSocket connections for seat updates
- **External integrations**: Wompi payment gateway, SMTP, OAuth providers  
- **Database operations**: Complex queries, transactions, audit logging
- **Multi-platform APIs**: REST APIs for web and mobile clients
- **Security requirements**: JWT authentication, role-based authorization
- **Performance needs**: Handle concurrent seat reservations, payment processing
- **Team constraints**: 2 developers with Java/Spring experience
- **Development velocity**: Rapid prototyping and iterative development

The technology stack must balance development speed, maintainability, performance, and team expertise.

---

## Evaluated alternatives

### Alternative A — Java 23 + Spring Boot 3.x (CHOSEN)
- **Pros:**
  - Team has strong Java expertise and Spring Boot experience
  - Mature ecosystem with extensive libraries (Security, Data, WebSocket, etc.)
  - Excellent IDE support and debugging tools
  - Built-in support for all required features (REST, WebSocket, JPA, Security)
  - Strong community and documentation
  - Production-ready with battle-tested patterns
  - Spring Boot Actuator for monitoring and health checks
  - Easy integration with PostgreSQL and Redis
  - Maven ecosystem familiar to team
- **Cons:**
  - Verbose compared to newer languages
  - JVM startup time (mitigated by GraalVM in future)
  - Memory footprint higher than compiled languages

### Alternative B — Node.js + Express/NestJS
- **Pros:**
  - JavaScript everywhere (frontend + backend)
  - Fast development for APIs and real-time features
  - Excellent WebSocket support
  - Large npm ecosystem
  - Good performance for I/O-intensive operations
- **Cons:**
  - Team lacks Node.js experience
  - Single-threaded nature challenging for CPU-intensive operations
  - Weaker typing support (even with TypeScript)
  - Less mature ecosystem for enterprise features (security, transactions)
  - Payment integration libraries less robust

### Alternative C — Python + Django/FastAPI
- **Pros:**
  - Clean, readable code
  - Rapid development capabilities
  - Good libraries for data processing and integrations
  - Strong ORM support
- **Cons:**
  - Team has limited Python experience
  - Performance concerns for real-time features
  - GIL limitations for concurrent operations
  - Weaker ecosystem for enterprise payment processing

### Alternative D — C# + .NET Core
- **Pros:**
  - Excellent performance and tooling
  - Strong typing system
  - Good WebSocket and real-time support
  - Mature ecosystem for enterprise applications
- **Cons:**
  - Team has no C# experience
  - Less familiar deployment ecosystem
  - Windows-centric community (though .NET Core is cross-platform)

### Alternative E — Go + Gin/Echo
- **Pros:**
  - Excellent performance and concurrency
  - Fast startup times
  - Good for microservices
  - Strong standard library
- **Cons:**
  - Team has no Go experience
  - Less mature ecosystem for complex business applications
  - Limited ORM options
  - Steeper learning curve for domain-driven design

---

## Decision

**Alternative A:** Java 23 + Spring Boot 3.x

**Core framework stack:**
- **Java 23** — Latest LTS with modern language features
- **Spring Boot 3.2+** — Application framework and auto-configuration
- **Spring Security** — Authentication, authorization, JWT handling
- **Spring Data JPA** — Database access and ORM
- **Spring WebSocket** — Real-time communication with SockJS fallback
- **Spring Web** — REST API development
- **Maven 3.9+** — Dependency management and build tool

**Key libraries:**
- **PostgreSQL Driver** — Primary database connectivity
- **Redis Client (Lettuce)** — Caching and session storage  
- **Jackson** — JSON serialization/deserialization
- **Bean Validation (Hibernate Validator)** — Input validation
- **SpringDoc OpenAPI** — API documentation generation
- **Resilience4j** — Circuit breaker for external services
- **Logback** — Structured logging with JSON output

**Development tools:**
- **Spring Boot DevTools** — Hot reloading in development
- **Spring Boot Actuator** — Health checks and monitoring
- **JUnit 5** — Unit and integration testing
- **Testcontainers** — Database integration testing
- **Mockito** — Mocking framework for tests

---

## Consequences

**Positive:**
- **Team velocity** — Leverages existing Java/Spring expertise for faster development
- **Feature completeness** — Spring ecosystem covers all requirements out-of-the-box
- **Production readiness** — Battle-tested framework with proven scalability
- **Maintainability** — Strong typing, excellent IDE support, comprehensive tooling
- **Integration capabilities** — Mature libraries for all external services (Wompi, SMTP, OAuth)
- **Security** — Spring Security provides enterprise-grade authentication/authorization
- **Real-time support** — Spring WebSocket with SockJS handles cross-browser compatibility
- **Monitoring** — Built-in health checks, metrics, and observability features

**Negative:**
- **Resource usage** — Higher memory footprint than lightweight alternatives
- **Startup time** — Slower application startup compared to compiled languages
- **Verbosity** — More boilerplate code compared to dynamic languages

**Risk mitigation:**
- **Memory optimization** — Use Spring Boot's layered JARs and optimize JVM settings
- **Development speed** — Use Spring Boot starters and auto-configuration to minimize setup
- **Code generation** — Leverage IDE templates and Spring Boot CLI for common patterns
- **Future optimization** — Consider GraalVM native compilation for production if startup time becomes critical

**Technology decisions:**
- **Java 23** chosen over Java 17 LTS for latest language improvements (pattern matching, virtual threads preparation)
- **Spring Boot 3.x** for modern Spring features and GraalVM compatibility
- **Maven** over Gradle for team familiarity and simpler configuration
- **SLF4J + Logback** over other logging frameworks for better JSON structured logging

---

## References

- Backend project structure → `05-architecture/evidence/overview.md#module-catalog`
- Database and caching strategy → `ADR-004-postgresql-redis-strategy.md`
- Security implementation → `ADR-005-jwt-refresh-token-auth.md`
- Real-time communication → `ADR-007-websocket-sockjs.md`
- Development environment setup → `TuEvento/README.md`