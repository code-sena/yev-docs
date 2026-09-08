# User Stories (HU) — TuEvento

> **Historias de usuario de TuEvento** — Descripción de funcionalidades desde la perspectiva del usuario.
> Basado en los requisitos funcionales y análisis de dominio del proyecto.

---

## HU-IAM: Identity & Access Management

### HU-IAM-001: User Registration

**As** a potential user
**I want** to create an account with email and password
**So that** I can access the TuEvento platform and purchase tickets

**Acceptance criteria:**
- [ ] **AC1:** Given a valid email and password, when I submit registration form, then account is created and activation email is sent
- [ ] **AC2:** Given an existing email, when I try to register, then I receive "Email already exists" error
- [ ] **AC3:** Given invalid password format, when I submit form, then I receive validation errors

**Technical notes:**
- **Responsible service(s):** Security Module
- **Endpoint(s) implemented:** POST /api/v1/auth/register
- **Events generated:** UserRegisteredEvent
- **Required permissions:** None (public endpoint)

---

### HU-IAM-002: User Login

**As** a registered user
**I want** to login with my email and password
**So that** I can access my account and use protected features

**Acceptance criteria:**
- [ ] **AC1:** Given valid credentials, when I login, then I receive JWT access token and refresh token
- [ ] **AC2:** Given invalid credentials, when I login, then I receive authentication error
- [ ] **AC3:** Given locked account, when I try to login, then I receive account locked message

**Technical notes:**
- **Responsible service(s):** Security Module
- **Endpoint(s) implemented:** POST /api/v1/auth/login
- **Events generated:** UserLoggedInEvent
- **Required permissions:** None (public endpoint)

---

### HU-IAM-003: Password Recovery

**As** a user who forgot my password
**I want** to recover access to my account via email
**So that** I can regain access without creating a new account

**Acceptance criteria:**
- [ ] **AC1:** Given a registered email, when I request password reset, then I receive reset email with secure token
- [ ] **AC2:** Given valid reset token, when I set new password, then password is updated and I can login
- [ ] **AC3:** Given expired token, when I try to reset password, then I receive token expired error

**Technical notes:**
- **Responsible service(s):** Security Module
- **Endpoint(s) implemented:** POST /api/v1/auth/forgot-password, POST /api/v1/auth/reset-password
- **Events generated:** PasswordResetRequestedEvent, PasswordResetCompletedEvent
- **Required permissions:** None (public endpoint)

---

### HU-IAM-004: Social Authentication

**As** a user
**I want** to login using my Google/Facebook account
**So that** I can access the platform without creating another password

**Acceptance criteria:**
- [ ] **AC1:** Given valid OAuth credentials, when I login via social provider, then account is created/linked and I receive JWT tokens
- [ ] **AC2:** Given existing email via different provider, when I try social login, then account is linked
- [ ] **AC3:** Given invalid OAuth response, when social login fails, then I receive appropriate error message

**Technical notes:**
- **Responsible service(s):** Security Module
- **Endpoint(s) implemented:** POST /api/v1/auth/oauth/google, POST /api/v1/auth/oauth/facebook
- **Events generated:** SocialLoginEvent
- **Required permissions:** None (public endpoint)

---

## HU-EVENT: Event Management

### HU-EVENT-001: Create Event

**As** an organizer
**I want** to create a new event with basic information
**So that** I can start setting up my event for attendees

**Acceptance criteria:**
- [ ] **AC1:** Given valid event details, when I create event, then event is saved in DRAFT status
- [ ] **AC2:** Given event with past date, when I try to create, then I receive validation error
- [ ] **AC3:** Given incomplete required fields, when I submit, then I receive field validation errors

**Technical notes:**
- **Responsible service(s):** Event Module
- **Endpoint(s) implemented:** POST /api/v1/events
- **Events generated:** EventCreatedEvent
- **Required permissions:** ORGANIZER role

---

### HU-EVENT-002: Design Venue Layout

**As** an organizer
**I want** to design the venue layout with sections and seats
**So that** attendees can select specific seats when purchasing tickets

**Acceptance criteria:**
- [ ] **AC1:** Given an event, when I create sections with coordinates, then layout is saved as JSONB
- [ ] **AC2:** Given overlapping seat positions, when I save layout, then I receive position conflict error
- [ ] **AC3:** Given valid layout, when I publish event, then seats become available for reservation

