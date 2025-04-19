# Contact List Screen

## Purpose

The Contact List Screen displays the user's contacts, allowing them to start new conversations, create group chats, and manage their contact list. It serves as a directory for finding and interacting with people within the application.

## UI Components

```text
┌────────────────────────────┐
│ ← Contacts        [Search] │
│                            │
│ ┌────────────────────────┐ │
│ │ New Group              │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ New Contact            │ │
│ └────────────────────────┘ │
│                            │
│ A                          │
│ ┌────────────────────────┐ │
│ │ Alice Johnson          │ │
│ │ Hey there!             │ │
│ └────────────────────────┘ │
│                            │
│ B                          │
│ ┌────────────────────────┐ │
│ │ Bob Williams           │ │
│ │ Available              │ │
│ └────────────────────────┘ │
│                            │
│  [Chats] [Calls] [Contacts]│
└────────────────────────────┘
```

### Components

- **App Bar**: Contains back button, title, and search button
- **Action Buttons**: "New Group" and "New Contact" options
- **Alphabetical Sections**: Contacts organized alphabetically with section headers
- **Contact Items**: Individual contact entries with:
  - Profile photo
  - Contact name
  - Status or last interaction
  - Online/offline indicator
- **Bottom Navigation Bar**: Tab navigation between main app sections

## User Interactions

- Tap on contact to open or start a chat
- Tap on "New Group" to begin group creation flow
- Tap on "New Contact" to add a new contact
- Tap on search to filter contacts by name
- Pull to refresh for contact list synchronization
- Long press on contact for additional options
- Swipe actions on contacts (if implemented)

## State Management

### State Provider

The screen uses `ContactListController` with Provider for state management:

```dart
class ContactListController extends ChangeNotifier {
  final UserRepository _userRepository;
  final ContactRepository _contactRepository;
  final ConnectivityService _connectivityService;
  
  List<Contact> _contacts = [];
  bool _isLoading = false;
  bool _hasError = false;
  String _errorMessage = '';
  bool _isOnline = true;
  String _searchQuery = '';
  List<Contact> _filteredContacts = [];
  
  List<Contact> get contacts => _searchQuery.isEmpty ? _contacts : _filteredContacts;
  bool get isLoading => _isLoading;
  bool get hasError => _hasError;
  String get errorMessage => _errorMessage;
  bool get isOnline => _isOnline;
  
  ContactListController({
    required UserRepository userRepository,
    required ContactRepository contactRepository,
    required ConnectivityService connectivityService,
  }) : _userRepository = userRepository,
       _contactRepository = contactRepository,
       _connectivityService = connectivityService {
    _initializeController();
  }
  
  void _initializeController() {
    _loadContacts();
    _listenToConnectivity();
    _listenToContactUpdates();
  }
  
  Future<void> _loadContacts() async {
    _isLoading = true;
    _hasError = false;
    notifyListeners();
    
    try {
      _contacts = await _contactRepository.getContacts();
      _sortContacts();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _isLoading = false;
      _hasError = true;
      _errorMessage = 'Failed to load contacts: ${e.toString()}';
      notifyListeners();
    }
  }
  
  void _sortContacts() {
    _contacts.sort((a, b) => a.name.compareTo(b.name));
  }
  
  void _listenToConnectivity() {
    _connectivityService.onConnectivityChanged.listen((isConnected) {
      _isOnline = isConnected;
      notifyListeners();
      
      if (isConnected && _contacts.isEmpty) {
        _loadContacts();
      }
    });
  }
  
  void _listenToContactUpdates() {
    _contactRepository.contactUpdates.listen((updatedContacts) {
      _contacts = updatedContacts;
      _sortContacts();
      
      if (_searchQuery.isNotEmpty) {
        _filterContacts();
      }
      
      notifyListeners();
    });
  }
  
  void searchContacts(String query) {
    _searchQuery = query.toLowerCase();
    
    if (_searchQuery.isEmpty) {
      _filteredContacts = [];
    } else {
      _filterContacts();
    }
    
    notifyListeners();
  }
  
  void _filterContacts() {
    _filteredContacts = _contacts.where((contact) {
      return contact.name.toLowerCase().contains(_searchQuery) ||
             contact.phoneNumber.contains(_searchQuery);
    }).toList();
  }
  
  Future<void> refreshContacts() async {
    if (!_isOnline) {
      return;
    }
    
    return _loadContacts();
  }
  
  Future<Contact?> addContact(String phoneNumber, String name) async {
    try {
      final newContact = await _contactRepository.addContact(phoneNumber, name);
      // Contact will be added through the stream listener
      return newContact;
    } catch (e) {
      _hasError = true;
      _errorMessage = 'Failed to add contact: ${e.toString()}';
      notifyListeners();
      return null;
    }
  }
  
  @override
  void dispose() {
    // Clean up any stream subscriptions
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `contacts`
- **Route Path**: `/contacts`

### Navigation Logic

```dart
// From Contacts to Individual Chat
void _navigateToChat(BuildContext context, String contactId, String contactName) {
  final chatId = _getChatIdForContact(contactId); // Helper method to get or create chat ID
  context.push('/chats/$chatId?recipientName=$contactName&isGroup=false');
}

// From Contacts to New Group
void _navigateToNewGroup(BuildContext context) {
  context.push('/new-group');
}

// From Contacts to New Contact
void _navigateToAddContact(BuildContext context) {
  context.push('/add-contact');
}

// From Contacts to Contact Info
void _navigateToContactInfo(BuildContext context, String contactId) {
  context.push('/contact-info/$contactId');
}
```

## Data Flow

### Displayed Data

- List of contacts with:
  - Contact name and profile photo
  - Status or last activity
  - Online/offline status
  - Last message or interaction (if applicable)

### Data Sources

- Firestore `users` collection (remote)
- Device contacts integration (optional)
- Local Hive storage for offline access

### Synchronization

- Real-time updates with Firestore listeners
- Background sync when connectivity changes
- Device contacts sync (if implemented)

## Error Handling

- Empty state handling (no contacts yet)
- Error state with retry option
- Failed contact additions with appropriate messages
- Network connectivity warnings
- Appropriate handling of permission issues (for device contacts)

## Offline Support

- Caches contact list in Hive for offline access
- Displays cached contacts with offline indicator
- Queues contact additions for execution when online
- Status indicators adjust based on connectivity
- Displays local timestamps for last activity

## Dependencies

### External Packages

- `cloud_firestore`: Firestore database access
- `provider`: State management
- `hive_flutter`: Local storage
- `cached_network_image`: Profile image loading and caching
- `contacts_service`: Device contacts integration (optional)
- `permission_handler`: Contact permission management
- `connectivity_plus`: Network connectivity detection
- `go_router`: Navigation and routing

### Services

- `ContactRepository`: Manages contact data operations
- `UserRepository`: User profile information
- `ChatRepository`: For creating/finding chat conversations
- `ConnectivityService`: Monitors network status

## Design Notes

### UX Considerations

- Alphabetical organization with fast-scroll capability
- Clear indicators for online/offline status
- Placeholder images during loading
- Search with real-time filtering
- Smooth animations for status changes
- Easy access to frequent contacts (if implemented)

### Accessibility

- Proper grouping and labeling of alphabetical sections
- Sufficient contrast for status indicators
- Screen reader support for contact information
- Support for system text scaling
- Keyboard navigation for searching and selection
