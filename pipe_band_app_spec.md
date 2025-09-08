# Scottish Pipe Band Instrument App - Clean Architecture Specification

## 1. Executive Summary

This specification outlines a cross-platform application for Scottish pipe band musicians, built using **platform-native languages** and **Clean Architecture** principles. The app targets bagpipes, bass drums, tenor drums, and snare drums with offline-first functionality and SMuFL-compliant music notation rendering across macOS, iOS, Android, Windows, and Linux platforms.

## 2. Architecture Overview

## 2.1 Clean Architecture Layers

### Presentation Layer
- UI Components & Rendering
- Layout Calculation Services  
- Font Metrics & Text Rendering
- Platform Graphics APIs
- User Interaction Handling

### Domain Layer
- Musical Entities & Rules
- Layout Intent & Policies
- Business Logic Use Cases
- Repository Interfaces

### Data Layer
- Score Persistence
- Cloud Storage Integration
- Audio File Management
- Repository Implementations

### 2.2 Technology Stack by Platform

- **Language**: Swift 6.0+ with complete concurrency enabled and strict data isolation
- **UI Framework**: SwiftUI with UIKit/AppKit bridging where necessary
- **Architecture**: MVVM with Swift Concurrency (async/await) and Observation framework
- **Audio Processing**: AVFoundation with AVAudioEngine, optional AudioKit integration
- **Local Storage**: Core Data with CloudKit integration, SQLite backing store
- **Music Notation**: Core Graphics with Text Kit 2, SMuFL font rendering via CTFont
- **Cloud Integration**: Document Provider framework, CloudKit for app data sync
- **Platform Optimization**: Mac Catalyst for shared codebase with platform-specific adaptations

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

## 3.1 Core Entities

### 3.1.1 Instrument Hierarchy

The domain layer defines the business logic in a platform-independent manner. Each platform implements these concepts using their native languages while maintaining identical behavior.

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

**Enhanced Instrument Capabilities:**
```
InstrumentCapabilities
├── TuningSystem (available tunings, default tuning)
├── ClefAssignments (primary clef, acceptable alternatives)  
├── NotationRules (ornament availability, special symbols)
├── AudioCharacteristics (frequency range, harmonics)
└── LayoutRequirements (spacing needs, multi-staff requirements)
```

### 3.1.2 Score Structure with Layout Integration

```
ScoreDocument
├── Metadata (title, composer, version, sync status, default orientation, paper size)
├── Pages[] (physical page layout assignments)
│   └── TuneLines[] (lines assigned to this page)
├── DocumentLayoutSettings (global layout preferences)
└── Tunes[] (1 or more tunes)
    ├── Tune Metadata (title, type, tempo, key, orientation override)
    ├── TuneLayoutPreference (page break policy, spacing preferences)
    └── TuneLines[] (sequential lines that make up the tune)
        ├── TextLine (title, composer, headers, footers)
        └── Part (A, B, C, Intro, Outro, etc.)
            ├── Part Metadata (name, letter, play order, repeat count)
            └── MusicalSystems[] (staff systems with measures)
                ├── SystemStartElements[] (per instrument)
                │   ├── Clef (for this instrument at system start)
                │   └── Accidentals[] (sharp signs for bagpipe notation)
                ├── SystemLayoutHints (spacing, break preferences)
                ├── Measures[] (synchronized across all instruments)
                │   ├── InstrumentMeasures[] (per instrument)
                │   │   └── MusicalElements[] (notes, rests, ornaments with layout)
                │   ├── MeasureLayoutHints (width, compression, break hints)
                │   └── TimeSignature (cascading)
                └── Barlines[] (system-wide, spanning all instruments)
```

### 3.1.3 Musical Elements with Embedded Layout

#### Enhanced Musical Element Hierarchy

**Note Entity with Layout Integration:**
```
Note
├── Musical Properties (Domain Core)
│   ├── pitch: Pitch
│   ├── duration: Duration
│   ├── ornaments: [Ornament]
│   ├── accidental: Accidental?
│   ├── tie: TieType?
│   └── articulation: [Articulation]
├── Layout Properties (Domain Embedded)
│   ├── position: CGPoint? (nil = auto-calculated)
│   ├── spacingHints: SpacingHints? (nil = use defaults)
│   ├── visualStyle: VisualStyle? (nil = use theme)
│   └── collisionAvoidance: CollisionHints?
└── Domain Methods
    ├── withPosition(CGPoint) -> Note
    ├── withSpacing(leading: CGFloat, trailing: CGFloat) -> Note
    ├── withVisualStyle(VisualStyle) -> Note
    └── invalidateLayout() -> Void
```

**Rest Entity:**
```
Rest
├── Musical Properties
│   ├── duration: Duration
│   ├── isVisible: Bool
│   └── type: RestType
├── Layout Properties
│   ├── position: CGPoint?
│   ├── verticalOffset: CGFloat?
│   └── size: CGFloat?
└── Domain Methods
    ├── withPosition(CGPoint) -> Rest
    └── withVerticalOffset(CGFloat) -> Rest
```

**Ornament Entity:**
```
Ornament
├── Musical Properties
│   ├── type: OrnamentType
│   ├── targetNote: NoteID
│   ├── graceNotePattern: [GraceNote]
│   ├── timing: OrnamentTiming
│   └── styleVariation: OrnamentStyle
├── Layout Properties
│   ├── position: CGPoint?
│   ├── symbolSpacing: CGFloat?
│   ├── verticalOffset: CGFloat?
│   └── size: CGFloat?
└── Domain Methods
    ├── withPosition(CGPoint) -> Ornament
    ├── withSymbolSpacing(CGFloat) -> Ornament
    └── getGraceNotePositions() -> [CGPoint]
```

#### Enhanced Measure Entity

```
Measure
├── Musical Properties (Domain Core)
│   ├── id: UUID
│   ├── timeSignature: TimeSignature?
│   ├── notes: [Note]
│   ├── rests: [Rest]
│   ├── ornaments: [Ornament]
│   ├── keySignature: KeySignature?
│   └── tempo: TempoMarking?
├── Layout Properties (Domain Embedded)
│   ├── width: CGFloat? (nil = auto-calculated)
│   ├── compressionFactor: Float? (nil = normal)
│   ├── breakHint: BreakHint? (nil = auto)
│   ├── minimumWidth: CGFloat?
│   └── spacingMultiplier: Float?
├── Barline Properties
│   ├── openingBarline: BarlineType
│   ├── closingBarline: BarlineType
│   ├── repeatDots: RepeatDotConfiguration?
│   └── endingBracket: EndingBracket?
└── Domain Methods
    ├── addNote(Note) -> Void
    ├── addRest(Rest) -> Void
    ├── addOrnament(Ornament) -> Void
    ├── setTimeSignature(TimeSignature) -> Void
    ├── setWidth(CGFloat) -> Void
    ├── applyCompressionFactor(Float) -> Void
    ├── setBreakHint(BreakHint) -> Void
    └── getEffectiveTimeSignature() -> TimeSignature
```

#### Enhanced MusicalSystem Entity

```
MusicalSystem
├── Musical Properties (Domain Core)
│   ├── id: UUID
│   ├── instruments: [InstrumentID]
│   ├── measures: [Measure]
│   ├── systemStartElements: [InstrumentID: SystemStartElement]
│   └── keySignature: KeySignature?
├── Layout Properties (Domain Embedded)
│   ├── staffSpacing: CGFloat? (nil = auto-calculated)
│   ├── systemBreakBefore: Bool (default: false)
│   ├── systemBreakAfter: Bool (default: false)
│   ├── alignmentHints: AlignmentHints? (nil = auto)
│   ├── verticalCompression: Float? (nil = normal)
│   └── instrumentSpacing: [InstrumentID: CGFloat]?
└── Domain Methods
    ├── addMeasure(Measure) -> Void
    ├── forceSystemBreak() -> Void
    ├── setStaffSpacing(CGFloat) -> Void
    ├── setInstrumentSpacing(InstrumentID, CGFloat) -> Void
    ├── alignMeasures() -> Void
    └── validateInstrumentSynchronization() -> [ValidationError]
```

### 3.1.4 Time Signature and Musical Intelligence

**Time Signature Cascading Logic:**
1. Check if current measure specifies time signature
2. If not, search backwards through previous measures in same system
3. If none found in system, search previous systems in same part
4. If none found in part, use tune's default time signature
5. Cache results for performance optimization

**Enhanced Time Signature Entity:**
```
TimeSignature
├── Musical Properties
│   ├── numerator: Int
│   ├── denominator: Int
│   ├── beatGrouping: [Int] (for complex meters)
│   ├── metronomeMarking: MetronomeMarking?
│   └── displayStyle: TimeSignatureDisplay
├── Layout Properties
│   ├── position: CGPoint?
│   ├── size: CGFloat?
│   ├── visibility: TimeSignatureVisibility
│   └── alignment: TimeSignatureAlignment
└── Domain Methods
    ├── getBeatsPerMeasure() -> Int
    ├── getBeatValue() -> Duration
    ├── getStrongBeats() -> [Int]
    ├── validateNoteDurations([Note]) -> Bool
    └── withVisibility(TimeSignatureVisibility) -> TimeSignature
```

**Clef Management with Layout:**
```
Clef
├── Musical Properties
│   ├── type: ClefType (treble, bass, alto, unpitched)
│   ├── instrumentCompatibility: [InstrumentType]
│   ├── octaveTransposition: Int?
│   └── isDefault: Bool
├── Layout Properties
│   ├── position: CGPoint?
│   ├── size: CGFloat?
│   └── visibility: ClefVisibility
└── Domain Methods
    ├── isCompatibleWith(InstrumentType) -> Bool
    ├── getStaffPositionFor(Pitch) -> StaffPosition
    ├── withSize(CGFloat) -> Clef
    └── shouldDisplayAt(SystemPosition) -> Bool
```

**Rendering Rules:**
- Display clef when: first measure of system, clef change occurs, after system break
- Default clefs: Bagpipes (Treble), Bass/Tenor Drums (Bass), Snare (Unpitched)
- Alternative clefs available per instrument type

### 3.1.5 Barline Types and Layout

**Enhanced Barline System:**
```
Barline
├── Musical Properties
│   ├── type: BarlineType
│   ├── repeatConfiguration: RepeatConfiguration?
│   ├── systemSpanning: Bool
│   └── musicalFunction: MusicalFunction
├── Layout Properties
│   ├── position: CGFloat?
│   ├── height: CGFloat? (for system-spanning)
│   ├── thickness: CGFloat?
│   └── spacing: BarlineSpacing?
└── Domain Methods
    ├── spansInstruments([InstrumentType]) -> Bool
    ├── withPosition(CGFloat) -> Barline
    ├── withSpacing(BarlineSpacing) -> Barline
    └── validatePlacement(SystemContext) -> [ValidationError]
```

**Barline Types with SMuFL Codepoints:**
- None (transparent)
- Single (U+E030)
- Double (U+E031) 
- Final Double (U+E032)
- Repeat Start (U+E040)
- Repeat End (U+E041)
- Repeat Both (U+E042)
- Dashed (U+E036)
- Heavy (U+E034)
- Dotted (U+E037)

### 3.1.6 Ornament System Architecture

**OrnamentDefinition Entity:**
```
OrnamentDefinition
├── Musical Properties
│   ├── type: OrnamentType
│   ├── graceNotePattern: [GraceNote]
│   ├── contextRules: [ContextRule]
│   ├── difficultyLevel: SkillLevel
│   └── regionalVariations: [RegionalStyle: OrnamentVariation]
├── Layout Properties
│   ├── symbolSequence: [SMuFLCodepoint]
│   ├── defaultSpacing: SpacingTemplate
│   ├── positioningRules: PositioningRules
│   └── collisionAvoidance: CollisionRules
└── Domain Methods
    ├── getPatternFor(targetNote: Note, style: RegionalStyle) -> [GraceNote]
    ├── getSymbolSequenceFor(context: MusicalContext) -> [SMuFLCodepoint]
    ├── calculatePositions(targetNote: Note, staffContext: StaffContext) -> [CGPoint]
    └── validatePlacement(context: MusicalContext) -> [ValidationError]
```

