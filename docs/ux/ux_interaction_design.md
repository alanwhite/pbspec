# User Experience and Interaction Design Specification

**Version:** 3.0  
**Last Updated:** October 13, 2025  
**Status:** Complete Integrated Specification 

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
**Primary Principle: Word Processor for Music**

The application follows the familiar interaction model of a word processor, adapted for musical notation:

**Just Like Writing a Document:**

- Type to Create: Click where you want a note and type (keyboard) or tap (touch) to add musical elements
- Select to Modify: Click or tap any element to select it, just like selecting text
- Style and Transform: Use tools, menus, or inspectors to modify selected elements
- Direct Manipulation: Drag to move, resize, or reposition elements visually

**Core Behaviors:**

The behaviours are instrument specific, in the sense that if the cursor is on a bagpipe staff there are different keyboard shortcuts and approach than when it's placed on a drum staff.

```
Add Content:
├── Click on staff → Cursor moves to staff position
├── Bagpipe: Type note letters (A-G, Shift-A for high A, shift-G for high G) → Notes added sequentially
├── Bagpipe: Type durations (1, 2, 4, 8) → Changes next note length
├── Drum: Type from the left most key in the middle row (a on a uk keyboard) for a whole note on the right hand, next key in for half etc. Shift and key enters on the left hand
└── Backspace/Delete → Remove notes like deleting text

Modify Content:
├── Click element → Select (like selecting a word)
├── Drag select → Select multiple (like highlighting text)
├── Right-click → Context menu (like in Word, Photoshop)
├── Inspector panel → Detailed properties (like formatting sidebar)
└── Undo/Redo → Standard document editing behavior
```

**Natural Workflows:**

**Adding Notes (Like Typing):**

- Click where you want to start on the staff
- Type note letters or click pitches in a tool
- Notes flow naturally like text on a page
- Current duration applies to new notes
- Press space to advance, backspace to delete

**Editing (Like Formatting Text):**

- Click a note to select it (highlights like selected text)
- Shift+click to select multiple notes (range selection)
- Ctrl/Cmd+click to add to selection (multi-select)
- Right-click for context menu with relevant actions
- Inspector shows properties, change them like font/paragraph settings

**Embellishments (Like Character Formatting):**

- Select one or more notes (like selecting words)
- Choose embellishment from menu or palette or inspector
- Embellishment applies to selection immediately
- Visible feedback like bold or italic being applied

**Rationale:**

- Universally familiar paradigm - everyone has used a word processor
- Minimal learning curve for musicians who aren't software experts
- Natural mapping: musical elements are like words and phrases
- Supports rapid input for expert users (keyboard typing)
- Accessible for casual users (pointing and clicking)
- Same mental model across all platforms and input methods
- Consistent with other creative applications (Photoshop, Illustrator, Word)

**What This Means for Design:**

- No mode indicators or mode switching required
- Tools and options always visible or contextually available
- Focus on content, not on application state
- Keyboard shortcuts enhance speed without being mandatory
- Touch and mouse interactions feel equivalent
- Learning one interaction teaches all similar actions

### 1.2 Input Method Support

**Multi-Modal Input Strategy:**

The application supports multiple simultaneous input methods, adapting intelligently to available hardware:

```
Input Methods (Platform-Dependent):
├── Keyboard 
│   ├── Bagpipe: Note entry via letter keys (A-G, with shift for high A and high G)
│   ├── Bagpipe: Duration selection via number keys
│   ├── Bass, Tenor or Snare Drum: Note or Rest entry via letter keys denoting note duration
│   ├── Bass, Tenor or Snare Drum: Unshifted key is left hand, shifted is right, ctrl is rest
│   ├── Navigation via arrow keys
│   ├── Context-aware action shortcuts
│   └── Paste Merge
├── Pointing Device (Mouse, Trackpad, Touch)
│   ├── Click/tap to select
│   ├── Right-click/long-press for context menu
│   ├── Drag for multiple selection
│   ├── Drag and Drop for moving selection
│   └── Hover for tooltips (non-touch platforms)
└── Touch Gestures (iOS, Android, Windows Touch)
    ├── Tap to select
    ├── Selection has grab areas to extend / reduce selection
    ├── Long-press for context menu
    ├── Pinch to zoom
    ├── Two-finger pan to scroll
    └── Swipe for navigation
```


**Selectable Elements**

```
Selectable Elements (Hierarchical):
├── Individual Notes and Rests
├── Instrument Measure (selects all content in a measure for one instrument)
├── Measure (selects the measure(s) across all Instruments)
├── Systems (selects all measures across all Instruments in the selected System(s))
├── Parts (selects all systems in part)
├── Tune Header (selects header of a tune, Inspector shows modifiable items)
├── Tunes (selects Tune Header and all parts in a Tune)
└── Full Score (selects everything)
```

