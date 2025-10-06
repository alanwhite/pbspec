# User Experience and Interaction Design Specification

**Version:** 2.0  
**Last Updated:** October 03, 2025  
**Status:** Complete Integrated Specification (Mode-Free Design)

---

## Document Purpose and Scope

This document defines the **platform-agnostic user experience patterns** and **interaction models** for the Scottish pipe band application. It establishes consistent interaction paradigms while allowing platform-specific implementations to follow native conventions.

**Reference Documents:**
- **Domain Architecture:** `pipe_band_app_architecture.md` (business logic and entities)
- **iOS/macOS Implementation:** `ios_macos_implementation.md` (Apple-specific UX patterns)
- **Android Implementation:** `android_implementation.md` (Material Design patterns)
- **Windows Implementation:** `windows_implementation.md` (Fluent Design patterns)
- **Linux Implementation:** `linux_implementation.md` (Desktop environment patterns)

**What This Document Contains:**
- Core interaction models (continuous entry, selection-based actions)
- Input method abstractions (keyboard, mouse, touch, pencil)
- Musical editing workflows (note entry, embellishment attachment, layout editing)
- Context-aware action patterns with keyboard shortcuts
- Accessibility requirements
- Platform adaptation guidelines

**What This Document Does NOT Contain:**
- Platform-specific widget implementations
- Exact visual designs or mockups
- Platform-specific keyboard shortcuts (defined in platform specs)
- Branding or visual identity specifications

---

## 1. Core Interaction Principles

### 1.1 Interaction Philosophy

**Primary Principle: Continuous Entry with Selection-Based Actions**

The application follows a **mode-free interaction model** where all actions are immediately available based on context:

1. **No Mode Switching**: Users never enter or exit "modes" - all actions work immediately
2. **Selection Determines Actions**: What's selected determines available shortcuts and actions
3. **Selection-Is-Replaced**: Any insertion action replaces current selection
4. **Clear Feedback**: Visual indication of selection state and available actions
5. **Undo/Redo Support**: All actions are undoable with clear history

**Critical Clarification: No Mode Switching**

The application operates in a continuous, mode-free manner:
- Note entry can happen at any time
- Any selection is replaced by note entry
- Any selection is replaced by paste operations
- No "Selection Mode" vs "Note Entry Mode" distinction
- Selection type determines available context-aware shortcuts

**Rationale:**
- Eliminates cognitive overhead of mode switching
- Matches professional notation software behavior (Dorico, Sibelius, MuseScore)
- Allows rapid workflow transitions without mode changes
- More intuitive for users from other notation software
- Keyboard-centric users can work continuously without mouse
- Touch users can tap-and-type without switching contexts

### 1.2 Input Method Support

**Multi-Modal Input Strategy:**

The application supports multiple simultaneous input methods, adapting intelligently to available hardware:

```
Input Methods (Platform-Dependent):
├── Keyboard
│   ├── Note entry via letter keys (A-G)
│   ├── Duration selection via number keys
│   ├── Navigation via arrow keys
│   ├── Context-aware action shortcuts
│   └── Paste Merge
    ├── Merges copied content with existing
    ├── Used for adding instruments to measures
    └── Validates no conflicts
```

**Cross-Document Copy/Paste:**

```
Cross-Document Behavior:
├── Clipboard Format
│   ├── Internal format (full fidelity)
│   ├── MusicXML (external compatibility)
│   └── Text representation (fallback)
├── Validation
│   ├── Check instrument compatibility
│   ├── Check time signature compatibility
│   └── Prompt for adaptations if needed
└── Adaptation
    ├── Transpose if key signatures differ
    ├── Adjust durations if time signatures differ
    └── Map instruments if types differ
```

---

## 8. Context Menus

### 8.1 Context Menu Principles

**Context-Aware Menus:**
- Menu items adapt to selected element type
- Disabled items shown as dimmed (provide affordance)
- Keyboard shortcuts displayed (platform-appropriate)
- Submenus for categorical actions
- Recently used actions promoted (optional)

### 8.2 Note Context Menu

**Menu Structure:**
```
Note Context Menu:
├── Cut                                    (⌘X)
├── Copy                                   (⌘C)
├── Paste                                  (⌘V)
├── Delete                                 (Delete)
├── ─────────────
├── Add Embellishment ▶
│   ├── Pipe Band ▶
│   │   ├── Doubling                       (⌘⇧D)
│   │   ├── Throw                          (⌘⇧T)
│   │   ├── Grip                           (⌘⇧L)
│   │   └── ... (all pipe embellishments)
│   └── Drum ▶
│       ├── Flam                           (⌘⇧F)
│       ├── Drag                           (⌘⇧G)
│       ├── Rough                          (⌘⇧R)
│       └── ... (all drum embellishments)
├── Change Embellishment ▶ (if note has embellishment)
│   └── [Same submenu as Add]
├── Remove Embellishment (if note has embellishment)
├── ─────────────
├── Duration ▶
│   ├── Whole Note                         (1)
│   ├── Half Note                          (2)
│   ├── Quarter Note                       (4)
│   ├── Eighth Note                        (8)
│   └── Sixteenth Note                     (16)
├── Accidental ▶
│   ├── Sharp                              (#)
│   ├── Flat                               (b)
│   └── Natural                            (n)
├── Articulation ▶
│   ├── Staccato
│   ├── Accent
│   ├── Tenuto
│   └── Fermata
├── ─────────────
├── Transpose Up                           (↑)
├── Transpose Down                         (↓)
├── ─────────────
└── Note Properties...                     (⌘I)
```

### 8.3 Measure Context Menu

**Menu Structure:**
```
Measure Context Menu:
├── Insert Measure Before
├── Insert Measure After
├── Delete Measure                         (Delete)
├── ─────────────
├── Time Signature...                      (⌘T)
├── Opening Barline ▶
│   ├── None
│   ├── Single
│   ├── Double
│   ├── Repeat Start
│   └── ... (all barline types)
├── Closing Barline ▶
│   └── [Same as Opening]
├── ─────────────
├── Add Ending ▶
│   ├── 1st Ending                         (⌘⇧1)
│   ├── 2nd Ending                         (⌘⇧2)
│   ├── 3rd Ending                         (⌘⇧3)
│   └── Custom Volta...
├── ─────────────
├── System Break After                     (⌘Return)
├── ─────────────
├── Select All Notes in Measure            (⌘A)
├── ─────────────
└── Measure Properties...                  (⌘I)
```

### 8.4 Platform Context Menu Adaptations

**iOS/macOS:**
- Long-press triggers context menu (iOS)
- Control+Click or Right-click triggers context menu (macOS)
- Context menu appears as popover with rounded corners
- Submenus appear inline or as new popovers

**Android:**
- Long-press triggers bottom sheet or popup menu
- Material Design styling
- Submenus expand inline or open new sheets

**Windows:**
- Right-click triggers Fluent Design context menu
- Submenus appear as flyouts
- Keyboard navigation with arrow keys

**Linux:**
- Right-click triggers desktop environment style menu
- Follows GTK/Qt theming
- Keyboard navigation standard

---

## 9. Inspector Panel Design

### 9.1 Inspector Purpose

**Properties Panel for Selected Elements:**

The inspector panel shows properties of currently selected element(s) and allows modification:

```
Inspector Panel Contents (Context-Aware):
├── When Note Selected:
│   ├── Pitch Picker
│   ├── Duration Picker
│   ├── Embellishment Picker              (⌘⇧Letter shortcuts shown)
│   ├── Accidental Toggle
│   ├── Articulation Checkboxes
│   └── Note Groups (Ties, Slurs, Tuplets)
├── When Measure Selected:
│   ├── Time Signature                    (⌘T to edit)
│   ├── Opening Barline
│   ├── Closing Barline
│   ├── Rehearsal Mark
│   ├── Volta/Ending                      (⌘⇧1, ⌘⇧2 shortcuts shown)
│   └── Measure Width (layout hint)
├── When System Selected:
│   ├── Staff Spacing
│   ├── System Break Toggle               (⌘Return shortcut shown)
│   └── Instrument Visibility Checkboxes
├── When Part Selected:
│   ├── Part Name
│   ├── Play Order
│   ├── Repeat Count
│   └── Part Letter
└── When Multiple Selected:
    └── Common Properties Only (batch editing)
```

