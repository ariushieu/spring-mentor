# Roadmap Template — Java Spring Boot

> This is a structural guide for generating ROADMAP.md after the first project scan.
> The agent must replace every placeholder with content specific to the actual project.
> Do not output generic tasks. Every task must name a real class, method, or package.

---

## Output Format for ROADMAP.md

```markdown
# Project Roadmap — [Project Name]

Generated: YYYY-MM-DD
Stack: Java [version] + Spring Boot [version] + [database]
Build: Maven / Gradle
Target: Internship-ready in 10 weeks

---

## Phase 1 — Understand & Diagnose (Week 1–2)

Goal: Fully understand the current codebase and identify all SOLID violations.

### Week 1 — Architecture Mapping

- [ ] 1.1 — Read and trace a full request lifecycle from [specific Controller]
        → [specific Service] → [specific Repository] → database and back
- [ ] 1.2 — Draw the current architecture as a diagram (ASCII or Mermaid).
        Must include: packages, key classes, dependency arrows.
        Student must explain it without looking at code.
- [ ] 1.3 — Learn SRP: explain it using [specific Service class that has too
        many responsibilities]
- [ ] 1.4 — Learn DIP: compare field injection (@Autowired) vs constructor
        injection using [specific class]. Explain why constructor injection
        is preferred.

**Checkpoint**: Student explains data flow in their own words. Diagram is correct.

### Week 2 — SOLID Audit

- [ ] 2.1 — Audit [controller package] for SRP violations (business logic
        in controllers), document in AUDIT_REPORT.md
- [ ] 2.2 — Audit [service package] for DIP violations (field injection,
        missing interfaces, new ConcreteClass())
- [ ] 2.3 — Audit OCP (switch/if-else on types), LSP, ISP across remaining
        source files
- [ ] 2.4 — Finalize AUDIT_REPORT.md with severity ratings and refactor order

**Checkpoint**: Student can explain each violation found and why it violates SOLID.
               Student can point to the exact line in the codebase.

---

## Phase 2 — Refactor to SOLID + Unit Testing (Week 3–4)

Goal: Apply SOLID to highest-priority violations. Start writing tests.

### Week 3 — SRP + DIP Refactoring

- [ ] 3.1 — Refactor [specific God class] → split into [Service A] and
        [Service B / Mapper / Validator] based on responsibilities
- [ ] 3.2 — Convert all field injection (@Autowired) to constructor injection
        in [specific package or all services]
- [ ] 3.3 — Create interfaces for services that are injected into other
        services: [specific concrete dependencies]
- [ ] 3.4 — Extract DTOs: ensure [specific Controller] returns DTO, not Entity.
        Create [specific DTO class] and [specific Mapper class].
- [ ] 3.5 — Write unit tests for [most critical Service class] using
        JUnit 5 + Mockito (minimum 5 test cases: 3 happy path, 2 error path)
- [ ] 3.6 — Student explains before/after difference without notes

**Checkpoint**: At least one full module refactored. Unit tests pass.
Student creates a PR with a description explaining SOLID reasoning.

### Week 4 — OCP + ISP + Test Coverage

- [ ] 4.1 — Refactor [specific switch/if-else] using Strategy pattern with
        Spring's @Qualifier or polymorphic bean injection
- [ ] 4.2 — Split [specific fat interface] into smaller, focused contracts
- [ ] 4.3 — Write unit tests for [second Service class] — focus on edge cases
- [ ] 4.4 — Add @WebMvcTest for [main Controller] — test all endpoints
- [ ] 4.5 — Re-run SOLID audit. Compare before/after violation count.
- [ ] 4.6 — Write a short document: "What we changed and why"

**Checkpoint**: Core business logic is SOLID-compliant. Student can add a new
feature to the refactored module without touching existing code.
Service layer has 80%+ test coverage.

---

## Phase 3 — Clean Architecture + Integration Testing (Week 5–6)

Goal: Restructure layers. Add integration tests for critical paths.

### Week 5 — Layer Design

- [ ] 5.1 — Learn the 4 layers: Presentation, Application, Domain, Infrastructure
        Read `.mentor/references/clean-architecture.md`
- [ ] 5.2 — Map current package structure to target layers:
        - controller/ → Presentation
        - service/ → Application
        - entity/ + domain rules → Domain
        - repository/ + config/ → Infrastructure
- [ ] 5.3 — Design target folder structure — agree with mentor before writing code
- [ ] 5.4 — Move [specific module/feature] to new layer structure (vertical slice)
- [ ] 5.5 — Write @DataJpaTest for custom repository queries (if any exist)

### Week 6 — Layer Enforcement + Integration Tests + API Docs

- [ ] 6.1 — Ensure all controllers contain ZERO business logic — only:
        validate input → call service → map response
- [ ] 6.2 — Ensure all services contain ZERO HTTP or SQL code — only:
        business rules + repository calls
- [ ] 6.3 — Implement proper exception handling with @ControllerAdvice:
        GlobalExceptionHandler that maps domain exceptions to HTTP responses
- [ ] 6.4 — Write @SpringBootTest integration test for the most critical
        user flow (e.g., create → retrieve → update → delete)
- [ ] 6.5 — Set up Swagger/OpenAPI: add springdoc-openapi dependency,
        annotate all controllers with @Tag, @Operation, @ApiResponses.
        Read `.mentor/references/api-documentation.md`
- [ ] 6.6 — Annotate all request/response DTOs with @Schema (include examples)
- [ ] 6.7 — Test at least 3 endpoints via Swagger UI at /swagger-ui.html
- [ ] 6.8 — Write README section: "How to add a new feature" (follow the layers)

**Checkpoint**: Student can trace a full request through all 4 layers and name
the responsible class at each step. Integration test passes.
API documented with Swagger.

### Gap-Filling Tasks (added from Intern Readiness Analysis)

> These tasks are only generated if the intern readiness gap analysis (SKILL.md
> step 4) detected missing features. Place them in the appropriate phase/week.
> Remove this section if no gaps were found.

**Phase 2 gap tasks (if applicable):**
- [ ] G.1 — Fix N+1 queries in [entity] — add `JOIN FETCH` or `@EntityGraph`
        to [repository method]. Student explains: what is N+1? why is it bad?
- [ ] G.2 — Add `@Valid` + constraint annotations (`@NotBlank`, `@Email`,
        `@Size`) to [DTO]. Verify 400 response on invalid input.
- [ ] G.3 — Replace `System.out.println` with SLF4J `Logger` in [classes]

**Phase 3 gap tasks (if applicable):**
- [ ] G.4 — Add JWT authentication: Spring Security + `jjwt` library.
        Implement: register → login → receive token → use token on protected endpoints.
        Student explains: what is JWT? what are the 3 parts? why stateless?
- [ ] G.5 — Add pagination to [list endpoint] using `Pageable`:
        `GET /api/users?page=0&size=10&sort=name,asc`
- [ ] G.6 — Create `GlobalExceptionHandler` with `@ControllerAdvice`.
        Map domain exceptions → proper HTTP status codes + error response body.
- [ ] G.7 — Configure CORS for [frontend URL] using `WebMvcConfigurer`

**Phase 4 gap tasks (if applicable):**
- [ ] G.8 — Replace `ddl-auto: update` with Flyway migration scripts.
        Student explains: why not use ddl-auto in production?

---

## Phase 4 — Docker (Week 7–8)

Goal: Project runs anywhere with a single command.

### Week 7 — Containerization

- [ ] 7.1 — Write Dockerfile for Spring Boot with multi-stage build:
        Stage 1: Maven build → Stage 2: JRE runtime (eclipse-temurin)
        Student explains each instruction.
- [ ] 7.2 — Write docker-compose.yml for app + [database type] + any other services
- [ ] 7.3 — Add .env.example and .dockerignore
- [ ] 7.4 — Verify: `docker compose up` starts everything from scratch
        App connects to database, health check passes.

### Week 8 — Production Image

- [ ] 8.1 — Optimize image: verify multi-stage actually reduces size
        (`docker images | grep <image>`)
- [ ] 8.2 — Configure Spring Boot Actuator health endpoint:
        `/actuator/health` returns UP when DB is connected
- [ ] 8.3 — Verify container does not run as root (non-root user in Dockerfile)
- [ ] 8.4 — Add Spring profiles: `application-prod.yml` with production settings
- [ ] 8.5 — Student explains: why multi-stage? why non-root? why profiles?

**Checkpoint**: `docker compose up` works with no manual setup.
Image uses multi-stage build. Container passes health check.
`docker images` shows reasonable size (< 300MB for Spring Boot).

---

## Phase 5 — VPS + CI/CD (Week 9–10)

Goal: Push to main → app deploys automatically.

### Week 9 — Manual Deploy

- [ ] 9.1 — Provision VPS: SSH key, non-root user, firewall rules (22, 80, 443)
- [ ] 9.2 — Install Docker + Compose on VPS
- [ ] 9.3 — Set up Nginx reverse proxy + SSL with Certbot
- [ ] 9.4 — Push image to GHCR, pull and run on VPS manually
        Student understands each step.

### Week 10 — GitHub Actions

- [ ] 10.1 — Write GitHub Actions workflow: test → build → push image → deploy
        `mvn verify` → Docker build → push to GHCR → SSH deploy
- [ ] 10.2 — Add all secrets to GitHub Actions Secrets (zero hardcoded values)
- [ ] 10.3 — Test full pipeline: merge PR → auto deploy
- [ ] 10.4 — Verify rollback: deploy a broken version, roll back manually
- [ ] 10.5 — Student explains every line of the workflow YAML file

**Checkpoint**: Push to main triggers automatic deployment.
App is live with HTTPS. Student can deploy, roll back, and read server logs.

---

## Definition of Done

- [ ] Student explains SOLID with examples from this project
- [ ] Student adds a feature without modifying existing code (OCP proof)
- [ ] Unit test coverage ≥ 80% for service layer
- [ ] `docker compose up` runs the full stack locally
- [ ] GitHub Actions deploys on every merge to main
- [ ] Student reads VPS logs and debugs a failing container
- [ ] README is clear enough for a new developer to onboard

---

## Progress

COMPLETED_TASKS: 0
TOTAL_TASKS: ~45
CURRENT_PHASE: 1
LAST_UPDATED: YYYY-MM-DD
```

---

## Instructions for the Agent

When generating ROADMAP.md:

1. Replace every `[specific class]`, `[specific Controller]`, `[specific Service]`
   with real values from the project scan.
2. The task descriptions must be concrete enough that the student knows exactly
   which file to open.
3. Phase 1–2 must be 100% project-specific. Phase 4–5 may stay slightly generic
   since they depend on decisions not yet made (VPS provider, database choice).
4. **Testing tasks must reference the actual classes discovered during scan.**
   Pick the most critical service for the first test target.
5. Set `COMPLETED_TASKS` to 0 and date in `LAST_UPDATED`.
6. Count total tasks and set `TOTAL_TASKS` accurately.
