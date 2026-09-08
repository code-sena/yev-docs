# ADR-002 — N-Layer DDD Architecture

| Field | Value |
|-------|-------|
| **ID** | ADR-002 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento is an event management platform with complex business logic around seat reservations, real-time updates, payment processing, and multi-user coordination. The system needs to handle:

- **Complex domain logic**: Event creation, seat layouts, reservations, tickets, payments
- **Real-time requirements**: Seat status updates via WebSockets
- **Multiple bounded contexts**: Security, Events, Payments, Notifications each with distinct rules
- **External integrations**: Wompi payments, SMTP notifications, OAuth providers
- **Multi-platform access**: Web organizers, mobile/web attendees
- **Small team size**: 2 developers need maintainable, well-structured code

We need an architecture that balances domain complexity with development velocity and maintainability.

---

## Evaluated alternatives

### Alternative A — N-Layer with Domain-Driven Design (CHOSEN)
**Structure:** Each module follows domain/application/infrastructure/interfaces layers within a Spring Boot monolith
- **Pros:** 
  - Clear separation of concerns with domain isolation
  - Bounded contexts map naturally to modules
  - Testable business logic independent of frameworks
  - Supports future microservices evolution
  - Familiar to team, good Spring Boot ecosystem support
  - Single deployment simplifies DevOps for small team
- **Cons:** 
  - More complex initial setup than simple layered architecture
  - Requires discipline to maintain layer boundaries
  - Potential for over-engineering simple CRUD operations

### Alternative B — Simple 3-Layer Architecture (MVC)
**Structure:** Controller → Service → Repository pattern across the entire application
- **Pros:** 
  - Simplest to implement and understand
  - Fast initial development
  - Standard Spring Boot approach
- **Cons:** 
  - Business logic scattered across service classes
  - Difficult to maintain clear bounded context boundaries
  - Hard to test complex domain interactions
  - Doesn't scale well with domain complexity

### Alternative C — Microservices from Day 1
**Structure:** Separate Spring Boot applications per bounded context
- **Pros:** 
  - Maximum service isolation
  - Independent deployments
  - Technology diversity possible
- **Cons:** 
  - Significant overhead for 2-person team
  - Complex distributed system debugging
  - Network latency for operations spanning contexts
  - Premature optimization for current team size

### Alternative D — Event Sourcing + CQRS
**Structure:** Event-driven architecture with separate read/write models
- **Pros:** 
  - Perfect audit trail for financial transactions
  - Natural fit for seat reservation events
  - Excellent for complex business workflows
- **Cons:** 
  - High complexity for team experience level
  - Steep learning curve
  - Overkill for current requirements
  - Added infrastructure complexity

---

## Decision

**Alternative A:** N-Layer Architecture with Domain-Driven Design

Each module follows hexagonal architecture with these layers:

```
modules/[module]/
├── domain/
│   ├── model/           # Entities, Value Objects, Domain Events
│   ├── repository/      # Repository interfaces
│   └── service/         # Domain Services
├── application/
│   ├── usecase/         # Use Cases (Application Services)
│   ├── dto/             # Request/Response DTOs
│   ├── port/            # Input/Output ports
│   └── mapper/          # DTO ↔ Domain mappers
├── infrastructure/
│   ├── persistence/     # JPA entities, Repository implementations
│   ├── messaging/       # Event publishing/handling
│   └── external/        # External service clients
└── interfaces/
    ├── rest/           # REST Controllers
    └── websocket/      # WebSocket endpoints (where needed)
```

**Module boundaries based on bounded contexts:**
- Security (Identity & Access)
- Event (Event Management) 
- Seat (Seat Management + Real-time)
- Payment (Payment Processing)
- Ticket (Ticketing System)
- Notification (Communication)
- Profile (User Management)

---

## Consequences

**Positive:**
- **Clear domain boundaries** enable focused development and testing
- **Business logic isolation** makes complex rules maintainable and testable
- **Future microservices path** — modules can be extracted when team grows
- **Technology flexibility** — each module can adopt different patterns as needed
- **Parallel development** — developers can work on different modules with minimal conflicts
- **Testability** — domain logic can be unit tested without Spring context

**Negative:**
- **Initial setup overhead** — more directories and interfaces than simple MVC
- **Learning curve** — team needs to understand DDD concepts and layer responsibilities
- **Potential over-engineering** — simple CRUD modules still need full layer structure

**Mitigation strategies:**
- Create module templates and generator scripts to reduce boilerplate
- Document clear guidelines for what goes in each layer
- Start with simpler modules (Profile, Category) to build team familiarity
- Use shared infrastructure components to avoid duplication

**Migration approach:**
- **Phase 1:** Implement core modules (Security, Event, Seat) with full N-Layer structure
- **Phase 2:** Add remaining modules following established patterns  
- **Phase 3:** Extract to microservices when team > 5 developers or scaling requirements demand it

---

## References

- Hexagonal Architecture implementation → `05-architecture/hexagonal-architecture.md`
- Domain model and bounded contexts → `02-domain/evidence/domain-map.md`
- Module catalog and responsibilities → `05-architecture/evidence/overview.md`
- Future microservices strategy → `05-architecture/evidence/overview.md#planned-evolution`