### 9.2 Inspector Placement

**Platform-Specific Placement:**

- **iOS**: Right sidebar (iPad landscape), bottom sheet (iPad portrait, iPhone)
- **macOS**: Right sidebar (collapsible), floating panel (optional)
- **Android**: Right drawer (tablets), bottom sheet (phones)
- **Windows**: Right task pane (collapsible)
- **Linux**: Right sidebar or floating palette (user configurable)

### 9.3 Inspector Interactions

**Property Editing:**
- Click picker → Opens dropdown or modal picker
- Checkboxes for boolean properties
- Sliders for numeric ranges
- Text fields for text properties
- Immediate application (no "Apply" button needed)
- Undo point created on change
- Keyboard shortcuts shown next to properties

---

## 10. Accessibility Requirements

### 10.1 Keyboard Navigation

**Full Keyboard Accessibility:**

```
Keyboard Navigation Requirements:
├── Tab Order
│   ├── Logical focus order through all interactive elements
│   ├── Toolbar → Score content → Inspector
│   └── Skip navigation link to content
├── Score Navigation
│   ├── Arrow keys move between notes/measures
│   ├── Tab moves between staves/instruments
│   ├── Home/End jump to start/end
│   └── Page Up/Down move between systems
├── Selection via Keyboard
│   ├── Space to select focused element
│   ├── Shift+Arrow for range selection
│   ├── Modifier+Space for multi-select
│   └── Escape to deselect all
└── Action Activation
    ├── Enter/Return to activate default action
    ├── Context menu key (or Shift+F10) for context menu
    └── All actions have keyboard shortcuts
```

### 10.2 Screen Reader Support

**Screen Reader Announcements:**

```
Announcement Requirements:
├── Current Context
│   └── "Note selected" / "Measure selected" / "System selected"
├── Selection Changes
│   └── "D quarter note selected" / "Measure 3 selected"
├── Action Feedback
│   └── "Doubling embellishment added to D quarter note"
│       "1st ending added to measure 4"
├── Navigation Updates
│   └── "Measure 5, Part A, Bagpipes"
├── Validation Errors
│   └── "Cannot add High G doubling: prior note must be lower than High G"
└── Context Hints
    └── "Note selected. Press ⌘⇧D for Doubling, ⌘⇧L for Grip"
```

**Screen Reader Friendly Labels:**
- All interactive elements have accessible labels
- Musical symbols have text equivalents
- Complex graphics have detailed descriptions
- Progress indicators announce percentage
- Keyboard shortcuts announced with actions

### 10.3 Visual Accessibility

**High Contrast and Color Blindness:**

```
Visual Accessibility Requirements:
├── High Contrast Mode Support
│   ├── Selection uses high contrast colors
│   ├── All UI elements visible in high contrast
│   └── No reliance on color alone for information
├── Color Blindness Considerations
│   ├── Use patterns/shapes in addition to color
│   ├── Sufficient contrast ratios (WCAG 2.1 AA)
│   └── Colorblind-safe palette options
├── Dynamic Type Support (iOS/macOS)
│   ├── UI scales with system text size
│   ├── Music notation scales proportionally
│   └── Maintains readability at all sizes
└── Zoom Support
    ├── Pinch-to-zoom on touch platforms
    ├── Keyboard zoom shortcuts
    └── Maintains layout at all zoom levels
```

### 10.4 Motor Accessibility

**Reduced Motion and Alternative Inputs:**

```
Motor Accessibility Requirements:
├── Reduced Motion Mode
│   ├── Disable animation for mode changes
│   ├── Instant page transitions
│   └── No parallax scrolling effects
├── Large Touch Targets
│   ├── Minimum 44x44 points (iOS)
│   ├── Minimum 48x48 dp (Android)
│   └── Adequate spacing between targets
├── Voice Control Support
│   ├── All actions have voice commands
│   ├── Element labels support voice selection
│   └── "Show numbers" for disambiguation
└── Switch Control Support (iOS)
    ├── All interactive elements reachable
    ├── Logical switch navigation order
    └── Switch actions for all primary functions
```

---

## 11. Search and Navigation

### 11.1 Search Functionality

**Search Scope:**

```
Searchable Elements:
├── Text Content
│   ├── Score titles
│   ├── Composer names
│   ├── Part names
│   ├── Rehearsal marks
│   └── Text annotations
├── Musical Content
│   ├── Pitch sequences (melodic search)
│   ├── Rhythm patterns
│   ├── Embellishment types
│   ├── Time signatures
│   └── Key signatures
└── Metadata
    ├── Creation date
    ├── Last modified
    ├── File size
    └── Custom tags
```

### 11.2 Search UI

**Search Interface:**

```
Search UI Components:
├── Search Field
│   ├── Prominent placement (toolbar or header)
│   ├── Search-as-you-type (live results)
│   ├── Recent searches dropdown
│   └── Search scope selector
├── Results Display
│   ├── List of matches with context
│   ├── Jump-to-result navigation
│   ├── Highlight matches in score
│   └── Result count indicator
└── Advanced Search (Optional)
    ├── Filter by element type
    ├── Filter by date range
    ├── Combine multiple criteria
    └── Save search as smart collection
```

### 11.3 Navigation Shortcuts

**Quick Navigation:**

```
Navigation Patterns:
├── Go To Actions
│   ├── Go to Page (by number)
│   ├── Go to Measure (by number)
│   ├── Go to Part (by letter)
│   └── Go to Rehearsal Mark
├── Sequential Navigation
│   ├── Next/Previous measure (Arrow keys)
│   ├── Next/Previous system (Page Up/Down)
│   ├── Next/Previous part
│   └── Next/Previous tune
├── Structural Navigation
│   ├── Jump to tune start (Home)
│   ├── Jump to tune end (End)
│   ├── Jump to part start
│   └── Jump to part end
└── Visual Navigation
    ├── Minimap overview (optional)
    ├── Page thumbnails
    ├── Outline view (tune structure)
    └── Timeline view
```

---

## 12. Responsive Design Patterns

### 12.1 Screen Size Adaptation

**Layout Adaptation Strategy:**

```
Screen Size Categories:
├── Small Phone (< 5.5")
│   ├── Single column layout
│   ├── Collapsible toolbars
│   ├── Bottom sheet inspector
│   ├── Fullscreen score view
│   └── Simplified navigation
├── Large Phone / Small Tablet (5.5" - 9")
│   ├── Two-pane layout (portrait)
│   ├── Split layout (landscape)
│   ├── Persistent toolbar (landscape)
│   ├── Side inspector (landscape)
│   └── Drawer navigation
├── Tablet (9" - 13")
│   ├── Three-pane layout
│   ├── Persistent inspector sidebar
│   ├── Full toolbar
│   ├── Document browser sidebar
│   └── Drag-and-drop support
└── Desktop (> 13")
    ├── Full IDE-style layout
    ├── Multiple inspectors/panels
    ├── Floating palettes
    ├── Multi-window support
    └── Professional tool density
```

### 12.2 Orientation Changes

**Portrait vs. Landscape:**

- **Portrait**:
  - Vertical score scrolling preferred
  - Bottom sheet inspector
  - Collapsible toolbars to maximize score space
  - Floating action buttons for primary actions
  - Full-width score view

- **Landscape**:
  - Horizontal score scrolling option (like a real score)
  - Sidebar inspector persistent
  - Full toolbar visible
  - Split view for navigation + score
  - Multi-column layout on tablets/desktop

**Orientation Transition:**
- Preserve scroll position and selection
- Animate layout changes smoothly
- Adapt pagination if needed (user preference)
- Maintain mode state through transition

### 12.3 Input Method Adaptation

**Touch vs. Mouse/Trackpad:**

