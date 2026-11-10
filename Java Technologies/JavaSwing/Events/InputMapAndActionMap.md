# InputMap and ActionMap — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `InputMap` and `ActionMap` are two cooperating classes in the `javax.swing` package that together implement **key bindings**—the recommended mechanism for associating keystrokes with actions in Swing. An `InputMap` maps a `KeyStroke` (a specific key press with modifier state) to a **command name** (an arbitrary `Object`, typically a `String`). An `ActionMap` maps that same command name to an `Action` whose `actionPerformed` method runs when the keystroke is pressed. Together they replace the obsolete `registerKeyboardAction` methods and are strongly preferred over `KeyListener` for keyboard shortcuts.

**Technical Definition.** `public class InputMap implements Serializable` maintains a mapping from `KeyStroke` objects to `Object` command names. `public class ActionMap implements Serializable` maintains a mapping from `Object` command names to `Action` objects. Every `JComponent` has three input maps—one for each of the conditions `WHEN_FOCUSED`, `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`, and `WHEN_IN_FOCUSED_WINDOW`—and one action map. The component's `processKeyBinding` method searches the appropriate input map when a key is pressed, finds the bound command name, looks it up in the action map, and calls `actionPerformed` on the resulting `Action`. The processing order is: the focused component's `WHEN_FOCUSED` map; then, walking up the ancestor hierarchy, each ancestor's `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` map; finally, every component in the window's `WHEN_IN_FOCUSED_WINDOW` map. Key bindings are the recommended alternative to `KeyListener` because they "are self-documenting, take the containment hierarchy into account, encourage reusable `Action` objects, allow actions to be easily removed, customized, or shared, and make it easy to change the key to which an action is bound".

**Beginner-Friendly Explanation.** `InputMap` and `ActionMap` are Swing's two-step system for keyboard shortcuts. Imagine a two-page address book: page one (`InputMap`) maps a keyboard shortcut like **Ctrl+S** to a *name* like “save-command”. Page two (`ActionMap`) maps that *name* to a piece of code (an `Action`) that actually saves the file. When the user presses Ctrl+S, Swing consults the `InputMap` to find the name, then consults the `ActionMap` to find the action, and finally runs the action. Why two steps? Because the indirection lets you (a) rebind the key without touching the action, (b) share one action across many keys, and (c) enable or disable the action independently of the key binding. The three **conditions** let you decide when the shortcut should be active: only when the component has focus (`WHEN_FOCUSED`), when any descendant has focus (`WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`), or whenever the window has focus (`WHEN_IN_FOCUSED_WINDOW`). Think of them as a “keyboard shortcut filing system” that Swing searches top-down whenever a key is pressed.

---

### Key Characteristics

