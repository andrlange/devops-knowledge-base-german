# Sprache/Language : [DE](FLUTTER_DART_INTRO.md) | [EN](FLUTTER_DART_INTRO_EN.md)

# 🎯 Quick Intro to Flutter with Dart

This intro is based on **Flutter current -> 3.32** and describes modern development for cross-platform and web.
Flutter
offers declarative UI development and is excellent for cross-platform development.

---

## 🚀 Why Flutter?

- **Single Codebase** for Web, iOS, Android, Desktop.
- **Fast Development Cycles** with Hot Reload.
- **Appealing UI** through custom rendering via Canvas (not dependent on DOM).
- **Future Support for WASM (WebAssembly)** for better performance.

---

## 🎯 Flutter for Web

Flutter Web currently uses a Canvas-rendering strategy, which offers full control over the UI, but is not SEO-optimized. With the planned transition to **WASM** (WebAssembly), Flutter Web will become even more performant.

---

## 🔤 What are Dart and Flutter?

- **Dart** is an object-oriented language that is strongly typed. It is similar in syntax and features to Kotlin, Swift, or Java.
- **Flutter** uses Dart as the programming language for UI description with "Widgets" that compose into complex interfaces.

---

## 🧪 Flutter/Dart Mini Examples

### Simple Component

```dart
class HelloWorld extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Text("Hello from Flutter Web!");
  }
}
```

### Button with Interaction
```dart
ElevatedButton(
  onPressed: () => print("Button pressed!"),
  child: Text("Click me"),
)
```

## ✅ Best Practices
- Use State Management: e.g., BLoC, Provider, etc.
- Separate UI, business logic, and data access ([Clean Architecture with DDD](CLEAN_ARCHITECTURE_EN.md)).
- Use [Responsive Layouts](RESPONSIVE_DESIGN_EN.md) (LayoutBuilder, MediaQuery).
- Reduce rebuilds through efficient widget design.
- Modularize components into separate widgets.


## 📚 Advanced Topics
- Integration with REST/GraphQL/gRPC APIs
- Routing with go_router
- [State Management: BLoC](STATEMANAGEMENT_BLOC_EN.md)
- Flutter Web + Backend e.g., via Spring Boot
- Packages and concepts for clean code and code automation/generation
    - FREEZED -> Boilerplate code generation for e.g., data classes
    - GET-IT & Injectable -> Dependency Injection & class access
    - RETROFIT & DIO -> API Client generation

## Back to Content:
[Back to Starting Point](../README_EN.md)