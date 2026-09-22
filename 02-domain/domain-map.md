# Domain Map — TuEvento Bounded Contexts

> **Domain Map for TuEvento** — Sistema de gestión de eventos desarrollado por CapySoft.
> Este mapa define los 7 bounded contexts principales basados en el análisis de los 14 módulos de base de datos.

---

## 1. Domain overview

**TuEvento** es una plataforma web y móvil que gestiona el ciclo completo de eventos desde la creación hasta la validación de ingreso. Los **organizadores** diseñan eventos con layout de asientos, gestionan ventas y controlan acceso. Los **asistentes** descubren eventos, compran boletos con selección de asientos en tiempo real, procesan pagos y validan ingreso via QR.

El sistema maneja múltiples roles (organizadores, asistentes, administradores), procesa pagos a través de Wompi, y soporta notificaciones multicanal con internacionalización completa.

---

## 2. Identified Bounded Contexts

### Bounded Context: Security & Identity

| Field | Value |
|-------|-------|
| **Name** | Security & Identity |
| **Responsibility** | Gestionar autenticación, autorización, roles y permisos de usuarios |
| **Owning team** | Ambos integrantes (Cristofer + Jhampier) |
| **Microservice(s)** | auth-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | User, Role, Permission, Session, Token |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| User | Entidad con credenciales y rol del sistema | No |
| Organizer | Usuario con rol ORGANIZER que puede crear eventos | Sí — en Events es "event owner" |
| Session | Token de acceso JWT + refresh token activo | No |
| Activation | Proceso de verificar email mediante código | No |

**Core entities:** `user`, `role`, `permission`, `auth_session`, `refresh_token`

---

### Bounded Context: Event Management

| Field | Value |
|-------|-------|
| **Name** | Event Management |
| **Responsibility** | Crear, publicar, gestionar eventos y su configuración de venue |
| **Owning team** | Cristofer (lead) + Jhampier (support) |
| **Microservice(s)** | event-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Event, Venue, Layout, Category, Rating |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Event | Evento con fechas, venue y capacidad | No |
| Layout | Configuración gráfica JSONB del venue | No |
| Site | Recinto físico donde se realiza el evento | Sí — en otros contextos es "location" |
| Status | Estado del evento (draft/published/cancelled/finished) | Sí — otros contextos tienen estados diferentes |

**Core entities:** `event`, `event_layout`, `site`, `category`, `event_rating`

---

### Bounded Context: Seating & Venue

| Field | Value |
|-------|-------|
| **Name** | Seating & Venue |
| **Responsibility** | Gestionar asientos, secciones y disponibilidad en tiempo real |
| **Owning team** | Cristofer (lead - WebSockets specialist) |
| **Microservice(s)** | seating-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Seat, Section, Block, Availability, Reservation |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Seat | Asiento individual con coordenadas y estado | No |
| Section | Zona del venue (platea, palco, general) | No |
| Block | Agrupación de asientos dentro de una sección | No |
| Status | Estado del asiento (available/reserved/sold/blocked) | Sí — diferente a event status |

**Core entities:** `seat`, `event_section`, `section_type`, `seat_block`

---

### Bounded Context: Ticketing & Orders

| Field | Value |
|-------|-------|
| **Name** | Ticketing & Orders |
| **Responsibility** | Generar boletos, gestionar órdenes y validar ingresos |
| **Owning team** | Jhampier (lead) + Cristofer (support) |
| **Microservice(s)** | ticket-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Ticket, Order, QR Code, Check-in, Validation |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Ticket | Boleto con QR único vinculado a asientos | No |
| Order | Orden de compra con múltiples tickets | Sí — en Payments es "payment order" |
| Check-in | Validación de ingreso al evento | No |
| Code | Código alfanumérico único del ticket | Sí — en Security es "activation code" |

**Core entities:** `ticket`, `order`, `seat_ticket`, `ticket_checkin`

---

### Bounded Context: Payment Processing

| Field | Value |
|-------|-------|
| **Name** | Payment Processing |
| **Responsibility** | Procesar pagos, reembolsos y gestión de billetera digital |
| **Owning team** | Cristofer (Wompi integration) + Jhampier (wallet) |
| **Microservice(s)** | payment-service, wallet-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Payment, Gateway, Wallet, Transaction, Refund |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Payment | Transacción procesada por Wompi | No |
| Order | Orden financiera vinculada a tickets | Sí — en Ticketing es "ticket order" |
| Wallet | Billetera virtual del usuario | No |
| Transaction | Movimiento de dinero (recarga/pago/reembolso) | No |

**Core entities:** `payment`, `wallet`, `wallet_transaction`, `refund`

---

### Bounded Context: Notifications & Communication

| Field | Value |
|-------|-------|
| **Name** | Notifications & Communication |
| **Responsibility** | Envío multicanal de notificaciones y gestión de entregas |
| **Owning team** | Jhampier (lead) + Cristofer (WebSockets) |
| **Microservice(s)** | notification-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Notification, Channel, Delivery, Template, Recipient |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Channel | Canal de entrega (email, push, SMS) | No |
| Notification | Mensaje enviado a usuario con estado de entrega | No |
| Template | Plantilla de mensaje con variables | No |
| Delivery | Estado de entrega (pending/sent/delivered/failed) | No |

**Core entities:** `notification`, `channel`, `notification_user`, `notification_type`

---

### Bounded Context: User Profile & Preferences