**Selection Methods**

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


**Cross-Document Copy/Paste:**

```
Cross-Document Behavior:
├── Clipboard Format
│   ├── Internal format (full fidelity)
└── Validation
    └── Check instrument compatibility
```

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

---

## 2. Note Entry Workflow

### 2.1 Continuous Entry Pattern

**Core Principle: Click and Type, Like a Word Processor**

The application allows immediate note entry at any time, just like typing in a document:

**Basic Entry Flow:**

- Click on staff → Cursor appears at that position
- Type note letter (A-G, Shift-A, Shift-G) → Note appears
- Type duration (1, 2, 4, 8) → Changes duration for next note
- Cursor moves forward to next insert position, based on note duration
- Continue typing → Notes flow sequentially
- Backspace/Delete → Removes previous note

**Entry with Existing Selection:**
```
If Note(s) or Rest(s) Selected:
├── Type note or rest letter → Replaces selection with new note or rest placed at the start of the selected range
├── Type duration → Changes selected note(s) duration
├── Paste → Replaces selection with pasted content if compatible with instrument
└── Embellishment shortcut → toggles embellishment presence

If Measure for all Instruments in the System Selected:
├── Shortcut key to Insert Measure and all Instruments before
├── Shortcut key to Insert Measure and all Instruments before
├── Shortcut keys to mark Measure as 1st, 2nd, 3rd or 4th time only
├── Paste → Replaces measure if copied data represents measure for all instruments
└── Click on Instrument staff → Deselects measure, positions cursor

If Measure for individual Instrument Selected:
├── Type note letter → Replaces measure content with new note
├── Paste → Replaces measure content if compatible with Instrument type
└── Click on staff → Deselects measure, positions cursor

If Nothing Selected:
├── Type note letter → Adds note at cursor position
└── Click on staff → Moves cursor to that position
```

**Visual Feedback:**
- Cursor (insertion point) always visible when no selection
- Selected elements highlighted (like selected text)
- New content appears immediately as you type
- Duration tool indicator shows what length notes will be if cursor on bagpipe instrument staff

**Rationale:**
- Identical mental model to word processing: click where you want content, start typing
- Selection acts as target for replacement (like selecting text and typing over it)
- Cursor acts as insertion point when nothing selected
- Rapid workflow: click-type-click-type without interruption
- Natural for musicians: think in terms of "put notes on staff"
- Keyboard shortcuts enhance speed without being mandatory

**Platform Implementation Notes:**
- Click/tap positioning must be accurate to staff space
- Cursor visibility must be clear (blinking vertical line)
- Selection highlight must be distinct from cursor
- Duration tool state must be persistently visible if cursor on bagpipe staff
- Undo must work seamlessly (Ctrl/Cmd+Z at any time)

### 2.2 Note Entry Actions (Platform-Agnostic)

**Action Trigger Methods:**

All platforms must support these conceptual actions through native input methods:

#### 2.2.1 Insert Note Action

**Trigger Options (Platform Implements Native Method):**
- **Keyboard**: Press note letter key (A-G, Shift-A, Shift-G) for bagpipes, or hand and duration for drum notes and rests
- **Click/Tap**: Click on staff line/space
- **Context Menu**: Right-click > Insert Note > [pitch]

**Behavior:**
- If cursor visible (no selection): Inserts note at cursor position
- If note(s) selected: Replaces selection with new note
- If measure for individual instrument selected: Replaces measure content, starts at beginning
- Duration determined by current duration tool setting
- Cursor advances to next logical position after insertion

**Required Parameters:**
- Pitch (determined by staff position or key press)
- Duration (from current tool setting)
- Instrument (from current staff context)

**Platform Implementations:**
- **iOS/macOS**: Keyboard shortcut, toolbar buttons, tap on staff, Apple Pencil stroke
- **Android**: Hardware keyboard (if present), tap on staff, on-screen palette
- **Windows**: Keyboard shortcut, ribbon button, click on staff, Surface Pen
- **Linux**: Keyboard shortcut, toolbar button, click on staff, stylus (if present)

#### 2.2.2 Set Note Duration Action

**Conceptual Triggers:**
- **Before Entry**: Select duration tool, then enter notes (they use that duration)
- **After Entry**: Select note(s), then change duration (modifies selection)
- **Keyboard Numbers**: 1=whole, 2=half, 4=quarter, 8=eighth, etc.

