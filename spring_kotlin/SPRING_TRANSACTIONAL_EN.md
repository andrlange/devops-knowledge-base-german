# Language/Sprache : [EN](SPRING_TRANSACTIONAL_EN.md) | [DE](SPRING_TRANSACTIONAL.md)

# Transactional Work with `@Transactional` in Spring Boot 3.5+ and Kotlin

## Introduction and Overview of Transactions and `@Transactional`

The `@Transactional` annotation is one of the most powerful tools in Spring Boot when it comes to consistent database operations. It ensures that multiple steps in a process are executed atomically: either all successfully or none at all.

Despite its simplicity in usage, `@Transactional` can bring some tricky pitfalls under the hood – especially when combined with Spring AOP and Kotlin. When used incorrectly, it can lead to transactions **not being active** without throwing an error. In production systems, this can result in **data inconsistency**, **resource blocking**, and **system failures**.


![Transactional](assets/transaction.svg)
---

## Effects and Application with Examples

A classic mistake is using `@Transactional` on **private methods** within the same class:

```kotlin
@Service
class UserService(
    private val userRepository: UserRepository,
    private val emailClient: EmailClient
) {
    fun register(dto: UserDto) {
        saveUser(dto) // No proxy call → No transaction!
        emailClient.sendWelcomeEmail(dto.email)
    }

    @Transactional
    private fun saveUser(dto: UserDto) {
        val user = User(dto.email, dto.name)
        userRepository.save(user)
    }
}
```

➡️ ***Problem:*** Since Spring is based on proxies, internal method calls within the same instance are not
intercepted. The saveUser() method is therefore executed without a transaction.

➡️ ***Consequence:*** No rollbacks on constraint violations, dirty reads, incomplete data states under high load.

***Correct Solution:*** Transactions in separate beans

```kotlin
@Service
class UserPersistenceService(
    private val userRepository: UserRepository
) {
    @Transactional
    fun saveUser(dto: UserDto): User {
        val user = User(dto.email, dto.name)
        return userRepository.save(user)
    }
}

@Service
class UserService(
    private val persistence: UserPersistenceService,
    private val emailClient: EmailClient
) {
    fun register(dto: UserDto) {
        val user = persistence.saveUser(dto)
        emailClient.sendWelcomeEmail(user.email)
    }
}
```
➡️ ***Advantage:*** The call to saveUser() occurs via a Spring-managed bean → Transaction is correctly
opened.

---
## Don'ts – What You Shouldn't Do
### Here are some typical sources of error:

❌ Annotating private or internal methods with @Transactional
→ Transactions only work via Spring proxies (i.e., external calls via Spring context)

❌ External I/O calls within a transaction (e.g., email sending, HTTP, RabbitMQ, etc.)
→ On timeouts or errors, the entire transaction can be rolled back (e.g., payment + inventory → rollback)

❌ Blind trust in @Transactional(propagation = REQUIRED)
→ Often leads to cross-cutting transactions, even where you don't want them (e.g., for notifications)

Example:

```kotlin
@Service
class OrderService(
    private val inventoryService: InventoryService,
    private val notificationService: NotificationService
) {
    @Transactional
    fun placeOrder(dto: OrderDto) {
        val order = createOrder(dto)
        inventoryService.reserveStock(order.items)
        notificationService.notifyCustomer(order) // Wrong place for transaction
    }
}
```
➡️ A timeout in the mail server will roll back the complete order, even though payment has already occurred.

---
### Best Practices and Summary

✅ Separation of orchestration and persistence logic
→ Database operations belong in separate, Spring-managed service beans

✅ No private methods with @Transactional
→ Only public methods in beans are correctly intercepted by Spring proxies

✅ Execute I/O outside of transactions
→ Use @Transactional(propagation = Propagation.NOT_SUPPORTED) for external calls

```kotlin
@Transactional(propagation = Propagation.NOT_SUPPORTED)
fun notifyCustomer(order: Order) {
    notificationClient.send(order.customerEmail)
}
```

### ✅ Monitoring System Metrics via Logs and Actuators
#### → Actively monitor:

```yaml
logging:
  level:
    org.springframework.transaction: DEBUG
    org.hibernate.SQL: DEBUG
    com.zaxxer.hikari: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: metrics,health
  metrics:
    export:
      prometheus:
        enabled: true
```

#### Monitor especially:

* hikari_connections_active
* spring_transactions_active
* spring_data_repository_invocations
---

## Conclusion
Spring's transaction management is powerful, but not magical. Those who ignore the underlying proxy mechanism build invisible bugs that only become apparent under load. The application of @Transactional requires conscious design of the application structure – especially with Kotlin, where default methods are final and additional caution is required.

> 🔒 A misplaced @Transactional is like an unlatched seatbelt: You only notice it when it's too late.

## TLDR Checklist

| Rule                                                                      | Description                                          |
|---------------------------------------------------------------------------| ---------------------------------------------------- |
| ✅ Annotate public methods in Spring beans with `@Transactional`          | Only this way the proxy works correctly              |
| ✅ **Never** embed external calls (e.g., email, RabbitMQ) in transactions | Use `Propagation.NOT_SUPPORTED`                      |
| ✅ Separate business logic: orchestration vs. persistence                 | Clear service layer structure                        |
| ✅ Monitor transaction and connection metrics                             | Early detection of bottlenecks                      |
| ❌ No private or internal calls to `@Transactional` methods              | Otherwise the transaction is silently ignored       |


## Back to Content:
[Back to Starting Point](../README_EN.md)