**Ornament Types (Bagpipes):**
- Basic: Cut, Strike, Grip
- Complex: Doubling, Half Doubling, Throw
- Advanced: Taorluath, Crunluath, Birl
- Movement: Leumluath, Cadences

**Ornament Types (Drums):**
- Snare: Rolls, Flams, Drags, Roughs (Traditional, Swiss), Open Drags
- Bass/Tenor: Flams, Accents, Rim Shots

### 3.1.7 Layout Value Objects

#### SpacingHints Value Object
```
SpacingHints
├── leading: CGFloat?
├── trailing: CGFloat?
├── above: CGFloat?
├── below: CGFloat?
├── internal: CGFloat? (for multi-symbol elements)
└── isDefault: Bool (computed property)
```

#### VisualStyle Value Object
```
VisualStyle
├── fontSize: CGFloat?
├── color: Color?
├── weight: FontWeight?
├── emphasis: EmphasisStyle?
├── opacity: Float?
└── isDefault: Bool (computed property)
```

#### BreakHint Value Object
```
BreakHint
├── type: BreakType (avoid, prefer, force, forbid)
├── priority: BreakPriority (low, normal, high, critical)
├── scope: BreakScope (measure, system, page)
├── reason: String? (user explanation)
└── musicalJustification: MusicalReason?
```

#### CollisionHints Value Object
```
CollisionHints
├── avoidanceStrategy: AvoidanceStrategy
├── preferredDirection: CollisionDirection?
├── minimumClearance: CGFloat
├── allowOverlap: Bool
└── priority: CollisionPriority
```

### 3.1.8 Layout Configuration Entities

#### TuneLayoutPreference Entity
```
TuneLayoutPreference
├── tuneId: UUID
├── pageBreakPolicy: PageBreakPolicy (mandatory, preferred, allowed, avoid)
├── compressionLevel: CompressionLevel (none, light, moderate, aggressive)
├── spacingPreference: SpacingPreference (loose, normal, tight, custom)
├── orientationOverride: PageOrientation?
├── systemBreakStrategy: SystemBreakStrategy
└── visualEmphasis: VisualEmphasis?
```

#### DocumentLayoutSettings Entity
```
DocumentLayoutSettings
├── defaultPaperSize: PaperSize
├── defaultOrientation: PageOrientation
├── defaultMargins: EdgeInsets
├── globalSpacingFactor: Float (0.5 to 2.0)
├── compressionStrategy: CompressionStrategy
├── breakOptimization: BreakOptimization
├── fontSizeAdjustment: Float
└── accessibilityMode: Bool
```

### 3.1.9 Musical Validation Rules

#### Layout Consistency Rules
```
LayoutValidationRules
├── notePositionBounds: BoundsRule
├── spacingMinimums: SpacingRule
├── systemBreakValidation: SystemBreakRule
├── pageBreakValidation: PageBreakRule
├── ornamentPositioning: OrnamentPositionRule
└── instrumentAlignment: AlignmentRule
```

#### Musical Integrity Rules
```
MusicalValidationRules
├── timeSignatureConsistency: TimeSignatureRule
├── keySignaturePropagation: KeySignatureRule
├── ornamentCompatibility: OrnamentCompatibilityRule
├── instrumentSynchronization: SynchronizationRule
├── barlineLogic: BarlineValidationRule
└── phraseBoundaryRespect: PhraseBoundaryRule
```

### 3.1.10 Performance and Caching Entities

#### LayoutCache Entity
```
LayoutCache
├── systemLayouts: [UUID: SystemLayout]
├── measureWidths: [UUID: CGFloat]
├── ornamentPositions: [UUID: [CGPoint]]
├── lastModified: [UUID: Date]
├── cacheHitCount: Int
└── cacheMissCount: Int
```

#### LayoutInvalidation Entity
```
LayoutInvalidation
├── affectedSystems: Set<UUID>
├── affectedMeasures: Set<UUID>
├── invalidationType: InvalidationType
├── cascadeLevel: CascadeLevel
├── timestamp: Date
└── reason: InvalidationReason
```

## 3.2 Use Cases (Business Logic)

### 3.2.1 Score Management Use Cases

#### CreateScoreUseCase
**Purpose**: Create new Score containing an initial single tune with default layout settings

**Input Parameters:**
- title: String
- composer: String?
- defaultInstruments: [InstrumentType]
- layoutSettings: DocumentLayoutSettings?

**Business Rules:**
- Score must have at least one tune
- Default tune must have at least one part (A part)
- Document layout settings use system defaults if not provided
- Initial tune gets auto-generated ID and default metadata

**Implementation:**
```
execute(title, composer, instruments, layoutSettings) -> ScoreDocument
├── Validate input parameters
├── Create DocumentLayoutSettings (use defaults if nil)
├── Create initial Tune with A Part
├── Create initial MusicalSystem with specified instruments
├── Create initial Measure with default time signature
├── Assign tune to Page[0] in TuneLines structure
├── Save Score through repository
└── Return created ScoreDocument
```

**Side Effects:**
- Triggers initial pagination calculation
- Creates default folder assignment
- Initializes layout cache entries

**Error Conditions:**
- InvalidTitleError (empty or too long)
- UnsupportedInstrumentCombinationError
- PersistenceError

#### DeleteScoreUseCase
**Purpose**: Delete a Score from the repository with proper cleanup

**Input Parameters:**
- scoreId: UUID

**Business Rules:**
- Cannot delete score that is currently open for editing
- Must clear all layout cache entries
- Must remove from any folder assignments
- Must cleanup associated audio files

**Implementation:**
```
execute(scoreId) -> Void
├── Validate score exists and not locked
├── Clear layout cache for all systems in score
├── Delete associated audio recordings
├── Remove from folder assignments
├── Delete score through repository
└── Publish ScoreDeletedEvent
```

**Side Effects:**
- Clears layout caches
- Removes audio files
- Updates folder contents
- Notifies UI of deletion

#### DuplicateScoreUseCase
**Purpose**: Create a copy of existing Score with new identity

**Input Parameters:**
- sourceScoreId: UUID
- newTitle: String
- copyLayoutPreferences: Bool

**Business Rules:**
- Duplicated score gets new UUID for all entities
- Layout preferences optionally copied or reset to defaults
- Audio recordings are not duplicated
- Placement in same folder as source

**Implementation:**
```
execute(sourceScoreId, newTitle, copyLayout) -> ScoreDocument
├── Load source score
├── Deep copy all musical entities with new UUIDs
├── Copy or reset layout preferences based on flag
├── Clear all audio recordings
├── Save duplicated score
├── Copy folder assignments
└── Return new ScoreDocument
```

#### CreateFolderUseCase
**Purpose**: Create a new Folder in the repository for score organization

**Input Parameters:**
- name: String
- parentFolderId: UUID?
- sortOrder: FolderSortOrder

**Business Rules:**
- Folder names must be unique within parent
- Maximum folder depth of 5 levels
- Root folders have nil parentFolderId

**Implementation:**
```
execute(name, parentId, sortOrder) -> Folder
├── Validate folder name uniqueness in parent
├── Validate maximum depth constraint
├── Create Folder entity
├── Save through repository
└── Return created Folder
```

#### SelectScoresUseCase
**Purpose**: Mark one or more Scores as selected for batch operations

**Input Parameters:**
- scoreIds: [UUID]
- selectionMode: SelectionMode (replace, add, toggle)

**Business Rules:**
- Cannot select scores that are locked for editing
- Selection state is transient (not persisted)
- Empty selection is valid state

**Implementation:**
```
execute(scoreIds, mode) -> SelectionState
├── Validate all scores exist and are not locked
├── Apply selection mode to current selection
├── Validate selection constraints
├── Update selection state
└── Publish SelectionChangedEvent
```

### 3.2.2 Score Editing Use Cases

#### OpenScoreForEditingUseCase
**Purpose**: Open selected score for editing with proper locking and layout preparation

**Input Parameters:**
- scoreId: UUID
- editingMode: EditingMode (read-only, collaborative, exclusive)

**Business Rules:**
- Only one exclusive edit session per score
- Read-only mode allows multiple concurrent sessions
- Collaborative mode requires conflict resolution setup

**Implementation:**
```
execute(scoreId, mode) -> EditingSession
├── Validate score exists and check lock status
├── Acquire appropriate lock based on editing mode
├── Load complete score with layout preferences
├── Initialize layout cache for visible systems
├── Setup undo/redo tracking
├── Create editing session
└── Publish ScoreOpenedEvent
```

**Side Effects:**
- Locks score for exclusive editing
- Prepares layout calculations
- Initializes change tracking
- Caches frequently used data

#### CloseScoreUseCase
**Purpose**: Close editor for Score with proper cleanup and save

**Input Parameters:**
- sessionId: UUID
- saveChanges: Bool

**Business Rules:**
- Must save or discard all pending changes
- Must release editing locks
- Must cleanup layout caches
- Must finalize undo/redo history

**Implementation:**
```
execute(sessionId, saveChanges) -> Void
├── Validate editing session exists
├── Save or discard pending changes based on flag
├── Finalize layout calculations if saved
├── Release editing locks
├── Clear layout caches for session
├── Cleanup undo/redo history
└── Publish ScoreClosedEvent
```

#### CreateTuneUseCase
**Purpose**: Create and insert a new Tune in a Score

**Input Parameters:**
- scoreId: UUID
- insertPosition: Int
- tuneTemplate: TuneTemplate
- layoutPreference: TuneLayoutPreference?

**Business Rules:**
- Insert position must be valid (0 to tunes.count)
- New tune gets default instruments from score
- Layout preference inherits from document if not specified
- Must trigger pagination recalculation

**Implementation:**
```
execute(scoreId, position, template, layoutPref) -> Tune
├── Validate score exists and is editable
├── Validate insert position
├── Create Tune from template
├── Set layout preference (inherit or use provided)
├── Insert tune at specified position
├── Update Pages[] structure for new tune
├── Trigger pagination recalculation
├── Save score
└── Return created Tune
```

**Side Effects:**
- Updates Pages[] assignment structure
- Triggers layout recalculation
- May affect page breaks for subsequent tunes
- Updates navigation elements

#### DeleteTuneUseCase
**Purpose**: Delete a Tune from the Score with layout cleanup

**Input Parameters:**
- scoreId: UUID
- tuneId: UUID

**Business Rules:**
- Cannot delete the last remaining tune in score
- Must cleanup all associated layout data
- Must update Pages[] structure
- Must preserve musical integrity of remaining tunes

**Implementation:**
```
execute(scoreId, tuneId) -> Void
├── Validate score has multiple tunes
├── Validate tune exists in score
├── Remove tune from score
├── Clear layout cache for tune's systems
├── Update Pages[] structure removing tune's TuneLines
├── Trigger pagination recalculation for affected pages
├── Save score
└── Publish TuneDeletedEvent
```

#### InsertInstrumentInTuneUseCase
**Purpose**: Insert an Instrument in a Tune with proper system layout updates

**Input Parameters:**
- tuneId: UUID
- instrumentType: InstrumentType
- insertPosition: Int

**Business Rules:**
- Instrument must be compatible with existing instruments
- Insert position within valid range (0 to instruments.count)
- Must add instrument to all systems in tune
- Must trigger system layout recalculation

**Implementation:**
```
execute(tuneId, instrumentType, position) -> Void
├── Validate tune exists and is editable
├── Validate instrument compatibility
├── Validate insert position
├── Add instrument to all systems in tune
├── Create default measures for new instrument
├── Update system layout cache
├── Trigger system layout recalculation
├── Save changes
└── Publish InstrumentAddedEvent
```

**Side Effects:**
- Updates all MusicalSystem entities in tune
- Recalculates staff spacing and positions
- May affect page layout if system height changes
- Updates instrument-specific layout caches

#### RemoveInstrumentFromTuneUseCase
**Purpose**: Remove an Instrument from a Tune with cleanup

**Input Parameters:**
- tuneId: UUID
- instrumentType: InstrumentType

**Business Rules:**
- Cannot remove last remaining instrument
- Must remove from all systems in tune
- Must cleanup associated layout data
- Must preserve synchronization of remaining instruments

