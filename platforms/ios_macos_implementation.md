# iOS/macOS Implementation Specification

**Version:** 1.0  
**Last Updated:** September 30, 2025  
**Platforms:** iOS 17.0+, macOS 14.0+  
**Primary Distribution:** Apple App Store

---

## Document Purpose and Scope

This document specifies the **iOS and macOS platform-specific implementation** of the Scottish pipe band application, building upon the platform-agnostic domain architecture defined in `pipe_band_app_architecture.md`.

**Reference Architecture:**
- **Domain Specification:** `pipe_band_app_architecture.md`
- **Domain Entities:** Section 3.1 (Instruments, Score, Tune, Part, Measure, Note, Embellishments)
- **Use Cases:** Section 3.4 (Score Management, Musical Editing, Pagination)
- **Repository Interfaces:** Section 3.5 (Data access contracts)
- **File Format:** Section 4 (.pbscore JSON specification)

**What This Document Contains:**
- Swift 6 concurrency architecture with strict data isolation
- SwiftUI presentation layer with Observation framework
- App Store distribution requirements and compliance
- iCloud Container + File Provider storage modes
- AVFoundation audio processing with actor isolation
- Core Graphics + Text Kit 2 SMuFL rendering
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
- **Swift 6.0+** with complete concurrency enabled
- **Strict data isolation** and sendable conformance throughout
- **Actor-based architecture** for thread-safe domain operations
- **Structured concurrency** (async/await, TaskGroup, AsyncSequence)
- **MainActor coordination** for all UI operations

**UI Framework:**
- **SwiftUI** as primary UI framework
- **Observation framework** (@Observable) replacing Combine/ObservableObject
- **UIKit/AppKit bridging** where necessary for advanced features
- **Mac Catalyst** for unified codebase with platform-specific adaptations

**Architecture Pattern:**
- **MVVM** with Swift Concurrency integration
- **Repository Pattern** with actor-isolated implementations
- **Coordinator Pattern** for navigation with type-safe routing
- **Clean Architecture** layers maintained through module boundaries

**Audio Processing:**
- **AVFoundation** with AVAudioEngine
- **Actor-isolated audio pipeline** for thread-safe processing
- **Hardware sample rate matching** (44.1kHz, 48kHz, 96kHz)
- **AudioKit integration** (optional) for advanced features

**Local Storage:**
- **Core Data** with SQLite backing store
- **NSPersistentCloudKitContainer** for iCloud sync
- **Actor-isolated Core Data contexts** for thread safety
- **Background processing** with proper context isolation

**Music Notation:**
- **Core Graphics** for vector rendering
- **Text Kit 2** for SMuFL text layout
- **CTFont** for SMuFL font rendering
- **Metal acceleration** (optional) for complex scores

**Cloud Integration:**
- **CloudKit** with CKShare for real-time collaboration (iCloud mode)
- **Document Provider** framework for universal cloud support (File Provider mode)
- **NSFileCoordinator** for coordinated file access
- **UICloudSharingController** for native sharing UI

### 1.2 App Store Distribution Requirements

**Required Entitlements:**
```
Entitlements Configuration:
├── iCloud Container
│   └── com.apple.developer.icloud-container-identifiers
├── CloudKit
│   └── com.apple.developer.icloud-services
├── File Provider
│   └── com.apple.developer.ubiquity-container-identifiers
├── Audio Input
│   └── com.apple.security.device.audio-input
└── Background Modes
    ├── audio (metronome, playback)
    ├── fetch (cloud sync)
    └── processing (audio analysis)
```

**Info.plist Privacy Descriptions:**
```
Required Privacy Descriptions:
├── NSMicrophoneUsageDescription
│   └── "Record practice sessions to track your progress and analyze performance"
├── NSUserNotificationsUsageDescription
│   └── "Remind you of practice sessions and notify about collaboration updates"
├── NSAppleMusicUsageDescription (if accessing music library)
│   └── "Import scores from your music library"
└── Additional descriptions as needed for optional features
```

**Privacy Manifest (PrivacyInfo.xcprivacy):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeOtherUserContent</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryDiskSpace</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>E174.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

**App Store Compliant Frameworks:**
```
Allowed Technologies:
├── StoreKit 2 (for future IAP if monetization added)
├── AppTrackingTransparency (required if any analytics added)
├── UICloudSharingController (native collaboration UI)
├── BackgroundTasks (efficient background sync)
├── CallKit (if audio session management needed)
└── WidgetKit (for home screen widgets - future enhancement)

Prohibited Technologies:
├── ❌ Web views for core functionality (violates 2.5.2)
├── ❌ JavaScript code execution or bridges (violates 2.5.2)
├── ❌ Hot code push or dynamic feature delivery
├── ❌ Private API usage (automatic rejection)
├── ❌ Third-party authentication SDKs without Sign in with Apple
└── ❌ UIWebView (deprecated, use WKWebView if absolutely needed)
```

---

## 2. Storage Architecture Implementation

### 2.1 Two-Tier Storage Strategy

**Storage Mode Enumeration:**
```swift
@frozen
enum StorageMode: Sendable, Codable {
    case iCloudContainer    // Primary, recommended for App Store
    case fileProvider       // Advanced users, universal cloud support
}
```

**Strategic Recommendation for App Store:**

**iCloud Container Mode (Primary)** is strongly recommended because:
- Seamless integration with user's existing Apple ID (no third-party auth)
- Automatic privacy compliance through Apple's infrastructure
- Enhanced App Store discoverability ("Works with iCloud" badge)
- Reduced App Review friction (well-understood Apple technology)
- Native collaboration UI (UICloudSharingController)
- Zero-configuration user experience (95%+ of iOS users have iCloud)
- Automatic encryption and security compliance
- Real-time collaboration features built-in

**File Provider Mode (Advanced)** is available for:
- Enterprise requirements with specific cloud provider mandates
- Cross-platform band collaboration needing specific cloud services
- Power users preferring specific cloud storage providers
- Organizations with existing cloud infrastructure investments

### 2.2 Document-Based Architecture

**UIDocument/NSDocument Integration:**

**Platform-Specific Document Class:**
```swift
@MainActor
final class PipeBandScoreDocument: UIDocument { // NSDocument on macOS
    
    // MARK: - Domain Model (Sendable)
    
    private(set) var scoreDocument: ScoreDocument // From domain layer
    
    // MARK: - Actor-Isolated Repository
    
    private let repository: DocumentRepository
    
    // MARK: - Initialization
    
    init(fileURL: URL, repository: DocumentRepository) {
        self.repository = repository
        self.scoreDocument = ScoreDocument() // Empty initial state
        super.init(fileURL: fileURL)
    }
    
    // MARK: - Document Loading (Async)
    
    override func load(fromContents contents: Any, ofType typeName: String?) async throws {
        guard let data = contents as? Data else {
            throw DocumentError.invalidContents
        }
        
        // Parse JSON on background actor
        let parsedDocument = try await DocumentParser.parse(data)
        
        // Validate against domain rules
        let validationErrors = await DocumentValidator.validate(parsedDocument)
        guard validationErrors.isEmpty else {
            throw DocumentError.validationFailed(validationErrors)
        }
        
        // Update on MainActor
        self.scoreDocument = parsedDocument
    }
    
    // MARK: - Document Saving (Async)
    
    override func contents(forType typeName: String) async throws -> Any {
        // Serialize on background actor
        let data = try await DocumentSerializer.serialize(scoreDocument)
        return data
    }
    
    // MARK: - Change Tracking
    
    func updateScore(_ updater: @Sendable (inout ScoreDocument) -> Void) {
        var mutableScore = scoreDocument
        updater(&mutableScore)
        scoreDocument = mutableScore
        updateChangeCount(.done)
    }
}
```

**Document Browser Integration:**
```swift
@MainActor
final class DocumentBrowserViewController: UIDocumentBrowserViewController {
    
    private let storageMode: StorageMode
    
    init(storageMode: StorageMode) {
        self.storageMode = storageMode
        super.init(forOpening: [.pbscore])
        
        configureAppearance()
        setupBrowserDelegate()
    }
    
    // MARK: - Storage Mode Configuration
    
    private func configureAppearance() {
        switch storageMode {
        case .iCloudContainer:
            // Show iCloud-specific UI elements
            allowsDocumentCreation = true
            allowsPickingMultipleItems = false
            
        case .fileProvider:
            // Show File Provider UI elements
            allowsDocumentCreation = true
            allowsPickingMultipleItems = false
        }
    }
    
    // MARK: - Document Creation
    
    func createNewDocument() async throws -> URL {
        let template = try await DocumentTemplate.createEmpty()
        
        switch storageMode {
        case .iCloudContainer:
            return try await createInICloudContainer(template)
            
        case .fileProvider:
            return try await createInFileProvider(template)
        }
    }
    
    private func createInICloudContainer(_ template: Data) async throws -> URL {
        guard let containerURL = FileManager.default.url(
            forUbiquityContainerIdentifier: nil
        ) else {
            throw StorageError.iCloudUnavailable
        }
        
        let documentsURL = containerURL.appendingPathComponent("Documents")
        let newFileURL = documentsURL.appendingPathComponent(
            "Untitled.pbscore",
            isDirectory: false
        )
        
        try template.write(to: newFileURL, options: .atomic)
        return newFileURL
    }
    
    private func createInFileProvider(_ template: Data) async throws -> URL {
        // Use UIDocumentPickerViewController for save location
        // Returns URL selected by user from any File Provider
        let pickerURL = try await presentSaveLocationPicker()
        try template.write(to: pickerURL, options: .atomic)
        return pickerURL
    }
}
```

