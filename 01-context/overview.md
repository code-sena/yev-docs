# Descripción General del Sistema — Tu Evento

> Este documento es la tarjeta de presentación del sistema. Cualquier persona nueva debe poder entender qué es "Tu Evento" y qué problema resuelve en menos de 5 minutos.

---

## ¿Qué es Tu Evento?

**Tu Evento** es una plataforma web y móvil desarrollada por **CapySoft** para la creación, administración y operación de eventos. Proporciona a los organizadores herramientas para diseñar la distribución de venues, configurar secciones y asientos, gestionar reservas y procesar pagos de forma segura. Los asistentes pueden explorar eventos, comprar entradas, seleccionar asientos, recibir notificaciones push y validar su entrada mediante código QR.

## Problema que resuelve

**Antes del sistema:** la organización de eventos (conciertos, conferencias, ferias) dependía de procesos manuales o herramientas desconectadas para vender entradas, gestionar aforos y controlar el acceso. Los organizadores no tenían visibilidad en tiempo real de las reservas, y los asistentes no podían seleccionar asientos ni recibir confirmaciones digitales.

**Con el sistema:** el organizador crea y configura su evento desde la web (diseño del venue, secciones, precios), mientras los asistentes compran entradas y seleccionan asientos desde la web o la app móvil, reciben su QR digital y validan su ingreso en la puerta. Todo el flujo queda registrado y auditable.

## Usuarios principales

| Rol | Descripción | Qué hace en el sistema |
|-----|-------------|----------------------|
| Organizador | Persona o empresa que crea y gestiona eventos | Crea eventos, diseña la distribución del venue, define secciones y precios, gestiona asistencia — **exclusivo web** |
| Asistente | Usuario que asiste a eventos | Explora eventos, compra entradas, selecciona asientos, recibe QR, valida ingreso — **web y móvil** |
| Administrador | Gestor del sistema a nivel global | Aprueba organizadores, gestiona roles y permisos, administra el sistema |
| Usuario anónimo | Visitante sin cuenta | Navega y consulta eventos públicos — **solo web** |

## Stack tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| Backend | Java 23 + Spring Boot | Stack robusto y ampliamente adoptado en la industria; formación SENA |
| Persistencia | Spring Data JPA | Abstracción del acceso a datos sobre el ORM estándar de Java |
| Base de datos | PostgreSQL 17 | Motor relacional open source, soporte avanzado para JSONB (layouts) |
| Autenticación | JWT HS256 + Refresh Token (24h) | Sesiones seguras y auditables sin estado en el servidor |
| Pasarela de pago | Wompi | Pasarela de pagos colombiana; soporta tarjeta, PSE, Nequi y efectivo (COP) |
| Notificaciones | SMTP (email) + Push | Cobertura multicanal para confirmaciones y alertas |
| Tiempo real | SockJS (WebSockets) | Selector de asientos en tiempo real (< 50ms latencia) |
| Estilo de API | RESTful (JSON / HTTP) | Estándar de la industria para integración web y móvil |
| Frontend web | Vite + React (localhost:5173) | Framework moderno; compatible con Chrome y Edge en Windows |
| App móvil | Expo + React Native (Android e iOS) | Desarrollo cross-platform; Android como plataforma principal |
| Infraestructura | Docker + Kubernetes + Jenkins + SonarQube | Pipeline CI/CD completo para automatización de calidad y despliegue |
| Almacenamiento | AWS S3 / MinIO | Almacenamiento de medios de eventos e imágenes de perfil |

## Arquitectura

El sistema sigue una **arquitectura modular N-Layer con Domain-Driven Design (DDD)**, orientada a microservicios. El API Gateway centraliza el enrutamiento y el control de acceso por rol. La comunicación cliente-servidor es exclusivamente HTTP con JSON.

Módulos principales del backend:

| Módulo | Responsabilidad |
|--------|----------------|
| Security | Autenticación, JWT, OAuth, roles y permisos |
| Profile | Perfil de usuario, preferencias, historial de actividad |
| Event | Ciclo de vida completo del evento (creación → publicación → cierre) |
| Section / Seat | Gestión de secciones, bloques y asientos individuales |
| Ticket | Emisión, QR, check-in y auditoría de entradas |
| Payment | Órdenes, pagos Wompi, reembolsos, webhooks |
| Wallet | Billetera digital por usuario (recarga, pago, retiro) |
| Notification | Notificaciones multicanal con seguimiento de entrega |
| Geolocation | Gestión de venues, ciudades y departamentos |
| Category | Categorización jerárquica de eventos |
| Language | Internacionalización: ES, EN, PT, JA |
| Theme | Temas de interfaz configurables por usuario |

## Estado actual del proyecto

| Campo | Detalle |
|-------|---------|
| Fase | En desarrollo activo |
| Versión actual | MVP en construcción |
| Componente activo | Backend (Spring Boot) |
| Pendiente | Integración progresiva del frontend web y móvil |
| Próximo hito | Integración completa del módulo de pagos y notificaciones |

## Equipo del proyecto

| Rol institucional | Nombre | Contacto |
|-------------------|--------|---------|
| Instructor / Product Owner | SENA — Sede Industria y los Servicios | — |
| Stakeholder original / Idea del proyecto | Jhon Corredor — Idea original / Stakeholder externo (ya no vinculado al SENA) | — |

### Roles en el equipo:
- **Tech Lead/Product Owner:** Cristofer David Lozano Contreras
- **Scrum Master/DevOps:** Jhampier Santos Ortiz
- **Desarrollador Backend (especialización):** Cristofer David Lozano Contreras
- **Desarrollador Frontend (especialización):** Jhampier Santos Ortiz
- **Nota:** Ambos trabajan full-stack según necesidad del proyecto

**Ficha:** 314556 (2026) — Programa: Tecnólogo en Análisis y Desarrollo de Software — SENA, Neiva, Huila.
**Grupo:** Grupo 11 — CapySoft
**Metodología:** Scrum + SDD (Software Design Documentation)
**Repositorio:** https://github.com/lozano2303/TuEvento
**Backlog:** https://trello.com/b/QqBfrvr5/tuevento-capysoft

---

## Correlaciones

- Alcance del sistema → `01-context/evidence/scope.md`
- Glosario de términos → `01-context/evidence/glossary.md`
- Requisitos funcionales → `02_Especificacion_Requisitos/`
- Diseño y arquitectura → `05_Diseno_Software/`
