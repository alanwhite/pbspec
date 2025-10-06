# Linux Implementation Specification

**Version:** 1.0  
**Last Updated:** October 06, 2025  
**Platforms:** Linux (Kernel 5.10+), X11 and Wayland  
**Primary Distribution:** Package Managers (Flatpak, Snap, AppImage, Native Packages)

---

## Document Purpose and Scope

This document specifies the **Linux platform-specific implementation** of the Scottish pipe band application, building upon the platform-agnostic domain architecture defined in `pipe_band_app_architecture.md`.

**Reference Architecture:**
- **Domain Specification:** `pipe_band_app_architecture.md`
- **Domain Entities:** Section 3.1 (Instruments, Score, Tune, Part, Measure, Note, Embellishments)
- **Use Cases:** Section 3.4 (Score Management, Musical Editing, Pagination)
- **Repository Interfaces:** Section 3.5 (Data access contracts)
- **File Format:** Section 4 (.pbscore JSON specification)
- **UX Patterns:** `ux_interaction_design.md` (interaction models and workflows)

**What This Document Contains:**
- Dual UI framework strategy (GTK4/libadwaita and Qt6/KDE Frameworks)
- C++ 20+ and Rust architecture options with async patterns
- Multiple desktop environment integration (GNOME, KDE, XFCE, others)
- Package manager distribution (Flatpak, Snap, AppImage, distro packages)
- XDG standards compliance and freedesktop.org specifications
- GIO/KIO cloud storage integration
- PipeWire/ALSA/JACK audio processing
- Cairo/Pango and Qt Graphics SMuFL rendering
- Linux-specific performance optimizations

**What This Document Does NOT Contain:**
- Domain logic definitions (see core architecture document)
- Business rules specifications (see core architecture document)
- Cross-platform architectural patterns (see core architecture document)
- Musical notation concepts (see core architecture document)

---

## 1. Technology Stack and Architecture

### 1.1 Dual UI Framework Strategy

**Framework Selection Rationale:**

Linux desktop diversity necessitates supporting both major desktop environment ecosystems to provide native experiences for the majority of users.

**GTK4/libadwaita Stack (GNOME/Elementary/Budgie users):**
- **GTK 4.12+** as primary UI toolkit
- **libadwaita 1.4+** for GNOME HIG compliance
- **C++ 20+** with gtkmm 4.x bindings as primary language
- **Rust** with gtk4-rs as alternative for safety-critical components
- **GLib/GIO** for async operations and file I/O
- **Cairo** for 2D vector graphics rendering
- **Pango** for complex text layout (SMuFL)
- **GSettings** for application settings storage

**Qt6/KDE Stack (KDE Plasma/LXQt users):**
- **Qt 6.5+** as primary UI framework
- **KDE Frameworks 6.0+** for Plasma integration
- **C++ 20+** with standard Qt/KDE patterns
- **Rust** with qt_core/qt_widgets crates as alternative
- **Qt Quick** for modern declarative UI (optional)
- **QPainter** for 2D vector graphics
- **QTextLayout** for complex text layout
- **KConfig** for application settings

**Decision Matrix for Users:**
- GNOME desktop → GTK4/libadwaita version
- KDE Plasma desktop → Qt6/KDE version
- Other desktops → User choice based on theme preferences
- Flatpak/Snap → Includes both versions as separate apps
- Native packages → Distro maintainers choose appropriate version

### 1.2 Core Technologies

**Language and Concurrency:**
- **C++ 20+** with concepts, ranges, coroutines
- **Rust 1.70+** as alternative for memory safety (optional)
- **Async/await patterns** through GLib MainLoop or Qt Event Loop
- **Thread pools** for parallel computation
- **std::async** and std::future for C++ concurrency
- **tokio** or **async-std** for Rust async runtime

**Architecture Pattern:**
- **MVVM** (Model-View-ViewModel) with signal/slot connections
- **Repository Pattern** with async implementations
- **Clean Architecture** layers maintained through module boundaries
- **Service Layer** for cross-cutting concerns
- **Observer Pattern** for reactive updates

**Audio Processing:**
- **PipeWire** (primary, modern audio stack)
- **ALSA** (fallback for older systems)
- **JACK** (professional audio workflow support)
- **PulseAudio** (legacy support via PipeWire compatibility)
- **GStreamer** for complex audio pipelines (optional)

**Local Storage:**
- **SQLite 3.35+** with custom C++/Rust wrappers
- **GIO File API** (GTK version) for file system access
- **QFile/QIODevice** (Qt version) for file system access
- **Async I/O** with proper thread coordination
- **Background task processing** with proper lifecycle management

**Music Notation:**
- **Cairo** (GTK) for hardware-accelerated vector rendering
- **Pango** (GTK) for SMuFL text layout
- **QPainter** (Qt) for hardware-accelerated vector rendering
- **QTextLayout** (Qt) for SMuFL text layout
- **OpenGL** optional acceleration for complex scores

**Cloud Integration:**
- **GIO/GVfs** (GTK) for cloud provider integration
- **KIO** (Qt) for KDE cloud integration
- **XDG Desktop Portal** for sandboxed file access
- **OAuth 2.0** for cloud provider authentication
- **D-Bus** for inter-process communication

### 1.3 Distribution Strategy

**Primary Distribution Methods:**

**Flatpak (Recommended Primary):**
- Sandboxed application with security benefits
- Universal across all Linux distributions
- Automatic updates via Flathub
- Proper XDG Portal integration required
- Both GTK and Qt versions available as separate apps
- Runtime isolation with declared permissions

**Snap:**
- Canonical's universal package format
- Good Ubuntu integration
- Automatic updates via Snap Store
- Confinement levels: strict (recommended) or classic
- Both GTK and Qt versions as separate snaps

**AppImage:**
- Self-contained executable
- No installation required
- User downloads and runs directly
- No automatic updates (manual download)
- Single AppImage per UI framework
- Good for users wanting portable applications

**Native Distribution Packages:**
- Debian/Ubuntu: .deb packages
- Fedora/RHEL: .rpm packages
- Arch: PKGBUILD in AUR
- openSUSE: .rpm with openSUSE Build Service
- Distro maintainers choose GTK or Qt version
- Integration with distribution package manager

### 1.4 XDG Standards Compliance

**Required XDG Specifications:**

**XDG Base Directory Specification:**
- Configuration: $XDG_CONFIG_HOME/pipebandapp
- Data: $XDG_DATA_HOME/pipebandapp
- Cache: $XDG_CACHE_HOME/pipebandapp
- State: $XDG_STATE_HOME/pipebandapp
- Runtime: $XDG_RUNTIME_DIR/pipebandapp