**Technical notes:**
- **Responsible service(s):** Event Module, Section Module, Seat Module
- **Endpoint(s) implemented:** PUT /api/v1/events/{id}/layout
- **Events generated:** VenueLayoutUpdatedEvent
- **Required permissions:** ORGANIZER role, event ownership

---

### HU-EVENT-003: Publish Event

**As** an organizer
**I want** to publish my event to make it visible to attendees
**So that** people can discover and purchase tickets for my event

**Acceptance criteria:**
- [ ] **AC1:** Given a complete event with layout, when I publish, then event status changes to PUBLISHED
- [ ] **AC2:** Given incomplete event setup, when I try to publish, then I receive validation errors
- [ ] **AC3:** Given published event, when viewed by attendees, then all event details are visible

**Technical notes:**
- **Responsible service(s):** Event Module
- **Endpoint(s) implemented:** PUT /api/v1/events/{id}/status
- **Events generated:** EventPublishedEvent
- **Required permissions:** ORGANIZER role, event ownership

---

## HU-TICKET: Ticketing & Seat Selection

### HU-TICKET-001: Browse Events

**As** an attendee
**I want** to browse available events by category and location
**So that** I can find events I'm interested in attending

**Acceptance criteria:**
- [ ] **AC1:** Given published events, when I browse events page, then I see list of available events
- [ ] **AC2:** Given category filter, when I apply filter, then only events in that category are shown
- [ ] **AC3:** Given location filter, when I apply filter, then only events in that area are shown

**Technical notes:**
- **Responsible service(s):** Event Module, Category Module
- **Endpoint(s) implemented:** GET /api/v1/events/public
- **Events generated:** None
- **Required permissions:** None (public endpoint)

---

### HU-TICKET-002: Select Seats

**As** an attendee
**I want** to select specific seats from the venue layout
**So that** I can choose the best available seats for my preferences

**Acceptance criteria:**
- [ ] **AC1:** Given available seats, when I click on seat, then seat is temporarily reserved for 15 minutes
- [ ] **AC2:** Given unavailable seat, when I try to select, then seat remains unselectable
- [ ] **AC3:** Given multiple seat selection, when I select seats, then all selected seats are reserved together

**Technical notes:**
- **Responsible service(s):** Seat Module, WebSocket Service
- **Endpoint(s) implemented:** PUT /api/v1/seats/{id}/reserve
- **Events generated:** SeatReservedEvent (via WebSocket)
- **Required permissions:** USER role

---

### HU-TICKET-003: Purchase Tickets

**As** an attendee
**I want** to purchase tickets for selected seats with secure payment
**So that** I can confirm my attendance and receive valid tickets

**Acceptance criteria:**
- [ ] **AC1:** Given reserved seats and payment info, when I complete purchase, then order is created and payment processed
- [ ] **AC2:** Given payment failure, when transaction fails, then seats are released and I receive error message
- [ ] **AC3:** Given successful payment, when purchase completes, then I receive tickets via email with QR codes

**Technical notes:**
- **Responsible service(s):** Ticket Module, Payment Module
- **Endpoint(s) implemented:** POST /api/v1/orders, POST /api/v1/payments/process
- **Events generated:** OrderCreatedEvent, PaymentProcessedEvent, TicketsGeneratedEvent
- **Required permissions:** USER role

---

### HU-TICKET-004: QR Code Validation

**As** an attendee
**I want** to use my QR code for event check-in
**So that** I can enter the event with valid proof of purchase

**Acceptance criteria:**
- [ ] **AC1:** Given valid QR code, when scanned at event, then check-in is recorded and entry is allowed
- [ ] **AC2:** Given already used ticket, when QR is scanned again, then entry is denied
- [ ] **AC3:** Given invalid/expired QR code, when scanned, then entry is denied with appropriate message

**Technical notes:**
- **Responsible service(s):** Ticket Module
- **Endpoint(s) implemented:** POST /api/v1/tickets/checkin
- **Events generated:** TicketUsedEvent
- **Required permissions:** ORGANIZER role (for scanning)

---

## HU-PAYMENT: Payment Processing

### HU-PAYMENT-001: Process Payment

**As** an attendee
**I want** to pay securely using multiple payment methods
**So that** I can complete my ticket purchase with my preferred payment option

**Acceptance criteria:**
- [ ] **AC1:** Given valid payment method (card/PSE/Nequi), when I process payment, then transaction is sent to Wompi
- [ ] **AC2:** Given successful payment, when Wompi confirms, then order status changes to PAID
- [ ] **AC3:** Given payment failure, when Wompi rejects, then order status changes to FAILED and seats released