```
Input Adaptation:
├── Touch Platforms
│   ├── Larger touch targets (44pt minimum)
│   ├── Context menus via long-press
│   ├── Gesture-based navigation
│   ├── No hover states
│   └── Touch-optimized tool palettes
├── Mouse/Trackpad Platforms
│   ├── Smaller, denser UI elements
│   ├── Hover states for tooltips
│   ├── Right-click context menus
│   ├── Precise click targeting
│   └── Drag-and-drop operations
└── Hybrid (Touch + Keyboard/Mouse)
    ├── Detect primary input method
    ├── Adapt UI density dynamically
    ├── Support both interaction models
    └── User preference override
```

---

## 13. Error Handling and Validation UX

### 13.1 Validation Feedback

**Real-Time Validation:**

```
Validation Timing:
├── During Entry (Proactive)
│   ├── Show valid drop targets
│   ├── Prevent invalid actions
│   ├── Highlight compatible elements
│   └── Disable incompatible tools
├── On Action (Immediate)
│   ├── Validate before applying
│   ├── Show error inline
│   ├── Offer correction suggestions
│   └── Allow force override (with warning)
└── On Save (Final Check)
    ├── Comprehensive validation
    ├── Error list with navigation
    ├── Fix-it actions where possible
    └── Allow save with warnings
```

### 13.2 Error Message Patterns

**User-Friendly Error Messages:**

```
Error Message Structure:
├── What Went Wrong (Clear statement)
│   └── "Cannot add High G doubling to this note"
├── Why It Failed (Musical reason)
│   └── "Prior note must be lower than High G"
├── How to Fix (Actionable guidance)
│   └── "Select a different embellishment or change the prior note"
└── Visual Context (Highlight problem area)
    └── Red outline on offending note
```

**Error Severity Levels:**

```
Severity Levels:
├── Error (Prevents action)
│   ├── Red color
│   ├── Error icon
│   ├── Blocks save/export
│   └── Must be fixed
├── Warning (Suspicious but allowed)
│   ├── Yellow/orange color
│   ├── Warning icon
│   ├── Allows save with confirmation
│   └── Should be reviewed
└── Info (Helpful suggestion)
    ├── Blue color
    ├── Info icon
    ├── Non-blocking
    └── Can be dismissed
```

### 13.3 Validation Examples

**Prior Note Constraint Validation:**

```
Scenario: User tries to add High G Doubling

If prior note is High G or higher:
├── Error appears immediately when selecting embellishment
├── Message: "Cannot add High G Doubling"
├── Reason: "Prior note (High G) must be lower than High G"
├── Suggestion: "Try Low G Doubling or change prior note to F or lower"
├── Visual: Red outline around prior note
└── Action buttons: [Change Prior Note] [Select Different Embellishment] [Cancel]
```

**Duration Overflow Validation:**

```
Scenario: User adds note that exceeds measure duration

When duration exceeds time signature:
├── Error appears during or after note entry
├── Message: "Measure is over capacity"
├── Reason: "Time signature is 4/4 (4 beats) but notes total 5 beats"
├── Suggestion: "Remove a note, change durations, or add a measure"
├── Visual: Red outline around measure, excess notes highlighted
└── Action buttons: [Add Measure] [Adjust Durations] [Remove Last Note] [Cancel]
```

---

## 14. Undo/Redo System

### 14.1 Undo Scope

**Undoable Actions:**

```
Undo System Coverage:
├── Musical Content Changes
│   ├── Add/delete/modify notes
│   ├── Add/change/remove embellishments
│   ├── Add/delete measures
│   ├── Change time signatures
│   └── Change barlines
├── Layout Changes
│   ├── Page breaks
│   ├── System breaks
│   ├── Staff spacing
│   └── Measure widths
├── Document Structure
│   ├── Add/delete parts
│   ├── Reorder parts
│   ├── Add/delete instruments
│   └── Tune metadata changes
└── Not Undoable (Explicit Save Points)
    ├── File operations (save, open)
    ├── Export operations
    ├── Cloud sync operations
    └── Preference changes
```

### 14.2 Undo UI Patterns

**Undo/Redo Access:**

```
Undo Access Methods:
├── Keyboard Shortcuts
│   ├── Undo: Cmd/Ctrl+Z
│   └── Redo: Cmd/Ctrl+Shift+Z or Cmd/Ctrl+Y
├── Menu Items
│   ├── Edit → Undo [Action Name]
│   └── Edit → Redo [Action Name]
├── Toolbar Buttons
│   ├── Undo button (back arrow)
│   └── Redo button (forward arrow)
└── Gesture (Touch Platforms)
    ├── Three-finger swipe left (undo)
    └── Three-finger swipe right (redo)
```

**Undo History UI:**

```
Undo History Features:
├── Action Descriptions
│   └── "Undo Add D Quarter Note"
│   └── "Redo Change Embellishment to Doubling"
├── Branching History (Optional)
│   ├── Linear undo by default
│   ├── Branch point markers
│   └── Navigate to alternate branches
└── History Panel (Optional)
    ├── List of all undo points
    ├── Jump to any point
    ├── Selective undo (advanced)
    └── History search
```

---

## 15. Platform-Specific UX Adaptations

### 15.1 iOS/macOS UX Patterns

**Apple Human Interface Guidelines Compliance:**

```
iOS-Specific Patterns:
├── Navigation
│   ├── Tab bar for main sections (iPhone)
│   ├── Navigation split view (iPad, Mac)
│   ├── Back button in navigation bar
│   └── Modal presentation for task flows
├── Actions
│   ├── Swipe actions on lists
│   ├── Share sheet integration
│   ├── Drag and drop between views
│   └── Context menu preview (Peek & Pop successor)
├── Input
│   ├── Pull to refresh
│   ├── Pinch to zoom
│   ├── Long-press for context menu
│   └── Apple Pencil integration (iPad)
└── Feedback
    ├── Haptic feedback on actions
    ├── System sounds for errors/success
    └── Progress indicators (activity, progress)

macOS-Specific Patterns:
├── Navigation
│   ├── Toolbar with icon buttons
│   ├── Sidebar with navigation tree
│   ├── Document tabs (if multi-document)
│   └── Inspector panel (right sidebar)
├── Actions
│   ├── Menu bar with full command set
│   ├── Right-click context menus
│   ├── Keyboard shortcuts for all actions
│   └── Touch Bar support (if applicable)
├── Windows
│   ├── Single-window document editing
│   ├── Auxiliary windows for palettes
│   ├── Full screen mode support
│   └── Split view support
└── Input
    ├── Trackpad gestures
    ├── Mouse scroll for navigation
    ├── Keyboard-centric workflows
    └── Dictation integration
```

**See `ios_macos_implementation.md` for detailed specifications**

### 15.2 Android UX Patterns

**Material Design Compliance:**

```
Android-Specific Patterns:
├── Navigation
│   ├── Bottom navigation bar (primary sections)
│   ├── Navigation drawer (secondary sections)
│   ├── Up button in app bar
│   └── Back button (system navigation)
├── Actions
│   ├── Floating Action Button (FAB) for primary action
│   ├── Extended FAB with menu
│   ├── Bottom app bar (optional)
│   └── Swipe actions on cards/lists
├── Input
│   ├── Pull to refresh
│   ├── Pinch to zoom
│   ├── Long-press for context menu
│   └── Stylus support (if available)
└── Feedback
    ├── Snackbar for notifications
    ├── Material ripple effects
    ├── Progress indicators (circular, linear)
    └── System haptics
```

**See `android_implementation.md` for detailed specifications**

### 15.3 Windows UX Patterns

**Fluent Design System Compliance:**

```
Windows-Specific Patterns:
├── Navigation
│   ├── NavigationView with pane
│   ├── Breadcrumb navigation
│   ├── Tabs for document switching
│   └── Command bar for actions
├── Actions
│   ├── Ribbon (Office-style, optional)
│   ├── Command bar (compact actions)
│   ├── Context menus (right-click)
│   └── Keyboard shortcuts
├── Input
│   ├── Mouse and keyboard primary
│   ├── Touch support (if available)
│   ├── Pen support (Surface Pen)
│   └── Dial support (Surface Dial, optional)
└── Feedback
    ├── Info bar for messages
    ├── Flyouts for contextual info
    ├── Progress ring/bar
    └── Acrylic material effects
```

