# Testing Strategy — Java Spring Boot

> Load this file during Phase 2–3.
> This guide covers testing with JUnit 5 + Mockito, Spring Boot Test slices,
> and integration testing strategy.

---

## Test Pyramid

```
        ╱ ╲
       ╱ E2E ╲           Few — slow, expensive, fragile
      ╱───────╲
     ╱ Integr. ╲         Some — test layer interactions
    ╱───────────╲
   ╱    Unit     ╲       Many — fast, isolated, cheap
  ╱───────────────╲
```

**Priority for Spring Boot projects:**
1. **Unit tests first** — service layer (business logic)
2. **Repository tests** — verify custom queries
3. **Controller tests** — verify HTTP contract
4. **Integration tests** — full slice with real beans

---

## Test Dependencies (Maven)

These should already be included via `spring-boot-starter-test`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

This includes: JUnit 5, Mockito, AssertJ, Spring Test, MockMvc.

---

## What to Test First

Start with the **service class that has the most business logic**.

| Layer | What to test | How |
|-------|-------------|-----|
| Service | Business rules, edge cases, error paths | Pure JUnit 5 + Mockito |
| Repository | Custom `@Query` methods, dynamic queries | `@DataJpaTest` |
| Controller | Request mapping, validation, response shape | `@WebMvcTest` |
| Full stack | End-to-end feature flow | `@SpringBootTest` |

---

## Unit Testing Services (JUnit 5 + Mockito)

**This is the most important test type.** Service tests should be fast, isolated,
and test business logic without Spring context.

```java
// Service under test
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final OrderMapper orderMapper;

    public OrderService(OrderRepository orderRepository, OrderMapper orderMapper) {
        this.orderRepository = orderRepository;
        this.orderMapper = orderMapper;
    }

    public OrderResponse createOrder(CreateOrderRequest request) {
        if (request.getTotal() <= 0) {
            throw new InvalidOrderException("Total must be positive");
        }
        Order order = orderMapper.toEntity(request);
        Order saved = orderRepository.save(order);
        return orderMapper.toResponse(saved);
    }
}
```

```java
// Test — NO Spring context needed
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private OrderMapper orderMapper;

    @InjectMocks
    private OrderService orderService;

    @Test
    void createOrder_validRequest_returnsOrderResponse() {
        // Arrange
        CreateOrderRequest request = new CreateOrderRequest(100.0);
        Order entity = new Order(1L, 100.0);
        OrderResponse expected = new OrderResponse(1L, 100.0);

        when(orderMapper.toEntity(request)).thenReturn(entity);
        when(orderRepository.save(entity)).thenReturn(entity);
        when(orderMapper.toResponse(entity)).thenReturn(expected);

        // Act
        OrderResponse result = orderService.createOrder(request);

        // Assert
        assertThat(result).isEqualTo(expected);
        verify(orderRepository).save(entity);
    }

    @Test
    void createOrder_negativeTotal_throwsException() {
        // Arrange
        CreateOrderRequest request = new CreateOrderRequest(-50.0);

        // Act & Assert
        assertThatThrownBy(() -> orderService.createOrder(request))
                .isInstanceOf(InvalidOrderException.class)
                .hasMessageContaining("positive");

        verify(orderRepository, never()).save(any());
    }
}
```

**Key points to teach:**
- `@ExtendWith(MockitoExtension.class)` — NOT `@SpringBootTest` for unit tests
- `@Mock` creates a mock; `@InjectMocks` injects all mocks into the class
- Use `when().thenReturn()` for stubbing
- Use `verify()` to check interactions
- Use AssertJ assertions (`assertThat`) — more readable than JUnit's `assertEquals`

---

## Repository Testing (`@DataJpaTest`)