### 2.3 iCloud Container Mode Implementation

**CloudKit Integration with Actor Isolation:**

**CloudKit Collaboration Manager:**
```swift
actor CloudKitCollaborationManager {
    
    private let container: CKContainer
    private let database: CKDatabase
    
    init(containerIdentifier: String? = nil) {
        self.container = CKContainer(identifier: containerIdentifier ?? "iCloud.com.yourapp.pipebandapp")
        self.database = container.privateCloudDatabase
    }
    
    // MARK: - Real-Time Collaboration
    
    func enableCollaboration(
        for document: PipeBandScoreDocument
    ) async throws -> CKShare {
        let share = CKShare(rootRecord: try await getDocumentRecord(document))
        share[CKShare.SystemFieldKey.title] = document.scoreDocument.metadata.title
        
        try await database.save(share)
        return share
    }
    
    func acceptShare(metadata: CKShare.Metadata) async throws {
        try await container.accept(metadata)
    }
    
    // MARK: - Sync Coordination
    
    func syncChanges(for documentID: UUID) async throws {
        // Fetch changes from CloudKit
        let changes = try await fetchChanges(since: lastSyncToken)
        
        // Apply changes with conflict resolution
        try await applyChanges(changes, resolveConflicts: true)
    }
    
    // MARK: - Conflict Resolution
    
    func resolveConflicts(
        local: ScoreDocument,
        remote: ScoreDocument
    ) async -> ScoreDocument {
        // Musical intelligence-based conflict resolution
        // Prefer most recent edits for musical content
        // Use timestamp-based resolution for metadata
        
        return await ConflictResolver.resolve(local: local, remote: remote)
    }
}
```

**NSPersistentCloudKitContainer Configuration:**
```swift
actor PersistentContainerConfigurator {
    
    static func configure() -> NSPersistentCloudKitContainer {
        let container = NSPersistentCloudKitContainer(name: "PipeBandApp")
        
        // Configure CloudKit sync
        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("Failed to retrieve persistent store description")
        }
        
        // Enable CloudKit integration
        description.setOption(
            true as NSNumber,
            forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey
        )
        
        description.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.yourapp.pipebandapp"
        )
        
        container.loadPersistentStores { storeDescription, error in
            if let error = error {
                fatalError("Failed to load persistent stores: \(error)")
            }
        }
        
        // Enable automatic merging
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        return container
    }
}
```

### 2.4 File Provider Mode Implementation

**Universal File Provider Support:**

**File Provider Repository:**
```swift
actor FileProviderRepository: DocumentRepository {
    
    // MARK: - Document Operations
    
    func save(document: MusicalDocument) async throws {
        let data = try await DocumentSerializer.serialize(document)
        
        // Use NSFileCoordinator for safe file access
        let coordinator = NSFileCoordinator()
        var coordinationError: NSError?
        
        try await withCheckedThrowingContinuation { continuation in
            coordinator.coordinate(
                writingItemAt: document.fileURL,
                options: .forReplacing,
                error: &coordinationError
            ) { url in
                do {
                    try data.write(to: url, options: .atomic)
                    continuation.resume()
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
        
        if let error = coordinationError {
            throw error
        }
    }
    
    func load(id: UUID) async throws -> MusicalDocument {
        guard let fileURL = try await getFileURL(for: id) else {
            throw RepositoryError.documentNotFound
        }
        
        let coordinator = NSFileCoordinator()
        var coordinationError: NSError?
        var documentData: Data?
        
        try await withCheckedThrowingContinuation { continuation in
            coordinator.coordinate(
                readingItemAt: fileURL,
                options: [],
                error: &coordinationError
            ) { url in
                do {
                    documentData = try Data(contentsOf: url)
                    continuation.resume()
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
        
        if let error = coordinationError {
            throw error
        }
        
        guard let data = documentData else {
            throw RepositoryError.failedToReadDocument
        }
        
        return try await DocumentParser.parse(data)
    }
    
    // MARK: - Cloud Provider Coordination
    
    func syncWithProvider() async throws {
        // File Provider extensions handle sync automatically
        // Monitor file system events for changes
        
        let fileMonitor = FileSystemMonitor()
        for await event in fileMonitor.events {
            switch event {
            case .fileModified(let url):
                try await handleExternalModification(at: url)
            case .fileDeleted(let url):
                try await handleExternalDeletion(at: url)
            }
        }
    }
}
```

### 2.5 First Launch Experience (App Store Compliant)

**iCloud Account Detection and Setup:**

**Onboarding Coordinator:**
```swift
@MainActor
@Observable
final class OnboardingCoordinator {
    
    var currentStep: OnboardingStep = .welcome
    var iCloudAvailable: Bool = false
    var selectedStorageMode: StorageMode?
    
    // MARK: - Initialization
    
    init() {
        Task {
            await checkiCloudAvailability()
        }
    }
    
    // MARK: - iCloud Availability Check
    
    func checkiCloudAvailability() async {
        let ubiquityToken = FileManager.default.ubiquityIdentityToken
        self.iCloudAvailable = (ubiquityToken != nil)
        
        if iCloudAvailable {
            // 95% of users - default to iCloud mode
            self.selectedStorageMode = .iCloudContainer
            self.currentStep = .explainICloudBenefits
        } else {
            // Rare case - guide user to enable iCloud or offer File Provider
            self.currentStep = .iCloudSetupGuidance
        }
    }
    
    // MARK: - Onboarding Flow
    
    enum OnboardingStep {
        case welcome
        case explainICloudBenefits
        case iCloudSetupGuidance
        case fileProviderOption
        case completed
    }
    
    func proceed() {
        switch currentStep {
        case .welcome:
            currentStep = iCloudAvailable ? .explainICloudBenefits : .iCloudSetupGuidance
            
        case .explainICloudBenefits:
            // User understands iCloud benefits, proceed to app
            currentStep = .completed
            
        case .iCloudSetupGuidance:
            // Offer File Provider as alternative
            currentStep = .fileProviderOption
            
        case .fileProviderOption:
            selectedStorageMode = .fileProvider
            currentStep = .completed
            
        case .completed:
            break
        }
    }
}
```

**Onboarding Views:**
```swift
struct OnboardingView: View {
    @State private var coordinator = OnboardingCoordinator()
    
    var body: some View {
        Group {
            switch coordinator.currentStep {
            case .welcome:
                WelcomeView(coordinator: coordinator)
                
            case .explainICloudBenefits:
                ICloudBenefitsView(coordinator: coordinator)
                
            case .iCloudSetupGuidance:
                ICloudSetupGuidanceView(coordinator: coordinator)
                
            case .fileProviderOption:
                FileProviderOptionView(coordinator: coordinator)
                
            case .completed:
                DocumentBrowserView(
                    storageMode: coordinator.selectedStorageMode ?? .iCloudContainer
                )
            }
        }
        .animation(.easeInOut, value: coordinator.currentStep)
    }
}

struct ICloudBenefitsView: View {
    let coordinator: OnboardingCoordinator
    
    var body: some View {
        VStack(spacing: 32) {
            Image(systemName: "icloud.and.arrow.up")
                .font(.system(size: 72))
                .foregroundStyle(.blue.gradient)
            
            VStack(spacing: 16) {
                Text("Your Scores, Everywhere")
                    .font(.largeTitle.bold())
                
                Text("iCloud keeps your scores in sync across all your Apple devices automatically.")
                    .font(.body)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
            }
            
            VStack(alignment: .leading, spacing: 20) {
                BenefitRow(
                    icon: "arrow.triangle.2.circlepath",
                    title: "Automatic Sync",
                    description: "Changes sync instantly across iPhone, iPad, and Mac"
                )
                
                BenefitRow(
                    icon: "person.2",
                    title: "Real-Time Collaboration",
                    description: "Edit scores together with your band members"
                )
                
                BenefitRow(
                    icon: "lock.shield",
                    title: "Secure & Private",
                    description: "Your scores are encrypted and only accessible by you"
                )
            }
            .padding(.vertical)
            
            Button("Get Started") {
                coordinator.proceed()
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
        }
        .padding()
    }
}

struct ICloudSetupGuidanceView: View {
    let coordinator: OnboardingCoordinator
    
    var body: some View {
        VStack(spacing: 32) {
            Image(systemName: "exclamationmark.icloud")
                .font(.system(size: 72))
                .foregroundStyle(.orange)
            
            VStack(spacing: 16) {
                Text("iCloud Not Enabled")
                    .font(.largeTitle.bold())
                
                Text("Enable iCloud in Settings to get the best experience with automatic sync and collaboration.")
                    .font(.body)
                    .foregroundStyle(.secondary)
                    .multilineTextAlignment(.center)
            }
            
            VStack(alignment: .leading, spacing: 12) {
                Text("To enable iCloud:")
                    .font(.headline)
                
                SetupStep(number: 1, text: "Open Settings app")
                SetupStep(number: 2, text: "Tap your name at the top")
                SetupStep(number: 3, text: "Tap iCloud")
                SetupStep(number: 4, text: "Turn on iCloud Drive")
            }
            .padding()
            .background(Color(.systemGroupedBackground))
            .cornerRadius(12)
            
            Button("Open Settings") {
                if let url = URL(string: UIApplication.openSettingsURLString) {
                    UIApplication.shared.open(url)
                }
            }
            .buttonStyle(.borderedProminent)
            .controlSize(.large)
            
            Button("Continue Without iCloud") {
                coordinator.proceed()
            }
            .buttonStyle(.bordered)
        }
        .padding()
    }
}
```