**Implementation:**
```
execute(tuneId, instrumentType) -> Void
├── Validate tune has multiple instruments
├── Validate instrument exists in tune
├── Remove instrument from all systems
├── Clear instrument-specific layout cache
├── Update system layout for remaining instruments
├── Trigger system layout recalculation
├── Save changes
└── Publish InstrumentRemovedEvent
```

#### HideInstrumentUseCase / ShowInstrumentUseCase
**Purpose**: Toggle instrument visibility without removing musical content

**Input Parameters:**
- tuneId: UUID
- instrumentType: InstrumentType
- visibility: Bool

**Business Rules:**
- Hidden instruments retain musical content
- Layout calculations exclude hidden instruments
- At least one instrument must remain visible
- Hidden state persists with score

**Implementation:**
```
execute(tuneId, instrumentType, visibility) -> Void
├── Validate at least one instrument remains visible
├── Update instrument visibility flag
├── Update system layout calculations
├── Trigger layout recalculation for affected systems
├── Save changes
└── Publish InstrumentVisibilityChangedEvent
```

### 3.2.3 Part Management Use Cases

#### CreatePartUseCase
**Purpose**: Create and insert a new Part in a Tune

**Input Parameters:**
- tuneId: UUID
- partLetter: String
- insertPosition: Int
- partTemplate: PartTemplate?

**Business Rules:**
- Part letters should be unique within tune (A, B, C, etc.)
- Insert position must be valid
- New part inherits instruments from tune
- Gets default number of measures based on template

**Implementation:**
```
execute(tuneId, letter, position, template) -> Part
├── Validate tune exists and is editable
├── Validate part letter uniqueness
├── Validate insert position
├── Create Part with systems for all tune instruments
├── Add default measures based on template
├── Insert part at specified position
├── Update TuneLines structure
├── Trigger pagination recalculation
├── Save changes
└── Return created Part
```

#### SelectPartsUseCase
**Purpose**: Mark one or more Parts as selected for batch operations

**Input Parameters:**
- partIds: [UUID]
- selectionMode: SelectionMode

**Business Rules:**
- Selected parts must be within same tune
- Selection enables copy/cut/delete operations
- Selection state is transient

**Implementation:**
```
execute(partIds, mode) -> PartSelection
├── Validate all parts exist and are in same tune
├── Apply selection mode to current selection
├── Validate selection constraints
├── Update part selection state
└── Publish PartSelectionChangedEvent
```

#### CopySelectedPartsUseCase
**Purpose**: Copy selected Parts to the Clipboard for paste operations

**Input Parameters:**
- partSelection: PartSelection

**Business Rules:**
- Copies complete part structure including all instruments
- Maintains relative timing and layout hints
- Clipboard content has expiration time
- Does not affect source parts

**Implementation:**
```
execute(partSelection) -> ClipboardContent
├── Validate parts are selected
├── Deep copy selected parts with all content
├── Preserve layout hints and preferences
├── Create clipboard content with metadata
├── Set clipboard expiration time
├── Store in clipboard repository
└── Publish PartscopiedEvent
```

#### PasteSelectedPartsUseCase
**Purpose**: Insert Parts from Clipboard into current Tune

**Input Parameters:**
- tuneId: UUID
- insertPosition: Int
- pasteMode: PasteMode (copy, move, reference)

**Business Rules:**
- Clipboard content must be compatible with target tune
- Instrument compatibility must be verified
- Pasted parts get new UUIDs
- Layout hints adapted to target context

**Implementation:**
```
execute(tuneId, position, mode) -> [Part]
├── Validate clipboard has compatible content
├── Validate insert position
├── Check instrument compatibility
├── Create new parts with fresh UUIDs
├── Adapt layout hints to target tune context
├── Insert parts at specified position
├── Update TuneLines structure
├── Trigger pagination recalculation
├── Save changes
└── Return pasted parts
```

### 3.2.4 Measure and Bar Management Use Cases

#### CreateBarUseCase
**Purpose**: Create and insert a new Bar (Measure) in a Part

**Input Parameters:**
- partId: UUID
- systemId: UUID
- insertPosition: Int
- timeSignature: TimeSignature?

**Business Rules:**
- New measure inherits time signature from previous or uses provided
- Must be added to all instruments in system simultaneously
- Gets default note content based on time signature
- Triggers layout recalculation for affected systems

**Implementation:**
```
execute(partId, systemId, position, timeSig) -> Measure
├── Validate part and system exist
├── Validate insert position
├── Determine effective time signature
├── Create measure for each instrument in system
├── Add default content (rests) based on time signature
├── Insert measures at specified position
├── Update system layout cache
├── Trigger layout recalculation
├── Save changes
└── Return created measure
```

#### DeleteBarUseCase
**Purpose**: Delete selected measure(s) with proper synchronization

**Input Parameters:**
- measureIds: [UUID]
- confirmDeletion: Bool

**Business Rules:**
- Cannot delete last measure in part
- Must delete from all instruments simultaneously
- Must preserve musical timing of remaining measures
- Confirmation required for non-empty measures

**Implementation:**
```
execute(measureIds, confirm) -> Void
├── Validate measures exist and are deletable
├── Check if measures contain content and require confirmation
├── Validate not deleting last measures in part
├── Remove measures from all affected instruments
├── Update system layout cache
├── Trigger layout recalculation for affected systems
├── Save changes
└── Publish MeasuresDeletedEvent
```

#### SetBarTimeSigUseCase
**Purpose**: Set time signature for measure with cascading effects

**Input Parameters:**
- measureId: UUID
- timeSignature: TimeSignature
- applyToSubsequent: Bool

**Business Rules:**
- Time signature change affects current and subsequent measures if specified
- Must validate existing note content fits new time signature
- May require layout recalculation due to spacing changes
- Cascading application stops at next explicit time signature

**Implementation:**
```
execute(measureId, timeSig, cascade) -> AffectedMeasures
├── Validate measure exists and is editable
├── Validate note content compatibility with new time signature
├── Apply time signature to target measure
├── If cascading, apply to subsequent measures until next explicit signature
├── Update layout cache for affected measures
├── Trigger layout recalculation for spacing changes
├── Save changes
├── Return list of affected measures
└── Publish TimeSignatureChangedEvent
```

**Side Effects:**
- May trigger pagination recalculation if spacing changes significantly
- Updates measure width calculations
- May affect system break positions

#### SetBarOpeningBarLineUseCase / SetBarClosingBarLineUseCase
**Purpose**: Set barline type at measure boundaries

**Input Parameters:**
- measureId: UUID
- barlineType: BarlineType
- position: BarlinePosition (opening/closing)

**Business Rules:**
- Barline must be compatible with musical context
- Repeat barlines require proper pairing
- System-spanning barlines affect all instruments
- Layout recalculation may be needed for barline spacing

**Implementation:**
```
execute(measureId, barlineType, position) -> Void
├── Validate measure exists and is editable
├── Validate barline type compatibility
├── Check repeat barline pairing if applicable
├── Set barline on measure
├── Update barline layout cache
├── Trigger layout recalculation if spacing affected
├── Save changes
└── Publish BarlineChangedEvent
```

### 3.2.5 Layout and Pagination Use Cases

#### RecalculatePaginationUseCase
**Purpose**: Recalculate document pagination when layout-affecting changes occur

**Input Parameters:**
- documentId: UUID
- scope: RecalculationScope (measure, system, page, document)
- trigger: RecalculationTrigger

**Business Rules:**
- Scope determines extent of recalculation needed
- Must preserve user layout preferences where possible
- Must respect orientation-based page break rules
- Must maintain musical phrase integrity

**Implementation:**
```
execute(documentId, scope, trigger) -> PaginationResult
├── Validate document exists
├── Determine recalculation scope based on trigger type
├── Load layout preferences and constraints
├── Calculate new layout within scope
├── Update Pages[] structure with new TuneLines assignments
├── Validate musical phrase preservation
├── Apply orientation-based page break rules
├── Cache new layout calculations
├── Save updated pagination
└── Return pagination result with change summary
```

**Trigger Types:**
- Font size change (document scope)
- Paper size change (document scope) 
- Instrument addition (system scope)
- Note spacing change (measure scope)
- Orientation change (page scope)

#### SetTuneLayoutPreferenceUseCase
**Purpose**: Set layout preferences for specific tune

**Input Parameters:**
- tuneId: UUID
- layoutPreference: TuneLayoutPreference

**Business Rules:**
- Preference must be compatible with tune content
- Changes may trigger pagination recalculation
- Preferences persist with document
- Must validate against document-level constraints

**Implementation:**
```
execute(tuneId, preference) -> Void
├── Validate tune exists
├── Validate preference compatibility with tune content
├── Check conflicts with document-level constraints
├── Update tune layout preference
├── Trigger pagination recalculation if layout-affecting
├── Save preference changes
└── Publish LayoutPreferenceChangedEvent
```

#### OptimizeLayoutUseCase
**Purpose**: Intelligent optimization of layout for readability and space efficiency

**Input Parameters:**
- documentId: UUID
- optimizationCriteria: OptimizationCriteria
- scope: OptimizationScope

**Business Rules:**
- Must respect user-specified layout preferences
- Must maintain musical integrity and readability
- Optimization cannot violate musical phrase boundaries
- Must consider performance vs practice context

**Implementation:**
```
execute(documentId, criteria, scope) -> OptimizationResult
├── Analyze current layout efficiency
├── Identify optimization opportunities within scope
├── Apply optimization algorithms respecting constraints
├── Validate musical phrase preservation
├── Calculate new optimal layout
├── Update layout caches
├── Generate optimization report
├── Save optimized layout
└── Return optimization result
```

### 3.2.6 Audio Integration Use Cases

#### RecordPracticeSessionUseCase
**Purpose**: Capture and store practice recordings linked to score

**Input Parameters:**
- scoreId: UUID
- recordingSettings: AudioRecordingSettings
- linkedToTune: UUID?

**Business Rules:**
- Recording must be linked to specific score
- Audio format must be platform-compatible
- Metadata includes tempo, key, and performance notes
- Storage location respects user privacy settings

**Implementation:**
```
execute(scoreId, settings, tuneId) -> PracticeRecording
├── Validate score exists and is accessible
├── Setup audio recording with specified settings
├── Begin recording session
├── Monitor audio levels and quality
├── Stop recording on user command
├── Process and compress audio file
├── Create recording metadata with score linkage
├── Save recording to designated storage
└── Return practice recording reference
```

#### PlayMetronomeUseCase
**Purpose**: Generate precise metronome based on time signature and tempo

**Input Parameters:**
- tempo: BPM
- timeSignature: TimeSignature
- accentPattern: AccentPattern?
- duration: TimeInterval?

**Business Rules:**
- Metronome timing must be sample-accurate
- Accent pattern must align with time signature
- Audio must not interfere with recording
- Tempo changes must be smooth

**Implementation:**
```
execute(tempo, timeSig, pattern, duration) -> MetronomeSession
├── Validate tempo and time signature parameters
├── Setup audio engine for metronome generation
├── Calculate beat timing and accent pattern
├── Generate audio samples for click sounds
├── Start metronome playback
├── Monitor for tempo changes or stop commands
├── Cleanup audio resources on completion
└── Return metronome session reference
```

#### AnalyzeIntonationUseCase
**Purpose**: Pitch detection and tuning feedback for practice

**Input Parameters:**
- audioData: AudioBuffer
- targetPitches: [Pitch]
- instrumentType: InstrumentType

**Business Rules:**
- Analysis must account for instrument characteristics
- Pitch detection accuracy depends on audio quality
- Feedback must be musically meaningful
- Results should guide practice improvement

**Implementation:**
```
execute(audioData, targets, instrument) -> IntonationAnalysis
├── Validate audio data quality and format
├── Apply instrument-specific analysis parameters
├── Perform pitch detection on audio samples
├── Compare detected pitches with targets
├── Calculate intonation accuracy metrics
├── Generate practice feedback recommendations
├── Store analysis results for progress tracking
└── Return intonation analysis report
```

### 3.2.7 Export and File Format Use Cases

