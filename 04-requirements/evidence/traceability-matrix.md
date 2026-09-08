# Requirements Traceability Matrix — TuEvento

> **Matriz de trazabilidad de TuEvento** — Mapeo de Historias de Usuario → Requisitos Funcionales → Casos de Prueba → Servicios.
> Garantiza cobertura completa y verificación de requisitos.

---

## Authentication & Security Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-IAM-001 | FR-001 | User registration with email validation | TC-AUTH-001 | Security Service | 📋 Planned |
| HU-IAM-001 | FR-009 | Account activation with 6-digit code | TC-AUTH-002 | Security Service | 📋 Planned |
| HU-IAM-002 | FR-002 | User authentication with email/password | TC-AUTH-003 | Security Service | 📋 Planned |
| HU-IAM-002 | FR-004 | JWT token generation with refresh | TC-AUTH-004 | Security Service | 📋 Planned |
| HU-IAM-002 | FR-008 | Account lockout after 5 failed attempts | TC-AUTH-005 | Security Service | 📋 Planned |
| HU-IAM-003 | FR-005 | Password recovery via email | TC-AUTH-006 | Security Service | 📋 Planned |
| HU-IAM-004 | FR-003 | Social authentication (OAuth) | TC-AUTH-007 | Security Service | 📋 Planned |
| HU-IAM-004 | FR-006 | Role management (USER/ORGANIZER/ADMIN) | TC-AUTH-008 | Security Service | 📋 Planned |

---

## Event Management Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-EVENT-001 | FR-011 | Create events with basic information | TC-EVENT-001 | Event Service | 📋 Planned |
| HU-EVENT-001 | FR-013 | Event status management (DRAFT/PUBLISHED) | TC-EVENT-002 | Event Service | 📋 Planned |
| HU-EVENT-001 | FR-018 | Validate event dates (finish > start) | TC-EVENT-003 | Event Service | 📋 Planned |
| HU-EVENT-002 | FR-012 | Configure venue layout (JSONB) | TC-EVENT-004 | Event Service | 📋 Planned |
| HU-EVENT-002 | FR-019 | Associate events with venues/sites | TC-EVENT-005 | Geolocation Service | 📋 Planned |
| HU-EVENT-003 | FR-020 | Audit event state changes | TC-EVENT-006 | Event Service | 📋 Planned |
| HU-TICKET-001 | FR-014 | Categorize events hierarchically | TC-EVENT-007 | Category Service | 📋 Planned |
| HU-TICKET-001 | FR-015 | Event multimedia gallery | TC-EVENT-008 | Event Service | 📋 Planned |

---

## Seating & Venue Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-TICKET-002 | FR-021 | Manage event sections (platea, palco) | TC-SEAT-001 | Section Service | 📋 Planned |
| HU-TICKET-002 | FR-022 | Create seats with coordinates | TC-SEAT-002 | Seat Service | 📋 Planned |
| HU-TICKET-002 | FR-024 | Real-time seat status (available/reserved/sold) | TC-SEAT-003 | Seat Service | 📋 Planned |
| HU-TICKET-002 | FR-025 | Reserve seats temporarily (15 min) | TC-SEAT-004 | Seat Service | 📋 Planned |
| HU-TICKET-002 | FR-026 | WebSocket seat status notifications | TC-SEAT-005 | WebSocket Service | 📋 Planned |
| HU-TICKET-002 | FR-027 | Auto-release expired reservations | TC-SEAT-006 | Background Jobs | 📋 Planned |
| HU-TICKET-002 | FR-023 | Seat types (standard/vip/accessible) | TC-SEAT-007 | Seat Service | 📋 Planned |
| HU-TICKET-002 | FR-028 | Validate unique seat positions | TC-SEAT-008 | Seat Service | 📋 Planned |

---

