# Chat Screen

## Purpose

The Chat Screen is the core messaging interface where users can view conversation history and exchange messages, media, and other content with individual contacts or groups. It supports both online and offline messaging with synchronization capabilities.

## UI Components

```text
┌────────────────────────────┐
│ ← John Smith       [Menu]  │
│ Online                     │
│ ┌────────────────────────┐ │
│ │        Today           │ │
│ └────────────────────────┘ │
│                            │
│ ┌───────────────┐          │
│ │Hi there!      │          │
│ │           9:30│          │
│ └───────────────┘          │
│                            │
│          ┌───────────────┐ │
│          │Hey, how are   │ │
│          │you doing?     │ │
│          │           9:35│ │
│          │             ✓✓│ │
│          └───────────────┘ │
│                            │
│ ┌───────────────┐          │
│ │[Image]        │          │
│ │Look at this!  │          │
│ │          10:15│          │
│ └───────────────┘          │
│                            │
│ [Network Status Indicator] │
│                            │
│ [📎] [📷] Message... [🎤]  │
└────────────────────────────┘
```

### Components

- **App Bar**: Contains recipient name, back button, online status, and menu button
- **Message List**: Scrollable list of message bubbles
- **Date Headers**: Separators showing date of messages
- **Message Bubbles**: Individual messages with different styles for sent/received
- **Media Messages**: Image, video, audio, document previews within bubbles
- **Message Status Indicators**: (sending, sent, delivered, read)
- **Timestamp**: Time each message was sent
- **Network Status Indicator**: Shows offline/online status
- **Message Composer**: Input area with text field and attachment options
- **Attachment Buttons**: Options for different media types (camera, gallery, etc.)
- **Voice Message Button**: Record audio messages
- **Typing Indicator**: Shows when recipient is typing

## User Interactions

- Send text messages
- Send media (images, videos, audio, documents, location)
- Record and send voice messages
- View media in fullscreen
- Copy, forward, delete, or reply to messages
- View message info (delivery status, read receipts)
- Scroll through message history with infinite loading
- Pull-to-refresh for manual sync
- Long press on messages for contextual menu
- Tap on attachments to view/play in appropriate viewer

## State Management

### State Provider

The screen uses `ChatController` with Provider for state management:

