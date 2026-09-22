# Convenciones de Modelado de Datos — Tu Evento

> Este documento describe las convenciones de nomenclatura, patrones de modelado
> y — crucialmente — las **inconsistencias reales detectadas** entre los scripts SQL de los módulos.
> Fuente: análisis directo de los 13 scripts SQL en `TuEvento-Docs/05_Diseno_Software/SQL/`
>
> **Propósito:** servir como referencia para unificar el modelado antes de la implementación final
> y evitar que las inconsistencias actuales se propaguen al código de producción.

---

## 1. Motor de base de datos

**Motor oficial del proyecto:** PostgreSQL 17

Todos los módulos deben usar PostgreSQL 17. Características usadas en el proyecto:
- `SERIAL` / `GENERATED ALWAYS AS IDENTITY` para PKs autoincrement
- `JSONB` para datos semiestructurados (layouts, paletas de temas, webhooks, configuración de canales)
- `ENUM` definidos como tipos (`CREATE TYPE`) — no inline
- Coordenadas geográficas con `DECIMAL(9,6)`
- `TIMESTAMPTZ` recomendado (algunos módulos usan `TIMESTAMP` sin zona — ver inconsistencias)

---

## 2. Convenciones de nomenclatura

### Tablas
- `snake_case`, en singular o plural según contexto del módulo
- Nombres descriptivos del negocio, no técnicos

```sql
-- Correcto
user, event, seat_block, ticket_checkin, wallet_transaction

-- Incorrecto (no se usa en el proyecto)
tbl_user, UserEvent, eventTBL
```

### Columnas
- `snake_case` descriptivo
- FKs siguen el patrón: `[tabla_referenciada]_id`

```sql
user_id        → FK a tabla user
event_id       → FK a tabla event
seat_block_id  → FK a tabla seat_block
```

### ENUMs
- Definidos como tipos con `CREATE TYPE` antes de las tablas
- Nombres con sufijo `_enum` o `_status` según contexto

```sql
CREATE TYPE seat_status_enum AS ENUM ('available', 'reserved', 'sold', 'courtesy');
CREATE TYPE ticket_status AS ENUM ('draft', 'payment_pending', 'paid', ...);
```

### Índices
- Patrón: `idx_[tabla]_[campo(s)]`
- Índices parciales para condiciones de negocio frecuentes

```sql
CREATE UNIQUE INDEX unique_pending_petition
ON organizer_petition(user_id)
WHERE status = 'Pending';
```

---

## 3. Convenciones de PKs (Primary Keys)

El proyecto usa **tres estilos diferentes** de PK según el módulo — ver sección de inconsistencias para el análisis completo.

**Estilo recomendado para nuevos módulos:**
```sql
-- Para PostgreSQL puro (recomendado)
id  SERIAL PRIMARY KEY
-- O el estilo más moderno:
id  INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

---

## 4. Campos de auditoría

**Convención objetivo** (no aplicada uniformemente — ver inconsistencias):

```sql
created_at  TIMESTAMP    -- Cuándo se creó el registro
updated_at  TIMESTAMP    -- Última modificación
deleted_at  TIMESTAMP    -- NULL = activo; con valor = eliminado (soft delete)
created_by  INT          -- FK a user(user_id)
updated_by  INT          -- FK a user(user_id)
```

**Estado actual por módulo:**

| Módulo | `created_at` | `updated_at` | `deleted_at` | `created_by` |
|--------|-------------|-------------|-------------|-------------|
| Security | Parcial (solo algunos) | No | No | No |
| Profile | Parcial | Parcial | No | Parcial |
| Event | Parcial (`created_at` en rating) | No | No | `changed_by` en log |
| Section | No | No | No | No |
| Seat | `changed_at` en log | No | No | `changed_by` en log |
| Ticket | `created_at` en order | No | No | `changed_by` en log |
| Payment | `processed_at`, `requested_at` | No | No | No |
| Wallet | `created_at` en transaction | No | No | No |
| Notification | `sent_at`, `read_at` | No | No | No |
| Geolocation | No | No | No | No |
| Category | No | No | No | No |
| Language | `created_at` en language | `updated_at` en language | No | `created_by`, `updated_by` |
| Theme | `created_at`, `updated_at` en user_theme | Sí | No | No |

---

## 5. Soft Delete

**Convención objetivo:** usar `deleted_at TIMESTAMP` (NULL = activo; con valor = eliminado lógicamente) en lugar de `DELETE` físico.

**Estado actual:** el soft delete **no está implementado** en ningún módulo de los scripts SQL revisados. Todos usarían borrado físico si se ejecutara un `DELETE`. Esto debe corregirse antes del despliegue en producción para los módulos con datos auditables (tickets, pagos, usuarios).

**Excepción válida:** tablas de logs y auditoría (`seat_log`, `ticket_log`, `payment_log`) — en estas tablas el borrado físico nunca debería ocurrir, por lo que no necesitan soft delete.

---

## 6. Inconsistencias detectadas entre módulos

> Esta sección documenta diferencias reales encontradas al analizar los 13 scripts SQL.
> No son errores de diseño necesariamente — pueden ser decisiones intencionales por módulo —
> pero deben resolverse antes de la implementación final para garantizar coherencia del código.

---

### Inconsistencia A — Tres estilos distintos de Primary Key

| Estilo | Módulos que lo usan | Ejemplo |
|--------|-------------------|---------|
| `SERIAL PRIMARY KEY` | Security, Event, Ticket, Payment, Wallet, Theme | `user_id SERIAL PRIMARY KEY` |
| `INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` | Section | `event_section_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| `INT PRIMARY KEY` (sin autoincrement) | Profile, Seat, Notification, Geolocation, Category, Language | `seat_id INT PRIMARY KEY` |

