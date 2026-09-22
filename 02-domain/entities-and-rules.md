# Entities, Value Objects, and Business Rules — TuEvento

> **Domain model for TuEvento** — Entidades, objetos de valor y reglas de negocio del sistema de gestión de eventos.
> Basado en la arquitectura N-Layer + DDD implementada en Java 23 + Spring Boot.

---

## Tactical DDD concepts aplicados a TuEvento

### Entity
Objetos del dominio con identidad única. En TuEvento, las entidades principales representan conceptos de negocio con ciclo de vida propio.

### Value Object (VO)
Objetos inmutables definidos por sus atributos. En TuEvento incluyen Email, Money, Address, etc.

### Aggregate
Cluster de entidades tratadas como unidad transaccional. En TuEvento cada aggregate tiene reglas de consistencia específicas.

---

## System entities

### Entity: User

**Context:** Security & Identity

**Description:** Usuario del sistema con credenciales, rol y estado de activación

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| user_id | INT | Identificador único | Yes | Auto-generated PK |
| user_status_id | INT | Estado del usuario | Yes | FK to user_status |
| role_id | INT | Rol del usuario | Yes | FK to role |
| alias | VARCHAR(50) | Nombre de usuario único | Yes | Unique constraint |
| activated | BOOLEAN | Cuenta activada | Yes | Default: false |
| created_at | TIMESTAMP | Fecha de creación | Yes | Auto-generated |

**Lifecycle / States:**

```
[PENDING_ACTIVATION] ──(activate)──▶ [ACTIVE] ──(suspend)──▶ [SUSPENDED]
         │                             │
    (expire_activation)          (deactivate)
         ▼                             ▼
    [EXPIRED]                     [INACTIVE]
```

**Invariants (Business rules):**

```
INV-USER-001: Alias único
  - Rule: No pueden existir dos usuarios con el mismo alias
  - Implementation: Unique constraint en DB + validación en aplicación

INV-USER-002: Rol obligatorio
  - Rule: Todo usuario debe tener un rol asignado
  - Implementation: NOT NULL constraint + validación en constructor

INV-USER-003: Activación requerida para organizadores
  - Rule: Usuarios con rol ORGANIZER deben estar activados
  - Implementation: Validación antes de asignar rol organizer
```

---

### Entity: Event

**Context:** Event Management

**Description:** Evento creado por un organizador con venue, fechas y configuración

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| event_id | INT | Identificador único | Yes | Auto-generated PK |
| user_id | INT | Organizador del evento | Yes | FK to user (role=ORGANIZER) |
| site_id | INT | Venue del evento | Yes | FK to site |
| event_name | VARCHAR(100) | Nombre del evento | Yes | Max 100 chars |
| description | VARCHAR(255) | Descripción | Yes | Max 255 chars |
| start_date | DATE | Fecha de inicio | Yes | Must be future date |
| finish_date | DATE | Fecha de fin | Yes | Must be > start_date |
| status | ENUM | Estado del evento | Yes | draft/published/cancelled/finished |
| is_public | BOOLEAN | Visibilidad pública | Yes | Default: true |
| available_seats | INT | Asientos disponibles | Yes | Must be > 0 |

**Lifecycle / States:**

```
[DRAFT] ──(publish)──▶ [PUBLISHED] ──(finish)──▶ [FINISHED]
   │                        │
(cancel)                (cancel)
   ▼                        ▼
[CANCELLED] ◄───────────[CANCELLED]
```

**Invariants (Business rules):**

```
INV-EVENT-001: Fechas válidas
  - Rule: finish_date > start_date
  - Implementation: CHECK constraint + domain validation

INV-EVENT-002: Organizador válido
  - Rule: Solo usuarios con rol ORGANIZER pueden crear eventos
  - Implementation: Validación en capa de aplicación

INV-EVENT-003: Capacidad positiva
  - Rule: available_seats debe ser mayor a 0
  - Implementation: CHECK constraint + validation

INV-EVENT-004: Estado inmutable después de finished
  - Rule: Un evento FINISHED no puede cambiar de estado
  - Implementation: Validación en método changeStatus()
```