## Ticketing & Orders Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-TICKET-003 | FR-031 | Generate orders with multiple tickets | TC-TICKET-001 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-032 | Generate unique ticket codes | TC-TICKET-002 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-033 | Generate QR codes for tickets | TC-TICKET-003 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-037 | Associate tickets with specific seats | TC-TICKET-004 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-038 | Calculate total order prices | TC-TICKET-005 | Ticket Service | 📋 Planned |
| HU-TICKET-004 | FR-035 | Check-in via QR code scanning | TC-TICKET-006 | Ticket Service | 📋 Planned |
| HU-TICKET-004 | FR-036 | Validate single-use tickets | TC-TICKET-007 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-034 | Ticket status lifecycle | TC-TICKET-008 | Ticket Service | 📋 Planned |
| HU-TICKET-003 | FR-040 | Expire pending orders (15 min) | TC-TICKET-009 | Background Jobs | 📋 Planned |

---

## Payment Processing Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-PAYMENT-001 | FR-041 | Wompi gateway integration | TC-PAY-001 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-042 | Multiple payment methods (card/PSE/Nequi) | TC-PAY-002 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-043 | Payment status management | TC-PAY-003 | Payment Service | 📋 Planned |
| HU-PAYMENT-002 | FR-044 | Process Wompi webhooks | TC-PAY-004 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-047 | Handle COP currency | TC-PAY-005 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-048 | Store gateway transaction IDs | TC-PAY-006 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-050 | Asynchronous payment processing | TC-PAY-007 | Payment Service | 📋 Planned |
| HU-PAYMENT-001 | FR-045 | Handle payment refunds | TC-PAY-008 | Payment Service | 📋 Planned |

---

## Digital Wallet Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-WALLET-001 | FR-051 | Create digital wallet per user | TC-WALLET-001 | Wallet Service | 📋 Planned |
| HU-WALLET-001 | FR-052 | Wallet recharge functionality | TC-WALLET-002 | Wallet Service | 📋 Planned |
| HU-WALLET-002 | FR-053 | Pay with wallet balance | TC-WALLET-003 | Wallet Service | 📋 Planned |
| HU-WALLET-002 | FR-058 | Validate sufficient balance | TC-WALLET-004 | Wallet Service | 📋 Planned |
| HU-WALLET-001 | FR-054 | Transaction types (deposit/withdrawal/payment) | TC-WALLET-005 | Wallet Service | 📋 Planned |
| HU-WALLET-001 | FR-055 | Real-time balance updates | TC-WALLET-006 | Wallet Service | 📋 Planned |
| HU-WALLET-002 | FR-059 | Process refunds to wallet | TC-WALLET-007 | Wallet Service | 📋 Planned |
| HU-WALLET-001 | FR-056 | Transaction origin references | TC-WALLET-008 | Wallet Service | 📋 Planned |

---

## Notifications Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-NOTIFY-001 | FR-061 | Send email notifications | TC-NOTIFY-001 | Notification Service | 📋 Planned |
| HU-NOTIFY-002 | FR-062 | Send push notifications (mobile) | TC-NOTIFY-002 | Notification Service | 📋 Planned |
| HU-NOTIFY-001 | FR-067 | Email tickets with QR codes | TC-NOTIFY-003 | Notification Service | 📋 Planned |
| HU-NOTIFY-001 | FR-066 | Registration confirmation email | TC-NOTIFY-004 | Notification Service | 📋 Planned |
| HU-NOTIFY-001 | FR-065 | Track delivery status | TC-NOTIFY-005 | Notification Service | 📋 Planned |
| HU-NOTIFY-001 | FR-068 | Event change notifications | TC-NOTIFY-006 | Notification Service | 📋 Planned |
| HU-NOTIFY-001 | FR-064 | Notification templates | TC-NOTIFY-007 | Notification Service | 📋 Planned |
| HU-NOTIFY-002 | FR-070 | Opt-out preferences | TC-NOTIFY-008 | Notification Service | 📋 Planned |

---

