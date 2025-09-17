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

# Section 3.1.6 - Embellishment System Architecture (Updated)

## Embellishment as Note Property

Embellishments are decorative musical elements that belong to individual notes rather than spanning between notes. They are implemented as properties of notes with sophisticated layout intelligence.

### Base Embellishment Entity

**Embellishment Component Architecture:**
```
Embellishment (Abstract Base Class)
├── Musical Properties (Domain Core)
│   ├── embellishmentType: Classification of embellishment variant
│   ├── graceNotes: Collection of grace note patterns
│   ├── traditionalName: Cultural terminology preservation
│   └── executionStyle: Performance interpretation guidance
├── Layout Properties (Domain Embedded)
│   ├── graceNotePitchLayout: Pitch positioning strategy
│   ├── handAssignment: Drum-specific hand coordination data
│   ├── distanceFromPrincipal: Spacing from main note
│   ├── interGraceSpacing: Grace note internal spacing
│   ├── verticalPlacement: Staff positioning requirements
│   ├── collisionAvoidance: Spatial conflict resolution
│   └── floatBeforeBarline: Visual positioning before barline when note is first in measure
└── Layout Methods (Behavior Interface)
    ├── calculateGraceNotePositions: Determine spatial coordinates
    ├── determineOptimalSpacing: Context-aware spacing calculation
    ├── resolveCollisions: Handle overlapping embellishments
    ├── shouldFloatBeforeBarline: Determine if embellishment should appear before barline
    └── validateLayoutFeasibility: Constraint validation
```

## Pipe Band Embellishments

### PipeBandEmbellishment Entity

**Pipe Band Specific Architecture:**
```
PipeBandEmbellishment : Embellishment
├── Musical Properties (Cultural Context)
│   ├── regionalStyle: Geographic and cultural tradition identifier
│   ├── fingeringSequence: Technical execution pattern
│   └── traditionalNotation: Historical accuracy preference
├── Layout Properties (Pipe-Specific)
│   ├── gracePitchStrategy: Pitch relationship calculation method
│   ├── fingeringOptimization: Technical execution enhancement
│   ├── staffPositionOverride: Manual positioning capability
│   └── beamingStrategy: Grace note connection approach
└── Layout Methods (Pipe-Specific Behavior)
    ├── calculateGracePitches: Determine pitch relationships
    ├── optimizeForFingeringFlow: Enhance technical execution
    └── determineStaffPositions: Calculate vertical placement
```

### Specific Pipe Band Embellishment Types

#### CutEmbellishment
**Cut Embellishment Specification:**
```
CutEmbellishment : PipeBandEmbellishment
├── cutType: Variant classification (High G cut, A cut, etc.)
└── Layout Calculation Requirements:
    ├── Grace pitch: Determined by cut type and principal note relationship
    ├── Distance: Standard grace note spacing rules application
    ├── Vertical position: Above staff with specific clearance requirements
    └── Collision handling: Spacing adjustment for nearby embellishments
```

#### StrikeEmbellishment
**Strike Embellishment Specification:**
```
StrikeEmbellishment : PipeBandEmbellishment  
├── strikeNote: Pitch specification for strike execution
├── strikeStyle: Style variant classification
└── Layout Calculation Requirements:
    ├── Grace note positioning: Placement before principal note
    ├── Pitch relationships: Specific intervals for different strike types
    ├── Beaming connections: Visual connection to principal note
    └── Staff position optimization: Readability-focused placement
```

#### DoublingEmbellishment
**Doubling Embellishment Specification:**
```
DoublingEmbellishment : PipeBandEmbellishment
├── doublingType: Variant classification (Full, half, thumb)
├── graceSequence: Grace note pattern specification (Low G, D, Low G)
├── beamingStyle: Grace note connection approach
└── Layout Calculation Requirements:
    ├── Complex grace note sequence: Multi-note positioning
    ├── Proper beaming: Connection across multiple grace notes
    ├── Spacing optimization: Readability-focused arrangement
    └── Collision avoidance: Adjacent embellishment coordination
```

#### ThrowEmbellishment
**Throw Embellishment Specification:**
```
ThrowEmbellishment : PipeBandEmbellishment
├── throwType: Variant classification (D throw, high G throw)
├── fingerPattern: Technical execution specification
└── Layout Calculation Requirements:
    ├── Grace note pattern: Type-specific note arrangements
    ├── Vertical positioning: Above staff placement
    ├── Horizontal spacing: Pre-principal note positioning
    └── Beaming style: Appropriate visual connections
```

#### TaorluathEmbellishment
**Taorluath Embellishment Specification:**
```
TaorluathEmbellishment : PipeBandEmbellishment
├── taorluathType: Variant classification (Regular, breabach)
├── complexPattern: Multi-grace note sequence specification
├── beamingPattern: Complex beaming style requirements
└── Layout Calculation Requirements:
    ├── Complex multi-grace positioning: Advanced spatial arrangement
    ├── Sophisticated beaming: Grace note group connections
    ├── Spacing optimization: Complex pattern accommodation
    └── Staff position management: Readability preservation
```

#### CrunluathEmbellishment
**Crunluath Embellishment Specification:**
```
CrunluathEmbellishment : PipeBandEmbellishment
├── crunluathType: Variant classification (Regular, a mach, breabach)
├── extendedPattern: Extended grace note sequence specification
├── traditionalSpelling: Notation style preference (traditional vs simplified)
└── Layout Calculation Requirements:
    ├── Extended grace note sequence: Long pattern management
    ├── Complex beaming and grouping: Advanced visual organization
    ├── Space-efficient layout: Optimization for lengthy sequences
    └── Traditional notation style: Historical accuracy preservation
```

## Drum Embellishments

### DrumEmbellishment Entity

**Drum-Specific Architecture:**
```
DrumEmbellishment : Embellishment
├── Musical Properties (Drum Context)
│   ├── rudimentName: Standard drum rudiment classification
│   └── stickingPattern: Hand coordination sequence specification
├── Layout Properties (Drum-Specific)
│   ├── stemDirection: Grace note stem direction (upward/downward)
│   ├── stickHeightVariation: Visual height differentiation
│   └── rudimentSpacing: Drum-specific spacing strategy
└── Layout Methods (Drum-Specific Behavior)
    ├── calculateStickHeights: Visual height determination
    └── optimizeForRudimentFlow: Technical execution enhancement
```

### Specific Drum Embellishment Types

#### FlamEmbellishment
**Flam Embellishment Specification:**
```
FlamEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 1 grace note for flam
├── handRelationship: OppositeHand (grace note always opposite from principal note)
├── flamTiming: Temporal characteristics (tight/open execution)
├── stemDirection: Upward (flam grace note stems always point upward)
└── Layout Calculation Requirements:
    ├── Single grace note positioning: Exactly one grace note before principal
    ├── Opposite hand coordination: Grace note uses opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, grace note automatically swaps
    ├── Lower visual position: Grace note positioned lower than principal stroke
    ├── Upward stem rendering: Grace note stem points upward
    ├── Horizontal offset: Precise flam appearance positioning
    ├── Hand assignment inheritance: Grace note hand derived from principal note hand
    └── Tight spacing: Spacing that clearly demonstrates flam relationship
```

#### DragEmbellishment
**Drag Embellishment Specification:**
```
DragEmbellishment : DrumEmbellishment  
├── graceNoteCount: Fixed at 2 grace notes for standard drag
├── handRelationship: OppositeHand (both grace notes opposite from principal note)
├── stemDirection: Upward (distinguishes from open drag)
└── Layout Calculation Requirements:
    ├── Two grace note positioning: Standard drag bounce representation
    ├── Opposite hand coordination: Both grace notes use opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, both grace notes automatically swap
    ├── Hand assignment inheritance: Grace note hands derived from principal note hand
    ├── Upward stem rendering: Grace note stems point upward
    ├── Tight spacing: Close grouping showing drag relationship
    └── Visual distinction: Upward stems differentiate from open drag downward stems
```

#### OpenDragEmbellishment
**Open Drag Embellishment Specification:**
```
OpenDragEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 2 grace notes for open drag
├── handRelationship: OppositeHand (grace notes always opposite from principal note)
├── openingSpacing: Temporal spacing between the two grace notes
└── Layout Calculation Requirements:
    ├── Two grace note positioning: Exactly two grace notes before principal
    ├── Opposite hand coordination: Grace notes use opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, grace notes automatically swap
    ├── Open spacing visualization: Clear temporal separation between grace notes
    └── Hand assignment inheritance: Grace note hands derived from principal note hand
```

