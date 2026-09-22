# TuEvento — Deployment Strategy

> **Deployment architecture and infrastructure requirements** for TuEvento across development, staging, and production environments.

---

## 1. Deployment Overview

**Architecture**: Containerized Spring Boot monolith with external services
**Orchestration**: Docker Compose for development, Kubernetes for production
**Database**: Managed PostgreSQL with Redis cache
**Scaling**: Horizontal scaling ready with load balancer

```
┌─────────────────────────────────────────────────────────────────┐
│                    Production Environment                        │
│                                                                 │
│  ┌──────────────┐    ┌─────────────────────────────────────┐   │
│  │ Load Balancer│    │         Application Layer           │   │
│  │   (NGINX)    │    │                                     │   │  
│  │              │    │  ┌─────────────┐ ┌─────────────┐   │   │
│  │     :80      │────▶  │TuEvento App │ │TuEvento App │   │   │
│  │    :443      │    │  │   :8080     │ │   :8080     │   │   │
│  └──────────────┘    │  │(Docker)     │ │(Docker)     │   │   │
│                      │  └─────────────┘ └─────────────┘   │   │
│                      └─────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Data Layer                               │   │
│  │                                                         │   │
│  │  ┌─────────────────┐      ┌─────────────────────────┐  │   │
│  │  │ PostgreSQL      │      │ Redis Cluster           │  │   │
│  │  │ (Managed)       │      │ (Cache + Sessions)      │  │   │
│  │  │   :5432         │      │   :6379                 │  │   │
│  │  └─────────────────┘      └─────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Environment Configurations

### Development Environment

**Target**: Local development and testing
**Orchestration**: Docker Compose

```yaml
# docker-compose.yml (simplified)
version: '3.8'
services:
  tuevento-app:
    build: ./TuEvento
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=development
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
      
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: tuevento_db
      POSTGRES_USER: tuevento
      POSTGRES_PASSWORD: dev_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
```

**Resource requirements:**
- **CPU**: 2 cores minimum
- **RAM**: 4GB minimum (2GB app + 1GB PostgreSQL + 512MB Redis)
- **Storage**: 20GB for development data and containers
- **Network**: Local only, no external access required

### Staging Environment  

**Target**: QA testing and integration validation
**Orchestration**: Docker Compose with production-like configuration

**Key differences from development:**
- **External database**: Managed PostgreSQL instance
- **SSL termination**: NGINX with Let's Encrypt certificates
- **Monitoring**: Application metrics and logging
- **Backup**: Automated database backups
- **Performance**: Production-like resource allocation

**Resource requirements:**
- **CPU**: 4 cores (2 app instances)
- **RAM**: 8GB total
- **Storage**: 100GB with backup retention
- **Network**: Public access with domain

### Production Environment

**Target**: Live user traffic
**Orchestration**: Kubernetes cluster with Helm charts

```yaml
# kubernetes/deployment.yaml (simplified)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tuevento-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tuevento
  template:
    spec:
      containers:
      - name: tuevento
        image: tuevento:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi" 
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
```

---

## 3. Infrastructure Requirements

### Compute Resources

| Environment | App Instances | CPU per Instance | RAM per Instance | Total CPU | Total RAM |
|-------------|--------------|------------------|------------------|-----------|-----------|
| Development | 1 | 1 core | 2GB | 1 core | 2GB |
| Staging | 2 | 2 cores | 2GB | 4 cores | 4GB |
| Production | 3-5 | 2 cores | 4GB | 6-10 cores | 12-20GB |

### Database Requirements

| Environment | PostgreSQL | RAM | Storage | Backup |
|-------------|------------|-----|---------|--------|
| Development | Local container | 1GB | 10GB | None |
| Staging | Managed instance | 4GB | 100GB | Daily |
| Production | Managed HA cluster | 16GB | 500GB+ | Continuous |

### Network Requirements

| Environment | Load Balancer | SSL/TLS | CDN | Bandwidth |
|-------------|--------------|---------|-----|-----------|
| Development | None | Self-signed | No | Local only |
| Staging | NGINX | Let's Encrypt | No | 100Mbps |
| Production | Cloud LB | Managed SSL | Yes | 1Gbps+ |

---

## 4. Container Configuration

### Application Dockerfile

```dockerfile
# Dockerfile (production optimized)
FROM openjdk:23-jdk-slim AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN ./mvnw clean package -DskipTests

FROM openjdk:23-jre-slim
WORKDIR /app
COPY --from=builder /app/target/tuevento-*.jar app.jar

# Security and optimization
RUN adduser --disabled-password --gecos '' appuser
USER appuser
EXPOSE 8080

# JVM optimization
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

