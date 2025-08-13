# Sprache/Language : [DE](CONTROLLER_ADVICE.md) | [EN](CONTROLLER_ADVICE_EN.md)

# 🎯 Controller Advice for REST Controllers in Spring Boot (Kotlin)

## ✅ Why `@ControllerAdvice`?

In REST APIs, consistent error handling is crucial:

- Avoids redundant `try-catch` blocks in every controller
- Separates error handling from business logic → cleaner code
- Uniform error response format for clients

With `@ControllerAdvice`, **global** or **controller-specific** exceptions can be handled centrally.

---

## 🌍 Global Exception Handling with `@ControllerAdvice`

Global error handling affects **all** controllers in the application.

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException::class)
    fun handleNotFound(ex: EntityNotFoundException): ResponseEntity<ApiError> {
        val error = ApiError("NOT_FOUND", ex.message ?: "Entity not found")
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error)
    }

    @ExceptionHandler(Exception::class)
    fun handleGeneral(ex: Exception): ResponseEntity<ApiError> {
        val error = ApiError("INTERNAL_ERROR", ex.localizedMessage)
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error)
    }
}

data class ApiError(
    val code: String,
    val message: String
)
```
- 📌 @RestControllerAdvice = @ControllerAdvice + @ResponseBody
- 📌 Is automatically applied to all @RestController instances.


## 🎯 Local Exception Handling in a Dedicated Controller
If you want to handle an exception only in a specific controller:
```kotlin
@RestController
@RequestMapping("/users")
class UserController {

    @GetMapping("/{id}")
    fun getUser(@PathVariable id: Long): User {
        if (id < 0) throw IllegalArgumentException("ID must be positive")
        // ...
        return User(id, "John Doe")
    }

    @ExceptionHandler(IllegalArgumentException::class)
    fun handleIllegalArgument(ex: IllegalArgumentException): ResponseEntity<ApiError> {
        val error = ApiError("BAD_REQUEST", ex.message ?: "Invalid input")
        return ResponseEntity.badRequest().body(error)
    }
}

data class User(val id: Long, val name: String)
```

## 🧠 Summary & Best Practices

| Best Practice                             | Recommendation                                                                                    |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ✅ Separation of error handling and logic | Use `@ControllerAdvice` for clean architectures                                                   |
| ✅ Uniform error structure                | Use `ApiError` data classes for readable errors                                                   |
| ✅ Don't forget logging                   | Log errors meaningfully for later analysis                                                        |
| ✅ Don't expose sensitive information     | Don't return stack traces or internal details in the response                                     |
| ✅ Local handlers only when needed        | Use local `@ExceptionHandler` when an error can **only locally** be handled meaningfully         |

## 📘 Advanced Topics
- Custom exceptions with their own HTTP codes
- Validation with @Valid and Bean Validation (jakarta.validation)
- Error handling in asynchronous endpoints



## Back to Content:
[Back to Starting Point](../README_EN.md)