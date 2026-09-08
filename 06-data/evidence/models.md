# Modelos de Datos por Módulo — Tu Evento

> Fuente primaria: scripts SQL en `TuEvento-Docs/05_Diseno_Software/SQL/`
> Motor de base de datos: **PostgreSQL 17** para todos los módulos.
> Diagrama ER completo: `TuEvento-Docs/05_Diseno_Software/DER_TuEvento.png`
> Diccionario de datos oficial: `TuEvento-Docs/05_Diseno_Software/Diccionario de datos Tu-Evento.xlsx`
>
> **Principio base:** cada módulo gestiona sus propias tablas.
> Ningún servicio accede directamente a las tablas de otro servicio — la comunicación es via API REST o eventos.

---

## ENUMs globales del sistema

Definidos en los scripts SQL por módulo. Se consolidan aquí para referencia cruzada.

| ENUM | Valores | Módulo que lo define |
|------|---------|---------------------|
| `event_status` | `DRAFT`, `PUBLISHED`, `CANCELLED`, `COMPLETED` | Event, Section, Payment, Ticket |
| `seat_status_enum` | `available`, `reserved`, `sold`, `courtesy` | Seat, Ticket |
| `seat_type_enum` | `regular`, `courtesy` | Seat, Ticket |
| `ticket_status` | `draft`, `payment_pending`, `paid`, `cancelled`, `refunded`, `used` | Ticket |
| `order_status` | `pending`, `processing`, `paid`, `failed`, `refunded` | Ticket, Payment |
| `payment_status_enum` | `pending`, `processing`, `paid`, `failed`, `refunded` | Payment, Profile |
| `payment_method_enum` | `card`, `pse`, `nequi`, `cash` | Payment |
| `gateway_enum` | `wompi` | Payment |
| `refund_status_enum` | `requested`, `approved`, `processed`, `failed` | Payment |
| `wallet_transaction_type` | `refund`, `top-up`, `withdrawal`, `payment` | Wallet |
| `wallet_transaction_status` | `pending`, `completed`, `failed` | Wallet |
| `wallet_entity_type` | `order`, `payment`, `refund` | Wallet |
| `organizer_petition_status` | `Pending`, `Approved`, `Rejected` | Security |
| `delivered_status_enum` | `Pending`, `Sent`, `Delivered`, `Failed` | Notification |

---

## Módulo: Security (`auth-service`)

**Motor:** PostgreSQL 17
**Justificación:** ACID obligatorio para sesiones, tokens y control de acceso. Datos críticos de autenticación.

| Tabla | Propósito |
|-------|-----------|
| `user_status` | Catálogo de estados de usuario (activo, suspendido, etc.) |
| `role` | Roles del sistema: ADMIN, ORGANIZER, ATTENDEE, ANONYMOUS |
| `permission` | Permisos granulares del sistema |
| `role_permission` | Tabla pivote rol ↔ permiso (N:M) |
| `user` | Entidad central del usuario — referenciada por todos los módulos |
| `user_status_history` | Auditoría de cambios de estado del usuario |
| `login_credentials` | Email + password_hash + last_login por usuario |
| `oauth_account` | Vinculación de cuenta con proveedor OAuth (Google, etc.) |
| `auth_session` | Sesiones activas con JWT, IP, user_agent, revocación |
| `refresh_token` | Tokens de renovación vinculados a sesión |
| `account_activation` | Código de activación de cuenta (6 dígitos, TTL) |
| `account_lockout` | Control de intentos fallidos y bloqueo temporal |
| `organizer_petition` | Solicitudes para obtener rol ORGANIZER — aprobación por ADMIN |
| `recover_password` | Código de recuperación de contraseña (6 dígitos, uso único) |
| `password_history` | Historial de contraseñas para prevenir reutilización |

**Campos clave:**
```sql
-- user
user_id        SERIAL PRIMARY KEY
alias          VARCHAR(50) NOT NULL UNIQUE
activated      BOOLEAN DEFAULT FALSE
role_id        INT → role(role_id)
user_status_id INT → user_status(user_status_id)

-- auth_session
access_token   VARCHAR(512)        -- JWT HS256
issued_at      TIMESTAMP
expires_at     TIMESTAMP           -- 24h
revoked        BOOLEAN DEFAULT FALSE
ip_address     VARCHAR(45)
user_agent     VARCHAR(200)

-- account_lockout
failed_attempts INT
window_start    TIMESTAMP
locked_until    TIMESTAMP          -- Bloqueo tras 5 intentos fallidos

-- organizer_petition
document        BYTEA NOT NULL     -- Documento adjunto del solicitante
status          organizer_petition_status
-- Índice parcial: solo 1 petición Pending por usuario
CREATE UNIQUE INDEX unique_pending_petition ON organizer_petition(user_id) WHERE status = 'Pending'
```

