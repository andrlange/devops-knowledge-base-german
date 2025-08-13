# Language/Sprache : [EN](CONTROLLER_ADVICE_EN.md) | [DE](CONTROLLER_ADVICE.md)

# 🎯 Controller Advice für Rest-Controller in Spring Boot (Kotlin)

## ✅ Warum `@ControllerAdvice`?

In REST-APIs ist eine konsistente Fehlerbehandlung entscheidend:

- Vermeidet redundante `try-catch`-Blöcke in jedem Controller
- Trennt Fehlerbehandlung von Geschäftslogik → sauberer Code
- Einheitliches Fehler-Response-Format für Clients

Mit `@ControllerAdvice` lassen sich **globale** oder **controller-spezifische** Ausnahmen zentral abfangen.

---

## 🌍 Globale Exception-Behandlung mit `@ControllerAdvice`

Globale Fehlerbehandlung betrifft **alle** Controller in der Anwendung.

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
- 📌 Wird automatisch auf alle @RestController-Instanzen angewendet.


## 🎯 Lokale Exception-Behandlung in einem dedizierten Controller
Falls du eine Ausnahme nur in einem bestimmten Controller behandeln möchtest:
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

## 🧠 Zusammenfassung & Best Practices

| Best Practice                             | Empfehlung                                                                                        |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ✅ Trennung von Fehlerbehandlung und Logik | Nutze `@ControllerAdvice` für saubere Architekturen                                               |
| ✅ Einheitliche Fehlerstruktur             | Verwende `ApiError`-Dataklassen für lesbare Fehler                                                |
| ✅ Logging nicht vergessen                 | Protokolliere Fehler sinnvoll für spätere Analyse                                                 |
| ✅ Keine sensiblen Infos preisgeben        | Gib keine Stacktraces oder internen Details im Response zurück                                    |
| ✅ Lokale Handler nur bei Bedarf           | Verwende lokale `@ExceptionHandler`, wenn ein Fehler **nur lokal** sinnvoll behandelt werden kann |

## 📘 Weiterführende Themen
- Custom Exceptions mit eigenen HTTP-Codes
- Validierung mit @Valid und Bean Validation (jakarta.validation)
- Fehlerbehandlung in asynchronen Endpunkten



## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)
