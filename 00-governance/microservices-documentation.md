# Estándar de Documentación por Microservicio - TuEvento

> Define exactamente qué documentos debe tener cada microservicio, quién los escribe,
> cuándo se crean y cuándo deben actualizarse. El incumplimiento bloquea el merge.

---

## Estructura requerida para cada servicio

Cada microservicio de TuEvento vive en `[Carpeta]/services/[nombre-servicio]/` y DEBE tener:

```
services/[nombre-servicio]/
├── README.md         ⭐ REQUERIDO desde Sprint 1
├── data-model.md     ⭐ REQUERIDO antes de crear migraciones
├── events.md         ⭐ REQUERIDO si el servicio emite/consume eventos
├── decisions.md      🔵 RECOMENDADO — decisiones técnicas internas del servicio
└── runbook.md        🟢 REQUERIDO antes del primer deploy a staging
```

Y su contrato OpenAPI en:
```
07-api/contracts/openapi/[nombre-servicio].yaml   ⭐ REQUERIDO si expone endpoints REST
```

---

## README.md — Ficha técnica del servicio

**Cuándo crearlo:** Al inicio del sprint donde se crea el servicio
**Responsable:** Desarrollador asignado al servicio
**Actualizar cuando:** Cambie responsabilidad, puertos o dependencias entre servicios

Contenido mínimo:

| Sección | Qué debe decir |
|---------|----------------|
| Responsabilidad | Una oración: qué hace y de qué datos es el dueño autoritativo |
| Ubicación arquitectónica | Puerto, repositorio, motor de BD, con quién se comunica |
| Responsabilidades (qué SÍ hace) | Lista de responsabilidades concretas |
| Fuera de scope (qué NO hace) | Qué delegó y a quién |
| Cómo ejecutar localmente | Comandos exactos, deben funcionar |
| Documentos relacionados | Links a los otros archivos del servicio |

---

## data-model.md — Modelo de datos del servicio

**Cuándo crearlo:** Antes del primer script de migración
**Responsable:** Desarrollador asignado al servicio
**Actualizar cuando:** Se cree o modifique una tabla/colección

Contenido mínimo:
- Diagrama ER (Mermaid) de las tablas del servicio
- Descripción de cada tabla con sus columnas, tipos, constraints y propósito
- Justificación del motor de BD elegido (PostgreSQL, MySQL, MongoDB, Redis, etc.)
- Estrategia de migración (Flyway, Liquibase o scripts manuales)

**Regla:** Un campo cuya razón de existir no es obvia DEBE tener comentario en el diagrama.

---

## events.md — Catálogo de eventos del servicio

**Cuándo crearlo:** Cuando el servicio publica o consume su primer evento de dominio
**Responsable:** Desarrollador asignado al servicio
**Actualizar cuando:** Se agregue, modifique o elimine un evento

Contenido mínimo:
- Tabla de eventos publicados: nombre, topic/exchange, cuándo se emite, schema
- Tabla de eventos consumidos: nombre, de qué servicio viene, qué acción dispara
- Schema del payload (puede referenciar el contrato OpenAPI o Avro)

Ver estructura estándar de eventos: `02-domain/domain-events.md`

---

## decisions.md — Decisiones técnicas del servicio

**Cuándo crearlo:** Cuando el equipo toma una decisión técnica no obvia sobre el servicio
**Responsable:** Quien tomó la decisión
**Actualizar cuando:** Se tome una nueva decisión o se revoque una anterior

Formato recomendado: miniADR (sin el rigor completo de un ADR arquitectónico):
```markdown
### Decisión: [nombre corto]
**Fecha:** [fecha]
**Contexto:** [qué problema se estaba resolviendo]
**Decisión:** [qué se decidió]
**Consecuencias:** [trade-offs conocidos]
```

---

## runbook.md — Manual de operaciones del servicio

**Cuándo crearlo:** Antes del primer deploy a staging
**Responsable:** Desarrollador responsable + DevOps
**Actualizar cuando:** Se descubra un nuevo problema operacional o cambie un procedimiento

Contenido mínimo:
- Cómo verificar que el servicio está saludable (health check, métricas clave)
- Síntomas conocidos y sus causas: "Si ves X, el problema es Y, la solución es Z"
- Cómo hacer rollback del servicio
- Cómo ejecutar migraciones de BD
- Alertas configuradas y qué hacer cuando se disparen

---

## Contrato OpenAPI

**Cuándo crearlo:** Antes de implementar el primer endpoint del servicio (API-first)
**Responsable:** Desarrollador asignado al servicio
**Actualizar cuando:** Se agregue, modifique o elimine un endpoint

**Regla API-First:** El contrato se escribe ANTES que el código. Los tests de contrato validan
que el código cumple el contrato, no al revés.

---

## Servicios identificados en TuEvento

Basado en el análisis de los módulos SQL existentes:

| # | Servicio | Módulos SQL relacionados | Responsabilidad |
|---|----------|-------------------------|-----------------|
| 01 | **auth-service** | Security, Profile | Autenticación, autorización y perfiles |
| 02 | **event-service** | Event, Category | Gestión de eventos y categorización |
| 03 | **payment-service** | Payment, Wallet | Procesamiento de pagos y billeteras |
| 04 | **ticket-service** | Ticket, Seat | Gestión de boletos y asientos |
| 05 | **notification-service** | Notification | Envío de notificaciones |
| 06 | **location-service** | Geolocation | Gestión de ubicaciones y sitios |
| 07 | **api-gateway** | — | Punto de entrada y enrutamiento |

---

## Cómo agregar un nuevo microservicio

1. Crear carpeta `services/[NN-nombre]/` 
2. Actualizar catálogo de servicios con la nueva entrada
3. Crear `services/[NN-nombre]/README.md` usando plantilla
4. Crear contrato OpenAPI en `07-api/contracts/openapi/[nombre].yaml`
5. Crear PR con al menos README.md y contrato API esbozado

---

## Plantillas disponibles

- **README del servicio:** `_template/service/README.md`
- **Contrato OpenAPI:** `07-api/contracts/openapi/_template-service.yaml`
- **Modelo de datos:** `_template/service/data-model.md`
- **Catálogo de eventos:** `_template/service/events.md`

---

## Correlaciones

- Módulos SQL del proyecto → `04_Diseno_Software/SQL/`
- Arquitectura general → `04_Diseno_Software/Diagrama de arquitectura.png`
- Reglas generales de documentación → `00-governance/documentation-rules.md`
- Diagramas UML por módulo → `02_Analisis_Software/class diagram/`