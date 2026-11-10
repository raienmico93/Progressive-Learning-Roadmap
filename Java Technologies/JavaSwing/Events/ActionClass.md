# Action Class — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `Action` is an interface in the `javax.swing` package that encapsulates a command’s **logic and state**—text, icon, tooltip, mnemonic, accelerator, enabled flag, and the action code itself—into a single reusable object. `AbstractAction` is the standard abstract implementation that provides a default property map and property-change support.

**Technical Definition.** `public interface Action extends ActionListener, Serializable`. Because `Action` extends `ActionListener`, every action must implement `actionPerformed(ActionEvent e)`—the same method used by `ActionListener`. `Action` adds `getValue(String key)`, `putValue(String key, Object value)`, `getKeys()`, `isEnabled()`, `setEnabled(boolean)`, `addPropertyChangeListener(PropertyChangeListener)`, and `removePropertyChangeListener(PropertyChangeListener)`. The property keys are predefined `String` constants: `NAME`, `SHORT_DESCRIPTION`, `LONG_DESCRIPTION`, `SMALL_ICON`, `LARGE_ICON_KEY`, `ACCELERATOR_KEY`, `MNEMONIC_KEY`, `ACTION_COMMAND_KEY`, and `ENABLED`. `AbstractAction` implements the property map with a `HashMap`-like store and provides the property-change firing. Components such as `JButton`, `JMenuItem`, `JCheckBoxMenuItem`, `JRadioButtonMenuItem`, and `JToolBar` can be constructed from an `Action` and automatically synchronize their text, icon, tooltip, mnemonic, accelerator, action command, and enabled state with the action.

**Beginner-Friendly Explanation.** `Action` is Swing’s answer to duplicated command logic. Suppose your application has a “Save” command available from a menu item, a toolbar button, and a keyboard shortcut (Ctrl+S). Without `Action`, you would write the save logic three times and keep the enabled state of three components manually synchronized. With `Action`, you write **one** `AbstractAction` that holds the command logic, the label “Save,” the icon, the Ctrl+S accelerator, the Alt+S mnemonic, and the enabled state. You then pass that single action to a `JMenuItem`, a `JButton`, and a keyboard binding—each component automatically reads the action’s properties and stays synchronized when they change. Think of an `Action` as a **reusable command object** that carries everything the UI needs to present and execute it.

---

### Key Characteristics

1. **Command-object pattern.** `Action` is Swing’s implementation of the Command design pattern—a single object encapsulates the request, its presentation, and its enabled state.
2. **Extends `ActionListener`.** An `Action` is also an `ActionListener`, so it can be registered directly with components or used as the listener for keyboard bindings.
3. **Predefined property keys.** Nine standard keys define the action’s text, tooltip, icons, accelerator, mnemonic, action command, and enabled flag. These keys are read by `JButton`, `JMenuItem`, and other components to configure themselves automatically.
4. **Property-change support.** `Action` provides `addPropertyChangeListener` and `removePropertyChangeListener`; components register listeners to update their own presentation when an action property changes.
5. **Shared enabled state.** Calling `setEnabled(false)` on an action disables **every** component constructed from it—menu items, toolbar buttons, and keyboard bindings—in one operation.
6. **`AbstractAction` convenience.** The abstract class provides a default property store, constructors accepting name and icon, and a basic enabled flag implementation. Most application actions extend `AbstractAction`.
7. **`ENABLED` is special.** Unlike other properties, `ENABLED` is not stored in the property map; it is managed via `isEnabled`/`setEnabled` and reported through property-change events.
8. **Serialization-friendly.** `Action` extends `Serializable`; actions can be serialized for session persistence (with the usual Swing serialization caveats).

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, lambda expressions, and method references.
- Familiarity with `JButton`, `JMenuItem`, `JToolBar`, and keyboard bindings.
- Understanding of `ActionListener`, `PropertyChangeListener`, and the `java.beans` package.
- Familiarity with `KeyStroke` and `InputEvent` modifier masks.
- Knowledge of the EDT and `SwingUtilities.invokeLater`.
- A JDK (version 8 or later; `Action` and `AbstractAction` are in `javax.swing`).

