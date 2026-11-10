# ActionListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `ActionListener` is a functional interface in the `java.awt.event` package that receives `ActionEvent`s. It is the primary mechanism by which Java programs react to button clicks, menu selections, text-field Enter presses, timer ticks, and any other component that fires semantic action events. Its single method, `actionPerformed(ActionEvent e)`, contains the reaction code.

**Technical Definition.** `public interface ActionListener extends EventListener`. It declares exactly one method: `void actionPerformed(ActionEvent e)`. Because it is a **functional interface** (a SAM — single abstract method — interface), it can be implemented by a lambda expression, a method reference, an anonymous inner class, or a named class. Listeners are registered on action-capable components (`JButton`, `JMenuItem`, `JTextField`, `JComboBox`, `Timer`, `JFileChooser` approval, etc.) via `addActionListener(ActionListener)`. All listener invocations occur on the **Event Dispatch Thread (EDT)**; the listener is invoked **synchronously** by the event source, which means a slow listener blocks subsequent event dispatch. `ActionEvent` itself extends `AWTEvent`, which extends `EventObject`; it carries the source, the action command string (`getActionCommand()`), the modifiers (`getModifiers()`), and the timestamp (`getWhen()`). Importantly, `ActionEvent` fires on **every activation**, even if the component's state has not changed — a distinction that matters greatly for toggle controls.

**Beginner-Friendly Explanation.** An `ActionListener` is the “click handler” for Swing. You write a small object whose `actionPerformed` method contains the code you want to run when the user presses a button, chooses a menu item, or presses Enter in a text field. You attach that object to the component with `addActionListener`, and from then on, every time the component fires an action, your method is called. Because the interface has only one method, you can write it as a compact lambda: `button.addActionListener(e -> doSomething());`. The listener runs on the EDT, so it must not perform long-running work — it should do its job quickly and, for anything heavy, hand the work to a `SwingWorker` or a background thread. Think of it as the “onClick” function of Swing: it is the universal entry point for user-triggered commands.

---

### Key Characteristics

