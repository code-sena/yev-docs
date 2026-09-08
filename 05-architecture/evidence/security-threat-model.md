# TuEvento — Security Threat Model (STRIDE)

> **Security threat analysis** using STRIDE methodology (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) for TuEvento platform.

---

## 1. System Overview & Assets

### Critical Assets
1. **User credentials and personal data** (emails, passwords, profiles)
2. **Payment information and financial transactions** 
3. **Event data and business intelligence** (sales, attendance)
4. **Seat reservations and real-time state**
5. **Authentication tokens and sessions**
6. **API keys and external service credentials**

### Trust Boundaries
1. **Public Internet ↔ Load Balancer/CDN**
2. **Load Balancer ↔ Application Layer** 
3. **Application Layer ↔ Database Layer**
4. **Application ↔ External Services** (Wompi, SMTP, OAuth)
5. **Web/Mobile Clients ↔ WebSocket Connections**

---

## 2. STRIDE Threat Analysis

## Spoofing (Identity Attacks)

### S-001: User Account Impersonation
**Threat**: Attacker gains access to user account through credential theft or session hijacking
**Impact**: HIGH - Unauthorized ticket purchases, access to personal data
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **Strong password requirements**: Min 8 chars, uppercase, lowercase, number, special char
- ✅ **Account lockout**: 5 failed attempts → 30-minute lockout
- ✅ **JWT with short expiration**: 1-hour access tokens
- ✅ **Refresh token rotation**: New refresh token on each refresh
- ✅ **Secure token storage**: HttpOnly cookies for web, secure storage for mobile
- ✅ **Session invalidation**: Proper logout implementation

**Additional Controls Needed:**
- 🔄 **Two-factor authentication** for high-value accounts (organizers)
- 🔄 **Device fingerprinting** for suspicious login detection
- 🔄 **IP-based anomaly detection**

### S-002: OAuth Provider Spoofing
**Threat**: Malicious OAuth redirect or man-in-the-middle attack during social login
**Impact**: MEDIUM - Account takeover through social login
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **HTTPS enforcement**: All OAuth redirects over secure connections
- ✅ **State parameter validation**: CSRF protection in OAuth flows
- ✅ **Authorized redirect URIs**: Strict whitelist of allowed redirect URLs
- ✅ **Provider certificate validation**: SSL/TLS certificate verification

### S-003: API Client Spoofing
**Threat**: Malicious client impersonates legitimate mobile/web application
**Impact**: MEDIUM - Unauthorized API access
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **JWT signature validation**: All API requests validate token signatures
- ✅ **User agent verification**: Basic client identification
- ✅ **Rate limiting**: Prevents automated attacks

**Additional Controls Needed:**
- 🔄 **API key authentication** for mobile applications
- 🔄 **Certificate pinning** in mobile apps

---

## Tampering (Data Integrity Attacks)

### T-001: Payment Transaction Tampering
**Threat**: Attacker modifies payment amounts or recipient information
**Impact**: CRITICAL - Financial fraud, revenue loss
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **HTTPS encryption**: All payment data encrypted in transit
- ✅ **Database constraints**: Foreign key constraints prevent data corruption
- ✅ **Transaction atomicity**: ACID compliance ensures consistency
- ✅ **Wompi webhook validation**: Signature verification for payment notifications
- ✅ **Audit logging**: Complete audit trail for all payment operations
- ✅ **Server-side validation**: All payment calculations performed server-side

**Additional Controls Needed:**
- 🔄 **Payment amount verification**: Cross-check with multiple sources
- 🔄 **Real-time fraud detection**: ML-based anomaly detection

### T-002: Seat Reservation Tampering
**Threat**: Attacker manipulates seat availability or reservation data
**Impact**: HIGH - Double bookings, revenue loss, customer dissatisfaction
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **Database locks**: Pessimistic locking for seat reservations
- ✅ **WebSocket authentication**: Authenticated WebSocket connections
- ✅ **Server-side validation**: All seat state changes validated server-side
- ✅ **Audit trail**: Complete log of seat state changes
- ✅ **Time-based expiration**: 15-minute reservation timeouts

### T-003: Event Data Tampering
**Threat**: Unauthorized modification of event details, pricing, or layouts
**Impact**: HIGH - Business disruption, financial impact
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Role-based access control**: Only organizers can modify their events
- ✅ **Ownership validation**: Event modification requires ownership verification
- ✅ **Input validation**: Comprehensive validation of all event data
- ✅ **Version control**: Optimistic locking prevents concurrent modifications