---

### Related Programming Areas

- **`ActionListener`:** The parent interface; `Action` is a richer `ActionListener`.
- **`AbstractAction`:** The standard abstract implementation.
- **`JButton(Action)`, `JMenuItem(Action)`, `JCheckBoxMenuItem(Action)`, `JRadioButtonMenuItem(Action)`:** Constructors that configure components from an action.
- **`InputMap` and `ActionMap`:** The key-binding infrastructure that invokes `actionPerformed` when a shortcut is pressed.
- **`PropertyChangeListener`:** Components register listeners to stay synchronized with action property changes.
- **`KeyStroke` and `InputEvent`:** Used with `ACCELERATOR_KEY` and `MNEMONIC_KEY`.
- **`Action.ENABLED`:** The special property that controls whether all components built from an action are interactive.

---

## Core Concepts and Features

---

## 1. Action — The Interface

### Definitions

**Core Definition.** `Action` is an interface for objects that encapsulate a command’s logic and presentation, enabling the same command to be shared across menus, toolbars, and keyboard shortcuts.

**Technical Definition.** `public interface Action extends ActionListener, Serializable`. It adds property-map access (`getValue`, `putValue`, `getKeys`), enabled-state access (`isEnabled`, `setEnabled`), and property-change listener registration to the `ActionListener` contract. The property keys are public `static final String` constants that components recognize.

**Beginner-Friendly Explanation.** `Action` is the “reusable command” contract. Implement it (usually by extending `AbstractAction`) and pass the same instance to multiple UI components to keep them synchronized.

---

### Purposes

- To encapsulate a command’s logic and presentation in a single reusable object.
- To keep menu items, toolbar buttons, and keyboard shortcuts synchronized automatically.
- To manage the enabled state of all components built from an action with one method call.
- To support property-change notifications so components update when the action changes.
- To enable the Command pattern in Swing applications.

---

### Syntax Rules and Structure

#### Complete General Syntax (Implementing an Action via AbstractAction)

```java
Action saveAction = new AbstractAction("Save") {
    @Override
    public void actionPerformed(ActionEvent e) {
        saveDocument();
    }
};
saveAction.putValue(Action.SHORT_DESCRIPTION, "Save the document");
saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);
saveAction.putValue(Action.ACCELERATOR_KEY,
    KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));

JButton button = new JButton(saveAction);
JMenuItem menuItem = new JMenuItem(saveAction);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new AbstractAction("Save") { ... }` | Creates an action with the given name and action logic. |
| `putValue(String, Object)` | Stores a property value in the action. |
| `Action.SHORT_DESCRIPTION` | The tooltip text. |
| `Action.MNEMONIC_KEY` | The keyboard mnemonic. |
| `Action.ACCELERATOR_KEY` | The global keyboard shortcut. |
| `new JButton(action)` | Creates a button configured from the action. |

#### Syntax Rules

- All keys must be the predefined `Action` constants; arbitrary strings are ignored by Swing components.
- `ENABLED` is not managed via `putValue`; use `setEnabled(boolean)`.
- Properties can be changed at any time; components update automatically via property-change listeners.

#### Constraints and Limitations

- `Action` does not manage the command’s enabled state beyond the boolean flag; state-dependent enabling must be driven by application logic.
- Custom property keys are allowed but are not automatically consumed by Swing components.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Shared Action Across Button and Menu Item

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class SharedAction {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared Action");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            Action saveAction = new AbstractAction("Save") {          // 1. Action
                @Override
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Saving...");
                }
            };
            saveAction.putValue(Action.SHORT_DESCRIPTION, "Save the file");
            saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);
            saveAction.putValue(Action.ACCELERATOR_KEY,
                KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));

            JButton button = new JButton(saveAction);                 // 2. Button
            JMenuItem menuItem = new JMenuItem(saveAction);           // 3. Menu item

            JMenuBar bar = new JMenuBar();
            JMenu file = new JMenu("File");
            file.add(menuItem);
            bar.add(file);
            frame.setJMenuBar(bar);

            frame.add(button, BorderLayout.CENTER);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Save” button and a “File → Save” menu item. Both display the same label, tooltip, mnemonic, and accelerator. Clicking either—or pressing Ctrl+S—prints “Saving...” to the console.