#### ExportToPDFUseCase
**Purpose**: Render score to PDF with SMuFL compliance and layout preservation

**Input Parameters:**
- documentId: UUID
- exportSettings: PDFExportSettings
- pageRange: PageRange?

**Business Rules:**
- PDF must embed SMuFL fonts for compatibility
- Layout must match screen rendering exactly
- Metadata must include title, composer, copyright
- Vector graphics ensure scalability

**Implementation:**
```
execute(documentId, settings, pageRange) -> PDFExportResult
├── Validate document exists and is accessible
├── Load complete document with layout data
├── Setup PDF generation context with embedded fonts
├── Render each page maintaining layout fidelity
├── Include metadata and copyright information
├── Optimize PDF for target use case (print/screen)
├── Save PDF to specified location
└── Return export result with file reference
```

#### BatchExportUseCase
**Purpose**: Process multiple scores with various formats efficiently

**Input Parameters:**
- scoreIds: [UUID]
- exportFormats: [ExportFormat]
- exportSettings: BatchExportSettings

**Business Rules:**
- Batch processing must handle errors gracefully
- Progress reporting must be accurate
- File naming must avoid conflicts
- Memory usage must be optimized for large batches

**Implementation:**
```
execute(scoreIds, formats, settings) -> BatchExportResult
├── Validate all scores exist and are accessible
├── Setup batch processing queue
├── For each score and format combination:
│   ├── Load score with layout
│   ├── Export to specified format
│   ├── Handle errors without stopping batch
│   └── Update progress reporting
├── Compile batch results and error reports
├── Cleanup temporary resources
└── Return batch export result
```

### 3.2.8 Synchronization Use Cases

#### SyncToCloudUseCase
**Purpose**: Upload changes to user's chosen cloud provider

**Input Parameters:**
- documentId: UUID
- syncMode: SyncMode (full, incremental, conflict-resolution)
- cloudProvider: CloudProvider

**Business Rules:**
- Sync must preserve document integrity
- Conflicts must be detected and reported
- Partial sync failures must be recoverable
- User privacy settings must be respected

**Implementation:**
```
execute(documentId, mode, provider) -> SyncResult
├── Validate document and cloud provider access
├── Determine changes since last sync
├── Prepare sync package with change metadata
├── Upload changes to cloud provider
├── Handle network interruptions gracefully
├── Verify upload integrity
├── Update local sync status
├── Resolve any conflicts detected
└── Return sync result with status
```

#### ResolveConflictsUseCase
**Purpose**: Merge concurrent edits with conflict resolution

**Input Parameters:**
- documentId: UUID
- conflictResolutionStrategy: ResolutionStrategy
- userChoices: [ConflictChoice]?

**Business Rules:**
- Musical integrity must be preserved during merge
- User must approve significant changes
- Backup created before applying resolution
- Audit trail maintained for all changes

**Implementation:**
```
execute(documentId, strategy, choices) -> ConflictResolution
├── Analyze conflicts and their scope
├── Create backup of current document state
├── Apply automatic resolution where safe
├── Present user choices for complex conflicts
├── Apply user-selected resolutions
├── Validate musical integrity of merged result
├── Save resolved document
├── Update audit trail
└── Return conflict resolution summary
```

## 3.3 Repository Interfaces

### 3.3.1 Core Data Repositories

#### MusicalDocumentRepository
**Purpose**: Unified repository for complete musical documents with embedded layout data

**Interface Definition:**
```
MusicalDocumentRepository
├── save(MusicalDocument) -> SaveResult
├── load(UUID) -> MusicalDocument?
├── delete(UUID) -> DeleteResult
├── exists(UUID) -> Bool
├── search(SearchCriteria) -> [DocumentSummary]
├── getMetadata(UUID) -> DocumentMetadata?
├── saveMetadata(UUID, DocumentMetadata) -> SaveResult
├── listDocuments(FolderID?) -> [DocumentSummary]
├── moveDocument(UUID, FolderID) -> MoveResult
└── validateDocument(UUID) -> [ValidationError]
```

**Implementation Requirements:**
- Support for JSON serialization with embedded layout data
- Atomic save operations with rollback capability
- Concurrent access protection with appropriate locking
- Version tracking for document changes
- Efficient partial loading for large documents
- Platform-specific storage optimization (Core Data, Room, etc.)

**Error Handling:**
- DocumentNotFoundError
- DocumentCorruptedError
- ConcurrentModificationError
- InsufficientStorageError
- SerializationError

#### TuneRepository
**Purpose**: Specialized repository for individual tune operations and queries

**Interface Definition:**
```
TuneRepository
├── saveTune(Tune) -> SaveResult
├── loadTune(UUID) -> Tune?
├── deleteTune(UUID) -> DeleteResult
├── getTunesByType(TuneType) -> [Tune]
├── getTunesByInstrument(InstrumentType) -> [Tune]
├── searchTunes(SearchCriteria) -> [TuneSummary]
├── getTuneMetadata(UUID) -> TuneMetadata?
├── updateTuneMetadata(UUID, TuneMetadata) -> SaveResult
├── getTunesInDateRange(DateRange) -> [TuneSummary]
└── validateTune(UUID) -> [ValidationError]
```

**Advanced Query Capabilities:**
- Full-text search across tune titles and composers
- Musical characteristic filtering (key, time signature, tempo)
- Layout preference matching
- Difficulty level categorization
- Geographic origin classification

#### FolderRepository
**Purpose**: Hierarchical organization and management of musical documents

**Interface Definition:**
```
FolderRepository
├── createFolder(name: String, parentID: UUID?) -> Folder
├── deleteFolder(UUID) -> DeleteResult
├── moveFolder(UUID, newParentID: UUID?) -> MoveResult
├── renameFolder(UUID, newName: String) -> RenameResult
├── getFolderContents(UUID) -> FolderContents
├── getFolderHierarchy(UUID?) -> FolderTree
├── searchFolders(name: String) -> [Folder]
├── getFolderPath(UUID) -> [Folder]
├── validateFolderOperation(FolderOperation) -> [ValidationError]
└── optimizeFolderStructure() -> OptimizationResult
```

**Business Logic Integration:**
- Maximum depth enforcement (5 levels)
- Circular reference prevention
- Name uniqueness within parent folder
- Cascade deletion with user confirmation
- Folder sorting and display order management

### 3.3.2 Layout and Cache Repositories

#### LayoutCacheRepository
**Purpose**: High-performance caching of calculated layout data

**Interface Definition:**
```
LayoutCacheRepository
├── cacheSystemLayout(UUID, SystemLayout) -> CacheResult
├── getCachedSystemLayout(UUID) -> SystemLayout?
├── cacheMeasureLayout(UUID, MeasureLayout) -> CacheResult
├── getCachedMeasureLayout(UUID) -> MeasureLayout?
├── cachePageLayout(UUID, PageLayout) -> CacheResult
├── getCachedPageLayout(UUID) -> PageLayout?
├── invalidateLayoutCache([UUID]) -> InvalidationResult
├── invalidateLayoutCacheForDocument(UUID) -> InvalidationResult
├── clearLayoutCache() -> ClearResult
├── getCacheStatistics() -> CacheStatistics
├── optimizeCache() -> OptimizationResult
└── setCachePolicy(CachePolicy) -> PolicyResult
```

**Performance Requirements:**
- Sub-millisecond cache lookup times
- LRU eviction policy with configurable size limits
- Memory pressure handling with automatic cleanup
- Cache hit ratio monitoring and optimization
- Thread-safe concurrent access patterns
- Platform-specific memory management integration

**Cache Invalidation Strategies:**
- Dependency-based invalidation (measure changes invalidate system)
- Time-based expiration for layout calculations
- Version-based invalidation using entity timestamps
- Manual invalidation for user-triggered layout changes
- Batch invalidation for efficient bulk operations

#### LayoutPreferenceRepository
**Purpose**: Persistent storage of user layout preferences and document settings

**Interface Definition:**
```
LayoutPreferenceRepository
├── saveDocumentLayoutSettings(UUID, DocumentLayoutSettings) -> SaveResult
├── loadDocumentLayoutSettings(UUID) -> DocumentLayoutSettings?
├── saveTuneLayoutPreference(UUID, TuneLayoutPreference) -> SaveResult
├── loadTuneLayoutPreference(UUID) -> TuneLayoutPreference?
├── saveGlobalLayoutDefaults(GlobalLayoutDefaults) -> SaveResult
├── loadGlobalLayoutDefaults() -> GlobalLayoutDefaults
├── saveUserLayoutProfile(UserLayoutProfile) -> SaveResult
├── loadUserLayoutProfile() -> UserLayoutProfile?
├── exportLayoutPreferences() -> LayoutPreferenceExport
├── importLayoutPreferences(LayoutPreferenceExport) -> ImportResult
└── resetLayoutPreferences(ResetScope) -> ResetResult
```

**Preference Hierarchy Management:**
- Global defaults → Document defaults → Tune overrides
- User profile preferences with theme support
- Context-specific preferences (performance vs practice)
- Regional and cultural preference variations
- Accessibility-driven layout adaptations

### 3.3.3 Audio and Media Repositories

#### AudioRepository
**Purpose**: Management of practice recordings and audio analysis data

**Interface Definition:**
```
AudioRepository
├── saveRecording(PracticeRecording) -> SaveResult
├── loadRecording(UUID) -> PracticeRecording?
├── deleteRecording(UUID) -> DeleteResult
├── getRecordingsForScore(UUID) -> [PracticeRecording]
├── getRecordingsForTune(UUID) -> [PracticeRecording]
├── getRecordingsByDateRange(DateRange) -> [PracticeRecording]
├── saveAudioAnalysis(UUID, AudioAnalysis) -> SaveResult
├── loadAudioAnalysis(UUID) -> AudioAnalysis?
├── compressAudioFile(UUID, CompressionSettings) -> CompressionResult
├── exportRecording(UUID, ExportFormat) -> ExportResult
├── getAudioStatistics() -> AudioStatistics
└── cleanupExpiredRecordings() -> CleanupResult
```

**Audio Processing Integration:**
- Platform-specific audio format optimization
- Automatic compression and quality management
- Metadata embedding with musical context
- Progress tracking for long operations
- Background processing for non-critical operations

#### CloudStorageRepository
**Purpose**: Abstract interface for various cloud storage providers

**Interface Definition:**
```
CloudStorageRepository
├── uploadDocument(UUID, CloudPath) -> UploadResult
├── downloadDocument(CloudPath) -> DownloadResult
├── deleteCloudDocument(CloudPath) -> DeleteResult
├── listCloudDocuments(CloudDirectory) -> [CloudFile]
├── syncDocument(UUID, SyncStrategy) -> SyncResult
├── resolveConflicts(UUID, ConflictResolution) -> ResolutionResult
├── getCloudQuota() -> QuotaInfo
├── getCloudSyncStatus(UUID) -> SyncStatus
├── configureCloudProvider(CloudProviderConfig) -> ConfigResult
├── validateCloudConnection() -> ConnectionStatus
└── cleanupCloudStorage() -> CleanupResult
```

**Multi-Provider Support:**
- Provider-agnostic interface implementation
- Automatic provider selection based on availability
- Conflict resolution with user intervention
- Offline queuing with automatic sync retry
- Bandwidth optimization and progress tracking

### 3.3.4 Export and Import Repositories

#### ExportRepository
**Purpose**: Multi-format export with layout preservation and platform optimization

**Interface Definition:**
```
ExportRepository
├── exportToPDF(UUID, PDFExportSettings) -> ExportResult
├── exportToPNG(UUID, ImageExportSettings) -> ExportResult
├── exportToSVG(UUID, SVGExportSettings) -> ExportResult
├── exportToMIDI(UUID, MIDIExportSettings) -> ExportResult
├── exportToMusicXML(UUID, MusicXMLExportSettings) -> ExportResult
├── batchExport([UUID], BatchExportSettings) -> BatchExportResult
├── getExportFormats() -> [ExportFormat]
├── validateExportSettings(ExportSettings) -> [ValidationError]
├── getExportProgress(UUID) -> ExportProgress
├── cancelExport(UUID) -> CancelResult
└── optimizeExportSettings(ExportContext) -> OptimizedSettings
```