**XDG Desktop Entry:**
- Application launcher: pipebandapp.desktop
- MIME type associations for .pbscore files
- Icon naming following Icon Theme Specification
- Categories: Audio;Sequencer;Midi;Music;

**XDG MIME Type:**
- MIME type: application/x-pipebandapp-score
- File extension: .pbscore
- Magic bytes for file type detection
- Shared MIME database integration

**XDG Desktop Portal:**
- File chooser portal (org.freedesktop.portal.FileChooser)
- Print portal (org.freedesktop.portal.Print)
- Screenshot portal (org.freedesktop.portal.Screenshot)
- Settings portal (org.freedesktop.portal.Settings)
- Access portal for sandboxed environments

---

## 2. Storage Architecture Implementation

### 2.1 Two-Tier Storage Strategy

**Storage Mode Enumeration:**
- CloudProvider: Integrated cloud storage (GIO/KIO)
- LocalFileSystem: Direct file system access with cloud sync

**Strategic Recommendation for Linux:**

**CloudProvider Mode (Recommended)** leveraging platform integration:
- GIO/GVfs integration for GTK version (Google Drive, OneDrive, Nextcloud, ownCloud via GNOME Online Accounts)
- KIO integration for Qt version (various protocols including cloud providers)
- Transparent cloud sync handled by desktop environment
- User's existing cloud provider setup leveraged
- No additional authentication required if already configured
- XDG Desktop Portal for sandboxed applications

**LocalFileSystem Mode (Power Users):**
- Direct file system access with user-chosen location
- User manages cloud sync with their preferred tools (rclone, Syncthing, etc.)
- Full control over file placement and organization
- Suitable for network shares and NAS systems
- No desktop environment dependencies

**Cloud Provider Integration Methods:**

**GNOME (GTK version):**
- GNOME Online Accounts (GOA) provides authentication
- GIO automatically mounts cloud providers as local paths
- Transparent file access via /run/user/UID/gvfs/
- No API keys or OAuth flows needed in app
- Supports: Google Drive, Microsoft OneDrive, ownCloud, Nextcloud, WebDAV

**KDE (Qt version):**
- KIO slaves handle protocol access
- Dolphin integration for file management
- Transparent access via kio:// URLs
- Supports: Google Drive, Dropbox, OneDrive, SMB, WebDAV, many others

**Sandboxed (Flatpak/Snap):**
- XDG Desktop Portal for file access
- User grants permission via portal dialog
- Portal provides transparent cloud provider access
- Security sandboxing maintained

### 2.2 File-Based Application Architecture

**Linux File System Integration:**

**File Handling Requirements:**
- Implement file watchers for external modifications
- Handle concurrent access with advisory file locking (flock)
- Support for .pbscore file associations via MIME types
- Recent files tracked via XDG recent file specification
- Proper handling of symlinks and hard links
- Network file system awareness (NFS, SMB, etc.)

**Key Patterns:**
- GFileMonitor (GTK) or QFileSystemWatcher (Qt) for change detection
- Atomic writes using temporary files and rename
- Advisory locking to prevent concurrent modification
- GFile API (GTK) or QFile API (Qt) for I/O operations
- Async I/O to prevent UI blocking

### 2.3 Cloud Provider Mode Implementation

**GIO/GVfs Integration (GTK version):**

**Architecture:**
- GFile API provides uniform interface to all storage
- GMount and GVolume for removable media and cloud
- GFileMonitor for watching file changes
- GAsyncResult for asynchronous operations
- No direct API calls to cloud providers needed

**Key Components:**
- GFile operations work with cloud URLs transparently
- GNOME Online Accounts handles authentication
- GVfs daemons handle protocol implementation
- Application sees mounted cloud storage as local paths
- Proper error handling for offline scenarios

**KIO Integration (Qt version):**

**Architecture:**
- KIO provides protocol-agnostic file operations
- KIOJob for asynchronous operations with progress
- StatJob, CopyJob, DeleteJob for file operations
- Supports kio:// URLs for various protocols
- Dolphin integration for file management

**Key Components:**
- KIO::stat for file metadata
- KIO::get and KIO::put for reading/writing
- KFileWidget for file dialogs with KIO support
- Built-in caching and offline mode
- Progress tracking for slow operations

### 2.4 XDG Desktop Portal Integration

**Portal Usage (Flatpak/Snap):**

**File Chooser Portal:**
- org.freedesktop.portal.FileChooser.OpenFile
- org.freedesktop.portal.FileChooser.SaveFile
- Proper filters for .pbscore files
- Multiple file selection support
- Directory selection for folder operations

**Document Portal:**
- org.freedesktop.portal.Documents
- Maintains access to user-selected files
- Persists across application restarts
- Security sandboxing enforced
- File handles managed by portal

**Settings Portal:**
- org.freedesktop.portal.Settings
- Query desktop theme (light/dark)
- Font rendering preferences
- Accessibility settings

**Implementation Requirements:**
- D-Bus communication for portal access
- Async D-Bus methods with proper error handling
- Graceful degradation if portals unavailable
- Direct file access fallback for unsandboxed environments

### 2.5 First Launch Experience

**Desktop Environment Detection:**

**Onboarding Flow:**
1. Detect desktop environment (check $XDG_CURRENT_DESKTOP)
2. Check for GNOME Online Accounts or KDE Accounts integration
3. If cloud accounts configured: Offer CloudProvider mode
4. If no cloud accounts: Offer LocalFileSystem mode with setup guide
5. Display help for configuring cloud accounts if desired
6. Store user preference for future launches

**Cloud Account Setup Guidance:**
- GNOME: Guide to Settings → Online Accounts
- KDE: Guide to System Settings → Online Accounts
- Generic: Point to desktop environment documentation
- Explain benefits of cloud integration
- Offer to skip and use local files

**Permission Request Pattern (Sandboxed):**
- Request microphone only when user taps "Record Practice Session"
- Request file system access via XDG Desktop Portal
- Explain why permission needed before requesting
- Handle denial gracefully with alternative workflows
- Never repeatedly prompt for denied permissions

---

## 3. C++ Architecture with Modern Patterns

### 3.1 Dependency Injection and Service Configuration

**Application Startup (GTK version):**

**Initialization Sequence:**
- Create Gtk::Application instance
- Register GResource bundle (icons, UI files)
- Setup GSettings schema
- Initialize dependency injection container
- Register services (repositories, use cases, domain services)
- Connect signal handlers for application lifecycle
- Create and show main window

**Service Lifetimes:**
- Singleton: Domain services, repositories, configuration
- Transient: ViewModels, Use Cases (new instance per operation)
- Scoped: Database connections, transaction contexts