**Problema real:** los módulos con `INT PRIMARY KEY` sin autoincrement no tienen mecanismo automático de generación de ID. Esto significa que la aplicación (Java/Spring) debe generar el ID antes del INSERT, o la tabla tiene un sequence separado no declarado en el script.

**Recomendación:** unificar en `SERIAL PRIMARY KEY` (PostgreSQL nativo y compatible con Spring Data JPA) o usar UUIDs con `DEFAULT gen_random_uuid()` para todos los módulos nuevos.

---

### Inconsistencia B — Mezcla de sintaxis MySQL y PostgreSQL

| Módulo | Sintaxis usada | Problema |
|--------|---------------|---------|
| `Event module.sql` | `AUTO_INCREMENT`, `ENUM('DRAFT','PUBLISHED',...)` inline | Sintaxis MySQL — no es válida en PostgreSQL |
| `Category module.sql` | `AUTO_INCREMENT`, `ENUM('DRAFT','PUBLISHED',...)` inline | Sintaxis MySQL — no es válida en PostgreSQL |
| Resto de módulos | `SERIAL`, `CREATE TYPE ... AS ENUM` | Sintaxis PostgreSQL correcta |

**Impacto:** si estos scripts se ejecutan directamente en PostgreSQL 17, `Event module.sql` y `Category module.sql` fallarán. Los ENUMs inline de MySQL (`ENUM('val1','val2')`) no son válidos en PostgreSQL — deben definirse como `CREATE TYPE` separado.

**Corrección necesaria para Event module:**
```sql
-- MySQL (incorrecto para PostgreSQL)
status ENUM('DRAFT','PUBLISHED','CANCELLED','COMPLETED')

-- PostgreSQL (correcto)
CREATE TYPE event_status AS ENUM ('DRAFT','PUBLISHED','CANCELLED','COMPLETED');
-- y luego en la tabla:
status event_status
```

---

### Inconsistencia C — Campos de auditoría ausentes en módulos críticos

Los módulos Section, Seat y Geolocation no tienen `created_at`, `updated_at` ni `deleted_at` en ninguna de sus tablas, a pesar de que contienen datos de negocio importantes (secciones de eventos, asientos, venues).

**Módulos sin auditoría básica:**

```sql
-- Section module — event_section no tiene campos de tiempo
CREATE TABLE event_section (
    event_section_id ...,
    event_id ...,
    capacity ...,
    price ...,
    is_active BOOLEAN
    -- Sin created_at, sin updated_at, sin deleted_at
);

-- Geolocation module — site no tiene auditoría
CREATE TABLE site (
    site_id ...,
    name ...,
    address ...
    -- Sin auditoría
);
```

**Impacto:** no es posible saber cuándo se creó o modificó una sección o un venue, ni filtrar registros borrados lógicamente.

---

