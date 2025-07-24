# Spring Boot 3.5+ – Scheduler mit Virtual Threads (Kotlin, Java 21+)

## 📝 Einleitung

Dieses Projekt zeigt ein einfaches Beispiel, wie man mit **Spring Boot 3.5+**, **Kotlin** und **Java 21+ Virtual 
Threads** einen wiederkehrenden Scheduler umsetzt, der nebenläufige Aufgaben effizient ausführt. Die Ergebnisse der Threads werden gesammelt und zentral gespeichert – thread-sicher mit einer `ConcurrentHashMap`.

---

## 🧵 Was sind Virtual Threads?

**Virtual Threads** sind ein Feature ab **Java 21**, das leichtgewichtige, vom JVM-Thread-Stack entkoppelte Threads bereitstellt. Sie ermöglichen:

- Tausende gleichzeitig laufende Tasks mit geringem Speicherverbrauch
- Verbesserte Skalierbarkeit ohne komplexe Thread-Pools
- Besonders nützlich bei I/O-lastigen Operationen

Beispiel:

```kotlin
Thread.startVirtualThread {
    println("Ich laufe als Virtual Thread!")
}
```

## 🔁 Nebenläufigkeit & Thread-Sicherheit
In Spring ist eine @Service-Bean per Default Singleton – das bedeutet:

- Es gibt nur eine Instanz pro Bean, auf die alle Threads zugreifen

- Methoden können gleichzeitig aufgerufen werden – je nach Kontext von unterschiedlichen Threads

### ❗ Wichtig:
- Zustandsbehaftete Felder in Singleton-Beans müssen thread-sicher sein

- Verwende für geteilten Zustand:

    - ConcurrentHashMap – thread-sicheres Map-Handling

    - AtomicInteger – atomare Zahlenoperationen

    - synchronized oder ReentrantLock – für kontrollierten Zugriff

- Besser: Beans zustandslos halten → alle Daten als lokale Variablen innerhalb der Methoden

## ✅ Beispiel: Scheduler mit synchronisierten und parallelen Virtual Threads


```kotlin

@Component
@EnableScheduling
class SchedulerExample {

    @Scheduled(fixedRate = 60_000) // alle 60 Sekunden
    fun runScheduler() {
        println("Scheduler gestartet um ${LocalTime.now()}")

        try {
            val thread1 = startVirtualThread("Thread 1",10_000)
            val thread3 = startVirtualThread("Thread 3",10_000)
            thread1.join() // warten bis Thread 1 fertig

            if (!thread1.isAlive) {
                val thread2 = startVirtualThread("Thread 2",10_000)
                thread2.join()
            }
            thread3.join()
        } catch (e: Exception) {
            println("Fehler im Scheduler: ${e.message}")
        }

        println("Scheduler-Durchlauf beendet um ${LocalTime.now()}")
    }

    private fun startVirtualThread(name: String, duration: Double): Thread {
        return Thread.startVirtualThread {
            println("$name gestartet")
            Thread.sleep(duration) // $duration Sekunden warten
            println("$name $number fertig")
        }
    }
}

```

### 🧪 Verhalten
Alle 60 Sekunden:

- Scheduler startet und zeigt Uhrzeit.
- Thread 1 wird als Virtual Thread gestartet, wartet 10 Sekunden, beendet.
- Gleichzeitig started Thread 3, macht das Gleiche.
- Wenn Thread1 fertig ist startet Thread 2 und macht das Gleiche.
- Scheduler-Durchlauf endet.







## ✅ Beispiel: Scheduler mit zwei Virtual Threads
Dieses Projekt zeigt:

- Ein Scheduler, der alle 60 Sekunden läuft

- Zwei Virtual Threads, die:

    - 10 Sekunden arbeiten

    - ein Ergebnis (String) zurückgeben

- Ergebnisse werden pro Durchlauf in einer ConcurrentHashMap gespeichert

### 🧩Ablauf
```text
1. Scheduler wird gestartet
2. Thread 1 wird als Virtual Thread gestartet
3. Nach 10 Sekunden: Ergebnis wird geliefert
4. Wenn erfolgreich → Thread 2 wird gestartet
5. Beide Ergebnisse werden gespeichert unter einem Run-Key (z. B. run-12:00:00)
```


#### 📄 Beispielcode (vereinfacht)
```kotlin
@Component
@EnableScheduling
class SchedulerExample {

    private val resultMap = ConcurrentHashMap<String, Map<String, String>>()

    @Scheduled(fixedRate = 60_000)
    fun runScheduler() {
        val runId = "run-" + LocalTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"))
        val resultForRun = mutableMapOf<String, String>()

        try {
            val result1 = runVirtualTask("Thread 1")
            resultForRun["Thread 1"] = result1

            val result2 = runVirtualTask("Thread 2")
            resultForRun["Thread 2"] = result2

        } catch (e: Exception) {
            println("Fehler: ${e.message}")
        }

        resultMap[runId] = resultForRun
    }

    private fun runVirtualTask(name: String): String {
        val future = CompletableFuture<String>()
        Thread.startVirtualThread {
            println("$name gestartet")
            Thread.sleep(10_000)
            val result = "Ergebnis von $name"
            future.complete(result)
        }
        return future.get()
    }
}
```

📦 Build Setup (Gradle)
```kotlin
plugins {
    id("org.springframework.boot") version "3.5.0"
    kotlin("jvm") version "1.9.10"
}

java {
    toolchain {
        languageVersion.set(JavaLanguageVersion.of(21))
    }
}
```

### 🚀 Voraussetzungen
- Java 21 (für Virtual Threads)
- Kotlin 1.9+
- Spring Boot 3.5+ (Denkt an die OS-Support Lifecycles -> https://endoflife.date/spring-boot)
- Gradle (oder Maven)

### 📈 Erweiterungsideen
- Fehlerbehandlung & Retry-Logik
- Parallelisierung von mehr als 2 Tasks
- Austausch von Virtual Threads gegen Coroutines (bei Voll-Kotlin-Projekten)
- Speicherung der Ergebnisse in einer Datenbank


## 🧠 Fazit
Virtual Threads erlauben extrem einfache und performante Nebenläufigkeit in Java 21 – besonders in Kombination mit Spring Boot. Wichtig bleibt jedoch: Thread-Sicherheit ist weiterhin entscheidend – besonders bei Singleton-Beans.