**Application Startup (Qt version):**

**Initialization Sequence:**
- Create QApplication instance
- Load Qt resource system (icons, UI files)
- Initialize QSettings with organization/application name
- Setup dependency injection container
- Register services
- Connect signals for application lifecycle
- Create and show main window

### 3.2 Async Domain Layer

**Domain Coordinator Pattern (GTK):**

**GLib Async Operations:**
- Use GTask for asynchronous operations
- Implement GAsyncReadyCallback for completion
- Proper cancellation with GCancellable
- Thread pool via GThreadPool for CPU-intensive work
- Main loop integration via g_idle_add for UI updates

**Key Patterns:**
- Async methods return void and take GAsyncReadyCallback
- Finish methods retrieve results or throw errors
- Cancellation tokens passed to all async operations
- Thread safety via GMutex for shared state
- GMainContext for thread-specific event loops

**Domain Coordinator Pattern (Qt):**

**Qt Async Operations:**
- Use QtConcurrent for parallel operations
- QFuture and QFutureWatcher for async results
- QThread for long-running background operations
- QTimer for delayed operations
- Signals/slots for cross-thread communication

**Key Patterns:**
- Worker threads for heavy computation
- Signals emitted from worker threads
- Slots executed on main thread via Qt::QueuedConnection
- QMutex/QReadWriteLock for thread synchronization
- moveToThread for object thread affinity

### 3.3 Concurrent Layout Calculation

**Layout Calculation Engine:**

**Parallel Processing (GTK):**
- GThreadPool with configurable worker count
- GTask for individual layout calculations
- Concurrent access control via GMutex
- Progress reporting via GAsyncResult progress callback
- Cancellation support throughout

**Parallel Processing (Qt):**
- QtConcurrent::map for parallel page calculation
- QFutureWatcher for progress and completion signals
- QThreadPool for worker thread management
- Atomic operations for shared counters
- Cancellation via QFutureSynchronizer

**Performance Goals:**
- Parallel efficiency: 70%+ on 4+ cores
- Cache hit rate: 80%+ for typical workflows
- Incremental update: < 100ms for single measure
- Full document: < 2s for 100 pages

---

## 4. UI Implementation

### 4.1 GTK4/libadwaita Implementation

**Main Application Window Structure:**

**Window Hierarchy:**
- AdwApplicationWindow as main window
- AdwHeaderBar with primary actions
- AdwNavigationSplitView for document browser + editor
- GtkScrolledWindow for score canvas
- AdwToastOverlay for notifications
- GtkPopoverMenu for context menus

**Score Editor Components:**
- Custom GtkWidget subclass for score canvas
- GtkDrawingArea with Cairo rendering
- GtkEventController for input handling
- GtkGesture for multi-touch gestures
- GtkShortcutController for keyboard shortcuts

**GNOME HIG Compliance:**
- AdwStyleManager for light/dark theme
- AdwPreferencesWindow for settings
- AdwAboutWindow for about dialog
- Adaptive layouts for different window sizes
- Proper spacing and margins (12px standard)

**Score Canvas Implementation (GTK):**

**Custom Widget Architecture:**
- Subclass GtkWidget
- Override snapshot vfunc for rendering
- Implement gesture controllers for input
- Cairo drawing operations for score rendering
- Pango for SMuFL text layout
- GdkPaintable for cached rendering (optional)

**Rendering Pipeline:**
1. Clear Cairo surface to white background
2. Apply zoom and pan transformations
3. Render staff lines using cairo_line_to
4. Render clefs, keys, time signatures with Pango
5. Render notes and embellishments
6. Render barlines and measure separators
7. Apply selection highlights
8. Handle damage regions for partial updates

### 4.2 Qt6/KDE Implementation

**Main Application Window Structure:**

**Window Hierarchy:**
- QMainWindow as main window
- KXmlGuiWindow for action integration (KDE version)
- QToolBar for primary actions
- QSplitter for document browser + editor
- QScrollArea for score canvas
- QStatusBar for status information
- QMenu for context menus

**Score Editor Components:**
- Custom QWidget subclass for score canvas
- QPainter for rendering in paintEvent
- QGraphicsView/QGraphicsScene as alternative architecture
- Mouse and touch event handlers
- QShortcut for keyboard bindings

**KDE Integration (Qt version):**
- KXmlGuiWindow for menu/toolbar integration
- KConfig for settings storage
- KIO for file operations
- KNotification for notifications
- Breeze icon theme integration
- KAboutData for about dialog

**Score Canvas Implementation (Qt):**

**Custom Widget Architecture:**
- Subclass QWidget
- Override paintEvent for rendering
- QPainter drawing operations
- QTextLayout for SMuFL rendering
- Event handlers for mouse/touch input
- QPixmapCache for rendering optimization

**Rendering Pipeline:**
1. Create QPainter on widget
2. Fill background with white
3. Apply QTransform for zoom/pan
4. Render staff lines with QPainter::drawLine
5. Render musical symbols with QTextLayout
6. Render notes and embellishments
7. Render barlines and separators
8. Draw selection rectangles
9. Handle paint regions for efficiency

### 4.3 SMuFL Rendering

**Font Loading and Caching (GTK):**

**Pango Font Integration:**
- Load Bravura.otf via fontconfig
- Create PangoFontDescription for SMuFL font
- PangoLayout for text layout
- PangoAttrList for font attributes
- Cache PangoLayout instances for reuse

**Rendering Musical Symbols:**
- UTF-8 encoded Unicode codepoints for symbols
- pango_layout_set_text with SMuFL codepoints
- pango_cairo_show_layout for rendering
- Proper font metrics via pango_layout_get_extents
- Baseline alignment corrections

**Font Loading and Caching (Qt):**

**Qt Font Integration:**
- QFontDatabase::addApplicationFont for Bravura.otf
- QFont with family name and size
- QFontMetrics for glyph measurements
- QTextLayout for complex text rendering
- QFontCache for automatic caching

**Rendering Musical Symbols:**
- QString with Unicode codepoints
- QPainter::drawText for simple glyphs
- QTextLayout for complex layouts
- Proper glyph positioning via QFontMetrics
- Baseline adjustments for alignment

### 4.4 Keyboard Shortcuts

**GTK Keyboard Handling:**

**GtkShortcutController:**
- Create GtkShortcutController per widget
- Define GtkShortcut with trigger and action
- Use accelerators format (e.g., "<Primary>s" for Ctrl+S)
- Signal activation callbacks for shortcuts
- Proper event propagation and handling

**Standard Shortcuts:**
- Ctrl+N: New score
- Ctrl+O: Open score
- Ctrl+S: Save score
- Ctrl+Z: Undo
- Ctrl+Shift+Z: Redo
- Ctrl+X/C/V: Cut/Copy/Paste
- Delete: Delete selection