---

## Módulo: Profile (`auth-service` — profile module)

**Motor:** PostgreSQL 17
**Justificación:** Datos relacionales del perfil vinculados al usuario central.

| Tabla | Propósito |
|-------|-----------|
| `profile` | Nombre completo, foto (URL S3/MinIO), bio, ciudad del usuario |
| `profile_log` | Auditoría de cambios en el perfil |
| `preferences` | Idioma, tema e icono de notificaciones por usuario |
| `activity_log` | Registro de acciones del usuario por módulo |
| `review` | Calificaciones y comentarios de usuarios a eventos |

**Campos clave:**
```sql
-- profile
profile_id   INT PRIMARY KEY
user_id      INT NOT NULL → user(user_id)
city_id      INT NOT NULL → city(city_id)
full_name    VARCHAR(100) NOT NULL
photo_url    VARCHAR(255) NOT NULL    -- URL en AWS S3 / MinIO
bio          VARCHAR(255)

-- preferences
preference_id INT PRIMARY KEY
user_id       INT NOT NULL → user(user_id)
language_id   INT NOT NULL → language(language_id)
theme_id      INT NOT NULL → theme(theme_id)
notifications BOOLEAN DEFAULT FALSE

-- review
rating        INT CHECK (rating BETWEEN 1 AND 5)
comment       VARCHAR(255) NOT NULL
```

---

## Módulo: Event (`event-service`)

**Motor:** PostgreSQL 17
**Justificación:** Datos relacionales con integridad referencial entre evento, venue, organizador y medios. Layout almacenado como JSON/JSONB.

| Tabla | Propósito |
|-------|-----------|
| `event` | Entidad central del evento — ciclo de vida DRAFT→PUBLISHED→COMPLETED/CANCELLED |
| `event_layout` | Distribución visual del venue en formato JSON (diseñado exclusivamente desde web) |
| `event_media` | URLs de imágenes del evento almacenadas en S3/MinIO |
| `event_media_log` | Historial de versiones de medios del evento |
| `event_rating` | Calificaciones (1–5) y comentarios de asistentes al evento |
| `event_comment_reply` | Respuestas anidadas a comentarios (auto-referencia) |
| `event_status_log` | Auditoría de cambios de estado del evento |

**Campos clave:**
```sql
-- event
event_id        SERIAL PRIMARY KEY
user_id         INT NOT NULL → user(user_id)     -- Organizador
site_id         INT NOT NULL → site(site_id)     -- Venue
event_name      VARCHAR(100) NOT NULL
start_date      DATE NOT NULL
finish_date     DATE NOT NULL
status          ENUM('DRAFT','PUBLISHED','CANCELLED','COMPLETED')
is_public       BOOLEAN DEFAULT TRUE
available_seats INT NOT NULL
CHECK (finish_date > start_date)
UNIQUE (event_name, start_date, site_id)

-- event_layout
layout_data     JSON NOT NULL    -- Diseño del venue: secciones, bloques, asientos

-- event_rating
rating          INT CHECK (rating BETWEEN 1 AND 5)
is_visible      BOOLEAN DEFAULT TRUE  -- Moderación de contenido

-- event_comment_reply
parent_reply_id INT → event_comment_reply(reply_id)  -- Auto-referencia para anidado
```

---

## Módulo: Section (`event-service` — section module)

**Motor:** PostgreSQL 17
**Justificación:** Datos relacionales estrictos para garantizar consistencia de capacidad y precios.

| Tabla | Propósito |
|-------|-----------|
| `section_type` | Catálogo de tipos de sección: VIP, General, Tribuna, etc. |
| `event_section` | Sección específica de un evento: capacidad, asientos disponibles, precio, estado |

**Campos clave:**
```sql
-- event_section
event_section_id  INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
event_id          INT NOT NULL → event(event_id) ON DELETE CASCADE
section_type_id   INT NOT NULL → section_type(section_type_id)
capacity          INT NOT NULL CHECK (capacity > 0)
available_seats   INT NOT NULL CHECK (available_seats <= capacity)
price             DECIMAL(10,2) NOT NULL CHECK (price >= 0)
is_active         BOOLEAN DEFAULT TRUE
UNIQUE (event_id, section_type_id)  -- Sin secciones duplicadas por evento
```

---

## Módulo: Seat (`ticket-service` — seat module)

**Motor:** PostgreSQL 17
**Justificación:** Datos de asientos con auditoría de cambios de estado — crítico para el selector en tiempo real (SockJS).

| Tabla | Propósito |
|-------|-----------|
| `seat_block` | Agrupación de asientos dentro de una sección |
| `seat` | Asiento individual: fila, posición, tipo, estado actual |
| `seat_log` | Historial de cambios de estado del asiento (disponible → reservado → vendido) |

