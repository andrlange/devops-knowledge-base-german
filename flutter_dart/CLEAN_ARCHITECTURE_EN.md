# Sprache/Language : [DE](CLEAN_ARCHITECTURE.md) | [EN](CLEAN_ARCHITECTURE_EN.md)

# Flutter Clean Architecture & Domain Driven Design Guide

A practical guide for clean and scalable Flutter applications with modern architecture patterns.

## 🏗️ What is Clean Architecture and why should we use it?

Clean Architecture is an architectural pattern developed by Robert C. Martin. It organizes code in concentric circles, where each circle represents a different layer of the software.

### Advantages:
- **Testability**: Business logic is independent of UI and external dependencies
- **Maintainability**: Clear separation of concerns
- **Scalability**: New features can be easily added
- **Flexibility**: UI or data sources can be replaced without changing the core logic

### The four main layers:

```
┌─────────────────────────────────────┐
│            Presentation             │
│       (UI + State Management)       │
├─────────────────────────────────────┤
│            Application              │
│            (Use Cases)              │
├─────────────────────────────────────┤
│              Domain                 │
│      (Entities + Repositories)      │
├─────────────────────────────────────┤
│          Infrastructure             │
│      (Data Sources + Services)      │
└─────────────────────────────────────┘
```

## 🎯 Domain Driven Design (DDD) + Clean Architecture

Domain Driven Design focuses on business logic and helps model complex domains. Combined with Clean Architecture, a powerful structure emerges:

### Domain Layer (Core of the application):

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

// Value Object for type safety
@freezed
class Email with _$Email {
  const factory Email(String value) = _Email;
  
  const Email._();
  
  bool get isValid => RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value);
}

// Repository Interface (Domain defines the contract)
abstract class UserRepository {
  Stream<Result<List<User>>> getUsers();
  Future<Result<User>> getUserById(String id);
  Future<Result<void>> saveUser(User user);
}
```

## 🔄 Layer separation with BLoC Pattern

### 1. Presentation Layer (UI + BLoC)

```dart
// BLoC for State Management
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
// Use Case encapsulates business logic
class GetUsersUseCase {
  final UserRepository _repository;

  GetUsersUseCase(this._repository);

  Stream<Result<List<User>>> call() {
    return _repository.getUsers();
  }
}

// Dependency Injection with get_it
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

## ⚡ Clean error handling with Result Pattern

Inspired by Kotlin's Result<T> for type-safe error handling:

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

// Extension for better usability
extension ResultX<T> on Result<T> {
  bool get isSuccess => this is Success<T>;
  bool get isError => this is Error<T>;
  
  T? get dataOrNull => when(
    success: (data) => data,
    error: (_) => null,
  );
}

// Usage in Repository
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

Freezed is a code generation package that helps create immutable data classes with less boilerplate code.

### What is Freezed?
Freezed automatically generates:
- **Immutable Classes** with `const` constructors
- **copyWith** methods for updates
- **Equality** and `hashCode` implementations
- **toString** methods for debugging
- **Union Types** for different states
- **JSON Serialization** (with json_annotation)

### Why use Freezed?

```dart
// Without Freezed - lots of boilerplate code
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

// With Freezed - clean and concise
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

### Union Types for State Management:

```dart
@freezed
class UserState with _$UserState {
  const factory UserState.initial() = _Initial;
  const factory UserState.loading() = _Loading;
  const factory UserState.loaded(List<User> users) = _Loaded;
  const factory UserState.error(String message) = _Error;
}

// Pattern Matching with when/map
Widget buildUI(UserState state) {
  return state.when(
    initial: () => const SizedBox(),
    loading: () => const CircularProgressIndicator(),
    loaded: (users) => UserList(users: users),
    error: (message) => ErrorWidget(message),
  );
}
```

## 💉 Injectable - Automatic Dependency Injection

Injectable automates dependency registration for get_it and significantly reduces manual setup code.

### What is Injectable?
Injectable uses annotations to provide:
- **Automatic registration** of services and repositories
- **Scoped dependencies** (Singleton, Factory, LazySingleton)
- **Environment-specific** registrations
- **Dependency resolution** at compile time

### Why use Injectable?

```dart
// Without Injectable - manual setup
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

