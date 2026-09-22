# Functional Requirements (FR) — TuEvento

> **Requisitos funcionales de TuEvento** — Especificación de funciones que el sistema debe proporcionar.
> Basado en el análisis de dominio, módulos SQL y arquitectura del proyecto.

---

## FR-001: Security & Identity Management

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-001 | Security | El sistema debe permitir registro de usuarios con validación de email | Domain Analysis | High |
| FR-002 | Security | El sistema debe autenticar usuarios con email y contraseña | Domain Analysis | High |
| FR-003 | Security | El sistema debe soportar autenticación social (OAuth) | Tech Stack | Medium |
| FR-004 | Security | El sistema debe generar JWT tokens con refresh token | Architecture | High |
| FR-005 | Security | El sistema debe permitir recuperación de contraseña via email | Domain Analysis | High |
| FR-006 | Security | El sistema debe gestionar roles (USER, ORGANIZER, ADMIN) | Security Module | High |
| FR-007 | Security | El sistema debe permitir peticiones para ser organizador | Security Module | Medium |
| FR-008 | Security | El sistema debe bloquear cuentas después de 5 intentos fallidos | Security Module | High |
| FR-009 | Security | El sistema debe activar cuentas mediante código de 6 dígitos | Security Module | High |
| FR-010 | Security | El sistema debe mantener historial de contraseñas | Security Module | Medium |

---

## FR-002: Event Management

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-011 | Events | El sistema debe permitir crear eventos con información básica | Event Module | High |
| FR-012 | Events | El sistema debe permitir configurar layout gráfico de venues (JSONB) | Event Module | High |
| FR-013 | Events | El sistema debe gestionar estados de evento (DRAFT/PUBLISHED/CANCELLED/FINISHED) | Event Module | High |
| FR-014 | Events | El sistema debe permitir categorizar eventos jerárquicamente | Category Module | Medium |
| FR-015 | Events | El sistema debe permitir subir galería multimedia de eventos | Event Module | Medium |
| FR-016 | Events | El sistema debe permitir calificar y comentar eventos | Event Module | Low |
| FR-017 | Events | El sistema debe permitir respuestas anidadas a comentarios | Event Module | Low |
| FR-018 | Events | El sistema debe validar fechas de evento (finish > start) | Event Module | High |
| FR-019 | Events | El sistema debe asociar eventos con sitios/venues | Geolocation Module | High |
| FR-020 | Events | El sistema debe auditar cambios de estado de eventos | Event Module | Medium |

---

## FR-003: Seating & Venue Management

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-021 | Seating | El sistema debe gestionar secciones de eventos (platea, palco, general) | Section Module | High |
| FR-022 | Seating | El sistema debe crear asientos individuales con coordenadas | Seat Module | High |
| FR-023 | Seating | El sistema debe gestionar tipos de asiento (standard/vip/accessible) | Seat Module | Medium |
| FR-024 | Seating | El sistema debe gestionar estados de asiento en tiempo real (available/reserved/sold/blocked) | Seat Module | High |
| FR-025 | Seating | El sistema debe reservar asientos temporalmente (15 min) | WebSockets | High |
| FR-026 | Seating | El sistema debe notificar cambios de estado via WebSockets | WebSockets | High |
| FR-027 | Seating | El sistema debe liberar reservas expiradas automáticamente | Background Jobs | High |
| FR-028 | Seating | El sistema debe validar unicidad de posiciones por sección | Seat Module | High |
| FR-029 | Seating | El sistema debe auditar cambios de estado de asientos | Seat Module | Medium |
| FR-030 | Seating | El sistema debe agrupar asientos en bloques dentro de secciones | Seat Module | Medium |

---