```dart
class ChatController extends ChangeNotifier {
  final MessageRepository _messageRepository;
  final ChatRepository _chatRepository;
  final MediaService _mediaService;
  final ConnectivityService _connectivityService;
  final String chatId;
  
  final TextEditingController messageController = TextEditingController();
  final ScrollController scrollController = ScrollController();
  
  List<Message> _messages = [];
  bool _isLoading = false;
  bool _isLoadingMore = false;
  bool _hasMoreMessages = true;
  String? _lastMessageId;
  bool _isOnline = true;
  bool _isRecipientTyping = false;
  Map<String, double> _mediaUploadProgress = {};
  
  List<Message> get messages => _messages;
  bool get isLoading => _isLoading;
  bool get isLoadingMore => _isLoadingMore;
  bool get hasMoreMessages => _hasMoreMessages;
  bool get isOnline => _isOnline;
  bool get isRecipientTyping => _isRecipientTyping;
  Map<String, double> get mediaUploadProgress => _mediaUploadProgress;
  
  ChatController({
    required this.chatId,
    required MessageRepository messageRepository,
    required ChatRepository chatRepository,
    required MediaService mediaService,
    required ConnectivityService connectivityService,
  }) : _messageRepository = messageRepository,
       _chatRepository = chatRepository,
       _mediaService = mediaService,
       _connectivityService = connectivityService {
    _initializeController();
  }
  
  void _initializeController() {
    _loadInitialMessages();
    _listenToConnectivity();
    _listenToMessages();
    _listenToTypingStatus();
    
    scrollController.addListener(_scrollListener);
  }
  
  void _scrollListener() {
    // Handle infinite scrolling when user reaches top of list
    if (scrollController.position.pixels >= scrollController.position.maxScrollExtent - 200) {
      _loadMoreMessages();
    }
  }
  
  Future<void> _loadInitialMessages() async {
    if (_isLoading) return;
    
    _isLoading = true;
    notifyListeners();
    
    try {
      final initialMessages = await _messageRepository.getMessages(
        chatId: chatId,
        limit: 100,
      );
      
      _messages = initialMessages;
      _lastMessageId = _messages.isNotEmpty ? _messages.first.id : null;
      _hasMoreMessages = initialMessages.length >= 100;
      _isLoading = false;
      notifyListeners();
      
      // Mark messages as read
      _markMessagesAsRead();
      
    } catch (e) {
      _isLoading = false;
      // Handle error
      notifyListeners();
    }
  }
  
  Future<void> _loadMoreMessages() async {
    if (_isLoadingMore || !_hasMoreMessages || _lastMessageId == null) return;
    
    _isLoadingMore = true;
    notifyListeners();
    
    try {
      final moreMessages = await _messageRepository.getMessages(
        chatId: chatId,
        limit: 50,
        lastMessageId: _lastMessageId,
      );
      
      if (moreMessages.isEmpty) {
        _hasMoreMessages = false;
      } else {
        _messages = [...moreMessages, ..._messages];
        _lastMessageId = moreMessages.first.id;
      }
      
      _isLoadingMore = false;
      notifyListeners();
      
    } catch (e) {
      _isLoadingMore = false;
      // Handle error
      notifyListeners();
    }
  }
  
  void _listenToMessages() {
    _messageRepository.getMessageStream(chatId).listen((updatedMessages) {
      // Add any new messages that aren't in the list yet
      for (final message in updatedMessages) {
        if (!_messages.any((m) => m.id == message.id)) {
          _messages = [..._messages, message];
        } else {
          // Update existing message if status changed
          final index = _messages.indexWhere((m) => m.id == message.id);
          if (index != -1) {
            _messages[index] = message;
          }
        }
      }
      
      // Sort messages by timestamp
      _messages.sort((a, b) => a.timestamp.compareTo(b.timestamp));
      
      notifyListeners();
      
      // Mark new messages as read
      _markMessagesAsRead();
    });
  }
  
  void _listenToConnectivity() {
    _connectivityService.onConnectivityChanged.listen((isConnected) {
      _isOnline = isConnected;
      notifyListeners();
      
      if (isConnected) {
        // Sync pending messages when coming back online
        _syncPendingMessages();
      }
    });
  }
  
  void _listenToTypingStatus() {
    _chatRepository.getTypingStatus(chatId).listen((isTyping) {
      _isRecipientTyping = isTyping;
      notifyListeners();
    });
  }
  
  Future<void> _markMessagesAsRead() async {
    if (_messages.isEmpty) return;
    
    try {
      final unreadMessages = _messages
          .where((m) => m.senderId != _messageRepository.currentUserId && m.status != MessageStatus.read)
          .map((m) => m.id)
          .toList();
          
      if (unreadMessages.isNotEmpty) {
        await _messageRepository.updateMessagesStatus(
          messageIds: unreadMessages,
          status: MessageStatus.read,
        );
      }
    } catch (e) {
      // Handle error silently
    }
  }
  
  Future<void> _syncPendingMessages() async {
    try {
      await _messageRepository.syncPendingMessages(chatId);
    } catch (e) {
      // Handle error
    }
  }
  
  void updateTypingStatus(bool isTyping) {
    _chatRepository.updateTypingStatus(chatId, isTyping);
  }
  
  Future<void> sendTextMessage(String text) async {
    if (text.trim().isEmpty) return;
    
    final message = Message(
      id: DateTime.now().millisecondsSinceEpoch.toString(),
      chatId: chatId,
      senderId: _messageRepository.currentUserId,
      text: text,
      timestamp: DateTime.now(),
      status: _isOnline ? MessageStatus.sending : MessageStatus.pending,
      media: [],
    );
    
    // Clear the text input
    messageController.clear();
    
    // Add to local list first for UI responsiveness
    _messages = [..._messages, message];
    notifyListeners();
    
    try {
      if (_isOnline) {
        // Send message directly if online
        await _messageRepository.sendMessage(message);
      } else {
        // Store locally if offline
        await _messageRepository.saveMessageLocally(message);
      }
    } catch (e) {
      // Handle error, update message status
      final index = _messages.indexWhere((m) => m.id == message.id);
      if (index != -1) {
        _messages[index] = _messages[index].copyWith(status: MessageStatus.failed);
        notifyListeners();
      }
    }
  }
  
  Future<void> sendMediaMessage(String text, List<MediaFile> mediaFiles) async {
    for (final mediaFile in mediaFiles) {
      final messageId = DateTime.now().millisecondsSinceEpoch.toString();
      
      final message = Message(
        id: messageId,
        chatId: chatId,
        senderId: _messageRepository.currentUserId,
        text: text,
        timestamp: DateTime.now(),
        status: _isOnline ? MessageStatus.sending : MessageStatus.pending,
        media: [
          Media(
            id: mediaFile.id,
            messageId: messageId,
            type: mediaFile.type,
            localPath: mediaFile.path,
            uploadStatus: _isOnline ? UploadStatus.uploading : UploadStatus.pending,
            thumbnailData: await _mediaService.generateThumbnail(mediaFile.path, mediaFile.type),
          ),
        ],
      );
      
      // Add to local list first for UI responsiveness
      _messages = [..._messages, message];
      _mediaUploadProgress[messageId] = 0.0;
      notifyListeners();
      
      try {
        if (_isOnline) {
          // Upload media and send message
          for (final media in message.media) {
            final uploadTask = _mediaService.uploadMedia(
              media.localPath,
              media.type,
              onProgress: (progress) {
                _mediaUploadProgress[messageId] = progress;
                notifyListeners();
              },
            );
            
            final mediaUrl = await uploadTask;
            
            final updatedMedia = media.copyWith(
              url: mediaUrl,
              uploadStatus: UploadStatus.uploaded,
            );
            
            final updatedMessage = message.copyWith(
              media: [updatedMedia],
              status: MessageStatus.sending,
            );
            
            await _messageRepository.sendMessage(updatedMessage);
            
            // Update local message
            final index = _messages.indexWhere((m) => m.id == messageId);
            if (index != -1) {
              _messages[index] = updatedMessage;
              notifyListeners();
            }
          }
        } else {
          // Store locally if offline
          await _messageRepository.saveMessageLocally(message);
        }
      } catch (e) {
        // Handle error, update message status
        final index = _messages.indexWhere((m) => m.id == messageId);
        if (index != -1) {
          _messages[index] = _messages[index].copyWith(status: MessageStatus.failed);
          notifyListeners();
        }
      } finally {
        // Remove progress tracking
        _mediaUploadProgress.remove(messageId);
        notifyListeners();
      }
    }
  }
  
  @override
  void dispose() {
    messageController.dispose();
    scrollController.dispose();
    // Update typing status to false when leaving
    updateTypingStatus(false);
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `/chat`
- **Route Path**: `/chats/:chatId`

### Navigation Logic

```dart
// From Chat to Media Preview
void _navigateToMediaPreview(BuildContext context, List<MediaFile> mediaFiles) {
  Navigator.of(context).pushNamed(
    '/media-preview',
    arguments: {
      'chatId': chatId,
      'mediaFiles': mediaFiles,
    },
  );
}

