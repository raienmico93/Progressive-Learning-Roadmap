# Document, AbstractDocument, PlainDocument & DefaultStyledDocument — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `Document` is the **data model interface** behind every Swing text component — `JTextField`, `JTextArea`, `JTextPane`, and `JEditorPane`. It holds the actual text content, exposes editing operations (`insertString`, `remove`), tracks the structured hierarchy of characters and attributes, and notifies registered `DocumentListener`s whenever the content changes. `AbstractDocument` is the **abstract base class** that implements the listener bookkeeping, editing machinery, thread-safety, and undo support. `PlainDocument` is the concrete implementation for **unstyled text** (the default for `JTextField` and `JTextArea`), and `DefaultStyledDocument` is the concrete implementation for **styled text** (the default for `JTextPane`).

**Technical Definition.** `public interface Document`. It declares methods for content access (`getText(int offset, int length)`, `getLength()`, `getStartPosition()`, `getEndPosition()`), content modification (`insertString(int offset, String str, AttributeSet a)`, `remove(int offs, int len)`), structural queries (`getDefaultRootElement()`, `getRootElements()`, `createPosition(int offs)`, `getProperty(Object key)`, `putProperty(Object key, Object value)`), and listener registration (`addDocumentListener`, `removeDocumentListener`, `addUndoableEditListener`, `removeUndoableEditListener`). `public abstract class AbstractDocument extends Object implements Document, Serializable` provides the machinery: it manages the `Element` hierarchy, `Position` objects, `DocumentFilter` invocation, `DocumentEvent` firing, asynchronous mutation support, and undo/redo listener plumbing. `public class PlainDocument extends AbstractDocument` provides a simple, unstyled, character-content implementation. `public class DefaultStyledDocument extends AbstractDocument implements StyledDocument` provides a styled implementation with `Style`, `AttributeSet`, and paragraph structure. All text-component mutations should occur on the Event Dispatch Thread (EDT) unless using the documented thread-safe methods of `AbstractDocument`.

**Beginner-Friendly Explanation.** A `Document` is the **"text brain"** of a text component. The visible `JTextField`, `JTextArea`, `JTextPane`, or `JEditorPane` is the **face**: it paints the caret, the selection, the text, and any styling. The `Document` holds the actual characters and their attributes. Swing offers **four levels of document implementation**:

- **`PlainDocument`** — "plain, unstyled text with a simple character hierarchy." The default for `JTextField` and `JTextArea`.
- **`DefaultStyledDocument`** — "styled text with character and paragraph attributes." The default for `JTextPane`.
- **`AbstractDocument`** — "the framework". Extend it to build a custom document with specialised structure or behaviour.
- **`Document`** — "the contract". Implement it only when you need complete control.

Think of a `Document` as a **notebook** with a certain structure. The characters are the ink; the `Element` hierarchy is the chapter/section/paragraph structure of the notebook; the `Position` objects are bookmarks that track specific character offsets even as text is inserted and removed around them; the `DocumentFilter` is the censor that can reject or transform incoming text; and the `DocumentListener` is the reader who is notified when the notebook changes.

---

### Key Characteristics