1. **Single-method functional interface.** `ActionListener` declares exactly one abstract method, making it lambda-friendly and method-reference-friendly. No adapter class is needed.
2. **Universal for commands.** It is the listener used by `JButton`, `JMenuItem`, `JCheckBoxMenuItem`, `JRadioButtonMenuItem`, `JTextField` (on Enter), `JComboBox` (on selection), `javax.swing.Timer`, `JFileChooser` approve actions, and `JOptionPane` standard dialogs’ custom buttons.
3. **Fires on every activation.** Unlike `ItemListener`, `ActionListener` fires every time the user activates the component, even if the state (e.g., a check box's selection) has not changed. This is why `ItemListener` is preferred for state-change logic on toggles.
4. **`ActionEvent` carries context.** The event provides `getSource()` (the component), `getActionCommand()` (a string identifier, defaulting to the button's text), `getModifiers()` (modifier keys), and `getWhen()` (timestamp).
5. **Synchronous EDT dispatch.** Listeners are invoked one at a time on the EDT; the event source does not continue until `actionPerformed` returns.
6. **Action command dispatching.** Multiple buttons can share a single listener and use `getActionCommand()` or `getSource()` to branch. `Action` objects provide a cleaner alternative for shared commands.
7. **Consumable.** `ActionEvent` supports `consume()` and `isConsumed()`, allowing a listener to prevent subsequent listeners or default handlers from processing the same event.
8. **Single registration per component is not enforced.** Multiple listeners may be added; they are invoked in registration order (though this order is not guaranteed by specification).
9. **EDT best practices apply.** No blocking operations, no off-EDT component access, and no long-running I/O inside `actionPerformed`.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, lambda expressions, and method references.
- Familiarity with `JButton`, `JTextField`, `JMenuItem`, and at least one Swing container.
- Understanding of the EDT and `SwingUtilities.invokeLater` (every `ActionListener` invocation happens on the EDT).
- Familiarity with `java.awt.event` and `javax.swing.event` package structure.
- Knowledge of `Action` and `AbstractAction` (the recommended way to share command state across menus and toolbars).
- A JDK (version 8 or later; lambdas are essential to modern `ActionListener` style).

---

### Related Programming Areas

- **`ActionEvent`:** The event object delivered to `ActionListener`s.
- **`EventListener`:** The root interface of all listener interfaces; `ActionListener` extends it.
- **`Action` and `AbstractAction`:** Encapsulate text, icon, mnemonic, accelerator, enabled state, and action command; used with `JButton(Action)`, `JMenuItem(Action)`, and keyboard bindings.
- **`ItemListener`:** The preferred listener for toggle-state semantics on `JCheckBox`, `JRadioButton`, and `JComboBox`.
- **`SwingUtilities.invokeLater` and `SwingWorker`:** The mechanisms for keeping long-running work off the EDT inside action handlers.
- **`javax.swing.Timer`:** Fires `ActionEvent`s on the EDT; its `ActionListener` is invoked at regular intervals.
- **InputMap and ActionMap:** Modern key-binding infrastructure that ultimately invokes `Action.actionPerformed(ActionEvent)`, which subsumes `ActionListener`.

---

## Core Concepts and Features

---

## 1. ActionListener — The Interface

### Definitions

**Core Definition.** `ActionListener` is a functional interface that receives `ActionEvent`s and provides a single `actionPerformed(ActionEvent e)` method where the reaction code lives.

**Technical Definition.** `public interface ActionListener extends EventListener`. It declares `void actionPerformed(ActionEvent e)`. Implementations can be anonymous classes, named classes, lambdas, or method references. Components register listeners via `addActionListener(ActionListener)` and remove them via `removeActionListener(ActionListener)`. The invocation is performed on the EDT by the component's UI delegate (via `fireActionPerformed`), synchronously, in the order listeners were registered.

**Beginner-Friendly Explanation.** `ActionListener` is the “clicked!” receiver. You implement its single method to specify what happens when the component is activated, and you attach it to the component.

---

### Purposes

- To provide a single, uniform mechanism for reacting to semantic command events across Swing components.
- To decouple the button/menu item from the logic that responds to its activation.
- To allow multiple components to share the same handler through `getSource()` or `getActionCommand()`.
- To enable lambda-style handling because of its single abstract method.
- To serve as the underlying reaction mechanism for `Action` objects, `Timer`s, and keyboard bindings.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
// Anonymous inner class (classic):
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        // reaction code
    }
});

// Lambda (modern):
button.addActionListener(e -> {
    // reaction code
});

// Method reference:
button.addActionListener(this::handleClick);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addActionListener(ActionListener)` | Registers the listener on the component. |
| `removeActionListener(ActionListener)` | Unregisters the listener. |
| `actionPerformed(ActionEvent e)` | The single method to implement; invoked on activation. |
| `e.getSource()` | The component that fired the event. |
| `e.getActionCommand()` | The string identifier of the action (defaults to button text). |
| `e.getModifiers()` | The modifier keys held during the event. |
| `e.getWhen()` | The timestamp of the event. |

#### Syntax Rules

- Only one method is required for the listener implementation.
- The listener must be registered before the event is delivered (typically during GUI construction).
- All listener code runs on the EDT; it must not block.
- Multiple listeners may be registered on the same component; they are invoked in registration order (order is not guaranteed across implementations).
- If the event is consumed in one listener via `e.consume()`, subsequent listeners should check `e.isConsumed()`.

#### Constraints and Limitations

- `ActionListener` fires on every activation, even if the state has not changed; use `ItemListener` for state-change semantics on toggles.
- Listeners are invoked synchronously; a slow listener blocks all subsequent events on the EDT.
- Listeners must not modify Swing components from other threads without `invokeLater`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Anonymous Inner Class Listener

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class AnonymousActionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Anonymous ActionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            button.addActionListener(new ActionListener() {          // 1. Register
                @Override
                public void actionPerformed(ActionEvent e) {         // 2. Implement
                    System.out.println("Button clicked!");
                }
            });

            frame.add(button, BorderLayout.CENTER);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a “Click Me” button. Each click prints “Button clicked!” to the console.

**Why the code produces this result.** The anonymous `ActionListener` is registered on the button. When the button fires an `ActionEvent`, the EDT invokes `actionPerformed`, which contains the print statement.

---

#### Example 2: Lambda Listener

```java
import javax.swing.*;
import java.awt.*;

