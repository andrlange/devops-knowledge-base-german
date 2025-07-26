# Flutter Clean Architecture & Domain Driven Design Guide

Ein praktischer Leitfaden für saubere und skalierbare Flutter-Anwendungen mit modernen Architektur-Patterns.

## 🏗️ Was ist Clean Architecture und warum sollten wir sie verwenden?

Clean Architecture ist ein Architekturmuster, das von Robert C. Martin entwickelt wurde. Es organisiert Code in konzentrische Kreise, wobei jeder Kreis eine andere Ebene der Software repräsentiert.

### Vorteile:
- **Testbarkeit**: Business Logic ist unabhängig von UI und externen Dependencies
- **Wartbarkeit**: Klare Trennung der Verantwortlichkeiten
- **Skalierbarkeit**: Neue Features lassen sich einfach hinzufügen
- **Flexibilität**: UI oder Datenquellen können ausgetauscht werden, ohne die Kernlogik zu ändern

### Die vier Hauptschichten:

```
┌─────────────────────────────────────┐
│           Presentation              │
│        (UI + State Management)      │
├─────────────────────────────────────┤
│           Application               │
│           (Use Cases)               │
├─────────────────────────────────────┤
│            Domain                   │
│     (Entities + Repositories)       │
├─────────────────────────────────────┤
│         Infrastructure              │
│     (Data Sources + Services)       │
└─────────────────────────────────────┘
```

## 🎯 Domain Driven Design (DDD) + Clean Architecture

Domain Driven Design fokussiert sich auf die Geschäftslogik und hilft dabei, komplexe Domänen zu modellieren. In Kombination mit Clean Architecture entsteht eine mächtige Struktur:

### Domain Layer (Kern der Anwendung):

```dart
// Domain Entity
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required Email email,
    required DateTime createdAt,
  }) = _User;
}

// Value Object für Typ-Sicherheit
@freezed
class Email with _$Email {
  const factory Email(String value) = _Email;
  
  const Email._();
  
  bool get isValid => RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value);
}

// Repository Interface (Domain definiert den Vertrag)
abstract class UserRepository {
  Stream<Result<List<User>>> getUsers();
  Future<Result<User>> getUserById(String id);
  Future<Result<void>> saveUser(User user);
}
```

## 🔄 Schichtentrennung mit BLoC Pattern

### 1. Presentation Layer (UI + BLoC)

```dart
// BLoC für State Management
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUsersUseCase _getUsersUseCase;
  final StreamSubscription? _usersSubscription;

  UserBloc({required GetUsersUseCase getUsersUseCase}) 
    : _getUsersUseCase = getUsersUseCase,
      super(const UserState.initial()) {
    
    on<UserEvent.loadUsers>(_onLoadUsers);
  }

  Future<void> _onLoadUsers(
    LoadUsers event,
    Emitter<UserState> emit,
  ) async {
    emit(const UserState.loading());
    
    await emit.forEach(
      _getUsersUseCase(),
      onData: (result) => result.when(
        success: (users) => UserState.loaded(users),
        error: (error) => UserState.error(error.message),
      ),
    );
  }
}

// UI Widget
class UserListPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<UserBloc, UserState>(
      builder: (context, state) {
        return state.when(
          initial: () => const SizedBox(),
          loading: () => const CircularProgressIndicator(),
          loaded: (users) => ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) => UserTile(user: users[index]),
          ),
          error: (message) => ErrorWidget(message),
        );
      },
    );
  }
}
```

### 2. Application Layer (Use Cases)

```dart
// Use Case kapselt Geschäftslogik
class GetUsersUseCase {
  final UserRepository _repository;

  GetUsersUseCase(this._repository);

  Stream<Result<List<User>>> call() {
    return _repository.getUsers();
  }
}

// Dependency Injection mit get_it
final getIt = GetIt.instance;

void setupDependencies() {
  // Infrastructure
  getIt.registerLazySingleton<UserRepository>(
    () => UserRepositoryImpl(getIt()),
  );
  
  // Application
  getIt.registerFactory(() => GetUsersUseCase(getIt()));
  
}
```

## ⚡ Saubere Fehlerbehandlung mit Result Pattern

Inspiriert von Kotlins Result<T> für typsichere Fehlerbehandlung:

```dart
@freezed
class Result<T> with _$Result<T> {
  const factory Result.success(T data) = Success<T>;
  const factory Result.error(AppError error) = Error<T>;
}

@freezed
class AppError with _$AppError {
  const factory AppError.network(String message) = NetworkError;
  const factory AppError.validation(String message) = ValidationError;
  const factory AppError.unknown(String message) = UnknownError;
}

// Extension für bessere Usability
extension ResultX<T> on Result<T> {
  bool get isSuccess => this is Success<T>;
  bool get isError => this is Error<T>;
  
  T? get dataOrNull => when(
    success: (data) => data,
    error: (_) => null,
  );
}

// Verwendung in Repository
class UserRepositoryImpl implements UserRepository {
  final UserDataSource _dataSource;

  @override
  Future<Result<User>> getUserById(String id) async {
    try {
      final user = await _dataSource.getUserById(id);
      return Result.success(user);
    } on NetworkException catch (e) {
      return Result.error(AppError.network(e.message));
    } catch (e) {
      return Result.error(AppError.unknown(e.toString()));
    }
  }
}
```

## 🧊 Freezed - Immutable Data Classes

Freezed ist ein Code-Generation Package, das dabei hilft, unveränderliche Datenklassen mit weniger Boilerplate-Code zu erstellen.

### Was ist Freezed?
Freezed generiert automatisch:
- **Immutable Classes** mit `const` Konstruktoren
- **copyWith** Methoden für Updates
- **Equality** und `hashCode` Implementierungen
- **toString** Methoden für Debugging
- **Union Types** für verschiedene Zustände
- **JSON Serialization** (mit json_annotation)

### Warum Freezed verwenden?

```dart
// Ohne Freezed - viel Boilerplate Code
class User {
  final String id;
  final String name;
  final String email;

  const User({required this.id, required this.name, required this.email});

  User copyWith({String? id, String? name, String? email}) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
    );
  }

  @override
  bool operator ==(Object other) {
    if (identical(this, other)) return true;
    return other is User && 
           other.id == id && 
           other.name == name && 
           other.email == email;
  }

  @override
  int get hashCode => Object.hash(id, name, email);

  @override
  String toString() => 'User(id: $id, name: $name, email: $email)';
}

// Mit Freezed - clean und präzise
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required String email,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

### Union Types für State Management:

```dart
@freezed
class UserState with _$UserState {
  const factory UserState.initial() = _Initial;
  const factory UserState.loading() = _Loading;
  const factory UserState.loaded(List<User> users) = _Loaded;
  const factory UserState.error(String message) = _Error;
}

// Pattern Matching mit when/map
Widget buildUI(UserState state) {
  return state.when(
    initial: () => const SizedBox(),
    loading: () => const CircularProgressIndicator(),
    loaded: (users) => UserList(users: users),
    error: (message) => ErrorWidget(message),
  );
}
```

## 💉 Injectable - Automatische Dependency Injection

Injectable automatisiert die Registrierung von Dependencies für get_it und reduziert manuellen Setup-Code erheblich.

### Was ist Injectable?
Injectable verwendet Annotations um:
- **Automatische Registrierung** von Services und Repositories
- **Scoped Dependencies** (Singleton, Factory, LazySingleton)
- **Environment-spezifische** Registrierungen
- **Abhängigkeitsauflösung** zur Compile-Zeit

### Warum Injectable verwenden?

```dart
// Ohne Injectable - manueller Setup
void setupDependencies() {
  // Infrastructure
  getIt.registerLazySingleton<HttpClient>(() => HttpClient());
  getIt.registerLazySingleton<UserDataSource>(() => UserDataSourceImpl(getIt()));
  getIt.registerLazySingleton<UserRepository>(() => UserRepositoryImpl(getIt()));
  
  // Application
  getIt.registerFactory(() => GetUsersUseCase(getIt()));
  getIt.registerFactory(() => CreateUserUseCase(getIt()));
  getIt.registerFactory(() => DeleteUserUseCase(getIt()));
  
  // Presentation
  getIt.registerFactory(() => UserBloc(getIt(), getIt()));
  getIt.registerFactory(() => UserDetailBloc(getIt()));
}

