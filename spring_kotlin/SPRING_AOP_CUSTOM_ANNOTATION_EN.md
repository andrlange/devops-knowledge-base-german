# Sprache/Language : [DE](SPRING_AOP_CUSTOM_ANNOTATION.md) | [EN](SPRING_AOP_CUSTOM_ANNOTATION_EN.md)

# ⏱️ Runtime Measurement of Functions with Spring AOP and SLF4J (Kotlin)

## 🧠 What is Spring AOP (quick intro)?

**AOP (Aspect-Oriented Programming)** enables the integration of cross-cutting concerns like logging,
performance measurement, security, or transactions in a **modular** and **transparent** way – without changing the business logic.

### 💡 Typical Use Cases:
- Logging method entry/exit
- Runtime measurement of functions
- Automatic exception logging
- Security checks (e.g., role verification)
- Transaction handling (`@Transactional` uses AOP!)

Spring AOP is based on **proxies** (JDK or CGLIB) and only works with **Spring Beans**.

---

## 🏷️ Custom Annotation for Runtime Measurement

Create your own annotation, e.g., `@LogExecutionTime`, to specifically mark methods:

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class LogExecutionTime
```

## ⚙️ AOP Aspect: Measure & Log Runtime

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
> 📌 @Around allows wrapping a method (Before + After + Exception)

## 🚀 Example Usage in a Service Class
```kotlin
@Service
class ReportService {

    @LogExecutionTime
    fun generateReport(): String {
        Thread.sleep(500) // Simulates expensive computation
        return "Report generated"
    }
}
```

When calling generateReport(), this appears in the logs:
```txt
INFO  Method ReportService.generateReport() executed in 502ms
```

## 🧠 Summary & Best Practices

| Best Practice                      | Recommendation                                                     |
| ---------------------------------- | ------------------------------------------------------------------ |
| ✅ Modularize cross-cutting logic  | Don't scatter logging throughout services/controllers              |
| ✅ Targeted usage                  | Only mark where measurement makes sense                            |
| ⚠️ Only works on Spring Beans     | `@Component`, `@Service` etc. required                            |
| ✅ Avoid short runtimes            | Consider logging overhead – don't use on very small methods       |
| ✅ Use logger correctly            | Use SLF4J with `LoggerFactory.getLogger()`                        |


## 📘 Advanced Topics

- 📦 @Around, @Before, @After, @AfterThrowing – different AOP advice types
- 🧵 ThreadLocal for tracing / request context
- 📈 Micrometer for metrics + Prometheus/Grafana integration
- 🔒 AOP for security (e.g., role checking via @Secured)
- 🐞 Exception logging via @AfterThrowing



## Back to Content:
[Back to Starting Point](../README_EN.md)