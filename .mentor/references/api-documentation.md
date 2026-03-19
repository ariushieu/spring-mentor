# API Documentation Reference — Java Spring Boot

> Load this file during Phase 3 (Week 5–6).
> After the architecture is clean, document the API so it's presentable.

---

## Why API Documentation Matters

When you work on a team (or go to an internship), the **frontend developer**
needs to know:
- What endpoints exist?
- What data to send? (request body, path params, query params)
- What data comes back? (response shape, status codes)
- What errors are possible?

Without documentation, they have to read your Java code — which is inefficient.
**Swagger/OpenAPI** generates interactive API docs automatically from your code.

---

## Setup — SpringDoc OpenAPI

Add to `pom.xml`:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.8.6</version>
</dependency>
```

That's it. Run the app, then visit:
- **Swagger UI**: `http://localhost:8080/swagger-ui.html`
- **OpenAPI JSON**: `http://localhost:8080/v3/api-docs`

---

## Annotating Controllers

### Basic endpoint documentation
```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "Users", description = "User management endpoints")
public class UserController {

    @Operation(
        summary = "Create a new user",
        description = "Registers a new user and returns the created resource"
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "User created"),
        @ApiResponse(responseCode = "400", description = "Invalid input"),
        @ApiResponse(responseCode = "409", description = "Email already exists")
    })
    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @Valid @RequestBody CreateUserRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(userService.createUser(request));
    }

    @Operation(summary = "Get user by ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(
            @Parameter(description = "User ID") @PathVariable Long id) {
        return ResponseEntity.ok(userService.getUser(id));
    }
}
```

### DTO documentation
```java
@Schema(description = "Request to create a new user")
public class CreateUserRequest {

    @Schema(description = "User's email address", example = "john@example.com")
    @NotBlank
    @Email
    private String email;

    @Schema(description = "User's full name", example = "John Doe")
    @NotBlank
    @Size(min = 2, max = 100)
    private String fullName;

    @Schema(description = "User's role", example = "USER", allowableValues = {"USER", "ADMIN"})
    private String role;
}
```

---

## Configuration (application.yml)

```yaml
springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: method
  info:
    title: ${spring.application.name} API
    version: 1.0.0
    description: API documentation for the application
```

### Disable in production (security)
```yaml
# application-prod.yml
springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

---

## Testing API with HTTP Client

While Swagger UI lets you test in the browser, you can also use:

### IntelliJ HTTP Client (`.http` files)
Create a file `requests.http` in the project root:
```http
### Create a user
POST http://localhost:8080/api/users
Content-Type: application/json

{
  "email": "john@example.com",
  "fullName": "John Doe",
  "role": "USER"
}

### Get user by ID
GET http://localhost:8080/api/users/1

### Get all users
GET http://localhost:8080/api/users

### Delete user
DELETE http://localhost:8080/api/users/1
```

### curl (command line)
```bash
# Create user
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","fullName":"John Doe","role":"USER"}'

# Get user
curl http://localhost:8080/api/users/1
```

---

## Checklist

Before Phase 3 is complete:

- [ ] `springdoc-openapi` dependency added to `pom.xml`
- [ ] Swagger UI accessible at `/swagger-ui.html`
- [ ] All controllers annotated with `@Tag`
- [ ] All endpoints annotated with `@Operation` and `@ApiResponses`
- [ ] All request DTOs annotated with `@Schema` (with examples)
- [ ] Swagger disabled in `application-prod.yml`
- [ ] Student can explain: what is OpenAPI? why document APIs?
- [ ] Student has tested at least 3 endpoints via Swagger UI

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No `@ApiResponse` annotations | Add expected status codes for each endpoint |
| No examples in `@Schema` | Add `example = "..."` so Swagger shows realistic data |
| Swagger enabled in production | Disable via `application-prod.yml` |
| Documenting only happy path | Include 400, 404, 409, 500 responses |
| No `@Tag` on controller | Group endpoints logically with tags |
