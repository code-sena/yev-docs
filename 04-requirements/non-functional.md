# Non-Functional Requirements (NFR) — TuEvento

> **Requisitos no funcionales de TuEvento** — Especificación de calidad, rendimiento y restricciones técnicas.
> Basado en la arquitectura del sistema, stack tecnológico y necesidades del proyecto.

---

## Performance Requirements

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-001 | API Response time | p95 < 200ms for CRUD operations | Load test with K6 |
| NFR-002 | WebSocket seat updates | < 100ms latency for real-time updates | WebSocket performance test |
| NFR-003 | Database query performance | < 50ms for simple queries, < 500ms for complex aggregations | Database performance monitoring |
| NFR-004 | Payment processing time | < 5 seconds from request to response | Wompi integration tests |
| NFR-005 | QR code generation | < 1 second per ticket | Performance benchmark |
| NFR-006 | File upload speed | Support files up to 10MB with < 30s upload time | File upload stress test |

---

## Scalability Requirements

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-010 | Concurrent users | Support 500 concurrent users per event | Load test simulation |
| NFR-011 | Real-time seat updates | Handle 100 simultaneous seat reservations | WebSocket stress test |
| NFR-012 | Database connections | Support up to 200 concurrent database connections | Connection pool monitoring |
| NFR-013 | Event capacity | Support events with up to 50,000 seats | Database capacity test |
| NFR-014 | API throughput | Handle 1000 requests per minute peak load | API performance test |

---

## Availability & Reliability

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-020 | System uptime | 99.5% monthly availability | Production monitoring |
| NFR-021 | Payment system availability | 99.9% availability during business hours | Wompi integration monitoring |
| NFR-022 | Database backup | Automated daily backups with 30-day retention | Backup verification script |
| NFR-023 | Recovery time | System recovery within 4 hours of failure | Disaster recovery test |
| NFR-024 | Seat reservation timeout | Automatic release after 15 minutes of inactivity | Timeout mechanism test |

---

## Security Requirements

| ID | Requirement | Description | How to verify |
|----|------------|-------------|--------------|
| NFR-030 | Authentication | JWT tokens with 1-hour expiration, refresh tokens with 7-day expiration | Token lifecycle test |
| NFR-031 | Password security | Minimum 8 characters, uppercase, lowercase, number, special character | Password validation test |
| NFR-032 | Account lockout | Lock account after 5 failed login attempts for 30 minutes | Security penetration test |
| NFR-033 | Data encryption | HTTPS/TLS 1.3 for all client-server communication | SSL certificate validation |
| NFR-034 | Payment security | PCI DSS compliance through Wompi gateway | Compliance audit |
| NFR-035 | SQL injection protection | Parameterized queries and input validation | Security code review |
| NFR-036 | API rate limiting | Maximum 100 requests per minute per user | Rate limiting test |

---

## Usability & Interface

| ID | Requirement | Description | How to verify |
|----|------------|-------------|--------------|
| NFR-040 | Responsive design | Support for mobile (320px+) and desktop (1024px+) | Cross-device testing |
| NFR-041 | Browser compatibility | Chrome 90+, Edge 90+, Safari 14+, Firefox 88+ | Browser compatibility test |
| NFR-042 | Mobile platform support | Android 11+, iOS 14+ (functional support) | Device testing |
| NFR-043 | Language support | Spanish, English, Portuguese, Japanese | Internationalization test |
| NFR-044 | Theme support | Light, dark, purple, blue themes | UI theme validation |
| NFR-045 | Accessibility | WCAG 2.1 AA compliance for key user flows | Accessibility audit |

---

## Data & Storage

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-050 | Database size | Support up to 1TB database size | Storage capacity monitoring |
| NFR-051 | File storage | Support up to 100GB for event media | File system monitoring |
| NFR-052 | Data retention | Keep transaction logs for 7 years | Data retention policy verification |
| NFR-053 | Backup integrity | 99.99% backup success rate | Backup monitoring |
| NFR-054 | Data consistency | Zero data loss during normal operations | Data integrity checks |

---

## Integration & Compatibility

| ID | Requirement | Description | How to verify |
|----|------------|-------------|--------------|
| NFR-060 | Payment gateway | Integration with Wompi API v2 | API integration test |
| NFR-061 | Notification services | SMTP email and push notification support | Notification delivery test |
| NFR-062 | WebSocket support | SockJS for real-time seat updates | WebSocket connectivity test |
| NFR-063 | Database compatibility | PostgreSQL 17+ required | Database version verification |
| NFR-064 | API versioning | Maintain backward compatibility for 2 major versions | API compatibility test |