**Campos clave:**
```sql
-- seat
seat_id           INT PRIMARY KEY
seat_block_id     INT NOT NULL → seat_block(seat_block_id)
event_section_id  INT NOT NULL → event_section(event_section_id)
code              VARCHAR(20) NOT NULL    -- Código visible del asiento (ej: A-12)
row               INT NOT NULL
position          INT NOT NULL
type              seat_type_enum          -- regular | courtesy
status            seat_status_enum        -- available | reserved | sold | courtesy

-- seat_log (auditoría para el selector en tiempo real)
old_status        seat_status_enum
new_status        seat_status_enum
changed_at        TIMESTAMP
changed_by        INT → user(user_id)
reason            VARCHAR(255)
```

---

## Módulo: Ticket (`ticket-service`)

**Motor:** PostgreSQL 17
**Justificación:** ACID obligatorio para garantizar que cada entrada sea única y el check-in sea idempotente.

| Tabla | Propósito |
|-------|-----------|
| `order` | Orden de compra — agrupa tickets antes del pago |
| `ticket` | Entrada individual con código QR único |
| `seat_ticket` | Relación N:M entre asientos y tickets (un ticket puede cubrir múltiples asientos) |
| `ticket_checkin` | Registro de validación QR en puerta |
| `ticket_log` | Auditoría del ciclo de vida de cada ticket |

**Campos clave:**
```sql
-- order
status          order_status    -- pending | processing | paid | failed | refunded
currency        VARCHAR(3) DEFAULT 'COP'

-- ticket
code            VARCHAR(20) NOT NULL    -- Código legible por humanos
qr_code         VARCHAR(255)            -- URL o contenido del QR
status          ticket_status           -- draft | payment_pending | paid | cancelled | refunded | used
expiration_date DATE

-- ticket_checkin
checkin_time    TIMESTAMP
validated_by    INT → user(user_id)    -- Quién escaneó el QR

-- seat_ticket (tabla pivote)
PRIMARY KEY(seat_id, ticket_id)
price           DECIMAL(10,2)          -- Precio en el momento de la compra
```

---

## Módulo: Payment (`payment-service`)

**Motor:** PostgreSQL 17
**Justificación:** ACID obligatorio para transacciones financieras. JSONB para payloads de webhooks de Wompi.

| Tabla | Propósito |
|-------|-----------|
| `order` | Orden de compra vinculada a usuario y evento |
| `payment` | Transacción procesada por Wompi (tarjeta, PSE, Nequi, efectivo) |
| `payment_log` | Auditoría de cambios de estado del pago |
| `refund` | Solicitud y estado de reembolso |
| `transaction_webhook` | Payload crudo del webhook de Wompi (idempotencia y auditoría) |

**Campos clave:**
```sql
-- payment
gateway                 gateway_enum        -- 'wompi' (único valor actual)
gateway_transaction_id  VARCHAR(100)        -- ID de Wompi para conciliación
status                  payment_status_enum
payment_method          payment_method_enum -- card | pse | nequi | cash
amount                  DECIMAL(10,2) NOT NULL

-- transaction_webhook (crítico para idempotencia)
payload                 JSONB NOT NULL      -- Payload completo del webhook de Wompi
received_at             TIMESTAMP

-- refund
status                  refund_status_enum  -- requested | approved | processed | failed
requested_at            TIMESTAMP
processed_at            TIMESTAMP
```

---

## Módulo: Wallet (`payment-service` — wallet module)

**Motor:** PostgreSQL 17
**Justificación:** Transacciones financieras internas requieren ACID.

| Tabla | Propósito |
|-------|-----------|
| `wallet` | Billetera digital del usuario: saldo en COP |
| `wallet_transaction` | Cada movimiento de saldo: recarga, pago, retiro, reembolso |
| `wallet_reference` | Referencia al entidad que originó la transacción (orden, pago, reembolso) |

**Campos clave:**
```sql
-- wallet
balance         DECIMAL(10,2) DEFAULT 0
currency        VARCHAR(3) DEFAULT 'COP'

-- wallet_transaction
type            wallet_transaction_type    -- refund | top-up | withdrawal | payment
status          wallet_transaction_status  -- pending | completed | failed

-- wallet_reference
entity_type     wallet_entity_type         -- order | payment | refund
entity_id       INT                        -- ID de la entidad relacionada
```

---

## Módulo: Notification (`notification-service`)

**Motor:** PostgreSQL 17
**Justificación:** Registros relacionales de envío con trazabilidad de entrega por usuario y canal.

| Tabla | Propósito |
|-------|-----------|
| `notification_type` | Catálogo de tipos: confirmación de compra, recordatorio, cancelación, etc. |
| `channel` | Canal de envío: email (SMTP), push. Campo `config` en JSONB para credenciales |
| `notification` | Notificación enviada: asunto, cuerpo, canal, tipo, evento asociado |
| `notification_user` | Estado de entrega por usuario: Pending → Sent → Delivered / Failed |

