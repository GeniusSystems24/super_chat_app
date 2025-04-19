# User Interface Screens for WhatsApp-Style Chat Application

This document outlines all the screens required for the chat application, including their layouts, functionality, features, and technical considerations.

## Table of Contents

- [User Interface Screens for WhatsApp-Style Chat Application](#user-interface-screens-for-whatsapp-style-chat-application)
  - [Table of Contents](#table-of-contents)
  - [Authentication Screens](#authentication-screens)
    - [Splash Screen](#splash-screen)
    - [Login Screen](#login-screen)
    - [Registration Screen](#registration-screen)
    - [Verification Screen](#verification-screen)
  - [Main Application Screens](#main-application-screens)
    - [Chat List Screen](#chat-list-screen)
    - [Chat Screen](#chat-screen)
    - [Media Selection \& Preview Screen](#media-selection--preview-screen)
    - [Contact List Screen](#contact-list-screen)
    - [Profile \& Settings Screen](#profile--settings-screen)
  - [Utility Screens](#utility-screens)
    - [Network Status Screen](#network-status-screen)
    - [Sync Progress Screen](#sync-progress-screen)
    - [Media Gallery Screen](#media-gallery-screen)
    - [Document Viewer Screen](#document-viewer-screen)
    - [Message Upload Status Screen](#message-upload-status-screen)
    - [Message Sending States](#message-sending-states)
      - [1. Message Sending State](#1-message-sending-state)
      - [2. Message Failed State](#2-message-failed-state)
      - [3. Media Upload Progress State](#3-media-upload-progress-state)
      - [4. Queued Message State (Offline Mode)](#4-queued-message-state-offline-mode)
    - [Bulk Upload Manager Screen](#bulk-upload-manager-screen)
    - [Media Compression Settings Screen](#media-compression-settings-screen)

---

## Authentication Screens

### Splash Screen

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

**Purpose**: Initial loading screen while the app checks authentication status and initializes necessary services.

**Features**:

- App logo and branding
- Loading indicator
- Automatic redirection to either Login or Chat List screen based on authentication status

**Technical Requirements**:

- Firebase Authentication check
- Local storage initialization (Hive)
- Network connectivity check
- State: Simple loading state

---

### Login Screen

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

**Purpose**: Authenticate existing users to access the application.

**Features**:

- Phone number/email input
- Password input with visibility toggle
- Login button
- Forgot password option
- Registration link for new users
- Form validation
- Error messaging for failed login attempts

**Technical Requirements**:

- Firebase Authentication integration
- Secure password handling
- Input validation
- State: Authentication state management
- Backend communication: User authentication

---

### Registration Screen

```text
┌────────────────────────────┐
│  ← Back                    │
│                            │
│  Create New Account        │
│                            │
│  ┌────────────────────┐    │
│  │     Full Name      │    │
│  └────────────────────┘    │
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
│  │  Confirm Password  │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │      Register      │    │
│  └────────────────────┘    │
│                            │
│  Already have an account?  │
│  Login                     │
│                            │
└────────────────────────────┘
```

**Purpose**: Allow new users to create an account.

**Features**:

- Name input
- Phone number/email input
- Password creation with strength indicator
- Password confirmation
- Form validation
- Terms of service agreement
- Registration button
- Login link for existing users

**Technical Requirements**:

- Firebase Authentication integration
- Secure password handling
- Input validation
- State: Registration form state management
- Backend communication: User creation

---

### Verification Screen

```text
┌────────────────────────────┐
│  ← Back                    │
│                            │
│  Verification Code         │
│                            │
│  We sent a verification    │
│  code to +1 (555) 123-4567 │
│                            │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐  │
│  │  │ │  │ │  │ │  │ │  │  │
│  └──┘ └──┘ └──┘ └──┘ └──┘  │
│                            │
│  Didn't receive code?      │
│  Resend (00:30)            │
│                            │
│  ┌────────────────────┐    │
│  │       Verify       │    │
│  └────────────────────┘    │
│                            │
└────────────────────────────┘
```

**Purpose**: Verify user's phone number through SMS verification code.

**Features**:

- OTP (One-time password) input fields
- Auto-advancing focus between digits
- Resend code option with countdown timer
- Verification button
- Clear error messages

**Technical Requirements**:

- Firebase Phone Authentication integration
- Timer management for code resending
- Input validation
- State: Verification state management
- Backend communication: Code verification

---

## Main Application Screens

### Chat List Screen

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

**Purpose**: Display all user conversations and provide access to other main features.

**Features**:

- List of recent chats sorted by most recent activity
- Preview of last message with timestamp
- Unread message count indicators
- Message status indicators (sent, delivered, read)
- Quick access to profile and settings
- Search functionality for chats
- Bottom navigation for accessing different sections
- Network status indicator
- FAB (Floating Action Button) to start new chat

**Dynamic Behaviors**:

- Real-time updates to message previews and timestamps
- Online/offline status updates for contacts
- Pull-to-refresh for manual sync
- Message status updates

**Technical Requirements**:

- Firestore integration for chat list
- Local storage for offline access
- State: Chat list state management
- Backend communication: Chat metadata
- Offline capabilities: Cache chat list for offline viewing

---

### Chat Screen

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

**Purpose**: Allow users to send and receive messages, media, and view chat history with a specific contact or group.

**Features**:

- Message bubble UI with sent/received styling
- Text message support
- Media message support (images, videos, audio, documents)
- Message status indicators (sending, sent, delivered, read)
- Timestamps
- Contact online/typing status
- Message composition with text input
- Media attachment options
- Voice message recording
- Date headers to group messages

**Dynamic Behaviors**:

- Real-time message updates
- Typing indicators
- Message status changes
- Infinite scroll to load older messages
- Auto-scroll to bottom for new messages
- Media upload progress indicators
- Message reaction animations
- Background image uploading

**Technical Requirements**:

- Firestore integration for messages
- Firebase Storage for media
- Local storage for offline messaging
- State: Chat message state management
- Backend communication: Message sending/receiving
- Offline capabilities: Store messages locally, queue outgoing messages

---

### Media Selection & Preview Screen

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

**Purpose**: Allow users to select, preview, and send media files.

**Features**:

- Media preview (images, videos, documents)
- Caption input
- Send button
- Media gallery access
- Camera access
- Recent media quick selection
- Edit options for images (crop, filters)
- Multiple media selection

**Dynamic Behaviors**:

- Video playback controls
- Audio playback controls
- Image zoom and pan
- File size indicator
- Upload quality options

**Technical Requirements**:

- Image picker integration
- Camera integration
- Media processing
- State: Media selection state management
- Backend communication: Media preparation
- Offline capabilities: Save draft media selections

---

### Contact List Screen

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

**Purpose**: Display and manage user contacts for starting new conversations.

**Features**:

- Alphabetically sorted contact list
- Contact status indicators
- Search functionality
- New contact addition
- New group creation
- Contact details access
- Quick chat initiation

**Dynamic Behaviors**:

- Online status updates
- Contact availability indicators
- Recently active indicators

**Technical Requirements**:

- Firebase Authentication for contact list
- Firestore for contact data
- Local storage for offline access
- State: Contact list state management
- Backend communication: User data
- Offline capabilities: Cache contact list

---

### Profile & Settings Screen

```text
┌────────────────────────────┐
│ ← Settings                 │
│                            │
│ ┌────────────────────────┐ │
│ │        [Avatar]        │ │
│ │                        │ │
│ │      John Smith        │ │
│ │                        │ │
│ │ Available              │ │
│ └────────────────────────┘ │
│                            │
│ Account                    │
│ ┌────────────────────────┐ │
│ │ Privacy                │ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ Security               │ │
│ └────────────────────────┘ │
│                            │
│ Chats                      │
│ ┌────────────────────────┐ │
│ │ Chat Wallpaper         │ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ Chat History          >│ │
│ └────────────────────────┘ │
│                            │
│ Notifications              │
│ ┌────────────────────────┐ │
│ │ Message Notifications  │ │
│ └────────────────────────┘ │
│                            │
│ Storage and Data           │
│ ┌────────────────────────┐ │
│ │ Network Usage          │ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ Storage Usage          │ │
│ └────────────────────────┘ │
│                            │
│ Help                       │
│ ┌────────────────────────┐ │
│ │ About                  │ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ Log Out                │ │
│ └────────────────────────┘ │
│                            │
└────────────────────────────┘
```

**Purpose**: Allow users to view and edit their profile information and application settings.

**Features**:

- Profile photo management
- Status/availability setting
- Personal information editing
- Privacy settings
- Notification preferences
- Chat settings (wallpaper, text size)
- Storage and data usage
- Help and support access
- Logout functionality

**Technical Requirements**:

- Firebase Authentication for profile data
- Firestore for user preferences
- Local storage for settings
- State: Settings state management
- Backend communication: User data updates
- Offline capabilities: Cache settings

---

## Utility Screens

### Network Status Screen

```text
┌────────────────────────────┐
│                            │
│                            │
│           [Icon]           │
│                            │
│      No Internet Connection│
│                            │
│  You're currently offline. │
│  Messages will be sent when│
│  you're back online.       │
│                            │
│                            │
│  ┌────────────────────┐    │
│  │   Continue Offline │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │      Try Again     │    │
│  └────────────────────┘    │
│                            │
└────────────────────────────┘
```

**Purpose**: Inform users about network status changes and offline mode.

**Features**:

- Clear network status information
- Offline mode explanation
- Retry connection option
- Continue offline option
- Visual indicator for connection state

**Dynamic Behaviors**:

- Auto-dismissal when connection is restored
- Connection quality indicator

**Technical Requirements**:

- Network connectivity monitoring
- State: Network status state management
- Offline capabilities: Display when offline

---

### Sync Progress Screen

```text
┌────────────────────────────┐
│                            │
│                            │
│           [Icon]           │
│                            │
│      Syncing Messages      │
│                            │
│  ████████░░░░░░ 60%        │
│                            │
│  Sending 12 of 20 messages │
│                            │
│                            │
│  ┌────────────────────┐    │
│  │   Continue in App  │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │       Cancel       │    │
│  └────────────────────┘    │
│                            │
└────────────────────────────┘
```

**Purpose**: Show synchronization progress when the app is uploading offline messages or media after regaining connectivity.

**Features**:

- Sync progress indicator
- Media upload counts
- Message sync counts
- Continue option (to use app during sync)
- Cancel option

**Dynamic Behaviors**:

- Real-time progress updates
- Auto-dismissal when sync is complete
- Background sync capability

**Technical Requirements**:

- Sync service integration
- State: Sync state management
- Backend communication: Synchronization process

---

### Media Gallery Screen

```text
┌────────────────────────────┐
│ ← Media Gallery    [Search]│
│                            │
│ All Media                  │
│ ┌────┬────┬────┬────┐      │
│ │    │    │    │    │      │
│ │    │    │    │    │      │
│ └────┴────┴────┴────┘      │
│ ┌────┬────┬────┬────┐      │
│ │    │    │    │    │      │
│ │    │    │    │    │      │
│ └────┴────┴────┴────┘      │
│                            │
│ Documents                  │
│ ┌────────────────────────┐ │
│ │ Report.pdf       2.1MB │ │
│ │ Yesterday             >│ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ Notes.txt        15KB  │ │
│ │ 10/15/2023            >│ │
│ └────────────────────────┘ │
│                            │
│ Audio                      │
│ ┌────────────────────────┐ │
│ │ Voice Message    0:45  │ │
│ │ 10/14/2023        ▶   │ │
│ └────────────────────────┘ │
│                            │
└────────────────────────────┘
```

**Purpose**: Display all media and files shared in a specific chat or across all chats.

**Features**:

- Categorized media display (images, videos, documents, audio)
- Grid view for visual media
- List view for documents
- Search functionality
- Filter options
- Sort options (date, size, type)
- Media download status

**Dynamic Behaviors**:

- Media thumbnail loading
- Lazy loading of media grid
- Download progress indicators

**Technical Requirements**:

- Firebase Storage integration
- Local media cache
- State: Media gallery state management
- Backend communication: Media retrieval
- Offline capabilities: Show cached media

---

### Document Viewer Screen

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

**Purpose**: Preview and interact with document files shared in chats.

**Features**:

- Document preview for supported formats
- File information display (name, size, type)
- Download/save option
- Share option
- Print option (where supported)
- Open with other apps

**Dynamic Behaviors**:

- Document loading indicator
- Download progress
- Page navigation for multi-page documents

**Technical Requirements**:

- Document viewer integration
- File handling
- State: Document viewer state management
- Backend communication: Document retrieval
- Offline capabilities: Cache viewed documents

---

### Message Upload Status Screen

```text
┌────────────────────────────┐
│                            │
│       Upload Progress      │
│                            │
│  ┌────────────────────┐    │
│  │██████████░░░░░░░░░░│    │
│  │       50%          │    │
│  └────────────────────┘    │
│                            │
│  Uploading video...        │
│  12 MB of 24 MB            │
│                            │
│  ┌────────────────────┐    │
│  │      Cancel        │    │
│  └────────────────────┘    │
│                            │
│  ┌────────────────────┐    │
│  │ Continue in Background   │
│  └────────────────────┘    │
│                            │
└────────────────────────────┘
```

**Purpose**: Display real-time progress of media uploads for messages, especially for large files.

**Features**:

- Progress bar with percentage
- File size information (uploaded/total)
- Media type indicator
- Cancel upload option
- Continue in background option
- Estimated time remaining

**Dynamic Behaviors**:

- Real-time progress updates
- Speed adaptation based on network conditions
- Auto-dismissal when upload completes
- Error handling with retry options

**Technical Requirements**:

- Firebase Storage upload monitoring
- Background upload service
- Network speed calculation
- State: Upload progress state management
- Backend communication: Upload process

---

### Message Sending States

The following screens represent different states that messages can have in the Chat Screen:

#### 1. Message Sending State

```text
┌────────────────────────────┐
│          ┌───────────────┐ │
│          │Hey, I'm sending│ │
│          │you a video     │ │
│          │           now  │ │
│          │            🕒 │ │
│          └───────────────┘ │
└────────────────────────────┘
```

**Features**:

- Clock icon indicating message is being processed
- Dimmed appearance compared to sent messages
- Ability to cancel sending (long press)

#### 2. Message Failed State

```text
┌────────────────────────────┐
│          ┌───────────────┐ │
│          │This message    │ │
│          │failed to send  │ │
│          │               │ │
│          │            ❌ │ │
│          └───────────────┘ │
└────────────────────────────┘
```

**Features**:

- Error icon indicating failed delivery
- Tap to retry sending
- Long press for more options (delete, copy, etc.)
- Visual indication of error state (red border or background)

#### 3. Media Upload Progress State

```text
┌────────────────────────────┐
│          ┌───────────────┐ │
│          │[Image with     │ │
│          │overlay]        │ │
│          │                │ │
│          │ ██████░░░░ 60% │ │
│          │            ⏸️  │ │
│          └───────────────┘ │
└────────────────────────────┘
```

**Features**:

- Progress indicator overlaid on media
- Percentage display
- Pause/resume button
- Cancel button
- Visual dimming of media while uploading

#### 4. Queued Message State (Offline Mode)

```text
┌────────────────────────────┐
│          ┌───────────────┐ │
│          │This will be    │ │
│          │sent when you're│ │
│          │online         │ │
│          │            📤 │ │
│          └───────────────┘ │
└────────────────────────────┘
```

**Features**:

- Queue icon indicating message is waiting to be sent
- Visual indication of queued state
- Counter for number of queued messages (if multiple)
- Information about when message will be sent

**Technical Requirements for Message States**:

- Local message state tracking
- Message queue management
- Firebase upload status monitoring
- Error handling and retry logic
- Network connectivity monitoring
- Background processing for uploads
- State: Message sending state management
- Offline capabilities: Queue management

---

### Bulk Upload Manager Screen

```text
┌────────────────────────────┐
│ ← Upload Manager   [Clear] │
│                            │
│ Completed (3)              │
│ ┌────────────────────────┐ │
│ │ [✓] Video.mp4    24 MB │ │
│ │ Today 12:34 PM        >│ │
│ └────────────────────────┘ │
│ ┌────────────────────────┐ │
│ │ [✓] Image1.jpg   2.5MB │ │
│ │ Today 12:32 PM        >│ │
│ └────────────────────────┘ │
│                            │
│ In Progress (2)            │
│ ┌────────────────────────┐ │
│ │ [⏳] Image2.jpg  3.1MB │ │
│ │ 68% - 30s remaining   >│ │
│ └────────────────────────┘ │
│                            │
│ Failed (1)                 │
│ ┌────────────────────────┐ │
│ │ [❌] Document.pdf 15MB │ │
│ │ Tap to retry          >│ │
│ └────────────────────────┘ │
│                            │
│ Queued (4)                 │
│ ┌────────────────────────┐ │
│ │ [📤] 4 items  Waiting │ │
│ │ Will upload when online>│ │
│ └────────────────────────┘ │
│                            │
└────────────────────────────┘
```

**Purpose**: Provide an overview and management interface for all ongoing, completed, and failed uploads.

**Features**:

- Categorized list of uploads by status
- Progress indicators for ongoing uploads
- Timestamp for completed uploads
- Retry options for failed uploads
- Clear completed uploads option
- Cancel ongoing upload option
- Batch operations (cancel all, retry all)

**Dynamic Behaviors**:

- Real-time progress updates
- Auto-categorization as uploads complete or fail
- Background uploading when app is minimized

**Technical Requirements**:

- Upload queue management system
- Background upload service
- Media metadata tracking
- State: Upload manager state management
- Backend communication: Multiple concurrent uploads
- Offline capabilities: Queue persistence

---

### Media Compression Settings Screen

```text
┌────────────────────────────┐
│ ← Media Quality Settings   │
│                            │
│ ┌────────────────────────┐ │
│ │ Image Upload Quality   │ │
│ │                        │ │
│ │ Low   Medium   High    │ │
│ │  ○      ●       ○      │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Video Upload Quality   │ │
│ │                        │ │
│ │ Low   Medium   High    │ │
│ │  ○      ●       ○      │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Auto-Compress Media    │ │
│ │ when on Mobile Data    │ │
│ │                        │ │
│ │             [ON]       │ │
│ └────────────────────────┘ │
│                            │
│ ┌────────────────────────┐ │
│ │ Download Media         │ │
│ │ Automatically          │ │
│ │                        │ │
│ │ Wi-Fi Only   [ON]      │ │
│ │ Mobile Data  [OFF]     │ │
│ └────────────────────────┘ │
│                            │
│ Estimated Storage Savings  │
│ with Current Settings:     │
│ ~60% (23 MB per 100 photos)│
│                            │
└────────────────────────────┘
```

**Purpose**: Allow users to configure media quality settings to optimize for data usage, storage space, and upload speed.

**Features**:

- Image quality settings
- Video quality settings
- Network-based automatic compression
- Download settings based on network type
- Storage savings estimation
- Data usage estimation

**Technical Requirements**:

- Image compression implementation
- Video compression implementation
- Network type detection
- Settings persistence
- Media size calculation
- State: Settings state management

---