**Permission Request Pattern (App Store Compliant):**
```swift
@MainActor
final class PermissionCoordinator: ObservableObject {
    
    // MARK: - Microphone Permission (Contextual)
    
    func requestMicrophonePermission() async -> Bool {
        // Only request when user taps "Record Practice Session"
        let status = AVCaptureDevice.authorizationStatus(for: .audio)
        
        switch status {
        case .authorized:
            return true
            
        case .notDetermined:
            // Show pre-permission explanation
            let userConsent = await showPrePermissionExplanation(
                title: "Microphone Access",
                message: "To record your practice sessions, Pipe Band Score Editor needs access to your microphone.",
                benefit: "Record and analyze your performance to track progress over time."
            )
            
            guard userConsent else { return false }
            
            // Show system permission dialog
            return await AVCaptureDevice.requestAccess(for: .audio)
            
        case .denied, .restricted:
            // Guide user to Settings
            await showSettingsGuidance(for: "Microphone")
            return false
            
        @unknown default:
            return false
        }
    }
    
    // MARK: - Notification Permission (After Positive Engagement)
    
    func requestNotificationPermission() async -> Bool {
        // Only request after user creates their first score
        let center = UNUserNotificationCenter.current()
        let settings = await center.notificationSettings()
        
        switch settings.authorizationStatus {
        case .authorized, .provisional:
            return true
            
        case .notDetermined:
            let userConsent = await showPrePermissionExplanation(
                title: "Practice Reminders",
                message: "Get notified about practice sessions and collaboration updates.",
                benefit: "Stay on track with your practice schedule."
            )
            
            guard userConsent else { return false }
            
            do {
                return try await center.requestAuthorization(options: [.alert, .sound, .badge])
            } catch {
                return false
            }
            
        case .denied:
            await showSettingsGuidance(for: "Notifications")
            return false
            
        @unknown default:
            return false
        }
    }
    
    // MARK: - Pre-Permission Explanation
    
    private func showPrePermissionExplanation(
        title: String,
        message: String,
        benefit: String
    ) async -> Bool {
        // Show custom explanation sheet before system dialog
        await withCheckedContinuation { continuation in
            // Present custom UI explaining benefits
            // Return true if user taps "Allow", false if "Not Now"
            continuation.resume(returning: true)
        }
    }
}
```

---

## 3. Swift 6 Concurrency Architecture

### 3.1 Actor-Based Domain Layer

**Sendable Domain Entities:**

All domain entities must conform to `Sendable` for safe cross-actor usage:

```swift
// Domain Entity Example
struct ScoreDocument: Sendable, Codable, Identifiable {
    let id: UUID
    var metadata: DocumentMetadata
    var documentLayout: DocumentLayoutSettings
    var tunes: [Tune]
    var noteGroups: [UUID: NoteGroup]
    
    init(
        id: UUID = UUID(),
        metadata: DocumentMetadata = DocumentMetadata(),
        documentLayout: DocumentLayoutSettings = DocumentLayoutSettings(),
        tunes: [Tune] = [],
        noteGroups: [UUID: NoteGroup] = [:]
    ) {
        self.id = id
        self.metadata = metadata
        self.documentLayout = documentLayout
        self.tunes = tunes
        self.noteGroups = noteGroups
    }
}

struct DocumentMetadata: Sendable, Codable {
    var title: String
    var composer: String?
    var arranger: String?
    var copyright: String?
    var created: Date
    var modified: Date
}

struct Tune: Sendable, Codable, Identifiable {
    let id: UUID
    var metadata: TuneMetadata
    var tuneLayoutPreference: TuneLayoutPreference?
    var parts: [Part]
}

// Note Type Hierarchy with Sendable
protocol Note: Sendable, Codable, Identifiable {
    var id: UUID { get }
    var noteType: NoteType { get }
    var durationAdjustment: Float { get }
    var embellishment: (any Embellishment)? { get }
    var articulation: [Articulation] { get }
    var noteGroups: [UUID] { get }
    
    func getBaseDuration() -> Duration
    func getEffectiveDuration() -> Duration
}

struct PipeNote: Note {
    let id: UUID
    var noteType: NoteType
    var durationAdjustment: Float
    var embellishment: (any Embellishment)?
    var articulation: [Articulation]
    var noteGroups: [UUID]
    
    // Pipe-specific
    var pitch: Pitch
    var accidental: Accidental?
    
    func getBaseDuration() -> Duration {
        noteType.baseDuration
    }
    
    func getEffectiveDuration() -> Duration {
        getBaseDuration() * Duration(durationAdjustment)
    }
}

struct SnareDrumNote: Note {
    let id: UUID
    var noteType: NoteType
    var durationAdjustment: Float
    var embellishment: (any Embellishment)?
    var articulation: [Articulation]
    var noteGroups: [UUID]
    
    // Snare-specific
    var hand: Hand
    var stickTechnique: StickTechnique
    var stickHeight: StickHeight?
    
    func getBaseDuration() -> Duration {
        noteType.baseDuration
    }
    
    func getEffectiveDuration() -> Duration {
        getBaseDuration() * Duration(durationAdjustment)
    }
}
```

**Actor-Isolated Use Cases:**

```swift
actor ScoreEditingCoordinator {
    
    private let repository: DocumentRepository
    private var openDocuments: [UUID: ScoreDocument] = [:]
    
    init(repository: DocumentRepository) {
        self.repository = repository
    }
    
    // MARK: - Use Case: Create Score
    
    func createScore(
        title: String,
        composer: String?,
        defaultInstruments: [InstrumentType],
        layoutSettings: DocumentLayoutSettings?
    ) async throws -> ScoreDocument {
        // Business rules validation
        guard !title.isEmpty else {
            throw UseCaseError.invalidTitle
        }
        
        // Create initial document
        var document = ScoreDocument(
            metadata: DocumentMetadata(
                title: title,
                composer: composer,
                created: Date(),
                modified: Date()
            ),
            documentLayout: layoutSettings ?? DocumentLayoutSettings()
        )
        
        // Create initial tune with A part
        let initialTune = Tune(
            metadata: TuneMetadata(
                title: title,
                tuneType: .march,
                tempo: 80
            ),
            parts: [
                Part(
                    partLetter: "A",
                    systems: [
                        MusicalSystem(
                            instruments: defaultInstruments,
                            measures: [
                                Measure(timeSignature: TimeSignature(numerator: 4, denominator: 4))
                            ]
                        )
                    ]
                )
            ]
        )
        
        document.tunes = [initialTune]
        
        // Save through repository
        try await repository.save(document: document)
        
        // Cache in memory
        openDocuments[document.id] = document
        
        return document
    }
    
    // MARK: - Use Case: Add Note to Measure
    
    func addNote(
        to measureId: UUID,
        instrumentId: InstrumentID,
        note: any Note,
        at position: Int,
        in documentId: UUID
    ) async throws {
        guard var document = openDocuments[documentId] else {
            throw UseCaseError.documentNotOpen
        }
        
        // Find measure and validate
        guard let measure = findMeasure(measureId, in: document) else {
            throw UseCaseError.measureNotFound
        }
        
        // Validate note type matches instrument
        try validateNoteType(note, for: instrumentId)
        
        // Validate measure duration doesn't overflow
        try validateDuration(adding: note, to: measure)
        
        // Add note to measure
        var updatedMeasure = measure
        updatedMeasure.addNote(note, at: position, for: instrumentId)
        
        // Update document
        document = replaceMeasure(measureId, with: updatedMeasure, in: document)
        
        // Save changes
        try await repository.save(document: document)
        
        // Update cache
        openDocuments[documentId] = document
    }
    
    // MARK: - Use Case: Attach Embellishment
    
    func attachEmbellishment(
        _ embellishment: any Embellishment,
        to noteId: UUID,
        in documentId: UUID
    ) async throws {
        guard var document = openDocuments[documentId] else {
            throw UseCaseError.documentNotOpen
        }
        
        // Find note and validate
        guard let note = findNote(noteId, in: document) else {
            throw UseCaseError.noteNotFound
        }
        
        // Validate embellishment compatibility
        try validateEmbellishmentCompatibility(embellishment, with: note)
        
        // Check prior note constraints if applicable
        if let priorNote = findPriorNote(to: noteId, in: document) {
            try validatePriorNoteConstraints(embellishment, priorNote: priorNote)
        }
        
        // Attach embellishment
        var updatedNote = note
        updatedNote.embellishment = embellishment
        
        // Update document
        document = replaceNote(noteId, with: updatedNote, in: document)
        
        // Save changes
        try await repository.save(document: document)
        
        // Update cache
        openDocuments[documentId] = document
    }
}
```

### 3.2 Concurrent Layout Calculation

**Layout Calculation Actor:**

