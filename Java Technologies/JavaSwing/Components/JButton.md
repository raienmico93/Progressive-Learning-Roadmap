# JButton — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JButton` is an implementation of a “push” button in Swing. It is a component that the user can click to trigger an action, and it can be configured with text, an icon, or both.

**Technical Definition.** `public class JButton extends AbstractButton implements Accessible`. `JButton` inherits most of its functionality from `AbstractButton`, including support for mnemonics, tooltips, icons, and action listeners. Buttons can be configured and controlled by `Action` objects, which provide benefits beyond directly configuring a button. Swing is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `JButton` is the clickable button you see in almost every application—the “OK,” “Cancel,” “Save,” or “Submit” button. You create it with a label (like “Click Me”), optionally add an icon, and then tell it what to do when clicked by attaching an `ActionListener`. It is the primary way for users to trigger commands in a Swing application.

---

### Key Characteristics

1. **Push button semantics.** `JButton` represents a push button that can be “pressed” and “released” to trigger an action.
2. **Text and icon support.** A button can display text, an icon, or both. The relative position of text and icon can be controlled.
3. **Mnemonic support.** A keyboard mnemonic (typically used with the Alt key) can be assigned to activate the button without a mouse.
4. **Tooltip support.** A tooltip can be displayed when the user hovers over the button, providing additional information.
5. **Enable/disable state.** Buttons can be enabled or disabled. A disabled button does not respond to clicks and is typically rendered in a grayed-out state.
6. **Action architecture.** Buttons can be configured with `Action` objects, which centralize text, icon, mnemonic, and enabled state, and keep the button synchronized with the action.
7. **Single-thread rule and serialization caveat.** `JButton` is not thread-safe and must be accessed on the EDT. Serialized instances are not compatible with future Swing releases.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, interfaces, and lambda expressions.
- Familiarity with Swing containers such as `JFrame`, `JPanel`, and layout managers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `ActionListener` and the `java.awt.event` package.
- A JDK (version 8 or later; `JButton` is available in all modern JDKs).

---

### Related Programming Areas

- **`AbstractButton`:** The superclass of `JButton`; provides most of the button API.
- **`Action` and `AbstractAction`:** Encapsulate button functionality and state.
- **`ActionListener`:** The event interface for button clicks.
- **`JToolBar` and `JMenuBar`:** Common containers for buttons.
- **`JOptionPane`:** Uses buttons internally for standard dialogs.

---

## Core Concepts and Features

---

## 1. JButton — The Class

### Definitions

**Core Definition.** `JButton` is the Swing class for creating push buttons that trigger actions when clicked.

**Technical Definition.** `public class JButton extends AbstractButton implements Accessible`. It provides constructors that accept text, an icon, or both. The button’s appearance and behavior are determined by the current look and feel.

**Beginner-Friendly Explanation.** `JButton` is the class you use to create a button. You create it with `new JButton("Label")`, customize it with methods like `setMnemonic` and `setToolTipText`, and attach an `ActionListener` to handle clicks.

---

### Purposes

- To provide a clickable control that triggers a specific action or command.
- To display text, an icon, or both as a visual affordance for the action.
- To support keyboard activation via mnemonics and accessible interaction.
- To act as a visual indicator of an available command, often with hover and pressed states.
- To serve as the primary interaction element in tool bars, dialogs, and forms.

---

### Syntax Rules and Structure

#### Complete General Syntax (Creation)

```java
JButton button = new JButton("Click Me");
JButton iconButton = new JButton(new ImageIcon("icon.png"));
JButton textAndIcon = new JButton("Save", new ImageIcon("save.png"));
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JButton(String text)` | Creates a button with the specified text. |
| `JButton(Icon icon)` | Creates a button with the specified icon. |
| `JButton(String text, Icon icon)` | Creates a button with both text and icon. |

#### Syntax Rules

- The button must be added to a container to be displayed.
- The button is enabled and visible by default.
- Construction must occur on the EDT.