#### RoughEmbellishment
**Rough Embellishment Specification:**
```
RoughEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 3 grace notes for rough
├── handRelationship: AlternatingPattern (starts opposite from principal, then alternates)
├── handPattern: OppositeHand -> SameHand -> OppositeHand (relative to principal note)
├── stemDirection: Upward (distinguishes rough embellishments)
└── Layout Calculation Requirements:
    ├── Three grace note positioning: Rough stroke cluster representation
    ├── Alternating hand coordination: Grace notes alternate hands starting opposite from principal
    ├── Automatic hand swapping: When principal note hand changes, entire alternating pattern swaps
    ├── Hand assignment inheritance: First grace note opposite, subsequent notes alternate
    ├── Upward stem rendering: Grace note stems point upward
    ├── Cluster spacing: Tight grouping for rough appearance
    ├── Pattern visualization: Clear indication of alternating hand pattern
    └── Principal note connection: Clear relationship to main stroke
```

#### SwissRoughEmbellishment
**Swiss Rough Embellishment Specification:**
```
SwissRoughEmbellishment : DrumEmbellishment
├── swissPattern: Swiss-specific execution pattern
├── strokeSequence: Characteristic Swiss rough stroke sequence
├── traditionalStyle: Adherence to Swiss pipe band tradition
├── stemDirection: Upward (consistent with rough embellishment family)
└── Layout Calculation Requirements:
    ├── Swiss-specific pattern: Traditional Swiss rough representation
    ├── Cultural notation style: Authentic Swiss visual formatting
    ├── Upward stem rendering: Grace note stems point upward
    ├── Pattern recognition: Distinctive Swiss rough appearance
    └── Traditional spacing: Culturally appropriate visual spacing
```

## Note Hierarchy with Instrument-Specific Properties

### Base Note Entity

**Abstract Note Base Class:**
```
Note (Abstract Base Class)
├── Musical Properties (Universal)
│   ├── noteType: Note type specification (crotchet, quaver, minim, etc.)
│   ├── durationAdjustment: Synthetic playback timing adjustment (percentage)
│   ├── embellishment: Optional embellishment property attachment
│   ├── articulation: Performance instruction collection
│   └── noteGroups: Reference collection to spanning relationships
├── Layout Properties (Universal)
│   ├── position: Spatial coordinate specification
│   ├── spacingHints: Layout guidance parameters
│   ├── visualStyle: Appearance customization options
│   └── collisionAvoidance: Spatial conflict resolution hints
└── Domain Methods (Universal Behavior Interface)
    ├── withEmbellishment: Embellishment attachment operation
    ├── withNoteType: Note type assignment (affects visual and base duration)
    ├── withDurationAdjustment: Playback timing fine-tuning
    ├── getBaseDuration: Calculate fundamental duration from note type
    ├── getEffectiveDuration: Calculate final playback duration (base + adjustment)
    ├── calculateTotalWidth: Comprehensive width calculation including embellishments
    ├── calculateEmbellishmentLayout: Embellishment spatial arrangement
    └── getTotalLayoutBounds: Complete spatial boundary determination
```

### Instrument-Specific Note Classes

#### PipeNote
**Pitched Instrument Note:**
```
PipeNote : Note
├── Musical Properties (Pipe-Specific)
│   ├── pitch: Fundamental pitch specification (required for pipes)
│   └── accidental: Pitch modification specification
├── Domain Methods (Pipe-Specific)
│   ├── withPitch: Pitch assignment transformation
│   ├── withAccidental: Accidental modification
│   └── validatePitchRange: Instrument range validation
```

#### SnareDrumNote  
**Unpitched Percussion Note:**
```
SnareDrumNote : Note
├── Musical Properties (Snare-Specific)
│   ├── hand: Hand assignment (Left, Right) - required for drums
│   ├── stickTechnique: Playing technique (Regular, BackStick)
│   └── stickHeight: Visual stick height indication
├── Domain Methods (Snare-Specific)
│   ├── withHand: Hand assignment transformation
│   ├── withStickTechnique: Stick technique assignment (regular/back-stick)
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateStickingPattern: Ergonomic pattern validation
```

#### TenorDrumNote
**Multi-Pitched Percussion Note:**
```
TenorDrumNote : Note
├── Musical Properties (Tenor-Specific)
│   ├── drumPosition: Specific drum within tenor set (1, 2, 3, 4, etc.)
│   ├── hand: Hand assignment (Left, Right) - required for drums
│   └── stickHeight: Visual stick height indication
├── Domain Methods (Tenor-Specific)
│   ├── withDrumPosition: Drum selection within set
│   ├── withHand: Hand assignment transformation
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateReachability: Physical reachability validation
```

#### BassDrumNote
**Single Unpitched Percussion Note:**
```
BassDrumNote : Note
├── Musical Properties (Bass-Specific)
│   └── hand: Hand assignment (Left, Right) - required for drums
├── Domain Methods (Bass-Specific)
│   ├── withHand: Hand assignment transformation
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateStickingPattern: Ergonomic pattern validation
```

### Layout Integration

**Embellishment Layout Coordination:**
```
Note Layout Integration Architecture:
├── calculateTotalWidth: Includes embellishment spatial requirements
├── calculateEmbellishmentLayout: Handles grace note positioning coordination
├── getTotalLayoutBounds: Encompasses complete embellishment boundaries
├── Collision detection: Considers embellishment spatial requirements
└── Staff position calculations: Accounts for embellishment height requirements
```

## Validation Integration

### EmbellishmentValidationRules

**Validation Architecture:**
```
EmbellishmentValidationRules
├── graceNoteCompatibility: Grace note relationship validation
├── fingeringPossibility: Technical execution feasibility for pipe band embellishments
├── traditionalAccuracy: Cultural and historical accuracy validation
├── layoutFeasibility: Spatial constraint satisfaction verification
├── performancePracticality: Playability assessment and verification
├── stylisticConsistency: Regional style adherence validation
└── musicalIntegrity: Musical logic and theory preservation validation
```

### Cultural Sensitivity and Accuracy

**Regional Style Validation:**
- Highland style embellishment patterns and conventions
- Border piping traditions and variations
- Competition vs traditional performance contexts
- Educational progression appropriateness
- Historical accuracy for traditional tunes

**Traditional Terminology Preservation:**
- Gaelic names maintained alongside English descriptions
- Traditional fingering patterns documented and preserved
- Regional pronunciation guides for embellishment names
- Cultural context provided for non-native practitioners
- Respectful presentation of Scottish musical heritage

This embellishment architecture treats decorative elements as integral properties of their principal notes while providing sophisticated layout intelligence for complex grace note patterns, hand assignments, and collision resolution. The system respects traditional pipe band terminology and cultural practices while enabling modern digital notation capabilities.


## 3.1.7 Note Group System Architecture

### Note Group Base Entity

Note groups represent musical relationships that span between two notes, such as slurs, ties, tuplets, and dynamic markings. They provide a unified system for managing connections between notes while maintaining referential integrity.

```
NoteGroup (Abstract Base Class)
├── Musical Properties (Domain Core)
│   ├── id: UUID
│   ├── headNote: NoteID
│   ├── tailNote: NoteID
│   ├── groupType: NoteGroupType
│   └── musicalFunction: MusicalFunction
├── Layout Properties (Domain Embedded)
│   ├── spanningLine: SpanningLineStyle?
│   ├── groupSymbol: SMuFLCodepoint?
│   ├── verticalPosition: VerticalPlacement
│   └── horizontalAlignment: HorizontalAlignment
└── Domain Methods
    ├── isHeadNote(NoteID) -> Bool
    ├── isTailNote(NoteID) -> Bool
    └── validateNoteSequence(context: MusicalContext) -> [ValidationError]
```

### Concrete Note Group Implementations

**Tuplet Group** (Essential for pipe band music with frequent triplets and duplets):
```
TupletGroup : NoteGroup
├── Musical Properties
│   ├── tupletRatio: TupletRatio (3:2, 2:3, etc.)
│   ├── bracketStyle: TupletBracketStyle
│   ├── numberDisplay: TupletNumberDisplay
│   └── rhythmicGrouping: RhythmicPattern
├── Layout Properties
│   ├── bracketClearance: CGFloat
│   ├── numberPosition: NumberPosition
│   └── spanningBracket: BracketStyle
└── Domain Methods
    ├── applyTupletRatio(Duration) -> Duration
    ├── calculateBracketBounds([NotePosition]) -> CGRect
    └── validateTupletSequence([Note]) -> [ValidationError]
```