```swift
actor LayoutCalculationEngine {
    
    private var layoutCache: [UUID: SystemLayout] = [:]
    
    // MARK: - Document Layout
    
    func calculateDocumentLayout(
        _ document: ScoreDocument,
        context: LayoutContext
    ) async throws -> DocumentLayoutResult {
        // Use TaskGroup for parallel page calculations
        try await withThrowingTaskGroup(of: PageLayoutResult.self) { group in
            var results: [PageLayoutResult] = []
            
            for pageIndex in 0..<document.pages.count {
                group.addTask {
                    try await self.calculatePageLayout(
                        document: document,
                        pageIndex: pageIndex,
                        context: context
                    )
                }
            }
            
            for try await result in group {
                results.append(result)
            }
            
            return DocumentLayoutResult(pages: results.sorted(by: { $0.pageIndex < $1.pageIndex }))
        }
    }
    
    // MARK: - System Layout
    
    func calculateSystemLayout(
        _ system: MusicalSystem,
        context: SystemLayoutContext
    ) async throws -> SystemLayoutResult {
        // Check cache first
        if let cached = layoutCache[system.id], !cached.isStale {
            return SystemLayoutResult(layout: cached)
        }
        
        // Calculate measure widths
        let measureLayouts = try await withThrowingTaskGroup(of: (UUID, MeasureLayout).self) { group in
            var layouts: [UUID: MeasureLayout] = [:]
            
            for measure in system.measures {
                group.addTask {
                    let layout = try await self.calculateMeasureLayout(
                        measure,
                        context: context.measureContext
                    )
                    return (measure.id, layout)
                }
            }
            
            for try await (id, layout) in group {
                layouts[id] = layout
            }
            
            return layouts
        }
        
        // Calculate system width and staff spacing
        let systemLayout = SystemLayout(
            systemId: system.id,
            measureLayouts: measureLayouts,
            staffSpacing: calculateOptimalStaffSpacing(system, context: context),
            totalWidth: measureLayouts.values.reduce(0) { $0 + $1.width }
        )
        
        // Cache result
        layoutCache[system.id] = systemLayout
        
        return SystemLayoutResult(layout: systemLayout)
    }
    
    // MARK: - Embellishment Layout
    
    func calculateEmbellishmentLayout(
        _ embellishment: any Embellishment,
        principalNote: any Note,
        context: LayoutContext
    ) async throws -> EmbellishmentLayout {
        // Calculate grace note positions
        let gracePositions = await embellishment.calculateGraceNotePositions(
            principalNote: principalNote,
            context: context
        )
        
        // Handle floating before barline
        let shouldFloat = embellishment.shouldFloatBeforeBarline && context.isFirstNoteInMeasure
        
        return EmbellishmentLayout(
            graceNotePositions: gracePositions,
            floatBeforeBarline: shouldFloat,
            totalWidth: gracePositions.reduce(0) { $0 + $1.width }
        )
    }
    
    // MARK: - Cache Management
    
    func invalidateCache(for entityIds: [UUID]) {
        for id in entityIds {
            layoutCache.removeValue(forKey: id)
        }
    }
}
```

### 3.3 MainActor UI Coordination

**Observable View Models:**

```swift
@MainActor
@Observable
final class ScoreEditorViewModel {
    
    // MARK: - Published State
    
    private(set) var document: ScoreDocument
    private(set) var currentPage: Int = 0
    private(set) var selectedNotes: Set<UUID> = []
    private(set) var isLoading: Bool = false
    
    // MARK: - Actor References
    
    private let coordinator: ScoreEditingCoordinator
    private let layoutEngine: LayoutCalculationEngine
    
    // MARK: - Initialization
    
    init(
        document: ScoreDocument,
        coordinator: ScoreEditingCoordinator,
        layoutEngine: LayoutCalculationEngine
    ) {
        self.document = document
        self.coordinator = coordinator
        self.layoutEngine = layoutEngine
    }
    
    // MARK: - User Actions
    
    func addNote(_ note: any Note, to measureId: UUID, at position: Int) async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            try await coordinator.addNote(
                to: measureId,
                instrumentId: note.instrumentId,
                note: note,
                at: position,
                in: document.id
            )
            
            // Fetch updated document
            document = try await coordinator.getDocument(document.id)
            
            // Trigger layout recalculation
            await recalculateLayout()
            
        } catch {
            handleError(error)
        }
    }
    
    func attachEmbellishment(_ embellishment: any Embellishment, to noteId: UUID) async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            try await coordinator.attachEmbellishment(
                embellishment,
                to: noteId,
                in: document.id
            )
            
            document = try await coordinator.getDocument(document.id)
            await recalculateLayout()
            
        } catch {
            handleError(error)
        }
    }
    
    // MARK: - Layout Updates
    
    private func recalculateLayout() async {
        do {
            let layoutResult = try await layoutEngine.calculateDocumentLayout(
                document,
                context: LayoutContext()
            )
            
            // Update UI on MainActor
            // Trigger view refresh
            
        } catch {
            handleError(error)
        }
    }
}
```

---

## 4. UI Implementation with SwiftUI

### 4.1 Score Editor View Hierarchy

**Main Editor View:**

```swift
struct ScoreEditorView: View {
    @State private var viewModel: ScoreEditorViewModel
    
    var body: some View {
        NavigationStack {
            GeometryReader { geometry in
                ScrollView {
                    VStack(spacing: 0) {
                        ForEach(viewModel.document.pages.indices, id: \.self) { pageIndex in
                            PageView(
                                page: viewModel.document.pages[pageIndex],
                                pageSize: geometry.size,
                                viewModel: viewModel
                            )
                            .frame(
                                width: geometry.size.width,
                                height: calculatePageHeight(for: pageIndex, in: geometry.size)
                            )
                        }
                    }
                }
                .overlay {
                    if viewModel.isLoading {
                        ProgressView()
                            .controlSize(.large)
                    }
                }
            }
            .navigationTitle(viewModel.document.metadata.title)
            .toolbar {
                ScoreEditorToolbar(viewModel: viewModel)
            }
        }
    }
}

struct PageView: View {
    let page: Page
    let pageSize: CGSize
    let viewModel: ScoreEditorViewModel
    
    var body: some View {
        Canvas { context, size in
            // Render page using Core Graphics
            renderPage(context: context, size: size)
        }
        .background(Color(.systemBackground))
        .cornerRadius(8)
        .shadow(radius: 4)
    }
    
    private func renderPage(context: GraphicsContext, size: CGSize) {
        // Render staves
        // Render notes
        // Render embellishments
        // Render barlines
    }
}
```

### 4.2 SMuFL Rendering with Core Graphics

**SMuFL Font Manager:**

```swift
@MainActor
final class SMuFLFontManager {
    
    static let shared = SMuFLFontManager()
    
    private let fontName = "Bravura"
    private var fontCache: [CGFloat: CTFont] = [:]
    
    // MARK: - Font Loading
    
    func loadFont(size: CGFloat) -> CTFont {
        if let cached = fontCache[size] {
            return cached
        }
        
        guard let font = CTFontCreateWithName(fontName as CFString, size, nil) else {
            fatalError("Failed to load SMuFL font: \(fontName)")
        }
        
        fontCache[size] = font
        return font
    }
    
    // MARK: - Symbol Rendering
    
    func drawSymbol(
        _ codepoint: Unicode.Scalar,
        at position: CGPoint,
        size: CGFloat,
        in context: CGContext
    ) {
        let font = loadFont(size: size)
        let string = String(codepoint) as CFString
        let attributedString = CFAttributedStringCreate(
            kCFAllocatorDefault,
            string,
            [kCTFontAttributeName: font] as CFDictionary
        )
        
        guard let attrString = attributedString else { return }
        
        let line = CTLineCreateWithAttributedString(attrString)
        
        context.saveGState()
        context.textPosition = position
        CTLineDraw(line, context)
        context.restoreGState()
    }
    
    // MARK: - Glyph Metrics
    
    func getGlyphBounds(_ codepoint: Unicode.Scalar, size: CGFloat) -> CGRect {
        let font = loadFont(size: size)
        let glyph = CTFontGetGlyphWithName(font, String(codepoint) as CFString)
        
        var bounds = CGRect.zero
        CTFontGetBoundingRectsForGlyphs(
            font,
            .horizontal,
            [glyph],
            &bounds,
            1
        )
        
        return bounds
    }
}
```

**Note Rendering:**

```swift
struct NoteRenderer {
    
    let fontManager = SMuFLFontManager.shared
    
    func renderNote(
        _ note: any Note,
        at position: CGPoint,
        staffHeight: CGFloat,
        in context: CGContext
    ) {
        // Render note head
        let noteHeadCodepoint = getSMuFLCodepoint(for: note.noteType)
        fontManager.drawSymbol(
            noteHeadCodepoint,
            at: position,
            size: staffHeight,
            in: context
        )
        
        // Render stem if needed
        if note.noteType.requiresStem {
            renderStem(for: note, at: position, staffHeight: staffHeight, in: context)
        }
        
        // Render flags if needed
        if note.noteType.requiresFlags {
            renderFlags(for: note, at: position, staffHeight: staffHeight, in: context)
        }
        
        // Render embellishment if present
        if let embellishment = note.embellishment {
            renderEmbellishment(
                embellishment,
                for: note,
                at: position,
                staffHeight: staffHeight,
                in: context
            )
        }
    }
    
    private func renderEmbellishment(
        _ embellishment: any Embellishment,
        for note: any Note,
        at position: CGPoint,
        staffHeight: CGFloat,
        in context: CGContext
    ) {
        // Calculate grace note positions
        let gracePositions = embellishment.calculateGraceNotePositions(
            principalNote: note,
            staffHeight: staffHeight
        )
        
        // Render each grace note
        for (index, gracePosition) in gracePositions.enumerated() {
            let graceNoteCodepoint = Unicode.Scalar(0xE560) // Grace note head
            fontManager.drawSymbol(
                graceNoteCodepoint,
                at: CGPoint(
                    x: position.x - gracePosition.offsetX,
                    y: position.y - gracePosition.offsetY
                ),
                size: staffHeight * 0.65, // Grace notes smaller
                in: context
            )
            
            // Render grace note stem with slash
            renderGraceNoteStem(at: gracePosition, staffHeight: staffHeight, in: context)
        }
        
        // Render beaming if needed
        if gracePositions.count > 1 {
            renderGraceNoteBeaming(positions: gracePositions, staffHeight: staffHeight, in: context)
        }
    }
    
    private func getSMuFLCodepoint(for noteType: NoteType) -> Unicode.Scalar {
        switch noteType {
        case .semibreve: return Unicode.Scalar(0xE0A2)!
        case .minim: return Unicode.Scalar(0xE0A3)!
        case .crotchet: return Unicode.Scalar(0xE0A4)!
        case .quaver: return Unicode.Scalar(0xE0A4)!
        case .semiquaver: return Unicode.Scalar(0xE0A4)!
        default: return Unicode.Scalar(0xE0A4)!
        }
    }
}
```