// Mit Injectable - automatisch generiert
@InjectableInit()
void configureDependencies() => getIt.init();
```

### Injectable Annotations:

```dart
// Singleton - eine Instanz für die gesamte App
@singleton
class UserRepository {
  final UserDataSource _dataSource;
  UserRepository(this._dataSource);
}

// LazySingleton - wird erst bei erstem Zugriff erstellt
@lazySingleton
class DatabaseService {
  // Wird nur erstellt wenn benötigt
}

// Factory - neue Instanz bei jedem getIt() Aufruf
@injectable
class GetUsersUseCase {
  final UserRepository _repository;
  GetUsersUseCase(this._repository);
}

// Named Dependencies
@Named('apiUrl')
@injectable
String get apiUrl => 'https://api.example.com';

@injectable
class ApiService {
  ApiService(@Named('apiUrl') String baseUrl);
}

// Environment-spezifische Registrierung
@Environment('dev')
@injectable
class DevUserRepository implements UserRepository {
  // Development Implementation
}

@Environment('prod')
@injectable
class ProdUserRepository implements UserRepository {
  // Production Implementation
}

// Abstract Classes registrieren
@injectable
class UserRepositoryImpl implements UserRepository {
  // Implementation
}

// Registrierung erfolgt automatisch als UserRepository
```

### Setup in main.dart:

```dart
// injection_container.dart
@InjectableInit(
  initializerName: 'init',
  preferRelativeImports: true,
  asExtension: true,
)
void configureDependencies() => getIt.init();

// main.dart
void main() {
  configureDependencies(); // Alle Dependencies automatisch registriert
  runApp(MyApp());
}
```

### Vorteile von Injectable:
- **Weniger Fehler**: Compile-Zeit Überprüfung der Dependencies
- **Automatisches Update**: Neue Services werden automatisch registriert
- **Environment Support**: Verschiedene Implementierungen für dev/prod
- **Type Safety**: Falsche Dependencies werden zur Build-Zeit erkannt


## 🌊 ReactiveX & rxdart

ReactiveX ist ein API für asynchrone Programmierung mit Observable Streams. `rxdart` erweitert Dart's Streams um mächtige Operatoren.

### Warum ReactiveX?
- **Komposition**: Streams können kombiniert und transformiert werden
- **Backpressure Handling**: Automatische Behandlung von schnellen Datenströmen
- **Fehlerbehandlung**: Eingebaute Error-Recovery Mechanismen

```dart
// RxDart Beispiel mit BehaviorSubject
class UserStore {
  final _usersSubject = BehaviorSubject<List<User>>.seeded([]);
  final _loadingSubject = BehaviorSubject<bool>.seeded(false);

  // Public Streams
  Stream<List<User>> get users => _usersSubject.stream;
  Stream<bool> get isLoading => _loadingSubject.stream;
  
  // Kombinierte Streams
  Stream<UserViewState> get viewState => Rx.combineLatest2(
    users,
    isLoading,
    (List<User> users, bool loading) => UserViewState(
      users: users,
      isLoading: loading,
    ),
  );

  void dispose() {
    _usersSubject.close();
    _loadingSubject.close();
  }
}
```

## 🔗 Event-getriebene Architektur mit rxdart

RxDart ermöglicht elegante Kommunikation zwischen den Schichten:

```dart
// Event Bus für Cross-Layer Communication
class EventBus {
  static final _instance = EventBus._internal();
  factory EventBus() => _instance;
  EventBus._internal();

  final _eventSubject = PublishSubject<AppEvent>();
  
  Stream<T> on<T extends AppEvent>() {
    return _eventSubject.stream.whereType<T>();
  }
  
  void emit(AppEvent event) {
    _eventSubject.add(event);
  }
}

// Events definieren
@freezed
class AppEvent with _$AppEvent {
  const factory AppEvent.userUpdated(User user) = UserUpdated;
  const factory AppEvent.networkStatusChanged(bool isOnline) = NetworkStatusChanged;
}

// In der Infrastructure Layer
class NetworkService {
  NetworkService() {
    _connectivityStream.listen((isConnected) {
      EventBus().emit(AppEvent.networkStatusChanged(isConnected));
    });
  }
}

// In der Application Layer
class UserSyncUseCase {
  UserSyncUseCase() {
    EventBus().on<NetworkStatusChanged>()
      .where((event) => event.isOnline)
      .listen((_) => _syncUsers());
  }
}

