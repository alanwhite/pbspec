# Android Implementation Specification

**Version:** 1.0  
**Last Updated:** October 06, 2025  
**Platforms:** Android 10 (API 29+), Android 14 (API 34+)  
**Primary Distribution:** Google Play Store

---

## Document Purpose and Scope

This document specifies the **Android platform-specific implementation** of the Scottish pipe band application, building upon the platform-agnostic domain architecture defined in `pipe_band_app_architecture.md`.

**Reference Architecture:**
- **Domain Specification:** `pipe_band_app_architecture.md`
- **Domain Entities:** Section 3.1 (Instruments, Score, Tune, Part, Measure, Note, Embellishments)
- **Use Cases:** Section 3.4 (Score Management, Musical Editing, Pagination)
- **Repository Interfaces:** Section 3.5 (Data access contracts)
- **File Format:** Section 4 (.pbscore JSON specification)
- **UX Patterns:** `ux_interaction_design.md` (Platform-agnostic interaction models)

**What This Document Contains:**
- Kotlin 1.9+ and Kotlin Coroutines architecture with structured concurrency
- Jetpack Compose presentation layer with Material Design 3
- Google Play Store distribution requirements and compliance
- Google Drive integration + Storage Access Framework modes
- MediaCodec and AudioTrack audio processing
- Canvas API + custom view SMuFL rendering
- Platform-specific performance optimizations

**What This Document Does NOT Contain:**
- Domain logic definitions (see core architecture document)
- Business rules specifications (see core architecture document)
- Cross-platform architectural patterns (see core architecture document)
- Musical notation concepts (see core architecture document)

---

## 1. Technology Stack and Architecture

### 1.1 Core Technologies

**Language and Concurrency:**
- **Kotlin 1.9+** with null safety and coroutines
- **Kotlin Coroutines** for structured concurrency
- **Flow** for reactive streams and state management
- **StateFlow/SharedFlow** for observable state
- **Structured concurrency** with proper cancellation
- **Dispatchers** for thread management (Main, IO, Default, Unconfined)

**UI Framework:**
- **Jetpack Compose** as primary UI framework
- **Material Design 3** (Material You) for modern Android look
- **Compose UI** for declarative UI with Kotlin DSL
- **Compose Navigation** for type-safe navigation
- **Custom Canvas drawing** for score rendering
- **Window Size Classes** for responsive layouts (phone, tablet, foldable)

**Architecture Pattern:**
- **MVVM** with Jetpack Architecture Components
- **Repository Pattern** with suspend functions and Flow
- **Use Case Pattern** for domain operations
- **Clean Architecture** layers via Gradle modules
- **Dependency Injection** with Hilt (Dagger)

**Audio Processing:**
- **MediaCodec** API for audio encoding/decoding
- **AudioTrack** for low-latency playback
- **AudioRecord** for microphone capture
- **Oboe library** (optional) for professional audio (C++ interop)
- **ExoPlayer** for complex audio scenarios (optional)

**Local Storage:**
- **Room Database** (SQLite abstraction) with Kotlin Flow
- **DataStore** for preferences (replacing SharedPreferences)
- **Coroutine-based DAOs** for reactive database access
- **WorkManager** for background processing and sync

**Music Notation:**
- **Canvas API** for custom drawing with hardware acceleration
- **Paint** and **Path** for vector rendering
- **TypefaceCompat** for SMuFL font loading
- **Custom Views** for complex notation elements
- **Compose Canvas** for Jetpack Compose integration

**Cloud Integration:**
- **Google Drive API** via Google Play Services (primary mode)
- **Storage Access Framework** for universal cloud support (File Provider mode)
- **DocumentsContract** for document provider integration
- **WorkManager** for background sync with constraints
- **Google Play Services Auth** for Google Account integration

### 1.2 Google Play Store Distribution Requirements

**Required Permissions:**
```xml
AndroidManifest.xml Configuration:
├── Internet Access
│   └── <uses-permission android:name="android.permission.INTERNET" />
├── Microphone Access (Runtime Permission)
│   └── <uses-permission android:name="android.permission.RECORD_AUDIO" />
├── Storage Access (Scoped Storage Compliant)
│   ├── <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
│   │                    android:maxSdkVersion="32" />
│   └── <uses-permission android:name="android.permission.READ_MEDIA_AUDIO"
│                        android:minSdkVersion="33" />
├── Foreground Service (Metronome, Recording)
│   └── <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
│       <uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />
└── Optional Permissions
    ├── <uses-permission android:name="android.permission.POST_NOTIFICATIONS"
    │                    android:minSdkVersion="33" />
    └── <uses-permission android:name="android.permission.WAKE_LOCK" />
```

**Privacy Manifest (Data Safety Form):**
```
Google Play Console Data Safety Requirements:
├── Data Collection Disclosure:
│   ├── Musical Scores: User-generated content (Google Drive mode)
│   ├── Audio Recordings: Practice sessions (local or cloud)
│   ├── Device ID: For cloud sync coordination (Google Drive)
│   ├── User Account: Google Account email (Google Drive mode)
│   └── Crash Logs: Anonymous crash reports (Firebase Crashlytics)
├── Data Sharing: None with third parties
├── Data Security:
│   ├── Data encrypted in transit (HTTPS/TLS)
│   ├── Data encrypted at rest (device encryption)
│   ├── User can request data deletion
│   └── Follows Play's Families policy (if targeting children)
├── Data Usage Purpose:
│   ├── App Functionality: Primary purpose
│   ├── Developer Communications: Update notifications
│   └── Fraud Prevention: Security monitoring
└── Optional Features:
    ├── Account Creation: Google Sign-In only
    └── Data Retention: User-controlled via app settings
```

**Feature Declarations:**
```xml
<uses-feature android:name="android.hardware.microphone" android:required="false" />
<uses-feature android:name="android.hardware.audio.output" android:required="true" />
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />
<uses-feature android:name="android.software.midi" android:required="false" />
```

