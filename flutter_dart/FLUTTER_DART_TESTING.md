# Flutter & Dart Testing Guide

Ein Leitfaden für das Testen von Flutter-Anwendungen mit Dart

## Inhaltsverzeichnis

1. [Allgemeine Grundlagen](#allgemeine-grundlagen)
2. [Unit Tests](#unit-tests)
3. [Widget Tests](#widget-tests)
4. [State Management Tests (BLoC)](#state-management-tests-bloc)
5. [Integration Tests](#integration-tests)
6. [BloC Tests](#state-management-tests-bloc)
7. [Golden Tests](#golden-tests)
8. [Asset Fonts in Flutter Tests](#asset-fonts-in-flutter-tests)
9. [Fazit](#fazit)

---


## Allgemeine Grundlagen

### Warum Tests überhaupt?

Tests sind essentiell für die Softwareentwicklung aus mehreren Gründen:

- **Qualitätssicherung**: Tests fangen Bugs frühzeitig ab, bevor sie in Produktion gehen
- **Refactoring-Sicherheit**: Code kann sicher umstrukturiert werden, ohne Angst vor Breaking Changes
- **Dokumentation**: Tests dokumentieren das erwartete Verhalten der Software
- **Wartbarkeit**: Gut getesteter Code ist einfacher zu warten und zu erweitern
- **Vertrauen**: Teams können mit mehr Selbstvertrauen neue Features entwickeln
- **Kostenersparnis**: Bugs in frühen Phasen zu finden ist deutlich günstiger als in Produktion

### Test-Driven Development (TDD) und Behavior-Driven Development (BDD)

#### Test-Driven Development (TDD)

TDD folgt dem **Red-Green-Refactor-Zyklus**:

1. **Red**: Schreibe einen fehlschlagenden Test
2. **Green**: Schreibe den minimal notwendigen Code, um den Test zu bestehen
3. **Refactor**: Verbessere den Code, ohne die Funktionalität zu ändern

```dart
// Beispiel TDD-Zyklus
// 1. RED: Test schreiben
test('should calculate area of rectangle', () {
  final calculator = AreaCalculator();
  expect(calculator.rectangleArea(5, 3), equals(15));
});

// 2. GREEN: Minimale Implementierung
class AreaCalculator {
  double rectangleArea(double width, double height) {
    return width * height;
  }
}

// 3. REFACTOR: Code verbessern (falls notwendig)
```

#### Behavior-Driven Development (BDD)

BDD erweitert TDD um geschäftliche Perspektiven und nutzt natürliche Sprache:

- **Given**: Ausgangssituation
- **When**: Aktion
- **Then**: Erwartetes Ergebnis

```dart
// BDD-Stil in Dart
group('User Authentication', () {
  testWidgets('Given user is not logged in, When user enters valid credentials, Then user should be logged in', (tester) async {
    // Given
    await tester.pumpWidget(MyApp());
    
    // When
    await tester.enterText(find.byKey(Key('email')), 'user@example.com');
    await tester.enterText(find.byKey(Key('password')), 'password123');
    await tester.tap(find.byKey(Key('login_button')));
    await tester.pumpAndSettle();
    
    // Then
    expect(find.text('Welcome'), findsOneWidget);
  });
});
```

### Unterschiede zwischen den Testtypen

**Test-Pyramide**: Die ideale Verteilung der Tests folgt einer Pyramidenstruktur:

```
       /\
      /  \     Integration Tests (wenige, langsam, teuer)
     /    \
    /______\   Widget Tests (moderate Anzahl, mittlere Geschwindigkeit)
   /        \
  /__________\  Unit Tests (viele, schnell, günstig)
```

- **Unit Tests**: Testen isolierte Funktionen, Methoden oder Klassen
- **Widget Tests**: Testen einzelne Widgets und deren Interaktionen
- **Integration Tests**: Testen das Zusammenspiel kompletter App-Bereiche
- **Golden Tests**: Testen das visuelle Erscheinungsbild von Widgets

---

## Unit Tests

### Kurze Einführung

Unit Tests sind die Grundlage jeder Test-Suite. Sie testen kleinste isolierte Einheiten des Codes wie einzelne Funktionen, Methoden oder Klassen. In Flutter nutzen wir das `test` Package für Unit Tests.

### Wann nutzt man Unit Tests?

**Verwende Unit Tests für:**
- Geschäftslogik und Algorithmen
- Datenmodelle und deren Methoden
- Utilities und Hilfsfunktionen
- Service-Klassen (z.B. API-Clients)
- State Management (Blocs, Providers, etc.)

**Nutze Unit Tests NICHT für:**
- UI-Komponenten (dafür Widget Tests)
- Datenbankoperationen (dafür Integration Tests)
- Navigation zwischen Screens

### Beispiele

#### Einfacher Unit Test

```dart
// test/models/user_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/models/user.dart';

void main() {
  group('User Model Tests', () {
    test('should create user with valid data', () {
      // Arrange
      const name = 'John Doe';
      const email = 'john@example.com';
      
      // Act
      final user = User(name: name, email: email);
      
      // Assert
      expect(user.name, equals(name));
      expect(user.email, equals(email));
      expect(user.isValid, isTrue);
    });
    
    test('should return false for invalid email', () {
      // Arrange
      final user = User(name: 'John', email: 'invalid-email');
      
      // Act & Assert
      expect(user.isValid, isFalse);
    });
  });
}
```

#### Service-Test mit Mocking

```dart
// test/services/api_service_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:http/http.dart' as http;
import 'package:my_app/services/api_service.dart';

@GenerateMocks([http.Client])
import 'api_service_test.mocks.dart';

void main() {
  group('ApiService Tests', () {
    late ApiService apiService;
    late MockClient mockClient;
    
    setUp(() {
      mockClient = MockClient();
      apiService = ApiService(client: mockClient);
    });
    
    test('should return user data on successful API call', () async {
      // Arrange
      const jsonResponse = '{"id": 1, "name": "John Doe"}';
      when(mockClient.get(any)).thenAnswer(
        (_) async => http.Response(jsonResponse, 200)
      );
      
      // Act
      final result = await apiService.getUser(1);
      
      // Assert
      expect(result.id, equals(1));
      expect(result.name, equals('John Doe'));
      verify(mockClient.get(Uri.parse('${apiService.baseUrl}/users/1')));
    });
    
    test('should throw exception on API error', () async {
      // Arrange
      when(mockClient.get(any)).thenAnswer(
        (_) async => http.Response('Not Found', 404)
      );
      
      // Act & Assert
      expect(
        () => apiService.getUser(999),
        throwsA(isA<ApiException>())
      );
    });
  });
}
```

### Worauf soll man achten?

- **AAA-Pattern**: Arrange, Act, Assert - strukturiere Tests klar
- **Ein Test, ein Konzept**: Jeder Test sollte nur eine Sache testen
- **Aussagekräftige Namen**: Test-Namen sollten das erwartete Verhalten beschreiben
- **Isolation**: Tests sollen unabhängig voneinander laufen können
- **Mocking**: Verwende Mocks für externe Abhängigkeiten
- **Edge Cases**: Teste nicht nur den Happy Path, sondern auch Fehlerfälle

### Besonderheiten und Voraussetzungen

- **Package**: `dev_dependencies: flutter_test: sdk: flutter`
- **Mocking**: Nutze `mockito` oder `mocktail` für komplexere Mocks
- **Async Tests**: Verwende `async`/`await` für asynchrone Operationen
- **Test-Ordnerstruktur**: Spiegele die `lib/`-Struktur in `test/` wider

---

## Widget Tests

### Kurze Einführung

Widget Tests testen einzelne Widgets und deren Interaktionen mit dem Benutzer. Sie sind Flutter-spezifisch und simulieren eine abgespeckte Flutter-Umgebung ohne echtes Rendering.

### Wann nutzt man Widget Tests?

**Verwende Widget Tests für:**
- Einzelne Custom Widgets
- Widget-Verhalten bei State-Änderungen
- User-Interaktionen (Taps, Eingaben)
- Widget-Animationen
- Layout-Verhalten bei verschiedenen Bildschirmgrößen

**Nutze Widget Tests NICHT für:**
- Komplette Screens mit vielen Abhängigkeiten
- Navigation zwischen verschiedenen Screens
- Platform-spezifische Features

### Beispiele

#### Einfacher Widget Test

```dart
// test/widgets/counter_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/counter_widget.dart';

void main() {
  group('CounterWidget Tests', () {
    testWidgets('should display initial count', (tester) async {
      // Arrange & Act
      await tester.pumpWidget(
        MaterialApp(
          home: CounterWidget(initialCount: 5),
        ),
      );
      
      // Assert
      expect(find.text('5'), findsOneWidget);
    });
    
    testWidgets('should increment count on button tap', (tester) async {
      // Arrange
      await tester.pumpWidget(
        MaterialApp(
          home: CounterWidget(initialCount: 0),
        ),
      );
      
      // Act
      await tester.tap(find.byIcon(Icons.add));
      await tester.pump(); // Rebuild auslösen
      
      // Assert
      expect(find.text('1'), findsOneWidget);
      expect(find.text('0'), findsNothing);
    });
  });
}
```

#### Widget Test mit State Management

```dart
// test/widgets/user_profile_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:provider/provider.dart';
import 'package:my_app/providers/user_provider.dart';
import 'package:my_app/widgets/user_profile.dart';
import 'package:my_app/models/user.dart';

void main() {
  group('UserProfile Widget Tests', () {
    testWidgets('should display user information', (tester) async {
      // Arrange
      final user = User(name: 'John Doe', email: 'john@example.com');
      final userProvider = UserProvider()..setUser(user);
      
      await tester.pumpWidget(
        MaterialApp(
          home: ChangeNotifierProvider.value(
            value: userProvider,
            child: UserProfile(),
          ),
        ),
      );
      
      // Assert
      expect(find.text('John Doe'), findsOneWidget);
      expect(find.text('john@example.com'), findsOneWidget);
    });
    
    testWidgets('should show edit dialog on edit button tap', (tester) async {
      // Arrange
      final user = User(name: 'John Doe', email: 'john@example.com');
      final userProvider = UserProvider()..setUser(user);
      
      await tester.pumpWidget(
        MaterialApp(
          home: ChangeNotifierProvider.value(
            value: userProvider,
            child: UserProfile(),
          ),
        ),
      );
      
      // Act
      await tester.tap(find.byIcon(Icons.edit));
      await tester.pumpAndSettle(); // Warten bis Animation fertig
      
      // Assert
      expect(find.byType(AlertDialog), findsOneWidget);
      expect(find.text('Edit Profile'), findsOneWidget);
    });
  });
}
```

### Worauf soll man achten?

- **MaterialApp wrapper**: Wickle Widgets in MaterialApp für Theme-Unterstützung
- **pumpAndSettle()**: Verwende für Animationen und asynchrone Operationen
- **Finder**: Nutze verschiedene Finder-Methoden (byType, byKey, text, etc.)
- **Key-Strategien**: Verwende Keys für bessere Testbarkeit
- **MediaQuery**: Teste verschiedene Bildschirmgrößen mit MediaQuery.override
- **Scroll-Verhalten**: Teste Scroll-Widgets mit tester.drag()

### Besonderheiten und Voraussetzungen

- **Pumping**: Verwende `pump()` für einzelne Frames, `pumpAndSettle()` für Animationen
- **Async Handling**: Widget Tests laufen in einer speziellen Test-Umgebung
- **Platform Channels**: Mock Platform Channels für native Funktionalitäten
- **Fonts**: Asset-Fonts müssen in flutter_test konfiguriert werden

---

## Integration Tests

### Kurze Einführung

Integration Tests testen das Zusammenspiel verschiedener App-Bereiche und die komplette User Journey. Sie laufen auf echten Geräten oder Emulatoren und testen die App End-to-End.

### Wann nutzt man Integration Tests?

**Verwende Integration Tests für:**
- Komplette User Flows (Login → Dashboard → Feature)
- Navigation zwischen verschiedenen Screens
- Datenpersistierung und -synchronisation
- Platform-spezifische Features
- Performance-kritische Bereiche
- App-Start und Deep Links

**Nutze Integration Tests NICHT für:**
- Einzelne Business Logic (dafür Unit Tests)
- Einzelne Widget-Verhalten (dafür Widget Tests)
- Zu granulare Testfälle

### Beispiele

#### Einfacher Integration Test

```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  group('App Integration Tests', () {
    testWidgets('complete user login flow', (tester) async {
      // App starten
      app.main();
      await tester.pumpAndSettle();
      
      // Zur Login-Seite navigieren
      expect(find.text('Welcome'), findsOneWidget);
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();
      
      // Login-Daten eingeben
      await tester.enterText(
        find.byKey(Key('email_field')), 
        'test@example.com'
      );
      await tester.enterText(
        find.byKey(Key('password_field')), 
        'password123'
      );
      
      // Login-Button drücken
      await tester.tap(find.byKey(Key('login_button')));
      await tester.pumpAndSettle(Duration(seconds: 3)); // Warten auf API-Call
      
      // Dashboard sollte sichtbar sein
      expect(find.text('Dashboard'), findsOneWidget);
      expect(find.byKey(Key('user_avatar')), findsOneWidget);
    });
    
    testWidgets('navigation through main features', (tester) async {
      // ... (Login-Flow wie oben)
      
      // Feature 1 testen
      await tester.tap(find.text('Settings'));
      await tester.pumpAndSettle();
      expect(find.text('Settings'), findsOneWidget);
      
      // Feature 2 testen
      await tester.tap(find.byIcon(Icons.arrow_back));
      await tester.pumpAndSettle();
      await tester.tap(find.text('Profile'));
      await tester.pumpAndSettle();
      expect(find.text('Profile'), findsOneWidget);
    });
  });
}
```

#### Integration Test mit echter API

```dart
// integration_test/api_integration_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;
import 'package:my_app/services/api_service.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  group('API Integration Tests', () {
    testWidgets('should sync data with backend', (tester) async {
      // App mit echter API-Konfiguration starten
      app.main();
      await tester.pumpAndSettle();
      
      // Login durchführen
      await performLogin(tester);
      
      // Daten erstellen
      await tester.tap(find.byKey(Key('create_item_button')));
      await tester.pumpAndSettle();
      
      await tester.enterText(
        find.byKey(Key('item_title')), 
        'Test Item ${DateTime.now().millisecondsSinceEpoch}'
      );
      
      await tester.tap(find.byKey(Key('save_button')));
      await tester.pumpAndSettle(Duration(seconds: 5)); // Warten auf Sync
      
      // Verifizieren dass Item in Liste erscheint
      expect(find.textContaining('Test Item'), findsOneWidget);
      
      // App neu starten und Daten verifizieren
      await tester.binding.defaultBinaryMessenger.handlePlatformMessage(
        'flutter/platform', 
        null, 
        (data) {}
      );
      
      app.main();
      await tester.pumpAndSettle();
      await performLogin(tester);
      
      // Daten sollten noch da sein
      expect(find.textContaining('Test Item'), findsOneWidget);
    });
  });
}

Future<void> performLogin(WidgetTester tester) async {
  await tester.enterText(find.byKey(Key('email')), 'test@example.com');
  await tester.enterText(find.byKey(Key('password')), 'password');
  await tester.tap(find.byKey(Key('login_button')));
  await tester.pumpAndSettle(Duration(seconds: 3));
}
```

### Worauf soll man achten?

- **Test-Isolation**: Jeder Test sollte mit sauberem Zustand starten
- **Timeouts**: Verwende angemessene Wartezeiten für API-Calls
- **Test-Daten**: Verwende eindeutige Test-Daten um Konflikte zu vermeiden
- **Performance**: Integration Tests sind langsam - halte sie fokussiert
- **Error Handling**: Teste auch Fehlerszenarien (Netzwerk-Ausfall, etc.)
- **Cleanup**: Räume Test-Daten nach Tests auf

### Besonderheiten und Voraussetzungen

- **Package**: `dev_dependencies: integration_test: sdk: flutter`
- **Gerät/Emulator**: Tests laufen auf echten Geräten oder Emulatoren
- **Firebase**: Integration Test Lab für Cloud-Testing
- **Command**: `flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart`
- **CI/CD**: Spezielle Runner-Konfiguration für Emulator/Device-Tests

---
## State Management Tests (BLoC)

### Kurze Einführung

State Management Tests mit BLoC (Business Logic Component) testen die Geschäftslogik und Zustandsübergänge einer App isoliert von der UI. BLoCs verarbeiten Events und emittieren States über Streams, was spezielle Testing-Ansätze erfordert.

### Wann nutzt man BLoC Tests?

**Verwende BLoC Tests für:**
- Geschäftslogik und State-Übergänge
- Event-Processing und State-Emission
- Async-Operationen (API-Calls, Datenbankzugriffe)
- Complex Business Rules und Validierungen
- Stream-basierte Datenverarbeitung
- Error Handling und Recovery-Szenarien

**Nutze BLoC Tests NICHT für:**
- UI-spezifische Logik (dafür Widget Tests)
- Reine Utility-Funktionen (dafür Unit Tests)
- Navigation-Logic (dafür Integration Tests)

### Beispiele

#### Einfacher BLoC Test

```dart
// lib/blocs/counter_bloc.dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';

// Events
abstract class CounterEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class CounterIncremented extends CounterEvent {}
class CounterDecremented extends CounterEvent {}
class CounterReset extends CounterEvent {}

// States
abstract class CounterState extends Equatable {
  @override
  List<Object> get props => [];
}

class CounterInitial extends CounterState {}

class CounterValue extends CounterState {
  final int value;
  
  CounterValue(this.value);
  
  @override
  List<Object> get props => [value];
}

// BLoC
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    on<CounterIncremented>(_onIncremented);
    on<CounterDecremented>(_onDecremented);
    on<CounterReset>(_onReset);
  }
  
  void _onIncremented(CounterIncremented event, Emitter<CounterState> emit) {
    final currentValue = _getCurrentValue();
    emit(CounterValue(currentValue + 1));
  }
  
  void _onDecremented(CounterDecremented event, Emitter<CounterState> emit) {
    final currentValue = _getCurrentValue();
    emit(CounterValue(currentValue - 1));
  }
  
  void _onReset(CounterReset event, Emitter<CounterState> emit) {
    emit(CounterValue(0));
  }
  
  int _getCurrentValue() {
    final currentState = state;
    return currentState is CounterValue ? currentState.value : 0;
  }
}
```

```dart
// test/blocs/counter_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/blocs/counter_bloc.dart';

void main() {
  group('CounterBloc Tests', () {
    late CounterBloc counterBloc;
    
    setUp(() {
      counterBloc = CounterBloc();
    });
    
    tearDown(() {
      counterBloc.close();
    });
    
    test('initial state is CounterInitial', () {
      expect(counterBloc.state, equals(CounterInitial()));
    });
    
    blocTest<CounterBloc, CounterState>(
      'emits [CounterValue(1)] when CounterIncremented is added',
      build: () => counterBloc,
      act: (bloc) => bloc.add(CounterIncremented()),
      expect: () => [CounterValue(1)],
    );
    
    blocTest<CounterBloc, CounterState>(
      'emits [CounterValue(1), CounterValue(2)] when CounterIncremented is added twice',
      build: () => counterBloc,
      act: (bloc) => bloc
        ..add(CounterIncremented())
        ..add(CounterIncremented()),
      expect: () => [CounterValue(1), CounterValue(2)],
    );
    
    blocTest<CounterBloc, CounterState>(
      'emits [CounterValue(1), CounterValue(0)] when incremented then decremented',
      build: () => counterBloc,
      act: (bloc) => bloc
        ..add(CounterIncremented())
        ..add(CounterDecremented()),
      expect: () => [CounterValue(1), CounterValue(0)],
    );
    
    blocTest<CounterBloc, CounterState>(
      'emits [CounterValue(0)] when CounterReset is added',
      build: () => counterBloc,
      seed: () => CounterValue(5),
      act: (bloc) => bloc.add(CounterReset()),
      expect: () => [CounterValue(0)],
    );
  });
}
```

#### BLoC Test mit Repository und Mocking

```dart
// lib/blocs/user_bloc.dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:my_app/models/user.dart';
import 'package:my_app/repositories/user_repository.dart';

// Events
abstract class UserEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class UserLoadRequested extends UserEvent {
  final int userId;
  
  UserLoadRequested(this.userId);
  
  @override
  List<Object> get props => [userId];
}

class UserUpdateRequested extends UserEvent {
  final User user;
  
  UserUpdateRequested(this.user);
  
  @override
  List<Object> get props => [user];
}

// States
abstract class UserState extends Equatable {
  @override
  List<Object> get props => [];
}

class UserInitial extends UserState {}
class UserLoading extends UserState {}
class UserLoaded extends UserState {
  final User user;
  
  UserLoaded(this.user);
  
  @override
  List<Object> get props => [user];
}

class UserError extends UserState {
  final String message;
  
  UserError(this.message);
  
  @override
  List<Object> get props => [message];
}

// BLoC
class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository userRepository;
  
  UserBloc({required this.userRepository}) : super(UserInitial()) {
    on<UserLoadRequested>(_onUserLoadRequested);
    on<UserUpdateRequested>(_onUserUpdateRequested);
  }
  
  Future<void> _onUserLoadRequested(
    UserLoadRequested event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    try {
      final user = await userRepository.getUserById(event.userId);
      emit(UserLoaded(user));
    } catch (error) {
      emit(UserError('Failed to load user: $error'));
    }
  }
  
  Future<void> _onUserUpdateRequested(
    UserUpdateRequested event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    try {
      final updatedUser = await userRepository.updateUser(event.user);
      emit(UserLoaded(updatedUser));
    } catch (error) {
      emit(UserError('Failed to update user: $error'));
    }
  }
}
```

```dart
// test/blocs/user_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:my_app/blocs/user_bloc.dart';
import 'package:my_app/models/user.dart';
import 'package:my_app/repositories/user_repository.dart';

@GenerateMocks([UserRepository])
import 'user_bloc_test.mocks.dart';

void main() {
  group('UserBloc Tests', () {
    late UserBloc userBloc;
    late MockUserRepository mockUserRepository;
    
    setUp(() {
      mockUserRepository = MockUserRepository();
      userBloc = UserBloc(userRepository: mockUserRepository);
    });
    
    tearDown(() {
      userBloc.close();
    });
    
    final testUser = User(id: 1, name: 'John Doe', email: 'john@example.com');
    
    test('initial state is UserInitial', () {
      expect(userBloc.state, equals(UserInitial()));
    });
    
    group('UserLoadRequested', () {
      blocTest<UserBloc, UserState>(
        'emits [UserLoading, UserLoaded] when user is loaded successfully',
        build: () {
          when(mockUserRepository.getUserById(1))
              .thenAnswer((_) async => testUser);
          return userBloc;
        },
        act: (bloc) => bloc.add(UserLoadRequested(1)),
        expect: () => [
          UserLoading(),
          UserLoaded(testUser),
        ],
        verify: (_) {
          verify(mockUserRepository.getUserById(1)).called(1);
        },
      );
      
      blocTest<UserBloc, UserState>(
        'emits [UserLoading, UserError] when loading fails',
        build: () {
          when(mockUserRepository.getUserById(1))
              .thenThrow(Exception('Network error'));
          return userBloc;
        },
        act: (bloc) => bloc.add(UserLoadRequested(1)),
        expect: () => [
          UserLoading(),
          UserError('Failed to load user: Exception: Network error'),
        ],
        verify: (_) {
          verify(mockUserRepository.getUserById(1)).called(1);
        },
      );
    });
    
    group('UserUpdateRequested', () {
      blocTest<UserBloc, UserState>(
        'emits [UserLoading, UserLoaded] when user is updated successfully',
        build: () {
          final updatedUser = testUser.copyWith(name: 'Jane Doe');
          when(mockUserRepository.updateUser(testUser))
              .thenAnswer((_) async => updatedUser);
          return userBloc;
        },
        act: (bloc) => bloc.add(UserUpdateRequested(testUser)),
        expect: () => [
          UserLoading(),
          UserLoaded(testUser.copyWith(name: 'Jane Doe')),
        ],
        verify: (_) {
          verify(mockUserRepository.updateUser(testUser)).called(1);
        },
      );
    });
  });
}
```

#### Komplexer BLoC Test mit Stream-Subscriptions

```dart
// lib/blocs/chat_bloc.dart
import 'dart:async';
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:my_app/models/message.dart';
import 'package:my_app/services/chat_service.dart';

// Events
abstract class ChatEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class ChatStarted extends ChatEvent {
  final String chatId;
  
  ChatStarted(this.chatId);
  
  @override
  List<Object> get props => [chatId];
}

class ChatMessageSent extends ChatEvent {
  final String message;
  
  ChatMessageSent(this.message);
  
  @override
  List<Object> get props => [message];
}

class _ChatMessageReceived extends ChatEvent {
  final Message message;
  
  _ChatMessageReceived(this.message);
  
  @override
  List<Object> get props => [message];
}

// States
abstract class ChatState extends Equatable {
  @override
  List<Object> get props => [];
}

class ChatInitial extends ChatState {}

class ChatConnecting extends ChatState {}

class ChatConnected extends ChatState {
  final List<Message> messages;
  
  ChatConnected(this.messages);
  
  @override
  List<Object> get props => [messages];
}

class ChatError extends ChatState {
  final String message;
  
  ChatError(this.message);
  
  @override
  List<Object> get props => [message];
}

// BLoC
class ChatBloc extends Bloc<ChatEvent, ChatState> {
  final ChatService chatService;
  StreamSubscription<Message>? _messageSubscription;
  String? _currentChatId;
  
  ChatBloc({required this.chatService}) : super(ChatInitial()) {
    on<ChatStarted>(_onChatStarted);
    on<ChatMessageSent>(_onMessageSent);
    on<_ChatMessageReceived>(_onMessageReceived);
  }
  
  Future<void> _onChatStarted(
    ChatStarted event,
    Emitter<ChatState> emit,
  ) async {
    emit(ChatConnecting());
    try {
      _currentChatId = event.chatId;
      final initialMessages = await chatService.getMessages(event.chatId);
      
      _messageSubscription?.cancel();
      _messageSubscription = chatService
          .messageStream(event.chatId)
          .listen((message) => add(_ChatMessageReceived(message)));
      
      emit(ChatConnected(initialMessages));
    } catch (error) {
      emit(ChatError('Failed to connect to chat: $error'));
    }
  }
  
  Future<void> _onMessageSent(
    ChatMessageSent event,
    Emitter<ChatState> emit,
  ) async {
    if (_currentChatId == null) return;
    
    try {
      await chatService.sendMessage(_currentChatId!, event.message);
    } catch (error) {
      emit(ChatError('Failed to send message: $error'));
    }
  }
  
  void _onMessageReceived(
    _ChatMessageReceived event,
    Emitter<ChatState> emit,
  ) {
    final currentState = state;
    if (currentState is ChatConnected) {
      final updatedMessages = [...currentState.messages, event.message];
      emit(ChatConnected(updatedMessages));
    }
  }
  
  @override
  Future<void> close() {
    _messageSubscription?.cancel();
    return super.close();
  }
}
```

```dart
// test/blocs/chat_bloc_test.dart
import 'dart:async';
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:my_app/blocs/chat_bloc.dart';
import 'package:my_app/models/message.dart';
import 'package:my_app/services/chat_service.dart';

@GenerateMocks([ChatService])
import 'chat_bloc_test.mocks.dart';

void main() {
  group('ChatBloc Tests', () {
    late ChatBloc chatBloc;
    late MockChatService mockChatService;
    late StreamController<Message> messageStreamController;
    
    setUp(() {
      mockChatService = MockChatService();
      messageStreamController = StreamController<Message>.broadcast();
      chatBloc = ChatBloc(chatService: mockChatService);
    });
    
    tearDown(() {
      messageStreamController.close();
      chatBloc.close();
    });
    
    final testMessages = [
      Message(id: '1', text: 'Hello', senderId: 'user1', timestamp: DateTime.now()),
      Message(id: '2', text: 'Hi there!', senderId: 'user2', timestamp: DateTime.now()),
    ];
    
    test('initial state is ChatInitial', () {
      expect(chatBloc.state, equals(ChatInitial()));
    });
    
    group('ChatStarted', () {
      blocTest<ChatBloc, ChatState>(
        'emits [ChatConnecting, ChatConnected] when chat connects successfully',
        build: () {
          when(mockChatService.getMessages('chat1'))
              .thenAnswer((_) async => testMessages);
          when(mockChatService.messageStream('chat1'))
              .thenAnswer((_) => messageStreamController.stream);
          return chatBloc;
        },
        act: (bloc) => bloc.add(ChatStarted('chat1')),
        expect: () => [
          ChatConnecting(),
          ChatConnected(testMessages),
        ],
        verify: (_) {
          verify(mockChatService.getMessages('chat1')).called(1);
          verify(mockChatService.messageStream('chat1')).called(1);
        },
      );
      
      blocTest<ChatBloc, ChatState>(
        'emits [ChatConnecting, ChatError] when connection fails',
        build: () {
          when(mockChatService.getMessages('chat1'))
              .thenThrow(Exception('Connection failed'));
          return chatBloc;
        },
        act: (bloc) => bloc.add(ChatStarted('chat1')),
        expect: () => [
          ChatConnecting(),
          ChatError('Failed to connect to chat: Exception: Connection failed'),
        ],
      );
    });
    
    group('Real-time messages', () {
      blocTest<ChatBloc, ChatState>(
        'updates state when new message is received via stream',
        build: () {
          when(mockChatService.getMessages('chat1'))
              .thenAnswer((_) async => testMessages);
          when(mockChatService.messageStream('chat1'))
              .thenAnswer((_) => messageStreamController.stream);
          return chatBloc;
        },
        act: (bloc) async {
          bloc.add(ChatStarted('chat1'));
          await Future.delayed(Duration(milliseconds: 100));
          
          // Simuliere neue Nachricht über Stream
          final newMessage = Message(
            id: '3',
            text: 'New message!',
            senderId: 'user3',
            timestamp: DateTime.now(),
          );
          messageStreamController.add(newMessage);
        },
        expect: () => [
          ChatConnecting(),
          ChatConnected(testMessages),
          ChatConnected([...testMessages, any]), // Neue Nachricht hinzugefügt
        ],
        wait: Duration(milliseconds: 300),
      );
    });
    
    group('ChatMessageSent', () {
      blocTest<ChatBloc, ChatState>(
        'calls sendMessage when message is sent',
        build: () {
          when(mockChatService.getMessages('chat1'))
              .thenAnswer((_) async => testMessages);
          when(mockChatService.messageStream('chat1'))
              .thenAnswer((_) => messageStreamController.stream);
          when(mockChatService.sendMessage('chat1', 'Hello World'))
              .thenAnswer((_) async {});
          return chatBloc;
        },
        seed: () => ChatConnected(testMessages),
        act: (bloc) {
          bloc.add(ChatStarted('chat1')); // Setup chat first
          bloc.add(ChatMessageSent('Hello World'));
        },
        verify: (_) {
          verify(mockChatService.sendMessage('chat1', 'Hello World')).called(1);
        },
      );
    });
  });
}
```

### Worauf soll man achten?

- **bloc_test Package**: Verwende `bloc_test` für strukturierte BLoC-Tests
- **Stream-Management**: Schließe Streams ordnungsgemäß in `tearDown()`
- **Async-Operations**: Verwende `wait` Parameter für zeitbasierte Tests
- **State Equality**: Implementiere `Equatable` für korrekte State-Vergleiche
- **Mocking Dependencies**: Mocke alle externen Abhängigkeiten (Repositories, Services)
- **Event Sequencing**: Teste verschiedene Event-Sequenzen und deren State-Übergänge
- **Error Scenarios**: Teste nicht nur Happy Path, sondern auch Fehlerfälle
- **Memory Leaks**: Stelle sicher, dass BLoCs ordnungsgemäß geschlossen werden

### Besonderheiten und Voraussetzungen

- **Package**: `dev_dependencies: bloc_test: ^9.1.0`
- **Equatable**: Nutze `Equatable` für Events und States
- **Async Testing**: Verwende `async`/`await` für asynchrone BLoC-Operationen
- **Stream Testing**: Teste Stream-Subscriptions mit `StreamController`
- **Timing**: Nutze `wait` und `skip` Parameter für zeitkritische Tests
- **Cleanup**: Implementiere `close()` Methode für Resource-Cleanup

```dart
// pubspec.yaml Dependencies - Prüfe aktuelle Versionen
dependencies:
  flutter_bloc: ^9.1.1
  equatable: ^2.0.7

dev_dependencies:
  bloc_test: ^10.0.0
  mockito: ^5.5.0
  build_runner: ^2.6.1
```

---

## Golden Tests

### Kurze Einführung

Golden Tests (auch Screenshot Tests) vergleichen das gerenderte Aussehen von Widgets mit vorab gespeicherten Referenz-Screenshots. Sie stellen sicher, dass visuelle Änderungen bewusst und erwünscht sind.

### Wann nutzt man Golden Tests?

**Verwende Golden Tests für:**
- Wichtige UI-Komponenten und deren visuelle Konsistenz
- Design System Components
- Responsive Layouts bei verschiedenen Bildschirmgrößen
- Theme-Variationen (Light/Dark Mode)
- Charts, Grafiken und komplexe Layouts
- Regression Testing für UI-Änderungen

**Nutze Golden Tests NICHT für:**
- Hochdynamische Inhalte (aktuelle Zeit, etc.)
- Tests die oft ändern (Prototyping-Phase)
- Platform-spezifische Unterschiede
- Interaktive Elemente (dafür Widget Tests)

### Beispiele

#### Einfacher Golden Test

```dart
// test/golden/button_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/custom_button.dart';

void main() {
  group('CustomButton Golden Tests', () {
    testWidgets('primary button default state', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomButton(
              text: 'Primary Button',
              onPressed: () {},
            ),
          ),
        ),
      );
      
      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('custom_button_primary.png')
      );
    });
    
    testWidgets('disabled button state', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CustomButton(
              text: 'Disabled Button',
              onPressed: null, // disabled
            ),
          ),
        ),
      );
      
      await expectLater(
        find.byType(CustomButton),
        matchesGoldenFile('custom_button_disabled.png')
      );
    });
  });
}
```

#### Golden Test für verschiedene Themes

```dart
// test/golden/theme_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/user_card.dart';
import 'package:my_app/models/user.dart';

void main() {
  group('Theme Golden Tests', () {
    final testUser = User(
      name: 'John Doe',
      email: 'john@example.com',
      avatar: 'assets/test_avatar.png'
    );
    
    testWidgets('user card light theme', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: ThemeData.light(),
          home: Scaffold(
            body: UserCard(user: testUser),
          ),
        ),
      );
      
      await expectLater(
        find.byType(UserCard),
        matchesGoldenFile('user_card_light.png')
      );
    });
    
    testWidgets('user card dark theme', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: ThemeData.dark(),
          home: Scaffold(
            body: UserCard(user: testUser),
          ),
        ),
      );
      
      await expectLater(
        find.byType(UserCard),
        matchesGoldenFile('user_card_dark.png')
      );
    });
  });
}
```

#### Golden Test für responsive Design

```dart
// test/golden/responsive_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/screens/dashboard_screen.dart';

void main() {
  group('Responsive Dashboard Golden Tests', () {
    testWidgets('dashboard mobile view', (tester) async {
      await tester.binding.setSurfaceSize(Size(375, 812)); // iPhone size
      
      await tester.pumpWidget(
        MaterialApp(
          home: DashboardScreen(),
        ),
      );
      
      await expectLater(
        find.byType(DashboardScreen),
        matchesGoldenFile('dashboard_mobile.png')
      );
    });
    
    testWidgets('dashboard tablet view', (tester) async {
      await tester.binding.setSurfaceSize(Size(768, 1024)); // iPad size
      
      await tester.pumpWidget(
        MaterialApp(
          home: DashboardScreen(),
        ),
      );
      
      await expectLater(
        find.byType(DashboardScreen),
        matchesGoldenFile('dashboard_tablet.png')
      );
    });
  });
}
```

### Worauf soll man achten?

- **Deterministische Inhalte**: Verwende feste Test-Daten statt dynamischer Inhalte
- **Font-Konsistenz**: Stelle sicher, dass Test-Fonts verfügbar sind
- **Platform-Unterschiede**: Golden Tests sind platform-spezifisch
- **Update-Workflow**: Nutze `--update-goldens` zum Aktualisieren der Referenz-Bilder
- **CI/CD**: Golden Tests können in verschiedenen Umgebungen unterschiedlich aussehen
- **Dateigröße**: Komprimiere Golden Files für bessere Performance

### Besonderheiten und Voraussetzungen

- **Command zum Updaten**: `flutter test --update-goldens`
- **Ordnerstruktur**: Goldens werden in `test/` gespeichert
- **CI/CD**: Nutze Docker oder spezielle Runner für konsistente Ergebnisse
- **Debugging**: Nutze `debugDisableShadows = true` für konsistente Schatten
- **Fonts**: Lade Test-Fonts in `flutter_test_config.dart`

```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  await loadAppFonts();
  return testMain();
}
```
---

# Asset Fonts in Flutter Tests

## Kurze Einführung

Bei Flutter Tests, insbesondere Golden Tests und Widget Tests, kann es zu Inkonsistenzen kommen, wenn Custom Fonts verwendet werden. Standardmäßig verwenden Tests das System-Font "Ahem", was zu unerwarteten Darstellungen führen kann. Diese Sektion zeigt, wie Asset Fonts korrekt in Tests geladen werden.

## Das Problem

```dart
// Dieses Widget verwendet eine Custom Font
class MyCustomText extends StatelessWidget {
  final String text;
  
  MyCustomText(this.text);
  
  @override
  Widget build(BuildContext context) {
    return Text(
      text,
      style: TextStyle(
        fontFamily: 'Roboto', // Custom Font aus Assets
        fontSize: 16,
      ),
    );
  }
}
```

```dart
// Test ohne Font-Konfiguration - PROBLEM!
testWidgets('should display custom text', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: MyCustomText('Hello World'),
    ),
  );
  
  await expectLater(
    find.byType(MyCustomText),
    matchesGoldenFile('custom_text.png') // Wird fehlschlagen!
  );
});
```

## Lösung 1: flutter_test_config.dart (Empfohlen)

Die eleganteste Lösung ist eine globale Test-Konfiguration:

```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  // Lade alle Asset-Fonts vor den Tests
  await loadAppFonts();
  return testMain();
}

Future<void> loadAppFonts() async {
  // Lade spezifische Fonts
  final robotoRegular = rootBundle.load('assets/fonts/Roboto-Regular.ttf');
  final robotoBold = rootBundle.load('assets/fonts/Roboto-Bold.ttf');
  final openSans = rootBundle.load('assets/fonts/OpenSans-Regular.ttf');
  
  final fontLoader = FontLoader('Roboto')
    ..addFont(robotoRegular)
    ..addFont(robotoBold);
  
  final openSansLoader = FontLoader('OpenSans')
    ..addFont(openSans);
  
  await Future.wait([
    fontLoader.load(),
    openSansLoader.load(),
  ]);
}
```

## Lösung 2: Per-Test Font Loading

Für spezielle Testfälle oder wenn nur einzelne Tests Custom Fonts benötigen:

```dart
// test/widgets/custom_text_test.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/custom_text.dart';

void main() {
  group('CustomText Widget Tests', () {
    setUpAll(() async {
      // Lade Fonts vor allen Tests in dieser Gruppe
      await loadCustomFonts();
    });
    
    testWidgets('should display text with custom font', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: MyCustomText('Hello Custom Font!'),
          ),
        ),
      );
      
      // Font ist jetzt korrekt geladen
      expect(find.text('Hello Custom Font!'), findsOneWidget);
      
      await expectLater(
        find.byType(MyCustomText),
        matchesGoldenFile('custom_text_with_font.png')
      );
    });
  });
}

Future<void> loadCustomFonts() async {
  final fontData = rootBundle.load('assets/fonts/Roboto-Regular.ttf');
  final fontLoader = FontLoader('Roboto')..addFont(fontData);
  await fontLoader.load();
}
```

## Lösung 3: Font-Family Fallback

Alternative Lösung durch Fallback-Fonts in Tests:

```dart
// test/widgets/text_with_fallback_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/custom_text.dart';

void main() {
  testWidgets('should work with fallback fonts', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        theme: ThemeData(
          // Fallback-Font für Tests definieren
          fontFamily: 'packages/flutter_test/fonts/MaterialIcons-Regular.otf',
        ),
        home: Scaffold(
          body: Text(
            'Test Text',
            style: TextStyle(
              fontFamily: 'Roboto', // Fallback zu Theme-Font
            ),
          ),
        ),
      ),
    );
    
    expect(find.text('Test Text'), findsOneWidget);
  });
}
```

## Erweiterte Font-Konfiguration

Für komplexere Szenarien mit mehreren Font-Varianten:

```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  await loadAllAppFonts();
  return testMain();
}

Future<void> loadAllAppFonts() async {
  // Roboto Font Family mit verschiedenen Gewichten
  final robotoFontLoader = FontLoader('Roboto');
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Thin.ttf'));
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Light.ttf'));
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Regular.ttf'));
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Medium.ttf'));
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Bold.ttf'));
  robotoFontLoader.addFont(rootBundle.load('assets/fonts/Roboto-Black.ttf'));
  
  // OpenSans Font Family
  final openSansFontLoader = FontLoader('OpenSans');
  openSansFontLoader.addFont(rootBundle.load('assets/fonts/OpenSans-Regular.ttf'));
  openSansFontLoader.addFont(rootBundle.load('assets/fonts/OpenSans-Bold.ttf'));
  openSansFontLoader.addFont(rootBundle.load('assets/fonts/OpenSans-Italic.ttf'));
  
  // Custom Icon Font
  final iconFontLoader = FontLoader('CustomIcons');
  iconFontLoader.addFont(rootBundle.load('assets/fonts/CustomIcons.ttf'));
  
  // Alle Fonts parallel laden
  await Future.wait([
    robotoFontLoader.load(),
    openSansFontLoader.load(),
    iconFontLoader.load(),
  ]);
  
  print('✅ All custom fonts loaded for tests');
}
```

## Golden Tests mit Fonts

Spezielle Berücksichtigung für Golden Tests:

```dart
// test/golden/typography_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/typography_showcase.dart';

void main() {
  group('Typography Golden Tests', () {
    setUpAll(() async {
      // Fonts müssen vor Golden Tests geladen sein
      await loadTypographyFonts();
    });
    
    testWidgets('typography showcase with all font weights', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            backgroundColor: Colors.white, // Konsistenter Hintergrund
            body: TypographyShowcase(),
          ),
        ),
      );
      
      // Stelle sicher, dass alle Fonts gerendert sind
      await tester.pumpAndSettle();
      
      await expectLater(
        find.byType(TypographyShowcase),
        matchesGoldenFile('typography_showcase.png')
      );
    });
    
    testWidgets('dark theme typography', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: ThemeData.dark(),
          home: Scaffold(
            body: TypographyShowcase(),
          ),
        ),
      );
      
      await tester.pumpAndSettle();
      
      await expectLater(
        find.byType(TypographyShowcase),
        matchesGoldenFile('typography_showcase_dark.png')
      );
    });
  });
}

Future<void> loadTypographyFonts() async {
  // Lade alle für Typography benötigten Fonts
  final fontLoader = FontLoader('Roboto')
    ..addFont(rootBundle.load('assets/fonts/Roboto-Regular.ttf'))
    ..addFont(rootBundle.load('assets/fonts/Roboto-Bold.ttf'))
    ..addFont(rootBundle.load('assets/fonts/Roboto-Light.ttf'));
  
  await fontLoader.load();
}
```

## pubspec.yaml Konfiguration

Stelle sicher, dass die Fonts korrekt in pubspec.yaml definiert sind:

```yaml
flutter:
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Thin.ttf
          weight: 100
        - asset: assets/fonts/Roboto-Light.ttf
          weight: 300
        - asset: assets/fonts/Roboto-Regular.ttf
          weight: 400
        - asset: assets/fonts/Roboto-Medium.ttf
          weight: 500
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
        - asset: assets/fonts/Roboto-Black.ttf
          weight: 900
    
    - family: OpenSans
      fonts:
        - asset: assets/fonts/OpenSans-Regular.ttf
          weight: 400
        - asset: assets/fonts/OpenSans-Bold.ttf
          weight: 700
        - asset: assets/fonts/OpenSans-Italic.ttf
          style: italic
```

## Troubleshooting

**Problem**: Fonts werden in Tests nicht angezeigt
```dart
// Lösung: Prüfe ob Font korrekt geladen wurde
setUpAll(() async {
  await loadCustomFonts();
  
  // Debug: Liste alle geladenen Fonts
  final fontManifest = await rootBundle.loadString('FontManifest.json');
  print('Available fonts: $fontManifest');
});
```

**Problem**: Golden Tests schlagen auf CI/CD fehl
```dart
// Lösung: Verwende konsistente Font-Loading Strategie
Future<void> loadAppFonts() async {
  // Immer alle Fonts laden, auch wenn nicht verwendet
  final fonts = [
    'Roboto-Regular.ttf',
    'Roboto-Bold.ttf',
    'OpenSans-Regular.ttf',
  ];
  
  for (final font in fonts) {
    try {
      final fontLoader = FontLoader(font.split('-').first)
        ..addFont(rootBundle.load('assets/fonts/$font'));
      await fontLoader.load();
    } catch (e) {
      print('Warning: Could not load font $font: $e');
    }
  }
}
```

## Best Practices

1. **Globale Konfiguration**: Verwende `flutter_test_config.dart` für app-weite Font-Loading
2. **Konsistenz**: Lade immer dieselben Fonts in derselben Reihenfolge
3. **Fehlerbehandlung**: Implementiere Fallbacks für fehlende Fonts
4. **Performance**: Lade Fonts nur einmal pro Test-Suite
5. **CI/CD**: Teste Font-Loading in verschiedenen Umgebungen
6. **Documentation**: Dokumentiere welche Fonts für Tests benötigt werden

---

## Fazit

### Zusammenfassung

Testing in Flutter und Dart ist ein vielschichtiges Thema, das verschiedene Ansätze für verschiedene Anforderungen bietet:

- **Unit Tests** bilden das Fundament und testen isolierte Geschäftslogik
- **Widget Tests** stellen sicher, dass UI-Komponenten korrekt funktionieren
- **Integration Tests** verifizieren komplette User Journeys
- **Golden Tests** bewachen die visuelle Konsistenz der Anwendung

Die **Test-Pyramide** sollte als Richtlinie dienen: Viele schnelle Unit Tests, moderate Anzahl an Widget Tests, wenige aber wichtige Integration Tests.

**Wichtige Prinzipien:**
- Tests sollen **schnell, isoliert und deterministisch** sein
- **TDD/BDD** helfen bei der strukturierten Entwicklung
- **Mocking** ermöglicht isolierte Tests
- **Continuous Testing** sollte Teil der Entwicklungskultur sein

### Weiterführende Themen

**Erweiterte Testing-Konzepte:**
- **Test Coverage**: Nutze `flutter test --coverage` um Coverage zu messen
- **Performance Testing**: Verwende `flutter driver` für Performance-Messungen
- **Accessibility Testing**: Teste mit `semantics` und Screen Readern
- **Flavor Testing**: Teste verschiedene App-Varianten (dev, staging, prod)

**Tools und Packages:**
- **Mockito/Mocktail**: Für erweiterte Mocking-Strategien
- **Patrol**: Moderne Alternative zu Integration Tests
- **Maestro**: Cross-Platform E2E Testing
- **Firebase Test Lab**: Cloud-basiertes Testing auf echten Geräten

**Automatisierung:**
- **CI/CD Pipelines**: GitHub Actions, GitLab CI, Bitrise
- **Test Sharding**: Parallele Ausführung von Tests
- **Automated Golden Updates**: Automatisches Update von Golden Files
- **Test Analytics**: Tracking von Test-Performance und -Stabilität

**Fortgeschrittene Patterns:**
- **Page Object Pattern**: Strukturierung von UI-Tests
- **Robot Pattern**: Abstraktion von Test-Aktionen
- **Test Data Builders**: Flexible Test-Daten-Erstellung
- **Parameterized Tests**: Tests mit verschiedenen Input-Daten

**Monitoring und Debugging:**
- **Test Flakiness Tracking**: Überwachung instabiler Tests
- **Test Execution Analytics**: Analyse von Test-Laufzeiten
- **Debug Tools**: Verwendung von `flutter inspector` in Tests

---

*Dieser Guide bildet die Grundlage für professionelles Testing in Flutter. Erweitere ihn kontinuierlich mit deinen Erfahrungen und den Anforderungen deines Teams.*