**Quality Assurance:**
- Layout consistency validation across formats
- SMuFL font embedding verification
- Color profile management for print output
- Resolution optimization for target devices
- Metadata preservation across format conversions

### 3.3.5 Repository Implementation Patterns

#### Base Repository Interface
**Purpose**: Common functionality and patterns for all repository implementations

**Interface Definition:**
```
BaseRepository<T>
├── validateEntity(T) -> [ValidationError]
├── getEntityVersion(UUID) -> EntityVersion
├── lockEntity(UUID, LockType) -> LockResult
├── unlockEntity(UUID) -> UnlockResult
├── getEntityLockStatus(UUID) -> LockStatus
├── beginTransaction() -> Transaction
├── commitTransaction(Transaction) -> CommitResult
├── rollbackTransaction(Transaction) -> RollbackResult
├── getRepositoryHealth() -> HealthStatus
└── optimizeRepository() -> OptimizationResult
```

**Cross-Cutting Concerns:**
- Audit trail logging for all repository operations
- Performance monitoring and metrics collection
- Error logging with contextual information
- Transaction management with nested transaction support
- Entity validation with business rule enforcement

## 3.4 Domain Services

### 3.4.1 Layout Calculation Services

#### LayoutCoordinationService
**Purpose**: Coordinate layout calculations across different scopes and triggers

**Service Interface:**
```
LayoutCoordinationService
├── calculateDocumentLayout(UUID, LayoutContext) -> DocumentLayoutResult
├── calculatePageLayout(UUID, PageLayoutContext) -> PageLayoutResult
├── calculateSystemLayout(UUID, SystemLayoutContext) -> SystemLayoutResult
├── calculateMeasureLayout(UUID, MeasureLayoutContext) -> MeasureLayoutResult
├── optimizeLayoutForContext(UUID, OptimizationContext) -> OptimizationResult
├── validateLayoutIntegrity(UUID) -> [ValidationError]
├── previewLayoutChanges(UUID, LayoutChanges) -> LayoutPreview
├── applyLayoutChanges(UUID, LayoutChanges) -> LayoutResult
├── getLayoutMetrics(UUID) -> LayoutMetrics
└── estimateLayoutCalculationTime(LayoutScope) -> TimeEstimate
```

**Coordination Logic:**
- Dependency analysis for efficient recalculation ordering
- Parallel calculation of independent layout regions
- Cache invalidation with minimal scope impact
- Progressive enhancement for incremental updates
- Rollback capability for failed layout operations

#### PaginationService
**Purpose**: Intelligent pagination with musical phrase awareness

**Service Interface:**
```
PaginationService
├── calculatePagination(UUID, PaginationContext) -> PaginationResult
├── optimizePageBreaks(UUID, OptimizationCriteria) -> PageBreakResult
├── validatePageBreaks(UUID) -> [ValidationError]
├── previewPaginationChanges(UUID, PaginationChanges) -> PaginationPreview
├── applyPaginationChanges(UUID, PaginationChanges) -> PaginationResult
├── getPageCapacity(PageDimensions, ContentType) -> CapacityInfo
├── analyzeMusicianPhrasing(UUID) -> PhrasingAnalysis
├── suggestOptimalBreaks(UUID, BreakCriteria) -> [BreakSuggestion]
├── handleOrientationChanges(UUID, OrientationChanges) -> OrientationResult
└── generatePaginationReport(UUID) -> PaginationReport
```

**Musical Intelligence:**
- Phrase boundary detection and preservation
- Part transition optimization for page breaks
- Rehearsal mark and section break consideration
- Instrument ensemble synchronization requirements
- Performance context adaptation (solo vs ensemble)

### 3.4.2 Musical Analysis Services

#### MusicalValidationService
**Purpose**: Comprehensive validation of musical content and structure

**Service Interface:**
```
MusicalValidationService
├── validateMusicalDocument(UUID) -> [ValidationError]
├── validateTune(UUID) -> [ValidationError]
├── validatePart(UUID) -> [ValidationError]
├── validateMusicalSystem(UUID) -> [ValidationError]
├── validateMeasure(UUID) -> [ValidationError]
├── validateTimeSignatureConsistency(UUID) -> [ValidationError]
├── validateKeySignaturePropagation(UUID) -> [ValidationError]
├── validateInstrumentCompatibility(UUID) -> [ValidationError]
├── validateOrnamentPlacement(UUID) -> [ValidationError]
├── validateBarlineSequence(UUID) -> [ValidationError]
├── suggestMusicalImprovements(UUID) -> [ImprovementSuggestion]
└── generateValidationReport(UUID) -> ValidationReport
```

**Validation Categories:**
- Musical theory compliance (time signatures, key relationships)
- Instrument-specific notation correctness
- Ensemble synchronization requirements
- Performance practicality assessment
- Regional style convention adherence

#### OrnamentAnalysisService
**Purpose**: Context-aware ornament analysis and suggestion system

**Service Interface:**
```
OrnamentAnalysisService
├── analyzeOrnamentPlacement(UUID) -> OrnamentAnalysis
├── suggestOrnamentVariations(UUID, StyleContext) -> [OrnamentSuggestion]
├── validateOrnamentCompatibility(UUID) -> [ValidationError]
├── optimizeOrnamentSpacing(UUID) -> SpacingOptimization
├── analyzeOrnamentComplexity(UUID) -> ComplexityAnalysis
├── compareOrnamentStyles([RegionalStyle]) -> StyleComparison
├── generateOrnamentGuide(InstrumentType, SkillLevel) -> OrnamentGuide
├── adaptOrnamentsForSkillLevel(UUID, SkillLevel) -> AdaptationResult
├── detectOrnamentPatterns(UUID) -> [OrnamentPattern]
└── exportOrnamentAnalysis(UUID, ExportFormat) -> ExportResult
```

**Cultural Sensitivity:**
- Regional style variation recognition
- Traditional vs modern ornament usage
- Competition vs traditional performance contexts
- Educational progression recommendations
- Historical accuracy assessment

### 3.4.3 Performance and Optimization Services

#### CacheManagementService
**Purpose**: Intelligent caching with performance optimization

**Service Interface:**
```
CacheManagementService
├── optimizeCacheStrategy(CacheUsagePattern) -> CacheStrategy
├── preloadCacheForDocument(UUID) -> PreloadResult
├── invalidateCacheIntelligently([UUID]) -> InvalidationResult
├── analyzeCachePerformance() -> CachePerformanceReport
├── adjustCacheSize(MemoryPressure) -> CacheAdjustmentResult
├── getCacheRecommendations() -> [CacheRecommendation]
├── warmupCacheForSession(SessionContext) -> WarmupResult
├── cleanupStaleCacheEntries() -> CleanupResult
├── migrateCacheFormat(CacheFormatVersion) -> MigrationResult
└── generateCacheReport() -> CacheReport
```

**Performance Optimization:**
- Predictive cache preloading based on user behavior
- Memory pressure response with intelligent eviction
- Cache hit ratio optimization through usage analysis
- Platform-specific cache tuning parameters
- Background cache maintenance operations

#### PerformanceMonitoringService
**Purpose**: System performance monitoring and optimization recommendations

**Service Interface:**
```
PerformanceMonitoringService
├── monitorLayoutPerformance(OperationType) -> PerformanceMetrics
├── analyzeBottlenecks() -> [PerformanceBottleneck]
├── generatePerformanceReport() -> PerformanceReport
├── optimizePerformanceSettings() -> OptimizationResult
├── predictPerformanceImpact(OperationType) -> PerformanceImpact
├── trackUserExperienceMetrics() -> UXMetrics
├── identifyPerformanceRegressions() -> [PerformanceRegression]
├── recommendPerformanceImprovements() -> [PerformanceRecommendation]
├── monitorMemoryUsage() -> MemoryUsageReport
└── optimizeForPlatform(PlatformType) -> PlatformOptimization
```

**Metrics Collection:**
- Layout calculation timing and complexity
- Memory usage patterns and peak consumption
- User interaction response times
- Cache effectiveness and hit ratios
- Platform-specific performance characteristics

### 3.4.4 Collaboration Services

#### ConflictResolutionService
**Purpose**: Intelligent conflict resolution for collaborative editing

**Service Interface:**
```
ConflictResolutionService
├── detectConflicts(UUID, ChangeSet) -> [Conflict]
├── analyzeConflictComplexity(Conflict) -> ConflictComplexity
├── suggestResolutionStrategies(Conflict) -> [ResolutionStrategy]
├── applyAutomaticResolution([Conflict]) -> ResolutionResult
├── prepareManualResolution(Conflict) -> ResolutionContext
├── applyManualResolution(Conflict, UserChoice) -> ResolutionResult
├── validateResolutionIntegrity(ResolutionResult) -> [ValidationError]
├── createResolutionAuditTrail(ResolutionResult) -> AuditTrail
├── rollbackConflictResolution(UUID) -> RollbackResult
└── generateConflictReport([Conflict]) -> ConflictReport
```

**Resolution Strategies:**
- Three-way merge with common ancestor analysis
- Musical priority-based automatic resolution
- User preference learning and application
- Collaborative annotation and discussion support
- Version history preservation and rollback capability

#### SynchronizationService
**Purpose**: Multi-device and cloud synchronization coordination

**Service Interface:**
```
SynchronizationService
├── synchronizeDocument(UUID, SyncStrategy) -> SyncResult
├── detectSyncConflicts(UUID) -> [SyncConflict]
├── resolveSyncConflicts(UUID, ConflictResolution) -> ResolutionResult
├── validateSyncIntegrity(UUID) -> [ValidationError]
├── optimizeSyncStrategy(SyncContext) -> OptimizedStrategy
├── monitorSyncProgress(UUID) -> SyncProgress
├── handleSyncFailure(UUID, FailureReason) -> RecoveryResult
├── scheduleSyncOperation(UUID, SyncSchedule) -> ScheduleResult
├── getSyncStatistics() -> SyncStatistics
└── configureSyncBehavior(SyncConfiguration) -> ConfigResult
```

**Synchronization Intelligence:**
- Bandwidth-aware sync optimization
- Offline operation queuing with priority handling
- Incremental sync with delta compression
- Multi-device conflict prevention strategies
- User notification and intervention protocols

## 3.5 Domain Events and Integration

### 3.5.1 Core Domain Events

#### Musical Content Events
**Purpose**: Notification system for musical content changes

**Event Definitions:**
```
ScoreCreatedEvent
├── scoreId: UUID
├── title: String
├── composer: String?
├── createdBy: UserID
├── timestamp: Date
└── initialLayout: DocumentLayoutSettings

TuneAddedEvent
├── scoreId: UUID
├── tuneId: UUID
├── insertPosition: Int
├── tuneMetadata: TuneMetadata
├── layoutPreference: TuneLayoutPreference?
├── triggeredBy: UserID
└── timestamp: Date

PartCreatedEvent
├── tuneId: UUID
├── partId: UUID
├── partLetter: String
├── insertPosition: Int
├── systemCount: Int
├── triggeredBy: UserID
└── timestamp: Date

MeasureAddedEvent
├── systemId: UUID
├── measureId: UUID
├── insertPosition: Int
├── timeSignature: TimeSignature?
├── affectedInstruments: [InstrumentType]
├── triggeredBy: UserID
└── timestamp: Date

NoteModifiedEvent
├── noteId: UUID
├── measureId: UUID
├── previousState: NoteState
├── newState: NoteState
├── layoutImpact: LayoutImpactLevel
├── triggeredBy: UserID
└── timestamp: Date
```

**Event Handling Patterns:**
- Asynchronous event processing with retry capability
- Event ordering preservation for sequential operations
- Event aggregation for batch operations
- Cross-platform event synchronization
- Event persistence for audit trails and undo operations

#### Layout and Pagination Events
**Purpose**: Coordination of layout-related changes and recalculations