**Play Store Compliant Technologies:**
```
Allowed Technologies:
├── Jetpack Libraries (Compose, Room, Navigation, Hilt, etc.)
├── Google Play Services (Drive API, Auth, Billing)
├── Material Components for Android
├── Kotlin Coroutines Flow
├── OkHttp and Retrofit (networking)
├── Gson or Kotlinx Serialization (JSON)
├── Coil or Glide (image loading)
├── WorkManager (background tasks)
└── Firebase SDK (Crashlytics, Analytics with consent)

Prohibited Technologies:
├── ❌ Root access or system-level modifications
├── ❌ Kernel drivers or low-level hardware access
├── ❌ Unsigned native libraries from untrusted sources
├── ❌ WebView for core functionality (policy 4.4)
├── ❌ Hot code push or dynamic code loading
├── ❌ Permissions not declared in manifest
├── ❌ Background location access without justification
└── ❌ Accessibility services for non-accessibility purposes
```

---

## 2. Storage Architecture Implementation

### 2.1 Two-Tier Storage Strategy

**Storage Mode Enumeration:**
```kotlin
enum class StorageMode {
    GOOGLE_DRIVE,    // Primary, recommended for Play Store
    FILE_PROVIDER    // Advanced users, universal cloud support
}
```

**Strategic Recommendation for Google Play Store:**

**Google Drive Mode (Primary)** is strongly recommended because:
- Seamless integration with user's Google Account (90%+ Android users signed in)
- Automatic privacy compliance through Google's infrastructure
- Enhanced Play Store discoverability ("Works with Google Drive" badge)
- Reduced Play Review friction (well-understood Google technology)
- Native collaboration via Drive API
- Zero-configuration user experience
- Automatic encryption and security compliance
- Real-time sync across Android, iOS, Web
- 15GB free storage per Google Account

**File Provider Mode (Advanced)** is available for:
- Enterprise requirements with specific cloud provider mandates
- Cross-platform band collaboration needing specific cloud services
- Power users preferring Dropbox, OneDrive, or other providers
- Organizations with existing cloud infrastructure investments

### 2.2 Document-Based Architecture with Storage Access Framework

**SAF Integration Pattern:**

**Document-Centric Design:**
- Use ACTION_CREATE_DOCUMENT for save operations
- Use ACTION_OPEN_DOCUMENT for open operations
- Persist URI permissions with takePersistableUriPermission
- Store document URIs in SharedPreferences or DataStore
- Coordinate file access through ContentResolver
- Handle document provider disconnections gracefully

**Key Components:**
- DocumentFile for file operations
- ContentResolver for reading/writing content
- DocumentsContract for querying document metadata
- MediaStore for audio file management (API 29+)
- Scoped Storage compliance (no direct file path access)

### 2.3 Google Drive Mode Implementation

**Google Drive API Integration:**

**Authentication and Authorization:**
- Google Sign-In via GoogleSignInClient
- Drive API v3 via Google Play Services
- Incremental authorization (request Drive scope when needed)
- Handle token expiration and refresh automatically
- Offline access with refresh tokens

**File Operations:**
- Chunked upload for large files using MediaHttpUploader
- Resumable uploads with upload session tokens
- Delta sync with changes.list API
- Real-time notifications via Drive API webhooks (advanced)
- Conflict resolution with user intervention

**Collaboration Features:**
- File sharing with permissions (view, edit, comment)
- Real-time collaboration metadata
- Comments and activity tracking
- Version history access
- Shared drives support (for band organizations)

### 2.4 File Provider Mode Implementation

**Storage Access Framework Integration:**

**Universal File Access:**
- Storage pickers for all document providers
- Automatic handling of permissions
- Works with any cloud provider supporting SAF
- Seamless local/cloud file access
- Respects user's preferred cloud provider

**Implementation Requirements:**
- Use Intent.ACTION_OPEN_DOCUMENT for opening
- Use Intent.ACTION_CREATE_DOCUMENT for saving
- Persist permissions with takePersistableUriPermission
- Handle file not found errors gracefully
- Support MIME type filtering for .pbscore files

### 2.5 First Launch Experience (Play Store Compliant)

**Onboarding Flow:**

**Storage Mode Selection:**
1. Welcome screen with app introduction
2. Check for Google Account availability automatically
3. If signed in: Explain Google Drive benefits and recommend
4. If not signed in: Offer sign-in or File Provider mode
5. Never block access to app features
6. Store preference for future launches

**Permission Request Pattern (Play Store Compliant):**
- Runtime permissions requested contextually
- Microphone permission when user taps "Record Practice"
- Notification permission after creating first score (Android 13+)
- Pre-permission explanation dialogs before system prompt
- Graceful degradation if permissions denied
- Settings deep-link guidance if permission denied

**Design Considerations:**
- Material Design 3 components throughout
- Dynamic color theming (Material You)
- Edge-to-edge layout with proper window insets
- Support for dark theme (follow system preference)
- Responsive layout for phones, tablets, foldables
- Accessibility support (TalkBack, Switch Access)

---

## 3. Kotlin Coroutines Architecture

### 3.1 Dependency Injection with Hilt

**Application Setup:**

**Hilt Configuration:**
- Application class annotated with HiltAndroidApp
- Activity classes annotated with AndroidEntryPoint
- ViewModels injected with HiltViewModel annotation
- Repository implementations provided via modules
- Singleton scope for domain services and caches
- ActivityRetainedScope for ViewModels
- ViewModelScope for ViewModel-specific coroutines

**Module Structure:**
- AppModule: Application-level singletons
- DatabaseModule: Room database and DAOs
- NetworkModule: Retrofit, OkHttp, API services
- RepositoryModule: Repository implementations
- UseCaseModule: Domain use case bindings
- AudioModule: Audio processing services