## FR-004: Ticketing & Orders

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-031 | Ticketing | El sistema debe generar órdenes de compra con múltiples tickets | Ticket Module | High |
| FR-032 | Ticketing | El sistema debe generar códigos únicos para tickets | Ticket Module | High |
| FR-033 | Ticketing | El sistema debe generar códigos QR para tickets | Ticket Module | High |
| FR-034 | Ticketing | El sistema debe gestionar estados de ticket (valid/used/cancelled/expired) | Ticket Module | High |
| FR-035 | Ticketing | El sistema debe permitir check-in via escaneo QR | Ticket Module | High |
| FR-036 | Ticketing | El sistema debe validar que un ticket solo se use una vez | Ticket Module | High |
| FR-037 | Ticketing | El sistema debe asociar tickets con asientos específicos | Ticket Module | High |
| FR-038 | Ticketing | El sistema debe calcular precios totales por orden | Ticket Module | High |
| FR-039 | Ticketing | El sistema debe auditar cambios de estado de tickets | Ticket Module | Medium |
| FR-040 | Ticketing | El sistema debe expirar órdenes pending_payment después de 15 min | Background Jobs | High |

---

## FR-005: Payment Processing

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-041 | Payments | El sistema debe integrar con gateway Wompi | Payment Module | High |
| FR-042 | Payments | El sistema debe soportar múltiples métodos de pago (tarjeta, PSE, Nequi, efectivo) | Payment Module | High |
| FR-043 | Payments | El sistema debe gestionar estados de pago (pending/approved/rejected/refunded/failed) | Payment Module | High |
| FR-044 | Payments | El sistema debe procesar webhooks de Wompi | Payment Module | High |
| FR-045 | Payments | El sistema debe gestionar reembolsos con estados | Payment Module | Medium |
| FR-046 | Payments | El sistema debe auditar transacciones de pago | Payment Module | High |
| FR-047 | Payments | El sistema debe manejar moneda COP por defecto | Payment Module | High |
| FR-048 | Payments | El sistema debe almacenar transaction IDs del gateway | Payment Module | High |
| FR-049 | Payments | El sistema debe validar montos positivos en pagos | Payment Module | High |
| FR-050 | Payments | El sistema debe procesar pagos de forma asíncrona | Architecture | High |

---

## FR-006: Wallet Management

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-051 | Wallet | El sistema debe crear billetera digital por usuario | Wallet Module | Medium |
| FR-052 | Wallet | El sistema debe permitir recarga de billetera | Wallet Module | Medium |
| FR-053 | Wallet | El sistema debe permitir pago con saldo de billetera | Wallet Module | Medium |
| FR-054 | Wallet | El sistema debe gestionar tipos de transacción (deposit/withdrawal/payment/refund) | Wallet Module | Medium |
| FR-055 | Wallet | El sistema debe mantener balance actualizado en tiempo real | Wallet Module | Medium |
| FR-056 | Wallet | El sistema debe referenciar origen de cada transacción | Wallet Module | Medium |
| FR-057 | Wallet | El sistema debe generar resumen de movimientos | Wallet Module | Low |
| FR-058 | Wallet | El sistema debe validar saldo suficiente antes de pagos | Wallet Module | High |
| FR-059 | Wallet | El sistema debe procesar reembolsos a billetera | Wallet Module | Medium |
| FR-060 | Wallet | El sistema debe auditar todas las transacciones | Wallet Module | Medium |

---

## FR-007: Notifications & Communication

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-061 | Notifications | El sistema debe enviar notificaciones por email | Notification Module | High |
| FR-062 | Notifications | El sistema debe enviar notificaciones push móviles | Notification Module | Medium |
| FR-063 | Notifications | El sistema debe gestionar canales de notificación | Notification Module | Medium |
| FR-064 | Notifications | El sistema debe usar plantillas de notificación | Notification Module | Medium |
| FR-065 | Notifications | El sistema debe rastrear estado de entrega (pending/sent/delivered/failed) | Notification Module | Medium |
| FR-066 | Notifications | El sistema debe enviar confirmación de registro | User Registration | High |
| FR-067 | Notifications | El sistema debe enviar tickets por email con QR | Ticket Purchase | High |
| FR-068 | Notifications | El sistema debe notificar cambios de evento a asistentes | Event Changes | Medium |
| FR-069 | Notifications | El sistema debe enviar recordatorios de evento | Event Reminders | Low |
| FR-070 | Notifications | El sistema debe permitir opt-out de notificaciones | User Preferences | Medium |

---