---

## 5. Audio Processing with Actor Isolation

### 5.1 Audio Engine Actor

**Audio Processing Engine:**

```swift
actor AudioProcessingEngine {
    
    private let audioEngine = AVAudioEngine()
    private let playerNode = AVAudioPlayerNode()
    private var isConfigured = false
    
    // MARK: - Setup
    
    func configure() async throws {
        guard !isConfigured else { return }
        
        // Configure audio session
        let session = AVAudioSession.sharedInstance()
        try session.setCategory(.playAndRecord, options: [.defaultToSpeaker, .allowBluetooth])
        try session.setActive(true)
        
        // Attach nodes
        audioEngine.attach(playerNode)
        
        // Connect nodes
        audioEngine.connect(
            playerNode,
            to: audioEngine.mainMixerNode,
            format: audioEngine.outputNode.outputFormat(forBus: 0)
        )
        
        // Start engine
        try audioEngine.start()
        
        isConfigured = true
    }
    
    // MARK: - Metronome
    
    func startMetronome(
        tempo: Int,
        timeSignature: TimeSignature,
        accentPattern: [Bool]
    ) async throws {
        try await configure()
        
        // Generate click sounds
        let clickBuffer = generateClickSound(frequency: 1000, duration: 0.05)
        let accentBuffer = generateClickSound(frequency: 1500, duration: 0.05)
        
        // Schedule clicks
        let interval = 60.0 / Double(tempo)
        var currentTime = AVAudioTime.now()
        
        for (index, isAccent) in accentPattern.enumerated().prefix(100) { // Pre-schedule 100 beats
            let buffer = isAccent ? accentBuffer : clickBuffer
            playerNode.scheduleBuffer(buffer, at: currentTime, options: [], completionHandler: nil)
            currentTime = currentTime.offset(seconds: interval)
        }
        
        playerNode.play()
    }
    
    func stopMetronome() {
        playerNode.stop()
    }
    
    // MARK: - Recording
    
    func startRecording() async throws -> URL {
        try await configure()
        
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        let recordingURL = documentsURL.appendingPathComponent("recording_\(UUID()).m4a")
        
        let settings: [String: Any] = [
            AVFormatIDKey: kAudioFormatMPEG4AAC,
            AVSampleRateKey: 44100.0,
            AVNumberOfChannelsKey: 1,
            AVEncoderAudioQualityKey: AVAudioQuality.high.rawValue
        ]
        
        let recorder = try AVAudioRecorder(url: recordingURL, settings: settings)
        recorder.record()
        
        return recordingURL
    }
    
    // MARK: - Pitch Analysis
    
    func analyzePitch(audioURL: URL) async throws -> [PitchAnalysisResult] {
        // Use vDSP and Accelerate for FFT analysis
        let audioFile = try AVAudioFile(forReading: audioURL)
        let format = audioFile.processingFormat
        let frameCount = UInt32(audioFile.length)
        
        guard let buffer = AVAudioPCMBuffer(pcmFormat: format, frameCapacity: frameCount) else {
            throw AudioError.bufferCreationFailed
        }
        
        try audioFile.read(into: buffer)
        
        // Perform FFT analysis
        let results = await performFFTAnalysis(buffer: buffer)
        
        return results
    }
    
    private func performFFTAnalysis(buffer: AVAudioPCMBuffer) async -> [PitchAnalysisResult] {
        // Use TaskGroup for parallel analysis of audio segments
        await withTaskGroup(of: PitchAnalysisResult.self) { group -> [PitchAnalysisResult] in
            var results: [PitchAnalysisResult] = []
            
            let segmentSize = 4096
            let segments = Int(buffer.frameLength) / segmentSize
            
            for segment in 0..<segments {
                group.addTask {
                    let startFrame = segment * segmentSize
                    let segmentBuffer = self.extractSegment(
                        from: buffer,
                        start: startFrame,
                        length: segmentSize
                    )
                    return self.analyzeSegment(segmentBuffer)
                }
            }
            
            for await result in group {
                results.append(result)
            }
            
            return results
        }
    }
    
    // MARK: - Audio Generation
    
    private func generateClickSound(frequency: Double, duration: Double) -> AVAudioPCMBuffer {
        let sampleRate = 44100.0
        let frameCount = AVAudioFrameCount(sampleRate * duration)
        
        guard let buffer = AVAudioPCMBuffer(
            pcmFormat: AVAudioFormat(
                standardFormatWithSampleRate: sampleRate,
                channels: 1
            )!,
            frameCapacity: frameCount
        ) else {
            fatalError("Failed to create audio buffer")
        }
        
        buffer.frameLength = frameCount
        
        let channelData = buffer.floatChannelData![0]
        for frame in 0..<Int(frameCount) {
            let value = sin(2.0 * .pi * frequency * Double(frame) / sampleRate)
            channelData[frame] = Float(value * 0.5) // 50% volume
        }
        
        return buffer
    }
}
```

### 5.2 Audio Session Management

**Audio Session Coordinator:**

```swift
@MainActor
final class AudioSessionCoordinator: ObservableObject {
    
    private let audioEngine: AudioProcessingEngine
    
    init(audioEngine: AudioProcessingEngine) {
        self.audioEngine = audioEngine
    }
    
    // MARK: - Session Management
    
    func beginAudioSession() async throws {
        try await audioEngine.configure()
    }
    
    func endAudioSession() {
        // Cleanup handled by actor
    }
    
    // MARK: - Interruption Handling
    
    func setupInterruptionHandling() {
        NotificationCenter.default.addObserver(
            forName: AVAudioSession.interruptionNotification,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            self?.handleInterruption(notification)
        }
    }
    
    private func handleInterruption(_ notification: Notification) {
        guard let userInfo = notification.userInfo,
              let typeValue = userInfo[AVAudioSessionInterruptionTypeKey] as? UInt,
              let type = AVAudioSession.InterruptionType(rawValue: typeValue) else {
            return
        }
        
        switch type {
        case .began:
            // Audio interrupted (phone call, Siri, etc.)
            Task {
                await audioEngine.stopMetronome()
            }
            
        case .ended:
            // Interruption ended, optionally resume
            if let optionsValue = userInfo[AVAudioSessionInterruptionOptionKey] as? UInt {
                let options = AVAudioSession.InterruptionOptions(rawValue: optionsValue)
                if options.contains(.shouldResume) {
                    // Resume audio if appropriate
                }
            }
            
        @unknown default:
            break
        }
    }
}
```

---

## 6. App Store Compliance Checklist

### 6.1 Pre-Submission Requirements

**Technical Requirements:**
- ✅ Privacy manifest (PrivacyInfo.xcprivacy) included and accurate
- ✅ All Info.plist privacy descriptions present and clear
- ✅ No private API usage (verified with static analysis)
- ✅ No crashes on fresh install and basic usage
- ✅ All entitlements properly configured in Xcode project
- ✅ Code signing with valid Apple Developer certificate
- ✅ App tested on minimum supported hardware (iPhone 13, M1 Mac)

**Content Requirements:**
- ✅ All screenshots show actual app functionality (no mockups)
- ✅ App description accurate and not misleading
- ✅ Metadata complete in all supported languages
- ✅ Age rating accurately reflects content (4+ recommended)
- ✅ Privacy nutrition label matches privacy manifest exactly
- ✅ No references to other platforms in App Store description

**Functionality Requirements:**
- ✅ All advertised features fully implemented and functional
- ✅ No placeholder content or "coming soon" features
- ✅ Proper error handling with user-friendly messages
- ✅ Acceptable performance on minimum supported hardware
- ✅ Dark Mode support throughout application
- ✅ VoiceOver support for all interactive elements

### 6.2 App Store Connect Configuration

**App Information:**
```
Configuration Checklist:
├── Name: "Pipe Band Score Editor" (example - finalize with team)
├── Subtitle: "Scottish Music Notation" (30 characters max)
├── Primary Category: Music
├── Secondary Category: Education
├── Age Rating: 4+ (no objectionable content)
├── Copyright: © 2025 Your Company Name
└── SKU: Unique identifier for internal tracking
```

**Pricing and Availability:**
```
Distribution Settings:
├── Price: Free (initial release - IAP possible in future)
├── Availability: All territories (or select specific regions)
├── Pre-order: Optional (consider for launch buzz)
└── Educational Discount: Consider for institutional users
```

