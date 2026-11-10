# Event Handling Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Event handling** is the mechanism by which a Java application responds to user actions (clicks, keystrokes, window resizes, menu selections) and to system-generated occurrences (timers, property changes, document changes, model updates). An **event** is an object that describes something that has happened; an **event listener** is an object that registers interest in a particular kind of event; and the **Event Dispatch Thread (EDT)** is the single thread on which all Swing events are delivered and all Swing components must be accessed. Together, these concepts form the **event-driven programming model** that underlies every Swing application.

**Technical Definition.** Java's modern event model is the **delegation event model** introduced in JDK 1.1, which replaced the JDK 1.0 inheritance-based model. It is built on `java.util.EventObject` (the root of all event objects), `java.util.EventListener` (the root of all listener interfaces), and a source-listener registration pattern: an **event source** (any object that can generate events) maintains a list of registered **listeners** and notifies them by invoking the appropriate method on the listener interface when an event occurs. Event sources typically provide `addXxxListener(XxxListener)` and `removeXxxListener(XxxListener)` methods. For Swing, all event dispatch and all component access occurs on the **Event Dispatch Thread**, a single non-daemon thread started by the AWT toolkit. The EDT dequeues events from the system event queue and dispatches them synchronously to registered listeners.

**Beginner-Friendly Explanation.** Event handling is how your program reacts to what the user does. When the user clicks a button, presses a key, resizes a window, or selects a menu item, Swing creates an **event object** describing what happened, packages it in the **event queue**, and the **Event Dispatch Thread** delivers it to every **listener** that has registered interest in that kind of event. You write **listener classes** (or lambdas) whose methods contain the reaction code, and you **register** those listeners with the components you care about. The program does not poll for clicks; it waits for Swing to call your listener methods. This "don't call us, we'll call you" pattern is the essence of event-driven programming. Think of it as a restaurant: the customer (user) places an order (event), the waiter (EDT) delivers it to the kitchen (your listener), and the kitchen prepares the dish (your reaction code). The waiter handles one order at a time; if the kitchen takes too long, the whole restaurant slows down. That is why you must never block the EDT with long-running work.

---

### Key Characteristics

1. **Delegation model.** Event sources do not handle their own events; they delegate to registered listeners. This separates the source's presentation from the application's reaction logic, enabling reuse and testability.
2. **Interface-based.** Every event type has a corresponding listener interface (e.g., `ActionEvent` → `ActionListener`). Some listeners have one method (functional interfaces suitable for lambdas); others have many (requiring adapters or full implementation).
3. **Source-listener registration.** Components expose `addXxxListener`/`removeXxxListener` methods. Multiple listeners can be registered; they are invoked in the order of registration (though this order is not strictly guaranteed across all implementations).
4. **Event objects carry context.** Every event carries the source (`getSource()`), a timestamp (`getWhen()`), and type-specific data (mouse coordinates, key characters, selected items, changed values).
5. **Single-threaded dispatch.** All Swing events are dispatched on the EDT. Components are not thread-safe, and manipulating them from other threads causes race conditions, painting artifacts, and deadlocks.
6. **Event consumption.** Some events (notably `MouseEvent`, `KeyEvent`, and `ActionEvent`) support consumption via `consume()`. A consumed event is not passed to other listeners or default handlers, enabling custom interception.
7. **Adapters for multi-method interfaces.** For listener interfaces with several methods, Swing provides abstract adapter classes (`MouseAdapter`, `WindowAdapter`, `KeyAdapter`, etc.) that implement all methods with empty bodies, so you override only the ones you need.
8. **Hierarchical and low-level events.** Swing distinguishes low-level events (`ComponentEvent`, `MouseEvent`, `KeyEvent`, `FocusEvent`, `WindowEvent`) from semantic events (`ActionEvent`, `ItemEvent`, `ListSelectionEvent`, `TreeSelectionEvent`, `ChangeEvent`, `DocumentEvent`). High-level events are almost always preferable to low-level ones.
9. **Event-driven, not polling.** Applications do not loop waiting for input; they respond when events arrive. This frees the CPU and keeps the UI responsive.
10. **Best practices are non-negotiable.** The EDT rule, avoidance of blocking operations, correct listener choice, and proper event consumption are essential to stable Swing applications.

---

### Prerequisites

- **Java language fundamentals:** classes, interfaces, inheritance, anonymous inner classes, lambda expressions, and the `@FunctionalInterface` annotation.
- **Basic Swing knowledge:** `JFrame`, `JPanel`, `JButton`, `JTextField`, and at least one layout manager.
- **Understanding of threads:** the difference between the main thread, background threads, and the EDT.
- **The `java.awt.event` and `javax.swing.event` packages:** familiarity with their package structure.
- **Familiarity with the `SwingUtilities.invokeLater` pattern** (used in every Swing GUI example).
- **A JDK (version 8 or later)** — lambdas and the `@FunctionalInterface` annotation are essential to modern event-handling style.

---

### Related Programming Areas