1. **Separable model architecture.** The text component is the view; the `Document` is the model. Multiple text components can share the same `Document` (e.g., two views of the same file).
2. **Structured content.** Every `Document` organises its content into an `Element` tree. The root element contains the block structure (typically paragraphs), and each block contains character-level leaf elements. `PlainDocument` uses a simple two-level structure (root → lines); `DefaultStyledDocument` supports arbitrary styled element hierarchies.
3. **`Position` tracking.** `Position` objects are bookmarks that automatically track their character offset as text is inserted or removed around them. This enables stable references across edits.
4. **Event-driven notification.** `DocumentListener` is notified of every content change via `DocumentEvent` (with `INSERT`, `REMOVE`, or `CHANGE` type, offset, and length). `UndoableEditListener` is notified of changes that can be undone.
5. **`DocumentFilter` interception.** Before mutations are applied, the `DocumentFilter` can inspect and transform the operation. It is the **designated** mechanism for input validation and transformation.
6. **Thread-safety via `AbstractDocument`.** `AbstractDocument` supports asynchronous mutations: listeners are notified on the mutating thread and must be thread-safe. Standard Swing components mutate their documents on the EDT, so listeners run on the EDT under normal use.
7. **Undo/redo support.** `AbstractDocument` fires `UndoableEditEvent`s that can be captured by a `UndoManager` for undo/redo functionality.
8. **Attribute support.** `StyledDocument` adds methods for character and paragraph attributes, providing the foundation for rich-text editing in `JTextPane`.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JTextField`, `JTextArea`, `JTextPane`, or `JEditorPane`.
- Understanding of `DocumentListener`, `DocumentEvent`, and the delegation event model.
- Familiarity with `AttributeSet`, `SimpleAttributeSet`, and `StyleConstants` (for styled documents).
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `DocumentFilter` for input transformation.
- A JDK (version 8 or later; all document classes are in `javax.swing.text`).

---

### Related Programming Areas

- **`DocumentListener` and `DocumentEvent`:** The event contract for content changes.
- **`DocumentFilter`:** The interception mechanism for input validation and transformation.
- **`AbstractDocument`:** The abstract base class with the editing machinery.
- **`PlainDocument` and `DefaultStyledDocument`:** The concrete implementations.
- **`Element`, `BranchElement`, and `LeafElement`:** The structural hierarchy of the document.
- **`Position`:** Stable bookmarks that track offsets across edits.
- **`AttributeSet` and `MutableAttributeSet`:** The styling data for styled documents.
- **`Style` and `StyleContext`:** Named attribute sets used by `DefaultStyledDocument`.
- **`UndoManager`:** Captures `UndoableEditEvent`s for undo/redo support.
- **`JTextComponent` and its subclasses:** The views that display the document.
- **`SwingUtilities.invokeLater`:** Ensures mutations occur on the EDT.

---

## Core Concepts and Features

---

## 1. Document — The Interface

### Definitions

**Core Definition.** `Document` is the interface that defines the contract for text-content storage, editing, structure, and notification in Swing text components.

**Technical Definition.** `public interface Document`. It declares methods for:
- **Content access:** `getText(int offset, int length)`, `getLength()`, `getStartPosition()`, `getEndPosition()`.
- **Content modification:** `insertString(int offset, String str, AttributeSet a)`, `remove(int offs, int len)`.
- **Structure:** `getDefaultRootElement()`, `getRootElements()`, `createPosition(int offs)`.
- **Properties:** `getProperty(Object key)`, `putProperty(Object key, Object value)`.
- **Listeners:** `addDocumentListener`, `removeDocumentListener`, `addUndoableEditListener`, `removeUndoableEditListener`.

**Beginner-Friendly Explanation.** `Document` is the "text data contract" that says: "I hold text, I can be queried and edited, I have a structure of elements, I have properties, and I notify listeners when I change."

---

### Purposes

- To provide a text-content model that is independent of the visual component.
- To expose editing operations that can be intercepted by a `DocumentFilter`.
- To maintain a structured hierarchy of elements for rendering and navigation.
- To notify listeners of content changes for validation, logging, and dependent UI.
- To enable shared documents across multiple views.
- To support undo/redo through undoable edit events.
- To serve as the foundation for plain and styled text editing.

---

### Syntax Rules and Structure

#### Complete General Syntax (Accessing and Editing a Document)

```java
JTextField field = new JTextField(20);
Document doc = field.getDocument();

doc.insertString(0, "Hello", null);                                   // 1. Insert
String text = doc.getText(0, doc.getLength());                        // 2. Read
doc.remove(0, 5);                                                     // 3. Remove
int len = doc.getLength();                                            // 4. Length
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getText(int offset, int length)` | Returns the text in the given range. |
| `getLength()` | Returns the number of characters. |
| `insertString(int offset, String str, AttributeSet a)` | Inserts text at the offset. |
| `remove(int offs, int len)` | Removes characters in the range. |
| `getStartPosition()` / `getEndPosition()` | Return `Position` objects for the beginning and end. |
| `createPosition(int offs)` | Creates a `Position` bookmark at the offset. |
| `getDefaultRootElement()` | Returns the root of the element hierarchy. |
| `addDocumentListener(...)` | Registers a content-change listener. |
| `addUndoableEditListener(...)` | Registers an undo/redo listener. |

#### Syntax Rules

- `insertString` and `remove` throw `BadLocationException` if the offset or range is invalid.
- Passing `null` for `AttributeSet` inserts unstyled text.
- `getText` throws `BadLocationException` if the range is invalid.
- Mutations should occur on the EDT for live components.

#### Constraints and Limitations

- The `Document` interface does not mandate thread safety; `AbstractDocument` provides it.
- Attempting to mutate the document from within a `DocumentListener` causes infinite loops.
- `insertString` and `remove` respect the `DocumentFilter`; to bypass it, use the `FilterBypass` mechanism.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Document Operations

```java
import javax.swing.*;
import javax.swing.text.BadLocationException;
import javax.swing.text.Document;

