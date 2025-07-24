# ✅ Custom Validator als Annotation für DTOs (Spring Boot + Kotlin)

## 🔍 Warum eigene Validatoren?

Standard-Validatoren wie `@NotBlank`, `@Size` usw. sind mächtig – aber manchmal nicht flexibel genug.

Beispiel: Du möchtest eine Liste von Strings prüfen, wobei:
- Leere Strings erlaubt oder verboten sein können
- Eine Mindest- oder Höchstlänge für die einzelnen Strings gilt
- Die Liste selbst nicht leer sein darf

➡️ Lösung: **Eigener Annotation-basierten Validator** schreiben.

---

## 🧱 Eigene Annotation: `@StringListValidator`

### Annotation definieren

```kotlin
@Target(AnnotationTarget.FIELD)
@Retention(AnnotationRetention.RUNTIME)
@Constraint(validatedBy = [StringListConstraintValidator::class])
@MustBeDocumented
annotation class StringListValidator(
    val allowBlank: Boolean = true,
    val minLength: Int = 0,
    val maxLength: Int = Int.MAX_VALUE,
    val message: String = "Invalid string list entry",
    val groups: Array<KClass<*>> = [],
    val payload: Array<KClass<out Payload>> = []
)
```

### 🧠 Der passende Validator
```kotlin
class StringListConstraintValidator : ConstraintValidator<StringListValidator, List<String>?> {

    private var allowBlank = true
    private var minLength = 0
    private var maxLength = Int.MAX_VALUE

    override fun initialize(constraint: StringListValidator) {
        allowBlank = constraint.allowBlank
        minLength = constraint.minLength
        maxLength = constraint.maxLength
    }

    override fun isValid(value: List<String>?, context: ConstraintValidatorContext): Boolean {
        if (value == null) return true // Optional-Feld → handled by @NotNull if needed

        return value.all { str ->
            if (!allowBlank && str.isBlank()) return false
            str.length in minLength..maxLength
        }
    }
}
```

### 🚀 Anwendung in einem DTO
```kotlin
data class CreateUserRequest(

    @field:StringListValidator(
        allowBlank = false,
        minLength = 2,
        maxLength = 20,
        message = "Each tag must be 2-20 characters and not blank"
    )
    val tags: List<String>,

    @field:NotNull
    val name: String
)
```

#### Beispiel-Fehlermeldung bei ungültigem Input:
```json
{
  "tags": [
    "Each tag must be 2-20 characters and not blank"
  ]
}
```

## 📌 Zusammenfassung & Best Practices

| Best Practice                         | Empfehlung                                                             |
| ------------------------------------- | ---------------------------------------------------------------------- |
| ✅ Validierung gehört ins DTO          | So bleibt der Controller sauber und klar                               |
| ✅ Wiederverwendbare Validatoren       | Nutze eigene Annotationen für wiederkehrende Regeln                    |
| ✅ Kleinteilige Logik                  | Validatoren sollten **nur validieren**, keine Geschäftslogik enthalten |
| ❌ Keine Exception in Validator werfen | Immer `false` zurückgeben + Fehlermeldung setzen                       |
| ✅ Kombination mit anderen Validatoren | z. B. `@NotEmpty`, `@Size` ergänzend nutzen                            |


## 🔄 Erweiterungen möglich

- Regex-Matching (Pattern.matches(...))
- Whitelist/Blacklist-Validierung von Werten
- Validator für Map<K, V> oder verschachtelte Objekte
- Internationalisierung der Fehlermeldung über messages.properties


## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)