**Behavior:**
- If no selection: Sets duration for next note to be entered
- If note(s) selected: Changes duration of selected note(s)
- Duration tool indicator updates to show current setting
- Validates that new duration fits within measure

**Platform Implementations:**
- **iOS/macOS**: Toolbar duration picker, keyboard numbers (1-8), duration palette
- **Android**: Bottom sheet duration selector, keyboard numbers, FAB menu
- **Windows**: Ribbon duration group, keyboard numbers, context menu
- **Linux**: Toolbar duration buttons, keyboard numbers, dropdown menu

#### 2.2.3 Position Cursor for Entry

**Conceptual Triggers:**
- **Click/Tap on Staff**: Positions cursor at that staff location
- **Arrow Keys**: Up/down moves cursor between instruments measures including to adjacent system, left/right moves cursor to the prior/next logical note position
- **Click Empty Space**: Deselects current selection, positions cursor

**Behavior:**
- Cursor appears as blinking vertical line at position
- Cursor height is the height of the 5 line staff
- Cursor position determines where next note appears
- Clicking staff while note selected deselects and moves cursor

**Platform Implementations:**
- **iOS/macOS**: Tap on staff, arrow key navigation, gesture to deselect
- **Android**: Tap on staff, hardware arrow keys (if present)
- **Windows**: Click on staff, arrow key navigation, Esc to deselect
- **Linux**: Click on staff, arrow key navigation, Esc to deselect

#### 2.2.4 Replace Selection with Note

**Conceptual Triggers:**
- **Type Note Letter**: When note(s) selected, typing replaces selection
- **Paste**: Replaces selection with clipboard content
- **Drag-and-Drop**: Drag note to selection replaces it

**Behavior:**
- Selected note(s) removed
- New note appears at position of first selected note
- Subsequent selected notes deleted
- Single undo operation for entire replacement
- Cursor advances to next position

**Platform Implementations:**
- **All Platforms**: Standard keyboard input, paste command, drag gestures
### 2.3 Rapid Note Entry (Expert Users)

**Keyboard-Driven Entry Pattern:**

For experienced users, enable rapid keyboard-only note entry without interruption:

```
Rapid Entry Workflow:
├── Click on staff to position cursor (one time)
├── Bagpipe: Press note letter (A-G, Shift-A, Shift-G) → Inserts note at cursor position and moves cursor to next logical position
├── Bagpipe: Press duration number (1, 2, 4, 8) → Changes duration for next note
├── Press Up/Down arrow → Moves cursor to prior/next instrument staff at same horizontal position
├── Press Space/Right arrow → Advances cursor to next logical note input position
├── Press Backspace/Delete → Removes note at cursor, moves cursor back
├── Press Left arrow → Moves cursor back one position
└── Continue typing → Notes flow rapidly without mouse interaction
```

**Example: Entering a Simple Melody (Bagpipe):**
```
User Actions:                          Result:
1. Click on staff at measure 1        → Cursor positioned
2. Type "4" (quarter note)             → Duration set to quarter
3. Type "D"                            → D quarter note appears
4. Type "E"                            → E quarter note appears
5. Type "F"                            → F quarter note appears
6. Type "2" (half note)                → Duration set to half
7. Type "G"                            → G half note appears
8. Type "4" (quarter note)             → Duration set to quarter
9. Type "A"                            → A quarter note appears
Total: 9 keystrokes for 5 notes with 2 duration changes
```

**Advanced Rapid Entry:**
```
Navigation:
├── Space → Advances cursor to next beat, including to next measure if no more in current measure
├── Tab → Jumps cursor to next measure
├── Shift+Tab → Jumps cursor to previous measure
├── Home → Jumps cursor to start of measure
└── End → Jumps cursor to end of measure

Copy/Paste:
├── Select note(s) with Shift+Arrow
├── Ctrl/Cmd+C → Copy selection
├── Position cursor with arrow keys
├── Ctrl/Cmd+V → Paste at cursor
└── Continue typing → Add more notes after pasted content
```

**Platform-Specific Shortcuts:**

Each platform defines its own modifier keys (⌘, Ctrl, Alt, etc.) and documents them in platform specifications:
- **iOS/macOS**: See `ios_macos_implementation.md` for Apple keyboard shortcuts
- **Android**: See `android_implementation.md` for Android keyboard shortcuts (hardware keyboard)
- **Windows**: See `windows_implementation.md` for Windows keyboard shortcuts
- **Linux**: See `linux_implementation.md` for Linux keyboard shortcuts

**Rationale:**
- Expert musicians can enter music as fast as they can think
- No context switching between mouse and keyboard
- Mimics typing speed for text entry
- Reduces repetitive strain from mouse movement
- Supports muscle memory for frequent operations
- Optional - beginners can ignore and use mouse/touch
- Keyboard shortcuts visible in menus for discovery

