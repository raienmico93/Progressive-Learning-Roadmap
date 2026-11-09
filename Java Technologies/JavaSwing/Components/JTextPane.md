# JTextPane — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JTextPane` is a Swing text component that can be marked up with attributes that are represented graphically. It is a subclass of `JEditorPane` that insists its document implement the `StyledDocument` interface, enabling rich character- and paragraph-level styling, embedded components, and embedded images within the flow of text.

**Technical Definition.** `public class JTextPane extends JEditorPane`. This component models paragraphs that are composed of runs of character-level attributes. Each paragraph may have a logical style attached to it which contains the default attributes to use if not overridden by attributes set on the paragraph or character run. Components and images may be embedded in the flow of text. A text pane requires that its document be an instance of a class that implements the `StyledDocument` interface; the default is `DefaultStyledDocument`. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`. Serialized `JTextPane` objects are not compatible with future Swing releases.

**Beginner-Friendly Explanation.** `JTextPane` is the “rich text editor” of Swing. While a `JTextArea` shows only plain, unstyled text and a `JEditorPane` can render HTML and RTF, a `JTextPane` gives you programmatic, fine-grained control over every character and paragraph. You can make one word bold, another red, center a paragraph, change line spacing, and even embed images or buttons directly into the text flow. Think of it as a document editor where you can select any range of text and apply formatting—exactly like the “Bold,” “Italic,” and “Font Color” buttons in a word processor. It is built on top of `JEditorPane` but specializes in styled text rather than web-page rendering.

---

### Key Characteristics

1. **StyledDocument model.** `JTextPane` requires its document to implement the `StyledDocument` interface. The default document is `DefaultStyledDocument`, which stores character and paragraph attributes and supports styled text.
2. **Character-level attributes.** Formatting can be applied to individual characters or contiguous ranges (e.g., bold, italic, font family, font size, foreground and background colors) using `setCharacterAttributes` or the document’s `setCharacterAttributes`.
3. **Paragraph-level attributes.** Formatting can be applied to entire paragraphs (e.g., alignment, line spacing, indentation, tab stops) using `setParagraphAttributes` or the document’s `setParagraphAttributes`.
4. **Logical styles.** Each paragraph may have a logical style attached, providing a default set of attributes that apply unless overridden by paragraph or character attributes.
5. **Embedded components and images.** Components and images may be embedded directly in the flow of text, enabling rich document layouts.
6. **`JEditorPane` heritage.** `JTextPane` inherits all the functionality of `JEditorPane`, including content-type support, URL loading, and hyperlink events (though its primary use is styled text rather than HTML rendering).
7. **EDT requirement and serialization caveat.** All construction and manipulation must occur on the EDT; serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, generics, and lambda expressions.
- Familiarity with `JTextComponent`, `JEditorPane`, and the Swing text component hierarchy.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `StyledDocument`, `SimpleAttributeSet`, and `StyleConstants` from `javax.swing.text`.
- Familiarity with `JScrollPane` for scrolling.
- A JDK (version 8 or later; `JTextPane` is available in all modern JDKs).

---

### Related Programming Areas

- **`StyledDocument` and `DefaultStyledDocument`:** The underlying model for styled content.
- **`StyleConstants` and `SimpleAttributeSet`:** The utility classes for defining character and paragraph attributes.
- **`Style` and `StyleContext`:** Named attribute sets that can be applied to documents.
- **`JEditorPane`:** The parent class; `JTextPane` adds styled-document requirements and convenience methods.
- **`JTextArea`:** The simpler alternative for unstyled, multi-line plain text.
- **`JScrollPane`:** The standard container for scrolling the text pane.
- **`DocumentListener` and `StyledDocument` events:** For observing changes to the document.

---

## Core Concepts and Features

---

## 1. JTextPane — The Class

### Definitions

**Core Definition.** `JTextPane` is the Swing class for creating a text component that supports character- and paragraph-level styling, embedded components, and embedded images.

**Technical Definition.** `public class JTextPane extends JEditorPane`. It provides constructors that create a pane with a default `StyledDocument`, or accept a specific `StyledDocument`. The class provides convenience methods such as `setCharacterAttributes`, `setParagraphAttributes`, and `getStyledDocument` that delegate to the document or styled editor kit.

**Beginner-Friendly Explanation.** `JTextPane` is the class you use when you need rich text editing or display. You create it, get its `StyledDocument`, and use `StyleConstants` to define formatting that you apply to ranges of text.

---

### Purposes

- To provide a text component that supports rich character- and paragraph-level formatting.
- To enable programmatic control over text styles for applications like editors, note-taking tools, and document viewers.
- To support embedded components and images within the text flow.
- To serve as the foundation for applications that need more than plain text but do not require full HTML rendering.
- To offer a convenient API for applying attributes to selections without directly manipulating the document.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JTextPane pane = new JTextPane();                                // default StyledDocument
JTextPane pane = new JTextPane(StyledDocument document);          // custom document
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JTextPane()` | Creates a text pane with a new `DefaultStyledDocument`. |
| `JTextPane(StyledDocument doc)` | Creates a text pane using the supplied styled document. |