**Why the code produces this result.** The `JButton` and `JMenuItem` constructors read the action’s `NAME`, `SHORT_DESCRIPTION`, `MNEMONIC_KEY`, and `ACCELERATOR_KEY` properties. They also register property-change listeners so any later change to the action is reflected in both components.

---

### Real-World Cases

- **Toolbars and menus:** Commands like Save, Open, Print, Cut, Copy, and Paste are typically implemented as shared actions.
- **Keyboard shortcuts:** `InputMap`/`ActionMap` bindings invoke the same action used by menu items.
- **Dynamic enabled state:** An action’s enabled state is toggled when the selection changes, document is modified, or a background task completes.

---

## 2. AbstractAction — The Implementation

### Definitions

**Core Definition.** `AbstractAction` is the abstract class that implements `Action` with a default property store, property-change support, and an enabled flag, allowing subclasses to implement only `actionPerformed`.

**Technical Definition.** `public abstract class AbstractAction extends Object implements Action, Cloneable, Serializable`. It provides constructors accepting name, icon, or neither. It stores property values in a map and fires `PropertyChangeEvent`s when values change. Its `isEnabled`/`setEnabled` manage the `enabled` flag and fire a `PropertyChangeEvent` for `ENABLED`. `putValue` and `getValue` are implemented using the internal store.

**Beginner-Friendly Explanation.** `AbstractAction` is the “base class” for actions. You extend it, implement `actionPerformed`, and optionally set properties. It handles the rest.

---

### Purposes

- To provide a ready-made base for custom actions.
- To reduce boilerplate by implementing the property map and listener registration.
- To support property-change synchronization with UI components.
- To provide a default enabled-state implementation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
public class SaveAction extends AbstractAction {
    public SaveAction() {
        super("Save");
        putValue(SHORT_DESCRIPTION, "Save the document");
        putValue(MNEMONIC_KEY, KeyEvent.VK_S);
    }
    @Override
    public void actionPerformed(ActionEvent e) {
        saveDocument();
    }
}
```

#### Component Breakdown

| Constructor | Meaning |
|-------------|---------|
| `AbstractAction()` | Creates an action with no name or icon. |
| `AbstractAction(String name)` | Creates an action with the given `NAME`. |
| `AbstractAction(String name, Icon icon)` | Creates an action with the given `NAME` and `SMALL_ICON`. |

#### Syntax Rules

- Subclasses must implement `actionPerformed`.
- Properties set in the constructor are available when components are constructed from the action.
- `setEnabled(false)` fires a `PropertyChangeEvent` for `ENABLED`.

#### Constraints and Limitations

- `AbstractAction` does not perform automatic state-dependent enabling; the application must call `setEnabled`.
- Serialization of `AbstractAction` subclasses requires the subclass itself to be serializable.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Named Action Subclass

```java
import javax.swing.*;
import java.awt.event.*;

public class NamedAction extends AbstractAction {
    public NamedAction() {
        super("Print", new ImageIcon("print.png"));                 // 1. Name + icon
        putValue(SHORT_DESCRIPTION, "Print the document");          // 2. Tooltip
        putValue(MNEMONIC_KEY, KeyEvent.VK_P);                       // 3. Mnemonic
        putValue(ACCELERATOR_KEY,
            KeyStroke.getKeyStroke(KeyEvent.VK_P, InputEvent.CTRL_DOWN_MASK));
    }
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Printing...");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Named Action");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton(new NamedAction()));
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Print” button (with icon if the file exists), tooltip “Print the document,” mnemonic Alt+P, and accelerator Ctrl+P. Clicking it prints “Printing...”.

