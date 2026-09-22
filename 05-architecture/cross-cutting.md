# TuEvento — Cross-cutting Concerns

> **Transversal technical concerns** that apply across all modules in TuEvento.
> These are implemented consistently to ensure maintainability, observability, and reliability.

---

## 1. Authentication & Authorization

### Implementation
- **Framework**: Spring Security with JWT token validation
- **Authentication**: Custom JWT authentication filter in security module
- **Authorization**: Method-level security with `@PreAuthorize` annotations

### Configuration
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/events/public/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtAuthenticationFilter(), 
                UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

### Usage Pattern
```java
@PreAuthorize("hasRole('ORGANIZER') and @eventSecurity.isEventOwner(#eventId, authentication.name)")
public EventDto updateEvent(String eventId, UpdateEventRequest request) {
    // Implementation
}
```

---

## 2. Logging & Correlation

### Implementation
- **Framework**: SLF4J with Logback JSON encoder
- **Correlation ID**: MDC (Mapped Diagnostic Context) for request tracing
- **Structured logging**: JSON format for consistent log parsing

### Configuration
```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp/>
                <logLevel/>
                <loggerName/>
                <mdc/>
                <message/>
                <stackTrace/>
            </providers>
        </encoder>
    </appender>
    
    <logger name="com.capysoft.tuevento" level="INFO"/>
    <logger name="org.springframework.security" level="WARN"/>
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

### Correlation ID Filter
```java
@Component
public class CorrelationIdFilter implements Filter {
    
    private static final String CORRELATION_ID_HEADER = "X-Correlation-ID";
    private static final String CORRELATION_ID_MDC_KEY = "correlationId";
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String correlationId = httpRequest.getHeader(CORRELATION_ID_HEADER);
        
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        
        try {
            MDC.put(CORRELATION_ID_MDC_KEY, correlationId);
            ((HttpServletResponse) response).setHeader(CORRELATION_ID_HEADER, correlationId);
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

---

## 3. Error Handling

### Global Exception Handler
```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.warn("Validation error: {}", ex.getMessage());
        return ResponseEntity.badRequest()
            .body(ErrorResponse.builder()
                .code("VALIDATION_ERROR")
                .message(ex.getMessage())
                .timestamp(Instant.now())
                .correlationId(MDC.get("correlationId"))
                .build());
    }
    
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) {
        log.warn("Entity not found: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.builder()
                .code("ENTITY_NOT_FOUND")
                .message(ex.getMessage())
                .timestamp(Instant.now())
                .correlationId(MDC.get("correlationId"))
                .build());
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ErrorResponse.builder()
                .code("INTERNAL_ERROR")
                .message("An unexpected error occurred")
                .timestamp(Instant.now())
                .correlationId(MDC.get("correlationId"))
                .build());
    }
}
```

### Standard Error Response
```java
@Data
@Builder
public class ErrorResponse {
    private String code;
    private String message;
    private Instant timestamp;
    private String correlationId;
    private List<FieldError> fieldErrors;
    
    @Data
    @Builder
    public static class FieldError {
        private String field;
        private String message;
        private Object rejectedValue;
    }
}
```

---

## 4. Input Validation

### Bean Validation (JSR-303)
```java
@RestController
@RequestMapping("/api/v1/events")
@Validated
public class EventController {
    
    public ResponseEntity<EventDto> createEvent(
            @Valid @RequestBody CreateEventRequest request) {
        // Implementation
    }
}

@Data
public class CreateEventRequest {
    
    @NotBlank(message = "Event name is required")
    @Size(min = 3, max = 100, message = "Event name must be between 3 and 100 characters")
    private String name;
    
    @NotNull(message = "Start date is required")
    @Future(message = "Start date must be in the future")
    private LocalDateTime startDate;
    
    @NotNull(message = "End date is required")
    @Future(message = "End date must be in the future")
    private LocalDateTime endDate;
    
    @Valid
    @NotNull(message = "Venue information is required")
    private VenueRequest venue;
    
    // Custom validation
    @ValidDateRange(start = "startDate", end = "endDate")
    public void validateDateRange() {
        // Implementation handled by validator
    }
}
```

### Custom Validators
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = DateRangeValidator.class)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    
    String start();
    String end();
}

