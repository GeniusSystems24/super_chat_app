# Media Gallery Screen

## Purpose

The Media Gallery Screen provides users with an organized view of all media (images, videos, documents) shared across their chats. It serves as a centralized location to browse, search, and manage media content without having to navigate through individual conversations.

## UI Components

```text
┌────────────────────────────┐
│ ← Media Gallery      ⋮     │
│ ┌─────┐ ┌─────┐ ┌─────┐    │
│ │ ALL │ │PHOTO│ │VIDEO│    │
│ └─────┘ └─────┘ └─────┘    │
│ ┌─────┐ ┌─────┐ ┌─────┐    │
│ │ DOC │ │AUDIO│ │LINKS│    │
│ └─────┘ └─────┘ └─────┘    │
│                            │
│ Today                      │
│ ┌──────┬──────┬──────┐     │
│ │      │      │      │     │
│ │  🖼️   │  🖼️   │  🖼️   │     │
│ │      │      │      │     │
│ └──────┴──────┴──────┘     │
│ ┌──────┬──────┬──────┐     │
│ │      │      │      │     │
│ │  🖼️   │  🎥  │  🖼️   │     │
│ │      │      │      │     │
│ └──────┴──────┴──────┘     │
│                            │
│ Yesterday                  │
│ ┌──────┬──────┬──────┐     │
│ │      │      │      │     │
│ │  🖼️   │  🖼️   │  📄  │     │
│ │      │      │      │     │
│ └──────┴──────┴──────┘     │
│                            │
│ April 2023                 │
│ ┌──────┬──────┬──────┐     │
│ │      │      │      │     │
│ │  🎥  │  🖼️   │  🖼️   │     │
│ │      │      │      │     │
│ └──────┴──────┴──────┘     │
│                            │
└────────────────────────────┘
```

### Components

- **App Bar**: Contains back button, screen title, and options menu
- **Filter Tabs**: Horizontal row of filter options (All, Photos, Videos, Documents, Audio, Links)
- **Time-based Sections**: Content grouped by timeframes (Today, Yesterday, Last Week, etc.)
- **Media Grid**: Grid layout of media items, with 3-4 items per row
- **Media Thumbnail**: Individual media item with type indicator (image, video, document)
- **Loading Indicators**: Progress indicators for media loading states
- **Empty State**: View shown when no media matches selected filters

## User Interactions

- Tap on filter tabs to switch between media types
- Tap on a media item to open it in full view
- Long press on media items to enter selection mode
- Multi-select media items for bulk actions (share, delete, save)
- Pull to refresh to load latest media
- Pinch to zoom in/out of the grid layout (change grid density)
- Search bar to find specific media by name or context
- Swipe between media items in full-view mode

## State Management

### State Provider

The screen uses `MediaGalleryController` with Provider for state management:

```dart
enum MediaType { all, photo, video, document, audio, link }

class MediaGalleryController extends ChangeNotifier {
  final MediaRepository _mediaRepository;
  final UserPreferences _userPreferences;
  
  bool _isLoading = true;
  bool _hasError = false;
  String _errorMessage = '';
  List<MediaItem> _mediaItems = [];
  MediaType _selectedType = MediaType.all;
  String? _searchQuery;
  
  bool get isLoading => _isLoading;
  bool get hasError => _hasError;
  String get errorMessage => _errorMessage;
  MediaType get selectedType => _selectedType;
  String? get searchQuery => _searchQuery;
  
  // Getters for filtered media
  List<MediaItem> get filteredMedia {
    if (_selectedType == MediaType.all && _searchQuery == null) {
      return _mediaItems;
    }
    
    return _mediaItems.where((item) {
      bool matchesType = _selectedType == MediaType.all || 
                        item.type == _selectedType;
      
      bool matchesSearch = _searchQuery == null || 
                          item.fileName.toLowerCase().contains(_searchQuery!.toLowerCase()) ||
                          item.chatName.toLowerCase().contains(_searchQuery!.toLowerCase());
      
      return matchesType && matchesSearch;
    }).toList();
  }
  
  // Getters for grouped media
  Map<String, List<MediaItem>> get groupedMedia {
    final Map<String, List<MediaItem>> grouped = {};
    
    for (var item in filteredMedia) {
      final groupKey = _getTimeGroupKey(item.timestamp);
      if (!grouped.containsKey(groupKey)) {
        grouped[groupKey] = [];
      }
      grouped[groupKey]!.add(item);
    }
    
    return grouped;
  }
  
  MediaGalleryController({
    required MediaRepository mediaRepository,
    required UserPreferences userPreferences,
  }) : _mediaRepository = mediaRepository,
       _userPreferences = userPreferences {
    _loadMedia();
  }
  
  Future<void> _loadMedia() async {
    _isLoading = true;
    _hasError = false;
    notifyListeners();
    
    try {
      _mediaItems = await _mediaRepository.getAllMedia();
      _isLoading = false;
      notifyListeners();
    } catch (e) {
      _isLoading = false;
      _hasError = true;
      _errorMessage = 'Failed to load media: ${e.toString()}';
      notifyListeners();
    }
  }
  
  void setMediaType(MediaType type) {
    if (_selectedType != type) {
      _selectedType = type;
      notifyListeners();
    }
  }
  
  void setSearchQuery(String? query) {
    if (_searchQuery != query) {
      _searchQuery = query?.trim().isNotEmpty == true ? query : null;
      notifyListeners();
    }
  }
  
  Future<void> deleteMedia(List<String> mediaIds) async {
    try {
      await _mediaRepository.deleteMedia(mediaIds);
      _mediaItems.removeWhere((item) => mediaIds.contains(item.id));
      notifyListeners();
    } catch (e) {
      _hasError = true;
      _errorMessage = 'Failed to delete media: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> saveMediaToGallery(List<String> mediaIds) async {
    try {
      for (var id in mediaIds) {
        final item = _mediaItems.firstWhere((item) => item.id == id);
        await _mediaRepository.saveMediaToGallery(item);
      }
    } catch (e) {
      _hasError = true;
      _errorMessage = 'Failed to save media: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> refreshMedia() async {
    return _loadMedia();
  }
  
  String _getTimeGroupKey(DateTime timestamp) {
    final now = DateTime.now();
    final today = DateTime(now.year, now.month, now.day);
    final yesterday = today.subtract(const Duration(days: 1));
    final date = DateTime(timestamp.year, timestamp.month, timestamp.day);
    
    if (date == today) {
      return 'Today';
    } else if (date == yesterday) {
      return 'Yesterday';
    } else if (today.difference(date).inDays <= 7) {
      return 'Last Week';
    } else if (today.month == date.month && today.year == date.year) {
      return 'This Month';
    } else {
      return '${_getMonthName(date.month)} ${date.year}';
    }
  }
  
  String _getMonthName(int month) {
    const months = [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ];
    return months[month - 1];
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

- **Route Name**: `media_gallery`
- **Route Path**: `/media/gallery`

### Navigation Logic

```dart
// From Media Gallery to Media Viewer
void _navigateToMediaViewer(BuildContext context, MediaItem item, List<MediaItem> allItems) {
  final encodedItems = Uri.encodeComponent(jsonEncode(allItems.map((i) => i.toJson()).toList()));
  context.push('/media-viewer?initialItem=${Uri.encodeComponent(jsonEncode(item.toJson()))}&mediaItems=$encodedItems');
}

// From Media Gallery to Chat Screen
void _navigateToChatWithMedia(BuildContext context, MediaItem item) {
  context.push('/chats/${item.chatId}?highlightMessageId=${item.messageId}');
}

// Back to previous screen
void _navigateBack(BuildContext context) {
  context.pop();
}
```

## Data Flow

### Displayed Data

- Media files categorized by type:
  - Images
  - Videos
  - Documents
  - Audio files
  - Links
- Media metadata:
  - Thumbnails
  - Duration (for videos/audio)
  - File size
  - File type indicator
  - Source chat name
  - Timestamp

### Data Sources

- Local database storing media metadata
- File system for cached media files
- Cloud storage for remote media assets
- Chat message database for context information

### Synchronization

- Lazy loading of media thumbnails
- Background prefetching for visible and adjacent media
- Caching strategy for frequently accessed media
- Scheduled background sync for new media items

## Error Handling

- Media loading failures with retry options
- Download failures with appropriate user feedback
- Permission handling for saving media to device
- Graceful UI for missing thumbnails or corrupted media
- Network connectivity warnings

## Offline Support

- Cached thumbnails available offline
- Downloaded media accessible without network
- Visual indicators for non-downloaded media
- Action queue for operations pending network access
- Smart refresh on network reconnection

## Dependencies

### External Packages

- `cached_network_image`: Media thumbnail loading
- `photo_view`: Enhanced image viewing capabilities
- `video_player`: Video playback support
- `chewie`: Video player UI wrapper
- `path_provider`: File system access for saving media
- `permission_handler`: Managing device permissions
- `provider`: State management
- `intl`: Date formatting
- `flutter_cache_manager`: Media caching
- `go_router`: Navigation and routing

### Services

- `MediaRepository`: Media access and management
- `FileService`: File operations
- `PermissionService`: Handle permission requests
- `UserPreferences`: User settings for media view
- `CacheManager`: Media caching strategies

## Design Notes

### UX Considerations

- Grid layout that adapts to screen size
- Smooth transitions between grid and full-view modes
- Clear visual indicators for media types
- Consistent thumbnail sizes with proper aspect ratios
- Loading states that don't disrupt browsing
- Efficient paging mechanism for large media collections
- Search functionality with visual feedback

### Accessibility

- Alt text descriptions for media content
- Screen reader support for media type and context
- Proper focus management for keyboard navigation
- Support for system text scaling
- High contrast mode support for media controls
- Haptic feedback for selection operations
