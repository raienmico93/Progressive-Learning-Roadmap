# JToolTip — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JToolTip` is a Swing component that displays a small “hint” popup when the mouse cursor rests over a component. It is the underlying class that renders the familiar tooltip text—the short description that appears after hovering over a button, an icon, or any other GUI element for a moment.

**Technical Definition.** `public class JToolTip extends JComponent implements Accessible`. `JToolTip` is used to display a “tip” for a component. Components generally provide APIs that automate the process of using tooltips; for example, any Swing component can use `JComponent`’s `setToolTipText` method to specify the text for a standard tooltip. Components that want to provide a custom tooltip display can override `JComponent`’s `createToolTip` method and use a subclass of `JToolTip`. The actual showing and hiding of tooltips is managed by the shared `ToolTipManager` singleton. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT). Serialized `JToolTip` objects are not compatible with future Swing releases.

**Beginner-Friendly Explanation.** `JToolTip` is the “little yellow box” that pops up when you hover your mouse over a button or icon in a desktop application. It tells you what the control does without requiring you to click it. Most of the time, you never need to touch `JToolTip` directly: you simply call `component.setToolTipText("Some helpful text")`, and Swing takes care of showing and hiding the tooltip for you. `JToolTip` is the class that does the actual rendering behind the scenes. If you want tooltips with custom fonts, colors, multi-line text, or even embedded components, you work with `JToolTip` (and its manager, `ToolTipManager`) directly.

---

### Key Characteristics

1. **Standardised via `JComponent`.** The primary API for tooltips lives in `JComponent`, not in `JToolTip` itself. Every Swing component except top-level containers inherits `setToolTipText` and `getToolTipText`.
2. **Manager-driven lifecycle.** Tooltips are shown and hidden by the `ToolTipManager` singleton, which listens for mouse events, waits for the initial delay, and dismisses the tooltip after the dismiss delay or when the mouse moves away.
3. **`createToolTip` factory.** When the `ToolTipManager` needs to display a tooltip, it calls the component’s `createToolTip()` method. By default, this returns a plain `JToolTip` whose text is set from `getToolTipText()`. Overriding `createToolTip()` is the standard way to supply a custom `JToolTip` subclass.
4. **Text can be HTML.** The tooltip text can contain HTML markup, enabling multi-line tooltips, bold/italic text, tables, and images without subclassing `JToolTip`.
5. **`getToolTipText(MouseEvent)` for context sensitivity.** `JComponent` provides an overload that accepts a `MouseEvent`, allowing multi-part components (such as `JTabbedPane`, `JTable`, and `JTree`) to return different tooltip text depending on the mouse position.
6. **`setToolTipText(null)` disables tooltips.** Passing `null` to `setToolTipText` turns off tooltips for that component and unregisters it from the `ToolTipManager`.
7. **`registerKeyboardAction` is obsolete.** The `registerKeyboardAction` method (inherited from `JComponent`) is documented as obsolete and should be replaced by `getActionMap()` and `getInputMap()`. It is not a tooltip-specific method and has no role in tooltip display.
8. **Single-thread rule.** `JToolTip` and all tooltip-related configuration must be handled on the EDT.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with `JComponent`, `JButton`, `JLabel`, and other Swing components.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `MouseListener` and `MouseEvent` for context-sensitive tooltips.
- Familiarity with `ActionMap` and `InputMap` (for replacing the obsolete `registerKeyboardAction`).
- A JDK (version 8 or later; `JToolTip` is available in all modern JDKs).

---

### Related Programming Areas

- **`ToolTipManager`:** The singleton that controls tooltip timing, enabling/disabling, and lightweight/heavyweight popup selection.
- **`JComponent.setToolTipText` / `getToolTipText`:** The primary API for standard tooltips.
- **`JComponent.createToolTip`:** The factory method for custom `JToolTip` subclasses.
- **`ActionMap` and `InputMap`:** The modern replacement for the obsolete `registerKeyboardAction`.
- **HTML in Swing:** The simplest way to create multi-line, styled tooltips without custom painting.
- **Look and Feel (L&F):** Controls the default appearance of `JToolTip` (font, background, border).

---

## Core Concepts and Features

---

## 1. JToolTip — The Class

### Definitions

**Core Definition.** `JToolTip` is the Swing class that renders a component’s hint as a small popup window.

