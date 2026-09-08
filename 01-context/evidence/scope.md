# Alcance del Sistema — Tu Evento

> Este documento define qué construye y qué NO construye el proyecto "Tu Evento". Es igualmente importante delimitar lo que está dentro como lo que está fuera del alcance. Revisar al inicio de cada ciclo de planificación.

---

## En Alcance (MVP)

Funcionalidades que el sistema **sí construye y mantiene**:

### Módulo de Seguridad y Acceso

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 1 | Registro de usuario con validación de datos | RF1 |
| 2 | Inicio de sesión con JWT + Refresh Token | RF3, RF15 |
| 3 | Autenticación social OAuth (Google, etc.) | RF2 |
| 4 | Recuperación de contraseña por email | RF5 |
| 5 | Cierre de sesión (revocación de token) | RF6 |
| 6 | Eliminación permanente de cuenta | RF7 |
| 7 | Control de acceso por roles (asistente, organizador, admin) | RF14 |

### Módulo de Gestión de Eventos

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 8 | Creación, edición, publicación y cancelación de eventos | RF9 |
| 9 | Diseño visual de distribución del venue (secciones y asientos) — solo web | RF9, RF19 |
| 10 | Gestión de secciones con capacidad, precio y estado | RF9 |
| 11 | Filtros y búsqueda de eventos por categoría, fecha y ubicación | RF23 |
| 12 | Sistema de calificación y comentarios de eventos | RF9 |
| 13 | Categorización jerárquica de eventos | RF9 |

### Módulo de Reservas y Entradas

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 14 | Selección y reserva de asientos individuales | RF10 |
| 14b | Selector de asientos en tiempo real vía SockJS (WebSockets) — web y móvil | RF10 |
| 15 | Compra de entradas desde la aplicación móvil | RF18 |
| 16 | Generación de código QR por entrada | RF11 |
| 17 | Validación de QR para control de acceso en puerta | RF24 |
| 18 | Registro completo del ciclo de vida de cada entrada | RF11 |

### Módulo de Pagos y Billetera

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 19 | Integración con pasarela Wompi (tarjeta, PSE, Nequi, efectivo) | RF17 |
| 20 | Gestión de órdenes y estados de transacción | RF17 |
| 21 | Solicitudes de reembolso | RF17 |
| 22 | Billetera digital por usuario (recarga, pago, retiro) | RF25 |

### Módulo de Perfil y Personalización

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 23 | Gestión de perfil de usuario | RF4 |
| 24 | Historial de compras y actividad | RF4 |
| 25 | Preferencias de usuario (idioma, tema, notificaciones) | RF4 |
| 26 | Sistema de temas de interfaz (light, dark, purple, blue) | RF22 |
| 27 | Soporte multilingüe: ES, EN, PT, JA | RF13 |

### Módulo de Notificaciones

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 28 | Notificaciones push (confirmación de compra, recordatorios) | RF16 |
| 29 | Notificaciones por email (SMTP) | RF16 |
| 30 | Seguimiento de estado de entrega por notificación | RF16 |

### Administración del Sistema

| # | Funcionalidad | RF relacionado |
|---|--------------|----------------|
| 31 | Panel de administración (gestión de usuarios, aprobación de organizadores) | RF8 |
| 32 | Gestión de roles y permisos | RF8, RF14 |

### Plataformas cubiertas

| Plataforma | Tecnología | Funcionalidad disponible |
|-----------|-----------|-------------------------|
| Web (Chrome / Edge en Windows) | Vite + React (localhost:5173) | Todas las funcionalidades incluyendo diseño de venue |
| App móvil Android | Expo + React Native (Android 11+) | Exploración, compra, selección de asiento en tiempo real, QR, notificaciones |
| App móvil iOS | Expo + React Native | Soporte funcional (mismas capacidades que Android) |

### Integraciones incluidas

| Sistema externo | Tipo | Propósito |
|----------------|------|----------|
| Wompi | REST API | Procesamiento de pagos en Colombia (tarjeta, PSE, Nequi, efectivo COP) |
| AWS S3 / MinIO | SDK | Almacenamiento de medios (imágenes de eventos, fotos de perfil) |
| SMTP (email) | Protocolo | Notificaciones por correo electrónico |
| OAuth (Google) | OAuth 2.0 | Autenticación social |
| SockJS (WebSockets) | Protocolo tiempo real | Selector de asientos sincronizado en tiempo real (< 50ms latencia) |

### Infraestructura y DevOps incluida

| Componente | Propósito |
|-----------|----------|
| Docker + Docker Compose | Contenerización y orquestación local |
| Kubernetes (Minikube) | Despliegue y escalabilidad |
| Jenkins | Pipeline CI/CD automatizado |
| SonarQube | Análisis de calidad de código (quality gate: >80% cobertura, 0 bugs críticos) |
| PostgreSQL 17 | Base de datos principal (localhost:5432/tuevento_db) |
| Redis | Caché y gestión de sesiones |
| Trello | Gestión del backlog y tablero Scrum del equipo |