**See `windows_implementation.md` for detailed specifications**

### 15.4 Linux UX Patterns

**Desktop Environment Adaptation:**

```
Linux-Specific Patterns:
├── GNOME/GTK
│   ├── Header bar with integrated menu
│   ├── Sidebar navigation
│   ├── Popover menus
│   └── Keyboard-centric workflows
├── KDE/Qt
│   ├── Traditional menu bar
│   ├── Toolbar with icons
│   ├── Dockable panels
│   └── Extensive keyboard shortcuts
├── Common Patterns
│   ├── Right-click context menus
│   ├── Middle-click actions
│   ├── Keyboard shortcuts
│   └── Desktop integration (file manager, etc.)
└── Input
    ├── Mouse and keyboard primary
    ├── Touchpad gestures (if supported)
    ├── Tablet support (if available)
    └── Wacom tablet integration
```

**See `linux_implementation.md` for detailed specifications**

---

## 16. Onboarding and Help

### 16.1 First-Time User Experience

**Onboarding Flow:**

```
Onboarding Steps:
├── Welcome Screen
│   ├── App overview
│   ├── Key features highlight
│   ├── Quick tour option
│   └── Skip to app option
├── Storage Setup (Platform-Dependent)
│   ├── Cloud storage benefits explanation
│   ├── Enable cloud prompt (iCloud/OneDrive/Google Drive)
│   ├── Alternative File Provider option
│   └── Local-only fallback
├── Sample Score
│   ├── Pre-loaded example score
│   ├── Interactive tooltips
│   ├── Guided tasks ("Try adding a note")
│   └── Completion celebration
└── Tutorial Series (Optional)
    ├── Basic note entry (continuous entry model)
    ├── Adding embellishments (context-aware shortcuts)
    ├── Creating parts
    └── Exporting scores
```

### 16.2 In-App Help

**Help Access Methods:**

```
Help System:
├── Contextual Help
│   ├── Tooltips on hover (desktop)
│   ├── Long-press hints (touch)
│   ├── Help button in dialogs
│   └── "What's This?" mode
├── Help Menu/Section
│   ├── Getting Started Guide
│   ├── Video Tutorials
│   ├── Keyboard Shortcut Reference (context-aware)
│   ├── Troubleshooting
│   └── Contact Support
├── Search Help
│   ├── Search across all help content
│   ├── AI-powered suggestions
│   └── Community Q&A (optional)
└── Interactive Tutorials
    ├── Step-by-step walkthroughs
    ├── Practice exercises
    ├── Skill assessments
    └── Progress tracking
```

### 16.3 Feature Discovery

**Progressive Disclosure:**

```
Feature Discovery:
├── Coach Marks
│   ├── Highlight new features on update
│   ├── One-time overlay with arrows
│   ├── Dismissible permanently
│   └── "Show me again" option
├── Empty States
│   ├── Helpful text when no content
│   ├── Action buttons to get started
│   ├── Illustration showing what goes here
│   └── Link to help documentation
├── Tip of the Day (Optional)
│   ├── Rotating helpful tips
│   ├── Appears on app launch
│   ├── Can be disabled in settings
│   └── Share tip option
└── What's New
    ├── Release notes on update
    ├── Highlight major new features
    ├── Link to detailed documentation
    └── Video demonstrations
```

---

## 17. Settings and Preferences

### 17.1 User Preferences

**Preference Categories:**

```
Settings Organization:
├── General
│   ├── Default paper size
│   ├── Default orientation
│   ├── Auto-save interval
│   ├── Recent files count
│   └── Language selection
├── Editing
│   ├── Default note duration
│   ├── Auto-advance after note entry
│   ├── Snap to grid
│   ├── Show measure numbers
│   └── Show page boundaries
├── Display
│   ├── Staff size (zoom level)
│   ├── Color scheme (light/dark/auto)
│   ├── Show embellishment names
│   ├── Highlight current measure
│   └── Page turn animation
├── Audio
│   ├── Metronome sound selection
│   ├── Metronome volume
│   ├── Audio input device
│   ├── Audio output device
│   └── Recording quality
├── Cloud & Sync
│   ├── Storage mode selection
│   ├── Auto-sync toggle
│   ├── Sync conflict resolution
│   └── Offline mode
└── Advanced
    ├── Performance mode (low latency)
    ├── Cache size limit
    ├── Debug mode
    └── Reset to defaults
```

### 17.2 Workspace Customization

**Customizable UI Elements:**

```
Customization Options:
├── Toolbar
│   ├── Add/remove buttons
│   ├── Reorder buttons
│   ├── Show/hide button labels
│   └── Icon size selection
├── Panels
│   ├── Show/hide panels
│   ├── Reposition panels (drag)
│   ├── Resize panels
│   └── Save workspace layout
├── Keyboard Shortcuts
│   ├── Customize shortcuts
│   ├── Import/export shortcut sets
│   ├── Conflict detection
│   └── Reset to defaults
└── Themes (Optional)
    ├── Light theme
    ├── Dark theme
    ├── High contrast theme
    └── Custom theme creator
```

---

## 18. Performance and Feedback

### 18.1 Loading States

**Progress Indication:**

```
Loading State Patterns:
├── Document Loading
│   ├── Progress bar (with percentage)
│   ├── Status message ("Loading measures...")
│   ├── Cancel button (if possible)
│   └── Estimated time remaining
├── Saving
│   ├── Saving indicator (subtle)
│   ├── "Saved" confirmation
│   ├── Error state with retry
│   └── Background saving (non-blocking)
├── Exporting
│   ├── Modal progress dialog
│   ├── Progress bar with stages
│   ├── Cancel option
│   └── Success notification with share options
└── Syncing
    ├── Sync icon (animated during sync)
    ├── Last sync timestamp
    ├── Sync error indicator
    └── Manual sync trigger
```

### 18.2 Action Feedback

**Confirmation and Feedback:**

```
Feedback Mechanisms:
├── Immediate Visual Feedback
│   ├── Button press states
│   ├── Selection highlights
│   ├── Hover states (non-touch)
│   └── Drag preview
├── Haptic Feedback (Touch Platforms)
│   ├── Light tap on selection
│   ├── Medium tap on action complete
│   ├── Heavy tap on error
│   └── Custom patterns for special actions
├── Audio Feedback (Optional)
│   ├── System sounds for actions
│   ├── Error sound
│   ├── Success sound
│   └── Mute option in settings
└── Status Messages
    ├── Toast/Snackbar for non-critical
    ├── Banner for important info
    ├── Modal dialog for critical
    └── Persistent status bar for ongoing
```

---

## 19. UX Testing and Validation

### 19.1 Usability Testing

**Testing Scenarios:**

```
Test Scenarios:
├── First-Time User Tasks
│   ├── Create a new score
│   ├── Add notes to first measure
│   ├── Add an embellishment (using context-aware shortcut)
│   ├── Change time signature
│   └── Export to PDF
├── Expert User Tasks
│   ├── Rapid keyboard note entry (continuous model)
│   ├── Batch embellishment application
│   ├── Complex layout adjustments
│   ├── Keyboard-only workflow
│   └── Multi-document management
├── Accessibility Tasks
│   ├── Navigate with keyboard only
│   ├── Use with screen reader
│   ├── Use with voice control
│   └── Use in high contrast mode
└── Cross-Platform Tasks
    ├── Create score on iPhone, edit on iPad
    ├── Collaborate with Mac user from Android
    ├── Export from Windows, import to Linux
    └── File compatibility verification
```

### 19.2 UX Metrics

**Success Metrics:**