**App Privacy:**
```
Privacy Configuration (Must Match Privacy Manifest):
├── Privacy Policy URL: [Required - must host accessible privacy policy]
├── Data Collection:
│   ├── Musical Scores: Linked to user (iCloud mode), App Functionality purpose
│   ├── Audio Recordings: Linked to user (iCloud mode), App Functionality purpose
│   ├── User ID: Linked to user (iCloud account), App Functionality purpose
│   └── Device ID: Linked to user (CloudKit), App Functionality purpose
├── Data Tracking: No
└── Third-Party SDKs: None (or disclose if any added)
```

**App Review Information:**
```
Review Team Guidance:
├── Contact Email: [technical contact for urgent issues]
├── Contact Phone: [for urgent review issues]
├── Demo Account: Not required (app functional without account)
├── Notes for Reviewer:
│   "iCloud collaboration features require multiple test accounts to
│   demonstrate real-time sync. App remains fully functional without
│   iCloud enabled. File Provider mode provides alternative cloud storage."
└── Attachments: Optional demo video showing core features
```

**Marketing Assets:**
```
Required Screenshots (per device size):
├── iPhone 6.7" (Pro Max): 1290 x 2796 or 2796 x 1290
├── iPhone 6.5" (11 Pro Max, XS Max): 1242 x 2688 or 2688 x 1242
├── iPad Pro 12.9": 2048 x 2732 or 2732 x 2048
└── Mac: 2880 x 1800 or 1600 x 1200 minimum (if Mac Catalyst)

App Preview Videos (Optional but Recommended):
├── Duration: 15-30 seconds
├── Content: Actual app footage only
├── Formats: Same device sizes as screenshots
└── Limit: Up to 3 previews per device size

App Icon Requirements:
├── 1024x1024 App Store icon (no transparency, no alpha)
├── All platform-specific sizes in asset catalog
└── Consistent design across all sizes
```

### 6.3 Build Configuration

**Xcode Project Settings:**

```swift
// Build Settings Configuration
Build Configuration:
├── Swift Language Version: Swift 6
├── Deployment Targets:
│   ├── iOS: 17.0
│   └── macOS: 14.0
├── Architectures:
│   ├── iOS: arm64
│   └── macOS: arm64, x86_64 (if supporting Intel Macs)
├── Code Signing:
│   ├── Automatic Signing: Enabled (recommended)
│   ├── Team: [Your Apple Developer Team]
│   └── Provisioning Profile: Automatic
├── Swift Compiler:
│   ├── Optimization Level: -O (Release), -Onone (Debug)
│   ├── Compilation Mode: Whole Module (Release)
│   └── Strict Concurrency Checking: Complete
└── Linker:
    ├── Dead Code Stripping: Yes (Release)
    └── Strip Debug Symbols: Yes (Release)
```

**Entitlements File (Pipe Band App.entitlements):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- iCloud Container -->
    <key>com.apple.developer.icloud-container-identifiers</key>
    <array>
        <string>iCloud.com.yourcompany.pipebandapp</string>
    </array>
    
    <!-- iCloud Services -->
    <key>com.apple.developer.icloud-services</key>
    <array>
        <string>CloudKit</string>
        <string>CloudDocuments</string>
    </array>
    
    <!-- File Provider -->
    <key>com.apple.developer.ubiquity-container-identifiers</key>
    <array>
        <string>iCloud.com.yourcompany.pipebandapp</string>
    </array>
    
    <!-- Background Modes -->
    <key>com.apple.developer.background-modes</key>
    <array>
        <string>audio</string>
        <string>fetch</string>
        <string>processing</string>
    </array>
    
    <!-- App Groups (for shared data between iOS/Mac) -->
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.yourcompany.pipebandapp</string>
    </array>
</dict>
</plist>
```

### 6.4 TestFlight Distribution

**Beta Testing Strategy:**

```
TestFlight Workflow:
├── Internal Testing (Before External):
│   ├── Add up to 100 internal testers
│   ├── No beta review required
│   ├── Immediate availability after upload
│   └── Iterate quickly on critical bugs
├── External Testing:
│   ├── Submit for beta app review (24-48 hours typically)
│   ├── Add up to 10,000 external testers
│   ├── Public link or email invitations
│   └── Collect feedback through TestFlight app
├── Build Management:
│   ├── Builds expire after 90 days
│   ├── Upload new builds for continued testing
│   └── Manage tester groups for targeted testing
└── Feedback Collection:
    ├── Review crash reports in App Store Connect
    ├── Read tester feedback and screenshots
    ├── Prioritize issues before App Store submission
    └── Iterate based on real-world usage patterns
```

**TestFlight Best Practices:**
```
Testing Phases:
├── Phase 1: Internal Team (Week 1-2)
│   ├── Core functionality verification
│   ├── Critical bug identification
│   ├── Performance testing on various devices
│   └── iCloud sync verification across devices
├── Phase 2: Limited External (Week 3-4)
│   ├── Invite 20-50 band members
│   ├── Real-world score creation testing
│   ├── Collaboration feature validation
│   └── Audio recording quality assessment
├── Phase 3: Expanded External (Week 5-6)
│   ├── Invite 200-500 musicians
│   ├── Stress test cloud synchronization
│   ├── Cross-platform compatibility (iOS/Mac)
│   └── Gather feature requests and UX feedback
└── Phase 4: Public Beta (Week 7-8)
    ├── Public TestFlight link for wider audience
    ├── Monitor crash-free rate (target: 99.5%+)
    ├── Final polish based on feedback
    └── Prepare for App Store submission
```

---

## 7. Performance Optimization

### 7.1 Layout Performance Strategies

**Incremental Layout Updates:**

```swift
actor IncrementalLayoutEngine {
    
    private var dirtyRegions: Set<UUID> = []
    private var layoutCache: LayoutCache
    
    // MARK: - Incremental Recalculation
    
    func markDirty(_ entityIds: [UUID]) {
        dirtyRegions.formUnion(entityIds)
    }
    
    func recalculateIfNeeded(
        document: ScoreDocument,
        context: LayoutContext
    ) async throws -> LayoutUpdateResult {
        guard !dirtyRegions.isEmpty else {
            return LayoutUpdateResult(updatedRegions: [])
        }
        
        // Determine scope of recalculation
        let scope = determineScopeFromDirtyRegions(dirtyRegions, in: document)
        
        var updatedRegions: [UUID: LayoutResult] = [:]
        
        switch scope {
        case .measure(let measureIds):
            // Recalculate only affected measures
            for measureId in measureIds {
                if let measure = document.findMeasure(measureId) {
                    let layout = try await calculateMeasureLayout(measure, context: context)
                    updatedRegions[measureId] = .measure(layout)
                }
            }
            
        case .system(let systemIds):
            // Recalculate affected systems
            for systemId in systemIds {
                if let system = document.findSystem(systemId) {
                    let layout = try await calculateSystemLayout(system, context: context)
                    updatedRegions[systemId] = .system(layout)
                }
            }
            
        case .page(let pageIds):
            // Recalculate affected pages
            for pageId in pageIds {
                if let page = document.findPage(pageId) {
                    let layout = try await calculatePageLayout(page, context: context)
                    updatedRegions[pageId] = .page(layout)
                }
            }
            
        case .document:
            // Full document recalculation (rare - paper size change, etc.)
            let layout = try await calculateDocumentLayout(document, context: context)
            updatedRegions[document.id] = .document(layout)
        }
        
        // Clear dirty regions
        dirtyRegions.removeAll()
        
        return LayoutUpdateResult(updatedRegions: updatedRegions)
    }
}
```

**Layout Caching Strategy:**

```swift
actor LayoutCache {
    
    private var measureLayouts: [UUID: CachedMeasureLayout] = [:]
    private var systemLayouts: [UUID: CachedSystemLayout] = [:]
    private var pageLayouts: [UUID: CachedPageLayout] = [:]
    
    private let maxCacheSize: Int = 1000
    private var accessOrder: [UUID] = [] // LRU tracking
    
    // MARK: - Cache Operations
    
    func getMeasureLayout(_ id: UUID) -> MeasureLayout? {
        guard let cached = measureLayouts[id], !cached.isStale else {
            return nil
        }
        
        // Update LRU
        updateAccessOrder(id)
        
        return cached.layout
    }
    
    func cacheMeasureLayout(_ layout: MeasureLayout, for id: UUID) {
        // Check cache size
        if measureLayouts.count >= maxCacheSize {
            evictLRU()
        }
        
        measureLayouts[id] = CachedMeasureLayout(
            layout: layout,
            timestamp: Date()
        )
        
        updateAccessOrder(id)
    }
    
    func invalidate(_ ids: [UUID]) {
        for id in ids {
            measureLayouts.removeValue(forKey: id)
            systemLayouts.removeValue(forKey: id)
            pageLayouts.removeValue(forKey: id)
            accessOrder.removeAll { $0 == id }
        }
    }
    
    func invalidateAll() {
        measureLayouts.removeAll()
        systemLayouts.removeAll()
        pageLayouts.removeAll()
        accessOrder.removeAll()
    }
    
    // MARK: - LRU Management
    
    private func updateAccessOrder(_ id: UUID) {
        accessOrder.removeAll { $0 == id }
        accessOrder.append(id)
    }
    
    private func evictLRU() {
        guard let oldest = accessOrder.first else { return }
        
        measureLayouts.removeValue(forKey: oldest)
        systemLayouts.removeValue(forKey: oldest)
        pageLayouts.removeValue(forKey: oldest)
        accessOrder.removeFirst()
    }
    
    // MARK: - Cache Statistics
    
    func getStatistics() -> CacheStatistics {
        CacheStatistics(
            measureCacheSize: measureLayouts.count,
            systemCacheSize: systemLayouts.count,
            pageCacheSize: pageLayouts.count,
            totalCacheSize: measureLayouts.count + systemLayouts.count + pageLayouts.count
        )
    }
}

