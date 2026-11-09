# JEditorPane — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JEditorPane` is a Swing component designed to edit and display various kinds of content, including styled text such as HTML and RTF. It is the foundation for Swing's styled text components and provides a mechanism through which you can add support for custom text formats.

**Technical Definition.** `public class JEditorPane extends JTextComponent`. A `JEditorPane` is a text component that uses implementations of the `EditorKit` class to accomplish its behavior. It effectively morphs into the proper kind of text editor for the kind of content it is given. The content type that the editor is bound to at any given time is determined by the `EditorKit` currently installed. By default, the following content types are known: `text/plain` (plain text, the default if the type is not recognized), `text/html` (HTML text, using `HTMLEditorKit` which provides HTML 3.2 support), and `text/rtf` (RTF text, using `RTFEditorKit` which provides limited support for the Rich Text Format). Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JEditorPane` is a powerful text area that can display more than just plain text. While a `JTextArea` shows only simple, unstyled text, a `JEditorPane` can render rich content like HTML pages (with formatting, images, and even hyperlinks) or RTF documents. Think of it as a mini web browser built into your Swing application. You can use it to show formatted help files, display rich text documents, or even create a simple HTML viewer. It is the foundation for Swing's more advanced text components, and it can be customized to support entirely new text formats if needed.

---

### Key Characteristics

1. **Multi-format content support.** `JEditorPane` is not limited to plain text. It can display HTML, RTF, and custom formats through pluggable `EditorKit` implementations. This makes it the foundation for Swing's styled text components.
2. **EditorKit architecture.** The behavior of the component is delegated to an `EditorKit`. When the content type changes (via `setContentType` or `setPage`), the appropriate `EditorKit` is installed, and the pane "morphs" into the proper kind of editor for that content.
3. **URL-based loading.** The `setPage(URL)` method can load content directly from a URL. The content type is determined from the URL, and the registered `EditorKit` for that type is used. This enables the classic "help viewer" use case.
4. **Hyperlink support.** When the pane is **not editable** (`setEditable(false)`), the HTML `EditorKit` generates hyperlink events. A `HyperlinkListener` can be registered to respond to link activation, enabling navigation between pages.
5. **`setText` semantics and thread safety.** The `setText` method replaces the contents of the current document by parsing the given string using the current `EditorKit`. While the method is documented as thread-safe, the recommended approach for large content is to create a new `Document` via `getEditorKit().createDefaultDocument()` and replace the existing one, or to load content using a `SwingWorker` to avoid freezing the EDT.
6. **EDT requirement and serialization caveat.** Like all Swing components, `JEditorPane` must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with `JTextComponent`, `JTextArea`, and the Swing text component hierarchy.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `java.net.URL` and basic I/O concepts.
- Familiarity with `HyperlinkListener` and the `javax.swing.event` package for hyperlink handling.
- A JDK (version 8 or later; `JEditorPane` is available in all modern JDKs).

---

### Related Programming Areas

- **`JTextPane`:** A subclass of `JEditorPane` that adds support for embedding images and components directly into the text.
- **`EditorKit` and its subclasses (`HTMLEditorKit`, `RTFEditorKit`, `DefaultEditorKit`):** The pluggable strategies that define how content is parsed, edited, and rendered.
- **`Document`:** The underlying model for the text content.
- **`HyperlinkListener`:** The event interface for responding to hyperlink activation.
- **`JScrollPane`:** The standard container for adding scrolling to a `JEditorPane`.
- **`SwingWorker`:** The mechanism for loading large content off the EDT.
- **`JTextArea`:** The simpler alternative for unstyled, multi-line plain text.

---

## Core Concepts and Features

---

## 1. JEditorPane — The Class

### Definitions

**Core Definition.** `JEditorPane` is the Swing class for creating a text component that can edit and display various kinds of content, including styled text like HTML and RTF.

**Technical Definition.** `public class JEditorPane extends JTextComponent`. It delegates behavior to an `EditorKit`. The content type (e.g., `text/html`, `text/rtf`, `text/plain`) determines which `EditorKit` is used. The default content type is `text/plain`. The class provides constructors that accept an initial URL, a content type and text, or neither.

**Beginner-Friendly Explanation.** `JEditorPane` is the class you use when you need more than just plain text. You create it, tell it what kind of content it should handle (HTML, RTF, or plain text), and load the content from a string or a URL. It handles the rendering and, if editable, the editing of that content.

---

### Purposes

- To display styled text such as HTML or RTF within a Swing application.
- To provide a foundation for custom text formats through the `EditorKit` architecture.
- To load and display content directly from a URL, enabling help viewers and simple browsers.
- To support hyperlink navigation when displaying HTML content.
- To serve as the parent class for `JTextPane`, which adds embedded components and images.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JEditorPane pane = new JEditorPane();
JEditorPane pane = new JEditorPane(String url);
JEditorPane pane = new JEditorPane(String type, String text);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JEditorPane()` | Creates a pane with no initial content; defaults to `text/plain`. |
| `JEditorPane(String url)` | Creates a pane initialized to the content at the given URL. |
| `JEditorPane(String type, String text)` | Creates a pane with the specified content type and initial text. This is a convenience constructor that calls `setContentType` and `setText`. |

