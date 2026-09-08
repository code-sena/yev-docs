# Domain Events — TuEvento

> **Domain Events for TuEvento** — Eventos de dominio que comunican hechos importantes del negocio entre bounded contexts.
> Implementados con Spring Boot + messaging para comunicación asíncrona entre microservicios.

---

## Event catalog

### Event: UserRegistered

| Field | Value |
|-------|-------|
| **Name** | `UserRegistered` |
| **Bounded Context** | Security & Identity |
| **Aggregate** | User |
| **Trigger** | Usuario completa registro exitosamente |
| **Consumers** | Profile Service, Notification Service |
| **Channel (topic)** | `user.events` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "UserRegistered",
  "aggregateId": "user-uuid",
  "aggregateType": "User",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "alias": "cristofer2303",
    "email": "cristoferlozano233@gmail.com",
    "role": "USER",
    "activationRequired": true
  },
  "metadata": {
    "correlationId": "registration-flow-001",
    "causationId": null,
    "userId": "550e8400-e29b-41d4-a716-446655440001"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Profile Service | Crea perfil inicial del usuario | Yes — usa userId como idempotency key |
| Notification Service | Envía email de bienvenida | Yes — verifica si email ya fue enviado |

---

### Event: EventPublished

| Field | Value |
|-------|-------|
| **Name** | `EventPublished` |
| **Bounded Context** | Event Management |
| **Aggregate** | Event |
| **Trigger** | Organizador publica evento (DRAFT → PUBLISHED) |
| **Consumers** | Notification Service, Seating Service |
| **Channel (topic)** | `event.lifecycle` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "EventPublished",
  "aggregateId": "event-uuid",
  "aggregateType": "Event",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "eventId": "550e8400-e29b-41d4-a716-446655440002",
    "eventName": "Concierto Foo Fighters",
    "organizerId": "550e8400-e29b-41d4-a716-446655440001",
    "siteId": "550e8400-e29b-41d4-a716-446655440003",
    "startDate": "2024-12-15",
    "finishDate": "2024-12-15",
    "totalCapacity": 50000,
    "isPublic": true
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Notification Service | Notifica a usuarios suscritos a la categoría | Yes — controla duplicados por eventId |
| Seating Service | Inicializa estado de asientos como AVAILABLE | Yes — verifica si ya fueron inicializados |

---

### Event: SeatReserved

| Field | Value |
|-------|-------|
| **Name** | `SeatReserved` |
| **Bounded Context** | Seating & Venue |
| **Aggregate** | Seat |
| **Trigger** | Usuario selecciona asiento en tiempo real |
| **Consumers** | Ticketing Service, WebSocket Clients |
| **Channel (topic)** | `seat.realtime` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "SeatReserved",
  "aggregateId": "seat-uuid",
  "aggregateType": "Seat",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "seatId": "550e8400-e29b-41d4-a716-446655440004",
    "eventId": "550e8400-e29b-41d4-a716-446655440002",
    "sectionId": "550e8400-e29b-41d4-a716-446655440005",
    "seatCode": "A-15",
    "reservedBy": "550e8400-e29b-41d4-a716-446655440001",
    "reservationToken": "temp-reservation-token-123",
    "expiresAt": "2024-09-08T14:45:00Z",
    "price": 150000.00
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| WebSocket Service | Notifica a todos los clientes conectados al evento | No — broadcast en tiempo real |
| Ticketing Service | Prepara pre-order temporal | Yes — usa reservationToken como key |

---

### Event: TicketPurchased

| Field | Value |
|-------|-------|
| **Name** | `TicketPurchased` |
| **Bounded Context** | Ticketing & Orders |
| **Aggregate** | Order |
| **Trigger** | Pago exitoso de orden (PENDING_PAYMENT → PAID) |
| **Consumers** | Payment Service, Notification Service, Seating Service |
| **Channel (topic)** | `order.lifecycle` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "TicketPurchased",
  "aggregateId": "order-uuid",
  "aggregateType": "Order",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "orderId": "550e8400-e29b-41d4-a716-446655440006",
    "eventId": "550e8400-e29b-41d4-a716-446655440002",
    "buyerId": "550e8400-e29b-41d4-a716-446655440001",
    "totalAmount": 300000.00,
    "currency": "COP",
    "tickets": [
      {
        "ticketId": "550e8400-e29b-41d4-a716-446655440007",
        "seatIds": ["550e8400-e29b-41d4-a716-446655440004"],
        "price": 150000.00,
        "qrCode": "TKT-ABC123-QR"
      }
    ]
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Payment Service | Registra transacción exitosa | Yes — usa orderId como key |
| Notification Service | Envía email con tickets (QR) | Yes — controla por orderId |
| Seating Service | Marca asientos como SOLD | Yes — verifica estado actual |

---

### Event: PaymentProcessed

| Field | Value |
|-------|-------|
| **Name** | `PaymentProcessed` |
| **Bounded Context** | Payment Processing |
| **Aggregate** | Payment |
| **Trigger** | Gateway de pago (Wompi) confirma transacción |
| **Consumers** | Ticketing Service, Wallet Service, Notification Service |
| **Channel (topic)** | `payment.events` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "PaymentProcessed",
  "aggregateId": "payment-uuid",
  "aggregateType": "Payment",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "paymentId": "550e8400-e29b-41d4-a716-446655440008",
    "orderId": "550e8400-e29b-41d4-a716-446655440006",
    "gateway": "wompi",
    "gatewayTransactionId": "wompi-txn-123456",
    "amount": 300000.00,
    "currency": "COP",
    "paymentMethod": "credit_card",
    "status": "approved",
    "processedAt": "2024-09-08T14:29:45Z"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Ticketing Service | Activa tickets de la orden | Yes — verifica estado de orden |
| Wallet Service | Registra movimiento si se usó wallet | Yes — usa paymentId como key |
| Notification Service | Confirma pago exitoso al usuario | Yes — controla duplicados |

---

### Event: TicketValidated

| Field | Value |
|-------|-------|
| **Name** | `TicketValidated` |
| **Bounded Context** | Ticketing & Orders |
| **Aggregate** | Ticket |
| **Trigger** | Check-in exitoso en el evento (QR escaneado) |
| **Consumers** | Event Service, Analytics Service, Notification Service |
| **Channel (topic)** | `ticket.validation` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "TicketValidated",
  "aggregateId": "ticket-uuid",
  "aggregateType": "Ticket",
  "occurredAt": "2024-12-15T19:30:00Z",
  "version": 1,
  "payload": {
    "ticketId": "550e8400-e29b-41d4-a716-446655440007",
    "eventId": "550e8400-e29b-41d4-a716-446655440002",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "validatedBy": "550e8400-e29b-41d4-a716-446655440009",
    "validatedAt": "2024-12-15T19:30:00Z",
    "seatCodes": ["A-15"],
    "checkinLocation": {
      "gate": "Puerta A",
      "coordinates": "2.9273,-75.2819"
    }
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| Event Service | Incrementa contador de asistencia | Yes — controla por ticketId |
| Analytics Service | Registra métrica de asistencia | Yes — usa ticketId único |
| Notification Service | Envía confirmación de ingreso | Yes — verifica si ya se envió |

---

### Event: WalletTransactionCreated

| Field | Value |
|-------|-------|
| **Name** | `WalletTransactionCreated` |
| **Bounded Context** | Payment Processing |
| **Aggregate** | Wallet |
| **Trigger** | Usuario hace recarga, pago o recibe reembolso |
| **Consumers** | Notification Service, Analytics Service |
| **Channel (topic)** | `wallet.transactions` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "WalletTransactionCreated",
  "aggregateId": "wallet-transaction-uuid",
  "aggregateType": "WalletTransaction",
  "occurredAt": "2024-09-08T14:30:00Z",
  "version": 1,
  "payload": {
    "transactionId": "550e8400-e29b-41d4-a716-446655440010",
    "walletId": "550e8400-e29b-41d4-a716-446655440011",
    "userId": "550e8400-e29b-41d4-a716-446655440001",
    "type": "payment",
    "amount": 150000.00,
    "currency": "COP",
    "status": "completed",
    "referenceEntity": {
      "type": "order",
      "id": "550e8400-e29b-41d4-a716-446655440006"
    },
    "newBalance": 50000.00
  }
}
```

---

## Event flows: Main business processes

### Flow: Compra de tickets

```
User ──(SelectSeats)──▶ [Seating Service] ──[SeatReserved]──▶ WebSocket Clients
                              │
                              ▼ (CreateOrder)
                        [Ticketing Service] ──[OrderCreated]──▶ Payment Service
                              │                                      │
                              │                                      ▼ (ProcessPayment)
                              │ ◄─────[PaymentProcessed]─────── [Payment Service]
                              ▼                                      │
                        [Generate Tickets]                          │
                              │                                      │
                              └──[TicketPurchased]──▶ Notification Service
                                      │                             │
                                      ▼                             ▼
                               [Update Seats]                 [Send Email QR]
```

### Flow: Publicación de evento

```
Organizer ──(CreateEvent)──▶ [Event Service] ──[EventCreated]──▶ Seating Service
                                   │                                   │
                                   │ (PublishEvent)                    ▼ (InitializeSeats)
                                   ▼                            [Create Seat Records]
                            [Event Service] ──[EventPublished]──▶ Notification Service
                                                                        │
                                                                        ▼ (NotifySubscribers)
                                                                 [Send New Event Alerts]
```

### Flow: Check-in en evento

```
User ──(ScanQR)──▶ [Ticket Service] ──[ValidateTicket]──▶ [Ticket Service]
                         │                                      │
                         │ (if valid)                          ▼
                         └──[TicketValidated]──▶ Event Service ──▶ Analytics
                                   │                  │
                                   ▼                  ▼
                           Notification Service   [Update Attendance]
                                   │
                                   ▼
                           [Send Welcome Message]
```

---

## Standard event envelope

Todos los eventos de TuEvento siguen este formato estándar:

```json
{
  "eventId": "UUID único del evento",
  "eventType": "NombreDelEvento en PascalCase",
  "aggregateId": "ID del aggregate que generó el evento",
  "aggregateType": "Tipo de aggregate",
  "occurredAt": "Timestamp ISO 8601 UTC",
  "version": "Versión del schema (integer)",
  "payload": {
    "// Datos específicos del evento"
  },
  "metadata": {
    "correlationId": "UUID para tracing cross-service",
    "causationId": "ID del evento/comando que causó este evento",
    "userId": "Usuario que inició la cadena (si aplica)",
    "sessionId": "Sesión del usuario (si aplica)"
  }
}
```

---

## Event schema evolution strategy

### Cambios compatibles ✅

- Agregar nuevo campo opcional al payload
- Agregar nuevo tipo de evento
- Cambiar campo requerido → opcional
- Agregar metadata adicional

### Cambios incompatibles ❌

- Eliminar campo del payload
- Cambiar tipo de datos de un campo
- Cambiar campo opcional → requerido  
- Cambiar nombre del evento

### Estrategia de versionado

Para cambios incompatibles, usar versionado de eventos:

```
EventoV1 (existente) → EventoV2 (nueva versión)

1. Publicar EventoV2 con cambios
2. Período de migración: publicar V1 y V2 simultáneamente  
3. Migrar consumers uno por uno a V2
4. Deprecar EventoV1 después de 1 sprint
5. Eliminar publicación de EventoV1
```

---

## Event summary table

| Event | Origin Context | Topic | Consumers | Delivery | Version |
|-------|---------------|-------|-----------|----------|---------|
| UserRegistered | Security & Identity | user.events | Profile, Notifications | At-least-once | v1 |
| EventPublished | Event Management | event.lifecycle | Notifications, Seating | At-least-once | v1 |
| SeatReserved | Seating & Venue | seat.realtime | WebSockets, Ticketing | At-least-once | v1 |
| TicketPurchased | Ticketing & Orders | order.lifecycle | Payment, Notifications, Seating | At-least-once | v1 |
| PaymentProcessed | Payment Processing | payment.events | Ticketing, Wallet, Notifications | At-least-once | v1 |
| TicketValidated | Ticketing & Orders | ticket.validation | Event, Analytics, Notifications | At-least-once | v1 |
| WalletTransactionCreated | Payment Processing | wallet.transactions | Notifications, Analytics | At-least-once | v1 |

---

## Resilience patterns

### Idempotency

Todos los consumers implementan idempotencia usando `eventId` como clave:

```java
@EventHandler
public void handle(TicketPurchasedEvent event) {
    // 1. Check if already processed
    if (eventStore.wasProcessed(event.getEventId())) {
        log.info("Event {} already processed, skipping", event.getEventId());
        return;
    }
    
    // 2. Process business logic
    processTicketPurchase(event.getPayload());
    
    // 3. Mark as processed (same transaction)
    eventStore.markAsProcessed(event.getEventId());
}
```

### Dead Letter Queue configuration

| Setting | Value | Reason |
|---------|-------|---------|
| Max retries | 5 | Balance entre reliability y performance |
| Backoff | Exponential (1s → 2s → 4s → 8s → 16s) | Avoid overwhelming downstream |
| DLQ retention | 7 days | Debugging window |
| Alert threshold | > 0 messages in DLQ | Immediate notification |

### WebSockets real-time events

Para eventos de tiempo real (SeatReserved, TicketValidated), se usa patrón dual:

1. **Async messaging** para persistencia y business logic
2. **WebSockets** para notificación inmediata a clientes

```java
@EventHandler
public void handle(SeatReservedEvent event) {
    // 1. Process business logic via messaging
    seatService.updateSeatStatus(event.getPayload());
    
    // 2. Broadcast to WebSocket clients immediately
    webSocketService.broadcast(
        "event." + event.getPayload().getEventId(),
        event
    );
}
```

---

## Technology implementation notes

**Message Broker:** Spring Boot + RabbitMQ / Apache Kafka  
**Serialization:** JSON with Jackson  
**WebSockets:** SockJS + STOMP protocol  
**Idempotency Storage:** PostgreSQL event store table  
**Monitoring:** Spring Boot Actuator + Micrometer metrics

> Ver implementación detallada en `09-microservices/services/XX-service/events.md`