public class DateRangeValidator implements ConstraintValidator<ValidDateRange, Object> {
    
    private String startFieldName;
    private String endFieldName;
    
    @Override
    public void initialize(ValidDateRange constraintAnnotation) {
        this.startFieldName = constraintAnnotation.start();
        this.endFieldName = constraintAnnotation.end();
    }
    
    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        try {
            LocalDateTime startDate = getFieldValue(value, startFieldName);
            LocalDateTime endDate = getFieldValue(value, endFieldName);
            
            return startDate == null || endDate == null || startDate.isBefore(endDate);
        } catch (Exception e) {
            return false;
        }
    }
}
```

---

## 5. Database Transactions

### Configuration
```java
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

### Usage Patterns
```java
@Service
@Transactional(readOnly = true)
public class EventService {
    
    @Transactional
    public EventDto createEvent(CreateEventRequest request) {
        // All operations in single transaction
        Event event = eventRepository.save(eventMapper.toEntity(request));
        eventMediaService.processMedia(event.getId(), request.getMedia());
        eventNotificationService.notifyEventCreated(event);
        return eventMapper.toDto(event);
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void reserveSeats(String eventId, List<String> seatIds, String userId) {
        // Complex operation with explicit rollback rules
        List<Seat> seats = seatRepository.findAllByIdIn(seatIds);
        validateSeatsAvailable(seats);
        
        seats.forEach(seat -> {
            seat.reserveFor(userId);
            seatRepository.save(seat);
            
            // Publish real-time update
            webSocketService.broadcastSeatUpdate(eventId, seat);
        });
        
        // Set expiration timer
        reservationExpirationService.scheduleExpiration(eventId, seatIds, userId);
    }
}
```

---

## 6. Audit Logging

### JPA Auditing Configuration
```java
@Configuration
@EnableJpaAuditing
public class AuditConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            return Optional.ofNullable(authentication)
                .map(Authentication::getName)
                .or(() -> Optional.of("system"));
        };
    }
}
```

### Auditable Base Entity
```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class AuditableEntity {
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
    
    @Version
    private Long version;
}
```

### Custom Audit Events
```java
@Component
@Slf4j
public class PaymentAuditService {
    
    @Async
    @EventListener
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        AuditLog auditLog = AuditLog.builder()
            .entityType("Payment")
            .entityId(event.getPaymentId())
            .action("PAYMENT_PROCESSED")
            .userId(event.getUserId())
            .details(Map.of(
                "amount", event.getAmount(),
                "currency", event.getCurrency(),
                "gateway", event.getGateway(),
                "transactionId", event.getTransactionId()
            ))
            .timestamp(Instant.now())
            .correlationId(MDC.get("correlationId"))
            .build();
            
        auditLogRepository.save(auditLog);
        log.info("Payment audit logged: paymentId={}, userId={}, amount={}", 
            event.getPaymentId(), event.getUserId(), event.getAmount());
    }
}
```

---

## 7. Caching Strategy

### Redis Configuration
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
                
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .transactionAware()
            .build();
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

### Caching Usage
```java
@Service
public class EventService {
    
    @Cacheable(value = "events", key = "#eventId")
    public EventDto getEvent(String eventId) {
        return eventRepository.findById(eventId)
            .map(eventMapper::toDto)
            .orElseThrow(() -> new EntityNotFoundException("Event not found"));
    }
    
    @CacheEvict(value = "events", key = "#eventId")
    public EventDto updateEvent(String eventId, UpdateEventRequest request) {
        // Update logic
    }
    
    @Cacheable(value = "event-list", key = "#categoryId + '-' + #page + '-' + #size")
    public Page<EventSummaryDto> getEventsByCategory(String categoryId, int page, int size) {
        // Implementation
    }
}
```

---

## 8. Health Checks & Monitoring

### Custom Health Indicators
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(1)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("status", "Connected")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "PostgreSQL")
                .withDetail("error", e.getMessage())
                .build();
        }
        
        return Health.down()
            .withDetail("database", "PostgreSQL")
            .withDetail("status", "Invalid connection")
            .build();
    }
}

