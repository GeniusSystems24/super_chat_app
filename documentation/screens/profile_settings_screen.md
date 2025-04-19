# Profile & Settings Screen

## Purpose

The Profile & Settings Screen allows users to view and edit their profile information, manage application preferences, and access various configuration options. It serves as the central hub for user account management and application customization.

## UI Components

```text
┌────────────────────────────┐
│ ← Profile                  │
│                            │
│        [PROFILE PHOTO]     │
│           John Doe         │
│       +1 (555) 123-4567    │
│                            │
│ ┌────────────────────────┐ │
│ │ Account                │>│
│ │ Security & privacy     │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Appearance             │>│
│ │ Dark mode, wallpapers  │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Chat settings          │>│
│ │ Messages, media, storage│ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Notifications          │>│
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Help                   │>│
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Log out                │ │
│ └────────────────────────┘ │
│                            │
└────────────────────────────┘
```

### Components

- **App Bar**: Contains back button and screen title
- **Profile Header**:
  - Profile photo with edit option
  - User name
  - Phone number
- **Settings Sections**:
  - Account settings
  - Security & privacy settings
  - Appearance settings
  - Chat settings
  - Notification settings
  - Help & support
  - Log out button

## User Interactions

- Tap on profile photo to view/change profile picture
- Tap on name/phone to edit profile information
- Tap on settings categories to navigate to specific settings screens
- Toggle switches for binary settings (e.g., dark mode)
- Tap on "Log out" to sign out of the account
- Pull to refresh profile data

## State Management

### State Provider

The screen uses `ProfileSettingsController` with Provider for state management:

```dart
class ProfileSettingsController extends ChangeNotifier {
  final UserRepository _userRepository;
  final AuthService _authService;
  final PreferencesService _preferencesService;
  final StorageService _storageService;
  
  UserProfile? _userProfile;
  bool _isLoading = false;
  bool _isSaving = false;
  bool _hasError = false;
  String _errorMessage = '';
  
  UserProfile? get userProfile => _userProfile;
  bool get isLoading => _isLoading;
  bool get isSaving => _isSaving;
  bool get hasError => _hasError;
  String get errorMessage => _errorMessage;
  
  // App preferences
  bool get isDarkMode => _preferencesService.isDarkMode;
  String get chatWallpaper => _preferencesService.chatWallpaper;
  bool get messageReadReceipts => _preferencesService.messageReadReceipts;
  bool get mediaAutoDownload => _preferencesService.mediaAutoDownload;
  NotificationLevel get notificationLevel => _preferencesService.notificationLevel;
  
  ProfileSettingsController({
    required UserRepository userRepository,
    required AuthService authService,
    required PreferencesService preferencesService,
    required StorageService storageService,
  }) : _userRepository = userRepository,
       _authService = authService,
       _preferencesService = preferencesService,
       _storageService = storageService {
    _loadUserProfile();
  }
  
  Future<void> _loadUserProfile() async {
    _isLoading = true;
    _hasError = false;
    notifyListeners();
    
    try {
      final userId = _authService.currentUser?.uid;
      if (userId == null) {
        throw Exception('User not authenticated');
      }
      
      _userProfile = await _userRepository.getUserProfile(userId);
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _isLoading = false;
      _hasError = true;
      _errorMessage = 'Failed to load profile: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> updateProfileName(String name) async {
    if (_userProfile == null) return;
    
    _isSaving = true;
    notifyListeners();
    
    try {
      await _userRepository.updateUserProfile(
        _userProfile!.id,
        {'name': name}
      );
      
      _userProfile = _userProfile!.copyWith(name: name);
      _isSaving = false;
      notifyListeners();
    } catch (e) {
      _isSaving = false;
      _hasError = true;
      _errorMessage = 'Failed to update name: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> updateProfileStatus(String status) async {
    if (_userProfile == null) return;
    
    _isSaving = true;
    notifyListeners();
    
    try {
      await _userRepository.updateUserProfile(
        _userProfile!.id,
        {'status': status}
      );
      
      _userProfile = _userProfile!.copyWith(status: status);
      _isSaving = false;
      notifyListeners();
    } catch (e) {
      _isSaving = false;
      _hasError = true;
      _errorMessage = 'Failed to update status: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> updateProfilePhoto(File imageFile) async {
    if (_userProfile == null) return;
    
    _isSaving = true;
    notifyListeners();
    
    try {
      // Upload image to storage
      final photoUrl = await _storageService.uploadProfileImage(
        _userProfile!.id,
        imageFile
      );
      
      // Update profile with new photo URL
      await _userRepository.updateUserProfile(
        _userProfile!.id,
        {'photoUrl': photoUrl}
      );
      
      _userProfile = _userProfile!.copyWith(photoUrl: photoUrl);
      _isSaving = false;
      notifyListeners();
    } catch (e) {
      _isSaving = false;
      _hasError = true;
      _errorMessage = 'Failed to update profile photo: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> toggleDarkMode(bool value) async {
    await _preferencesService.setDarkMode(value);
    notifyListeners();
  }
  
  Future<void> setChatWallpaper(String wallpaperPath) async {
    await _preferencesService.setChatWallpaper(wallpaperPath);
    notifyListeners();
  }
  
  Future<void> setMessageReadReceipts(bool value) async {
    await _preferencesService.setMessageReadReceipts(value);
    notifyListeners();
  }
  
  Future<void> setMediaAutoDownload(bool value) async {
    await _preferencesService.setMediaAutoDownload(value);
    notifyListeners();
  }
  
  Future<void> setNotificationLevel(NotificationLevel level) async {
    await _preferencesService.setNotificationLevel(level);
    notifyListeners();
  }
  
  Future<void> logout() async {
    try {
      await _authService.signOut();
    } catch (e) {
      _hasError = true;
      _errorMessage = 'Failed to log out: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> refreshProfile() async {
    return _loadUserProfile();
  }
  
  @override
  void dispose() {
    // Clean up any resources
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `profile_settings`
- **Route Path**: `/profile/settings`

### Navigation Logic

```dart
// From Profile Settings to Account Settings
void _navigateToAccountSettings(BuildContext context) {
  context.push('/account-settings');
}

