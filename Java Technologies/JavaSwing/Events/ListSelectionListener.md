# ListSelectionListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `ListSelectionListener` is an interface in the `javax.swing.event` package for receiving **list selection events**—notifications that the selection in a `JList` or `JTable` is **changing** or **has just changed**. It declares a single method, `valueChanged(ListSelectionEvent e)`, which is the callback invoked whenever the selection state of a list or table is modified, whether by user interaction or programmatic control.

**Technical Definition.** `public interface ListSelectionListener extends EventListener`. The interface declares exactly one method: `void valueChanged(ListSelectionEvent e)`. Because it is a **functional interface**, it can be implemented by a lambda expression, a method reference, an anonymous inner class, or a named class. Listeners are registered on an object that implements the `ListSelectionModel` interface—either directly on the selection model (obtained via `getSelectionModel()` on `JList` or `JTable`) or, for convenience, directly on the `JList` itself using `addListSelectionListener`. `ListSelectionEvent` extends `EventObject` and provides `getFirstIndex()`, `getLastIndex()`, and `getValueIsAdjusting()`. The selection model fires events in **chains** during drag operations; `getValueIsAdjusting()` returns `true` for intermediate events and `false` only for the final event in the chain. All list selection events are dispatched on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** `ListSelectionListener` is the “someone changed the selection” listener. When the user clicks an item in a `JList` or a row in a `JTable`, or when your code programmatically selects an item, the list's selection model fires a `ListSelectionEvent` to every registered `ListSelectionListener`. The event tells you that the selection changed and—through `getValueIsAdjusting()`—whether the user is still in the middle of a selection gesture (e.g., dragging the mouse across multiple items) or has finished. This distinction is crucial: without checking `getValueIsAdjusting()`, a single click can trigger your handler twice—once when the mouse button goes down and once when it is released. Think of `ListSelectionListener` as a “selection change feed” for your list: it reports every adjustment, and you decide whether to act on all of them or only on the final, settled selection.

---

### Key Characteristics

1. **Single-method functional interface.** `valueChanged` is the only method; lambdas and method references are idiomatic.
2. **Fires during and after selection changes.** Events are dispatched while the user is dragging or shift-clicking (`valueIsAdjusting == true`) and once more when the selection stabilizes (`valueIsAdjusting == false`).
3. **Selection-model based.** The event source is the `ListSelectionModel`, not the `JList` itself. You can register on the model directly (`list.getSelectionModel().addListSelectionListener(...)`) or on the `JList` (which delegates).
4. **Lead and Anchor selections.** The selection model tracks two special indices: the **anchor** (where the selection started) and the **lead** (the focused item). Moving the lead with Ctrl held fires events even if the actual selection set does not change.
5. **Three selection modes.** `SINGLE_SELECTION`, `SINGLE_INTERVAL_SELECTION`, and `MULTIPLE_INTERVAL_SELECTION` control what selections are permitted and how many events are fired.
6. **EDT dispatch.** All list selection events are delivered on the EDT; handlers must be fast and must not block.
7. **Programmatic changes fire events.** Calling `setSelectedIndex`, `setSelectedIndices`, or manipulating the selection model programmatically fires `ListSelectionEvent`s just like user interaction does.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, lambda expressions, and method references.
- Familiarity with `JList`, `JTable`, and the `ListSelectionModel` interface.
- Understanding of the EDT and `SwingUtilities.invokeLater`.
- Familiarity with `ListSelectionModel` selection modes (`SINGLE_SELECTION`, `SINGLE_INTERVAL_SELECTION`, `MULTIPLE_INTERVAL_SELECTION`).
- Knowledge of `JScrollPane` (lists and tables are typically wrapped in one).
- A JDK (version 8 or later; `ListSelectionListener` and `ListSelectionEvent` are in `javax.swing.event`).

---

### Related Programming Areas

- **`ListSelectionModel`:** The interface that declares the selection state and fires the events. Implementations include `DefaultListSelectionModel`.
- **`ListSelectionEvent`:** The event object carrying the first index, last index, and adjusting flag.
- **`JList` and `JTable`:** The primary components whose selection models fire these events.
- **`TableModelListener`:** For data changes in tables (complementary to selection changes).
- **`getValueIsAdjusting()`:** The method that distinguishes intermediate from final events.
- **`SwingUtilities.invokeLater` and `SwingWorker`:** The mechanisms for keeping long-running work off the EDT inside `valueChanged`.
- **`ListSelectionListener` on `JList` vs. on the model:** Registering on the list gives `getSource() == list`; registering on the model gives `getSource() == model`.

