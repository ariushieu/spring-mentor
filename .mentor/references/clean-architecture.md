# Clean Architecture Reference — Java Spring Boot

> Load this file during Phase 3 (Week 5–6).

---

## The Four Layers in Spring Boot

```
┌──────────────────────────────────────────────┐
│            Presentation Layer                │
│  @RestController · @ControllerAdvice         │
│  Request/Response DTOs · @Valid              │
│  @ExceptionHandler · Swagger annotations     │
├──────────────────────────────────────────────┤
│            Application Layer                 │
│  @Service · Use Cases · Orchestration        │
│  @Transactional · Application DTOs           │
│  Business rule validation · Mapper calls     │
├──────────────────────────────────────────────┤
│              Domain Layer                    │
│  @Entity · Value Objects · Enums             │
│  Domain exceptions · Business rules          │
│  Repository interfaces (contracts only)      │
├──────────────────────────────────────────────┤
│          Infrastructure Layer                │
│  Repository implementations (JPA/Hibernate)  │
│  @Configuration · External API clients       │
│  Email/SMS services · File storage           │
│  Spring Security config · Bean definitions   │
└──────────────────────────────────────────────┘
           ↑ Dependencies flow inward only ↑
```

---

## The One Rule

Dependencies only point inward. Inner layers know nothing about outer layers.

| Layer          | Knows about         | Must NOT know about            |
|----------------|---------------------|--------------------------------|
| Domain         | itself              | everything else                |
| Application    | Domain              | Presentation, Infrastructure   |
| Infrastructure | Domain, Application | Presentation                   |
| Presentation   | Application         | Infrastructure (directly)      |

---

## Spring Boot Folder Structure

### Before (flat / mixed — typical student project)
```
src/main/java/com/example/myapp/
├── MyAppApplication.java
├── User.java                    ← entity
├── UserController.java          ← controller
├── UserService.java             ← service + business logic + mapping
├── UserRepository.java          ← repository
└── SecurityConfig.java          ← config
```

### After (layered — clean architecture)
```
src/main/java/com/example/myapp/
├── MyAppApplication.java
├── controller/
│   ├── UserController.java
│   ├── dto/
│   │   ├── CreateUserRequest.java
│   │   └── UserResponse.java
│   └── GlobalExceptionHandler.java
├── service/
│   ├── UserService.java                ← interface
│   ├── impl/
│   │   └── UserServiceImpl.java        ← implementation
│   └── mapper/
│       └── UserMapper.java
├── domain/
│   ├── entity/
│   │   └── User.java
│   ├── enums/
│   │   └── UserRole.java
│   └── exception/
│       ├── UserNotFoundException.java
│       └── DuplicateEmailException.java
├── repository/
│   └── UserRepository.java
└── config/
    ├── SecurityConfig.java
    └── AppConfig.java
```

---

## Common Mistakes in Spring Boot

### 1. Controller calling the repository directly

```java
// 🔴 BAD: Presentation reaching into data layer
@RestController
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @GetMapping("/users")
    public List<User> getUsers() {
        return userRepository.findAll();  // ← skips service layer, returns entity
    }
}

// ✅ GOOD: Controller calls service, returns DTO
@RestController
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping("/users")
    public ResponseEntity<List<UserResponse>> getUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }
}
```

### 2. Service knowing about HTTP

```java
// 🔴 BAD: Service layer importing HTTP classes
@Service
public class UserService {
    public ResponseEntity<User> getUser(HttpServletRequest request) {
        String token = request.getHeader("Authorization");  // ← HTTP concern
        // ...
    }
}

// ✅ GOOD: Service takes plain data, returns plain data
@Service
public class UserService {
    public UserResponse getUser(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException(id));
        return userMapper.toResponse(user);
    }
}
```

### 3. Business rule in Repository

```java
// 🔴 BAD: Business logic inside repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // This is fine — it's a query
    List<User> findByRole(UserRole role);
}

// But this is BAD — custom implementation with business logic
public class UserRepositoryImpl {
    public User createUser(User user) {
        if (user.getAge() < 18) {
            throw new RuntimeException("Must be adult");  // ← business rule!
        }
        return entityManager.persist(user);
    }
}

// ✅ GOOD: Business rule stays in Service
@Service
public class UserService {
    public UserResponse createUser(CreateUserRequest request) {
        if (request.getAge() < 18) {
            throw new UnderageException("Must be 18 or older");
        }
        User user = userMapper.toEntity(request);
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }
}
```

### 4. Entity returned from Controller

```java
// 🔴 BAD: Exposing JPA entity via REST API
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
    // Problems:
    // - Exposes database schema to client
    // - Lazy-loading exceptions (LazyInitializationException)
    // - Circular references in JSON (e.g., User ↔ Order)
    // - Can't vary response shape per use case
}

// ✅ GOOD: Return a DTO
@GetMapping("/users/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(userService.getUser(id));
}
```

### 5. @Transactional on Controller

```java
// 🔴 BAD: Transaction management in presentation layer
@RestController
public class OrderController {
    @Transactional  // ← wrong layer!
    @PostMapping("/orders")
    public Order createOrder(@RequestBody OrderRequest req) { ... }
}

// ✅ GOOD: @Transactional on service method
@Service
public class OrderService {
    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request) { ... }
}
```

---

## Repository Pattern in Spring Boot

The interface is defined by Spring Data (lives conceptually in the Domain layer).
The implementation is provided by Spring Data JPA (Infrastructure).

```java
// Repository interface — Domain boundary
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByRole(UserRole role);

    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
}

// Spring auto-generates the implementation at runtime.
// If you need custom logic, create a custom implementation:

public interface UserRepositoryCustom {
    List<User> searchUsers(UserSearchCriteria criteria);
}

public class UserRepositoryCustomImpl implements UserRepositoryCustom {
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> searchUsers(UserSearchCriteria criteria) {
        // Custom query logic using EntityManager
    }
}

// Combine both:
public interface UserRepository
        extends JpaRepository<User, Long>, UserRepositoryCustom {
    Optional<User> findByEmail(String email);
}
```

---

## DTO Mapping Strategies

### Option 1: Manual Mapper (recommended for learning)
```java
@Component
public class UserMapper {
    public UserResponse toResponse(User entity) {
        return new UserResponse(
                entity.getId(),
                entity.getEmail(),
                entity.getFullName(),
                entity.getRole().name()
        );
    }

    public User toEntity(CreateUserRequest request) {
        User user = new User();
        user.setEmail(request.getEmail());
        user.setFullName(request.getFullName());
        user.setRole(UserRole.valueOf(request.getRole()));
        return user;
    }
}
```

### Option 2: MapStruct (production projects)
```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User entity);
    User toEntity(CreateUserRequest request);
}
// MapStruct generates the implementation at compile time.
```

---

## Exception Handling with @ControllerAdvice

Centralize all error handling in one place:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                ex.getMessage(),
                LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .collect(Collectors.joining(", "));
        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                message,
                LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "An unexpected error occurred",
                LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

## Migration Strategy

When refactoring an existing Spring Boot project into Clean Architecture:

1. **Do not rewrite everything at once**
2. Start with **one vertical slice** (one feature, e.g., User CRUD, end-to-end)
3. Create the new folder structure first, move files one at a time
4. After each move: run `mvn test` to verify nothing broke
5. Extract DTOs before moving other things — this cuts entity exposure first
6. Create `@ControllerAdvice` early — centralizes error handling
7. Delete old files only after the new structure is verified working
8. Convert field injection to constructor injection as you touch each class
