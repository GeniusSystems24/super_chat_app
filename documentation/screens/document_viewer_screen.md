# Document Viewer Screen

## Purpose

The Document Viewer Screen provides a dedicated interface for viewing and interacting with documents shared in chats. It allows users to preview various document formats, access document information, and perform actions such as downloading, sharing, and opening with other applications.

## UI Components

```text
┌────────────────────────────┐
│ ← Document.pdf    [Share]  │
│                            │
│ ┌────────────────────────┐ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │   [Document Preview]   │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ │                        │ │
│ └────────────────────────┘ │
│                            │
│ Project Report             │
│ 2.1 MB • PDF               │
│                            │
│ ┌────────────────────────┐ │
│ │     Save to Device     │ │
│ └────────────────────────┘ │
│                            │
└────────────────────────────┘
```

### Components

- **App Bar**: Contains back button, document name, and share button
- **Document Preview Area**: Main viewer for document content with:
  - Page rendering for PDFs, text files, spreadsheets, etc.
  - Zoom and pan capabilities
  - Page navigation controls for multi-page documents
- **Document Information**: Displays metadata such as:
  - File name
  - File size
  - File type
  - Last modified date (optional)
- **Action Buttons**: Options for document interaction:
  - Save to device
  - Print document (if supported)
  - Open with other apps
  - Copy text (for text-based documents)

## User Interactions

- Tap back button to return to previous screen
- Tap share button to share document via other apps
- Pinch to zoom in/out of document
- Drag to pan around document
- Swipe to navigate between document pages
- Tap "Save to Device" to download document to local storage
- Single/double tap for additional viewing options
- Long press for text selection (in supported documents)

## State Management

### State Provider

The screen uses `DocumentViewerController` with Provider for state management:

```dart
enum DocumentLoadingState { initial, loading, loaded, error }

class DocumentViewerController extends ChangeNotifier {
  final FileRepository _fileRepository;
  final PermissionService _permissionService;
  final ConnectivityService _connectivityService;
  
  DocumentModel? _document;
  DocumentLoadingState _loadingState = DocumentLoadingState.initial;
  String _errorMessage = '';
  int _currentPage = 1;
  int _totalPages = 1;
  double _zoomLevel = 1.0;
  bool _isDownloading = false;
  double _downloadProgress = 0.0;
  
  DocumentModel? get document => _document;
  DocumentLoadingState get loadingState => _loadingState;
  String get errorMessage => _errorMessage;
  int get currentPage => _currentPage;
  int get totalPages => _totalPages;
  double get zoomLevel => _zoomLevel;
  bool get isDownloading => _isDownloading;
  double get downloadProgress => _downloadProgress;
  
  DocumentViewerController({
    required FileRepository fileRepository,
    required PermissionService permissionService,
    required ConnectivityService connectivityService,
    DocumentModel? initialDocument,
  }) : _fileRepository = fileRepository,
       _permissionService = permissionService,
       _connectivityService = connectivityService,
       _document = initialDocument {
    if (initialDocument != null) {
      _loadDocument(initialDocument.id, initialDocument.url);
    }
  }
  
  Future<void> _loadDocument(String documentId, String documentUrl) async {
    _loadingState = DocumentLoadingState.loading;
    notifyListeners();
    
    try {
      final localPath = await _fileRepository.getLocalPath(documentId);
      
      if (localPath != null && await File(localPath).exists()) {
        // Document exists locally
        _document = _document?.copyWith(localPath: localPath);
        await _initializeDocument(localPath);
      } else if (_connectivityService.isConnected) {
        // Document needs to be downloaded
        final tempPath = await _fileRepository.downloadFile(
          documentUrl,
          documentId,
          (progress) {
            _downloadProgress = progress;
            notifyListeners();
          },
        );
        
        _document = _document?.copyWith(localPath: tempPath);
        await _initializeDocument(tempPath);
      } else {
        throw Exception('Document not available offline and no network connection');
      }
      
      _loadingState = DocumentLoadingState.loaded;
      notifyListeners();
    } catch (e) {
      _loadingState = DocumentLoadingState.error;
      _errorMessage = 'Failed to load document: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> _initializeDocument(String filePath) async {
    try {
      // For PDFs, get page count
      if (_document?.mimeType == 'application/pdf') {
        _totalPages = await _fileRepository.getPdfPageCount(filePath);
      }
      
      // For other document types, perform necessary initialization
    } catch (e) {
      _errorMessage = 'Failed to initialize document: ${e.toString()}';
    }
  }
  
  void setCurrentPage(int page) {
    if (page > 0 && page <= _totalPages && _currentPage != page) {
      _currentPage = page;
      notifyListeners();
    }
  }
  
  void nextPage() {
    if (_currentPage < _totalPages) {
      _currentPage++;
      notifyListeners();
    }
  }
  
  void previousPage() {
    if (_currentPage > 1) {
      _currentPage--;
      notifyListeners();
    }
  }
  
  void setZoomLevel(double level) {
    if (level >= 0.5 && level <= 3.0) {
      _zoomLevel = level;
      notifyListeners();
    }
  }
  
  Future<bool> saveDocumentToDevice() async {
    if (_document == null || _document?.localPath == null) {
      return false;
    }
    
    _isDownloading = true;
    notifyListeners();
    
    try {
      final hasPermission = await _permissionService.requestStoragePermission();
      if (!hasPermission) {
        throw Exception('Storage permission denied');
      }
      
      final savedPath = await _fileRepository.saveToDownloads(
        _document!.localPath!,
        _document!.name,
      );
      
      _isDownloading = false;
      notifyListeners();
      
      return savedPath != null;
    } catch (e) {
      _isDownloading = false;
      _errorMessage = 'Failed to save document: ${e.toString()}';
      notifyListeners();
      return false;
    }
  }
  
  Future<void> shareDocument() async {
    if (_document == null || _document?.localPath == null) {
      return;
    }
    
    try {
      await _fileRepository.shareFile(_document!.localPath!, _document!.mimeType);
    } catch (e) {
      _errorMessage = 'Failed to share document: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> openWithExternalApp() async {
    if (_document == null || _document?.localPath == null) {
      return;
    }
    
    try {
      await _fileRepository.openWithExternalApp(_document!.localPath!, _document!.mimeType);
    } catch (e) {
      _errorMessage = 'Failed to open document: ${e.toString()}';
      notifyListeners();
    }
  }
  
  Future<void> reloadDocument() async {
    if (_document == null) {
      return;
    }
    
    return _loadDocument(_document!.id, _document!.url);
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

- **Route Name**: `document_viewer`
- **Route Path**: `/document/view`

### Navigation Logic

```dart
// Navigate to Document Viewer
void navigateToDocumentViewer(BuildContext context, DocumentModel document) {
  final encodedDocument = Uri.encodeComponent(jsonEncode(document.toJson()));
  context.push('/document/view?document=$encodedDocument');
}