### 3.2 Coroutine-Based Domain Layer

**Domain Coordinator Pattern:**

**Async Coordinators:**
- Use suspend functions for async operations
- Return Result type for error handling
- Use StateFlow for observable state
- Use SharedFlow for one-time events
- Implement proper cancellation with Job
- Use supervisorScope for concurrent operations
- Handle exceptions with try-catch and Result

**Threading Strategy:**
- Dispatchers.Main for UI updates
- Dispatchers.IO for file and network operations
- Dispatchers.Default for CPU-intensive calculations
- Custom dispatcher for audio processing (optional)
- withContext for dispatcher switching
- ensureActive() checks for cooperative cancellation

**Concurrency Patterns:**
- async/await for parallel operations
- Flow for reactive data streams
- StateFlow for UI state management
- SharedFlow for events and side effects
- Mutex for mutual exclusion where needed
- Channel for producer-consumer patterns

### 3.3 Concurrent Layout Calculation

**Layout Calculation Engine:**

**Parallel Processing:**
- Use async builders with custom dispatcher
- Process independent pages in parallel
- Use supervisorScope to isolate failures
- Collect results with awaitAll
- Cancel in-progress calculations on new request
- Monitor progress with Flow updates

**Caching Strategy:**
- LRU cache for layout results
- Thread-safe concurrent collections
- Automatic cache invalidation on changes
- Cache size limits based on device memory
- Eviction policy for memory pressure
- Metrics collection for cache effectiveness

**Performance Targets:**
- Measure layout: < 15ms per measure
- System layout: < 75ms per system
- Page layout: < 250ms per page
- Full document: < 3s for 100 pages
- Incremental update: < 150ms for single change

---

## 4. UI Implementation with Jetpack Compose

### 4.1 Score Editor Composables

**Main Editor Structure:**

**Composable Hierarchy:**
- ScoreEditorScreen: Top-level screen composable
- ScoreAppBar: Top app bar with actions
- ScoreCanvas: Custom view for rendering (AndroidView wrapper)
- BottomToolbar: Bottom toolbar with primary tools
- FloatingActionButton: Primary action (add note, etc.)
- NavigationDrawer: Side navigation for document structure
- ModalBottomSheet: Context actions and properties

**State Management:**
- ViewModel holds UI state in StateFlow
- Compose observes state with collectAsState
- State hoisting for reusable composables
- Remember and rememberSaveable for local state
- Derived state with derivedStateOf for performance
- Side effects with LaunchedEffect and DisposableEffect

### 4.2 SMuFL Rendering with Custom Views

**Custom Canvas Drawing:**

**Android View Integration:**
- Custom View class for score rendering
- Canvas.drawText for SMuFL glyphs
- Paint configuration for anti-aliasing and hardware acceleration
- Path drawing for staff lines and beams
- Proper measure/layout for view sizing
- Invalidation strategy for updates
- Compose integration with AndroidView

**Rendering Pipeline:**
1. Measure phase: Calculate required canvas size
2. Layout phase: Position drawable elements
3. Draw phase: Render to Canvas
4. Staff lines with Paint.drawLine
5. SMuFL glyphs with Canvas.drawText
6. Grace notes with scaled Paint
7. Beaming with Path and drawPath
8. Collision detection and avoidance
9. Hardware acceleration via View.setLayerType

**Font Management:**
- Load SMuFL fonts (Bravura) from assets
- Create Typeface from font file
- Configure Paint with typeface and size
- Calculate glyph bounds with Paint.measureText
- Handle font fallback for missing glyphs

### 4.3 Material Design 3 Components

**UI Components:**
- TopAppBar with Material3 styling
- NavigationBar for bottom navigation
- NavigationDrawer for hierarchy navigation
- Button, IconButton for actions
- TextField for text input
- Card for content containers
- AlertDialog for confirmations
- ModalBottomSheet for context actions
- Snackbar for feedback messages
- CircularProgressIndicator for loading

**Theming:**
- Material3 dynamic color scheme (Material You)
- Light and dark theme support
- Custom color scheme for brand identity
- Typography scale from Material3
- Shape system for rounded corners
- Elevation and shadows
- Motion and animation

### 4.4 Responsive Design

**Window Size Classes:**
- Compact: Phone portrait (width < 600dp)
- Medium: Phone landscape, small tablet (600dp - 840dp)
- Expanded: Large tablet, foldable unfolded (> 840dp)

**Layout Adaptation:**
- Compact: Single column, bottom navigation
- Medium: Two columns, side navigation rail
- Expanded: Three columns, permanent drawer
- Foldable-aware layouts with WindowMetrics
- Adaptive navigation with NavigationSuiteScaffold

---

## 5. Audio Processing with Android APIs

### 5.1 Audio Engine Implementation

**Audio Processing Architecture:**

**Recording Pipeline:**
- AudioRecord for microphone capture
- Audio format: PCM 16-bit, 44.1kHz, mono
- Buffer management with optimal size calculation
- Real-time processing in coroutine with Dispatcher.Default
- File writing with MediaCodec or AudioFormat.ENCODING_PCM_16BIT
- Output to AAC or FLAC format for efficient storage

**Playback Pipeline:**
- AudioTrack for low-latency playback
- Metronome click generation with sine wave synthesis
- Tempo-accurate scheduling with System.nanoTime
- Accent pattern based on time signature
- Background playback with foreground service

**Audio Session Management:**
- AudioManager for audio focus handling
- AudioFocusRequest for ducking and pausing
- Respond to audio becoming noisy (headphones unplugged)
- Handle phone calls and other interruptions
- Restore audio state after interruption

### 5.2 Metronome Service

**Foreground Service Implementation:**

**Service Design:**
- Foreground service for metronome playback
- Notification for user awareness and control
- MediaSession for media controls integration
- Start with startForegroundService
- Stop with stopForeground and stopSelf
- Handle service binding for UI communication