public class BasicDocumentOps {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic Document Operations");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            Document doc = field.getDocument();                          // 1. Access

            try {
                doc.insertString(0, "Hello, ", null);                     // 2. Insert
                doc.insertString(doc.getLength(), "World!", null);
                System.out.println("Text: " + doc.getText(0, doc.getLength()));
                System.out.println("Length: " + doc.getLength());         // 3. Length
                doc.remove(0, 7);                                          // 4. Remove
                System.out.println("After remove: " + doc.getText(0, doc.getLength()));
            } catch (BadLocationException e) {
                e.printStackTrace();
            }

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.**
```
Text: Hello, World!
Length: 13
After remove: World!
```

**Why the code produces this result.** `insertString` adds text at the specified offset. `getText(0, getLength())` retrieves the full content. `remove(0, 7)` deletes the first seven characters. `BadLocationException` is caught because the operations may fail if the offsets are out of range.

---

### Real-World Cases

- **Shared documents** between multiple views (e.g., a split editor).
- **Validation and logging** via `DocumentListener`.
- **Input transformation** via `DocumentFilter`.
- **Undo/redo** via `UndoableEditListener` and `UndoManager`.
- **Styled text** via `StyledDocument` and `DefaultStyledDocument`.

---

## 2. AbstractDocument — The Framework

### Definitions

**Core Definition.** `AbstractDocument` is the abstract base class that provides the editing machinery, element hierarchy management, listener bookkeeping, and thread-safety infrastructure for all Swing text documents.

**Technical Definition.** `public abstract class AbstractDocument extends Object implements Document, Serializable`. It implements the `Document` interface and provides the protected methods and inner classes that subclasses use: `insertUpdate`, `removeUpdate`, `fireInsertUpdate`, `fireRemoveUpdate`, `fireChangedUpdate`, and `fireUndoableEditUpdate`. It manages a `DocumentFilter`, an `Element` tree (with `getDefaultRootElement`), `Position` objects (via `createPosition`), and listener lists. It supports both synchronous and asynchronous mutations: if a mutation occurs off the EDT, listeners are notified on the mutating thread. It defines nested classes `AbstractDocument.Content`, `AbstractDocument.AbstractElement`, `AbstractDocument.BranchElement`, `AbstractDocument.LeafElement`, and `AbstractDocument.DefaultDocumentEvent`.

**Beginner-Friendly Explanation.** `AbstractDocument` is the "framework" of the text model. You rarely use it directly, but you extend it (or its subclasses) when you need custom document behaviour. It handles the listener bookkeeping, element structure, and position tracking.

---

### Purposes

- To provide the common infrastructure for `PlainDocument` and `DefaultStyledDocument`.
- To reduce boilerplate for custom document implementations.
- To enable custom `Element` hierarchies, `DocumentFilter`s, and `Position` tracking.
- To support asynchronous mutations with thread-safety support.
- To provide undoable edit events for undo/redo integration.

---

### Syntax Rules and Structure

#### Complete General Syntax (Extending AbstractDocument)

```java
public class MyDocument extends AbstractDocument {
    public MyDocument(Content c) {
        super(c);
    }
    // Override insertUpdate, removeUpdate, etc. as needed.
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `insertUpdate(DefaultDocumentEvent, AttributeSet)` | Abstract method; called after an insertion. |
| `removeUpdate(DefaultDocumentEvent)` | Abstract method; called after a removal. |
| `fireInsertUpdate(DocumentEvent)` | Notifies `DocumentListener`s of an insertion. |
| `fireRemoveUpdate(DocumentEvent)` | Notifies `DocumentListener`s of a removal. |
| `fireChangedUpdate(DocumentEvent)` | Notifies `DocumentListener`s of an attribute change. |
| `fireUndoableEditUpdate(UndoableEditEvent)` | Notifies `UndoableEditListener`s. |
| `getDefaultRootElement()` | Returns the root `Element`. |
| `createPosition(int offs)` | Creates a `Position` bookmark. |
| `setDocumentFilter(DocumentFilter)` | Installs the filter. |
| `getDocumentFilter()` | Returns the current filter. |

#### Syntax Rules

- `AbstractDocument` is abstract; subclasses must implement `insertUpdate` and `removeUpdate`.
- The `Content` implementation stores the raw characters; the subclass handles the structure.
- `DocumentFilter` interception is optional; install one to validate or transform input.

#### Constraints and Limitations

- Asynchronous mutations notify listeners on the mutating thread; listeners must be thread-safe.
- Direct manipulation of the element tree requires careful bookkeeping; use the provided helper methods where possible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom Document with DocumentFilter for Uppercasing

```java
import javax.swing.*;
import javax.swing.text.*;

public class UppercaseDocument extends PlainDocument {
    public UppercaseDocument() {
        setDocumentFilter(new DocumentFilter() {                      // 1. Filter
            @Override
            public void insertString(FilterBypass fb, int offset,
                String text, AttributeSet attrs) throws BadLocationException {
                super.insertString(fb, offset, text.toUpperCase(), attrs);
            }
            @Override
            public void replace(FilterBypass fb, int offset, int length,
                String text, AttributeSet attrs) throws BadLocationException {
                super.replace(fb, offset, length, text.toUpperCase(), attrs);
            }
        });
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Uppercase Document");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            JTextField field = new JTextField(new UppercaseDocument(), "", 20); // 2. Use
            frame.add(field);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing lowercase letters results in uppercase characters in the field.

**Why the code produces this result.** The `DocumentFilter` intercepts insertions and replacements, uppercasing the text before passing it to `super`. The `PlainDocument` subclass installs the filter in its constructor.

---

### Real-World Cases

- **Custom documents** with specialised structure (e.g., a document with embedded forms).
- **Uppercase or masked input** via `DocumentFilter`.
- **Undo/redo-enabled documents** integrating with `UndoManager`.
- **Async-loaded documents** for large text.

---

## 3. PlainDocument — The Unstyled Implementation

### Definitions

**Core Definition.** `PlainDocument` is the concrete `Document` implementation for unstyled text, providing a simple character-content hierarchy.

**Technical Definition.** `public class PlainDocument extends AbstractDocument`. It maintains a `GapContent` (the default) for storing characters and a simple two-level element hierarchy: a root branch element containing one leaf element per line. The default root element's children are the lines of text. `PlainDocument` is the default document for `JTextField` and `JTextArea`.

**Beginner-Friendly Explanation.** `PlainDocument` is the "plain-text notebook." It stores plain characters and organises them into lines, without any styling information. It is what you get by default when you create a `JTextField` or `JTextArea`.

---

### Purposes

- To provide the default document for `JTextField` and `JTextArea`.
- To manage unstyled text with a simple line-based structure.
- To serve as a base class for custom unstyled documents.
- To support `DocumentFilter`s and `DocumentListener`s for plain text.
- To enable shared plain-text documents across multiple views.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
PlainDocument doc = new PlainDocument();
JTextField field = new JTextField(doc, "", 20);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `PlainDocument()` | Creates an empty document with a `GapContent` buffer. |
| `PlainDocument(AbstractDocument.Content c)` | Creates a document with the specified content storage. |

#### Syntax Rules

- `PlainDocument` fires `insertUpdate` and `removeUpdate` for content changes; it never fires `changedUpdate` (there are no attributes).
- The default root element is a `BranchElement` whose children are line elements.
- The default content implementation is `GapContent`.

#### Constraints and Limitations

- `PlainDocument` does not support styled text; use `DefaultStyledDocument` for that.
- The line structure is determined by newline characters in the text.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Sharing a PlainDocument Between Two Text Areas

```java
import javax.swing.*;
import java.awt.*;

public class SharedPlainDocument {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared PlainDocument");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            PlainDocument doc = new PlainDocument();                    // 1. Document
            JTextArea area1 = new JTextArea(doc, "", 5, 20);            // 2. View 1
            JTextArea area2 = new JTextArea(doc, "", 5, 20);            // 3. View 2

            JPanel panel = new JPanel(new GridLayout(1, 2, 5, 5));
            panel.add(new JScrollPane(area1));
            panel.add(new JScrollPane(area2));

            frame.add(panel);
            frame.setSize(500, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two text areas showing the same document. Typing in one appears simultaneously in the other, because both share the same `PlainDocument`.

**Why the code produces this result.** Both `JTextArea` instances use the same `PlainDocument` instance. The document fires `DocumentEvent`s to both views, keeping them synchronized.

---

#### Example 2: Programmatic Insertion and Removal

```java
PlainDocument doc = new PlainDocument();
doc.insertString(0, "Line 1\nLine 2\nLine 3", null);
// doc.getLength() == 20
doc.remove(0, 7);   // removes "Line 1\n"
```

**Expected Output.** After insertion, the document contains three lines. After removal, it contains only “Line 2\nLine 3”.

---

### Real-World Cases

- **Plain text editors** (notepad-style).
- **Single-line inputs** with custom filters.
- **Shared document views** for side-by-side comparison.

---

## 4. DefaultStyledDocument — The Styled Implementation

### Definitions

**Core Definition.** `DefaultStyledDocument` is the concrete `StyledDocument` implementation for styled text, providing character and paragraph attributes, style support, and a structured element hierarchy.

**Technical Definition.** `public class DefaultStyledDocument extends AbstractDocument implements StyledDocument`. It maintains a `GapContent` and a styled element hierarchy with `StyleContext` support. It provides methods for character attributes (`setCharacterAttributes`, `getCharacterElement`), paragraph attributes (`setParagraphAttributes`, `getParagraphElement`), style management (`addStyle`, `removeStyle`, `getStyle`, `getStyleNames`), font and colour helpers (`setFont`, `setForeground`), and logical style assignment. It is the default document for `JTextPane`.

**Beginner-Friendly Explanation.** `DefaultStyledDocument` is the "rich-text notebook." It stores characters, styled text with fonts, colours, and paragraph alignment, and it supports named styles (e.g., “Heading”, “Body”). It is what you get by default when you create a `JTextPane`.

---

### Purposes

- To provide the default document for `JTextPane`.
- To manage styled text with character and paragraph attributes.
- To support named styles and logical style assignment.
- To enable rich-text editing (fonts, colours, alignment, etc.).
- To serve as the base class for custom styled documents.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
DefaultStyledDocument doc = new DefaultStyledDocument();
JTextPane pane = new JTextPane(doc);

// Apply character attributes
SimpleAttributeSet attrs = new SimpleAttributeSet();
StyleConstants.setBold(attrs, true);
doc.setCharacterAttributes(0, 5, attrs, false);

// Apply paragraph attributes
SimpleAttributeSet para = new SimpleAttributeSet();
StyleConstants.setAlignment(para, StyleConstants.ALIGN_CENTER);
doc.setParagraphAttributes(0, 5, para, false);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setCharacterAttributes(int offset, int length, AttributeSet s, boolean replace)` | Applies character-level attributes. |
| `setParagraphAttributes(int offset, int length, AttributeSet s, boolean replace)` | Applies paragraph-level attributes. |
| `getCharacterElement(int pos)` | Returns the character element at the position. |
| `getParagraphElement(int pos)` | Returns the paragraph element at the position. |
| `addStyle(String nm, Style parent)` | Adds a named style. |
| `removeStyle(String nm)` | Removes a named style. |
| `getStyle(String nm)` | Returns a named style. |
| `getStyleNames()` | Returns all style names. |
| `setLogicalStyle(int pos, Style s)` | Assigns a logical style to a paragraph. |

#### Syntax Rules

- Character attributes apply to individual characters; paragraph attributes apply to entire paragraphs.
- The `replace` flag controls whether to replace existing attributes or merge.
- Named styles are stored in the document's `StyleContext`.
- The `changedUpdate` event is fired for style changes but not for content changes.

#### Constraints and Limitations

- `DefaultStyledDocument` uses a `StyleContext` that may be shared across documents.
- Attribute changes fire `changedUpdate` events; content changes fire `insertUpdate` and `removeUpdate`.
- Direct manipulation of the element hierarchy requires careful bookkeeping.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Applying Bold and Alignment Programmatically

```java
import javax.swing.*;
import javax.swing.text.*;
import java.awt.*;

public class StyledDocumentDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("StyledDocument Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();
            DefaultStyledDocument doc = (DefaultStyledDocument) pane.getDocument();

            try {
                doc.insertString(0, "Bold and centered\n", null);        // 1. Insert
            } catch (BadLocationException e) { e.printStackTrace(); }

            SimpleAttributeSet charAttrs = new SimpleAttributeSet();       // 2. Char
            StyleConstants.setBold(charAttrs, true);
            StyleConstants.setForeground(charAttrs, Color.BLUE);
            doc.setCharacterAttributes(0, 4, charAttrs, false);

            SimpleAttributeSet paraAttrs = new SimpleAttributeSet();       // 3. Para
            StyleConstants.setAlignment(paraAttrs, StyleConstants.ALIGN_CENTER);
            doc.setParagraphAttributes(0, doc.getLength(), paraAttrs, false);

            frame.add(new JScrollPane(pane));
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The first four characters (“Bold”) appear bold and blue; the paragraph is centered.

**Why the code produces this result.** `setCharacterAttributes` applies character-level styling to the first four characters. `setParagraphAttributes` applies paragraph-level alignment. Both fire `changedUpdate` events to `DocumentListener`s.

---

#### Example 2: Using Named Styles

```java
DefaultStyledDocument doc = new DefaultStyledDocument();
Style heading = doc.addStyle("Heading", null);
StyleConstants.setBold(heading, true);
StyleConstants.setFontSize(heading, 18);
StyleConstants.setForeground(heading, Color.DARK_GRAY);

doc.insertString(0, "Title\n", null);
doc.setLogicalStyle(0, heading);
```

**Expected Output.** The “Title” paragraph uses the bold, 18-point, dark-gray “Heading” style.

---

### Real-World Cases

- **Rich-text editors** with formatting toolbars.
- **HTML editors** using `JEditorPane` with `HTMLEditorKit`.
- **Document viewers** displaying styled content from a database.
- **Note-taking applications** with headings, bold, italic, and colours.

---

## 5. Document Methods — Complete Reference

### Definitions

**Core Definition.** The `Document` methods provide content access, modification, structure querying, property management, and listener registration.

**Technical Definition.** The table below summarises the key methods.

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `insertString(int offset, String str, AttributeSet a)` | `void` | Inserts text at the offset. |
| `remove(int offs, int len)` | `void` | Removes characters in the range. |
| `getText(int offset, int length)` | `String` | Returns the text in the range. |
| `getLength()` | `int` | Returns the number of characters. |
| `getStartPosition()` | `Position` | Returns the start position. |
| `getEndPosition()` | `Position` | Returns the end position. |
| `createPosition(int offs)` | `Position` | Creates a position bookmark. |
| `getDefaultRootElement()` | `Element` | Returns the root element. |
| `getRootElements()` | `Element[]` | Returns the root elements. |
| `getProperty(Object key)` | `Object` | Returns the property value. |
| `putProperty(Object key, Object value)` | `void` | Stores a property. |
| `addDocumentListener(...)` | `void` | Registers a listener. |
| `removeDocumentListener(...)` | `void` | Unregisters a listener. |
| `addUndoableEditListener(...)` | `void` | Registers an undo listener. |
| `removeUndoableEditListener(...)` | `void` | Unregisters an undo listener. |

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `insertString` | The fundamental insertion operation; respects the `DocumentFilter`. |
| `remove` | The fundamental removal operation; respects the `DocumentFilter`. |
| `getText` | Reads a substring. |
| `getLength` | Returns the total character count. |
| `createPosition` | Creates a bookmark that tracks its offset. |
| `getDefaultRootElement` | Returns the root of the element hierarchy. |

#### Syntax Rules

- `insertString` and `remove` throw `BadLocationException` for invalid offsets.
- `getText` throws `BadLocationException` for invalid ranges.
- `createPosition` throws `BadLocationException` for invalid offsets.
- Mutations should occur on the EDT for live components.

#### Constraints and Limitations

- `insertString` and `remove` cannot be used to mutate the document from within a `DocumentListener` without causing infinite loops; use `DocumentFilter` for transformation.
- Position objects remain valid across edits but may become invalid if the entire document is replaced.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using Position Bookmarks

```java
import javax.swing.*;
import javax.swing.text.*;

public class PositionDemo {
    public static void main(String[] args) throws Exception {
        PlainDocument doc = new PlainDocument();
        doc.insertString(0, "Hello World", null);
        Position pos = doc.createPosition(5);                             // 1. Bookmark
        System.out.println("Offset before: " + pos.getOffset());          // 5

        doc.insertString(0, "XX", null);                                  // 2. Insert
        System.out.println("Offset after: " + pos.getOffset());           // 7 (shifted)
    }
}
```

**Expected Output.**
```
Offset before: 5
Offset after: 7
```

**Why the code produces this result.** `createPosition(5)` creates a bookmark at offset 5. When two characters are inserted at the beginning, the bookmark automatically shifts to offset 7, preserving its reference to the same text location.

---

#### Example 2: Reading and Writing Properties

```java
doc.putProperty("filename", "notes.txt");
String filename = (String) doc.getProperty("filename");
System.out.println("File: " + filename);
```

**Expected Output.** `File: notes.txt`.

**Why the code produces this result.** The `Document` interface provides a general-purpose property map. Swing uses it for standard properties (e.g., `Document.TitleProperty`, `Document.StreamDescriptionProperty`).

---

### Real-World Cases

- **Bookmarks** for caret tracking across edits.
- **Document properties** for file associations.
- **Substring access** for search and replace.
- **Length queries** for character counters.

---

## 6. Document Best Practices

### Definitions

**Core Definition.** Document best practices are the conventions and patterns that ensure text handling is correct, thread-safe, and maintainable.

**Technical Definition.** Best practices include: using the `Document` API rather than the component's `setText` for programmatic edits; using `DocumentFilter` for input transformation (not `DocumentListener`); mutating the document on the EDT for live components; using `PlainDocument` for unstyled text and `DefaultStyledDocument` for styled text; using `Position` objects for stable bookmarks; using `UndoManager` with `UndoableEditListener` for undo/redo; and respecting the `BadLocationException` contract.

**Beginner-Friendly Explanation.** Best practices for documents are the "rules of thumb" that prevent infinite loops, frozen GUIs, and unmaintainable code. They include things like "use `DocumentFilter` to transform input, not `DocumentListener`," "mutate on the EDT," and "use the right document type."

---

### Purposes

- To prevent infinite loops caused by mutating the document from within a `DocumentListener`.
- To keep the GUI responsive by respecting the EDT.
- To choose the right document implementation for the content type.
- To enable undo/redo through `UndoManager`.
- To maintain stable references across edits via `Position`.
- To handle errors gracefully through `BadLocationException`.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JTextField field = new JTextField(20);
    ((AbstractDocument) field.getDocument()).setDocumentFilter(
        new DocumentFilter() {                                         // 1. Filter
            @Override
            public void insertString(FilterBypass fb, int offs,
                String str, AttributeSet a) throws BadLocationException {
                super.insertString(fb, offs, transform(str), a);
            }
        });

    field.getDocument().addDocumentListener(new DocumentListener() {   // 2. Listen
        public void insertUpdate(DocumentEvent e) { SwingUtilities.invokeLater(this::validate); }
        public void removeUpdate(DocumentEvent e) { SwingUtilities.invokeLater(this::validate); }
        public void changedUpdate(DocumentEvent e) {}
        private void validate() { /* live validation */ }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `DocumentFilter` for input transformation | Install via `setDocumentFilter` |
| Use `DocumentListener` for observation | Never mutate in the listener |
| Mutate on the EDT | `SwingUtilities.invokeLater(...)` |
| Use `PlainDocument` for plain text | Default for `JTextField`/`JTextArea` |
| Use `DefaultStyledDocument` for styled text | Default for `JTextPane` |
| Use `Position` for bookmarks | `createPosition(offset)` |
| Use `UndoManager` for undo/redo | `addUndoableEditListener(undoManager)` |
| Catch `BadLocationException` | Wrap insert/remove/getText in try-catch |

#### Syntax Rules

- **Never mutate the document from within a `DocumentListener`**; use `DocumentFilter` for transformation or `SwingUtilities.invokeLater` for deferred mutation.
- **Always mutate on the EDT** for live components.
- **Always use `DocumentFilter`** for input validation and transformation.
- **Always catch `BadLocationException`** when calling `insertString`, `remove`, `getText`, or `createPosition`.
- **Always use the right document type**: `PlainDocument` for unstyled text, `DefaultStyledDocument` for styled text.

#### Constraints and Limitations

- `DocumentListener` cannot safely mutate the document.
- `DocumentFilter` cannot be used for post-hoc validation (it intercepts before mutation).
- `AbstractDocument` supports off-EDT mutation but notifies listeners on the mutating thread, requiring thread-safe listeners.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Live Validation with DocumentFilter

```java
import javax.swing.*;
import javax.swing.text.*;
import java.awt.*;

public class BestPracticeDocument {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeDocument::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Document");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JTextField email = new JTextField(20);
        JLabel status = new JLabel(" ");

        ((AbstractDocument) email.getDocument()).setDocumentFilter(       // 1. Filter
            new DocumentFilter() {
                @Override
                public void insertString(FilterBypass fb, int offset,
                    String text, AttributeSet attrs) throws BadLocationException {
                    super.insertString(fb, offset, text.replace(" ", ""), attrs);
                }
            });

        email.getDocument().addDocumentListener(new DocumentListener() {  // 2. Listen
            public void insertUpdate(DocumentEvent e) { update(); }
            public void removeUpdate(DocumentEvent e) { update(); }
            public void changedUpdate(DocumentEvent e) {}
            private void update() {
                SwingUtilities.invokeLater(() -> {                       // 3. Defer
                    String text = email.getText();
                    status.setText(text.contains("@") ? "Looks valid" : "Missing @");
                });
            }
        });

        JPanel panel = new JPanel(new BorderLayout(5, 5));
        panel.add(new JLabel("Email:"), BorderLayout.NORTH);
        panel.add(email, BorderLayout.CENTER);
        panel.add(status, BorderLayout.SOUTH);

        frame.add(panel);
        frame.setSize(400, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** As the user types, spaces are stripped by the filter, and the status label updates to “Looks valid” or “Missing @.”

**Why the code produces this result.** The `DocumentFilter` transforms input before it reaches the document (removing spaces). The `DocumentListener` observes changes and defers the validation with `SwingUtilities.invokeLater` to avoid modifying the document during the listener.

---

#### Example 2: Common Mistake — Mutating from a DocumentListener

```java
import javax.swing.*;
import javax.swing.event.*;

public class BadDocumentMutation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Document Mutation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.getDocument().addDocumentListener(new DocumentListener() {
                public void insertUpdate(DocumentEvent e) {
                    try {
                        e.getDocument().insertString(                       // WRONG
                            e.getDocument().getLength(), "!", null);
                    } catch (Exception ex) { ex.printStackTrace(); }
                }
                public void removeUpdate(DocumentEvent e) {}
                public void changedUpdate(DocumentEvent e) {}
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing any character triggers an infinite loop: the listener inserts “!”, which fires another `insertUpdate`, which inserts another “!”, recursively, until the application freezes or throws `StackOverflowError`.

**Why this is wrong.** As the Java API documentation states: "implementations of the `DocumentListener` may not mutate the source of the event (i.e. the associated `Document`)". The correct approach is `DocumentFilter` for transformation or `SwingUtilities.invokeLater` for deferred mutation.

---

#### Example 3: Best-Practice Undo/Redo with UndoManager

```java
import javax.swing.*;
import javax.swing.undo.UndoManager;
import java.awt.*;

public class UndoRedoDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Undo/Redo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextArea area = new JTextArea(10, 30);
            UndoManager undo = new UndoManager();
            area.getDocument().addUndoableEditListener(undo);             // 1. Undo

            JButton undoBtn = new JButton("Undo");
            undoBtn.addActionListener(e -> { if (undo.canUndo()) undo.undo(); });

            JButton redoBtn = new JButton("Redo");
            redoBtn.addActionListener(e -> { if (undo.canRedo()) undo.redo(); });

            JPanel controls = new JPanel();
            controls.add(undoBtn);
            controls.add(redoBtn);

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(area), BorderLayout.CENTER);
            frame.add(controls, BorderLayout.SOUTH);
            frame.setSize(450, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing, clicking “Undo” reverses the last edit; clicking “Redo” reapplies it.

**Why the code produces this result.** `UndoManager` listens to `UndoableEditEvent`s from the document, records them, and provides `undo()` and `redo()` operations.

---

### Real-World Cases

- **Production Swing applications** use `DocumentFilter` for input transformation, `DocumentListener` for validation, and `UndoManager` for undo/redo.
- **Rich-text editors** use `DefaultStyledDocument` with `StyleConstants` and `SimpleAttributeSet`.
- **Shared documents** between multiple views use the same `Document` instance.
- **Educational examples** use the best-practice skeleton to demonstrate correct document usage from the start.

---

## References

Oracle. *Interface Document*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/Document.html

Oracle. *Class AbstractDocument*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/AbstractDocument.html

Oracle. *Class PlainDocument*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/PlainDocument.html

Oracle. *Class DefaultStyledDocument*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/DefaultStyledDocument.html

Oracle. *How to Use Text Fields*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/textfield.html

Oracle. *How to Use Text Areas*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/textarea.html

Oracle. *Text Component Features*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/generaltext.html

Oracle. *Implementing a Document Filter*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/generaltext.html#filter

Oracle. *How to Write a Document Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/documentlistener.html

Oracle. *Interface StyledDocument*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/StyledDocument.html

Oracle. *Class StyleConstants*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/StyleConstants.html

Oracle. *Class SimpleAttributeSet*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/SimpleAttributeSet.html

Oracle. *Class UndoManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/undo/UndoManager.html

Oracle. *Interface Position*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/Position.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Stack Overflow. *DocumentListener should not be used to edit the document*. https://stackoverflow.com/questions/ (confirmation of the no-mutation rule)

Stack Overflow. *Autocomplete Text box with Document Listener in Java*. https://stackoverflow.com/ (DocumentFilter recommendation)

Stack Overflow. *JTextArea shared Document example*. https://stackoverflow.com/