**Event Definitions:**
```
LayoutPreferenceChangedEvent
├── entityId: UUID
├── entityType: EntityType
├── previousPreference: LayoutPreference?
├── newPreference: LayoutPreference
├── affectedScope: LayoutScope
├── recalculationRequired: Bool
├── triggeredBy: UserID
└── timestamp: Date

PaginationRecalculatedEvent
├── documentId: UUID
├── affectedPages: [UUID]
├── previousPageCount: Int
├── newPageCount: Int
├── changedTuneLines: [TuneLineChange]
├── recalculationTrigger: RecalculationTrigger
├── calculationDuration: TimeInterval
└── timestamp: Date

SystemLayoutUpdatedEvent
├── systemId: UUID
├── previousLayout: SystemLayout?
├── newLayout: SystemLayout
├── layoutChanges: [LayoutChange]
├── affectedMeasures: [UUID]
├── cacheInvalidationRequired: [UUID]
└── timestamp: Date

CacheInvalidatedEvent
├── invalidatedEntities: [UUID]
├── invalidationType: InvalidationType
├── invalidationScope: InvalidationScope
├── cascadeEffects: [CascadeEffect]
├── recalculationEstimate: TimeInterval
└── timestamp: Date
```

**Layout Event Coordination:**
- Dependency-aware event ordering for layout calculations
- Batch event processing for performance optimization
- Progressive layout updates with user feedback
- Cache invalidation coordination across event types
- Performance impact monitoring and optimization

### 3.5.2 Audio and Practice Events

#### Practice Session Events
**Purpose**: Tracking and coordination of practice-related activities

**Event Definitions:**
```
PracticeSessionStartedEvent
├── sessionId: UUID
├── scoreId: UUID
├── tuneId: UUID?
├── practiceGoals: [PracticeGoal]
├── sessionSettings: PracticeSettings
├── startedBy: UserID
└── timestamp: Date

RecordingCapturedEvent
├── recordingId: UUID
├── sessionId: UUID
├── scoreId: UUID
├── tuneId: UUID?
├── duration: TimeInterval
├── audioQuality: AudioQualityMetrics
├── linkedToScore: Bool
└── timestamp: Date

IntonationAnalyzedEvent
├── analysisId: UUID
├── recordingId: UUID
├── targetPitches: [Pitch]
├── detectedPitches: [Pitch]
├── accuracyMetrics: IntonationMetrics
├── improvementSuggestions: [ImprovementSuggestion]
└── timestamp: Date

MetronomeSessionEvent
├── sessionId: UUID
├── tempo: BPM
├── timeSignature: TimeSignature
├── duration: TimeInterval
├── accuracyMetrics: TimingMetrics?
└── timestamp: Date
```

#### Audio Processing Events
**Purpose**: Coordination of audio analysis and processing operations

**Event Definitions:**
```
AudioProcessingStartedEvent
├── processingId: UUID
├── audioData: AudioDataReference
├── processingType: AudioProcessingType
├── estimatedDuration: TimeInterval
└── timestamp: Date

AudioAnalysisCompletedEvent
├── analysisId: UUID
├── processingId: UUID
├── analysisResults: AudioAnalysisResults
├── processingDuration: TimeInterval
├── qualityMetrics: AudioQualityMetrics
└── timestamp: Date

AudioCompressionCompletedEvent
├── compressionId: UUID
├── originalSize: FileSize
├── compressedSize: FileSize
├── compressionRatio: Float
├── qualityLoss: AudioQualityImpact
└── timestamp: Date
```

### 3.5.3 Synchronization and Collaboration Events

#### Cloud Synchronization Events
**Purpose**: Coordination of cloud storage operations and conflict resolution

**Event Definitions:**
```
CloudSyncStartedEvent
├── syncId: UUID
├── documentId: UUID
├── syncDirection: SyncDirection
├── cloudProvider: CloudProvider
├── expectedDuration: TimeInterval
└── timestamp: Date

CloudSyncCompletedEvent
├── syncId: UUID
├── syncResult: SyncResult
├── conflictsDetected: [SyncConflict]
├── bytesTransferred: Int
├── syncDuration: TimeInterval
└── timestamp: Date

SyncConflictDetectedEvent
├── conflictId: UUID
├── documentId: UUID
├── conflictType: ConflictType
├── localVersion: EntityVersion
├── remoteVersion: EntityVersion
├── requiresUserIntervention: Bool
└── timestamp: Date

ConflictResolvedEvent
├── conflictId: UUID
├── resolutionStrategy: ResolutionStrategy
├── resolvedBy: UserID?
├── resolutionResult: ConflictResolutionResult
├── auditTrailCreated: Bool
└── timestamp: Date
```

#### Collaborative Editing Events
**Purpose**: Real-time coordination for multi-user editing scenarios

**Event Definitions:**
```
CollaborativeSessionStartedEvent
├── sessionId: UUID
├── documentId: UUID
├── participants: [UserID]
├── sessionSettings: CollaborationSettings
├── lockingStrategy: LockingStrategy
└── timestamp: Date

UserJoinedSessionEvent
├── sessionId: UUID
├── userId: UserID
├── userRole: CollaborationRole
├── joinPermissions: [Permission]
└── timestamp: Date

ConcurrentEditDetectedEvent
├── sessionId: UUID
├── editingUsers: [UserID]
├── conflictingOperations: [EditOperation]
├── autoResolutionAttempted: Bool
├── requiresModeration: Bool
└── timestamp: Date

SessionLockAcquiredEvent
├── sessionId: UUID
├── lockId: UUID
├── entityId: UUID
├── lockType: LockType
├── acquiredBy: UserID
├── lockDuration: TimeInterval?
└── timestamp: Date
```

### 3.5.4 System and Performance Events

#### Performance Monitoring Events
**Purpose**: System performance tracking and optimization guidance

**Event Definitions:**
```
PerformanceThresholdExceededEvent
├── operationType: OperationType
├── actualDuration: TimeInterval
├── thresholdDuration: TimeInterval
├── performanceImpact: PerformanceImpact
├── optimizationSuggestions: [OptimizationSuggestion]
└── timestamp: Date

CacheEfficiencyEvent
├── cacheType: CacheType
├── hitRatio: Float
├── missCount: Int
├── totalRequests: Int
├── recommendedActions: [CacheRecommendation]
└── timestamp: Date

MemoryPressureEvent
├── currentMemoryUsage: MemoryUsage
├── memoryPressureLevel: MemoryPressureLevel
├── triggeringOperation: OperationType?
├── recommendedActions: [MemoryOptimization]
└── timestamp: Date

LayoutCalculationPerformanceEvent
├── calculationType: LayoutCalculationType
├── entityCount: Int
├── calculationDuration: TimeInterval
├── cacheUtilization: CacheUtilization
├── performanceRating: PerformanceRating
└── timestamp: Date
```

#### Error and Recovery Events
**Purpose**: Error tracking and automatic recovery coordination

**Event Definitions:**
```
DomainErrorOccurredEvent
├── errorId: UUID
├── errorType: DomainErrorType
├── entityId: UUID?
├── errorSeverity: ErrorSeverity
├── errorContext: ErrorContext
├── recoveryAttempted: Bool
├── userNotificationRequired: Bool
└── timestamp: Date

AutoRecoveryAttemptedEvent
├── errorId: UUID
├── recoveryStrategy: RecoveryStrategy
├── recoveryResult: RecoveryResult
├── dataIntegrityPreserved: Bool
├── userInterventionRequired: Bool
└── timestamp: Date

DataIntegrityValidationEvent
├── validationId: UUID
├── entityId: UUID
├── validationType: ValidationType
├── validationResult: ValidationResult
├── integrityIssues: [IntegrityIssue]
├── autoRepairAttempted: Bool
└── timestamp: Date
```

### 3.5.5 Event Processing Patterns

#### Event Handler Registration
**Purpose**: Flexible event subscription and processing coordination

**Registration Patterns:**
```
EventSubscription
├── eventType: EventType
├── handler: EventHandler
├── priority: EventPriority
├── processingMode: ProcessingMode (sync/async)
├── errorHandling: ErrorHandlingStrategy
├── retryPolicy: RetryPolicy?
└── filterCriteria: EventFilterCriteria?

EventHandlerChain
├── primaryHandler: EventHandler
├── secondaryHandlers: [EventHandler]
├── chainBreakConditions: [BreakCondition]
├── aggregationRules: AggregationRules?
└── failureEscalation: EscalationPolicy
```

#### Cross-Platform Event Coordination
**Purpose**: Consistent event processing across different platform implementations

**Coordination Patterns:**
- Event serialization for cross-platform compatibility
- Platform-specific event adapter implementations
- Event ordering preservation across async boundaries
- Event persistence for offline scenario handling
- Event replay capability for synchronization recovery

**Performance Optimization:**
- Event batching for high-frequency operations
- Event filtering to reduce processing overhead
- Priority-based event processing queues
- Memory-efficient event data structures
- Background event processing with user notification


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

### Section 4.3 Pagination Data Management

#### 4.3.1 Caching Strategy
**Layout Cache Structure:**
- Page-level layout snapshots
- System-level measurement cache
- Font metrics cache per size/style
- Dependency invalidation tracking
- **Paper size dimension cache**
- **Cross-platform paper size conversion tables**
- **Pages[] assignment cache for quick lookup**
- **TuneLines to page mapping cache**

**Persistence Strategy:**
- Transient cache for active session
- Optional layout cache persistence for large documents
- Platform-specific storage optimization
- Memory pressure handling
- **Efficient Pages[] structure serialization**

#### 4.3.2 Change Detection Infrastructure
**Attribute Monitoring:**
- Deep property change detection
- Cascade effect analysis
- Change event aggregation
- Performance impact assessment
- **Orientation change detection and validation**
- **Cross-tune dependency tracking**
- **Paper size change detection and global impact analysis**
- **Regional paper size preference monitoring**
- **Pages[] structure change tracking**

**Event Sourcing:**
- Track all layout-affecting changes
- Enable sophisticated undo/redo operations
- Support collaborative editing scenarios
- Audit trail for debugging complex layouts
- **Pages[] structure versioning for rollback support**

## 5. Presentation Layer Architecture

### 5.1 Platform-Specific UI Patterns

#### 5.1.1 iOS/macOS - SwiftUI with Swift 6 Concurrency

**Modern Architecture Pattern:**
- **MVVM with Observation Framework**: @Observable classes replacing ObservableObject for better performance
- **Repository Pattern**: Protocol-based repositories with sendable Core Data implementations
- **Coordinator Pattern**: Navigation flow management with type-safe routing and actor isolation
- **Swift Concurrency Integration**: Actor-based data flows with MainActor UI updates and background processing

**Swift 6 State Management:**
- **@Observable**: New observation system for automatic UI updates with better performance than @Published
- **@State**: Enhanced state management with improved compiler optimization
- **Sendable Conformance**: Full data race safety with sendable ViewModels and data structures
- **Actor Isolation**: MainActor for UI updates, background actors for heavy processing
- **Strict Concurrency**: Complete data race elimination through Swift 6's enhanced type system

**Advanced Concurrency Patterns:**
- **TaskGroup Operations**: Parallel processing for pagination calculations and audio analysis
- **AsyncSequence Integration**: Real-time audio data streaming and cloud synchronization updates
- **Actor-based Repositories**: Thread-safe data access with isolated actor boundaries
- **Structured Concurrency**: Proper task lifecycle management with automatic cancellation
- **Custom AsyncSequence**: Musical event streaming for real-time collaboration features

**SwiftUI Enhanced Features:**
- **Hierarchical State**: Parent-child state relationships with proper sendable boundaries
- **Environment Injection**: Shared services via environment with sendable requirements
- **Custom Observation**: Domain-specific observation patterns for musical state changes
- **ViewModifiers with Actors**: Reusable SMuFL symbol styling with thread-safe calculations
- **Async View Loading**: Concurrent image and notation loading with structured concurrency

**Platform-Specific Swift 6 Adaptations:**
- **iOS Navigation**: NavigationStack with async deep linking and sendable route parameters
- **macOS Navigation**: NavigationSplitView with actor-isolated sidebar state management
- **Toolbar Management**: Sendable toolbar configurations with async state updates
- **Keyboard Shortcuts**: Thread-safe keyboard command handling with MainActor coordination
- **Gesture Recognition**: Concurrent gesture processing with proper actor isolation

