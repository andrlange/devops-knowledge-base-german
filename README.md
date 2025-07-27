!In Bearbeitung, viele der Links im Inhaltsverzeichnis sind vorerst Platzhalter!


# 📚 Wissenssammlung DevOps & Entwicklung 
- Full Stack -> Full Chain : Spring Boot - Kotlin -> Flutter - Dart, Backing Services
- GitOps "like" -> Git -> Cloud-Native-Buildpacks -> Tekton -> Harbor Regestry -> ArgoCD -> Kubernetes oder andere 
  Run-Platformen (Heroku, Cloudfoundry und mehr)
- Best Practices, Security, Observability

Willkommen zu dieser fortlaufenden **Wissenssammlung rund um DevOps und Softwareentwicklung**. Diese Sammlung dient als strukturierter, wachsender Leitfaden für Entwickler*innen, IT-nahe Rollen und alle, die sich für moderne Softwarebereitstellung, Automatisierung und Entwicklungstechnologien interessieren.

## 🎯 Ziel

Das Ziel dieser Sammlung ist es, **praxisnahe Informationen und Erfahrungen** aus verschiedensten Themengebieten zu bündeln, um sie im Alltag als Nachschlagewerk oder Lernquelle nutzbar zu machen. Die Inhalte entstehen aus realen Projekten, Experimenten und Erkenntnissen – keine rein theoretischen Abhandlungen, sondern direkt anwendbares Wissen.

## 📄 Warum Markdown?

Die gesamte Dokumentation basiert auf **Markdown** – einem leichtgewichtigen Auszeichnungssystem, das Klarheit und Lesbarkeit in den Vordergrund stellt.

### Was ist Markdown?

**Markdown** ist eine einfache Textformatierungssprache, die es erlaubt, strukturierte Inhalte wie Überschriften, 
Listen, Codeblöcke oder Tabellen mit minimalem Aufwand in Klartext zu schreiben. Sie wird in vielen Entwickler-Tools,
Dokumentationssystemen und Plattformen wie GitHub, GitLab oder IntelliJ, VS Code und vieles mehr unterstützt.

### Warum verwenden wir Markdown?

- **Einfach lesbar und schreibbar** – auch ohne spezielle Tools
- **Versionskontrolle-freundlich** – ideal für Git-Repositories
- **Plattformunabhängig** – kann überall angezeigt werden
- **Nahtlos integrierbar** in statische Seiten, Wikis und Build-Prozesse

Markdown ist damit ein ideales Format für eine offene, durchsuchbare und gemeinsam pflegbare Wissenssammlung.


## 🧭 Themenüberblick
Die gewählten Sprachen, Frameworks Tool-Chain wurden gewählt, um möglichst schnell und effizient die Themen zu 
erlernen, als auch mit wenig Resourcen eine Komplette Dev-Ops Umgebung mit Laufzeitumgebung aufzusetzen.

Die wichtigsten Themen beschäftigen sich mit den Grundkonzepten wie. Clean-Architecture mit Domain Driven Design 
Patterns (DDD), sauberen Code, Lesbarkeit, Modularisierung und Fehlerbehandlung, Rubustheit und Betriebliche Aspekte. 

Alle Sprachen, Frameworks oder Komponenten können auch ersetzt werden.

Die Inhalte sind thematisch gegliedert in Unterordner, die jeweils eigenständige Bereiche behandeln, z. B.:

- **Programmiersprachen & Frameworks & Grundkonzepte**
    - `Kotlin` mit `Spring Boot`
    - `Dart` mit `Flutter`

- **CI/CD & Automatisierung**
    - Schreiben von Pipelines mit `Tekton`
    - GitOps mit `ArgoCD`
    - Automatisiertes Container-Building und Deployment

- **Container & Identity & Access Management & Infrastruktur**
    - `Docker`, `Harbor Registry`, `Keycloak IAM`, `Technitium DNS`
    - Kubernetes Grundlagen und erweiterte Use Cases

- **Tooling & Best Practices**
    - Observability, Logging, Monitoring
    - Security-Aspekte in der DevOps-Pipeline
    - Clean Code und Architekturüberlegungen

## 👥 Zielgruppe

Diese Sammlung richtet sich an:

- **Entwickler*innen**, die über den Tellerrand des Codes hinausblicken möchten
- **DevOps-Engineers**, die wiederkehrendes Wissen dokumentieren wollen
- **IT-nahe Fachkräfte**, die sich systematisch in neue Technologien einarbeiten möchten
- **Alle Interessierten**, die Interesse an den Themen haben oder sich einfach nur Informieren möchten