```
Measurable UX Goals:
├── Efficiency
│   ├── Time to create first score: < 2 minutes
│   ├── Time to add 10 notes: < 30 seconds (keyboard)
│   ├── Time to add embellishment: < 5 seconds (shortcut)
│   └── Time to find feature: < 10 seconds
├── Learnability
│   ├── First note entered: < 1 minute from launch
│   ├── First embellishment added: < 5 minutes from launch
│   ├── First context-aware shortcut used: < 10 minutes
│   ├── Part created: < 10 minutes from launch
│   └── Score exported: < 15 minutes from launch
├── Error Rate
│   ├── Invalid embellishment attempts: < 5%
│   ├── Measure overflow errors: < 10%
│   ├── Lost work incidents: 0%
│   └── Crash rate: < 0.1%
└── Satisfaction
    ├── User rating: > 4.5/5.0
    ├── Task completion rate: > 90%
    ├── Feature discoverability: > 80%
    └── Would recommend: > 85%
```

---

## 20. Summary and Best Practices

### 20.1 Core UX Principles Recap

**Essential Interaction Patterns:**

1. **Continuous Entry Model**: No mode switching - all actions work immediately
2. **Selection-Is-Replaced**: Any insertion action replaces current selection
3. **Context-Aware Shortcuts**: Shortcuts adapt to what's selected
4. **Multi-Modal Input**: Support keyboard, mouse/trackpad, touch, and stylus
5. **Platform Native**: Follow platform-specific UX conventions
6. **Progressive Disclosure**: Show advanced features when needed
7. **Immediate Feedback**: Visual/haptic/audio confirmation of actions
8. **Graceful Errors**: Clear, actionable error messages
9. **Undo Everything**: Comprehensive undo/redo support
10. **Responsive Design**: Adapt to screen sizes and input methods

### 20.2 Key UX Innovations

**Unique Aspects of This Design:**

```
Innovative UX Features:
├── Mode-Free Operation
│   └── Eliminates cognitive overhead of mode switching
│       Matches professional notation software behavior
│
├── Context-Aware Shortcuts
│   └── Shortcuts become available based on selection
│       Reduces shortcut memorization burden
│       Shown in tooltips and context menus for discoverability
│
├── Selection-Replacement Model
│   └── Natural workflow for rapid editing
│       Paste always replaces selection
│       Note entry always replaces selection
│
├── Cursor Management
│   └── Cursor advances after insertion
│       Cursor moves to deletion position
│       Enables rapid sequential entry
│
└── Embellishment Shortcuts
    └── Modifier+Shift+Letter (when note selected)
        Immediate application without mode change
        Visual feedback with grace notes
```

### 20.3 Implementation Priority

**Phase 1 (MVP UX):**
- Continuous entry model (no modes)
- Selection and basic navigation
- Note entry via keyboard and click/tap
- Context menus for embellishments
- Basic toolbar with essential tools
- Context-aware keyboard shortcuts for common actions
- Standard undo/redo

**Phase 2 (Enhanced UX):**
- Inspector panel for property editing
- Advanced selection (drag, shift-click)
- Keyboard shortcut customization
- Touch gestures (pinch, swipe)
- Stylus/pencil support
- Search functionality
- Complete context-aware shortcut system

**Phase 3 (Professional UX):**
- Collaboration features (presence, comments)
- Workspace customization
- Advanced keyboard workflows
- Command palette
- Macro recording (optional)
- AI-assisted features (optional)

### 20.4 Platform Implementation References

**Cross-Reference to Platform Specs:**

- **iOS/macOS**: See `ios_macos_implementation.md` Section 4 (UI Implementation)
  - Specific keyboard shortcuts (⌘ modifiers)
  - SwiftUI component usage
  - Apple HIG patterns
  
- **Android**: See `android_implementation.md` Section 4 (UI Implementation)
  - Specific keyboard shortcuts (Ctrl modifiers)
  - Material Design components
  - Android design patterns
  
- **Windows**: See `windows_implementation.md` Section 4 (UI Implementation)
  - Specific keyboard shortcuts (Ctrl modifiers)
  - Fluent Design components
  - Windows design patterns
  
- **Linux**: See `linux_implementation.md` Section 4 (UI Implementation)
  - Specific keyboard shortcuts (Ctrl/Super modifiers)
  - GTK/Qt component usage
  - Desktop environment patterns

Each platform spec will detail:
- Specific keyboard shortcuts (with modifier keys)
- Native UI component usage
- Platform gesture support
- Accessibility API integration
- Performance optimization techniques

---

## 21. Glossary

**UX Terms:**
- **Continuous Entry**: Mode-free interaction where note entry works at any time
- **Selection-Is-Replaced**: Pattern where insertion actions replace current selection
- **Context-Aware Shortcuts**: Keyboard shortcuts that activate based on selection type
- **Cursor**: Visual indicator showing next insertion point
- **Inspector Panel**: Context-sensitive property editor
- **Context Menu**: Right-click/long-press menu showing available actions

**Musical Terms:**
- **Embellishment**: Decorative musical element attached to a note (grace notes, ornaments)
- **Grace Note**: Small ornamental note played quickly before a principal note
- **Principal Note**: Main note to which an embellishment is attached
- **Measure/Bar**: Rhythmic unit defined by time signature
- **System**: Group of staves for multiple instruments, played simultaneously
- **Part**: Named section of a tune (A, B, C, Intro, Outro, etc.)
- **Volta**: Ending bracket indicating 1st, 2nd, etc. endings

**Platform Terms:**
- **Modifier Key**: Platform-specific key (⌘ on Mac, Ctrl on Windows/Linux/Android)
- **Long-Press**: Touch gesture equivalent to right-click
- **Haptic Feedback**: Vibration feedback on touch devices
- **Toast/Snackbar**: Temporary notification message

---

## 22. Document Maintenance

**Version History:**
- Version 2.0 (2025-10-03): Complete rewrite with mode-free design, context-aware shortcuts
- Version 1.0 (2025-09-30): Initial UX specification (mode-based, superseded)

**Change Process:**
- UX changes require review and approval
- Breaking UX changes require major version increment
- Platform teams notified of all UX changes
- User testing required for major interaction model changes

**Review Schedule:**
- Quarterly review of UX patterns and effectiveness
- Annual comprehensive specification review
- Ad-hoc reviews for significant feature additions
- Post-release retrospectives for lessons learned

---

**End of User Experience and Interaction Design Specification v2.0**

This specification provides a complete, mode-free interaction model for the Scottish pipe band application. Platform-specific implementations must refer to their respective implementation documents for detailed UI component usage and platform-specific patterns while maintaining the core UX principles defined in this document.

**Key Changes from v1.0:**
- Removed mode-based editing (Selection Mode vs. Note Entry Mode)
- Introduced continuous entry model (selection-is-replaced)
- Added context-aware keyboard shortcuts
- Enhanced embellishment workflow with shortcuts
- Improved cursor management and insertion point logic
- Added comprehensive shortcut discoverability features
- Refined error handling and validation patterns Modifier keys for variants
├── Pointing Device (Mouse, Trackpad, Touch)
│   ├── Click/tap to select
│   ├── Right-click/long-press for context menu
│   ├── Drag for multiple selection
│   ├── Double-click/tap for editing
│   └── Hover for tooltips (non-touch platforms)
├── Touch Gestures (iOS, Android)
│   ├── Tap to select
│   ├── Long-press for context menu
│   ├── Pinch to zoom
│   ├── Two-finger pan to scroll
│   └── Swipe for navigation
├── Stylus/Pencil (iPad, Surface, Linux tablets)
│   ├── Direct note placement
│   ├── Gesture recognition for ornaments
│   ├── Pressure sensitivity (future enhancement)
│   └── Palm rejection
└── Voice Input (Accessibility)
    ├── Note dictation
    └── Command execution
```

### 1.3 Context-Aware Actions

**Actions adapt to:**
- Currently selected element type
- Available platform capabilities
- User's skill level (beginner vs. expert modes)
- Platform input methods (keyboard vs. touch)

---

## 2. Note Entry Workflow

### 2.1 Continuous Entry Pattern (No Modes)

**Core Principle: Selection-Is-Replaced-On-Entry**

The application operates with a single unified interaction model where note entry can occur at any time, regardless of selection state:

```
Continuous Entry Model:
├── Default State
│   └── No selection → Cursor position determines insertion point
├── Selected Note(s)
│   └── Entry action → Replaces selection with new note
├── Selected Measure(s)
│   └── Entry action → Replaces first note position with new note
├── Selected System(s)
│   └── Entry action → Replaces first measure first position with new note
└── Any Entry Method Works Immediately
    ├── Keyboard shortcuts (A-G for pitches, 1-8 for durations)
    ├── Click/tap on staff
    ├── MIDI input (if available)
    └── Voice input (accessibility)