---

## Repudiation (Non-repudiation Attacks)

### R-001: Payment Transaction Repudiation
**Threat**: User denies making a payment or claims fraud
**Impact**: HIGH - Chargebacks, disputes, revenue loss
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **Complete audit trail**: All payment steps logged with timestamps
- ✅ **Correlation IDs**: Track requests across system boundaries  
- ✅ **User authentication logs**: Record of user authentication at payment time
- ✅ **Wompi integration logs**: External payment gateway transaction records
- ✅ **Email confirmations**: Automated payment confirmation emails

**Additional Controls Needed:**
- 🔄 **Digital signatures**: Cryptographic proof of transaction consent
- 🔄 **Video/screenshot capture**: Evidence of user actions (where legally permitted)

### R-002: Seat Reservation Repudiation
**Threat**: User denies making seat reservations or claims system error
**Impact**: MEDIUM - Customer service overhead, potential revenue loss
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Real-time audit logs**: All seat actions logged with user ID and timestamp
- ✅ **WebSocket message logging**: Record of all real-time communications
- ✅ **Browser session tracking**: Correlation between UI actions and backend operations

---

## Information Disclosure (Confidentiality Attacks)

### I-001: Personal Data Exposure
**Threat**: Unauthorized access to user personal information and profiles
**Impact**: HIGH - Privacy violation, GDPR compliance issues
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **Database encryption**: Sensitive data encrypted at rest
- ✅ **HTTPS enforcement**: All data encrypted in transit
- ✅ **Access control**: Users can only access their own data
- ✅ **Query filtering**: Automatic filtering by user ownership
- ✅ **Data minimization**: Only collect necessary personal data

**Additional Controls Needed:**
- 🔄 **Field-level encryption**: Additional encryption for PII
- 🔄 **Data anonymization**: Anonymous analytics and reporting
- 🔄 **Access logging**: Log all access to personal data

### I-002: Payment Information Disclosure
**Threat**: Exposure of payment card details or financial information
**Impact**: CRITICAL - Financial fraud, regulatory compliance violations
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **No card storage**: Payment card details never stored in system
- ✅ **Wompi tokenization**: External payment processor handles sensitive data
- ✅ **PCI DSS compliance**: Reduce scope through external payment processing
- ✅ **Network segmentation**: Database isolated from public networks

### I-003: Business Intelligence Leakage
**Threat**: Unauthorized access to event sales data, revenue information
**Impact**: MEDIUM - Competitive disadvantage, business intelligence loss
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Role-based reporting**: Organizers only see their event data
- ✅ **API endpoint protection**: Sales endpoints require authentication
- ✅ **Data aggregation**: Individual transaction details not exposed in APIs

---

## Denial of Service (Availability Attacks)

### D-001: API Rate Limiting Bypass
**Threat**: Automated attacks overwhelm API endpoints
**Impact**: HIGH - Service unavailability, performance degradation
**Likelihood**: HIGH

**Mitigations Implemented:**
- ✅ **Application-level rate limiting**: 100 requests/minute per user/IP
- ✅ **WebSocket connection limits**: Maximum connections per user
- ✅ **Database connection pooling**: Prevent connection exhaustion
- ✅ **Circuit breaker pattern**: Protect external service integrations

**Additional Controls Needed:**
- 🔄 **CDN with DDoS protection**: Cloudflare or similar service
- 🔄 **Advanced rate limiting**: Sliding window, burst capacity
- 🔄 **Auto-scaling**: Horizontal scaling based on load

### D-002: WebSocket Connection Flooding
**Threat**: Attacker opens excessive WebSocket connections
**Impact**: MEDIUM - Real-time service disruption
**Likelihood**: MEDIUM

**Mitigations Implemented:**
- ✅ **Connection authentication**: All WebSocket connections require valid JWT
- ✅ **Per-user limits**: Maximum connections per authenticated user
- ✅ **Connection timeouts**: Idle connections automatically closed
- ✅ **Resource monitoring**: Track connection counts and memory usage

### D-003: Database Resource Exhaustion
**Threat**: Malicious queries consume excessive database resources
**Impact**: HIGH - Complete service unavailability
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Query timeouts**: Maximum execution time for database queries
- ✅ **Connection pooling**: Limited database connection pool
- ✅ **Pagination**: Large result sets automatically paginated
- ✅ **Query optimization**: Indexes on frequently accessed columns

