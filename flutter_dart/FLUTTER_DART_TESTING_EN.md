# Sprache/Language : [DE](FLUTTER_DART_TESTING.md) | [EN](FLUTTER_DART_TESTING_EN.md)

# Flutter & Dart Testing Guide

A guide for testing Flutter applications with Dart

## Table of Contents

1. [General Fundamentals](#general-fundamentals)
2. [Unit Tests](#unit-tests)
3. [Widget Tests](#widget-tests)
4. [State Management Tests (BLoC)](#state-management-tests-bloc)
5. [Integration Tests](#integration-tests)
6. [BloC Tests](#state-management-tests-bloc)
7. [Golden Tests](#golden-tests)
8. [Asset Fonts in Flutter Tests](#asset-fonts-in-flutter-tests)
9. [Conclusion](#conclusion)

---


## General Fundamentals

### Why Tests at All?

Tests are essential for software development for several reasons:

- **Quality Assurance**: Tests catch bugs early before they go into production
- **Refactoring Safety**: Code can be safely restructured without fear of breaking changes
- **Documentation**: Tests document the expected behavior of the software
- **Maintainability**: Well-tested code is easier to maintain and extend
- **Confidence**: Teams can develop new features with more confidence
- **Cost Savings**: Finding bugs in early phases is significantly cheaper than in production

### Test-Driven Development (TDD) and Behavior-Driven Development (BDD)

#### Test-Driven Development (TDD)

TDD follows the **Red-Green-Refactor cycle**:

1. **Red**: Write a failing test
2. **Green**: Write the minimal necessary code to make the test pass
3. **Refactor**: Improve the code without changing functionality

```dart
// Example TDD cycle
// 1. RED: Write test
test('should calculate area of rectangle', () {
  final calculator = AreaCalculator();
  expect(calculator.rectangleArea(5, 3), equals(15));
});

// 2. GREEN: Minimal implementation
class AreaCalculator {
  double rectangleArea(double width, double height) {
    return width * height;
  }
}

// 3. REFACTOR: Improve code (if necessary)
```

#### Behavior-Driven Development (BDD)

BDD extends TDD with business perspectives and uses natural language:

- **Given**: Initial situation
- **When**: Action
- **Then**: Expected result

```dart
// BDD style in Dart
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

### Differences Between Test Types

**Test Pyramid**: The ideal distribution of tests follows a pyramid structure:

```
       /\
      /  \     Integration Tests (few, slow, expensive)
     /    \
    /______\   Widget Tests (moderate number, medium speed)
   /        \
  /__________\  Unit Tests (many, fast, cheap)
```

- **Unit Tests**: Test isolated functions, methods, or classes
- **Widget Tests**: Test individual widgets and their interactions
- **Integration Tests**: Test the interplay of complete app areas
- **Golden Tests**: Test the visual appearance of widgets

---

## Unit Tests

### Brief Introduction

Unit tests are the foundation of any test suite. They test the smallest isolated units of code such as individual functions, methods, or classes. In Flutter, we use the `test` package for unit tests.

### When to Use Unit Tests?

**Use Unit Tests for:**
- Business logic and algorithms
- Data models and their methods
- Utilities and helper functions
- Service classes (e.g., API clients)
- State management (Blocs, Providers, etc.)

**Do NOT Use Unit Tests for:**
- UI components (use Widget Tests for that)
- Database operations (use Integration Tests for that)
- Navigation between screens

### Examples

#### Simple Unit Test

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

#### Service Test with Mocking

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

### What to Pay Attention To?

- **AAA Pattern**: Arrange, Act, Assert - structure tests clearly
- **One Test, One Concept**: Each test should only test one thing
- **Meaningful Names**: Test names should describe the expected behavior
- **Isolation**: Tests should be able to run independently of each other
- **Mocking**: Use mocks for external dependencies
- **Edge Cases**: Test not only the happy path, but also error cases

### Special Features and Prerequisites

- **Package**: `dev_dependencies: flutter_test: sdk: flutter`
- **Mocking**: Use `mockito` or `mocktail` for more complex mocks
- **Async Tests**: Use `async`/`await` for asynchronous operations
- **Test Folder Structure**: Mirror the `lib/` structure in `test/`

---

## Widget Tests

### Brief Introduction

Widget tests test individual widgets and their interactions with the user. They are Flutter-specific and simulate a lightweight Flutter environment without actual rendering.

### When to Use Widget Tests?

**Use Widget Tests for:**
- Individual custom widgets
- Widget behavior during state changes
- User interactions (taps, inputs)
- Widget animations
- Layout behavior on different screen sizes

**Do NOT Use Widget Tests for:**
- Complete screens with many dependencies
- Navigation between different screens
- Platform-specific features

### Examples

#### Simple Widget Test

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
      await tester.pump(); // Trigger rebuild
      
      // Assert
      expect(find.text('1'), findsOneWidget);
      expect(find.text('0'), findsNothing);
    });
  });
}
```

#### Widget Test with State Management

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
      await tester.pumpAndSettle(); // Wait until animation is finished
      
      // Assert
      expect(find.byType(AlertDialog), findsOneWidget);
      expect(find.text('Edit Profile'), findsOneWidget);
    });
  });
}
```

### What to Pay Attention To?

- **MaterialApp wrapper**: Wrap widgets in MaterialApp for theme support
- **pumpAndSettle()**: Use for animations and asynchronous operations
- **Finder**: Use different finder methods (byType, byKey, text, etc.)
- **Key Strategies**: Use keys for better testability
- **MediaQuery**: Test different screen sizes with MediaQuery.override
- **Scroll Behavior**: Test scroll widgets with tester.drag()

### Special Features and Prerequisites

- **Pumping**: Use `pump()` for single frames, `pumpAndSettle()` for animations
- **Async Handling**: Widget tests run in a special test environment
- **Platform Channels**: Mock platform channels for native functionalities
- **Fonts**: Asset fonts must be configured in flutter_test

---

## Integration Tests

### Brief Introduction

Integration tests test the interplay of different app areas and the complete user journey. They run on real devices or emulators and test the app end-to-end.

### When to Use Integration Tests?

**Use Integration Tests for:**
- Complete user flows (Login → Dashboard → Feature)
- Navigation between different screens
- Data persistence and synchronization
- Platform-specific features
- Performance-critical areas
- App startup and deep links

**Do NOT Use Integration Tests for:**
- Individual business logic (use Unit Tests for that)
- Individual widget behavior (use Widget Tests for that)
- Too granular test cases

### Examples

#### Simple Integration Test

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
      // Start app
      app.main();
      await tester.pumpAndSettle();
      
      // Navigate to login page
      expect(find.text('Welcome'), findsOneWidget);
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();
      
      // Enter login data
      await tester.enterText(
        find.byKey(Key('email_field')), 
        'test@example.com'
      );
      await tester.enterText(
        find.byKey(Key('password_field')), 
        'password123'
      );
      
      // Press login button
      await tester.tap(find.byKey(Key('login_button')));
      await tester.pumpAndSettle(Duration(seconds: 3)); // Wait for API call
      
      // Dashboard should be visible
      expect(find.text('Dashboard'), findsOneWidget);
      expect(find.byKey(Key('user_avatar')), findsOneWidget);
    });
    
    testWidgets('navigation through main features', (tester) async {
      // ... (Login flow as above)
      
      // Test feature 1
      await tester.tap(find.text('Settings'));
      await tester.pumpAndSettle();
      expect(find.text('Settings'), findsOneWidget);
      
      // Test feature 2
      await tester.tap(find.byIcon(Icons.arrow_back));
      await tester.pumpAndSettle();
      await tester.tap(find.text('Profile'));
      await tester.pumpAndSettle();
      expect(find.text('Profile'), findsOneWidget);
    });
  });
}
```

#### Integration Test with Real API

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
      // Start app with real API configuration
      app.main();
      await tester.pumpAndSettle();
      
      // Perform login
      await performLogin(tester);
      
      // Create data
      await tester.tap(find.byKey(Key('create_item_button')));
      await tester.pumpAndSettle();
      
      await tester.enterText(
        find.byKey(Key('item_title')), 
        'Test Item ${DateTime.now().millisecondsSinceEpoch}'
      );
      
      await tester.tap(find.byKey(Key('save_button')));
      await tester.pumpAndSettle(Duration(seconds: 5)); // Wait for sync
      
      // Verify that item appears in list
      expect(find.textContaining('Test Item'), findsOneWidget);
      
      // Restart app and verify data
      await tester.binding.defaultBinaryMessenger.handlePlatformMessage(
        'flutter/platform', 
        null, 
        (data) {}
      );
      
      app.main();
      await tester.pumpAndSettle();
      await performLogin(tester);
      
      // Data should still be there
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

### What to Pay Attention To?

- **Test Isolation**: Each test should start with a clean state
- **Timeouts**: Use appropriate wait times for API calls
- **Test Data**: Use unique test data to avoid conflicts
- **Performance**: Integration tests are slow - keep them focused
- **Error Handling**: Also test error scenarios (network failure, etc.)
- **Cleanup**: Clean up test data after tests

### Special Features and Prerequisites

- **Package**: `dev_dependencies: integration_test: sdk: flutter`
- **Device/Emulator**: Tests run on real devices or emulators
- **Firebase**: Integration Test Lab for cloud testing
- **Command**: `flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart`
- **CI/CD**: Special runner configuration for emulator/device tests

---
## State Management Tests (BLoC)

### Brief Introduction

State management tests with BLoC (Business Logic Component) test the business logic and state transitions of an app isolated from the UI. BLoCs process events and emit states via streams, which requires special testing approaches.

### When to Use BLoC Tests?

**Use BLoC Tests for:**
- Business logic and state transitions
- Event processing and state emission
- Async operations (API calls, database access)
- Complex business rules and validations
- Stream-based data processing
- Error handling and recovery scenarios

**Do NOT Use BLoC Tests for:**
- UI-specific logic (use Widget Tests for that)
- Pure utility functions (use Unit Tests for that)
- Navigation logic (use Integration Tests for that)

### Examples

#### Simple BLoC Test

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

#### BLoC Test with Repository and Mocking

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

#### Complex BLoC Test with Stream Subscriptions

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
          
          // Simulate new message via stream
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
          ChatConnected([...testMessages, any]), // New message added
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

### What to Pay Attention To?

- **bloc_test Package**: Use `bloc_test` for structured BLoC tests
- **Stream Management**: Close streams properly in `tearDown()`
- **Async Operations**: Use `wait` parameter for time-based tests
- **State Equality**: Implement `Equatable` for correct state comparisons
- **Mocking Dependencies**: Mock all external dependencies (Repositories, Services)
- **Event Sequencing**: Test different event sequences and their state transitions
- **Error Scenarios**: Test not only happy path, but also error cases
- **Memory Leaks**: Ensure BLoCs are properly closed

### Special Features and Prerequisites

- **Package**: `dev_dependencies: bloc_test: ^9.1.0`
- **Equatable**: Use `Equatable` for events and states
- **Async Testing**: Use `async`/`await` for asynchronous BLoC operations
- **Stream Testing**: Test stream subscriptions with `StreamController`
- **Timing**: Use `wait` and `skip` parameters for time-critical tests
- **Cleanup**: Implement `close()` method for resource cleanup

```dart
// pubspec.yaml Dependencies - Check current versions
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

### Brief Introduction

Golden tests (also screenshot tests) compare the rendered appearance of widgets with pre-stored reference screenshots. They ensure that visual changes are deliberate and desired.

### When to Use Golden Tests?

**Use Golden Tests for:**
- Important UI components and their visual consistency
- Design system components
- Responsive layouts on different screen sizes
- Theme variations (Light/Dark Mode)
- Charts, graphics, and complex layouts
- Regression testing for UI changes

**Do NOT Use Golden Tests for:**
- Highly dynamic content (current time, etc.)
- Tests that change frequently (prototyping phase)
- Platform-specific differences
- Interactive elements (use Widget Tests for that)

### Examples

#### Simple Golden Test

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

#### Golden Test for Different Themes

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

#### Golden Test for Responsive Design

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

### What to Pay Attention To?

- **Deterministic Content**: Use fixed test data instead of dynamic content
- **Font Consistency**: Ensure test fonts are available
- **Platform Differences**: Golden tests are platform-specific
- **Update Workflow**: Use `--update-goldens` to update reference images
- **CI/CD**: Golden tests can look different in various environments
- **File Size**: Compress golden files for better performance

### Special Features and Prerequisites

- **Update Command**: `flutter test --update-goldens`
- **Folder Structure**: Goldens are stored in `test/`
- **CI/CD**: Use Docker or special runners for consistent results
- **Debugging**: Use `debugDisableShadows = true` for consistent shadows
- **Fonts**: Load test fonts in `flutter_test_config.dart`

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

## Brief Introduction

With Flutter tests, especially Golden tests and Widget tests, inconsistencies can occur when custom fonts are used. By default, tests use the system font "Ahem", which can lead to unexpected displays. This section shows how to correctly load asset fonts in tests.

## The Problem

```dart
// This widget uses a custom font
class MyCustomText extends StatelessWidget {
  final String text;
  
  MyCustomText(this.text);
  
  @override
  Widget build(BuildContext context) {
    return Text(
      text,
      style: TextStyle(
        fontFamily: 'Roboto', // Custom font from assets
        fontSize: 16,
      ),
    );
  }
}
```

```dart
// Test without font configuration - PROBLEM!
testWidgets('should display custom text', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: MyCustomText('Hello World'),
    ),
  );
  
  await expectLater(
    find.byType(MyCustomText),
    matchesGoldenFile('custom_text.png') // Will fail!
  );
});
```

## Solution 1: flutter_test_config.dart (Recommended)

The most elegant solution is a global test configuration:

```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  // Load all asset fonts before tests
  await loadAppFonts();
  return testMain();
}

