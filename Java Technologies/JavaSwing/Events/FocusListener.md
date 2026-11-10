# FocusListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `FocusListener` is a low-level event listener interface in the `java.awt.event` package for receiving keyboard focus events on a component. It declares two methods—`focusGained` and `focusLost`—that are invoked when a component gains or loses the keyboard focus. `FocusAdapter` is an abstract adapter class that implements both methods with empty bodies, allowing developers to override only the events of interest.

**Technical Definition.** `public interface FocusListener extends EventListener`. The interface declares two methods: `void focusGained(FocusEvent e)` and `void focusLost(FocusEvent e)`. Focus events are fired whenever a component gains or loses the keyboard focus, whether the change occurs through the mouse, the keyboard, or programmatically . A class interested in focus events either implements this interface (and both methods) or extends `FocusAdapter` and overrides only the methods of interest . Listeners are registered on a component using `addFocusListener` . All focus events are delivered on the Event Dispatch Thread (EDT). The event object, `FocusEvent`, extends `ComponentEvent` and provides additional context: `isTemporary()` indicates whether the focus change is temporary (e.g., when the window loses focus or a popup menu appears) ; `getOppositeComponent()` returns the other component involved in the focus change—the component that lost focus for a `FOCUS_GAINED` event, or the component that gained focus for a `FOCUS_LOST` event .

**Beginner-Friendly Explanation.** `FocusListener` is the "who has the keyboard" listener. When the user clicks into a text field, tabs to a button, or clicks away from a component, that component either gains or loses the "focus"—the right to receive keyboard input. `FocusListener` tells you when this happens. You can use it to select all text when a field gains focus, validate input when a field loses focus, highlight the active component, or track which field the user is currently editing. Unlike `ActionListener`, which fires on button clicks, `FocusListener` fires on the much lower-level event of focus moving between components. The companion `FocusAdapter` lets you handle just one of the two events without writing empty methods for the other. Think of focus as a spotlight that moves around the interface: `focusGained` fires when a component steps into the spotlight, and `focusLost` fires when it steps out. The `FocusListener` is your cue sheet for when those transitions happen.

---

### Key Characteristics

