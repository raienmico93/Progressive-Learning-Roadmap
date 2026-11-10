# DocumentListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `DocumentListener` is an interface in the `javax.swing.event` package for receiving **document events**—notifications that the **content** of a text document has changed. It declares three methods: `insertUpdate`, `removeUpdate`, and `changedUpdate`. It is the standard listener for observing modifications to the `Document` model behind `JTextField`, `JTextArea`, `JTextPane`, and `JEditorPane`.

**Technical Definition.** `public interface DocumentListener extends EventListener`. The interface declares three methods: `void insertUpdate(DocumentEvent e)`, `void removeUpdate(DocumentEvent e)`, and `void changedUpdate(DocumentEvent e)`. A class interested in document events implements this interface (and all three methods) or, since there is **no adapter class**, must provide empty implementations for the methods it does not need. The listener is registered on the text component's `Document`—not on the component itself—using `textComponent.getDocument().addDocumentListener(DocumentListener)`. **Critical constraint:** "implementations of the `DocumentListener` may not mutate the source of the event (i.e. the associated `Document`)". Attempting to modify the document from within a document listener requires `SwingUtilities.invokeLater` and can produce an **infinite loop**; use `DocumentFilter` instead for mutation. All document events are dispatched on the Event Dispatch Thread (EDT) under normal Swing threading; if the document is mutated from another thread (as `AbstractDocument` supports), listeners are notified on the mutating thread and must be **thread-safe**.

