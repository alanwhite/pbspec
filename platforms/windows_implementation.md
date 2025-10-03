# Windows Implementation Specification

**Version:** 1.0  
**Last Updated:** October 03, 2025  
**Platforms:** Windows 10 (Build 19041+), Windows 11  
**Primary Distribution:** Microsoft Store

---

## Document Purpose and Scope

This document specifies the **Windows platform-specific implementation** of the Scottish pipe band application, building upon the platform-agnostic domain architecture defined in `pipe_band_app_architecture.md`.

**Reference Architecture:**
- **Domain Specification:** `pipe_band_app_architecture.md`
- **Domain Entities:** Section 3.1 (Instruments, Score, Tune, Part, Measure, Note, Embellishments)
- **Use Cases:** Section 3.4 (Score Management, Musical Editing, Pagination)
- **Repository Interfaces:** Section 3.5 (Data access contracts)
- **File Format:** Section 4 (.pbscore JSON specification)

**What This Document Contains:**
- C# 12+ and .NET 8+ architecture with async/await patterns
- WinUI 3 presentation layer with Fluent Design System
- Microsoft Store distribution requirements and compliance
- OneDrive integration + File Provider storage modes
- Windows Media Foundation audio processing
- Direct2D + DirectWrite SMuFL rendering
- Platform-specific performance optimizations

**What This Document Does NOT Contain:**
- Domain logic definitions (see core architecture document)
- Business rules specifications (see core architecture document)
- Cross-platform architectural patterns (see core architecture document)
- Musical notation concepts (see core architecture document)

---

## 1. Technology Stack and Architecture

### 1.1 Core Technologies

**Language and Framework:**
- **C# 12.0+** with nullable reference types enabled
- **.NET 8.0+** (Long Term Support release)
- **Async/await patterns** throughout with ConfigureAwait best practices
- **Task-based asynchronous programming** (TAP)
- **IAsyncEnumerable** for streaming data operations

**UI Framework:**
- **WinUI 3** (Windows App SDK 1.5+) as primary UI framework
- **Fluent Design System** compliance for modern Windows look
- **XAML** for declarative UI with C# code-behind
- **Composition APIs** for advanced animations and effects
- **Win2D** for custom canvas rendering (optional)

**Architecture Pattern:**
- **MVVM** (Model-View-ViewModel) with dependency injection
- **Repository Pattern** with async implementations
- **Service Layer** for cross-cutting concerns
- **Clean Architecture** layers maintained through project boundaries

**Audio Processing:**
- **Windows Media Foundation** for audio playback and recording
- **NAudio** library for advanced audio processing (optional)
- **WASAPI** (Windows Audio Session API) for low-latency audio
- **Audio Graph API** for complex audio routing

**Local Storage:**
- **Entity Framework Core 8+** with SQLite backing store
- **Windows Storage APIs** for file system access
- **Async repository implementations** with cancellation support
- **Background task processing** with proper lifecycle management

**Music Notation:**
- **Direct2D** for hardware-accelerated vector rendering
- **DirectWrite** for SMuFL text layout and rendering
- **Win2D** for XAML-integrated Direct2D rendering
- **Composition APIs** for smooth scrolling and zooming

**Cloud Integration:**
- **OneDrive API** via Microsoft Graph SDK (primary mode)
- **Windows Storage Pickers** for universal file access (File Provider mode)
- **Background transfer API** for reliable file synchronization
- **Microsoft Graph** for real-time collaboration features

### 1.2 Microsoft Store Distribution Requirements

**Required Capabilities:**
```xml
Package.appxmanifest Configuration:
├── Identity
│   └── Package identity and publisher information
├── Capabilities
│   ├── internetClient (cloud sync)
│   ├── internetClientServer (collaboration)
│   ├── musicLibrary (optional - score library)
│   ├── removableStorage (external drive support)
│   ├── microphone (practice recording)
│   └── backgroundMediaPlayback (metronome)
├── Declarations
│   ├── File Type Associations (.pbscore)
│   ├── Share Target (receive scores)
│   └── Background Tasks (sync, audio)
└── Visual Assets
    ├── App icons (all sizes)
    ├── Splash screen
    ├── Tile images (small, medium, wide, large)
    └── Store logo
```