**Slur Group** (For musical phrasing and legato connections):
```
SlurGroup : NoteGroup
├── Musical Properties
│   ├── slurType: SlurType (slur, phrase)
│   ├── curvatureDirection: CurvatureDirection
│   ├── phraseBoundary: Bool
│   └── musicalIntention: PhrasalFunction
├── Layout Properties
│   ├── curvatureHeight: CGFloat
│   ├── endpointAdjustment: EndpointAdjustment
│   └── collisionAvoidance: CollisionStrategy
└── Domain Methods
    ├── calculateSlurCurve([NotePosition]) -> BezierPath
    ├── determineOptimalCurvature([Note]) -> CurvatureParameters
    └── validateSlurPlacement([Note]) -> [ValidationError]
```

**Tie Group** (Replacing the previous tie property on notes):
```
TieGroup : NoteGroup
├── Musical Properties
│   ├── tieType: TieType (start, continue, end)
│   ├── tieDirection: TieDirection
│   └── crossSystemTie: Bool
├── Layout Properties
│   ├── tieThickness: CGFloat
│   ├── tieHeight: CGFloat
│   └── systemBreakHandling: SystemBreakStyle
└── Domain Methods
    ├── validatePitchConsistency([Note]) -> [ValidationError]
    ├── calculateTieCurve(startPos: CGPoint, endPos: CGPoint) -> BezierPath
    └── handleSystemBreak() -> SystemBreakTieResult
```

**Dynamic Group** (For crescendos, diminuendos, and other dynamic markings):
```
DynamicGroup : NoteGroup
├── Musical Properties
│   ├── dynamicType: DynamicType (cresc, dim, etc.)
│   ├── startDynamic: DynamicLevel?
│   ├── endDynamic: DynamicLevel?
│   └── curvature: DynamicCurvature
├── Layout Properties
│   ├── hairpinThickness: CGFloat
│   ├── textPosition: TextPosition
│   └── verticalOffset: CGFloat
└── Domain Methods
    ├── calculateHairpinPath([NotePosition]) -> HairpinPath
    ├── determineDynamicTextPlacement() -> TextPlacement
    └── validateDynamicProgression([Note]) -> [ValidationError]
```

### Note Integration

Notes maintain a simple list of note group references, eliminating redundant properties:

```
Note (Updated)
├── Musical Properties (Domain Core)
│   ├── pitch: Pitch
│   ├── duration: Duration
│   ├── embellishment: Embellishment?
│   ├── accidental: Accidental?
│   ├── articulation: [Articulation]
│   └── noteGroups: [NoteGroupID]          // Single list of all note groups
├── Layout Properties (Domain Embedded)
│   ├── [existing layout properties...]
└── Domain Methods
    ├── [existing methods...]
    ├── addToNoteGroup(NoteGroupID) -> Void
    ├── removeFromNoteGroup(NoteGroupID) -> Void
    ├── getNoteGroupsByType(NoteGroupType) -> [NoteGroupID]
    ├── getTies() -> [TieGroup]             // Convenience method
    ├── getSlurs() -> [SlurGroup]           // Convenience method
    └── getDynamics() -> [DynamicGroup]     // Convenience method
```

**Eliminated Properties:**
- ~~`tie: TieType?`~~ - Now represented as `TieGroup` in `noteGroups`

### Document Storage

Note groups are stored at the document level for efficient lookup while notes maintain direct references:

```
ScoreDocument (Updated)
├── Metadata (title, composer, version, sync status, etc.)
├── Pages[] (physical page layout assignments)
├── DocumentLayoutSettings (global layout preferences)
├── noteGroups: [UUID: NoteGroup]          // Note group registry
├── Tunes[] (1 or more tunes)
└── Domain Methods
    ├── [existing methods...]
    ├── addNoteGroup(NoteGroup) -> Void
    ├── removeNoteGroup(UUID) -> Void
    ├── getNoteGroup(UUID) -> NoteGroup?
    └── validateNoteGroupIntegrity() -> [ValidationError]
```

### Processing Pattern

Note groups are processed naturally during measure iteration without requiring separate spanned note collections:

```
// Layout processing pattern
func layoutMeasure(measure: Measure) {
    var activeTuplets: [TupletGroup] = []
    var activeSlurs: [SlurGroup] = []
    
    for note in measure.notes {
        // Start new groups
        for groupID in note.noteGroups {
            if let group = document.getNoteGroup(groupID) {
                if group.isHeadNote(note.id) {
                    // Begin processing this group
                }
            }
        }
        
        // Apply effects from active groups
        // Process note layout with group influence
        
        // End completed groups
        activeTuplets.removeAll { $0.isTailNote(note.id) }
        activeSlurs.removeAll { $0.isTailNote(note.id) }
    }
}
```

### Repository Integration

```
MusicalDocumentRepository (Updated)
├── [existing methods...]
├── saveNoteGroup(NoteGroup) -> SaveResult
├── deleteNoteGroup(UUID) -> DeleteResult
├── getNoteGroupsForNote(NoteID) -> [NoteGroup]
├── validateNoteGroupIntegrity() -> [ValidationError]
└── optimizeNoteGroupLayout() -> OptimizationResult
```

### Validation Rules

```
NoteGroupValidationRules
├── headTailNoteExistence: ExistenceRule
├── noteSequenceIntegrity: SequenceRule
├── groupTypeConsistency: TypeConsistencyRule
├── musicalLogicValidation: MusicalLogicRule
└── layoutFeasibilityCheck: LayoutFeasibilityRule
```

This note group architecture provides a unified system for managing musical relationships while maintaining clean separation between connection types (note groups) and decorative elements (embellishments). The processing pattern naturally handles spanned notes through iteration without requiring redundant storage or complex query methods.


### 3.1.8 Layout Value Objects

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

### 3.1.9 Layout Configuration Entities

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

### 3.1.10 Musical Validation Rules

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

### 3.1.11 Performance and Caching Entities

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

# Section 3.1.6 - Embellishment System Architecture (Updated)

## Embellishment as Note Property

Embellishments are decorative musical elements that belong to individual notes rather than spanning between notes. They are implemented as properties of notes with sophisticated layout intelligence.

### Base Embellishment Entity

**Embellishment Component Architecture:**
```
Embellishment (Abstract Base Class)
├── Musical Properties (Domain Core)
│   ├── embellishmentType: Classification of embellishment variant
│   ├── graceNotes: Collection of grace note patterns
│   ├── traditionalName: Cultural terminology preservation
│   └── executionStyle: Performance interpretation guidance
├── Layout Properties (Domain Embedded)
│   ├── graceNotePitchLayout: Pitch positioning strategy
│   ├── handAssignment: Drum-specific hand coordination data
│   ├── distanceFromPrincipal: Spacing from main note
│   ├── interGraceSpacing: Grace note internal spacing
│   ├── verticalPlacement: Staff positioning requirements
│   ├── collisionAvoidance: Spatial conflict resolution
│   └── floatBeforeBarline: Visual positioning before barline when note is first in measure
└── Layout Methods (Behavior Interface)
    ├── calculateGraceNotePositions: Determine spatial coordinates
    ├── determineOptimalSpacing: Context-aware spacing calculation
    ├── resolveCollisions: Handle overlapping embellishments
    ├── shouldFloatBeforeBarline: Determine if embellishment should appear before barline
    └── validateLayoutFeasibility: Constraint validation
```

## Pipe Band Embellishments

### PipeBandEmbellishment Entity

**Pipe Band Specific Architecture:**
```
PipeBandEmbellishment : Embellishment
├── Musical Properties (Cultural Context)
│   ├── regionalStyle: Geographic and cultural tradition identifier
│   ├── fingeringSequence: Technical execution pattern
│   └── traditionalNotation: Historical accuracy preference
├── Layout Properties (Pipe-Specific)
│   ├── gracePitchStrategy: Pitch relationship calculation method
│   ├── fingeringOptimization: Technical execution enhancement
│   ├── staffPositionOverride: Manual positioning capability
│   └── beamingStrategy: Grace note connection approach
└── Layout Methods (Pipe-Specific Behavior)
    ├── calculateGracePitches: Determine pitch relationships
    ├── optimizeForFingeringFlow: Enhance technical execution
    └── determineStaffPositions: Calculate vertical placement
```

### Specific Pipe Band Embellishment Types

#### CutEmbellishment
**Cut Embellishment Specification:**
```
CutEmbellishment : PipeBandEmbellishment
├── cutType: Variant classification (High G cut, A cut, etc.)
└── Layout Calculation Requirements:
    ├── Grace pitch: Determined by cut type and principal note relationship
    ├── Distance: Standard grace note spacing rules application
    ├── Vertical position: Above staff with specific clearance requirements
    └── Collision handling: Spacing adjustment for nearby embellishments
```