**Qt Keyboard Handling:**

**QShortcut and QAction:**
- Create QAction instances for all actions
- Set keyboard shortcuts via setShortcut
- Connect triggered signal to slots
- QKeySequence for platform-independent shortcuts
- Proper action enabled/disabled state

**Standard Shortcuts:**
- Ctrl+N: New score
- Ctrl+O: Open score
- Ctrl+S: Save score
- Ctrl+Z: Undo
- Ctrl+Y: Redo
- Ctrl+X/C/V: Cut/Copy/Paste
- Delete: Delete selection

### 4.5 Theme Integration

**GTK Theme Support:**

**libadwaita Theming:**
- AdwStyleManager::get_default() for theme access
- Automatic light/dark mode switching
- System accent color integration
- Proper CSS styling with GTK CSS
- Custom CSS for score canvas if needed

**Qt Theme Support:**

**KDE/Qt Theming:**
- QApplication::palette() for system colors
- KColorScheme for KDE color access
- Automatic Breeze theme integration on KDE
- Qt style sheets for custom styling
- Follow system theme changes via signals

---

## 5. Audio Processing

### 5.1 Audio Stack Selection

**PipeWire (Primary, Modern):**

**Architecture:**
- PipeWire as unified audio/video server
- GStreamer PipeWire elements
- Low-latency audio graph processing
- Automatic device discovery
- Session management via WirePlumber

**Use Cases:**
- Metronome playback with precise timing
- Audio recording with low latency
- Simultaneous playback and recording
- Automatic device fallback on disconnection

**ALSA (Fallback, Legacy):**

**Architecture:**
- Direct ALSA API for simple use cases
- snd_pcm_open for device access
- Period-based buffer management
- Manual device enumeration
- Lower-level control for compatibility

**JACK (Professional Audio):**

**Architecture:**
- JACK client registration
- Port connections for audio routing
- Sample-accurate synchronization
- Integration with professional audio workflow
- Support for external audio interfaces

**Implementation Strategy:**
- Detect available audio systems at runtime
- PipeWire preferred if available
- Fall back to ALSA if PipeWire unavailable
- JACK support optional for pro users
- Graceful degradation of features

### 5.2 Audio Engine Implementation

**Metronome Service:**

**Requirements:**
- Generate click sounds programmatically
- Support variable tempo (40-240 BPM)
- Accent patterns based on time signature
- Start/stop with minimal latency
- Background operation while editing
- Synchronized visual feedback

**Audio Recording:**

**Requirements:**
- Record from microphone or line input
- Save to Ogg Vorbis or FLAC format
- Low latency monitoring
- Proper buffer management
- Device selection UI
- Progress indication for long recordings

**Pitch Analysis:**

**Requirements:**
- FFT-based pitch detection
- Real-time feedback during recording
- Note accuracy visualization
- Tuning reference comparison
- Practice session analytics

### 5.3 Audio Permissions

**PipeWire Permissions:**
- Access granted via Flatpak/Snap permissions
- XDG Desktop Portal for sandboxed access
- org.freedesktop.portal.Device portal
- User confirmation for microphone access

**Native Package Permissions:**
- Standard ALSA/PulseAudio access
- Group membership for audio access (audio group)
- No special permissions required
- Standard user access patterns

---

## 6. Distribution and Packaging

### 6.1 Flatpak Configuration

**Flatpak Manifest (org.pipebandapp.PipeBandApp):**

**Required Sections:**
- app-id: org.pipebandapp.PipeBandApp
- runtime: org.gnome.Platform (GTK) or org.kde.Platform (Qt)
- runtime-version: Latest stable
- sdk: org.gnome.Sdk or org.kde.Sdk
- command: pipebandapp executable name
- finish-args: Permissions and access controls

**Permissions (finish-args):**
- Wayland/X11 access: --socket=wayland, --socket=fallback-x11
- PulseAudio/PipeWire: --socket=pulseaudio
- File system access: --filesystem=xdg-documents, --filesystem=xdg-download
- D-Bus: --socket=session-bus for portal access
- Device access: --device=all for microphone
- Network: --share=network for cloud sync

**Build Process:**
- Flatpak-builder for manifest-based build
- Multi-stage build for dependencies
- Runtime caching for rebuild speed
- Automatic updates via Flathub OSTree

### 6.2 Snap Configuration

**Snapcraft YAML (pipebandapp):**

**Required Sections:**
- name: pipebandapp
- version: Semantic versioning
- summary: One-line description
- description: Full description
- grade: stable or devel
- confinement: strict recommended
- base: core22 or core24
- architectures: amd64, arm64

**Plugs (Permissions):**
- desktop: Desktop environment integration
- desktop-legacy: Legacy desktop support
- wayland: Wayland compositor access
- x11: X11 server access
- audio-playback: PulseAudio/PipeWire playback
- audio-record: Microphone access
- home: User home directory access
- network: Internet access for cloud sync

**Parts:**
- pipebandapp: Main application build
- Dependencies: All required libraries
- Assets: Icons, desktop files, etc.

### 6.3 AppImage Configuration

**AppImage Structure:**

**Components:**
- AppRun script for entry point
- .DirIcon for application icon
- .desktop file for integration
- All required shared libraries bundled
- Bravura.otf font bundled

**Build Process:**
- Use linuxdeploy or appimagetool
- Bundle all dependencies not in base system
- Proper RPATH configuration for libraries
- Test on multiple distributions
- Sign AppImage with GPG key

**Distribution:**
- GitHub Releases for downloads
- AppImageHub listing
- Direct download from website
- Update mechanism via AppImageUpdate

### 6.4 Native Distribution Packages

**Debian/Ubuntu (.deb):**

**Package Structure:**
- debian/control: Package metadata and dependencies
- debian/rules: Build instructions
- debian/install: File installation mapping
- debian/copyright: License information
- debian/changelog: Version history

**Fedora/RHEL (.rpm):**

**Package Structure:**
- SPEC file with metadata
- BuildRequires and Requires dependencies
- %build, %install, %files sections
- Desktop file and icon installation
- AppStream metadata

**Arch Linux (PKGBUILD):**

**Package Structure:**
- PKGBUILD script
- depends and makedepends arrays
- build() and package() functions
- Desktop file and icon installation
- AUR submission for community packages

---

## 7. Desktop Environment Integration

### 7.1 D-Bus Integration

**Session Bus Services:**

**Application Activation:**
- org.freedesktop.Application interface
- Activate method for single-instance enforcement
- Open method for opening files from external sources
- ActivateAction for custom actions

