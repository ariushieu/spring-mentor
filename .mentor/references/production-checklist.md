# Production Readiness Checklist — Java Spring Boot

> Use this at any phase to check current state.
> Run a full check before starting Phase 4 and Phase 5.

---

## Code Quality
- [ ] No `System.out.println` or debug logging left in production paths
- [ ] No credentials or secrets hardcoded anywhere in source files
- [ ] Error handling at every external boundary (DB, API, file system)
- [ ] Errors do not expose stack traces to the client (use `@ControllerAdvice`)
- [ ] Input validated at controller layer (`@Valid`, `@NotNull`, `@Size`, etc.)
- [ ] No commented-out dead code
- [ ] No `@SuppressWarnings` hiding real issues
- [ ] No raw types (e.g., `List` instead of `List<User>`)

## Git Workflow (all phases)
- [ ] `.gitignore` configured (no `target/`, `.env`, `.idea/`)
- [ ] All commits follow Conventional Commits format (`feat`, `fix`, `refactor`, etc.)
- [ ] No commit contains unrelated changes
- [ ] Student uses feature branches for Phase 2+
- [ ] Commit history is readable: `git log --oneline` tells a clear story

## Modern Java (if Java 17+)
- [ ] DTOs use Java Records instead of classes with getters/setters
- [ ] Switch expressions used instead of if/else chains on enums
- [ ] `var` used for local variables where type is obvious
- [ ] `Optional` handled properly (`.orElseThrow()`, `.map()`, never `.get()`)
- [ ] Text blocks used for multi-line strings in tests

## SOLID Compliance (after Phase 2)
- [ ] Controllers contain no business logic — only: validate → call service → respond
- [ ] Services contain no HTTP or SQL code
- [ ] All dependencies injected via constructor (no `@Autowired` on fields)
- [ ] Services depend on interfaces, not concrete implementations
- [ ] No `new ConcreteClass()` inside service methods
- [ ] No God classes (> 300 lines or > 5 injected dependencies)

## Architecture (after Phase 3)
- [ ] Clear package structure: controller / service / repository / entity / dto / config
- [ ] Controllers return DTOs, never JPA entities
- [ ] `@Transactional` is on service methods, not controllers
- [ ] Exception handling centralized in `@ControllerAdvice`
- [ ] No circular dependencies between packages
- [ ] Mapper classes separate from service classes
- [ ] Swagger/OpenAPI set up with `springdoc-openapi`
- [ ] All endpoints documented with `@Operation` and `@ApiResponses`
- [ ] Swagger UI disabled in production profile

## Testing (after Phase 2–3)
- [ ] Unit tests exist for all service classes (JUnit 5 + Mockito)
- [ ] Service layer test coverage ≥ 80%
- [ ] Custom repository queries tested with `@DataJpaTest`
- [ ] Controller endpoints tested with `@WebMvcTest`
- [ ] At least one integration test with `@SpringBootTest`
- [ ] Tests run successfully: `./mvnw verify`
- [ ] Error/edge cases covered (not just happy path)

## Security
- [ ] `application.properties` / `.env` is in `.gitignore` — never committed
- [ ] `.env.example` exists with all keys but no real values
- [ ] Dependencies have no known high-severity vulnerabilities
  (`./mvnw dependency-check:check` or `mvn verify -P owasp`)
- [ ] Spring Security configured if authentication is needed
- [ ] No SQL built with string concatenation (use `@Query` with `@Param`
  or Spring Data derived queries)
- [ ] CORS configured properly if frontend exists

## Configuration
- [ ] `application-prod.yml` exists with production settings
- [ ] `spring.jpa.hibernate.ddl-auto` set to `validate` in prod
- [ ] `spring.jpa.show-sql` set to `false` in prod
- [ ] Sensitive values externalized via environment variables
- [ ] Logging level appropriate: `WARN` for root, `INFO` for app package

## Docker (after Phase 4)
- [ ] `Dockerfile` uses multi-stage build (Maven → JRE)
- [ ] Base image is `eclipse-temurin` with specific version tag
- [ ] Container runs as non-root user
- [ ] JVM flags set for container awareness (`MaxRAMPercentage`)
- [ ] `.dockerignore` excludes `target/`, `.git`, `.env`, `.idea`, `.mentor`
- [ ] `docker compose up` starts full stack with no manual steps
- [ ] Spring Boot Actuator `/actuator/health` returns 200 when app is ready
- [ ] Production image size is reasonable (< 300MB)
- [ ] `start_period` set in healthcheck (Spring Boot needs ~30s to start)

## CI/CD (after Phase 5)
- [ ] Tests run in the pipeline before deploy (`./mvnw verify`)
- [ ] All secrets stored in GitHub Actions Secrets (zero hardcoded values)
- [ ] Deploy only triggers on push to `main` (not on PRs)
- [ ] Health check auto-rollback implemented in deployment script
- [ ] Student knows how to manually roll back
- [ ] App is live at HTTPS URL

## Observability
- [ ] Logs include timestamp, level, and enough context to debug
- [ ] Student knows: `docker logs app -f`
- [ ] Student knows: `docker ps` and `docker inspect`
- [ ] Student knows: how to grep for exceptions in logs
- [ ] Spring Boot Actuator health includes DB status
