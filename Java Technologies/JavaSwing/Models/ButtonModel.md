# ButtonModel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `ButtonModel` is the **state model** behind every Swing button — including `JButton`, `JCheckBox`, `JRadioButton`, and `JToggleButton`. It stores the button's state (what its keyboard mnemonic is, whether it's enabled, selected, pressed, or rolled over) and fires the events that drive the button's visual feedback and action dispatch. The interface is the "M" in Swing's separable model architecture for buttons.

**Technical Definition.** `public interface ButtonModel extends ItemSelectable`. It declares methods for querying and mutating the button's state (`isArmed`, `isPressed`, `isEnabled`, `isSelected`, `isRollover`, `getMnemonic`, `getActionCommand`, `getGroup`), for registering listeners (`addActionListener`, `addChangeListener`, `addItemListener`, and their removal counterparts), and for mutating state (`setArmed`, `setPressed`, `setEnabled`, `setSelected`, `setRollover`, `setMnemonic`, `setActionCommand`, `setGroup`). The standard implementation is `DefaultButtonModel`. In practice, **a button's UI delegate takes the responsibility of calling methods on its model** to manage state — application code rarely interacts with the model directly.

**Beginner-Friendly Explanation.** `ButtonModel` is the **"brain" of a button**. The visible button (the `JButton` object) is the **face** — it paints the label, the border, and the pressed/unpressed appearance. The `ButtonModel` is the **brain** that remembers: "Am I pressed? Am I armed? Am I enabled? Am I selected? Is the mouse hovering over me?" When you click a button, the look-and-feel's UI delegate updates the model, and the model fires events that tell the button how to repaint and tell your listeners that the button was clicked. Most of the time you never touch the model — you just call `button.addActionListener(...)` and the button handles the model internally. But when you need fine-grained control — sharing state between two buttons, tracking press/release/rollover separately, or implementing a custom toggle — the `ButtonModel` is the object you interact with.

---

### Key Characteristics

1. **State machine for buttons.** The model tracks five boolean states: **enabled**, **pressed**, **armed**, **selected**, and **rollover**, plus the keyboard **mnemonic**, the **action command**, and the **ButtonGroup** (for radio buttons).
2. **Armed vs. pressed.** **Pressed** means the mouse button (or keyboard key) is currently down. **Armed** means the button is "primed" — the mouse is both pressed **and** within the button's bounds. A button is only **triggered** (fires an `ActionEvent`) when the mouse is released while the model is **armed**.
3. **Selected vs. pressed.** For regular buttons, **selected** is not used. For `JCheckBox`, `JRadioButton`, and `JToggleButton`, **selected** records the toggle state (checked/unchecked).
4. **Rollover.** **Rollover** is `true` when the mouse is positioned over the button, driving hover effects.
5. **Shared models.** Two buttons can share the same `ButtonModel` instance, so that their state (pressed, selected, enabled) stays synchronized.
6. **Three listener types.** The model fires **`ActionEvent`** (button triggered), **`ChangeEvent`** (any state change), and **`ItemEvent`** (selected/deselected) to registered listeners.
7. **UI-delegate driven.** The look-and-feel's `ButtonUI` calls the model's mutators in response to mouse and keyboard input; application code typically calls only `setEnabled` and reads state.
8. **Default implementation.** `DefaultButtonModel` is the concrete model used by all standard Swing buttons; it implements the full state machine and listener bookkeeping.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, and lambda expressions.
- Familiarity with `JButton`, `JCheckBox`, `JRadioButton`, and `ButtonGroup`.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `ActionListener`, `ChangeListener`, and `ItemListener`.
- Knowledge of `ButtonUI` and look-and-feel architecture (helpful but not required).
- A JDK (version 8 or later; `ButtonModel` and `DefaultButtonModel` are in `javax.swing`).

---

### Related Programming Areas