---

## Elevation of Privilege (Authorization Attacks)

### E-001: Role Escalation
**Threat**: User gains unauthorized administrative or organizer privileges
**Impact**: CRITICAL - Complete system compromise
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Role-based access control**: Strict role assignment and validation
- ✅ **Method-level security**: `@PreAuthorize` annotations on sensitive operations
- ✅ **Organizer approval process**: Manual approval required for organizer role
- ✅ **Admin role protection**: Admin role cannot be self-assigned
- ✅ **JWT claims validation**: Role claims verified on every request

**Additional Controls Needed:**
- 🔄 **Privilege escalation monitoring**: Alert on role changes
- 🔄 **Segregation of duties**: Multiple approvals for critical operations

### E-002: Event Ownership Bypass
**Threat**: User modifies events they don't own
**Impact**: HIGH - Unauthorized business operations
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **Ownership validation**: Every event operation verifies ownership
- ✅ **Database-level constraints**: Foreign key relationships enforce ownership
- ✅ **Query filtering**: Automatic filtering by ownership in repository layer
- ✅ **API parameter validation**: Event IDs validated against user permissions

### E-003: Cross-User Data Access
**Threat**: User accesses another user's tickets, reservations, or personal data
**Impact**: HIGH - Privacy violation, data breach
**Likelihood**: LOW

**Mitigations Implemented:**
- ✅ **User context filtering**: All queries filtered by current user
- ✅ **Resource ownership validation**: Verify ownership before data access
- ✅ **API authorization**: Each endpoint validates user permissions
- ✅ **Database views**: Use views that automatically filter by user context

---

## 3. Security Implementation Summary

### Implemented Controls (✅)

| Category | Controls Implemented | Coverage |
|----------|---------------------|----------|
| **Authentication** | JWT + Refresh Token, Account Lockout, Password Policy | 85% |
| **Authorization** | RBAC, Method Security, Ownership Validation | 90% |
| **Data Protection** | HTTPS, Database Encryption, Input Validation | 80% |
| **Audit & Logging** | Complete audit trail, Correlation IDs | 85% |
| **Rate Limiting** | API limits, Connection limits | 70% |
| **External Integration** | Wompi validation, OAuth security | 90% |

### Priority Security Enhancements (🔄)

| Priority | Enhancement | Timeline | Impact |
|----------|-------------|----------|--------|
| **P1** | CDN with DDoS protection | Sprint 4 | HIGH |
| **P1** | Advanced rate limiting | Sprint 4 | HIGH |
| **P2** | Two-factor authentication | Sprint 6 | MEDIUM |
| **P2** | API key authentication | Sprint 6 | MEDIUM |
| **P3** | Field-level encryption | Sprint 8 | LOW |
| **P3** | Fraud detection ML | Future | LOW |

### Security Monitoring

**Real-time Alerts:**
- Failed authentication attempts > 10/minute
- Privilege escalation attempts
- Unusual payment patterns
- WebSocket connection flooding
- Database query timeouts

**Security Metrics:**
- Authentication success/failure rates
- Authorization violation attempts
- API rate limiting triggers
- Payment transaction anomalies
- Data access patterns

---

## 4. Incident Response Plan

### Security Incident Classification

| Severity | Examples | Response Time |
|----------|----------|---------------|
| **Critical** | Data breach, Payment fraud, System compromise | < 15 minutes |
| **High** | Authentication bypass, Privilege escalation | < 1 hour |
| **Medium** | DoS attacks, Suspicious access patterns | < 4 hours |
| **Low** | Failed login attempts, Rate limiting triggers | < 24 hours |

### Response Procedures

1. **Detection**: Automated monitoring alerts + manual reporting
2. **Assessment**: Classify severity and determine scope
3. **Containment**: Isolate affected systems, block malicious IPs
4. **Investigation**: Analyze logs, determine root cause
5. **Remediation**: Fix vulnerabilities, restore services
6. **Recovery**: Verify system integrity, resume operations  
7. **Lessons Learned**: Update security measures, improve monitoring

---

## References

- Security implementation → `modules/security/`
- Authentication flow → `ADR-005-jwt-refresh-token-auth.md`
- Rate limiting implementation → `evidence/cross-cutting.md#rate-limiting`
- Audit logging → `evidence/cross-cutting.md#audit-logging`
- Security policies → `00-governance/evidence/security-rules.md`
- Payment security → `ADR-006-wompi-payment-gateway.md` (if exists)