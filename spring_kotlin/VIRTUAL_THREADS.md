# Language/Sprache : [EN](VIRTUAL_THREADS_EN.md) | [DE](VIRTUAL_THREADS.md)

# Spring Boot 3.5+ – Nebenläufigkeit, z.B. Scheduler mit Virtual Threads (Kotlin, Java 21+)

## 📝 Einleitung

Dieses Kapitel zeigt ein einfaches Beispiel, wie man mit **Spring Boot 3.5+**, **Kotlin** und **Java 21+ Virtual 
Threads** einen wiederkehrenden Scheduler umsetzt, der nebenläufige Aufgaben effizient ausführt. Beispiel: Ergebnisse 
von Threads werden gesammelt und zentral gespeichert – thread-sicher mit einer `ConcurrentHashMap`.

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

## Nebenläufigkeit in Spring Boot (Kotlin) mit Semaphore & Virtual Threads
Dieses Unterkapitel vertieft die Nutzung von Semaphores, Virtual Threads (Java 21) und Nebenläufigkeit in Spring Boot mit einfachen, praxisnahen Beispielen. Die „Jobrunner“ simulieren zeitintensive I/O-Operationen (z. B. Dateizugriffe, Netzwerkanfragen) mit Thread.sleep.

### Voraussetzungen
* Spring Boot 3.2+
* JDK 21 (für Virtual Threads)
* Kotlin + Gradle, kotlin("plugin.spring") empfohlen (macht Klassen/Methoden open für Proxies/AOP)

#### Basics: Nebenläufigkeit kurz & knackig
   Concurrency vs. Parallelism
   Concurrency = Aufgaben werden überlappend bearbeitet (zeitlich verschachtelt).
   Parallelism = Aufgaben laufen gleichzeitig auf mehreren CPU-Kernen.

Threads & Tasks
Ein Task ist die Arbeitseinheit, ein Thread ist die Ausführungseinheit. Executor-Services nehmen Tasks entgegen und führen sie auf Threads aus.

Blocking vs. Non-Blocking
I/O blockiert oft Threads. Mit Virtual Threads wird Blocking günstiger (weniger Ressourcen), ohne dass man auf 
asynchrone Callbacks umsteigen muss.

### Synchronisations-Primitiven

* Semaphore(n): Max. n gleichzeitige Zugriffe (Rate-Limit, Gate).
* ReentrantLock: Exklusiver Zugriff (kritischer Abschnitt).
* CountDownLatch, CyclicBarrier: Koordination von Start/Ende.

---

#### Virtual Threads (Java 21) – warum & wie? Was?
Leichtgewichtige Threads, die vom JVM-Scheduler auf wenige Carrier-Threads gemultiplext werden. Ideal für viele blockierende I/O-Tasks.

**Vorteile**:

* Sehr viele gleichzeitige, blockierende Tasks möglich.
* Bekannte Java-APIs (InputStream, JDBC, Thread.sleep) funktionieren unverändert.
* Kein spezielles „async/await“ nötig.

**Wann nicht?**

Rechenintensive, CPU-lastige Aufgaben bringen keinen Zusatznutzen durch Virtual Threads – dafür lieber ein fixes Thread-Pool-Limit.

**Executors:**

* Executors.newVirtualThreadPerTaskExecutor() – 1 Virtual Thread pro Task.
* Executors.newFixedThreadPool(n) – fester Pool klassischer Plattform-Threads.
* ForkJoinPool.commonPool() – Work-Stealing, gut für viele kurze Tasks.

---

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
Ablauf:
![Nebenläufigkeit](assets/nebenlaeufigkeit.svg)

Code-Beispiel:
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

---
## Spring-Scheduling: die Basis richtig setzen
Spring Boot startet bei @EnableScheduling standardmäßig einen Scheduler mit einer kleinen Threadanzahl.

### Option A – per Properties erhöhen (wenn nötig):
```properties
# application.properties
spring.task.scheduling.pool.size=4
```
### Option B – explizit konfigurieren:
```kotlin
@Configuration
@EnableScheduling
class SchedulingConfig : SchedulingConfigurer {
  override fun configureTasks(registrar: ScheduledTaskRegistrar) {
    val scheduler = Executors.newScheduledThreadPool(2) // klassische Threads für die Ticks
    registrar.setScheduler(scheduler)
  }
}
```
Merke: Der Scheduler-Thread triggert nur — die eigentliche Arbeit lagern wir auf Virtual Threads aus.