#### Syntax Rules

- The content type determines which `EditorKit` is used.
- The pane must be added to a container (typically a `JScrollPane`) to be displayed.
- Construction must occur on the EDT.

#### Constraints and Limitations

- The default HTML support is limited to HTML 3.2; CSS and JavaScript are not supported.
- `JEditorPane` is for text content; for complex rich text with embedded components, use `JTextPane`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JEditorPane with HTML Content

```java
import javax.swing.*;
import java.awt.*;

public class MinimalEditorPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal JEditorPane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane();                   // 1. Create
            pane.setContentType("text/html");                       // 2. Set type
            pane.setText("<html><body><h1>Hello</h1>"
                + "<p>This is <b>bold</b> and <i>italic</i>.</p>"
                + "</body></html>");                                // 3. Set content
            pane.setEditable(false);                                // 4. Read-only

            JScrollPane scroll = new JScrollPane(pane);              // 5. Scroll pane

            frame.add(scroll, BorderLayout.CENTER);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window displaying an HTML rendering with a large “Hello” heading and a paragraph containing bold and italic text. The pane is read-only.

**Why the code produces this result.** `setContentType("text/html")` installs the `HTMLEditorKit`, and `setText` parses the HTML string and renders it. `setEditable(false)` prevents user modification. The scroll pane provides scrolling.

---

### Real-World Cases

- **Help viewers:** Displaying HTML help files bundled with the application.
- **Simple browsers:** Navigating local or remote HTML pages via `setPage`.
- **Rich text editors:** Editing HTML or RTF documents in a Swing application.
- **Documentation viewers:** Rendering RTF or HTML documentation inline.

---

## 2. Editor Pane Creation

### Definitions

**Core Definition.** Editor pane creation is the process of instantiating a `JEditorPane` with a specific content type and initial content, either from a string or a URL.

**Technical Definition.** `JEditorPane` provides three constructors. The no-argument constructor creates a pane with the default `text/plain` content type. The `(String url)` constructor loads content from a URL. The `(String type, String text)` constructor sets both the content type and the initial text.

**Beginner-Friendly Explanation.** Creating an editor pane means deciding what kind of content it should handle (HTML, RTF, or plain text) and what content to show initially.

---

### Purposes

- To instantiate the pane with the appropriate content type for the intended use.
- To load initial content from a string or a URL.
- To prepare the pane for editing (if editable) or display (if read-only).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JEditorPane htmlPane = new JEditorPane("text/html", "<html>...</html>");
JEditorPane rtfPane = new JEditorPane("text/rtf", "{\\rtf1 ...}");
JEditorPane plainPane = new JEditorPane("text/plain", "Hello");
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String type` | The MIME content type (e.g., `text/html`, `text/rtf`, `text/plain`). |
| `String text` | The initial content, in the format of the specified type. |

#### Syntax Rules

- The content type must be supported by a registered `EditorKit`.
- If the type is not recognized, `text/plain` is used as the default.

#### Constraints and Limitations

- The convenience constructor is equivalent to calling `setContentType` and then `setText`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating an HTML Pane with Initial Content