1. **Two-step indirection.** `InputMap` maps `KeyStroke` → command name; `ActionMap` maps command name → `Action`. The indirection decouples the binding from the action, enabling rebinding and sharing.
2. **Three input maps per component.** Every `JComponent` provides `getInputMap(int condition)` for `WHEN_FOCUSED`, `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`, and `WHEN_IN_FOCUSED_WINDOW`; and `getActionMap()` for the single action map.
3. **Hierarchy-aware search.** When a key is pressed, Swing searches the focused component's `WHEN_FOCUSED` map; then, walking up the ancestor chain, each ancestor's `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` map; finally, all components in the window with `WHEN_IN_FOCUSED_WINDOW` bindings.
4. **Uses `KeyStroke`, not key codes.** A `KeyStroke` combines a `KeyEvent.VK_*` code with modifier masks (`CTRL_DOWN_MASK`, `SHIFT_DOWN_MASK`, `ALT_DOWN_MASK`, `META_DOWN_MASK`). `KeyStroke.getKeyStroke(...)` is the standard factory method.
5. **Replaces `registerKeyboardAction`.** The obsolete `JComponent.registerKeyboardAction` methods are superseded by `InputMap`/`ActionMap`, which are clearer and more flexible.
6. **Preferred over `KeyListener`.** Key bindings work regardless of focus (with the right condition), integrate with `Action` state, and are easier to maintain.
7. **Built-in defaults.** Every look and feel provides default bindings (e.g., arrow keys in `JTable`, Tab navigation, Space to activate buttons). The three input maps each form a **chain** that is consulted in order.
8. **EDT requirement.** All input and action map manipulation should occur on the EDT.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JComponent`, `Action`, and `AbstractAction`.
- Understanding of `KeyEvent` virtual key codes and `InputEvent` modifier masks.
- Familiarity with the EDT and `SwingUtilities.invokeLater`.
- Knowledge of `KeyStroke` construction.
- A JDK (version 8 or later; `InputMap`, `ActionMap`, and `KeyStroke` are in `javax.swing`).

---

### Related Programming Areas

- **`KeyStroke`:** The keystroke representation stored as keys in `InputMap`.
- **`Action` and `AbstractAction`:** The command objects stored as values in `ActionMap`.
- **`KeyListener`:** The low-level alternative; key bindings are preferred for shortcuts.
- **`JComponent.registerKeyboardAction`:** The obsolete predecessor of `InputMap`/`ActionMap`.
- **`InputMap` chains and parent maps:** Custom input maps fall back to look-and-feel defaults.
- **`processKeyBinding`:** The internal method that performs the search and dispatch.

---

## Core Concepts and Features

---

## 1. InputMap — The Keystroke-to-Command Map

### Definitions

**Core Definition.** `InputMap` maps a `KeyStroke` to a command name (an arbitrary `Object`). When a keystroke is pressed and the input map is active (based on its condition), Swing looks up the corresponding command name.

**Technical Definition.** `public class InputMap implements Serializable`. It provides `put(KeyStroke keyStroke, Object actionMapKey)`, `get(KeyStroke keyStroke)`, `remove(KeyStroke keyStroke)`, `allKeys()`, `clear()`, `setParent(InputMap parent)`, and `getParent()`. Each `JComponent` exposes three input maps via `getInputMap(int condition)`. The condition constants are `WHEN_FOCUSED`, `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`, and `WHEN_IN_FOCUSED_WINDOW`.

**Beginner-Friendly Explanation.** An `InputMap` is a dictionary from keyboard shortcuts to command names. You put a `KeyStroke` and a name into it; when the user presses that key, Swing finds the name.

---

### Purposes

- To map a specific keystroke to a command name that the `ActionMap` can resolve.
- To provide three levels of binding scope via the condition constants.
- To allow custom and default bindings to coexist via the parent map chain.
- To enable programmatic inspection and modification of key bindings.
- To decouple keystrokes from actions, allowing either to change independently.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
InputMap im = component.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW);
im.put(KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK),
       "saveAction");
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getInputMap(int condition)` | Returns the input map for the given condition. |
| `WHEN_FOCUSED` | Active only when the component itself has focus. |
| `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` | Active when the component or any descendant has focus. |
| `WHEN_IN_FOCUSED_WINDOW` | Active whenever the component's window has focus. |
| `put(KeyStroke, Object)` | Binds the keystroke to the command name. |
| `get(KeyStroke)` | Returns the bound command name, or `null`. |
| `remove(KeyStroke)` | Removes the binding. |
| `allKeys()` | Returns all bound keystrokes. |
| `setParent(InputMap)` / `getParent()` | Manages the parent chain. |

#### Syntax Rules

- The command name is typically a `String`, but any `Object` works; the same object must be used in the `ActionMap`.
- Each condition has its own input map; bindings do not automatically transfer between conditions.
- Custom input maps chain to look-and-feel defaults through the parent map.

#### Constraints and Limitations