#### StrikeEmbellishment
**Strike Embellishment Specification:**
```
StrikeEmbellishment : PipeBandEmbellishment  
├── strikeNote: Pitch specification for strike execution
├── strikeStyle: Style variant classification
└── Layout Calculation Requirements:
    ├── Grace note positioning: Placement before principal note
    ├── Pitch relationships: Specific intervals for different strike types
    ├── Beaming connections: Visual connection to principal note
    └── Staff position optimization: Readability-focused placement
```

#### DoublingEmbellishment
**Doubling Embellishment Specification:**
```
DoublingEmbellishment : PipeBandEmbellishment
├── doublingType: Variant classification (Full, half, thumb)
├── graceSequence: Grace note pattern specification (Low G, D, Low G)
├── beamingStyle: Grace note connection approach
└── Layout Calculation Requirements:
    ├── Complex grace note sequence: Multi-note positioning
    ├── Proper beaming: Connection across multiple grace notes
    ├── Spacing optimization: Readability-focused arrangement
    └── Collision avoidance: Adjacent embellishment coordination
```

#### ThrowEmbellishment
**Throw Embellishment Specification:**
```
ThrowEmbellishment : PipeBandEmbellishment
├── throwType: Variant classification (D throw, high G throw)
├── fingerPattern: Technical execution specification
└── Layout Calculation Requirements:
    ├── Grace note pattern: Type-specific note arrangements
    ├── Vertical positioning: Above staff placement
    ├── Horizontal spacing: Pre-principal note positioning
    └── Beaming style: Appropriate visual connections
```

#### TaorluathEmbellishment
**Taorluath Embellishment Specification:**
```
TaorluathEmbellishment : PipeBandEmbellishment
├── taorluathType: Variant classification (Regular, breabach)
├── complexPattern: Multi-grace note sequence specification
├── beamingPattern: Complex beaming style requirements
└── Layout Calculation Requirements:
    ├── Complex multi-grace positioning: Advanced spatial arrangement
    ├── Sophisticated beaming: Grace note group connections
    ├── Spacing optimization: Complex pattern accommodation
    └── Staff position management: Readability preservation
```

#### CrunluathEmbellishment
**Crunluath Embellishment Specification:**
```
CrunluathEmbellishment : PipeBandEmbellishment
├── crunluathType: Variant classification (Regular, a mach, breabach)
├── extendedPattern: Extended grace note sequence specification
├── traditionalSpelling: Notation style preference (traditional vs simplified)
└── Layout Calculation Requirements:
    ├── Extended grace note sequence: Long pattern management
    ├── Complex beaming and grouping: Advanced visual organization
    ├── Space-efficient layout: Optimization for lengthy sequences
    └── Traditional notation style: Historical accuracy preservation
```

## Drum Embellishments

### DrumEmbellishment Entity

**Drum-Specific Architecture:**
```
DrumEmbellishment : Embellishment
├── Musical Properties (Drum Context)
│   ├── rudimentName: Standard drum rudiment classification
│   └── stickingPattern: Hand coordination sequence specification
├── Layout Properties (Drum-Specific)
│   ├── stemDirection: Grace note stem direction (upward/downward)
│   ├── stickHeightVariation: Visual height differentiation
│   └── rudimentSpacing: Drum-specific spacing strategy
└── Layout Methods (Drum-Specific Behavior)
    ├── calculateStickHeights: Visual height determination
    └── optimizeForRudimentFlow: Technical execution enhancement
```

### Specific Drum Embellishment Types

#### FlamEmbellishment
**Flam Embellishment Specification:**
```
FlamEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 1 grace note for flam
├── handRelationship: OppositeHand (grace note always opposite from principal note)
├── flamTiming: Temporal characteristics (tight/open execution)
├── stemDirection: Upward (flam grace note stems always point upward)
└── Layout Calculation Requirements:
    ├── Single grace note positioning: Exactly one grace note before principal
    ├── Opposite hand coordination: Grace note uses opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, grace note automatically swaps
    ├── Lower visual position: Grace note positioned lower than principal stroke
    ├── Upward stem rendering: Grace note stem points upward
    ├── Horizontal offset: Precise flam appearance positioning
    ├── Hand assignment inheritance: Grace note hand derived from principal note hand
    └── Tight spacing: Spacing that clearly demonstrates flam relationship
```

#### DragEmbellishment
**Drag Embellishment Specification:**
```
DragEmbellishment : DrumEmbellishment  
├── graceNoteCount: Fixed at 2 grace notes for standard drag
├── handRelationship: OppositeHand (both grace notes opposite from principal note)
├── stemDirection: Upward (distinguishes from open drag)
└── Layout Calculation Requirements:
    ├── Two grace note positioning: Standard drag bounce representation
    ├── Opposite hand coordination: Both grace notes use opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, both grace notes automatically swap
    ├── Hand assignment inheritance: Grace note hands derived from principal note hand
    ├── Upward stem rendering: Grace note stems point upward
    ├── Tight spacing: Close grouping showing drag relationship
    └── Visual distinction: Upward stems differentiate from open drag downward stems
```

#### OpenDragEmbellishment
**Open Drag Embellishment Specification:**
```
OpenDragEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 2 grace notes for open drag
├── handRelationship: OppositeHand (grace notes always opposite from principal note)
├── openingSpacing: Temporal spacing between the two grace notes
└── Layout Calculation Requirements:
    ├── Two grace note positioning: Exactly two grace notes before principal
    ├── Opposite hand coordination: Grace notes use opposite hand from principal note
    ├── Automatic hand swapping: When principal note hand changes, grace notes automatically swap
    ├── Open spacing visualization: Clear temporal separation between grace notes
    └── Hand assignment inheritance: Grace note hands derived from principal note hand
```

#### RoughEmbellishment
**Rough Embellishment Specification:**
```
RoughEmbellishment : DrumEmbellishment
├── graceNoteCount: Fixed at 3 grace notes for rough
├── handRelationship: AlternatingPattern (starts opposite from principal, then alternates)
├── handPattern: OppositeHand -> SameHand -> OppositeHand (relative to principal note)
├── stemDirection: Upward (distinguishes rough embellishments)
└── Layout Calculation Requirements:
    ├── Three grace note positioning: Rough stroke cluster representation
    ├── Alternating hand coordination: Grace notes alternate hands starting opposite from principal
    ├── Automatic hand swapping: When principal note hand changes, entire alternating pattern swaps
    ├── Hand assignment inheritance: First grace note opposite, subsequent notes alternate
    ├── Upward stem rendering: Grace note stems point upward
    ├── Cluster spacing: Tight grouping for rough appearance
    ├── Pattern visualization: Clear indication of alternating hand pattern
    └── Principal note connection: Clear relationship to main stroke
```

#### SwissRoughEmbellishment
**Swiss Rough Embellishment Specification:**
```
SwissRoughEmbellishment : DrumEmbellishment
├── swissPattern: Swiss-specific execution pattern
├── strokeSequence: Characteristic Swiss rough stroke sequence
├── traditionalStyle: Adherence to Swiss pipe band tradition
├── stemDirection: Upward (consistent with rough embellishment family)
└── Layout Calculation Requirements:
    ├── Swiss-specific pattern: Traditional Swiss rough representation
    ├── Cultural notation style: Authentic Swiss visual formatting
    ├── Upward stem rendering: Grace note stems point upward
    ├── Pattern recognition: Distinctive Swiss rough appearance
    └── Traditional spacing: Culturally appropriate visual spacing
```

## Note Hierarchy with Instrument-Specific Properties

### Base Note Entity

**Abstract Note Base Class:**
```
Note (Abstract Base Class)
├── Musical Properties (Universal)
│   ├── noteType: Note type specification (crotchet, quaver, minim, etc.)
│   ├── durationAdjustment: Synthetic playback timing adjustment (percentage)
│   ├── embellishment: Optional embellishment property attachment
│   ├── articulation: Performance instruction collection
│   └── noteGroups: Reference collection to spanning relationships
├── Layout Properties (Universal)
│   ├── position: Spatial coordinate specification
│   ├── spacingHints: Layout guidance parameters
│   ├── visualStyle: Appearance customization options
│   └── collisionAvoidance: Spatial conflict resolution hints
└── Domain Methods (Universal Behavior Interface)
    ├── withEmbellishment: Embellishment attachment operation
    ├── withNoteType: Note type assignment (affects visual and base duration)
    ├── withDurationAdjustment: Playback timing fine-tuning
    ├── getBaseDuration: Calculate fundamental duration from note type
    ├── getEffectiveDuration: Calculate final playback duration (base + adjustment)
    ├── calculateTotalWidth: Comprehensive width calculation including embellishments
    ├── calculateEmbellishmentLayout: Embellishment spatial arrangement
    └── getTotalLayoutBounds: Complete spatial boundary determination
```

