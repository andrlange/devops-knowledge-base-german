# Language/Sprache : [EN](NULLHANDLING_KOTLIN_EN.md) | [DE](NULLHANDLING_KOTLIN.md)

# Null-Handling in Kotlin: Professioneller Umgang mit Nullable-Typen in Spring Boot

## 📌 Einführung

Der Umgang mit `null` ist eine der häufigsten Ursachen für Fehler in modernen Anwendungen. Obwohl Kotlin bereits viele Schutzmechanismen gegen `NullPointerException` (NPE) bietet, liegt die Verantwortung für sauberes, durchdachtes Null-Handling letztlich bei den Entwicklern.

In diesem Guide lernst du, wie du `null` nicht einfach „zulässt“, sondern es **bewusst einsetzt oder vermeidest** – 
wie es erfahrene Entwickler tun. Wir orientieren uns dabei an Best Practices, insbesondere im Kontext von **Spring Boot 3.5+** mit Kotlin.

---

## ⚠️ Problemstellung: "Null schreiben ist Junior-Level"

Im verlinkten Artikel ["Writing null? That’s what junior devs do"](https://blog.stackademic.com/writing-null-thats-what-junior-devs-do-here-s-the-senior-way-6c2a6c08cf18) wird deutlich: Wer achtlos `null` verwendet oder zulässt, produziert nicht nur fehleranfälligen Code, sondern überträgt die Komplexität auf andere.

Typische Probleme:
- Undefinierte Zustände durch `null`
- Unnötige Nullable-Typen (`String?`, `Int?`) ohne echten Nutzen
- Unsichere APIs, die Konsumenten zwingen, mit `null` umzugehen
- Verwendung von `null` als Kontrollfluss oder Rückgabewert statt klarer Alternativen

Ein erfahrener Entwickler fragt sich:
> Muss das wirklich nullable sein? Oder gibt es eine bessere Lösung?

---

## ✅ Best Practices in Spring Boot mit Kotlin

### 1. **Vermeide unnötige Nullable-Typen**

```kotlin
// ❌ Schlecht
data class UserDto(
    val name: String?,
    val email: String?
)

// ✅ Besser
data class UserDto(
    val name: String,
    val email: String
)
```
Nur wenn es eine echte Geschäftslogik dafür gibt, sollte ein Feld null sein dürfen – z. B. optionale Felder im Request oder partielle Updates.

### 2. Verwende Optional/Result-Wrapper für Rückgaben
#### Don't: Gib null zurück.
```kotlin
// ❌
fun findUserById(id: Long): User? = userRepository.findById(id).orElse(null)
```
#### Do: Result Pattern (empfohlen für bessere Fehlerbehandlung)
```kotlin
// ✅
fun findUserById(id: Long): Result<User> =
    userRepository.findById(id)
        .map { Result.success(it) }
        .orElse(Result.failure(UserNotFoundException("User with id $id not found")))
```
#### Do: Custom sealed class (für komplexere Domain Logic)
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

#### Verwendung:
```kotlin
// Mit nullable
val user = findUserById(1L)
user?.let { println("Found: ${it.name}") } ?: println("Not found")

// Mit Result
findUserById(1L)
    .onSuccess { println("Found: ${it.name}") }
    .onFailure { println("Error: ${it.message}") }
```

> Empfehlung: Für einfache Fälle nutze nullable Types (User?), für komplexere Fehlerbehandlung das Result Pattern. Optional solltest du in Kotlin vermeiden, da es nicht idiomatisch ist.

### 3. Klar definierte Nullbarkeit in DTOs und Validierung
Bei Spring Boot mit Kotlin und @RequestBody-DTOs gilt:
- Nullable nur, wenn es wirklich optional ist
- Validierung nicht vergessen

```kotlin
data class CreateUserRequest(
    @field:NotBlank val name: String,
    @field:Email val email: String,
    val phone: String? // Optional
)
```

### 4. Fallbacks nicht mit Elvis-Missbrauch verstecken
```kotlin
// ❌
val name = user.name ?: "Unbekannt"
```

Klingt harmlos, aber oft deutet das auf ein tieferes Problem hin:
- Warum ist name nullable?
- Sollte das Backend nicht sicherstellen, dass name nie null ist?

**Besser:** Mache das Feld non-nullable und setze Defaultwerte bei der Konstruktion.

### 5. Nutze Contracts und requireNotNull für Kontrollfluss
```kotlin
fun processUser(user: User?) {
    requireNotNull(user) { "User darf nicht null sein" }
    // Jetzt garantiert non-null
    println(user.name)
}
```

Oder mit Contracts für smart casting:

```kotlin
fun User?.isValid(): Boolean {
    contract {
        returns(true) implies (this@isValid != null)
    }
    return this != null && this.name.isNotBlank()
}
```

## 🧠 Dinge die Berücksicht werden müssen
Folgende Dinge solltest du dir merken:

>Virtuelle Threads sind mächtig – aber du musst lernen, nebenläufiges Programmieren komplett neu zu denken.

Folgende Checkliste solltest du beachten:

### ✅ Concurrency-Checkliste
- Blockiere niemals innerhalb eines Locks oder synchronized-Blocks.
Verschiebe alle I/O-Operationen konsequent nach außen. Wenn das nicht möglich ist, verwende nicht-blockierende Muster oder überdenke das Design.


- Bevorzuge lockfreie Datenstrukturen oder zumindest fein granulare Locks.
Schau dir alte Collections-Verwendungen oder synchronisierte Methoden in Legacy-Code nochmals kritisch an.


- Mach dich mit den neuen Debugging-Tools vertraut.
Ein einfaches jstack reicht nicht mehr aus, virtuelle Threads werden dort nicht zuverlässig dargestellt. Nutze jcmd, 
  moderne Thread-Dumps und Heap-Analyzer mit Virtual-Thread-Unterstützung.


- Last, teste mit realitätsnaher Nebenläufigkeit.
Was lokal bei 10 Anfragen funktioniert, bricht bei echter Last vielleicht komplett zusammen. Teste unter realen Bedingungen.


- Überwache Ressourcen auf Betriebssystem- und JVM-Ebene.
Achte auf ausgelastete Threadpools, offene Sockets und blockierte Ressourcen – das sind oft die ersten Warnzeichen.


- Vertrau keinen „grünen“ Dashboards blind.
Nur weil die JVM läuft, heißt das nicht, dass deine App auch wirklich arbeitet. Messe echte Durchsatzraten und Antwortzeiten.



## 🧾 Zusammenfassung

| ✅ Tu es so                                 | ❌ Vermeide                                 |
|--------------------------------------------| ------------------------------------------ |
| Verwende non-nullable Typen, wo möglich    | Unüberlegtes `String?`, `Int?`             |
| Nutze `sealed classes` oder `Result Pattern` | Rückgabe von `null`                        |
| Validiere DTOs explizit                    | Alles nullable machen „zur Sicherheit“     |
| Verwende `requireNotNull`, Contracts       | Fallbacks mit `?:` ohne Ursache zu beheben |


## 📚 Weitere Infos & Empfehlungen

Kotlin Null Safety: https://kotlinlang.org/docs/null-safety.html

Effective Kotlin: https://leanpub.com/effectivekotlin

Artikel auf Stackademic: [Writing null? That’s what junior devs do](https://blog.stackademic.com/writing-null-thats-what-junior-devs-do-here-s-the-senior-way-6c2a6c08cf18)

JetBrains Guide: [Idiomatic Kotlin](https://kotlinlang.org/docs/idioms.html)

## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)