- Bindings in one condition do not apply to another.
- Removing a binding from a custom input map does not prevent the parent (look-and-feel) binding from firing.
- Some keystrokes are consumed by the look and feel before reaching custom bindings.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Binding a Keystroke to a Command Name

```java
import javax.swing.*;
import java.awt.event.*;

public class InputMapExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("InputMap Example");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.setFocusable(true);

            InputMap im = panel.getInputMap(
                JComponent.WHEN_IN_FOCUSED_WINDOW);              // 1. Condition
            im.put(KeyStroke.getKeyStroke(KeyEvent.VK_S,
                    InputEvent.CTRL_DOWN_MASK),                   // 2. KeyStroke
                   "saveCommand");                                // 3. Command name

            frame.add(panel);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The binding is installed, but no action fires yet because no `ActionMap` entry resolves `"saveCommand"` (the next section completes the pattern).

**Why the code produces this result.** `InputMap.put` stores the keystroke-to-name mapping. Without an `ActionMap` entry, the dispatch machinery finds the name but has nothing to invoke.

---

### Real-World Cases

- **Global shortcuts** (Ctrl+S, Ctrl+O, F1) bound at `WHEN_IN_FOCUSED_WINDOW`.
- **Component-local shortcuts** (arrow keys, Enter) bound at `WHEN_FOCUSED`.
- **Custom widgets** that need their own key handling without interfering with other components.

---

## 2. ActionMap — The Command-to-Action Map

### Definitions

**Core Definition.** `ActionMap` maps a command name to an `Action`. When the `InputMap` resolves a keystroke to a command name, Swing looks up the action in the component's `ActionMap` and invokes it.

**Technical Definition.** `public class ActionMap implements Serializable`. It provides `put(Object key, Action action)`, `get(Object key)`, `remove(Object key)`, `allKeys()`, `clear()`, `setParent(ActionMap parent)`, and `getParent()`. Each `JComponent` exposes its action map via `getActionMap()`. The action map also chains to look-and-feel defaults through its parent map.

**Beginner-Friendly Explanation.** An `ActionMap` is a dictionary from command names to `Action` objects. The `InputMap` gives you the name; the `ActionMap` gives you the code.

---

### Purposes

- To map command names to the `Action` objects that implement them.
- To share a single `Action` across multiple keystrokes, menu items, and toolbar buttons.
- To enable or disable commands via `Action.setEnabled`.
- To allow the `InputMap` to rebind keystrokes without changing actions.
- To chain to look-and-feel defaults through the parent map.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
ActionMap am = component.getActionMap();
am.put("saveCommand", saveAction);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getActionMap()` | Returns the component's action map. |
| `put(Object, Action)` | Maps the command name to the action. |
| `get(Object)` | Returns the action for the given name. |
| `remove(Object)` | Removes the mapping. |
| `setParent(ActionMap)` / `getParent()` | Manages the parent chain. |

#### Syntax Rules

- The command name must exactly match the object used in `InputMap.put`.
- The action’s `actionPerformed` method is invoked when the binding fires.
- The action map chains to defaults; removing a custom entry may expose a look-and-feel binding.

#### Constraints and Limitations

- The same command name used in a custom action map shadows any look-and-feel action of the same name.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Complete InputMap + ActionMap Pair