### Instrument-Specific Note Classes

#### PipeNote
**Pitched Instrument Note:**
```
PipeNote : Note
├── Musical Properties (Pipe-Specific)
│   ├── pitch: Fundamental pitch specification (required for pipes)
│   └── accidental: Pitch modification specification
├── Domain Methods (Pipe-Specific)
│   ├── withPitch: Pitch assignment transformation
│   ├── withAccidental: Accidental modification
│   └── validatePitchRange: Instrument range validation
```

#### SnareDrumNote  
**Unpitched Percussion Note:**
```
SnareDrumNote : Note
├── Musical Properties (Snare-Specific)
│   ├── hand: Hand assignment (Left, Right) - required for drums
│   ├── stickTechnique: Playing technique (Regular, BackStick)
│   └── stickHeight: Visual stick height indication
├── Domain Methods (Snare-Specific)
│   ├── withHand: Hand assignment transformation
│   ├── withStickTechnique: Stick technique assignment (regular/back-stick)
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateStickingPattern: Ergonomic pattern validation
```

#### TenorDrumNote
**Multi-Pitched Percussion Note:**
```
TenorDrumNote : Note
├── Musical Properties (Tenor-Specific)
│   ├── drumPosition: Specific drum within tenor set (1, 2, 3, 4, etc.)
│   ├── hand: Hand assignment (Left, Right) - required for drums
│   └── stickHeight: Visual stick height indication
├── Domain Methods (Tenor-Specific)
│   ├── withDrumPosition: Drum selection within set
│   ├── withHand: Hand assignment transformation
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateReachability: Physical reachability validation
```

#### BassDrumNote
**Single Unpitched Percussion Note:**
```
BassDrumNote : Note
├── Musical Properties (Bass-Specific)
│   └── hand: Hand assignment (Left, Right) - required for drums
├── Domain Methods (Bass-Specific)
│   ├── withHand: Hand assignment transformation
│   ├── swapHands: Automatic hand coordination with embellishments
│   └── validateStickingPattern: Ergonomic pattern validation
```

### Layout Integration

**Embellishment Layout Coordination:**
```
Note Layout Integration Architecture:
├── calculateTotalWidth: Includes embellishment spatial requirements
├── calculateEmbellishmentLayout: Handles grace note positioning coordination
├── getTotalLayoutBounds: Encompasses complete embellishment boundaries
├── Collision detection: Considers embellishment spatial requirements
└── Staff position calculations: Accounts for embellishment height requirements
```

## Validation Integration

### EmbellishmentValidationRules

**Validation Architecture:**
```
EmbellishmentValidationRules
├── graceNoteCompatibility: Grace note relationship validation
├── fingeringPossibility: Technical execution feasibility for pipe band embellishments
├── traditionalAccuracy: Cultural and historical accuracy validation
├── layoutFeasibility: Spatial constraint satisfaction verification
├── performancePracticality: Playability assessment and verification
├── stylisticConsistency: Regional style adherence validation
└── musicalIntegrity: Musical logic and theory preservation validation
```

### Cultural Sensitivity and Accuracy

**Regional Style Validation:**
- Highland style embellishment patterns and conventions
- Border piping traditions and variations
- Competition vs traditional performance contexts
- Educational progression appropriateness
- Historical accuracy for traditional tunes

**Traditional Terminology Preservation:**
- Gaelic names maintained alongside English descriptions
- Traditional fingering patterns documented and preserved
- Regional pronunciation guides for embellishment names
- Cultural context provided for non-native practitioners
- Respectful presentation of Scottish musical heritage

This embellishment architecture treats decorative elements as integral properties of their principal notes while providing sophisticated layout intelligence for complex grace note patterns, hand assignments, and collision resolution. The system respects traditional pipe band terminology and cultural practices while enabling modern digital notation capabilities.


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

# Section 4 - Data Layer Implementation (Updated)

## 4.1 Simplified Storage Architecture

### Two-Tier Storage Model for iOS/macOS

**Prerequisites**: iCloud account required for optimal experience (standard for 95%+ of target users)

Users can select between two storage approaches:

#### 4.1.1 iCloud Container + Real-time Collaboration (Default/Recommended)
- **Primary Storage**: CloudKit + CKShare integration with iCloud container
- **Synchronization**: Automatic sync across all user's Apple devices using NSPersistentCloudKitContainer
- **Collaboration**: Real-time collaborative editing between iOS/macOS users via CKShare
- **Sharing UI**: Native iOS/macOS sharing interfaces (Messages, Mail, AirDrop)
- **Conflict Resolution**: CloudKit automatic conflict resolution with custom musical logic
- **Offline Support**: Local changes queued for sync when connection restored
- **Document Versioning**: Native iCloud document versioning and "Revert To" functionality
- **Zero Configuration**: Works automatically with user's existing iCloud account

#### 4.1.2 File Provider + Universal Cloud Support (Advanced Users)
- **Architecture**: Document-based app using UIDocument/NSDocument framework
- **Cloud Providers**: Universal support (OneDrive, Google Drive, Dropbox, Box, etc.)
- **File Integration**: Files appear as native directories through File Provider extensions
- **Collaboration**: File system-based collaboration through shared cloud folders
- **Conflict Handling**: Cloud provider native conflict resolution (file versioning)
- **Cross-Platform**: Works with any cloud provider across all platforms
- **Use Cases**: Enterprise requirements, cross-platform band collaboration, power user preferences

### Android/Windows/Linux Storage Architecture

**Three-Tier Model for Non-Apple Platforms:**

#### 4.1.3 Local Only + Manual Sharing
- **Primary Storage**: Platform-native local storage (Room on Android, Entity Framework Core on Windows, SQLite on Linux)
- **Document Format**: .pbscore files stored in application documents directory
- **Sharing Method**: Export/import workflow for sharing via email, messaging, or file transfer
- **File Operations**: Standard file system operations (copy, move, rename, delete)
- **Backup Strategy**: Platform-native backup systems
- **Offline Capability**: 100% offline functionality with no cloud dependency

#### 4.1.4 Platform-Native Cloud Integration
- **Android**: Google Drive native integration with Storage Access Framework
- **Windows**: OneDrive native integration with Windows Storage APIs
- **Linux**: Cloud provider REST API integration (Google Drive, Dropbox, OneDrive)
- **Collaboration**: File system-based collaboration through shared cloud folders
- **Conflict Handling**: Cloud provider native conflict resolution

#### 4.1.5 File Provider + Universal Cloud Support
- **Universal Support**: Any cloud provider through platform file systems
- **Cross-Platform Compatibility**: Works with any cloud provider across all platforms
- **Enterprise Integration**: Corporate cloud storage requirements
- **Mixed Platform Bands**: Collaboration across different operating systems

## 4.2 Document-Based Architecture Pattern

### Apple Ecosystem Implementation

**Core Framework Choice:**
- **UIDocument (iOS)** and **NSDocument (macOS)** for both storage modes
- Document browser integration for universal file access
- Native document lifecycle management (auto-save, version history)
- Automatic support for all File Provider extensions when in File Provider mode
- iCloud container integration when in iCloud mode
- Quick Look integration for score previews

**Repository Architecture:**
```
DocumentRepository Protocol:
├── createNewDocument(title, storageMode) -> DocumentReference
├── openDocument(documentReference) -> Document
├── saveDocument(document) -> SaveResult  
├── duplicateDocument(sourceReference) -> DocumentReference
├── deleteDocument(reference) -> DeleteResult
└── listDocuments(storageMode) -> [DocumentReference]
```

**Benefits of Simplified Apple Architecture:**
- **Streamlined User Experience**: Only two clear choices instead of three
- **iCloud Integration**: Leverages user's existing Apple ecosystem investment
- **Professional Collaboration**: Real-time editing with other band members on Apple devices
- **Advanced Flexibility**: File Provider mode for specific enterprise or cross-platform needs
- **Reduced Complexity**: Eliminates rarely-used "local only" mode
- **Better Performance**: iCloud container optimized for document-based apps

### First Launch Experience

**iCloud Account Detection:**
```
Startup Flow:
├── Check iCloud account availability
├── If iCloud available (95% of users):
│   └── Default to iCloud Container mode with collaboration features
├── If iCloud unavailable:
│   ├── Prompt user to enable iCloud (recommended)
│   └── Offer File Provider mode as alternative
└── Advanced users can switch to File Provider mode in settings
```