- **`AbstractButton` and `JButton`:** The components that use `ButtonModel`; `AbstractButton` provides `getModel`/`setModel`.
- **`DefaultButtonModel`:** The standard implementation.
- **`ButtonGroup`:** Manages mutual exclusivity for radio buttons; the model stores a reference to its group.
- **`ButtonUI`:** The look-and-feel delegate that mutates the model in response to input.
- **`Action`:** An alternative, higher-level encapsulation of button state and logic.
- **`ItemSelectable`:** The parent interface that declares `addItemListener`/`removeItemListener`.
- **`ChangeListener` and `ActionListener`:** The event interfaces the model fires.

---

## Core Concepts and Features

---

## 1. ButtonModel — The Interface

### Definitions

**Core Definition.** `ButtonModel` is the interface that defines the state and event contract for Swing buttons.

**Technical Definition.** `public interface ButtonModel extends ItemSelectable`. It declares getters and setters for `armed`, `enabled`, `pressed`, `rollover`, `selected`, `mnemonic`, `actionCommand`, and `group`; methods for adding and removing `ActionListener`, `ChangeListener`, and `ItemListener`; and no-argument query methods for each state boolean.

**Beginner-Friendly Explanation.** `ButtonModel` is the contract that describes everything a button needs to remember about itself.

---

### Purposes

- To encapsulate the button's state independently of its visual representation.
- To allow multiple buttons to share state via a shared model.
- To fire `ActionEvent`, `ChangeEvent`, and `ItemEvent` notifications when state changes.
- To support custom button implementations and look-and-feels.
- To provide a testable, headless representation of button state.

---

### Syntax Rules and Structure

#### Complete General Syntax (Accessing a Button's Model)

```java
ButtonModel model = button.getModel();
if (model.isPressed()) { /* ... */ }
model.setEnabled(false);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `button.getModel()` | Returns the button's `ButtonModel`. |
| `model.isPressed()` | Returns `true` if the button is currently pressed. |
| `model.setEnabled(false)` | Disables the button (all views sharing the model update). |

#### Syntax Rules

- The model is created automatically when the button is constructed; `DefaultButtonModel` is the default.
- The model can be replaced with `button.setModel(otherModel)`.
- The model's state is read by the UI delegate during painting.
- All model access should occur on the EDT.

#### Constraints and Limitations

- Directly mutating the model (e.g., `model.setPressed(true)`) does not automatically fire the button's `ActionListener`; only the UI delegate's proper press/release sequence triggers the action.
- Sharing a model across buttons means they share enabled, pressed, and selected state — use with care.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting Button State

```java
import javax.swing.*;
import java.awt.*;