```java
import javax.swing.*;

public class HtmlCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("HTML Pane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String html = "<html><body>"
                + "<h2>Welcome</h2>"
                + "<ul><li>Item 1</li><li>Item 2</li></ul>"
                + "</body></html>";
            JEditorPane pane = new JEditorPane("text/html", html);  // 1. Type + text
            pane.setEditable(false);

            frame.add(new JScrollPane(pane));
            frame.setSize(350, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window displaying an HTML-rendered heading “Welcome” and a bulleted list with two items.

**Why the code produces this result.** The constructor sets the content type to `text/html` and parses the HTML string, producing the rendered output.

---

### Real-World Cases

- **Loading help content** from an HTML string generated at runtime.
- **Displaying RTF documents** loaded from a database or file.

---

## 3. Editor Pane Methods — setContentType()

### Definitions

**Core Definition.** `setContentType(String type)` sets the type of content that the editor handles, which determines the `EditorKit` used to parse and render the content.

**Technical Definition.** `public final void setContentType(String type)`. The method calls `getEditorKitForContentType(type)`. If an editor kit for the type can be successfully located, it is set via `setEditorKit`. If the type is not recognized, `text/plain` is used as the default.

**Beginner-Friendly Explanation.** `setContentType` tells the editor pane what kind of content it should expect—HTML, RTF, or plain text—so it knows how to render it.

---

### Purposes

- To switch the pane's content type at runtime.
- To ensure the correct `EditorKit` is used for parsing and rendering.
- To enable HTML hyperlink support (when combined with `setEditable(false)`).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
pane.setContentType("text/html");
pane.setContentType("text/rtf");
pane.setContentType("text/plain");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setContentType(String)` | Sets the content type and installs the corresponding `EditorKit`. |

#### Syntax Rules

- Must be called before `setText` if the content is in a non-plain format.
- The change may reset the document model.

#### Constraints and Limitations

- Unrecognized types default to `text/plain`.
- HTML support is limited to HTML 3.2.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Content Type

```java
import javax.swing.*;

public class ContentTypeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Content Type");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane();
            pane.setContentType("text/html");                       // 1. HTML
            pane.setText("<html><b>Bold</b> text</html>");
            pane.setEditable(false);

            JButton plainButton = new JButton("Switch to Plain");
            plainButton.addActionListener(e -> {
                pane.setContentType("text/plain");                   // 2. Plain
                pane.setText("<b>This is not bold</b>");
            });

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(pane), BorderLayout.CENTER);
            panel.add(plainButton, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially, the pane renders “Bold text” with “Bold” in bold. After clicking the button, the content type switches to plain text, and the literal string `<b>This is not bold</b>` is displayed.

**Why the code produces this result.** `setContentType` installs the appropriate `EditorKit`. The `HTMLEditorKit` renders the tags; the `DefaultEditorKit` displays them as literal text.

---

### Real-World Cases

- **Format switching** in document editors (e.g., switching between HTML and plain text views).
- **Loading content** whose type is determined at runtime.

---

## 4. Editor Pane Methods — setPage()

### Definitions

**Core Definition.** `setPage(String url)` or `setPage(URL url)` loads the content at the specified URL into the editor pane, determining the content type from the URL.

**Technical Definition.** `public void setPage(String url)` and `public void setPage(URL url)`. The method resolves the URL, determines the content type, installs the appropriate `EditorKit`, and loads the content. If the content type is HTML, relative references (e.g., for images) cannot be resolved unless a `<base>` tag is used or the `Base` property on `HTMLDocument` is set.

**Beginner-Friendly Explanation.** `setPage` is how you load a web page or local HTML file into the editor pane. You give it a URL, and it fetches and displays the content.

---

### Purposes

- To load content directly from a URL (web page, local file, or resource).
- To display HTML help files bundled with an application.
- To implement simple hyperlink navigation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
pane.setPage(new URL("https://example.com/page.html"));
pane.setPage("https://example.com/page.html");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setPage(URL)` | Loads content from the URL. |
| `setPage(String)` | Resolves the string as a URL and loads it. |

#### Syntax Rules