// From Profile Settings to Security & Privacy
void _navigateToSecurityPrivacy(BuildContext context) {
  context.push('/security-privacy');
}

// From Profile Settings to Appearance Settings
void _navigateToAppearanceSettings(BuildContext context) {
  context.push('/appearance-settings');
}

// From Profile Settings to Chat Settings
void _navigateToChatSettings(BuildContext context) {
  context.push('/chat-settings');
}

// From Profile Settings to Notification Settings
void _navigateToNotificationSettings(BuildContext context) {
  context.push('/notification-settings');
}

// From Profile Settings to Help & Support
void _navigateToHelpSupport(BuildContext context) {
  context.push('/help-support');
}

// On Log Out
void _onLogout(BuildContext context) async {
  final confirmed = await showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      title: Text('Log Out'),
      content: Text('Are you sure you want to log out?'),
      actions: [
        TextButton(
          onPressed: () => Navigator.of(context).pop(false),
          child: Text('Cancel'),
        ),
        TextButton(
          onPressed: () => Navigator.of(context).pop(true),
          child: Text('Log Out'),
        ),
      ],
    ),
  );
  
  if (confirmed == true) {
    await context.read<ProfileSettingsController>().logout();
    context.go('/login');
  }
}
```

## Data Flow

### Displayed Data

- User profile information:
  - Name
  - Phone number
  - Profile photo
  - Status (if applicable)
- Application settings:
  - Theme preferences
  - Notification settings
  - Privacy settings
  - Chat preferences
  - Data usage settings

### Data Sources

- Firestore `users` collection (profile data)
- SharedPreferences or Hive (app settings)
- Firebase Storage (profile images)

### Synchronization

- Real-time updates for profile changes
- Local storage for settings preferences
- Background sync for profile updates

## Error Handling

- Profile loading errors with retry option
- Settings update failures with appropriate messages
- Image upload failures with fallback options
- Authentication state validation
- Network connectivity warnings

## Offline Support

- Caches profile data for offline viewing
- Stores settings changes locally and syncs when online
- Queues profile updates for execution when online
- Graceful handling of logout during offline mode

## Dependencies

### External Packages

- `firebase_auth`: Authentication services
- `cloud_firestore`: User profile data
- `firebase_storage`: Profile image storage
- `provider`: State management
- `shared_preferences` or `hive_flutter`: Local settings storage
- `image_picker`: Profile photo selection
- `cached_network_image`: Profile image loading and caching
- `connectivity_plus`: Network connectivity detection
- `go_router`: Navigation and routing

### Services

- `UserRepository`: Manages user profile data
- `AuthService`: Handles authentication state
- `PreferencesService`: Manages application settings
- `StorageService`: Handles file uploads and storage
- `ConnectivityService`: Monitors network status

## Design Notes

### UX Considerations

- Clear grouping of related settings
- Visual feedback for settings changes
- Smooth transitions between settings screens
- Confirmation dialogs for important actions (logout)
- Profile photo editing with preview
- Settings search functionality (if many options)

### Accessibility

- Proper labeling of settings categories
- Sufficient contrast for text and controls
- Screen reader support for all settings
- Support for system text scaling
- Keyboard navigation support
