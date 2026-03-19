# SOLID Audit Protocol — Java Spring Boot

> Load this file during Phase 1–2.
> This is the methodology for auditing a Spring Boot codebase against SOLID principles.

---

## How to Run the Audit

Scan every `.java` file under `src/main/java/`. For each file, apply the
checklist below. Record all violations in `.mentor/AUDIT_REPORT.md` with:
- Fully qualified class name
- Method name (if applicable)
- Line reference
- Severity (HIGH / MEDIUM / LOW)

---

## S — Single Responsibility Principle

**Definition**: A class should have only one reason to change.

**Scan for in Spring Boot:**

- **Controllers with business logic**:
  ```java
  // 🔴 BAD: Controller doing validation, business logic, AND persistence
  @RestController
  public class OrderController {
      @PostMapping("/orders")
      public ResponseEntity<?> createOrder(@RequestBody OrderRequest req) {
          // validation logic here
          if (req.getTotal() < 0) throw new RuntimeException("Invalid");
          // business rule
          double discount = req.getTotal() > 100 ? req.getTotal() * 0.1 : 0;
          // database call
          Order order = new Order();
          order.setTotal(req.getTotal() - discount);
          orderRepository.save(order);
          return ResponseEntity.ok(order);
      }
  }
  ```

- **God Service classes** — services with 300+ lines, or methods longer than 30 lines
- **Service classes that import from unrelated domains** (e.g., imports both
  `EmailService` and `PaymentRepository` AND `FileStorageService`)
- **Names containing "Utils", "Helper", "Common", "Manager"** — SRP graveyards

**Severity rules:**
- **HIGH**: Business logic mixed with HTTP handling or DB queries in the same method
- **HIGH**: A `@Service` class with 5+ injected dependencies (too many responsibilities)
- **MEDIUM**: A class with 3+ distinct responsibilities
- **LOW**: Utility class with loosely related static methods

---

## O — Open/Closed Principle

**Definition**: Open for extension, closed for modification.

**Scan for in Spring Boot:**

- **if/else or switch on type strings**:
  ```java
  // 🔴 BAD: Must modify this method to add a new notification type
  public void sendNotification(String type, String message) {
      if ("email".equals(type)) {
          emailService.send(message);
      } else if ("sms".equals(type)) {
          smsService.send(message);
      } else if ("push".equals(type)) {
          pushService.send(message);
      }
      // Adding Slack? Must edit this method.
  }
  ```

- **Functions that take a `type` / `category` / `role` string and branch on it**
- **Long conditionals that grow every time a feature is added**
- **Missing use of Strategy pattern or Spring's polymorphic bean injection**

**How to confirm:** Ask: "If I add a new type, do I need to edit this class?" → If yes, OCP violation.

**Severity rules:**
- **HIGH**: Core business logic with type-switching that changes frequently
- **MEDIUM**: Utility or mapping functions with type-branching
- **LOW**: One-off branching unlikely to grow

---

## L — Liskov Substitution Principle

**Definition**: Subtypes must be substitutable for their base types.

**Scan for in Spring Boot:**

- **Methods that throw `UnsupportedOperationException`** in a subclass:
  ```java
  // 🔴 BAD: ReadOnlyUserRepository can't substitute UserRepository
  public class ReadOnlyUserRepository implements UserRepository {
      @Override
      public void save(User user) {
          throw new UnsupportedOperationException("Read only!");
      }
  }
  ```

- **Subclasses that override a method but do nothing** (empty body)
- **Code that checks `instanceof` before calling a method**:
  ```java
  // 🔴 BAD: Leaking abstraction
  if (payment instanceof CreditCardPayment) {
      ((CreditCardPayment) payment).verifyCVV();
  }
  ```
- **Subclass that weakens preconditions or strengthens postconditions**

**Severity rules:**
- **HIGH**: A throw or no-op override in a subclass used in production flow
- **MEDIUM**: `instanceof` check in business logic
- **LOW**: Incomplete implementation in a rarely-used subclass

---

## I — Interface Segregation Principle

**Definition**: Clients should not be forced to depend on interfaces they do not use.

**Scan for in Spring Boot:**

- **Interfaces with 6+ unrelated methods**:
  ```java
  // 🔴 BAD: One interface forcing two roles
  public interface UserService {
      User findById(Long id);
      List<User> findAll();
      User save(User user);
      void delete(Long id);
      void sendWelcomeEmail(User user);    // ← unrelated: email concern
      void generateReport(User user);      // ← unrelated: reporting concern
  }
  ```