**Why the code produces this result.** The action’s constructor sets all standard properties. `new JButton(action)` reads them and configures the button accordingly.

---

### Real-World Cases

- **Application commands** (New, Open, Save, Close) implemented as reusable action subclasses.
- **Undo/redo actions** with state-dependent enabled flags.
- **Formatting actions** in text editors that check the current selection.

---

## 3. Action Properties

### 3.1 NAME

**Definitions.** **Core:** `NAME` is the display label for the action. **Technical:** `Action.NAME` is a `String` key; `putValue(NAME, "...")` stores the text used as the component’s label. **Beginner:** The “Save” or “Print” text shown on buttons and menu items.

**Purposes.** To provide the visible label for components built from the action; to enable consistent naming across menus, toolbars, and keyboard bindings.

**Syntax Rules.** `putValue(Action.NAME, "Save");` Components use this as their text when constructed from the action.

**Constraints.** Long names may be truncated in toolbars; keep them concise.

**Example.**

```java
Action a = new AbstractAction() {
    public void actionPerformed(ActionEvent e) {}
};
a.putValue(Action.NAME, "Open");
```

**Expected Output.** A button created from `a` displays “Open”.

**Real-World.** Every command’s display name.

---

### 3.2 SHORT_DESCRIPTION

**Definitions.** **Core:** `SHORT_DESCRIPTION` is the tooltip text for the action. **Technical:** `Action.SHORT_DESCRIPTION` is a `String` key; components use it as their tooltip text. **Beginner:** The small popup text shown when hovering over a button or menu item.

**Purposes.** To provide helpful hover text; to explain the command’s effect.

**Syntax Rules.** `putValue(Action.SHORT_DESCRIPTION, "Save the current file");`

**Constraints.** Tooltips are not guaranteed to appear on all platforms/look-and-feels.

**Example.**

```java
a.putValue(Action.SHORT_DESCRIPTION, "Open a file");
```

**Expected Output.** Hovering over the button shows the tooltip.

**Real-World.** Toolbar buttons and menu items with hover help.

---

### 3.3 LONG_DESCRIPTION

**Definitions.** **Core:** `LONG_DESCRIPTION` is a longer description of the action. **Technical:** `Action.LONG_DESCRIPTION` is a `String` key; components may use it in status bars or context help. **Beginner:** A full-sentence explanation, longer than the tooltip.

**Purposes.** To provide detailed help text; to support context-sensitive help systems.

**Syntax Rules.** `putValue(Action.LONG_DESCRIPTION, "Opens a file chooser dialog...");`

**Constraints.** Not automatically displayed by standard components; the application must use it.

**Example.**

```java
a.putValue(Action.LONG_DESCRIPTION, "Opens a file chooser so the user can select a file to open.");
```

**Real-World.** Status-bar help messages, context help.

---

### 3.4 SMALL_ICON

**Definitions.** **Core:** `SMALL_ICON` is the small icon for the action. **Technical:** `Action.SMALL_ICON` is an `Icon` key; components use it as their icon. **Beginner:** The small picture shown on a toolbar button or menu item.

**Purposes.** To provide a visual representation of the command; to save space in toolbars.

**Syntax Rules.** `putValue(Action.SMALL_ICON, new ImageIcon("icon.png"));`

**Constraints.** Icon size is platform-dependent; use standard 16×16 or 24×24 icons.

**Example.**

```java
a.putValue(Action.SMALL_ICON, new ImageIcon("save.png"));
```

**Expected Output.** The button displays the icon.

**Real-World.** Toolbar buttons, menu items with icons.

---

### 3.5 LARGE_ICON_KEY

**Definitions.** **Core:** `LARGE_ICON_KEY` is the large icon for the action. **Technical:** `Action.LARGE_ICON_KEY` is an `Icon` key; introduced in JDK 6 for high-DPI displays and larger toolbar buttons. **Beginner:** The bigger version of the icon for use in larger UI contexts.