// Navigate back to previous screen
void _navigateBack(BuildContext context) {
  context.pop();
}

// Navigate to chat where document was shared
void _navigateToChatWithDocument(BuildContext context, DocumentModel document) {
  context.push('/chats/${document.chatId}?messageId=${document.messageId}');
}
```

## Data Flow

### Displayed Data

- Document content based on file type:
  - Rendered PDF pages
  - Text file content
  - Spreadsheet data
  - Presentation slides
  - Fallback preview for unsupported formats
- Document metadata:
  - File name
  - File size
  - File format/type
  - Creation/modification date
  - Source chat (optional)

### Data Sources

- Local file system for cached documents
- Firebase Storage for remote documents
- Document metadata from message database

### Synchronization

- Automatic caching of viewed documents
- Background prefetching of multi-page documents
- Prioritized loading of current page
- Download state persistence for interrupted downloads

## Error Handling

- Document loading failures with retry options
- Unsupported format handling with appropriate messages
- Download failures with appropriate user feedback
- Permission handling for saving documents
- Network connectivity issues with offline fallbacks

## Offline Support

- Cached documents available offline
- Downloaded state tracking for documents
- Clear indicators for documents requiring download
- Queued operations for when connectivity is restored
- Reduced functionality mode for offline viewing

## Dependencies

### External Packages

- `pdf_render`: PDF rendering and viewing
- `flutter_pdfview`: PDF viewer widget
- `native_pdf_view`: Alternative PDF viewer
- `path_provider`: File system access for saving documents
- `permission_handler`: Device permission management
- `share_plus`: Document sharing functionality
- `open_file`: Opening documents with external apps
- `provider`: State management
- `connectivity_plus`: Network connectivity detection
- `go_router`: Navigation and routing

### Services

- `FileRepository`: Document file operations
- `PermissionService`: Handles permission requests
- `ConnectivityService`: Monitors network status
- `AnalyticsService`: Tracks document interactions

## Design Notes

### UX Considerations

- Smooth page transitions for multi-page documents
- Intuitive zoom and pan gestures
- Clear loading states with progress indicators
- Preview generation for common document types
- Error states with actionable recovery options
- Consistent document navigation across different formats
- Night mode support for document viewing

### Accessibility

- Screen reader support for document content
- Keyboard navigation for document browsing
- Text-to-speech capabilities for text documents
- Support for system text scaling
- High contrast mode for document viewing
- Alternative text descriptions for document content