**MPRIS2 (Media Player Interface):**
- org.mpris.MediaPlayer2 interface
- PlayPause, Stop methods for metronome control
- Metadata property for current score information
- Integration with media keys and system controls

**Custom D-Bus Service:**
- org.pipebandapp.PipeBandApp service
- OpenScore method for external app integration
- ExportScore method for automation
- Collaboration signals for multi-user editing

### 7.2 Desktop File Integration

**.desktop File (pipebandapp.desktop):**

**Required Keys:**
- Type=Application
- Name: Application name (localized)
- Comment: Short description (localized)
- Icon: Icon name following Icon Theme Spec
- Exec: Command to execute
- Terminal: false
- Categories: Audio;Sequencer;Midi;Music;
- MimeType: application/x-pipebandapp-score;
- StartupWMClass: For window grouping

**Actions:**
- NewScore: Create new score action
- OpenRecent: Open recent files submenu

### 7.3 MIME Type Registration

**MIME Type (application-x-pipebandapp-score.xml):**

**Required Elements:**
- mime-type element with type attribute
- glob pattern for .pbscore files
- icon element for file icon
- sub-class-of for inheritance (application/json)
- comment element (localized)
- magic element for file detection (optional)

**Installation:**
- Install to /usr/share/mime/packages/
- Run update-mime-database after install
- Update desktop database for associations

### 7.4 Icon Theme Integration

**Icon Requirements:**

**Sizes and Formats:**
- 16x16, 22x22, 24x24, 32x32, 48x48, 64x64, 128x128, 256x256
- SVG for scalable icon (symbolic and full-color)
- PNG for raster sizes
- Follow Icon Naming Specification

**Installation Locations:**
- /usr/share/icons/hicolor/ for raster icons
- /usr/share/icons/hicolor/scalable/ for SVG
- Icon cache update after installation

**Icon Naming:**
- Application icon: org.pipebandapp.PipeBandApp
- MIME type icon: application-x-pipebandapp-score
- Symbolic icon: org.pipebandapp.PipeBandApp-symbolic

---

## 8. Accessibility

### 8.1 GTK Accessibility (ATK/AT-SPI)

**Accessible Widget Implementation:**

**Requirements:**
- All widgets must have accessible names
- Implement AtkObject interface for custom widgets
- Proper role assignment (ATK_ROLE_*)
- State management (ATK_STATE_*)
- Action interface for interactive elements
- Text interface for text content

**Score Canvas Accessibility:**
- Implement AtkText for note content
- AtkSelection for note selection
- AtkComponent for geometric information
- Keyboard navigation for all elements
- Screen reader announcements for actions

### 8.2 Qt Accessibility (QAccessible)

**Accessible Widget Implementation:**

**Requirements:**
- QAccessibleInterface implementation
- Proper role() method return values
- State flag management (QAccessible::State)
- Accessible text interface
- Accessible action interface
- Child widget accessibility tree

**Score Canvas Accessibility:**
- QAccessibleTextInterface for notes
- QAccessibleValueInterface for properties
- QAccessibleActionInterface for editing actions
- Keyboard focus indicators
- Screen reader integration via AT-SPI bridge

### 8.3 Keyboard Navigation

**Navigation Requirements:**

**Essential Patterns:**
- Tab/Shift+Tab for focus traversal
- Arrow keys for selection movement
- Enter/Space for activation
- Escape for cancellation
- Context menu key (Menu key) for context menu
- F10 for menu bar activation

**Score Navigation:**
- Tab moves between score elements
- Arrow keys move within measures and staves
- Home/End jump to start/end of measure
- Page Up/Down move between systems
- Ctrl+Home/End jump to document start/end
- Shift+Arrow extends selection

---

## 9. Performance Optimization

### 9.1 Rendering Performance

**Cairo Optimization (GTK):**

**Damage Region Tracking:**
- Track changed regions with cairo_region_t
- Only redraw damaged areas
- Use cairo_push_group for complex compositing
- Cache rendered elements in cairo_surface_t
- Release cached surfaces on memory pressure

**Rendering Strategies:**
- Double buffering via GdkPaintable
- Partial updates for small changes
- Background thread rendering for complex scores
- Progressive rendering for large documents
- GPU acceleration via OpenGL (optional)

**QPainter Optimization (Qt):**

**Efficient Painting:**
- Use QPainterPath for complex shapes
- Cache rendered content in QPixmap
- QPixmapCache for automatic caching
- setClipRegion to limit painting area
- Antialiasing only where needed
- Native painting with setRenderHint

**Rendering Strategies:**
- Double buffering automatic in Qt
- Update() instead of repaint() for batching
- Partial updates via update(QRect)
- Background thread rendering with QImage
- OpenGL widget for hardware acceleration

### 9.2 Memory Management

**Large Document Handling:**

**GTK Memory Strategies:**
- GSlice allocator for small objects
- g_object_unref for proper cleanup
- Weak references via g_object_add_weak_pointer
- Memory pools for frequently allocated types
- Monitor memory via GMemoryMonitor

**Qt Memory Strategies:**
- Parent-child ownership for automatic cleanup
- QSharedPointer for reference counting
- QWeakPointer for non-owning references
- Object pools for reusable instances
- QScopedPointer for RAII

**Page Virtualization:**
- Load only visible pages plus buffer (2 before, 2 after)
- Unload pages outside viewport
- LRU cache for recently viewed pages
- Lazy loading of page content
- Memory pressure response

### 9.3 Thread Management

**GTK Threading (GLib):**

**Thread Safety:**
- GMainContext per thread for event loops
- g_idle_add for main thread callbacks
- GMutex for critical sections
- GCond for thread synchronization
- GThreadPool for worker threads

**Background Operations:**
- GTask for async operations
- GCancellable for cancellation
- GAsyncQueue for thread communication
- Proper context switching to main thread for UI updates

**Qt Threading:**

**Thread Safety:**
- QObject thread affinity
- Qt::QueuedConnection for cross-thread signals
- QMutex/QReadWriteLock for synchronization
- QWaitCondition for signaling
- QThreadPool for worker management

**Background Operations:**
- QtConcurrent::run for simple tasks
- QThread subclass for complex workers
- moveToThread for object relocation
- Signals/slots for thread-safe communication

### 9.4 Startup Performance

**Fast Startup Requirements:**

**GTK Optimization:**
- Lazy load non-essential resources
- Defer heavy initialization
- Use GResource for bundled assets
- Minimize initial widget creation
- Profile with sysprof

**Qt Optimization:**
- Lazy plugin loading
- Defer non-critical initialization
- Use Qt resource system efficiently
- Minimize initial object creation
- Profile with Qt Creator profiler