## User Profile Module

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-PROFILE-001 | FR-071 | Manage public user profile | TC-PROFILE-001 | Profile Service | 📋 Planned |
| HU-PROFILE-001 | FR-076 | Edit personal information | TC-PROFILE-002 | Profile Service | 📋 Planned |
| HU-PROFILE-001 | FR-072 | Language preferences | TC-PROFILE-003 | Profile Service | 📋 Planned |
| HU-PROFILE-001 | FR-073 | Theme selection (light/dark/purple/blue) | TC-PROFILE-004 | Profile Service | 📋 Planned |
| HU-PROFILE-002 | FR-075 | Purchase history display | TC-PROFILE-005 | Profile Service | 📋 Planned |
| HU-PROFILE-001 | FR-077 | City/department association | TC-PROFILE-006 | Profile Service | 📋 Planned |
| HU-PROFILE-001 | FR-079 | Multi-language support | TC-PROFILE-007 | Language Service | 📋 Planned |
| HU-PROFILE-001 | FR-074 | Activity history tracking | TC-PROFILE-008 | Profile Service | 📋 Planned |

---

## Geolocation & Storage Modules

| HU ID | FR ID | Description | Test Case | Service | Status |
|-------|-------|-------------|-----------|---------|--------|
| HU-EVENT-002 | FR-081 | Geographic structure (departments/cities) | TC-GEO-001 | Geolocation Service | 📋 Planned |
| HU-EVENT-001 | FR-082 | Venue/site information management | TC-GEO-002 | Geolocation Service | 📋 Planned |
| HU-EVENT-002 | FR-083 | GPS coordinates for venues | TC-GEO-003 | Geolocation Service | 📋 Planned |
| HU-EVENT-002 | FR-084 | Venue capacity validation | TC-GEO-004 | Geolocation Service | 📋 Planned |
| HU-EVENT-001 | FR-086 | File upload management | TC-STORAGE-001 | Storage Service | 📋 Planned |
| HU-EVENT-001 | FR-088 | File categorization by type | TC-STORAGE-002 | Storage Service | 📋 Planned |
| HU-EVENT-001 | FR-090 | Signed URLs for secure access | TC-STORAGE-003 | Storage Service | 📋 Planned |

---

## Coverage Summary by Module

| Module | Total Mappings | HU Coverage | FR Coverage | Test Cases | Services |
|--------|----------------|-------------|-------------|------------|----------|
| Authentication & Security | 8 | 4/4 (100%) | 8/10 (80%) | 8 | 1 |
| Event Management | 8 | 3/3 (100%) | 8/10 (80%) | 8 | 3 |
| Seating & Venue | 8 | 1/1 (100%) | 8/10 (80%) | 8 | 3 |
| Ticketing & Orders | 9 | 2/2 (100%) | 9/10 (90%) | 9 | 2 |
| Payment Processing | 8 | 2/2 (100%) | 8/10 (80%) | 8 | 1 |
| Digital Wallet | 8 | 2/2 (100%) | 8/10 (80%) | 8 | 1 |
| Notifications | 8 | 2/2 (100%) | 8/10 (80%) | 8 | 1 |
| User Profile | 8 | 2/2 (100%) | 8/10 (80%) | 8 | 2 |
| Geolocation & Storage | 7 | 2/2 (100%) | 7/10 (70%) | 7 | 2 |
| **TOTAL** | **72** | **19/19 (100%)** | **72/90 (80%)** | **72** | **16** |

---

## Test Case Categories

### Unit Tests (TC-XXX-001 to TC-XXX-030)
**Focus:** Individual component functionality
- Service layer business logic
- Domain model validation
- Data transformation accuracy
- Error handling scenarios

### Integration Tests (TC-XXX-031 to TC-XXX-060)  
**Focus:** Component interaction
- API endpoint functionality
- Database operations
- External service integration (Wompi, SMTP)
- WebSocket communication

### End-to-End Tests (TC-XXX-061 to TC-XXX-072)
**Focus:** Complete user workflows
- Registration to ticket purchase flow
- Event creation to publication workflow
- Payment processing complete cycle
- QR code generation to validation