**Privacy and Permissions:**
```xml
Capability Declarations:
<Capabilities>
  <!-- Network access for cloud sync -->
  <Capability Name="internetClient" />
  <Capability Name="internetClientServer" />
  
  <!-- File system access -->
  <uap:Capability Name="musicLibrary" />
  <uap:Capability Name="removableStorage" />
  
  <!-- Audio recording -->
  <DeviceCapability Name="microphone" />
  
  <!-- Background audio for metronome -->
  <uap3:Capability Name="backgroundMediaPlayback" />
</Capabilities>
```

**File Type Association:**
```xml
<Extensions>
  <uap:Extension Category="windows.fileTypeAssociation">
    <uap:FileTypeAssociation Name="pbscore">
      <uap:SupportedFileTypes>
        <uap:FileType>.pbscore</uap:FileType>
      </uap:SupportedFileTypes>
      <uap:DisplayName>Pipe Band Score</uap:DisplayName>
      <uap:Logo>Assets\FileIcon.png</uap:Logo>
      <uap:EditFlags OpenIsSafe="true" />
    </uap:FileTypeAssociation>
  </uap:Extension>
</Extensions>
```

**Microsoft Store Compliant Technologies:**
```
Allowed Technologies:
├── Windows App SDK (WinUI 3)
├── Microsoft Graph SDK (OneDrive integration)
├── Windows Community Toolkit
├── Entity Framework Core
├── NAudio (audio processing)
├── Win2D (canvas rendering)
├── CommunityToolkit.Mvvm
└── Microsoft.Extensions.* (DI, logging, etc.)

Prohibited Technologies:
├── ❌ Win32 APIs requiring elevated permissions
├── ❌ Kernel drivers or system-level hooks
├── ❌ Unsigned native code or DLLs
├── ❌ Web views for core functionality (policy 10.2)
├── ❌ Hot code push or dynamic feature delivery
├── ❌ Private API usage or undocumented APIs
└── ❌ Third-party authentication without Microsoft Account option
```

---

## 2. Storage Architecture Implementation

### 2.1 Two-Tier Storage Strategy

**Storage Mode Enumeration:**
```csharp
public enum StorageMode
{
    OneDrive,        // Primary, recommended for Microsoft Store
    FileProvider     // Advanced users, universal cloud support
}
```

**Strategic Recommendation for Microsoft Store:**

**OneDrive Mode (Primary)** is strongly recommended because:
- Seamless integration with user's Microsoft Account (99%+ Windows 10/11 users)
- Automatic privacy compliance through Microsoft's infrastructure
- Enhanced Store discoverability ("Works with OneDrive" badge)
- Reduced App Review friction (well-understood Microsoft technology)
- Native collaboration UI via Microsoft Graph
- Zero-configuration user experience
- Automatic encryption and security compliance
- Real-time collaboration features built-in
- Cross-device sync across Windows, Xbox, iOS, Android

**File Provider Mode (Advanced)** is available for:
- Enterprise requirements with specific cloud provider mandates
- Cross-platform band collaboration needing specific cloud services
- Power users preferring specific cloud storage providers
- Organizations with existing cloud infrastructure investments

### 2.2 File-Based Application Architecture

**Windows App SDK File System Integration:**

Platform implementations must provide file handling services using Windows Storage APIs with proper async patterns, file coordination, and cloud provider integration.

**Key Requirements:**
- Use FileSavePicker and FileOpenPicker for user-initiated file operations
- Implement CachedFileManager for cloud provider coordination
- Maintain FutureAccessList for quick file access
- Add files to MostRecentlyUsedList for OS integration
- Support file type associations for .pbscore files
- Handle concurrent access with proper locking