### 2.4 Direct Manipulation Entry (Touch/Stylus)

**Touch/Pencil Entry Pattern:**

For touch and stylus-enabled platforms, provide direct manipulation that feels natural:

```
Direct Entry Workflow:
├── Tap duration tool in palette → Sets duration for next entry
├── Tap on staff line/space → moves cursor to staff position
├── Tap on note tool → inserts note at current cursor position
├── Drag vertically on existing note → Changes pitch
├── Drag horizontally on note → Moves note timing position
├── Two-finger tap → Undo last action
├── Long-press note → Opens context menu
└── Pinch/spread on canvas → Zoom in/out
```

**Touch Entry Behaviors:**

**Insert New Note (Bagpipe):**
```
1. Ensure duration tool selected (quarter note button highlighted)
2. Tap on staff to position cursor
3. Tap required pitch from the note tool
3. Note appears immediately
4. Tap again to add next note
5. Change duration tool anytime to vary note lengths
```

**Modify Existing Note:**
```
Vertical Drag (Pitch Change):
├── Tap and hold note
├── Drag up/down
├── Note pitch updates in real-time
├── Release to confirm
└── Snaps to staff lines/spaces

Horizontal Drag (Timing Adjustment):
├── Tap and hold note
├── Drag left/right
├── Note position updates in real-time
├── Release to confirm
└── Snaps to beat positions (optional grid)

Tap to Select:
├── Single tap → Selects note
├── Tap empty space → Deselects
├── Tap other note → Switches selection
└── Long-press → Opens context menu
```

**Duration Changes:**
```
Without Dragging:
├── Tap note to select
├── Tap different duration tool
├── Selected note duration changes immediately
└── Or: Long-press → Context menu → Duration

With Pinch Gesture (Optional):
├── Tap note to select
├── Pinch horizontally on note
├── Note duration increases/decreases
└── Visual feedback shows new duration
```

**Stylus-Specific Enhancements:**

**Apple Pencil (iPad):**
- Double-tap Pencil → Toggles between select and erase
- Pencil hover → Shows pitch preview before committing
- Pressure sensitivity → Reserved for future dynamics feature
- Tilt → Reserved for future expression feature

**Surface Pen (Windows):**
- Barrel button → Right-click context menu
- Eraser tip → Deletes notes when touched
- Pressure sensitivity → Reserved for future dynamics feature
- Hover → Shows pitch preview before committing

**Stylus on Linux Tablets:**
- Button 1 → Context menu
- Button 2 → Eraser function
- Hover → Pitch preview (if hardware supports)
- Pressure → Reserved for future features

**Palm Rejection:**
- All touch platforms must implement palm rejection
- Only deliberate taps/touches register
- Resting palm on screen ignored
- Stylus takes priority over touch when both detected

**Rationale:**
- Direct manipulation feels natural on touch devices
- No need for keyboard on tablet/phone
- Visual feedback immediate and clear
- Gestures match user expectations from other apps
- Stylus provides precision for detailed work
- No mode switching - context determines behavior

## 3. Musical Editing Actions

### 3.1 Note Editing Actions

**Common Note Operations:**

#### 3.1.1 Delete Note(s)

**Triggers:**
- Select note(s) → Press Delete/Backspace
- Select note(s) → Context menu → Delete
- Select note(s) → Edit menu → Delete

**Behavior:**
- Removes note(s) from measure
- Adjusts measure duration (may add rest)
- Cursor moves to deletion position
- Undo point created

#### 3.1.2 Change Note Pitch

**Triggers:**
- Select note → Up/Down arrow keys (transpose)
- Select note → Drag vertically (touch/mouse)
- Select note → Inspector → Pitch picker

**Behavior:**
- Updates pitch within instrument range
- Validates pitch compatibility
- Visual update immediate
- Cursor stays at note

#### 3.1.3 Change Note Duration

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

#### 3.1.4 Add Accidental

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

#### 3.1.5 Add Articulation

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

### 3.2 Measure Operations

**Measure-Level Actions:**

#### 3.2.1 Insert Measure

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

#### 3.2.2 Delete Measure

**Triggers:**
- Select measure → Delete/Backspace
- Select measure → Context menu → Delete Measure
- Menu → Edit → Delete Measure

**Behavior:**
- Removes entire measure (all instruments)
- Shifts subsequent measures left
- Confirmation dialog if measure contains notes
- Cursor moves to deletion position

#### 3.2.3 Change Time Signature

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

#### 3.2.4 Set Barline Type