---

## Platform Constraints

| ID | Requirement | Description | Validation |
|----|------------|-------------|-----------|
| NFR-070 | Internet connectivity | Required for all system operations | Connection dependency test |
| NFR-071 | Web platform exclusive | Venue layout design only on web interface | Feature availability verification |
| NFR-072 | Mobile platform features | Event browsing, seat selection, QR scanning on mobile | Mobile feature test |
| NFR-073 | Local deployment | All services deployable in local environment | Local deployment test |

---

## Quality & Monitoring

| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-080 | Code coverage | Minimum 80% unit test coverage | SonarQube analysis |
| NFR-081 | Build time | Maven build completes in < 5 minutes | CI/CD pipeline monitoring |
| NFR-082 | Critical vulnerabilities | Zero critical security vulnerabilities | Security scanning |
| NFR-083 | Code quality | SonarQube Quality Gate passing | Code quality analysis |
| NFR-084 | Error logging | Comprehensive logging for all critical operations | Log analysis verification |

---

## Hardware & Infrastructure Requirements

### Minimum Server Requirements
| Component | Specification |
|-----------|--------------|
| CPU | 4 cores, 2.0 GHz |
| RAM | 8 GB |
| Storage | 100 GB SSD |
| Network | 100 Mbps |

### Recommended Server Requirements
| Component | Specification |
|-----------|--------------|
| CPU | 8 cores, 2.5 GHz |
| RAM | 16 GB |
| Storage | 500 GB SSD |
| Network | 1 Gbps |

### Development Environment
| Component | Specification |
|-----------|--------------|
| Java | OpenJDK 23+ |
| Database | PostgreSQL 17 |
| Container | Docker Desktop 4.0+ |
| Build Tool | Maven 3.9+ |

---

## Non-Functional Requirements Summary

| Category | Total NFRs | Critical | High Priority | Medium Priority |
|----------|------------|----------|---------------|----------------|
| Performance | 6 | 3 | 2 | 1 |
| Scalability | 5 | 2 | 2 | 1 |
| Availability | 5 | 1 | 3 | 1 |
| Security | 7 | 4 | 3 | 0 |
| Usability | 6 | 0 | 4 | 2 |
| Data & Storage | 5 | 1 | 2 | 2 |
| Integration | 5 | 2 | 2 | 1 |
| Platform | 4 | 2 | 1 | 1 |
| Quality | 5 | 2 | 2 | 1 |
| **TOTAL** | **48** | **17** | **21** | **10** |

---

## Implementation Priority Matrix

### Critical (Must Have) - MVP Blockers
**17 NFRs** que son bloqueadores para el MVP:
- Autenticación JWT segura
- Actualizaciones WebSocket en tiempo real
- Integración con Wompi
- Compatibilidad móvil y web básica

### High Priority (Should Have) - Enhanced MVP
**21 NFRs** que mejoran significativamente la experiencia:
- Rendimiento optimizado de APIs
- Monitoreo y respaldos
- Seguridad avanzada
- Soporte multi-idioma completo

### Medium Priority (Could Have) - Future Enhancements
**10 NFRs** para versiones futuras:
- Optimizaciones avanzadas de rendimiento
- Características de accesibilidad completas
- Herramientas de monitoreo avanzado

---

## Correlations with other sections

- **Functional Requirements** → `04-requirements/evidence/functional.md`
- **Architecture Decisions** → `05-architecture/decisions/records/`
- **Testing Strategy** → `11-quality/testing-strategy.md`
- **Security Policies** → `00-governance/evidence/security-rules.md`
- **Performance Testing** → `12-testing/performance-tests.md`
- **Infrastructure** → `13-deployment/infrastructure.md`

---

## Verification & Monitoring

Cada NFR debe tener:
1. **Métrica cuantificable** — Valores específicos a cumplir
2. **Método de verificación** — Cómo validar el cumplimiento
3. **Herramientas de monitoreo** — Para seguimiento continuo
4. **Criterios de aceptación** — Definición clara de éxito/fallo

### Tools for NFR Verification
- **Performance**: K6, JMeter, Apache Bench
- **Security**: SonarQube, OWASP ZAP, Burp Suite
- **Quality**: SonarQube, SpotBugs, Checkstyle
- **Monitoring**: Prometheus, Grafana, ELK Stack
- **Testing**: JUnit, TestContainers, Selenium