```java
import javax.swing.*;
import java.awt.event.*;

public class ActionMapExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("ActionMap Example");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.setFocusable(true);

            InputMap im = panel.getInputMap(
                JComponent.WHEN_IN_FOCUSED_WINDOW);              // 1. InputMap
            im.put(KeyStroke.getKeyStroke(KeyEvent.VK_S,
                    InputEvent.CTRL_DOWN_MASK), "saveCommand");

            ActionMap am = panel.getActionMap();                  // 2. ActionMap
            am.put("saveCommand", new AbstractAction("Save") {    // 3. Action
                @Override
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Saving via Ctrl+S");
                }
            });

            frame.add(panel);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Ctrl+S anywhere in the window prints “Saving via Ctrl+S” to the console.

**Why the code produces this result.** The `InputMap` binds Ctrl+S to `"saveCommand"` under `WHEN_IN_FOCUSED_WINDOW`. The `ActionMap` binds `"saveCommand"` to the anonymous `AbstractAction`. When Ctrl+S is pressed, Swing finds the name and runs the action.

---

### Real-World Cases

- **Global commands** shared between keyboard shortcuts, menus, and toolbars.
- **Context-sensitive actions** whose enabled state changes with selection.
- **Look-and-feel overrides** that replace default key bindings.

---

## 3. Key Bindings — The Combined System

### Definitions

**Core Definition.** A **key binding** is a complete `InputMap` + `ActionMap` pair: the `InputMap` maps a `KeyStroke` to a command name, and the `ActionMap` maps that name to an `Action`. Together they implement a keyboard shortcut.

**Technical Definition.** When a key is pressed, Swing calls `JComponent.processKeyBinding`. The method searches the appropriate input map (based on the condition), resolves the command name, looks it up in the action map, and invokes `actionPerformed`. If the action is disabled, the binding is treated as not bound, allowing the search to continue up the chain. The three conditions define the scope of the search:

- `WHEN_FOCUSED`: only the focused component's map.
- `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`: the focused component and each of its ancestors up to the window.
- `WHEN_IN_FOCUSED_WINDOW`: every component in the window that has bindings under this condition.

**Beginner-Friendly Explanation.** Key bindings are the “complete shortcut” assembled from the two maps. The three conditions let you decide how focused the user must be for the shortcut to work: on the component, on the component or its children, or anywhere in the window.

---

### Purposes

- To provide keyboard shortcuts that work regardless of which component has focus (with the correct condition).
- To integrate shortcuts with the `Action` architecture, sharing commands with menus and toolbars.
- To allow rebinding without touching the action logic.
- To respect look-and-feel defaults through the input and action map chains.
- To replace `KeyListener` and `registerKeyboardAction` with a cleaner, more maintainable mechanism.

---

### Syntax Rules and Structure

#### Complete General Syntax (All Three Conditions)

```java
// WHEN_FOCUSED — only when this component has focus
component.getInputMap(JComponent.WHEN_FOCUSED)
         .put(KeyStroke.getKeyStroke(KeyEvent.VK_ENTER, 0), "enterAction");

// WHEN_ANCESTOR_OF_FOCUSED_COMPONENT — when this component or a child has focus
component.getInputMap(JComponent.WHEN_ANCESTOR_OF_FOCUSED_COMPONENT)
         .put(KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE, 0), "escapeAction");

// WHEN_IN_FOCUSED_WINDOW — whenever the window has focus
component.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)
         .put(KeyStroke.getKeyStroke(KeyEvent.VK_F1, 0), "helpAction");