#### Constraints and Limitations

- A button without text or icon will appear empty.
- The button’s preferred size is determined by its content and the layout manager.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JButton

```java
import javax.swing.*;
import java.awt.*;

public class MinimalButton {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal Button");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");               // 1. Create button
            button.addActionListener(e ->                          // 2. Attach listener
                System.out.println("Button clicked!"));

            frame.add(button, BorderLayout.CENTER);                 // 3. Add to frame
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Click Me” button. Clicking the button prints “Button clicked!” to the console.

**Why the code produces this result.** The button is created with the text “Click Me.” The `ActionListener` (a lambda) handles the click event. The button is added to the frame’s content pane and displayed.

---

#### Example 2: Button with Icon

```java
import javax.swing.*;
import java.awt.*;

public class IconButton {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Icon Button");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                32, 32, java.awt.image.BufferedImage.TYPE_INT_RGB));

            JButton button = new JButton("Save", icon);             // 1. Text + icon
            button.setToolTipText("Save the current file");          // 2. Tooltip

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button displaying both the text “Save” and a 32×32 icon. Hovering over the button shows the tooltip “Save the current file.”

**Why the code produces this result.** The constructor accepts both text and icon. `setToolTipText` adds the hover message.

---

### Real-World Cases

- **Dialog buttons:** “OK,” “Cancel,” and “Apply” buttons in dialogs.
- **Toolbar buttons:** Save, Open, Print, and Undo buttons in tool bars.
- **Form submission:** A “Submit” button that validates and processes form data.
- **Game controls:** Buttons for actions such as “Start,” “Pause,” and “Restart.”

---

## 2. Button Creation

### Definitions

**Core Definition.** Button creation is the process of instantiating a `JButton` with the desired text and/or icon, and preparing it for addition to a container.

**Technical Definition.** `JButton` provides three commonly used constructors: `JButton()`, `JButton(String text)`, `JButton(Icon icon)`, and `JButton(String text, Icon icon)`. After construction, the button can be customized with properties and methods before being added to a container.

**Beginner-Friendly Explanation.** Creating a button means choosing whether it should show text, an icon, or both. You then configure it—give it a tooltip, a mnemonic, enable or disable it—and add it to your window.

---

### Purposes

- To instantiate a button with the appropriate visual content.
- To prepare the button for event listener attachment.
- To establish the button’s initial state (enabled, visible, etc.).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JButton button = new JButton("Text");
JButton button = new JButton(icon);
JButton button = new JButton("Text", icon);
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `JButton()` | Creates a button with no text or icon. |
| `JButton(String)` | Creates a button with text. |
| `JButton(Icon)` | Creates a button with an icon. |
| `JButton(String, Icon)` | Creates a button with both. |

#### Syntax Rules

- The button is enabled and visible by default.
- The button’s text and icon can be changed later with `setText` and `setIcon`.

#### Constraints and Limitations

- An empty button has no visible content and may be difficult for users to identify.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating Buttons with Different Constructors

```java
import javax.swing.*;
import java.awt.*;

public class ButtonConstructors {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Button Constructors");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JButton textButton = new JButton("Text Only");          // 1. Text
            JButton iconButton = new JButton(new ImageIcon(          // 2. Icon
                new java.awt.image.BufferedImage(16, 16,
                    java.awt.image.BufferedImage.TYPE_INT_RGB)));
            JButton bothButton = new JButton("Both",                 // 3. Both
                new ImageIcon(new java.awt.image.BufferedImage(
                    16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB)));

            frame.add(textButton);
            frame.add(iconButton);
            frame.add(bothButton);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with three buttons: one with text only, one with an icon only, and one with both text and icon.

**Why the code produces this result.** Each constructor determines the initial content of the button.

---

### Real-World Cases

- **Creating toolbar buttons** with icons for compact interfaces.
- **Creating dialog buttons** with text for clarity.
- **Creating buttons with both** for accessibility and visual appeal.

---

## 3. Button Properties

### Definitions

**Core Definition.** Button properties are the configurable attributes that determine the button’s appearance and behavior: text, icon, mnemonic, tool tip, enabled state, and visible state.

**Technical Definition.** These properties are accessed and modified through the methods inherited from `AbstractButton` and `JComponent`. The text property is managed by `setText`/`getText`. The icon property is managed by `setIcon`/`getIcon`. The mnemonic is managed by `setMnemonic`. The tooltip is managed by `setToolTipText`. Enabled state is managed by `setEnabled`/`isEnabled`. Visible state is managed by `setVisible`/`isVisible`.

**Beginner-Friendly Explanation.** Button properties are the “settings” of a button: what it says, what picture it shows, what keyboard shortcut activates it, what help text appears when you hover, whether it can be clicked, and whether it is shown at all.

---

### Purposes

- To control the button’s visual content (text and icon).
- To provide keyboard accessibility (mnemonic).
- To offer additional information on hover (tool tip).
- To control whether the button can be interacted with (enabled).
- To control whether the button is displayed (visible).

---

### Syntax Rules and Structure

#### Complete General Syntax (Property Methods)

```java
button.setText("New Text");
button.setIcon(icon);
button.setMnemonic(KeyEvent.VK_S);
button.setToolTipText("Save the file");
button.setEnabled(false);
button.setVisible(false);
```

#### Component Breakdown

| Property | Setter | Getter |
|----------|--------|--------|
| Text | `setText(String)` | `getText()` |
| Icon | `setIcon(Icon)` | `getIcon()` |
| Mnemonic | `setMnemonic(int)` | `getMnemonic()` |
| Tool tip | `setToolTipText(String)` | `getToolTipText()` |
| Enabled | `setEnabled(boolean)` | `isEnabled()` |
| Visible | `setVisible(boolean)` | `isVisible()` |

#### Syntax Rules

- The mnemonic is a key code, typically from `KeyEvent` (e.g., `KeyEvent.VK_S`).
- The tooltip text can be `null` to remove the tooltip.
- A disabled button does not respond to clicks and is rendered in a disabled state.
- A hidden button is not displayed and does not occupy space in the layout.

#### Constraints and Limitations

- The mnemonic must correspond to a character in the button’s text (the first occurrence is underlined).
- The effect of `setVisible(false)` depends on the layout manager; some layout managers may still reserve space.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Configuring All Properties

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

public class ButtonProperties {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Button Properties");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JButton button = new JButton("Save");                   // 1. Text
            button.setMnemonic(KeyEvent.VK_S);                       // 2. Mnemonic
            button.setToolTipText("Save the current file");          // 3. Tooltip
            button.setEnabled(true);                                 // 4. Enabled
            button.setVisible(true);                                 // 5. Visible

            ImageIcon icon = new ImageIcon(new java.awt.image.BufferedImage(
                16, 16, java.awt.image.BufferedImage.TYPE_INT_RGB));
            button.setIcon(icon);                                    // 6. Icon

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A button labeled “Save” with an icon. Pressing Alt+S activates the button. Hovering shows the tooltip. The button is enabled and visible.

**Why the code produces this result.** Each property is set explicitly, configuring the button’s complete appearance and behavior.

---

#### Example 2: Disabling and Hiding a Button

```java
import javax.swing.*;
import java.awt.*;

public class DisableHideButton {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Disable/Hide");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JButton disableButton = new JButton("Disable Me");
            disableButton.setEnabled(false);                         // 1. Disabled