// RxDart Operatoren für komplexe Logik
class SearchUseCase {
  final UserRepository _repository;
  
  SearchUseCase(this._repository);

  Stream<List<User>> search(Stream<String> queryStream) {
    return queryStream
      .debounceTime(const Duration(milliseconds: 300))
      .where((query) => query.length >= 2)
      .distinctUntilChanged()
      .switchMap((query) => _repository.searchUsers(query))
      .map((result) => result.dataOrNull ?? []);
  }
}
```

## 📁 Projektstruktur

```
lib/
├── core/
│   ├── error/
│   ├── usecase/
│   └── utils/
├── features/
│   └── user/
│       ├── data/
│       │   ├── datasources/
│       │   ├── models/
│       │   └── repositories/
│       ├── domain/
│       │   ├── entities/
│       │   ├── repositories/
│       │   └── usecases/
│       └── presentation/
│           ├── bloc/
│           ├── pages/
│           └── widgets/
└── injection_container.dart
```

## ✨ Best Practices & Zusammenfassung

### Do's:
- **Dependency Inversion**: Abstrakte Interfaces in der Domain Layer definieren
- **Single Responsibility**: Jede Klasse hat eine klar definierte Aufgabe
- **Immutable Entities**: Verwendung von `freezed` für unveränderliche Datenstrukturen
- **Stream Disposal**: Immer Streams und Subscriptions schließen
- **Error Boundaries**: Fehler auf der richtigen Ebene behandeln

### Don'ts:
- Domain Layer sollte niemals Flutter/UI Dependencies haben
- Direkte Datenbankzugriffe in Use Cases vermeiden
- Zu viele BehaviorSubjects (Memory Leaks)
- Business Logic in Widgets

### Code-Qualität sicherstellen:

```dart
// Testing mit MockTail
class MockUserRepository extends Mock implements UserRepository {}

void main() {
  group('GetUsersUseCase', () {
    late MockUserRepository mockRepository;
    late GetUsersUseCase useCase;

    setUp(() {
      mockRepository = MockUserRepository();
      useCase = GetUsersUseCase(mockRepository);
    });

    test('should return users when repository call is successful', () async {
      // Arrange
      final users = [User(id: '1', name: 'Test', email: Email('test@test.com'))];
      when(() => mockRepository.getUsers())
          .thenAnswer((_) => Stream.value(Result.success(users)));

      // Act
      final result = await useCase().first;

      // Assert
      expect(result.isSuccess, true);
      expect(result.dataOrNull, users);
    });
  });
}
```

## 📚 Weiterführende Themen

### Fortgeschrittene Patterns:
- **CQRS** (Command Query Responsibility Segregation)
- **Event Sourcing** für komplexe Geschäftslogik
- **Saga Pattern** für verteilte Transaktionen
- **Repository Pattern** mit lokaler und Remote-Datenquelle

### Performance Optimierung:
- **Code Generation** mit `build_runner`
- **Lazy Loading** mit Streams
- **Memory Management** bei RxDart Streams
- **Widget Rebuilds** minimieren mit BlocBuilder

### Testing Strategien:
- **Unit Tests** für Use Cases und Entities
- **Widget Tests** für UI Komponenten
- **Integration Tests** für End-to-End Flows
- **Golden Tests** für UI Consistency

### Tools & Libraries:
```yaml
dependencies:
  # State Management
  flutter_bloc: ^9.1.1
  
  # Reactive Programming
  rxdart: ^0.28.0
  
  # Code Generation
  freezed: ^3.2.0
  json_annotation: ^4.9.0
  
  # Dependency Injection
  get_it: ^8.0.3
  injectable: ^2.5.0
  
dev_dependencies:
  # Code Generation
  build_runner: ^2.6.0
  freezed: ^3.2.0
  json_serializable: ^6.10.0
  injectable_generator: ^2.7.0
  
  # Testing
  mocktail: ^1.0.4
  bloc_test: ^10.0.0
```

---

### **Fazit**: 
Clean Architecture kombiniert mit DDD und modernen Flutter-Tools ermöglicht es, skalierbare und wartbare Anwendungen zu entwickeln. RxDart und BLoC bieten mächtige Werkzeuge für State Management und reaktive Programmierung.

## Zurück zum Inhalt:
[Zurück zum Startpunkt](../README.md)
