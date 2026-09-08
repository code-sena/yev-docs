# Convenciones Ágiles - Equipo TuEvento

> Define cómo trabaja el equipo durante sus ciclos de desarrollo. Acordar y firmar
> con todo el equipo antes del primer sprint. Actualizar cuando el equipo decida cambiar algo.

---

## Estructura de sprint

| Campo | Valor |
|-------|-------|
| Duración | Variable según complejidad (1-4 semanas) |
| Inicio de sprint | Al completar tarea/módulo anterior |
| Fin de sprint | Al completar funcionalidad y hacer merge |
| Sprint actual | Módulo Seat Selection + WebSockets — 13 agosto a 8+ septiembre |
| Capacidad estimada | 1 módulo completo por sprint (variable según complejidad) |

### Tipos de sprint por complejidad:
- **Sprint rápido (1-3 días):** Fixes, features simples, ajustes menores
- **Sprint medio (1-2 semanas):** Módulos estándar (CRUD, APIs básicas)
- **Sprint complejo (3-4 semanas):** Módulos avanzados (WebSockets, tiempo real, UX complejo)

---

## Ceremonias

### Sprint Planning
- **Cuándo:** Al completar tarea/módulo anterior — durante horario de clases (12:00-6:00 PM)
- **Duración:** 30-45 minutos para módulos complejos, 15-20 minutos para tareas simples
- **Quién:** Todo el equipo
- **Objetivo:** Definir qué módulo/tarea desarrollar, dividir responsabilidades, estimar complejidad
- **Artefacto de salida:** Sprint Backlog actualizado en Trello

### Daily Check-in (formato híbrido)
- **Cuándo:** Cada 2-3 días o cuando hay cambios importantes
- **Duración:** 5-15 minutos
- **Formato:** 
  - **Presencial:** Durante clases (12:00-6:00 PM) - conversaciones informales
  - **Virtual:** WhatsApp para notificar PRs, bloqueos, cambios
- **Contenido:** Estado actual, bloqueos técnicos, PRs pendientes

### Sprint Review
- **Cuándo:** Al completar el módulo/tarea
- **Duración:** 15-30 minutos (según complejidad)
- **Quién:** Ambos integrantes
- **Objetivo:** Demostrar funcionalidad completa, validar que cumple requisitos

### Sprint Retrospective
- **Cuándo:** Después del review o al iniciar nueva tarea
- **Duración:** 10-20 minutos
- **Formato:** ¿Qué funcionó bien? ¿Qué mejorar? ¿Qué aprendimos?
- **Enfoque:** Mejora continua para el siguiente módulo

### Backlog Refinement (continuo)
- **Cuándo:** Durante el desarrollo, cuando surgen nuevas necesidades
- **Duración:** Variable
- **Objetivo:** Ajustar prioridades, agregar tareas técnicas, replantear si es necesario

---

## Estimación

### Escala de Estimación (por complejidad)
| Complejidad | Duración estimada | Ejemplos |
|-------------|------------------|----------|
| **Trivial** | 1-2 días | Fixes menores, ajustes de UI, correcciones |
| **Simple** | 3-5 días | CRUDs básicos, validaciones, APIs simples |
| **Medio** | 1-2 semanas | Módulos estándar, integraciones básicas |
| **Complejo** | 3-4 semanas | WebSockets, tiempo real, UX avanzado, integraciones complejas |
| **Muy complejo** | 1+ mes | Refactoring mayor, arquitectura nueva |

**Técnica:** Consenso por experiencia y complejidad técnica
**Herramienta:** Trello + estimación empírica

### Regla de estimación
- Si una tarea parece "muy compleja" (1+ mes), dividirla en subtareas más pequeñas
- Reevaluar complejidad si la tarea toma 50% más tiempo del estimado
- Documentar lecciones aprendidas para mejorar estimaciones futuras

---

## Herramienta de backlog

**Herramienta:** Trello
**URL del board:** https://trello.com/b/QqBfrvr5/tuevento-capysoft

### Columnas del board (sugeridas)
| Columna | Significado |
|---------|-------------|
| Ideas / Backlog | Funcionalidades pendientes por desarrollar |
| Ready | Tareas definidas y listas para empezar |
| In Progress | Alguien está trabajando activamente |
| Testing | Funcionalidad completa, en pruebas |
| Done | Completado y mergeado |

### Comunicación del equipo
- **Plataforma principal:** WhatsApp
- **Horario de trabajo:** 12:00 PM - 6:00 PM (horario de clases SENA)
- **Notificaciones importantes:** PRs, bloqueos, cambios de alcance
- **Reuniones presenciales:** Durante clases en SENA

---

## Velocidad del equipo

| Sprint/Módulo | Complejidad | Duración real | Funcionalidad | Notas |
|---------------|-------------|---------------|---------------|--------|
| Seat Selection + WebSockets | Muy complejo | 26+ días (13 ago - 8+ sep) | Selector tiempo real web + móvil | Feature compleja con múltiples plataformas |
| [Módulo anterior] | — | — | — | — |
| [Módulo anterior] | — | — | — | — |

### Patrones identificados:
- **Features simples:** 1-3 días consistentemente
- **Módulos estándar:** 1-2 semanas
- **Features con tiempo real/WebSockets:** 3-4 semanas+
- **Cross-platform (web + móvil):** Aumenta 50-100% el tiempo estimado

---

## Contexto específico TuEvento

### Módulos del sistema (para organizar historias de usuario):
- **Gestión de Eventos** - Crear, editar, publicar eventos
- **Sistema de Pagos** - Procesamiento de transacciones
- **Gestión de Boletos** - Compra, validación, check-in
- **Autenticación** - Login, registro, perfiles
- **Notificaciones** - Email, SMS, push notifications
- **Reportes** - Analytics y métricas del sistema

### Roles en el equipo:
- **Tech Lead/Product Owner:** Cristofer David Lozano Contreras
- **Scrum Master/DevOps:** Jhampier Santos Ortiz
- **Desarrollador Backend (especialización):** Cristofer David Lozano Contreras
- **Desarrollador Frontend (especialización):** Jhampier Santos Ortiz
- **Nota:** Ambos trabajan full-stack según necesidad del proyecto

### Definition of Ready específica para TuEvento:
- Historia tiene criterios de aceptación claros
- Está estimada por el equipo
- Dependencias técnicas identificadas
- Mock-ups disponibles si involucra UI
- Modelo de datos definido si hay cambios de BD

### Tecnologías del proyecto TuEvento:
- **Repositorio:** GitHub - https://github.com/lozano2303/TuEvento
- **Backend:** Java 23 + Spring Boot (http://localhost:8080)
- **Frontend Web:** Vite (http://localhost:5173)
- **Mobile App:** Expo (React Native)
- **Base de datos:** PostgreSQL 17 (localhost:5432/tuevento_db)
- **Autenticación:** JWT + Refresh Token
- **Tiempo real:** SockJS (WebSockets) para selector de asientos
- **Pagos:** Gateway Wompi (tarjeta, PSE, Nequi, efectivo)
- **Arquitectura:** N-Layer + DDD orientado a microservicios
- **DevOps:** Jenkins + SonarQube + Docker + Kubernetes
- **Comunicación:** WhatsApp + presencial SENA (12:00-6:00 PM)

---

## Documentos relacionados

- Definition of Ready → `00-governance/definition-of-ready.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Gestión de riesgos → `02_Analisis_Software/matriz de riesgos-Tu evento.xlsx`
- Backlog técnico → [ubicación por definir]