## 🧱 Struktur & Mitgestaltung

Alle Inhalte sind modular aufgebaut und können stetig erweitert werden. Beiträge sind willkommen – sei es in Form von Codebeispielen, kurzen How-Tos oder tiefergehenden Analysen.

---

> Diese Sammlung ist lebendig und wächst mit jeder neuen Herausforderung und Lösung. Sie soll helfen, Wissen zu bewahren, weiterzugeben und sich gemeinsam weiterzuentwickeln.


---

# 🗂️ Inhaltsverzeichnis der Wissenssammlung

Eine thematisch strukturierte Übersicht aller bisher erfassten Inhalte. Die Pfade sind relativ, sodass sie direkt in gängigen Git-Repository-Viewern wie GitHub oder GitLab funktionieren.

---

## 📦 Sprachen & Frameworks

### Markdown
- [Mardown HowTo](markdown/MARKDOWN_HOWTO.md)

### Kurz-Intro 
- [Spring Boot mit Kotlin](spring_kotlin/SPRINGBOOT_KOTLIN_INTRO.md)
- [Flutter Framework](flutter_dart/FLUTTER_DART_INTRO.md)
- 
### Kotlin
- [Virtuelle Threads mit Kotlin & Spring Boot](spring_kotlin/VIRTUAL_THREADS.md)
- [Null Handling in Kotlin, warum `!=null` vermeiden!](spring_kotlin/NULLHANDLING_KOTLIN.md)

### Spring & Spring Boot
- [Rest Controller und Fehlerbehandlung mit Advices](spring_kotlin/CONTROLLER_ADVICE.md)
- [Eigene Validatoren für komplexere DTOs](spring_kotlin/CUSTOM_VALIDATORS.md)
- [Laufzeitmessung mit AOP und ANNOTATION](spring_kotlin/SPRING_AOP_CUSTOM_ANNOTATION.md)
- [Einführung in Caching Strategien mit Caffeine, Redis/Valkey](spring_kotlin/CACHING_INTRO.md)

### Dart & Flutter
- [Flutter State Management im Überblick mit BLoC Patterns](flutter_dart/STATEMANAGEMENT_BLOC.md)
- [Responsive Design Pattern für UIs mit Flutter](flutter_dart/RESPONSIVE_DESIGN.md)
- [Clean Architecture, Services und Asynchronität mit Streams](flutter_dart/CLEAN_ARCHITECTURE.md)
- [Testing in Flutter-Projekten](flutter_dart/TESTING.md)

---

## 🔁 CI/CD & Automatisierung

### Tekton
- [Erstellung von Pipelines mit Tekton](tekton/PIPELINES.md)
- [Parametrisierung und Triggers](tekton/TRIGGERS.md)

### ArgoCD
- [GitOps mit ArgoCD](argocd/GITOPS_WORKFLOW.md)
- [Helm & Kustomize in ArgoCD](argocd/HELM_KUSTOMIZE.md)

### Allgemein
- [CI/CD Prinzipien und Best Practices](cicd/BEST_PRACTICES.md)

---

## 🐳 Container & Infrastruktur

- [Docker – Einführung und Einsatz](container/DOCKER_INTRO.md)
- [Container Images sicher und effizient bauen](container/SECURE_BUILDS.md)
- [Einführung in Kubernetes](infrastructure/KUBERNETES_BASICS.md)
- [Kubernetes Deployment Patterns](infrastructure/K8S_PATTERNS.md)

---

## 🛠️ Tooling & Operations

- [Monitoring mit Prometheus & Grafana](tooling/MONITORING_PROMETHEUS.md)
- [Logging mit Loki, Fluent Bit und Grafana](tooling/LOGGING_STACK.md)
- [Alerting richtig konfigurieren](tooling/ALERTING.md)
- [DevOps Tooling Übersicht](tooling/TOOLS_OVERVIEW.md)

---

## 🔐 Sicherheit & Qualität

- [Security Scanning in der CI/CD-Pipeline](security/SECURITY_SCANS.md)
- [Secret Management Best Practices](security/SECRETS.md)
- [Codequalität und Linting im Team etablieren](quality/CODE_QUALITY.md)

---

> 📌 **Hinweis:** Diese Übersicht wird fortlaufend erweitert. Neue Themen und Beiträge finden ihren Platz in den jeweiligen Kategorien. Pull Requests willkommen!