**Playback Logic:**
- Coroutine-based tick generation
- AudioTrack buffering for smooth playback
- Dynamic tempo changes without restart
- Accent pattern calculation from time signature
- Beat events via SharedFlow for UI sync

### 5.3 Permission Handling

**Runtime Permissions:**
- Check permission with ContextCompat.checkSelfPermission
- Request permission with ActivityResultContracts.RequestPermission
- Show rationale before requesting if shouldShowRequestPermissionRationale
- Handle permission denial with graceful degradation
- Deep-link to app settings for manual permission grant

---

## 6. Google Play Store Compliance Checklist

### 6.1 Pre-Submission Requirements

**Technical Requirements:**
- ✅ AndroidManifest.xml properly configured with all permissions
- ✅ targetSdkVersion 34 (Android 14) minimum
- ✅ compileSdkVersion 34 or higher
- ✅ Scoped storage compliance (no direct file paths)
- ✅ No crashes on fresh install and basic usage
- ✅ App signed with release keystore
- ✅ ProGuard/R8 rules configured for release builds
- ✅ APK or App Bundle size optimized (< 150MB recommended)

**Content Requirements:**
- ✅ Store listing complete with accurate descriptions
- ✅ Screenshots show actual app functionality (minimum 2, maximum 8)
- ✅ Feature graphic: 1024x500 pixels
- ✅ App icon: 512x512 pixels (PNG, 32-bit with alpha)
- ✅ Privacy policy URL provided (required for microphone permission)
- ✅ Content rating questionnaire completed
- ✅ Target audience correctly specified

**Functionality Requirements:**
- ✅ All advertised features fully implemented
- ✅ No placeholder content or "coming soon" features
- ✅ Proper error handling with user-friendly messages
- ✅ Acceptable performance on mid-range devices
- ✅ Dark theme support throughout
- ✅ Accessibility tested with TalkBack

### 6.2 Google Play Console Configuration

**App Information:**
```
Play Console Configuration:
├── App Name: "Pipe Band Score Editor"
├── Short Description: (80 characters max)
│   └── "Professional pipe band music notation with authentic embellishments"
├── Full Description: (4000 characters max)
│   └── Detailed feature list and benefits
├── Category: Music & Audio
├── Tags: Music Creation, Productivity
├── Content Rating: Everyone (IARC)
├── Target Audience: Age 13+ (Google Play Families if targeting younger)
└── Pricing: Free (IAP optional for future features)
```

**Store Listing Assets:**
```
Required Assets:
├── App Icon: 512x512 PNG, 32-bit, alpha channel
├── Feature Graphic: 1024x500 PNG or JPEG
├── Screenshots:
│   ├── Phone: Minimum 2, maximum 8 (16:9 or 9:16 aspect ratio)
│   ├── 7-inch Tablet: Minimum 1, maximum 8 (optional)
│   ├── 10-inch Tablet: Minimum 1, maximum 8 (optional)
│   └── All screenshots must show actual app interface
├── Promo Video: YouTube link (optional but recommended)
└── Privacy Policy: Publicly accessible URL (required)
```

**Data Safety Form:**
```
Data Safety Disclosure:
├── Data Collection:
│   ├── Personal Info: Email address (Google Drive mode only)
│   ├── Device IDs: Android ID for sync coordination
│   ├── App Activity: Musical scores created
│   ├── Files and Docs: Audio recordings
│   └── App Info and Performance: Crash logs
├── Data Usage:
│   ├── App Functionality: Primary purpose for all data
│   └── Analytics: Crash reports (Firebase Crashlytics)
├── Data Sharing:
│   └── None with third parties
├── Security Practices:
│   ├── Data encrypted in transit: Yes (HTTPS/TLS)
│   ├── Data encrypted at rest: Yes (device encryption)
│   ├── Users can request deletion: Yes (in-app settings)
│   └── Data handled per Play's Families Policy: Yes (if applicable)
└── Optional Features:
    ├── Account Creation: Google Sign-In only
    ├── Data Retention: User-controlled via settings
    └── Contact Developer: support@yourdomain.com
```

### 6.3 Build Configuration

**Gradle Configuration:**

**Project-Level build.gradle:**
- Kotlin version 1.9+
- Android Gradle Plugin 8.1+
- Kotlin Serialization plugin
- Hilt Gradle plugin
- Google Services plugin (for Firebase)

**App-Level build.gradle:**
- compileSdk 34
- targetSdk 34
- minSdk 29 (Android 10)
- versionCode incremental integer
- versionName semantic versioning
- applicationId unique package name
- Proguard/R8 configuration for release
- Signing config with keystore

**Required Dependencies:**
- androidx.core:core-ktx
- androidx.compose.ui:ui
- androidx.compose.material3:material3
- androidx.lifecycle:lifecycle-viewmodel-compose
- androidx.navigation:navigation-compose
- androidx.room:room-runtime and room-ktx
- androidx.datastore:datastore-preferences
- androidx.work:work-runtime-ktx
- com.google.dagger:hilt-android
- com.google.android.gms:play-services-auth
- com.google.android.gms:play-services-drive
- org.jetbrains.kotlinx:kotlinx-coroutines-android
- org.jetbrains.kotlinx:kotlinx-serialization-json
- com.google.firebase:firebase-crashlytics
- io.coil-kt:coil-compose

### 6.4 Release Build Configuration

**ProGuard/R8 Rules:**
- Keep domain model classes
- Keep serialization classes
- Keep Hilt-injected classes
- Optimize and obfuscate release builds
- Keep source file names and line numbers for crash reports
- Custom rules for third-party libraries

**Signing Configuration:**
- Release keystore stored securely
- Key alias and passwords in gradle.properties (excluded from VCS)
- Automatic signing with signingConfig
- Upload key different from app signing key (Play App Signing)

