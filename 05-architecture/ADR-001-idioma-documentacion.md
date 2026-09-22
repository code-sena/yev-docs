# ADR-001 — Documentation Language

| Field | Value |
|-------|-------|
| **ID** | ADR-001 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento is a multilingual event management platform supporting Spanish (ES), English (EN), Portuguese (PT), and Japanese (JA) through a comprehensive internationalization system. However, the technical documentation, codebase, and development artifacts require a consistent language strategy.

The software industry standard — Stack Overflow, library documentation, technical articles, frameworks, and open-source tooling — operates in English. Mixing languages across artifacts (Spanish docs, English code, Spanish API descriptions) creates a cognitive translation boundary that slows onboarding, increases errors in naming, and makes it harder to search for help online.

A single, clear rule established from day one prevents inconsistencies: mismatched variable names, Spanish comments in English code, and conflicting terminology between documentation and implementation.

**TuEvento's multilingual architecture:**
- **End-user content**: Translated via `language` table and `*_translation` tables
- **Technical artifacts**: Need consistent language for development team
- **Database schema**: 14+ translation tables supporting dynamic content localization
- **Business domain**: Colombian market with Spanish business terms

---

## Evaluated alternatives

### Alternative A — Everything in English (CHOSEN)
- **Pros:** 
  - Industry standard for technical documentation and code
  - Easy to hire external developers and integrate with international teams
  - Libraries and frameworks documentation available in English
  - Eliminates translation boundary between technical docs and code
  - GitHub, Stack Overflow, and AI tools work optimally with English content
  - Spring Boot, PostgreSQL, React ecosystem all documented in English
  - Error messages and logs searchable in English documentation
- **Cons:** 
  - May require more effort from team members less confident in English
  - Business domain terms need careful translation from Spanish originals

### Alternative B — Everything in Spanish
- **Pros:** 
  - Natural communication with Colombian business stakeholders
  - Domain terms like "silla", "evento", "boleto" preserved exactly as understood by users
  - Easier for team members whose primary language is Spanish
- **Cons:** 
  - Creates uncomfortable mix with English keywords (if, for, return, class, etc.)
  - Inconsistent with library ecosystem (Spring Boot, React, PostgreSQL docs)
  - External contributors cannot participate effectively
  - Technical problem-solving requires translating error messages and concepts
  - Database naming becomes inconsistent with ORM conventions

### Alternative C — Split by Audience (discarded)
**Technical artifacts in English, business documentation in Spanish**
- **Pros:** 
  - Each artifact uses most natural language for its audience
  - Business stakeholders get Spanish documentation
  - Developers get English technical docs
- **Cons:** 
  - Requires strict discipline and explicit rules for every artifact type
  - Creates permanent translation boundary between business and technical domains
  - Domain terms accumulate two canonical names (Spanish business + English technical)
  - Onboarding complexity increases significantly

### Alternative D — Hybrid with Glossary (discarded)
**English for technical, Spanish for domain, with comprehensive glossary**
- **Pros:**
  - Preserves business domain authenticity
  - Maintains technical documentation standards
- **Cons:**
  - High maintenance overhead for glossary
  - Still creates cognitive translation boundary
  - Inconsistency leads to errors and confusion

---

## Decision

**Alternative A:** Use English for all technical documentation, code, and development artifacts.

| Artifact Category | Language | Justification |
|------------------|----------|---------------|
| **Code (variables, functions, classes)** | English | Consistency with Java/Spring conventions and international libraries |
| **Database schema (tables, columns)** | English | Coherence with JPA entity mappings and Spring Data repositories |
| **API contracts (OpenAPI descriptions)** | English | Industry standard for REST APIs, readable by international consumers |
| **Git commits (Conventional Commits)** | English | Established standard, readable on GitHub, searchable |
| **Branch names and PR titles** | English | Consistent with commits and GitHub ecosystem |
| **Technical documentation (ADRs, architecture)** | English | Eliminates translation boundary, searchable, reusable by other teams |
| **Error messages and logs** | English | Facilitates search in library documentation and Stack Overflow |
| **Configuration files and deployment scripts** | English | Standard practice in DevOps and infrastructure tools |

**End-user facing content remains multilingual:**
- **Event names, descriptions**: Translated via `event_translation` table
- **UI labels and messages**: Handled by frontend internationalization
- **Notifications**: Stored in multiple languages via `notification_translation`
- **Categories and metadata**: Support all four languages (ES/EN/PT/JA)

**Domain term mapping:**
Critical business terms maintain English canonical form in technical artifacts:

| Spanish Business Term | English Technical Term | Usage Context |
|----------------------|----------------------|---------------|
| Evento | Event | Database tables, API endpoints, class names |
| Silla/Asiento | Seat | Entity names, reservation logic |
| Boleto/Entrada | Ticket | Payment processing, QR generation |
| Organizador | Organizer | Role-based access control |
| Sección | Section | Venue layout, seating management |

---

## Consequences

**Positive:**
- **Single source of truth**: Domain terms have one canonical form in the codebase (English)
- **Reduced cognitive load**: No mental translation required between docs and implementation
- **External collaboration**: International developers can contribute without language barriers
- **Problem-solving efficiency**: Error messages directly searchable in English documentation
- **AI tool compatibility**: ChatGPT, GitHub Copilot work optimally with English code comments
- **Recruitment flexibility**: Easier to onboard developers regardless of Spanish proficiency

**Negative:**
- **Learning curve**: Team members need to map Spanish business concepts to English technical terms
- **Business communication**: Requires translation layer when discussing technical implementation with Spanish-speaking stakeholders

**Mitigation strategies:**
- **Comprehensive glossary**: Maintain `01-context/glossary.md` with Spanish ↔ English business term mappings
- **Documentation standards**: All technical documentation includes business context in both languages where relevant
- **Code comments**: Use English for technical comments, include Spanish business context when helpful
- **Database documentation**: Include business meaning for technical entities

**Implementation guidelines:**
```java
// GOOD: English technical terms with business context
@Entity
@Table(name = "event")
public class Event {
    @Column(name = "event_name")
    private String name; // "Nombre del evento" in Spanish UI
    
    @Column(name = "start_date")  
    private LocalDateTime startDate; // "Fecha de inicio"
}

// GOOD: English API endpoints
@GetMapping("/events/{eventId}/seats")
public ResponseEntity<List<SeatDto>> getEventSeats(@PathVariable String eventId) {
    // Implementation
}

// GOOD: English commit messages
// feat(seating): add real-time seat reservation with WebSocket
// fix(payment): resolve Wompi webhook signature validation
```

**Future considerations:**
- **Team growth**: English documentation enables international team expansion
- **Open source potential**: Technical artifacts ready for open source contribution
- **Market expansion**: English technical foundation supports international market entry

---

## References

- Domain glossary → `01-context/glossary.md`
- Internationalization implementation → `TuEvento/DataBase/MR/DBML/modulo_idiomas.dbml`
- Language module documentation → `modules/language/README.md`
- Team documentation conventions → `00-governance/evidence/documentation-rules.md`
- Frontend internationalization → Frontend localization configuration