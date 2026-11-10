# KeyListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `KeyListener` is a low-level event listener interface in the `java.awt.event` package for receiving keyboard events (keystrokes) on a component. It declares three methods—`keyTyped`, `keyPressed`, and `keyReleased`—that are invoked when a key is typed, pressed, or released while the component has keyboard focus. `KeyAdapter` is an abstract adapter class that implements all three methods with empty bodies, allowing developers to override only the events of interest.

**Technical Definition.** `public interface KeyListener extends EventListener`. The interface declares three methods: `void keyTyped(KeyEvent e)`, `void keyPressed(KeyEvent e)`, and `void keyReleased(KeyEvent e)`. Listeners are registered via `Component.addKeyListener(KeyListener)`. Keyboard events are generated when a key is pressed, released, or typed, and the relevant method is invoked on the registered listener with a `KeyEvent` object. `KeyAdapter` provides empty implementations of all three methods, "so you can only have to define methods for events you care about". However, Swing's official guidance strongly recommends **key bindings** (`InputMap` and `ActionMap`) over `KeyListener` for most use cases, because key bindings are more flexible, more maintainable, and work regardless of which component has focus. `KeyListener` remains appropriate for low-level keyboard monitoring—custom components that need raw key events, or situations where you must observe every keystroke regardless of focus (e.g., debugging or global hotkeys within an application window).

**Beginner-Friendly Explanation.** `KeyListener` is the "raw keyboard feed" for a component. When the component has focus and the user presses a key, the listener fires `keyPressed`; when the key is released, `keyReleased`; and when a character is actually produced (a printable Unicode character), `keyTyped` fires. You implement these methods to react to specific keys—for example, moving a game character with the arrow keys, or intercepting the Enter key in a custom component. However, Swing's designers strongly prefer **key bindings** over `KeyListener` because key listeners are fragile: they only work when the component has focus, they are hard to maintain, and they do not integrate well with Swing's action architecture. Think of `KeyListener` as a stethoscope pressed directly against the keyboard hardware: it hears every key, but it is a low-level tool that requires careful handling. For most application-level keyboard shortcuts, use key bindings instead.

---

### Key Characteristics