**Core Data with Swift 6:**
- **NSPersistentCloudKitContainer**: Actor-isolated Core Data operations with sendable model objects
- **Background Processing**: Structured concurrency for Core Data operations with proper context isolation
- **Batch Operations**: Concurrent bulk operations using TaskGroup for large score processing
- **Sendable Models**: Core Data model objects designed for safe cross-actor usage
- **Migration Strategy**: Async migration processes with progress reporting through AsyncSequence

**Performance and Safety:**
- **Complete Data Race Safety**: Swift 6's strict concurrency eliminates all data races at compile time
- **Improved Performance**: @Observable framework provides better performance than Combine-based solutions
- **Memory Safety**: Enhanced memory management with improved ARC and actor isolation
- **Compile-Time Guarantees**: Static analysis ensures thread safety without runtime overhead
- **Debugging Improvements**: Better debugging experience with Swift 6's enhanced concurrency tools

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

### Section 5.3 Dynamic Pagination Architecture

#### 5.3.1 iOS/macOS - Swift 6 Pagination with Actor Isolation

**Actor-Based Layout Engine:**
- **PaginationActor**: Isolated actor managing all pagination calculations with sendable data structures
- **MainActor Integration**: UI updates coordinated through MainActor with structured concurrency
- **AsyncSequence Changes**: Layout attribute changes streamed through custom AsyncSequence for debouncing
- **TaskGroup Processing**: Parallel pagination calculations using structured concurrency for performance
- **Cancellation Support**: Proper task cancellation when layout parameters change during processing

**Modern SwiftUI Integration:**
- **@Observable Layout State**: New observation framework for pagination state with automatic UI updates
- **Sendable Layout Models**: Thread-safe layout models that can be safely passed between actors
- **Custom Layout Protocol**: iOS 16+ Layout conformance with actor-safe measurement and arrangement
- **AsyncSequence UI Updates**: Real-time layout updates streamed to UI through AsyncSequence
- **Structured View Updates**: Coordinated view updates using Swift 6's enhanced concurrency model

**Enhanced Performance Architecture:**
- **Background Pagination Actor**: Heavy calculations performed on background actor with sendable results
- **Memory-Safe Caching**: Actor-isolated layout cache with automatic cleanup and thread safety
- **Concurrent Measurement**: Parallel measurement of musical elements using TaskGroup operations
- **Optimized State Changes**: Minimal state updates through Swift 6's enhanced observation system
- **Resource Management**: Automatic memory management with improved ARC and actor isolation

**Thread-Safe Layout Coordination:**
- **Sendable Configuration**: All layout configuration objects conform to Sendable for safe actor usage
- **Actor-Isolated Repositories**: Layout preference storage through actor-isolated repository pattern
- **Cross-Actor Communication**: Safe data passing between pagination actor and UI layer
- **Deadlock Prevention**: Swift 6's actor system eliminates potential deadlocks in layout calculations
- **Atomic Updates**: Layout changes applied atomically to prevent intermediate invalid states

**Advanced Animation Coordination:**
- **MainActor Animations**: All animation triggers coordinated through MainActor for UI thread safety
- **Async Animation Completion**: Animation completion handled through async/await patterns
- **Concurrent Preparation**: Animation preparation work performed concurrently while maintaining UI responsiveness
- **State Synchronization**: Layout state and animation state kept synchronized through actor isolation
- **Smooth Transitions**: Enhanced transition smoothness through Swift 6's improved concurrency performance

**CloudKit Integration with Actors:**
- **CloudKit Actor**: Dedicated actor for cloud synchronization with sendable data models
- **Conflict Resolution**: Thread-safe conflict resolution through structured concurrency
- **Offline State Management**: Actor-isolated offline state with automatic synchronization
- **Cross-Device Coordination**: Sendable pagination preferences synchronized across devices
- **Background Sync**: CloudKit operations performed on background actors with UI updates on MainActor

#### 5.3.2 Android - Compose Implementation  
**State Observation**: StateFlow monitoring of layout-affecting properties
**Layout Calculation**: Compose layout phase integration
**Performance**: Coroutine-based background processing
**State Hoisting**: Pagination state and Pages[] management in ViewModel layer
**Animation**: Compose animations for smooth pagination updates

#### 5.3.3 Windows - WinUI 3 Implementation
**Property Change Detection**: DependencyProperty change callbacks
**Layout System**: Integration with WinUI layout cycle
**Performance**: Task-based async recalculation
**MVVM Integration**: RelayCommand for pagination operations and Pages[] updates
**Animation**: Composition animations for layout transitions

#### 5.3.4 Linux - Qt Implementation
**Signal-Slot Architecture**: Property change signal emission
**Layout Management**: QLayout system integration
**Performance**: QThread-based background processing
**Model Updates**: QAbstractItemModel change notifications for Pages[] structure
**Animation**: Qt Animation Framework for smooth updates

## 6. Audio Processing Architecture

### 6.1 Platform-Specific Audio Engines

#### 6.1.1 iOS/macOS - Professional Audio Architecture with Swift 6

**AVAudioEngine with Actor Isolation:**
- **AudioEngine Actor**: Dedicated actor for all audio processing with sendable audio data structures
- **Hardware Sample Rate Matching**: Actor-isolated adaptation to device capabilities (44.1kHz, 48kHz, 96kHz)
- **Buffer Management**: Thread-safe buffer operations with sendable PCM buffer wrappers
- **Audio Unit Chain**: Actor-coordinated audio unit graph with structured concurrency for real-time processing
- **Spatial Audio Integration**: AVAudioEnvironmentNode management through dedicated audio actor

**Swift 6 Concurrency for Real-Time Audio:**
- **MainActor UI Updates**: Audio state changes coordinated through MainActor for thread-safe UI updates
- **Background Audio Processing**: Heavy audio analysis performed on background actors with sendable results
- **AsyncSequence Audio Streams**: Real-time audio data streaming through custom AsyncSequence implementations
- **TaskGroup Analysis**: Parallel audio analysis using structured concurrency for pitch detection and waveform processing
- **Cancellation Support**: Proper task cancellation for audio operations when user interrupts or changes settings

**Thread-Safe Audio State Management:**
- **@Observable Audio Models**: Audio state management using Swift 6's observation framework for automatic UI updates
- **Sendable Audio Configuration**: All audio settings and state objects conform to Sendable for safe actor usage
- **Actor-Isolated Session Management**: AVAudioSession management through dedicated actor with proper lifecycle handling
- **Cross-Actor Audio Communication**: Safe audio data passing between processing actors and UI layer
- **Memory-Safe Audio Buffers**: Enhanced memory management for audio buffers with automatic cleanup

**Enhanced Audio Analysis:**
- **Concurrent FFT Processing**: Parallel frequency analysis using TaskGroup operations with vDSP and Accelerate framework
- **Actor-Isolated ML Models**: Core ML audio models managed through dedicated actors for thread safety
- **Real-Time Visualization**: Metal-rendered audio visualization with data safely passed from audio actors to MainActor
- **Background Analysis Continuation**: Continued audio processing during device lock using structured concurrency
- **Performance Monitoring**: Built-in performance monitoring with Swift 6's enhanced debugging capabilities

#### 6.1.2 Android Audio Implementation
**Oboe Integration**: AAudio backend for minimal latency on supported devices
**Coroutine-Based Processing**: Kotlin coroutines for audio analysis and processing
**StateFlow Audio State**: Audio state management through reactive streams
**Background Processing**: Audio operations on background dispatchers with UI updates on Main

#### 6.1.3 Windows Audio Implementation
**WASAPI Integration**: Windows Audio Session API for professional audio
**Task-Based Processing**: .NET Task and async/await for audio operations
**Observable Audio State**: Audio state management through observable properties
**Thread Coordination**: Proper thread coordination between audio and UI threads

#### 6.1.4 Linux Audio Implementation
**ALSA/PulseAudio**: Direct audio system integration for low latency
**Thread Management**: Platform-specific threading for audio processing
**Signal-Slot Communication**: Qt signals for audio state communication
**Background Processing**: Dedicated audio processing threads
**Format Support**: Platform-optimized codecs (AAC, OGG, FLAC)

### 6.2 Audio Processing Features

#### 6.2.1 Metronome System
**Timing Accuracy**: Sample-accurate timing using audio callbacks
**Time Signature Support**: Complex meters with accent patterns
**Sound Customization**: Different tones for downbeats and subdivisions
**Visual Synchronization**: UI updates synchronized with audio beats

#### 6.2.2 Recording and Analysis
**Practice Session Recording**: Automatic recording with metadata
**Intonation Analysis**: Real-time pitch detection for tuning assistance
**Performance Metrics**: Timing accuracy and rhythm analysis
**Audio Quality**: Professional recording standards (44.1kHz+, 16-bit+)

### Section 6.3 Pagination Performance Architecture 

#### 6.3.1 Performance Optimization Strategies

**Incremental Recalculation:**
- Only recalculate affected page ranges (except for Global triggers like paper size)
- Preserve unaffected Pages[] assignments when possible
- Cache intermediate layout results
- Minimize full document relayout events
- **Full Document Recalculation**: Required for paper size and global changes
- **Smart Pages[] Updates**: Only rebuild affected page assignments

**Smart Batching:**
- Combine multiple rapid changes into single recalculation
- Debounce user input to prevent excessive calculations
- Queue non-critical pagination updates
- Prioritize visible page recalculation
- **Batch Pages[] structure updates** for performance

**Memory Management:**
- Lazy loading of off-screen page layouts
- Intelligent caching of layout calculations
- Garbage collection of unused pagination data
- Platform-specific memory optimization
- **Efficient Pages[] structure management**

#### 6.3.2 Musical Intelligence

**Phrase-Aware Page Breaks:**
- Analyze musical structure for optimal break points
- Avoid breaking within ornament sequences
- Preserve measure groupings where possible
- Maintain part continuity across pages
- **Respect orientation-mandated page breaks while optimizing musical flow**
- **Balance partially-filled pages caused by orientation changes**
- **Optimize TuneLines distribution across Pages[]**

**System Break Intelligence:**
- Balance system density across pages
- Optimize staff spacing for readability
- Respect instrument groupings
- Handle multi-instrument alignment
- **Accommodate orientation-specific page dimensions**
- **Optimize system count for different orientations**
- **Adapt system layout for different paper sizes**
- **Maintain proportional spacing across paper size changes**
- **Intelligent TuneLines assignment to maximize page utilization**

**User Experience Preservation:**
- Maintain current view context during recalculation
- Preserve user selection across pagination changes
- Smooth animation between layout states
- Undo/redo support for pagination-affecting changes
- **Maintain page navigation consistency during Pages[] updates**

## 7. SMuFL Music Notation System

### 7.1 SMuFL Implementation Strategy

#### 7.1.1 iOS/macOS - Advanced Typography with Swift 6 Integration

**Actor-Safe Typography System:**
- **Typography Actor**: Dedicated actor for font loading and glyph calculations with sendable font descriptors
- **Thread-Safe Font Management**: CTFont integration with actor isolation for safe concurrent font operations
- **Sendable Glyph Models**: Font metrics and glyph bounds wrapped in sendable structures for cross-actor usage
- **Concurrent Font Loading**: Parallel font variant loading using TaskGroup operations
- **Memory-Safe Font Caching**: Actor-isolated font cache with automatic cleanup and thread safety

**Enhanced Text Kit 2 Integration:**
- **MainActor Text Layout**: All text layout operations coordinated through MainActor for UI thread safety
- **Background Measurement**: Heavy typography calculations performed on background actors with sendable results
- **AsyncSequence Typography Updates**: Dynamic font changes streamed through custom AsyncSequence
- **Structured Text Processing**: Text processing operations using structured concurrency for better performance
- **Safe Baseline Calculations**: Thread-safe musical baseline alignment with actor-isolated font metrics

