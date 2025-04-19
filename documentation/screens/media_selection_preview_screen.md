# Media Selection & Preview Screen

## Purpose

The Media Selection & Preview Screen allows users to select, preview, and prepare media files (images, videos, documents) before sending them in chat conversations. It provides editing capabilities, quality options, and the ability to add captions to media content.

## UI Components

```text
┌────────────────────────────┐
│ ← Back            [Send]   │
│                            │
│ ┌────────────────────────┐ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │      [Media Preview]   │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Add a caption...       │ │
│ └────────────────────────┘ │
│                            │
│ Recent                     │
│ ┌──┐┌──┐┌──┐┌──┐┌──┐┌──┐   │
│ │  ││  ││  ││  ││  ││  │   │
│ └──┘└──┘└──┘└──┘└──┘└──┘   │
│                            │
└────────────────────────────┘
```

### Components

- **App Bar**: Contains back button and send button
- **Media Preview**: Large preview area for selected media with:
  - Image/video display
  - Document thumbnail preview
  - Edit controls for images
  - Playback controls for videos/audio
  - Multi-media navigation (if multiple files selected)
- **Caption Input**: Text field for adding a message with the media
- **Recent Media**: Horizontal scrollable list of recently captured/accessed media
- **Media Selection Tools**: Options for:
  - Camera access
  - Gallery access
  - Document picker
  - Location sharing (optional)
  - Contact sharing (optional)

## User Interactions

- Tap back button to return to chat screen
- Tap send button to send media with optional caption
- Swipe between multiple selected media items
- Tap on recent media to select it
- Tap camera icon to capture new photo/video
- Tap gallery icon to access media gallery
- Pinch to zoom images in preview
- Tap edit button to modify images (crop, filters, markup)
- Play/pause/scrub video content
- Long press on preview for additional options
- Tap on caption field to add text description

## State Management

### State Provider

The screen uses `MediaSelectionController` with Provider for state management:

```dart
enum MediaCompressionQuality { low, medium, high, original }

class MediaSelectionController extends ChangeNotifier {
  final MediaRepository _mediaRepository;
  final PreferencesService _preferencesService;
  final ConnectivityService _connectivityService;
  
  List<MediaItem> _selectedMedia = [];
  MediaItem? _currentPreviewItem;
  int _currentPreviewIndex = 0;
  String _caption = '';
  bool _isProcessing = false;
  bool _isEditing = false;
  MediaCompressionQuality _compressionQuality = MediaCompressionQuality.medium;
  List<MediaItem> _recentMedia = [];
  
  // Getters
  List<MediaItem> get selectedMedia => _selectedMedia;
  MediaItem? get currentPreviewItem => _currentPreviewItem;
  int get currentPreviewIndex => _currentPreviewIndex;
  String get caption => _caption;
  bool get isProcessing => _isProcessing;
  bool get isEditing => _isEditing;
  MediaCompressionQuality get compressionQuality => _compressionQuality;
  List<MediaItem> get recentMedia => _recentMedia;
  bool get hasSelectedMedia => _selectedMedia.isNotEmpty;
  bool get isConnected => _connectivityService.isConnected;
  
  MediaSelectionController({
    required MediaRepository mediaRepository,
    required PreferencesService preferencesService,
    required ConnectivityService connectivityService,
    List<MediaItem>? initialSelection,
  }) : _mediaRepository = mediaRepository,
       _preferencesService = preferencesService,
       _connectivityService = connectivityService {
    _initialize(initialSelection);
  }
  
  Future<void> _initialize(List<MediaItem>? initialSelection) async {
    _loadCompressionSetting();
    
    if (initialSelection != null && initialSelection.isNotEmpty) {
      _selectedMedia = List.from(initialSelection);
      _currentPreviewItem = _selectedMedia.first;
    }
    
    await _loadRecentMedia();
  }
  
  Future<void> _loadRecentMedia() async {
    try {
      _recentMedia = await _mediaRepository.getRecentMedia(limit: 20);
      notifyListeners();
    } catch (e) {
      // Handle error silently
    }
  }
  
  void _loadCompressionSetting() {
    final savedQuality = _preferencesService.mediaCompressionQuality;
    if (savedQuality != null) {
      _compressionQuality = savedQuality;
    } else {
      // Default to medium quality
      _compressionQuality = MediaCompressionQuality.medium;
    }
  }
  
  void setCurrentPreviewIndex(int index) {
    if (index >= 0 && index < _selectedMedia.length && index != _currentPreviewIndex) {
      _currentPreviewIndex = index;
      _currentPreviewItem = _selectedMedia[index];
      notifyListeners();
    }
  }
  
  void nextPreview() {
    if (_currentPreviewIndex < _selectedMedia.length - 1) {
      setCurrentPreviewIndex(_currentPreviewIndex + 1);
    }
  }
  
  void previousPreview() {
    if (_currentPreviewIndex > 0) {
      setCurrentPreviewIndex(_currentPreviewIndex - 1);
    }
  }
  
  void setCaption(String caption) {
    _caption = caption;
    notifyListeners();
  }
  
  Future<void> addMediaFromGallery() async {
    try {
      final pickedItems = await _mediaRepository.pickMediaFromGallery(
        allowMultiple: true,
        maxSelections: 10 - _selectedMedia.length,
      );
      
      if (pickedItems.isNotEmpty) {
        _selectedMedia.addAll(pickedItems);
        
        if (_currentPreviewItem == null) {
          _currentPreviewItem = _selectedMedia.first;
          _currentPreviewIndex = 0;
        }
        
        notifyListeners();
      }
    } catch (e) {
      // Handle error
    }
  }
  
  Future<void> captureMediaFromCamera({bool isVideo = false}) async {
    try {
      final capturedItem = await _mediaRepository.captureMediaFromCamera(
        isVideo: isVideo,
      );
      
      if (capturedItem != null) {
        _selectedMedia.add(capturedItem);
        _currentPreviewItem = capturedItem;
        _currentPreviewIndex = _selectedMedia.length - 1;
        notifyListeners();
      }
    } catch (e) {
      // Handle error
    }
  }
  
  Future<void> pickDocument() async {
    try {
      final document = await _mediaRepository.pickDocument();
      if (document != null) {
        _selectedMedia.add(document);
        _currentPreviewItem = document;
        _currentPreviewIndex = _selectedMedia.length - 1;
        notifyListeners();
      }
    } catch (e) {
      // Handle error
    }
  }
  
  void selectRecentMedia(MediaItem item) {
    if (_selectedMedia.length < 10) {
      _selectedMedia.add(item);
      _currentPreviewItem = item;
      _currentPreviewIndex = _selectedMedia.length - 1;
      notifyListeners();
    }
  }
  
  void removeCurrentMedia() {
    if (_currentPreviewItem != null && _selectedMedia.isNotEmpty) {
      final index = _currentPreviewIndex;
      _selectedMedia.removeAt(index);
      
      if (_selectedMedia.isEmpty) {
        _currentPreviewItem = null;
        _currentPreviewIndex = 0;
      } else if (index >= _selectedMedia.length) {
        _currentPreviewIndex = _selectedMedia.length - 1;
        _currentPreviewItem = _selectedMedia[_currentPreviewIndex];
      } else {
        _currentPreviewItem = _selectedMedia[_currentPreviewIndex];
      }
      
      notifyListeners();
    }
  }
  
  void setCompressionQuality(MediaCompressionQuality quality) {
    _compressionQuality = quality;
    _preferencesService.setMediaCompressionQuality(quality);
    notifyListeners();
  }
  
  Future<void> startEditing() async {
    if (_currentPreviewItem == null || 
        _currentPreviewItem!.type != MediaType.image ||
        _isEditing) {
      return;
    }
    
    _isEditing = true;
    notifyListeners();
  }
  
  Future<void> applyImageEdit(MediaItem originalItem, File editedFile) async {
    try {
      final editedItem = await _mediaRepository.createMediaItemFromFile(
        editedFile,
        MediaType.image,
        originalItem.fileName,
      );
      
      // Replace the original item with edited item
      final index = _selectedMedia.indexWhere((item) => item.id == originalItem.id);
      if (index != -1) {
        _selectedMedia[index] = editedItem;
        if (_currentPreviewIndex == index) {
          _currentPreviewItem = editedItem;
        }
      }
      
      _isEditing = false;
      notifyListeners();
    } catch (e) {
      _isEditing = false;
      notifyListeners();
      // Handle error
    }
  }
  
  Future<void> cancelEditing() async {
    _isEditing = false;
    notifyListeners();
  }
  
  Future<List<PreparedMediaItem>> prepareMediaForSending() async {
    _isProcessing = true;
    notifyListeners();
    
    try {
      final compressedItems = <PreparedMediaItem>[];
      
      for (final item in _selectedMedia) {
        // Skip compression for documents or if quality is set to original
        if (item.type == MediaType.document || 
            _compressionQuality == MediaCompressionQuality.original) {
          compressedItems.add(
            PreparedMediaItem(
              originalItem: item,
              preparedFile: File(item.localPath),
              originalSize: await File(item.localPath).length(),
              compressedSize: await File(item.localPath).length(),
            ),
          );
          continue;
        }
        
        // Compress image/video based on quality setting
        final compressedFile = await _mediaRepository.compressMedia(
          item,
          _compressionQuality,
        );
        
        compressedItems.add(
          PreparedMediaItem(
            originalItem: item,
            preparedFile: compressedFile,
            originalSize: await File(item.localPath).length(),
            compressedSize: await compressedFile.length(),
          ),
        );
      }
      
      _isProcessing = false;
      notifyListeners();
      return compressedItems;
    } catch (e) {
      _isProcessing = false;
      notifyListeners();
      // Handle error
      return [];
    }
  }
  
  @override
  void dispose() {
    // Clean up resources
    super.dispose();
  }
}
```