- **`java.util.EventObject` and `java.util.EventListener`:** The root classes of the event model.
- **`java.awt.AWTEvent` and `java.awt.event`:** Low-level AWT events and their listeners.
- **`javax.swing.event`:** Swing-specific semantic events (list selection, tree selection, table model, document, change, etc.).
- **`java.awt.EventQueue` and `EventDispatchThread`:** The infrastructure that dispatches events.
- **`javax.swing.SwingUtilities`:** Utility methods for transferring control to the EDT.
- **`javax.swing.SwingWorker`:** The standard mechanism for running long tasks off the EDT and posting interim updates to it.
- **`javax.swing.Timer`:** A timer whose `ActionEvent`s are dispatched on the EDT, making it safe for UI updates.
- **`javax.swing.Action` and `AbstractAction`:** Encapsulate command logic and state for use across menus, toolbars, and keyboard bindings.
- **`InputMap` and `ActionMap`:** Modern key-binding infrastructure that supersedes the old `registerKeyboardAction` methods.

---

## Core Concepts and Features

---

## 1. Events — The Event Object Model

### Definitions

**Core Definition.** An **event** is an object that describes something that has happened in the program, such as a button click, a mouse movement, a key press, or a property change. The event object carries information about the occurrence—the source, the time, and type-specific details—and is passed to every listener that has registered interest in that type of event.

**Technical Definition.** All event objects ultimately extend `java.util.EventObject`, which provides `getSource()` and a timestamp. AWT and Swing events extend `java.awt.AWTEvent`, which adds `getID()` (an integer identifying the event subtype) and `consume()`/`isConsumed()` for event consumption. Concrete event classes include `ActionEvent`, `MouseEvent`, `KeyEvent`, `WindowEvent`, `ItemEvent`, `ChangeEvent`, `ListSelectionEvent`, `TreeSelectionEvent`, `TreeExpansionEvent`, `TableModelEvent`, `DocumentEvent`, and many others. Each event class is paired with a listener interface that declares methods for receiving it.

**Beginner-Friendly Explanation.** An event object is a "message" that says "this just happened." It knows what happened (its ID), who it happened to (its source), when it happened (its timestamp), and any details relevant to that kind of event (which button was clicked, which key was pressed, where the mouse was). The event object is handed to your listener so your code can inspect it and decide what to do.

---

### Purposes

- To encapsulate information about an occurrence in a single, reusable object.
- To decouple the source of an event from the code that reacts to it.
- To carry type-specific context (coordinates, key codes, selection state) to listeners.
- To support event consumption so that one listener can prevent downstream processing.
- To enable a uniform, interface-based dispatch mechanism across all event types.

---

### Syntax Rules and Structure

#### Complete General Syntax (Event Object Hierarchy)

```java
java.util.EventObject
    └── java.awt.AWTEvent
            ├── java.awt.event.ActionEvent
            ├── java.awt.event.MouseEvent
            ├── java.awt.event.KeyEvent
            ├── java.awt.event.WindowEvent
            ├── java.awt.event.ItemEvent
            ├── java.awt.event.FocusEvent
            ├── java.awt.event.ComponentEvent
            └── javax.swing.event.ChangeEvent
            ├── javax.swing.event.ListSelectionEvent
            ├── javax.swing.event.TreeSelectionEvent
            ├── javax.swing.event.TreeExpansionEvent
            ├── javax.swing.event.TableModelEvent
            └── javax.swing.event.DocumentEvent
```

#### Component Breakdown

| Type | Meaning |
|------|---------|
| `EventObject` | Root class; provides `getSource()` and a timestamp. |
| `AWTEvent` | Base for AWT/Swing events; adds `getID()` and `consume()`. |
| `ActionEvent` | Fired by buttons, menu items, text fields on Enter, timers. |
| `MouseEvent` | Mouse press, release, click, move, drag, enter, exit. |
| `KeyEvent` | Key press, release, and typed characters. |
| `WindowEvent` | Window opened, closing, closed, iconified, etc. |
| `ItemEvent` | Checkboxes, radio buttons, combo boxes, list selection changes. |
| `ChangeEvent` | Spinners, sliders, progress bars, color choosers. |
| `ListSelectionEvent` | List and table selection changes. |
| `DocumentEvent` | Text component content changes. |

#### Syntax Rules

- Every event is an object; listeners receive the object as a method parameter.
- Event IDs are integer constants declared in the concrete event class (e.g., `ActionEvent.ACTION_PERFORMED`, `MouseEvent.MOUSE_CLICKED`, `KeyEvent.KEY_PRESSED`).
- Event objects are immutable with respect to their core data (though consumption state can be toggled).

#### Constraints and Limitations

- Event objects do not describe the *reaction*; they only describe the occurrence.
- Low-level events (`MouseEvent`, `KeyEvent`) fire frequently and can be expensive to handle; prefer high-level events when possible.
- Event IDs are type-specific; a single `AWTEvent` variable may require casting to access type-specific methods.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting an ActionEvent

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class InspectActionEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect ActionEvent");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            button.addActionListener(e -> {                           // 1. Receive event
                System.out.println("Source: " + e.getSource());       // 2. Get source
                System.out.println("Command: " + e.getActionCommand()); // 3. Command
                System.out.println("When: " + e.getWhen());           // 4. Timestamp
                System.out.println("Modifiers: " + e.getModifiers()); // 5. Modifiers
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the button prints the button instance (source), the action command (`"Click Me"`), the event timestamp, and the modifier flags to the console.