### 2.3 OneDrive Mode Implementation

**Microsoft Graph SDK Integration:**

Platform implementations must integrate with OneDrive using Microsoft Graph SDK for:
- File upload/download with chunked transfer for large files
- Real-time collaboration via sharing and permissions
- Delta sync for efficient change tracking
- Conflict resolution with user intervention
- Background synchronization with proper task scheduling

**Key Components:**
- Authentication via Microsoft Identity Client (MSAL)
- Graph SDK for OneDrive API access
- Background tasks for sync coordination
- Conflict resolution UI for user decisions
- Progress tracking for long-running operations

### 2.4 File Provider Mode Implementation

**Universal File Access via Storage Pickers:**

Platform implementations must support universal file access through:
- Storage pickers for save/open operations
- FutureAccessList for maintaining file access permissions
- NSFileCoordinator equivalent coordination for safe access
- Support for any cloud provider via Windows File Provider system
- Automatic handling of cloud sync status

### 2.5 First Launch Experience (Microsoft Store Compliant)

**Storage Mode Selection and Setup:**

Platform implementations must provide onboarding flow that:
- Detects Microsoft Account availability automatically
- Explains OneDrive benefits without being pushy
- Offers File Provider mode as alternative
- Guides users to enable Microsoft Account if needed
- Never blocks users from using the app
- Stores user preference for future launches

**Onboarding Steps:**
1. Welcome screen with app overview
2. Check for Microsoft Account availability
3. If available: Explain OneDrive benefits and recommend
4. If unavailable: Show setup guidance or offer File Provider mode
5. Complete onboarding and proceed to main app

**Permission Request Pattern (Microsoft Store Compliant):**
- Request microphone only when user taps "Record Practice Session"
- Request notifications only after user creates first score
- Show pre-permission explanation dialogs
- Guide to Settings if permission denied
- Never repeatedly prompt for denied permissions

---

## 3. .NET Architecture with Async Patterns

### 3.1 Dependency Injection and Service Configuration

**Application Startup:**

Platform implementations must configure dependency injection container at app startup with:
- ViewModels (transient or scoped as appropriate)
- Domain services (singleton for stateless, scoped for stateful)
- Use cases (transient for per-operation logic)
- Repositories (singleton or scoped based on storage mode)
- Infrastructure services (file, audio, rendering)
- Platform services (permissions, navigation, dialogs)
- Database context (scoped with proper disposal)

**Service Lifetimes:**
- **Transient**: ViewModels, Use Cases (new instance per request)
- **Scoped**: Database contexts, per-request services
- **Singleton**: Domain services, caches, configuration

### 3.2 Actor-Based Domain Layer (Task-Based Async)

**Domain Coordinator Pattern:**

Platform implementations must provide async coordinators that:
- Use SemaphoreSlim for thread-safe operations
- Implement ConcurrentDictionary for in-memory caching
- Coordinate between repositories and domain services
- Validate business rules before persistence
- Manage document lifecycle (open, edit, save, close)
- Implement proper cancellation token support
- Handle errors with appropriate recovery strategies

**Key Patterns:**
- Async/await throughout (no blocking calls)
- CancellationToken support in all async methods
- ConfigureAwait(false) in library code
- ConfigureAwait(true) or default in UI code
- Proper disposal with IAsyncDisposable where appropriate

### 3.3 Concurrent Layout Calculation

**Layout Calculation Engine:**

Platform implementations must provide layout calculation with:
- Parallel processing using Parallel.ForEachAsync
- Concurrent dictionaries for thread-safe caching
- MaxDegreeOfParallelism based on processor count
- Cancellation token support for responsive cancellation
- Cache invalidation with minimal scope
- LRU eviction for memory management
- Incremental recalculation for performance