Only test **custom queries** — Spring Data's built-in methods (`findById`, `save`)
don't need testing.

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByStatus_returnsMatchingOrders() {
        // Arrange
        Order order1 = new Order(null, 100.0, OrderStatus.PENDING);
        Order order2 = new Order(null, 200.0, OrderStatus.COMPLETED);
        entityManager.persistAndFlush(order1);
        entityManager.persistAndFlush(order2);

        // Act
        List<Order> pending = orderRepository.findByStatus(OrderStatus.PENDING);

        // Assert
        assertThat(pending).hasSize(1);
        assertThat(pending.get(0).getTotal()).isEqualTo(100.0);
    }
}
```

**Key points:**
- `@DataJpaTest` boots only the JPA slice — fast, uses in-memory H2 by default
- Use `TestEntityManager` to set up test data
- DO NOT test `findById`, `findAll`, `save`, `delete` — those are JPA guarantees

---

## Controller Testing (`@WebMvcTest`)

Test the HTTP layer in isolation — mock the service.

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderService orderService;

    @Test
    void createOrder_validRequest_returns201() throws Exception {
        // Arrange
        OrderResponse response = new OrderResponse(1L, 100.0);
        when(orderService.createOrder(any())).thenReturn(response);

        // Act & Assert
        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                            {"total": 100.0}
                            """))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.total").value(100.0));
    }

    @Test
    void createOrder_invalidRequest_returns400() throws Exception {
        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                            {"total": -50.0}
                            """))
                .andExpect(status().isBadRequest());
    }
}
```

**Key points:**
- `@WebMvcTest` boots only the web layer — no service, no repository, no DB
- `@MockBean` replaces the real bean with a mock in Spring context
- Use `MockMvc` to simulate HTTP requests
- Test: status codes, response body shape, validation errors, headers

---

## Integration Testing (`@SpringBootTest`)

Use sparingly — these are slow. Test critical paths end-to-end.

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional  // rolls back after each test
class OrderIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void createAndRetrieveOrder() throws Exception {
        // Create
        String response = mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                            {"total": 100.0}
                            """))
                .andExpect(status().isCreated())
                .andReturn()
                .getResponse()
                .getContentAsString();

        Long id = JsonPath.parse(response).read("$.id", Long.class);

        // Retrieve
        mockMvc.perform(get("/api/orders/" + id))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.total").value(100.0));
    }
}
```

---

## Test Naming Convention

Use descriptive names: `methodName_condition_expectedBehavior`

```java
void createOrder_validRequest_returnsOrderResponse()
void createOrder_negativeTotal_throwsInvalidOrderException()
void findByStatus_noMatches_returnsEmptyList()
void deleteOrder_nonExistentId_throwsNotFoundException()
```

---

## AAA Pattern (Arrange-Act-Assert)

Every test follows three sections:

```java
@Test
void methodName_condition_expectedBehavior() {
    // Arrange — set up test data and mocks

    // Act — call the method under test

    // Assert — verify the result
}
```

Do not mix these sections. One "Act" per test.

---

## Common Mistakes

| Mistake | Why it's wrong | Fix |
|---------|----------------|-----|
| `@SpringBootTest` for every test | Slow, loads entire context | Use `@ExtendWith(MockitoExtension.class)` for unit tests |
| Testing getters/setters | No business value | Test behavior, not structure |
| No error path tests | Misses edge cases | Always test invalid input, null, empty |
| Testing Spring Data methods | Already tested by Spring | Only test custom `@Query` methods |
| `@Autowired` in unit tests | Makes them integration tests | Use `@Mock` + `@InjectMocks` |
| No assertions | Test always passes | Every test must assert something |

---

## Coverage Targets

| Layer | Target | Notes |
|-------|--------|-------|
| Service | 80%+ | Most critical — all business logic |
| Repository | Custom queries only | ~50% is fine |
| Controller | 70%+ | All endpoints, error cases |
| Entity/DTO | Skip | No behavior to test |
| Config | Skip | Unless complex |

**How to measure:**
```bash
mvn test jacoco:report
# Report at: target/site/jacoco/index.html
```

Add JaCoCo plugin to `pom.xml`:
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