**Why the code produces this result.** `ActionEvent` carries the source component, the action command string (defaulting to the button's text), the event time, and the modifier keys. Each getter retrieves one piece of context.

---

#### Example 2: Inspecting a MouseEvent

```java
import javax.swing.*;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

public class InspectMouseEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect MouseEvent");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click or Right-Click");
            button.addMouseListener(new MouseAdapter() {               // 1. Adapter
                @Override
                public void mouseClicked(MouseEvent e) {
                    System.out.println("Button: " + e.getButton());   // 2. Which button
                    System.out.println("Click count: " + e.getClickCount()); // 3. Count
                    System.out.println("Point: " + e.getPoint());     // 4. Coordinates
                    System.out.println("Popup trigger: " + e.isPopupTrigger()); // 5. Trigger
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

**Expected Output.** Left-clicking prints button `1`, click count `1`, and the click point; right-clicking prints button `3` and may set `isPopupTrigger` to `true` on some platforms.

**Why the code produces this result.** `MouseEvent` carries the mouse button identifier (`BUTTON1`, `BUTTON2`, `BUTTON3`), the number of clicks (`getClickCount()`), the coordinates (`getPoint()`), and the platform-specific popup trigger flag (`isPopupTrigger()`).

---

### Real-World Cases

- **Buttons and menu items** fire `ActionEvent`s that carry the action command for dispatching.
- **Text fields** fire `ActionEvent`s on Enter, enabling form submission.
- **Tables and lists** fire `ListSelectionEvent`s when the user changes the selection.
- **Text components** fire `DocumentEvent`s on every character insertion or removal.
- **Spinners, sliders, and color choosers** fire `ChangeEvent`s when the user adjusts the value.
- **Windows** fire `WindowEvent`s on opening, closing, iconification, and activation.

---

## 2. Event Handling — The Delegation Model

### Definitions

**Core Definition.** **Event handling** is the process by which an event source notifies interested listeners when an event occurs, and those listeners react by executing application logic. In Java's delegation model, the source does not contain the reaction logic; instead, it maintains a list of listener objects and invokes the appropriate method on each when an event is fired.

**Technical Definition.** The delegation event model comprises four roles: (1) the **event source**, an object that generates events; (2) the **event object**, which encapsulates the occurrence; (3) the **event listener**, an object implementing a listener interface; and (4) the **event source's registration mechanism** (`addXxxListener` / `removeXxxListener`). When the source detects an event, it iterates over its registered listeners and invokes the interface method corresponding to the event type, passing the event object as an argument. The order of listener invocation is the order of registration, though this is not guaranteed by the specification. If a listener consumes the event, subsequent listeners and default handlers may not be invoked.

**Beginner-Friendly Explanation.** Event handling is the "chain of responsibility" that connects a user action to a program reaction. The button (source) does not know what to do when clicked; it only knows how to tell registered listeners that it was clicked. You write a listener object containing the reaction code, register it with the button, and the button calls your code when the click happens. This separation keeps the button reusable and your logic testable.

---

### Purposes

- To decouple event sources from application-specific reaction logic.
- To allow multiple listeners to react to the same event.
- To enable reuse of event sources across applications with different behaviours.
- To provide a uniform, interface-based mechanism for all event types.
- To support event consumption so that default behaviour can be intercepted or suppressed.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
// 1. Create the source.
JButton button = new JButton("Click");

// 2. Create and register a listener.
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        // reaction code
    }
});

// 3. Or with a lambda:
button.addActionListener(e -> {
    // reaction code
});
```

#### Component Breakdown

| Role | Example | Responsibility |
|------|---------|----------------|
| Event source | `JButton` | Generates events and maintains listeners. |
| Registration | `addActionListener(...)` | Associates a listener with a source. |
| Listener interface | `ActionListener` | Declares methods for receiving events. |
| Event object | `ActionEvent` | Carries context about the occurrence. |
| Reaction method | `actionPerformed(...)` | Contains the application logic. |

#### Syntax Rules

- Listeners must be registered **before** the event can be delivered (typically during GUI construction).
- Multiple listeners can be registered on the same source.
- The source invokes listener methods synchronously; a slow listener blocks subsequent ones (and, for Swing, the EDT).
- A listener may consume the event (`e.consume()`), preventing default processing.

#### Constraints and Limitations

- Listeners are invoked on the EDT; they must not perform long-running operations.
- Registration order is not strictly guaranteed by the specification; do not rely on it for critical logic.
- A listener that throws an exception can prevent subsequent listeners from being notified.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Anonymous Inner Class Listener

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class AnonymousListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Anonymous Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            button.addActionListener(new ActionListener() {           // 1. Register
                @Override
                public void actionPerformed(ActionEvent e) {          // 2. React
                    System.out.println("Button was clicked!");
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

**Expected Output.** Clicking the button prints "Button was clicked!" to the console.

**Why the code produces this result.** The anonymous `ActionListener` is registered on the button. When the button fires an `ActionEvent`, the EDT invokes `actionPerformed`, which contains the reaction code.

---

#### Example 2: Lambda Listener with Event Consumption

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class ConsumingListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Consuming Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JTextField field = new JTextField(15);
            JButton button = new JButton("Submit");

            // Consuming listener: prevents the default action
            ActionListener consumer = e -> {
                System.out.println("Intercepted: " + e.getActionCommand());
                e.consume();                                        // 1. Consume
            };
            field.addActionListener(consumer);

            // Secondary listener: still runs if not consumed
            field.addActionListener(e -> {
                if (!e.isConsumed()) {                              // 2. Check
                    System.out.println("Also handled");
                }
            });

            frame.setLayout(new FlowLayout());
            frame.add(field);
            frame.add(button);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Pressing Enter in the text field prints "Intercepted: " and does not print "Also handled" because the first listener consumed the event.

**Why the code produces this result.** `ActionEvent` supports consumption via `consume()`. Once consumed, `isConsumed()` returns `true`, and subsequent listeners may skip processing. This mechanism is used to intercept default behaviour (e.g., form submission).

---

### Real-World Cases

- **Form validation:** A text field's Enter listener validates and submits the form, consuming the event to prevent default submission.
- **Keyboard shortcuts:** Key bindings (via `InputMap`/`ActionMap`) intercept keystrokes before they reach the focused component.
- **Context menus:** A `MouseListener` intercepts right-clicks to show a `JPopupMenu`, consuming the event so it does not propagate.
- **Shared commands:** An `Action` object is registered as the listener for both a menu item and a toolbar button, ensuring both stay synchronised.

---

## 3. Event Listeners

### Definitions

**Core Definition.** An **event listener** is an object that implements a listener interface—a subinterface of `java.util.EventListener`—and thereby declares its interest in a specific type of event. When the event source fires, it invokes the listener's method(s) with the event object as an argument.

**Technical Definition.** Every AWT/Swing event type has a corresponding listener interface. Examples include `ActionListener` (single method `actionPerformed`), `MouseListener` (five methods: `mouseClicked`, `mousePressed`, `mouseReleased`, `mouseEntered`, `mouseExited`), `WindowListener` (seven methods), `KeyListener` (three methods), `ItemListener` (single method `itemStateChanged`), `ChangeListener` (single method `stateChanged`), `ListSelectionListener` (single method `valueChanged`), `TreeSelectionListener` (single method `valueChanged`), and `DocumentListener` (three methods). Listener interfaces with a single abstract method are **functional interfaces** and may be implemented with lambdas. Listener interfaces with multiple methods require either an adapter class or a full implementation.

**Beginner-Friendly Explanation.** An event listener is a "hearing aid" for a component. It listens for a specific kind of event and does something when it hears it. Some listeners care about a single method (e.g., `ActionListener` for clicks), so you can write a short lambda. Others care about several methods (e.g., `MouseListener` for clicks, presses, releases, entering, and exiting), so you use an adapter class to override only the methods you care about.

---

### Purposes

- To declare, in a type-safe way, interest in a specific kind of event.
- To receive event objects with full context for reacting appropriately.
- To allow multiple independent components to share or have separate listeners.
- To support functional-interface style (lambdas) for single-method listeners.
- To enable composition of multiple listeners on a single source.

---

### Syntax Rules and Structure

#### Complete General Syntax (Single-Method Listener)

```java
component.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        // reaction
    }
});
// or with a lambda:
component.addActionListener(e -> { /* reaction */ });
```

#### Complete General Syntax (Multi-Method Listener)

```java
component.addMouseListener(new MouseListener() {
    public void mouseClicked(MouseEvent e)  { /* ... */ }
    public void mousePressed(MouseEvent e)  { /* ... */ }
    public void mouseReleased(MouseEvent e) { /* ... */ }
    public void mouseEntered(MouseEvent e)  { /* ... */ }
    public void mouseExited(MouseEvent e)   { /* ... */ }
});
```

#### Component Breakdown

| Listener | Methods | Type | Adapter |
|----------|---------|------|---------|
| `ActionListener` | `actionPerformed` | Single | None (use lambda) |
| `ItemListener` | `itemStateChanged` | Single | None (use lambda) |
| `ChangeListener` | `stateChanged` | Single | None (use lambda) |
| `ListSelectionListener` | `valueChanged` | Single | None (use lambda) |
| `MouseListener` | 5 methods | Multi | `MouseAdapter` |
| `KeyListener` | 3 methods | Multi | `KeyAdapter` |
| `WindowListener` | 7 methods | Multi | `WindowAdapter` |
| `FocusListener` | 2 methods | Multi | `FocusAdapter` |
| `ComponentListener` | 4 methods | Multi | `ComponentAdapter` |
| `DocumentListener` | 3 methods | Multi | None (implement all three) |

#### Syntax Rules

- Single-method listeners are functional interfaces; lambdas are idiomatic.
- Multi-method listeners are best handled via adapters to avoid boilerplate.
- Listeners must be registered with the source via `addXxxListener`.
- Listener methods are invoked on the EDT.

#### Constraints and Limitations

- Multi-method listeners require implementing every method (use adapters to avoid this).
- A listener that throws an exception in its method can disrupt the EDT; wrap logic in try-catch where necessary.
- Listeners should be removed when no longer needed to avoid memory leaks (long-lived sources holding references to short-lived listeners).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Single-Method Listener with Lambda

```java
import javax.swing.*;
import java.awt.*;

public class SingleMethodListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Single-Method Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click");
            button.addActionListener(e -> {                           // 1. Lambda
                System.out.println("Action: " + e.getActionCommand());
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the button prints "Action: Click" to the console.

**Why the code produces this result.** `ActionListener` is a functional interface, so the lambda satisfies it. The `addActionListener` method registers the lambda; when the event fires, the lambda's body executes.

---

#### Example 2: Multi-Method Listener with Full Implementation

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;

public class MultiMethodListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Multi-Method Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Hover or Click");
            button.addMouseListener(new MouseListener() {              // 1. Full
                public void mouseClicked(MouseEvent e) {
                    System.out.println("Clicked");
                }
                public void mousePressed(MouseEvent e) {
                    System.out.println("Pressed");
                }
                public void mouseReleased(MouseEvent e) {
                    System.out.println("Released");
                }
                public void mouseEntered(MouseEvent e) {
                    System.out.println("Entered");
                }
                public void mouseExited(MouseEvent e) {
                    System.out.println("Exited");
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

**Expected Output.** Moving the mouse over the button prints "Entered"; pressing prints "Pressed"; releasing prints "Released" and "Clicked"; moving away prints "Exited".

**Why the code produces this result.** All five `MouseListener` methods are implemented; the mouse fires the corresponding event for each interaction.

---

### Real-World Cases

- **`ActionListener`** on buttons, menu items, and text fields (Enter) for command dispatch.
- **`ItemListener`** on check boxes, radio buttons, and combo boxes for state changes.
- **`DocumentListener`** on text components for real-time validation and search-as-you-type.
- **`ListSelectionListener`** on lists and tables for selection-driven detail views.
- **`WindowListener`** on frames for save-on-close, iconification handling, and cleanup.
- **`TreeSelectionListener`** on trees for master-detail views and lazy loading.

---

## 4. Event Adapters

### Definitions

**Core Definition.** An **event adapter** is an abstract class that implements a multi-method listener interface with empty method bodies. It allows you to override only the methods you care about, avoiding the boilerplate of implementing every method when you need just one or two.

**Technical Definition.** Adapters exist only for listener interfaces with more than one method. The standard adapters are `MouseAdapter` (implements `MouseListener`, `MouseWheelListener`, and `MouseMotionListener`), `KeyAdapter` (implements `KeyListener`), `WindowAdapter` (implements `WindowListener`, `WindowStateListener`, and `WindowFocusListener`), `FocusAdapter` (implements `FocusListener`), `ComponentAdapter` (implements `ComponentListener`), `ContainerAdapter` (implements `ContainerListener`), `HierarchyBoundsAdapter` (implements `HierarchyBoundsListener`), and `MouseMotionAdapter` (implements `MouseMotionListener`). Each adapter provides empty implementations of every method, so subclasses can override only the methods of interest.

**Beginner-Friendly Explanation.** An adapter is a "pre-filled form" for a multi-method listener. Instead of writing five empty mouse methods just to handle one click, you extend `MouseAdapter` and override only `mouseClicked`. The adapter provides the empty methods so you do not have to.

---

### Purposes

- To reduce boilerplate when implementing multi-method listener interfaces.
- To allow selective overriding of only the methods of interest.
- To enable clean, readable anonymous inner class implementations.
- To support composition of multiple listener interfaces in a single adapter (e.g., `MouseAdapter` handles mouse, wheel, and motion events).

---

### Syntax Rules and Structure

#### Complete General Syntax (Adapter Usage)

```java
component.addMouseListener(new MouseAdapter() {
    @Override
    public void mouseClicked(MouseEvent e) {
        // reaction
    }
});
```

#### Component Breakdown

| Adapter | Implements | Methods |
|---------|-----------|---------|
| `MouseAdapter` | `MouseListener`, `MouseWheelListener`, `MouseMotionListener` | 9 methods |
| `KeyAdapter` | `KeyListener` | 3 methods |
| `WindowAdapter` | `WindowListener`, `WindowStateListener`, `WindowFocusListener` | 10 methods |
| `FocusAdapter` | `FocusListener` | 2 methods |
| `ComponentAdapter` | `ComponentListener` | 4 methods |
| `ContainerAdapter` | `ContainerListener` | 2 methods |
| `MouseMotionAdapter` | `MouseMotionListener` | 2 methods |

#### Syntax Rules

- Extend the adapter class and override only the methods you need.
- Register the adapter with the appropriate `addXxxListener` method.
- Adapters are abstract classes, so you cannot use lambdas; anonymous inner classes are the standard idiom.

#### Constraints and Limitations

- Adapters implement multiple listener interfaces; you must register the adapter with each relevant `addXxxListener` method for the corresponding events.
- Extending an adapter means you cannot extend another class (single inheritance).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MouseAdapter for Click Handling

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

public class MouseAdapterDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("MouseAdapter Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click Me");
            button.addMouseListener(new MouseAdapter() {               // 1. Extend
                @Override
                public void mouseClicked(MouseEvent e) {              // 2. Override
                    System.out.println("Clicked at " + e.getPoint());
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

**Expected Output.** Clicking the button prints "Clicked at java.awt.Point[x=..., y=...]" with the click coordinates.

**Why the code produces this result.** `MouseAdapter` implements all five `MouseListener` methods with empty bodies. Only `mouseClicked` is overridden, so only that method is invoked when a click occurs.

---

#### Example 2: WindowAdapter for Close Confirmation

```java
import javax.swing.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class WindowAdapterDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("WindowAdapter Demo");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);

            frame.addWindowListener(new WindowAdapter() {               // 1. Extend
                @Override
                public void windowClosing(WindowEvent e) {              // 2. Override
                    int choice = JOptionPane.showConfirmDialog(
                        frame, "Are you sure you want to exit?");
                    if (choice == JOptionPane.YES_OPTION) {
                        frame.dispose();
                    }
                }
            });

            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the close button shows a confirmation dialog. Choosing "Yes" disposes the frame; choosing "No" or "Cancel" keeps it open.

**Why the code produces this result.** `WindowAdapter` implements all seven `WindowListener` methods. Only `windowClosing` is overridden, and `DO_NOTHING_ON_CLOSE` prevents the default close operation, allowing the adapter to handle the close request manually.

---

### Real-World Cases

- **`MouseAdapter`** for context menus, drag-and-drop, and click handling.
- **`WindowAdapter`** for save-on-close, iconification handling, and cleanup.
- **`KeyAdapter`** for keyboard shortcuts (though `InputMap`/`ActionMap` is preferred for text components).
- **`FocusAdapter`** for select-all-on-focus, validation-on-blur, and tooltip updates.
- **`ComponentAdapter`** for resize handling and custom painting invalidation.

---

## 5. The Event Dispatch Thread (EDT)

### Definitions

**Core Definition.** The **Event Dispatch Thread (EDT)** is a single, special thread that handles all events dispatched to Swing components and performs all painting. Because Swing components are not thread-safe, all access to them must occur on the EDT; otherwise, race conditions, painting artifacts, and deadlocks can occur.

**Technical Definition.** The EDT is started by the AWT toolkit when the first Swing component is displayed. It dequeues events from the system event queue (`java.awt.EventQueue`) and dispatches them synchronously to registered listeners. `SwingUtilities.invokeLater(Runnable)` schedules a task on the EDT and returns immediately; `SwingUtilities.invokeAndWait(Runnable)` schedules and blocks until completion (and must not be called from the EDT itself). `SwingUtilities.isEventDispatchThread()` returns whether the current thread is the EDT. The EDT is a non-daemon thread; it keeps the JVM alive as long as there are displayable windows. Long-running tasks on the EDT block event processing and freeze the UI; `SwingWorker` (or a background thread plus `invokeLater`) must be used for such tasks.

**Beginner-Friendly Explanation.** The EDT is the "single waiter" of the Swing restaurant. Every click, keystroke, and repaint is handled by this one thread, one at a time. If the waiter takes too long with one order, all other customers wait. That is why Swing applications must never block the EDT with long operations. When you need to do heavy work, you hand it to a background cook (`SwingWorker` or a background thread) and ask the waiter to deliver the result when it is ready (`SwingUtilities.invokeLater`).

---

### Purposes

- To serialize all Swing component access, preventing race conditions and data corruption.
- To dispatch user-generated events (clicks, keys, window events) to registered listeners.
- To perform repainting and layout updates in a consistent, single-threaded manner.
- To provide a predictable execution context for event-handling code.
- To keep the GUI responsive by processing events one at a time in a dedicated thread.

---

### Syntax Rules and Structure

#### Complete General Syntax (EDT Interaction)

```java
// Schedule a task on the EDT (non-blocking):
SwingUtilities.invokeLater(() -> {
    // GUI code runs on the EDT
});

// Schedule and wait (must not be called from the EDT):
SwingUtilities.invokeAndWait(() -> {
    // GUI code runs on the EDT
});

// Check if current thread is the EDT:
boolean isEdt = SwingUtilities.isEventDispatchThread();

// Background task with progress updates:
new SwingWorker<Void, Integer>() {
    protected Void doInBackground() {
        // long work here (not on EDT)
        return null;
    }
    protected void process(java.util.List<Integer> chunks) {
        // runs on the EDT
    }
    protected void done() {
        // runs on the EDT
    }
}.execute();
```

#### Component Breakdown

| Method / Class | Meaning |
|----------------|---------|
| `SwingUtilities.invokeLater(Runnable)` | Schedules a task on the EDT and returns immediately. |
| `SwingUtilities.invokeAndWait(Runnable)` | Schedules a task on the EDT and blocks until it completes. |
| `SwingUtilities.isEventDispatchThread()` | Returns `true` if the calling thread is the EDT. |
| `SwingWorker<T, V>` | Runs a long task in the background and posts results to the EDT. |
| `javax.swing.Timer` | Fires `ActionEvent`s on the EDT at regular intervals. |

#### Syntax Rules

- **All Swing component construction and manipulation must occur on the EDT.**
- **All event listeners are invoked on the EDT.**
- **Long-running tasks must be offloaded** to `SwingWorker` or a background thread.
- **Background threads must post UI updates via `SwingUtilities.invokeLater`.**
- **`invokeAndWait` must not be called from the EDT** (deadlock).

#### Constraints and Limitations

- The EDT is a single thread; heavy work in a listener freezes the GUI.
- Swing components are not thread-safe; even reading a component's state from a non-EDT thread is unsafe.
- `SwingWorker`'s `doInBackground` runs off the EDT; `process` and `done` run on the EDT.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Verifying the EDT

```java
import javax.swing.*;

public class EdtCheck {
    public static void main(String[] args) {
        System.out.println("main thread is EDT? "
            + SwingUtilities.isEventDispatchThread());             // 1. false

        SwingUtilities.invokeLater(() -> {
            System.out.println("Lambda thread is EDT? "
                + SwingUtilities.isEventDispatchThread());          // 2. true
        });
    }
}
```

**Expected Output.**
```
main thread is EDT? false
Lambda thread is EDT? true
```

**Why the code produces this result.** The `main` thread is not the EDT. `invokeLater` schedules the lambda on the EDT, where `isEventDispatchThread()` returns `true`.

---

#### Example 2: Safe Background Update with SwingWorker

```java
import javax.swing.*;
import java.awt.*;
import java.util.List;

public class SwingWorkerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SwingWorker Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JLabel status = new JLabel("Starting...");
            JProgressBar bar = new JProgressBar(0, 100);
            bar.setStringPainted(true);

            new SwingWorker<Void, Integer>() {                          // 1. Worker
                @Override
                protected Void doInBackground() throws Exception {
                    for (int i = 1; i <= 100; i++) {
                        Thread.sleep(50);                                // 2. Work
                        publish(i);                                       // 3. Post
                    }
                    return null;
                }
                @Override
                protected void process(List<Integer> chunks) {
                    int latest = chunks.get(chunks.size() - 1);
                    bar.setValue(latest);                                 // 4. EDT
                    status.setText("Progress: " + latest + "%");
                }
                @Override
                protected void done() {
                    status.setText("Complete!");                         // 5. EDT
                }
            }.execute();

            frame.setLayout(new BorderLayout());
            frame.add(bar, BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A progress bar fills from 0% to 100% over about five seconds, with the status label updating to the current percentage and finally showing "Complete!"—without freezing the UI.

**Why the code produces this result.** `SwingWorker.doInBackground` runs on a background thread, so the EDT remains responsive. `publish` queues progress chunks; `process` and `done` run on the EDT, where updating the bar and label is safe.

---

#### Example 3: Common Mistake — Blocking the EDT

```java
import javax.swing.*;

public class BlockingEdt {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Blocking EDT");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Freeze for 5s");
            button.addActionListener(e -> {
                try {
                    Thread.sleep(5000);                                // WRONG
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
                System.out.println("Done sleeping");
            });

            frame.add(button);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the button freezes the entire GUI for five seconds—no repaints, no other clicks, no window movement—because the EDT is blocked inside the listener.

**Why this is wrong.** `Thread.sleep` on the EDT prevents it from processing events. The correct approach is `SwingWorker` or a background thread, with `invokeLater` (or `SwingWorker.done`) updating the UI when finished.

---

### Real-World Cases

- **Every Swing application** uses `SwingUtilities.invokeLater` in `main` to construct the GUI.
- **File I/O, network access, and database queries** are offloaded to `SwingWorker` to keep the UI responsive.
- **Progress bars and status labels** are updated via `SwingWorker.publish`/`process`.
- **Timers and animations** use `javax.swing.Timer`, which fires on the EDT.
- **Long-running computations** are split into chunks, with intermediate results posted to the EDT.

---

## 6. Event Handling Best Practices

### Definitions

**Core Definition.** Event handling best practices are the conventions and patterns that ensure event-driven Swing applications are thread-safe, responsive, maintainable, and accessible.

**Technical Definition.** Best practices include: constructing and manipulating all Swing components on the EDT; offloading long-running work to `SwingWorker` or background threads; never blocking the EDT with I/O, sleeps, or expensive computations; preferring high-level semantic events (`ActionEvent`, `ItemEvent`, `ListSelectionEvent`) over low-level ones (`MouseEvent`, `KeyEvent`); using adapters for multi-method listeners; using `Action` objects to share command state across menus, toolbars, and keyboard bindings; checking `getValueIsAdjusting()` in `ChangeListener` and `ListSelectionListener`; removing listeners when no longer needed; and never registering both `ActionListener` and `ItemListener` on the same toggle component.

**Beginner-Friendly Explanation.** Best practices for event handling are the "rules of the road" that keep Swing apps stable and pleasant to use. They include things like "build the GUI on the EDT," "don't sleep in a click handler," "use `ActionListener` for buttons and `ItemListener` for check boxes," and "remove listeners when you're done with them."

---

### Purposes

- To maintain thread safety by respecting the EDT.
- To keep the GUI responsive by offloading long-running work.
- To avoid duplicate or conflicting event handling by choosing the right listener for the component.
- To reduce boilerplate by using adapters and lambdas.
- To prevent memory leaks by removing listeners from long-lived sources.
- To support accessibility and internationalisation through high-level event choices.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {                                // 1. EDT
    JFrame frame = new JFrame("Best Practice");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

    JButton button = new JButton("Do Work");
    button.setMnemonic(KeyEvent.VK_D);                             // 2. Mnemonic
    button.setToolTipText("Starts a long-running task");           // 3. Tooltip
    button.addActionListener(e -> startWork(frame));                // 4. Listener

    JCheckBox toggle = new JCheckBox("Enable option");
    toggle.addItemListener(e -> {                                  // 5. ItemListener
        boolean on = (e.getStateChange() == ItemEvent.SELECTED);
        // react to state
    });

    frame.add(button, BorderLayout.NORTH);
    frame.add(toggle, BorderLayout.SOUTH);
    frame.setSize(400, 200);
    frame.setLocationRelativeTo(null);
    frame.setVisible(true);
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Construct on the EDT | `SwingUtilities.invokeLater(...)` |
| Offload long tasks | `SwingWorker` or background thread |
| Never block the EDT | No `Thread.sleep`, I/O, or heavy loops in listeners |
| High-level events | `ActionListener`, `ItemListener`, `ListSelectionListener` |
| Avoid `KeyListener` for text | Use `DocumentListener` or `DocumentFilter` |
| Use adapters | `MouseAdapter`, `WindowAdapter`, etc. |
| Share command state | `Action` objects for menus + toolbars + buttons |
| Check adjusting flag | `getValueIsAdjusting()` in change/list listeners |
| Remove listeners | `removeXxxListener` when no longer needed |
| Avoid both listeners | Never register `ActionListener` + `ItemListener` on the same toggle |

#### Syntax Rules

- **Always construct and manipulate Swing components on the EDT.**
- **Always offload long-running work** to `SwingWorker`; never block the EDT.
- **Always prefer high-level events** (`ActionEvent`, `ItemEvent`, `ListSelectionEvent`) over low-level ones.
- **Always use `ItemListener` (not `ActionListener`) for state-change semantics** on check boxes, radio buttons, and combo boxes.
- **Always check `getValueIsAdjusting()`** in `ChangeListener` and `ListSelectionListener` to avoid reacting to intermediate states.
- **Always remove listeners** from long-lived sources when the listening component is disposed.

#### Constraints and Limitations

- Listeners are invoked on the EDT, so heavy work in a listener freezes the GUI.
- `ActionListener` fires on every activation, even if the state has not changed; `ItemListener` fires only on state changes.
- Low-level listeners (`MouseListener`, `KeyListener`) can miss events or interfere with text components; prefer high-level alternatives.
- `registerKeyboardAction` is obsolete; use `InputMap` and `ActionMap`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Button with Action and SwingWorker

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.KeyEvent;

public class BestPracticeEventHandling {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeEventHandling::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Event Handling");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        Action workAction = new AbstractAction("Do Work") {           // 1. Action
            @Override
            public void actionPerformed(ActionEvent e) {
                startWork();
            }
        };
        workAction.putValue(Action.MNEMONIC_KEY, KeyEvent.VK_D);
        workAction.putValue(Action.SHORT_DESCRIPTION, "Starts a long-running task");

        JButton button = new JButton(workAction);                     // 2. From Action
        JProgressBar bar = new JProgressBar(0, 100);
        bar.setStringPainted(true);
        JLabel status = new JLabel("Ready");

        frame.add(button, BorderLayout.NORTH);
        frame.add(bar, BorderLayout.CENTER);
        frame.add(status, BorderLayout.SOUTH);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }

    private static void startWork() {
        new SwingWorker<Void, Integer>() {                            // 3. Background
            @Override
            protected Void doInBackground() throws Exception {
                for (int i = 1; i <= 100; i++) {
                    Thread.sleep(30);
                    publish(i);
                }
                return null;
            }
            @Override
            protected void process(java.util.List<Integer> chunks) {
                int latest = chunks.get(chunks.size() - 1);
                // update progress bar (from EDT)
            }
            @Override
            protected void done() {
                // final UI update (from EDT)
            }
        }.execute();
    }
}
```

**Expected Output.** A button with mnemonic and tooltip starts a background task; the UI remains responsive while the task runs, and progress updates appear on the EDT.

**Why the code produces this result.** The `Action` encapsulates the command, mnemonic, and tooltip; `SwingWorker` performs the long-running work off the EDT and posts progress updates safely via `publish`/`process`/`done`.

---

#### Example 2: Common Mistake — Registering Both ActionListener and ItemListener

```java
import javax.swing.*;
import java.awt.event.ItemEvent;

public class BadListenerRegistration {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Listeners");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JCheckBox cb = new JCheckBox("Option");
            cb.addItemListener(e ->                                  // WRONG: both
                System.out.println("Item: " + e.getStateChange()));
            cb.addActionListener(e ->                                // WRONG: both
                System.out.println("Action: " + cb.isSelected()));

            frame.add(cb);
            frame.setSize(300, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the check box prints both "Item: ..." and "Action: ..." messages, causing duplicate or inconsistent handling.

**Why this is wrong.** A check box fires both `ActionEvent` (on every activation) and `ItemEvent` (only on state change). Registering both listeners leads to duplicate or conflicting logic. The correct approach is to choose one: `ItemListener` for state-dependent logic, or `ActionListener` for user-activation semantics.

---

#### Example 3: Removing a Listener

```java
import javax.swing.*;
import java.awt.event.ActionListener;

public class RemoveListenerDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Remove Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JButton button = new JButton("Click");
            ActionListener listener = e ->
                System.out.println("Clicked");

            button.addActionListener(listener);                       // 1. Add

            JButton remove = new JButton("Remove Listener");
            remove.addActionListener(e -> {
                button.removeActionListener(listener);                // 2. Remove
                System.out.println("Listener removed");
            });

            frame.setLayout(new java.awt.FlowLayout());
            frame.add(button);
            frame.add(remove);
            frame.setSize(400, 150);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Initially, clicking "Click" prints "Clicked". After clicking "Remove Listener", clicking "Click" no longer prints anything.

**Why the code produces this result.** `removeActionListener` unregisters the listener, so the button no longer notifies it. This prevents memory leaks when the listening component is disposed or no longer cares about the event.

---

### Real-World Cases

- **Production Swing applications** use `SwingWorker` for long-running tasks and update UI via `process`/`done`.
- **IDE and editor plugins** follow the same EDT discipline and remove listeners when panels are disposed.
- **Educational examples** use the best-practice skeleton to demonstrate correct EDT usage, listener selection, and threading from the start.

---

## References

Oracle. *The Event Dispatch Thread*. Java Platform, Standard Edition 8 API Specification (Swing threading policy). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Lesson: Writing Event Listeners*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/index.html

Oracle. *Delegation Event Model*. Java SE Documentation. https://docs.oracle.com/javase/8/docs/api/java/awt/event/package-summary.html

Oracle. *Interface ActionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/ActionListener.html

Oracle. *Class MouseAdapter*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/MouseAdapter.html

Oracle. *Class WindowAdapter*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/WindowAdapter.html

Oracle. *Class SwingUtilities*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html

Oracle. *Class EventQueue*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/EventQueue.html

Oracle. *Class Timer (javax.swing.Timer)*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/Timer.html

Oracle. *Class AbstractAction*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractAction.html

Oracle. *How to Use Actions*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/action.html

Oracle. *How to Use Key Bindings*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/misc/keybinding.html