---

### Entity: Seat

**Context:** Seating & Venue

**Description:** Asiento individual dentro de una sección con estado en tiempo real

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| seat_id | INT | Identificador único | Yes | Auto-generated PK |
| seat_block_id | INT | Bloque contenedor | Yes | FK to seat_block |
| event_section_id | INT | Sección del evento | Yes | FK to event_section |
| code | VARCHAR(20) | Código del asiento | Yes | Unique per section |
| seat_row | INT | Fila del asiento | Yes | Positive integer |
| position | INT | Posición en la fila | Yes | Positive integer |
| type | ENUM | Tipo de asiento | Yes | standard/vip/accessible |
| status | ENUM | Estado actual | Yes | available/reserved/sold/blocked |

**Lifecycle / States:**

```
[AVAILABLE] ──(reserve)──▶ [RESERVED] ──(purchase)──▶ [SOLD]
     │                          │                        │
  (block)                  (release)                (refund)
     ▼                          ▼                        ▼
[BLOCKED] ◄─────────────[AVAILABLE]◄──────────[AVAILABLE]
```

**Invariants (Business rules):**

```
INV-SEAT-001: Posición única por sección
  - Rule: No pueden existir dos asientos con misma row + position en una sección
  - Implementation: Composite unique constraint

INV-SEAT-002: Reserva temporal
  - Rule: Un asiento RESERVED debe liberarse automáticamente después de 15 minutos
  - Implementation: Background job + timestamp tracking

INV-SEAT-003: Venta única
  - Rule: Un asiento SOLD solo puede venderse una vez
  - Implementation: Estado inmutable + validación
```

---

### Entity: Ticket

**Context:** Ticketing & Orders

**Description:** Boleto generado para un evento con QR único y vinculación a asientos

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| ticket_id | INT | Identificador único | Yes | Auto-generated PK |
| event_id | INT | Evento del boleto | Yes | FK to event |
| user_id | INT | Comprador | Yes | FK to user |
| order_id | INT | Orden de compra | Yes | FK to order |
| code | VARCHAR(20) | Código único | Yes | Unique globally |
| qr_code | TEXT | Código QR | No | Generated after payment |
| status | ENUM | Estado del boleto | Yes | valid/used/cancelled/expired |
| expiration_date | DATE | Fecha de expiración | No | Based on event date |
| total_price | DECIMAL(10,2) | Precio total | Yes | Must be >= 0 |

**Lifecycle / States:**

```
[VALID] ──(check_in)──▶ [USED]
   │                      │
(cancel)              (N/A - terminal)
   ▼                      
[CANCELLED]          [EXPIRED] ◄── (auto_expire)
```

**Invariants (Business rules):**

```
INV-TICKET-001: Código único global
  - Rule: No pueden existir dos tickets con el mismo code
  - Implementation: Unique constraint + generation algorithm

INV-TICKET-002: Check-in único
  - Rule: Un ticket solo puede usarse una vez (VALID → USED)
  - Implementation: Estado inmutable + validación en check-in

INV-TICKET-003: Precio coherente
  - Rule: total_price debe ser la suma de los precios de los asientos
  - Implementation: Calculation in aggregate + validation
```

---

### Entity: Order

**Context:** Payment Processing

**Description:** Orden de compra que agrupa múltiples tickets con información de pago

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| order_id | INT | Identificador único | Yes | Auto-generated PK |
| user_id | INT | Usuario comprador | Yes | FK to user |
| event_id | INT | Evento de la compra | Yes | FK to event |
| total_amount | DECIMAL(10,2) | Monto total | Yes | Must be > 0 |
| currency | VARCHAR(3) | Moneda | Yes | Default: 'COP' |
| status | ENUM | Estado de la orden | Yes | created/pending_payment/paid/cancelled/refunded |
| created_at | TIMESTAMP | Fecha de creación | Yes | Auto-generated |

**Lifecycle / States:**