**Technical notes:**
- **Responsible service(s):** Payment Module
- **Endpoint(s) implemented:** POST /api/v1/payments/wompi
- **Events generated:** PaymentInitiatedEvent, PaymentCompletedEvent
- **Required permissions:** USER role

---

### HU-PAYMENT-002: Handle Webhooks

**As** the system
**I want** to receive and process Wompi webhook notifications
**So that** payment status is updated accurately in real-time

**Acceptance criteria:**
- [ ] **AC1:** Given Wompi webhook, when payment status changes, then order status is updated accordingly
- [ ] **AC2:** Given webhook signature validation, when signature is invalid, then webhook is rejected
- [ ] **AC3:** Given successful payment webhook, when processed, then tickets are generated and sent

**Technical notes:**
- **Responsible service(s):** Payment Module
- **Endpoint(s) implemented:** POST /api/v1/webhooks/wompi
- **Events generated:** PaymentStatusUpdatedEvent
- **Required permissions:** None (webhook endpoint with signature validation)

---

## HU-WALLET: Digital Wallet

### HU-WALLET-001: Wallet Recharge

**As** a user
**I want** to add money to my digital wallet
**So that** I can make faster payments for future ticket purchases

**Acceptance criteria:**
- [ ] **AC1:** Given valid recharge amount, when I add funds, then wallet balance is increased
- [ ] **AC2:** Given payment failure for recharge, when transaction fails, then wallet balance remains unchanged
- [ ] **AC3:** Given successful recharge, when completed, then transaction history is updated

**Technical notes:**
- **Responsible service(s):** Wallet Module, Payment Module
- **Endpoint(s) implemented:** POST /api/v1/wallet/recharge
- **Events generated:** WalletRechargedEvent
- **Required permissions:** USER role

---

### HU-WALLET-002: Pay with Wallet

**As** a user
**I want** to pay for tickets using my wallet balance
**So that** I can complete purchases faster without entering payment details

**Acceptance criteria:**
- [ ] **AC1:** Given sufficient wallet balance, when I pay with wallet, then purchase is completed and balance deducted
- [ ] **AC2:** Given insufficient balance, when I try to pay, then payment is rejected with balance error
- [ ] **AC3:** Given partial wallet balance, when I pay, then remaining amount is charged to secondary payment method

**Technical notes:**
- **Responsible service(s):** Wallet Module, Payment Module
- **Endpoint(s) implemented:** POST /api/v1/payments/wallet
- **Events generated:** WalletPaymentEvent
- **Required permissions:** USER role

---

## HU-NOTIFY: Notifications

### HU-NOTIFY-001: Email Notifications

**As** a user
**I want** to receive important notifications via email
**So that** I stay informed about my account and ticket activities

**Acceptance criteria:**
- [ ] **AC1:** Given successful ticket purchase, when order completes, then confirmation email with tickets is sent
- [ ] **AC2:** Given event changes, when organizer updates event, then notification email is sent to attendees
- [ ] **AC3:** Given email delivery failure, when email can't be sent, then delivery status is logged as failed

**Technical notes:**
- **Responsible service(s):** Notification Module
- **Endpoint(s) implemented:** Internal messaging system
- **Events generated:** EmailNotificationSentEvent
- **Required permissions:** System internal

---

### HU-NOTIFY-002: Push Notifications

**As** a mobile user
**I want** to receive push notifications for important updates
**So that** I get timely information about my events and tickets

**Acceptance criteria:**
- [ ] **AC1:** Given mobile app installed, when important event occurs, then push notification is sent
- [ ] **AC2:** Given notification preferences, when user opts out, then push notifications are disabled
- [ ] **AC3:** Given event reminder, when event is approaching, then reminder notification is sent

**Technical notes:**
- **Responsible service(s):** Notification Module
- **Endpoint(s) implemented:** Push notification service integration
- **Events generated:** PushNotificationSentEvent
- **Required permissions:** USER role, notification consent

---

## HU-PROFILE: User Profile

### HU-PROFILE-001: Manage Profile

**As** a user
**I want** to update my personal information and preferences
**So that** I can keep my account information current and customize my experience

**Acceptance criteria:**
- [ ] **AC1:** Given valid profile data, when I update information, then changes are saved and reflected
- [ ] **AC2:** Given invalid email format, when I update email, then validation error is shown
- [ ] **AC3:** Given language preference change, when I update, then interface language changes accordingly

