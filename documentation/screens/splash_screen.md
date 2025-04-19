# Splash Screen

## Purpose

The Splash Screen serves as the initial entry point to the application, displaying while the app checks authentication status and initializes necessary services. It provides visual brand identity during app loading and handles startup logic.

## UI Components

```text
┌────────────────────────────┐
│                            │
│                            │
│                            │
│                            │
│           [LOGO]           │
│                            │
│       WhatsApp Clone       │
│                            │
│                            │
│                            │
│                            │
│        Loading...          │
│                            │
└────────────────────────────┘
```

### Components

- **App Logo**: Centered brand logo
- **Application Name**: Text displaying app name
- **Loading Indicator**: Circular progress indicator showing loading status

## User Interactions

- No direct user interactions
- Automatic redirection after initialization completes

## State Management

### State Provider

The screen uses a `SplashController` with Provider for simple state management:

```dart
class SplashController extends ChangeNotifier {
  bool _isInitializing = true;
  bool _hasError = false;
  String _errorMessage = '';

  bool get isInitializing => _isInitializing;
  bool get hasError => _hasError;
  String get errorMessage => _errorMessage;

  Future<void> initialize() async {
    try {
      await _initializeServices();
      _isInitializing = false;
      notifyListeners();
    } catch (e) {
      _hasError = true;
      _errorMessage = e.toString();
      notifyListeners();
    }
  }

  Future<void> _initializeServices() async {
    // Initialize core services
    await Future.wait([
      _initializeFirebase(),
      _initializeLocalStorage(),
      _checkConnectivity()
    ]);
  }
  
  // Service initialization methods...
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `/` (Root route)
- **Route Path**: `/splash`

### Navigation Logic

```dart
void _handleNavigation(BuildContext context) {
  final authService = Provider.of<AuthService>(context, listen: false);
  
  if (authService.isAuthenticated) {
    context.go('/chats');
  } else {
    context.go('/login');
  }
}
```

## Data Flow

### Initialization Process

1. Initialize Firebase services (Auth, Firestore, etc.)
2. Initialize local storage (Hive)
3. Check network connectivity
4. Verify authentication status
5. Load essential configuration data
6. Navigate to appropriate screen

### Services Used

- AuthService (Firebase Authentication)
- HiveService (Local storage)
- ConnectivityService (Network status)

## Error Handling

- Catches initialization errors and displays appropriate messages
- Provides retry mechanism for critical service failures
- Logs errors to analytics service
- Falls back to login screen if authentication cannot be determined

## Offline Support

- Detects initial network status
- Can proceed to appropriate screens even without connectivity
- Loads cached authentication state from local storage

## Dependencies

### External Packages

- `firebase_core`: Firebase initialization
- `firebase_auth`: Authentication services
- `hive_flutter`: Local storage initialization
- `connectivity_plus`: Network connectivity detection
- `flutter_native_splash`: Native splash screen configuration
- `go_router`: Navigation and routing

### Services

- `AuthService`: Handles user authentication state
- `HiveService`: Manages local storage initialization
- `ConnectivityService`: Monitors network status
- `AnalyticsService`: Tracks app launch events

## Design Notes

### UX Considerations

- Displays for minimum of 2 seconds to prevent flickering
- Provides visual feedback during initialization
- Uses brand colors and imagery for recognition
- Minimalist design to optimize loading performance

### Accessibility

- Logo includes semantic label for screen readers
- High contrast between text and background
- Loading indicator has appropriate semantic meaning