**Performance Goals:**
- Large document pagination: < 2 seconds for 100 pages
- Incremental layout update: < 100ms for single measure change
- Parallel efficiency: 70%+ utilization on 4+ cores
- Cache hit rate: 80%+ for typical editing workflows

---

## 4. UI Implementation with WinUI 3

### 4.1 Score Editor View Hierarchy

**Main Editor Components:**

Platform implementations must provide:
- CommandBar with primary editing actions
- ScrollViewer with zoom support (0.25x to 4.0x)
- Custom ScoreCanvas control for rendering
- Loading overlay with ProgressRing
- Status bar with mode and position indicators
- Context menus for element-specific actions
- Keyboard shortcuts for all actions

**XAML Requirements:**
- Use x:Bind for performance (compiled bindings)
- Implement proper data binding with Mode=OneWay/TwoWay
- Use ThemeResource for dynamic theming
- Support both light and dark themes
- Implement proper accessibility with AutomationProperties
- Use SystemControlAcrylicWindowBrush for translucency effects

### 4.2 SMuFL Rendering with Direct2D

**Direct2D Score Renderer:**

Platform implementations must render scores using:
- CanvasDrawingSession for Direct2D rendering
- SMuFL fonts (Bravura recommended) with proper Unicode codepoints
- Vector-based rendering (no rasterization)
- Proper staff line positioning (5 lines, 4 spaces)
- Correct musical symbol placement
- Grace note rendering with slashed stems
- Beaming for grace note groups
- Barline rendering with proper types
- Time signature and key signature rendering
- Clef rendering with correct positioning

**Rendering Pipeline:**
1. Clear canvas to white background
2. Apply zoom transform
3. Render each page's tune lines
4. For each system: render staff lines, clefs, keys, time signatures
5. For each measure: render notes, embellishments, barlines
6. Handle collision avoidance for overlapping elements
7. Apply optional visual effects (shadows, etc.)

### 4.3 Custom Score Canvas Control

**ScoreCanvas Implementation:**

Platform implementations must provide custom canvas control that:
- Extends CanvasControl from Win2D
- Implements dependency properties for data binding
- Handles pointer events (press, move, release, right-tap)
- Implements hit testing to find elements at position
- Invalidates and redraws on property changes
- Supports zoom and pan gestures
- Maintains selection state visually
- Coordinates with ViewModel for user actions

---

## 5. Audio Processing with Windows Media Foundation

### 5.1 Audio Engine Implementation

**Audio Processing Engine Interface:**

Platform implementations must provide audio engine that:
- Initializes AudioGraph with lowest latency settings
- Creates device input/output nodes
- Generates metronome click sounds programmatically
- Records audio to M4A format with high quality
- Performs pitch analysis using FFT
- Handles device loss and recreation gracefully
- Manages audio session lifecycle properly
- Respects microphone permissions

**Audio Features:**
- Metronome with configurable tempo and accent patterns
- Audio recording to persistent storage
- Pitch analysis for practice feedback
- Background audio support (metronome while app inactive)
- Proper audio session management for interruptions

### 5.2 Metronome Service

**Metronome Coordinator:**

Platform implementations must provide metronome service that:
- Starts/stops metronome on demand
- Generates beat events for visual feedback
- Creates accent patterns based on time signature
- Maintains accurate timing across long durations
- Handles tempo changes dynamically
- Supports cancellation for responsive stopping
- Raises events for UI synchronization

---

## 6. Microsoft Store Compliance Checklist

### 6.1 Pre-Submission Requirements

**Technical Requirements:**
- ✅ Package.appxmanifest properly configured with all capabilities
- ✅ Privacy policy URL included (required for microphone usage)
- ✅ All visual assets provided (logos, tiles, splash screen)
- ✅ No crashes on fresh install and basic usage
- ✅ Application certificate properly signed
- ✅ Runs on Windows 10 (Build 19041+) and Windows 11
- ✅ Proper version numbering (major.minor.build.revision)