**Metal-Accelerated Rendering with Concurrency:**
- **Rendering Actor**: GPU-accelerated glyph rendering managed through dedicated actor
- **Concurrent Shader Compilation**: Parallel shader compilation using TaskGroup for complex musical symbols
- **Thread-Safe Texture Management**: Actor-isolated texture cache with sendable texture descriptors
- **Background Glyph Generation**: Symbol texture generation performed concurrently with UI updates on MainActor
- **Memory-Safe GPU Resources**: Enhanced GPU memory management with Swift 6's improved resource handling

**SwiftUI Text Integration with @Observable:**
- **@Observable Text Models**: Typography state management using Swift 6's observation framework
- **Sendable Text Attributes**: All text styling and positioning data conform to Sendable for safe actor usage
- **Actor-Coordinated Text Selection**: Thread-safe text selection handling with proper MainActor coordination
- **Concurrent Pasteboard Operations**: Background pasteboard processing with UI updates on MainActor
- **Dynamic Type with Actors**: Accessibility-compliant symbol scaling with actor-isolated calculations

#### 7.1.2 Cross-Platform SMuFL Standards
**Unicode Compliance**: Full SMuFL specification adherence across all platforms
**Scaling System**: Proportional scaling based on staff size with platform optimization
**Anti-aliasing**: Smooth rendering at all display densities using platform-specific techniques
**Color Support**: Configurable symbol colors with platform-appropriate color management

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
**iOS/macOS**: XCTest framework with Swift 6 concurrency testing and actor isolation verification
**Android**: JUnit 5 with MockK for Kotlin-specific testing
**Windows**: xUnit with Moq for .NET testing
**Linux**: Google Test (C++) or built-in Rust testing

#### 10.1.2 Integration Testing
**Audio System Testing**: Latency measurement and quality verification
**Cloud Integration Testing**: Multi-provider synchronization validation
**Export Testing**: Cross-platform format consistency verification
**Localization Testing**: Cultural adaptation and translation accuracy

**Pagination-Specific Testing**
**Unit Tests:**
- Layout calculation accuracy
- Trigger detection reliability
- Performance benchmarking
- Musical phrase preservation
- **Paper size conversion accuracy**
- **Cross-platform paper size consistency**
- **Regional default validation**
- **Pages[] structure integrity validation**
- **TuneLines assignment accuracy**

**Integration Tests:**
- Cross-platform layout consistency
- Cloud synchronization with pagination changes
- Export accuracy after dynamic pagination
- Accessibility during layout transitions
- **Paper size change synchronization across devices**
- **Export format compatibility with different paper sizes**
- **Regional settings impact on pagination**
- **Pages[] structure synchronization and conflict resolution**

**Performance Tests:**
- Large document pagination performance
- Rapid change handling capability
- Memory usage during recalculation
- Platform-specific optimization validation
- **Pages[] structure rebuild performance**
- **TuneLines redistribution efficiency**

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
**iOS**: iOS 17.0+, iPhone 13+ recommended, 2GB free storage, A14 Bionic or newer
**macOS**: macOS 14.0+, M1 or Intel Core i7+, 2GB free storage, 8GB RAM recommended
**Android**: API 28+, 6GB RAM, 2GB free storage, ARMv8-A or x86_64 architecture
**Windows**: Windows 11 22H2+, 8GB RAM, 2GB free storage, modern CPU with hardware acceleration
**Linux**: Modern distribution with Wayland/X11, 8GB RAM, 2GB free storage, OpenGL 3.3+ support

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
**Team Structure**: Specialized teams for each platform with shared architecture coordination
**Shared Domain Logic**: Common business rules implemented consistently across platforms
**Code Reviews**: Cross-platform consistency validation with architectural compliance checks
**Integration Points**: Regular synchronization of domain changes with specification updates

#### 13.1.2 Quality Assurance
**Continuous Integration**: Automated builds and testing for all platforms
**Device Testing**: Physical device testing across target hardware
**Beta Testing**: Closed beta with pipe band communities
**Performance Monitoring**: Real-time performance and crash analytics

### 13.2 Release Strategy

#### 13.2.1 Phased Rollout
**Phase 1**: Core functionality with basic score editing and SMuFL rendering
**Phase 2**: Advanced notation features with cloud synchronization and collaboration
**Phase 3**: Audio features with recording, metronome, and analysis capabilities
**Phase 4**: Community features with sharing, marketplace, and social integration

#### 13.2.2 Platform Distribution
**iOS**: App Store with TestFlight beta distribution and enterprise deployment options
**macOS**: Mac App Store and direct distribution with notarization and Gatekeeper compliance
**Android**: Google Play Store with Play Console beta and alternative store support
**Windows**: Microsoft Store and direct installer with code signing and SmartScreen compatibility
**Linux**: Package managers (APT, RPM, AUR) and universal formats (AppImage, Flatpak, Snap)

### 13.3 Repository Architecture

#### 13.3.1 iOS/macOS Repository Architecture with Swift 6

**Modern Swift Package Architecture:**
- **Swift 6 Workspace Structure**: Single workspace with iOS app target, macOS app target, and Swift 6 package targets
- **Sendable Domain Models**: All shared domain logic implemented with Sendable conformance for thread safety
- **Actor-Isolated Repositories**: Repository implementations using actor isolation for safe concurrent data access
- **Structured Concurrency Integration**: All async operations using Swift 6's structured concurrency patterns
- **Platform-Optimized Targets**: Separate app targets with Swift 6 concurrency optimizations for each platform

**Enhanced Clean Architecture:**
- **Domain Package**: Pure Swift 6 package with sendable entities, actor-based use cases, and protocol repositories
- **Data Package**: Actor-isolated Core Data operations with sendable model transformations and CloudKit integration
- **Presentation iOS**: SwiftUI with @Observable ViewModels and MainActor coordination for iOS-specific features
- **Presentation macOS**: SwiftUI/AppKit integration with actor-safe state management and macOS-specific UI patterns
- **Audio Package**: Actor-isolated AVAudioEngine operations with sendable audio data models and cross-platform optimization

**Swift 6 Dependency Management:**
- **Package.swift with Swift 6**: Updated package manifests targeting Swift 6 language mode
- **Sendable Dependencies**: All package dependencies verified for Sendable conformance and actor safety
- **Concurrency-Safe APIs**: Internal package APIs designed with Swift 6 concurrency in mind
- **Testing with Actors**: Comprehensive test suites using Swift 6's testing improvements and actor isolation
- **CI/CD Swift 6**: Build pipelines configured for Swift 6 compilation and concurrency checking

**Thread-Safe Code Sharing:**
- **100% Sendable Shared**: Business logic, domain entities, and use cases with complete Sendable conformance
- **Actor-Isolated Shared**: Audio processing, Core Data operations, and cloud sync with proper actor boundaries
- **Platform-Adapted**: Navigation and layout patterns adapted for each platform's concurrency requirements
- **Platform-Specific**: UI conventions and platform APIs with MainActor coordination where appropriate
- **Concurrency Testing**: Shared business logic tested for thread safety with Swift 6's enhanced testing tools

**Development Workflow with Swift 6:**
- **Strict Concurrency Mode**: All targets compiled with complete concurrency checking enabled
- **Actor Isolation Reviews**: Code review process includes verification of proper actor isolation
- **Performance Profiling**: Swift 6-optimized performance testing using latest Instruments capabilities
- **Migration Strategy**: Incremental adoption of Swift 6 features with proper deprecation handling
- **Cross-Platform Consistency**: Identical concurrency patterns across iOS and macOS implementations

#### 13.3.2 Other Platform Repositories

**Android Repository**
- Standalone Android Studio project with Kotlin implementation
- Independent domain logic implementation following Clean Architecture principles
- Android-specific Room database and Jetpack Compose UI
- No code sharing dependencies with other platforms

**Windows Repository**
- Standalone Visual Studio solution with C#/.NET implementation
- Independent domain logic implementation following Clean Architecture principles
- WinUI 3 presentation layer with Entity Framework Core data persistence
- No code sharing dependencies with other platforms

**Linux Repository**
- Standalone repository with Qt/C++ or Rust/Tauri implementation
- Independent domain logic implementation following Clean Architecture principles
- Platform-specific UI and data storage solutions
- No code sharing dependencies with other platforms

#### 13.3.2 Cross-Platform Consistency Strategy

**Domain Logic Parity**: Each platform implements identical business rules and use cases using platform-native languages and patterns, ensuring behavioral consistency without code sharing.

**Specification-Driven Development**: This architectural specification serves as the single source of truth for consistent implementation across all repositories.

**Interface Contracts**: Standardized API contracts and data formats ensure interoperability where needed (exports, cloud sync) without requiring shared codebases.

**Independent Evolution**: Each platform repository can evolve independently while maintaining architectural and functional parity through specification adherence.

This approach maximizes platform-native optimization while maintaining consistency through disciplined architectural alignment rather than code sharing.

## 14. Future Enhancements

### 14.1 Advanced Features Roadmap

#### 14.1.1 Phase 2 Enhancements
**Real-time Collaboration**: Multiple users editing simultaneously with operational transformation and conflict resolution
**Advanced Audio Analysis**: Pitch correction, timing feedback, and performance evaluation with machine learning
**AI Practice Assistant**: Intelligent practice recommendations based on performance analysis and learning patterns
**Video Integration**: Record practice sessions with synchronized score display and multi-angle recording

#### 14.1.2 Phase 3 Enhancements
**Hardware Integration**: MIDI controller support and electronic instrument connectivity
**AR/VR Support**: Augmented reality music stand with spatial audio and immersive practice environments
**Community Features**: Score sharing marketplace with rating system and collaborative editing
**Competition Integration**: Direct integration with pipe band competition systems and adjudication tools


### 14.2 Platform Evolution

#### 14.2.1 Emerging Platforms
**Web Platform**: Progressive Web App for cross-platform accessibility with WebAssembly audio processing
**Smart TV**: Large screen display for band practice sessions with wireless device connectivity
**Wearable Devices**: Practice tracking and metronome on smartwatches with haptic feedback
**IoT Integration**: Wireless metronome synchronization across band with mesh networking

#### 14.2.2 Technology Evolution
**Machine Learning**: Advanced audio analysis and practice feedback with on-device processing
**Cloud Computing**: Server-side audio processing and analysis with edge computing integration
**Blockchain**: Decentralized score sharing and copyright protection with smart contracts
**5G Integration**: Real-time collaboration over high-speed mobile networks with ultra-low latency


## 15. Conclusion

This Clean Architecture specification provides a comprehensive foundation for developing a professional Scottish pipe band application across multiple platforms using native technologies. The architecture ensures:

**Maintainability**: Clear separation of concerns with platform-agnostic domain logic
**Scalability**: Modular design supporting feature additions and platform extensions
**Performance**: Native platform optimization for audio processing and UI rendering
**User Experience**: SMuFL-compliant notation with engaging defaults for pipe band music
**Global Accessibility**: Comprehensive internationalization for worldwide pipe band communities
**Data Sovereignty**: User control over data storage with flexible cloud integration

**Key Architectural Benefits:**

**Platform-Native Excellence**: Each platform uses its optimal technologies and patterns for best performance and user experience while maintaining architectural consistency
**Universal Domain Logic**: Business rules implemented consistently across all platforms while leveraging platform-specific optimizations and concurrency models
**Offline-First Design**: Complete functionality without network dependency, ensuring reliability in any location or connectivity situation
**Professional Music Notation**: SMuFL-compliant rendering delivering publication-quality scores with proper typography and musical intelligence
**Flexible Cloud Integration**: Support for any cloud provider through platform-native file system integration without vendor lock-in
**Cultural Sensitivity**: Thoughtful internationalization respecting global pipe band traditions and regional preferences

**Swift 6 Integration**: The iOS/macOS implementation leverages Swift 6's complete concurrency model for thread-safe audio processing, layout calculations, and real-time collaboration features, providing industry-leading performance and reliability.

The specification addresses real-world needs of the international pipe band community, from Highland Games to European competitions, ensuring musicians can practice, collaborate, and perform at the highest level regardless of their location or preferred platform. The offline-first approach combined with professional notation rendering ensures reliability and quality that meets the demanding standards of competitive pipe band music.

This architecture will deliver a world-class application that serves the global pipe band community while maintaining the highest standards of software engineering, user experience, and musical authenticity.
