# Login Screen

## Purpose

The Login Screen allows existing users to authenticate and access the application. It serves as the primary entry point for returning users and provides options for account creation and password recovery.

## UI Components

```text
┌────────────────────────────┐
│                            │
│           [LOGO]           │
│                            │
│  ┌────────────────────┐    │
│  │    Phone Number    │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │      Password      │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │        Login       │    │
│  └────────────────────┘    │
│                            │
│  Forgot Password?          │
│                            │
│  Don't have an account?    │
│  Register Now              │
│                            │
└────────────────────────────┘
```

### Components

- **App Logo**: Centered brand logo
- **Phone Number Input**: Text field for entering phone number with country code
- **Password Input**: Secure text field with visibility toggle
- **Login Button**: Primary action button
- **Forgot Password Link**: Text button for password recovery
- **Register Link**: Text button for new account creation
- **Error Messages**: Validation and authentication error display area

## User Interactions

- Enter phone number with proper validation
- Enter password with visibility toggle option
- Submit login form
- Navigate to forgot password flow
- Navigate to registration screen
- Form validation feedback
- Authentication error handling

## State Management

### State Provider

The screen uses `LoginController` with Provider for state management:

```dart
class LoginController extends ChangeNotifier {
  final TextEditingController phoneController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();
  
  bool _isLoading = false;
  bool _passwordVisible = false;
  String? _phoneError;
  String? _passwordError;
  String? _authError;
  
  bool get isLoading => _isLoading;
  bool get passwordVisible => _passwordVisible;
  String? get phoneError => _phoneError;
  String? get passwordError => _passwordError;
  String? get authError => _authError;
  
  void togglePasswordVisibility() {
    _passwordVisible = !_passwordVisible;
    notifyListeners();
  }
  
  Future<bool> login() async {
    if (!_validateForm()) return false;
    
    _isLoading = true;
    _authError = null;
    notifyListeners();
    
    try {
      await authService.signInWithPhoneAndPassword(
        phoneController.text,
        passwordController.text
      );
      _isLoading = false;
      notifyListeners();
      return true;
    } catch (e) {
      _isLoading = false;
      _authError = _mapAuthErrorToMessage(e);
      notifyListeners();
      return false;
    }
  }
  
  bool _validateForm() {
    bool isValid = true;
    
    // Phone validation
    if (phoneController.text.isEmpty) {
      _phoneError = 'Phone number is required';
      isValid = false;
    } else if (!_isValidPhoneNumber(phoneController.text)) {
      _phoneError = 'Enter a valid phone number';
      isValid = false;
    } else {
      _phoneError = null;
    }
    
    // Password validation
    if (passwordController.text.isEmpty) {
      _passwordError = 'Password is required';
      isValid = false;
    } else if (passwordController.text.length < 6) {
      _passwordError = 'Password must be at least 6 characters';
      isValid = false;
    } else {
      _passwordError = null;
    }
    
    notifyListeners();
    return isValid;
  }
  
  String _mapAuthErrorToMessage(dynamic error) {
    // Map Firebase auth errors to user-friendly messages
    if (error is FirebaseAuthException) {
      switch (error.code) {
        case 'user-not-found':
          return 'No user found with this phone number';
        case 'wrong-password':
          return 'Incorrect password';
        case 'invalid-credential':
          return 'Invalid login credentials';
        case 'too-many-requests':
          return 'Too many failed login attempts. Try again later';
        default:
          return 'Authentication failed: ${error.message}';
      }
    }
    return 'An error occurred during login';
  }
  
  @override
  void dispose() {
    phoneController.dispose();
    passwordController.dispose();
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `login`
- **Route Path**: `/login`

### Navigation Logic

```dart
// From Login Screen to Registration Screen
void _navigateToRegistration(BuildContext context) {
  context.push('/register');
}

// From Login Screen to Forgot Password
void _navigateToForgotPassword(BuildContext context) {
  context.push('/forgot-password');
}

// On successful login
void _onLoginSuccess(BuildContext context) {
  context.go('/chats');
}
```

## Data Flow

### Input Data

- Phone Number: User's registered phone number with country code
- Password: User's account password

### Authentication Process

1. Validate input fields
2. Submit credentials to Firebase Authentication
3. Handle authentication response
4. Store authentication token securely
5. Navigate to main app screen on success

### Services Used

- AuthService (Firebase Authentication)
- SecureStorageService (for token storage)
- AnalyticsService (for login event tracking)

## Error Handling

- Field validation errors with inline feedback
- Authentication errors with user-friendly messages
- Network connectivity errors
- Rate limiting detection and feedback
- Account status issues (disabled, deleted)

## Offline Support

- Detects offline status and shows appropriate messages
- Caches form inputs for retry when connectivity returns
- Prevents submission attempts when offline

## Dependencies

### External Packages

- `firebase_auth`: Authentication services
- `provider`: State management
- `flutter_secure_storage`: Secure token storage
- `intl_phone_field`: International phone input with validation
- `connectivity_plus`: Network connectivity detection
- `go_router`: Navigation and routing

### Services

- `AuthService`: Handles authentication logic
- `SecureStorageService`: Manages secure token storage
- `AnalyticsService`: Tracks authentication events
- `ConnectivityService`: Monitors network status

## Design Notes

### UX Considerations

- Clear error messages with actionable feedback
- Loading indicators during authentication process
- Focus management for form fields
- Keyboard handling (next/done actions)
- Save form state during screen rotation

### Accessibility

- Semantic labels for all input fields
- Error messages announced by screen readers
- Sufficient contrast for text elements
- Support for large text sizes
- Tab navigation support for keyboard users