**Content Requirements:**
- ✅ Store listing complete with accurate descriptions
- ✅ Screenshots show actual app functionality (no mockups)
- ✅ Age rating accurately reflects content (PEGI 3 / ESRB Everyone)
- ✅ Privacy policy explains data collection clearly
- ✅ Support contact information provided
- ✅ Keywords relevant and not misleading

**Functionality Requirements:**
- ✅ All advertised features fully implemented
- ✅ No placeholder content or "coming soon" features
- ✅ Proper error handling with user-friendly messages
- ✅ Acceptable performance on minimum spec hardware
- ✅ Supports both light and dark themes
- ✅ Keyboard navigation for all interactive elements

### 6.2 Microsoft Store Connect Configuration

**App Identity:**
```
Store Configuration:
├── App Name: "Pipe Band Score Editor"
├── Category: Music & Video > Music Production
├── Subcategory: Music Creation
├── Age Rating: PEGI 3 / ESRB Everyone
├── Price: Free (initial release - IAP possible in future)
├── Market Availability: All markets
└── Platform Support: Windows 10 (19041+), Windows 11
```

**Privacy Configuration:**
```
Privacy Policy Requirements:
├── Privacy Policy URL: [Required - must be accessible]
├── Data Collection Disclosure:
│   ├── Musical Scores: User-created content (OneDrive mode)
│   ├── Audio Recordings: Practice sessions (local or OneDrive)
│   ├── User Account: Microsoft Account (OneDrive mode)
│   ├── Device ID: For OneDrive sync coordination
│   └── Usage Analytics: Optional (with user consent)
├── Data Sharing: None with third parties
├── Data Retention: User-controlled (can delete anytime)
└── GDPR Compliance: Full support for user rights
```

**Store Listing:**
```
Marketing Information:
├── Description: 
│   └── Professional pipe band music notation editor with authentic
│       Scottish embellishments, real-time collaboration via OneDrive,
│       and publication-quality PDF export. Supports bagpipes, snare
│       drum, tenor drum, and bass drum with SMuFL-compliant rendering.
│
├── Features:
│   ├── • Authentic pipe band embellishments (doublings, grips, throws)
│   ├── • Real-time collaboration via OneDrive
│   ├── • SMuFL-compliant music notation rendering
│   ├── • PDF, MusicXML, and MIDI export
│   ├── • Practice recording with pitch analysis
│   ├── • Integrated metronome with accent patterns
│   └── • Universal .pbscore file format
│
├── Screenshots (Required - minimum 1, recommended 4-8):
│   ├── 1920x1080 or 3840x2160 resolution
│   ├── Must show actual app interface
│   ├── Capture key features (editor, embellishments, export)
│   └── No added text or overlays (pure screenshots)
│
├── Keywords:
│   └── pipe band, bagpipes, snare drum, music notation, score editor,
│       scottish music, embellishments, SMuFL, music composition
│
└── Support Contact:
    ├── Email: support@yourdomain.com
    ├── Website: https://yourdomain.com/support
    └── Privacy Policy: https://yourdomain.com/privacy
```

### 6.3 Package Manifest Configuration

**Package.appxmanifest Structure:**

Platform implementations must provide complete manifest with:
- Identity (Name, Publisher, Version)
- Properties (DisplayName, PublisherDisplayName, Logo)
- Dependencies (TargetDeviceFamily with min/max versions)
- Resources (Languages)
- Application (Executable, EntryPoint, VisualElements)
- Extensions (FileTypeAssociation, ShareTarget, BackgroundTasks)
- Capabilities (internetClient, microphone, etc.)

**Visual Assets Requirements:**
- Square150x150Logo: 150x150 pixels (medium tile)
- Square44x44Logo: 44x44 pixels (app list icon)
- Wide310x150Logo: 310x150 pixels (wide tile)
- Square71x71Logo: 71x71 pixels (small tile)
- Square310x310Logo: 310x310 pixels (large tile)
- SplashScreen: 620x300 pixels
- StoreLogo: 50x50 pixels