**Prerequisites Communication:**
- App Store description: "Requires iCloud account for document sync and collaboration"
- First launch: Seamless setup for users with iCloud accounts
- Clear explanation of collaboration benefits for iCloud users
- File Provider option clearly marked as "Advanced" for specific use cases

### File-Based Storage Rationale

**Decision: Simple files over SQLite for document storage**

**Why Files Are Better for Musical Scores:**
- Scores are conceptually single documents (like Word docs or PDFs)
- Natural file operations users expect (copy, move, rename, share, backup)
- Platform file associations and Quick Look integration
- Standard backup through file system operations (iCloud, File Provider backups)
- No database complexity, migrations, or ORM overhead
- Cross-platform sharing through standard file operations

**Repository Implementation Pattern:**
```
iOS/macOS DocumentRepository:
├── iCloud Container mode: CloudKit document storage with real-time sync
├── File Provider mode: UIDocument/NSDocument with universal cloud support
├── JSON serialization for .pbscore format in both modes
├── Automatic conflict resolution through CloudKit or cloud provider
├── Document validation and recovery procedures
└── Cross-mode migration support for advanced users
```

## 4.3 Platform-Specific Storage Implementation

### 4.3.1 iOS/macOS Implementation (Simplified)
- **iCloud Container Mode**: CloudKit + CKShare for real-time collaboration, NSPersistentCloudKitContainer for automatic sync
- **File Provider Mode**: UIDocument/NSDocument with automatic integration for all installed cloud providers
- **App Preferences**: Lightweight Core Data store for app-level settings and recently opened files only
- **Security**: Keychain for sensitive settings, app sandbox with iCloud container or File Provider access
- **Migration Support**: Seamless switching between iCloud Container and File Provider modes

### 4.3.2 Android Implementation  
- **Room Database**: App preferences and metadata only
- **Storage Access Framework**: Primary document access across all storage modes
- **Intent Filters**: .pbscore file association and sharing integration
- **Document Provider**: Integration with Google Drive, OneDrive, Dropbox
- **Security**: Android Keystore for sensitive data, scoped storage compliance

### 4.3.3 Windows Implementation
- **Entity Framework Core**: App preferences with SQLite backing store
- **WinUI Document Model**: Document-based app architecture
- **File Type Association**: .pbscore registration in app manifest
- **Cloud Integration**: Native OneDrive API with File Provider fallback
- **Security**: Windows Credential Manager integration

### 4.3.4 Linux Implementation
- **SQLite**: Direct SQLite integration for app preferences
- **File System Integration**: Standard file operations with cloud folder monitoring
- **MIME Type Registration**: .pbscore file association through desktop files
- **Cloud Provider APIs**: Direct REST API integration for major providers
- **Security**: Platform keyring integration (GNOME Keyring, KWallet)

## 4.4 Universal File Format Specification

### 4.4.1 .pbscore File Format

**Format Principles:**
- **File Extension**: `.pbscore` (Pipe Band Score)
- Single universal JSON format across all platforms
- No platform-specific information or metadata
- No compatibility modes or format variants
- Pure musical content with universal layout properties
- Human-readable and debuggable format

**Example .pbscore Structure:**
```json
{
  "formatVersion": "1.0",
  "metadata": {
    "title": "Scotland the Brave",
    "composer": "Traditional",
    "created": "2024-01-15T10:30:00Z",
    "modified": "2024-01-16T14:22:00Z"
  },
  "documentLayout": {
    "paperSize": "A4",
    "orientation": "portrait", 
    "margins": { "top": 20, "bottom": 20, "left": 15, "right": 15 }
  },
  "tunes": [
    {
      "id": "tune-1",
      "title": "Scotland the Brave",
      "type": "march",
      "parts": [ /* musical content */ ]
    }
  ]
}
```

### 4.4.2 Cross-Platform Compatibility Standards

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

### 4.4.3 File Sharing Integration

**Platform-Specific File Association Setup:**

**iOS/macOS UTI Declaration Requirements:**
```
Universal Type Identifier Configuration:
├── UTTypeIdentifier: com.yourapp.pbscore
├── UTTypeDescription: Pipe Band Score
├── FileExtension: pbscore
├── MIMEType: application/pbscore
├── ConformsTo: public.data, public.content
└── DocumentIconIntegration: Custom document icon registration
```

**Android Intent Filter Requirements:**
```
Intent Filter Configuration:
├── Action: android.intent.action.VIEW
├── Category: android.intent.category.DEFAULT
├── MIMEType: application/pbscore
├── Schemes: file, content (for multiple content sources)
└── Priority: High priority for .pbscore file handling
```

**Windows File Association Requirements:**
```
File Type Association Configuration:
├── Extension Category: windows.fileTypeAssociation
├── FileType: .pbscore
├── DisplayName: Pipe Band Score
├── DefaultIcon: Application-specific icon
└── ExecutableIntegration: Launch application with file parameter
```

**Linux MIME Type Registration:**
```
Desktop Entry Configuration:
├── MIMEType: application/pbscore
├── Categories: Audio, Music, Education
├── FilePattern: *.pbscore
├── IconTheme: Custom application icon
└── DesktopEnvironment: Compatible with major Linux desktops
```

## 4.5 Data Migration and Backup Strategy

### 4.5.1 Storage Mode Migration (Simplified for iOS/macOS)
**Between iCloud Container and File Provider Mode:**
1. Export current documents to neutral .pbscore files location
2. Create backup in current storage system
3. Switch repository implementation to target storage mode
4. Import documents to new storage system with preserved metadata
5. Verify data integrity and collaboration features in new mode
6. Clean up temporary files (user confirmation for permanent deletion)

**Migration Scenarios:**
- **iCloud to File Provider**: For users needing specific cloud provider or enterprise requirements
- **File Provider to iCloud**: For users wanting real-time collaboration features
- **Seamless Process**: No data loss during migration, maintains document relationships

### 4.5.2 Backup and Recovery
**Automatic Backup Strategy:**
- **iCloud Container Mode**: Automatic CloudKit backup with device sync and version history
- **File Provider Mode**: Cloud provider native backup systems (OneDrive, Google Drive, etc.)
- **Other Platforms**: Platform-native backup systems (Android backup, Windows backup)
- **Manual Export**: Always available regardless of storage mode for additional safety

**Recovery Procedures:**
- **iCloud Mode**: Version recovery through CloudKit and "Revert To" functionality
- **File Provider Mode**: Version recovery through cloud provider systems
- **Corrupted Files**: Automatic detection and repair attempts for .pbscore files
- **Emergency Export**: Always available for data rescue scenarios across all modes

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

# Section 8.1 - Multi-Format Export Engine (Updated)

## 8.1.1 Supported Export Formats

### Native Format Export

**Primary Format: .pbscore**
- **Full Fidelity**: Complete preservation of all musical content and layout information
- **Cross-Platform Compatibility**: Universal format readable by all platform implementations
- **Human Readable**: JSON-based format for debugging and version control
- **Future Proof**: Standard format that will remain accessible long-term
- **Version Support**: Forward and backward compatibility through format versioning

**Export Characteristics:**
- Preserves all embellishments with complete instrument-specific execution details
- Maintains exact layout preferences and pagination settings including floating embellishments
- Includes complete metadata (title, composer, creation date, etc.)
- Retains document structure (tunes, parts, systems, measures) with note type hierarchy
- Preserves instrument-specific properties (hand assignments, stick techniques, drum positions)
- Supports all note types (PipeNote, SnareDrumNote, TenorDrumNote, BassDrumNote)
- Maintains embellishment-note relationships and hand coordination patterns

### Professional Output Formats

**PDF Export**
- **Vector-Based Output**: Scalable graphics for all resolution requirements
- **Font Embedding**: Complete SMuFL font subset embedding for universal compatibility
- **Print Optimization**: 300+ DPI resolution for professional printing requirements
- **Metadata Integration**: PDF metadata includes title, composer, copyright information
- **Color Management**: Consistent color reproduction across different printers and displays
- **Accessibility**: PDF/A compliance for long-term archival and accessibility requirements

**PNG Export**
- **High Resolution**: Configurable DPI settings (72, 150, 300, 600 DPI)
- **Transparency Support**: Optional transparent backgrounds for flexible usage
- **Color Depth**: 24-bit color with optional alpha channel
- **Page Options**: Single page or multi-page export to separate files
- **Compression**: Optimized PNG compression for file size efficiency
- **Batch Export**: Multiple scores or pages exported simultaneously

