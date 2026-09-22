# 00-governance — Reglas de Equipo TuEvento

> Esta sección define los acuerdos que el equipo se compromete a seguir durante todo el proyecto TuEvento.
> Cada miembro del equipo debe leer todos los documentos de esta sección antes de hacer su primer commit.

---

## Documentos en esta sección

| Archivo | Propósito |
|---------|-----------|
| [git-conventions.md](./git-conventions.md) | Estrategia de ramas, formato de commits, política de PRs |
| [agile-conventions.md](./agile-conventions.md) | Estructura de sprints, ceremonias, estimación |
| [definition-of-done.md](./definition-of-done.md) | Checklist que debe cumplir cada historia de usuario completada |
| [definition-of-ready.md](./definition-of-ready.md) | Checklist para que una historia de usuario entre al sprint |
| [documentation-rules.md](./documentation-rules.md) | Cómo escribir, actualizar y eliminar documentación |
| [microservices-documentation.md](./microservices-documentation.md) | Documentos requeridos por cada microservicio |
| [security-rules.md](./security-rules.md) | Reglas de seguridad a nivel de código: secretos, auth, validaciones |

---

## Cómo aplica governance

Las reglas de governance aplican a **todo el proyecto TuEvento** — todas las secciones, todos los servicios, todos los miembros del equipo.
Si una regla entra en conflicto con una convención local, governance gana a menos que una nueva decisión la anule.

> Cambiar una regla de governance requiere acuerdo del equipo completo.
> Documenta el cambio y la razón. Anúncialo antes del próximo sprint.

---

## Contexto del proyecto TuEvento

### Información del equipo
- **Proyecto:** Sistema de gestión de eventos TuEvento
- **Grupo:** 11 - SENA - CapySoft
- **Metodología:** Scrum + SDD (Software Design Documentation)
- **Repositorio:** https://github.com/lozano2303/TuEvento
- **Tecnologías principales:** 
  - Backend: Java 23 + Spring Boot (localhost:8080)
  - Frontend Web: Vite (localhost:5173) 
  - Mobile: Expo (React Native)
  - Base de datos: PostgreSQL 17 (localhost:5432/tuevento_db)
  - WebSockets: SockJS (tiempo real)
  - Pagos: Gateway Wompi

### Integrantes del equipo
- **Tech Lead/Product Owner:** Cristofer David Lozano Contreras
  - Especialización: Backend + Base de datos
  - Tel: 313 4605214 | Email: cristoferlozano233@gmail.com
- **Scrum Master/DevOps:** Jhampier Santos Ortiz
  - Especialización: Frontend + Testing/QA
  - Tel: 302 7700760 | Email: ortizjhampier@gmail.com
- **Stakeholder Original:** Instructor Jhon Corredor (idea del proyecto)

### Estado actual
- ✅ Análisis y diseño técnico completo
- 🚧 Implementación en progreso
- 📋 Documentación en migración a SDD

---

## Responsabilidades de governance

| Rol | Responsable | Responsabilidad en governance |
|-----|-------------|------------------------------|
| **Tech Lead** | Cristofer David Lozano | Velar por el cumplimiento de todas las reglas técnicas |
| **Scrum Master** | Jhampier Santos Ortiz | Asegurar que las ceremonias y DoD/DoR se cumplan |
| **Desarrolladores** | Ambos integrantes | Seguir las convenciones de código, git y documentación |
| **Todo el equipo** | Ambos integrantes | Proponer mejoras y cambios a las reglas cuando sea necesario |

### Modelo de trabajo híbrido
- **Especialización base:** Cada uno tiene su área de fortaleza
- **Flexibilidad total:** Ambos pueden trabajar en cualquier módulo según necesidad
- **Prioridad por dependencias:** Módulos base primero, luego los dependientes