### 6.4 Build Configuration

**Project File Settings:**

Platform implementations must configure .csproj with:
- OutputType: WinExe
- TargetFramework: net8.0-windows10.0.19041.0
- Platforms: x64, ARM64
- UseWinUI: true
- EnableMsixTooling: true
- Nullable: enable
- LangVersion: 12.0
- Optimization settings for Release builds
- Package references for all required NuGet packages

**Required NuGet Packages:**
- Microsoft.WindowsAppSDK
- Microsoft.Windows.SDK.BuildTools
- CommunityToolkit.WinUI.UI.Controls
- CommunityToolkit.Mvvm
- Microsoft.EntityFrameworkCore.Sqlite
- Microsoft.Graph
- Microsoft.Identity.Client
- NAudio
- Microsoft.Graphics.Win2D
- System.Text.Json
- Microsoft.Extensions.Hosting

---

## 7. Development Workflow

### 7.1 Solution Structure

**Visual Studio Solution Organization:**

```
PipeBandApp.sln
├── src/
│   ├── PipeBandApp/                    # Main Windows App SDK project
│   ├── PipeBandApp.Domain/             # Domain Layer (shared logic)
│   ├── PipeBandApp.Infrastructure/     # Infrastructure Layer
│   └── PipeBandApp.Background/         # Background Tasks project
├── tests/
│   ├── PipeBandApp.Domain.Tests/
│   ├── PipeBandApp.Infrastructure.Tests/
│   └── PipeBandApp.UI.Tests/
└── docs/
    ├── architecture/
    └── api/
```

**Project Organization:**
- Main app: Views, ViewModels, Controls, Converters, Services, Assets
- Domain: Entities, UseCases, Repositories (interfaces), Services (interfaces)
- Infrastructure: Persistence, Cloud, Audio, Rendering, Serialization
- Background: Background task implementations
- Tests: Unit tests, integration tests, UI tests

### 7.2 Build and Packaging

**Build Configurations:**
- Debug: No optimization, full debugging symbols
- Release: Full optimization, ReadyToRun compilation, no debug symbols

**Publishing Profiles:**
- win-x64.pubxml: x64 architecture targeting
- win-arm64.pubxml: ARM64 architecture targeting
- Both: SelfContained=true, PublishReadyToRun=true

**Package Creation:**
- Use makeappx to create MSIX packages
- Sign with signtool using code signing certificate
- Create bundle for multi-architecture distribution
- Test installation on clean Windows instance

### 7.3 Testing Strategy

**Unit Testing:**
- xUnit for test framework
- Moq for mocking dependencies
- 95%+ code coverage for domain layer
- 90%+ code coverage for use cases
- Test all business rules and validation logic

**UI Testing:**
- WinAppDriver for UI automation
- Test critical user workflows
- Verify accessibility with screen reader
- Test on multiple display scales (100%, 125%, 150%, 200%)
- Test both light and dark themes

### 7.4 Continuous Integration

**GitHub Actions Workflow:**
- Trigger on push/PR to main/develop branches
- Setup .NET 8 SDK
- Setup MSBuild
- Restore dependencies
- Build solution in Release configuration
- Run unit tests with code coverage
- Build MSIX packages for x64 and ARM64
- Upload artifacts for deployment

---

## 8. Performance Optimization

### 8.1 Layout Performance Strategies

**Incremental Layout Updates:**

Platform implementations must optimize layout calculation by:
- Tracking dirty regions (changed entities)
- Determining minimal recalculation scope
- Using parallel processing for independent calculations
- Caching layout results with LRU eviction
- Invalidating only affected cache entries
- Measuring layout performance with metrics

**Performance Targets:**
- Measure layout: < 10ms per measure
- System layout: < 50ms per system
- Page layout: < 200ms per page
- Full document: < 2s for 100 pages
- Incremental update: < 100ms for single change