```

### 2.2 Note Entry Actions

**Action Trigger Methods:**

All note entry actions work identically whether or not elements are selected.

#### 2.2.1 Insert Note Action

**Trigger Options (Platform Implements Native Method):**
- **Keyboard**: Press note letter key (A-G)
- **Click/Tap**: Click on staff line/space
- **Context Menu**: Right-click > Insert Note > [pitch]
- **Button**: Toolbar button > click pitch on staff
- **Voice**: "Insert D quarter note" (accessibility)

**Required Parameters:**
- Pitch (determined by staff position or key press)
- Duration (from current tool setting or modifier key)
- Instrument (from current staff context)

**Behavior:**
```
If no selection:
└── Insert note at cursor position

If note(s) selected:
└── Replace selection with new note

If measure(s) selected:
└── Replace first note position in first measure

If system(s) selected:
└── Replace first note position in first measure of first staff
```

**Platform Implementations:**
- **iOS/macOS**: Keyboard shortcut, toolbar buttons, tap on staff, Apple Pencil stroke
- **Android**: Floating action button, tap on staff, hardware keyboard (if present)
- **Windows**: Keyboard shortcut, ribbon button, click on staff, Surface Pen
- **Linux**: Keyboard shortcut, toolbar button, click on staff, stylus (if present)

#### 2.2.2 Set Note Duration Action

**Conceptual Triggers:**
- **Before Entry**: Select duration tool, then enter notes
- **After Entry**: Select note(s), then change duration
- **During Entry**: Keyboard numbers (1=whole, 2=half, 4=quarter, 8=eighth)

**Platform Implementations:**
- **iOS/macOS**: Toolbar duration picker, keyboard numbers (1-8), duration palette
- **Android**: Bottom sheet duration selector, keyboard numbers, FAB menu
- **Windows**: Ribbon duration group, keyboard numbers, context menu
- **Linux**: Toolbar duration buttons, keyboard numbers, dropdown menu

#### 2.2.3 Select Instrument for Entry

**Conceptual Triggers:**
- **Staff Selection**: Click on staff selects that instrument context
- **Explicit Selection**: Instrument picker/dropdown
- **Keyboard**: Cycle through instruments with modifier key

**Platform Implementations:**
- **iOS/macOS**: Tap staff, instrument picker in toolbar, ⌘[1-4] shortcuts
- **Android**: Tap staff, drawer menu for instruments, hardware keys
- **Windows**: Click staff, ribbon instrument selector, Ctrl+[1-4]
- **Linux**: Click staff, toolbar dropdown, Ctrl+[1-4]

### 2.3 Cursor and Insertion Point Management

**Cursor Behavior:**

```
Cursor State Management:
├── After Deletion
│   └── Cursor moves to position of deleted element
├── After Insertion
│   └── Cursor advances to next note position (right of inserted note)
├── After Arrow Key Navigation
│   └── Cursor moves to adjacent element
├── After Click/Tap
│   └── Cursor moves to clicked position
└── Visual Feedback
    ├── Blinking cursor line at insertion point
    ├── Cursor shown between notes (not on notes)
    └── Cursor respects staff line quantization
```

**Insertion Point Rules:**

```
Insertion Point Determination:
├── No Selection + No Cursor
│   └── Insert at beginning of first measure
├── No Selection + Cursor Present
│   └── Insert at cursor position
├── Note Selected
│   └── Replace selected note(s)
├── Measure Selected
│   └── Replace first note position in measure
└── System Selected
    └── Replace first note position in first measure
```

### 2.4 Rapid Entry Workflow (Expert Users)

**Keyboard-Driven Entry Pattern:**

For experienced users, enable rapid keyboard-only note entry:

```
Keyboard Entry Workflow:
├── Press note letter (A-G) → inserts note at cursor, advances cursor
├── Press duration number (1, 2, 4, 8) → changes next note duration
├── Press Up/Down arrow → transposes last inserted note
├── Press Space/Enter → advances to next beat position
├── Press modifier + letter → adds accidental (sharp/flat)
├── Press Backspace/Delete → removes last note, moves cursor back
└── Selection shortcuts work immediately (⌘⇧D for doubling, etc.)
```

**Platform-Specific Shortcuts Defined in Platform Specs:**
- Each platform defines its own modifier keys (⌘, Ctrl, Alt, etc.)
- Follows platform conventions (e.g., macOS uses ⌘, Windows uses Ctrl)
- Documented in platform implementation specifications

### 2.5 Touch/Stylus Direct Manipulation

**Touch/Pencil Entry Pattern:**

For touch and stylus-enabled platforms:

```
Direct Entry Workflow:
├── Tap note duration tool in palette
├── Tap on staff line/space → inserts note at that pitch
├── Drag vertically on note → changes pitch
├── Drag horizontally on note → changes timing position
├── Pinch note → changes duration
├── Long-press note → opens context menu
└── Two-finger tap → undo last action
```

**Stylus-Specific Enhancements (iPad, Surface, Linux tablets):**
- Handwriting recognition for note letters (future enhancement)
- Gesture recognition for common embellishments
- Pressure sensitivity for velocity/dynamics (future enhancement)

### 2.6 Error Prevention and Validation

**Real-Time Validation During Entry:**

```
Entry Validation:
├── Duration Validation
│   ├── Check if note fits in remaining measure duration
│   ├── Show red outline if would overflow
│   ├── Offer to split note or add measure
│   └── Block insertion until resolved or user confirms
├── Pitch Validation
│   ├── Check if pitch is in instrument range
│   ├── Warn if unusual pitch (rare but allowed)
│   └── Transpose suggestion if out of range
└── Context Validation
    ├── Ensure note type matches instrument
    ├── Validate against current time signature
    └── Check for musical logic errors
```

### 2.7 Undo/Redo Integration

**Undo Granularity for Note Entry:**

```
Undo Point Creation:
├── After Each Note Insertion
│   └── Undo removes last inserted note, restores cursor
├── After Duration Change
│   └── Undo reverts duration, keeps note
├── After Pitch Change
│   └── Undo reverts pitch, keeps note
├── After Selection Replacement
│   └── Undo restores original selection
└── Batch Operations
    └── Multiple rapid entries can be grouped (< 2 seconds apart)
