**Short Description:** A lightweight Java Swing desktop text editor with font customization, real-time statistics, and file I/O.

# Desktop Text Editor Java
----------------------
> A desktop text editor built in Java, exploring Swing's event-driven programming model, the EDT threading contract, and Java 9's Platform Module System.

## About The Project

This project is a functional desktop text editor implemented entirely with Java Swing and AWT — zero external dependencies beyond the JDK. It was built to move beyond console-based Java and engage directly with event-driven GUI architecture: how user actions propagate through the AWT event queue, how the EDT serializes all UI mutations, and how JTextArea's PlainDocument stores text internally using a **gap buffer** — the same data structure used by GNU Emacs.

The application demonstrates several core concerns of desktop software: file I/O with safe resource management, real-time document statistics via a CaretListener, dynamic font enumeration from the local GraphicsEnvironment, and Java 9 JPMS module declarations for proper platform encapsulation.

It is not presented as production software. It is presented as a focused engineering exploration with documented limitations and a clear understanding of what a production-grade version would require.

## Live Demo

This is a desktop application — there is no hosted deployment. To run it locally, see the [Installation](#installation) section.

## Project Type

Desktop GUI Application · Developer Tool · Java Swing Prototype

## Project Status

**Experimental Prototype** — functional with documented known issues. Not intended for production use. Active exploration of Swing internals and Java platform features.

## Why I Built This

The motivation was deliberate: most Java learning stays at the console level. This project was built specifically to engage with:

*   **Swing's event threading model** — understanding why all UI mutation must occur on the EDT and what goes wrong when it doesn't
    
*   **Observer pattern in practice** — ActionListener, CaretListener, and ChangeListener as real-world implementations of publish/subscribe
    
*   **Java I/O with resource safety** — using try-with-resources correctly with Scanner and PrintWriter
    
*   **JPMS (Project Jigsaw)** — declaring a named module and understanding why requires java.desktop is necessary post-Java 9
    

The secondary goal was to build something with enough complexity to generate non-trivial engineering discussions — architecture decisions, identifiable tradeoffs, concrete improvement paths.

## Features

### Core Features

*   Open and display .txt files from the local filesystem
    
*   Save editor content to any file path
    
*   Real-time word count and line count in the status bar
    
*   Word-wrap enabled for readability on long lines
    

### Font Customization

*   Select any font family from the system's installed fonts via GraphicsEnvironment
    
*   Adjust font size via a JSpinner control with live preview
    
*   Choose text color using JColorChooser with a full color picker dialog
    

### Engineering Features

*   Java 9 JPMS module declaration (module-info.java) with explicit requires java.desktop
    
*   Try-with-resources for all file stream management — guaranteed close on exception
    
*   CaretListener for real-time document statistics without polling
    
*   FileNameExtensionFilter limiting the open dialog to .txt files
    

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| GUI Framework | Java Swing | Ships with JDK; zero external dependencies |
| Layout | Java AWT (`FlowLayout`) | Built-in; adequate for this scope |
| Text Engine | `JTextArea` + `PlainDocument` | Gap-buffer backed editing |
| File I/O | `java.io` (`Scanner`, `PrintWriter`) | Standard library support |
| Module System | JPMS (`module-info.java`) | Java 9+ encapsulation |
| Build | Manual `javac` | Prototype simplicity |

## Architecture

This is a single-process desktop application. There is no network layer.  
The complete lifecycle of a user action:

```text
User Action (click / keystroke)
         │
         ▼
AWT Native Event Queue  (OS-level input capture)
         │
         ▼
Event Dispatch Thread   (Swing's single UI thread)
         │
         ├─ ActionListener.actionPerformed()   → File I/O, color, font
         ├─ CaretListener.caretUpdate()        → updateStatus()
         └─ ChangeListener.stateChanged()      → Font size update
                              │
                              ▼
                    PlainDocument (gap buffer)
                              │
                              ▼
                    Swing repaint cycle → screen
```

---

## Component Hierarchy

```text
JFrame (Texteditor)
├── JMenuBar
│   └── JMenu ("File")
│       ├── JMenuItem ("Open")
│       ├── JMenuItem ("Save")
│       └── JMenuItem ("Exit")
├── JLabel       (status bar — word/line count)
├── JLabel       ("Font:")
├── JSpinner     (font size)
├── JButton      (color picker)
├── JComboBox    (font family)
└── JScrollPane
    └── JTextArea  ← PlainDocument (gap buffer)
```

---

## On the Gap Buffer

`JTextArea` stores text via `PlainDocument`, which uses a **gap buffer**: two contiguous arrays with a gap positioned at the current edit point.

- Insert and delete at the gap are **O(1) amortized**
- Moving the gap costs **O(distance)**

This is the same structure used by Emacs and many production editors — and it's why typing performance remains fast regardless of document length.

---

## Deployment Architecture

| Concern | Current State | Production Approach |
|---|---|---|
| Packaging | Raw `.class` files | Fat JAR via Maven/Gradle |
| Distribution | Manual `javac + java` | `jpackage` (Java 14+) for native installer |
| Platform | Theoretically cross-platform | Tested on Windows; untested on macOS/Linux |
| JRE Requirement | Requires installed JDK | GraalVM native-image would eliminate this |
| CI/CD | None | GitHub Actions → build → artifact upload |

---

## Folder Structure

```text
java-swing-text-editor/
├── src/
│   ├── editor/
│   │   ├── Main.java
│   │   │   # Entry point — instantiates Texteditor
│   │   └── Texteditor.java
│   │       # Core application: UI, event handling, file I/O
│   └── module-info.java
│       # JPMS module declaration: requires java.desktop
├── texteditor.png
│   # Application icon (working-directory relative)
└── README.md
```

## Installation

### Requirements

- JDK 11 or later
- Recommended: JDK 17

### Clone the Repository

```bash
git clone https://github.com/Heramb1221/java-swing-text-editor.git
cd java-swing-text-editor
```

### Compile with Module System

```bash
javac -d out --module-source-path src -m TextEditor
```

### Run the Application

```bash
java --module-path out -m TextEditor/editor.Main
```

> **Note:** The application icon (`texteditor.png`) must be present in the working directory from which you run the application. See [Known Issues](#known-issues) for details.

---

## Usage

| Action | How |
|---|---|
| Open a file | File → Open — filtered to `.txt` files |
| Save | File → Save — saves current content to chosen path |
| Change font family | Select from the dropdown (all system fonts enumerated) |
| Change font size | Adjust the spinner — preview updates immediately |
| Change text color | Click **Color** — pick from the color chooser dialog |
| Word / line count | Displayed live in the status bar at the top |
## Screenshots

<img width="607" height="615" alt="Screenshot 2026-05-15 215539" src="https://github.com/user-attachments/assets/e25ce4c5-8b93-4d99-a339-2419785b831b" />

<img width="627" height="617" alt="Screenshot 2026-05-15 215637" src="https://github.com/user-attachments/assets/7b606228-c6b6-4154-9266-f372033d599f" />

<img width="783" height="620" alt="Screenshot 2026-05-15 215710" src="https://github.com/user-attachments/assets/f423b9c3-7a2a-4255-aa8c-9c5d69dfa135" />


## Performance Considerations

### What is efficient

*   JTextArea's gap buffer makes character-level editing near the cursor O(1) amortized
    
*   textArea.getLineCount() delegates to the document model directly — O(1)
    
*   Try-with-resources ensures streams are closed immediately, no handle leaks
    

## Current Bottlenecks

### Word Count on Every Keystroke

```java
// Fires on every caret movement
textArea.getText();         // O(n) — full document copy
text.split("\\s+").length;  // O(n) — regex NFA evaluation + array allocation
```

For a 100,000-word document, this creates significant GC pressure.

### Recommended Fix

Use:

- Debouncing via `javax.swing.Timer`
- Cached compiled `Pattern`

```java
private static final Pattern WHITESPACE =
    Pattern.compile("\\s+");

// Scheduled via Timer, not on every keystroke
```

---

### File I/O on the EDT

Open and save operations currently run on the **Event Dispatch Thread (EDT)**.

Problem:

- Large files freeze the UI
- Slow disks block repainting and interaction

### Production Fix

Use:

```java
SwingWorker<String, Void>
```

for background file reading with progress indication.

---

## Security Considerations

| Issue | Severity | Status |
|---|---|---|
| `JColorChooser.showDialog()` returns `null` on cancel → `NullPointerException` | Medium | Open — see Known Issues |
| File save silently overwrites existing files without confirmation | Medium | Open |
| Font size spinner accepts values outside `[1, 200]` — size `0` produces invisible text | Low | Open |
| Icon loaded via relative path `"texteditor.png"` — fails silently if CWD differs | Low | Open |

This is a local single-user desktop application.

There is:

- No network layer
- No authentication
- No multi-user attack surface

---

## Tradeoffs & Limitations

## FlowLayout for the Main Window

`FlowLayout` arranges components left-to-right with wrapping.

It was chosen for simplicity.

### Consequence

Resizing the window to narrow widths causes components to reflow unpredictably.

### Production Pattern

```text
BorderLayout
├── NORTH → Toolbar JPanel
└── CENTER → JScrollPane
```

---

## God Class Architecture

`Texteditor.java` contains:

- UI construction
- Event dispatch
- File I/O

This violates the **Single Responsibility Principle (SRP)**.

It was acceptable for the project's scale but would not survive a production code review.

### Proper Refactoring

```text
EditorView        → UI only
EditorController  → Event → model coordination
FileService       → File I/O
```

---

## No Undo/Redo

The absence of undo/redo is the largest feature gap.

Swing already provides:

```java
UndoManager
UndoableEditListener
```

Implementation complexity is relatively small (~15 lines).

---

## Scanner vs BufferedReader

`Scanner` is currently used for reading files.

### Problem

`Scanner` performs regex-based tokenization, which is slower.

### Better Alternative

```java
BufferedReader.readLine()
```

This is more efficient for sequential line reads.

The difference is negligible for small files but meaningful for large ones.

---

## Known Issues

| # | Issue | Reproducible? | Fix |
|---|---|---|---|
| 1 | Cancelling the color picker dialog crashes the application (`NullPointerException`) | Yes — click **Color**, then **Cancel** | Add `if (color != null)` guard |
| 2 | Empty document displays `"Words: 1"` instead of `"Words: 0"` | Yes | `text.trim().isEmpty() ? 0 : split(...).length` |
| 3 | UI constructed on main thread, not EDT (`SwingUtilities.invokeLater` missing) | Intermittent | Wrap constructor call in `SwingUtilities.invokeLater()` |
| 4 | Application icon fails silently if launched from a different working directory | Yes | Use `getClass().getResource("/texteditor.png")` |
| 5 | Font spinner accepts size `0`, making text invisible | Yes | Use `SpinnerNumberModel(20, 1, 200, 1)` |
## Technical Debt

*   **No build system** — Maven or Gradle needed for reproducible builds, dependency management, and JAR packaging
    
*   **No tests** — the updateStatus() word-count logic has an identifiable edge case bug that a unit test would have caught
    
*   **Monolithic event handler** — actionPerformed() uses a sequential if-chain; grows linearly with every new feature; should be a dispatch table or Command Pattern
    
*   **No error UX** — all exceptions are swallowed with e.printStackTrace(); users receive no feedback on failure
    

## Scalability Discussion

This application scales with document size, not concurrent users. Current practical limits:

Document SizeBehavior< 1 MBFully functional1–10 MBWord count begins lagging; noticeable GC pauses> 10 MBFile I/O freezes UI; word count becomes unusable> 100 MBtextArea.getText() may throw OutOfMemoryError

A production text editor at scale (VS Code, Sublime) uses a **piece table** or **rope** data structure instead of a gap buffer, streams file I/O, and computes word count incrementally via DocumentListener events rather than full re-scans.

## Challenges Faced

**The EDT threading model** was the most non-obvious constraint. Java's documentation is clear in principle — _"all Swing components must be created and accessed on the EDT"_ — but the failure mode is intermittent rather than immediate, which makes the violation easy to miss and hard to reproduce.

**Font enumeration latency:** GraphicsEnvironment.getLocalGraphicsEnvironment().getAvailableFontFamilyNames() blocks during JComboBox population and can return 300–500 entries depending on the system. On some machines this added noticeable startup latency — a case for lazy loading.

**split() edge case on empty strings:** The standard pattern text.split("\\\\s+").length returns 1 on an empty string, producing an incorrect word count. This required understanding how Java's split handles empty input differently from most developers' mental model.

## What I Learned

*   **EDT is non-negotiable.** Swing has one rule: touch components only on the EDT. Violating it produces race conditions with intermittent, hard-to-reproduce symptoms — identical to general multithreading bugs. SwingUtilities.invokeLater() is not optional.
    
*   **Null safety is a discipline, not an afterthought.** JColorChooser.showDialog() cancellation returning null is documented behavior. The null crash is a failure of defensive programming, not a framework bug.
    
*   **The Observer pattern is everywhere.** ActionListener, CaretListener, ChangeListener — these are all implementations of the same publish/subscribe contract. Recognizing patterns in existing APIs is as important as implementing them from scratch.
    
*   **Gap buffers explain editor performance.** Understanding that JTextArea uses a gap buffer — and what that means for cursor-proximate vs. random-access operations — connects a GUI widget to a fundamental data structure.
    
*   **A God Class is fast to write and slow to maintain.** The decision to put everything in one class was never a decision — it happened by default. Recognizing that as a structural choice, understanding its consequences, and knowing what to do differently is the actual learning.
    

## Future Scope

| Priority | Improvement | Notes |
|---|---|---|
| High | Fix null crash on color picker cancel | One-line fix |
| High | Correct EDT threading with `SwingUtilities.invokeLater()` | One-line fix |
| High | Fix empty-document word count bug | One-line fix |
| Medium | Add `SwingWorker` for file I/O | Prevents UI freeze on large files |
| Medium | Implement undo/redo via `UndoManager` | ~15 lines using Swing's built-in support |
| Medium | Replace `FlowLayout` with `BorderLayout` + toolbar panel | Proper resize behavior |
| Medium | Add Maven/Gradle build | Reproducible builds, JAR packaging |
| Low | Refactor into MVC: `EditorView`, `EditorController`, `FileService` | Separation of concerns |
| Low | Debounce word count with `javax.swing.Timer` | Performance at large document sizes |
| Low | Package with `jpackage` for native installer | Distribution |
| Future | Syntax highlighting via `RSyntaxTextArea` | Would require build system |
| Future | Collaborative editing over WebSocket + OT | Architectural redesign |
    

## Repository Philosophy

This repository is **learning-oriented with production awareness** — built to be functional, then analyzed for its gaps. The goal was not to ship a product but to understand a class of problems: event-driven UI, threading contracts, data structures in standard libraries, and the architectural consequences of early structural decisions.

The known issues and technical debt are documented not because they couldn't be fixed, but because naming them accurately is part of the engineering discipline.

## Contributing

This is a personal learning project, but suggestions and issues are welcome.

1.  Fork the repository
    
2.  Create a branch: git checkout -b improvement/your-feature
    
3.  Commit with context: git commit -m "Fix: null guard for JColorChooser cancel"
    
4.  Push and open a pull request with a description of what and why
    

## License

Distributed under the MIT License. See LICENSE for details.

## Contact

### Connect With Me

[![GitHub](https://img.shields.io/badge/GitHub-Heramb1221-black?style=for-the-badge&logo=github)](https://github.com/Heramb1221)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Heramb%20Chaudhari-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/heramb-chaudhari)

[![Email](https://img.shields.io/badge/Email-hchaudhari1221%40gmail.com-red?style=for-the-badge&logo=gmail)](mailto:hchaudhari1221@gmail.com)

---

### Project Link

https://github.com/Heramb1221/java-swing-text-editor