public class InspectButtonModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect ButtonModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            ButtonModel model = button.getModel();                    // 1. Get model

            button.addChangeListener(e -> {                            // 2. State change
                System.out.println("Pressed: " + model.isPressed()
                    + ", Armed: " + model.isArmed()
                    + ", Rollover: " + model.isRollover());
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Moving the mouse over the button prints “Rollover: true”; pressing prints “Pressed: true, Armed: true”; releasing prints “Pressed: false, Armed: false”.

**Why the code produces this result.** The `ChangeListener` fires on every state change. The model’s booleans reflect the current interaction state: rollover on hover, pressed and armed on mouse-down.

---

### Real-World Cases

- **Custom button implementations** that need to react to press/release/rollover separately.
- **Shared state controls** where two buttons mirror each other’s enabled state.
- **Debugging and testing** button state without relying on visual inspection.

---

## 2. Button Model States and Properties

### 2.1 Enabled

**Definitions.** **Core:** Whether the button can be interacted with. **Technical:** `isEnabled()` / `setEnabled(boolean)`. **Beginner:** The button is “greyed out” when disabled.

**Purposes.** To disable commands that are not currently applicable.

**Syntax Rules.** `model.setEnabled(false);` disables all buttons sharing the model.

**Example.**

```java
button.getModel().setEnabled(false);
```

**Real-World.** Disabling “Save” when there are no changes.

---

### 2.2 Pressed

**Definitions.** **Core:** Whether the mouse button (or keyboard key) is currently down on the button. **Technical:** `isPressed()` / `setPressed(boolean)`. **Beginner:** The button looks “pushed in.”

**Purposes.** To track the physical press state for visual feedback.

**Syntax Rules.** The UI delegate sets this during mouse-down and clears it on mouse-up.

**Example.**

```java
if (button.getModel().isPressed()) System.out.println("Being pressed");
```

**Real-World.** Custom pressed-state rendering.

---

### 2.3 Armed

**Definitions.** **Core:** Whether the button is “primed” — pressed **and** the mouse is within the button’s bounds. **Technical:** `isArmed()` / `setArmed(boolean)`. **Beginner:** The button is ready to fire when you release the mouse.

**Purposes.** To determine whether a mouse release will trigger the action.

**Syntax Rules.** The model becomes armed on mouse-press inside the button; it becomes unarmed if the mouse leaves the button while pressed.

**Example.**

```java
button.getModel().addChangeListener(e ->
    System.out.println("Armed: " + button.getModel().isArmed()));
```

**Real-World.** Buttons that only fire when released inside their bounds.

---

### 2.4 Selected

**Definitions.** **Core:** The toggle state for check boxes, radio buttons, and toggle buttons. **Technical:** `isSelected()` / `setSelected(boolean)`. **Beginner:** Whether a check box is checked or a radio button is chosen.

**Purposes.** To record the selection state of toggle buttons.

**Syntax Rules.** `model.setSelected(true);` fires an `ItemEvent` (SELECTED) and a `ChangeEvent`.

**Example.**

```java
JCheckBox cb = new JCheckBox("Option");
cb.getModel().addItemListener(e ->
    System.out.println("Selected: " + cb.isSelected()));
```

**Real-World.** Check boxes, radio buttons, toggle buttons.

---

### 2.5 Rollover

**Definitions.** **Core:** Whether the mouse is positioned over the button. **Technical:** `isRollover()` / `setRollover(boolean)`. **Beginner:** The button is being “hovered.”

**Purposes.** To drive hover effects and tooltip display.

**Syntax Rules.** The UI delegate sets rollover on mouse-enter and clears it on mouse-exit.

**Example.**

```java
button.getModel().addChangeListener(e -> {
    if (button.getModel().isRollover()) button.setBorder(BorderFactory.createLineBorder(Color.BLUE));
});
```

**Real-World.** Hover-highlighted buttons.

---

### 2.6 Mnemonic

**Definitions.** **Core:** The keyboard shortcut character (Alt-key) for the button. **Technical:** `getMnemonic()` / `setMnemonic(int)`. **Beginner:** The underlined letter in the button’s label.

**Purposes.** To provide keyboard access to the button.

**Syntax Rules.** `model.setMnemonic(KeyEvent.VK_S);` — the character must appear in the button’s text.

**Example.**

```java
button.getModel().setMnemonic(KeyEvent.VK_S);
```

**Real-World.** Accessible keyboard navigation.

---

### 2.7 ActionCommand

**Definitions.** **Core:** The command string sent in the `ActionEvent` when the button is triggered. **Technical:** `getActionCommand()` / `setActionCommand(String)`. **Beginner:** The text that identifies the button in an action event.

**Purposes.** To dispatch shared listeners based on command strings.

**Syntax Rules.** `model.setActionCommand("SAVE");` — if not set, defaults to the button’s text.

**Example.**

```java
button.getModel().setActionCommand("SAVE");
```

**Real-World.** Shared action listeners that branch on `getActionCommand()`.

---

### 2.8 Group

**Definitions.** **Core:** The `ButtonGroup` the button belongs to (for radio buttons). **Technical:** `getGroup()` / `setGroup(ButtonGroup)`. **Beginner:** The radio-button group that enforces mutual exclusivity.

**Purposes.** To enforce that only one radio button in a group is selected.

**Syntax Rules.** The `ButtonGroup` is set when the button is added to it via `group.add(button)`.

**Example.**

```java
ButtonGroup group = new ButtonGroup();
group.add(radio1);
group.add(radio2);
```

**Real-World.** Radio button groups.

---

### Real-World Cases (All States)

- **Custom button renderers** that change appearance based on `pressed`, `armed`, `rollover`, and `enabled`.
- **Shared models** that synchronize enabled state across duplicate buttons.
- **Testing** button state transitions programmatically.

---

## 3. ButtonModel Methods — Complete Reference

### Definitions

**Core Definition.** The `ButtonModel` methods provide access to the button's state and the ability to mutate it and register listeners.

**Technical Definition.** The methods fall into three categories: **state getters**, **state setters**, and **listener registration/removal**. The table below summarizes them.

| Method | Return Type | Purpose |
|--------|-------------|---------|
| `isArmed()` | `boolean` | Whether the button is armed. |
| `isEnabled()` | `boolean` | Whether the button is enabled. |
| `isPressed()` | `boolean` | Whether the button is pressed. |
| `isRollover()` | `boolean` | Whether the mouse is over the button. |
| `isSelected()` | `boolean` | Whether the button is selected. |
| `getMnemonic()` | `int` | The keyboard mnemonic. |
| `getActionCommand()` | `String` | The action command string. |
| `getGroup()` | `ButtonGroup` | The button’s group. |
| `setArmed(boolean)` | `void` | Sets the armed state. |
| `setEnabled(boolean)` | `void` | Sets the enabled state. |
| `setPressed(boolean)` | `void` | Sets the pressed state. |
| `setRollover(boolean)` | `void` | Sets the rollover state. |
| `setSelected(boolean)` | `void` | Sets the selected state. |
| `setMnemonic(int)` | `void` | Sets the mnemonic. |
| `setActionCommand(String)` | `void` | Sets the action command. |
| `setGroup(ButtonGroup)` | `void` | Sets the button’s group. |
| `addActionListener(ActionListener)` | `void` | Adds an action listener. |
| `removeActionListener(ActionListener)` | `void` | Removes an action listener. |
| `addChangeListener(ChangeListener)` | `void` | Adds a change listener. |
| `removeChangeListener(ChangeListener)` | `void` | Removes a change listener. |
| `addItemListener(ItemListener)` | `void` | Adds an item listener. |
| `removeItemListener(ItemListener)` | `void` | Removes an item listener. |

#### Component Breakdown

| State / Method | Meaning |
|----------------|---------|
| `isArmed` / `setArmed` | The button is pressed and the pointer is inside. |
| `isPressed` / `setPressed` | The mouse button (or keyboard key) is down. |
| `isSelected` / `setSelected` | The toggle state (check box, radio button). |
| `isRollover` / `setRollover` | The mouse is hovering over the button. |
| `getMnemonic` / `setMnemonic` | The Alt-key shortcut. |
| `getActionCommand` / `setActionCommand` | The command string in the `ActionEvent`. |
| `addChangeListener` | Observes every state change. |

#### Syntax Rules

- `setEnabled` and `setSelected` are the setters most commonly used by application code.
- `setPressed` and `setArmed` are typically managed by the UI delegate, not application code.
- `addChangeListener` is the general-purpose observer; `addActionListener` fires only when the button is triggered; `addItemListener` fires only on selection changes.

#### Constraints and Limitations

- Manually calling `setPressed` or `setArmed` does not trigger an `ActionEvent`; only the full press-release-armed sequence does.
- Setting `selected` on a regular `JButton` has no visible effect; it is meaningful for toggle buttons.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Reading All State Booleans

```java
import javax.swing.*;
import java.awt.*;

public class ReadAllStates {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Read All States");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JCheckBox cb = new JCheckBox("Option");
            ButtonModel model = cb.getModel();

            cb.addChangeListener(e -> {
                System.out.println("enabled=" + model.isEnabled()
                    + ", pressed=" + model.isPressed()
                    + ", armed=" + model.isArmed()
                    + ", selected=" + model.isSelected()
                    + ", rollover=" + model.isRollover());
            });

            frame.add(cb);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Each state change prints the current values of all five booleans.

**Why the code produces this result.** The `ChangeListener` fires on every state mutation, and the model’s getters expose the current state.

---

### Real-World Cases

- **Debugging** button state transitions.
- **Custom UI delegates** that read state during painting.
- **Test harnesses** that verify state changes without a GUI.

---

## 4. DefaultButtonModel — The Implementation

### Definitions

**Core Definition.** `DefaultButtonModel` is the concrete implementation of `ButtonModel` used by all standard Swing buttons.

**Technical Definition.** `public class DefaultButtonModel extends Object implements ButtonModel, Serializable`. It implements the full state machine, stores listener lists, and fires `ActionEvent`, `ChangeEvent`, and `ItemEvent` appropriately. It provides protected `fireActionPerformed`, `fireStateChanged`, and `fireItemStateChanged` methods for subclasses.

**Beginner-Friendly Explanation.** `DefaultButtonModel` is the “ready-made brain” that every standard button uses. You rarely need to create one yourself.

---

### Purposes

- To provide the standard button state machine out of the box.
- To reduce boilerplate for custom button implementations.
- To serve as the base class for specialized button models.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
ButtonModel model = new DefaultButtonModel();
JButton button = new JButton("Click");
button.setModel(model);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new DefaultButtonModel()` | Creates a fresh model. |
| `button.setModel(model)` | Installs the model on the button. |

#### Syntax Rules

- `DefaultButtonModel` is the default for all `AbstractButton` subclasses.
- Custom models can extend `DefaultButtonModel` and override `fireXxx` methods.

#### Constraints and Limitations

- Setting a custom model on a check box or radio button may break the look-and-feel’s expectations unless the model fully implements the required state machine.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Sharing a Model Between Two Buttons

```java
import javax.swing.*;
import java.awt.*;

public class SharedButtonModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared ButtonModel");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            DefaultButtonModel shared = new DefaultButtonModel();      // 1. Shared
            JButton a = new JButton("A");
            JButton b = new JButton("B");
            a.setModel(shared);                                        // 2. Share
            b.setModel(shared);

            JButton toggle = new JButton("Toggle Enabled");
            toggle.addActionListener(e ->
                shared.setEnabled(!shared.isEnabled()));               // 3. Mutate

            JPanel panel = new JPanel();
            panel.add(a);
            panel.add(b);
            panel.add(toggle);

            frame.add(panel);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Toggle Enabled” enables or disables both buttons A and B simultaneously, because they share the same model.

**Why the code produces this result.** Both buttons use the same `DefaultButtonModel`. When the model’s enabled state changes, both buttons repaint with the new state.

---

### Real-World Cases

- **Synchronized buttons** that should always share the same enabled or selected state.
- **Custom button models** that extend `DefaultButtonModel` to add domain-specific state.

---

## 5. ButtonModel Events

### Definitions

**Core Definition.** `ButtonModel` fires three types of events: **`ActionEvent`** when the button is triggered, **`ChangeEvent`** when any state changes, and **`ItemEvent`** when the selection state changes.

**Technical Definition.** `ActionEvent` fires when the button is triggered (mouse released while armed, or keyboard equivalent). `ChangeEvent` fires on every state mutation. `ItemEvent` fires when `selected` changes, with `SELECTED` or `DESELECTED` state change. The model’s listener registration methods are `addActionListener`, `addChangeListener`, and `addItemListener`.

**Beginner-Friendly Explanation.** The model tells you three kinds of things: “the button was clicked” (`ActionEvent`), “something about the button changed” (`ChangeEvent`), and “the toggle state changed” (`ItemEvent`).

---

### Purposes

- `ActionEvent`: to execute the command when the button is triggered.
- `ChangeEvent`: to observe any state change (pressed, armed, rollover, enabled, selected).
- `ItemEvent`: to react specifically to selection changes on toggle buttons.

---

### Syntax Rules and Structure

```java
model.addActionListener(e -> System.out.println("Triggered"));
model.addChangeListener(e -> System.out.println("State changed"));
model.addItemListener(e -> {
    if (e.getStateChange() == ItemEvent.SELECTED) System.out.println("Selected");
});
```

#### Component Breakdown

| Event | Fired When |
|-------|-----------|
| `ActionEvent` | The button is triggered (press + release while armed). |
| `ChangeEvent` | Any state boolean changes. |
| `ItemEvent` | The `selected` state changes. |

#### Constraints and Limitations

- `ActionEvent` does **not** fire if the mouse is released outside the button (the model is unarmed).
- `ChangeEvent` fires very frequently; keep handlers lightweight.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Observing All Three Event Types

```java
import javax.swing.*;
import java.awt.event.*;

public class AllButtonEvents {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("All Button Events");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JCheckBox cb = new JCheckBox("Toggle");
            ButtonModel model = cb.getModel();

            model.addActionListener(e -> System.out.println("ACTION"));
            model.addChangeListener(e -> System.out.println("CHANGE"));
            model.addItemListener(e -> System.out.println("ITEM"));

            frame.add(cb);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the check box prints a sequence of `CHANGE` and `ITEM` events (and `ACTION` when triggered), in the order the model fires them.

**Why the code produces this result.** The model fires `ChangeEvent` for every state mutation, `ItemEvent` for selection changes, and `ActionEvent` when the button is triggered.

---

### Real-World Cases

- **Custom button implementations** that need fine-grained state observation.
- **Testing** that verifies the exact sequence of state transitions.

---

## 6. ButtonModel Best Practices

### Definitions

**Core Definition.** Button model best practices are the conventions and patterns that ensure button state handling is correct, thread-safe, and maintainable.

**Technical Definition.** Best practices include: working through the `JButton` API (`doClick`, `setEnabled`, `addActionListener`) rather than manipulating the model directly; using `Action` objects for shared command state; sharing a `ButtonModel` only when synchronized state is genuinely desired; checking `isArmed`/`isPressed`/`isRollover` for custom rendering; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for button models are the “rules of thumb” that prevent broken buttons, unexpected side effects, and unmaintainable code. They include things like “use the button, not the model, for normal tasks,” “share models only when you mean to,” and “don't call `setPressed` directly.”

---

### Purposes

- To avoid bypassing the button's built-in action-dispatch logic.
- To keep state synchronized across shared components intentionally.
- To use the highest-level API appropriate for the task.
- To keep custom button implementations correct and maintainable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
// Preferred: work through the button API.
JButton button = new JButton("Save");
button.setEnabled(false);                                          // 1. Button API
button.addActionListener(e -> save());                             // 2. Listener

// Only when you need state details:
ButtonModel model = button.getModel();
if (model.isArmed() && model.isPressed()) { /* ... */ }             // 3. Read state
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Prefer button API | `button.setEnabled(...)`, `button.doClick()`, `button.addActionListener(...)` |
| Use `Action` for shared state | `new JButton(action)` |
| Share model only intentionally | `buttonB.setModel(buttonA.getModel())` |
| Read state for custom rendering | `model.isArmed()`, `model.isPressed()`, `model.isRollover()` |
| Respect the EDT | All model access on the EDT |

#### Syntax Rules

- **Prefer the `JButton`/`AbstractButton` API** (`setEnabled`, `doClick`, `addActionListener`) over direct model manipulation for standard tasks.
- **Use `Action` objects** when the same command appears in menus, toolbars, and buttons; the action’s enabled state automatically synchronizes all components built from it.
- **Share a `ButtonModel` only when you intend to synchronize state** across buttons; otherwise, use separate models.
- **Read model state** (`isArmed`, `isPressed`, `isRollover`) when implementing custom button rendering or interaction logic.
- **Never call `setPressed` or `setArmed` directly** in application code; these are the UI delegate's responsibility.

#### Constraints and Limitations

- Direct model mutation can desynchronize the button's visual state from its logical state.
- Shared models cause all sharing buttons to change state together — including enabled state.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Button State via `Action`

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class BestPracticeButtonModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeButtonModel::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice ButtonModel");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action saveAction = new AbstractAction("Save") {               // 1. Action
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Saved");
            }
        };

        JButton saveButton = new JButton(saveAction);                  // 2. Button
        JMenuItem saveItem = new JMenuItem(saveAction);

        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        file.add(saveItem);
        bar.add(file);
        frame.setJMenuBar(bar);

        JButton toggle = new JButton("Toggle Enabled");
        toggle.addActionListener(e ->
            saveAction.setEnabled(!saveAction.isEnabled()));           // 3. State

        JPanel panel = new JPanel();
        panel.add(saveButton);
        panel.add(toggle);

        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Clicking “Toggle Enabled” enables or disables both the Save button and the File → Save menu item simultaneously, because both derive their enabled state from the same `Action`.

**Why the code produces this result.** The `Action` encapsulates the enabled state. Both components observe it and update automatically. This is the preferred way to share state — not by sharing a `ButtonModel` directly.

---

#### Example 2: Common Mistake — Directly Setting `Pressed` or `Armed`

```java
import javax.swing.*;

public class BadModelMutation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Model Mutation");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click");
            button.addActionListener(e -> System.out.println("Action fired"));

            button.getModel().setPressed(true);                        // WRONG
            button.getModel().setArmed(true);                          // WRONG

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The button may appear pressed, but the `ActionListener` does **not** fire — because the proper press–release–armed sequence never occurred. The button’s visual state is desynchronized from its logical action state.

**Why this is wrong.** `setPressed` and `setArmed` are the UI delegate’s responsibility. Manually setting them bypasses the action-dispatch logic, which requires a release while armed. Use `button.doClick()` to simulate a full click programmatically.

---

#### Example 3: Common Mistake — Sharing a Model Unintentionally

```java
import javax.swing.*;

public class BadSharedModel {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Shared Model");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton a = new JButton("A");
            JButton b = new JButton("B");
            b.setModel(a.getModel());                                  // WRONG: unwanted

            a.setEnabled(false);                                       // Disables B too!
            frame.add(a);
            frame.add(b);

            frame.setLayout(new java.awt.FlowLayout());
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Disabling button A also disables button B, because they share the same model. The buttons are unintentionally coupled.

**Why this is wrong.** Sharing a `ButtonModel` synchronizes **all** state, including enabled, pressed, selected, and rollover. This should be done only when the buttons are meant to behave as a single logical control. Use separate models (the default) or an `Action` for intentional shared command state.

---

### Real-World Cases

- **Production Swing applications** use `Action` objects for shared command state and rarely touch `ButtonModel` directly.
- **Custom button renderers** read `isArmed`, `isPressed`, and `isRollover` to paint the correct visual state.
- **Test harnesses** verify state transitions through the model without a GUI.
- **Educational examples** use the best-practice skeleton to demonstrate correct button state handling from the start.

---

## References

Oracle. *Interface ButtonModel*. Java® Platform, Standard Edition & Java Development Kit Version 21 API Specification. https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/ButtonModel.html

Oracle. *Interface ButtonModel*. Java SE 10 API Specification. https://cr.openjdk.org/~iris/se/18.3/edr/java-se-18.3-edr-spec/api/javax/swing/ButtonModel.html

Oracle. *Class DefaultButtonModel*. Java SE 11 & JDK 11 API Specification. https://cr.openjdk.org/

Oracle. *How to Use Models*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/model.html

Oracle. *Interface ItemSelectable*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/ItemSelectable.html

Oracle. *Class AbstractButton*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractButton.html

Oracle. *Class JButton*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JButton.html

Oracle. *How to Use Buttons, Check Boxes, and Radio Buttons*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/button.html

Oracle. *How to Use Actions*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/action.html

Stack Overflow. *How do I use a JButton's Model?*. https://stackoverflow.com/questions/61021671/how-do-i-use-a-jbuttons-model

Stack Overflow. *Setting button model makes check box non-operational*. https://stackoverflow.com/

Stack Overflow. *ButtonModel best practices — getModel().isPressed()*. https://stackoverflow.com/