**Technical notes:**
- **Responsible service(s):** Profile Module
- **Endpoint(s) implemented:** PUT /api/v1/profile
- **Events generated:** ProfileUpdatedEvent
- **Required permissions:** USER role, profile ownership

---

### HU-PROFILE-002: Purchase History

**As** a user
**I want** to view my complete ticket purchase history
**So that** I can track my spending and access past event details

**Acceptance criteria:**
- [ ] **AC1:** Given past purchases, when I view history, then all orders and tickets are displayed chronologically
- [ ] **AC2:** Given specific order, when I click details, then full order information including tickets is shown
- [ ] **AC3:** Given refunded orders, when viewing history, then refund status is clearly indicated

**Technical notes:**
- **Responsible service(s):** Profile Module, Ticket Module
- **Endpoint(s) implemented:** GET /api/v1/profile/purchases
- **Events generated:** None
- **Required permissions:** USER role, profile ownership

---

## User Stories Summary

| Module | Total HUs | Epic Priority | Story Points Est. |
|--------|-----------|---------------|-------------------|
| Identity & Access | 4 | Critical | 21 |
| Event Management | 3 | High | 21 |
| Ticketing & Seats | 4 | Critical | 34 |
| Payment Processing | 2 | Critical | 13 |
| Digital Wallet | 2 | Medium | 13 |
| Notifications | 2 | Medium | 8 |
| User Profile | 2 | Low | 8 |
| **TOTAL** | **19** | - | **118** |

---

## Epic Prioritization for MVP

### Critical (Must Have) - Sprint 1-3
**10 HUs** esenciales para funcionalidad básica:
- Autenticación completa (registro, login, recuperación)
- Creación y publicación de eventos
- Selección de asientos y compra de tickets
- Procesamiento de pagos con Wompi

### High Priority (Should Have) - Sprint 4-6  
**3 HUs** que mejoran significativamente la experiencia:
- Diseño avanzado de venues
- Funciones organizador completas
- Gestión de eventos publicados

### Medium Priority (Could Have) - Sprint 7-9
**4 HUs** para características adicionales:
- Sistema de billetera digital
- Notificaciones push y email
- Características sociales básicas

### Low Priority (Won't Have in MVP) - Future Releases
**2 HUs** para versiones futuras:
- Gestión avanzada de perfil
- Reportes y analytics

---

## Sprint Planning Breakdown

### Sprint 1 (3 weeks) - Authentication Foundation
- HU-IAM-001: User Registration (5 pts)
- HU-IAM-002: User Login (3 pts)
- HU-IAM-003: Password Recovery (5 pts)

### Sprint 2 (3 weeks) - Event Management Core
- HU-EVENT-001: Create Event (5 pts)
- HU-EVENT-003: Publish Event (3 pts)
- HU-TICKET-001: Browse Events (3 pts)

### Sprint 3 (4 weeks) - Ticketing System
- HU-EVENT-002: Design Venue Layout (8 pts)
- HU-TICKET-002: Select Seats (8 pts)

### Sprint 4 (3 weeks) - Payment Integration
- HU-TICKET-003: Purchase Tickets (8 pts)
- HU-PAYMENT-001: Process Payment (5 pts)
- HU-PAYMENT-002: Handle Webhooks (5 pts)

### Sprint 5 (2 weeks) - QR & Validation
- HU-TICKET-004: QR Code Validation (8 pts)
- HU-IAM-004: Social Authentication (3 pts)

---

## Definition of Ready (DoR) Checklist

Each User Story must have:
- [ ] Clear acceptance criteria with Given/When/Then format
- [ ] Technical implementation notes
- [ ] Identified responsible services
- [ ] API endpoints defined
- [ ] Story points estimated by team
- [ ] Dependencies identified
- [ ] Mockups/wireframes available (for UI stories)
- [ ] Performance criteria defined (if applicable)

---

## Correlations with other sections

- **Functional Requirements** → `04-requirements/evidence/functional.md`
- **Domain Events** → `02-domain/evidence/domain-events.md`  
- **API Contracts** → `07-api/contracts/openapi/`
- **Service Architecture** → `09-microservices/service-catalog.md`
- **Testing Strategy** → `11-quality/testing-strategy.md`
- **Definition of Done** → `00-governance/evidence/definition-of-done.md`