- The URL must be resolvable and accessible.
- The content type is determined from the URL (e.g., `.html` → `text/html`).
- Throws `IOException` if the URL cannot be read.

#### Constraints and Limitations

- Relative references in HTML (images, links) require a `<base>` tag or the `Base` property to be set.
- Network access may block the EDT; use a `SwingWorker` for remote URLs.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Loading HTML from a URL

```java
import javax.swing.*;
import java.io.IOException;
import java.net.URL;

public class SetPageDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setPage Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane();
            pane.setEditable(false);                                // 1. Read-only

            try {
                URL url = new URL("https://example.com");            // 2. URL
                pane.setPage(url);                                   // 3. Load
            } catch (IOException e) {
                pane.setText("Could not load page: " + e.getMessage());
            }

            frame.add(new JScrollPane(pane));
            frame.setSize(600, 400);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The pane loads and displays the content of `https://example.com` (or an error message if the page cannot be loaded).

**Why the code produces this result.** `setPage` fetches the URL content and uses the appropriate `EditorKit` based on the URL's content type.

---

### Real-World Cases

- **Help viewers** that load HTML help files from the application's resources.
- **Simple browsers** that navigate between linked pages.

---

## 5. Editor Pane Methods — getText() / setText()

### Definitions

**Core Definition.** `getText()` returns the current content of the editor pane as a `String`. `setText(String t)` sets the content, parsing it according to the current content type.

**Technical Definition.** `getText()` is inherited from `JTextComponent`. `setText(String t)` removes the contents of the current document and replaces them by parsing the given string using the current `EditorKit`. The assumption is that the previous content is relatively small and does not have side effects. For large content or content with lingering state, creating a new document via `getEditorKit().createDefaultDocument()` and replacing it is recommended.

**Beginner-Friendly Explanation.** `getText` reads what is in the pane; `setText` puts content into it. When the content type is HTML, the string you pass to `setText` should be HTML.

---

### Purposes

- To read the current content for saving or processing.
- To display content from a string (as opposed to a URL).
- To replace the pane's content dynamically.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
String content = pane.getText();
pane.setText("<html><body>New content</body></html>");
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getText()` | Returns the current content as a `String`. |
| `setText(String)` | Sets the content, parsed according to the current content type. |

#### Syntax Rules

- The string passed to `setText` must be in the format of the current content type.
- For large content, the recommendation is to create a new `Document` rather than reusing the existing one.

#### Constraints and Limitations

- `setText` may not cleanly reset the document's state for some formats (e.g., HTML styles).
- For very large content, `setText` on the EDT can freeze the GUI.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Getting and Setting Content

```java
import javax.swing.*;

