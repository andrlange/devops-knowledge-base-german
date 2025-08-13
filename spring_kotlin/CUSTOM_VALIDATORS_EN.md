# Sprache/Language : [DE](CUSTOM_VALIDATORS.md) | [EN](CUSTOM_VALIDATORS_EN.md)

# ✅ Custom Validator as Annotation for DTOs (Spring Boot + Kotlin)

## 🔍 Why Custom Validators?

Standard validators like `@NotBlank`, `@Size`, etc. are powerful – but sometimes not flexible enough.

Example: You want to validate a list of strings where:
- Empty strings can be allowed or forbidden
- A minimum or maximum length applies to individual strings
- The list itself must not be empty

➡️ Solution: Write your own **annotation-based validator**.

---

## 🧱 Custom Annotation: `@StringListValidator`

### Define Annotation

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

### 🧠 The Corresponding Validator
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
        if (value == null) return true // Optional field → handled by @NotNull if needed

        return value.all { str ->
            if (!allowBlank && str.isBlank()) return false
            str.length in minLength..maxLength
        }
    }
}
```

### 🚀 Usage in a DTO
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

#### Example error message for invalid input:
```json
{
  "tags": [
    "Each tag must be 2-20 characters and not blank"
  ]
}
```

## ⚙️ Dependency: Spring Boot Validator (JSR-380 / Jakarta Validation)

For custom validators like `@StringListValidator` to work, the **Bean Validation API** (JSR-380 / Jakarta Validation) must be included in the project.

### 📦 Gradle (Kotlin DSL)

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-validation")
}

```

### 📦 Maven
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```


>⚠️ This dependency brings the necessary infrastructure like jakarta.validation and
> automatic validation via @Valid.


>💡 Note: From Spring Boot 3.x onwards, jakarta.validation is used by default, no longer javax.validation. When using
> custom validators, make sure to use the correct imports:

```kotlin
import jakarta.validation.Constraint
import jakarta.validation.ConstraintValidator
import jakarta.validation.ConstraintValidatorContext
import jakarta.validation.Payload
```



## 📌 Summary & Best Practices

| Best Practice                          | Recommendation                                                          |
| -------------------------------------- | ----------------------------------------------------------------------- |
| ✅ Validation belongs in the DTO       | This keeps the controller clean and clear                              |
| ✅ Reusable validators                 | Use custom annotations for recurring rules                             |
| ✅ Fine-grained logic                  | Validators should **only validate**, not contain business logic        |
| ❌ Don't throw exceptions in validator | Always return `false` + set error message                              |
| ✅ Combine with other validators       | e.g., use `@NotEmpty`, `@Size` as supplements                          |


## 🔄 Possible Extensions

- Regex matching (Pattern.matches(...))
- Whitelist/blacklist validation of values
- Validator for Map<K, V> or nested objects
- Internationalization of error messages via messages.properties


## Back to Content:
[Back to Starting Point](../README_EN.md)