**Triggers:**
- Select measure → Context menu → Barline → [type]
- Select measure → Inspector → Opening/Closing Barline picker
- Click barline directly → Picker appears

**Behavior:**
- Changes opening or closing barline type
- Updates visual rendering
- Handles repeat barlines with dots
- Cursor stays at measure

#### 3.2.5 Add Ending Brackets (Volta)

**Triggers:**
- Select measure → Keyboard shortcut (⌘⇧1 for 1st ending, ⌘⇧2 for 2nd)
- Select measure → Context menu → Add Ending → [1st/2nd/etc.]
- Select measure → Inspector → Volta picker

**Behavior:**
- Adds ending bracket above measure
- Configures ending number and repeat behavior
- Visual bracket rendered
- Cursor stays at measure

### 3.3 Part and Tune Operations

**Higher-Level Actions:**

#### 3.3.1 Add Part

**Triggers:**
- Toolbar → Add Part button
- Menu → Insert → Part
- Context menu on part list → Add Part

**Behavior:**
- Inserts new part (A, B, C, etc.)
- Prompts for part letter/name
- Inherits instruments from tune
- Creates empty measures with default time signature

#### 3.3.2 Reorder Parts

**Triggers:**
- Drag part header in part list (touch/mouse)
- Part list context menu → Move Up/Down
- Inspector → Part order spinner

**Behavior:**
- Changes play order
- Updates navigation
- Affects pagination

#### 3.3.3 Add Tune

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

## 4. Context Menus

### 4.1 Context Menu Principles

**Context-Aware Menus:**
- Menu items adapt to selected element type
- Disabled items shown as dimmed (provide affordance)
- Keyboard shortcuts displayed (platform-appropriate)
- Submenus for categorical actions
- Recently used actions promoted (optional)

### 4.2 Note Context Menu