public class GetSetTextDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("getText/setText");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane("text/html",
                "<html><body><p>Original content</p></body></html>");
            pane.setEditable(false);

            JButton readButton = new JButton("Read");
            readButton.addActionListener(e ->
                System.out.println("Content: " + pane.getText()));   // 1. Get

            JButton setButton = new JButton("Set New");
            setButton.addActionListener(e ->
                pane.setText("<html><body><h2>New content</h2></body></html>")); // 2. Set

            JPanel panel = new JPanel(new BorderLayout());
            panel.add(new JScrollPane(pane), BorderLayout.CENTER);
            JPanel south = new JPanel();
            south.add(readButton);
            south.add(setButton);
            panel.add(south, BorderLayout.SOUTH);

            frame.add(panel);
            frame.setSize(450, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Read” prints the HTML source string to the console. Clicking “Set New” replaces the pane's content with a new heading.

**Why the code produces this result.** `getText` returns the raw HTML string. `setText` parses the new HTML and renders it.

---

### Real-World Cases

- **Saving edited content:** Retrieving the HTML string to save to a file.
- **Loading content from a string:** Displaying generated HTML reports.

---

## 6. Editor Pane Methods — setEditable()

### Definitions

**Core Definition.** `setEditable(boolean b)` controls whether the user can edit the content of the editor pane.

**Technical Definition.** `public void setEditable(boolean b)` is inherited from `JTextComponent`. When set to `false`, the user cannot modify the content, but can still select and copy it. Additionally, when the content type is HTML and the pane is **not editable**, the `HTMLEditorKit` generates hyperlink events, enabling hyperlink interaction.

**Beginner-Friendly Explanation.** `setEditable(false)` makes the pane read-only. For HTML content, this also enables clickable hyperlinks.

---

### Purposes

- To make the pane read-only for display purposes.
- To enable hyperlink navigation in HTML content.
- To prevent accidental modification of important content.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
pane.setEditable(false);   // read-only, hyperlinks enabled for HTML
pane.setEditable(true);    // editable
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setEditable(boolean)` | `false` makes the pane read-only; `true` allows editing. |

#### Syntax Rules

- For HTML hyperlinks to work, `setEditable(false)` must be called.
- A read-only pane still allows selection and copying.

#### Constraints and Limitations

- Hyperlink events are generated only by the HTML `EditorKit` when the pane is not editable.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Read-Only HTML Pane with Hyperlinks

```java
import javax.swing.*;
import javax.swing.event.*;
import java.awt.*;

public class HyperlinkDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Hyperlinks");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane("text/html",
                "<html><body>"
                + "<p>Visit <a href='https://example.com'>Example</a></p>"
                + "</body></html>");
            pane.setEditable(false);                                // 1. Enable links

            pane.addHyperlinkListener(e -> {                        // 2. Listener
                if (e.getEventType() == HyperlinkEvent.EventType.ACTIVATED) {
                    try {
                        pane.setPage(e.getURL());                    // 3. Navigate
                    } catch (Exception ex) {
                        ex.printStackTrace();
                    }
                }
            });

            frame.add(new JScrollPane(pane));
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The pane displays a clickable “Example” link. Clicking it attempts to load `https://example.com`.

**Why the code produces this result.** `setEditable(false)` enables the HTML `EditorKit` to generate hyperlink events. The `HyperlinkListener` handles the activation and navigates via `setPage`.

---

### Real-World Cases

- **Help systems** with clickable links between topics.
- **Document viewers** that allow navigation but not editing.

---

## 7. JEditorPane Best Practices

### Definitions

**Core Definition.** `JEditorPane` best practices are the conventions and patterns that ensure editor panes are used effectively, thread-safely, and performantly in Swing applications.

**Technical Definition.** Best practices include: always using `JScrollPane` to provide scrolling; setting the correct content type before loading content; calling `setEditable(false)` for read-only HTML to enable hyperlinks; loading large content off the EDT using `SwingWorker` (or creating a new `Document` and replacing it); using layout managers instead of `setBounds`; and disposing of unused resources.

**Beginner-Friendly Explanation.** Best practices for editor panes are the “rules of thumb” that prevent frozen GUIs, missing scroll bars, and incorrect rendering. They include things like “always put it in a scroll pane,” “set the content type first,” “load big content in the background,” and “don't use absolute positioning.”

---

### Purposes

- To ensure content scrolls correctly and is properly sized.
- To avoid EDT freezes when loading large documents.
- To enable hyperlink interaction for HTML content.
- To maintain correct rendering by setting the content type before content.
- To keep the GUI responsive and maintainable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JEditorPane pane = new JEditorPane();                         // 1. Create
    pane.setContentType("text/html");                             // 2. Type first
    pane.setEditable(false);                                      // 3. Read-only

    JScrollPane scroll = new JScrollPane(pane);                    // 4. Scroll pane

    new SwingWorker<String, Void>() {                             // 5. Background load
        @Override
        protected String doInBackground() throws Exception {
            return loadHtmlFromSource();                          // long task
        }
        @Override
        protected void done() {
            try {
                pane.setText(get());                              // 6. EDT update
            } catch (Exception ex) {
                pane.setText("Error: " + ex.getMessage());
            }
        }
    }.execute();
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use a scroll pane | `new JScrollPane(editorPane)` |
| Set content type before text | `setContentType(...)` then `setText(...)` |
| Read-only for hyperlinks | `setEditable(false)` |
| Background loading | `SwingWorker` for large or remote content |
| Replace document for large content | `getEditorKit().createDefaultDocument()` and `setDocument` |
| Avoid `setBounds` | Use layout managers. |
| EDT | `SwingUtilities.invokeLater(...)` |