#### Syntax Rules

- The document must implement `StyledDocument`; passing a plain `Document` is not allowed.
- The pane must be added to a container (typically a `JScrollPane`) to be displayed.
- Construction must occur on the EDT.

#### Constraints and Limitations

- `JTextPane` is not a full HTML renderer; for HTML content, use `JEditorPane` with `HTMLEditorKit`.
- Complex document manipulation should be done through the `StyledDocument` rather than `setText` for large content.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JTextPane with Initial Text

```java
import javax.swing.*;
import javax.swing.text.*;
import java.awt.*;

public class MinimalTextPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JTextPane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();                     // 1. Create
            pane.setText("Hello, styled world!");                  // 2. Set text

            StyledDocument doc = pane.getStyledDocument();         // 3. Get model
            SimpleAttributeSet bold = new SimpleAttributeSet();    // 4. Define style
            StyleConstants.setBold(bold, true);
            doc.setCharacterAttributes(0, 5, bold, false);         // 5. Apply

            JScrollPane scroll = new JScrollPane(pane);            // 6. Scroll pane
            frame.add(scroll, BorderLayout.CENTER);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing “**Hello**, styled world!” with the first five characters (“Hello”) in bold.

**Why the code produces this result.** The `JTextPane` is created with a default `StyledDocument`. `setText` inserts plain text. A `SimpleAttributeSet` is configured with bold, and `doc.setCharacterAttributes(0, 5, bold, false)` applies the bold attribute to characters 0–4 without replacing existing attributes.

---

### Real-World Cases

- **Note-taking applications** where users apply bold, italic, and colors to selected text.
- **Document viewers** that display styled content loaded from a database or file.
- **Rich-text editors** that need programmatic control over formatting.
- **Chat applications** that render styled messages (e.g., colored usernames, bold text).

---

## 2. Text Pane Creation

### Definitions

**Core Definition.** Text pane creation is the process of instantiating a `JTextPane` with either the default `DefaultStyledDocument` or a custom `StyledDocument`.

**Technical Definition.** `JTextPane` provides two constructors: a no-argument constructor that creates a pane with a new `DefaultStyledDocument`, and a constructor that accepts a `StyledDocument`. The document serves as the model for the pane’s content and styling.

**Beginner-Friendly Explanation.** Creating a text pane means deciding whether to use the default styled document or a custom one. Most applications use the default.

---

### Purposes

- To instantiate the styled text component with the appropriate document model.
- To prepare the pane for text insertion, attribute application, and container addition.
- To supply a custom `StyledDocument` when specialized storage or behavior is required.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JTextPane pane = new JTextPane();
// or
StyledDocument customDoc = new DefaultStyledDocument();
JTextPane pane = new JTextPane(customDoc);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `StyledDocument doc` | The document model for the pane’s content and attributes. |

#### Syntax Rules

- The document must implement `StyledDocument`.
- The pane’s document can be replaced later with `setDocument`, provided the new document is a `StyledDocument`.

#### Constraints and Limitations

- Passing a non-`StyledDocument` document throws `IllegalArgumentException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Text Pane with a Custom StyledDocument