---

## Quality Gates & Test Automation

### Automated Test Requirements
| Test Type | Coverage Target | Execution | Tools |
|-----------|----------------|-----------|-------|
| Unit Tests | 80% code coverage | Pre-commit | JUnit 5, Mockito |
| Integration Tests | 70% API coverage | CI/CD Pipeline | TestContainers, REST Assured |
| Performance Tests | All critical paths | Nightly builds | K6, JMeter |
| Security Tests | OWASP Top 10 | Weekly scans | SonarQube, OWASP ZAP |

### Manual Test Requirements
| Test Type | Coverage | Frequency | Responsible |
|-----------|----------|-----------|-------------|
| Usability Tests | Key user journeys | Per sprint | QA Team |
| Cross-browser Tests | Chrome, Edge, Safari, Firefox | Per release | QA Team |
| Mobile Device Tests | Android 11+, iOS 14+ | Per release | QA Team |
| Accessibility Tests | WCAG 2.1 AA compliance | Per major release | Accessibility Expert |

---

## Risk-Based Testing Priority

### Critical Path Tests (Must Pass)
**High business impact, high failure risk:**
- Payment processing (TC-PAY-001 to TC-PAY-008)
- Seat reservation with WebSockets (TC-SEAT-003 to TC-SEAT-006) 
- Authentication & security (TC-AUTH-001 to TC-AUTH-008)
- QR code generation & validation (TC-TICKET-003, TC-TICKET-006, TC-TICKET-007)

### Important Path Tests (Should Pass)
**Medium business impact, medium failure risk:**
- Event management lifecycle (TC-EVENT-001 to TC-EVENT-008)
- Notification delivery (TC-NOTIFY-001 to TC-NOTIFY-008)
- Profile management (TC-PROFILE-001 to TC-PROFILE-008)

### Supporting Path Tests (Could Defer)
**Low business impact, low failure risk:**
- Advanced wallet features (TC-WALLET-005 to TC-WALLET-008)
- File storage operations (TC-STORAGE-001 to TC-STORAGE-003)
- Geolocation features (TC-GEO-001 to TC-GEO-004)

---

## Requirements Coverage Gaps

### Missing Functional Requirements (10 FRs not mapped)
**Require additional user stories or technical tasks:**
- FR-007: Organizer petition process
- FR-010: Password history management  
- FR-016: Event rating and comments
- FR-017: Nested comment replies
- FR-029: Seat state audit logging
- FR-039: Ticket state audit logging
- FR-046: Payment audit logging
- FR-057: Wallet movement summaries
- FR-069: Event reminder notifications
- FR-078: Profile change audit logging

### Recommended Actions
1. **Create technical user stories** for audit/logging requirements
2. **Add system administration stories** for FR-007 (organizer petitions)
3. **Plan social features epic** for FR-016, FR-017 (ratings/comments)
4. **Define notification enhancement epic** for FR-069 (reminders)

---

## Correlations with other sections

- **User Stories** → `04-requirements/evidence/user-stories.md`
- **Functional Requirements** → `04-requirements/evidence/functional.md`
- **API Contracts** → `07-api/contracts/openapi/`
- **Service Architecture** → `09-microservices/service-catalog.md`
- **Testing Strategy** → `11-quality/testing-strategy.md`
- **Definition of Done** → `00-governance/evidence/definition-of-done.md`

---

## Maintenance Guidelines

### Monthly Reviews
- Verify test case execution results
- Update coverage percentages
- Identify gaps in requirements mapping
- Review risk priorities based on production issues

### Sprint Updates
- Add new user stories to matrix
- Map new functional requirements
- Update test case statuses (Planned → In Progress → Completed)
- Review failed test cases and root causes

### Release Validation
- Confirm 100% critical path test coverage
- Validate all "Must Have" requirements are tested
- Review end-to-end scenario coverage
- Sign-off on traceability completeness