### Beispiel: „Skippen“, solange ein Job läuft (Semaphore + Virtual Thread)
**Use Case: Ein Job wird alle 60 s gestartet. Läuft der vorige noch, wird geskippt, statt parallel zu starten.**
```kotlin
@Component
class FsJob(
    @Value("\${app.fs.inbox:/tmp/inbox}") private val inbox: Path,
    @Value("\${app.fs.outbox:/tmp/outbox}") private val outbox: Path
) {
    private val log = LoggerFactory.getLogger(javaClass)

    // genau ein gleichzeitiger Job erlaubt
    private val gate = Semaphore(1) // non-fair reicht meist; für Fairness: Semaphore(1, true)

    // ein Virtual Thread pro Task
    private val vt = Executors.newVirtualThreadPerTaskExecutor()

    @Scheduled(fixedRate = 60_000)
    fun tick() {
        if (!gate.tryAcquire()) {
            log.debug("Skip: Job läuft bereits.")
            return
        }
        vt.execute {
            try {
                runJob() // zeitintensives I/O simuliert
            } catch (t: Throwable) {
                log.error("Job fehlgeschlagen", t)
            } finally {
                gate.release()
            }
        }
    }

    // Beispiel: „langes I/O“ – hier via Sleep simuliert
    open fun runJob() {
        log.info("Job gestartet")
        Files.createDirectories(outbox)
        // Simulation: 3 Dateien „verarbeiten“
        repeat(3) { idx ->
            Thread.sleep(1_000) // stellvertretend für langsamem I/O (Netz/FS/DB)
            val target = outbox.resolve("processed-$idx.txt")
            Files.writeString(target, "done at ${Instant.now()}")
            log.debug("Datei {} verarbeitet", target.fileName)
        }
        log.info("Job fertig")
    }

    @PreDestroy
    fun shutdown() = vt.shutdown()
}
```

### Warum funktioniert das?

* tryAcquire() verhindert Parallel-Start.
* Virtual Threads sorgen dafür, dass blockierendes I/O „billig“ bleibt, ohne den Scheduler-Thread zu belegen.

### Beispiel: Begrenzte Parallelität (Semaphore mit mehreren Permits)
**Use Case: Es gibt 100 Dateien, aber wir wollen max. 5 gleichzeitig verarbeiten (sanftes Rate-Limit).**

```kotlin
@Component
class BoundedParallelismJob {
    private val vt = Executors.newVirtualThreadPerTaskExecutor()
    private val permits = Semaphore(5) // max 5 gleichzeitige Worker

    fun processAll(files: List<Path>) {
        val futures = files.map { file ->
            vt.submit {
                if (!permits.tryAcquire(10, TimeUnit.SECONDS)) {
                    // optional: Task überspringen, wenn zu lange gewartet
                    return@submit
                }
                try {
                    processOne(file)
                } finally {
                    permits.release()
                }
            }
        }
        // optional: Ergebnisse abwarten
        futures.forEach { it.get() }
    }

    private fun processOne(file: Path) {
        // I/O simulieren
        Thread.sleep(500)
        Files.writeString(file.resolveSibling("${file.fileName}.ok"), "ok")
    }
}
```
#### Varianten:

* Fairness (Semaphore(5, true)) gewährleistet FIFO-Vergabe.
* Timeout (tryAcquire(timeout, unit)) um Skips oder Backpressure zu implementieren.

---
### Beispiel: „Rotate-then-Ship“ – Scheduler + Virtual Threads
Typisches Log-Shipping-Muster: Erst rotieren, dann die fertige Datei wegschicken (keine konkurrierenden Schreiber mehr).

```kotlin
@Component
class RotateAndShip(
    private val shipper: ShipperService
) {
    private val vt = Executors.newVirtualThreadPerTaskExecutor()
    private val oneAtATime = Semaphore(1)

    @Scheduled(cron = "0 0/5 * * * *") // alle 5 Minuten
    fun rotateAndShip() {
        if (!oneAtATime.tryAcquire()) return
        vt.execute {
            try {
                rotateLogs()   // blockierendes FS-I/O
                shipper.ship() // z. B. HTTP Upload -> blockierend akzeptabel
            } finally {
                oneAtATime.release()
            }
        }
    }

    private fun rotateLogs() {
        Thread.sleep(1_000) // stellvertretend für I/O
    }
}

@Service
class ShipperService {
    fun ship() {
        Thread.sleep(1_500) // Upload simuliert
    }
}
```
## Häufige Stolperfallen & Tipps
* AOP/Self-Invocation: Ruft eine Methode sich im selben Bean selbst auf (this.runJob()), greift Spring-AOP dafür 
nicht (Proxy wird umgangen). Wenn ihr z. B. @LogExecutionTime nutzt, ruft die Methode über ein anderes Bean oder über den AOP-Proxy auf.