**Technical Definition.** `public class JToolTip extends JComponent implements Accessible`. It provides a no-argument constructor `JToolTip()` and methods `setTipText(String)`, `getTipText()`, `setComponent(JComponent)`, and `getComponent()`. The component it describes is set by the `ToolTipManager` before the tooltip is shown.

**Beginner-Friendly Explanation.** `JToolTip` is the class behind the tooltip popup. Most developers interact with it indirectly via `setToolTipText`, but when they need custom rendering or behavior, they subclass `JToolTip` and override `createToolTip` on the component.

---

### Purposes

- To provide a standard mechanism for displaying short contextual help text.
- To serve as the rendering component for the `ToolTipManager`'s popup.
- To allow customisation of tooltip appearance and content through subclassing.
- To support HTML-formatted tooltips for rich text and multi-line display.
- To integrate with `JComponent`'s `getToolTipText(MouseEvent)` for context-sensitive tooltips.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JToolTip tip = new JToolTip();
tip.setTipText("This is a custom tooltip.");
tip.setComponent(component);
```

#### Component Breakdown

| Constructor / Method | Meaning |
|-----------------------|---------|
| `JToolTip()` | Creates a tooltip with no text. |
| `setTipText(String)` | Sets the text displayed by the tooltip. |
| `getTipText()` | Returns the current tooltip text. |
| `setComponent(JComponent)` | Sets the component the tooltip describes (usually set by the manager). |
| `getComponent()` | Returns the component the tooltip describes. |

#### Syntax Rules

- Direct instantiation is rare; tooltips are normally created via `JComponent.createToolTip()`.
- The `component` property is typically set by the `ToolTipManager` before display.
- All construction and manipulation must occur on the EDT.

#### Constraints and Limitations

- `JToolTip` does not manage its own visibility; the `ToolTipManager` handles show/hide timing.
- The default tooltip is a single line of text unless HTML is used or `getTipText` is overridden.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Standard Tooltip via setToolTipText

```java
import javax.swing.*;
import java.awt.*;

public class StandardToolTip {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Standard ToolTip");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Hover Me");
            button.setToolTipText("This button does something important."); // 1. Set tooltip

            frame.add(button, BorderLayout.CENTER);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Hover Me” button. When the mouse hovers over the button for the default initial delay, a small tooltip appears with the text “This button does something important.”

**Why the code produces this result.** `setToolTipText` registers the component with the `ToolTipManager`. When the mouse enters and rests, the manager calls `createToolTip()` on the button (which returns a plain `JToolTip`), sets its text, and displays it near the cursor.

---

### Real-World Cases

- **Toolbars** where icon-only buttons need textual explanations.
- **Form fields** where tooltips explain expected input format.
- **Data tables and trees** where cell-specific tooltips show full content or metadata.
- **Disabled controls** where tooltips explain why an option is unavailable.

---

## 2. Tool Tip Creation

### Definitions

**Core Definition.** Tooltip creation is the process of associating tooltip text with a component, either through the standard `setToolTipText` API or by supplying a custom `JToolTip` via `createToolTip`.

**Technical Definition.** The standard path is `JComponent.setToolTipText(String)`, which registers the component with the `ToolTipManager` and stores the text. When the manager decides to show a tooltip, it invokes `component.createToolTip()`. The default implementation returns a plain `JToolTip` whose text is set from `getToolTipText()`. For custom tooltips, override `createToolTip()` to return a `JToolTip` subclass.

**Beginner-Friendly Explanation.** Creating a tooltip means telling a component what text to show on hover. In the simplest case, you call `setToolTipText`. For custom look or behavior, you override `createToolTip` and return your own `JToolTip` subclass.

---

### Purposes

- To attach contextual help to a component.
- To register the component with the `ToolTipManager` for automatic show/hide behavior.
- To enable custom tooltip rendering through subclassing.
- To provide HTML-rich or multi-line tooltips.

---

### Syntax Rules and Structure

#### Complete General Syntax (Standard and Custom)