**Campos clave:**
```sql
-- channel
config              JSONB       -- Configuración del canal (SMTP server, push keys, etc.)

-- notification_user
email_address       VARCHAR(255) NOT NULL    -- Dirección de entrega
delivered_status    delivered_status_enum    -- Pending | Sent | Delivered | Failed
error_message       TEXT                     -- Motivo del fallo si aplica
read_at             TIMESTAMP                -- Cuándo fue leída
UNIQUE (notification_id, user_id)            -- Sin duplicados por usuario
```

---

## Módulo: Geolocation (`event-service` — geolocation module)

**Motor:** PostgreSQL 17
**Justificación:** Datos maestros geográficos de Colombia: departamentos, ciudades y venues.

| Tabla | Propósito |
|-------|-----------|
| `department` | Departamentos de Colombia |
| `city` | Ciudades vinculadas a departamento |
| `site` | Venue físico: nombre, dirección, coordenadas GPS, capacidad total |

**Campos clave:**
```sql
-- site
name        VARCHAR(150) NOT NULL
address     VARCHAR(200) NOT NULL
capacity    INT
latitude    DECIMAL(9,6)    -- Coordenadas GPS del venue
longitude   DECIMAL(9,6)
city_id     INT NOT NULL → city(city_id)
```

---

## Módulo: Category (`event-service` — category module)

**Motor:** PostgreSQL 17
**Justificación:** Árbol de categorías con auto-referencia para jerarquía padre-hijo.

| Tabla | Propósito |
|-------|-----------|
| `category` | Categoría jerárquica con auto-referencia (`dad_id`) |
| `category_event` | Relación N:M entre categorías y eventos |

**Campos clave:**
```sql
-- category
dad_id      INT → category(category_id) ON DELETE SET NULL   -- Padre de la categoría
active      BOOLEAN DEFAULT TRUE
visible     BOOLEAN DEFAULT TRUE
UNIQUE (category_id, event_id)    -- En category_event: sin duplicados
```

---

## Módulo: Language (`language-service`)

**Motor:** PostgreSQL 17
**Justificación:** Internacionalización: traducciones de entidades del dominio a ES, EN, PT, JA.

| Tabla | Propósito |
|-------|-----------|
| `language` | Catálogo de idiomas soportados: ES, EN, PT, JA |
| `event_translation` | Nombre y descripción del evento por idioma |
| `category_translation` | Nombre y descripción de categoría por idioma |
| `profile_translation` | Bio del perfil por idioma |
| `theme_translation` | Nombre y descripción del tema por idioma |
| `review_translation` | Comentario de reseña por idioma |

**Patrón de traducción (consistente en todas las tablas):**
```sql
-- Ejemplo: event_translation
event_id            INT NOT NULL → event(event_id)
language_id         INT NOT NULL → language(language_id)
translated_name     VARCHAR(100)
translated_description VARCHAR(255)
source              VARCHAR(50)     -- manual | auto-translated
status              VARCHAR(20)     -- draft | published
UNIQUE (event_id, language_id)      -- Una traducción por idioma por entidad
```

---

## Módulo: Theme (`auth-service` — theme module)

**Motor:** PostgreSQL 17
**Justificación:** JSONB para la paleta de colores; datos relacionales para el historial de personalización.

| Tabla | Propósito |
|-------|-----------|
| `theme` | Tema visual del sistema: light, dark, purple, blue — paleta en JSONB |
| `user_theme` | Tema activo por usuario |
| `theme_customization` | Personalizaciones individuales de propiedades del tema |
| `theme_log` | Historial de cambios de personalización |

**Campos clave:**
```sql
-- theme
default_palette     JSONB    -- Paleta de colores por defecto del tema

-- user_theme
is_active           BOOLEAN DEFAULT TRUE
UNIQUE(user_id, theme_id)    -- Un tema activo por usuario

-- theme_customization
property            VARCHAR(100)    -- Propiedad CSS o de diseño
value               VARCHAR(255)    -- Valor personalizado
```

---

## Correlaciones

- Inconsistencias y convenciones reales → `06-data/evidence/modeling-conventions.md`
- Diagrama ER completo → `TuEvento-Docs/05_Diseno_Software/DER_TuEvento.png`
- Diccionario de datos oficial → `TuEvento-Docs/05_Diseno_Software/Diccionario de datos Tu-Evento.xlsx`
- Scripts SQL fuente → `TuEvento-Docs/05_Diseno_Software/SQL/`
- Datos por servicio en detalle → `09-microservices/services/`
- Contratos de API que exponen estos datos → `07-api/contracts/openapi/`