**Menu Structure:**
```
Note Context Menu:
├── Cut                                    (⌘X)
├── Copy                                   (⌘C)
├── Paste                                  (⌘V)
├── Delete                                 (Delete)
├── ─────────────
├── Toggle Embellishment ▶
│   ├── Bagpipe ▶
│   │   ├── Doubling                       (⌘⇧D)
│   │   ├── Throw                          (⌘⇧T)
│   │   ├── Grip                           (⌘⇧L)
│   │   └── ... (all pipe embellishments)
│   └── Drum ▶
│       ├── Flam                           (⌘⇧F)
│       ├── Drag                           (⌘⇧G)
│       ├── Rough                          (⌘⇧R)
│       └── ... (all drum embellishments)
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

Note that Toggle means if the referenced item is already present it is removed, if not present it is added. In the case of embellishments, if there is an existing embellishment that is not the menu item then it is replaced with that represented by the menu item.

The contents of the context menu are also context sensitive, meaning that if the selected notes are on a bagpipe staff then under Toggle Embellishment shows the list of bagpipe relevant entries, no need for a sub-menu that says Bagpipe. For Drum instruments there is no Transpose entry for changing pitch but there is a Swap Hands instead.

### 4.3 Measure Context Menu

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

### 4.4 Platform Context Menu Adaptations

**iOS/macOS:**
- Long-press triggers context menu (iOS)
- Control+Click or Right-click triggers context menu (macOS)
- Context menu appears as per the current design in the SDK
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

## 5. Inspector Panel Design

### 5.1 Inspector Purpose

**Properties Panel for Selected Elements (Like Word's Formatting Sidebar)**

The inspector panel shows properties of currently selected element(s) and allows modification, just like the formatting sidebar in Microsoft Word or the Inspector in design tools:

**Core Behavior:**
- Always visible (or easily accessible via button/shortcut)
- Automatically adapts to show properties of current selection
- Changes apply immediately (no "Apply" button needed)
- Empty state when nothing selected (shows instructions)
- Common properties only when multiple items selected (batch editing)

```
Inspector Panel Contents (Context-Aware):
├── Nothing Selected:
│   └── Empty State
│       ├── Icon and message: "Select an element to view properties"
│       ├── Quick tips for getting started
│       └── Recent actions or frequently used tools
│
├── When Note Selected:
│   ├── Note Properties
│   │   ├── Pitch Picker (dropdown or piano keyboard)
│   │   ├── Duration Picker (whole, half, quarter, etc.)
│   │   ├── Embellishment Picker (None, Doubling, Grip, etc.)
│   │   ├── Accidental Toggle (None, Sharp, Flat, Natural)
│   │   ├── Articulation Checkboxes (Staccato, Accent, Tenuto)
│   │   └── Note Groups (Ties, Slurs, Tuplets - references)
│   └── Instrument-Specific Properties
│       ├── For Pipe Notes: Pitch class and octave
│       ├── For Snare Notes: Hand (L/R), Stick technique
│       ├── For Tenor Notes: Drum position, Hand
│       └── For Bass Notes: Hand (L/R)
│
├── When Measure Selected:
│   ├── Measure Properties
│   │   ├── Time Signature (4/4, 3/4, 6/8, etc.)
│   │   ├── Opening Barline (None, Single, Double, Repeat Start)
│   │   ├── Closing Barline (Single, Double, Repeat End, Final)
│   │   ├── Rehearsal Mark (text field)
│   │   └── Measure Width (layout hint - slider or number)
│   └── Content Summary
│       ├── Note count per instrument
│       ├── Total duration vs. time signature
│       └── Validation warnings (if any)
│
├── When System Selected:
│   ├── System Properties
│   │   ├── Staff Spacing (slider - distance between instruments)
│   │   ├── System Break Toggle (force break after this system)
│   │   ├── Instrument Visibility (checkboxes - show/hide per instrument)
│   │   └── Clef Changes (per instrument if needed)
│   └── System Summary
│       ├── Measure count
│       ├── Instruments included
│       └── Total width
│
├── When Part Selected:
│   ├── Part Properties
│   │   ├── Part Name (text field - "Part A", "Intro", etc.)
│   │   ├── Part Letter (single character - A, B, C)
│   │   ├── Play Order (number - sequence in tune)
│   │   ├── Repeat Count (number - how many times to repeat)
│   │   └── Tempo Override (optional - BPM for this part)
│   └── Part Summary
│       ├── System count
│       ├── Measure count
│       └── Duration (calculated play time)
│
├── When Embellishment Selected (via note):
│   ├── Embellishment Properties
│   │   ├── Embellishment Type (Doubling, Grip, Throw, etc.)
│   │   ├── Regional Style (Highland, Border, Competition)
│   │   ├── Grace Note Spacing (tight, normal, loose)
│   │   ├── Float Before Barline (checkbox - for first note in measure)
│   │   └── Execution Notes (read-only info about performance)
│   └── Validation Status
│       ├── Prior Note Compatibility (check mark or warning)
│       ├── Principal Note Compatibility (check mark or warning)
│       └── Musical Context (appropriate for tune type?)
│
└── When Multiple Items Selected:
    ├── Common Properties Only
    │   ├── If all notes: Duration, Articulation (batch edit)
    │   ├── If all measures: Time Signature (batch edit)
    │   └── If mixed types: Very limited common properties
    ├── Selection Summary
    │   ├── Count of selected items by type
    │   ├── Total duration (if applicable)
    │   └── Instruments affected
    └── Batch Actions
        ├── Clear All (remove all selected)
        ├── Copy Properties (copy from first to all)
        └── Reset to Default (reset all to defaults)
```

### 5.2 Inspector Placement (Platform-Specific)

**Platform-Specific Placement:**

- **iOS/macOS**: 
  - Right sidebar (iPad landscape, Mac)
  - Bottom sheet (iPad portrait, iPhone)
  - Collapsible with button or swipe gesture
  - Persists across sessions (remember state)

- **Android**: 
  - Right drawer (tablets)
  - Bottom sheet (phones)
  - Swipe from right edge to open/close
  - Material Design styling

- **Windows**: 
  - Right task pane (collapsible)
  - Docked or floating (user choice)
  - Fluent Design acrylic effects
  - Resizable width

- **Linux**: 
  - Right sidebar (dockable)
  - Floating palette (optional)
  - Desktop environment theme integration
  - User-configurable position

### 5.3 Inspector Interactions

**Property Editing Patterns:**

**Pickers and Dropdowns:**
```
Click Picker → Opens Dropdown/Modal:
├── Shows available options (e.g., embellishment types)
├── Current value highlighted
├── Click option → Applies immediately
├── Close picker → Change saved
└── Undo available if mistake
```

**Checkboxes (Boolean Properties):**
```
Click Checkbox → Toggles State:
├── Checked → Enabled
├── Unchecked → Disabled
├── Intermediate (if batch editing) → Mixed state
└── Change applies immediately
```

**Sliders (Numeric Ranges):**
```
Drag Slider → Updates Value:
├── Real-time preview as dragging
├── Value label shows number
├── Release → Change applied
└── Can also type number directly
```

**Text Fields:**
```
Click Field → Enter Text:
├── Cursor appears in field
├── Type or paste text
├── Press Enter → Confirms change
├── Press Escape → Cancels edit
└── Blur (click away) → Confirms change
```

**Apply Pattern:**
- **No "Apply" button** - changes apply immediately
- Undo available for all changes (Ctrl/Cmd+Z)
- Visual feedback on change (highlight, flash, or animation)
- Validation errors shown inline with helpful messages
- Invalid values prevented (can't type letters in number field)

**Batch Editing:**
```
Multiple Items Selected:
├── Only common properties shown
├── Mixed values shown as placeholder text or intermediate state
├── Changing property → Applies to all selected
├── Warning if some items cannot accept change
└── Single undo operation for batch change
```

**Performance Considerations:**
- Changes to multiple items processed efficiently
- Large selections (100+ items) show progress indicator
- Real-time preview disabled for very large selections
- Commit change after slider release, not during drag

**Discoverability:**
- Tooltips on hover (desktop platforms)
- Help icon next to complex properties
- Link to documentation for advanced features
- Keyboard shortcuts shown in tooltips
- Context-sensitive help based on selection type

**Accessibility:**
- All controls keyboard navigable
- Screen reader announces property names and values
- Value changes announced to screen reader
- High contrast support
- Large touch targets on touch platforms

### 5.4 Inspector Empty State

**When Nothing Selected:**

```
Empty State Content:
├── Icon (magnifying glass or selection cursor)
├── Primary Message
│   └── "Select an element to view and edit its properties"
├── Secondary Message (Tips)
│   ├── "Click a note to see pitch and duration options"
│   ├── "Select a measure to change time signature or barlines"
│   ├── "Choose multiple items to batch edit common properties"
│   └── [Rotate through tips on each view]
└── Quick Actions (Optional)
    ├── Create New Score
    ├── Open Recent Score
    └── View Tutorial