component.getActionMap().put("enterAction", enterAction);
component.getActionMap().put("escapeAction", escapeAction);
component.getActionMap().put("helpAction", helpAction);
```

#### Component Breakdown

| Condition | Scope |
|-----------|-------|
| `WHEN_FOCUSED` | Only the component with focus. |
| `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` | The component and any ancestor of the focused component. |
| `WHEN_IN_FOCUSED_WINDOW` | Every component in the window with such a binding, regardless of focus. |

#### Syntax Rules

- All three conditions share the same `ActionMap`; only the `InputMap` differs per condition.
- The action map is per-component but chains to the look and feel's defaults.
- If an action is disabled, the binding is not considered bound and the search continues.
- Bindings registered under `WHEN_IN_FOCUSED_WINDOW` are searched after `WHEN_FOCUSED` and `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` bindings.

#### Constraints and Limitations

- Multiple `WHEN_IN_FOCUSED_WINDOW` bindings for the same keystroke in the same window are undefined; the last one registered may win, or dispatch order may vary.
- Some look-and-feel bindings (e.g., Tab for focus traversal) may consume keystrokes before custom bindings fire.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: All Three Conditions in One Application

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class AllConditions {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("All Conditions");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.setFocusable(true);
            JTextField field = new JTextField(10);
            panel.add(field);

            Action enterAction = new AbstractAction("Enter") {
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Enter (WHEN_FOCUSED)");
                }
            };
            Action escapeAction = new AbstractAction("Escape") {
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Escape (ANCESTOR)");
                }
            };
            Action helpAction = new AbstractAction("Help") {
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Help (IN_FOCUSED_WINDOW)");
                }
            };

            panel.getInputMap(JComponent.WHEN_FOCUSED)
                 .put(KeyStroke.getKeyStroke(KeyEvent.VK_ENTER, 0), "enter");
            panel.getInputMap(JComponent.WHEN_ANCESTOR_OF_FOCUSED_COMPONENT)
                 .put(KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE, 0), "escape");
            panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)
                 .put(KeyStroke.getKeyStroke(KeyEvent.VK_F1, 0), "help");

            ActionMap am = panel.getActionMap();
            am.put("enter", enterAction);
            am.put("escape", escapeAction);
            am.put("help", helpAction);

            frame.add(panel);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.**
- Focus on the panel itself, press Enter: prints “Enter (WHEN_FOCUSED)”.
- Focus on the text field, press Escape: prints “Escape (ANCESTOR)” (because the panel is an ancestor of the focused text field).
- Focus anywhere in the window, press F1: prints “Help (IN_FOCUSED_WINDOW)”.

**Why the code produces this result.** Each binding is registered under a different condition. The dispatch search finds the appropriate binding based on the current focus and the condition's scope.

---

### Real-World Cases

- **Application-wide shortcuts:** F1 for help, Ctrl+Q to quit, Ctrl+Comma for preferences—all bound under `WHEN_IN_FOCUSED_WINDOW`.
- **Component-local editing:** Enter to submit a text field (via `WHEN_FOCUSED`).
- **Container-level navigation:** Escape to cancel a form (via `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT`).

---

## 4. KeyStroke — The Keystroke Representation

### Definitions

**Core Definition.** `KeyStroke` represents a single key press or release on a specific key with modifier state. It is the key type of an `InputMap`.

**Technical Definition.** `public class KeyStroke extends AWTKeyStroke`. `KeyStroke.getKeyStroke(...)` is the standard factory method; `AWTKeyStroke.getAWTKeyStroke(...)` also works. The `KeyStroke` encapsulates a `KeyEvent.VK_*` key code and modifier masks (`InputEvent.SHIFT_DOWN_MASK`, `CTRL_DOWN_MASK`, `ALT_DOWN_MASK`, `META_DOWN_MASK`). It also records whether the stroke is a press (`KEY_PRESSED`) or release (`KEY_RELEASED`)—typed events are not supported by `KeyStroke`.

**Beginner-Friendly Explanation.** `KeyStroke` is the “specific key combination” object. It says “the S key with Ctrl held down” or “the F1 key with no modifiers” or “the Left arrow key”. You construct one with `KeyStroke.getKeyStroke(...)` and use it as a key in the `InputMap`.

---

### Purposes

- To represent a specific keystroke for binding in an `InputMap`.
- To support modifier combinations (Ctrl, Shift, Alt, Meta).
- To support both key-press and key-release bindings.
- To serve as the value for `Action.ACCELERATOR_KEY`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
KeyStroke ks1 = KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK);
KeyStroke ks2 = KeyStroke.getKeyStroke(KeyEvent.VK_F1, 0);
KeyStroke ks3 = KeyStroke.getKeyStroke("ctrl S");
KeyStroke ks4 = KeyStroke.getKeyStroke("pressed F1");
```

#### Component Breakdown