---

## Core Concepts and Features

---

## 1. ListSelectionListener — The Interface

### Definitions

**Core Definition.** `ListSelectionListener` is a functional interface for receiving notifications when the selection in a `JList` or `JTable` changes.

**Technical Definition.** `public interface ListSelectionListener extends EventListener`. It declares `void valueChanged(ListSelectionEvent e)`. The interface is implemented by classes interested in selection changes. Listeners are registered with `addListSelectionListener` on either a `ListSelectionModel` or a `JList`. The method is invoked on the EDT.

**Beginner-Friendly Explanation.** `ListSelectionListener` is the “selection changed” contract. Implement it (or write a lambda) and register it with a list or its selection model to hear about selection changes.

---

### Purposes

- To react when the user selects or deselects items in a `JList` or rows in a `JTable`.
- To update a detail view, status label, or dependent UI based on the current selection.
- To enable or disable buttons that act on the selected items.
- To synchronize selection between multiple lists or tables.
- To perform validation or commit changes when a selection is finalized.

---

### Syntax Rules and Structure

#### Complete General Syntax (Registration and Handling)

```java
// Registering on the JList (convenience method):
list.addListSelectionListener(new ListSelectionListener() {
    @Override
    public void valueChanged(ListSelectionEvent e) {
        if (!e.getValueIsAdjusting()) {
            // react to final selection
        }
    }
});

// Registering on the selection model (preferred for shared models):
list.getSelectionModel().addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        // react to final selection
    }
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `addListSelectionListener(ListSelectionListener)` | Registers the listener (on `JList` or on `ListSelectionModel`). |
| `removeListSelectionListener(...)` | Unregisters the listener. |
| `valueChanged(ListSelectionEvent e)` | The single method to implement; invoked on selection change. |
| `e.getValueIsAdjusting()` | `true` for intermediate events in a chain; `false` for the final event. |
| `e.getFirstIndex()` | The lower bound of the changed range. |
| `e.getLastIndex()` | The upper bound of the changed range. |

#### Syntax Rules

- Only one method is required for the listener implementation.
- The listener must be registered before the selection change occurs.
- All listener code runs on the EDT; it must not block.
- **Always check `getValueIsAdjusting()`** if the handler should run only once per selection gesture.
- Registering on the `JList` directly sets `event.getSource()` to the list; registering on the model sets it to the model.

#### Constraints and Limitations

- A single user click can fire **two** events (one adjusting, one final). Without filtering, handlers run twice.
- Programmatic selection changes also fire events; this can cause re-entrancy if not handled carefully.
- The selection model fires events for lead/anchor movements even when the selected set has not changed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Selection Listener on a JList

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import javax.swing.event.ListSelectionListener;
import java.awt.*;

public class BasicSelectionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Basic Selection Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"Apple", "Banana", "Cherry", "Date"};
            JList<String> list = new JList<>(items);
            list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

            JLabel status = new JLabel("Select an item");

            list.addListSelectionListener(new ListSelectionListener() { // 1. Register
                @Override
                public void valueChanged(ListSelectionEvent e) {         // 2. React
                    if (!e.getValueIsAdjusting()) {                       // 3. Filter
                        String selected = list.getSelectedValue();
                        status.setText(selected != null
                            ? "Selected: " + selected
                            : "Nothing selected");
                    }
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(list), BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking an item updates the status label to “Selected: [item].” The label is updated only once per click because of the `getValueIsAdjusting()` filter.

**Why the code produces this result.** The listener is registered on the `JList`. Without the `!e.getValueIsAdjusting()` guard, a single mouse click would fire two events (one on mouse-down and one on mouse-up), updating the label twice. The guard ensures only the final event triggers the update.

---

#### Example 2: Listening on the Selection Model Directly

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import java.awt.*;

public class ModelSelectionListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Model Selection Listener");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"Red", "Green", "Blue"};
            JList<String> list = new JList<>(items);
            JLabel status = new JLabel("No selection");

            list.getSelectionModel().addListSelectionListener(e -> {  // 1. Model
                if (!e.getValueIsAdjusting()) {
                    int index = list.getSelectedIndex();
                    status.setText(index >= 0
                        ? "Index: " + index + " — " + list.getSelectedValue()
                        : "No selection");
                }
            });

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(list), BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(300, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting an item shows its index and value in the status label. The event source is the `ListSelectionModel`, not the `JList`.

**Why the code produces this result.** The listener is registered on the selection model directly. This is the preferred approach when the selection model is shared with another component (e.g., a `JTable` and a `JList` sharing the same model) or when you need to distinguish the model as the event source.

---

### Real-World Cases

- **Master-detail views:** A list of items on the left and a detail panel on the right that updates when the selection changes.
- **Multi-list synchronization:** Two lists share a selection model so selecting in one selects the corresponding item in the other.
- **Form enabling:** A “Delete” button that is enabled only when at least one item is selected.
- **Data browsers:** A table whose rows are filtered or highlighted based on the selection.

---

## 2. valueChanged() — The Method

### Definitions

**Core Definition.** `valueChanged(ListSelectionEvent e)` is the single abstract method of `ListSelectionListener`. It is invoked when the selection in a list or table changes.

**Technical Definition.** `void valueChanged(ListSelectionEvent e)`. The method is invoked on the EDT by the `ListSelectionModel` whenever the selection state changes. The `ListSelectionEvent` provides the range of indices whose selection may have changed (`getFirstIndex()` and `getLastIndex()`) and the adjusting flag (`getValueIsAdjusting()`). If the method throws an unchecked exception, it propagates up the EDT.

**Beginner-Friendly Explanation.** `valueChanged` is the “selection just changed” method. Everything you write inside it runs on the EDT when the selection changes.

---

### Purposes

- To contain the reaction logic for selection changes.
- To receive the `ListSelectionEvent` context for deciding what to do based on the changed range.
- To trigger immediate UI updates (detail views, status labels, dependent controls).
- To synchronize selection state across components.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void valueChanged(ListSelectionEvent e) {
    if (e.getValueIsAdjusting()) {
        return; // ignore intermediate events
    }
    // react to final selection
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `@Override` | Confirms the method overrides the interface declaration. |
| `ListSelectionEvent e` | The event object with the changed range and adjusting flag. |
| `e.getValueIsAdjusting()` | `true` during an ongoing selection gesture. |

#### Syntax Rules

- The method must be `public` and accept a single `ListSelectionEvent` parameter.
- The method should perform its work quickly; long tasks must be offloaded.
- Always check `getValueIsAdjusting()` if the handler should run only once per gesture.

#### Constraints and Limitations

- The EDT is blocked until the method returns.
- Uncaught exceptions can disrupt the event pipeline.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Guarding Against Intermediate Events

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;

public class GuardedSelectionHandler {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Guarded Handler");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"One", "Two", "Three"};
            JList<String> list = new JList<>(items);

            list.addListSelectionListener(e -> {
                if (e.getValueIsAdjusting()) return;                  // 1. Guard
                System.out.println("Final selection: "
                    + list.getSelectedValue());
            });

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Two” prints “Final selection: Two” exactly once. The intermediate event (fired on mouse-down) is ignored.

**Why the code produces this result.** The guard `if (e.getValueIsAdjusting()) return;` ensures the handler runs only for the final event in the chain, which is the one fired when the mouse button is released and the selection has stabilized.

---

### Real-World Cases

- **Detail panel updates:** Only update the detail view when the selection is final, not on every intermediate drag event.
- **Button enabling:** Enable “Delete” only when the selection is finalized.
- **Selection synchronization:** Update a second list only once per selection gesture.

---

## 3. ListSelectionEvent — The Event Object

### Definitions

**Core Definition.** `ListSelectionEvent` is the event object delivered to `ListSelectionListener`s. It describes a change in the selection of a `JList` or `JTable`.

**Technical Definition.** `public class ListSelectionEvent extends EventObject`. It provides `getFirstIndex()` (the lower bound of the changed range), `getLastIndex()` (the upper bound), and `getValueIsAdjusting()` (whether the event is part of an ongoing chain). The event is fired by `ListSelectionModel` implementations, typically `DefaultListSelectionModel`.

**Beginner-Friendly Explanation.** `ListSelectionEvent` is the “what changed” note delivered to your listener. It tells you the range of indices that may have changed and whether the user is still in the middle of a selection gesture.

---

### Purposes

- To identify the range of indices whose selection may have changed.
- To distinguish intermediate events (`getValueIsAdjusting() == true`) from final events (`false`).
- To support efficient handling without querying the entire selection state.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
if (!e.getValueIsAdjusting()) {
    int first = e.getFirstIndex();
    int last = e.getLastIndex();
    // react to the changed range
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getFirstIndex()` | The lower bound of the changed range. |
| `getLastIndex()` | The upper bound of the changed range. |
| `getValueIsAdjusting()` | `true` if part of an ongoing chain of events. |