            JButton hideButton = new JButton("Hide Me");
            hideButton.setVisible(false);                            // 2. Hidden

            frame.add(disableButton);
            frame.add(hideButton);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The “Disable Me” button is grayed out and cannot be clicked. The “Hide Me” button does not appear.

**Why the code produces this result.** `setEnabled(false)` disables the button. `setVisible(false)` removes it from the display.

---

### Real-World Cases

- **Form validation:** Disable the “Submit” button until all required fields are filled.
- **Progressive disclosure:** Hide advanced options until the user requests them.
- **Accessibility:** Provide mnemonics and tooltips for every button.

---

## 4. Button Methods

### Definitions

**Core Definition.** Button methods are the API operations that allow programmatic control of the button’s state and behavior: `setText`, `setIcon`, `setMnemonic`, `setToolTipText`, `setEnabled`, `setVisible`, and `doClick`.

**Technical Definition.** These methods are defined in `AbstractButton`, `JButton`, or inherited from `JComponent` and `Component`. The `doClick()` method programmatically simulates a button click, firing an `ActionEvent` and invoking registered `ActionListener`s.

**Beginner-Friendly Explanation.** These are the methods you call to change a button after it has been created. You can rename it, change its icon, enable or disable it, show or hide it, or even “click” it from code.

---

### Purposes

- To dynamically update the button’s appearance and state.
- To simulate a click programmatically for testing or automation.
- To respond to application state changes (e.g., disabling a button during a long operation).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
button.setText("New Text");
button.setIcon(newIcon);
button.setMnemonic(KeyEvent.VK_N);
button.setToolTipText("New tooltip");
button.setEnabled(false);
button.setVisible(false);
button.doClick();               // simulate click
button.doClick(100);            // simulate click with 100ms press time
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setText(String)` | Changes the button’s text. |
| `setIcon(Icon)` | Changes the button’s icon. |
| `setMnemonic(int)` | Sets the keyboard mnemonic. |
| `setToolTipText(String)` | Sets the tooltip text. |
| `setEnabled(boolean)` | Enables or disables the button. |
| `setVisible(boolean)` | Shows or hides the button. |
| `doClick()` | Programmatically triggers an action event. |
| `doClick(int pressTime)` | Triggers an action event with a visible press effect. |

#### Syntax Rules

- `doClick()` fires the button’s `ActionEvent` as if the user had clicked it.
- `setVisible(false)` removes the button from the layout (behavior may vary by layout manager).
- All methods should be called on the EDT.

#### Constraints and Limitations

- `doClick()` does not generate mouse events, only action events.
- Disabling a button while it is being clicked may produce inconsistent results.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using `doClick()`

```java
import javax.swing.*;
import java.awt.*;

public class DoClickDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("doClick Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Auto Click");
            button.addActionListener(e ->
                System.out.println("Button action fired!"));          // 1. Listener