| Factory Overload | Meaning |
|------------------|---------|
| `getKeyStroke(int keyCode, int modifiers)` | `KEY_PRESSED` with the given key and modifiers. |
| `getKeyStroke(int keyCode, int modifiers, boolean onKeyRelease)` | Specifies whether the stroke is on release. |
| `getKeyStroke(String s)` | Parses a string such as `"ctrl S"` or `"pressed F1"`. |
| `getKeyStroke(char c)` | A stroke for a typed character. |
| `getKeyStrokeForEvent(KeyEvent e)` | A stroke matching the given event. |

#### Syntax Rules

- Modifier masks are combined with bitwise OR: `InputEvent.CTRL_DOWN_MASK | InputEvent.SHIFT_DOWN_MASK`.
- String form uses lowercase modifiers and key names: `"ctrl shift S"`, `"alt F4"`.
- `KeyStroke` is immutable and safe to reuse.

#### Constraints and Limitations

- `KeyStroke` does not support `KEY_TYPED` events; use `getKeyStroke(char)` for typed-character bindings, or handle the typed event via a `KeyListener` if truly needed.
- `getKeyStrokeForEvent` is useful for rebinding but should not be stored as a binding—it describes an event, not a fixed stroke.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Multiple KeyStroke Forms

```java
import javax.swing.*;
import java.awt.event.*;

public class KeyStrokeForms {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("KeyStroke Forms");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.setFocusable(true);

            InputMap im = panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW);
            im.put(KeyStroke.getKeyStroke("ctrl S"),     "save");       // 1. String
            im.put(KeyStroke.getKeyStroke("ctrl shift S"), "saveAll");  // 2. String
            im.put(KeyStroke.getKeyStroke(KeyEvent.VK_F1, 0), "help");  // 3. Code
            im.put(KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE,
                InputEvent.ALT_DOWN_MASK), "escape");                  // 4. Modifier

            ActionMap am = panel.getActionMap();
            am.put("save",    logAction("Save"));
            am.put("saveAll", logAction("Save All"));
            am.put("help",    logAction("Help"));
            am.put("escape",  logAction("Escape"));

            frame.add(panel);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static Action logAction(String name) {
        return new AbstractAction(name) {
            public void actionPerformed(ActionEvent e) {
                System.out.println(name + " invoked");
            }
        };
    }
}
```

**Expected Output.** Pressing Ctrl+S prints “Save invoked”; Ctrl+Shift+S prints “Save All invoked”; F1 prints “Help invoked”; Alt+Escape prints “Escape invoked”.

**Why the code produces this result.** `KeyStroke.getKeyStroke` supports both the numeric form (with `InputEvent` modifier masks) and the string form (`"ctrl S"`). All four strokes are bound to distinct command names and actions.

---

### Real-World Cases

- **Menu accelerators:** Stored as `Action.ACCELERATOR_KEY` and used by `InputMap` bindings.
- **Rebinding preference dialogs:** Display the current `KeyStroke` as text and capture a new one with `getKeyStrokeForEvent`.
- **Look-and-feel key bindings:** All built-in bindings use `KeyStroke`.

---

## 5. Key Binding Best Practices

### Definitions

**Core Definition.** Key binding best practices are the conventions and patterns that ensure keyboard shortcuts are correct, conflict-free, and consistent with the application's command architecture.

**Technical Definition.** Best practices include: preferring `InputMap`/`ActionMap` over `KeyListener` and `registerKeyboardAction`; using `WHEN_IN_FOCUSED_WINDOW` for global shortcuts; sharing `Action` objects between key bindings, menus, and toolbars; using the string form of `KeyStroke.getKeyStroke` for readability; inspecting existing bindings before overriding look-and-feel defaults; and removing bindings when they are no longer needed.

**Beginner-Friendly Explanation.** Best practices for key bindings are the “rules of thumb” that prevent keyboard conflicts, ignored shortcuts, and unmaintainable code. They include things like “use key bindings, not `KeyListener`,” “pick the right condition,” and “share the action with the menu item.”