#### Syntax Rules

- `getValueIsAdjusting()` should always be checked if the handler should run only once per selection gesture.
- `getFirstIndex()` and `getLastIndex()` define an inclusive range, but the actual selection may not have changed for every index in that range.

#### Constraints and Limitations

- The event does not carry the new selection state; query the list or model via `getSelectedIndex`, `getSelectedIndices`, `getSelectedValue`, or `getSelectedValuesList`.
- Moving the lead with Ctrl held can fire events even when the selection set has not changed.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the Event Range

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;

public class InspectSelectionEvent {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Inspect Selection Event");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"A", "B", "C", "D"};
            JList<String> list = new JList<>(items);
            list.setSelectionMode(
                ListSelectionModel.MULTIPLE_INTERVAL_SELECTION);

            list.addListSelectionListener(e -> {
                if (e.getValueIsAdjusting()) return;                 // 1. Filter
                System.out.println("Changed range: "
                    + e.getFirstIndex() + " to " + e.getLastIndex()); // 2. Range
                System.out.println("Selected: "
                    + list.getSelectedValuesList());                  // 3. State
            });

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting “B” prints “Changed range: 1 to 1” and “Selected: [B]”. Shift-clicking to extend the selection to “D” prints “Changed range: 1 to 3” and “Selected: [B, C, D]”.

**Why the code produces this result.** The event range indicates which indices were involved in the change. The actual selection state is retrieved from the list after the event has settled.

---

### Real-World Cases

- **Efficient range updates:** Only refresh the portion of a custom view that corresponds to the changed range.
- **Logging and auditing:** Record the exact range of indices affected by each selection change.

---

## 4. List Selection Best Practices

### Definitions

**Core Definition.** List selection best practices are the conventions and patterns that ensure selection handling is correct, thread-safe, and performant.

**Technical Definition.** Best practices include: always checking `getValueIsAdjusting()` before acting on a selection change; registering on the `ListSelectionModel` rather than the `JList` when the model is shared or when you need to distinguish the source; avoiding heavy work in `valueChanged`; using `SwingUtilities.invokeLater` to defer UI updates that depend on the selection having fully settled; removing listeners when components are disposed; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for list selection handling are the “rules of thumb” that prevent duplicate handling, frozen GUIs, and memory leaks. They include things like “check `getValueIsAdjusting()`,” “don't do heavy work in the handler,” and “remove listeners when you're done.”

---

### Purposes

- To avoid duplicate handling by filtering intermediate events.
- To keep the GUI responsive by offloading long work.
- To prevent memory leaks by removing listeners.
- To choose the correct registration target (model vs. list).
- To ensure thread safety by respecting the EDT.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {
    JList<String> list = new JList<>(items);
    list.getSelectionModel().addListSelectionListener(e -> {        // 1. Model
        if (e.getValueIsAdjusting()) return;                        // 2. Guard
        // 3. Lightweight reaction
        updateDetailView(list.getSelectedValue());
    });
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Guard against intermediate events | `if (e.getValueIsAdjusting()) return;` |
| Register on the model for shared selections | `list.getSelectionModel().addListSelectionListener(...)` |
| Lightweight handlers | No heavy computation or I/O in `valueChanged`. |
| Offload long work | `SwingWorker` or background thread for expensive reactions. |
| Remove listeners | `removeListSelectionListener(...)` on disposal. |
| EDT | `SwingUtilities.invokeLater(...)` for construction. |

#### Syntax Rules

- **Always check `getValueIsAdjusting()`** to avoid running the handler twice per click.
- **Prefer registering on the `ListSelectionModel`** when the selection is shared or when you need the model as the event source.
- **Never perform heavy work directly in `valueChanged`**; use `SwingWorker`.
- **Always remove the listener** from long-lived models when the listening component is disposed.

#### Constraints and Limitations

- `getValueIsAdjusting()` does not fire for keyboard navigation the same way as mouse selection; keyboard arrow selection typically fires a single event with `getValueIsAdjusting() == false`.
- Moving the lead with Ctrl held can fire events without changing the actual selection.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Master-Detail View

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import java.awt.*;

public class BestPracticeSelection {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeSelection::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice Selection");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        String[] items = {"Alpha", "Beta", "Gamma", "Delta"};
        JList<String> list = new JList<>(items);
        list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

        JLabel detail = new JLabel("Select an item to see details.");

        list.getSelectionModel().addListSelectionListener(e -> {     // 1. Model
            if (e.getValueIsAdjusting()) return;                     // 2. Guard
            String selected = list.getSelectedValue();
            detail.setText(selected != null
                ? "Details for: " + selected
                : "No selection");
        });

        frame.setLayout(new BorderLayout());
        frame.add(new JScrollPane(list), BorderLayout.WEST);
        frame.add(detail, BorderLayout.CENTER);
        frame.setSize(450, 250);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Selecting an item updates the detail label exactly once, after the selection is finalized.

**Why the code produces this result.** The listener is registered on the selection model, the `getValueIsAdjusting()` guard prevents duplicate handling, and the reaction is lightweight. This is the recommended pattern for master-detail views.

---

#### Example 2: Common Mistake — Ignoring `getValueIsAdjusting()`

```java
import javax.swing.*;

public class BadSelectionHandler {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Selection Handler");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"One", "Two", "Three"};
            JList<String> list = new JList<>(items);

            list.addListSelectionListener(e -> {                     // WRONG: no guard
                System.out.println("Selected: " + list.getSelectedValue());
            });

            frame.add(new JScrollPane(list));
            frame.setSize(250, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking “Two” prints “Selected: Two” **twice**—once for the intermediate event and once for the final event.

**Why this is wrong.** As documented in the Java bug database and confirmed by community discussions, a single mouse click on a `JList` fires two `valueChanged` events: one with `valueIsAdjusting == true` (mouse-down) and one with `valueIsAdjusting == false` (mouse-up). Without the guard, the handler runs twice, which can cause duplicate side effects (e.g., database writes, UI flicker, or double-processing).

---

#### Example 3: Best-Practice with Offloaded Work

```java
import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import java.util.List;

public class OffloadedSelectionWork {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Offloaded Selection Work");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            String[] items = {"Report A", "Report B", "Report C"};
            JList<String> list = new JList<>(items);
            JLabel status = new JLabel("Ready");

            list.addListSelectionListener(e -> {
                if (e.getValueIsAdjusting()) return;
                String selected = list.getSelectedValue();
                if (selected == null) return;

                new SwingWorker<String, Void>() {                     // 1. Offload
                    @Override
                    protected String doInBackground() {
                        return loadReportData(selected);              // 2. Background
                    }
                    @Override
                    protected void done() {
                        try {
                            status.setText("Loaded: " + get());       // 3. EDT
                        } catch (Exception ex) {
                            status.setText("Error loading report");
                        }
                    }
                }.execute();
            });

            frame.setLayout(new BorderLayout());
            frame.add(new JScrollPane(list), BorderLayout.CENTER);
            frame.add(status, BorderLayout.SOUTH);
            frame.setSize(350, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static String loadReportData(String name) {
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        return name + " data";
    }
}
```

**Expected Output.** Selecting a report loads its data in the background and updates the status label when complete. The UI remains responsive during loading.

**Why the code produces this result.** The selection handler is lightweight—it only checks the selection and starts a `SwingWorker`. The heavy data-loading work runs off the EDT, and the UI update occurs in `done()`, which runs on the EDT. This pattern is essential for any selection handler that triggers I/O or computation.

---

### Real-World Cases

- **Production Swing applications** always guard `valueChanged` with `getValueIsAdjusting()` and register on the selection model for shared selections.
- **Master-detail interfaces** use the guard to update the detail view only once per selection gesture.
- **Data-intensive applications** offload report loading or database queries to `SwingWorker` inside the selection handler.
- **Educational examples** use the best-practice skeleton to demonstrate correct selection handling from the start.

---

## References

Oracle. *How to Write a List Selection Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/listselectionlistener.html

Oracle. *Interface ListSelectionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ListSelectionListener.html

Oracle. *Class ListSelectionEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/event/ListSelectionEvent.html

Oracle. *Interface ListSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/ListSelectionModel.html

Oracle. *Class JList — addListSelectionListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JList.html#addListSelectionListener-javax.swing.event.ListSelectionListener-

Oracle. *Class DefaultListSelectionModel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/DefaultListSelectionModel.html

Stack Overflow. *JList valueChanged fires twice*. https://stackoverflow.com/revisions/f9c1abac-da75-4293-a315-a744a85cfb54/view-source

Stack Overflow. *What exactly does getValueIsAdjusting do?*. https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/10860419/what-exactly-does-getvalueisadjusting-do

Stack Overflow. *JList ListSelectionListener best practices*. https://stackoverflow.com/questions/ (community consensus on guarding and model registration)

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *Class SwingWorker*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html