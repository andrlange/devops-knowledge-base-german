# Language/Sprache : [EN](SPRINGBOOT_KOTLIN_INTRO_EN.md) | [DE](SPRINGBOOT_KOTLIN_INTRO.md)

# 🌱 Quick Intro to Spring Boot with Kotlin

This documentation describes **Spring Boot 3.5+** in combination with **Kotlin 2.x+** to develop modern, maintainable, and
production-ready backend services. Kotlin offers a precise and expressive syntax that harmonizes particularly well with Spring.

---

## 🚀 Why Kotlin?

- **Expressive & concise:** Less boilerplate than Java.
- **Null safety** through the type system (e.g., `String?` vs. `String`).
- **Modern features:** Coroutines, Data Classes, Sealed Classes.
- **Similar syntax to Dart**, which simplifies collaboration with Flutter developers.

---

## 🧰 What is Spring & Spring Boot?

- **Spring Framework** is a widely used Java framework for developing robust applications.
- **Spring Boot** is an extension that drastically simplifies configuration. It brings embedded server setup, auto-configuration, and production readiness "out of the box".

---

## 🧪 Kotlin Mini Examples

### Simple REST Controller

```kotlin
@RestController
@RequestMapping("/api/hello")
class HelloController {

    @GetMapping
    fun sayHello(): String = "Hello from Spring Boot with Kotlin!"
}
```

### Data Class and Service
```kotlin
data class User(val id: Long, val name: String)

@Service
class UserService {
    fun getUser(): User = User(1, "Max")
}
```

## ✅ Best Practices
- Use Kotlin DSLs (e.g., for application.yml via application.kts when needed).
- Rely on constructor injection instead of field injection.
- Use record classes with data class for DTOs.
- Use virtual threads or Kotlin coroutines strategically (e.g., for IO-intensive operations).
- Strictly separate controller, service, and repository.
- Enable strict null-checks.
- [For more details, read the NULL-HANDLING chapter.](NULLHANDLING_KOTLIN_EN.md)


## 📚 Advanced Topics
- Spring Security with Kotlin
- WebFlux and Coroutines
- OpenAPI/Swagger with springdoc-openapi
- Persistence with JPA & Spring Data
- Integration with Flutter via REST/GraphQL/gRPC APIs

## Back to Content:
[Back to Starting Point](../README_EN.md)