public class LambdaActionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Lambda ActionListener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel status = new JLabel("Ready");
            JButton button = new JButton("Say Hello");

            button.addActionListener(e -> {                          // 1. Lambda
                status.setText("Hello, Swing!");
            });

            frame.add(button, BorderLayout.NORTH);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Say Hello” updates the status label to “Hello, Swing!”.

**Why the code produces this result.** The lambda implements the single abstract method of `ActionListener`. The `addActionListener` method accepts the lambda directly because the interface is functional.

---

### Real-World Cases

- **Buttons and menu items:** Every button click and menu command.
- **Text fields on Enter:** Submitting a form or performing a search when Enter is pressed.
- **Combo box selection:** Reacting to a user-chosen item (though `ItemListener` is often better).
- **Timers:** `javax.swing.Timer` fires `ActionEvent`s periodically for animations and polling.
- **File chooser approval:** Buttons in standard dialogs implement action-performed internally.
- **Keyboard bindings:** `ActionMap` entries are `Action` objects whose `actionPerformed` is the same method.

---

## 2. actionPerformed() — The Method

### Definitions

**Core Definition.** `actionPerformed(ActionEvent e)` is the single abstract method of `ActionListener`. It is invoked by the component when the user activates it, and it contains the reaction code.

**Technical Definition.** `void actionPerformed(ActionEvent e)`. The method is invoked on the EDT by the component's UI delegate (typically via an internal `fireActionPerformed` call). It receives the `ActionEvent` object describing the activation. If the method throws an unchecked exception, the exception propagates up the EDT; the event source does not catch it. If the method blocks (e.g., performs I/O or sleeps), the EDT cannot process further events until it returns.

**Beginner-Friendly Explanation.** `actionPerformed` is the “when clicked, do this” block. Everything you write inside it runs on the EDT when the component is activated.

---

### Purposes

