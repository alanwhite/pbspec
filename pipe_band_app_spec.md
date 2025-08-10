# Scottish Pipe Band Instrument App - Clean Architecture Specification

## 1. Executive Summary

This specification outlines a cross-platform application for Scottish pipe band musicians, built using **platform-native languages** and **Clean Architecture** principles. The app targets bagpipes, bass drums, tenor drums, and snare drums with offline-first functionality and SMuFL-compliant music notation rendering across macOS, iOS, Android, Windows, and Linux platforms.

## 2. Architecture Overview

### 2.1 Clean Architecture Layers

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│   (Platform UI, ViewModels/BLoC,    │
│    Controllers, Platform-specific)  │
├─────────────────────────────────────┤
│           Domain Layer              │
│  (Entities, Use Cases, Repository   │
│    Interfaces, Value Objects)       │
├─────────────────────────────────────┤
│             Data Layer              │
│ (Repository Implementations, Data   │
│   Sources, Models, Platform APIs)   │
└─────────────────────────────────────┘
```

### 2.2 Technology Stack by Platform

#### 2.2.1 iOS/macOS Implementation
- **Language**: Swift 5.8+ with SwiftUI/UIKit
- **Architecture**: MVVM with Combine framework
- **Audio Processing**: AVFoundation, AudioKit
- **Local Storage**: Core Data with SQLite backing
- **Music Notation**: Core Graphics with SMuFL fonts
- **Cloud Integration**: Document Provider framework for universal cloud access

#### 2.2.2 Android Implementation
- **Language**: Kotlin 1.8+ with Coroutines
- **Architecture**: MVVM with ViewModels and StateFlow
- **Audio Processing**: Oboe (AAudio), MediaPlayer
- **Local Storage**: Room database with SQLite
- **Music Notation**: Canvas API with SMuFL fonts
- **Cloud Integration**: Storage Access Framework for universal provider support

#### 2.2.3 Windows Implementation
- **Language**: C# with .NET 6+ and WinUI 3
- **Architecture**: MVVM with Community Toolkit
- **Audio Processing**: NAudio, Windows.Media.Audio
- **Local Storage**: Entity Framework Core with SQLite
- **Music Notation**: Win2D/Canvas with SMuFL fonts
- **Cloud Integration**: Provider-specific APIs (OneDrive native, others via API)

#### 2.2.4 Linux Implementation
- **Language**: C++ with Qt 6 or Rust with Tauri
- **Architecture**: MVP/MVVM pattern
- **Audio Processing**: ALSA/PulseAudio
- **Local Storage**: SQLite with native bindings
- **Music Notation**: QPainter/Skia with SMuFL fonts
- **Cloud Integration**: Provider-specific APIs

#### 2.2.5 SMuFL Compliance Standards
- **Primary Font**: Bravura (recommended), Petaluma, or other SMuFL-compliant fonts
- **Unicode Ranges**: Musical symbols (U+1D100-U+1D1FF) and SMuFL Private Use Area
- **Engaging Defaults**: Optimized spacing, sizing, and positioning for pipe band notation
- **Cross-Platform Consistency**: Identical rendering across all platforms

## 3. Domain Layer (Platform-Agnostic)

### 3.1 Core Entities

The domain layer defines the business logic in a platform-independent manner. Each platform implements these concepts using their native languages while maintaining identical behavior.

#### 3.1.1 Instrument Hierarchy
```
Instrument (Abstract)
├── Bagpipes
├── BassDrum  
├── TenorDrum
└── SnareDrum
```

**Core Properties:**
- Unique identifier
- Instrument type enumeration
- Available tunings list
- Default clef assignment
- SMuFL symbol mappings

#### 3.1.2 Score Structure
```
ScoreDocument
├── Metadata (title, composer, version, sync status)
└── Tunes[] (1 or more tunes)
    ├── Tune Metadata (title, type, tempo, key, orientation)
    └── TuneLines[] (sequential lines that make up the tune)
        ├── TextLine (title, composer, headers, footers)
        └── Part (A, B, C, Intro, Outro, etc.)
            ├── Part Metadata (name, letter, play order, repeat count)
            └── MusicalSystems[] (staff systems with measures)
                ├── SystemStartElements[] (per instrument)
                │   ├── Clef (for this instrument at system start)
                │   └── Accidentals[] (sharp signs for bagpipe notation)
                ├── Measures[] (synchronized across all instruments)
                │   ├── InstrumentMeasures[] (per instrument)
                │   │   └── MusicalElements[] (notes, rests, ornaments)
                │   └── TimeSignature (cascading)
                └── Barlines[] (system-wide, spanning all instruments)