```

**Rationale:**
- Educates new users about inspector purpose
- Provides actionable guidance
- Not blank/confusing when nothing selected
- Helpful tips improve feature discovery


---

## 6. Accessibility Requirements

### 6.1 Keyboard Navigation

**Full Keyboard Accessibility:**

The application must support complete keyboard navigation without mode switching. All actions available via mouse/touch must also be keyboard-accessible.

```
Keyboard Navigation Requirements:
├── Tab Order
│   ├── Logical focus order through all interactive elements
│   ├── Toolbar → Score content → Inspector panel
│   ├── Within score: Staff-by-staff, measure-by-measure
│   ├── Shift+Tab reverses direction
│   └── Skip links available for fast navigation
│
├── Score Navigation (Without Selection)
│   ├── Arrow keys navigate cursor position
│   │   ├── Up/Down → Changes pitch (staff line/space)
│   │   ├── Left/Right → Moves timing position
│   │   └── Visual cursor always visible at current position
│   ├── Tab → Next staff/instrument
│   ├── Shift+Tab → Previous staff/instrument
│   ├── Home → Start of current measure
│   ├── End → End of current measure
│   ├── Ctrl/Cmd+Home → Start of score
│   ├── Ctrl/Cmd+End → End of score
│   ├── Page Up → Previous system
│   └── Page Down → Next system
│
├── Score Navigation (With Selection)
│   ├── Arrow keys extend or move selection
│   │   ├── Arrow alone → Moves selection to adjacent element
│   │   ├── Shift+Arrow → Extends selection (range select)
│   │   └── Ctrl/Cmd+Arrow → Moves focus without changing selection
│   ├── Tab → Next element (note/measure/system)
│   ├── Shift+Tab → Previous element
│   ├── Home → First element in current container
│   ├── End → Last element in current container
│   ├── Ctrl/Cmd+Home → First element in score
│   └── Ctrl/Cmd+End → Last element in score
│
├── Selection via Keyboard
│   ├── Space → Select/deselect focused element
│   ├── Shift+Arrow → Range selection (extends from current)
│   ├── Ctrl/Cmd+Space → Add focused element to selection (multi-select)
│   ├── Ctrl/Cmd+A → Select all in current scope
│   ├── Escape → Clear all selections, return to cursor
│   └── Shift+Click emulation via Shift+Space
│
├── Action Activation
│   ├── Enter/Return → Activate default action for selected element
│   ├── Context menu key (or Shift+F10) → Open context menu
│   ├── All actions accessible via keyboard shortcuts
│   ├── Shortcut discovery via menus (shortcuts shown)
│   └── Help overlay (?) shows all shortcuts
│
└── Entry and Editing
    ├── Type note letter (A-G) → Inserts/replaces note
    ├── Type duration (1, 2, 4, 8) → Sets duration
    ├── Backspace/Delete → Removes note at cursor or selection
    ├── Ctrl/Cmd+C → Copy selection
    ├── Ctrl/Cmd+X → Cut selection
    ├── Ctrl/Cmd+V → Paste at cursor or replace selection
    └── Ctrl/Cmd+Z → Undo
