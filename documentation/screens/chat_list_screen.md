# Chat List Screen

## Purpose

The Chat List Screen is the main entry point of the application after login, displaying all user conversations sorted by recent activity. It provides an overview of all chats, with quick access to individual conversations, settings, and contact management.

## UI Components

```text
┌────────────────────────────┐
│  [Profile]    WhatsApp     │
│                   [Search] │
│ ┌────────────────────────┐ │
│ │         Chats          │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ John Smith        12:34│ │
│ │ Hey, how are you? ✓✓   │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Group Chat         10:15│ │
│ │ Alice: Check this... 2 │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Mom                09:45│ │
│ │ Call me when you... ✓  │ │
│ └────────────────────────┘ │
│                            │
│ [Network Status Indicator] │
│                            │
│  [Chats] [Calls] [Contacts]│
└────────────────────────────┘
```

### Components

- **App Bar**: Contains profile button, app title, and search button
- **Tab Bar**: Navigation between Chats, Calls, and Contacts sections
- **Chat List**: Scrollable list of chat preview items
- **Chat Item**: Individual chat preview with:
  - Contact/group name
  - Last message preview
  - Timestamp
  - Message status indicators (sent, delivered, read)
  - Unread message count badge
- **Floating Action Button**: Create new chat/message
- **Network Status Indicator**: Shows offline/online status
- **Bottom Navigation Bar**: Tab navigation between main app sections

## User Interactions

- Tap on chat item to open individual chat
- Tap on profile button to access profile settings
- Tap on search to filter chats by keyword
- Long press on chat for additional options (archive, delete, mute)
- Pull to refresh for manual sync
- Tap on FAB to start new chat
- Switch between tabs (Chats, Calls, Contacts)
- Swipe actions on chat items (configurable, e.g., archive, pin)

## State Management

### State Provider

The screen uses `ChatListController` with Provider for state management:

```dart
class ChatListController extends ChangeNotifier {
  final ChatRepository _chatRepository;
  final ConnectivityService _connectivityService;
  
  List<Chat> _chats = [];
  bool _isLoading = false;
  bool _hasError = false;
  String _errorMessage = '';
  bool _isOnline = true;
  
  List<Chat> get chats => _chats;
  bool get isLoading => _isLoading;
  bool get hasError => _hasError;
  String get errorMessage => _errorMessage;
  bool get isOnline => _isOnline;
  
  ChatListController({
    required ChatRepository chatRepository,
    required ConnectivityService connectivityService,
  }) : _chatRepository = chatRepository,
       _connectivityService = connectivityService {
    _initializeController();
  }
  
  void _initializeController() {
    _loadChats();
    _listenToConnectivity();
    _listenToChatUpdates();
  }
  
  Future<void> _loadChats() async {
    _isLoading = true;
    _hasError = false;
    notifyListeners();
    
    try {
      _chats = await _chatRepository.getChats();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _isLoading = false;
      _hasError = true;
      _errorMessage = 'Failed to load chats: ${e.toString()}';
      notifyListeners();
    }
  }
  
  void _listenToConnectivity() {
    _connectivityService.onConnectivityChanged.listen((isConnected) {
      _isOnline = isConnected;
      notifyListeners();
      
      if (isConnected && _chats.isEmpty) {
        _loadChats();
      }
    });
  }
  
  void _listenToChatUpdates() {
    _chatRepository.chatUpdates.listen((updatedChats) {
      _chats = updatedChats;
      notifyListeners();
    });
  }
  
  Future<void> refreshChats() async {
    if (!_isOnline) {
      // Handle offline refresh
      return;
    }
    
    return _loadChats();
  }
  
  Future<void> archiveChat(String chatId) async {
    try {
      await _chatRepository.archiveChat(chatId);
      // Remove from current list if successful
      _chats.removeWhere((chat) => chat.id == chatId);
      notifyListeners();
    } catch (e) {
      // Handle error
    }
  }
  
  Future<void> deleteChat(String chatId) async {
    try {
      await _chatRepository.deleteChat(chatId);
      // Remove from current list if successful
      _chats.removeWhere((chat) => chat.id == chatId);
      notifyListeners();
    } catch (e) {
      // Handle error
    }
  }
  
  Future<void> muteChat(String chatId, Duration duration) async {
    try {
      await _chatRepository.muteChat(chatId, duration);
      // Update the local chat object
      final index = _chats.indexWhere((chat) => chat.id == chatId);
      if (index != -1) {
        _chats[index] = _chats[index].copyWith(
          isMuted: true,
          muteExpiresAt: DateTime.now().add(duration),
        );
        notifyListeners();
      }
    } catch (e) {
      // Handle error
    }
  }
  
  @override
  void dispose() {
    // Clean up any stream subscriptions or controllers
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `chat_list`
- **Route Path**: `/chats`

### Navigation Logic

```dart
// From Chat List to Individual Chat
void _navigateToChat(BuildContext context, String chatId, String chatName, bool isGroup) {
  context.push('/chats/$chatId?recipientName=$chatName&isGroup=${isGroup.toString()}');
}

// From Chat List to Profile Settings
void _navigateToProfile(BuildContext context) {
  context.push('/profile/settings');
}

// From Chat List to Contacts (for new chat)
void _navigateToContacts(BuildContext context) {
  context.push('/contacts');
}

// From Chat List to Search
void _navigateToSearch(BuildContext context) {
  context.push('/search');
}
```

## Data Flow

### Displayed Data

- List of chat conversations with:
  - Contact/group name and profile photo
  - Most recent message preview
  - Timestamp of last message
  - Unread message count
  - Message status (sent, delivered, read)
  - Online/offline status of contacts

### Data Sources

- Firestore `chats` collection (remote)
- Local Hive storage for offline access
- User's contacts database

### Synchronization

- Real-time updates with Firestore listeners
- Background sync of message statuses
- Queue management for pending operations

## Error Handling

- Empty state handling (no chats yet)
- Error state with retry option
- Network connectivity warnings
- Sync failure notifications
- Partial data loading with placeholders

## Offline Support

- Caches chat list in Hive for offline access
- Displays cached messages with offline indicator
- Shows pending message status for unsent messages
- Updates timestamps with relative time indicators ("2 min ago")
- Queues operations (delete, archive) for execution when online

## Dependencies

### External Packages

- `cloud_firestore`: Firestore database access
- `provider`: State management
- `hive_flutter`: Local storage
- `cached_network_image`: Profile image loading and caching
- `timeago`: Relative timestamp formatting
- `connectivity_plus`: Network connectivity detection
- `flutter_slidable`: Swipe actions for chat items
- `go_router`: Navigation and routing

### Services

- `ChatRepository`: Manages chat data operations
- `MessageRepository`: Provides message data
- `UserRepository`: User profile information
- `SyncService`: Handles background synchronization
- `ConnectivityService`: Monitors network status

## Design Notes

### UX Considerations

- Smooth animations for status changes
- Clear indicators for unread messages
- Visual differentiation for muted chats
- Optimized list rendering for performance
- Placeholder images during loading
- Contextual swipe actions

### Accessibility

- Meaningful content descriptions for list items
- Sufficient contrast for read/unread states
- Screen reader announcements for message status changes
- Support for system text scaling
- Keyboard navigation support