```java
component.setToolTipText("Standard tooltip text");

// Custom tooltip:
JButton custom = new JButton("Custom") {
    @Override
    public JToolTip createToolTip() {
        return new JToolTip() {
            @Override
            public String getTipText() {
                return "Custom tooltip via createToolTip";
            }
        };
    }
};
custom.setToolTipText("dummy"); // still required to register
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `setToolTipText(String)` | Registers the component and sets the text. |
| `createToolTip()` | Factory method returning a `JToolTip` instance. |
| `getTipText()` | Returns the text for a custom tooltip. |

#### Syntax Rules

- `setToolTipText` is mandatory for the `ToolTipManager` to register the component; even custom tooltips require a non-null value to enable tooltip display.
- `createToolTip` is called every time the tooltip is about to be shown; return a new instance or a cached one as appropriate.
- Custom `JToolTip` subclasses should override `getTipText()` if they need dynamic text.

#### Constraints and Limitations

- Without `setToolTipText`, the `ToolTipManager` does not know the component has a tooltip.
- Custom `JToolTip` instances must still be lightweight `JComponent` subclasses.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Custom JToolTip Subclass

```java
import javax.swing.*;
import java.awt.*;

public class CustomToolTip {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Custom ToolTip");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Hover for custom tip") {
                @Override
                public JToolTip createToolTip() {                   // 1. Override
                    JToolTip tip = new JToolTip() {
                        @Override
                        public String getTipText() {                 // 2. Dynamic text
                            return "Custom tooltip with HTML: <b>Bold</b> text";
                        }
                    };
                    tip.setFont(new Font("SansSerif", Font.PLAIN, 14));
                    tip.setBackground(Color.YELLOW);
                    return tip;
                }
            };
            button.setToolTipText("enabled");                        // 3. Register

            frame.add(button);
            frame.setSize(350, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Hovering over the button shows a yellow-background tooltip with the bolded text “Bold” rendered from HTML.

**Why the code produces this result.** The anonymous `JButton` subclass overrides `createToolTip()` to return a custom `JToolTip` whose `getTipText()` supplies HTML text. `setToolTipText("enabled")` registers the button with the manager. The custom font and background are applied to the tooltip instance.

---

### Real-World Cases

- **Custom-themed tooltips** that match an application’s branding.
- **Tooltips with embedded components** (e.g., a mini chart or image).
- **Multi-line tooltips** using either HTML or a custom `JMultiLineToolTip` subclass.

---

## 3. Tool Tip Methods — setToolTipText()

### Definitions

**Core Definition.** `setToolTipText(String text)` is a `JComponent` method that registers the component with the `ToolTipManager` and specifies the text to display when the mouse rests over the component.

**Technical Definition.** `public void setToolTipText(String text)` is inherited from `JComponent`. If the string is non-`null`, the method registers the component as having a tooltip and makes the tooltip display the specified text. If the argument is `null`, the method turns off tooltips for the component.

**Beginner-Friendly Explanation.** `setToolTipText` is the one method you need for standard tooltips. Give it a string, and the component will show that string on hover. Give it `null`, and tooltips are disabled for that component.

---

### Purposes

- To enable tooltips for a component.
- To set or change the tooltip text at runtime.
- To disable tooltips by passing `null`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
component.setToolTipText("Helpful text");
component.setToolTipText(null); // disable
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `String text` | The tooltip text, or `null` to disable. |

#### Syntax Rules

- Must be called on the EDT.
- The component is automatically registered with the `ToolTipManager` when the text is non-`null`.
- The text may contain HTML markup.

#### Constraints and Limitations

- The tooltip text is a single string; for multiple lines without HTML, use a custom `JToolTip` subclass.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting and Clearing Tooltip Text

```java
import javax.swing.*;

public class SetToolTipTextDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("setToolTipText");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click");
            button.setToolTipText("This is a standard tooltip.");     // 1. Set

            JButton clearButton = new JButton("Remove Tooltip");
            clearButton.addActionListener(e ->
                button.setToolTipText(null));                          // 2. Clear

            JPanel panel = new JPanel();
            panel.add(button);
            panel.add(clearButton);

            frame.add(panel);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Hovering over “Click” shows the tooltip. Clicking “Remove Tooltip” disables the tooltip, and subsequent hovers show nothing.

**Why the code produces this result.** `setToolTipText` updates the component’s tooltip registration with the manager. Passing `null` unregisters the tooltip.

---

### Real-World Cases

- **Dynamic help text** that changes based on application state.
- **Disabling tooltips** for components where they would be distracting.

---

## 4. Tool Tip Methods — getToolTipText()

### Definitions

**Core Definition.** `getToolTipText()` returns the tooltip text currently associated with a component. The overload `getToolTipText(MouseEvent)` allows context-sensitive tooltips based on the mouse position.

**Technical Definition.** `public String getToolTipText()` returns the string previously set with `setToolTipText`. `public String getToolTipText(MouseEvent event)` by default returns the same value, but multi-part components such as `JTabbedPane`, `JTable`, and `JTree` override it to return text associated with the mouse event location.

**Beginner-Friendly Explanation.** `getToolTipText()` tells you what tooltip text a component currently has. The `MouseEvent` version lets a single component show different tooltips depending on where the mouse is.

---

### Purposes

- To read the current tooltip text.
- To implement context-sensitive tooltips in multi-part components.
- To support custom `JToolTip` subclasses that override `getTipText`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
String text = component.getToolTipText();
String contextText = component.getToolTipText(mouseEvent);
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getToolTipText()` | Returns the text set by `setToolTipText`. |
| `getToolTipText(MouseEvent)` | Returns context-dependent text; default returns `getToolTipText()`. |

#### Syntax Rules

- The no-argument version returns `null` if no tooltip is set.
- The `MouseEvent` version is overridden by scrolling-savvy and multi-part components.

#### Constraints and Limitations

- The base `JComponent` implementation of `getToolTipText(MouseEvent)` ignores the event and simply delegates to `getToolTipText()`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Context-Sensitive Tooltip on a JTable

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.event.MouseEvent;

public class ContextToolTip {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Context ToolTip");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultTableModel model = new DefaultTableModel(
                new Object[][]{{"Alice", 30}, {"Bob", 25}},
                new Object[]{"Name", "Age"});

            JTable table = new JTable(model) {
                @Override
                public String getToolTipText(MouseEvent e) {         // 1. Override
                    int row = rowAtPoint(e.getPoint());
                    int col = columnAtPoint(e.getPoint());
                    if (row >= 0 && col >= 0) {
                        return "Cell [" + row + "," + col + "]: "
                            + getValueAt(row, col);
                    }
                    return null;
                }
            };
            table.setToolTipText("");                                 // 2. Register

            frame.add(new JScrollPane(table));
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Hovering over a cell in the table shows a tooltip with the cell’s row, column, and value. Hovering outside a cell shows no tooltip.

**Why the code produces this result.** The overridden `getToolTipText(MouseEvent)` uses `rowAtPoint` and `columnAtPoint` to determine the cell under the cursor and returns a cell-specific string. `setToolTipText("")` registers the table with the manager.

---

### Real-World Cases

- **Tables and trees** where each cell or node has its own tooltip.
- **Tabbed panes** where each tab shows a different tooltip.
- **Multi-part components** that need position-dependent help text.

---

## 5. Tool Tip Methods — registerKeyboardAction() (Obsolete)

### Definitions

**Core Definition.** `registerKeyboardAction` is a legacy `JComponent` method for binding a keyboard action to a component. It is **not** a tooltip-specific method; it is mentioned here because it is sometimes confused with tooltip functionality.

**Technical Definition.** `public void registerKeyboardAction(ActionListener anAction, String aCommand, KeyStroke aKeyStroke, int aCondition)` is documented as **obsolete**. The official guidance is to use a combination of `getActionMap()` and `getInputMap()` for similar behavior. The `aCondition` constants include `WHEN_FOCUSED`, `WHEN_IN_FOCUSED_WINDOW`, and `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`.

**Beginner-Friendly Explanation.** `registerKeyboardAction` is an old way to make a keyboard shortcut trigger an action. It has nothing to do with tooltips. Modern Swing code should use `InputMap` and `ActionMap` instead.

---

### Purposes

- **Historical only:** To bind a keyboard shortcut to an action in older Swing code.
- **Migration reference:** To understand and replace legacy code that uses this obsolete method.

---

### Syntax Rules and Structure

#### Complete General Syntax (Obsolete)

```java
component.registerKeyboardAction(actionListener,
    "command", keyStroke, JComponent.WHEN_FOCUSED);
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `ActionListener` | The action to invoke. |
| `String aCommand` | The command string (passed in the event). |
| `KeyStroke aKeyStroke` | The key combination. |
| `int aCondition` | Focus condition (`WHEN_FOCUSED`, etc.). |

#### Syntax Rules

- **This method is obsolete.** Use `getInputMap().put(keyStroke, command)` and `getActionMap().put(command, action)` instead.
- The `aCondition` constants determine when the binding is active.

#### Constraints and Limitations

- The method is not annotated with `@Deprecated` but is documented as obsolete and may be removed in future releases.
- It has **no relationship** to tooltip display.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Modern Replacement Using InputMap and ActionMap

```java
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.KeyEvent;

public class KeyboardBindingReplacement {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Keyboard Binding");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Press Ctrl+Enter");
            Action action = new AbstractAction() {                   // 1. Action
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Ctrl+Enter pressed!");
                }
            };

            KeyStroke ks = KeyStroke.getKeyStroke(
                KeyEvent.VK_ENTER, java.awt.event.InputEvent.CTRL_DOWN_MASK);
            button.getInputMap(JComponent.WHEN_FOCUSED).put(ks, "doIt"); // 2. InputMap
            button.getActionMap().put("doIt", action);                    // 3. ActionMap

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the button has focus, pressing Ctrl+Enter prints “Ctrl+Enter pressed!” to the console.