```

**Keyboard Navigation Principles:**

1. **Focus Always Visible**: Current focus indicated by visible outline or highlight
2. **Focus Never Lost**: Focus always on a valid element or cursor
3. **Predictable Movement**: Arrow keys move focus in logical directions
4. **Escape Hatch**: Escape key always available to cancel or return to safe state
5. **Discoverability**: Tooltips and menus show keyboard shortcuts
6. **Consistency**: Same keys perform same actions across contexts

**Platform-Specific Key Mappings:**

- **macOS**: Command (⌘) for primary shortcuts, Option (⌥) for alternatives
- **Windows**: Ctrl for primary shortcuts, Alt for alternatives
- **Linux**: Ctrl for primary shortcuts, Alt for alternatives
- **All**: Enter/Return for activation, Escape for cancel, Space for select

**Focus Indicators:**

```
Visual Focus Requirements:
├── Keyboard Focus
│   ├── 2px solid outline around focused element
│   ├── High contrast color (blue default, adapts to theme)
│   ├── Distinct from selection highlight
│   └── Always visible (not hidden behind other elements)
├── Cursor (No Selection)
│   ├── Blinking vertical line (1px wide)
│   ├── Height matches staff space
│   ├── Position indicates pitch and timing
│   └── Blinks at 1Hz rate
└── Selection Highlight
    ├── Filled background color (semi-transparent)
    ├── Distinct from focus (different color)
    ├── Multiple items → All highlighted
    └── Does not interfere with note visibility
```

**Keyboard-Only Workflow Example:**

```
Complete Score Creation Without Mouse:
1. Ctrl/Cmd+N → New score dialog opens
2. Tab → Navigate through fields
3. Type title, composer, etc.
4. Enter → Creates score
5. Tab → Focus on score canvas
6. Click on staff sets cursor (or Tab to position)
7. Type "4" → Quarter note duration
8. Type "D" → D quarter note appears
9. Type "E" → E quarter note appears
10. Type "2" → Half note duration
11. Type "F" → F half note appears
12. Shift+Left arrow → Select last note
13. Shift+Left arrow → Extend selection
14. Ctrl/Cmd+Shift+D → Apply Doubling embellishment
15. Ctrl/Cmd+S → Save score
```

**Accessibility Best Practices:**

- Test with keyboard only (no mouse)
- Test with screen reader enabled
- Verify focus order is logical
- Ensure all actions keyboard-accessible
- Provide keyboard shortcut reference
- Support platform accessibility features
- Use semantic HTML/native controls where possible

### 6.2 Screen Reader Support

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

### 6.3 Visual Accessibility

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

### 6.4 Motor Accessibility

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

## 7. Search and Navigation

### 7.1 Search Functionality

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

### 7.2 Search UI

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

### 7.3 Navigation Shortcuts

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

## 8. Responsive Design Patterns

### 8.1 Screen Size Adaptation

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

### 8.2 Orientation Changes

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

### 8.3 Input Method Adaptation

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

## 9. Error Handling and Validation UX

### 9.1 Validation Feedback

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

### 9.2 Error Message Patterns

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

### 9.3 Validation Examples

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

## 10. Undo/Redo System

### 10.1 Undo Scope

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

### 10.2 Undo UI Patterns

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

## 11. Platform-Specific UX Adaptations

### 11.1 iOS/macOS UX Patterns

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

### 11.2 Android UX Patterns

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

### 11.3 Windows UX Patterns

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

### 11.4 Linux UX Patterns

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

## 12. Onboarding and Help

### 12.1 First-Time User Experience

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

### 12.2 In-App Help

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

### 12.3 Feature Discovery

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

## 13. Settings and Preferences

### 13.1 User Preferences

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

### 13.2 Workspace Customization

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

## 14. Performance and Feedback

### 14.1 Loading States

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

### 14.2 Action Feedback

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

## 15. UX Testing and Validation

### 15.1 Usability Testing

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

### 15.2 UX Metrics

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

## 16. Summary and Best Practices

### 16.1 Core UX Principles Recap

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

### 16.2 Key UX Innovations

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

### 16.3 Implementation Priority

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

### 16.4 Platform Implementation References

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

## 17. Glossary

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

**End of User Experience and Interaction Design Specification v2.0**

This specification provides a complete, mode-free interaction model for the Scottish pipe band application. Platform-specific implementations must refer to their respective implementation documents for detailed UI component usage and platform-specific patterns while maintaining the core UX principles defined in this document.