- To contain the reaction logic for a component activation.
- To receive the `ActionEvent` context for inspecting the source and modifiers.
- To trigger immediate, short-running UI updates in response to user action.
- To serve as the entry point for shared command handling via `Action` objects.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void actionPerformed(ActionEvent e) {
    // reaction code
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `@Override` | Confirms the method overrides the interface declaration (helps catch typos). |
| `ActionEvent e` | The event object with context (source, command, modifiers, timestamp). |
| Method body | Contains the reaction; runs on the EDT. |

#### Syntax Rules

- The method must be `public` and must accept a single `ActionEvent` parameter.
- The method should perform its work quickly; long tasks must be offloaded.
- Exceptions thrown from the method propagate up the EDT, potentially breaking subsequent dispatch.

#### Constraints and Limitations

- The EDT is blocked until the method returns.
- Uncaught exceptions can disrupt the event pipeline; wrap risky logic in try-catch where appropriate.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Safe Reaction with Exception Handling

```java
import javax.swing.*;
import java.awt.*;

public class SafeActionPerformed {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Safe actionPerformed");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField input = new JTextField(10);
            JButton parse = new JButton("Parse");

            parse.addActionListener(e -> {
                try {
                    int value = Integer.parseInt(input.getText());   // 1. Risky
                    JOptionPane.showMessageDialog(frame,
                        "Parsed: " + value);
                } catch (NumberFormatException ex) {                  // 2. Handle
                    JOptionPane.showMessageDialog(frame,
                        "Not a number: " + input.getText(),
                        "Error", JOptionPane.ERROR_MESSAGE);
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(input);
            frame.add(parse);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Typing a valid number and clicking “Parse” shows the parsed value; typing non-numeric text shows an error dialog. The EDT remains responsive because the listener catches the exception.

**Why the code produces this result.** The `actionPerformed` method wraps `Integer.parseInt` in a try-catch. If an exception escaped, it would propagate up the EDT and could disrupt subsequent event processing.

---

#### Example 2: Shared Listener with `getSource()`

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class SharedActionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Shared Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            ActionListener shared = e -> {                            // 1. Shared
                JButton source = (JButton) e.getSource();
                System.out.println("Clicked: " + source.getText());
            };

            JButton one = new JButton("One");
            JButton two = new JButton("Two");
            one.addActionListener(shared);                            // 2. Same
            two.addActionListener(shared);

            frame.setLayout(new FlowLayout());
            frame.add(one);
            frame.add(two);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “One” prints “Clicked: One”; clicking “Two” prints “Clicked: Two.”

**Why the code produces this result.** Both buttons share the same `ActionListener`. `e.getSource()` identifies which button fired the event, and `getText()` retrieves its label.

---

### Real-World Cases

- **Form submission:** Validating and processing input on Enter or button click.
- **Navigation:** Switching panels or views in response to button presses.
- **Command dispatch:** Branching on `getActionCommand()` to handle multiple commands with one listener.
- **Keyboard bindings:** `ActionMap` delegates to the same `actionPerformed` logic.

---

## 3. ActionEvent — The Event Object

### Definitions

**Core Definition.** `ActionEvent` is the event object delivered to `ActionListener`s. It describes a semantic action (button click, menu selection, Enter press, timer tick) and carries the source, action command, modifier flags, and timestamp.

**Technical Definition.** `public class ActionEvent extends AWTEvent`. Two constructors: `ActionEvent(Object source, int id, String command)` and `ActionEvent(Object source, int id, String command, int modifiers)`. Constants include `ACTION_PERFORMED`, `ALT_MASK`, `CTRL_MASK`, `META_MASK`, `SHIFT_MASK`. The action command defaults to the button's label if not set explicitly via `setActionCommand`. The event supports `consume()` and `isConsumed()`.

**Beginner-Friendly Explanation.** An `ActionEvent` is the “what just happened” note delivered to your listener. It tells you which component fired the event, what command string identifies the action, and what modifier keys were held.

---

### Purposes

- To carry the identity of the event source to the listener.
- To provide a command string for dispatching multiple actions from a shared listener.
- To convey modifier key state for keyboard-enhanced interactions.
- To support event consumption for interception.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// Inspecting the event:
if (e.getSource() == myButton) { /* ... */ }
String command = e.getActionCommand();
int modifiers = e.getModifiers();
long when = e.getWhen();
e.consume();
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getSource()` | The component that fired the event. |
| `getActionCommand()` | The command string (button text by default). |
| `getModifiers()` | Modifier keys held (Alt, Ctrl, Meta, Shift). |
| `getWhen()` | Event timestamp (milliseconds). |
| `consume()` / `isConsumed()` | Consume the event to prevent downstream handling. |

#### Syntax Rules

- `getSource()` returns the component instance; cast to the specific type for component-specific methods.
- The action command can be overridden via `setActionCommand(String)` on buttons and menu items.
- Modifier flags are bitmasks; use `ActionEvent.ALT_MASK` etc. to test.

#### Constraints and Limitations

- `getSource()` returns the actual source instance; relying on it for dispatch is fine for simple cases but is less flexible than `getActionCommand` or `Action`.
- Modifier flags are platform-dependent in meaning; test carefully.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dispatching via Action Command

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class CommandDispatch {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Command Dispatch");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton save = new JButton("Save");
            save.setActionCommand("CMD_SAVE");                        // 1. Override

            JButton load = new JButton("Load");
            load.setActionCommand("CMD_LOAD");                        // 2. Override

            ActionListener dispatcher = e -> {
                String cmd = e.getActionCommand();                    // 3. Dispatch
                switch (cmd) {
                    case "CMD_SAVE": System.out.println("Saving..."); break;
                    case "CMD_LOAD": System.out.println("Loading..."); break;
                    default: System.out.println("Unknown: " + cmd);
                }
            };

            save.addActionListener(dispatcher);
            load.addActionListener(dispatcher);

            frame.setLayout(new FlowLayout());
            frame.add(save);
            frame.add(load);
            frame.setSize(350, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Save” prints “Saving...”; clicking “Load” prints “Loading...”.

**Why the code produces this result.** Each button has a distinct action command. The shared listener dispatches based on `getActionCommand()`.

---

#### Example 2: Consuming an ActionEvent

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class ConsumingActionEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Consuming Event");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(15);

            field.addActionListener(e -> {                            // 1. First
                System.out.println("First listener");
                e.consume();                                           // 2. Consume
            });

            field.addActionListener(e -> {                            // 3. Second
                if (!e.isConsumed()) {
                    System.out.println("Second listener");
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

**Expected Output.** Pressing Enter in the text field prints “First listener” and not “Second listener,” because the first listener consumed the event.

**Why the code produces this result.** `consume()` marks the event as handled; subsequent listeners check `isConsumed()` before reacting. This allows one listener to override another's handling or prevent default behavior.

---

### Real-World Cases

- **Shared command handlers** for multiple buttons using action commands.
- **Keyboard-enhanced buttons** that behave differently with modifier keys held.
- **Audit logging** that uses `getWhen()` for timing information.
- **Intercepting default behavior** (e.g., a text field's Enter action) by consuming the event.

---

## 4. ActionListener Best Practices

### Definitions

**Core Definition.** `ActionListener` best practices are the conventions and patterns that ensure action handling is thread-safe, responsive, maintainable, and appropriate for the component type.

**Technical Definition.** Best practices include: using `ActionListener` for command activation and `ItemListener` for state-change semantics on toggles; never registering both on the same component; offloading long-running work to `SwingWorker` or background threads; using `Action` objects for shared commands across menus, toolbars, and keyboard bindings; keeping handlers short and focused; capturing the actual source with `getSource()` or using `getActionCommand()` for shared handlers; and removing listeners when components are disposed.

**Beginner-Friendly Explanation.** Best practices for `ActionListener` are the “rules of thumb” that keep action handling clean and correct. They include things like “use `ActionListener` for buttons, `ItemListener` for checkboxes,” “don’t do heavy work inside `actionPerformed`,” “use `Action` when the same command appears in multiple places,” and “remove listeners you no longer need.”

---

### Purposes

- To choose the right listener for the component's semantics.
- To avoid blocking the EDT and freezing the GUI.
- To share command state across menus, toolbars, and keyboard shortcuts through `Action` objects.
- To prevent memory leaks by removing listeners when components are disposed.
- To keep handlers short, testable, and maintainable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    Action saveAction = new AbstractAction("Save") {              // 1. Action
        @Override
        public void actionPerformed(ActionEvent e) {
            saveDocument();                                        // 2. Short logic
        }
    };

    JButton saveButton = new JButton(saveAction);                  // 3. Use in button
    JMenuItem saveItem = new JMenuItem(saveAction);                // 4. Use in menu

    JTextField field = new JTextField(15);
    field.addActionListener(e -> {                                // 5. Short lambda
        String text = field.getText();
        new SwingWorker<Void, Void>() {                            // 6. Offload
            @Override
            protected Void doInBackground() {
                processText(text);                                // 7. Background
                return null;
            }
            @Override
            protected void done() {
                statusLabel.setText("Done");                      // 8. EDT update
            }
        }.execute();
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use for buttons/menu items/timers | `button.addActionListener(...)` |
| Use `ItemListener` for toggles | `checkbox.addItemListener(...)` — not `ActionListener` |
| Never both listeners on same toggle | Choose one listener per component. |
| Offload long tasks | `SwingWorker` inside the handler. |
| Share commands with `Action` | `new JButton(action)` and `new JMenuItem(action)` |
| Keep handlers short | No `Thread.sleep`, I/O, or heavy loops. |
| Dispatch with action command | `setActionCommand` + `getActionCommand`. |
| Remove listeners | `removeActionListener` when component disposed. |
| EDT | All listener code runs on the EDT. |

#### Syntax Rules

- **Always use `ActionListener` for command activation**, and `ItemListener` for state changes on toggles.
- **Never register both `ActionListener` and `ItemListener`** on the same toggle component.
- **Always offload long-running work** to `SwingWorker`; never block the EDT.
- **Always use `Action` objects** when the same command appears in menus, toolbars, and buttons.
- **Always remove listeners** from long-lived sources when the listener is no longer needed.

#### Constraints and Limitations

- Action listeners are synchronous; a slow handler freezes the GUI.
- Uncaught exceptions inside `actionPerformed` propagate up the EDT.
- Multiple listeners on the same component all fire; use `consume()` only when you understand the implications.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Action with Shared State

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.KeyEvent;

public class BestPracticeActionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeActionListener::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice ActionListener");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action saveAction = new AbstractAction("Save") {              // 1. Action
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Saving via action.");
            }
        };
        saveAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_S);       // 2. Mnemonic
        saveAction.putValue(Action.ACCELERATOR_KEY,
            KeyStroke.getKeyStroke(KeyEvent.VK_S,
                java.awt.event.InputEvent.CTRL_DOWN_MASK));           // 3. Accelerator

        JButton saveButton = new JButton(saveAction);                  // 4. Button
        JMenuItem saveMenuItem = new JMenuItem(saveAction);            // 5. Menu item

        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        file.add(saveMenuItem);
        bar.add(file);

        JPanel panel = new JPanel(new FlowLayout());
        panel.add(saveButton);

        frame.setJMenuBar(bar);
        frame.add(panel);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window with a “Save” button and a “File → Save” menu item (Ctrl+S). Both trigger the same action and print “Saving via action.” to the console; both share mnemonic and accelerator.

**Why the code produces this result.** The `Action` encapsulates the command logic, mnemonic, and accelerator. When passed to `JButton` and `JMenuItem`, both components display consistent text, tooltip, mnemonic, enabled state, and keyboard shortcut.

---

#### Example 2: Common Mistake — Using ActionListener Instead of ItemListener

```java
import javax.swing.*;

public class WrongListenerChoice {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Wrong Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JCheckBox cb = new JCheckBox("Enable option");
            cb.addActionListener(e ->                                 // WRONG: for toggles
                System.out.println("Selected? " + cb.isSelected()));

            frame.add(cb);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the check box fires the action listener on every activation, including re-clicking an already-checked box, which can trigger unwanted duplicate processing.

**Why this is wrong.** `ActionListener` fires on every activation regardless of state change, while `ItemListener` fires only when the selection state changes. For state-change semantics, `ItemListener` is the appropriate choice. Registering both also causes duplicate handling.

---

#### Example 3: Common Mistake — Blocking the EDT in actionPerformed

```java
import javax.swing.*;

public class BlockingListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Blocking Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Slow");
            button.addActionListener(e -> {
                try {
                    Thread.sleep(3000);                                // WRONG
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
                System.out.println("Done");
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the button freezes the entire UI for three seconds.

**Why this is wrong.** `Thread.sleep` on the EDT blocks event dispatch and painting. The correct approach is to move the work to a `SwingWorker` or background thread and update the UI via `done()` or `invokeLater`.

---

### Real-World Cases

- **Production Swing applications** use `Action` objects for shared commands across menus, toolbars, and keyboard bindings, and always offload long-running work.
- **IDE and editor plugins** follow the same EDT discipline and remove listeners on panel disposal.
- **Educational examples** use the best-practice skeleton to demonstrate correct listener choices and threading from the start.

---

## References

Oracle. *Interface ActionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionListener.html

Oracle. *Class ActionEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionEvent.html

Oracle. *How to Write an Action Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/actionlistener.html

Oracle. *How to Use Actions*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/action.html

Oracle. *How to Use Buttons, Check Boxes, and Radio Buttons*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/button.html

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html

Oracle. *Class AbstractAction*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractAction.html

Oracle. *Class javax.swing.Timer*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/Timer.html

Stack Overflow. *JCheckBox ActionListener vs ItemListener*. https://stackoverflow.com/questions/9882845/jcheckbox-actionlistener-and-itemlistener

Stack Overflow. *When to use ActionListener vs ItemListener*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/