```

---

## 3. Selection Patterns

### 3.1 Selection Scope

**Selectable Element Types:**

```
Selectable Elements (Hierarchical):
├── Individual Notes
├── Rests
├── Embellishments (grace notes)
├── Measures (selects all content)
├── Systems (selects all instruments' measures)
├── Parts (selects all systems in part)
├── Tunes (selects all parts)
└── Full Score (selects everything)
```

### 3.2 Selection Methods

**Single Selection:**
- **Click/Tap**: Single element selection
- **Visual Feedback**: Selected element highlighted with selection color
- **Deselection**: Click/tap on empty space or press Escape

**Multiple Selection:**
- **Shift+Click**: Extend selection (range selection for sequential elements)
- **Modifier+Click**: Add/remove individual elements (⌘/Ctrl+Click)
- **Drag Rectangle**: Select all elements within dragged area
- **Select All**: Keyboard shortcut (⌘A/Ctrl+A) or menu command

**Hierarchical Selection:**
- **Click Measure**: Selects entire measure (all notes, rests)
- **Click System**: Selects entire system (all instruments, all measures)
- **Click Part Header**: Selects entire part (all systems)
- **Context Determines Scope**: Single click on measure bar selects measure

### 3.3 Selection Feedback

**Visual Indication Requirements:**

```
Selection States:
├── Selected (Primary)
│   ├── Highlight color (platform theme-aware)
│   ├── Selection handles (for drag/resize)
│   └── Bounding box (for groups)
├── Hover/Focus (Non-Touch)
│   ├── Subtle highlight or outline
│   └── Cursor change (pointer, hand)
├── Multi-Selection
│   ├── All selected elements highlighted
│   └── Selection count indicator
└── Disabled (Cannot Select)
    └── Dimmed appearance, no interaction
```

**Accessibility Requirements:**
- Screen reader announces selection changes
- Selection visible in high contrast modes
- Keyboard-only navigation supports all selection methods
- Focus indicator separate from selection indicator

---

## 4. Context-Aware Keyboard Shortcuts

### 4.1 Context-Aware Shortcut Philosophy

**Principle: Shortcuts Adapt to Selection**

Rather than having fixed shortcuts for all actions, shortcuts become available based on what's currently selected:

```
Context-Aware Shortcut Model:
├── Note Selected
│   ├── Embellishment shortcuts active
│   ├── Duration shortcuts active
│   ├── Pitch shortcuts active
│   └── Articulation shortcuts active
├── Measure Selected
│   ├── Time signature shortcuts active
│   ├── Barline shortcuts active
│   ├── Volta/ending shortcuts active
│   └── System break shortcuts active
├── System Selected
│   ├── Staff spacing shortcuts active
│   ├── System break shortcuts active
│   └── Instrument visibility shortcuts active
├── Part Selected
│   ├── Part reordering shortcuts active
│   ├── Repeat count shortcuts active
│   └── Part properties shortcuts active
└── No Selection
    ├── Navigation shortcuts active
    ├── File operations active
    └── View operations active
```

### 4.2 Note Context Shortcuts (When Note Selected)

**Embellishment Application:**
- Modifier+Shift+Letter combinations for embellishments
- Examples (platform defines actual modifiers):
  - D: Doubling
  - L: Grip (Leamluath)
  - T: Taorluath
  - F: Flam (drums)
  - R: Rough (drums)

**Duration Changes:**
- Number keys (1, 2, 4, 8, 16, 32) change selected note duration
- Dot key adds/removes duration dot

**Pitch Changes:**
- Up/Down arrows transpose selected note
- Accidental keys add sharp/flat/natural

### 4.3 Measure Context Shortcuts (When Measure Selected)

**Barline and Volta Operations:**
- Modifier+Shift+Number for ending brackets
- Examples:
  - 1: First ending
  - 2: Second ending
  - V: Generic volta

**Time Signature:**
- Modifier+T: Open time signature dialog
- Quick shortcuts for common signatures (4/4, 3/4, 6/8)

**System Breaks:**
- Modifier+Return: Insert system break after measure

### 4.4 System Context Shortcuts (When System Selected)

**Layout Operations:**
- Modifier+Return: Insert system break
- Modifier+Shift+S: Adjust staff spacing
- Modifier+Shift+I: Toggle instrument visibility

### 4.5 Universal Shortcuts (Work in Any Context)

**File Operations:**
- Modifier+N: New score
- Modifier+O: Open score
- Modifier+S: Save score
- Modifier+P: Print/Export

**Edit Operations:**
- Modifier+Z: Undo
- Modifier+Shift+Z or Modifier+Y: Redo
- Modifier+X: Cut
- Modifier+C: Copy
- Modifier+V: Paste (replaces selection)
- Delete/Backspace: Delete selected element(s)

**View Operations:**
- Modifier+Plus: Zoom in
- Modifier+Minus: Zoom out
- Modifier+0: Actual size
- Modifier+9: Fit to width

**Navigation:**
- Arrow keys: Navigate between elements
- Page Up/Down: Navigate between systems
- Home/End: Jump to start/end
- Modifier+F: Find/Search

### 4.6 Shortcut Discoverability

**Teaching Users the Context-Aware Shortcuts:**

```
Shortcut Discoverability Features:
├── Context Menu Integration
│   └── Show keyboard shortcut next to every menu item
│       Examples: 
│       - "Doubling        ⌘⇧D" (note context)
│       - "1st Ending      ⌘⇧1" (measure context)
│       - "System Break    ⌘Return" (system context)
│
├── Toolbar Tooltips
│   └── Hover over button shows:
│       - Action name
│       - Keyboard shortcut
│       - Required selection type
│       Example: "Doubling (⌘⇧D) - Requires note selection"
│
├── Help Overlay (Press ? or F1)
│   ├── Searchable list of ALL shortcuts
│   ├── Organized by selection context:
│   │   ├── "Note Actions"
│   │   ├── "Measure Actions"
│   │   ├── "System Actions"
│   │   ├── "Part Actions"
│   │   └── "Global Actions"
│   ├── Visual keyboard diagram with shortcuts highlighted
│   ├── Filter by selection type currently active
│   └── Press Escape to dismiss
│
├── Inspector Panel
│   └── Each property section shows shortcut hints
│       Examples:
│       - "Embellishment: Doubling (⌘⇧D)"
│       - "Volta: 1st Ending (⌘⇧1)"
│       - "System Break (⌘Return)"
│
├── Status Bar Context Hints
│   └── Shows available shortcuts for current selection
│       Examples:
│       - "Note selected: ⌘⇧D-Doubling, ⌘⇧L-Grip, ⌘⇧T-Taorluath"
│       - "Measure selected: ⌘⇧1-1st End, ⌘⇧2-2nd End, ⌘⇧V-Volta"
│
└── First-Time Tutorial
    ├── Introduce context-aware shortcuts during onboarding
    ├── Interactive tutorial: "Select a note, then press ⌘⇧D"
    ├── Show context menu with shortcuts highlighted
    └── Optional: Show shortcut hints on first few uses
```

---

## 5. Embellishment Attachment Workflow

### 5.1 Embellishment Workflow Pattern

**Context-Aware Embellishment Actions:**

```
Embellishment Attachment Workflow:
├── Step 1: Select Note(s)
│   ├── Click/tap on principal note
│   └── Multiple notes for batch embellishment
├── Step 2: Choose Embellishment
│   ├── Context menu → Embellishments submenu
│   ├── Keyboard shortcut (⌘⇧Letter) when note selected
│   ├── Toolbar button → Embellishment palette
│   └── Inspector panel embellishment picker
└── Step 3: Confirm & Apply
    ├── Embellishment attached to note
    ├── Visual feedback (grace notes rendered)
    ├── Validation warnings (if prior note incompatible)
    └── Undo point created
```

### 5.2 Embellishment Selection Methods

**Platform-Agnostic Action Triggers:**

#### 5.2.1 Keyboard Shortcut Approach (Primary for Experts)

**Workflow:**
1. Select principal note(s)
2. Press context-aware embellishment shortcut
3. Embellishment applied immediately

**Example Shortcuts (Platform Adapts Modifiers):**
- **Note selected** + Modifier+Shift+D → Doubling
- **Note selected** + Modifier+Shift+T → Taorluath
- **Note selected** + Modifier+Shift+L → Grip (Leamluath)
- **Note selected** + Modifier+Shift+F → Flam (drums)
- **Note selected** + Modifier+Shift+R → Rough (drums)

#### 5.2.2 Context Menu Approach (Secondary)

**Workflow:**
1. Select principal note(s)
2. Right-click / Long-press / Two-finger tap
3. Context menu appears with "Add Embellishment" option
4. Submenu shows categorized embellishments:
   - Pipe Band Embellishments (Doubling, Throw, Grip, etc.)
   - Drum Embellishments (Flam, Drag, Rough, etc.)
5. Click embellishment name to apply (shortcut shown in menu)
6. Embellishment appears on note immediately

**Platform Implementations:**
- **iOS/macOS**: Long-press or Control+Click → Context menu
- **Android**: Long-press → Bottom sheet or popup menu
- **Windows**: Right-click → Context menu (Fluent Design)
- **Linux**: Right-click → Context menu (desktop environment style)

#### 5.2.3 Toolbar/Palette Approach (Tertiary)

**Workflow:**
1. Select principal note(s)
2. Click embellishment palette button in toolbar
3. Palette appears showing embellishment options with icons/names
4. Click embellishment to apply to selection
5. Palette remains open for applying to additional notes

**Platform Implementations:**
- **iOS/macOS**: Toolbar button → Popover palette
- **Android**: FAB → Expanding menu or full-screen picker
- **Windows**: Ribbon tab → Embellishment gallery
- **Linux**: Toolbar button → Dropdown palette

#### 5.2.4 Inspector Panel Approach (Quaternary)

**Workflow:**
1. Select principal note(s)
2. Inspector panel shows note properties
3. Embellishment section shows "None" or current embellishment
4. Click embellishment picker
5. Choose from categorized list
6. Applied immediately to selection

**Platform Implementations:**
- **iOS/macOS**: Right sidebar inspector, iPad bottom sheet inspector
- **Android**: Right drawer or bottom sheet
- **Windows**: Right pane inspector (Task Pane)
- **Linux**: Right sidebar or floating palette

### 5.3 Changing Embellishments

**Modification Pattern:**

```
Change Existing Embellishment:
├── Select note with embellishment
├── Inspector shows current embellishment type
├── Methods to change:
│   ├── Keyboard shortcut → New embellishment (replaces current)
│   ├── Context menu → Change Embellishment → [new type]
│   ├── Inspector → Embellishment picker → [new type]
│   └── Delete embellishment (Delete key) → Add new one
└── Validation checks prior note constraints
```

**Remove Embellishment:**
- Select note with embellishment
- Press Delete/Backspace (if note selected, removes embellishment only)
- OR Context menu → Remove Embellishment
- OR Inspector → Embellishment picker → None

### 5.4 Batch Embellishment Operations

**Apply to Multiple Notes:**

```
Batch Embellishment Workflow:
├── Select multiple principal notes (Shift+Click or drag select)
├── Choose embellishment via any method
├── Application rules:
│   ├── Validate each note individually
│   ├── Skip incompatible notes (show warning)
│   ├── Apply to all compatible notes
│   └── Show summary: "Applied to 5 of 7 notes, 2 skipped"
└── Single undo operation for entire batch
```

---

## 6. Musical Editing Actions

### 6.1 Note Editing Actions

**Common Note Operations:**

#### 6.1.1 Delete Note(s)

**Triggers:**
- Select note(s) → Press Delete/Backspace
- Select note(s) → Context menu → Delete
- Select note(s) → Edit menu → Delete

**Behavior:**
- Removes note(s) from measure
- Adjusts measure duration (may add rest)
- Cursor moves to deletion position
- Undo point created

#### 6.1.2 Change Note Pitch

**Triggers:**
- Select note → Up/Down arrow keys (transpose)
- Select note → Drag vertically (touch/mouse)
- Select note → Inspector → Pitch picker

**Behavior:**
- Updates pitch within instrument range
- Validates pitch compatibility
- Visual update immediate
- Cursor stays at note

#### 6.1.3 Change Note Duration

**Triggers:**
- Select note(s) → Press duration number key (1, 2, 4, 8)
- Select note(s) → Toolbar duration picker
- Select note(s) → Inspector → Duration picker
- Select note(s) → Context menu → Duration → [value]

**Behavior:**
- Updates note duration
- Validates measure duration doesn't overflow
- Adjusts subsequent note positions if needed
- Cursor stays at note

#### 6.1.4 Add Accidental

**Triggers:**
- Select note → Keyboard shortcut (platform-specific: # for sharp, b for flat)
- Select note → Context menu → Add Accidental → [Sharp/Flat/Natural]
- Select note → Inspector → Accidental picker
- Select note → Toolbar accidental buttons

**Behavior:**
- Adds accidental to note (sharp, flat, natural)
- Updates pitch accordingly
- Renders accidental symbol before note
- Cursor stays at note

#### 6.1.5 Add Articulation

**Triggers:**
- Select note(s) → Context menu → Articulation → [type]
- Select note(s) → Inspector → Articulation checkboxes
- Select note(s) → Keyboard shortcut (platform-specific)
- Select note(s) → Toolbar articulation palette

**Articulation Types:**
- Staccato (dot above/below note)
- Accent (> symbol)
- Tenuto (line above/below note)
- Fermata (hold)
- Dynamic markings (integrated with NoteGroup system)

### 6.2 Measure Operations

**Measure-Level Actions:**

#### 6.2.1 Insert Measure

**Triggers:**
- Click between measures → Insert button
- Select measure → Context menu → Insert Before/After
- Keyboard shortcut (platform-specific)
- Menu → Insert → Measure

**Behavior:**
- Inserts empty measure (with rests)
- Inherits time signature from previous measure
- Shifts subsequent measures
- Cursor moves to new measure

#### 6.2.2 Delete Measure

**Triggers:**
- Select measure → Delete/Backspace
- Select measure → Context menu → Delete Measure
- Menu → Edit → Delete Measure

**Behavior:**
- Removes entire measure (all instruments)
- Shifts subsequent measures left
- Confirmation dialog if measure contains notes
- Cursor moves to deletion position

#### 6.2.3 Change Time Signature

**Triggers:**
- Select measure → Context menu → Time Signature
- Select measure → Inspector → Time Signature picker
- Select measure → Keyboard shortcut (⌘T / Ctrl+T)
- Double-click time signature → Edit dialog

**Options:**
- Apply to this measure only
- Apply to this measure and all subsequent (cascade)
- Apply to entire tune

**Behavior:**
- Updates time signature
- Validates note durations still fit
- May prompt to adjust existing content
- Cursor stays at measure

#### 6.2.4 Set Barline Type

**Triggers:**
- Select measure → Context menu → Barline → [type]
- Select measure → Inspector → Opening/Closing Barline picker
- Click barline directly → Picker appears

**Behavior:**
- Changes opening or closing barline type
- Updates visual rendering
- Handles repeat barlines with dots
- Cursor stays at measure

#### 6.2.5 Add Ending Brackets (Volta)

**Triggers:**
- Select measure → Keyboard shortcut (⌘⇧1 for 1st ending, ⌘⇧2 for 2nd)
- Select measure → Context menu → Add Ending → [1st/2nd/etc.]
- Select measure → Inspector → Volta picker

**Behavior:**
- Adds ending bracket above measure
- Configures ending number and repeat behavior
- Visual bracket rendered
- Cursor stays at measure

### 6.3 Part and Tune Operations

**Higher-Level Actions:**

#### 6.3.1 Add Part

**Triggers:**
- Toolbar → Add Part button
- Menu → Insert → Part
- Context menu on part list → Add Part

**Behavior:**
- Inserts new part (A, B, C, etc.)
- Prompts for part letter/name
- Inherits instruments from tune
- Creates empty measures with default time signature

#### 6.3.2 Reorder Parts

**Triggers:**
- Drag part header in part list (touch/mouse)
- Part list context menu → Move Up/Down
- Inspector → Part order spinner

**Behavior:**
- Changes play order
- Updates navigation
- Affects pagination

#### 6.3.3 Add Tune

**Triggers:**
- Document menu → Add Tune
- Toolbar → New Tune button
- Context menu in tune list → Add Tune

**Behavior:**
- Creates new tune in score
- Prompts for title, type, tempo
- Creates default A part
- Updates pagination

---

## 7. Copy/Paste Operations

### 7.1 Copy/Paste Scope

**Copyable Elements:**

```
Copy/Paste Support:
├── Individual Notes
│   ├── Copies pitch, duration, embellishment
│   ├── Pastes at cursor or replaces selection
│   └── Validates compatibility with target
├── Measures
│   ├── Copies entire measure content (all instruments)
│   ├── Copies time signature and barlines
│   └── Pastes as new measure or replaces selection
├── Systems
│   ├── Copies all measures in system
│   ├── All instruments included
│   └── Pastes as new systems
└── Parts
    ├── Copies entire part structure
    ├── All systems and instruments
    └── Pastes as new part
```

### 7.2 Copy/Paste Behavior

**Paste Logic:**

```
Paste Behavior:
├── Paste at Cursor
│   ├── Inserts copied content at current position
│   ├── Shifts subsequent content right
│   └── Validates measure duration doesn't overflow
├── Paste Replace (Selection Present)
│   ├── Replaces selected content with copied content
│   ├── Maintains measure boundaries
│   └── Asks for confirmation if replacing non-empty
└──