// From Chat to Document Viewer
void _navigateToDocumentViewer(BuildContext context, String documentUrl, String documentName) {
  Navigator.of(context).pushNamed(
    '/document-viewer',
    arguments: {
      'documentUrl': documentUrl,
      'documentName': documentName,
    },
  );
}

// From Chat to Media Gallery
void _navigateToMediaGallery(BuildContext context) {
  Navigator.of(context).pushNamed(
    '/media-gallery',
    arguments: {
      'chatId': chatId,
    },
  );
}

// From Chat to Contact Info
void _navigateToContactInfo(BuildContext context) {
  Navigator.of(context).pushNamed(
    '/contact-info',
    arguments: {
      'contactId': recipientId,
    },
  );
}
```

## Data Flow

### Displayed Data

- Message history with:
  - Text messages
  - Media messages (images, videos, audio, documents, locations)
  - Message status indicators
  - Timestamps
  - User information (sender name, profile picture)

### Data Sources

- Firestore `messages` subcollection within `chats/{chatId}/messages` (remote)
- Local Hive storage for offline access
- Firebase Storage for media files
- User repository for sender information

### Message Flow

1. **Sending a Message**:
   - Create message object with pending/sending status
   - Add to local UI immediately
   - If online: Upload media (if any) → Send message to Firestore
   - If offline: Store message locally → Queue for sync
   - Update UI based on message status changes

2. **Receiving Messages**:
   - Listen to Firestore collection in real-time
   - Update local cache and UI
   - Mark as delivered/read as appropriate
   - Download media thumbnails automatically
   - Download full media on demand or based on settings

3. **Message Pagination**:
   - Load initial batch (100 messages)
   - Load more (50 at a time) when scrolling up
   - Maintain scroll position during loads

## Error Handling

- Failed message sending with retry option
- Media upload failures with retry
- Network disconnection handling
- Message sync conflicts resolution
- Media loading fallbacks (thumbnail display)
- Graceful degradation for unsupported message types

## Offline Support

- Local storage of all viewed messages
- Pending message queue for offline composed messages
- Media caching for viewed content
- Status indicator for pending/failed messages
- Background sync when connectivity is restored
- Timestamp formatting that works offline

## Dependencies

### External Packages

- `cloud_firestore`: Firestore database access
- `firebase_storage`: Media file storage
- `provider`: State management
- `hive_flutter`: Local storage
- `cached_network_image`: Image loading and caching
- `video_player`: Video message playback
- `audio_waveforms`: Audio visualization and recording
- `flutter_chat_ui`: Core chat UI components (optional)
- `intl`: Date/time formatting

### Services

- `MessageRepository`: Manages message data operations
- `ChatRepository`: Provides chat metadata and typing status
- `MediaService`: Handles media uploads, downloads, and processing
- `SyncService`: Manages background synchronization
- `ConnectivityService`: Monitors network status

## Design Notes

### UX Considerations

- Smooth scrolling and pagination
- Visual distinctions between sent/received messages
- Clear status indicators (sending, sent, delivered, read)
- Progress indicators for media uploads/downloads
- Appropriate keyboard handling for message composition
- Message composition state preservation

### Accessibility

- Proper content descriptions for media messages
- Clear status announcements for screen readers
- Support for system text scaling
- Keyboard navigation support
- Alternative text for media content