## Navigation and Routing

### Route Information

- **Route Name**: `media_selection`
- **Route Path**: `/media/selection`

### Navigation Logic

```dart
// Navigate to Media Selection from Chat
void navigateToMediaSelection(BuildContext context, {List<MediaItem>? initialItems}) {
  final encodedItems = initialItems != null 
      ? Uri.encodeComponent(jsonEncode(initialItems.map((i) => i.toJson()).toList()))
      : null;
  
  final queryParams = encodedItems != null ? '?initialItems=$encodedItems' : '';
  context.push('/media/selection$queryParams');
}

// Navigate back to Chat (cancel)
void _navigateBack(BuildContext context) {
  context.pop();
}

// Navigate back to Chat with prepared media
void _sendMediaAndNavigateBack(
  BuildContext context, 
  List<PreparedMediaItem> preparedItems,
  String caption
) {
  context.pop({
    'mediaItems': preparedItems,
    'caption': caption,
  });
}

// Navigate to Image Editor
void _navigateToImageEditor(BuildContext context, MediaItem imageItem) {
  final encodedItem = Uri.encodeComponent(jsonEncode(imageItem.toJson()));
  context.push('/image-editor?imageItem=$encodedItem');
}
```

## Data Flow

### Displayed Data

- Selected media previews:
  - Images with aspect ratio preservation
  - Video thumbnails with duration indicator
  - Document thumbnails with type indicator
- Media metadata:
  - File size (original and compressed)
  - Media dimensions (for images/videos)
  - Duration (for videos/audio)
  - Filename (for documents)
- Recent media thumbnails

### Data Sources

- Device photo gallery
- Device file system
- Camera input
- Local media cache

### Synchronization

- Automatic caching of viewed/selected media
- Storage of edited media versions
- Compression processing in background
- Progress tracking for media preparation

## Error Handling

- Media selection failures with appropriate messages
- Camera access failures with permission guidance
- Media processing errors with retry options
- File size limitations with user feedback
- Network awareness for large media files
- Graceful handling of unsupported file types

## Offline Support

- Fully functional without network connection
- Local file access for media selection
- Media compression without cloud dependency
- Queue prepared media for sending when offline
- Clear indicators for pending network operations

## Dependencies

### External Packages

- `image_picker`: Image and video picking from gallery/camera
- `file_picker`: Document selection
- `camera`: Direct camera integration for photo/video capture
- `video_compress`: Video compression
- `flutter_image_compress`: Image compression
- `photo_view`: Enhanced image viewing capabilities
- `video_player`: Video preview and playback
- `path_provider`: File system access
- `permission_handler`: Device permission management
- `provider`: State management
- `image_editor`: Image editing capabilities (crop, filters, etc.)
- `connectivity_plus`: Network connectivity detection
- `go_router`: Navigation and routing

### Services

- `MediaRepository`: Media access and processing
- `PreferencesService`: User preferences for media quality
- `ConnectivityService`: Network status monitoring
- `PermissionService`: Permission handling
- `StorageService`: File operations

## Design Notes

### UX Considerations

- Smooth transitions between media items
- Intuitive controls for multi-media selection
- Clear visual feedback during media processing
- Quality options that balance size and clarity
- Efficient layout for thumbnail selection
- Preview size that adequately represents final result
- Progress indicators for media preparation
- Edit controls with real-time preview

### Accessibility

- Alternative methods for media selection
- Clear labeling of media controls
- Screen reader support for media types
- Adequate touch targets for media selection
- Support for system text scaling in caption input
- High contrast mode support for media controls
- Haptic feedback for selection operations