Future<void> loadAppFonts() async {
  // Load specific fonts
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

## Solution 2: Per-Test Font Loading

For special test cases or when only individual tests need custom fonts:

```dart
// test/widgets/custom_text_test.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/custom_text.dart';

void main() {
  group('CustomText Widget Tests', () {
    setUpAll(() async {
      // Load fonts before all tests in this group
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
      
      // Font is now correctly loaded
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

## Solution 3: Font-Family Fallback

Alternative solution through fallback fonts in tests:

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
          // Define fallback font for tests
          fontFamily: 'packages/flutter_test/fonts/MaterialIcons-Regular.otf',
        ),
        home: Scaffold(
          body: Text(
            'Test Text',
            style: TextStyle(
              fontFamily: 'Roboto', // Fallback to theme font
            ),
          ),
        ),
      ),
    );
    
    expect(find.text('Test Text'), findsOneWidget);
  });
}
```

## Extended Font Configuration

For more complex scenarios with multiple font variants:

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
  // Roboto Font Family with different weights
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
  
  // Load all fonts in parallel
  await Future.wait([
    robotoFontLoader.load(),
    openSansFontLoader.load(),
    iconFontLoader.load(),
  ]);
  
  print('✅ All custom fonts loaded for tests');
}
```

## Golden Tests with Fonts

Special consideration for Golden tests:

```dart
// test/golden/typography_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/typography_showcase.dart';