```java
import javax.swing.*;
import javax.swing.text.*;

public class CustomDocumentCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom Document");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultStyledDocument doc = new DefaultStyledDocument(); // 1. Custom doc
            JTextPane pane = new JTextPane(doc);                     // 2. Use it

            try {
                doc.insertString(0, "Custom document content", null); // 3. Insert
            } catch (BadLocationException e) {
                e.printStackTrace();
            }

            frame.add(new JScrollPane(pane));
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window showing the text “Custom document content” in a styled text pane backed by the supplied `DefaultStyledDocument`.

**Why the code produces this result.** The custom document is created and passed to the constructor. Text is inserted directly into the document model, and the pane renders it.

---

### Real-World Cases

- **Document-centric applications** that maintain a reference to the `StyledDocument` for programmatic manipulation.
- **Undo/redo systems** that operate on the document model.

---

## 3. Text Pane Methods — setCharacterAttributes()

### Definitions

**Core Definition.** `setCharacterAttributes(AttributeSet attr, boolean replace)` applies character-level attributes to the current selection (or to the input attributes if there is no selection).

**Technical Definition.** `public void setCharacterAttributes(AttributeSet attr, boolean replace)`. The method delegates to the `StyledEditorKit` and applies the attributes to character content. If `replace` is `true`, the new attributes replace the existing character attributes; if `false`, they are merged.

**Beginner-Friendly Explanation.** `setCharacterAttributes` is how you make selected text bold, change its color, or set its font. If nothing is selected, it sets the attributes that will be used for the next typed characters.

---

### Purposes

- To apply formatting (bold, italic, font, color, etc.) to a range of characters.
- To set the input attributes for text that will be typed next.
- To programmatically style selections without directly manipulating the document.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SimpleAttributeSet attrs = new SimpleAttributeSet();
StyleConstants.setBold(attrs, true);
StyleConstants.setForeground(attrs, Color.RED);
pane.setCharacterAttributes(attrs, false);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `AttributeSet attr` | The attributes to apply (e.g., bold, foreground). |
| `boolean replace` | `true` replaces existing character attributes; `false` merges. |

#### Syntax Rules

- If there is a selection, the attributes apply to the selected characters.
- If there is no selection, the attributes become the input attributes for subsequent typing.
- Use `StyleConstants` to define attributes portably.

#### Constraints and Limitations

- The method operates on the current selection of the pane; for arbitrary ranges, use the document’s `setCharacterAttributes(int offset, int length, AttributeSet s, boolean replace)` instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Applying Bold and Color to a Selection

```java
import javax.swing.*;
import javax.swing.text.*;
import java.awt.*;

