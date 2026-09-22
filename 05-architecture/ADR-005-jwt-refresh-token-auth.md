# ADR-005 — JWT + Refresh Token Authentication

| Field | Value |
|-------|-------|
| **ID** | ADR-005 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento requires a secure authentication system that supports:

- **Multi-platform access**: Web (Vite), Mobile (Expo), future API integrations
- **Stateless scalability**: No server-side session storage dependency
- **Security requirements**: Secure token handling, account lockout, password recovery
- **User roles**: USER, ORGANIZER, ADMIN with different permissions
- **Social login**: OAuth integration with Google, Facebook
- **Mobile-friendly**: Long-lived sessions without frequent re-authentication
- **Security compliance**: Protection against token theft, replay attacks
- **Performance**: Fast authentication checks without database hits

The solution must balance security, user experience, and development simplicity for a small team.

---

## Evaluated alternatives

### Alternative A — JWT + Refresh Token (CHOSEN)
**Short-lived JWT access tokens + long-lived refresh tokens stored in Redis**
- **Pros:**
  - Stateless authentication - no server session storage required
  - Mobile-friendly with automatic token refresh
  - Scalable - tokens contain all necessary user information
  - Cross-platform compatibility (web, mobile, API)
  - Industry standard with extensive Spring Security support
  - Social login integration straightforward
  - Short access token lifetime limits damage from token theft
  - Refresh tokens can be revoked centrally
- **Cons:**
  - More complex implementation than sessions
  - Refresh token management adds Redis dependency
  - Token size larger than session cookies

### Alternative B — Traditional Sessions + Cookies
**Server-side sessions stored in Redis with HttpOnly cookies**
- **Pros:**
  - Simpler implementation with Spring Session
  - Smaller request overhead (session ID only)
  - Easy session invalidation
  - Built-in CSRF protection
- **Cons:**
  - Mobile app complications with cookie handling
  - CORS complexity for cross-domain requests
  - Less suitable for API integrations
  - Session storage scaling challenges
  - Mobile platforms don't handle cookies naturally

### Alternative C — OAuth-Only (No local auth)
**Delegate all authentication to external OAuth providers**
- **Pros:**
  - No password management complexity
  - Leverages user's existing accounts
  - Reduced security liability
- **Cons:**
  - External dependency for core functionality
  - User experience issues when OAuth providers unavailable
  - Limited control over user management
  - Not all users have social accounts
  - Business model constraints (platform dependency)

### Alternative D — API Keys
**Simple API key-based authentication**
- **Pros:**
  - Very simple to implement
  - No token expiration complexity
- **Cons:**
  - No user identity/role information
  - Difficult key rotation
  - Not suitable for end-user applications
  - Poor security practices for user-facing auth

### Alternative E — Session + JWT Hybrid
**Sessions for web, JWT for mobile**
- **Pros:**
  - Optimized per platform
  - Leverages best of both approaches
- **Cons:**
  - Dual implementation complexity
  - Inconsistent security model
  - More testing scenarios
  - Code duplication and maintenance overhead

---

## Decision

**Alternative A:** JWT Access Tokens + Refresh Token pattern

**Authentication flow:**
```
1. Login (email/password or OAuth) → Server validates credentials
2. Server generates:
   - JWT access token (1 hour expiration)
   - Refresh token (7 days expiration)
3. Access token sent in response, refresh token stored in Redis
4. Client uses access token for API requests (Authorization: Bearer)
5. When access token expires, client uses refresh token to get new pair
6. Refresh tokens can be revoked for logout/security
```

**JWT access token payload:**
```json
{
  "sub": "user-uuid",
  "username": "user@example.com", 
  "roles": ["USER", "ORGANIZER"],
  "iat": 1693123456,
  "exp": 1693127056
}
```

**Security measures:**
- **Access tokens**: 1-hour expiration, stateless validation
- **Refresh tokens**: 7-day expiration, stored in Redis with user mapping
- **Token rotation**: New refresh token issued on each refresh
- **Revocation**: Refresh tokens deleted from Redis on logout
- **Account lockout**: 5 failed attempts → 30-minute lockout
- **Password requirements**: Minimum 8 chars, uppercase, lowercase, number, special char

**Implementation details:**
- **JWT library**: Spring Security OAuth2 Resource Server
- **Token storage**: Redis with TTL matching token expiration
- **Signing algorithm**: HS256 with secure random secret
- **Role mapping**: Spring Security authorities from JWT claims
- **Social login**: Spring Security OAuth2 Client for Google/Facebook

---

## Consequences

**Positive:**
- **Scalability** — Stateless tokens eliminate session storage bottlenecks
- **Mobile support** — Natural token handling in mobile applications
- **Security** — Short access token lifetime limits exposure window
- **Flexibility** — Easy integration with future API consumers
- **Performance** — No database lookup for authentication checks
- **Cross-platform** — Consistent authentication across web and mobile
- **Social integration** — Straightforward OAuth provider integration

**Negative:**
- **Complexity** — More implementation complexity than simple sessions
- **Token size** — Larger request headers compared to session cookies
- **Refresh logic** — Client applications must handle token refresh
- **Redis dependency** — Refresh token storage requires Redis availability

**Security considerations:**
- **Token storage**: Mobile apps use secure storage (Keychain/Keystore)
- **Web storage**: Access tokens in memory only, refresh tokens in HttpOnly cookies
- **HTTPS enforcement**: All token transmission over secure connections
- **Token validation**: Signature verification and expiration checks on every request
- **Refresh rotation**: Prevents refresh token reuse attacks

**Implementation phases:**
- **Phase 1**: Basic JWT authentication with email/password
- **Phase 2**: Refresh token mechanism and proper token rotation
- **Phase 3**: OAuth social login integration
- **Phase 4**: Account lockout and security policies

**Monitoring and security:**
- **Failed login tracking**: Monitor for brute force attempts
- **Token usage analytics**: Track token refresh patterns for anomaly detection
- **Security alerts**: Unusual login locations or device patterns

---

## References

- Security module implementation → `modules/security/`
- Account lockout policies → `00-governance/evidence/security-rules.md`
- Redis session configuration → `ADR-004-postgresql-redis-strategy.md`
- API authentication patterns → `07-api/contracts/openapi/_shared.yaml`
- OAuth provider configuration → `modules/security/infrastructure/external/`