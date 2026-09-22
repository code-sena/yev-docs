# Definition of Ready (DoR) - TuEvento

> Una Historia de Usuario está **Ready** cuando todo el equipo puede empezarla en el próximo sprint
> sin necesidad de resolver preguntas fundamentales a mitad del sprint.
> Si una historia no cumple este DoR, regresa a refinamiento.

---

## Checklist DoR

Antes de mover una Historia de Usuario a "Ready for Sprint", verificar:

### Claridad

- [ ] La historia está escrita en formato: **Como [rol], quiero [acción], para que [beneficio]**
- [ ] El rol es específico (no "como usuario" — "como organizador autenticado")
- [ ] El beneficio esperado es claro y verificable

### Criterios de Aceptación

- [ ] Hay al menos 2 criterios de aceptación escritos en formato **Dado / Cuando / Entonces**
- [ ] Los criterios cubren el flujo feliz Y los principales casos de error
- [ ] Los criterios son testeable (es posible escribir un test automatizado para cada uno)
- [ ] No hay criterios ambiguos ("la respuesta debe ser rápida" no es válido)

### Dependencias

- [ ] Todas las dependencias externas (otros servicios, APIs, datos) están identificadas
- [ ] Las dependencias bloqueantes están resueltas O se definió un workaround
- [ ] Si depende de otra historia, esa historia ya está Done o In Progress

### Estimación

- [ ] El equipo ha estimado la historia (story points o t-shirt size)
- [ ] Hay acuerdo en que la historia cabe en un sprint
- [ ] Si es muy grande, se ha dividido en historias más pequeñas

### Preparación técnica

- [ ] Los accesos y ambientes necesarios están disponibles
- [ ] Los contratos de API están definidos si la historia involucra nuevos endpoints
- [ ] Hay definición del modelo de datos si hay cambios de BD
- [ ] El impacto en otros servicios está identificado

### Requerimientos no funcionales

- [ ] Requerimientos de performance especificados (si aplica)
- [ ] Consideraciones de seguridad incluidas (autenticación, autorización, validaciones)
- [ ] Requerimientos de observabilidad incluidos (logs, métricas, trazas)

### Específico para módulos TuEvento

#### Para historias del módulo Events
- [ ] Mockups o wireframes disponibles si hay cambios de UI
- [ ] Validaciones de negocio definidas (fechas, capacidad, overlapping)
- [ ] Estados del evento especificados claramente
- [ ] Integración con módulo Site definida

#### Para historias del módulo Payments
- [ ] Gateway de pago especificado (Wompi, PSE, etc.)
- [ ] Flujo de estados de pago documentado
- [ ] Manejo de errores y refunds definido
- [ ] Integración con Wallet especificada

#### Para historias del módulo Tickets
- [ ] Formato de código QR especificado
- [ ] Proceso de check-in definido
- [ ] Manejo de boletos de cortesía clarificado
- [ ] Integración con Seats definida

#### Para historias del módulo Security
- [ ] Roles y permisos especificados
- [ ] Flujo de autenticación definido
- [ ] Criterios de validación de sesión
- [ ] Logs de auditoría requeridos

---

## Razones comunes por las que una historia NO está ready

| Problema | Qué hacer |
|---------|-----------|
| Requerimientos poco claros | Programar sesión de refinamiento de 30min con PO |
| Faltan criterios de aceptación | PO agrega criterios antes del siguiente sprint |
| Dependencias desconocidas | Tech Lead revisa y documenta dependencias |
| Muy grande (> 8 SP) | Dividirla en historias más pequeñas |
| Sin acceso a ambiente de pruebas | DevOps genera credenciales antes del sprint |
| Contrato de API poco claro | Acordar contrato (OpenAPI) antes de empezar |
| Modelo de datos indefinido | Definir esquema SQL en el módulo correspondiente |

---

## DoR vs DoD

| | Definition of Ready (DoR) | Definition of Done (DoD) |
|-|--------------------------|--------------------------|
| **Cuándo** | Antes de empezar la historia | Después de terminar la historia |
| **Quién verifica** | Equipo en planning/refinement | Equipo en review |
| **Propósito** | Asegurar que el equipo puede empezar sin bloqueos | Asegurar que el incremento es desplegable |

---

## Plantillas de criterios de aceptación

### Para funcionalidad de Events:
```
Dado que soy un organizador autenticado
Cuando creo un evento con fecha [específica] y capacidad [número]
Entonces el sistema debe [acción específica]
Y debo poder [verificación]
```

### Para funcionalidad de Payments:
```
Dado que tengo un carrito con boletos por valor $[monto]
Cuando selecciono el método de pago [método]
Entonces el sistema debe [procesar/validar/confirmar]
Y el estado del pago debe ser [estado específico]
```

### Para funcionalidad de Security:
```
Dado que soy un usuario con rol [rol específico]
Cuando intento acceder a [recurso específico]
Entonces el sistema debe [permitir/denegar]
Y debe registrar [log específico]
```

---

## Correlaciones

- DoD completo → `00-governance/definition-of-done.md`
- Plantilla de historia de usuario → `01_Especificacion_Requisitos/Historias_Usuario_TuEvento.docx`
- Backlog de historias → `01_Especificacion_Requisitos/`
- Módulos SQL → `04_Diseno_Software/SQL/`