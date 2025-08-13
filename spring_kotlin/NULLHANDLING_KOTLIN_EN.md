# Sprache/Language : [DE](NULLHANDLING_KOTLIN.md) | [EN](NULLHANDLING_KOTLIN_EN.md)

# Null Handling in Kotlin: Professional Approach to Nullable Types in Spring Boot

## 📌 Introduction

Dealing with `null` is one of the most common causes of errors in modern applications. Although Kotlin already provides many protective mechanisms against `NullPointerException` (NPE), the responsibility for clean, thoughtful null handling ultimately lies with the developers.

In this guide, you'll learn how to not simply "allow" `null`, but **consciously use or avoid it** –
as experienced developers do. We'll follow best practices, particularly in the context of **Spring Boot 3.5+** with Kotlin.

---

## ⚠️ Problem Statement: "Writing null is junior level"

The linked article ["Writing null? That's what junior devs do"](https://blog.stackademic.com/writing-null-thats-what-junior-devs-do-here-s-the-senior-way-6c2a6c08cf18) makes it clear: Anyone who carelessly uses or allows `null` not only produces error-prone code but also transfers complexity to others.

Typical problems:
- Undefined states through `null`
- Unnecessary nullable types (`String?`, `Int?`) without real benefit
- Unsafe APIs that force consumers to deal with `null`
- Using `null` as control flow or return value instead of clear alternatives

An experienced developer asks:
> Does this really need to be nullable? Or is there a better solution?

---

## ✅ Best Practices in Spring Boot with Kotlin

### 1. **Avoid unnecessary nullable types**

```kotlin
// ❌ Bad
data class UserDto(
    val name: String?,
    val email: String?
)

// ✅ Better
data class UserDto(
    val name: String,
    val email: String
)
```
Only if there's real business logic for it should a field be allowed to be null – e.g., optional fields in requests or partial updates.

### 2. Use Optional/Result wrappers for returns
#### Don't: Return null.
```kotlin
// ❌
fun findUserById(id: Long): User? = userRepository.findById(id).orElse(null)
```
#### Do: Result Pattern (recommended for better error handling)
```kotlin
// ✅
fun findUserById(id: Long): Result<User> =
    userRepository.findById(id)
        .map { Result.success(it) }
        .orElse(Result.failure(UserNotFoundException("User with id $id not found")))
```
#### Do: Custom sealed class (for more complex domain logic)
```kotlin
// ✅
sealed class UserResult {
    data class Found(val user: User) : UserResult()
    data object NotFound : UserResult()
    data class Error(val exception: Throwable) : UserResult()
}

fun findUserById(id: Long): UserResult = try {
    userRepository.findById(id)
        .map { UserResult.Found(it) }
        .orElse(UserResult.NotFound)
} catch (e: Exception) {
    UserResult.Error(e)
}
```

#### Usage:
```kotlin
// With nullable
val user = findUserById(1L)
user?.let { println("Found: ${it.name}") } ?: println("Not found")

// With Result
findUserById(1L)
    .onSuccess { println("Found: ${it.name}") }
    .onFailure { println("Error: ${it.message}") }
```

> Recommendation: For simple cases, use nullable types (User?), for more complex error handling use the Result pattern. You should avoid Optional in Kotlin as it's not idiomatic.

### 3. Clearly defined nullability in DTOs and validation
With Spring Boot with Kotlin and @RequestBody DTOs:
- Nullable only if it's really optional
- Don't forget validation

```kotlin
data class CreateUserRequest(
    @field:NotBlank val name: String,
    @field:Email val email: String,
    val phone: String? // Optional
)
```

### 4. Don't hide fallbacks with Elvis abuse
```kotlin
// ❌
val name = user.name ?: "Unknown"
```

Sounds harmless, but often indicates a deeper problem:
- Why is name nullable?
- Shouldn't the backend ensure that name is never null?

**Better:** Make the field non-nullable and set default values during construction.

### 5. Use contracts and requireNotNull for control flow
```kotlin
fun processUser(user: User?) {
    requireNotNull(user) { "User must not be null" }
    // Now guaranteed non-null
    println(user.name)
}
```

Or with contracts for smart casting:

```kotlin
fun User?.isValid(): Boolean {
    contract {
        returns(true) implies (this@isValid != null)
    }
    return this != null && this.name.isNotBlank()
}
```

## 🧠 Things to Consider

When dealing with null handling in Kotlin, keep these important considerations in mind:

### ✅ Null Safety Checklist
- **Question nullability early in design**  
  Ask yourself: "Is this field truly optional from a business perspective?" Don't make things nullable just because it seems easier.

- **Prefer immutable data classes**  
  Immutable objects with non-nullable fields are safer and more predictable than mutable objects with nullable fields.

- **Use validation at boundaries**  
  Validate inputs at API boundaries (controllers, DTOs) to prevent null values from entering your system.

- **Leverage Kotlin's null safety features**  
  Use safe calls (`?.`), Elvis operator (`?:`), and `let` blocks appropriately, but don't overuse them as band-aids.

- **Consider the consumer perspective**  
  When designing APIs, think about how consumers will handle your nullable returns. Make their lives easier, not harder.

- **Document nullability intentions**  
  When nullability is intentional, document why and what null represents in your domain model.

## 🧾 Summary

| ✅ Do this                                 | ❌ Avoid                                   |
|--------------------------------------------| ------------------------------------------ |
| Use non-nullable types where possible     | Thoughtless `String?`, `Int?`              |
| Use `sealed classes` or `Result Pattern`  | Returning `null`                           |
| Validate DTOs explicitly                  | Making everything nullable "for safety"    |
| Use `requireNotNull`, contracts           | Fallbacks with `?:` without fixing cause  |


## 📚 Further Information & Recommendations

Kotlin Null Safety: https://kotlinlang.org/docs/null-safety.html

Effective Kotlin: https://leanpub.com/effectivekotlin

Article on Stackademic: [Writing null? That's what junior devs do](https://blog.stackademic.com/writing-null-thats-what-junior-devs-do-here-s-the-senior-way-6c2a6c08cf18)

JetBrains Guide: [Idiomatic Kotlin](https://kotlinlang.org/docs/idioms.html)

## Back to Content:
[Back to Starting Point](../README_EN.md)