**Targets:**
- Cold start: < 2 seconds to visible window
- Warm start: < 1 second
- Document open: < 1 second for typical scores
- Initial layout: < 500ms

---

## 10. Testing Strategy

### 10.1 Unit Testing

**GTK Unit Tests:**

**Framework:**
- GLib testing framework (g_test_*)
- Mock objects via dependency injection
- Test fixtures with setup/teardown
- Async test support via GMainLoop
- Coverage analysis with gcov/lcov

**Test Organization:**
- Domain layer: 95%+ coverage
- Use cases: 90%+ coverage
- Repositories: 85%+ coverage
- UI logic: 70%+ coverage

**Qt Unit Tests:**

**Framework:**
- Qt Test framework (QTest namespace)
- Mock objects via dependency injection
- Test fixtures with initTestCase/cleanupTestCase
- Signal spy for async testing
- Coverage analysis with gcov/lcov

**Test Categories:**
- Unit tests for domain logic
- Integration tests for repositories
- Widget tests for UI components
- End-to-end tests for workflows

### 10.2 Integration Testing

**Desktop Environment Testing:**

**Test Matrix:**
- GNOME 45+ (GTK version)
- KDE Plasma 6+ (Qt version)
- XFCE 4.18+ (both versions)
- MATE (both versions)
- Cinnamon (both versions)

**Platform Testing:**
- X11 window system
- Wayland compositor
- HiDPI displays (1.5x, 2x, 3x scaling)
- Multiple monitors
- Touch screens

### 10.3 Package Testing

**Distribution Testing:**

**Test Targets:**
- Ubuntu 22.04 LTS and 24.04 LTS
- Fedora latest stable
- Arch Linux rolling
- Debian stable
- openSUSE Leap and Tumbleweed

**Package Formats:**
- Flatpak on all distributions
- Snap on Ubuntu and derivatives
- AppImage universal compatibility
- Native packages on respective distributions

### 10.4 Accessibility Testing

**Testing Requirements:**

**Assistive Technology:**
- Orca screen reader (GNOME)
- Gnome-shell magnifier
- On-screen keyboard (onboard)
- High contrast themes
- Large text settings

**Verification:**
- All interactive elements reachable via keyboard
- Screen reader announces element types and states
- Focus indicators visible
- No information conveyed by color alone
- Text alternatives for graphical elements

---

## 11. Security and Sandboxing

### 11.1 Flatpak Sandboxing

**Security Model:**

**Filesystem Access:**
- --filesystem=xdg-documents:rw for document access
- --filesystem=xdg-download:rw for exports
- No --filesystem=home without justification
- XDG Desktop Portal for additional access
- User grants permission via portal dialogs

**Network Access:**
- --share=network for cloud sync
- TLS/SSL for all network communications
- Certificate validation enforced
- No unencrypted credential storage

**Device Access:**
- --device=all for microphone (only if needed)
- Alternative: --device=dri for graphics acceleration
- Justification required for device access

### 11.2 Snap Confinement

**Confinement Levels:**

**Strict Confinement (Recommended):**
- AppArmor security policies enforced
- Interfaces for controlled access
- Proper plug/slot connections
- Snap Store review process

**Required Interfaces:**
- desktop, desktop-legacy: Desktop integration
- audio-playback, audio-record: Audio access
- home: User home directory
- network: Internet connectivity
- removable-media: External drives (optional)

### 11.3 Credential Storage

**Secret Storage Integration:**

**libsecret (GTK/GNOME):**
- Store OAuth tokens securely
- GNOME Keyring integration
- Secret Service API (D-Bus)
- Per-user encrypted storage
- Automatic unlock with login

**KWallet (Qt/KDE):**
- Store credentials securely
- KWallet integration via KIO
- D-Bus API access
- Encrypted storage
- Session or permanent storage options

**Fallback:**
- Encrypted file storage
- Strong encryption (AES-256)
- Secure key derivation (PBKDF2)
- Warning about less secure storage

---

## 12. Localization and Internationalization

### 12.1 GTK Localization

**gettext Integration:**

**Implementation:**
- Use gettext for all user-facing strings
- Mark strings with _() or gettext()
- Context with C_() for disambiguation
- Plural forms with ngettext()
- Extract strings with xgettext
- Compile with msgfmt

**Translation Files:**
- POT template: pipebandapp.pot
- PO files per language: de.po, fr.po, es.po
- MO compiled files: de.mo, fr.mo, es.mo
- Install to /usr/share/locale/

**Locale Support:**
- setlocale(LC_ALL, "") at startup
- bindtextdomain for message catalog
- bind_textdomain_codeset for UTF-8
- textdomain for default domain

### 12.2 Qt Localization

**Qt Linguist Integration:**

**Implementation:**
- Use tr() for all user-facing strings
- Context via class names automatically
- Plural forms with tr() plural argument
- Extract with lupdate
- Translate with Qt Linguist
- Compile with lrelease

**Translation Files:**
- TS files: pipebandapp_de.ts, pipebandapp_fr.ts
- QM compiled: pipebandapp_de.qm, pipebandapp_fr.qm
- Load with QTranslator
- Install QTranslator on QApplication

**Locale Support:**
- QLocale for number/date formatting
- QTranslator for UI strings
- Automatic right-to-left layout (Arabic, Hebrew)
- Font selection per language

### 12.3 Cultural Adaptations

**Regional Formats:**

**Date and Time:**
- Use system locale settings
- GDateTime (GTK) or QDateTime (Qt)
- Localized month and day names
- Proper calendar systems

**Number Formatting:**
- Decimal separators per locale
- Thousands separators per locale
- Currency formatting (future)

**Paper Sizes:**
- Default A4 for most regions
- US Letter for US locale
- JIS B sizes for Japan
- Proper DPI handling

---

## 13. Distribution-Specific Considerations

### 13.1 Debian/Ubuntu Packaging

**Package Relationships:**

**Build Dependencies:**
- debhelper-compat
- cmake or meson
- GTK: libgtk-4-dev, libadwaita-1-dev
- Qt: qtbase6-dev, libkf6coreaddons-dev
- Audio: libpipewire-0.3-dev, libasound2-dev
- Database: libsqlite3-dev
- JSON: nlohmann-json3-dev or rapidjson-dev

**Runtime Dependencies:**
- GTK: libgtk-4-1, libadwaita-1-0
- Qt: qt6-base, kf6-kcoreaddons
- Audio: pipewire or pulseaudio-utils
- Fonts: fonts-bravura (if packaged) or bundled

**Lintian Compliance:**
- No errors allowed
- Warnings addressed or overridden with justification
- Proper copyright file
- Reproducible builds

