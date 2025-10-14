# Scottish Pipe Band Application - Core Architecture Specification

**Version:** 1.0  
**Last Updated:** September 30, 2025  
**Status:** Foundation Specification

---

## Document Purpose and Scope

This document defines the **platform-agnostic architectural foundation** for a Scottish pipe band music notation application. It specifies the domain model, business logic, and architectural patterns that must be consistently implemented across all platform-specific implementations (iOS, macOS, Android, Windows, Linux).

**What This Document Contains:**
- Clean Architecture layer definitions
- Domain entities and business rules
- Musical notation specifications (SMuFL compliance)
- Use cases and repository interfaces
- Domain services and validation rules
- Universal file format specifications
- Cross-platform architectural patterns

**What This Document Does NOT Contain:**
- Platform-specific implementation details (Swift, Kotlin, C#, etc.)
- UI framework specifications (SwiftUI, Compose, WinUI, Qt)
- Platform-specific storage implementations
- App store submission requirements
- Platform-specific audio processing details

**Related Documents:**
- `ux_interaction_design.md` - UX specification for interacting with scores
- `ios_macos_implementation.md` - iOS/macOS implementation with App Store compliance
- `android_implementation.md` - Android implementation with Play Store guidelines
- `windows_implementation.md` - Windows implementation with Microsoft Store requirements
- `linux_implementation.md` - Linux implementation with package manager distribution

---

## 1. Executive Summary

This specification outlines a cross-platform application for Scottish pipe band musicians, built using **platform-native languages** and **Clean Architecture** principles. The app targets bagpipes, bass drums, tenor drums, and snare drums with offline-first functionality and SMuFL-compliant music notation rendering across macOS, iOS, Android, Windows, and Linux platforms.

### 1.1 Target Instruments
- **Bagpipes:** Great Highland Bagpipe with traditional embellishments
- **Bass Drum:** Single unpitched percussion with hand notation
- **Tenor Drum:** Multi-drum pitched percussion (4-6 drums typical)
- **Snare Drum:** Unpitched percussion with rudiment support and hand notation

### 1.2 Core Design Principles

**Offline-First Architecture:**
- Complete functionality without network connectivity
- Local-first data storage with optional cloud synchronization
- Graceful degradation when cloud services unavailable

**Cultural Authenticity:**
- Authentic Scottish pipe band notation standards
- Traditional Gaelic terminology preservation
- Regional style variations (Highland, Border, competition vs. traditional)
- Educational support for non-native practitioners

**Professional Quality:**
- SMuFL-compliant music notation rendering
- Publication-quality score output
- Professional audio processing capabilities
- Competition-ready score preparation

**Universal Accessibility:**
- Cross-platform file format compatibility
- Multi-language support with cultural sensitivity
- Accessibility compliance for screen readers and assistive technologies
- Flexible cloud provider integration

---

## 2. Clean Architecture Overview

### 2.1 Architectural Layers

The application follows Clean Architecture principles with clear separation of concerns across four distinct layers:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Presentation Layer                          │
│  (Platform-Specific: SwiftUI, Compose, WinUI, Qt)               │
│  - UI Components & Rendering                                    │
│  - User Interaction Handling                                    │
│  - Platform Graphics APIs                                       │
└─────────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                          │
│  (Use Cases & Application Services)                             │
│  - Score Management Use Cases                                   │
│  - Musical Editing Operations                                   │
│  - Export and Import Services                                   │
│  - Audio Processing Coordination                                │
└─────────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Domain Layer                              │
│  (Platform-Agnostic Business Logic - THIS DOCUMENT)             │
│  - Musical Entities (Score, Tune, Part, Measure, Note)          │
│  - Embellishment System (Pipe & Drum)                           │
│  - Business Rules & Validation                                  │
│  - Repository Interfaces                                        │
│  - Domain Services                                              │
└─────────────────────────────────────────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Data Layer                               │
│  (Platform-Specific: Core Data, Room, EF Core, SQLite)          │
│  - Score Persistence                                            │
│  - Cloud Storage Integration                                    │
│  - Audio File Management                                        │
│  - Repository Implementations                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Layer Responsibilities

**Presentation Layer (Platform-Specific):**
- Renders musical notation using platform graphics APIs
- Handles user input and gesture recognition
- Implements platform-specific UI patterns and conventions
- Manages view lifecycle and navigation

**Application Layer (Platform-Specific with Domain Integration):**
- Orchestrates use case execution
- Coordinates between domain logic and presentation
- Manages application state and workflows
- Handles cross-cutting concerns (logging, analytics)

**Domain Layer (Platform-Agnostic - Specified in This Document):**
- Defines core business entities and rules
- Implements musical validation logic
- Specifies repository contracts
- Provides domain services for complex operations
- **Must be implemented identically across all platforms**

**Data Layer (Platform-Specific):**
- Implements repository interfaces from domain layer
- Handles data persistence and retrieval
- Manages cloud synchronization
- Provides caching and performance optimization

### 2.3 Dependency Rule

**Critical Architectural Constraint:**
- Dependencies point **inward only**
- Domain layer has **zero dependencies** on outer layers
- Domain layer knows nothing about UI frameworks, databases, or platforms
- Outer layers depend on domain layer abstractions (interfaces/protocols)
- Changes to outer layers **never require domain layer changes**

---

## 3. Domain Layer Specification

This section defines the platform-agnostic domain model that must be consistently implemented across all platforms using native languages and patterns.

### 3.1 Core Domain Entities

#### 3.1.1 Instrument Hierarchy

**Instrument Classification:**
```
Instrument (Abstract Base)
├── Bagpipes (Pitched melodic instrument)
├── BassDrum (Unpitched percussion)
├── TenorDrum (Multi-drum pitched percussion)
└── SnareDrum (Unpitched percussion with rudiments)
```

**Core Instrument Properties:**
- `id`: Unique identifier (UUID/GUID)
- `instrumentType`: Enumeration (Bagpipes, BassDrum, TenorDrum, SnareDrum)
- `availableTunings`: List of supported tuning systems
- `defaultClef`: Primary clef assignment for notation
- `smuflSymbolMappings`: Musical symbol Unicode mappings

**Instrument Capabilities:**
```
InstrumentCapabilities
├── TuningSystem
│   ├── availableTunings: List of tuning options
│   └── defaultTuning: Standard tuning configuration
├── ClefAssignments
│   ├── primaryClef: Default clef for instrument
│   └── acceptableAlternatives: Optional alternative clefs
├── NotationRules
│   ├── staffConfiguration: StaffConfiguration
│   │   ├── lineCount: Integer (5 for bagpipes, 1 for drums)
│   │   ├── staffType: StaffType (Standard, Percussion)
│   │   └── spacingRequirements: Float
│   ├── ornamentAvailability: Supported embellishment types
│   └── specialSymbols: Instrument-specific notation symbols
├── AudioCharacteristics
│   ├── frequencyRange: Playable frequency range
│   └── harmonics: Harmonic series information
└── LayoutRequirements
    ├── spacingNeeds: Visual spacing requirements
    └── multiStaffRequirements: Staff system layout needs
```

**Default Clef Assignments:**
- Bagpipes: Treble Clef
- Bass Drum: Bass Clef
- Tenor Drum: Bass Clef
- Snare Drum: Unpitched Percussion Clef

#### 3.1.2 Score Document Structure

**Primary Entity: ScoreDocument**
```
ScoreDocument
├── Metadata
│   ├── title: String
│   ├── composer: String (optional)
│   ├── arranger: String (optional)
│   ├── copyright: String (optional)
│   ├── version: Integer
│   ├── created: DateTime
│   ├── modified: DateTime
│   ├── defaultOrientation: PageOrientation
│   └── defaultPaperSize: PaperSize
├── DocumentLayoutSettings
│   ├── globalSpacingFactor: Float (0.5 to 2.0)
│   ├── defaultMargins: EdgeInsets
│   ├── compressionStrategy: CompressionStrategy
│   └── fontSizeAdjustment: Float
├── Pages[]
│   └── TuneLines[]
│       └── lineReference: Reference to specific Part or TextLine
├── NoteGroups: Dictionary<UUID, NoteGroup>
└── Tunes[]
    └── (See Tune structure below)
```

**Tune Structure:**
```
Tune
├── Metadata
│   ├── id: UUID
│   ├── title: String
│   ├── composer: String (optional)
│   ├── tuneType: TuneType (March, Strathspey, Reel, Jig, etc.)
│   ├── tempo: TempoMarking
│   ├── keySignature: KeySignature
│   └── orientationOverride: PageOrientation (optional)
├── TuneLayoutPreference
│   ├── pageBreakPolicy: PageBreakPolicy
│   ├── compressionLevel: CompressionLevel
│   ├── spacingPreference: SpacingPreference
│   └── systemBreakStrategy: SystemBreakStrategy
└── Parts[]
    └── (See Part structure below)
```

**Part Structure:**
```
Part (Structural Section - Part 1, Part 2, Part 3, etc.)
├── Metadata
│   ├── id: UUID
│   ├── partNumber: Integer (1, 2, 3, 4, 5... - defines both identity and play sequence)
│   └── name: String (optional - descriptive name like "Intro", "Outro")
└── MusicalSystems[]
    └── (See MusicalSystem structure below)

Business Rules:
├── Part numbers must be unique within a Tune
├── Part numbers define play order (Part 1 plays first, Part 2 plays second, etc.)
├── Part numbers must be positive integers starting from 1
├── Optional names supplement part numbers (e.g., Part 1 named "Intro")
├── Each Part contains one or more MusicalSystems
├── Each MusicalSystem spans ALL instruments in the ensemble
├── Parts play in numerical sequence: 1, 2, 3, 4...
└── Repeat behavior is NOT a Part property - it is defined by musical notation:
    ├── Repeat barlines (RepeatStart, RepeatEnd) in measures
    └── Volta brackets (1st ending, 2nd ending, etc.) for alternative endings

Numbering Conventions:
├── Standard: Part 1, Part 2, Part 3, Part 4
├── Named: Part 1 "Intro", Part 2, Part 3, Part 4 "Outro"
└── Display: Show "Part {partNumber}" or "{name}" if provided, or "Part {partNumber} - {name}"
```

**MusicalSystem Structure:**
```
MusicalSystem
├── id: UUID
├── instruments: List<InstrumentID> (ALL instruments in this system)
├── SystemStartElements
│   └── PerInstrument
│       ├── clef: Clef
│       ├── keySignature: KeySignature (optional)
│       └── timeSignature: TimeSignature (optional)
├── SystemLayoutHints
│   ├── staffSpacing: Float (optional)
│   ├── systemBreakBefore: Boolean
│   ├── systemBreakAfter: Boolean
│   ├── alignmentHints: AlignmentHints (optional)
│   └── instrumentSpacing: Dictionary<InstrumentID, Float>
├── Measures[]
│   └── Each Measure contains InstrumentMeasure for each instrument (See Measure structure below)
└── Barlines[]
    └── Span across all instruments in system (See Barline structure below)
```

#### 3.1.3 Measure Structure with Musical Intelligence

**Measure Entity:**
```
Measure
├── Musical Properties
│   ├── id: UUID
│   ├── timeSignature: TimeSignature (optional - cascades if null)
│   ├── keySignature: KeySignature (optional)
│   ├── tempo: TempoMarking (optional)
│   ├── rehearsalMark: String (optional)
│   └── instrumentMeasures: Dictionary<InstrumentID, InstrumentMeasure>
├── Layout Properties
│   ├── width: Float (optional - auto-calculated if null)
│   ├── compressionFactor: Float (optional - 1.0 = normal)
│   ├── breakHint: BreakHint (optional)
│   ├── minimumWidth: Float (optional)
│   └── spacingMultiplier: Float (optional)
└── Barline Configuration
    ├── openingBarline: BarlineType
    ├── closingBarline: BarlineType
    ├── repeatDots: RepeatDotConfiguration (optional)
    └── endingBracket: EndingBracket (optional)
```

**InstrumentMeasure:**
```
InstrumentMeasure
├── instrumentId: InstrumentID
├── notes: List<Note>
├── rests: List<Rest>
└── validationState: ValidationState
```

**Time Signature Cascading Logic:**

Time signatures cascade backward through the musical structure:
1. Check current measure's explicit time signature
2. If null, search backward through previous measures in same system
3. If none found in system, search previous systems in same part
4. If none found in part, use tune's default time signature
5. Cache resolved time signature for performance

**Implementation Requirement:** Platform implementations must cache cascaded time signatures to avoid repeated backward searches.

#### 3.1.4 Note Type Hierarchy (Abstract Base)

**Note Abstract Base Class:**
```
Note (Abstract - Platform implementations provide concrete classes)
├── Universal Musical Properties
│   ├── id: UUID
│   ├── noteType: NoteType (crotchet, quaver, minim, semibreve, etc.)
│   ├── durationAdjustment: Float (percentage adjustment for playback)
│   ├── embellishment: Embellishment (optional)
│   ├── articulation: List<Articulation>
│   └── noteGroups: List<NoteGroupID> (references to spanning relationships)
├── Universal Layout Properties
│   ├── position: Point (optional - auto-calculated if null)
│   ├── spacingHints: SpacingHints (optional)
│   ├── visualStyle: VisualStyle (optional)
│   └── collisionAvoidance: CollisionHints (optional)
└── Domain Methods (Contract)
    ├── getBaseDuration() -> Duration (from noteType)
    ├── getEffectiveDuration() -> Duration (base + adjustment)
    ├── calculateTotalWidth() -> Float (includes embellishments)
    ├── calculateEmbellishmentLayout() -> LayoutResult
    └── getTotalLayoutBounds() -> Rectangle
```

**Instrument-Specific Note Subclasses:**

Platform implementations must provide concrete note classes that extend the abstract base:

```
PipeNote : Note
├── Pipe-Specific Properties
│   ├── pitch: Pitch (required for pitched instruments)
│   └── accidental: Accidental (optional)
└── Methods
    ├── validatePitchRange() -> ValidationResult
    └── getPitchClass() -> PitchClass

SnareDrumNote : Note
├── Snare-Specific Properties
│   ├── hand: Hand (Left, Right - required)
│   ├── stickTechnique: StickTechnique (Regular, BackStick)
│   └── stickHeight: StickHeight (visual indication)
└── Methods
    ├── swapHands() -> SnareDrumNote
    └── validateStickingPattern(previousNote) -> ValidationResult

TenorDrumNote : Note
├── Tenor-Specific Properties
│   ├── drumPosition: Integer (1, 2, 3, 4, etc.)
│   ├── hand: Hand (Left, Right - required)
│   └── stickHeight: StickHeight
└── Methods
    ├── validateReachability(previousNote) -> ValidationResult
    └── getDrumConfiguration() -> DrumConfiguration

BassDrumNote : Note
├── Bass-Specific Properties
│   └── hand: Hand (Left, Right - required)
└── Methods
    ├── swapHands() -> BassDrumNote
    └── validateStickingPattern(previousNote) -> ValidationResult
```

**Note Type Duration Mappings:**
```
NoteType Enumeration:
├── Semibreve (Whole Note): Base duration = 1.0
├── Minim (Half Note): Base duration = 0.5
├── Crotchet (Quarter Note): Base duration = 0.25
├── Quaver (Eighth Note): Base duration = 0.125
├── Semiquaver (Sixteenth Note): Base duration = 0.0625
├── Demisemiquaver (Thirty-second Note): Base duration = 0.03125
└── Hemidemisemiquaver (Sixty-fourth Note): Base duration = 0.015625
```

**Duration Adjustment:** The `durationAdjustment` property allows synthetic playback timing modifications (e.g., swing feel) without changing visual notation.

#### 3.1.5 Rest Entity

**Rest Structure:**
```
Rest
├── Musical Properties
│   ├── id: UUID
│   ├── duration: Duration
│   ├── isVisible: Boolean
│   └── restType: RestType
├── Layout Properties
│   ├── position: Point (optional)
│   ├── verticalOffset: Float (optional)
│   └── size: Float (optional)
└── Methods
    ├── calculateWidth() -> Float
    └── getSMuFLCodepoint() -> Unicode
```

#### 3.1.6 Clef and Key Signature Management

**Clef Entity:**
```
Clef
├── Musical Properties
│   ├── clefType: ClefType (Treble, Bass, Alto, Unpitched)
│   ├── instrumentCompatibility: List<InstrumentType>
│   ├── octaveTransposition: Integer (optional)
│   └── isDefault: Boolean
├── Layout Properties
│   ├── position: Point (optional)
│   ├── size: Float (optional)
│   └── visibility: ClefVisibility
└── Methods
    ├── isCompatibleWith(InstrumentType) -> Boolean
    ├── getStaffPositionFor(Pitch) -> StaffPosition
    └── shouldDisplayAt(SystemPosition) -> Boolean
```

**Clef Rendering Rules:**
- Display clef at: first measure of system, clef change, after system break
- Default clefs: Bagpipes (Treble), Bass/Tenor Drums (Bass), Snare (Unpitched)
- Alternative clefs available per instrument type
- Clef changes mid-system must be marked with cautionary clef

**Key Signature Entity:**
```
KeySignature
├── Musical Properties
│   ├── sharps: Integer (-7 to +7, negative = flats)
│   ├── mode: Mode (Major, Minor, Dorian, Mixolydian, etc.)
│   └── customAccidentals: List<Accidental> (for non-standard keys)
├── Layout Properties
│   ├── position: Point (optional)
│   └── spacing: Float (optional)
└── Methods
    ├── getAccidentalPositions(Clef) -> List<StaffPosition>
    ├── getPitchAlteration(Pitch) -> Accidental
    └── requiresNaturalCancellation(previousKey) -> Boolean
```

**Bagpipe Key Signature Convention:**
- Traditional bagpipe music uses 2 sharps (F# and C#) for D Major
- Display at start of each system for clarity
- Modern notation may use concert pitch key signatures

#### 3.1.7 Time Signature Entity

**Time Signature Structure:**
```
TimeSignature
├── Musical Properties
│   ├── numerator: Integer
│   ├── denominator: Integer (power of 2: 1, 2, 4, 8, 16)
│   ├── beatGrouping: List<Integer> (for complex meters, e.g., [3,3,2] for 8/8)
│   ├── metronomeMarking: MetronomeMarking (optional)
│   └── displayStyle: TimeSignatureDisplay (Numeric, Common, Cut)
├── Layout Properties
│   ├── position: Point (optional)
│   ├── size: Float (optional)
│   ├── visibility: TimeSignatureVisibility
│   └── alignment: TimeSignatureAlignment
└── Methods
    ├── getSuggestedTempoNoteValue() -> Duration
    │   └── Returns appropriate note value for tempo marking
    │       ├── 2/4, 3/4, 4/4 → Quarter Note
    │       ├── 6/8, 9/8, 12/8 → Dotted Quarter Note
    │       ├── 2/2, 3/2 → Half Note
    │       └── Irregular meters → Context-dependent
    ├── validateTempoCompatibility(TempoMarking) -> ValidationResult
    │   └── Checks if tempo note value makes musical sense
    │       ├── Warning: Quarter note tempo with 6/8 time (should use dotted quarter)
    │       ├── Warning: Dotted quarter with 4/4 time (should use quarter note)
    │       └── Error: Nonsensical combinations (whole note tempo with 2/4 time)
    ├── getDefaultTempo() -> TempoMarking
    │    └── Returns culturally appropriate default tempo for time signature
    │       ├── 4/4 → ♩ = 90 (standard march)
    │       ├── 6/8 → ♩. = 96 (standard jig)
    │       ├── 2/4 → ♩ = 90 (standard march)
    │       └── Other signatures → context-dependent defaults
    ├── getBeatsPerMeasure() -> Integer
    ├── getBeatValue() -> Duration
    ├── getStrongBeats() -> List<Integer>
    ├── validateNoteDurations(List<Note>) -> ValidationResult
    └── getTotalMeasureDuration() -> Duration
```

**Common Pipe Band Time Signatures:**
- 2/4 (March, Polka)
- 3/4 (Slow Air, Waltz)
- 4/4 (March, Strathspey)
- 6/8 (Jig, March)
- 9/8 (Slip Jig, Compound Time March)

**Beat Grouping and Tempo Relationship:**
```
Complex Meter Examples:
├── 5/4 with grouping [3,2]
│   ├── Could be marked: ♩ = 120
│   └── Alternative: (♩ = 120, grouped 3+2)
├── 7/8 with grouping [3,2,2]
│   ├── Could be marked: ♪ = 200
│   └── Alternative: ♩. = 100, ♩ = 100, ♩ = 100 (three separate beats)
└── 9/8 traditional vs compound
    ├── Compound: ♩. = 100 (three beats)
    └── Irregular: ♪ = 300 (nine beats) - rare
```

#### 3.1.8 Barline System

**Barline Entity:**
```
Barline
├── Musical Properties
│   ├── barlineType: BarlineType
│   ├── repeatConfiguration: RepeatConfiguration (optional)
│   ├── systemSpanning: Boolean
│   └── musicalFunction: MusicalFunction
├── Layout Properties
│   ├── position: Float (optional)
│   ├── height: Float (optional - for system-spanning)
│   ├── thickness: Float (optional)
│   └── spacing: BarlineSpacing (optional)
└── Methods
    ├── spansInstruments(List<InstrumentType>) -> Boolean
    ├── validatePlacement(SystemContext) -> ValidationResult
    └── getSMuFLCodepoint() -> Unicode
```

**Barline Types with SMuFL Codepoints:**
```
BarlineType Enumeration:
├── None (Transparent): No visual barline
├── Single (U+E030): Standard measure division
├── Double (U+E031): Section ending or emphasis
├── Final (U+E032): End of piece (thick-thin double bar)
├── RepeatStart (U+E040): Beginning of repeated section
├── RepeatEnd (U+E041): End of repeated section
├── RepeatBoth (U+E042): Both start and end of repeat
├── Dashed (U+E036): Optional division or phrase marker
├── Heavy (U+E034): Strong division (thicker single bar)
└── Dotted (U+E037): Subtle division marker
```

**Repeat Configuration:**
```
RepeatConfiguration
├── repeatType: RepeatType (Start, End, Both)
├── repeatCount: Integer (number of times to repeat)
├── endingNumbers: List<Integer> (for 1st/2nd ending brackets)
└── voltaBracket: VoltaBracket (optional ending bracket specification)
```

#### 3.1.9 Tempo Marking Entity

**TempoMarking Structure:**
```
TempoMarking
├── Musical Properties
│   ├── noteValue: Duration (required - the note duration that defines the beat)
│   │   ├── Representation as note type (whole, half, quarter, eighth, etc.)
│   │   ├── Support for dotted notes (dotted quarter, dotted half, etc.)
│   │   └── Support for compound values (half + quarter for complex meters)
│   ├── beatsPerMinute: Integer (required - count of noteValue per minute)
│   │   ├── Valid range: 30-300 (typical musical range)
│   │   ├── Extended range: 20-400 (for extreme cases)
│   │   └── Integer precision sufficient for musical accuracy
│   ├── textualMarking: String (optional - traditional Italian terms)
│   │   ├── Examples: "Allegro", "Andante", "Moderato", "Presto"
│   │   ├── Pipe band specific: "Quick March", "Slow March", "Strathspey Time"
│   │   └── Custom text allowed for specialized cases
│   └── modifier: TempoModifier (optional - performance instructions)
│       ├── "circa" (~) - approximately the marked tempo
│       ├── "a tempo" - return to previous tempo
│       ├── "ritardando" - gradually slowing
│       ├── "accelerando" - gradually speeding up
│       └── Other standard tempo modifiers
├── Display Properties
│   ├── position: Point (optional - placement on staff)
│   ├── alignment: TempoAlignment (left, center, right)
│   └── visibility: TempoVisibility (show/hide)
├── Validation Rules
│   ├── beatsPerMinute must be in valid range
│   ├── noteValue must be valid Duration type
│   ├── textualMarking must be recognized term if provided
│   └── noteValue should logically relate to time signature
└── Methods
    ├── calculateMillisecondsPerBeat() -> Float
    │   └── Returns: (60000.0 / beatsPerMinute)
    ├── isEquivalentTo(TempoMarking) -> Boolean
    │   └── Checks if two tempo markings produce same playback speed
    ├── getSMuFLRepresentation() -> String
    │   └── Returns: Unicode string for notation rendering
    ├── getPlaybackDuration(Duration) -> Milliseconds
    │   └── Calculates real-time duration for any note value
    └── validateCompatibility(TimeSignature) -> ValidationResult
        └── Checks if tempo note value makes musical sense for time signature
```

**Tempo Note Value Types:**
```
Duration Enumeration (for tempo marking):
├── Whole Note (Semibreve): Entire measure duration (rare in tempo markings)
├── Dotted Half Note: 3 quarter note beats
├── Half Note (Minim): 2 quarter note beats
├── Dotted Quarter Note: 3 eighth note beats (common in compound time)
├── Quarter Note (Crotchet): Standard beat unit (most common)
├── Dotted Eighth Note: 3 sixteenth note beats (rare)
├── Eighth Note (Quaver): Half a quarter beat (fast tempos)
└── Sixteenth Note (Semiquaver): Quarter of a quarter beat (very rare)
```

**Pipe Band Tempo Conventions:**
```
Standard Pipe Band Tempos:
├── March (2/4 or 4/4)
│   ├── Slow March: ♩ = 70-80 (solemn, processional)
│   ├── Standard March: ♩ = 80-90 (parade tempo)
│   ├── Quick March: ♩ = 90-96 (spirited)
│   └── Competition March: ♩ = 85-90 (regulated for competition)
├── Strathspey (4/4)
│   ├── Traditional: ♩ = 168-176 (slow, stately, with "Scotch snap")
│   ├── Competition: ♩ = 168-172 (regulated)
│   └── Alternative notation: ♪ = 336-344 (some prefer eighth note reference)
├── Reel (4/4 or 2/2)
│   ├── Standard: ♩ = 112-120 (lively, flowing)
│   ├── Fast Reel: ♩ = 120-126 (energetic)
│   └── Competition: ♩ = 112-116 (regulated)
├── Jig (6/8)
│   ├── Standard: ♩. = 88-104 (dotted quarter = beat unit)
│   ├── Slow Jig: ♩. = 80-88 (more relaxed)
│   └── Competition: ♩. = 92-96 (regulated)
├── Slip Jig (9/8)
│   ├── Standard: ♩. = 96-112 (dotted quarter = beat unit)
│   └── Competition: ♩. = 100-108 (regulated)
├── Hornpipe (4/4)
│   ├── Traditional: ♩ = 104-120 (with triplet or dotted feel)
│   └── Scottish Hornpipe: ♩ = 112-120 (distinct from Irish style)
└── Slow Air (Various)
    ├── Extremely flexible: ♩ = 40-72 (highly expressive)
    ├── Rubato common: tempo varies throughout
    └── Often marked "Freely" or "Ad libitum"
```

**Musical Logic and Validation:**

**Time Signature Compatibility:**
```
Recommended Tempo Note Values by Time Signature:
├── Simple Time (2/4, 3/4, 4/4, 2/2, 3/2)
│   ├── Primary: Quarter note (♩)
│   ├── Alternative: Half note (𝅗𝅥) for slower pieces
│   └── Alternative: Eighth note (♪) for very fast pieces
├── Compound Duple (6/8, 6/4, 6/16)
│   ├── Primary: Dotted quarter note (♩.)
│   ├── Alternative: Dotted half note (𝅗𝅥.) for very slow pieces
│   └── Never: Quarter note (creates ambiguity)
├── Compound Triple (9/8, 9/4, 9/16)
│   ├── Primary: Dotted quarter note (♩.)
│   └── Alternative: Dotted half note (𝅗𝅥.) for slow pieces
├── Compound Quadruple (12/8, 12/4, 12/16)
│   ├── Primary: Dotted quarter note (♩.)
│   └── Alternative: Dotted half note (𝅗𝅥.) for slow pieces
└── Irregular Meters (5/4, 7/8, etc.)
    ├── Varies based on beat grouping
    ├── May use compound note values
    └── Often requires explicit beat subdivision marking
```

**Tempo Equivalence Calculations:**
```
Tempo Equivalence Examples:
├── ♩ = 60 is equivalent to:
│   ├── 𝅗𝅥 = 30 (half notes)
│   ├── ♪ = 120 (eighth notes)
│   └── ♩. = 40 (dotted quarters)
├── ♩. = 90 (6/8 jig tempo) is equivalent to:
│   ├── ♩ = 135 (if counted in quarters - not recommended)
│   └── ♪ = 270 (if counted in eighths - not useful)
└── ♩ = 120 (4/4 reel) is equivalent to:
    ├── 𝅗𝅥 = 60 (half note reference)
    └── ♪ = 240 (eighth note reference)
```

**Display Rendering:**

**SMuFL Rendering Requirements:**
```
Tempo Marking Display Components:
├── Note Symbol: SMuFL glyph for note value
│   ├── Quarter Note: U+E1D5 (♩)
│   ├── Dotted Quarter: U+E1D5 + U+E1E7 (♩.)
│   ├── Half Note: U+E1D3 (𝅗𝅥)
│   ├── Eighth Note: U+E1D7 (♪)
│   └── Other durations: corresponding SMuFL codepoints
├── Equals Sign: Standard Unicode U+003D (=)
├── Beats Per Minute: Standard numerals
├── Textual Marking: Standard text (bold or italic)
│   ├── Position: Above or before numeric tempo
│   └── Font: Match document text style
└── Complete Example: "♩ = 120" or "Allegro (♩ = 120)"
```

**Placement Guidelines:**
```
Tempo Marking Placement:
├── Position: Above first staff of system
├── Alignment: Left-aligned with first measure
├── Clearance: Minimum 2 staff spaces above highest staff line
├── Changes: New tempo markings at point of change
└── Returns: "a tempo" marking when returning to previous tempo
```

**Playback Calculation:**

**Milliseconds Per Beat Calculation:**
```
Formula: millisecondsPerBeat = 60000.0 / beatsPerMinute

Examples:
├── ♩ = 120 → 60000 / 120 = 500ms per quarter note
├── ♩. = 90 → 60000 / 90 = 666.67ms per dotted quarter
└── ♩ = 80 → 60000 / 80 = 750ms per quarter note
```

**Note Duration Calculation:**
```
To calculate real-time duration for any note:
1. Determine note's relationship to tempo note value
2. Calculate proportional duration
3. Apply any duration adjustments

Example (♩ = 120, playing an eighth note):
├── Tempo: ♩ = 120 → 500ms per quarter note
├── Eighth note: Half of quarter note
└── Result: 500ms / 2 = 250ms for eighth note

Example (♩. = 90, playing a quarter note):
├── Tempo: ♩. = 90 → 666.67ms per dotted quarter
├── Quarter note: 2/3 of dotted quarter note
└── Result: 666.67ms × (2/3) = 444.44ms for quarter note
```

**Export Considerations:**

**MIDI Export:**
```
MIDI Tempo Conversion:
├── MIDI uses microseconds per quarter note
├── Conversion formula:
│   └── microsecondsPerQuarterNote = 60000000 / (beatsPerMinute × conversionFactor)
├── Conversion factors:
│   ├── If tempo is ♩ = X → factor = 1.0
│   ├── If tempo is ♩. = X → factor = 1.5 (dotted quarter to quarter)
│   ├── If tempo is 𝅗𝅥 = X → factor = 0.5 (half note to quarter)
│   └── If tempo is ♪ = X → factor = 2.0 (eighth to quarter)
└── Result: MIDI tempo event with calculated value
```

**MusicXML Export:**
```
MusicXML Tempo Representation:
<sound tempo="120"/>  <!-- Always in quarter notes per minute -->
<direction>
  <direction-type>
    <metronome>
      <beat-unit>quarter</beat-unit>
      <per-minute>120</per-minute>
    </metronome>
  </direction-type>
</direction>

For dotted notes:
<metronome>
  <beat-unit>quarter</beat-unit>
  <beat-unit-dot/>  <!-- Indicates dotted note -->
  <per-minute>90</per-minute>
</metronome>
```

**Educational Considerations:**

**Beginner-Friendly Display:**
```
Progressive Complexity:
├── Basic Display: "♩ = 120" (numeric only)
├── Intermediate: "Quick March (♩ = 90)" (adds context)
├── Advanced: "♩ = 90 (circa)" (adds performance nuance)
└── Educational: Show relationship between note values and tempo
```

### 3.2 Embellishment System Architecture

Embellishments are decorative musical elements that belong to individual notes. They are implemented as properties of notes with sophisticated layout intelligence.

#### 3.2.1 Base Embellishment Entity

**Embellishment Abstract Base:**
```
Embellishment (Abstract)
├── Musical Properties
│   ├── embellishmentType: EmbellishmentType
│   ├── traditionalName: String (cultural terminology)
│   └── executionStyle: ExecutionStyle
├── Layout Properties
│   ├── distanceFromPrincipal: Float
│   ├── interGraceSpacing: Float
│   ├── verticalPlacement: VerticalPlacement
│   ├── collisionAvoidance: CollisionStrategy
│   └── floatBeforeBarline: Boolean
└── Methods
    ├── calculateGraceNotePositions() -> List<Point>
    ├── determineOptimalSpacing(context) -> SpacingResult
    ├── resolveCollisions(adjacentElements) -> CollisionResolution
    ├── shouldFloatBeforeBarline() -> Boolean
    └── validateLayoutFeasibility() -> ValidationResult
```

**Floating Before Barline:**
When a note with an embellishment is the first note in a measure, the embellishment's grace notes can optionally visually appear before the preceeding barline for an alternate musical readability. This is typically used in SnareDrum music only. 

#### 3.2.2 Pipe Band Embellishments

**PipeBandEmbellishment Base:**
```
PipeBandEmbellishment : Embellishment
├── Musical Properties
│   ├── regionalStyle: RegionalStyle (Highland, Border, Competition, Traditional)
│   ├── fingeringSequence: FingeringPattern
│   └── traditionalNotation: Boolean
├── Layout Properties
│   ├── gracePitchStrategy: PitchCalculationMethod
│   ├── fingeringOptimization: Boolean
│   ├── staffPositionOverride: StaffPosition (optional)
│   └── beamingStrategy: BeamingStrategy
└── Methods
    ├── calculateGracePitches(principalPitch) -> List<Pitch>
    ├── optimizeForFingeringFlow() -> FingeringOptimization
    └── determineStaffPositions() -> List<StaffPosition>
```

**Specific Pipe Band Embellishment Types:**

**Double Strike:**
```
DoubleStrikeEmbellishment : PipeBandEmbellishment
├── graceSequence: 3 grace notes (pattern calculated from principal note)
├── beamingStyle: BeamingStyle
└── Pattern Logic:
    ├── Grace 1: High G (always)
    ├── Grace 2: Same pitch as principal note (always)
    ├── Grace 3: Conditional based on principal note pitch
    │   ├── If principal < D → Low G grace note
    │   ├── If principal == D → Low G OR C grace note (user choice)
    │   └── If principal > D → (principal - 1 semitone) grace note
    └── Principal Note: Original note
```

**Doubling:**
```
DoublingEmbellishment : PipeBandEmbellishment
├── doublingType: DoublingType (upToF, highG, highA)
├── graceSequence: Variable length based on doubling type
└── Pattern Logic:
    ├── upToF: Doubling on notes up to and including F
    │   ├── Grace 1: High G (always)
    │   ├── Grace 2: Same pitch as principal note (always)
    │   ├── Grace 3: If principal < D → D grace note
    │   │           If principal ≥ D → (principal + 1) grace note
    │   └── Principal Note: Original note
    ├── highG: Doubling specifically on high G
    │   └── Grace pattern: High G, F, then principal High G note
    └── highA: Doubling specifically on high A
        └── Grace pattern: High A, High G, then principal High A note
```

**Grip (Leamluath):**
```
GripEmbellishment : PipeBandEmbellishment
├── gripType: GripType (Regular, Breabach)
├── traditionalName: "Leamluath" (Gaelic preservation)
├── graceSequence: Fixed 3 grace notes (Low G, D, Low G)
└── Pattern Logic:
    ├── Grace 1: Low G (always written, may be omitted in performance)
    ├── Grace 2: D (always played)
    ├── Grace 3: Low G (always played)
    ├── Principal Note: Original note (any pitch)
    └── Performance Rule: If prior note == Low G, first grace note omitted in execution but still notated
```

**Taorluath:**
```
TaorluathEmbellishment : PipeBandEmbellishment
├── taorluathType: TaorluathType (Regular, Breabach)
├── graceSequence: Fixed 4 grace notes (Low G, D, Low G, E)
└── Pattern Logic:
    ├── Grace 1: Low G (always written, may be omitted in performance)
    ├── Grace 2: D (always played)
    ├── Grace 3: Low G (always played)
    ├── Grace 4: E (conditionally played based on principal note)
    ├── Principal Note: Original note (any pitch)
    └── Performance Rules:
        ├── If prior note == Low G, first grace note omitted in execution but still notated
        └── E grace note only played if principal note < E
```

**Light D Throw:**
```
LightDThrowEmbellishment : PipeBandEmbellishment
├── graceSequence: Fixed 3 grace notes (Low G, D, C)
└── Pattern Logic:
    ├── Grace 1: Low G (always written, may be omitted in performance)
    ├── Grace 2: D (always played)
    ├── Grace 3: C (always played)
    ├── Principal Note: D (specific to D principal)
    └── Performance Rule: If prior note == Low G, first grace note omitted in execution but still notated
```

**Heavy D Throw:**
```
HeavyDThrowEmbellishment : PipeBandEmbellishment
├── graceSequence: Fixed 4 grace notes (Low G, D, Low G, C)
└── Pattern Logic:
    ├── Grace 1: Low G (always written, may be omitted in performance)
    ├── Grace 2: D (always played)
    ├── Grace 3: Low G (always played)
    ├── Grace 4: C (always played)
    ├── Principal Note: D (specific to D principal)
    └── Performance Rule: If prior note == Low G, first grace note omitted in execution but still notated
```

**Low A Birl:**
```
LowABirlEmbellishment : PipeBandEmbellishment
├── graceSequence: Conditional (4 or 3 grace notes based on prior note)
└── Pattern Logic:
    ├── If prior note != Low A:
    │   ├── Grace 1: Low A (written and played)
    │   ├── Grace 2: Low G (always played)
    │   ├── Grace 3: Low A (always played)
    │   ├── Grace 4: Low G (always played)
    │   └── Principal Note: Low A
    ├── If prior note == Low A:
    │   ├── Grace 1: (omitted from notation and performance)
    │   ├── Grace 2: Low G (always played)
    │   ├── Grace 3: Low A (always played)
    │   ├── Grace 4: Low G (always played)
    │   └── Principal Note: Low A
    └── Notation Rule: First grace note completely omitted when prior note is Low A
```

**G Gracenote Birl:**
```
GGracenoteBirlEmbellishment : PipeBandEmbellishment
├── graceSequence: Fixed 5 grace notes (High G, Low A, Low G, Low A, Low G)
└── Pattern Logic:
    ├── Grace 1: High G (always written and played)
    ├── Grace 2: Low A (always written and played)
    ├── Grace 3: Low G (always written and played)
    ├── Grace 4: Low A (always written and played)
    ├── Grace 5: Low G (always written and played)
    ├── Principal Note: Low A (specific to Low A principal)
    └── No Exceptions: All grace notes always written and played regardless of prior note
```

#### 3.2.3 Drum Embellishments

**DrumEmbellishment Base:**
```
DrumEmbellishment : Embellishment
├── Musical Properties
│   ├── rudimentName: String (standard drum rudiment classification)
│   └── stickingPattern: StickingPattern
├── Layout Properties
│   ├── stemDirection: StemDirection (Upward, Downward)
│   ├── stickHeightVariation: StickHeightVariation
│   └── rudimentSpacing: Float
└── Methods
    ├── calculateStickHeights() -> List<Float>
    └── optimizeForRudimentFlow() -> RudimentOptimization
```

**Specific Drum Embellishment Types:**

**Flam:**
```
FlamEmbellishment : DrumEmbellishment
├── graceNoteCount: 1 (fixed)
├── handRelationship: OppositeHand (grace note opposite from principal)
├── flamTiming: FlamTiming (Tight, Open)
├── stemDirection: Upward (flam grace notes always stem upward)
└── Layout Requirements:
    ├── Single grace note before principal
    ├── Opposite hand coordination (grace opposite to principal)
    ├── Automatic hand swapping when principal note hand changes
    ├── Lower visual position (grace note below principal stroke)
    ├── Upward stem rendering
    ├── Hand assignment inheritance from principal note
    └── Tight horizontal spacing
```

**Drag:**
```
DragEmbellishment : DrumEmbellishment
├── graceNoteCount: 2 (fixed)
├── handRelationship: OppositeHand (both grace notes opposite from principal)
├── stemDirection: Upward (distinguishes from open drag)
└── Layout Requirements:
    ├── Two grace notes before principal (standard drag bounce)
    ├── Opposite hand coordination (both grace notes opposite to principal)
    ├── Automatic hand swapping when principal note hand changes
    ├── Hand assignment inheritance from principal note
    ├── Upward stem rendering
    ├── Tight grouping spacing
    └── Visual distinction via upward stems
```

**Open Drag:**
```
OpenDragEmbellishment : DrumEmbellishment
├── graceNoteCount: 2 (fixed)
├── handRelationship: OppositeHand (grace notes opposite from principal)
├── openingSpacing: Float (temporal spacing between grace notes)
└── Layout Requirements:
    ├── Two grace notes before principal
    ├── Opposite hand coordination
    ├── Automatic hand swapping
    ├── Open spacing visualization (clear temporal separation)
    └── Hand assignment inheritance
```

**Rough:**
```
RoughEmbellishment : DrumEmbellishment
├── graceNoteCount: 3 (fixed)
├── handRelationship: AlternatingPattern (starts opposite, then alternates)
├── handPattern: OppositeHand -> SameHand -> OppositeHand (relative to principal)
├── stemDirection: Upward
└── Layout Requirements:
    ├── Three grace notes before principal (rough stroke cluster)
    ├── Alternating hand coordination starting opposite from principal
    ├── Automatic pattern swapping when principal note hand changes
    ├── Hand assignment inheritance with alternating pattern
    ├── Upward stem rendering
    ├── Cluster spacing (tight grouping)
    ├── Pattern visualization (clear alternating indication)
    └── Principal note connection clarity
```

**Swiss Rough:**
```
SwissRoughEmbellishment : DrumEmbellishment
├── swissPattern: SwissPattern (Swiss-specific execution)
├── strokeSequence: SwissStrokeSequence
├── traditionalStyle: Boolean (Swiss pipe band tradition adherence)
├── stemDirection: Upward
└── Layout Requirements:
    ├── Swiss-specific pattern representation
    ├── Cultural notation style (authentic Swiss formatting)
    ├── Upward stem rendering
    ├── Distinctive Swiss rough appearance
    └── Traditional spacing (culturally appropriate)
```

#### 3.2.4 Embellishment Validation Rules

**Prior Note Constraint Validation:**
```
PriorNoteConstraintRules:
├── upToFDoublingRule: NoPriorNoteRestriction
│   └── Always valid regardless of prior note pitch
├── highGDoublingRule: PriorNoteLowerThanHighG
│   └── Prior note must be lower than High G (Low G, Low A, B, C, D, E, F)
├── highADoublingRule: PriorNoteLowerThanHighA
│   └── Prior note must be lower than High A (Low G through High G)
├── doubleStrikeRule: NoPriorNoteRestriction
│   └── Always valid regardless of prior note pitch
├── gripRule: NoPriorNoteRestriction
│   └── Always valid (note: first grace omitted in execution if prior == Low G)
├── taorluathRule: NoPriorNoteRestriction
│   └── Always valid (note: first grace omitted if prior == Low G, E only if principal < E)
├── lightDThrowRule: NoPriorNoteRestriction
│   └── Always valid (note: first grace omitted if prior == Low G)
├── heavyDThrowRule: NoPriorNoteRestriction
│   └── Always valid (note: first grace omitted if prior == Low G)
├── lowABirlRule: NoPriorNoteRestriction
│   └── Always valid (note: first grace completely omitted if prior == Low A)
├── gGracenoteBirlRule: NoPriorNoteRestriction
│   └── Always valid (no exceptions, all grace notes always written)
└── drumEmbellishmentRules: Apply drum-specific constraints
```

**Cultural Sensitivity Validation:**
```
CulturalValidationRules:
├── Regional Style Adherence
│   ├── Highland style conventions
│   ├── Border piping variations
│   ├── Competition vs. traditional contexts
│   └── Educational progression appropriateness
├── Traditional Terminology Preservation
│   ├── Gaelic names maintained alongside English
│   ├── Traditional fingering patterns documented
│   ├── Regional pronunciation guides
│   └── Cultural context for non-native practitioners
└── Historical Accuracy
    ├── Traditional tune authenticity
    └── Period-appropriate ornamentation
```

### 3.3 Note Group System

Note groups represent musical relationships that span between notes (slurs, ties, tuplets, dynamics).

#### 3.3.1 Note Group Base Entity

**NoteGroup Abstract Base:**
```
NoteGroup (Abstract)
├── Musical Properties
│   ├── id: UUID
│   ├── headNote: NoteID (starting note reference)
│   ├── tailNote: NoteID (ending note reference)
│   ├── groupType: NoteGroupType
│   └── musicalFunction: MusicalFunction
├── Layout Properties
│   ├── spanningLine: SpanningLineStyle (optional)
│   ├── groupSymbol: SMuFLCodepoint (optional)
│   ├── verticalPosition: VerticalPlacement
│   └── horizontalAlignment: HorizontalAlignment
└── Methods
    ├── isHeadNote(NoteID) -> Boolean
    ├── isTailNote(NoteID) -> Boolean
    └── validateNoteSequence(context) -> ValidationResult
```

#### 3.3.2 Concrete Note Group Types

**Tuplet Group:**
```
TupletGroup : NoteGroup
├── Musical Properties
│   ├── tupletRatio: TupletRatio (3:2 triplet, 2:3 duplet, etc.)
│   ├── bracketStyle: TupletBracketStyle
│   ├── numberDisplay: TupletNumberDisplay
│   └── rhythmicGrouping: RhythmicPattern
├── Layout Properties
│   ├── bracketClearance: Float
│   ├── numberPosition: NumberPosition
│   └── spanningBracket: BracketStyle
└── Methods
    ├── applyTupletRatio(Duration) -> Duration
    ├── calculateBracketBounds(List<NotePosition>) -> Rectangle
    └── validateTupletSequence(List<Note>) -> ValidationResult
```

**Common Pipe Band Tuplets:**
- Triplets (3:2) - Three notes in the time of two
- Duplets (2:3) - Two notes in the time of three (in compound time)

**Slur Group:**
```
SlurGroup : NoteGroup
├── Musical Properties
│   ├── slurType: SlurType (Slur, Phrase)
│   ├── curvatureDirection: CurvatureDirection
│   ├── phraseBoundary: Boolean
│   └── musicalIntention: PhrasalFunction
├── Layout Properties
│   ├── curvatureHeight: Float
│   ├── endpointAdjustment: EndpointAdjustment
│   └── collisionAvoidance: CollisionStrategy
└── Methods
    ├── calculateSlurCurve(List<NotePosition>) -> BezierPath
    ├── determineOptimalCurvature(List<Note>) -> CurvatureParameters
    └── validateSlurPlacement(List<Note>) -> ValidationResult
```

**Tie Group:**
```
TieGroup : NoteGroup
├── Musical Properties
│   ├── tieType: TieType (Start, Continue, End)
│   ├── tieDirection: TieDirection
│   └── crossSystemTie: Boolean
├── Layout Properties
│   ├── tieThickness: Float
│   ├── tieHeight: Float
│   └── systemBreakHandling: SystemBreakStyle
└── Methods
    ├── validatePitchConsistency(List<Note>) -> ValidationResult
    ├── calculateTieCurve(startPos, endPos) -> BezierPath
    └── handleSystemBreak() -> SystemBreakTieResult
```

**Dynamic Group:**
```
DynamicGroup : NoteGroup
├── Musical Properties
│   ├── dynamicType: DynamicType (Crescendo, Diminuendo, etc.)
│   ├── startDynamic: DynamicLevel (optional)
│   ├── endDynamic: DynamicLevel (optional)
│   └── curvature: DynamicCurvature
├── Layout Properties
│   ├── hairpinThickness: Float
│   ├── textPosition: TextPosition
│   └── verticalOffset: Float
└── Methods
    ├── calculateHairpinPath(List<NotePosition>) -> HairpinPath
    ├── determineDynamicTextPlacement() -> TextPlacement
    └── validateDynamicProgression(List<Note>) -> ValidationResult
```

#### 3.3.3 Note Integration

**Note-to-NoteGroup Relationship:**
```
Note.noteGroups: List<NoteGroupID>

Convenience Methods:
├── getTies() -> List<TieGroup>
├── getSlurs() -> List<SlurGroup>
├── getTuplets() -> List<TupletGroup>
├── getDynamics() -> List<DynamicGroup>
├── addToNoteGroup(NoteGroupID) -> Void
└── removeFromNoteGroup(NoteGroupID) -> Void
```

**Document-Level Note Group Storage:**
```
ScoreDocument.noteGroups: Dictionary<UUID, NoteGroup>

Document Methods:
├── addNoteGroup(NoteGroup) -> Void
├── removeNoteGroup(UUID) -> Void
├── getNoteGroup(UUID) -> NoteGroup (optional)
└── validateNoteGroupIntegrity() -> ValidationResult
```

### 3.4 Use Cases (Application Layer Contracts)

This section defines the business operations that platform implementations must provide. Platform implementations will use their native patterns (Swift async/await, Kotlin coroutines, C# Tasks, etc.) while maintaining identical business logic.

#### 3.4.1 Score Management Use Cases

**CreateScoreUseCase:**
```
Input:
├── title: String
├── composer: String (optional)
├── defaultInstruments: List<InstrumentType>
└── layoutSettings: DocumentLayoutSettings (optional)

Business Rules:
├── Score must have at least one tune
├── Default tune must have at least one part (A part)
├── Document layout settings use system defaults if not provided
└── Initial tune gets auto-generated ID and default metadata

Output: ScoreDocument

Side Effects:
├── Triggers initial pagination calculation
├── Creates default folder assignment
└── Initializes layout cache entries

Error Conditions:
├── InvalidTitleError (empty or too long)
├── UnsupportedInstrumentCombinationError
└── PersistenceError
```

**DeleteScoreUseCase:**
```
Input: scoreId (UUID)

Business Rules:
├── Cannot delete score currently open for editing
├── Must clear all layout cache entries
├── Must remove from any folder assignments
└── Must cleanup associated audio files

Output: Void

Side Effects:
├── Clears layout caches
├── Removes audio files
├── Updates folder contents
└── Notifies UI of deletion

Error Conditions:
├── ScoreNotFoundError
├── ScoreLockedException (score is open for editing)
└── PersistenceError
```

**OpenScoreForEditingUseCase:**
```
Input:
├── scoreId: UUID
└── editingMode: EditingMode (ReadOnly, Collaborative, Exclusive)

Business Rules:
├── Only one exclusive edit session per score
├── Read-only mode allows multiple concurrent sessions
└── Collaborative mode requires conflict resolution setup

Output: EditingSession

Side Effects:
├── Locks score for exclusive editing (if applicable)
├── Prepares layout calculations
├── Initializes change tracking
└── Caches frequently used data

Error Conditions:
├── ScoreNotFoundError
├── ScoreAlreadyLockedException
└── InsufficientPermissionsError
```

#### 3.4.2 Musical Content Editing Use Cases

**CreateTuneUseCase:**
```
Input:
├── scoreId: UUID
├── insertPosition: Integer
├── tuneTemplate: TuneTemplate
│   ├── title: String
│   ├── tuneType: TuneType
│   ├── tempo: TempoMarking (defaults based on tuneType if not provided)
│   ├── keySignature: KeySignature
│   └── defaultTimeSignature: TimeSignature
└── layoutPreference: TuneLayoutPreference (optional)

Business Rules:
├── Insert position must be valid (0 to tunes.count)
├── New tune gets default instruments from score
├── Tempo defaults applied based on tune type if not specified:
│   ├── March → ♩ = 90
│   ├── Strathspey → ♩ = 168
│   ├── Reel → ♩ = 112
│   ├── Jig → ♩. = 96
│   └── Other types → context-dependent defaults
├── Tempo note value must be compatible with time signature
├── Layout preference inherits from document if not specified
└── Must trigger pagination recalculation

Output: Tune

Side Effects:
├── Updates Pages[] assignment structure
├── Triggers layout recalculation
├── May affect page breaks for subsequent tunes
├── Updates navigation elements
└── Caches default tempo for tune type

Error Conditions:
├── ScoreNotFoundError
├── InvalidInsertPositionError
├── IncompatibleTempoError (tempo note value incompatible with time signature)
└── PersistenceError
```

**InsertInstrumentInTuneUseCase:**
```
Input:
├── tuneId: UUID
├── instrumentType: InstrumentType
└── insertPosition: Integer

Business Rules:
├── Instrument must be compatible with existing instruments
├── Insert position within valid range
├── Must add instrument to all systems in tune
└── Must trigger system layout recalculation

Output: Void

Side Effects:
├── Updates all MusicalSystem entities in tune
├── Recalculates staff spacing and positions
├── May affect page layout if system height changes
└── Updates instrument-specific layout caches

Error Conditions:
├── TuneNotFoundError
├── IncompatibleInstrumentError
├── InvalidInsertPositionError
└── PersistenceError
```

**CreatePartUseCase:**
```
CreatePartUseCase:
Input:
├── tuneId: UUID
├── partNumber: Integer (determines both identity and sequence)
├── name: String (optional - e.g., "Intro", "Main Theme")
└── partTemplate: PartTemplate (optional)

Business Rules:
├── Part number must be unique within tune
├── Part number must be positive integer (1, 2, 3...)
├── Inserting part may renumber subsequent parts
├── Part number determines play order
├── New part inherits instruments from tune
└── Repeat behavior NOT set on part - configured via measure barlines

Output: Part

Side Effects:
├── May renumber existing parts if inserted in sequence
├── Updates tune's part list
├── Triggers pagination recalculation
└── Updates navigation elements

Error Conditions:
├── TuneNotFoundError
├── DuplicatePartNumberError
├── InvalidPartNumberError (if zero or negative)
└── PersistenceError
```

**CreateMeasureUseCase:**
```
Input:
├── partId: UUID
├── systemId: UUID
├── insertPosition: Integer
└── timeSignature: TimeSignature (optional)

Business Rules:
├── New measure inherits time signature from previous or uses provided
├── Must be added to all instruments in system simultaneously
├── Gets default note content (rests) based on time signature
└── Triggers layout recalculation for affected systems

Output: Measure

Side Effects:
├── Updates system layout cache
├── Triggers layout recalculation
└── Updates measure navigation

Error Conditions:
├── PartNotFoundError
├── SystemNotFoundError
├── InvalidInsertPositionError
└── PersistenceError
```

**AddNoteToMeasureUseCase:**
```
Input:
├── measureId: UUID
├── instrumentId: InstrumentID
├── note: Note
└── insertPosition: Integer

Business Rules:
├── Note type must match instrument (PipeNote for bagpipes, etc.)
├── Note must fit within measure duration (time signature validation)
├── Insert position must be valid
└── Must trigger layout recalculation for measure

Output: Void

Side Effects:
├── Updates measure layout cache
├── Triggers layout recalculation
└── May affect measure width and system breaks

Error Conditions:
├── MeasureNotFoundError
├── InstrumentNotFoundError
├── InvalidNoteTypeError
├── DurationOverflowError
└── PersistenceError
```

**AttachEmbellishmentToNoteUseCase:**
```
Input:
├── noteId: UUID
└── embellishment: Embellishment

Business Rules:
├── Embellishment must be compatible with note type and instrument
├── Only one embellishment per note
├── Must validate prior note constraints (for certain embellishments)
└── Must trigger layout recalculation for measure

Output: Void

Side Effects:
├── Updates note with embellishment
├── Triggers embellishment layout calculation
└── May affect measure width and spacing

Error Conditions:
├── NoteNotFoundError
├── IncompatibleEmbellishmentError
├── PriorNoteConstraintViolationError
└── PersistenceError
```

#### 3.4.3 Pagination and Layout Use Cases

**RecalculatePaginationUseCase:**
```
Input:
├── documentId: UUID
├── scope: RecalculationScope (Measure, System, Page, Document)
└── trigger: RecalculationTrigger

Business Rules:
├── Scope determines extent of recalculation needed
├── Must preserve user layout preferences where possible
├── Must respect orientation-based page break rules
└── Must maintain musical phrase integrity

Output: PaginationResult

Side Effects:
├── Updates Pages[] structure with new TuneLines assignments
├── Caches new layout calculations
└── Updates navigation elements

Trigger Types:
├── Font size change (Document scope)
├── Paper size change (Document scope)
├── Instrument addition (System scope)
├── Note spacing change (Measure scope)
└── Orientation change (Page scope)

Error Conditions:
├── DocumentNotFoundError
└── LayoutCalculationError
```

**OptimizeLayoutUseCase:**
```
Input:
├── documentId: UUID
├── optimizationCriteria: OptimizationCriteria
└── scope: OptimizationScope

Business Rules:
├── Must respect user-specified layout preferences
├── Must maintain musical integrity and readability
├── Optimization cannot violate musical phrase boundaries
└── Must consider performance vs. practice context

Output: OptimizationResult

Side Effects:
├── Updates layout caches
└── Generates optimization report

Error Conditions:
├── DocumentNotFoundError
└── OptimizationFailedError
```

### 3.5 Repository Interfaces

Platform implementations must provide concrete implementations of these repository interfaces using their native storage technologies.

#### 3.5.1 Core Data Repositories

**MusicalDocumentRepository:**
```
Interface:
├── save(document: MusicalDocument) -> Result<Void, RepositoryError>
├── load(id: UUID) -> Result<MusicalDocument, RepositoryError>
├── delete(id: UUID) -> Result<Void, RepositoryError>
├── exists(id: UUID) -> Boolean
├── search(criteria: SearchCriteria) -> Result<List<DocumentSummary>, RepositoryError>
├── getMetadata(id: UUID) -> Result<DocumentMetadata, RepositoryError>
├── listDocuments(folderId: UUID?) -> Result<List<DocumentSummary>, RepositoryError>
└── validateDocument(id: UUID) -> Result<List<ValidationError>, RepositoryError>

Requirements:
├── Support for JSON serialization with embedded layout data
├── Atomic save operations with rollback capability
├── Concurrent access protection with appropriate locking
├── Version tracking for document changes
├── Efficient partial loading for large documents
└── Platform-specific storage optimization

Error Types:
├── DocumentNotFoundError
├── DocumentCorruptedError
├── ConcurrentModificationError
├── InsufficientStorageError
└── SerializationError
```

**FolderRepository:**
```
Interface:
├── createFolder(name: String, parentId: UUID?) -> Result<Folder, RepositoryError>
├── deleteFolder(id: UUID) -> Result<Void, RepositoryError>
├── moveFolder(id: UUID, newParentId: UUID?) -> Result<Void, RepositoryError>
├── renameFolder(id: UUID, newName: String) -> Result<Void, RepositoryError>
├── getFolderContents(id: UUID) -> Result<FolderContents, RepositoryError>
├── getFolderHierarchy(rootId: UUID?) -> Result<FolderTree, RepositoryError>
├── searchFolders(name: String) -> Result<List<Folder>, RepositoryError>
└── getFolderPath(id: UUID) -> Result<List<Folder>, RepositoryError>

Business Rules:
├── Maximum depth enforcement (5 levels)
├── Circular reference prevention
├── Name uniqueness within parent folder
├── Cascade deletion with user confirmation
└── Folder sorting and display order management
```

**AudioRepository:**
```
Interface:
├── saveRecording(recording: PracticeRecording) -> Result<Void, RepositoryError>
├── loadRecording(id: UUID) -> Result<PracticeRecording, RepositoryError>
├── deleteRecording(id: UUID) -> Result<Void, RepositoryError>
├── getRecordingsForScore(scoreId: UUID) -> Result<List<PracticeRecording>, RepositoryError>
├── getRecordingsForTune(tuneId: UUID) -> Result<List<PracticeRecording>, RepositoryError>
├── getRecordingsByDateRange(range: DateRange) -> Result<List<PracticeRecording>, RepositoryError>
├── saveAudioAnalysis(id: UUID, analysis: AudioAnalysis) -> Result<Void, RepositoryError>
├── compressAudioFile(id: UUID, settings: CompressionSettings) -> Result<CompressionResult, RepositoryError>
└── exportRecording(id: UUID, format: ExportFormat) -> Result<ExportResult, RepositoryError>

Requirements:
├── Platform-specific audio format optimization
├── Automatic compression and quality management
├── Metadata embedding with musical context
├── Progress tracking for long operations
└── Background processing for non-critical operations
```

#### 3.5.2 Cloud Storage Repository

**CloudStorageRepository:**
```
Interface:
├── uploadDocument(id: UUID, path: CloudPath) -> Result<UploadResult, RepositoryError>
├── downloadDocument(path: CloudPath) -> Result<DownloadResult, RepositoryError>
├── deleteCloudDocument(path: CloudPath) -> Result<Void, RepositoryError>
├── listCloudDocuments(directory: CloudDirectory) -> Result<List<CloudFile>, RepositoryError>
├── syncDocument(id: UUID, strategy: SyncStrategy) -> Result<SyncResult, RepositoryError>
├── resolveConflicts(id: UUID, resolution: ConflictResolution) -> Result<ResolutionResult, RepositoryError>
├── getCloudQuota() -> Result<QuotaInfo, RepositoryError>
├── getCloudSyncStatus(id: UUID) -> Result<SyncStatus, RepositoryError>
└── validateCloudConnection() -> Result<ConnectionStatus, RepositoryError>

Multi-Provider Support:
├── Provider-agnostic interface implementation
├── Automatic provider selection based on availability
├── Conflict resolution with user intervention
├── Offline queuing with automatic sync retry
└── Bandwidth optimization and progress tracking

Platform-Specific Implementations:
├── iOS/macOS: iCloud Container (primary), File Provider (advanced)
├── Android: Google Drive native, Storage Access Framework
├── Windows: OneDrive native, File Provider fallback
└── Linux: Cloud provider REST APIs
```

### 3.6 Domain Services

Domain services encapsulate complex business logic that doesn't naturally belong to a single entity.

#### 3.6.1 Layout Calculation Services

**LayoutCoordinationService:**
```
Interface:
├── calculateDocumentLayout(id: UUID, context: LayoutContext) -> Result<DocumentLayoutResult, LayoutError>
├── calculatePageLayout(id: UUID, context: PageLayoutContext) -> Result<PageLayoutResult, LayoutError>
├── calculateSystemLayout(id: UUID, context: SystemLayoutContext) -> Result<SystemLayoutResult, LayoutError>
├── calculateMeasureLayout(id: UUID, context: MeasureLayoutContext) -> Result<MeasureLayoutResult, LayoutError>
├── optimizeLayoutForContext(id: UUID, context: OptimizationContext) -> Result<OptimizationResult, LayoutError>
├── validateLayoutIntegrity(id: UUID) -> Result<List<ValidationError>, LayoutError>
├── previewLayoutChanges(id: UUID, changes: LayoutChanges) -> Result<LayoutPreview, LayoutError>
└── getLayoutMetrics(id: UUID) -> Result<LayoutMetrics, LayoutError>

Coordination Logic:
├── Dependency analysis for efficient recalculation ordering
├── Parallel calculation of independent layout regions (platform-dependent)
├── Cache invalidation with minimal scope impact
├── Progressive enhancement for incremental updates
└── Rollback capability for failed layout operations
```

**PaginationService:**
```
Interface:
├── calculatePagination(id: UUID, context: PaginationContext) -> Result<PaginationResult, PaginationError>
├── optimizePageBreaks(id: UUID, criteria: OptimizationCriteria) -> Result<PageBreakResult, PaginationError>
├── validatePageBreaks(id: UUID) -> Result<List<ValidationError>, PaginationError>
├── previewPaginationChanges(id: UUID, changes: PaginationChanges) -> Result<PaginationPreview, PaginationError>
├── getPageCapacity(dimensions: PageDimensions, content: ContentType) -> CapacityInfo
├── analyzeMusicianPhrasing(id: UUID) -> Result<PhrasingAnalysis, PaginationError>
└── suggestOptimalBreaks(id: UUID, criteria: BreakCriteria) -> Result<List<BreakSuggestion>, PaginationError>

Musical Intelligence:
├── Phrase boundary detection and preservation
├── Part transition optimization for page breaks
├── Rehearsal mark and section break consideration
├── Instrument ensemble synchronization requirements
└── Performance context adaptation (solo vs. ensemble)
```

**EmbellishmentLayoutService:**
```
Interface:
├── calculateGraceNotePositions(embellishment: Embellishment, principalNote: Note) -> Result<List<Point>, LayoutError>
├── resolveEmbellishmentCollisions(measure: Measure) -> Result<CollisionResolution, LayoutError>
├── optimizeEmbellishmentSpacing(measure: Measure) -> Result<SpacingOptimization, LayoutError>
├── validateEmbellishmentLayout(embellishment: Embellishment, context: LayoutContext) -> Result<List<ValidationError>, LayoutError>
└── calculateFloatingPosition(embellishment: Embellishment, measure: Measure) -> Result<Point, LayoutError>

Layout Intelligence:
├── Context-aware grace note positioning
├── Collision detection and avoidance with adjacent elements
├── Beaming calculations for grace note groups
├── Floating before barline logic for first note embellishments
└── Instrument-specific spacing rules (pipe vs. drum embellishments)
```

#### 3.6.2 Validation Services

**MusicalValidationService:**
```
Interface:
├── validateMeasureDuration(measure: Measure, timeSignature: TimeSignature) -> Result<List<ValidationError>, ValidationError>
├── validateInstrumentRange(note: Note, instrument: Instrument) -> Result<List<ValidationError>, ValidationError>
├── validateEmbellishmentCompatibility(embellishment: Embellishment, note: Note) -> Result<List<ValidationError>, ValidationError>
├── validatePriorNoteConstraints(embellishment: Embellishment, priorNote: Note) -> Result<List<ValidationError>, ValidationError>
├── validateNoteGroupIntegrity(noteGroup: NoteGroup, notes: List<Note>) -> Result<List<ValidationError>, ValidationError>
└── validateScoreIntegrity(score: ScoreDocument) -> Result<List<ValidationError>, ValidationError>

Validation Rules:
├── Time signature consistency and duration validation
├── Key signature propagation and accidental validation
├── Embellishment compatibility with instrument types
├── Prior note constraints for conditional embellishments
├── Note group span validation (head and tail notes exist)
└── Musical logic validation (no impossible fingerings, etc.)
```

#### 3.6.3 Export Services

**ExportCoordinationService:**
```
Interface:
├── exportToPDF(id: UUID, settings: PDFExportSettings) -> Result<ExportResult, ExportError>
├── exportToPNG(id: UUID, settings: ImageExportSettings) -> Result<ExportResult, ExportError>
├── exportToSVG(id: UUID, settings: SVGExportSettings) -> Result<ExportResult, ExportError>
├── exportToMIDI(id: UUID, settings: MIDIExportSettings) -> Result<ExportResult, ExportError>
├── exportToMusicXML(id: UUID, settings: MusicXMLExportSettings) -> Result<ExportResult, ExportError>
├── batchExport(ids: List<UUID>, settings: BatchExportSettings) -> Result<BatchExportResult, ExportError>
├── validateExportSettings(settings: ExportSettings) -> Result<List<ValidationError>, ExportError>
└── getExportFormats() -> List<ExportFormat>

Quality Assurance:
├── Layout consistency validation across formats
├── SMuFL font embedding verification
├── Color profile management for print output
├── Resolution optimization for target devices
└── Metadata preservation across format conversions
```

### 3.7 Domain Events

Domain events enable loose coupling between domain components and support architectural extensibility.

#### 3.7.1 Musical Content Events

```
ScoreCreatedEvent:
├── scoreId: UUID
├── title: String
├── composer: String (optional)
├── createdBy: UserID
├── timestamp: DateTime
└── initialLayout: DocumentLayoutSettings

TuneAddedEvent:
├── scoreId: UUID
├── tuneId: UUID
├── insertPosition: Integer
├── tuneMetadata: TuneMetadata
├── triggeredBy: UserID
└── timestamp: DateTime

PartCreatedEvent:
├── tuneId: UUID
├── partId: UUID
├── partLetter: String
├── insertPosition: Integer
├── triggeredBy: UserID
└── timestamp: DateTime

NoteModifiedEvent:
├── noteId: UUID
├── measureId: UUID
├── previousState: NoteState
├── newState: NoteState
├── layoutImpact: LayoutImpactLevel
├── triggeredBy: UserID
└── timestamp: DateTime

EmbellishmentAttachedEvent:
├── noteId: UUID
├── embellishment: Embellishment
├── triggeredBy: UserID
└── timestamp: DateTime
```

#### 3.7.2 Layout and Pagination Events

```
LayoutPreferenceChangedEvent:
├── entityId: UUID
├── entityType: EntityType
├── previousPreference: LayoutPreference (optional)
├── newPreference: LayoutPreference
├── affectedScope: LayoutScope
├── recalculationRequired: Boolean
├── triggeredBy: UserID
└── timestamp: DateTime

PaginationRecalculatedEvent:
├── documentId: UUID
├── affectedPages: List<UUID>
├── previousPageCount: Integer
├── newPageCount: Integer
├── changedTuneLines: List<TuneLineChange>
├── recalculationTrigger: RecalculationTrigger
├── calculationDuration: Duration
└── timestamp: DateTime

SystemLayoutUpdatedEvent:
├── systemId: UUID
├── previousLayout: SystemLayout (optional)
├── newLayout: SystemLayout
├── layoutChanges: List<LayoutChange>
├── affectedMeasures: List<UUID>
├── cacheInvalidationRequired: List<UUID>
└── timestamp: DateTime

CacheInvalidatedEvent:
├── invalidatedEntities: List<UUID>
├── invalidationType: InvalidationType
├── invalidationScope: InvalidationScope
├── cascadeEffects: List<CascadeEffect>
├── recalculationEstimate: Duration
└── timestamp: DateTime
```

#### 3.7.3 Validation and Error Events

```
ValidationFailedEvent:
├── entityId: UUID
├── entityType: EntityType
├── validationErrors: List<ValidationError>
├── validationContext: ValidationContext
├── severity: ErrorSeverity
└── timestamp: DateTime

DomainErrorOccurredEvent:
├── errorId: UUID
├── errorType: DomainErrorType
├── entityId: UUID (optional)
├── errorSeverity: ErrorSeverity
├── errorContext: ErrorContext
├── recoveryAttempted: Boolean
├── userNotificationRequired: Boolean
└── timestamp: DateTime
```

---

## 4. Universal File Format Specification

### 4.1 .pbscore File Format

**Format Principles:**
- File Extension: `.pbscore` (Pipe Band Score)
- Single universal JSON format across all platforms
- No platform-specific information or metadata
- Human-readable and debuggable format
- Pure musical content with universal layout properties

**File Structure Overview:**
```
.pbscore (JSON format)
├── formatVersion: String (e.g., "1.0")
├── metadata: DocumentMetadata
├── documentLayout: DocumentLayoutSettings
├── tunes: List<Tune>
└── noteGroups: Dictionary<UUID, NoteGroup>
```

**Metadata Structure:**
```json
{
  "formatVersion": "1.0",
  "metadata": {
    "title": "Scotland the Brave",
    "composer": "Traditional",
    "arranger": "John Smith",
    "copyright": "Public Domain",
    "created": "2025-01-15T10:30:00Z",
    "modified": "2025-01-16T14:22:00Z"
  }
}
```

**Document Layout Structure:**
```json
{
  "documentLayout": {
    "paperSize": "A4",
    "orientation": "portrait",
    "margins": {
      "top": 20.0,
      "bottom": 20.0,
      "left": 15.0,
      "right": 15.0
    },
    "globalSpacingFactor": 1.0,
    "compressionStrategy": "moderate",
    "fontSizeAdjustment": 1.0
  }
}
```

**Tune Structure:**
```json
{
  "tunes": [
    {
      "id": "tune-uuid-1",
      "title": "Scotland the Brave",
      "composer": "Traditional",
      "tuneType": "march",
      "tempo": {
        "noteValue": "quarter",
        "beatsPerMinute": 80,
        "textualMarking": "March Time",
        "modifier": null
      },
      "keySignature": {
        "sharps": 2,
        "mode": "major"
      },
      "parts": [
        {
          "id": "part-uuid-1",
          "partNumber": 1,
          "name": null,
          "systems": [
            {
              "id": "system-uuid-1",
              "measures": [
                {
                  "id": "measure-uuid-1",
                  "openingBarline": "Single",
                  "closingBarline": "Single"
                },
                {
                  "id": "measure-uuid-2",
                  "openingBarline": "Single",
                  "closingBarline": "RepeatEnd",
                  "endingBracket": null
                }
              ]
            }
          ]
        },
        {
          "id": "part-uuid-2",
          "partNumber": 2,
          "name": null,
          "systems": [...]
        }
      ]
    }
  ]
}
```

**Note Serialization (Instrument-Specific):**
```json
{
  "notes": [
    {
      "type": "PipeNote",
      "id": "note-uuid-1",
      "noteType": "crotchet",
      "pitch": {
        "pitchClass": "D",
        "octave": 4
      },
      "durationAdjustment": 1.0,
      "embellishment": {
        "type": "DoublingEmbellishment",
        "doublingType": "upToF"
      },
      "noteGroups": ["tuplet-uuid-1"]
    },
    {
      "type": "SnareDrumNote",
      "id": "note-uuid-2",
      "noteType": "quaver",
      "hand": "Right",
      "stickTechnique": "Regular",
      "embellishment": {
        "type": "FlamEmbellishment",
        "flamTiming": "tight"
      }
    }
  ]
}
```

**Note Groups Serialization:**
```json
{
  "noteGroups": {
    "tuplet-uuid-1": {
      "type": "TupletGroup",
      "headNote": "note-uuid-1",
      "tailNote": "note-uuid-3",
      "tupletRatio": {
        "actual": 3,
        "normal": 2
      },
      "bracketStyle": "bracket",
      "numberDisplay": "number"
    },
    "tie-uuid-1": {
      "type": "TieGroup",
      "headNote": "note-uuid-5",
      "tailNote": "note-uuid-6",
      "tieDirection": "up",
      "crossSystemTie": false
    }
  }
}
```

### 4.2 Cross-Platform Compatibility Standards

**Universal File Handling Requirements:**
- Identical JSON structure read/written by all platforms
- Platform file associations for .pbscore files
- Email/cloud sharing compatibility across all platforms
- No import/export complexity or conversion layers needed
- Standard UTF-8 encoding for international character support

**Excluded Information:**
- Platform identifiers or application version metadata
- Platform-specific layout preferences or rendering hints
- Compatibility matrices or feature support flags
- Any iOS/Android/Windows/Linux specific data structures
- Cloud provider specific metadata or synchronization data

**Validation Requirements:**
- JSON schema validation before reading/writing
- Version compatibility checking (format version)
- Graceful handling of future format extensions
- Warning for deprecated fields (but still parse them)
- Automatic migration for older format versions

### 4.3 Export Format Specifications

#### 4.3.1 PDF Export Requirements

**Quality Standards:**
- Vector-based output (scalable graphics)
- SMuFL font embedding (complete subset)
- Minimum 300 DPI for professional printing
- PDF/A compliance for archival (optional)
- Metadata embedding (title, composer, copyright)

**Layout Preservation:**
- Exact visual fidelity to screen rendering
- Correct page dimensions and margins
- Proper color reproduction (RGB or CMYK)
- Print optimization (no screen-only elements)

#### 4.3.2 MusicXML Export Requirements

**Standard Compliance:**
- MusicXML 4.0 specification adherence
- Complete musical content preservation
- Layout hints where applicable
- Metadata transfer (title, composer, etc.)

**Embellishment Translation:**
- Grace note representation for all embellishments
- Annotation for pipe band specific ornaments
- Best-effort mapping to standard MusicXML elements
- Notes in comments for non-standard elements

**Limitations:**
- Some pipe band specific embellishments may require simplification
- Drum hand assignments may not transfer to all applications
- Layout preferences may not be preserved exactly
- Regional embellishment styles may lose nuance

#### 4.3.3 MIDI Export Requirements

**Basic Playback Support:**
- MIDI file format 1 (multi-track)
- Tempo preservation with changes
- Instrument mapping (General MIDI)
- Multi-track support (separate tracks per instrument)

**Embellishment Interpretation:**
- Grace notes converted to MIDI sequences
- Timing approximations for ornaments
- Duration adjustments applied to playback
- Dynamics translated to velocity changes

**Hand Assignment Translation:**
- Drum hand assignments converted to velocity patterns
- Left hand = slightly lower velocity (optional)
- Right hand = slightly higher velocity (optional)
- Alternating hand patterns preserved in timing

**Note Type Duration:**
- Accurate MIDI timing based on note types
- Duration adjustments applied for playback feel
- Tempo changes reflected in MIDI events

**Limitations:**
- Audio quality limited by MIDI synthesis
- Nuanced bagpipe ornamentation difficult to express
- Drum techniques simplified to basic MIDI
- No authentic pipe band sound without quality synthesizer

---

## 5. SMuFL Music Notation Standards

### 5.1 SMuFL Compliance Requirements

**Standard Music Font Layout (SMuFL) Specification:**
- Full compliance with SMuFL 1.4+ specification
- Unicode-based musical symbol encoding
- Proper glyph positioning and spacing
- Consistent rendering across all platforms

**Primary Font Recommendation:**
- Bravura font (recommended, SIL Open Font License)
- Petaluma font (alternative, handwritten style)
- Any SMuFL-compliant font acceptable
- Font embedding required for exports

### 5.2 Core Musical Symbols

**Note Heads (SMuFL Codepoints):**
```
Note Heads:
├── Whole Note (Semibreve): U+E0A2
├── Half Note (Minim): U+E0A3
├── Quarter Note (Crotchet): U+E0A4
├── Grace Note (Small): U+E560
└── Percussion Note: U+E0A9
```

**Stems and Flags:**
```
Stems and Flags:
├── Stem: U+E210
├── Flag 8th Up: U+E240
├── Flag 8th Down: U+E241
├── Flag 16th Up: U+E242
├── Flag 16th Down: U+E243
├── Flag 32nd Up: U+E244
└── Flag 32nd Down: U+E245
```

**Accidentals:**
```
Accidentals:
├── Sharp: U+E262
├── Flat: U+E260
├── Natural: U+E261
├── Double Sharp: U+E263
└── Double Flat: U+E264
```

**Clefs:**
```
Clefs:
├── Treble Clef: U+E050
├── Bass Clef: U+E062
├── Alto Clef: U+E05C
└── Percussion Clef: U+E069
```

**Time Signatures:**
```
Time Signatures:
├── Common Time (4/4): U+E08A
├── Cut Time (2/2): U+E08B
├── Numerals 0-9: U+E080 to U+E089
└── Time Signature Separator: (constructed)
```

**Barlines:**
```
Barlines (already specified in section 3.1.8):
├── Single: U+E030
├── Double: U+E031
├── Final: U+E032
├── Repeat Start: U+E040
├── Repeat End: U+E041
├── Repeat Both: U+E042
├── Dashed: U+E036
├── Heavy: U+E034
└── Dotted: U+E037
```

**Rests:**
```
Rests:
├── Whole Rest: U+E4E3
├── Half Rest: U+E4E4
├── Quarter Rest: U+E4E5
├── Eighth Rest: U+E4E6
├── 16th Rest: U+E4E7
├── 32nd Rest: U+E4E8
└── 64th Rest: U+E4E9
```

### 5.3 Pipe Band Specific Rendering

**Grace Note Rendering:**
- Smaller note head size (typically 60-70% of principal note)
- Slashed stem (diagonal line through stem)
- Proper spacing before principal note
- Beam connections for grace note sequences
- Consistent vertical alignment

**Embellishment Layout Rules:**
- Grace notes positioned above staff
- Minimum clearance from staff lines
- Collision avoidance with other elements
- Beaming follows standard SMuFL patterns
- Floating before barline for first note embellishments

**Staff and System Layout:**
- Standard 5-line staff for pitched instruments
- Single-line percussion staff for snare drum
- Proper staff spacing for multi-instrument systems
- System barlines span all instruments
- Clef changes handled gracefully

### 5.4 Font Metrics and Spacing

**Staff Height:**
- Staff height: 4 spaces (standard music notation)
- Space unit: Base measurement for all spacing
- Scaling factor applied consistently to all elements

**Symbol Sizing:**
- Note heads: 1.0 space units
- Grace note heads: 0.6-0.7 space units
- Accidentals: 1.5 space units height
- Clefs: 3.5-4.0 space units height
- Time signatures: 2.0 space units height

**Horizontal Spacing:**
- Minimum note spacing: 1.5 space units
- Grace note spacing: 0.5-0.8 space units
- Barline spacing: 0.3 space units
- System start spacing: 2.0 space units

**Vertical Spacing:**
- System separation: 8-12 space units
- Staff separation (multi-instrument): 6-8 space units
- Grace note elevation: 2-3 space units above staff

---

## 6. Internationalization Strategy

### 6.1 Language Support

**Target Languages (Priority Order):**
1. English (UK) - British spellings, metric system
2. English (US) - American spellings, imperial/metric mixed
3. French (FR) - European French, metric system
4. Spanish (ES) - European Spanish, metric system
5. German (DE) - Standard German, metric system

**Localization Framework Requirements:**
- Platform-native localization systems
- String externalization (no hardcoded text)
- Cultural adaptation (date formats, number formats)
- Dynamic language switching at runtime
- Fallback to English for missing translations

### 6.2 Musical Terminology Management

**Standardized Term Database:**
- Tune Types: March, Strathspey, Reel, Jig, Hornpipe, etc.
- Instruments: Bagpipes, Snare Drum, Tenor Drum, Bass Drum
- Technical Terms: Pipe band specific terminology
- Ornament Names: Balance between translation and Gaelic preservation

**Cultural Sensitivity:**
- Preserve Scottish Gaelic terms where appropriate
- Provide explanations for non-native speakers
- Respect regional variations in terminology
- Support for different literacy levels

**Translation Guidelines:**
- Technical accuracy prioritized over literal translation
- Musical terms may remain in original language
- Gaelic embellishment names preserved with translations
- Cultural context provided in help documentation

### 6.3 Regional Formatting

**Date and Time Formats:**
- UK: DD/MM/YYYY, 24-hour time
- US: MM/DD/YYYY, 12-hour time with AM/PM
- EU: DD.MM.YYYY or YYYY-MM-DD, 24-hour time
- Localized month and day names

**Number Formatting:**
- Decimal separator: Period (.) or comma (,) based on locale
- Thousands separator: Comma (,) or period (.) based on locale
- Measurement units: Metric (mm, cm) or Imperial (inches) based on region

**Paper Sizes:**
- UK/EU: A4, A3, A5 (ISO 216 standard)
- US: Letter, Legal, Tabloid (ANSI standard)
- Regional defaults based on user location

---

## 7. Testing Strategy

### 7.1 Domain Logic Testing

**Unit Testing Requirements:**
- All domain entities must have comprehensive unit tests
- Business rules validated through test cases
- Edge cases and boundary conditions tested
- Validation logic thoroughly tested
- Repository interfaces mocked for testing

**Test Coverage Targets:**
- Domain entities: 95%+ code coverage
- Use cases: 90%+ code coverage
- Validation services: 95%+ code coverage
- Repository interfaces: 80%+ (interface contracts)

### 7.2 Musical Accuracy Testing

**Notation Validation:**
- SMuFL symbol rendering accuracy
- Layout calculation correctness
- Embellishment positioning accuracy
- Multi-instrument alignment verification
- Page break logic validation

**Musical Logic Testing:**
- Time signature validation
- Duration calculations
- Pitch range validation
- Embellishment compatibility
- Prior note constraint validation

### 7.3 Cross-Platform Consistency Testing

**File Format Compatibility:**
- .pbscore files readable across all platforms
- JSON serialization/deserialization consistency
- Round-trip testing (save and reload)
- Version migration testing
- Corrupt file handling

**Domain Logic Parity:**
- Identical business rule implementation across platforms
- Consistent validation results
- Matching calculation outputs
- Uniform error handling

### 7.4 Performance Testing

**Layout Performance:**
- Large document pagination performance (100+ page scores)
- Rapid change handling (user typing speed)
- Memory usage during complex layouts
- Cache effectiveness measurement

**Repository Performance:**
- Load time for large scores
- Save time optimization
- Search performance (1000+ scores)
- Concurrent access handling

---

## 8. Security and Privacy Principles

### 8.1 Data Protection

**Encryption Requirements:**
- Sensitive local data encrypted at rest
- Cloud data encryption in transit (HTTPS/TLS)
- Optional client-side encryption before cloud upload
- Secure key storage using platform mechanisms

**Access Control:**
- User authentication for cloud features
- File-level permissions where applicable
- Collaborative editing authorization
- Audit logging for sensitive operations

### 8.2 Privacy Requirements

**Data Minimization:**
- Collect only data necessary for app functionality
- Local-first architecture minimizes cloud data
- No analytics without explicit user consent
- No third-party data sharing

**User Rights:**
- Data portability (export all user data)
- Right to deletion (complete data removal)
- Transparency (clear privacy policy)
- User control over cloud storage choices

**Regulatory Compliance:**
- GDPR compliance (EU users)
- CCPA compliance (California users)
- COPPA compliance (users under 13)
- Platform-specific privacy requirements

---

## 9. Architecture Governance

### 9.1 Domain Layer Integrity

**Mandatory Consistency Rules:**
- Domain layer identical across all platforms (behavior, not code)
- Business rules implemented consistently
- Validation logic produces identical results
- Repository interfaces respected by all implementations

**Prohibited Practices:**
- Platform-specific business logic variations
- Inconsistent validation rules across platforms
- Different calculation results between platforms
- Breaking domain layer dependency rules

### 9.2 Architectural Review Process

**Change Management:**
- Domain layer changes require architectural review
- Breaking changes require major version increment
- New use cases follow established patterns
- Repository interface changes coordinated across platforms

**Documentation Requirements:**
- This specification is the single source of truth
- Platform implementations document deviations (if any)
- Architectural decisions recorded with rationale
- API contracts versioned and maintained

### 9.3 Quality Gates

**Pre-Release Requirements:**
- All domain layer tests passing on all platforms
- Cross-platform file format compatibility verified
- Performance benchmarks met
- Security review completed
- Privacy compliance validated

---

## 10. Glossary

**Domain Terms:**
- **Embellishment**: Decorative musical element attached to a note (grace notes, ornaments)
- **Grace Note**: Small ornamental note played quickly before a principal note
- **Principal Note**: Main note to which an embellishment is attached
- **Measure/Bar**: Rhythmic unit defined by time signature
- **System**: Group of staves for multiple instruments, played simultaneously
- **Part**: Numbered section of a tune (1, 2, 3, etc.)
- **Tune**: Complete musical composition with one or more parts
- **Score**: Document containing one or more tunes

**Musical Terms:**
- **Dotted Quarter Note**: Note duration equal to 1.5 quarter notes, standard beat unit in compound time
- **Compound Time**: Time signature where beats divide into three (6/8, 9/8, 12/8)
- **Simple Time**: Time signature where beats divide into two (2/4, 3/4, 4/4)
- **Beat Unit**: Note duration representing one beat in a time signature

**Pipe Band Terms:**
- **Doubling**: Type of bagpipe embellishment with multiple grace notes
- **Grip (Leamluath)**: Specific bagpipe embellishment pattern
- **Taorluath**: Complex bagpipe embellishment with four grace notes
- **Throw**: Bagpipe embellishment thrown before a note
- **Birl**: Rapid alternating fingering pattern on bagpipes

**Drum Terms:**
- **Flam**: Drum embellishment with single grace note (opposite hand)
- **Drag**: Drum embellishment with two grace notes (both opposite hand)
- **Rough**: Drum embellishment with three grace notes (alternating hands)
- **Rudiment**: Standard drum pattern or technique
- **Sticking**: Hand pattern for drum notation (Left, Right)

**Technical Terms:**
- **SMuFL**: Standard Music Font Layout (Unicode musical symbols)
- **Clean Architecture**: Software architecture pattern with layered dependencies
- **Domain Layer**: Platform-agnostic business logic layer
- **Repository Pattern**: Data access abstraction pattern
- **Use Case**: Business operation specification
- **Entity**: Domain object with identity and lifecycle
- **Tempo Marking**: Specification of performance speed using note value and count per minute
- **Note Value (Tempo)**: Duration type used as beat unit in tempo marking (quarter, dotted quarter, etc.)
- **Beats Per Minute**: Count of specified note values played in one minute
- **Tempo Equivalence**: Mathematical relationship between different tempo markings producing same playback speed

---

## 11. References

**Musical Standards:**
- Standard Music Font Layout (SMuFL) Specification 1.4+
- MusicXML 4.0 Specification
- MIDI 1.0 Specification
- Traditional Scottish pipe band notation conventions

**Architectural Patterns:**
- Clean Architecture (Robert C. Martin)
- Domain-Driven Design (Eric Evans)
- Repository Pattern
- MVVM Pattern (platform presentations)

**Platform Documentation:**
- iOS/macOS: See `ios_macos_implementation.md`
- Android: See `android_implementation.md`
- Windows: See `windows_implementation.md`
- Linux: See `linux_implementation.md`

---

## 12. Document Maintenance

**Version History:**
- Version 1.0 (2025-09-30): Initial core domain specification

**Change Process:**
- Architectural changes require review and approval
- Domain layer changes affect all platform implementations
- Breaking changes require major version increment
- Platform teams notified of all domain changes

**Review Schedule:**
- Quarterly review of architectural decisions
- Annual comprehensive specification review
- Ad-hoc reviews for significant feature additions
- Post-release retrospectives for lessons learned

---

**End of Core Domain Architecture Specification**

Platform-specific implementations must refer to their respective implementation documents:
- `ios_macos_implementation.md` for iOS/macOS with App Store compliance
- `android_implementation.md` for Android with Play Store guidelines
- `windows_implementation.md` for Windows with Microsoft Store requirements
- `linux_implementation.md` for Linux with package manager distribution