- **Classes that implement an interface but leave methods empty or throwing**
- **A single interface used across completely different contexts**
- **Spring `@Repository` interfaces extending `JpaRepository` plus many custom
  methods that not all callers need** — consider splitting into focused query interfaces

**Severity rules:**
- **HIGH**: Interface with empty/throw implementations forced on concrete classes
- **MEDIUM**: Interface that could be meaningfully split into 2
- **LOW**: Slightly overloaded interface where all methods are used by most implementors

---

## D — Dependency Inversion Principle

**Definition**: High-level modules must not depend on low-level modules.
Both should depend on abstractions.

> **This is typically the #1 violation in student Spring Boot projects.**

**Scan for in Spring Boot:**

- **Field injection (`@Autowired` on fields)**:
  ```java
  // 🔴 BAD: Field injection — hidden dependencies, hard to test
  @Service
  public class OrderService {
      @Autowired
      private OrderRepository orderRepository;
      @Autowired
      private EmailService emailService;
  }

  // ✅ GOOD: Constructor injection — explicit, testable
  @Service
  public class OrderService {
      private final OrderRepository orderRepository;
      private final EmailService emailService;

      public OrderService(OrderRepository orderRepository,
                          EmailService emailService) {
          this.orderRepository = orderRepository;
          this.emailService = emailService;
      }
  }
  ```

- **`new ConcreteClass()` inside services** — creating dependencies instead
  of receiving them
- **Service directly using `EntityManager`, `JdbcTemplate`, or raw SQL**
  instead of going through a Repository interface
- **Hardcoded values** (URLs, connection strings, magic numbers) inside classes
  instead of `@Value` or `@ConfigurationProperties`
- **Services depending on concrete classes** instead of interfaces:
  ```java
  // 🔴 BAD: Depends on concrete implementation
  private final PostgresUserRepository repo;

  // ✅ GOOD: Depends on abstraction
  private final UserRepository repo;
  ```

- **No interfaces between Controller → Service or Service → Repository**

**Severity rules:**
- **HIGH**: Field injection (`@Autowired` on fields) across the project
- **HIGH**: No interfaces between layers at all
- **HIGH**: Business logic directly calling `EntityManager` or external APIs
- **MEDIUM**: Mixed DI styles (some constructor, some field injection)
- **MEDIUM**: Hardcoded config values that should be externalized
- **LOW**: Minor hardcoded value in a test or config class

---

## Audit Report Format

After completing the scan, write `.mentor/AUDIT_REPORT.md` using this structure:

```markdown
# SOLID Audit Report

Generated: YYYY-MM-DD
Project: [name]
Spring Boot: [version]
Java: [version]
Files scanned: [N]

---

## Summary

| Principle | Violations | HIGH | MEDIUM | LOW |
|-----------|------------|------|--------|-----|
| SRP       | N          | N    | N      | N   |
| OCP       | N          | N    | N      | N   |
| LSP       | N          | N    | N      | N   |
| ISP       | N          | N    | N      | N   |
| DIP       | N          | N    | N      | N   |

---

## HIGH Priority Violations

### [PRINCIPLE] — [com.example.package.ClassName.java:line]
**What**: [description of the violation]
**Why it matters**: [consequence if not fixed]
**Suggested direction**: [hint, not full solution]
**Spring pattern to apply**: [e.g., constructor injection, @ControllerAdvice, Strategy with @Qualifier]

---

## MEDIUM Priority Violations
...

## LOW Priority Violations
...

---

## Refactor Order (recommended)

1. [violation] — [reason it should go first]
2. ...
```

---

## Spring Boot-Specific Scan Hints

When scanning a Spring Boot project, pay special attention to:

| Pattern | Where to look | Common violation |
|---------|---------------|-----------------|
| `@Autowired` on fields | All `@Service`, `@Component` classes | DIP — switch to constructor injection |
| Business logic in `@RestController` | Controller classes | SRP — extract to service |
| `@Transactional` on controller | Controller classes | Wrong layer — move to service |
| Entity returned from controller | Controller return types | SRP — use DTOs |
| `new ArrayList<>()` / `new HashMap<>()` in service for manual mapping | Service classes | SRP — extract mapper class |
| Single `application.properties` | Config | Consider profiles for env separation |
| No `@ControllerAdvice` | Exception handling | SRP — centralize error handling |
| Repository with custom business logic | Repository classes | SRP — business rules go in service |
