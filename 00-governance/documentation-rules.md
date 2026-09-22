# Reglas de Documentación - TuEvento

> Estas reglas determinan cómo se escribe, organiza y mantiene la documentación en el proyecto TuEvento.
> La documentación que no siga estas reglas puede ser rechazada en code review.

---

## Principio fundamental

> **"La documentación es código. Si no está actualizada, está rota."**

Cada HU que modifica el comportamiento del sistema DEBE incluir actualizar los documentos afectados.
El DoD lo requiere.

---

## Idioma

| Artefacto | Idioma |
|----------|--------|
| Código fuente (variables, funciones, clases) | Inglés |
| Comentarios de código | Español |
| Commits | Español (Conventional Commits) |
| Nombres de ramas | Inglés |
| Documentación Markdown | Español |
| Contratos OpenAPI (descripciones) | Español |
| Mensajes de error devueltos al frontend | Español |
| Logs internos del sistema | Inglés |

> **Regla:** Una vez elegido el idioma para cada categoría, es vinculante para todo el proyecto.
> Mezclar idiomas en la misma categoría es motivo de rechazo de PR.

---

## Estructura de archivos

```
Cada sección tiene su README.md que explica el propósito de la carpeta.
Los documentos de contenido usan kebab-case.md (ej: domain-map.md, risk-register.md).
Las plantillas tienen prefijo _ para aparecer primero (ej: _template-hu.md, _template-adr.md).
Los ADRs se numeran secuencialmente: ADR-001-titulo-corto.md.
```

---

## Qué documentar y qué NO

### SÍ documentar

| Qué | Dónde |
|-----|-------|
| Decisiones arquitectónicas no obvias | `05-architecture/decisions/records/ADR-NNN.md` |
| Reglas de negocio e invariantes del dominio | `02-domain/entities-and-rules.md` |
| Contratos de API para cada servicio | `07-api/contracts/openapi/[servicio].yaml` |
| Cambios en modelo de datos | `04_Diseno_Software/SQL/[Module].sql` |
| Procedimientos operacionales | `13-operations/` |
| Riesgos identificados | `02_Analisis_Software/matriz de riesgos-Tu evento.xlsx` |

### NO documentar

- Lo que el código ya dice claramente (no repetir en comentarios lo que se puede leer en el código)
- Decisiones temporales o experimentos que se van a revertir
- Detalles de implementación de librerías externas (esas tienen su propia documentación)
- Historial de cambios (para eso está git log)

## Responsables por sección

| Sección | Responsable | Frecuencia de revisión |
|---------|-------------|----------------------|
| `00-governance/` | Cristofer David Lozano (Tech Lead) | Inicio de cada sprint |
| `01_Especificacion_Requisitos/` | Cristofer David Lozano (Product Owner) | Cada sprint |
| `02_Analisis_Software/` | Ambos integrantes | Cuando cambia el análisis |
| `04_Diseno_Software/` | Cristofer David Lozano (Backend) | Cada cambio de modelo |
| `05-architecture/` | Cristofer David Lozano (Tech Lead) | Cada decisión de diseño |
| `06_Pruebas/` | Jhampier Santos Ortiz (QA) | Cada release |
| `07_Implementacion/` | Jhampier Santos Ortiz (DevOps) | Después de cada incidente |
| `09_Manuales/` | Ambos integrantes | Cada release mayor |

---

## Formato de documentos

| Rol | Responsabilidad en governance |
|-----|------------------------------|
| **Tech Lead** | Cristofer David Lozano - Velar por el cumplimiento de todas las reglas técnicas |
| **Scrum Master** | Jhampier Santos Ortiz - Asegurar que las ceremonias y DoD/DoR se cumplan |
| **Desarrolladores** | Ambos integrantes - Seguir las convenciones de código, git y documentación |
| **Todo el equipo** | Ambos integrantes - Proponer mejoras y cambios a las reglas cuando sea necesario |

---

## Formato de documentos

### Encabezados
- `# H1` — solo uno por archivo; es el título
- `## H2` — secciones principales
- `### H3` — subsecciones
- No usar H4 o más profundo; si lo necesitas, el documento tiene demasiada jerarquía

### Tablas
Usar tablas para comparaciones, registros y matrices. No usar tablas para listas simples.

### Código
Siempre usar bloques de código con el lenguaje especificado:
````
```sql
CREATE TABLE event (
    event_id INT AUTO_INCREMENT PRIMARY KEY
);
```
````

### Instrucciones de plantilla
Los bloques marcados `> [!NOTE] INSTRUCCIONES` indican que el documento es una plantilla sin llenar.
Eliminarlos cuando el documento esté completo.

---

## Proceso de actualización

1. El desarrollador identifica qué documentos afecta su cambio
2. Actualiza los documentos junto con el código (mismo PR)
3. El reviewer verifica que la documentación esté actualizada
4. Si el PR cierra una HU que tuvo impacto de API → el contrato OpenAPI debe estar actualizado

---

## Específico para TuEvento

### Referencias entre documentos
Usar paths relativos para referencias:
```markdown
- Módulo de pagos → `04_Diseno_Software/SQL/Payment module.sql`
- Matriz de riesgos → `02_Analisis_Software/matriz de riesgos-Tu evento.xlsx`
- Historias de usuario → `01_Especificacion_Requisitos/Historias_Usuario_TuEvento.docx`
```

### Naming conventions para archivos SQL:
```
[Module] module.sql
Ejemplos:
- Event module.sql
- Payment module.sql
- Security module.sql
```

### Versionado de documentos importantes:
- Los contratos de API usan versionado semántico
- Los cambios de esquema de BD se documentan con fecha
- Los ADRs no se modifican, se crean nuevos que anulan anteriores

### Herramientas del proyecto:
- **Diagramas:** Draw.io, PlantUML, Mermaid
- **Mockups:** Figma, Balsamiq
- **Documentos:** Markdown + Word para entregables formales
- **Versionado:** Git para toda la documentación

---

## Correlaciones

- Convenciones git → `00-governance/git-conventions.md`
- Estándar de documentación por microservicio → `00-governance/microservices-documentation.md`
- Definition of Done (docs como parte de DoD) → `00-governance/definition-of-done.md`
- Arquitectura del proyecto → `04_Diseno_Software/Diagrama de arquitectura.png`
---

## Configuración específica TuEvento

### Herramientas de diseño
- **UX/UI Mockups:** Figma (referencia en carpeta `11_Prototipo/Mockup/`)
- **Diagramas técnicos:** Draw.io + PlantUML
- **Diagramas de arquitectura:** Disponibles en `04_Diseno_Software/`
- **User flows:** Documentados en `11_Prototipo/User Flow/`

### Métricas de calidad
- **Cobertura de código:** > 80% (JaCoCo + Vitest)
- **Tiempo de build:** < 5 minutos (Jenkins)
- **Performance API:** < 200ms respuesta
- **Vulnerabilidades críticas:** 0 permitidas

### Stack tecnológico completo
- **Backend:** Java 23 + Spring Boot (localhost:8080)
- **Frontend Web:** Vite + React (localhost:5173)
- **Mobile:** Expo + React Native
- **Base de datos:** PostgreSQL 17 (localhost:5432/tuevento_db)
- **Tiempo real:** SockJS WebSockets
- **Autenticación:** JWT + Refresh Token (HS256, 24h)
- **Pagos:** Wompi (Tarjeta, PSE, Nequi, Efectivo, COP)
- **DevOps:** Jenkins + SonarQube + Docker + Kubernetes