# Reglas de Seguridad - TuEvento

> Estas reglas definen cómo el equipo maneja la seguridad a nivel de código y políticas.
> El incumplimiento de estas reglas es motivo de rechazo inmediato en code review.

---

## Principio fundamental

> **"Security by design, not as an afterthought"**

La seguridad se implementa desde el diseño, no como parche después.
Cada feature debe considerar sus implicaciones de seguridad desde el día 1.

---

## Gestión de secretos

### Reglas absolutas
- ❌ **NUNCA** hacer commit de secretos, tokens, passwords, API keys
- ❌ **NUNCA** hardcodear credenciales en el código fuente
- ❌ **NUNCA** loggear información sensible (passwords, tokens, datos personales)
- ✅ **SIEMPRE** usar variables de ambiente para secretos
- ✅ **SIEMPRE** usar .gitignore para archivos de configuración local

### Variables de ambiente requeridas

```bash
# Base de datos
DB_HOST=localhost
DB_PORT=5432
DB_NAME=tuevento_dev
DB_USER=tuevento_user
DB_PASSWORD=***SECRETO***

# JWT
JWT_SECRET=***SECRETO***
JWT_EXPIRATION=86400

# Gateways de pago
WOMPI_API_KEY=***SECRETO***
WOMPI_WEBHOOK_SECRET=***SECRETO***

# Notificaciones
EMAIL_SMTP_PASSWORD=***SECRETO***
SMS_API_KEY=***SECRETO***

# Encriptación
ENCRYPTION_KEY=***SECRETO***
SALT_ROUNDS=12
```

### Archivos que NUNCA deben estar en git:
```
.env
.env.local
.env.production
config/secrets.json
credentials.json
private_keys/
certificates/*.pem
```

---

## Autenticación y autorización

### JWT Token Management
- **Expiración:** Tokens deben expirar en máximo 24 horas
- **Refresh:** Implementar refresh tokens para sesiones largas
- **Invalidación:** Poder invalidar tokens comprometidos
- **Algoritmo:** Usar HS256 o RS256, nunca "none"

### Roles y permisos (basado en Security module.sql)

| Rol | Permisos |
|-----|----------|
| **ADMIN** | Gestión completa del sistema, acceso a todos los módulos |
| **ORGANIZER** | Crear eventos, gestionar sus propios eventos, ver reportes |
| **USER** | Comprar boletos, gestionar perfil, ver eventos públicos |
| **GUEST** | Solo lectura de eventos públicos |

### Implementación de autorización
```javascript
// Ejemplo de middleware de autorización
function requireRole(roles) {
    return (req, res, next) => {
        const userRole = req.user.role;
        if (!roles.includes(userRole)) {
            return res.status(403).json({error: 'Acceso denegado'});
        }
        next();
    };
}

// Uso
app.get('/admin/users', authenticateToken, requireRole(['ADMIN']), getUserList);
app.post('/events', authenticateToken, requireRole(['ORGANIZER', 'ADMIN']), createEvent);
```

---

## Validación de entrada

### Validaciones obligatorias
- **Sanitización:** Limpiar toda entrada del usuario
- **Longitud:** Validar límites máximos y mínimos
- **Formato:** Usar regex para validar emails, teléfonos, fechas
- **Tipo:** Validar tipos de datos antes del procesamiento
- **Encoding:** Escapar caracteres especiales para prevenir XSS

### Ejemplos de validación por módulo

#### Events Module
```javascript
const eventValidation = {
    event_name: {
        type: 'string',
        maxLength: 100,
        required: true,
        sanitize: true
    },
    start_date: {
        type: 'date',
        format: 'YYYY-MM-DD',
        futureOnly: true
    },
    capacity: {
        type: 'integer',
        min: 1,
        max: 50000
    }
};
```

#### Payment Module
```javascript
const paymentValidation = {
    amount: {
        type: 'decimal',
        min: 1000,  // Mínimo $1000 COP
        max: 10000000,  // Máximo $10M COP
        precision: 2
    },
    payment_method: {
        type: 'enum',
        values: ['card', 'pse', 'nequi', 'cash']
    }
};
```

