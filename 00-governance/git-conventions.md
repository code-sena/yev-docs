# Convenciones Git - TuEvento

> **Lee este documento antes de hacer tu primer commit en el proyecto TuEvento.**

## Estrategia de ramas

```
main        ← Producción. Merge solo desde release. Siempre estable.
  └── dev   ← Integración continua. Merge desde features.
        └── feat/[descripcion]     ← Una rama por feature/historia de usuario
        └── fix/[descripcion]      ← Una rama por bugfix
        └── chore/[descripcion]    ← Infraestructura, docs, dependencias
        └── hotfix/[descripcion]   ← Fixes urgentes directo a main
```

**Reglas:**
- Nadie hace commit directo a `main` o `dev`
- Cada tarea = una rama + un Pull Request
- Una rama = una tarea (no mezclar diferentes features)
- Las ramas se eliminan después del merge

---

## Formato de nombres de ramas

```
[tipo]/[descripcion-en-kebab-case]

Ejemplos:
feat/autenticacion-oauth2
fix/calculo-overlap-horarios
chore/actualizar-dependencias-spring
hotfix/expiracion-token-nulo
```

---

## Formato de commits (Conventional Commits)

```
[tipo]([scope]): [descripción en minúsculas, modo imperativo, sin punto final]

[cuerpo opcional — explica el POR QUÉ, no el qué]

[footer opcional — referencias a issues/historias de usuario]
```

**Tipos:**
| Tipo | Cuándo usarlo |
|------|---------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Solo documentación |
| `style` | Formato, espacios en blanco (sin cambio de lógica) |
| `refactor` | Refactoring de código sin cambio de comportamiento |
| `test` | Agregar o modificar tests |
| `chore` | Tooling, dependencias, CI |
| `perf` | Mejora de performance |

**Ejemplos:**
```
feat(events): implementar creación de eventos

fix(payments): corregir validación de overlapping de horarios
Closes #42

docs(api): actualizar contrato OpenAPI del servicio de eventos

chore(deps): actualizar Spring Boot a 3.2.0
```

---

## Política de Pull Requests

- **Tamaño:** máximo 400 líneas de código (excluyendo tests). Si es mayor, dividir.
- **Reviewers:** mínimo 1 aprobación antes del merge
- **Tiempo de review:** el reviewer tiene máximo 24 horas hábiles
- **Template:** usar el template en `.github/pull_request_template.md`
- **CI verde:** el merge solo procede si todos los checks del pipeline pasan

---

## Política de merge

- Usar **Squash and Merge** para features (mantiene el historial de `dev` limpio)
- Usar **Merge Commit** para releases a `main` (preserva historial completo)
- **NO usar** Rebase & Merge (crea confusión en historial compartido)

---

## Tags y versionado

Seguir [SemVer](https://semver.org/): `MAJOR.MINOR.PATCH`

```bash
# Al hacer release a producción
git tag -a v1.2.0 -m "Release v1.2.0: agregar módulo de reportes"
git push origin v1.2.0
```

---

## Contexto específico TuEvento

### Scopes recomendados para commits:
- `events` - Funcionalidad de eventos
- `payments` - Sistema de pagos
- `tickets` - Gestión de boletos
- `users` - Gestión de usuarios
- `security` - Autenticación y autorización
- `notifications` - Sistema de notificaciones
- `api` - Cambios en APIs REST
- `db` - Migraciones y cambios de BD
- `tests` - Suite de testing
- `docs` - Documentación

### Branches protegidas:
- `main`: Requiere PR + 1 aprobación + CI verde
- `dev`: Requiere PR + CI verde

### Integración con metodología Scrum:
- Crear rama al empezar una historia de usuario
- Commit frecuente durante el desarrollo
- PR al completar historia de usuario
- Merge a `dev` después de review
- Release desde `dev` a `main` al final del sprint