## FR-008: User Profile & Preferences

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-071 | Profile | El sistema debe gestionar perfil público de usuarios | Profile Module | Medium |
| FR-072 | Profile | El sistema debe permitir configurar preferencias de idioma | Language Module | Medium |
| FR-073 | Profile | El sistema debe permitir seleccionar tema visual (light/dark/purple/blue) | Theme Module | Low |
| FR-074 | Profile | El sistema debe mantener historial de actividad del usuario | Profile Module | Low |
| FR-075 | Profile | El sistema debe mostrar historial de compras | Profile Module | Medium |
| FR-076 | Profile | El sistema debe permitir edición de información personal | Profile Module | Medium |
| FR-077 | Profile | El sistema debe asociar perfil con ciudad/departamento | Geolocation Module | Medium |
| FR-078 | Profile | El sistema debe auditar cambios de perfil | Profile Module | Low |
| FR-079 | Profile | El sistema debe soportar múltiples idiomas (ES/EN/PT/JA) | Language Module | Medium |
| FR-080 | Profile | El sistema debe personalizar tema por usuario | Theme Module | Low |

---

## FR-009: Geolocation & Venues

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-081 | Geolocation | El sistema debe gestionar estructura geográfica (departamentos/ciudades) | Geolocation Module | Medium |
| FR-082 | Geolocation | El sistema debe gestionar información de sitios/venues | Geolocation Module | High |
| FR-083 | Geolocation | El sistema debe almacenar coordenadas GPS de venues | Geolocation Module | Medium |
| FR-084 | Geolocation | El sistema debe validar capacidad de venues | Geolocation Module | Medium |
| FR-085 | Geolocation | El sistema debe asociar eventos con venues específicos | Event-Location Link | High |

---

## FR-010: File & Media Management

| ID | Module | Description | Source | Priority |
|----|--------|-------------|--------|----------|
| FR-086 | Storage | El sistema debe gestionar subida de archivos | Storage Module | Medium |
| FR-087 | Storage | El sistema debe soportar múltiples proveedores de storage | Storage Module | Low |
| FR-088 | Storage | El sistema debe categorizar archivos por tipo | Storage Module | Medium |
| FR-089 | Storage | El sistema debe auditar operaciones de archivos | Storage Module | Low |
| FR-090 | Storage | El sistema debe generar URLs firmadas para acceso seguro | Storage Module | Medium |

---

## Functional Requirements Summary

| Module | Total FRs | High Priority | Medium Priority | Low Priority |
|--------|-----------|---------------|----------------|--------------|
| Security & Identity | 10 | 7 | 3 | 0 |
| Event Management | 10 | 4 | 4 | 2 |
| Seating & Venue | 10 | 6 | 4 | 0 |
| Ticketing & Orders | 10 | 8 | 2 | 0 |
| Payment Processing | 10 | 8 | 2 | 0 |
| Wallet Management | 10 | 1 | 8 | 1 |
| Notifications | 10 | 3 | 6 | 1 |
| User Profile | 10 | 0 | 6 | 4 |
| Geolocation | 5 | 2 | 3 | 0 |
| File Management | 5 | 0 | 4 | 1 |
| **TOTAL** | **90** | **39** | **42** | **9** |

---

## Implementation Priority Matrix

### Must Have (High Priority) - MVP Core
**39 FRs** que son críticos para la funcionalidad básica del sistema:
- Autenticación y roles básicos
- Creación y publicación de eventos  
- Gestión de asientos en tiempo real
- Compra de tickets con QR
- Procesamiento de pagos via Wompi

### Should Have (Medium Priority) - Enhanced MVP  
**42 FRs** que mejoran la experiencia pero no bloquean funcionalidad core:
- Características avanzadas de perfil
- Billetera digital
- Sistema de notificaciones completo
- Gestión avanzada de venues

### Could Have (Low Priority) - Future Enhancements
**9 FRs** para versiones futuras:
- Sistema de comentarios y ratings
- Personalización avanzada de temas
- Features de comunidad y social

---

## Correlations with other sections

- **Domain entities** → `02-domain/evidence/entities-and-rules.md`
- **User Stories** → `04-requirements/evidence/user-stories.md` 
- **API Contracts** → `07-api/contracts/openapi/`
- **Service Architecture** → `09-microservices/service-catalog.md`
- **Testing Strategy** → `11-quality/testing-strategy.md`