---

## Fuera de Alcance (MVP)

Lo que el sistema **NO construye** en esta versión:

| # | Qué está fuera de alcance | Motivo | ¿Versión futura? |
|---|--------------------------|--------|-----------------|
| 1 | Aplicación de escritorio (Windows/macOS) | No es el canal objetivo definido | No planificado |
| 2 | Integración con otras pasarelas de pago (PayU, MercadoPago) | Wompi es suficiente para el contexto colombiano del MVP | Posible |
| 3 | Sistema de streaming o eventos virtuales | Fuera del dominio de eventos presenciales | Posible — v2 |
| 4 | Módulo de reportes avanzados / Business Intelligence | Requiere stack adicional de analítica | Posible — v2 |
| 5 | Integración con redes sociales para publicación automática | No es crítico para el flujo de compra | Posible — v2 |
| 6 | App en tiendas (Google Play Store / App Store) — publicación oficial | El alcance formativo cubre el desarrollo, no la publicación | Post-grado |
| 7 | Soporte para eventos completamente gratuitos con cupo ilimitado | Requiere lógica de negocio adicional en pagos | Posible — v2 |
| 8 | Módulo de facturación electrónica (DIAN) | Requiere certificación e integración regulatoria | Pendiente evaluación |
| 9 | Chat en tiempo real entre organizador y asistentes | Fuera del dominio core de gestión de eventos | No planificado |

### Lo que gestiona otro componente / equipo

| Funcionalidad | Quién lo gestiona | Por qué no este sistema |
|--------------|------------------|------------------------|
| CDN para distribución de medios | AWS S3 / MinIO | Servicio externo reutilizado |
| Proveedor de email transaccional | Servidor SMTP externo | Delegado a proveedor de infraestructura |

---

## Supuestos del alcance

> Se asumen verdaderos. Si cambian, el alcance debe renegociarse con el instructor/PO.

| # | Supuesto | Consecuencia si es falso |
|---|---------|--------------------------|
| 1 | Wompi tiene API disponible y documentada para integración | Se tendría que evaluar otra pasarela de pago |
| 2 | Los usuarios disponen de conexión a internet para todas las operaciones | El sistema no funciona offline; requeriría rediseño |
| 3 | El diseño de venue se gestiona solo desde web; el móvil es para consumo | Habría que rediseñar la UX móvil para incluir herramientas de diseño |
| 4 | El volumen inicial de datos es manejable con una instancia de PostgreSQL | Si escala masivamente, se necesitaría sharding o réplicas |
| 5 | Android 11+ cubre la mayoría de dispositivos objetivo en Colombia | Si el target usa versiones anteriores, hay que ampliar compatibilidad |

---

## Restricciones

| Tipo | Descripción |
|------|-------------|
| **Tiempo** | Alcance ajustado al período formativo del Tecnólogo — Ficha 314556 (2026) |
| **Equipo** | 2 desarrolladores: Cristofer David Lozano Contreras y Jhampier Santos Ortiz |
| **Tecnología** | Stack definido: Java 23 + Spring Boot + PostgreSQL 17 + Expo + React Native |
| **Plataforma web** | Vite + React, compatible con Chrome y Edge en Windows |
| **Plataforma móvil** | Expo + React Native — Android 11+ como plataforma principal; iOS funcional |
| **Metodología** | Scrum + SDD (Software Design Documentation) — Grupo 11, CapySoft |
| **Formativo** | Proyecto evaluado en el marco del programa SENA — ADSO, Neiva, Huila |

---

## Cómo gestionar cambios de alcance

Si surge una nueva funcionalidad o se necesita modificar el alcance:

1. Documentar el cambio propuesto en este archivo
2. Evaluar el impacto en tiempo y esfuerzo
3. Obtener aprobación del instructor (Product Owner institucional — SENA)
4. Actualizar las historias de usuario en `02_Especificacion_Requisitos/`
5. Registrar la decisión técnica si aplica en `05_Diseno_Software/`

---

## Correlaciones

- Descripción general del sistema → `01-context/evidence/overview.md`
- Glosario de términos → `01-context/evidence/glossary.md`
- Historias de usuario → `02_Especificacion_Requisitos/Historias_Usuario_TuEvento.docx`
- Requisitos funcionales → `02_Especificacion_Requisitos/Manual SRS Tu Evento.docx`
- Diseño y arquitectura → `05_Diseno_Software/`
- Backlog del equipo → https://trello.com/b/QqBfrvr5/tuevento-capysoft