#### Syntax Rules

- **Always wrap `JEditorPane` in a `JScrollPane`.** The pane does not provide scrolling on its own.
- **Always set the content type before loading content.** Otherwise, the content may be parsed as plain text.
- **Always call `setEditable(false)` for read-only HTML** to enable hyperlink events.
- **Never perform long-running content loading on the EDT.** Use `SwingWorker` or a background thread.
- **Never use `setBounds`** on the editor pane or its scroll pane; use layout managers.

#### Constraints and Limitations

- The HTML support is limited to HTML 3.2; complex modern web pages may not render correctly.
- `setText` on very large content can be slow; consider replacing the `Document` instead.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice HTML Viewer with Background Loading

```java
import javax.swing.*;
import java.awt.*;
import java.net.URL;

public class BestPracticeEditorPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeEditorPane::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice EditorPane");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JEditorPane pane = new JEditorPane();
        pane.setContentType("text/html");                             // 1. Type
        pane.setEditable(false);                                      // 2. Read-only

        JScrollPane scroll = new JScrollPane(pane);                    // 3. Scroll
        frame.add(scroll, BorderLayout.CENTER);

        new SwingWorker<String, Void>() {                             // 4. Background
            @Override
            protected String doInBackground() throws Exception {
                URL url = new URL("https://example.com");
                return new String(url.openStream().readAllBytes());
            }
            @Override
            protected void done() {
                try {
                    pane.setText(get());                              // 5. EDT update
                } catch (Exception ex) {
                    pane.setText("Failed to load: " + ex.getMessage());
                }
            }
        }.execute();

        frame.setSize(700, 500);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** The window displays the HTML content of the URL after it loads in the background. The UI remains responsive during the load.

**Why the code produces this result.** The `SwingWorker` fetches the content off the EDT and applies it in `done()`, which runs on the EDT. The content type is set before the text is applied, and the pane is read-only.

---

#### Example 2: Common Mistake — Blocking the EDT with `setText`

```java
import javax.swing.*;

public class BadEditorPane {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad EditorPane");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JEditorPane pane = new JEditorPane();
            pane.setContentType("text/html");

            // WRONG: loading large content on the EDT
            StringBuilder huge = new StringBuilder("<html><body>");
            for (int i = 0; i < 100000; i++) {
                huge.append("<p>Line ").append(i).append("</p>");
            }
            huge.append("</body></html>");
            pane.setText(huge.toString());                            // 1. Freezes

            frame.add(new JScrollPane(pane));
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The application freezes while the large HTML string is parsed and rendered on the EDT.

**Why this is wrong.** Parsing and rendering a very large HTML document on the EDT blocks event processing, freezing the GUI. The correct approach is to create the content in a background thread and apply it via `setText` in `done()`, or to build a new `Document` and replace the existing one.

---

### Real-World Cases

- **Production Swing applications** use `SwingWorker` to load remote HTML help content without freezing the UI.
- **Help viewers** set `text/html` and `setEditable(false)` to enable hyperlink navigation.
- **Documentation tools** replace the `Document` for large content to avoid lingering state.

---

## References

Oracle. *How to Use Editor Panes and Text Panes*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/editorpane.html

Oracle. *Class JEditorPane*. Java Platform, Standard Edition 11 API Specification. https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/JEditorPane.html

Oracle. *Class JEditorPane*. Java Platform, Standard Edition 7 API Specification. https://docs.oracle.com/javase/7/docs/api/javax/swing/JEditorPane.html

Stack Overflow. *Best practice for dealing with long running JEditorPane setText process*. https://stackoverflow.com/questions/25704978/best-practice-for-dealing-with-long-running-jeditorpane-settext-process

Stack Overflow. *JEditorPane setBounds and JScrollPane layout*. https://stackoverflow.com/revisions/d7937d3d-6c85-4f91-875d-ff4c919cc68f/view-source

Oracle. *JEditorPane thread safety and setText semantics*. https://people.csail.mit.edu/dfhuynh/research/javadoc/jdk1.5.0/javax/swing/JEditorPane.html