```

#### 3.1.3 Musical Elements

**Time Signature Cascading Logic:**
1. Check if current measure specifies time signature
2. If not, search backwards through previous measures
3. If none found, use tune's default time signature
4. Cache results for performance optimization

**Clef Management:**
- Default clefs: Bagpipes (Treble), Bass/Tenor Drums (Bass), Snare (Unpitched)
- Alternative clefs available per instrument type
- Render clef symbol when: first measure, clef change, system break

**Barline Types:**
- None, Single, Double, Final Double
- Repeat Start (|:), Repeat End (:|), Repeat Both (:|:)
- Dashed, Heavy, Dotted
- SMuFL codepoints for consistent rendering

### 3.2 Use Cases (Business Logic)

#### 3.2.1 Score Management Use Cases
- **CreateTuneUseCase**: Validate and create new tunes with proper structure
- **UpdateMeasureTimeSignatureUseCase**: Change time signatures with cascading validation
- **ValidateBarlineSequenceUseCase**: Ensure logical barline combinations
- **ReorderPartsUseCase**: Modify part play order and repeat counts

#### 3.2.2 Audio Use Cases
- **RecordPracticeSessionUseCase**: Capture and store practice recordings
- **PlayMetronomeUseCase**: Generate precise metronome based on time signature
- **AnalyzeIntonationUseCase**: Pitch detection and tuning feedback

#### 3.2.3 Export Use Cases
- **ExportToPDFUseCase**: Render score to PDF with SMuFL compliance
- **ExportToImageUseCase**: Generate PNG/SVG with configurable DPI
- **BatchExportUseCase**: Process multiple scores with various formats

#### 3.2.4 Synchronization Use Cases
- **SyncToCloudUseCase**: Upload changes to user's chosen cloud provider
- **ResolveConflictsUseCase**: Merge concurrent edits with conflict resolution
- **OfflineQueueUseCase**: Queue changes for later synchronization

### 3.3 Repository Interfaces

#### 3.3.1 Data Repositories
**TuneRepository**
- getAllTunes() -> List<Tune>
- getTuneById(id) -> Tune
- saveTune(tune) -> void
- deleteTune(id) -> void
- searchTunes(criteria) -> List<Tune>

**CloudStorageRepository**
- uploadScore(score, path) -> void
- downloadScore(path) -> ScoreDocument
- listFiles(directory) -> List<CloudFile>
- syncChanges() -> SyncResult

**AudioRepository**
- recordAudio(path, settings) -> void
- playAudio(path) -> void
- playMetronome(bpm, timeSignature) -> void
- analyzeIntonation(audioData) -> IntonationResult

## 4. Data Layer Implementation

### 4.1 Local Storage Strategy

#### 4.1.1 Platform-Specific Storage Solutions
- **iOS/macOS**: Core Data entities with JSON serialization for complex objects
- **Android**: Room database with type converters for complex types
- **Windows**: Entity Framework Core with SQLite provider
- **Linux**: SQLite with platform-specific bindings (Qt SQL or native C++)

#### 4.1.2 Data Persistence Architecture
**Primary Storage**: Local SQLite database for all user data
**Document Storage**: JSON serialization for score documents
**Audio Storage**: Platform-optimized audio formats (AAC, OGG, WAV)
**Cache Management**: LRU cache for frequently accessed scores
**Backup Strategy**: Automatic local backups with versioning

### 4.2 Cloud Storage Integration

#### 4.2.1 Universal Cloud Support Strategy

**iOS/macOS - Document Provider Framework**
- Automatic support for any installed File Provider extension
- Unified interface works with iCloud, Google Drive, Dropbox, OneDrive, Box, etc.
- No provider-specific code required in app
- User chooses destination through system picker

**Android - Storage Access Framework**
- Works with any DocumentsProvider implementation
- Automatic support for Google Drive, Dropbox, OneDrive, etc.
- Persistent URI permissions for reliable access
- User selects provider through system interface

**Windows - Hybrid Approach**
- Native OneDrive integration through Windows.Storage APIs
- Provider-specific APIs for Google Drive, Dropbox
- Fallback to local file picker for unsupported providers
- Manual provider registration required

**Linux - API-Based Integration**
- Provider-specific REST APIs
- OAuth 2.0 authentication flow
- Manual integration for each supported provider
- Local storage as primary option

#### 4.2.2 Conflict Resolution Strategy
**Three-Way Merge**: Compare local, remote, and common ancestor versions
**Last-Write-Wins**: Simple timestamp-based resolution for non-conflicting changes
**Manual Resolution**: Present user with diff interface for complex conflicts
**Backup Creation**: Always create backup before applying remote changes

## 5. Presentation Layer Architecture

### 5.1 Platform-Specific UI Patterns

#### 5.1.1 iOS/macOS - SwiftUI/UIKit
**Architecture**: MVVM with Combine for reactive programming
**State Management**: @StateObject, @ObservableObject, @Published properties
**Navigation**: NavigationView/NavigationStack with coordinator pattern
**Custom Views**: SMuFL symbol rendering with Core Graphics
**Accessibility**: VoiceOver support with proper semantic markup

#### 5.1.2 Android - Jetpack Compose
**Architecture**: MVVM with StateFlow and ViewModel
**State Management**: Compose state hoisting with remember/rememberSaveable
**Navigation**: Compose Navigation with type-safe arguments
**Custom Composables**: SMuFL rendering with Canvas API
**Accessibility**: TalkBack support with semantic properties

#### 5.1.3 Windows - WinUI 3
**Architecture**: MVVM with CommunityToolkit.Mvvm
**State Management**: ObservableObject with RelayCommand
**Navigation**: Frame navigation with dependency injection
**Custom Controls**: UserControls for SMuFL symbol rendering
**Accessibility**: Narrator support with automation properties

#### 5.1.4 Linux - Qt/C++
**Architecture**: MVP/MVVM with Qt's model-view framework
**State Management**: QML property bindings or C++ signals/slots
**Navigation**: QStackedWidget or QML StackView
**Custom Widgets**: QPainter-based SMuFL rendering
**Accessibility**: Qt Accessibility framework support

### 5.2 SMuFL Music Notation Rendering

#### 5.2.1 Cross-Platform Rendering Strategy
**Font Management**: Embed Bravura font in application bundle
**Symbol Rendering**: Platform-specific text rendering with proper font metrics
**Layout Engine**: Custom layout algorithms for musical spacing
**Export Consistency**: Identical rendering across all output formats

#### 5.2.2 SMuFL Integration Standards
**Codepoint Mapping**: Standardized Unicode to symbol mapping
**Symbol Sizing**: Consistent scaling based on staff height
**Positioning**: Proper baseline alignment and character spacing
**Fallback Handling**: Graceful degradation for missing symbols

## 6. Audio Processing Architecture

### 6.1 Platform-Specific Audio Engines

#### 6.1.1 Low-Latency Audio Requirements
**iOS/macOS**: AVAudioEngine with exclusive audio session
**Android**: Oboe library targeting AAudio for lowest latency
**Windows**: WASAPI exclusive mode for professional audio
**Linux**: ALSA direct access or PulseAudio with low-latency configuration

#### 6.1.2 Audio Features Implementation
**Metronome Generation**: Sample-accurate timing with accent patterns
**Recording Capability**: High-quality audio capture with real-time monitoring
**Playback Engine**: Gapless playback with tempo adjustment
**Audio Analysis**: Pitch detection and frequency analysis
**Format Support**: Platform-optimized codecs (AAC, OGG, FLAC)

### 6.2 Audio Processing Features

#### 6.2.1 Metronome System
**Timing Accuracy**: Sub-millisecond precision using audio callbacks
**Time Signature Support**: Complex meters with accent patterns
**Sound Customization**: Different tones for downbeats and subdivisions
**Visual Synchronization**: UI updates synchronized with audio beats

#### 6.2.2 Recording and Analysis
**Practice Session Recording**: Automatic recording with metadata
**Intonation Analysis**: Real-time pitch detection for tuning assistance
**Performance Metrics**: Timing accuracy and rhythm analysis
**Audio Quality**: Professional recording standards (44.1kHz+, 16-bit+)

## 7. SMuFL Music Notation System

### 7.1 SMuFL Implementation Strategy

#### 7.1.1 Font Integration Approach
**Font Embedding**: Bravura font bundled with application
**Platform Registration**: Proper font registration on each platform
**Fallback Fonts**: Petaluma or other SMuFL fonts as alternatives
**Custom Glyphs**: Platform-specific rendering for complex ornaments

#### 7.1.2 Symbol Rendering Standards
**Unicode Compliance**: Full SMuFL specification adherence
**Scaling System**: Proportional scaling based on staff size
**Anti-aliasing**: Smooth rendering at all display densities
**Color Support**: Configurable symbol colors for different elements

### 7.2 Pipe Band Notation Specifics

#### 7.2.1 Ornament Rendering
**Grace Note Patterns**: Proper spacing and beam connections
**Pipe Band Ornaments**: Doublings, throws, cuts, strikes
**Complex Ornaments**: Taorluath, crunluath with proper symbol sequences
**Ornament Positioning**: Above staff with consistent spacing

#### 7.2.2 Multi-Instrument Layout
**Staff Arrangement**: Proper spacing between instrument staves
**Clef Management**: Appropriate clefs for each instrument type
**System Breaks**: Intelligent page layout with musical phrase awareness
**Part Separation**: Clear visual distinction between tune parts

## 8. Export and File Format System

### 8.1 Multi-Format Export Engine

#### 8.1.1 Supported Export Formats
**PDF**: Vector-based output with embedded fonts
**PNG**: High-resolution raster images with configurable DPI
**SVG**: Scalable vector graphics for web compatibility
**MIDI**: Basic playback support for audio preview
**MusicXML**: Standard interchange format for score sharing

#### 8.1.2 Export Quality Standards
**Print Resolution**: 300+ DPI for professional printing
**Font Embedding**: Proper font subset embedding in PDF
**Color Management**: Consistent color reproduction across formats
**Metadata Inclusion**: Title, composer, copyright information in exports

### 8.2 Cloud Export Integration

#### 8.2.1 Direct Cloud Export
**Seamless Upload**: Export directly to user's chosen cloud provider
**Format Selection**: User choice of export format at save time
**Batch Processing**: Multiple scores exported simultaneously
**Progress Tracking**: Real-time feedback during export operations

#### 8.2.2 Sharing Capabilities
**Email Integration**: Direct email sharing with format selection
**Social Sharing**: Platform-specific sharing mechanisms
**Print Services**: Direct integration with system print services
**Collaboration**: Share with editing permissions for band members

## 9. Internationalization and Localization

### 9.1 Language Support Strategy

#### 9.1.1 Target Languages (Phase 1)
**UK English (en-GB)**: British spellings, metric system, 24-hour time
**US English (en-US)**: American spellings, imperial/metric mixed, 12-hour time
**French (fr-FR)**: European French, metric system, 24-hour time
**Spanish (es-ES)**: European Spanish, metric system, 24-hour time
**German (de-DE)**: Standard German, metric system, 24-hour time

#### 9.1.2 Localization Framework
**Platform Integration**: Native localization systems for each platform
**Cultural Adaptation**: Regional preferences for measurements and formatting
**Dynamic Language Switching**: Runtime language changes without restart
**Fallback Strategy**: Graceful degradation to default language

### 9.2 Musical Terminology Management

#### 9.2.1 Standardized Term Database
**Tune Types**: Consistent translation of march, strathspey, reel, jig, etc.
**Instruments**: Proper terminology for each instrument in target languages
**Technical Terms**: Pipe band specific terminology with cultural sensitivity
**Ornament Names**: Balance between translation and preservation of Gaelic terms

#### 9.2.2 Cultural Sensitivity
**Regional Variations**: Acknowledge local musical traditions
**Traditional Terms**: Preserve Scottish Gaelic terms where appropriate
**Educational Context**: Provide explanations for non-native speakers
**Accessibility**: Support for different literacy levels

## 10. Testing Strategy

### 10.1 Comprehensive Testing Approach

#### 10.1.1 Unit Testing (Platform-Specific)
**iOS/macOS**: XCTest framework with Quick/Nimble for BDD
**Android**: JUnit 5 with MockK for Kotlin-specific testing
**Windows**: xUnit with Moq for .NET testing
**Linux**: Google Test (C++) or built-in Rust testing

#### 10.1.2 Integration Testing
**Audio System Testing**: Latency measurement and quality verification
**Cloud Integration Testing**: Multi-provider synchronization validation
**Export Testing**: Cross-platform format consistency verification
**Localization Testing**: Cultural adaptation and translation accuracy

#### 10.1.3 UI Testing
**Platform-Specific**: Native UI testing frameworks for each platform
**Cross-Platform Validation**: Identical behavior verification
**Accessibility Testing**: Screen reader and keyboard navigation support
**Performance Testing**: Rendering speed and memory usage optimization

### 10.2 Quality Assurance Standards

#### 10.2.1 Performance Benchmarks
**Startup Time**: < 3 seconds cold start on target hardware
**Audio Latency**: < 20ms round-trip on all platforms
**Export Speed**: < 5 seconds for 10-page PDF generation
**Memory Usage**: < 200MB during active score editing

#### 10.2.2 Reliability Standards
**Offline Functionality**: 100% feature availability without network
**Data Integrity**: Zero data loss during synchronization conflicts
**Error Recovery**: Graceful handling of all error conditions
**Cross-Platform Consistency**: Identical behavior across all platforms

## 11. Security and Privacy

### 11.1 Data Protection Strategy

#### 11.1.1 Local Data Security
**Encryption at Rest**: AES-256 encryption for sensitive local data
**Secure Storage**: Platform-specific secure storage mechanisms
**Access Control**: Biometric authentication where available
**Data Isolation**: Sandboxed application data protection

#### 11.1.2 Cloud Security
**Provider Security**: Leverage existing cloud provider security measures
**End-to-End Encryption**: Optional client-side encryption before upload
**OAuth Integration**: Secure authentication without password storage
**Token Management**: Secure storage and automatic refresh of access tokens

### 11.2 Privacy Compliance

#### 11.2.1 Data Minimization
**Local-First Design**: Minimize cloud data requirements
**User Consent**: Explicit consent for all data sharing
**Data Portability**: Easy export of all user data
**Right to Deletion**: Complete data removal on user request

#### 11.2.2 Regulatory Compliance
**GDPR Compliance**: Full compliance for European users
**COPPA Compliance**: Safe for users under 13
**Platform Guidelines**: Adherence to app store privacy requirements
**Transparency**: Clear privacy policy and data handling practices

## 12. Performance Requirements

### 12.1 Platform-Specific Targets

#### 12.1.1 System Requirements
**iOS**: iOS 15.0+, iPhone 12+ recommended, 1GB free storage
**macOS**: macOS 12.0+, M1 or Intel Core i5+, 1GB free storage
**Android**: API 26+, 4GB RAM, 1GB free storage
**Windows**: Windows 10 1809+, 4GB RAM, 1GB free storage
**Linux**: Modern distribution, 4GB RAM, 1GB free storage

#### 12.1.2 Performance Benchmarks
**Audio Latency**: Platform-optimized for minimum achievable latency
**Rendering Performance**: 60fps during score scrolling and editing
**Export Performance**: Real-time rendering for most common scenarios
**Startup Performance**: Quick launch from cold start on target hardware

### 12.2 Scalability Considerations

#### 12.2.1 Data Scalability
**Large Score Support**: Efficient handling of complex multi-part arrangements
**Library Management**: Performance with 1000+ scores in library
**Search Performance**: Sub-second search across entire score library
**Memory Efficiency**: Intelligent loading and caching of score data

#### 12.2.2 Feature Scalability
**Modular Architecture**: Easy addition of new instruments and features
**Plugin Architecture**: Future support for third-party extensions
**Cloud Provider Support**: Framework for adding new storage providers
**Platform Extension**: Architecture supports additional target platforms

## 13. Development and Deployment

### 13.1 Development Methodology

#### 13.1.1 Platform-Native Development
**Team Structure**: Specialized teams for each platform
**Shared Domain Logic**: Common business rules across platforms
**Code Reviews**: Cross-platform consistency validation
**Integration Points**: Regular synchronization of domain changes

#### 13.1.2 Quality Assurance
**Continuous Integration**: Automated builds and testing for all platforms
**Device Testing**: Physical device testing across target hardware
**Beta Testing**: Closed beta with pipe band communities
**Performance Monitoring**: Real-time performance and crash analytics

### 13.2 Release Strategy

#### 13.2.1 Phased Rollout
**Phase 1**: Core functionality with basic score editing
**Phase 2**: Advanced notation features and cloud synchronization
**Phase 3**: Audio features and advanced collaboration
**Phase 4**: Community features and marketplace integration

#### 13.2.2 Platform Distribution
**iOS**: App Store with TestFlight beta distribution
**macOS**: Mac App Store and direct distribution
**Android**: Google Play Store with Play Console beta
**Windows**: Microsoft Store and direct installer
**Linux**: Package managers and AppImage distribution

## 14. Future Enhancements

### 14.1 Advanced Features Roadmap

#### 14.1.1 Phase 2 Enhancements
**Real-time Collaboration**: Multiple users editing simultaneously
**Advanced Audio Analysis**: Pitch correction and timing feedback
**AI Practice Assistant**: Intelligent practice recommendations
**Video Integration**: Record practice sessions with synchronized score

#### 14.1.2 Phase 3 Enhancements
**Hardware Integration**: MIDI controller and electronic instrument support
**AR/VR Support**: Augmented reality music stand and immersive practice
**Community Features**: Score sharing marketplace and social features
**Competition Integration**: Direct integration with pipe band competition systems

### 14.2 Platform Evolution

#### 14.2.1 Emerging Platforms
**Web Platform**: Progressive Web App for cross-platform accessibility
**Smart TV**: Large screen display for band practice sessions
**Wearable Devices**: Practice tracking and metronome on smartwatches
**IoT Integration**: Wireless metronome synchronization across band

#### 14.2.2 Technology Evolution
**Machine Learning**: Advanced audio analysis and practice feedback
**Cloud Computing**: Server-side audio processing and analysis
**Blockchain**: Decentralized score sharing and copyright protection
**5G Integration**: Real-time collaboration over high-speed mobile networks

## 15. Conclusion

This Clean Architecture specification provides a comprehensive foundation for developing a professional Scottish pipe band application across multiple platforms using native technologies. The architecture ensures:

**Maintainability**: Clear separation of concerns with platform-agnostic domain logic
**Scalability**: Modular design supporting feature additions and platform extensions
**Performance**: Native platform optimization for audio processing and UI rendering
**User Experience**: SMuFL-compliant notation with engaging defaults for pipe band music
**Global Accessibility**: Comprehensive internationalization for worldwide pipe band communities
**Data Sovereignty**: User control over data storage with flexible cloud integration

**Key Architectural Benefits:**

**Platform-Native Excellence**: Each platform uses its optimal technologies and patterns for best performance and user experience
**Universal Domain Logic**: Business rules implemented consistently across all platforms while leveraging platform-specific optimizations
**Offline-First Design**: Complete functionality without network dependency, ensuring reliability in any location
**Professional Music Notation**: SMuFL-compliant rendering delivering publication-quality scores
**Flexible Cloud Integration**: Support for any cloud provider through platform-native file system integration
**Cultural Sensitivity**: Thoughtful internationalization respecting global pipe band traditions

The specification addresses real-world needs of the international pipe band community, from Highland Games to European competitions, ensuring musicians can practice, collaborate, and perform at the highest level regardless of their location or preferred platform. The offline-first approach combined with professional notation rendering ensures reliability and quality that meets the demanding standards of competitive pipe band music.

This architecture will deliver a world-class application that serves the global pipe band community while maintaining the highest standards of software engineering, user experience, and musical authenticity.
