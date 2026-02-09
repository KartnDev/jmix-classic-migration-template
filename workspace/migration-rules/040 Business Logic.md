# Business Logic Migration Rules (Jmix 1 -> Jmix 2)

## Overview

Business logic in Jmix 1.x and Jmix 2.x is **nearly identical**. Both use Spring Framework, DataManager, and the same service patterns. The main changes are related to Jakarta EE namespace migration.

## Services

Services remain the same. No changes needed:

```java
// Same in both Jmix 1.x and 2.x
@Component
public class CustomerService {

    @Autowired
    private DataManager dataManager;

    public List<Customer> findActiveCustomers() {
        return dataManager.load(Customer.class)
                .query("select c from Customer c where c.active = true")
                .list();
    }
}
```

## DataManager Usage

DataManager API is identical:

```java
// Same in both Jmix 1.x and 2.x
@Autowired
private DataManager dataManager;

// Load list
List<Customer> customers = dataManager.load(Customer.class)
        .query("select c from Customer c")
        .fetchPlan("customer-fetchPlan")
        .list();

// Load single
Customer customer = dataManager.load(Customer.class)
        .id(customerId)
        .fetchPlan("customer-detail")
        .one();

// Save
dataManager.save(customer);

// Remove
dataManager.remove(customer);

// Create new
Customer newCustomer = dataManager.create(Customer.class);
```

## Transactional Operations

Transaction management is identical:

```java
// Same in both Jmix 1.x and 2.x
@Autowired
private TransactionTemplate transactionTemplate;

public void processOrders(List<Order> orders) {
    transactionTemplate.executeWithoutResult(status -> {
        for (Order order : orders) {
            order.setStatus(OrderStatus.PROCESSED);
            dataManager.save(order);
        }
    });
}
```

## Entity Listeners

Entity listeners work the same way:

```java
// Same in both Jmix 1.x and 2.x
@Component
public class CustomerEntityListener {

    @Autowired
    private TimeSource timeSource;

    @EventListener
    public void onCustomerChanging(EntityChangedEvent<Customer> event) {
        if (event.getType() == EntityChangedEvent.Type.CREATED) {
            // Handle creation
        }
    }
}
```

## REST Client

REST client configuration is identical:

```java
// Same in both Jmix 1.x and 2.x
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// Usage
@Autowired
private RestTemplate restTemplate;

public Account getAccountByCode(String code) {
    String url = UriComponentsBuilder.fromHttpUrl(serviceUrl + "/v2/accounts/")
            .queryParam("code", code)
            .toUriString();

    AccountResponse response = restTemplate.getForObject(url, AccountResponse.class);
    return response != null ? response.getAccount() : null;
}
```

## Configuration Properties

Configuration properties work the same:

```java
// Simple property
@Value("${main.url}")
private String mainUrl;

// Structured properties
@Component
@ConfigurationProperties("myapp")
public class ApplicationProperties {
    private String apiUrl;
    private int timeout;

    // getters and setters
}
```

```properties
# application.properties
myapp.api-url=https://api.example.com
myapp.timeout=30000
```

## Import Changes (Main Difference)

The only significant change is the namespace migration:

```java
// Jmix 1.x
import javax.persistence.EntityManager;
import javax.transaction.Transactional;
```

```java
// Jmix 2.x
import jakarta.persistence.EntityManager;
import jakarta.transaction.Transactional;
```

## Migration Checklist

| Aspect           | Changes Required    |
|------------------|---------------------|
| Services         | No changes          |
| DataManager      | No changes          |
| Transactions     | No changes          |
| Entity listeners | No changes          |
| REST clients     | No changes          |
| Configuration    | No changes          |
| javax.* imports  | Change to jakarta.* |