**App Bundle vs APK:**
- Prefer Android App Bundle for Play Store (AAB format)
- Automatic APK generation per device configuration
- Dynamic feature modules support (optional)
- Reduced download size for users

---

## 7. Development Workflow

### 7.1 Project Structure

**Gradle Multi-Module Organization:**

```
PipeBandApp/
├── app/                              # Main Android app module
│   ├── src/main/
│   │   ├── java/com/yourcompany/pipebandapp/
│   │   │   ├── ui/                   # Compose UI and screens
│   │   │   ├── viewmodel/            # ViewModels
│   │   │   ├── di/                   # Hilt modules
│   │   │   ├── navigation/           # Navigation graphs
│   │   │   └── MainActivity.kt
│   │   ├── res/                      # Resources (layouts, strings, etc.)
│   │   └── AndroidManifest.xml
│   └── build.gradle.kts
├── domain/                           # Domain layer module
│   ├── src/main/java/com/yourcompany/pipebandapp/domain/
│   │   ├── model/                    # Domain entities
│   │   ├── usecase/                  # Use case classes
│   │   ├── repository/               # Repository interfaces
│   │   └── service/                  # Domain services
│   └── build.gradle.kts
├── data/                             # Data layer module
│   ├── src/main/java/com/yourcompany/pipebandapp/data/
│   │   ├── repository/               # Repository implementations
│   │   ├── local/                    # Room database
│   │   ├── remote/                   # API clients
│   │   ├── serialization/            # JSON serialization
│   │   └── worker/                   # WorkManager workers
│   └── build.gradle.kts
├── rendering/                        # Custom view rendering module
│   ├── src/main/java/com/yourcompany/pipebandapp/rendering/
│   │   ├── canvas/                   # Canvas drawing logic
│   │   ├── layout/                   # Layout calculation
│   │   └── smufl/                    # SMuFL font handling
│   └── build.gradle.kts
├── audio/                            # Audio processing module
│   ├── src/main/java/com/yourcompany/pipebandapp/audio/
│   │   ├── recording/                # AudioRecord wrapper
│   │   ├── playback/                 # AudioTrack wrapper
│   │   └── metronome/                # Metronome service
│   └── build.gradle.kts
└── build.gradle.kts                  # Root project Gradle file
```

**Module Dependencies:**
- app depends on domain, data, rendering, audio
- data depends on domain
- rendering depends on domain
- audio depends on domain
- domain has no Android dependencies (pure Kotlin)

### 7.2 Build and Deployment

**Build Variants:**
- debug: Development builds with debugging enabled
- release: Production builds with ProGuard/R8 optimization

**Build Types:**
- Debug: debuggable, minifyEnabled false, no ProGuard
- Release: not debuggable, minifyEnabled true, ProGuard enabled, signed

**Flavor Dimensions (Optional):**
- Environment: dev, staging, production
- API level: minApi29, minApi34 (different feature sets)

**Signing Configuration:**
- Debug builds: Automatic debug keystore
- Release builds: Release keystore with credentials from gradle.properties

### 7.3 Testing Strategy

**Unit Testing:**
- JUnit 5 for test framework
- MockK for Kotlin mocking
- Turbine for Flow testing
- Kotlinx-coroutines-test for coroutine testing
- 95%+ code coverage for domain layer
- 90%+ code coverage for use cases

**Instrumentation Testing:**
- AndroidX Test framework
- Espresso for UI testing
- Compose UI testing with onNodeWithText, etc.
- Hilt testing for dependency injection
- Test critical user workflows end-to-end
- Test on multiple device configurations

**CI/CD Pipeline:**
- GitHub Actions or GitLab CI
- Build on push/PR to main/develop branches
- Run unit tests and instrumentation tests
- Generate APK/AAB artifacts
- Upload to Play Console via Fastlane or Gradle Play Publisher
- Automated Play Store deployment to internal testing track

### 7.4 Debugging and Profiling

**Debugging Tools:**
- Android Studio debugger with breakpoints
- Logcat for log messages with structured logging
- Layout Inspector for Compose UI hierarchy
- Network Profiler for API calls
- Database Inspector for Room database
- Crashlytics for production crash reports

**Performance Profiling:**
- CPU Profiler for method tracing
- Memory Profiler for heap analysis
- Energy Profiler for battery consumption
- Network Profiler for data transfer
- Compose Performance Metrics
- Systrace for frame timing analysis

---

## 8. Performance Optimization

### 8.1 Layout Performance Strategies

**Incremental Layout Updates:**
- Track dirty regions with efficient data structures
- Determine minimal recalculation scope
- Use coroutines for parallel page calculations
- Cache layout results with LRU policy
- Invalidate only affected entries
- Measure and log performance metrics

**Performance Targets:**
- Measure layout: < 20ms per measure
- System layout: < 100ms per system
- Page layout: < 300ms per page
- Full document: < 5s for 100 pages (mid-range device)
- Incremental update: < 200ms for single change

### 8.2 Memory Management

**Large Document Handling:**
- Virtualize page rendering (render only visible viewport)
- Page rendering cache with size limit (5 pages)
- Bitmap recycling for memory efficiency
- Monitor memory pressure with onTrimMemory
- Release caches on low memory callback
- Use RecyclerView for paginated lists

**Memory Optimization:**
- Avoid memory leaks from ViewModels
- Use WeakReference for large cached objects
- Dispose of unused resources promptly
- Profile memory with Android Studio Memory Profiler
- Target max memory: 150MB on low-end devices

### 8.3 Rendering Performance