struct CachedMeasureLayout {
    let layout: MeasureLayout
    let timestamp: Date
    
    var isStale: Bool {
        Date().timeIntervalSince(timestamp) > 300 // 5 minutes
    }
}
```

### 7.2 Memory Management

**Large Document Handling:**

```swift
actor LargeDocumentManager {
    
    private let viewportSize: Int = 5 // Pages visible + buffer
    private var loadedPages: Set<Int> = []
    
    // MARK: - Lazy Loading
    
    func loadPagesForViewport(
        currentPage: Int,
        document: ScoreDocument
    ) async throws -> [Int: PageLayout] {
        let startPage = max(0, currentPage - 2)
        let endPage = min(document.pages.count - 1, currentPage + 2)
        
        var layouts: [Int: PageLayout] = [:]
        
        // Load visible pages
        for pageIndex in startPage...endPage {
            if !loadedPages.contains(pageIndex) {
                let layout = try await loadPage(pageIndex, from: document)
                layouts[pageIndex] = layout
                loadedPages.insert(pageIndex)
            }
        }
        
        // Unload far-away pages
        let pagesToUnload = loadedPages.filter { page in
            page < startPage - 2 || page > endPage + 2
        }
        
        for page in pagesToUnload {
            unloadPage(page)
            loadedPages.remove(page)
        }
        
        return layouts
    }
    
    private func loadPage(_ index: Int, from document: ScoreDocument) async throws -> PageLayout {
        // Load page content
        // Calculate layout
        // Return result
        PageLayout(pageIndex: index, content: [])
    }
    
    private func unloadPage(_ index: Int) {
        // Release memory for page
    }
}
```

**Memory Pressure Handling:**

```swift
@MainActor
final class MemoryPressureMonitor: ObservableObject {
    
    private let layoutCache: LayoutCache
    private let documentManager: LargeDocumentManager
    
    init(layoutCache: LayoutCache, documentManager: LargeDocumentManager) {
        self.layoutCache = layoutCache
        self.documentManager = documentManager
        
        setupMemoryWarningObserver()
    }
    
    private func setupMemoryWarningObserver() {
        NotificationCenter.default.addObserver(
            forName: UIApplication.didReceiveMemoryWarningNotification,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleMemoryWarning()
        }
    }
    
    private func handleMemoryWarning() {
        Task {
            // Clear layout cache
            await layoutCache.invalidateAll()
            
            // Unload non-visible pages
            // Keep only current viewport
            
            // Log memory pressure event
            print("Memory warning received - cleared caches")
        }
    }
}
```

### 7.3 Rendering Performance

**Metal-Accelerated Rendering (Optional):**

```swift
import MetalKit

@MainActor
final class MetalScoreRenderer {
    
    private let device: MTLDevice
    private let commandQueue: MTLCommandQueue
    private let pipelineState: MTLRenderPipelineState
    
    init() throws {
        guard let device = MTLCreateSystemDefaultDevice() else {
            throw RenderError.metalUnavailable
        }
        
        self.device = device
        
        guard let commandQueue = device.makeCommandQueue() else {
            throw RenderError.commandQueueCreationFailed
        }
        
        self.commandQueue = commandQueue
        
        // Create pipeline state
        let library = device.makeDefaultLibrary()
        let vertexFunction = library?.makeFunction(name: "vertexShader")
        let fragmentFunction = library?.makeFunction(name: "fragmentShader")
        
        let pipelineDescriptor = MTLRenderPipelineDescriptor()
        pipelineDescriptor.vertexFunction = vertexFunction
        pipelineDescriptor.fragmentFunction = fragmentFunction
        pipelineDescriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        
        self.pipelineState = try device.makeRenderPipelineState(descriptor: pipelineDescriptor)
    }
    
    // MARK: - Rendering
    
    func render(score: ScoreDocument, in view: MTKView) throws {
        guard let commandBuffer = commandQueue.makeCommandBuffer(),
              let renderPassDescriptor = view.currentRenderPassDescriptor,
              let renderEncoder = commandBuffer.makeRenderCommandEncoder(descriptor: renderPassDescriptor) else {
            throw RenderError.renderSetupFailed
        }
        
        renderEncoder.setRenderPipelineState(pipelineState)
        
        // Render score using Metal
        // Much faster for complex scores with many notes
        
        renderEncoder.endEncoding()
        
        if let drawable = view.currentDrawable {
            commandBuffer.present(drawable)
        }
        
        commandBuffer.commit()
    }
}
```

---

## 8. Development Workflow

### 8.1 Xcode Project Structure

**Swift Package Organization:**

```
PipeBandApp.xcworkspace/
├── PipeBandApp (iOS)
│   ├── App/
│   │   ├── PipeBandAppApp.swift
│   │   ├── ContentView.swift
│   │   └── Info.plist
│   ├── Views/
│   │   ├── ScoreEditor/
│   │   ├── DocumentBrowser/
│   │   └── Settings/
│   ├── ViewModels/
│   │   ├── ScoreEditorViewModel.swift
│   │   └── DocumentBrowserViewModel.swift
│   ├── Resources/
│   │   ├── Assets.xcassets
│   │   ├── Bravura.otf
│   │   └── Localizable.strings
│   └── Entitlements/
│       └── PipeBandApp.entitlements
├── PipeBandApp (macOS)
│   ├── App/
│   ├── Views/
│   └── Entitlements/
├── Packages/
│   ├── PBDomain/
│   │   ├── Package.swift
│   │   └── Sources/
│   │       └── PBDomain/
│   │           ├── Entities/
│   │           │   ├── Score.swift
│   │           │   ├── Tune.swift
│   │           │   ├── Note.swift
│   │           │   └── Embellishment.swift
│   │           ├── UseCases/
│   │           │   ├── ScoreEditingCoordinator.swift
│   │           │   └── LayoutCalculationEngine.swift
│   │           ├── Repositories/
│   │           │   └── DocumentRepository.swift
│   │           └── Services/
│   │               └── ValidationService.swift
│   ├── PBDocument/
│   │   ├── Package.swift
│   │   └── Sources/
│   │       └── PBDocument/
│   │           ├── PipeBandScoreDocument.swift
│   │           ├── DocumentSerializer.swift
│   │           └── DocumentParser.swift
│   ├── PBCloudKit/
│   │   ├── Package.swift
│   │   └── Sources/
│   │       └── PBCloudKit/
│   │           ├── CloudKitCollaborationManager.swift
│   │           └── CloudKitSyncCoordinator.swift
│   ├── PBAudio/
│   │   ├── Package.swift
│   │   └── Sources/
│   │       └── PBAudio/
│   │           ├── AudioProcessingEngine.swift
│   │           └── AudioAnalysisService.swift
│   └── PBLayout/
│       ├── Package.swift
│       └── Sources/
│           └── PBLayout/
│               ├── LayoutCalculationEngine.swift
│               ├── PaginationService.swift
│               └── LayoutCache.swift
└── Tests/
    ├── PBDomainTests/
    ├── PBDocumentTests/
    ├── PBLayoutTests/
    └── UITests/
```

**Package.swift Example (PBDomain):**

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "PBDomain",
    platforms: [
        .iOS(.v17),
        .macOS(.v14)
    ],
    products: [
        .library(
            name: "PBDomain",
            targets: ["PBDomain"]
        )
    ],
    dependencies: [],
    targets: [
        .target(
            name: "PBDomain",
            dependencies: [],
            swiftSettings: [
                .enableExperimentalFeature("StrictConcurrency")
            ]
        ),
        .testTarget(
            name: "PBDomainTests",
            dependencies: ["PBDomain"]
        )
    ]
)
```

### 8.2 Build Configurations

**Debug vs Release Settings:**

```swift
// Debug Configuration
#if DEBUG
let isDebugMode = true
let enableVerboseLogging = true
let layoutDebugVisualization = true
let cacheStatisticsEnabled = true
#else
let isDebugMode = false
let enableVerboseLogging = false
let layoutDebugVisualization = false
let cacheStatisticsEnabled = false
#endif
```

**Compiler Flags:**

```
Debug Build Settings:
├── Swift Optimization Level: -Onone
├── Swift Compilation Mode: Incremental
├── Debug Information Format: DWARF with dSYM
├── Enable Testability: YES
└── Strict Concurrency Checking: Complete

Release Build Settings:
├── Swift Optimization Level: -O
├── Swift Compilation Mode: Whole Module
├── Debug Information Format: DWARF with dSYM
├── Enable Testability: NO
├── Strict Concurrency Checking: Complete
├── Dead Code Stripping: YES
└── Strip Debug Symbols: YES
```

### 8.3 Testing Strategy

**Unit Tests:**