---

### Purposes

- To use the modern, maintainable mechanism (`InputMap`/`ActionMap`) instead of legacy alternatives.
- To avoid keyboard conflicts by inspecting existing bindings.
- To ensure shortcuts work regardless of which child has focus.
- To keep commands synchronized across keyboard, menu, and toolbar.
- To prevent memory leaks by cleaning up bindings when components are disposed.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
Action saveAction = new AbstractAction("Save") {                   // 1. Action
    public void actionPerformed(ActionEvent e) { saveDocument(); }
};

JComponent root = panel;
root.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)               // 2. Condition
    .put(KeyStroke.getKeyStroke("ctrl S"), "save");                // 3. Keystroke
root.getActionMap().put("save", saveAction);                       // 4. Action
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Prefer key bindings over `KeyListener` | `InputMap`/`ActionMap` |
| Avoid `registerKeyboardAction` | It is obsolete |
| Use `WHEN_IN_FOCUSED_WINDOW` for global | `getInputMap(WHEN_IN_FOCUSED_WINDOW)` |
| Use `WHEN_FOCUSED` for component-local | `getInputMap(WHEN_FOCUSED)` |
| Share `Action` objects | Same `Action` in menu, toolbar, and binding |
| Inspect existing bindings | `im.allKeys()` to check for conflicts |
| Use string `KeyStroke` form | `KeyStroke.getKeyStroke("ctrl S")` |
| Clean up bindings | `im.remove(keystroke)` when done |
| EDT | `SwingUtilities.invokeLater(...)` for construction |

#### Syntax Rules

- **Always prefer `InputMap`/`ActionMap`** over `KeyListener` or `registerKeyboardAction` for shortcuts.
- **Always choose the correct condition**: `WHEN_FOCUSED` for component-local keys, `WHEN_IN_FOCUSED_WINDOW` for application-wide shortcuts.
- **Always share `Action` objects** between the key binding, the menu item, and the toolbar button, so all three stay synchronized.
- **Always inspect existing bindings** (via `InputMap.allKeys()` and `InputMap.get(keystroke)`) before overriding look-and-feel defaults.
- **Always use the string form of `KeyStroke.getKeyStroke`** for readability when possible.
- **Always remove bindings** from long-lived components when they are no longer needed.

#### Constraints and Limitations

- Multiple `WHEN_IN_FOCUSED_WINDOW` bindings for the same keystroke in the same window are undefined; ensure uniqueness.
- Bindings that conflict with look-and-feel defaults may be shadowed or may shadow the defaults, depending on the condition and registration order.
- `KeyStroke` cannot represent typed-character events directly.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Global Shortcut with Shared Action

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class BestPracticeKeyBinding {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeKeyBinding::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Key Binding");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action saveAction = new AbstractAction("Save") {              // 1. Shared Action
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Saving...");
            }
        };

        JPanel panel = new JPanel(new BorderLayout());
        panel.setFocusable(true);

        panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)          // 2. Condition
             .put(KeyStroke.getKeyStroke("ctrl S"), "save");          // 3. Keystroke
        panel.getActionMap().put("save", saveAction);                 // 4. Action

        JButton button = new JButton(saveAction);                     // 5. Share
        JMenuItem menuItem = new JMenuItem(saveAction);
        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        file.add(menuItem);
        bar.add(file);
        frame.setJMenuBar(bar);

        panel.add(button, BorderLayout.CENTER);
        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Pressing Ctrl+S, clicking the “Save” button, or choosing “File → Save” all execute the same action and print “Saving...”.

**Why the code produces this result.** The single `Action` is shared by the `InputMap`/`ActionMap` binding, the `JButton`, and the `JMenuItem`. All three paths invoke the same `actionPerformed` method.

---

#### Example 2: Common Mistake — Using `KeyListener` for a Global Shortcut