| Field | Value |
|-------|-------|
| **Name** | User Profile & Preferences |
| **Responsibility** | Gestionar perfiles públicos, preferencias e historial de usuarios |
| **Owning team** | Jhampier (lead) + Cristofer (activity tracking) |
| **Microservice(s)** | profile-service |
| **Database** | PostgreSQL (tuevento_db) |
| **Ubiquitous Language** | Profile, Preferences, Theme, Language, Activity |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Profile | Información pública del usuario | No |
| Theme | Tema visual (light/dark/purple/blue) | No |
| Language | Idioma de interfaz (ES/EN/PT/JA) | No |
| Activity | Registro de acciones del usuario | No |

**Core entities:** `profile`, `preferences`, `activity_log`, `theme`, `language`

---

## 3. Context Map

```
┌─────────────────────┐    events/users    ┌──────────────────────┐
│  Security &         │◄──────────────────►│  Event Management    │
│  Identity           │                    │                      │
│                     │                    │  Domain: eventos,    │
│  Domain: users,     │                    │  venues, categorías  │
│  auth, permisos     │                    └──────────────────────┘
└─────────────────────┘                             │
         │                                          │ event/layout
         │ user/profile                             ▼
         ▼                               ┌──────────────────────┐
┌─────────────────────┐                 │  Seating & Venue     │
│  User Profile &     │                 │                      │
│  Preferences        │                 │  Domain: asientos,   │
│                     │◄────────────────┤  secciones, bloques  │
│  Domain: perfil,    │  user/activity  └──────────────────────┘
│  preferencias, tema │                             │
└─────────────────────┘                             │ seat/reservation
                                                    ▼
┌─────────────────────┐    notifications  ┌──────────────────────┐
│  Notifications &    │◄──────────────────┤  Ticketing & Orders  │
│  Communication      │                   │                      │
│                     │                   │  Domain: tickets,    │
│  Domain: notif,     │                   │  órdenes, check-in   │
│  canales, entrega   │                   └──────────────────────┘
└─────────────────────┘                             │
         ▲                                          │ order/payment
         │                                          ▼
         │ payment/notification       ┌──────────────────────┐
         └────────────────────────────┤  Payment Processing  │
                                      │                      │
                                      │  Domain: pagos,      │
                                      │  wallet, reembolsos  │
                                      └──────────────────────┘
```

### Relationships table

| Context A | Relationship | Context B | Communication | Contract |
|-----------|-------------|-----------|--------------|----------|
| Security & Identity | U → D | Event Management | REST API | User authentication |
| Event Management | U → D | Seating & Venue | WebSockets + REST | Event and layout data |
| Seating & Venue | U → D | Ticketing & Orders | WebSockets + Events | Seat reservations |
| Ticketing & Orders | U → D | Payment Processing | Events + REST | Order payments |
| Payment Processing | U → D | Notifications | Domain Events | Payment notifications |
| User Profile | C/S | Security & Identity | Shared User ID | User profile sync |
| Notifications | OHS | ALL CONTEXTS | Event Bus | System-wide notifications |

**Legend:**
- **U → D**: Upstream → Downstream (dependency)  
- **C/S**: Customer/Supplier (negotiated relationship)
- **OHS**: Open Host Service (public protocol)

---

## 4. Core Domain, Supporting, Generic

| Bounded Context | Type | Investment | Justification |
|----------------|------|-----------|---------------|
| **Event Management** | Core Domain | MAXIMUM | Diferenciador clave: layout designer, configuración de venues |
| **Seating & Venue** | Core Domain | MAXIMUM | Competencia principal: tiempo real, WebSockets, UX selector |
| **Ticketing & Orders** | Core Domain | HIGH | Lógica de negocio específica: QR, check-in, validaciones |
| **Payment Processing** | Supporting | MEDIUM | Necesario pero no diferenciador (Wompi integration) |
| **Security & Identity** | Generic | MINIMUM | Commodity: JWT estándar, OAuth social |
| **Notifications** | Generic | MINIMUM | Commodity: SMTP, push notifications estándar |
| **User Profile** | Supporting | MEDIUM | Soporte necesario para personalización |

---

## 5. Modeling decisions

### How were these decisions made?

- **Source:** Análisis de 14 módulos de base de datos existentes
- **Method:** Agrupación por cohesión funcional y acoplamiento de datos
- **Team validation:** Cristofer (architecture) + Jhampier (implementation)
- **Date:** Septiembre 2024

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Separar Seating de Events | Mantener en Event Management | Seating tiene lógica tiempo real compleja (WebSockets) |
| Payment + Wallet en mismo contexto | Wallet como contexto separado | Wallet es extensión natural de Payment, no dominio independiente |
| Profile separado de Security | Profile dentro de Security | Profile tiene lógica de preferencias independiente de auth |
| 7 contextos vs 14 módulos | Un contexto por módulo | Algunos módulos son técnicos (idiomas, temas) no de dominio |

---

## 6. How to update this map

1. **Before adding microservices:** Verificar si pertenece a contexto existente
2. **Language evolution:** Si el lenguaje ubiquo cambia, revisar límites de contexto  
3. **Database changes:** Mantener sincronía con módulos SQL en `DataBase/MR/`
4. **Architecture sync:** Sincronizar con diagramas C4 en `05-architecture/overview.md`

> **Correlaciones importantes:**
> - Bounded Contexts → `09-microservices/service-catalog.md`
> - Módulos SQL → `TuEvento/DataBase/MR/DBML/`
> - Arquitectura → `05-architecture/decisions/`