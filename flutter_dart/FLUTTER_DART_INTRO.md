# 🎯 Kurzes Intro in Flutter mit Dart

Dieses Intro basiert auf **Flutter aktuell -> 3.32** und beschreibt moderne Entwicklung für Cross-Plattform und Web. 
Flutter 
bietet eine deklarative UI Entwicklung und eignet sich hervorragend für Cross-Plattform-Entwicklung.

---

## 🚀 Warum Flutter?

- **Single Codebase** für Web, iOS, Android, Desktop.
- **Schnelle Entwicklungszyklen** mit Hot Reload.
- **Ansprechende UI** durch eigenes Rendering via Canvas (nicht auf DOM angewiesen).
- **Zukünftige Unterstützung für WASM (WebAssembly)** für bessere Performance.

---

## 🎯 Flutter für Web

Flutter Web nutzt aktuell eine Canvas-Rendering-Strategie, was volle Kontrolle über die UI bietet, allerdings nicht SEO-optimiert ist. Mit dem geplanten Umstieg auf **WASM** (WebAssembly) wird Flutter Web noch performanter.

---

## 🔤 Was ist Dart und Flutter?

- **Dart** ist eine objektorientierte Sprache, die stark typisiert ist. Sie ähnelt in Syntax und Features Kotlin, Swift oder Java.
- **Flutter** nutzt Dart als Programmiersprache für UI-Beschreibung mit „Widgets“, die sich zu komplexen Interfaces 
  zusammensetzen.

---

## 🧪 Flutter/Dart Minibeispiele

### Einfache Komponente

```dart
class HelloWorld extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Text("Hallo von Flutter Web!");
  }
}
```

### Button mit Interaktion
```dart
ElevatedButton(
  onPressed: () => print("Button gedrückt!"),
  child: Text("Klick mich"),
)
```

## ✅ Best Practices
- Nutze State Management: z. B. BLoC, Provider etc.
- Trenne UI, Business-Logik und Datenzugriff ([Clean Architecture mit DDD](CLEAN_ARCHITECTURE.md)).
- Nutze [Responsive Layouts](RESPONSIVE_DESIGN.md) (LayoutBuilder, MediaQuery).
- Reduziere Rebuilds durch effizientes Widget-Design.
- Modularisiere Komponenten in eigene Widgets.


## 📚 Weiterführende Themen
- Integration mit REST/GraphQL/gRPC-APIs
- Routing mit go_router
- [State Management: BLoC](STATEMANAGEMENT_BLOC.md)
- Flutter Web + Backend z. B. via Spring Boot
- Packages und Konzepte für sauberen Code und Code-Automatisierung/Generierung
  - FREEZED -> Boilerplate Code Generation für z.B. Daten-Klassen
  - GET-IT & Injectable -> Dependency Injection & Klassen-Zugriffe
  - RETROFIT & DIO -> API Client Generierung

## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)