### Inconsistencia D — `TIMESTAMP` sin zona vs. `TIMESTAMPTZ`

La plantilla de `models.md` recomienda `TIMESTAMPTZ` (con zona horaria), pero todos los scripts SQL usan `TIMESTAMP` (sin zona). En un sistema desplegado en Colombia (UTC-5) que podría tener usuarios en otras zonas, esto puede causar problemas de interpretación de fechas.

| Recomendación | Estado actual |
|--------------|--------------|
| `TIMESTAMPTZ NOT NULL DEFAULT NOW()` | `TIMESTAMP` sin zona en todos los módulos |

---

### Inconsistencia E — Relación `event_layout` — JSON vs. JSONB

| Módulo | Tipo usado | Implicación |
|--------|-----------|-------------|
| `Event module.sql` | `layout_data JSON` | No indexable, almacenado como texto |
| `Notification module.sql` | `config JSONB` | Indexable, binario — correcto |
| `Payment module.sql` | `payload JSONB` | Correcto |
| `Theme module.sql` | `default_palette JSONB` | Correcto |

**Recomendación:** cambiar `event_layout.layout_data` de `JSON` a `JSONB` para permitir consultas e índices sobre el layout del venue. Esto es especialmente importante dado el uso intensivo que el selector de asientos hace de esta estructura.

---

## 7. Decisiones de modelado documentadas

| Decisión | Módulo | Justificación |
|----------|--------|--------------|
| `BYTEA` para documento en `organizer_petition` | Security | El documento de solicitud se almacena como binario en la BD, no en S3. Puede reconsiderarse para archivos grandes — almacenar URL en S3 y referencia en BD |
| Auto-referencia en `event_comment_reply` | Event | Permite comentarios anidados sin límite de profundidad; requiere manejo cuidadoso en el ORM para evitar N+1 queries |
| Índice parcial en `organizer_petition` | Security | `WHERE status = 'Pending'` garantiza a nivel de BD que un usuario solo tenga una petición pendiente simultánea |
| `wallet_reference` como tabla separada | Wallet | Desacopla la transacción de la entidad que la originó; permite referenciar órdenes, pagos o reembolsos sin columnas nullable |
| `seat_ticket` como tabla pivote | Ticket | Un ticket puede cubrir múltiples asientos; el precio se fija en el momento de la compra para evitar variaciones posteriores |
| `UNIQUE(user_id, provider)` en `oauth_account` | Security | Un usuario no puede vincular el mismo proveedor OAuth más de una vez |

---

## 8. Acciones de corrección pendientes

> Registradas aquí para que el equipo las atienda antes del despliegue final.

| # | Acción | Módulo(s) afectado(s) | Prioridad | Responsable |
|---|--------|----------------------|-----------|-------------|
| 1 | Corregir sintaxis MySQL → PostgreSQL en Event y Category modules | Event, Category | Alta | Cristofer David Lozano (Tech Lead) |
| 2 | Unificar estilo de PK: decidir entre `SERIAL` o `GENERATED ALWAYS AS IDENTITY` | Todos | Media | Cristofer David Lozano (Tech Lead) |
| 3 | Agregar `created_at` y `updated_at` a Section, Seat y Geolocation | Section, Seat, Geolocation | Media | Cristofer David Lozano (Tech Lead) |
| 4 | Cambiar `event_layout.layout_data` de `JSON` a `JSONB` | Event | Media | Cristofer David Lozano (Tech Lead) |
| 5 | Evaluar `TIMESTAMP` → `TIMESTAMPTZ` en módulos críticos (Payment, Ticket) | Payment, Ticket | Media | ⬜ Estado por confirmar con el equipo |
| 6 | Evaluar soft delete (`deleted_at`) para tablas de negocio principales | User, Event, Ticket, Section | Baja | ⬜ Estado por confirmar con el equipo |

---

## Correlaciones

- Modelos de datos completos → `06-data/evidence/models.md`
- Scripts SQL fuente → `TuEvento-Docs/05_Diseno_Software/SQL/`
- Diagrama ER completo → `TuEvento-Docs/05_Diseno_Software/DER_TuEvento.png`
- Decisiones arquitectónicas relacionadas → `05-architecture/decisions/`
- Estrategia de migraciones → ⬜ Por documentar (herramienta: Flyway o Liquibase — sin evidencia confirmada)
