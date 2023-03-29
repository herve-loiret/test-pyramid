# Testing Strategy for Hexagonal Projects
*******

# General

## Snake case for test methods 

Test method names often describe a specific scenario or a set of conditions. 
Snake_case can make these long and descriptive names more readable and easier to understand. 
The use of underscores can clearly separate words, making the test purpose immediately clear.

## Test dependencies

### AssertJ

AssertJ is a Java library that provides fluent assertions, making it easy to write clear and expressive tests. Here are some of the benefits of using AssertJ:

1. Readability: AssertJ's fluent API provides readable assertions that are easy to understand and maintain. It allows developers to express their intentions in a clear and concise manner.
2. Flexibility: AssertJ provides a wide range of assertions and matchers that can be easily customized to suit specific use cases. This allows developers to write tests that accurately reflect the behavior of their application.
3. Error Messages: AssertJ's error messages are clear and descriptive, making it easy to identify the root cause of test failures. The error messages are designed to help developers quickly diagnose and fix issues in their code.

#### Maven Dependency 

```xml 
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <scope>test</scope>
</dependency>
```

# Domain


## Model
## Service Adapters

In this example we will write domain tests using custom stubs for simulating 
the behaviors of external dependencies.

```java 
public interface ExternalOrderService {
    Order fetchOrder(Long id);
    Order saveOrder(Order order);
}
```

```java 
public class ExternalOrderServiceStub implements ExternalOrderService {
    private final Map<Long, Order> orders = new HashMap<>();

    @Override
    public Order fetchOrder(Long id) {
        return orders.get(id);
    }

    @Override
    public Order saveOrder(Order order) {
        orders.put(order.getId(), order);
        return order;
    }
}
```

```java 
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class OrderServiceTest {
    private ExternalOrderServiceStub orderService;
    private ExternalOrderService externalOrderService;

    @BeforeEach
    public void setUp() {
        externalOrderService = new ExternalOrderServiceStub();
        orderService = new OrderService(externalOrderService);
    }

    @Test
    public void get_order_from_external_service_success() {
        Order order = new Order(1L, "Item 1", 2, 100.0);
        externalOrderService.saveOrder(order);

        Order fetchedOrder = orderService.getOrderFromExternalService(order.getId());

        Assertions.assertThat(fetchedOrder)
                .usingRecursiveComparison()
                .isEqualTo(order);
    }

    @Test
    public void save_order_to_external_service_success() {
        Order order = new Order(2L, "Item 2", 3, 200.0);

        Order savedOrder = orderService.saveOrderToExternalService(order);

        assertThat(savedOrder).isEqualTo(order);
        assertThat(externalOrderService.fetchOrder(order.getId())).isEqualTo(order);
    }
}
```

*******
# Left Adapters

## Rest module

### Controllers

``` java
@Configuration
public class TestConfig {

    @Bean
    public OrderService orderService() {
        return new OrderServiceStub();
    }
}
```

``` java
@WebMvcTest(OrderController.class)
public class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ApplicationContext applicationContext;
    
    @Autowired
    private OrderController orderController;

    @Test
    public void getAllOrders_shouldReturnOrders() throws Exception {
        mockMvc.perform(get("/orders"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$").isArray())
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].id").value(1L))
                .andExpect(jsonPath("$[0].name").value("Item 1"))
                .andExpect(jsonPath("$[0].quantity").value(2))
                .andExpect(jsonPath("$[0].price").value(100.0))
                .andExpect(jsonPath("$[1].id").value(2L))
                .andExpect(jsonPath("$[1].name").value("Item 2"))
                .andExpect(jsonPath("$[1].quantity").value(3))
                .andExpect(jsonPath("$[1].price").value(200.0));
    }
}
```

## batch module

# Right Adapters
## jpa module
## api module
## s3 module