```swift
import XCTest
@testable import PBDomain

final class ScoreEditingCoordinatorTests: XCTestCase {
    
    var coordinator: ScoreEditingCoordinator!
    var mockRepository: MockDocumentRepository!
    
    override func setUp() async throws {
        mockRepository = MockDocumentRepository()
        coordinator = ScoreEditingCoordinator(repository: mockRepository)
    }
    
    func testCreateScore() async throws {
        // Given
        let title = "Test Score"
        let composer = "Test Composer"
        let instruments: [InstrumentType] = [.bagpipes, .snareDrum]
        
        // When
        let document = try await coordinator.createScore(
            title: title,
            composer: composer,
            defaultInstruments: instruments,
            layoutSettings: nil
        )
        
        // Then
        XCTAssertEqual(document.metadata.title, title)
        XCTAssertEqual(document.metadata.composer, composer)
        XCTAssertEqual(document.tunes.count, 1)
        XCTAssertEqual(document.tunes[0].parts.count, 1)
        XCTAssertEqual(document.tunes[0].parts[0].partLetter, "A")
    }
    
    func testAddNoteValidatesDuration() async throws {
        // Given
        let document = try await coordinator.createScore(
            title: "Test",
            composer: nil,
            defaultInstruments: [.bagpipes],
            layoutSettings: nil
        )
        
        let measureId = document.tunes[0].parts[0].systems[0].measures[0].id
        
        // When/Then - should throw when total duration exceeds time signature
        let note1 = PipeNote(noteType: .minim, pitch: Pitch(pitchClass: .d, octave: 4))
        let note2 = PipeNote(noteType: .minim, pitch: Pitch(pitchClass: .e, octave: 4))
        let note3 = PipeNote(noteType: .minim, pitch: Pitch(pitchClass: .f, octave: 4))
        
        try await coordinator.addNote(to: measureId, instrumentId: .bagpipes, note: note1, at: 0, in: document.id)
        try await coordinator.addNote(to: measureId, instrumentId: .bagpipes, note: note2, at: 1, in: document.id)
        
        // This should throw - exceeds 4/4 time signature
        await XCTAssertThrowsError(
            try await coordinator.addNote(to: measureId, instrumentId: .bagpipes, note: note3, at: 2, in: document.id)
        )
    }
    
    func testEmbellishmentValidation() async throws {
        // Test prior note constraints
        // Test embellishment compatibility
        // Test grace note calculations
    }
}
```

**UI Tests:**

```swift
import XCTest

final class ScoreEditorUITests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }
    
    func testCreateNewScore() {
        // Tap create button
        app.buttons["Create Score"].tap()
        
        // Enter title
        let titleField = app.textFields["Score Title"]
        titleField.tap()
        titleField.typeText("My First Score")
        
        // Select instrument
        app.buttons["Add Bagpipes"].tap()
        
        // Create
        app.buttons["Create"].tap()
        
        // Verify editor appears
        XCTAssertTrue(app.navigationBars["My First Score"].exists)
    }
    
    func testAddNote() {
        // Create score first
        testCreateNewScore()
        
        // Select note tool
        app.buttons["Quarter Note"].tap()
        
        // Tap on staff to add note
        let staff = app.otherElements["Staff"]
        staff.tap()
        
        // Verify note appears
        XCTAssertTrue(app.otherElements["Quarter Note"].exists)
    }
}
```

### 8.4 Continuous Integration

**GitHub Actions Workflow:**

```yaml
name: iOS CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-and-test:
    runs-on: macos-14
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Select Xcode
      run: sudo xcode-select -s /Applications/Xcode_15.2.app
    
    - name: Build
      run: |
        xcodebuild build \
          -workspace PipeBandApp.xcworkspace \
          -scheme PipeBandApp \
          -destination 'platform=iOS Simulator,name=iPhone 15 Pro,OS=17.2'
    
    - name: Run Tests
      run: |
        xcodebuild test \
          -workspace PipeBandApp.xcworkspace \
          -scheme PipeBandApp \
          -destination 'platform=iOS Simulator,name=iPhone 15 Pro,OS=17.2' \
          -enableCodeCoverage YES
    
    - name: Upload Coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage.xml
```

---

## 9. Post-Launch Maintenance

### 9.1 Crash Monitoring

**Xcode Organizer Integration:**

```swift
// Crash reporting is automatic via Xcode Organizer
// No third-party SDK needed for App Store builds

// Optional: Custom crash context
enum CrashContext {
    static func recordBreadcrumb(_ message: String) {
        #if DEBUG
        print("🍞 Breadcrumb: \(message)")
        #endif
        // Production: crashes automatically include recent logs
    }
}
```

### 9.2 Performance Monitoring

**MetricKit Integration:**

```swift
import MetricKit

@MainActor
final class PerformanceMonitor: NSObject, MXMetricManagerSubscriber {
    
    override init() {
        super.init()
        MXMetricManager.shared.add(self)
    }
    
    deinit {
        MXMetricManager.shared.remove(self)
    }
    
    // MARK: - MXMetricManagerSubscriber
    
    func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            // CPU metrics
            if let cpuMetrics = payload.cpuMetrics {
                print("CPU Time: \(cpuMetrics.cumulativeCPUTime)")
            }
            
            // Memory metrics
            if let memoryMetrics = payload.memoryMetrics {
                print("Peak Memory: \(memoryMetrics.peakMemoryUsage)")
            }
            
            // App launch metrics
            if let launchMetrics = payload.applicationLaunchMetrics {
                print("Launch Time: \(launchMetrics.histogrammedTimeToFirstDraw)")
            }
        }
    }
    
    func didReceive(_ payloads: [MXDiagnosticPayload]) {
        for payload in payloads {
            // Crash diagnostics
            if let crashDiagnostics = payload.crashDiagnostics {
                for diagnostic in crashDiagnostics {
                    print("Crash: \(diagnostic.exceptionType ?? "Unknown")")
                }
            }
            
            // Hang diagnostics
            if let hangDiagnostics = payload.hangDiagnostics {
                for diagnostic in hangDiagnostics {
                    print("Hang duration: \(diagnostic.hangDuration)")
                }
            }
        }
    }
}
```

### 9.3 Update Strategy

**Version Management:**

```swift
struct AppVersion {
    static let current = "1.0.0"
    static let build = "1"
    
    static func checkForUpdates() async -> UpdateInfo? {
        // Check App Store API for newer version
        // Prompt user to update if available
        return nil
    }
}
```

---

## 10. Platform-Specific Considerations

### 10.1 Mac Catalyst Optimizations

**Platform-Specific UI Adaptations:**

```swift
#if targetEnvironment(macCatalyst)
extension ScoreEditorView {
    func configureMacInterface() {
        // Add Mac-specific toolbar
        // Configure window chrome
        // Setup keyboard shortcuts
    }
}
#endif
```

### 10.2 iPad Multitasking

**Split View Support:**

```swift
struct ScoreEditorView: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    
    var body: some View {
        Group {
            if horizontalSizeClass == .compact {
                // Compact layout for split view
                CompactScoreEditor()
            } else {
                // Full layout
                RegularScoreEditor()
            }
        }
    }
}
```

### 10.3 Apple Pencil Integration

**Pencil Input Handling:**

```swift
struct ScorePencilModifier: ViewModifier {
    @State private var currentStroke: [CGPoint] = []
    
    func body(content: Content) -> some View {
        content
            .gesture(
                DragGesture(minimumDistance: 0)
                    .onChanged { value in
                        currentStroke.append(value.location)
                    }
                    .onEnded { _ in
                        processStroke(currentStroke)
                        currentStroke.removeAll()
                    }
            )
    }
    
    private func processStroke(_ points: [CGPoint]) {
        // Interpret pencil stroke as musical input
        // Could add notes, embellishments, etc.
    }
}
```

---

## 11. Summary and Best Practices

### 11.1 Key Architectural Decisions

**Swift 6 Concurrency:**
- Complete data race safety at compile time
- Actor isolation for all domain operations
- MainActor coordination for UI updates
- Structured concurrency throughout

**Storage Strategy:**
- iCloud Container mode as primary (App Store advantage)
- File Provider mode for advanced users
- Document-based architecture (UIDocument/NSDocument)
- Proper file coordination for cloud safety

**Performance:**
- Incremental layout recalculation
- LRU caching with actor isolation
- Lazy loading for large documents
- Metal acceleration for complex scores (optional)

**App Store Compliance:**
- Privacy manifest configured correctly
- All entitlements properly documented
- No prohibited technologies
- Contextual permission requests
- Clear user benefits communicated

### 11.2 Development Priorities

**Phase 1 (MVP):**
1. Document-based architecture with iCloud
2. Basic score editing (notes, measures, parts)
3. SMuFL rendering with Core Graphics
4. Simple embellishment support
5. PDF export

**Phase 2 (Enhancement):**
1. Real-time collaboration via CloudKit
2. Audio recording and playback
3. Metronome with time signature support
4. Complete embellishment system
5. File Provider mode

**Phase 3 (Polish):**
1. Advanced layout optimization
2. Metal-accelerated rendering
3. Apple Pencil integration
4. Widgets and shortcuts
5. macOS-specific features

### 11.3 Maintenance Checklist

**Regular Tasks:**
- [ ] Monitor crash reports weekly via Xcode Organizer
- [ ] Review App Store ratings and respond to feedback
- [ ] Update for new iOS/macOS releases (annually)
- [ ] Test on new device models
- [ ] Update privacy manifest if data collection changes
- [ ] Refresh marketing screenshots for new devices
- [ ] Monitor performance metrics via MetricKit
- [ ] Update dependencies (Swift packages)
- [ ] Run security audits quarterly
- [ ] Backup CloudKit schemas and data

---

**End of iOS/macOS Implementation Specification**

This specification provides comprehensive guidance for implementing the Pipe Band Score application on Apple platforms with full App Store compliance. All implementations must adhere to the domain architecture defined in `pipe_band_app_architecture.md` while leveraging platform-native capabilities for optimal user experience.