**Why the code produces this result.** `InputMap` binds the keystroke to the command name `"doIt"`, and `ActionMap` maps that command name to the `Action`. This is the modern replacement for `registerKeyboardAction`.

---

### Real-World Cases

- **Migrating legacy Swing code** that still uses `registerKeyboardAction`.
- **Understanding old tutorials** that predate the `InputMap`/`ActionMap` API.

---

## 6. JToolTip Best Practices

### Definitions

**Core Definition.** `JToolTip` best practices are the conventions and patterns that ensure tooltips are implemented safely, accessibly, and with good user experience.

**Technical Definition.** Best practices include: using `setToolTipText` for standard tooltips; overriding `createToolTip` for custom `JToolTip` subclasses; using HTML for multi-line or styled text; configuring `ToolTipManager` for timing and enabling/disabling tooltips; avoiding the obsolete `registerKeyboardAction`; and ensuring tooltip text is concise, descriptive, and accessible.

**Beginner-Friendly Explanation.** Best practices for tooltips are the “rules of thumb” that make tooltips helpful rather than annoying: keep them short and clear, use them consistently, make sure they are readable by screen readers, and don’t clutter the UI with too many of them.

---

### Purposes

- To provide helpful, concise contextual information.
- To ensure tooltips are accessible to assistive technologies.
- To maintain consistent timing and appearance across the application.
- To use the correct API (standard vs. custom) for the requirement.
- To avoid obsolete methods and threading pitfalls.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JButton button = new JButton("Save");
    button.setToolTipText("Save the current file (Ctrl+S)");      // 1. Standard

    // For custom tooltips:
    JButton custom = new JButton("Custom") {
        @Override
        public JToolTip createToolTip() {                           // 2. Custom
            JToolTip tip = new JToolTip() {
                @Override
                public String getTipText() {
                    return "<html><b>Custom</b> tooltip</html>";
                }
            };
            tip.setBackground(Color.LIGHT_GRAY);
            return tip;
        }
    };
    custom.setToolTipText("");                                      // 3. Register
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Standard tooltip | `setToolTipText("...")` |
| Custom tooltip | Override `createToolTip()` and return a `JToolTip` subclass. |
| Registration | Always call `setToolTipText` (even for custom tooltips). |
| Multi-line / rich text | Use HTML markup in the tooltip string. |
| Timing control | `ToolTipManager.sharedInstance().setInitialDelay(...)`, `setDismissDelay(...)` |
| Enabling/disabling | `ToolTipManager.sharedInstance().setEnabled(false)` |
| Accessibility | Provide descriptive text; test with screen readers. |
| Avoid obsolete API | Use `InputMap`/`ActionMap`, not `registerKeyboardAction`. |