**SVG Export**
- **Scalable Vector Graphics**: Infinite scalability without quality loss
- **Web Compatibility**: Direct integration with web browsers and applications
- **Text Preservation**: Selectable text where appropriate for accessibility
- **Layer Support**: Organized SVG structure with logical grouping
- **Font Options**: Embedded fonts or system font fallbacks
- **Interactive Elements**: Optional interactive elements for web presentation

### Audio and Interchange Formats

**MIDI Export**
- **Basic Playback Support**: MIDI file generation for audio preview and practice
- **Instrument Mapping**: Appropriate MIDI instrument assignments for each pipe band instrument
- **Tempo Preservation**: Accurate tempo markings and changes with duration adjustments
- **Embellishment Interpretation**: Conversion of pipe band and drum embellishments to MIDI sequences
- **Multi-Track Support**: Separate MIDI tracks for each instrument in ensemble scores
- **Dynamics**: Basic dynamic markings converted to MIDI velocity changes
- **Hand Assignment Translation**: Drum hand assignments converted to appropriate MIDI channel/velocity patterns
- **Note Type Duration**: Accurate MIDI timing based on note types (crotchet, quaver, etc.) with synthetic adjustments

**MusicXML Export**
- **Standard Interchange**: Industry-standard format for music notation software compatibility
- **Embellishment Support**: Pipe band and drum embellishments exported with grace note representations
- **Layout Preservation**: Page layout and formatting information including floating embellishment positions
- **Metadata Transfer**: Complete metadata preservation in MusicXML format
- **Multi-Instrument**: Full ensemble score export with proper staff assignments
- **Instrument-Specific Properties**: Hand assignments, stick techniques, and drum positions where supported
- **Note Type Accuracy**: Proper note type representation (crotchet, quaver, etc.) with visual accuracy
- **Limitations**: Some pipe band specific embellishments and drum hand coordination may require simplification

## 8.1.2 Export Quality Standards

### Print Resolution Standards
**Professional Printing Requirements:**
- **Minimum Resolution**: 300 DPI for professional print quality
- **Recommended Resolution**: 600 DPI for high-end commercial printing
- **Vector Priority**: PDF and SVG preferred for professional printing
- **Color Accuracy**: CMYK color space support for commercial printing
- **Paper Size Support**: Full range of standard and custom paper sizes
- **Bleed Margins**: Configurable bleed areas for commercial printing requirements

### Font and Typography Standards
**SMuFL Font Embedding:**
- **Complete Font Subsets**: All required musical symbols embedded in exports
- **Fallback Strategies**: System font fallbacks for unsupported viewing systems
- **Unicode Compliance**: Full Unicode support for international text content
- **Font Licensing**: Respect for font licensing requirements in embedded exports
- **Cross-Platform Consistency**: Identical rendering across different operating systems
- **Accessibility**: Screen reader compatible text representation where applicable

### Color Management and Consistency
**Color Reproduction Standards:**
- **ICC Profile Support**: Embedded color profiles for accurate reproduction
- **Color Space Options**: RGB for screen display, CMYK for print production
- **Accessibility Compliance**: High contrast options and colorblind-friendly palettes
- **Monochrome Support**: Optimized black and white rendering for traditional printing
- **Custom Color Schemes**: User-defined color schemes for branding or preference
- **Print Preview**: Accurate print preview matching final output

### Metadata and Copyright Protection
**Comprehensive Metadata Inclusion:**
- **Document Properties**: Title, composer, arranger, copyright information
- **Creation Details**: Creation date, modification history, application version
- **Performance Rights**: Copyright and performance rights information
- **Contact Information**: Composer/arranger contact details where appropriate
- **Custom Properties**: User-defined metadata fields for organizational purposes
- **Watermarking**: Optional visible or invisible watermarking for copyright protection

## 8.1.3 Export Configuration and Customization

### Export Settings Management
**User Preference Profiles:**
- **Export Profiles**: Saved export configurations for different use cases
- **Template Support**: Predefined export templates for common scenarios
- **Batch Settings**: Consistent settings across multiple file exports
- **Quality Presets**: Quick selection of quality levels (draft, standard, professional)
- **Format-Specific Options**: Optimized settings for each export format
- **Default Configuration**: Intelligent defaults based on export format and intended use

### Advanced Export Options
**Layout and Formatting Control:**
- **Page Range Selection**: Export specific pages or page ranges
- **Layout Modifications**: Temporary layout adjustments for export purposes
- **Scaling Options**: Proportional scaling for different output sizes
- **Margin Adjustments**: Export-specific margin modifications
- **Header/Footer Control**: Optional headers and footers for exported documents
- **Crop Marks**: Professional printing guides and registration marks

### Batch Export Capabilities
**Multi-Document Processing:**
- **Batch File Export**: Process multiple .pbscore files simultaneously
- **Format Combinations**: Export single source to multiple formats in one operation
- **Progress Monitoring**: Real-time progress reporting for large batch operations
- **Error Handling**: Graceful error handling with detailed error reporting
- **Queue Management**: Queued processing for resource-intensive export operations
- **Background Processing**: Non-blocking exports with notification upon completion

## 8.1.4 Platform-Specific Export Integration

### iOS/macOS Export Features
**Native Integration:**
- **Share Sheet Integration**: Direct sharing through iOS/macOS share mechanisms supporting both iCloud and File Provider modes
- **Print Services**: Direct integration with AirPrint and system print services
- **Document Provider**: Save directly to user's chosen cloud storage (iCloud Container or File Provider)
- **Quick Look Integration**: Instant preview of exported files with proper embellishment rendering
- **Files App Integration**: Seamless integration with iOS Files app across storage modes
- **Shortcuts Support**: iOS Shortcuts automation for export workflows with storage mode awareness

### Android Export Features
**Android-Specific Capabilities:**
- **Share Intent**: Native Android sharing to any compatible application
- **Storage Access Framework**: Save to any configured storage provider
- **Print Framework**: Integration with Android print services
- **Intent Handling**: Support for external application integration
- **Background Export**: Android background service for large export operations
- **Notification System**: Progress and completion notifications

### Windows Export Features
**Windows Integration:**
- **File Association**: Automatic application association for exported formats
- **Print Queue Integration**: Direct integration with Windows print spooler
- **Cloud Provider APIs**: Native integration with OneDrive and other providers
- **Task Scheduler**: Scheduled export operations for automated workflows
- **PowerShell Support**: Command-line export capabilities for automation
- **Windows Store Compliance**: All export features compliant with Store requirements

### Linux Export Features
**Linux Compatibility:**
- **CUPS Integration**: Standard Linux printing system compatibility
- **Desktop Integration**: FreeDesktop.org standard compliance for file associations
- **Package Manager**: Export utilities available through standard package managers
- **Command Line Tools**: Full command-line export interface for scripting
- **Service Integration**: SystemD service integration for background operations
- **Multiple Desktop Environments**: Support for GNOME, KDE, XFCE, and others

## 8.1.5 Quality Assurance and Validation

### Export Validation Process
**Pre-Export Validation:**
- **Content Integrity**: Verification that all musical content will export correctly
- **Layout Validation**: Confirmation that layout will render properly in target format
- **Font Availability**: Verification of required fonts for target format
- **Size Limitations**: Check against format-specific size limitations
- **Feature Compatibility**: Warning for features not supported in target format
- **Performance Estimation**: Time and resource estimation for export operation

### Post-Export Verification
**Quality Assurance Checks:**
- **Visual Comparison**: Automated comparison between source and exported content
- **Metadata Verification**: Confirmation that all metadata transferred correctly
- **File Integrity**: Validation of exported file structure and content
- **Cross-Platform Testing**: Verification of exported files on different platforms
- **Performance Metrics**: Monitoring of export speed and resource usage
- **User Feedback Integration**: Collection and analysis of export quality feedback

### Error Recovery and Reporting
**Robust Error Handling:**
- **Graceful Degradation**: Partial export completion when possible
- **Detailed Error Reports**: Comprehensive error reporting with resolution suggestions
- **Automatic Retry**: Intelligent retry mechanisms for transient failures
- **Backup Creation**: Automatic backup before attempting complex exports
- **Recovery Procedures**: Step-by-step recovery instructions for failed exports
- **Support Integration**: Direct integration with technical support for complex issues


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

# Section 13.3.1 - iOS/macOS Document-Based Repository Architecture

## Project Structure and Organization

### Unified Workspace Architecture

**Workspace Component Organization:**
```
PipeBandApp.xcworkspace
├── iOS App Target (iOS 17.0+ minimum deployment)
├── macOS App Target (macOS 14.0+ minimum deployment)
├── Shared UI Components (Cross-platform SwiftUI components)
└── Swift Package Dependencies:
    ├── PBDomain (Pure domain logic with sendable entities)
    ├── PBDocument (Document-based architecture implementation)
    ├── PBCloudKit (Optional collaboration features)
    ├── PBAudio (Audio processing with actor isolation)
    └── PBLayout (Layout calculation engine)
```