            JButton trigger = new JButton("Trigger doClick");
            trigger.addActionListener(e ->
                button.doClick());                                     // 2. doClick

            frame.setLayout(new FlowLayout());
            frame.add(button);
            frame.add(trigger);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Trigger doClick” programmatically clicks the “Auto Click” button, printing “Button action fired!” to the console.

**Why the code produces this result.** `doClick()` simulates a click on the target button, firing its `ActionEvent` and invoking its listener.

---

#### Example 2: Updating Button State

```java
import javax.swing.*;
import java.awt.*;

public class StateUpdateDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("State Update");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Original");
            JLabel status = new JLabel("Ready");

            JButton updateButton = new JButton("Update");
            updateButton.addActionListener(e -> {
                button.setText("Updated");                             // 1. Text
                button.setEnabled(false);                              // 2. Disable
                status.setText("Button updated and disabled.");
            });

            frame.setLayout(new FlowLayout());
            frame.add(button);
            frame.add(updateButton);
            frame.add(status);
            frame.setSize(450, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially, the button says “Original” and is enabled. Clicking “Update” changes its text to “Updated,” disables it, and updates the status label.

**Why the code produces this result.** The `ActionListener` on the update button calls `setText` and `setEnabled` on the target button.

---

### Real-World Cases

- **Progress dialogs:** Disable the “OK” button until the operation completes.
- **Wizard navigation:** Enable the “Next” button only when the current step is valid.
- **Testing:** Use `doClick()` to simulate user interaction in automated tests.
- **Accessibility:** Update the button’s tooltip when its state changes.

---

## 5. Button Events — ActionListener and actionPerformed

### Definitions

**Core Definition.** `ActionListener` is an interface for receiving action events. The `actionPerformed(ActionEvent e)` method is invoked when the user clicks a button (or presses Enter/Space while the button has focus).

**Technical Definition.** `public interface ActionListener extends EventListener` declares the single method `void actionPerformed(ActionEvent e)`. When registered with `button.addActionListener(listener)`, the listener is notified each time the button fires an action event. The `ActionEvent` contains information about the event, including the source component (`getSource()`).

**Beginner-Friendly Explanation.** An `ActionListener` is the “click handler” for a button. You create a listener object (often with a lambda) that contains the code you want to run when the button is clicked, and you attach it to the button with `addActionListener`.

---

### Purposes

- To specify the action that should occur when a button is clicked.
- To decouple the button’s visual representation from the code that implements its behavior.
- To allow multiple buttons to share the same listener and branch based on the event source.
- To support keyboard activation (Enter/Space) in addition to mouse clicks.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        // code to execute
    }
});

// or with a lambda:
button.addActionListener(e -> {
    // code to execute
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addActionListener(ActionListener)` | Registers a listener. |
| `actionPerformed(ActionEvent e)` | Called when the button is clicked. |
| `e.getSource()` | Returns the component that fired the event. |

#### Syntax Rules

- The listener is invoked on the EDT.
- Multiple listeners can be registered; they are called in the order of registration.
- A single listener can be shared across multiple buttons, using `getSource()` to distinguish them.

#### Constraints and Limitations

- The listener should perform short tasks; long-running operations should be offloaded to a background thread.
- The `ActionEvent` does not contain the button’s text; use `getSource()` and cast to `JButton` if needed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Anonymous Inner Class Listener

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class AnonymousListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Anonymous Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            button.addActionListener(new ActionListener() {            // 1. Anonymous
                @Override
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Button clicked!");
                }
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the button prints “Button clicked!” to the console.

**Why the code produces this result.** The anonymous `ActionListener` is registered with the button. When clicked, `actionPerformed` is invoked.

---

#### Example 2: Shared Listener with `getSource()`

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class SharedListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            ActionListener shared = e -> {                              // 1. Shared
                JButton source = (JButton) e.getSource();
                System.out.println("Clicked: " + source.getText());
            };

            JButton button1 = new JButton("One");
            JButton button2 = new JButton("Two");
            button1.addActionListener(shared);
            button2.addActionListener(shared);

            frame.add(button1);
            frame.add(button2);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “One” prints “Clicked: One”; clicking “Two” prints “Clicked: Two.”

**Why the code produces this result.** Both buttons share the same listener. `e.getSource()` identifies which button was clicked, and the button’s text is retrieved.

---

### Real-World Cases

- **Dialog confirmation:** An “OK” button listener that closes the dialog and returns a result.
- **Form submission:** A “Submit” button listener that validates and saves form data.
- **Navigation:** “Next” and “Previous” button listeners that change the displayed panel.

---

## 6. JButton Best Practices

### Definitions

**Core Definition.** `JButton` best practices are the conventions and patterns that ensure buttons are used effectively, efficiently, and maintainably in Swing applications.

**Technical Definition.** Best practices include: using `Action` objects instead of raw `JButton` instances when the same command is also available in menus or tool bars; providing mnemonics and tooltips for accessibility; avoiding heavy work on the EDT; using layout managers instead of absolute positioning; and respecting Java naming conventions and Swing threading rules.

**Beginner-Friendly Explanation.** Best practices are the “rules of thumb” for buttons. They include things like “always use a layout manager,” “give your buttons tooltips,” “don’t do heavy work inside a button click,” and “use `Action` if the same command appears in multiple places.”

---

### Purposes

- To keep button functionality synchronized across menus, tool bars, and buttons.
- To improve accessibility through mnemonics and tooltips.
- To maintain a responsive GUI by keeping the EDT free of long-running tasks.
- To create maintainable layouts by using layout managers instead of absolute positioning.
- To follow established Java conventions for code clarity.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JButton button = new JButton("Save");                       // 1. Create
    button.setMnemonic(KeyEvent.VK_S);                           // 2. Mnemonic
    button.setToolTipText("Save the current file");              // 3. Tooltip
    button.addActionListener(e -> saveFile());                   // 4. Listener

    JPanel panel = new JPanel(new FlowLayout());                 // 5. Layout
    panel.add(button);
    frame.add(panel);
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT | `SwingUtilities.invokeLater(...)` |
| Use `Action` for shared commands | `button.setAction(action)` |
| Mnemonic | `button.setMnemonic(KeyEvent.VK_X)` |
| Tooltip | `button.setToolTipText("...")` |
| Layout manager | Use `FlowLayout`, `GridLayout`, etc., not `setBounds`. |
| Short listener | Perform only quick updates; use `SwingWorker` for long tasks. |
| Meaningful variable names | `saveButton`, `cancelButton`, etc. |

#### Syntax Rules

- **Always use a layout manager** instead of absolute positioning (`setBounds`).
- **Always provide mnemonics and tooltips** for accessibility.
- **Never perform long-running operations inside `actionPerformed`**; use `SwingWorker`.
- **Use `Action` objects** when the same command appears in multiple places (menu, tool bar, button).
- **Do not use `this` as the action listener** if it exposes too many public methods; prefer a private inner class or lambda.

#### Constraints and Limitations

- Using `null` layout and `setBounds` makes the GUI non-portable and fragile.
- Blocking the EDT inside `actionPerformed` freezes the GUI.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Button

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;

public class BestPracticeButton {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeButton::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Button");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JButton saveButton = new JButton("Save");                  // 1. Create
        saveButton.setMnemonic(KeyEvent.VK_S);                      // 2. Mnemonic
        saveButton.setToolTipText("Save the current document");     // 3. Tooltip
        saveButton.addActionListener(e -> {                         // 4. Listener
            System.out.println("Saving...");
        });

        JPanel panel = new JPanel(new FlowLayout());                // 5. Layout
        panel.add(saveButton);

        frame.add(panel);
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A “Save” button with the mnemonic Alt+S and a tooltip. Clicking it prints “Saving...”.

**Why the code produces this result.** The button follows all best practices: it uses a layout manager, provides a mnemonic and tooltip, and has a short listener.

---

#### Example 2: Common Mistake — Absolute Positioning

```java
import javax.swing.*;

public class BadButtonLayout {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Layout");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(null);                                   // WRONG: null layout

            JButton button = new JButton("Click");
            button.setBounds(50, 50, 100, 30);                       // Manual position
            frame.add(button);

            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The button appears at (50, 50) on the current platform, but its position may be wrong on different platforms or when the font size changes.

**Why this is wrong.** Using a `null` layout and `setBounds` makes the GUI fragile and non-portable. Layout managers should be used instead.

---

### Real-World Cases

- **Production Swing applications** use layout managers, mnemonics, and tooltips to create consistent, accessible button interfaces.
- **IDE and editor plugins** follow these practices when embedding buttons in host applications.
- **Educational examples** use the best-practice skeleton to demonstrate correct `JButton` usage from the start.

---

## References

Oracle. *Class JButton*. Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JButton.html 

Oracle. *How to Use Buttons, Check Boxes, and Radio Buttons*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/button.html 

Oracle. *Interface ActionListener*. Java Platform SE 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionListener.html 

Stack Overflow. *doClick() method of JButton*. https://stackoverflow.com/questions/41448629/ 

Stack Overflow. *JButton best practices*. https://stackoverflow.com/questions/... (layout managers, EDT, Action usage) 

MIT. *On To Java — Event Handling*. https://people.csail.mit.edu/phw/OnToJava/ONTOJAVA925.HTML 