### 13.2 Fedora/RHEL Packaging

**RPM Spec File:**

**Build Requirements:**
- RPM_BUILD_ROOT handling
- BuildRequires: Proper package names
- GTK: gtk4-devel, libadwaita-devel
- Qt: qt6-qtbase-devel, kf6-kcoreaddons-devel
- Proper macros usage (%{_bindir}, etc.)

**Runtime Requirements:**
- Requires: Runtime dependencies
- Automatic dependency detection
- Proper file ownership
- SELinux context preservation

**rpmlint Compliance:**
- Clean rpmlint output
- Proper Group tags (deprecated but conventional)
- License tag accuracy
- Buildroot cleaning

### 13.3 Arch Linux Packaging

**PKGBUILD Standards:**

**Package Function:**
- pkgname, pkgver, pkgrel properly set
- arch=('x86_64' 'aarch64')
- depends array with runtime deps
- makedepends array with build deps
- Source array with download URLs
- sha256sums for verification

**Build Function:**
- Use proper build directory ($srcdir)
- Install to $pkgdir
- Proper cmake/meson invocation
- Strip debug symbols (handled automatically)

**AUR Guidelines:**
- PKGBUILD comments for clarity
- Proper .SRCINFO generation
- Maintainer contact information
- Orphan handling policy

### 13.4 openSUSE Packaging

**OBS (Open Build Service):**

**Project Structure:**
- _service file for source fetching
- .spec file for package definition
- .changes file for changelog
- Multiple architecture support

**Build Requirements:**
- Proper BuildRequires syntax
- openSUSE-specific package names
- %suse_update_desktop_file macro
- Icon cache updates in %post/%postun

---

## 14. Development Workflow

### 14.1 Build Systems

**CMake Configuration (GTK version):**

**Project Structure:**
- CMakeLists.txt in root
- find_package for dependencies (PkgConfig)
- GTK4, Adwaita, Cairo, Pango
- Source file organization
- Install targets for binaries, data, icons
- CPack for package generation

**CMake Configuration (Qt version):**

**Project Structure:**
- CMakeLists.txt in root
- find_package(Qt6 COMPONENTS Core Widgets)
- find_package(KF6 COMPONENTS CoreAddons Config)
- Automatic MOC, UIC, RCC
- Install targets
- CPack for packaging

**Meson Configuration (Alternative):**

**Project Structure:**
- meson.build in root
- dependency() for finding packages
- GTK4, Adwaita, Cairo, Pango
- Source compilation targets
- Install configuration
- Introspection data (optional)

### 14.2 IDE Support

**Recommended IDEs:**

**GNOME Builder (GTK development):**
- Native flatpak support
- Integrated debugging
- GTK-specific templates
- Vala/C/C++/Rust support
- GSettings schema editing

**Qt Creator (Qt development):**
- Native Qt project support
- Integrated Qt Designer
- CMake and qmake support
- Excellent debugger integration
- Code completion and refactoring

**Visual Studio Code:**
- C++/Rust extensions
- CMake Tools extension
- GTK/Qt snippets available
- Integrated terminal
- Git integration

**CLion (JetBrains):**
- Excellent CMake support
- Powerful refactoring
- Code analysis
- Cross-platform development
- Professional debugging

### 14.3 Continuous Integration

**GitLab CI/CD:**

**Pipeline Stages:**
- Build stage: Compile for multiple distributions
- Test stage: Run unit and integration tests
- Package stage: Create Flatpak, Snap, AppImage
- Deploy stage: Upload to distribution platforms

**Docker Containers:**
- Ubuntu container for .deb testing
- Fedora container for .rpm testing
- Arch container for PKGBUILD testing
- Flatpak build container

**GitHub Actions:**

**Workflow Configuration:**
- Matrix strategy for multiple distributions
- Artifact upload for packages
- Automated release creation
- Code coverage reporting
- Static analysis integration

### 14.4 Debugging Tools

**GTK Debugging:**

**Tools:**
- GTK Inspector (Ctrl+Shift+D or GTK_DEBUG=interactive)
- GDB for C/C++ debugging
- Valgrind for memory analysis
- Sysprof for performance profiling
- Bustle for D-Bus traffic analysis

**Qt Debugging:**

**Tools:**
- Qt Creator debugger integration
- GammaRay for Qt introspection
- GDB/LLDB for native debugging
- Valgrind for memory issues
- Heaptrack for memory profiling
- QML profiler for Qt Quick (if used)

---

## 15. Documentation and Help

### 15.1 User Documentation

**Help System Integration:**

**GNOME Help (yelp):**
- Write documentation in Mallard XML
- Install to /usr/share/help/
- Localized documentation support
- Context-sensitive help via Yelp
- Topic organization with pages
- Search functionality built-in

**KDE Help (khelpcenter):**
- Write documentation in DocBook XML
- Install to /usr/share/doc/
- KHelpCenter integration
- F1 context help support
- Section organization
- Search and index

**Universal Formats:**
- HTML documentation for web viewing
- PDF user manual for printing
- Markdown for developer docs
- Man pages for CLI tools (if any)

### 15.2 Tooltips and Contextual Help

**GTK Tooltips:**

**Implementation:**
- gtk_widget_set_tooltip_text for simple tooltips
- gtk_widget_set_tooltip_markup for formatted
- GtkTooltip for custom content
- Delay and timeout configuration
- Keyboard activation via Shift+F1

**Qt Tooltips:**

**Implementation:**
- setToolTip for widget tooltips
- Rich text formatting supported
- QWhatsThis for context help
- Shift+F1 for What's This mode
- Status tip for status bar messages

### 15.3 Online Resources

**Documentation Hosting:**
- ReadTheDocs for comprehensive guides
- GitHub Wiki for community docs
- Website documentation section
- Video tutorials on YouTube
- Community forum or Discord

**API Documentation:**
- Doxygen for C++ API docs
- gtk-doc for GTK version (optional)
- Qt Assistant integration for Qt version
- Reference manual generation
- Example code repository

---

## 16. Community and Contribution

### 16.1 Open Source Licensing

**License Selection:**

**Recommended: GPL-3.0-or-later**
- Strong copyleft for protection
- Compatible with GTK and Qt
- Ensures freedom for users
- Allows commercial use
- Patent protection included

**Alternative: LGPL-3.0-or-later**
- Allows proprietary linking
- More permissive for libraries
- Still protects core code

**Assets and Resources:**
- Bravura font: SIL Open Font License
- Icons: Creative Commons or GPL
- Documentation: Creative Commons BY-SA

### 16.2 Contribution Guidelines

**Development Process:**