---

## Protección contra vulnerabilidades

### SQL Injection
- ✅ **SIEMPRE** usar prepared statements
- ❌ **NUNCA** concatenar strings para queries
- ✅ **SIEMPRE** validar parámetros antes de queries

```sql
-- ✅ Correcto
SELECT * FROM user WHERE user_id = ? AND role = ?

-- ❌ Incorrecto 
SELECT * FROM user WHERE user_id = ${userId} AND role = '${role}'
```

### XSS (Cross-Site Scripting)
- ✅ Escapar output HTML
- ✅ Usar Content Security Policy headers
- ✅ Validar y sanitizar inputs
- ✅ Usar bibliotecas de sanitización (DOMPurify, validator.js)

### CSRF (Cross-Site Request Forgery)
- ✅ Usar tokens CSRF para forms
- ✅ Verificar header Origin/Referer
- ✅ Usar SameSite cookies

---

## Logging y monitoreo de seguridad

### Eventos que DEBEN loggearse

#### Authentication Events
```javascript
// Login exitoso
log.security('USER_LOGIN', {userId, ip, userAgent, timestamp});

// Login fallido
log.security('LOGIN_FAILED', {email, ip, reason, timestamp});

// Logout
log.security('USER_LOGOUT', {userId, sessionDuration, timestamp});
```

#### Authorization Events
```javascript
// Acceso denegado
log.security('ACCESS_DENIED', {userId, resource, requiredRole, timestamp});

// Escalación de privilegios
log.security('PRIVILEGE_ESCALATION', {userId, oldRole, newRole, changedBy, timestamp});
```

#### Payment Events
```javascript
// Transacción iniciada
log.security('PAYMENT_INITIATED', {userId, amount, paymentMethod, timestamp});

// Transacción fallida
log.security('PAYMENT_FAILED', {userId, amount, errorCode, timestamp});
```

### Formato de logs de seguridad
```json
{
    "timestamp": "2024-01-15T10:30:00Z",
    "level": "SECURITY",
    "event": "LOGIN_FAILED",
    "userId": null,
    "ip": "192.168.1.100",
    "userAgent": "Mozilla/5.0...",
    "details": {
        "email": "user@example.com",
        "reason": "invalid_password",
        "attempt": 3
    }
}
```

---

## Configuraciones de seguridad

### Headers de seguridad obligatorios
```javascript
// Express.js ejemplo
app.use((req, res, next) => {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
    next();
});
```

### Rate limiting
```javascript
// Limitar intentos de login
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutos
    max: 5, // máximo 5 intentos
    message: 'Demasiados intentos de login'
});

app.post('/login', loginLimiter, authenticateUser);
```

---

## Manejo de incidentes de seguridad

### Proceso de respuesta

1. **Detección** — Alertas automáticas o reporte manual
2. **Contención** — Bloquear acceso/usuario comprometido
3. **Investigación** — Revisar logs, determinar impacto
4. **Remediación** — Aplicar fixes, cambiar credenciales
5. **Documentación** — Registrar incidente y lecciones aprendidas

### Contactos de emergencia
- **Tech Lead:** Cristofer David Lozano Contreras - 313 4605214 / cristoferlozano233@gmail.com
- **DevOps:** Jhampier Santos Ortiz - 302 7700760 / ortizjhampier@gmail.com
- **Stakeholder Original:** Instructor Jhon Corredor (idea del proyecto)

### Herramientas de monitoreo
- **Pipeline CI/CD:** Jenkins + SonarQube + Docker + Kubernetes
- **Repositorio:** GitHub - https://github.com/lozano2303/TuEvento
- **Base de datos:** PostgreSQL 17
- **Logs centralizados:** Spring Boot Actuator + SLF4J

---

## Correlaciones con otros documentos

- **Definition of Done** → Include security validations
- **Modelo de datos** → Security module (`04_Diseno_Software/SQL/Security module.sql`)
- **API contracts** → Include security schemas in OpenAPI
- **Testing** → Include security test cases