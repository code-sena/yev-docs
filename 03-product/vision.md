# Visión del Producto — Tu Evento

> La visión es la estrella polar del equipo. Todos los sprints, decisiones de diseño
> y trade-offs se evalúan contra esta visión.
> Fuentes: `README.md` del repositorio del proyecto + `00-governance/evidence/agile-conventions.md` + `01-context/evidence/overview.md`

---

## Declaración de visión

**Para** organizadores de eventos y asistentes en Colombia
**que** necesitan crear, publicar y asistir a eventos de forma digital con reserva de asientos y pagos seguros
**Tu Evento**
**es una** plataforma web y móvil de gestión de eventos
**que** permite diseñar venues con asientos en tiempo real, procesar pagos con métodos colombianos (Wompi) y validar ingresos mediante código QR.
**A diferencia de** procesos manuales, hojas de cálculo o plataformas internacionales sin soporte para PSE/Nequi,
**nuestro producto** combina un diseñador de venue web, un selector de asientos en tiempo real (SockJS), una app móvil cross-platform (Expo + React Native) y una integración nativa con la pasarela de pagos colombiana Wompi.

---

## Misión del equipo

**CapySoft** (Grupo 11 — SENA, Ficha 314556) existe para construir una solución completa de gestión de eventos que demuestre dominio técnico en arquitectura N-Layer con DDD, desarrollo full-stack y prácticas profesionales de ingeniería de software, como proyecto de grado del programa Tecnólogo en Análisis y Desarrollo de Software.

---

## Pilares estratégicos

| Pilar | Descripción | Métrica de éxito |
|-------|-------------|-----------------|
| **Experiencia del asistente** | Selector de asientos en tiempo real sin conflictos de reserva simultánea | Latencia SockJS < 50ms; 0 dobles reservas en el mismo asiento |
| **Herramientas del organizador** | Diseñador visual de venue exclusivo en web; ciclo de vida completo del evento | Organizador puede publicar un evento end-to-end sin soporte técnico |
| **Pagos colombianos** | Integración nativa con Wompi: tarjeta, PSE, Nequi, efectivo COP | Tasa de éxito de transacciones Wompi en sandbox > 95% |
| **Multiplataforma** | Backend único; frontend web (Vite + React) y móvil (Expo + React Native) coherentes | Funcionalidad de compra y QR disponible en Android e iOS |

---

## Roadmap por fases

> **Nota de honestidad:** los estados de cada fase se basan únicamente en fuentes confirmadas.
> Fuente 1: `README.md` del repositorio — describe qué módulos del backend están implementados.
> Fuente 2: `agile-conventions.md` — registra el sprint actual con fecha real.
> Cualquier módulo sin evidencia directa en esas fuentes se marca como **⬜ Estado por confirmar**.

### Fase 1 — Fundamentos (Auth + Eventos + Layout)

| Módulo | Descripción | Estado | Fuente |
|--------|-------------|--------|--------|
| Security / Auth | Registro, login, OAuth, JWT HS256, Refresh Token, recuperación de contraseña, bloqueo de cuenta | ✅ Implementado | README — "Backend is currently the active component under development"; módulo Security listado con todas sus tablas |
| Event | Creación, edición, publicación, cancelación de eventos; gestión de medios y layout JSONB | ✅ Implementado | README — módulo Event listado con tablas completas |
| Section / Layout web | Diseño visual del venue, configuración de secciones y precios desde web | ✅ Implementado | README — módulo Section listado; RF19 descrito como funcional |
| Category / Geolocation | Categorización jerárquica y gestión de venues/ciudades | ✅ Implementado | README — módulos listados con tablas |

### Fase 2 — Tiempo real (Sprint activo)

| Módulo | Descripción | Estado | Fuente |
|--------|-------------|--------|--------|
| Seat Selection + WebSockets | Selector de asientos sincronizado en tiempo real vía SockJS; web y móvil | 🟡 En progreso | `agile-conventions.md`: *"Sprint actual: Módulo Seat Selection + WebSockets — 13 agosto a 8+ septiembre 2026"* |

### Fase 3 — Monetización

| Módulo | Descripción | Estado | Fuente |
|--------|-------------|--------|--------|
| Payment / Wompi | Procesamiento de órdenes, pagos (tarjeta, PSE, Nequi, efectivo), reembolsos, webhooks | ⬜ Estado por confirmar | README lista tablas y RF17, pero no hay evidencia directa del estado de implementación |
| Wallet | Billetera digital por usuario: recarga, pago, retiro, reembolso | ⬜ Estado por confirmar | README lista tablas de wallet; sin evidencia de estado |

### Fase 4 — Entradas y Validación

| Módulo | Descripción | Estado | Fuente |
|--------|-------------|--------|--------|
| Ticket / QR | Emisión de entradas con QR único, ciclo de vida del ticket, check-in | ⬜ Estado por confirmar | README lista tablas y RF11/RF24; sin evidencia de estado |

### Fase 5 — Comunicación y Personalización

| Módulo | Descripción | Estado | Fuente |
|--------|-------------|--------|--------|
| Notification | Notificaciones push y email multicanal con seguimiento de entrega | ⬜ Estado por confirmar | README lista tablas y RF16; sin evidencia de estado |
| Theme / Language | Temas de interfaz (light, dark, purple, blue) e internacionalización ES/EN/PT/JA | ⬜ Estado por confirmar | README lista módulos; sin evidencia de estado de implementación |

---

## Principios del producto

1. **Colombia primero:** los métodos de pago, el idioma por defecto y las restricciones de plataforma reflejan el contexto colombiano. Wompi sobre cualquier otra pasarela.

2. **Web para crear, móvil para consumir:** el diseño del venue y la gestión de eventos son exclusivos de la web. El móvil se optimiza para explorar, comprar y validar.

3. **Tiempo real sin fricción:** el selector de asientos debe ser instantáneo. Una reserva duplicada es un fallo crítico del producto.

4. **Calidad no negociable:** cobertura > 80%, 0 bugs críticos en SonarQube y pipeline CI/CD verde antes de cada merge. La calidad técnica es parte de la entrega formativa.

---

## OKRs del producto (proyecto formativo)

**Objetivo:** Demostrar una plataforma de gestión de eventos funcional y de calidad profesional como proyecto de grado SENA.

| Key Result | Baseline | Target | Cómo medirlo |
|------------|---------|--------|-------------|
| KR1: Cobertura de tests | 0% | > 80% | JaCoCo (backend) + Vitest (frontend) en Jenkins |
| KR2: Tiempo de respuesta API | Sin medir | < 200ms en endpoints principales | Load test con JMeter/k6 en staging |
| KR3: Latencia WebSockets | Sin medir | < 50ms en selector de asientos | Prueba de integración SockJS |
| KR4: Módulos del backend completados | Fase 1 completa | Fases 1–4 completas | Revisión de estado en repositorio |
| KR5: Vulnerabilidades críticas en producción | Sin medir | 0 | SonarQube Security en pipeline |

---

## Correlaciones

- Problema que resuelve → `03-product/evidence/problem-framing.md`
- Alcance detallado → `01-context/evidence/scope.md`
- Backlog de historias de usuario → `04-requirements/evidence/user-stories.md` *(pendiente de commit)*
- Descripción general del sistema → `01-context/evidence/overview.md`
- Convenciones ágiles y velocidad del equipo → `00-governance/evidence/agile-conventions.md`