**Getting Started:**
- Fork repository on GitHub/GitLab
- Clone and setup development environment
- Review architecture documentation
- Check issue tracker for tasks
- Join developer chat/mailing list

**Code Standards:**
- Follow existing code style
- GTK: GNOME coding style
- Qt: Qt coding conventions
- Run linters and formatters
- Write tests for new features
- Update documentation

**Pull Request Process:**
- Create feature branch
- Commit with descriptive messages
- Push to fork
- Open pull request
- Address review feedback
- Squash commits if requested

### 16.3 Package Maintainer Coordination

**Distribution Maintainer Support:**

**Communication Channels:**
- Mailing list for maintainers
- Issue tracker for packaging bugs
- Release announcements
- Security vulnerability disclosure
- Breaking change notifications

**Maintainer Resources:**
- Packaging guides for each distro
- Changelog and release notes
- Build instructions
- Dependency documentation
- Version compatibility matrix

---

## 17. Performance Benchmarks

### 17.1 Target Metrics

**Application Performance:**

**Startup Times:**
- Cold start: < 2 seconds
- Warm start: < 1 second
- Document load (small): < 500ms
- Document load (large 100 pages): < 3 seconds

**Rendering Performance:**
- Page render: < 100ms per page
- Scroll 60fps maintained
- Zoom smooth at all levels
- Export PDF: < 5 seconds for 100 pages

**Memory Usage:**
- Base memory: < 50 MB
- Small document: < 100 MB
- Large document (100 pages): < 300 MB
- Cached pages: Evict to stay under 500 MB total

**Responsiveness:**
- UI interaction: < 16ms (60 fps)
- Note insertion: < 50ms
- Layout recalculation: < 200ms
- Save operation: < 1 second

### 17.2 Optimization Priorities

**Critical Paths:**
1. Initial window display
2. Document rendering
3. User input response
4. Layout calculation
5. File operations

**Profiling Tools:**
- sysprof for GTK version
- perf for system-wide profiling
- Valgrind/Cachegrind for cache analysis
- Heaptrack for memory profiling
- Qt Creator profiler for Qt version

---

## 18. Known Limitations and Workarounds

### 18.1 Platform Limitations

**Wayland Compositor Limitations:**
- Some compositors lack proper tablet support
- Screen capture restrictions (security feature)
- Window positioning limitations (security)
- Workaround: X11 fallback for compatibility

**X11 Limitations:**
- No per-monitor scaling (fractional scaling issues)
- Less secure than Wayland
- Workaround: Use Wayland when available

**Audio Stack Fragmentation:**
- Multiple audio systems (ALSA/Pulse/PipeWire/JACK)
- Different APIs and capabilities
- Workaround: Runtime detection and fallback

### 18.2 Distribution Differences

**Package Availability:**
- Some dependencies not in all distros
- Different package names across distros
- Version skew between rolling and stable releases
- Workaround: Document alternatives, bundle if necessary

**Desktop Environment Variations:**
- Theming inconsistencies
- Different keyboard shortcuts
- File picker behaviors
- Workaround: Test on multiple DEs, use XDG portals

---

## 19. Future Enhancements

### 19.1 Planned Features

**Phase 1 (Post-MVP):**
- Wayland screen recording via portal
- Touchscreen gesture optimization
- HiDPI refinements
- Additional desktop environment testing

**Phase 2 (Advanced Features):**
- MIDI device integration via ALSA sequencer
- Real-time collaboration via custom protocol
- Plugin system for extensions
- Advanced audio analysis features

**Phase 3 (Ecosystem Integration):**
- Integration with Ardour/LMMS/MuseScore
- Jack Transport synchronization
- LV2 plugin hosting for effects
- Linux VST support for virtual instruments

### 19.2 Research Areas

**Emerging Technologies:**
- Pipewire MIDI support (when stable)
- Flatpak portals for new capabilities
- Wayland protocol extensions
- GPU compute for audio processing (Vulkan)

---

## 20. Summary and Best Practices

### 20.1 Key Architectural Decisions

**Dual UI Framework Strategy:**
- GTK4/libadwaita for GNOME ecosystem native feel
- Qt6/KDE for Plasma ecosystem native integration
- Separate packages to maintain quality and focus
- Shared domain layer ensures consistent behavior

**Distribution Strategy:**
- Flatpak primary for universal distribution and security
- Snap for Ubuntu ecosystem
- AppImage for portability and user choice
- Native packages for deep integration

**Storage Strategy:**
- XDG Base Directory compliance
- Cloud integration via GIO/KIO
- XDG Desktop Portal for sandboxed access
- Universal .pbscore format for portability

**Performance:**
- Hardware-accelerated rendering
- Intelligent caching and virtualization
- Parallel processing for heavy operations
- Memory-conscious design for large documents

### 20.2 Development Priorities

**Phase 1 (MVP):**
1. GTK4 version with basic score editing
2. SMuFL rendering with Cairo
3. Local file storage with XDG compliance
4. Simple embellishment support
5. PDF export

**Phase 2 (Full Features):**
1. Qt6 version with equivalent features
2. Cloud storage integration (GIO/KIO)
3. Complete embellishment system
4. Audio recording and playback
5. Metronome with visual sync

**Phase 3 (Polish and Distribution):**
1. All package formats (Flatpak, Snap, AppImage, native)
2. Comprehensive desktop environment testing
3. Accessibility audit and improvements
4. Performance optimization
5. Translation completion

### 20.3 Maintenance Checklist

**Regular Tasks:**
- [ ] Monitor issue trackers for bug reports
- [ ] Respond to package maintainer requests
- [ ] Test on latest distribution releases
- [ ] Update dependencies for security patches
- [ ] Verify Flatpak/Snap functionality
- [ ] Test on new desktop environment versions
- [ ] Profile performance on various hardware
- [ ] Review and merge community contributions
- [ ] Update documentation for new features
- [ ] Coordinate security vulnerability fixes

**Release Cycle:**
- [ ] Feature freeze 2 weeks before release
- [ ] Beta testing period with community
- [ ] Final testing on all supported distributions
- [ ] Update translations
- [ ] Generate release notes and changelog
- [ ] Build all package formats
- [ ] Upload to distribution platforms
- [ ] Announce on community channels
- [ ] Monitor for critical issues post-release
- [ ] Prepare hotfix if necessary

---

**End of Linux Implementation Specification**

This specification provides comprehensive guidance for implementing the Pipe Band Score application on Linux with support for major desktop environments and distribution methods. All implementations must adhere to the domain architecture defined in `pipe_band_app_architecture.md` and UX patterns in `ux_interaction_design.md` while leveraging Linux-native capabilities for optimal user experience across the diverse Linux ecosystem.