1. **Two-method interface.** `focusGained` and `focusLost` cover the entire focus lifecycle; there are no other methods.
2. **Adapter class available.** `FocusAdapter` implements both methods with empty bodies, so you can override only the one you need .
3. **Focus changes are paired.** When focus moves from component A to component B, A fires `focusLost` first, then B fires `focusGained`. The `FocusEvent` of each describes the "opposite component" .
4. **Temporary vs. permanent.** A focus change can be temporary (e.g., when the window loses focus or a popup menu is displayed) or permanent. `FocusEvent.isTemporary()` distinguishes the two .
5. **Focus subsystem dependency.** Focus events are driven by the AWT focus subsystem. Components must be focusable (e.g., `setFocusable(true)`) and part of a focus cycle to receive focus events.
6. **EDT dispatch.** All focus events are delivered on the EDT; handlers must be fast and must not block.
7. **Not for controlling focus.** Swing's official guidance discourages using `FocusListener` to *control* focus (e.g., forcing focus to a particular component); focus should move naturally with the Tab key and user clicks .
8. **Asynchronous delivery.** Focus events can arrive asynchronously relative to other events. For operations like `selectAll()` that depend on the component's state, wrapping the logic in `SwingUtilities.invokeLater` is often necessary .

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JComponent`, `JTextField`, `JButton`, and basic Swing containers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `FocusEvent` and the AWT focus subsystem.
- Knowledge of `KeyboardFocusManager` and `PropertyChangeListener` for tracking focus across multiple components.
- A JDK (version 8 or later; `FocusListener` and `FocusAdapter` are in `java.awt.event`).

---

### Related Programming Areas

- **`FocusAdapter`:** The abstract adapter class for `FocusListener`.
- **`FocusEvent`:** The event object carrying the component, the opposite component, and the temporary flag.
- **`WindowFocusListener` and `WindowAdapter`:** For tracking focus at the window level (as opposed to component level) .
- **`KeyboardFocusManager` and `PropertyChangeListener`:** The recommended way to track focus changes across many components .
- **`SwingUtilities.invokeLater`:** Used to defer focus-related UI operations (e.g., `selectAll`) until the focus change has fully settled .
- **`JComponent.setFocusable` and `setRequestFocusEnabled`:** Control whether a component can receive focus at all.
- **`InputMap` and `ActionMap`:** The modern alternative for focus-independent keyboard shortcuts.

---

## Core Concepts and Features

---

## 1. FocusListener — The Interface

### Definitions

**Core Definition.** `FocusListener` is an interface for receiving keyboard focus events on a component. It declares `focusGained` and `focusLost`, which are invoked when the component acquires or relinquishes the keyboard focus.

**Technical Definition.** `public interface FocusListener extends EventListener`. The interface declares two methods: `void focusGained(FocusEvent e)` and `void focusLost(FocusEvent e)`. A class interested in focus events either implements this interface (and both methods) or extends `FocusAdapter` and overrides only the methods of interest . Listeners are registered on a component using `addFocusListener`. Focus events are fired whenever a component gains or loses the keyboard focus, regardless of whether the change occurs through the mouse, the keyboard, or programmatically .

**Beginner-Friendly Explanation.** `FocusListener` is the "focus events" contract. Implement it (or extend `FocusAdapter`) and register it with a component to hear about when the component gains or loses keyboard focus.

---

### Purposes

- To receive notifications when a component gains keyboard focus (e.g., the user clicks into a text field).
- To receive notifications when a component loses keyboard focus (e.g., the user clicks elsewhere or presses Tab).
- To perform focus-dependent actions such as selecting all text on entry, validating input on exit, or highlighting the active component.
- To track which component the user is currently interacting with.
- To implement custom focus-driven behaviour in specialized components (e.g., canvas widgets that must know when they are active).

---

### Syntax Rules and Structure

#### Complete General Syntax (Full Implementation)

```java
component.addFocusListener(new FocusListener() {
    public void focusGained(FocusEvent e) { /* gained focus */ }
    public void focusLost(FocusEvent e)   { /* lost focus */ }
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `focusGained(FocusEvent e)` | The component gains the keyboard focus. |
| `focusLost(FocusEvent e)` | The component loses the keyboard focus. |

#### Syntax Rules

- Both methods must be implemented when implementing the interface directly.
- Use `FocusAdapter` to override only the methods of interest.
- Register the listener with `addFocusListener`.
- Listener methods are invoked on the EDT.
- The component must be focusable (`setFocusable(true)`) to receive focus events.

#### Constraints and Limitations

- Focus events are only delivered to focusable components that are part of the focus cycle.
- Focus changes can be temporary (e.g., window loses focus, popup menus); check `FocusEvent.isTemporary()` to distinguish.
- `FocusListener` is not the right tool for controlling focus; let the focus subsystem manage natural focus movement .

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full FocusListener Implementation

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusEvent;
import java.awt.event.FocusListener;

public class FullFocusListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Full FocusListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addFocusListener(new FocusListener() {               // 1. Register
                @Override
                public void focusGained(FocusEvent e) {                // 2. Gained
                    System.out.println("Focus gained");
                }
                @Override
                public void focusLost(FocusEvent e) {                  // 3. Lost
                    System.out.println("Focus lost");
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(new JLabel("Name:"));
            frame.add(field);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the window opens, the text field gains focus and prints "Focus gained". Clicking outside the field prints "Focus lost". Clicking back into the field prints "Focus gained" again.

**Why the code produces this result.** The `FocusListener` is registered on the text field. When the field acquires focus (either at startup or when clicked), `focusGained` fires. When it loses focus (clicking elsewhere), `focusLost` fires.

---

### Real-World Cases

- **Select-all-on-focus:** A text field that automatically selects all its text when it gains focus, making it easy for the user to replace the content .
- **Input validation on blur:** Validating a field's content when it loses focus (e.g., checking that an email address is well-formed).
- **Highlighting the active component:** A custom component that draws a border or changes its background when it gains focus .
- **Tracking the last active field:** A toolbar that acts on the last-focused text field by tracking focus changes with a shared listener.
- **Accessibility:** Providing visual focus indicators for keyboard users.

---

## 2. FocusAdapter — The Adapter Class

### Definitions

**Core Definition.** `FocusAdapter` is an abstract adapter class that implements `FocusListener` with empty method bodies, allowing subclasses to override only the methods of interest.

**Technical Definition.** `public abstract class FocusAdapter extends Object implements FocusListener`. The class provides empty implementations of `focusGained` and `focusLost`. It exists as a convenience for creating listener objects: "Extend this class to create a `FocusEvent` listener and override the methods for the events of interest. (If you implement the `FocusListener` interface, you have to define all of the methods in it. This abstract class defines null methods for them all, so you can only have to define methods for events you care about.)" . Registration is done via `addFocusListener` .

**Beginner-Friendly Explanation.** `FocusAdapter` is the "pre-filled form" for focus events. Instead of implementing two empty methods just to handle one, you extend `FocusAdapter` and override only the method you need.

---

### Purposes

- To reduce boilerplate when implementing `FocusListener`.
- To override only `focusGained` or only `focusLost` cleanly.
- To enable readable anonymous inner class implementations.
- To avoid implementing unused methods.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
component.addFocusListener(new FocusAdapter() {
    @Override
    public void focusGained(FocusEvent e) {
        // reaction code
    }
});
```

#### Component Breakdown

| Adapter | Implements | Methods |
|---------|-----------|---------|
| `FocusAdapter` | `FocusListener` | 2 (both empty) |

#### Syntax Rules

- Extend the adapter and override only the methods you need.
- Register via `addFocusListener`.
- Adapters are abstract classes; lambdas cannot replace them (though the interface has only two methods, so anonymous classes are the standard idiom).

#### Constraints and Limitations

- Extending an adapter prevents extending another class (single inheritance).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: FocusAdapter for Select-All-on-Focus

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;
import javax.swing.text.JTextComponent;

public class SelectAllOnFocus {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Select All on Focus");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField("Default text", 20);
            field.addFocusListener(new FocusAdapter() {                // 1. Extend
                @Override
                public void focusGained(FocusEvent e) {                // 2. Override
                    JTextComponent text = (JTextComponent) e.getComponent();
                    SwingUtilities.invokeLater(() ->                   // 3. Defer
                        text.selectAll());
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the text field gains focus, all its text is selected, allowing the user to type over it immediately.

**Why the code produces this result.** `FocusAdapter` overrides only `focusGained`. The `selectAll()` call is wrapped in `SwingUtilities.invokeLater` because focus events are asynchronous and the selection may otherwise be cleared by the time the mouse click completes . The text component is obtained via `e.getComponent()` and cast to `JTextComponent`.

---

### Real-World Cases

- **Select-all-on-focus** for text fields (as in the example above).
- **Blur validation** for fields that should validate when the user clicks away.
- **Single-purpose focus handlers** where only one of the two events matters.

---

## 3. Focus Events — focusGained()

### Definitions

**Core Definition.** `focusGained(FocusEvent e)` is invoked when the component gains the keyboard focus.

**Technical Definition.** `public void focusGained(FocusEvent e)`. The method is called just after the listened-to component gets the focus . The `FocusEvent` carries the component that gained focus (`getComponent()`) and the component that lost focus (`getOppositeComponent()`, which may be `null` if the focus came from outside the Java application) . The event may be temporary (e.g., when a popup menu appears); check `isTemporary()` .

**Beginner-Friendly Explanation.** `focusGained` fires when a component becomes the focus owner—when the user clicks into it, tabs to it, or the program requests focus for it.

---

### Purposes

- To perform an action when a component becomes active (e.g., selecting all text, starting an animation, highlighting the component).
- To record which component is now active for later reference.
- To update UI state (e.g., enabling a toolbar action that applies to the focused field).
- To implement focus-sensitive custom painting (e.g., drawing a focus border).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void focusGained(FocusEvent e) {
    Component focused = e.getComponent();
    Component previous = e.getOppositeComponent();
    boolean temporary = e.isTemporary();
    // react
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `e.getComponent()` | The component that gained focus. |
| `e.getOppositeComponent()` | The component that lost focus (may be `null`). |
| `e.isTemporary()` | `true` if the focus change is temporary. |

#### Syntax Rules

- The component must be focusable.
- The event may be temporary; check `isTemporary()` if the distinction matters.
- The handler runs on the EDT.

#### Constraints and Limitations

- Do not use `focusGained` to force focus to another component; that interferes with natural focus flow .
- Some operations (e.g., `selectAll`) may need `SwingUtilities.invokeLater` to work reliably .

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Highlighting on Focus

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;

public class HighlightOnFocus {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Highlight on Focus");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addFocusListener(new FocusAdapter() {
                @Override
                public void focusGained(FocusEvent e) {                // 1. Gained
                    field.setBackground(Color.YELLOW);                  // 2. Highlight
                }
                @Override
                public void focusLost(FocusEvent e) {                   // 3. Lost
                    field.setBackground(Color.WHITE);                   // 4. Restore
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The text field turns yellow when it gains focus and reverts to white when it loses focus.

**Why the code produces this result.** `focusGained` sets the background to yellow; `focusLost` restores it to white. This provides visual feedback about which component is active.

---

### Real-World Cases

- **Select-all-on-focus** for text fields .
- **Focus indicators** for custom-painted components (e.g., drawing a red border around the focused component) .
- **Lazy initialization** triggered by focus (e.g., loading data into a panel when it becomes active).
- **Toolbar synchronization** (e.g., enabling formatting buttons for the focused text field).

---

## 4. Focus Events — focusLost()

### Definitions

**Core Definition.** `focusLost(FocusEvent e)` is invoked when the component loses the keyboard focus.

**Technical Definition.** `public void focusLost(FocusEvent e)`. The method is called just after the listened-to component loses the focus . The `FocusEvent` carries the component that lost focus (`getComponent()`) and the component that gained focus (`getOppositeComponent()`, which may be `null` if focus moved outside the Java application) . The event may be temporary (e.g., when the window loses focus); check `isTemporary()` .

**Beginner-Friendly Explanation.** `focusLost` fires when a component is no longer the focus owner—when the user clicks elsewhere, tabs away, or the window loses focus.

---

### Purposes

- To validate input when a field loses focus (e.g., checking that a value is well-formed).
- To commit changes when a field is deactivated.
- To revert focus-related visual changes (e.g., removing a highlight).
- To track when a component is no longer active for state management.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void focusLost(FocusEvent e) {
    Component lost = e.getComponent();
    Component next = e.getOppositeComponent();
    boolean temporary = e.isTemporary();
    // react
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `e.getComponent()` | The component that lost focus. |
| `e.getOppositeComponent()` | The component that gained focus (may be `null`). |
| `e.isTemporary()` | `true` if the focus loss is temporary. |

#### Syntax Rules

- The event may be temporary (e.g., the window loses focus); check `isTemporary()` if the distinction matters.
- The handler runs on the EDT.

#### Constraints and Limitations

- A temporary focus loss (e.g., when the user clicks a popup menu or another window) should not trigger validation that might discard the user's in-progress input.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Validation on Focus Loss

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;

public class ValidateOnBlur {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Validate on Blur");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField email = new JTextField(20);
            JLabel status = new JLabel(" ");

            email.addFocusListener(new FocusAdapter() {
                @Override
                public void focusLost(FocusEvent e) {                  // 1. Lost
                    if (e.isTemporary()) return;                       // 2. Ignore temp
                    String text = email.getText();
                    if (text.contains("@")) {
                        status.setText("Looks valid");
                    } else {
                        status.setText("Missing @");
                    }
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(new JLabel("Email:"));
            frame.add(email);
            frame.add(status);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the user clicks out of the email field, the status label updates to indicate whether the input contains an “@” symbol. If the focus loss is temporary (e.g., the user clicks another window), the validation is skipped.

**Why the code produces this result.** `focusLost` checks `e.isTemporary()` to avoid validating when focus is lost temporarily (e.g., switching to another application). For permanent focus loss, it validates the content and updates the status label.

---

### Real-World Cases

- **Email validation on blur.**
- **Saving edits** when a field loses focus.
- **Reverting highlights** when a component is no longer active.
- **Commit-on-blur** patterns in data-entry forms.

---

## 5. Focus Listener Best Practices

### Definitions

**Core Definition.** Focus listener best practices are the conventions and patterns that ensure focus handling is robust, non-intrusive, and aligned with Swing's focus subsystem.

**Technical Definition.** Best practices include: using `FocusAdapter` to override only the methods of interest; not using `FocusListener` to **control** focus (let the Tab key and user clicks manage focus naturally) ; using `SwingUtilities.invokeLater` for operations like `selectAll()` that depend on the focus change having fully settled ; checking `FocusEvent.isTemporary()` to avoid acting on temporary focus changes ; using `KeyboardFocusManager` and `PropertyChangeListener` when tracking focus across many components ; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for focus handling are the "rules of thumb" that prevent focus fights, unreliable text selection, and accessibility problems. They include things like "don't force focus," "defer selectAll with invokeLater," "ignore temporary focus losses," and "use a focus manager for global tracking."

---

### Purposes

- To avoid interfering with the focus subsystem's natural movement (Tab key, clicks).
- To ensure focus-dependent operations (e.g., `selectAll`) execute reliably.
- To distinguish meaningful focus changes from temporary ones.
- To track focus across many components without registering a listener on each.
- To maintain thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {                                // 1. EDT
    JTextField field = new JTextField(20);
    field.addFocusListener(new FocusAdapter() {                   // 2. Adapter
        @Override
        public void focusGained(FocusEvent e) {                   // 3. Gained
            if (e.isTemporary()) return;                           // 4. Temp
            SwingUtilities.invokeLater(() ->                       // 5. Defer
                ((JTextComponent) e.getComponent()).selectAll());
        }
        @Override
        public void focusLost(FocusEvent e) {                     // 6. Lost
            if (e.isTemporary()) return;                           // 7. Temp
            validate((JTextComponent) e.getComponent());
        }
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `FocusAdapter` | Extend and override only what you need . |
| Don't control focus | Let Tab and clicks manage focus naturally . |
| Defer risky operations | `SwingUtilities.invokeLater` for `selectAll` and similar . |
| Check temporary flag | `if (e.isTemporary()) return;` . |
| Track many components | Use `KeyboardFocusManager` + `PropertyChangeListener` . |
| EDT | All focus handling runs on the EDT. |

#### Syntax Rules

- **Always use `FocusAdapter`** when you only need one of the two methods.
- **Never use `FocusListener` to force focus** to a specific component; this breaks natural navigation and accessibility .
- **Always check `isTemporary()`** before performing permanent actions like validation or committing data.
- **Always wrap operations like `selectAll()` in `SwingUtilities.invokeLater`** to avoid timing issues with the focus subsystem .
- **Always use `KeyboardFocusManager`** when tracking focus across many components, rather than registering a listener on each .

#### Constraints and Limitations

- Focus is asynchronous; operations that depend on the focus change completing may need deferral.
- Temporary focus losses (window deactivation, popup menus) should not trigger destructive actions.
- `FocusListener` is component-level; `WindowFocusListener` is for window-level focus, and `KeyboardFocusManager` is for global tracking.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Select-All-on-Focus with Temporary Check

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;
import javax.swing.text.JTextComponent;

public class BestPracticeFocus {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeFocus::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Focus");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JTextField field = new JTextField("Select me", 20);
        field.addFocusListener(new FocusAdapter() {                    // 1. Adapter
            @Override
            public void focusGained(FocusEvent e) {                    // 2. Gained
                if (e.isTemporary()) return;                            // 3. Temp
                SwingUtilities.invokeLater(() ->                       // 4. Defer
                    ((JTextComponent) e.getComponent()).selectAll());
            }
        });

        frame.setLayout(new FlowLayout());
        frame.add(field);
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** When the field gains focus, its text is selected. The selection works reliably because it is deferred until the focus change has fully settled. Temporary focus changes (e.g., from a popup menu) do not trigger the selection.

**Why the code produces this result.** The adapter overrides only `focusGained`. The `isTemporary()` check filters out temporary focus gains. `SwingUtilities.invokeLater` ensures `selectAll()` runs after the focus subsystem has completed its processing, preventing the selection from being immediately cleared by the mouse click that caused the focus .

---

#### Example 2: Common Mistake — Using FocusListener to Control Focus

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.FocusAdapter;
import java.awt.event.FocusEvent;

public class BadFocusControl {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Focus Control");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field1 = new JTextField(10);
            JTextField field2 = new JTextField(10);

            field1.addFocusListener(new FocusAdapter() {
                @Override
                public void focusLost(FocusEvent e) {
                    if (field1.getText().isEmpty()) {
                        field1.requestFocus();                      // WRONG
                    }
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field1);
            frame.add(field2);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The user cannot move focus away from `field1` when it is empty. Tabbing or clicking to `field2` causes focus to immediately snap back to `field1`, trapping the user. The application becomes difficult to navigate, especially for keyboard and assistive-technology users.

**Why this is wrong.** As the community guidance states, "you should NOT be using a `FocusListener` to try to control focus of components. Focus will move naturally when you use the Tab key" . Forcing focus back to a component creates a "focus trap" that breaks navigation and accessibility. The correct approach is to use input validation or to disable the action that requires the field, not to trap focus.

---

#### Example 3: Tracking Focus Across Many Components with KeyboardFocusManager

```java
import javax.swing.*;
import java.awt.*;
import java.beans.PropertyChangeEvent;
import java.beans.PropertyChangeListener;

public class GlobalFocusTracking {
    private static JLabel status = new JLabel("No focus");

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Global Focus Tracking");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            KeyboardFocusManager.getCurrentKeyboardFocusManager()
                .addPropertyChangeListener("focusOwner",                  // 1. Manager
                    evt -> {
                        Object newFocus = evt.getNewValue();
                        if (newFocus instanceof JComponent) {
                            status.setText("Focus: "
                                + ((JComponent) newFocus).getName());
                        }
                    });

            JTextField f1 = new JTextField(10);
            f1.setName("Field 1");
            JTextField f2 = new JTextField(10);
            f2.setName("Field 2");

            JPanel panel = new JPanel(new FlowLayout());
            panel.add(f1);
            panel.add(f2);
            panel.add(status);

            frame.add(panel);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The status label updates to show the name of the component that currently has focus, regardless of which component it is.

**Why the code produces this result.** Instead of registering a `FocusListener` on every component, this approach registers a single `PropertyChangeListener` on the `KeyboardFocusManager` for the `"focusOwner"` property. This is the recommended pattern for tracking focus across many components .

---

### Real-World Cases

- **Production Swing applications** use `FocusAdapter` with `invokeLater` for select-all-on-focus, and `KeyboardFocusManager` for global focus tracking.
- **Data-entry forms** validate on permanent focus loss (checking `isTemporary()`) to avoid interrupting the user during temporary focus changes.
- **Accessibility-conscious applications** never trap focus and always allow natural Tab navigation .

---

## References

Oracle. *Interface FocusListener*. Java Platform, Standard Edition 25 API Specification. https://docs.oracle.com/javase/jp/25/docs/api/java.desktop/java/awt/event/FocusListener.html 

Oracle. *Class FocusAdapter*. Java Development Kit Version 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/java/awt/event/FocusAdapter.html 

Oracle. *How to Write a Focus Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/focuslistener.html 

Oracle. *How to Use the Focus Subsystem*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/focus.html

Oracle. *Class FocusEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/FocusEvent.html

Stack Overflow. *General advice on FocusListener best practices*. https://stackoverflow.com/revisions/4ca1e33f-00bd-420c-aefa-28582c3a98da/view-source 

Stack Overflow. *FocusListener and invokeLater for selectAll*. https://stackoverflow.com/revisions/66a658bb-c069-44f1-821a-53c7fdcedc0e/view-source 

GitHub. *SelectAll on focusGained example*. https://github.com/aterai/java-swing-tips/blob/main/examples/SelectAll/src/java/example/MainPanel.java 