CMD ["java", "$JAVA_OPTS", "-jar", "app.jar"]
```

### Build Strategy

**Multi-stage builds**: Separate build and runtime environments
**Layer optimization**: Dependency layers cached separately from application code
**Security**: Non-root user, minimal base image
**Size optimization**: Alpine-based images, dependency analysis

---

## 5. Deployment Pipeline

### CI/CD Workflow

```yaml
# .github/workflows/deploy.yml (simplified)
name: Deploy TuEvento
on:
  push:
    branches: [main, develop]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        java-version: '23'
        
    - name: Run tests
      run: ./mvnw test
      
    - name: Build Docker image  
      run: docker build -t tuevento:${{ github.sha }} .
      
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        docker tag tuevento:${{ github.sha }} tuevento:staging
        # Deploy to staging environment
        
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'  
      run: |
        docker tag tuevento:${{ github.sha }} tuevento:production
        # Deploy to production environment
```

### Deployment Stages

1. **Build**: Compile, test, create Docker image
2. **Security scan**: Container vulnerability scanning
3. **Deploy staging**: Automatic deployment for QA testing
4. **Integration tests**: Automated API and UI tests
5. **Manual approval**: Required for production deployment
6. **Deploy production**: Blue-green deployment strategy
7. **Health checks**: Verify application health post-deployment
8. **Rollback**: Automatic rollback on health check failures

---

## 6. Scaling Strategy

### Horizontal Scaling

**Scaling triggers:**
- CPU usage > 70% for 5 minutes
- Memory usage > 80% for 5 minutes  
- Response time > 500ms for 95th percentile
- Active WebSocket connections > 400 per instance

**Auto-scaling configuration:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: tuevento-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: tuevento-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Database Scaling

**Read replicas**: For heavy read workloads (event browsing)
**Connection pooling**: Optimized HikariCP configuration
**Query optimization**: Regular performance analysis
**Caching strategy**: Redis for frequently accessed data

---

## 7. Security Configuration

### Network Security

- **Firewall rules**: Restrict access to database and Redis ports
- **VPC/Private networks**: Isolate backend services
- **SSL/TLS**: End-to-end encryption for all communications
- **API rate limiting**: Prevent abuse and DDoS attacks

### Application Security

```yaml
# Security configuration
security:
  jwt:
    secret: ${JWT_SECRET:must-be-set-in-production}
    access-token-validity: 3600 # 1 hour
    refresh-token-validity: 604800 # 7 days
  
  cors:
    allowed-origins: 
      - https://tuevento.com
      - https://app.tuevento.com
    allowed-methods: [GET, POST, PUT, DELETE, OPTIONS]
    
  rate-limiting:
    requests-per-minute: 60
    burst-capacity: 10
```

### Secrets Management

- **Environment variables**: Sensitive configuration
- **Kubernetes secrets**: Database credentials, JWT keys
- **External secret management**: HashiCorp Vault or cloud KMS
- **Secret rotation**: Automated credential rotation

---

## 8. Monitoring and Observability

### Health Checks

```java
// Custom health indicators
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // Check database connectivity
        // Check Redis connectivity  
        // Check external service status (Wompi)
    }
}
```

### Metrics Collection

- **Application metrics**: Custom business metrics (seat reservations, payments)
- **Infrastructure metrics**: CPU, memory, network, disk usage
- **Database metrics**: Connection pool, query performance
- **WebSocket metrics**: Active connections, message rates

### Logging Strategy

```yaml
# logback-spring.xml
logging:
  level:
    com.capysoft.tuevento: INFO
    org.springframework.security: DEBUG # Only in non-production
  pattern:
    console: "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: /var/log/tuevento/application.log
```

---

## 9. Disaster Recovery

### Backup Strategy

| Component | Frequency | Retention | Recovery Time |
|-----------|-----------|-----------|---------------|
| PostgreSQL | Continuous WAL + Daily snapshot | 30 days | < 15 minutes |
| Redis | Daily snapshot | 7 days | < 5 minutes |
| Application config | On change | Indefinite | < 1 minute |
| File uploads | Daily | 90 days | < 30 minutes |

### Recovery Procedures

1. **Database recovery**: Point-in-time recovery from WAL
2. **Application recovery**: Redeploy from container registry
3. **Configuration recovery**: Restore from version control
4. **Data validation**: Automated data integrity checks

### Business Continuity

- **RTO** (Recovery Time Objective): 30 minutes maximum downtime
- **RPO** (Recovery Point Objective): 1 hour maximum data loss
- **Failover**: Automated failover for critical components
- **Communication**: Automated status page updates

---

## References

- Infrastructure as Code → `deployment/terraform/` or `deployment/helm/`
- CI/CD pipeline configuration → `.github/workflows/` or `.gitlab-ci.yml`
- Environment-specific configs → `src/main/resources/application-{env}.yml`
- Container security guidelines → `00-governance/evidence/security-rules.md`
- Monitoring setup → `13-deployment/monitoring.md`