**Canvas Drawing Optimization:**
- Hardware acceleration via View.setLayerType(LAYER_TYPE_HARDWARE)
- Clip drawing to dirty regions with Canvas.clipRect
- Reuse Paint objects (don't create in onDraw)
- Cache complex paths with Path.op operations
- Use PorterDuff blend modes for effects
- Measure drawing performance with Systrace

**Compose Performance:**
- Minimize recomposition with stable/immutable data
- Use derivedStateOf for expensive calculations
- Defer state reads with Modifier.drawBehind
- Use keys in lists for efficient recomposition
- Profile with Compose Layout Inspector

---

## 9. Post-Launch Maintenance

### 9.1 Crash Monitoring

**Firebase Crashlytics Integration:**
- Automatic crash reporting in production
- Custom keys for debugging context
- Log breadcrumbs for user actions
- Non-fatal exceptions for recoverable errors
- User identification for crash correlation
- Alert on crash-free rate drop

**Error Handling:**
- Global exception handler for uncaught exceptions
- Graceful degradation on errors
- User-friendly error messages
- Retry logic for network errors
- Offline mode for core features

### 9.2 Performance Monitoring

**Firebase Performance Monitoring:**
- Automatic app start time tracking
- Screen rendering performance metrics
- Network request duration tracking
- Custom traces for critical operations
- Identify slow frames and jank
- Monitor battery consumption patterns

**Custom Metrics:**
- Layout calculation duration
- Document load time
- Export operation performance
- Audio processing latency
- Cache hit/miss ratios
- User action completion times

### 9.3 Update Strategy

**Play Store Update Management:**
- Automatic updates enabled by default
- In-app update API for flexible updates
- Immediate updates for critical fixes
- Flexible updates for feature releases
- Check for updates on app launch (optional)
- Handle update cancellation gracefully

**Update Types:**
- Immediate: Forces user to update before continuing
- Flexible: Downloads in background, applies later
- Use immediate for security patches
- Use flexible for feature updates

### 9.4 Analytics and User Feedback

**Analytics Integration:**
- Firebase Analytics for user behavior
- Track feature usage patterns
- Monitor user engagement metrics
- Funnel analysis for onboarding
- Cohort analysis for retention
- Always respect user privacy preferences

**User Feedback:**
- In-app feedback form
- Play Store review prompts (Google Play Core)
- Bug report functionality with logs
- Feature request submission
- Community forum links
- Email support integration

---

## 10. Platform-Specific Considerations

### 10.1 Android Version Compatibility

**API Level Support:**
- Minimum SDK: 29 (Android 10, released 2019)
- Target SDK: 34 (Android 14, latest stable)
- Compile SDK: 34 or higher

**Version-Specific Features:**
- Android 10 (API 29): Scoped storage, dark theme
- Android 11 (API 30): One-time permissions, auto-reset permissions
- Android 12 (API 31): Material You, splash screen API
- Android 13 (API 33): Photo picker, notification permissions
- Android 14 (API 34): Predictive back gesture, enhanced security

**Backwards Compatibility:**
- Use AndroidX libraries for compatibility
- Check SDK version with Build.VERSION.SDK_INT
- Provide fallbacks for newer APIs
- Test on multiple Android versions
- Use @RequiresApi annotation for clarity

### 10.2 Device Form Factor Optimization

**Phone Optimization:**
- Vertical scrolling for portrait orientation
- Single column layout in compact width
- Bottom navigation bar for primary sections
- Floating action button for primary action
- Responsive font scaling with sp units
- Touch target size minimum 48dp

**Tablet Optimization:**
- Two or three column layout in expanded width
- Navigation rail or permanent drawer
- Master-detail split view
- Dual-pane editing interface
- Larger canvas for score rendering
- Keyboard shortcuts for productivity

**Foldable Support:**
- Window metrics API for fold detection
- Adaptive layouts for folded/unfolded states
- Spanning mode for dual-screen display
- App continuity across fold transitions
- Hinge-aware layouts (avoid content on hinge)
- Test on Samsung Fold, Surface Duo emulators

**Chrome OS / Large Screen:**
- Window resizing support (resizeableActivity=true)
- Free-form window mode
- Keyboard and mouse input optimization
- Right-click context menus
- Hover effects for mouse pointers
- Desktop-class productivity features

### 10.3 Input Method Support

**Touch Input:**
- Minimum touch target: 48dp x 48dp
- Gesture support: pinch-to-zoom, two-finger pan
- Long-press for context menus
- Swipe gestures for navigation
- Haptic feedback on actions
- Palm rejection for stylus use

**Stylus Support:**
- Samsung S Pen integration
- Lenovo Precision Pen support
- Generic stylus input handling
- Pressure sensitivity (optional)
- Palm rejection algorithms
- Hover events for preview

**Keyboard and Mouse:**
- Keyboard shortcuts for common actions
- Tab navigation through UI elements
- Enter/Space for activation
- Escape for cancellation
- Mouse hover effects
- Right-click context menus

**Gamepad Support (Optional):**
- D-pad navigation
- A button for select, B button for back
- Triggers for zoom in/out
- Useful for Android TV version (future)

### 10.4 Accessibility

**TalkBack Support:**
- Meaningful content descriptions on all interactive elements
- Semantic heading structure
- Announce state changes clearly
- Navigate by headings, links, controls
- Support explore-by-touch
- Test thoroughly with TalkBack enabled

**Switch Access:**
- All interactive elements reachable
- Logical navigation order
- Group related elements
- Support point scanning
- Action menu for multi-step operations

**Other Accessibility Features:**
- Support for large text sizes (up to 200%)
- High contrast mode compatibility
- Reduce motion preferences
- Screen reader friendly error messages
- Keyboard-only navigation
- Color contrast ratios (WCAG 2.1 AA minimum)

---

## 11. Security and Privacy

### 11.1 Data Protection

**Encryption Requirements:**
- HTTPS/TLS for all network communications
- Encrypted device storage (user must enable)
- Keystore for sensitive credentials
- Certificate pinning for API calls (optional)
- ProGuard/R8 obfuscation for release builds
- No sensitive data in logs (release builds)

**Secure Storage:**
- EncryptedSharedPreferences for sensitive preferences
- Room database encryption with SQLCipher (optional)
- Keystore-backed encryption keys
- Biometric authentication for app unlock (optional)
- Secure file deletion (overwrite before delete)

### 11.2 Privacy Compliance

**GDPR Compliance:**
- Data minimization principle
- Explicit consent for data collection
- Right to access (export all user data)
- Right to erasure (delete all user data)
- Data portability (.pbscore export)
- Privacy policy easily accessible

**Google Play Policies:**
- Data Safety form accurately filled
- Permissions used only as declared
- No surprise data collection
- User control over data sharing
- Clear privacy policy
- Children's privacy (COPPA) if targeting kids

**User Rights:**
- In-app privacy settings
- Data export functionality
- Account deletion option
- Opt-out of analytics
- Review and revoke permissions
- Transparency about data usage

### 11.3 Secure Coding Practices

**Input Validation:**
- Validate all user input
- Sanitize file paths and URIs
- Prevent injection attacks
- Validate MIME types
- Check file sizes before processing
- Rate limiting for network requests

**Authentication Security:**
- Use OAuth 2.0 for Google Sign-In
- Never store passwords locally
- Token refresh on expiration
- Secure token storage in Keystore
- Logout clears all tokens
- Handle account removal gracefully

---

## 12. Distribution Strategy

### 12.1 Google Play Store Primary Distribution

**Release Tracks:**
- Internal Testing: For QA team (up to 100 testers)
- Closed Testing: For beta testers (up to 1000 testers per track)
- Open Testing: For public beta (unlimited)
- Production: For all users

**Staged Rollout:**
- Start with 5% rollout
- Monitor crash-free rate and ratings
- Increase to 10%, 25%, 50%, 100%
- Halt rollout if issues detected
- Rollback if critical bugs found

**App Bundle Benefits:**
- Smaller download sizes (40% average reduction)
- Automatic APK generation per device configuration
- Language-specific APK splitting
- Dynamic feature modules (optional)
- Play Feature Delivery

### 12.2 Alternative Distribution Channels

**Samsung Galaxy Store:**
- Additional distribution channel for Samsung devices
- Similar app listing process
- May reach users not on Google Play
- Requires separate developer account

**Amazon Appstore:**
- Distribution for Amazon Fire tablets
- May require compatibility adjustments
- Additional revenue opportunity
- Separate developer portal

**Direct APK Distribution (Advanced):**
- For enterprise deployments
- Install from unknown sources (user must enable)
- No automatic updates
- Self-managed distribution
- Use for beta testing outside Play Store

**Considerations:**
- Maintain feature parity across stores
- Separate build variants for each store (if needed)
- Monitor each store separately
- Update all stores simultaneously (when possible)

### 12.3 Monetization Options (Future)

**Free with Optional IAP:**
- Base app free forever
- Premium features via in-app purchases
- Google Play Billing Library integration
- Subscription model for cloud features
- One-time purchases for export formats
- No ads in music notation app (poor UX)

**Potential Premium Features:**
- Advanced embellishment library
- Real-time collaboration (beyond basic sharing)
- Unlimited cloud storage
- Priority support
- Professional templates
- Batch export operations

---

## 13. Localization and Internationalization

### 13.1 Language Support

**Target Languages (Priority):**
1. English (US) - Default
2. English (UK) - British spellings
3. French (France)
4. Spanish (Spain)
5. German (Germany)

**Implementation:**
- Strings externalized in strings.xml
- Separate strings.xml per language (values-fr, values-es, etc.)
- Plurals support with quantity strings
- RTL layout support (Arabic, Hebrew if added)
- Date/time formatting per locale
- Number formatting per locale

### 13.2 Cultural Adaptation

**Musical Terminology:**
- Preserve Scottish Gaelic terms with translations
- Provide glossary of pipe band terms
- Educational content for non-native speakers
- Regional embellishment style variations
- Respect cultural authenticity

**Regional Preferences:**
- Paper sizes: A4 (international) vs Letter (US)
- Date formats: DD/MM/YYYY vs MM/DD/YYYY
- Measurement units: Metric vs Imperial
- First day of week: Monday vs Sunday
- Time format: 24-hour vs 12-hour

---

## 14. Testing and Quality Assurance

### 14.1 Test Coverage Requirements

**Unit Tests:**
- Domain layer: 95%+ code coverage
- Use cases: 90%+ code coverage
- Validation logic: 95%+ code coverage
- Repository implementations: 85%+ code coverage

**Instrumentation Tests:**
- Critical user flows: 100% coverage
- UI components: 80%+ coverage
- Database operations: 90%+ coverage
- File operations: 85%+ coverage

**UI Tests:**
- Compose UI tests with semantics
- Screenshot tests for visual regression
- Accessibility tests with TalkBack
- Performance tests with Macrobenchmark

### 14.2 Device Testing Matrix

**Required Test Devices:**
- Pixel 6 or newer (stock Android 14)
- Samsung Galaxy S23 (One UI)
- Tablet: Galaxy Tab S9 (large screen)
- Budget phone: < $200 device (performance baseline)
- Foldable: Galaxy Z Fold 5 (optional)

**Emulator Testing:**
- Multiple API levels (29, 31, 33, 34)
- Different screen sizes and densities
- Various system languages
- Low RAM configurations
- Hardware-accelerated emulators

### 14.3 Beta Testing Program

**Open Beta on Play Store:**
- Recruit 100-500 beta testers
- Real-world usage patterns
- Crash and ANR reports
- User feedback collection
- Performance metrics
- Iterate before production release

**Feedback Collection:**
- In-app feedback form
- Google Play Console reviews
- Firebase Crashlytics
- Firebase Performance
- User surveys
- Community forums

---

## 15. Documentation

### 15.1 Developer Documentation

**Code Documentation:**
- KDoc comments for public APIs
- README files for each module
- Architecture decision records (ADR)
- API documentation with Dokka
- Contributing guidelines
- Code style guide (Kotlin conventions)

**Technical Documentation:**
- Setup instructions for development environment
- Build and deployment procedures
- Testing guidelines
- Performance optimization tips
- Debugging common issues
- Architecture overview diagrams

### 15.2 User Documentation

**Help System:**
- In-app help and tutorials
- Getting started guide
- Feature explanations
- Video tutorials (YouTube)
- FAQ page
- Troubleshooting guide

**Support Resources:**
- Support email: support@yourdomain.com
- Knowledge base website
- Community forum
- Social media presence
- Bug reporting instructions
- Feature request process

---

## 16. Summary and Best Practices

### 16.1 Key Architectural Decisions

**Kotlin and Coroutines:**
- Type-safe language with null safety
- Structured concurrency for async operations
- Flow for reactive programming
- Proper cancellation and exception handling
- Efficient threading with Dispatchers

**Storage Strategy:**
- Google Drive mode as primary (seamless Google Account integration)
- Storage Access Framework for universal cloud support
- Universal .pbscore format for cross-platform compatibility
- Scoped storage compliance

**Performance:**
- Incremental layout calculation
- LRU caching for layout results
- Hardware-accelerated rendering
- Virtualized page rendering for large documents

**Play Store Compliance:**
- All permissions properly declared and justified
- Privacy policy and Data Safety form complete
- No prohibited APIs or behaviors
- Material Design 3 and modern Android UX

### 16.2 Development Priorities

**Phase 1 (MVP):**
1. Document-based architecture with Google Drive
2. Basic score editing (notes, measures, parts)
3. SMuFL rendering with Canvas
4. Simple embellishment support
5. PDF export

**Phase 2 (Enhancement):**
1. Real-time collaboration via Drive API
2. Audio recording and playback
3. Metronome with time signature support
4. Complete embellishment system
5. Storage Access Framework mode

**Phase 3 (Polish):**
1. Advanced layout optimization
2. Stylus/S Pen integration
3. Tablet-optimized UI
4. Widgets for home screen
5. Chrome OS optimization

### 16.3 Maintenance Checklist

**Regular Tasks:**
- [ ] Monitor crash reports via Firebase Crashlytics
- [ ] Review Play Store ratings and respond to reviews
- [ ] Update for new Android versions annually
- [ ] Test on new device models as released
- [ ] Update Data Safety form if data collection changes
- [ ] Refresh marketing screenshots for new Android versions
- [ ] Monitor performance metrics via Firebase Performance
- [ ] Update dependencies (AndroidX, Kotlin, Gradle)
- [ ] Run security audits quarterly
- [ ] Test Google Drive API changes

**Quarterly Reviews:**
- [ ] Analyze crash-free rate trends
- [ ] Review user feedback themes
- [ ] Assess performance metrics
- [ ] Update documentation
- [ ] Plan next feature releases
- [ ] Review competitor apps

---

## 17. Glossary

**Android-Specific Terms:**
- **Activity**: UI screen component
- **Fragment**: Modular UI component within Activity
- **ViewModel**: UI-related data holder surviving configuration changes
- **LiveData**: Observable data holder (replaced by Flow in modern apps)
- **StateFlow**: Hot observable state holder
- **SharedFlow**: Hot observable event stream
- **Room**: SQLite database abstraction library
- **Hilt**: Dependency injection framework based on Dagger
- **Jetpack Compose**: Declarative UI toolkit
- **Material You**: Dynamic theming system in Material Design 3
- **SAF**: Storage Access Framework for universal file access
- **Scoped Storage**: Privacy feature restricting direct file system access

**Musical Terms:**
- See Glossary in `pipe_band_app_architecture.md` Section 10

---

## 18. References

**Android Documentation:**
- Android Developer Guides: https://developer.android.com/guide
- Jetpack Compose Documentation: https://developer.android.com/jetpack/compose
- Material Design 3: https://m3.material.io
- Google Play Console Help: https://support.google.com/googleplay/android-developer

**Libraries and Frameworks:**
- Kotlin Coroutines: https://kotlinlang.org/docs/coroutines-overview.html
- Hilt Dependency Injection: https://dagger.dev/hilt
- Room Database: https://developer.android.com/training/data-storage/room
- Google Drive API: https://developers.google.com/drive/api/v3/about-sdk

**Musical Standards:**
- Standard Music Font Layout (SMuFL) Specification 1.4+
- MusicXML 4.0 Specification
- MIDI 1.0 Specification
- Traditional Scottish pipe band notation conventions

**Platform Documentation:**
- iOS/macOS: See `ios_macos_implementation.md`
- Windows: See `windows_implementation.md`
- Linux: See `linux_implementation.md`
- UX Patterns: See `ux_interaction_design.md`

**Architectural Patterns:**
- Clean Architecture (Robert C. Martin)
- Domain-Driven Design (Eric Evans)
- Repository Pattern
- MVVM Pattern

---

## 19. Document Maintenance

**Version History:**
- Version 1.0 (2025-10-06): Initial Android platform specification

**Change Process:**
- Platform-specific changes reviewed by Android team
- Domain layer changes coordinated across all platforms
- Breaking changes require major version increment
- UX changes aligned with Material Design guidelines

**Review Schedule:**
- Quarterly review of Android-specific decisions
- Annual comprehensive specification review
- Ad-hoc reviews for major Android OS releases
- Post-release retrospectives for lessons learned

---

**End of Android Implementation Specification**

This specification provides comprehensive guidance for implementing the Pipe Band Score application on Android with full Google Play Store compliance. All implementations must adhere to the domain architecture defined in `pipe_band_app_architecture.md` and the interaction patterns in `ux_interaction_design.md` while leveraging Android-native capabilities for optimal user experience on phones, tablets, foldables, and Chrome OS devices.