// With Injectable - automatically generated
@InjectableInit()
void configureDependencies() => getIt.init();
```

### Injectable Annotations:

```dart
// Singleton - one instance for the entire app
@singleton
class UserRepository {
  final UserDataSource _dataSource;
  UserRepository(this._dataSource);
}

// LazySingleton - created only on first access
@lazySingleton
class DatabaseService {
  // Created only when needed
}

// Factory - new instance on every getIt() call
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

// Environment-specific registration
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

// Register abstract classes
@injectable
class UserRepositoryImpl implements UserRepository {
  // Implementation
}

// Registration happens automatically as UserRepository
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
  configureDependencies(); // All dependencies automatically registered
  runApp(MyApp());
}
```

### Benefits of Injectable:
- **Fewer errors**: Compile-time verification of dependencies
- **Automatic updates**: New services are automatically registered
- **Environment support**: Different implementations for dev/prod
- **Type safety**: Wrong dependencies are caught at build time


## 🌊 ReactiveX & rxdart

ReactiveX is an API for asynchronous programming with Observable Streams. `rxdart` extends Dart's Streams with powerful operators.

### Why ReactiveX?
- **Composition**: Streams can be combined and transformed
- **Backpressure handling**: Automatic handling of fast data streams
- **Error handling**: Built-in error recovery mechanisms

```dart
// RxDart example with BehaviorSubject
class UserStore {
  final _usersSubject = BehaviorSubject<List<User>>.seeded([]);
  final _loadingSubject = BehaviorSubject<bool>.seeded(false);

  // Public Streams
  Stream<List<User>> get users => _usersSubject.stream;
  Stream<bool> get isLoading => _loadingSubject.stream;
  
  // Combined Streams
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

## 🔗 Event-driven architecture with rxdart

RxDart enables elegant communication between layers:

```dart
// Event Bus for Cross-Layer Communication
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

// Define Events
@freezed
class AppEvent with _$AppEvent {
  const factory AppEvent.userUpdated(User user) = UserUpdated;
  const factory AppEvent.networkStatusChanged(bool isOnline) = NetworkStatusChanged;
}

// In the Infrastructure Layer
class NetworkService {
  NetworkService() {
    _connectivityStream.listen((isConnected) {
      EventBus().emit(AppEvent.networkStatusChanged(isConnected));
    });
  }
}

// In the Application Layer
class UserSyncUseCase {
  UserSyncUseCase() {
    EventBus().on<NetworkStatusChanged>()
      .where((event) => event.isOnline)
      .listen((_) => _syncUsers());
  }
}

// RxDart operators for complex logic
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

## 📁 Project Structure

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

## ✨ Best Practices & Summary

### Do's:
- **Dependency Inversion**: Define abstract interfaces in the Domain Layer
- **Single Responsibility**: Each class has a clearly defined purpose
- **Immutable Entities**: Use `freezed` for immutable data structures
- **Stream Disposal**: Always close streams and subscriptions
- **Error Boundaries**: Handle errors at the appropriate level

### Don'ts:
- Domain Layer should never have Flutter/UI dependencies
- Avoid direct database access in Use Cases
- Too many BehaviorSubjects (Memory Leaks)
- Business logic in Widgets

### Ensuring code quality:

```dart
// Testing with MockTail
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

## 📚 Advanced Topics

### Advanced Patterns:
- **CQRS** (Command Query Responsibility Segregation)
- **Event Sourcing** for complex business logic
- **Saga Pattern** for distributed transactions
- **Repository Pattern** with local and remote data sources

### Performance Optimization:
- **Code Generation** with `build_runner`
- **Lazy Loading** with Streams
- **Memory Management** with RxDart Streams
- **Minimize Widget Rebuilds** with BlocBuilder

### Testing Strategies:
- **Unit Tests** for Use Cases and Entities
- **Widget Tests** for UI Components
- **Integration Tests** for End-to-End Flows
- **Golden Tests** for UI Consistency

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

### **Conclusion**:
Clean Architecture combined with DDD and modern Flutter tools enables the development of scalable and maintainable applications. RxDart and BLoC provide powerful tools for state management and reactive programming.

## Back to Content:
[Back to Starting Point](../README_EN.md)