void main() {
  group('Typography Golden Tests', () {
    setUpAll(() async {
      // Fonts must be loaded before Golden tests
      await loadTypographyFonts();
    });
    
    testWidgets('typography showcase with all font weights', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            backgroundColor: Colors.white, // Consistent background
            body: TypographyShowcase(),
          ),
        ),
      );
      
      // Ensure all fonts are rendered
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
  // Load all fonts needed for typography
  final fontLoader = FontLoader('Roboto')
    ..addFont(rootBundle.load('assets/fonts/Roboto-Regular.ttf'))
    ..addFont(rootBundle.load('assets/fonts/Roboto-Bold.ttf'))
    ..addFont(rootBundle.load('assets/fonts/Roboto-Light.ttf'));
  
  await fontLoader.load();
}
```

## pubspec.yaml Configuration

Ensure fonts are correctly defined in pubspec.yaml:

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

**Problem**: Fonts are not displayed in tests
```dart
// Solution: Check if font is correctly loaded
setUpAll(() async {
  await loadCustomFonts();
  
  // Debug: List all loaded fonts
  final fontManifest = await rootBundle.loadString('FontManifest.json');
  print('Available fonts: $fontManifest');
});
```

**Problem**: Golden tests fail on CI/CD
```dart
// Solution: Use consistent font loading strategy
Future<void> loadAppFonts() async {
  // Always load all fonts, even if not used
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

1. **Global Configuration**: Use `flutter_test_config.dart` for app-wide font loading
2. **Consistency**: Always load the same fonts in the same order
3. **Error Handling**: Implement fallbacks for missing fonts
4. **Performance**: Load fonts only once per test suite
5. **CI/CD**: Test font loading in different environments
6. **Documentation**: Document which fonts are needed for tests

---

## Conclusion

### Summary

Testing in Flutter and Dart is a multi-layered topic that offers different approaches for different requirements:

- **Unit Tests** form the foundation and test isolated business logic
- **Widget Tests** ensure that UI components function correctly
- **Integration Tests** verify complete user journeys
- **Golden Tests** guard the visual consistency of the application

The **Test Pyramid** should serve as a guideline: Many fast unit tests, moderate number of widget tests, few but important integration tests.

**Important Principles:**
- Tests should be **fast, isolated, and deterministic**
- **TDD/BDD** help with structured development
- **Mocking** enables isolated tests
- **Continuous Testing** should be part of the development culture

### Advanced Topics

**Extended Testing Concepts:**
- **Test Coverage**: Use `flutter test --coverage` to measure coverage
- **Performance Testing**: Use `flutter driver` for performance measurements
- **Accessibility Testing**: Test with `semantics` and screen readers
- **Flavor Testing**: Test different app variants (dev, staging, prod)

**Tools and Packages:**
- **Mockito/Mocktail**: For advanced mocking strategies
- **Patrol**: Modern alternative to integration tests
- **Maestro**: Cross-platform E2E testing
- **Firebase Test Lab**: Cloud-based testing on real devices

**Automation:**
- **CI/CD Pipelines**: GitHub Actions, GitLab CI, Bitrise
- **Test Sharding**: Parallel execution of tests
- **Automated Golden Updates**: Automatic update of golden files
- **Test Analytics**: Tracking test performance and stability

**Advanced Patterns:**
- **Page Object Pattern**: Structuring UI tests
- **Robot Pattern**: Abstraction of test actions
- **Test Data Builders**: Flexible test data creation
- **Parameterized Tests**: Tests with different input data

**Monitoring and Debugging:**
- **Test Flakiness Tracking**: Monitoring unstable tests
- **Test Execution Analytics**: Analysis of test run times
- **Debug Tools**: Using `flutter inspector` in tests

---

## Back to Content:
[Back to Starting Point](../README_EN.md)