**Package Dependency Hierarchy:**
```
iOS/macOS Applications
├── Depend on: PBDocument, PBCloudKit, PBAudio, PBLayout
PBDocument Package
├── Depends on: PBDomain
PBCloudKit Package
├── Depends on: PBDomain, PBDocument
PBAudio Package
├── Depends on: PBDomain
PBLayout Package
├── Depends on: PBDomain
PBDomain Package
└── Pure Swift with no external dependencies
```

## Core Package Architecture

### Domain Package (PBDomain)

**Pure Domain Logic Architecture:**

**Package Configuration Requirements:**
- Swift Tools Version: 6.0 minimum
- Platform Support: iOS 17.0+, macOS 14.0+
- Concurrency: Strict concurrency checking enabled
- Dependencies: None (pure domain logic)

**Sendable Domain Entity Requirements:**
```
MusicalDocument Entity:
├── Sendable conformance for thread safety
├── Codable conformance for persistence
├── Immutable value semantics
├── Complete data integrity validation
└── Cross-actor safe data transfer capability

Tune Entity:
├── Sendable and Codable conformance
├── Hierarchical part relationship management
├── Layout preference integration
├── Metadata preservation requirements
└── Musical content validation rules

Embellishment Entity:
├── Sendable conformance with immutable design
├── Grace note pattern specification
├── Cultural terminology preservation
├── Execution style classification
└── Layout hint integration capability
```

**Actor-Based Use Case Architecture:**
```
ScoreEditingCoordinator Actor:
├── Thread-safe document state management
├── Concurrent operation coordination
├── Musical integrity validation
├── Change tracking and history management
└── Cross-actor communication protocols
```

### Document Package (PBDocument)

**UIDocument/NSDocument Integration Architecture:**

**Document Class Requirements:**
```
PipeBandScoreDocument:
├── Platform-specific inheritance (UIDocument/NSDocument)
├── MainActor isolation for UI thread safety
├── Async document loading and saving operations
├── Thread-safe document state management
├── Actor coordination for editing operations
└── Automatic change tracking and persistence
```

**Repository Implementation Architecture:**
```
DocumentRepository Actor:
├── Actor isolation for thread-safe repository operations
├── Document cache management with weak references
├── Storage mode abstraction (local, iCloud, File Provider)
├── Async file I/O operations with error handling
├── Document lifecycle management
└── Cross-storage-mode data migration support
```

**Document Operations Interface:**
- createNewDocument: New document creation with intelligent storage mode selection (defaults to iCloud)
- openDocument: Document loading from either iCloud Container or File Provider location
- saveDocument: Persistent storage with change tracking and real-time sync (iCloud mode)
- documentCache: Weak reference management for open documents across storage modes
- storageMode: Dynamic storage mode determination and switching between iCloud/File Provider
- collaborationCapabilities: Real-time collaboration available only in iCloud Container mode

### CloudKit Package (PBCloudKit)

**Real-Time Collaboration Architecture (iCloud Container Mode Only):**

**CloudKitCollaborationManager Actor:**
```
CloudKitCollaborationManager:
├── Actor isolation for CloudKit operations
├── iCloud Container and CKDatabase management
├── CKShare creation and management for document collaboration
├── Real-time sync coordination with document changes
├── Conflict resolution for concurrent editing scenarios
├── Cross-device collaboration state synchronization
└── Integration with iCloud Container document architecture
```

**Collaboration Operations (iCloud Mode Only):**
- enableCollaboration: CKShare creation for document sharing within iCloud ecosystem
- acceptShare: Invitation acceptance and shared document integration
- syncChanges: Real-time document synchronization across Apple devices
- resolveConflicts: Automated and manual conflict resolution with musical intelligence
- managePermissions: Collaboration permission management for band members
- collaborationStatus: Real-time collaboration availability detection

### Audio Package (PBAudio)

**Actor-Isolated Audio Processing:**

**AudioProcessingEngine Actor:**
```
AudioProcessingEngine:
├── Actor isolation for thread-safe audio operations
├── AVAudioEngine integration with hardware sample rate matching
├── Concurrent audio analysis using structured concurrency
├── Real-time metronome generation and scheduling
├── Background audio processing with MainActor UI coordination
└── Memory-safe audio buffer management
```

**Audio Processing Capabilities:**
- startMetronome: Tempo-accurate metronome with time signature support
- analyzeIntonation: Parallel pitch analysis using task groups
- recordPracticeSession: High-quality audio recording with metadata
- processAudioBackground: Concurrent audio analysis operations
- updateUIWithAnalysis: MainActor-coordinated UI updates

### Layout Package (PBLayout)

**Concurrent Layout Calculation Architecture:**

**LayoutCalculationEngine Actor:**
```
LayoutCalculationEngine:
├── Actor isolation for thread-safe layout calculations
├── Parallel page layout processing using structured concurrency
├── Font management and metrics caching with thread safety
├── Embellishment layout coordination with collision resolution
├── Memory-efficient layout result caching
└── Cross-actor safe layout result communication
```

**Layout Calculation Capabilities:**
- calculateDocumentLayout: Parallel page layout processing using task groups
- calculateEmbellishmentLayout: Grace note positioning with collision resolution
- optimizeLayoutPerformance: Background layout optimization with UI updates
- cacheLayoutResults: Actor-isolated caching with automatic cleanup
- validateLayoutIntegrity: Musical and spatial layout validation

## Universal Storage Mode Support

### Simplified Storage Mode Abstraction

**Repository Factory Architecture:**
```
DocumentRepositoryFactory:
├── Storage mode enumeration (iCloudContainer, fileProvider)
├── Cloud provider classification for File Provider mode (oneDrive, googleDrive, dropbox, other)
├── Repository instance creation based on selected storage mode
├── Dynamic repository switching with data migration support
└── MainActor coordination for repository selection UI
```

**Simplified Storage Mode Interface Contract:**
```
DocumentRepositoryProtocol:
├── Document lifecycle operations (create, open, save, delete)
├── Storage capability reporting (collaboration available in iCloud mode only)
├── Cross-storage-mode migration support (iCloud ↔ File Provider)
├── Error handling with storage-specific error types
└── Performance characteristics documentation per storage mode
```

### Document Browser Integration

**Simplified Document Browser Architecture:**

**DocumentBrowser Requirements:**
```
PipeBandDocumentBrowserViewController:
├── MainActor isolation for UI operations
├── Two-tier storage mode support (iCloud Container + File Provider)
├── Document creation workflow with intelligent storage mode defaults
├── Async document opening with loading states
├── iCloud account detection and setup guidance
├── Error handling with user-friendly error presentation
└── Integration with platform sharing and export capabilities
```

**Document Browser Capabilities:**
- **Default iCloud Experience**: Automatic iCloud Container mode for users with iCloud accounts
- **Advanced File Provider Option**: Available for users needing specific cloud providers
- **Storage Mode Detection**: Automatic detection of iCloud availability
- **Seamless Setup**: Zero-configuration experience for iCloud users
- **Migration Support**: Easy switching between iCloud Container and File Provider modes
- **Prerequisites Communication**: Clear guidance for users without iCloud accounts

## Thread Safety and Performance

### Concurrency Architecture Benefits

**Complete Data Race Safety:**
- Actor isolation eliminates data races at compile time through Swift's type system
- Sendable conformance ensures safe cross-actor data transfer
- Structured concurrency prevents callback complexity and memory leaks
- MainActor coordination guarantees UI updates on the main thread
- Automatic deadlock prevention through actor system design

**Performance Optimization Strategies:**
- TaskGroup operations enable parallel processing for layout calculations and audio analysis
- Actor-isolated caches provide efficient data access with thread safety guarantees  
- Background processing coordination with MainActor UI updates maintains responsiveness
- Optimized JSON encoding and decoding with streaming for large documents
- Lazy loading patterns for memory efficiency with large score collections

**Memory Management Architecture:**
- Automatic resource cleanup through structured concurrency task lifecycle management
- Weak reference patterns for document caching prevent retain cycles
- Lazy loading implementation for large document sections reduces memory pressure
- Platform-optimized memory pressure handling with automatic cleanup triggers
- Enhanced GPU memory management for Metal-accelerated rendering operations

This architecture provides a robust, thread-safe foundation for document-based pipe band score editing while leveraging the full capabilities of modern Swift concurrency systems and maintaining compatibility with all three storage modes through clean architectural abstraction.


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