1. **Three event methods.** `keyTyped` fires when a Unicode character is generated; `keyPressed` fires when a key is pressed down; `keyReleased` fires when a key is released. These are distinct events with different semantics and different data.
2. **Focus dependency.** `KeyListener` only receives events when the registered component **has keyboard focus**. This is a major limitation compared to key bindings, which can be configured to work when the component does not have focus (e.g., `WHEN_IN_FOCUSED_WINDOW`).
3. **Low-level vs. high-level.** `keyPressed` and `keyReleased` are low-level events that depend on the platform and keyboard layout, and report a virtual key code (`getKeyCode()`). `keyTyped` is a high-level event that reports a Unicode character (`getKeyChar()`) and generally does not depend on the platform or keyboard layout.
4. **Adapter available.** `KeyAdapter` implements all three methods with empty bodies, reducing boilerplate when you only need to handle one or two event types.
5. **Key bindings preferred.** Swing's official documentation recommends key bindings (`InputMap`/`ActionMap`) over `KeyListener` for responding to individual keys, because key bindings are more maintainable, self-documenting, and focus-independent.
6. **`getKeyChar()` vs. `getKeyCode()`.** `getKeyChar()` is meaningful only for `KEY_TYPED` events; `getKeyCode()` is meaningful only for `KEY_PRESSED` and `KEY_RELEASED` events. Mixing them up is a common source of bugs.
7. **EDT dispatch.** All key events are delivered on the Event Dispatch Thread (EDT); handlers must be fast and must not block.
8. **Event consumption.** `KeyEvent` supports `consume()`; a consumed key event is not passed to other listeners or default handlers, enabling custom interception.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JComponent`, `JPanel`, and basic Swing containers.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `KeyEvent` and `InputEvent`.
- Knowledge of `InputMap`, `ActionMap`, and `Action` (the recommended alternative for most keyboard handling).
- A JDK (version 8 or later; `KeyListener` and `KeyAdapter` are in `java.awt.event`).

---

### Related Programming Areas

- **`KeyAdapter`:** The abstract adapter class for `KeyListener`.
- **`KeyEvent`:** The event object carrying key codes, key characters, and modifier state.
- **`InputEvent`:** The parent class of `KeyEvent`, providing modifier mask constants.
- **`InputMap` and `ActionMap`:** The key-binding infrastructure that Swing recommends over `KeyListener`.
- **`Action` and `AbstractAction`:** Encapsulate command logic for use with key bindings.
- **`WHEN_FOCUSED`, `WHEN_IN_FOCUSED_WINDOW`, `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`:** Input map conditions that control when a key binding is active.
- **`JComponent.registerKeyboardAction`:** An obsolete method replaced by `InputMap`/`ActionMap`.

---

## Core Concepts and Features

---

## 1. KeyListener — The Interface

### Definitions

**Core Definition.** `KeyListener` is a low-level event listener interface for receiving keyboard events (key presses, releases, and typed characters) on a component that has keyboard focus.

**Technical Definition.** `public interface KeyListener extends EventListener`. The interface declares three methods: `keyTyped`, `keyPressed`, and `keyReleased`. A class interested in keyboard events either implements this interface (and all its methods) or extends `KeyAdapter` (overriding only the methods of interest). The listener is registered with a component using `addKeyListener`. A keyboard event is generated when a key is pressed, released, or typed, and the relevant method is invoked with a `KeyEvent`.

**Beginner-Friendly Explanation.** `KeyListener` is the interface you implement to listen for raw keyboard events on a focused component. You override the methods for the events you care about and register the listener with the component.

---

### Purposes

- To receive low-level keyboard events (`keyPressed`, `keyReleased`) for custom components that need raw key state.
- To receive `keyTyped` events for character input detection when key bindings are not appropriate.
- To implement custom keyboard-driven interactions (e.g., game controls, custom canvases).
- To observe every keystroke for debugging or logging purposes within a specific component.

---

### Syntax Rules and Structure

#### Complete General Syntax (Full Implementation)

```java
component.addKeyListener(new KeyListener() {
    public void keyTyped(KeyEvent e)    { /* character typed */ }
    public void keyPressed(KeyEvent e)  { /* key pressed */ }
    public void keyReleased(KeyEvent e) { /* key released */ }
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `keyTyped` | A Unicode character is generated (press + release producing a printable character). |
| `keyPressed` | A key is pressed down on the component. |
| `keyReleased` | A key is released on the component. |

#### Syntax Rules

- All three methods must be implemented when implementing the interface directly.
- Use `KeyAdapter` to override only the methods of interest.
- The component must have keyboard focus to receive key events.
- Listener methods are invoked on the EDT.

#### Constraints and Limitations

- `KeyListener` only works when the component has focus; key bindings are preferred for most application-level shortcuts.
- `keyTyped` does not fire for keys that do not generate Unicode characters (e.g., action keys, modifier keys, arrow keys).
- `getKeyChar()` is meaningful only for `keyTyped`; `getKeyCode()` is meaningful only for `keyPressed`/`keyReleased`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full KeyListener Implementation

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

public class FullKeyListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Full KeyListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addKeyListener(new KeyListener() {                   // 1. Register
                @Override
                public void keyTyped(KeyEvent e) {                     // 2. Typed
                    System.out.println("Typed: '" + e.getKeyChar() + "'");
                }
                @Override
                public void keyPressed(KeyEvent e) {                   // 3. Pressed
                    System.out.println("Pressed: " + KeyEvent.getKeyText(e.getKeyCode()));
                }
                @Override
                public void keyReleased(KeyEvent e) {                  // 4. Released
                    System.out.println("Released: " + KeyEvent.getKeyText(e.getKeyCode()));
                }
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing “a” prints “Pressed: A”, “Typed: 'a'”, and “Released: A” in sequence. Pressing a non-character key (e.g., F1) prints “Pressed: F1” and “Released: F1” but no “Typed” line.

**Why the code produces this result.** `keyPressed` fires first with the virtual key code, then `keyTyped` fires with the Unicode character (if one is produced), then `keyReleased` fires with the virtual key code. Modifier keys and action keys generate pressed/released events but no typed events.

---

### Real-World Cases

- **Game controls** where arrow keys or WASD keys move a character.
- **Custom drawing canvases** that respond to Delete or Escape keys.
- **Debugging tools** that log every keystroke on a focused component.
- **Legacy code** that predates the key-binding API and has not been migrated.

---

## 2. KeyAdapter — The Adapter Class

### Definitions

**Core Definition.** `KeyAdapter` is an abstract adapter class that implements `KeyListener` with empty method bodies, allowing subclasses to override only the methods of interest.

**Technical Definition.** `public abstract class KeyAdapter extends Object implements KeyListener`. The class provides empty implementations of `keyTyped`, `keyPressed`, and `keyReleased`. It exists as a convenience for creating listener objects: "If you implement the `KeyListener` interface, you have to define all of the methods in it. This abstract class defines null methods for them all, so you can only have to define methods for events you care about". Registration is done via `addKeyListener`.

**Beginner-Friendly Explanation.** `KeyAdapter` is the "pre-filled form" for keyboard events. Instead of implementing three empty methods just to handle one key, you extend `KeyAdapter` and override only the method you need.

---

### Purposes

- To reduce boilerplate when implementing `KeyListener`.
- To override only `keyPressed`, `keyTyped`, or `keyReleased` cleanly.
- To enable readable anonymous inner class implementations.
- To avoid implementing unused methods.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
component.addKeyListener(new KeyAdapter() {
    @Override
    public void keyPressed(KeyEvent e) {
        // reaction code
    }
});
```

#### Component Breakdown

| Adapter | Implements | Methods |
|---------|-----------|---------|
| `KeyAdapter` | `KeyListener` | 3 (all empty) |

#### Syntax Rules

- Extend the adapter and override only the methods you need.
- Register via `addKeyListener`.
- Adapters are abstract classes; lambdas cannot replace them (though the interface has three methods, so an anonymous class is the standard idiom).

#### Constraints and Limitations

- Extending an adapter prevents extending another class (single inheritance).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: KeyAdapter for Enter Key Detection

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class KeyAdapterDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("KeyAdapter Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addKeyListener(new KeyAdapter() {                    // 1. Extend
                @Override
                public void keyPressed(KeyEvent e) {                   // 2. Override
                    if (e.getKeyCode() == KeyEvent.VK_ENTER) {
                        System.out.println("Enter pressed!");
                    }
                }
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Enter in the text field prints “Enter pressed!”. Other keys are silently ignored because only `keyPressed` is overridden.

**Why the code produces this result.** The adapter provides empty implementations for `keyTyped` and `keyReleased`; only `keyPressed` contains logic, so only that event is handled.

---

### Real-World Cases

- **Single-key interception** (e.g., capturing Escape or Enter in a custom component).
- **Custom game controls** where only key-pressed events matter.
- **Minimal listeners** that do not need all three event types.

---

## 3. Key Events

### 3.1 keyTyped()

#### Definitions

**Core Definition.** `keyTyped(KeyEvent e)` is invoked when a key has been typed—that is, when a key press is followed by a key release that produces a valid Unicode character.

**Technical Definition.** `public void keyTyped(KeyEvent e)`. This is a **high-level** event that is generally independent of the platform and keyboard layout. It is generated when a Unicode character is entered. For key typed events, `getKeyChar()` returns the Unicode character (or `CHAR_UNDEFINED`), and `getKeyCode()` always returns `VK_UNDEFINED`. No key typed events are generated for keys that do not produce Unicode characters (e.g., action keys, modifier keys, arrow keys).

**Beginner-Friendly Explanation.** `keyTyped` fires when an actual character is produced—such as “a”, “A”, or “€”. It does not fire for arrow keys, function keys, or modifier keys, because those do not produce characters.

---

#### Purposes

- To detect character input for custom text-processing components.
- To capture Unicode characters independent of keyboard layout.
- To implement character-level input handling where key bindings are not appropriate.

---

#### Syntax Rules and Structure

```java
@Override
public void keyTyped(KeyEvent e) {
    char c = e.getKeyChar();
    // react to the character
}
```

| Method | Meaning |
|--------|---------|
| `e.getKeyChar()` | The Unicode character typed. |
| `e.isConsumed()` | Whether the event has been consumed. |

#### Constraints and Limitations

- Does not fire for non-character keys (arrows, function keys, modifiers).
- `getKeyCode()` is always `VK_UNDEFINED` for typed events.

---

#### Annotated Complete Step-by-Step Code Examples

```java
import javax.swing.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class TypedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("keyTyped Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addKeyListener(new KeyAdapter() {
                @Override
                public void keyTyped(KeyEvent e) {
                    char c = e.getKeyChar();
                    if (Character.isDigit(c)) {
                        System.out.println("Digit typed: " + c);
                    }
                }
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing digit characters prints “Digit typed: 5”, etc. Typing letters or pressing arrow keys prints nothing.

---

### 3.2 keyPressed()

#### Definitions

**Core Definition.** `keyPressed(KeyEvent e)` is invoked when a key is pressed down on the component.

**Technical Definition.** `public void keyPressed(KeyEvent e)`. This is a **low-level** event that depends on the platform and keyboard layout. It reports the virtual key code via `getKeyCode()` (e.g., `VK_A`, `VK_ENTER`, `VK_SHIFT`) and the extended key code via `getExtendedKeyCode()`. Unlike `keyTyped`, it fires for every key, including modifier and action keys.

**Beginner-Friendly Explanation.** `keyPressed` fires the moment any key goes down—including arrow keys, function keys, and modifiers. It tells you which physical key was pressed via a virtual key code.

---

#### Purposes

- To detect key presses for game controls and custom navigation.
- To intercept specific keys (e.g., Escape to close a dialog).
- To observe modifier key state for custom interactions.
- To implement key-hold behaviour by combining with `keyReleased`.

---

#### Syntax Rules and Structure

```java
@Override
public void keyPressed(KeyEvent e) {
    int keyCode = e.getKeyCode();
    if (keyCode == KeyEvent.VK_ESCAPE) {
        // handle escape
    }
}
```

| Method | Meaning |
|--------|---------|
| `e.getKeyCode()` | The virtual key code (e.g., `VK_ENTER`). |
| `e.getModifiersEx()` | Extended modifier state. |
| `e.consume()` | Consume the event. |

#### Constraints and Limitations

- Fires repeatedly if the key is held down (auto-repeat), depending on the platform.
- The component must have focus.

---

#### Annotated Complete Step-by-Step Code Examples

```java
import javax.swing.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class PressedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("keyPressed Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addKeyListener(new KeyAdapter() {
                @Override
                public void keyPressed(KeyEvent e) {
                    if (e.getKeyCode() == KeyEvent.VK_ESCAPE) {
                        System.out.println("Escape pressed — closing dialog");
                    }
                }
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Escape prints “Escape pressed — closing dialog”. Other keys are ignored.

---

### 3.3 keyReleased()

#### Definitions

**Core Definition.** `keyReleased(KeyEvent e)` is invoked when a key is released on the component.

**Technical Definition.** `public void keyReleased(KeyEvent e)`. This is a **low-level** event that reports the virtual key code of the released key. It fires after `keyPressed` (and possibly `keyTyped`) for the same key. It is useful for detecting the end of a key-hold gesture and for cleaning up state established in `keyPressed`.

**Beginner-Friendly Explanation.** `keyReleased` fires when you let go of a key. It is the counterpart to `keyPressed` and is used to detect the end of a key-hold action.

---

#### Purposes

- To detect the end of a key-hold gesture.
- To reset state established in `keyPressed` (e.g., stop moving a character).
- To implement multi-key combinations where the release of one key changes behaviour.

---

#### Syntax Rules and Structure

```java
@Override
public void keyReleased(KeyEvent e) {
    int keyCode = e.getKeyCode();
    // clean up state
}
```

#### Constraints and Limitations

- The component must have focus.
- May not fire if the component loses focus while the key is held.

---

#### Annotated Complete Step-by-Step Code Examples

```java
import javax.swing.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class ReleasedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("keyReleased Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(20);
            field.addKeyListener(new KeyAdapter() {
                @Override
                public void keyPressed(KeyEvent e) {
                    if (e.getKeyCode() == KeyEvent.VK_SHIFT) {
                        System.out.println("Shift held");
                    }
                }
                @Override
                public void keyReleased(KeyEvent e) {
                    if (e.getKeyCode() == KeyEvent.VK_SHIFT) {
                        System.out.println("Shift released");
                    }
                }
            });

            frame.add(field);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Shift prints “Shift held”; releasing Shift prints “Shift released”.

---

## 4. Key Listener Best Practices

### Definitions

**Core Definition.** Key listener best practices are the conventions and patterns that ensure keyboard handling is robust, maintainable, and aligned with Swing's architecture. The single most important best practice is to **prefer key bindings over `KeyListener`** for application-level keyboard shortcuts.

**Technical Definition.** Swing's official documentation states: "An alternative to key bindings is using key listeners. Key listeners have their place as a low-level interface to keyboard input, but for responding to individual keys key bindings are more appropriate and tend to result in more easily maintained code. Key listeners are also difficult if the key binding is to be active when the component doesn't have focus". Key bindings offer advantages: they are self-documenting, take the containment hierarchy into account, encourage reusable `Action` objects, allow actions to be easily removed, customized, or shared, and make it easy to change the key to which an action is bound. `Action` objects also have an enabled state that can be toggled without tracking which component they are attached to.

**Beginner-Friendly Explanation.** The best practice for keyboard handling in Swing is simple: **use key bindings, not `KeyListener`**, unless you have a specific reason to need raw low-level key events. Key bindings are more reliable because they work regardless of focus, they integrate with Swing's `Action` architecture, and they are easier to maintain. `KeyListener` is a low-level tool for custom components and specialized scenarios.

---

### Purposes

- To use the right tool for keyboard handling: key bindings for shortcuts, `KeyListener` only for low-level needs.
- To ensure keyboard shortcuts work regardless of which component has focus (via `WHEN_IN_FOCUSED_WINDOW`).
- To integrate keyboard handling with `Action` objects for reusable, stateful commands.
- To avoid the focus-dependency pitfalls of `KeyListener`.
- To maintain clean, testable, and maintainable keyboard-handling code.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton (Key Bindings)

```java
// The recommended approach: key bindings
JComponent component = ...;
component.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)
    .put(KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK),
         "saveAction");
component.getActionMap().put("saveAction", new AbstractAction("Save") {
    @Override
    public void actionPerformed(ActionEvent e) {
        saveDocument();
    }
});
```

#### Complete General Syntax (KeyListener — Only for Low-Level Needs)

```java
component.addKeyListener(new KeyAdapter() {
    @Override
    public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ESCAPE) {
            closeDialog();
        }
    }
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Prefer key bindings | `getInputMap(...).put(KeyStroke, actionName)` + `getActionMap().put(actionName, action)` |
| Use `WHEN_IN_FOCUSED_WINDOW` for global shortcuts | `getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)` |
| Use `Action` objects | `new AbstractAction("Command") { ... }` |
| Use `KeyListener` only for low-level | Custom components that need raw key state. |
| Check `getKeyCode()` in `keyPressed`/`keyReleased` | Not `getKeyChar()` (which is for `keyTyped`). |
| Check `getKeyChar()` in `keyTyped` | Not `getKeyCode()` (which is `VK_UNDEFINED` for typed events). |
| Consume events when needed | `e.consume()` to prevent default handling. |
| Remove listeners on disposal | `removeKeyListener(...)` when no longer needed. |
| EDT | All key handling runs on the EDT. |

#### Syntax Rules

- **Always prefer key bindings over `KeyListener`** for application-level keyboard shortcuts.
- **Always use `getKeyCode()` in `keyPressed`/`keyReleased`** and **`getKeyChar()` in `keyTyped`**; do not mix them up.
- **Always use `WHEN_IN_FOCUSED_WINDOW`** for shortcuts that should work regardless of which component has focus.
- **Always use `Action` objects with key bindings** to encapsulate command logic and enabled state.
- **Always consume events** in `KeyListener` when you want to prevent default handling.
- **Always remove key listeners** from long-lived components when they are no longer needed.

#### Constraints and Limitations

- `KeyListener` only receives events when the component has focus; this is the primary reason key bindings are preferred.
- `keyTyped` does not fire for non-character keys; use `keyPressed` for arrow keys, function keys, and modifiers.
- `KeyListener` does not integrate with Swing's `Action` enabled-state mechanism.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Key Bindings Instead of KeyListener

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.InputEvent;
import java.awt.event.KeyEvent;

public class BestPracticeKeyBindings {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeKeyBindings::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Key Bindings");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JPanel panel = new JPanel();
        panel.setFocusable(true);

        Action escapeAction = new AbstractAction("Close") {             // 1. Action
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Escape pressed — closing");
                frame.dispose();
            }
        };

        panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)            // 2. InputMap
            .put(KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE, 0),
                 "closeAction");
        panel.getActionMap().put("closeAction", escapeAction);          // 3. ActionMap

        JButton focusButton = new JButton("Click to focus elsewhere");
        panel.add(focusButton);

        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Pressing Escape anywhere in the window (regardless of which component has focus) triggers the “Close” action and disposes the frame.

**Why the code produces this result.** The `InputMap` binds the Escape key to the action name `"closeAction"`, and the `ActionMap` maps that name to the `Action`. The `WHEN_IN_FOCUSED_WINDOW` condition ensures the binding is active even when the focus is on a child component. This is the recommended approach; `KeyListener` on the panel would not fire when the button has focus.

---

#### Example 2: Common Mistake — Using KeyListener for a Global Shortcut

```java
import javax.swing.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class BadKeyListenerShortcut {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad KeyListener Shortcut");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.addKeyListener(new KeyAdapter() {                     // WRONG
                @Override
                public void keyPressed(KeyEvent e) {
                    if (e.getKeyCode() == KeyEvent.VK_ESCAPE) {
                        System.out.println("Escape pressed");
                    }
                }
            });

            JButton button = new JButton("Click me first");
            panel.add(button);

            frame.add(panel);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Escape works only if the panel itself has focus. If the button (or any other child) has focus, the `KeyListener` does not receive the event, and nothing happens.

**Why this is wrong.** `KeyListener` is focus-dependent. The panel only receives key events when it is the focus owner; when a child component has focus, the key events go to that child instead. The correct approach is to use key bindings with `WHEN_IN_FOCUSED_WINDOW`, which ensure the binding is active regardless of focus.

---

#### Example 3: Correct Use of KeyListener for a Low-Level Custom Component

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;

public class LegitimateKeyListener {
    private static int x = 100, y = 100;
    private static JPanel canvas;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Low-Level KeyListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            canvas = new JPanel() {
                @Override
                protected void paintComponent(Graphics g) {
                    super.paintComponent(g);
                    g.setColor(Color.RED);
                    g.fillOval(x - 10, y - 10, 20, 20);
                }
            };
            canvas.setBackground(Color.WHITE);
            canvas.setPreferredSize(new Dimension(400, 300));
            canvas.setFocusable(true);                                   // 1. Focusable

            canvas.addKeyListener(new KeyAdapter() {                     // 2. KeyListener
                @Override
                public void keyPressed(KeyEvent e) {
                    switch (e.getKeyCode()) {
                        case KeyEvent.VK_LEFT:  x -= 10; canvas.repaint(); break;
                        case KeyEvent.VK_RIGHT: x += 10; canvas.repaint(); break;
                        case KeyEvent.VK_UP:    y -= 10; canvas.repaint(); break;
                        case KeyEvent.VK_DOWN:  y += 10; canvas.repaint(); break;
                    }
                }
            });

            canvas.requestFocusInWindow();                               // 3. Focus
            frame.add(canvas);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A red dot moves by 10 pixels in the direction of the arrow keys, but only while the canvas has focus.

**Why the code produces this result.** This is a legitimate use of `KeyListener`: a custom painting canvas that needs raw key state for continuous movement (not a command-action pattern). The canvas is made focusable and requests focus so it receives key events. For command shortcuts, key bindings would still be preferred, but for game-style continuous key handling, `KeyListener` is appropriate.

---

### Real-World Cases

- **Game development:** `KeyListener` on a focusable canvas for continuous key-hold movement; key bindings for discrete commands (pause, restart).
- **IDE and editor plugins:** key bindings for all shortcuts; `KeyListener` only for raw key capture in specialized custom components.
- **Production applications:** key bindings with `WHEN_IN_FOCUSED_WINDOW` for global shortcuts like Ctrl+S (save) or F1 (help).
- **Educational examples:** demonstrate both approaches, with emphasis on the key-binding preference.

---

## References

Oracle. *Interface KeyListener*. Java Platform, Standard Edition 8 API Specification. https://cr.openjdk.org/~bpatel/docs/api/java/awt/event/KeyListener.html

Oracle. *Class KeyAdapter*. Java Platform, Standard Edition 8 API Specification. https://cr.openjdk.org/~bpatel/8068491/webrev.00/src/share/classes/java/awt/event/KeyAdapter.java.sdiff.html

Oracle. *Class KeyEvent*. Java SE 9 API Specification. https://cr.openjdk.org/~iris/se/9/java-se-9-fr-spec-01/api/java/awt/event/KeyEvent.html

Oracle. *How to Use Key Bindings*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/keybinding.html

Oracle. *Handling Keyboard Events*. Oracle ADF Swing Whitepaper. https://www.oracle.com/technetwork/developer-tools/jdev/swingkeyboardshortcuts-133803.pdf

Stack Overflow. *Key bindings vs. key listeners in Java*. https://stackoverflow.com/revisions/1105e02a-12b7-40b1-8364-a7cc8b4da180/view-source

Oracle. *How to Write a Key Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/keylistener.html