**Purposes.** To support high-resolution or large-format icons; to let components choose an icon size appropriate to their context.

**Syntax Rules.** `putValue(Action.LARGE_ICON_KEY, largeIcon);`

**Constraints.** Not all components use `LARGE_ICON_KEY`; support depends on the look and feel.

**Example.**

```java
a.putValue(Action.LARGE_ICON_KEY, new ImageIcon("save-large.png"));
```

**Real-World.** HiDPI displays, large toolbar buttons.

---

### 3.6 ACCELERATOR_KEY

**Definitions.** **Core:** `ACCELERATOR_KEY` is the keyboard shortcut for the action. **Technical:** `Action.ACCELERATOR_KEY` is a `KeyStroke` key; components use it as their accelerator. **Beginner:** The global shortcut (e.g., Ctrl+S) that activates the command.

**Purposes.** To provide keyboard-driven access to commands; to display the shortcut in menu items.

**Syntax Rules.** `putValue(Action.ACCELERATOR_KEY, KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));`

**Constraints.** Accelerators must be unique within the window; conflicts cause undefined behaviour.

**Example.**

```java
a.putValue(Action.ACCELERATOR_KEY,
    KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK));
```

**Expected Output.** The menu item shows “Ctrl+S” and the shortcut works globally.

**Real-World.** File commands (Ctrl+S, Ctrl+O), edit commands (Ctrl+C, Ctrl+V).

---

### 3.7 MNEMONIC_KEY

**Definitions.** **Core:** `MNEMONIC_KEY` is the keyboard mnemonic for the action. **Technical:** `Action.MNEMONIC_KEY` is an `Integer` key holding a `KeyEvent.VK_*` code; components use it as their mnemonic. **Beginner:** The Alt-key shortcut (e.g., Alt+S) that activates the command when the menu is open.

**Purposes.** To provide keyboard access via mnemonics; to underline a character in the component’s label.

**Syntax Rules.** `putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);`

**Constraints.** The mnemonic character should appear in the action’s `NAME` for the underline to be visible.

**Example.**

```java
a.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);
```

**Expected Output.** The “S” in “Save” is underlined.

**Real-World.** Standard menu mnemonics.

---

### 3.8 ACTION_COMMAND_KEY

**Definitions.** **Core:** `ACTION_COMMAND_KEY` is the action command string passed in the `ActionEvent` when the action is invoked. **Technical:** `Action.ACTION_COMMAND_KEY` is a `String` key; components use it as their action command. **Beginner:** The string that identifies the command when it arrives at an action listener.

**Purposes.** To dispatch shared listeners based on command strings; to provide a stable identifier for the command independent of its display text.

**Syntax Rules.** `putValue(Action.ACTION_COMMAND_KEY, "SAVE");`

**Constraints.** If not set, the action command defaults to the component’s text.

**Example.**

```java
a.putValue(Action.ACTION_COMMAND_KEY, "SAVE");
```

**Expected Output.** The `ActionEvent`’s `getActionCommand()` returns “SAVE”.

**Real-World.** Shared listeners that dispatch by command string.

---

### 3.9 ENABLED

**Definitions.** **Core:** `ENABLED` is the property that controls whether the action is currently executable. **Technical:** `Action.ENABLED` is a special `String` key; unlike other keys, it is not managed via `putValue`/`getValue` but via `isEnabled()`/`setEnabled(boolean)`. Changes fire a `PropertyChangeEvent` with `propertyName` “enabled”. **Beginner:** The “on/off” switch that greys out all components built from the action.

**Purposes.** To disable commands that are not currently applicable; to synchronize the enabled state of all components built from the action.

**Syntax Rules.** `action.setEnabled(false);`

**Constraints.** `putValue(Action.ENABLED, false)` has no effect; use `setEnabled`.

**Example.**

```java
saveAction.setEnabled(false);
```

**Expected Output.** All buttons and menu items built from `saveAction` become greyed out and unclickable.