```java
import javax.swing.*;
import java.awt.event.*;

public class BadKeyListenerShortcut {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Shortcut");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel();
            panel.setFocusable(true);
            panel.addKeyListener(new KeyAdapter() {                   // WRONG
                @Override
                public void keyPressed(KeyEvent e) {
                    if (e.isControlDown() && e.getKeyCode() == KeyEvent.VK_S) {
                        System.out.println("Save via KeyListener");
                    }
                }
            });

            JButton button = new JButton("Click me first");
            panel.add(button);
            frame.add(panel);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Ctrl+S works only if the panel itself has focus. Once the user clicks the button, the panel loses focus, and Ctrl+S stops working.

**Why this is wrong.** `KeyListener` is focus-dependent. Key bindings registered under `WHEN_IN_FOCUSED_WINDOW` work regardless of which component has focus, and integrate with the `Action` architecture for shared command state. The correct approach is `InputMap`/`ActionMap` with `WHEN_IN_FOCUSED_WINDOW`.

---

#### Example 3: Common Mistake — Conflicting Bindings

```java
import javax.swing.*;
import java.awt.event.*;

public class BadConflictingBindings {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Conflicting Bindings");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel a = new JPanel();
            JPanel b = new JPanel();
            a.setFocusable(true);
            b.setFocusable(true);

            // WRONG: two WHEN_IN_FOCUSED_WINDOW bindings for the same key
            a.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)
             .put(KeyStroke.getKeyStroke("ctrl S"), "saveA");
            b.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW)
             .put(KeyStroke.getKeyStroke("ctrl S"), "saveB");

            a.getActionMap().put("saveA", logAction("A"));
            b.getActionMap().put("saveB", logAction("B"));

            frame.setLayout(new java.awt.GridLayout(2, 1));
            frame.add(a);
            frame.add(b);
            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static Action logAction(String name) {
        return new AbstractAction(name) {
            public void actionPerformed(ActionEvent e) {
                System.out.println("Save " + name);
            }
        };
    }
}
```

**Expected Output.** Only one of the two actions fires when Ctrl+S is pressed, and which one is undefined—the behaviour depends on registration order and dispatch mechanism.

**Why this is wrong.** Multiple `WHEN_IN_FOCUSED_WINDOW` bindings for the same keystroke in the same window cause conflicting dispatch. The correct approach is to bind application-wide shortcuts on a **single** component (e.g., the frame's root pane or a top-level panel) and share actions, or to use `WHEN_FOCUSED`/`WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` for context-sensitive bindings.

---

### Real-World Cases

- **Production Swing applications** bind all application-wide shortcuts on the frame's root pane or content pane under `WHEN_IN_FOCUSED_WINDOW`, sharing actions with menus and toolbars.
- **Editor components** use `WHEN_FOCUSED` for editing-specific shortcuts (e.g., Enter to insert a newline).
- **Dialogs** use `WHEN_IN_FOCUSED_WINDOW` for Escape-to-cancel and `WHEN_ANCESTOR_OF_FOCUSED_COMPONENT` for form-level navigation.

---

## References

Oracle. *Class InputMap*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/InputMap.html

Oracle. *Class ActionMap*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ActionMap.html

Oracle. *Class KeyStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/KeyStroke.html

Oracle. *How to Use Key Bindings*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/keybinding.html

Oracle. *How to Use Actions*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/action.html

Oracle. *Class JComponent — registerKeyboardAction (obsolete)*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *Class JComponent — getInputMap / getActionMap*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html

Oracle. *How to Use Key Bindings* (mirror). https://www.cse.iitb.ac.in/~cs296/Java/SunJavaTutorial/uiswing/misc/keybinding.html

Oracle. *Class KeyEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/KeyEvent.html

Oracle. *Class InputEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/InputEvent.html

Oracle. *Class AbstractAction*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractAction.html

Oracle. *Class JComponent — processKeyBinding*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JComponent.html