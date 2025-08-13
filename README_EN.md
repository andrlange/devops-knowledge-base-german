# Sprache/Language : [DE](README.md) | [EN](README_EN.md)

!Work in progress, many of the links in the table of contents are placeholders for now!


# 📚 DevOps & Development Knowledge Collection
- Full Stack -> Full Chain : Spring Boot - Kotlin -> Flutter - Dart, Backing Services
- GitOps "like" -> Git -> Cloud-Native-Buildpacks -> Tekton -> Harbor Registry -> ArgoCD -> Kubernetes or other
  runtime platforms (Heroku, Cloud Foundry and more)
- Best Practices, Security, Observability

[Jump directly to the content](#table-of-contents-of-the-knowledge-collection)

Welcome to this ongoing **knowledge collection on DevOps and software development**. This collection serves as a structured, growing guide for developers, IT-related roles, and anyone interested in modern software delivery, automation, and development technologies.

## 🎯 Goal

The goal of this collection is to bundle **practical information and experiences** from various subject areas to make them usable in everyday work as a reference or learning resource. The content is derived from real projects, experiments, and insights – not purely theoretical treatises, but directly applicable knowledge.

## 📄 Why Markdown?

The entire documentation is based on **Markdown** – a lightweight markup system that prioritizes clarity and readability.

### What is Markdown?

**Markdown** is a simple text formatting language that allows you to write structured content like headings,
lists, code blocks, or tables with minimal effort in plain text. It is supported in many developer tools,
documentation systems, and platforms like GitHub, GitLab, IntelliJ, VS Code, and much more.

### Why do we use Markdown?

- **Easy to read and write** – even without special tools
- **Version control friendly** – ideal for Git repositories
- **Platform independent** – can be displayed anywhere
- **Seamlessly integrable** into static sites, wikis, and build processes

Markdown is therefore an ideal format for an open, searchable, and collaboratively maintained knowledge collection.


## Topic Overview
The chosen languages, frameworks, and tool chain were selected to learn the topics as quickly and efficiently as possible, as well as to set up a complete DevOps environment with runtime environment using minimal resources.

The most important topics deal with fundamental concepts such as Clean Architecture with Domain Driven Design
patterns (DDD), clean code, readability, modularization and error handling, robustness, and operational aspects.

All languages, frameworks, or components can also be replaced.

The content is thematically organized into subdirectories, each covering independent areas, e.g.:

- **Programming Languages & Frameworks & Core Concepts**
    - `Kotlin` with `Spring Boot`
    - `Dart` with `Flutter`

- **CI/CD & Automation**
    - Writing pipelines with `Tekton`
    - GitOps with `ArgoCD`
    - Automated container building and deployment

- **Container & Identity & Access Management & Infrastructure**
    - `Docker`, `Harbor Registry`, `Keycloak IAM`, `Technitium DNS`
    - Kubernetes fundamentals and advanced use cases

- **Tooling & Best Practices**
    - Observability, Logging, Monitoring
    - Security aspects in the DevOps pipeline
    - Clean Code and architectural considerations

## 👥 Target Audience

This collection is aimed at:

- **Developers** who want to look beyond the scope of code
- **DevOps Engineers** who want to document recurring knowledge
- **IT-related professionals** who want to systematically familiarize themselves with new technologies
- **All interested parties** who have an interest in the topics or simply want to inform themselves

## 🧱 Structure & Collaboration

All content is modularly structured and can be continuously expanded. Contributions are welcome – whether in the form of code examples, short how-tos, or in-depth analyses.

---

> This collection is alive and grows with every new challenge and solution. It should help preserve knowledge, pass it on, and develop together.


---

# Table of Contents of the Knowledge Collection

A thematically structured overview of all content captured so far. The paths are relative, so they work directly in common Git repository viewers like GitHub or GitLab.

---

## 📦 Languages & Frameworks

### Markdown
- [Markdown HowTo](markdown/MARKDOWN_HOWTO_EN.md)

### Quick Intro
- [Spring Boot with Kotlin](spring_kotlin/SPRINGBOOT_KOTLIN_INTRO_EN.md)
- [Flutter Framework](flutter_dart/FLUTTER_DART_INTRO_EN.md)

### Kotlin
- [Concurrency, Virtual Threads with Kotlin & Spring Boot](spring_kotlin/VIRTUAL_THREADS_EN.md)
- [Null Handling in Kotlin, why avoid `!=null`!](spring_kotlin/NULLHANDLING_KOTLIN_EN.md)

### Spring & Spring Boot
- [Rest Controller and Error Handling with Advices](spring_kotlin/CONTROLLER_ADVICE_EN.md)
- [Custom Validators for Complex DTOs](spring_kotlin/CUSTOM_VALIDATORS_EN.md)
- [Runtime Measurement with AOP and ANNOTATION](spring_kotlin/SPRING_AOP_CUSTOM_ANNOTATION_EN.md)
- [Introduction to Caching Strategies with Caffeine, Redis/Valkey](spring_kotlin/CACHING_INTRO_EN.md)
- [Best Practices for Transactions with @Transactional](spring_kotlin/SPRING_TRANSACTIONAL_EN.md)

### Dart & Flutter
- [Flutter State Management Overview with BLoC Patterns](flutter_dart/STATEMANAGEMENT_BLOC_EN.md)
- [Responsive Design Pattern for UIs with Flutter](flutter_dart/RESPONSIVE_DESIGN_EN.md)
- [Clean Architecture, Services and Asynchronicity with Streams](flutter_dart/CLEAN_ARCHITECTURE_EN.md)
- [Testing in Flutter Projects](flutter_dart/FLUTTER_DART_TESTING_EN.md)

---

## 🔁 CI/CD & Automation

### Tekton
- [Creating Pipelines with Tekton](tekton/PIPELINES_EN.md)
- [Parameterization and Triggers](tekton/TRIGGERS_EN.md)

### ArgoCD
- [GitOps with ArgoCD](argocd/GITOPS_WORKFLOW_EN.md)
- [Helm & Kustomize in ArgoCD](argocd/HELM_KUSTOMIZE_EN.md)

### General
- [CI/CD Principles and Best Practices](cicd/BEST_PRACTICES_EN.md)

---

## 🐳 Container & Infrastructure

- [Docker – Introduction and Usage](container/DOCKER_INTRO_EN.md)
- [Building Container Images Securely and Efficiently](container/SECURE_BUILDS_EN.md)
- [Introduction to Kubernetes](infrastructure/KUBERNETES_BASICS_EN.md)
- [Kubernetes Deployment Patterns](infrastructure/K8S_PATTERNS_EN.md)

---

## 🛠️ Tooling & Operations

- [Monitoring with Prometheus & Grafana](tooling/MONITORING_PROMETHEUS_EN.md)
- [Logging with Loki, Fluent Bit and Grafana](tooling/LOGGING_STACK_EN.md)
- [Configuring Alerting Properly](tooling/ALERTING_EN.md)
- [DevOps Tooling Overview](tooling/TOOLS_OVERVIEW_EN.md)

---

## 🔐 Security & Quality

- [Security Scanning in the CI/CD Pipeline](security/SECURITY_SCANS_EN.md)
- [Secret Management Best Practices](security/SECRET_ENS.md)
- [Establishing Code Quality and Linting in Teams](quality/CODE_QUALITY_EN.md)

---

> 📌 **Note:** This overview is continuously expanded. New topics and contributions find their place in the respective categories. Pull requests welcome!