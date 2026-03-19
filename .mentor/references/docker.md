# Docker Reference — Java Spring Boot

> Load this file during Phase 4 (Week 7–8).
> All examples use Maven + Spring Boot with eclipse-temurin JRE images.

---

## Core Concepts (teach in this order)

1. **Image** — a blueprint (read-only snapshot of the app + OS + JRE + deps)
2. **Container** — a running instance of an image
3. **Volume** — persistent storage that survives container restarts
4. **Network** — how containers talk to each other
5. **Registry** — where images are stored (Docker Hub, GHCR)

---

## Dockerfile Checklist

A production Dockerfile for Spring Boot must:

- [ ] Use a specific base image version tag (never `latest`)
- [ ] Use multi-stage build (Maven build → JRE runtime)
- [ ] Run as a non-root user
- [ ] Copy only the final JAR, not source code or Maven cache
- [ ] Set active Spring profile to `prod`
- [ ] Expose the correct port (default 8080)
- [ ] Have a clear ENTRYPOINT with JVM flags

---

## Multi-Stage Build — Spring Boot

```dockerfile
# ============================================
# Stage 1: Build (contains Maven, JDK, source code)
# ============================================
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

# Copy Maven wrapper and POM first (layer caching for dependencies)
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN chmod +x mvnw

# Download dependencies (cached if pom.xml doesn't change)
RUN ./mvnw dependency:go-offline -B

# Copy source code and build
COPY src/ src/
RUN ./mvnw package -DskipTests -B

# ============================================
# Stage 2: Production (only JRE + JAR)
# ============================================
FROM eclipse-temurin:21-jre-alpine AS production

WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy only the built JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Switch to non-root user
USER appuser

# Expose Spring Boot default port
EXPOSE 8080

# JVM flags for containers:
# - UseContainerSupport: respect container memory limits
# - MaxRAMPercentage: use 75% of container memory for heap
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-Dspring.profiles.active=prod", \
    "-jar", "app.jar"]
```

**Build stage**: ~500MB+ (JDK + Maven cache + source code)
**Production stage**: ~200MB (JRE + single JAR)

Teach the student to measure: `docker images | grep <image-name>`

---

## Docker Compose — Spring Boot + PostgreSQL

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/${DB_NAME}
      - SPRING_DATASOURCE_USERNAME=${DB_USER}
      - SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}
      - SPRING_JPA_HIBERNATE_DDL_AUTO=validate
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  db_data:
```

### MySQL Variant

Replace the `db` service:
```yaml
  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```

And update the app's datasource URL:
```yaml
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/${DB_NAME}
```

---

## .env.example

```properties
DB_NAME=myapp
DB_USER=myapp_user
DB_PASSWORD=change_me_in_production
DB_ROOT_PASSWORD=change_me_root
```

---

## .dockerignore

```
.git
.gitignore
.mentor
*.md
target/
.idea/
*.iml
.mvn/wrapper/maven-wrapper.jar
.env
docker-compose.yml
```

---

## Spring Boot Actuator Health Check

Add the Actuator dependency to `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Configure in `application-prod.yml`:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health
  endpoint:
    health:
      show-details: never          # don't expose internals
      show-components: never
  health:
    db:
      enabled: true                # check DB connectivity
```

Actuator auto-creates `/actuator/health`:
- Returns `{"status": "UP"}` when DB is connected
- Returns `{"status": "DOWN"}` with 503 when DB is down

---

## Spring Profiles for Docker

### application.yml (default — development)
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp_dev
    username: dev_user
    password: dev_password
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

server:
  port: 8080
```

### application-prod.yml (production — Docker)
```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

server:
  port: 8080

logging:
  level:
    root: WARN
    com.example: INFO
```

---

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---------|----------------|-----|
| `FROM openjdk:latest` | Deprecated, bloated, breaks on updates | Use `eclipse-temurin:21-jre-alpine` |
| Single-stage build | Image includes JDK + Maven + source (1GB+) | Multi-stage: build → runtime |
| Running as root | Security risk | `adduser` + `USER appuser` |
| Copying `target/` in build stage | Layer cache invalidation | Copy `pom.xml` first, then source |
| No health check | Orchestrators can't detect failures | Use Spring Boot Actuator |
| `ddl-auto: update` in production | Unsafe schema changes | Use `validate` + Flyway/Liquibase |
| Hardcoding DB credentials in Dockerfile | Exposes secrets in image layers | Use environment variables |
| Not setting JVM memory flags | JVM ignores container limits | Use `MaxRAMPercentage` |
| Not using `start_period` in healthcheck | App fails health check while starting | Set `start_period: 40s` for Spring Boot |