#### Syntax Rules

- **Keep tooltips concise but descriptive**—short enough to read quickly, specific enough to be useful.
- **Use simple language** and maintain consistent styling across all tooltips.
- **Ensure tooltips are accessible**; screen readers can read tooltip text, so keep it meaningful.
- **Do not use `registerKeyboardAction`**; use `InputMap` and `ActionMap` for keyboard bindings.
- **Configure `ToolTipManager` globally** for timing preferences (initial delay, dismiss delay, re-show delay).

#### Constraints and Limitations

- Tooltips are transient; they should not contain critical information that the user must act upon.
- Very long tooltips may be truncated or overwhelm the user; use HTML line breaks or a custom multi-line tooltip.
- `setToolTipText(null)` disables tooltips for a component.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Tooltip with ToolTipManager Configuration

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeToolTip {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeToolTip::createAndShow);
    }

    private static void createAndShow() {
        // 1. Configure global tooltip timing
        ToolTipManager.sharedInstance().setInitialDelay(500);      // 0.5s
        ToolTipManager.sharedInstance().setDismissDelay(4000);     // 4s

        JFrame frame = new JFrame("Best Practice ToolTip");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JButton save = new JButton("Save");
        save.setToolTipText("Save the current file (Ctrl+S)");     // 2. Concise text

        JTextField email = new JTextField(20);
        email.setToolTipText("Enter your email address (user@example.com)"); // 3. Descriptive

        JButton custom = new JButton("Custom") {
            @Override
            public JToolTip createToolTip() {                      // 4. Custom tooltip
                JToolTip tip = new JToolTip() {
                    @Override
                    public String getTipText() {
                        return "<html><b>Custom</b> tooltip with HTML</html>";
                    }
                };
                tip.setBackground(new Color(255, 255, 200));
                tip.setForeground(Color.DARK_GRAY);
                tip.setFont(new Font("SansSerif", Font.PLAIN, 13));
                return tip;
            }
        };
        custom.setToolTipText("");                                  // 5. Register

        JPanel panel = new JPanel(new GridLayout(3, 1, 5, 5));
        panel.add(save);
        panel.add(email);
        panel.add(custom);

        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Three components with tooltips. The “Save” and email field show standard tooltips after a 0.5-second delay and dismiss after 4 seconds. The “Custom” button shows an HTML-formatted tooltip with a light-yellow background and dark-gray text.

**Why the code produces this result.** `ToolTipManager` is configured globally for timing. Standard tooltips are set via `setToolTipText`. The custom button overrides `createToolTip` to return a `JToolTip` subclass with custom text, background, foreground, and font. `setToolTipText("")` registers the custom button with the manager.

---

#### Example 2: Common Mistake — Using `registerKeyboardAction` for Tooltip Activation

```java
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class BadKeyboardAction {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Keyboard Action");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Button");
            // WRONG: registerKeyboardAction is obsolete
            button.registerKeyboardAction(                            // 1. Obsolete
                new ActionListener() {
                    public void actionPerformed(ActionEvent e) {
                        System.out.println("Action triggered");
                    }
                },
                KeyStroke.getKeyStroke("control ENTER"),
                JComponent.WHEN_FOCUSED);

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The keyboard binding may work, but the code uses an obsolete API that is documented as subject to removal and is not the recommended approach.

**Why this is wrong.** `registerKeyboardAction` is obsolete and should be replaced by `getInputMap()` and `getActionMap()`. The modern approach is clearer, more flexible, and not subject to removal.

---

### Real-World Cases

- **Production Swing applications** use `setToolTipText` for standard tooltips, HTML for multi-line text, and `createToolTip` overrides for custom rendering.
- **Accessibility-focused applications** ensure every tooltip has descriptive text and test with screen readers.
- **Educational examples** use the best-practice skeleton to demonstrate correct tooltip usage and avoid obsolete APIs.

---

## References

Oracle. *Class JToolTip*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/jp/8/docs/api/javax/swing/JToolTip.html

Oracle. *How to Use Tool Tips*. The Java Tutorials. http://www.cs.cmu.edu/afs/cs.cmu.edu/user/gchen/www/download/java/JavaTutorial/ui/swing/tooltip.html

Oracle. *Class JComponent — registerKeyboardAction*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *ToolTipManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ToolTipManager.html

Stack Overflow. *Custom tooltip via createToolTip override*. https://stackoverflow.com/questions/ (multiple revisions discussing `createToolTip` and `JToolTip` subclassing)

Stack Overflow. *registerKeyboardAction is obsolete*. https://stackoverflow.com/questions/ (discussion of `InputMap`/`ActionMap` replacement)

ProjectAI. *Implementing Tooltips for Validation in Java Swing — Best Practices*. https://projectai.in/projects/...

Stack Overflow. *Make all tooltips multiline in Java (Swing)*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/9932617/make-all-tooltips-multiline-in-java-swing