```
[CREATED] ──(initiate_payment)──▶ [PENDING_PAYMENT] ──(payment_success)──▶ [PAID]
    │                                     │                                    │
 (cancel)                              (timeout)                          (refund)
    ▼                                     ▼                                    ▼
[CANCELLED] ◄─────────────────────[CANCELLED]                          [REFUNDED]
```

**Invariants (Business rules):**

```
INV-ORDER-001: Monto positivo
  - Rule: total_amount debe ser mayor a 0
  - Implementation: CHECK constraint + validation

INV-ORDER-002: Timeout de pago
  - Rule: Una orden PENDING_PAYMENT expira en 15 minutos
  - Implementation: Background job + status transition

INV-ORDER-003: Reembolso condicionado
  - Rule: Solo órdenes PAID pueden ser reembolsadas
  - Implementation: State validation before refund
```

---

## System Value Objects

### Value Object: Email

**Description:** Dirección de correo electrónico válida

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| value | VARCHAR(255) | Email address |

**Validation rules:**
- Email debe tener formato válido: texto@dominio.extensión
- La extensión debe tener al menos 2 caracteres
- Longitud máxima: 255 caracteres
- Case insensitive (normalizado a lowercase)

```java
// Java implementation
public class Email {
    private static final Pattern EMAIL_PATTERN = 
        Pattern.compile("^[A-Za-z0-9+_.-]+@([A-Za-z0-9.-]+\\.[A-Za-z]{2,})$");
    
    private final String value;
    
    public Email(String email) {
        if (!EMAIL_PATTERN.matcher(email).matches()) {
            throw new DomainException("Invalid email format: " + email);
        }
        this.value = email.toLowerCase();
    }
    
    public String getValue() { return value; }
}
```

---

### Value Object: Money

**Description:** Cantidad monetaria con moneda

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| amount | DECIMAL(10,2) | Cantidad |
| currency | VARCHAR(3) | Código de moneda ISO |

**Validation rules:**
- Amount debe ser >= 0
- Currency debe ser código ISO válido (COP, USD, EUR)
- Precision: 2 decimales

```java
// Java implementation
public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, String currencyCode) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new DomainException("Amount cannot be negative");
        }
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = Currency.getInstance(currencyCode);
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new DomainException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency.getCurrencyCode());
    }
}
```

---

### Value Object: DateRange

**Description:** Rango de fechas con validaciones

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| startDate | LocalDate | Fecha de inicio |
| endDate | LocalDate | Fecha de fin |

**Validation rules:**
- endDate debe ser posterior a startDate
- Ambas fechas son requeridas
- No se permiten fechas en el pasado para eventos

---

## System Aggregates

### Aggregate: Event Management

**Aggregate Root:** Event

**Internal entities:**
- EventLayout — Layout gráfico JSONB del evento
- EventRating — Calificaciones y comentarios del evento
- EventMedia — Galería multimedia del evento

**Value Objects:**
- DateRange, Money, EventStatus

**Aggregate invariants:**

```
AGGR-INV-001: Layout coherente
  - Un Event puede tener máximo un EventLayout
  - El layout debe ser consistente con las secciones configuradas

AGGR-INV-002: Ratings válidos
  - Solo usuarios que compraron tickets pueden calificar
  - Una calificación por usuario por evento

AGGR-INV-003: Fechas inmutables después de publicación
  - Un evento PUBLISHED no puede cambiar sus fechas
  - Solo se permite cambio de estado a CANCELLED o FINISHED
```

---

### Aggregate: Ticket Purchase

**Aggregate Root:** Order

**Internal entities:**
- Ticket — Boletos individuales de la orden
- SeatTicket — Relación muchos-a-muchos entre tickets y asientos

**Value Objects:**
- Money, TicketCode, QRCode

**Aggregate invariants:**

```
AGGR-INV-001: Coherencia financiera
  - Order.total_amount = suma de Ticket.total_price
  - Currency consistente en toda la orden

AGGR-INV-002: Asientos únicos
  - Un asiento no puede estar en múltiples tickets de la misma orden
  - Cada ticket debe tener al menos un asiento asignado

AGGR-INV-003: Estado consistente
  - Si Order está PAID, todos sus Tickets deben estar VALID
  - Si Order está CANCELLED, todos sus Tickets deben estar CANCELLED
```