**Real-World.** Disabling “Paste” when the clipboard is empty; disabling “Save” when there are no changes.

---

## 4. Action Methods

### Definitions

**Core Definition.** `Action` methods provide the command’s executable logic, property-map access, enabled-state management, and property-change listener registration.

**Technical Definition.** Key methods: `actionPerformed(ActionEvent)`, `getValue(String)`, `putValue(String, Object)`, `getKeys()`, `isEnabled()`, `setEnabled(boolean)`, `addPropertyChangeListener(PropertyChangeListener)`, `removePropertyChangeListener(PropertyChangeListener)`.

**Beginner-Friendly Explanation.** The methods let you read/write properties, enable/disable the action, and listen for changes.

---

### Purposes

- To execute the command (`actionPerformed`).
- To read and write property values (`getValue`, `putValue`, `getKeys`).
- To enable or disable the action (`isEnabled`, `setEnabled`).
- To observe action property changes (`addPropertyChangeListener`, `removePropertyChangeListener`).

---

### Syntax Rules and Structure

```java
action.actionPerformed(new ActionEvent(source, id, command));
Object name = action.getValue(Action.NAME);
action.putValue(Action.SHORT_DESCRIPTION, "Tooltip");
Object[] keys = action.getKeys();
boolean enabled = action.isEnabled();
action.setEnabled(false);
action.addPropertyChangeListener(evt -> { ... });
action.removePropertyChangeListener(listener);
```

| Method | Meaning |
|--------|---------|
| `actionPerformed(ActionEvent)` | The command’s executable logic. |
| `getValue(String)` | Returns the value for the given key, or `null`. |
| `putValue(String, Object)` | Stores a value for the given key. |
| `getKeys()` | Returns all currently stored property keys. |
| `isEnabled()` | Returns `true` if the action is executable. |
| `setEnabled(boolean)` | Enables or disables the action and fires a `PropertyChangeEvent`. |
| `addPropertyChangeListener` | Registers a listener for property changes. |
| `removePropertyChangeListener` | Unregisters the listener. |

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Property Change Listener

```java
import javax.swing.*;
import java.beans.PropertyChangeEvent;

public class PropertyChangeAction {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Action action = new AbstractAction("Test") {
                public void actionPerformed(java.awt.event.ActionEvent e) {}
            };
            action.addPropertyChangeListener(evt ->
                System.out.println(evt.getPropertyName() + " → " + evt.getNewValue()));

            action.putValue(Action.SHORT_DESCRIPTION, "Hello");       // fires "ShortDescription"
            action.setEnabled(false);                                  // fires "enabled"
        });
    }
}
```

**Expected Output.** Two lines: `ShortDescription → Hello` and `enabled → false`.

**Why the code produces this result.** `putValue` and `setEnabled` both fire `PropertyChangeEvent`s that are delivered to the registered listener.

---

#### Example 2: Enabling All Components with One Call