* Virtual Threads ≠ Turbo für CPU-Last: Für CPU-intensive Arbeit lieber newFixedThreadPool(n) nutzen, damit ihr das parallel ausgeführte Kontingent bewusst begrenzt.


* Shutdown nicht vergessen: Executor in @PreDestroy schließen. Für kurzlebige Hilfs-Tasks kann auch Thread.ofVirtual().start { ... } reichen.


* Semaphore fair/unfair: Unfaire Semaphores sind schneller, faire verhindern Starvation.


* Fehlerbehandlung: In Worker-Lambdas Exceptions abfangen und loggen, sonst gehen sie „still“ verloren.


---
## Kleine API-Rezepte
### Einfacher Virtual Thread Start (ohne Executor):
```kotlin
val v = Thread.ofVirtual().start {
    Thread.sleep(500)
    println("Hello from VT")
}
v.join()
```

### CompletableFuture mit Virtual Threads:
```kotlin
val vt = Executors.newVirtualThreadPerTaskExecutor()
val cf = CompletableFuture.supplyAsync({
    Thread.sleep(300) // I/O
    42
}, vt)
println(cf.get())
vt.shutdown()
```
---
## Tests: „Skip während Lauf“ verifizieren
```kotlin
class FsJobTest {
    @Test
    fun `zweiter Tick wird geskippt`() {
        val job = FsJob(Path.of("in"), Path.of("out"))
        // ersten Tick starten -> hält 3s
        job.tick()
        // unmittelbar erneut -> sollte skippen
        job.tick()
        // Assertions über Logs/Latches wären hier sinnvoll
    }
}
```
>Für deterministische Tests statt Thread.sleep lieber CountDownLatch oder Semaphore einsetzen, um Start/Ende zu steuern.

### Exkurs: Executors im Überblick
* newSingleThreadExecutor() – seriell, deterministische Reihenfolge.
* newFixedThreadPool(n) – konstante Parallelität, gut für CPU-Arbeit.
* newCachedThreadPool() – unbounded, Vorsicht bei Lastspitzen.
* ForkJoinPool – Work-Stealing für viele kleine Aufgaben.
* newVirtualThreadPerTaskExecutor() – ein VT je Task, ideal für I/O-lastig.

---
## Hinweis zu Kotlin Coroutines
Kotlin bringt Coroutines als leichtgewichtiges Nebenläufigkeits-Modell mit: ```suspend, flow, withContext(Dispatchers.IO)``` usw.
In Spring-Projekten könnt ihr wahlweise:

bei „klassischem“ Java-Stil bleiben (Virtual Threads), oder
idiomatisch Coroutines nutzen (z. B. ```spring-boot-starter-webflux``` + ```kotlinx-coroutines```).
Beides adressiert ähnliche Probleme mit unterschiedlichen Abstraktionen. Für einfache I/O-Jobs sind Virtual Threads oft der schnellste Einstieg ohne API-Wechsel.

---

## Mini-Cheat-Sheet
* Skip solange läuft: Semaphore(1) + tryAcquire()/release().


* I/O-Tasks: Virtual Threads (newVirtualThreadPerTaskExecutor()), Thread.sleep unproblematisch.


* Max. Parallelität N: Semaphore(N) vor der Arbeit.


* AOP-Timing: Aufruf über Proxy/anderes Bean, nicht this.


* CPU-Last: Fixed Pool statt Virtual Threads.
---

### 🚀 Voraussetzungen
- Java 21 (für Virtual Threads)
- Kotlin 1.9+
- Spring Boot 3.2+ (Denkt an die OS-Support Lifecycles, aktuell Spring Boot 3.5 (08/2025) -> https://endoflife.
  date/spring-boot)
- Gradle (oder Maven)

### 📈 Erweiterungsideen
- Fehlerbehandlung & Retry-Logik
- Parallelisierung von mehr als 2 Tasks
- Austausch von Virtual Threads gegen Coroutines (bei Voll-Kotlin-Projekten)
- Speicherung der Ergebnisse in einer Datenbank


## 🧠 Fazit
Virtual Threads erlauben extrem einfache und performante Nebenläufigkeit in Java 21 – besonders in Kombination mit Spring Boot. Wichtig bleibt jedoch: Thread-Sicherheit ist weiterhin entscheidend – besonders bei Singleton-Beans.

## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)