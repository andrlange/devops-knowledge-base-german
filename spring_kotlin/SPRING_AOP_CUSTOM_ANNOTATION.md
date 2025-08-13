# ⏱️ Laufzeitmessung von Funktionen mit Spring AOP und SLF4J (Kotlin)

## 🧠 Was ist Spring AOP?

**AOP (Aspect-Oriented Programming)** ermöglicht es, Querschnittsfunktionen (Cross-Cutting-Concerns) wie Logging, 
Performance-Messung, Security oder Transaktionen **modular** und **transparent** zu integrieren – ohne die Business-Logik zu verändern.

### 💡 Typische Use-Cases:
- Logging von Methodeneinstieg/-austritt
- Laufzeitmessung von Funktionen
- Automatisches Exception-Logging
- Security-Prüfungen (z. B. Rollencheck)
- Transaktionshandling (`@Transactional` nutzt AOP!)

Spring AOP basiert auf **Proxies** (JDK oder CGLIB) und funktioniert nur bei **Spring Beans**.

---

## 🏷️ Custom Annotation für Laufzeitmessung

Erstelle eine eigene Annotation, z. B. `@LogExecutionTime`, um gezielt Methoden zu markieren:

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class LogExecutionTime
```

## ⚙️ AOP-Aspect: Laufzeit messen & loggen

```kotlin
@Aspect
@Component
class LoggingAspect {

    private val logger = LoggerFactory.getLogger(this::class.java)

    @Around("@annotation(LogExecutionTime)")
    fun logExecutionTime(joinPoint: ProceedingJoinPoint): Any? {
        val start = System.currentTimeMillis()
        var result : Any?

        try {
            result = joinPoint.proceed()
            val duration = System.currentTimeMillis() - start
            logger.info("Method ${joinPoint.signature.toShortString()} executed in ${duration}ms")
        } catch(e : Exception) {
            val duration = System.currentTimeMillis() - start
            logger.info("Method ${joinPoint.signature.toShortString()} has thrown Exception after ${duration}ms")
            throw e
        }
        return result

    }
}
```
> 📌 @Around erlaubt das Umschließen einer Methode (Before + After + Exception)

## 🚀 Beispielverwendung in einer Serviceklasse
```kotlin@Service
class ReportService {

    @LogExecutionTime
    fun generateReport(): String {
        Thread.sleep(500) // Simuliert aufwendige Berechnung
        return "Report generated"
    }
}
```

Beim Aufruf von generateReport() erscheint in den Logs:
```txt
INFO  Method ReportService.generateReport() executed in 502ms
```

## 🧠 Zusammenfassung & Best Practices

| Best Practice                      | Empfehlung                                                         |
| ---------------------------------- | ------------------------------------------------------------------ |
| ✅ Querschnittslogik modularisieren | Kein Logging im Service/Controller verstreuen                      |
| ✅ Zielgerichteter Einsatz          | Nur dort markieren, wo Messung sinnvoll ist                        |
| ⚠️ Nur auf Spring Beans anwendbar  | `@Component`, `@Service` etc. notwendig                            |
| ✅ Kurze Laufzeiten vermeiden       | Logging overhead beachten – nicht bei sehr kleinen Methoden nutzen |
| ✅ Logger korrekt einsetzen         | SLF4J mit `LoggerFactory.getLogger()` verwenden                    |


## 📘 Weiterführende Themen

- 📦 @Around, @Before, @After, @AfterThrowing – verschiedene AOP Advice-Arten
- 🧵 ThreadLocal für Tracing / Request Context
- 📈 Micrometer für Metriken + Prometheus/Grafana Integration
- 🔒 AOP für Security (z. B. Rollenprüfung via @Secured)
- 🐞 Exception Logging via @AfterThrowing



## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)