**Beginner-Friendly Explanation.** `DocumentListener` is the "text just changed" listener. When the user types a character, presses backspace, pastes text, or cuts a selection, the text component's underlying `Document` model fires a `DocumentEvent` to every registered `DocumentListener`. The listener's methods are called, and you can read the new document content (or the event's location and length) to decide what to do. Unlike `KeyListener`, which fires on raw key presses and misses pasted text, `DocumentListener` fires for **every content change**, regardless of how it occurred—typing, pasting, cutting, or programmatic `setText`. It is the correct tool for live validation, character counting, search-as-you-type, and any logic that must react to the document's actual content. Think of `DocumentListener` as a content-change feed for a text model: it reports every insertion and removal, and you decide which changes deserve a reaction.

---

### Key Characteristics

1. **Three-method interface, no adapter.** `insertUpdate`, `removeUpdate`, and `changedUpdate` must all be implemented; the class must provide empty bodies for unused methods because there is no `DocumentAdapter`.
2. **Registered on the Document, not the component.** You must call `textComponent.getDocument().addDocumentListener(...)`; registering on the component itself has no effect.
3. **Fires for all content changes.** Typing, backspace, cut, paste, and programmatic `setText` all fire document events. Paste fires **one** event for the entire pasted block, not one per character.
4. **Cannot mutate the document.** The listener is notified **after** the change has already occurred; modifying the document from within the listener risks an infinite loop and is explicitly prohibited.
5. **`changedUpdate` is style-only.** It fires only for `StyledDocument` implementations (e.g., `JTextPane`, `JEditorPane`), not for `PlainDocument` (e.g., `JTextField`, `JTextArea`).
6. **Threading caveat.** If the document is mutated off the EDT, listeners are notified on the mutating thread and must be thread-safe.
7. **Event ordering is not guaranteed.** There is no guarantee about the order of delivery to listeners, and all listeners must be notified before further document mutations.
8. **`DocumentEvent` provides context.** The event provides `getDocument()`, `getOffset()`, `getLength()`, and `getType()` to describe what changed.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JTextField`, `JTextArea`, and `JTextComponent`.
- Understanding of the `Document` model (`PlainDocument`, `StyledDocument`, `AbstractDocument`).
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `DocumentFilter` (the correct tool for mutating the document).
- A JDK (version 8 or later; `DocumentListener` and `DocumentEvent` are in `javax.swing.event`).

---

### Related Programming Areas

- **`Document` and `AbstractDocument`:** The model that fires the events.
- **`DocumentEvent`:** The event object carrying the document, offset, length, and event type.
- **`DocumentFilter`:** The correct mechanism for **mutating** the document in response to changes.
- **`StyledDocument`:** The document type that fires `changedUpdate` for style changes.
- **`UndoManager`:** Often used alongside document listeners for undo/redo.
- **`SwingUtilities.invokeLater`:** Used to defer UI updates that must happen after the document change settles.
- **`DocumentListener` vs. `KeyListener`:** `DocumentListener` catches paste and programmatic changes; `KeyListener` does not.

---

## Core Concepts and Features

---

## 1. DocumentListener — The Interface

### Definitions

**Core Definition.** `DocumentListener` is an interface for receiving notifications when a text document's content changes.

**Technical Definition.** `public interface DocumentListener extends EventListener`. It declares `insertUpdate`, `removeUpdate`, and `changedUpdate`. There is **no adapter class**; every implementing class must define all three methods. The listener is registered on the `Document` via `addDocumentListener`. Under standard Swing threading, events are delivered on the EDT.

**Beginner-Friendly Explanation.** `DocumentListener` is the "content changed" contract. Implement all three methods (leaving empty bodies for the ones you do not need) and register the listener on a text component's `Document`.

---

### Purposes

- To react when text is inserted into a document (typing, pasting).
- To react when text is removed from a document (backspace, cut, selection deletion).
- To react when character or paragraph attributes change (in styled documents).
- To perform live validation as the user types.
- To update character counts, previews, or dependent UI in real time.
- To implement search-as-you-type, auto-completion, and input masking.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
JTextField field = new JTextField(20);
field.getDocument().addDocumentListener(new DocumentListener() {
    public void insertUpdate(DocumentEvent e)  { /* text inserted */ }
    public void removeUpdate(DocumentEvent e)  { /* text removed */ }
    public void changedUpdate(DocumentEvent e) { /* style changed */ }
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `insertUpdate(DocumentEvent e)` | Text is inserted into the document. |
| `removeUpdate(DocumentEvent e)` | Text is removed from the document. |
| `changedUpdate(DocumentEvent e)` | Character or paragraph attributes change ( `StyledDocument` only). |

#### Syntax Rules

- **All three methods must be implemented**; there is no adapter class.
- Register on `component.getDocument()`, not on the component itself.
- Do **not** mutate the document from within the listener.
- If the document may be mutated off the EDT, the listener must be thread-safe.

#### Constraints and Limitations

- `changedUpdate` never fires for `PlainDocument` ( `JTextField`, `JTextArea` ).
- Mutating the document from within the listener causes an infinite loop or requires `invokeLater`.
- Listener invocation order is not guaranteed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full DocumentListener Implementation

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import java.awt.*;

public class FullDocumentListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Full DocumentListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            JLabel status = new JLabel("Length: 0", SwingConstants.CENTER);

            field.getDocument().addDocumentListener(new DocumentListener() { // 1. Register
                @Override
                public void insertUpdate(DocumentEvent e) {           // 2. Insert
                    updateStatus(e);
                }
                @Override
                public void removeUpdate(DocumentEvent e) {           // 3. Remove
                    updateStatus(e);
                }
                @Override
                public void changedUpdate(DocumentEvent e) {          // 4. Style
                    // PlainDocument does not fire changedUpdate
                }
                private void updateStatus(DocumentEvent e) {
                    status.setText("Length: " + e.getDocument().getLength());
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(field, BorderLayout.NORTH);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing characters updates the length label with each insertion; pressing backspace updates it with each removal. `changedUpdate` is implemented but never fires because `JTextField` uses a `PlainDocument`.

**Why the code produces this result.** The listener is registered on the field's `Document`. Each insertion or removal fires the corresponding method, and the listener reads `e.getDocument().getLength()` to update the label.

---

### Real-World Cases

- **Live form validation:** Checking email format as the user types.
- **Character counters:** Displaying remaining characters in a text area.
- **Search-as-you-type:** Filtering a list based on the current document content.
- **Auto-completion:** Suggesting matches as the user types.
- **Dependent field calculations:** Summing numeric fields in real time.

---

## 2. Document Events — insertUpdate(), removeUpdate(), changedUpdate()

### 2.1 insertUpdate()

#### Definitions

**Core Definition.** `insertUpdate(DocumentEvent e)` is invoked when text is **inserted** into the document.

**Technical Definition.** `void insertUpdate(DocumentEvent e)`. The range given by the `DocumentEvent` bounds the freshly inserted region. One event fires per insertion operation; pasting a block of text fires **one** event, not one per character.

**Beginner-Friendly Explanation.** `insertUpdate` fires when text enters the document—whether by typing, pasting, or programmatic insertion.

---

#### Purposes

- To react to characters being typed.
- To react to pasted content.
- To trigger validation or formatting on newly inserted text.
- To update dependent displays when new content appears.

---

#### Syntax Rules and Structure

```java
@Override
public void insertUpdate(DocumentEvent e) {
    int offset = e.getOffset();
    int length = e.getLength();
    // react to the insertion
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `e.getOffset()` | The starting position of the insertion. |
| `e.getLength()` | The number of characters inserted. |
| `e.getDocument()` | The document that changed. |

#### Constraints and Limitations

- Pasting fires one event for the entire block, not per character.
- Do not mutate the document from within this method.

---

#### Annotated Complete Step-by-Step Code Examples

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;

public class InsertDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("insertUpdate Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            JLabel log = new JLabel(" ", SwingConstants.CENTER);

            field.getDocument().addDocumentListener(new DocumentListener() {
                public void insertUpdate(DocumentEvent e) {           // 1. Insert
                    log.setText("Inserted " + e.getLength()
                        + " char(s) at offset " + e.getOffset());
                }
                public void removeUpdate(DocumentEvent e) {}
                public void changedUpdate(DocumentEvent e) {}
            });

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(field, java.awt.BorderLayout.NORTH);
            frame.add(log, java.awt.BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing “abc” fires three separate insert events (one per character). Pasting “hello” fires one event with length 5.

---

### 2.2 removeUpdate()

#### Definitions

**Core Definition.** `removeUpdate(DocumentEvent e)` is invoked when text is **removed** from the document.

**Technical Definition.** `void removeUpdate(DocumentEvent e)`. The range is given in terms of what the view last saw (before updating sticky positions). One event fires per removal operation; deleting a selection fires **one** event for the entire deletion.

**Beginner-Friendly Explanation.** `removeUpdate` fires when text leaves the document—backspace, cut, selection deletion, or programmatic removal.

---

#### Purposes

- To react to backspace and delete operations.
- To react to cut and selection-deletion.
- To re-validate or reformat after removal.
- To update counts and dependent UI when content shrinks.

---

#### Syntax Rules and Structure

```java
@Override
public void removeUpdate(DocumentEvent e) {
    int offset = e.getOffset();
    int length = e.getLength();
    // react to the removal
}
```

#### Constraints and Limitations

- Deleting a selection fires one event for the whole deletion.
- Do not mutate the document from within this method.

---

### 2.3 changedUpdate()

#### Definitions

**Core Definition.** `changedUpdate(DocumentEvent e)` is invoked when an **attribute or set of attributes** changes in a document.

**Technical Definition.** `void changedUpdate(DocumentEvent e)`. This event is fired **only from a `StyledDocument`**—a `PlainDocument` does not fire these events. It is used for style changes (font, color, paragraph alignment) rather than text content changes.

**Beginner-Friendly Explanation.** `changedUpdate` fires when the formatting of the text changes, not when the text itself changes. It only fires for styled documents like `JTextPane`.

---

#### Purposes

- To react to style changes in `JTextPane` or `JEditorPane`.
- To update UI that reflects formatting state (e.g., bold/italic indicators).
- To synchronize style-related state with a model.

---

#### Syntax Rules and Structure

```java
@Override
public void changedUpdate(DocumentEvent e) {
    // style changed
}
```

#### Constraints and Limitations

- **Never fires for `PlainDocument`** (`JTextField`, `JTextArea`).
- Many `DocumentListener` implementations leave this method empty for plain-text components.

---

### Annotated Complete Step-by-Step Code Examples (changedUpdate)

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import javax.swing.text.StyledDocument;

public class ChangedUpdateDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("changedUpdate Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();                      // StyledDocument
            JLabel status = new JLabel(" ", SwingConstants.CENTER);

            pane.getDocument().addDocumentListener(new DocumentListener() {
                public void insertUpdate(DocumentEvent e) {}
                public void removeUpdate(DocumentEvent e) {}
                public void changedUpdate(DocumentEvent e) {       // 1. Style
                    status.setText("Style changed at offset " + e.getOffset());
                }
            });

            pane.setText("Hello");
            // Programmatically change style → fires changedUpdate
            SwingUtilities.invokeLater(() -> {
                javax.swing.text.SimpleAttributeSet attrs =
                    new javax.swing.text.SimpleAttributeSet();
                javax.swing.text.StyleConstants.setBold(attrs, true);
                pane.getStyledDocument().setCharacterAttributes(0, 5, attrs, false);
            });

            frame.setLayout(new java.awt.BorderLayout());
            frame.add(new JScrollPane(pane), java.awt.BorderLayout.CENTER);
            frame.add(status, java.awt.BorderLayout.SOUTH);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the bold attribute is applied programmatically, `changedUpdate` fires and the status label updates. `insertUpdate` and `removeUpdate` are empty because this example only cares about style changes.

---

## 3. DocumentEvent — The Event Object

### Definitions

**Core Definition.** `DocumentEvent` is the event object delivered to `DocumentListener`s. It describes a change to a `Document`'s content or attributes.

**Technical Definition.** `public interface DocumentEvent`. It provides `getDocument()` (the changed document), `getOffset()` (the starting position), `getLength()` (the number of characters affected), and `getType()` (the `EventType`: `INSERT`, `REMOVE`, or `CHANGE`). The event is typically an instance of `DefaultDocumentEvent` defined in `AbstractDocument`.

**Beginner-Friendly Explanation.** `DocumentEvent` is the "what changed" note delivered to your listener. It tells you which document changed, where the change started, how many characters were affected, and what kind of change it was.

---

### Purposes

- To identify the changed document.
- To locate the change via offset and length.
- To distinguish insertion, removal, and style changes via the event type.

---

### Syntax Rules and Structure

```java
Document doc = e.getDocument();
int offset = e.getOffset();
int length = e.getLength();
DocumentEvent.EventType type = e.getType();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getDocument()` | The changed `Document`. |
| `getOffset()` | The starting position of the change. |
| `getLength()` | The number of characters inserted or removed. |
| `getType()` | `INSERT`, `REMOVE`, or `CHANGE`. |

---

### Annotated Complete Step-by-Step Code Examples

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;

public class InspectDocumentEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect DocumentEvent");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.getDocument().addDocumentListener(new DocumentListener() {
                public void insertUpdate(DocumentEvent e) { log(e); }
                public void removeUpdate(DocumentEvent e) { log(e); }
                public void changedUpdate(DocumentEvent e) { log(e); }
                private void log(DocumentEvent e) {
                    System.out.println("Type: " + e.getType()
                        + ", Offset: " + e.getOffset()
                        + ", Length: " + e.getLength());
                }
            });

            frame.add(field);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing “a” prints “Type: INSERT, Offset: 0, Length: 1”. Pressing backspace prints “Type: REMOVE, Offset: 0, Length: 1”.

---

## 4. Document Listener Best Practices

### Definitions

**Core Definition.** Document listener best practices are the conventions and patterns that ensure text-change handling is correct, thread-safe, and free of infinite loops.

**Technical Definition.** Best practices include: **never mutating the document from within a `DocumentListener`** (use `DocumentFilter` instead); using `SwingUtilities.invokeLater` to defer UI updates that depend on the document change having settled; implementing all three methods (leaving unused ones empty) because there is no adapter; being aware that `changedUpdate` only fires for `StyledDocument`; and ensuring thread safety if the document may be mutated off the EDT.

**Beginner-Friendly Explanation.** Best practices for document listeners are the "rules of thumb" that prevent infinite loops, threading bugs, and missing events. They include things like "don't edit the document from inside the listener—use `DocumentFilter`," "use `invokeLater` for UI updates," and "remember `changedUpdate` only fires for styled documents."

---

### Purposes

- To prevent infinite loops caused by mutating the document from within the listener.
- To use the correct mechanism (`DocumentFilter`) for input transformation.
- To keep the GUI responsive by deferring heavy work.
- To correctly handle styled versus plain documents.
- To ensure thread safety when documents are mutated asynchronously.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JTextField field = new JTextField(20);
    field.getDocument().addDocumentListener(new DocumentListener() { // 1. Document
        public void insertUpdate(DocumentEvent e) { validate(); }
        public void removeUpdate(DocumentEvent e) { validate(); }
        public void changedUpdate(DocumentEvent e) { }              // 2. Empty
        private void validate() {
            SwingUtilities.invokeLater(() -> {                      // 3. Defer
                String text = field.getText();
                // live validation
            });
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Never mutate from listener | Use `DocumentFilter` for transformations |
| Defer UI updates | `SwingUtilities.invokeLater(...)` |
| Implement all three methods | Empty bodies for unused ones |
| Understand `changedUpdate` | Only fires for `StyledDocument` |
| Thread safety | Make listener thread-safe if document may be mutated off EDT |
| No adapter available | Must implement all three methods |

#### Syntax Rules

- **Never mutate the document from within `insertUpdate`, `removeUpdate`, or `changedUpdate`.** This causes an infinite loop; use `DocumentFilter` for input transformation.
- **Always implement all three methods**—there is no adapter class.
- **Always use `SwingUtilities.invokeLater`** to defer UI updates that depend on the document change having fully settled.
- **Always check whether the document is a `StyledDocument`** if `changedUpdate` matters; it never fires for `PlainDocument`.

#### Constraints and Limitations

- `DocumentListener` is for **observation**, not mutation. `DocumentFilter` is for mutation.
- The notification order is not guaranteed.
- Asynchronous document mutations notify listeners on the mutating thread, requiring thread-safe listener implementations.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Live Validation with Deferred Update

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import java.awt.*;

public class BestPracticeDocumentListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeDocumentListener::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice DocumentListener");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JTextField email = new JTextField(20);
        JLabel status = new JLabel(" ", SwingConstants.CENTER);

        email.getDocument().addDocumentListener(new DocumentListener() { // 1. Document
            public void insertUpdate(DocumentEvent e) { schedule(); }
            public void removeUpdate(DocumentEvent e) { schedule(); }
            public void changedUpdate(DocumentEvent e) { }            // 2. Empty
            private void schedule() {
                SwingUtilities.invokeLater(() -> {                     // 3. Defer
                    String text = email.getText();
                    status.setText(text.contains("@")
                        ? "Looks valid" : "Missing @");
                });
            }
        });

        frame.setLayout(new BorderLayout());
        frame.add(new JLabel("Email:"), BorderLayout.NORTH);
        frame.add(email, BorderLayout.CENTER);
        frame.add(status, BorderLayout.SOUTH);
        frame.setSize(400, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** As the user types, the status label updates to “Looks valid” or “Missing @”. The update is deferred via `invokeLater`, avoiding re-entrancy issues during the document mutation.

**Why the code produces this result.** The listener implements all three methods ( `changedUpdate` empty because `JTextField` does not fire it ). `insertUpdate` and `removeUpdate` call `schedule`, which defers the validation to run after the current document operation completes. This is the safe pattern for reading `getText()` in a document listener.

---

#### Example 2: Common Mistake — Mutating the Document from a DocumentListener

```java
import javax.swing.*;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;

public class BadMutation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Mutation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.getDocument().addDocumentListener(new DocumentListener() {
                public void insertUpdate(DocumentEvent e) {
                    try {
                        String text = e.getDocument().getText(0, e.getDocument().getLength());
                        if (text.contains("a")) {
                            e.getDocument().insertString(          // WRONG: mutate
                                e.getDocument().getLength(), "!", null);
                        }
                    } catch (Exception ex) { ex.printStackTrace(); }
                }
                public void removeUpdate(DocumentEvent e) {}
                public void changedUpdate(DocumentEvent e) {}
            });

            frame.add(field);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing “a” triggers an infinite loop: the listener inserts “!”, which fires another `insertUpdate`, which sees “a” again and inserts another “!”, recursively. The application may freeze or throw a `StackOverflowError`.

**Why this is wrong.** As the Java API documentation states, “implementations of the `DocumentListener` may not mutate the source of the event (i.e. the associated `Document`)”. The community consensus confirms: “`DocumentListener` should not be used to edit the document. Doing so would require a call to `SwingUtilities.invokeLater(...)` and since `DocumentListener` detects changes to the document, any edits would result in an infinite loop. Best bet is to use `DocumentFilter`”.

---

#### Example 3: Correct Mutation with DocumentFilter

```java
import javax.swing.*;
import javax.swing.text.AbstractDocument;
import javax.swing.text.AttributeSet;
import javax.swing.text.BadLocationException;
import javax.swing.text.DocumentFilter;

public class CorrectFilter {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Correct Filter");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            ((AbstractDocument) field.getDocument())
                .setDocumentFilter(new DocumentFilter() {                // 1. Filter
                    @Override
                    public void insertString(FilterBypass fb, int offset,
                        String text, AttributeSet attrs) throws BadLocationException {
                        super.insertString(fb, offset, text.toUpperCase(), attrs);
                    }
                });

            frame.add(field);
            frame.setSize(300, 100);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing lowercase letters results in uppercase characters in the field. No infinite loop occurs because the filter transforms the input **before** the document is mutated.

**Why this is correct.** `DocumentFilter` is the designated mechanism for transforming input. It intercepts insertions and removals **before** they reach the document, allowing controlled modification without triggering recursive document events.

---

### Real-World Cases

- **Live validation:** Email, phone, and URL validation as the user types.
- **Character counters:** Remaining-character displays in text areas.
- **Auto-calculation:** Summing numeric fields in real time.
- **Search-as-you-type:** Filtering a list based on document content.
- **Input masking/formatting:** Using `DocumentFilter` (not `DocumentListener`) to format phone numbers, dates, or currency.

---

## References

Oracle. *Interface DocumentListener*. Java Platform, Standard Edition 6 API Specification. https://docs.oracle.com/javase/6/docs/api/javax/swing/event/DocumentListener.html

Oracle. *How to Write a Document Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/documentlistener.html

Oracle. *How to Write a Document Listener* (mirror). https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/uiswing/events/documentlistener.html

OpenJDK. *DocumentListener.java source*. https://android.googlesource.com/platform/libcore.git/+/a43d67da105aa2b421ea3028491eb47055d1e0df

Stack Overflow. *Autocomplete Text box with Document Listener in Java* (DocumentListener should not mutate the document; use DocumentFilter). https://stackoverflow.com/feeds/question/40793999

Oracle. *javax.swing.event.DocumentListener* (Javadoc, MIT mirror). https://people.csail.mit.edu/dfhuynh/research/javadoc/jdk1.5.0/javax/swing/event/DocumentListener.html

Oracle. *How to Write a Document Listener* (DocumentEvent demo). https://docs.oracle.com/javase/tutorial/uiswing/events/documentlistener.html

Oracle. *Implementing a Document Filter*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/generaltext.html#filter