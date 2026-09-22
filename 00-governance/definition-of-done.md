# Definition of Done (DoD) - TuEvento

> Una Historia de Usuario está **TERMINADA** cuando cumple TODOS los criterios de esta checklist.
> Si falta aunque sea uno, la historia NO está done — regresa a In Progress.

## Checklist obligatorio

### Código
- [ ] El código implementa todos los criterios de aceptación de la historia de usuario
- [ ] El código fue revisado y aprobado por al menos 1 miembro del equipo (PR review)
- [ ] El código sigue los estándares del proyecto (linting y formateo pasan en CI)
- [ ] No se introdujo deuda técnica sin registrarla en el backlog técnico

### Tests
- [ ] Tests unitarios escritos para nueva lógica de negocio
- [ ] La cobertura de tests no disminuye del baseline del proyecto
- [ ] Todos los tests pasan localmente y en CI
- [ ] Criterios de aceptación verificados (manual o automatizado)

### Integración
- [ ] Los cambios no rompen otros servicios (tests de integración pasan)
- [ ] Si hay cambios de API: contrato OpenAPI actualizado
- [ ] Si hay cambios de modelo de datos: documentación del módulo SQL actualizada
- [ ] Si hay eventos nuevos/modificados: catálogo de eventos actualizado

### Base de Datos (específico TuEvento)
- [ ] Scripts de migración creados y probados
- [ ] Cambios de esquema documentados en el módulo SQL correspondiente
- [ ] Datos de prueba actualizados si es necesario
- [ ] Performance de queries validada para volúmenes esperados

### Deployment
- [ ] El código se puede mergear a `dev` (no hay conflictos)
- [ ] CI/CD verde en la rama
- [ ] Desplegado en ambiente de staging
- [ ] Smoke test básico pasando en staging

### Documentación
- [ ] README del servicio actualizado si cambió la interfaz pública
- [ ] Si se tomó una decisión técnica significativa: documentada en el módulo correspondiente

### Específico para módulos TuEvento

#### Para módulo Events
- [ ] Validaciones de fecha y capacidad funcionando
- [ ] Estados de evento manejados correctamente (DRAFT/PUBLISHED/CANCELLED/COMPLETED)
- [ ] Relaciones con Site y User validadas

#### Para módulo Payments  
- [ ] Estados de pago manejados (pending/processing/paid/failed/refunded)
- [ ] Integración con gateway de pago probada en sandbox
- [ ] Logs de transacciones generados correctamente

#### Para módulo Tickets
- [ ] Códigos QR generados y únicos
- [ ] Estados de ticket manejados correctamente
- [ ] Validación de check-in funcionando

#### Para módulo Security
- [ ] Autenticación y autorización validadas
- [ ] Permisos por rol verificados
- [ ] Logs de seguridad generados

---

## Excepciones permitidas

Las siguientes excepciones deben ser acordadas explícitamente por el Tech Lead:
- Tests E2E omitidos por limitaciones del ambiente (documentar el riesgo)
- Documentación diferida para entrega urgente (crear ticket de deuda técnica)

---

## Lo que NO es criterio de Done

- "El código está en mi máquina" — debe estar en el repositorio
- "Funciona en mi ambiente local" — debe funcionar en staging
- "El PM/PO lo aprobó" — ese es el Definition of Done de producto, no del código

---

## Herramientas para validar DoD

- **Backend:** Java 23 + Spring Boot (localhost:8080)
- **Frontend Web:** Vite (localhost:5173)
- **Mobile:** Expo (React Native)
- **Base de datos:** PostgreSQL 17 (localhost:5432/tuevento_db)
- **Linting:** SonarQube + Spring Boot Checkstyle
- **Tests:** JUnit 5 (Backend) + Jest/Testing Library (Frontend)
- **Coverage:** JaCoCo (Backend) + Vitest Coverage (Frontend)
- **CI/CD:** Jenkins + Docker + Kubernetes
- **WebSockets:** SockJS para funciones tiempo real

---

## Proceso de validación

1. **Desarrollador** verifica checklist antes de crear PR
2. **Reviewer** valida que DoD se cumple antes de aprobar
3. **CI/CD** ejecuta validaciones automáticas
4. **Tech Lead** valida excepciones si las hay
5. Solo después se hace merge a `dev`

---

## Métricas de calidad TuEvento

| Métrica | Target | Herramienta |
|---------|--------|-------------|
| Cobertura de código | > 80% | JaCoCo (Backend) + Vitest (Frontend) |
| Tiempo de build | < 5 min | Jenkins CI/CD |
| Tests unitarios | 100% pasan | JUnit 5 + Jest |
| Linting | 0 errores críticos | SonarQube |
| Vulnerabilidades críticas | 0 permitidas | SonarQube Security |

### Performance targets
- **Tiempo de respuesta API:** < 200ms (endpoints principales)
- **Capacidad de usuarios concurrentes:** 100+ usuarios simultáneos
- **WebSockets (tiempo real):** < 50ms latencia para selector de asientos