public class CharacterAttributesDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Character Attributes");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();
            pane.setText("Select some text and click Bold.");

            JButton boldButton = new JButton("Bold");
            boldButton.addActionListener(e -> {
                SimpleAttributeSet attrs = new SimpleAttributeSet(); // 1. Attributes
                StyleConstants.setBold(attrs, true);                 // 2. Bold
                StyleConstants.setForeground(attrs, Color.BLUE);     // 3. Blue
                pane.setCharacterAttributes(attrs, false);           // 4. Apply
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(pane), BorderLayout.CENTER);
            panel.add(boldButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(450, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting a portion of the text and clicking “Bold” applies bold and blue foreground to the selected characters.

**Why the code produces this result.** `setCharacterAttributes` applies the `SimpleAttributeSet` (containing bold and blue foreground) to the current selection. Because `replace` is `false`, existing attributes are preserved and the new ones are merged.

---

### Real-World Cases

- **Formatting toolbars** in rich-text editors (Bold, Italic, Underline, Font Color).
- **Syntax highlighting** where keywords are colored by applying character attributes programmatically.
- **Chat clients** that style usernames or mentions.

---

## 4. Text Pane Methods — setParagraphAttributes()

### Definitions

**Core Definition.** `setParagraphAttributes(AttributeSet attr, boolean replace)` applies paragraph-level attributes (e.g., alignment, line spacing, indentation) to the paragraphs containing the current selection.

**Technical Definition.** `public void setParagraphAttributes(AttributeSet attr, boolean replace)`. The method applies the attributes to the paragraphs that intersect the selection. If `replace` is `true`, existing paragraph attributes are replaced; if `false`, they are merged.

**Beginner-Friendly Explanation.** `setParagraphAttributes` is how you center a paragraph, change line spacing, or adjust indentation. It affects whole paragraphs, not individual characters.

---

### Purposes

- To apply paragraph-level formatting such as alignment (left, center, right, justify).
- To set line spacing, indentation, and tab stops.
- To style entire paragraphs without affecting character-level attributes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SimpleAttributeSet paraAttrs = new SimpleAttributeSet();
StyleConstants.setAlignment(paraAttrs, StyleConstants.ALIGN_CENTER);
pane.setParagraphAttributes(paraAttrs, false);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `AttributeSet attr` | The paragraph attributes to apply. |
| `boolean replace` | `true` replaces existing paragraph attributes; `false` merges. |

#### Syntax Rules

- The attributes apply to every paragraph touched by the selection.
- Use `StyleConstants` constants for alignment (`ALIGN_LEFT`, `ALIGN_CENTER`, `ALIGN_RIGHT`, `ALIGN_JUSTIFIED`).

#### Constraints and Limitations

- For precise control over individual paragraphs, use the document’s `setParagraphAttributes(int offset, int length, AttributeSet s, boolean replace)`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Centering a Paragraph

```java
import javax.swing.*;
import javax.swing.text.*;

public class ParagraphAttributesDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Paragraph Attributes");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();
            pane.setText("This paragraph will be centered.\n"
                + "This one will remain left-aligned.");

            JButton centerButton = new JButton("Center First Paragraph");
            centerButton.addActionListener(e -> {
                pane.setSelectionStart(0);                            // 1. Select first
                pane.setSelectionEnd(35);                             // 2. paragraph
                SimpleAttributeSet attrs = new SimpleAttributeSet();  // 3. Attributes
                StyleConstants.setAlignment(attrs, StyleConstants.ALIGN_CENTER); // 4. Center
                pane.setParagraphAttributes(attrs, false);            // 5. Apply
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(pane), BorderLayout.CENTER);
            panel.add(centerButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(450, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The first paragraph becomes centered, while the second remains left-aligned.

**Why the code produces this result.** The selection is set to the first paragraph, and `setParagraphAttributes` applies the center alignment attribute to the paragraphs intersecting the selection.

---

### Real-World Cases

- **Document editors** with alignment controls for headings and body text.
- **Report generators** that center titles and justify body paragraphs.

---

## 5. Text Pane Methods — getStyledDocument()

### Definitions

**Core Definition.** `getStyledDocument()` returns the pane’s document as a `StyledDocument`, providing direct access to the model for attribute manipulation and text insertion.

**Technical Definition.** `public StyledDocument getStyledDocument()`. The method returns the document as a `StyledDocument` if it implements that interface (which it must for a `JTextPane`). It provides access to methods such as `insertString`, `setCharacterAttributes`, and `setParagraphAttributes` with offset and length parameters.

**Beginner-Friendly Explanation.** `getStyledDocument()` gives you the underlying document model. You use it to insert text at specific positions and apply attributes to arbitrary ranges, rather than relying on the current selection.

---

### Purposes

- To access the document model for direct insertion of styled text.
- To apply attributes to specific ranges without changing the selection.
- To read or modify the document’s content and structure programmatically.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
StyledDocument doc = pane.getStyledDocument();
doc.insertString(doc.getLength(), "Appended text", attributes);
doc.setCharacterAttributes(offset, length, attributes, replace);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getStyledDocument()` | Returns the pane’s `StyledDocument`. |
| `insertString(int offset, String str, AttributeSet a)` | Inserts text at the specified offset. |
| `setCharacterAttributes(int offset, int length, AttributeSet s, boolean replace)` | Applies attributes to a specific range. |

#### Syntax Rules

- `insertString` throws `BadLocationException` if the offset is invalid.
- Passing `null` for the attributes inserts plain text.
- All document manipulation must occur on the EDT.

#### Constraints and Limitations

- Direct document manipulation bypasses the pane’s selection; use it when you need precise control.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Appending Styled Text via the Document

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
            StyledDocument doc = pane.getStyledDocument();           // 1. Get document

            SimpleAttributeSet redBold = new SimpleAttributeSet();   // 2. Attributes
            StyleConstants.setForeground(redBold, Color.RED);
            StyleConstants.setBold(redBold, true);

            try {
                doc.insertString(doc.getLength(), "Normal text. ", null); // 3. Plain
                doc.insertString(doc.getLength(), "Red bold text.", redBold); // 4. Styled
            } catch (BadLocationException e) {
                e.printStackTrace();
            }

            frame.add(new JScrollPane(pane));
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The pane displays “Normal text. **Red bold text.**” with the second phrase in red and bold.

**Why the code produces this result.** The `StyledDocument` is obtained from the pane. `insertString` appends plain text with `null` attributes, then appends styled text with the `redBold` attribute set.

---

### Real-World Cases

- **Log viewers** that append colored severity levels (INFO, WARN, ERROR) to a document.
- **Programmatic document generation** where text is inserted and styled without user interaction.

---

## 6. JTextPane Best Practices

### Definitions

**Core Definition.** `JTextPane` best practices are the conventions and patterns that ensure styled text panes are used effectively, thread-safely, and performantly in Swing applications.

**Technical Definition.** Best practices include: always using `JScrollPane` to provide scrolling; manipulating the `StyledDocument` directly for programmatic insertion and styling rather than repeatedly calling `setText`; using `StyleConstants` for portable attribute definitions; keeping heavy document operations off the EDT; and distinguishing between `setCharacterAttributes` (for character formatting) and `setParagraphAttributes` (for paragraph formatting).

**Beginner-Friendly Explanation.** Best practices for text panes are the “rules of thumb” that prevent frozen GUIs, lost formatting, and inefficient code. They include things like “always use a scroll pane,” “append to the document instead of resetting the text,” and “use `StyleConstants` so your formatting works with any look and feel.”

---

### Purposes

- To maintain thread safety by respecting the EDT.
- To preserve formatting by manipulating the document rather than resetting text.
- To ensure portability of styles across look-and-feel implementations.
- To keep the GUI responsive during large document operations.
- To use the correct method for character versus paragraph formatting.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JTextPane pane = new JTextPane();                             // 1. Create
    StyledDocument doc = pane.getStyledDocument();                // 2. Model

    SimpleAttributeSet attrs = new SimpleAttributeSet();          // 3. StyleConstants
    StyleConstants.setBold(attrs, true);
    StyleConstants.setForeground(attrs, Color.BLUE);

    try {
        doc.insertString(doc.getLength(), "Styled text", attrs);  // 4. Append
    } catch (BadLocationException e) {
        e.printStackTrace();
    }

    JScrollPane scroll = new JScrollPane(pane);                   // 5. Scroll
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use a scroll pane | `new JScrollPane(pane)` |
| Append via the document | `doc.insertString(doc.getLength(), text, attrs)` |
| Use `StyleConstants` | `StyleConstants.setBold(...)`, `setForeground(...)` |
| Character vs. paragraph | `setCharacterAttributes` for runs; `setParagraphAttributes` for paragraphs |
| Off-EDT heavy operations | Use `SwingWorker` for large document loading or processing |
| Avoid repeated `setText` | Use `insertString` to append without reparsing the entire document |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always wrap `JTextPane` in a `JScrollPane`** for scrolling.
- **Always append using `StyledDocument.insertString`** rather than `setText(getText() + newText)`, which re-parses the entire document and loses attributes.
- **Always use `StyleConstants`** to define attributes for portability.
- **Never perform heavy document manipulation on the EDT**; use a background thread and update the document in chunks.
- **Choose the correct attribute method:** `setCharacterAttributes` for character formatting, `setParagraphAttributes` for paragraph formatting.

#### Constraints and Limitations

- `setText` resets the document and loses all custom attributes.
- Large document inserts on the EDT can freeze the GUI.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Styled Text Pane with Appending

```java
import javax.swing.*;
import javax.swing.text.*;
import java.awt.*;

public class BestPracticeTextPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeTextPane::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice TextPane");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JTextPane pane = new JTextPane();                             // 1. Create
        StyledDocument doc = pane.getStyledDocument();                // 2. Model

        SimpleAttributeSet heading = new SimpleAttributeSet();        // 3. StyleConstants
        StyleConstants.setBold(heading, true);
        StyleConstants.setFontSize(heading, 18);
        StyleConstants.setForeground(heading, Color.DARK_GRAY);

        SimpleAttributeSet body = new SimpleAttributeSet();           // 4. Body style
        StyleConstants.setFontSize(body, 12);

        try {
            doc.insertString(doc.getLength(), "Document Title\n", heading); // 5. Append
            doc.insertString(doc.getLength(), "This is body text with normal formatting.", body);
        } catch (BadLocationException e) {
            e.printStackTrace();
        }

        JScrollPane scroll = new JScrollPane(pane);                   // 6. Scroll
        frame.add(scroll, BorderLayout.CENTER);
        frame.setSize(500, 300);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window showing a bold, dark-gray, 18-point “Document Title” followed by normal 12-point body text, all within a scrollable styled text pane.

**Why the code produces this result.** The `StyledDocument` is used to append text with the appropriate attribute sets. `StyleConstants` defines the formatting portably. The scroll pane provides scrolling. No `setText` calls are used, so attributes are preserved.

---

#### Example 2: Common Mistake — Using `setText` to Append

```java
import javax.swing.*;

public class BadTextPaneAppend {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Append");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextPane pane = new JTextPane();
            pane.setText("Original");                                // 1. Set
            // WRONG: re-parses the entire document and loses attributes
            pane.setText(pane.getText() + " appended");               // 2. Bad append

            frame.add(new JScrollPane(pane));
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text may display “Original appended,” but any character attributes previously applied are lost because `setText` replaces the document.

**Why this is wrong.** As the community guidance explains, using `setText(getText() + newText)` to append “recreates the Document and loses all the custom attributes you added previously”. The correct approach is to use `StyledDocument.insertString(doc.getLength(), newText, attributes)`.

---

### Real-World Cases

- **Production rich-text editors** use `StyledDocument.insertString` and `StyleConstants` to apply formatting without losing existing styles.
- **Log viewers** append colored severity levels to the document directly, preserving previous entries’ formatting.
- **Educational examples** use the best-practice skeleton to demonstrate correct document manipulation and EDT usage from the start.

---

## References

Oracle. *Class JTextPane*. Java Platform, Standard Edition 11 API Specification. https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/JTextPane.html

OpenJDK. *JTextPane (Java Platform SE 8 b00)*. https://cr.openjdk.org/~jjg/8008367/docs/api/javax/swing/JTextPane.html

Oracle. *Text Component Features*. The Java Tutorials. https://www.cse.iitb.ac.in/~cs296/Java/Swing/uiswing/components/generaltext.html

Oracle. *StyledDocument (Java Platform SE 8)*. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/StyledDocument.html

Stack Overflow. *JTextPane appending a new string*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/4059198/jtextpane-appending-a-new-string

Stack Overflow. *Making a single line bold in a JTextPane without using HTML*. https://stackoverflow.com/questions/...

Oracle. *JTextPane AttributeSet — setCharacterAttributes vs setParagraphAttributes*. https://stackoverflow.com/questions/...

Oracle. *StyleConstants (Java Platform SE 8)*. https://docs.oracle.com/javase/8/docs/api/javax/swing/text/StyleConstants.html