### 8.2 Memory Management

**Large Document Handling:**

Platform implementations must manage memory efficiently by:
- Virtualizing page rendering (render only visible pages)
- Caching rendered pages with size limit
- Evicting least recently used pages
- Disposing graphics resources promptly
- Monitoring memory pressure
- Responding to low memory notifications

**Memory Limits:**
- Page cache: Maximum 5 pages
- Layout cache: Maximum 1000 entries
- Graphics resources: Dispose after use
- Background tasks: Limit memory allocation

### 8.3 Rendering Performance

**Hardware Acceleration:**

Platform implementations must leverage hardware acceleration by:
- Using Win2D for canvas rendering
- Leveraging Direct2D for vector graphics
- Using GPU for complex effects
- Caching rendered textures where beneficial
- Handling device loss gracefully
- Profiling rendering performance

---

## 9. Post-Launch Maintenance

### 9.1 Crash and Error Monitoring

**Telemetry Integration:**

Platform implementations should integrate telemetry for:
- Exception tracking with Application Insights
- Event tracking for user actions
- Performance metric tracking
- Page view tracking
- Custom metric tracking

**Error Handling:**
- Global exception handlers for unhandled exceptions
- Graceful degradation on errors
- User-friendly error messages
- Automatic error reporting
- Optional user feedback collection

### 9.2 Update Strategy

**Microsoft Store Auto-Update:**

Platform implementations benefit from automatic updates, but can:
- Check for updates programmatically
- Prompt user to download updates
- Track update success/failure
- Provide release notes to users
- Support rollback if needed

---

## 10. Summary and Best Practices

### 10.1 Key Architectural Decisions

**C# and .NET Platform:**
- Complete type safety with nullable reference types
- Async/await throughout for responsive UI
- Task-based concurrency for multi-core utilization
- Strong dependency injection for testability

**Storage Strategy:**
- OneDrive mode as primary (Microsoft Account integration)
- File Provider mode for advanced/enterprise users
- Universal .pbscore format for cross-platform compatibility
- Proper file coordination for cloud safety

**Performance:**
- Incremental layout with intelligent caching
- Parallel calculation on multi-core systems
- Hardware-accelerated rendering with Win2D/Direct2D
- Virtualized rendering for large documents

**Microsoft Store Compliance:**
- All capabilities properly declared
- Privacy policy and data handling transparency
- No prohibited technologies or APIs
- Universal Windows Platform best practices

### 10.2 Development Priorities

**Phase 1 (MVP):**
1. Document-based architecture with OneDrive
2. Basic score editing (notes, measures, parts)
3. SMuFL rendering with Direct2D
4. Simple embellishment support
5. PDF export

**Phase 2 (Enhancement):**
1. Real-time collaboration via Microsoft Graph
2. Audio recording and playback
3. Metronome with time signature support
4. Complete embellishment system
5. File Provider mode

**Phase 3 (Polish):**
1. Advanced layout optimization
2. Win2D hardware acceleration
3. Surface Pen integration
4. Widgets and live tiles
5. Xbox console support (optional)

### 10.3 Maintenance Checklist

**Regular Tasks:**
- [ ] Monitor crash reports via Application Insights
- [ ] Review Microsoft Store ratings and respond to feedback
- [ ] Update for new Windows releases (annually)
- [ ] Test on new device form factors (Surface, tablets)
- [ ] Update privacy policy if data collection changes
- [ ] Refresh marketing screenshots for new devices
- [ ] Monitor performance metrics
- [ ] Update NuGet package dependencies
- [ ] Run security audits quarterly
- [ ] Test OneDrive API changes

---

**End of Windows Implementation Specification**

This specification provides comprehensive guidance for implementing the Pipe Band Score application on Windows with full Microsoft Store compliance. All implementations must adhere to the domain architecture defined in `pipe_band_app_architecture.md` while leveraging Windows-native capabilities for optimal user experience.