```java
import javax.swing.*;
import java.awt.*;

public class SharedEnabledState {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Action action = new AbstractAction("Command") {
                public void actionPerformed(java.awt.event.ActionEvent e) {
                    System.out.println("Executed");
                }
            };

            JButton button = new JButton(action);
            JMenuItem item = new JMenuItem(action);
            JCheckBoxMenuItem checkItem = new JCheckBoxMenuItem(action);

            JMenuBar bar = new JMenuBar();
            JMenu menu = new JMenu("Menu");
            menu.add(item);
            menu.add(checkItem);
            bar.add(menu);

            JFrame frame = new JFrame("Shared Enabled State");
            frame.setJMenuBar(bar);
            frame.add(button, BorderLayout.CENTER);

            JButton toggle = new JButton("Toggle Enabled");
            toggle.addActionListener(e -> action.setEnabled(!action.isEnabled()));
            frame.add(toggle, BorderLayout.SOUTH);

            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Toggle Enabled” enables or disables the button and both menu items simultaneously, because all three share the same action’s enabled state.

**Why the code produces this result.** Components built from an action register a `PropertyChangeListener` on the action. When `setEnabled` fires an “enabled” property change, each component updates its own enabled state.

---

### Real-World Cases

- **Context-sensitive toolbars:** Disabling edit commands when no text is selected.
- **Application state:** Disabling “Start” while a task runs and re-enabling it when the task completes.
- **Multi-component synchronization:** Menu items, toolbar buttons, and keyboard shortcuts staying in sync.

---

## 5. Action Best Practices

### Definitions

**Core Definition.** Action best practices are the conventions and patterns that ensure actions are used effectively, maintainably, and consistently across menus, toolbars, and keyboard shortcuts.

**Technical Definition.** Best practices include: using `Action` objects instead of raw `JButton`/`JMenuItem` listeners for shared commands; putting all standard properties (name, tooltip, icon, mnemonic, accelerator) in the action rather than duplicating them across components; using `setEnabled` (not `putValue`) to control the enabled state; registering property-change listeners only when the application needs to react to action changes; and extending `AbstractAction` rather than implementing `Action` directly.

**Beginner-Friendly Explanation.** Best practices for actions are the “rules of thumb” that keep commands DRY (Don’t Repeat Yourself) and synchronized. They include things like “put the label, icon, and shortcut in the action, not the components,” “use `setEnabled`, not `putValue(ENABLED)`, ” and “extend `AbstractAction`.”

---

### Purposes

- To avoid duplicating command logic across menus, toolbars, and shortcuts.
- To keep presentation properties in one place for consistency.
- To synchronize enabled state across all components built from an action.
- To leverage property-change notifications for dynamic updates.
- To use the correct abstraction (`AbstractAction`) for new commands.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
Action saveAction = new AbstractAction("Save") {                       // 1. Abstract
    @Override
    public void actionPerformed(ActionEvent e) {
        saveDocument();                                               // 2. Logic
    }
};
saveAction.putValue(Action.SHORT_DESCRIPTION, "Save");                // 3. Tooltip
saveAction.putValue(Action.SMALL_ICON, saveIcon);                     // 4. Icon
saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);              // 5. Mnemonic
saveAction.putValue(Action.ACCELERATOR_KEY,
    KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK)); // 6. Shortcut

JButton saveButton = new JButton(saveAction);                         // 7. Share
JMenuItem saveItem = new JMenuItem(saveAction);

saveAction.setEnabled(false);                                         // 8. State
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `AbstractAction` | `new AbstractAction("Name") { ... }` |
| Put presentation in the action | `putValue(NAME, ...)`, `putValue(SMALL_ICON, ...)`, etc. |
| Use `setEnabled` for state | `action.setEnabled(false)` — not `putValue(ENABLED, false)` |
| Share one action across components | `new JButton(action)`, `new JMenuItem(action)` |
| Use `ACTION_COMMAND_KEY` for dispatch | `putValue(ACTION_COMMAND_KEY, "SAVE")` |
| Register property listeners only when needed | `action.addPropertyChangeListener(...)` |
| EDT | Construct and manipulate actions on the EDT |

#### Syntax Rules

- **Always extend `AbstractAction`** rather than implementing `Action` directly, unless you have a specific reason to provide a custom implementation.
- **Always put presentation properties in the action**, not on each component; this ensures consistency and allows components to stay synchronized.
- **Always use `setEnabled`** to control the enabled state; `putValue(Action.ENABLED, ...)` has no effect.
- **Always use `ACTION_COMMAND_KEY`** when multiple actions share a listener and need to be distinguished.
- **Never duplicate command logic** across menu items and toolbar buttons; share the `Action` instance.

#### Constraints and Limitations

- `Action` properties are ignored by components that are not constructed from the action; manually setting properties on a component built from an action can cause desynchronization.
- Component-specific customizations (e.g., a unique font) should be applied **after** construction from the action.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Action with Shared Components

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class BestPracticeAction {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeAction::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Action");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action saveAction = new AbstractAction("Save") {              // 1. Abstract
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Save executed");
            }
        };
        saveAction.putValue(Action.SHORT_DESCRIPTION, "Save the file"); // 2. Tooltip
        saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);        // 3. Mnemonic
        saveAction.putValue(Action.ACCELERATOR_KEY,
            KeyStroke.getKeyStroke(KeyEvent.VK_S, InputEvent.CTRL_DOWN_MASK)); // 4. Shortcut

        JButton saveButton = new JButton(saveAction);                    // 5. Share
        JMenuItem saveItem = new JMenuItem(saveAction);

        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        file.add(saveItem);
        bar.add(file);
        frame.setJMenuBar(bar);

        JCheckBoxMenuItem autoSave = new JCheckBoxMenuItem("Enable Auto-Save");
        autoSave.addActionListener(e -> saveAction.setEnabled(autoSave.isSelected())); // 6. State

        JPanel panel = new JPanel();
        panel.add(saveButton);
        panel.add(autoSave);

        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window with a “Save” button and a “File → Save” menu item. Both share the same tooltip, mnemonic, and Ctrl+S accelerator. The “Enable Auto-Save” check box enables or disables the save command.

**Why the code produces this result.** The action encapsulates all presentation and logic. Both components read from the same action, so they stay synchronized. `setEnabled` fires an `ENABLED` property change that both components observe.

---

#### Example 2: Common Mistake — Setting Properties on the Component Instead of the Action

```java
import javax.swing.*;