---

### Aggregate: Seating Management

**Aggregate Root:** EventSection

**Internal entities:**
- Seat — Asientos individuales
- SeatBlock — Bloques de asientos

**Value Objects:**
- SeatPosition (row + position), SeatType, SeatStatus

**Aggregate invariants:**

```
AGGR-INV-001: Capacidad coherente
  - EventSection.capacity = suma de Seat count en la sección
  - available_seats <= capacity siempre

AGGR-INV-002: Posiciones únicas
  - No dos asientos pueden tener misma row + position en una sección
  - Códigos de asientos únicos por sección

AGGR-INV-003: Estados válidos en tiempo real
  - Un asiento RESERVED tiene un timeout automático
  - Las transiciones de estado deben ser atómicas (WebSockets)
```

---

## Domain Services

### PriceCalculationService

**Context:** Payment Processing

**Responsibility:** Calcular precios totales considerando descuentos, impuestos y fees

```java
@DomainService
public class PriceCalculationService {
    
    public Money calculateOrderTotal(List<SeatTicket> seatTickets, 
                                   List<Discount> discounts) {
        Money subtotal = seatTickets.stream()
            .map(SeatTicket::getPrice)
            .reduce(Money.zero("COP"), Money::add);
            
        Money withDiscounts = discounts.stream()
            .reduce(subtotal, (total, discount) -> discount.apply(total));
            
        return withDiscounts;
    }
}
```

### SeatReservationService

**Context:** Seating & Venue

**Responsibility:** Gestionar reservas temporales en tiempo real con WebSockets

```java
@DomainService
public class SeatReservationService {
    
    public ReservationResult reserveSeats(List<SeatId> seatIds, UserId userId, Duration timeout) {
        // Validate all seats are available
        // Create temporary reservation
        // Start timeout countdown
        // Notify via WebSockets
        // Return reservation token
    }
    
    public void releaseExpiredReservations() {
        // Background job to release expired reservations
        // Notify clients via WebSockets
    }
}
```

---

## Business Rules Summary

### Critical Business Rules (Cannot be violated)

| Rule ID | Description | Contexts Affected |
|---------|-------------|------------------|
| BR-001 | Un asiento no puede venderse dos veces | Seating, Ticketing |
| BR-002 | Solo organizadores activados pueden crear eventos | Security, Events |
| BR-003 | Fechas de evento: finish_date > start_date | Events |
| BR-004 | Reservas temporales expiran en 15 minutos | Seating |
| BR-005 | Check-in de ticket es único e inmutable | Ticketing |
| BR-006 | Órdenes pending_payment expiran en 15 minutos | Payments |
| BR-007 | Total de orden = suma de precios de tickets | Payments, Ticketing |

### Soft Business Rules (Warnings, not errors)

| Rule ID | Description | Action |
|---------|-------------|--------|
| BR-S01 | Evento con menos de 10% asientos vendidos | Warning to organizer |
| BR-S02 | Usuario con más de 3 eventos cancelados | Flag for review |
| BR-S03 | Ticket sin check-in después del evento | Mark as no-show |

---

## Correlation with code structure

| Domain artifact | Java package | Key classes |
|----------------|-------------|-------------|
| Event Aggregate | `com.capysoft.tuevento.modules.event.domain` | `Event.java`, `EventLayout.java` |
| User Entity | `com.capysoft.tuevento.modules.security.domain` | `User.java`, `Role.java` |
| Seat Aggregate | `com.capysoft.tuevento.modules.seat.domain` | `Seat.java`, `EventSection.java` |
| Order Aggregate | `com.capysoft.tuevento.modules.payment.domain` | `Order.java`, `Ticket.java` |
| Money VO | `com.capysoft.tuevento.shared.domain.valueobject` | `Money.java` |
| Email VO | `com.capysoft.tuevento.shared.domain.valueobject` | `Email.java` |

> See full hexagonal structure in `05-architecture/hexagonal-architecture.md`