@Component
public class WompiHealthIndicator implements HealthIndicator {
    
    @Autowired
    private WompiApiClient wompiApiClient;
    
    @Override
    public Health health() {
        try {
            WompiHealthResponse response = wompiApiClient.checkHealth();
            return Health.up()
                .withDetail("service", "Wompi")
                .withDetail("status", response.getStatus())
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("service", "Wompi")
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

### Application Metrics
```java
@Component
public class BusinessMetrics {
    
    private final Counter seatReservations = Metrics.counter("seat.reservations.total");
    private final Counter paymentTransactions = Metrics.counter("payment.transactions.total");
    private final Gauge activeWebSocketConnections = Gauge.builder("websocket.connections.active")
        .register(Metrics.globalRegistry);
    private final Timer paymentProcessingTime = Metrics.timer("payment.processing.duration");
    
    public void recordSeatReservation() {
        seatReservations.increment();
    }
    
    public void recordPaymentTransaction(String status) {
        paymentTransactions.increment(Tags.of("status", status));
    }
    
    public Timer.Sample startPaymentTimer() {
        return Timer.start(Metrics.globalRegistry);
    }
}
```

---

## 9. CORS Configuration

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins(
                "http://localhost:3000",  // Development web
                "https://tuevento.com",   // Production web
                "https://app.tuevento.com" // Production app
            )
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
            
        registry.addMapping("/ws/**")
            .allowedOrigins(
                "http://localhost:3000",
                "https://tuevento.com",
                "https://app.tuevento.com"
            )
            .allowCredentials(true);
    }
}
```

---

## 10. Rate Limiting

```java
@Component
public class RateLimitingInterceptor implements HandlerInterceptor {
    
    private final RedisTemplate<String, String> redisTemplate;
    private static final String RATE_LIMIT_KEY_PREFIX = "rate_limit:";
    private static final int DEFAULT_LIMIT = 100; // requests per minute
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
                           Object handler) throws Exception {
        
        String clientId = getClientId(request);
        String key = RATE_LIMIT_KEY_PREFIX + clientId;
        
        String currentCountStr = redisTemplate.opsForValue().get(key);
        int currentCount = currentCountStr != null ? Integer.parseInt(currentCountStr) : 0;
        
        if (currentCount >= DEFAULT_LIMIT) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("{\"error\":\"Rate limit exceeded\"}");
            return false;
        }
        
        // Increment counter
        redisTemplate.opsForValue().increment(key);
        redisTemplate.expire(key, Duration.ofMinutes(1));
        
        // Add rate limit headers
        response.setHeader("X-RateLimit-Limit", String.valueOf(DEFAULT_LIMIT));
        response.setHeader("X-RateLimit-Remaining", String.valueOf(DEFAULT_LIMIT - currentCount - 1));
        
        return true;
    }
    
    private String getClientId(HttpServletRequest request) {
        String userId = getCurrentUserId(request);
        return userId != null ? userId : request.getRemoteAddr();
    }
}
```

---

## Implementation Guidelines

### Module Integration
Each module integrates these cross-cutting concerns through:

1. **Dependency Injection**: Shared components autowired in module services
2. **Aspect-Oriented Programming**: Cross-cutting aspects applied via annotations
3. **Configuration**: Module-specific configuration in `application.yml`
4. **Documentation**: Usage examples in module README files

### Testing Considerations
- **Unit tests**: Mock external dependencies (Redis, external APIs)
- **Integration tests**: Use `@TestContainers` for full stack testing
- **Performance tests**: Monitor impact of cross-cutting concerns on response times

### Evolution Strategy
- **Backward compatibility**: Maintain existing interfaces when enhancing features
- **Feature flags**: Use configuration to enable/disable new cross-cutting features
- **Monitoring**: Track performance impact of new cross-cutting implementations

---

## References

- Spring Security configuration → `modules/security/infrastructure/config/`
- Logging configuration → `src/main/resources/logback-spring.xml`
- Cache configuration → `shared/infrastructure/config/CacheConfig.java`
- Health check implementations → `shared/infrastructure/health/`
- Validation examples → `modules/*/application/dto/`
- Error handling → `shared/interfaces/advice/GlobalExceptionHandler.java`