public class BadPropertyManagement {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Action action = new AbstractAction("Save") {              // 1. Action
                public void actionPerformed(java.awt.event.ActionEvent e) {}
            };

            JButton button = new JButton(action);
            button.setText("Save");                                   // WRONG: duplicate
            button.setToolTipText("Save the file");                   // WRONG: duplicate
            button.setMnemonic('S');                                  // WRONG: duplicate

            // Later, if the action changes, the button will not reflect it.
            action.putValue(Action.NAME, "Save As");                  // 2. Action change
            // The button still says "Save" — desynchronized!
        });
    }
}
```

**Expected Output.** When the action’s `NAME` is changed to “Save As,” the button still displays “Save” because its text was set directly on the component, overriding the action’s value.

**Why this is wrong.** Components constructed from an action automatically synchronize their properties. Manually setting `setText`, `setToolTipText`, or `setMnemonic` on the component **overrides** the action-derived value and breaks the synchronization. The correct approach is to put all properties in the action and let the component derive them.

---

### Real-World Cases

- **Production Swing applications** use `Action` objects for every shared command, with all presentation properties defined on the action and components constructed from the action.
- **Keyboard binding systems** use `ActionMap` and `InputMap` to bind keystrokes to the same actions used by menus and toolbars.
- **Educational examples** use the best-practice skeleton to demonstrate the Command pattern and shared component synchronization.

---

## References

Oracle. *Interface Action*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/en/java/javase/26/docs/api/java.desktop/javax/swing/Action.html

Oracle. *Class AbstractAction*. Java Platform, Standard Edition 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/AbstractAction.html

Oracle. *How to Use Actions*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/action.html

Oracle. *How to Use Key Bindings*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/keybinding.html

Oracle. *Class JButton — JButton(Action)*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JButton.html#JButton-javax.swing.Action-

Oracle. *Class JMenuItem — JMenuItem(Action)*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JMenuItem.html#JMenuItem-javax.swing.Action-

Oracle. *Class KeyStroke*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/KeyStroke.html

Oracle. *Interface PropertyChangeListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/beans/PropertyChangeListener.html

Oracle. *Class PropertyChangeEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/beans/PropertyChangeEvent.html

Stack Overflow. *Action vs. ActionListener best practices*. https://stackoverflow.com/

Stack Overflow. *AbstractAction properties — SHORT_DESCRIPTION vs LONG_DESCRIPTION*. https://stackoverflow.com/

Oracle. *Class KeyEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/KeyEvent.html

Oracle. *Class InputEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/InputEvent.html