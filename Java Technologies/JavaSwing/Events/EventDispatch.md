# Event Dispatch Thread (EDT) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** The **Event Dispatch Thread (EDT)** is the single thread on which all Swing event handling and painting code executes. It is the only thread permitted to construct, modify, or query Swing components and their models.

**Technical Definition.** Swing's threading policy states: "In general Swing is **not thread safe**. All Swing components and related classes, unless otherwise documented, must be accessed on the **event dispatching thread**". The EDT is a non-daemon thread started automatically by the AWT toolkit. It dequeues events from the system event queue (`java.awt.EventQueue`) and dispatches them **synchronously** to registered listeners. This synchronous dispatch ensures that each event handler finishes executing before the next one begins, and that painting is never interrupted by events. Because all user-generated events are dispatched on the EDT, most developers are not directly impacted by the restriction—but constructing and showing a Swing application must be transferred to the EDT, and any long-running work must be moved **off** the EDT.

**Beginner-Friendly Explanation.** The EDT is the "single waiter" of a Swing application. Every click, keystroke, repaint, and window event is handled by this one thread, one at a time. If the waiter takes too long with one order, every other customer waits—the GUI freezes. Swing components are not thread-safe, so if multiple threads tried to modify them simultaneously, the display could become corrupted or the application could deadlock. The EDT prevents this by serializing all component access through a single thread. You interact with the EDT in two directions: you **schedule work onto it** (`invokeLater`, `invokeAndWait`) when you need to touch the GUI from another thread, and you **move work off it** (`SwingWorker`, background threads) when you need to perform time-consuming operations without freezing the interface.

---

### Key Characteristics

1. **Single-threaded dispatch.** All events are delivered sequentially on the EDT. A slow listener blocks every subsequent event, including repaints, producing an unresponsive UI.
2. **Not thread-safe components.** Swing components and models must be accessed **only** on the EDT. Modifying a model on a separate thread "runs the risk of exceptions and possible display corruption".
3. **Main thread is not the EDT.** Calls to an application's `main` method are not invoked on the EDT. Transferring control to the EDT via `invokeLater` is mandatory for GUI construction.
4. **Scheduling APIs.** `SwingUtilities.invokeLater` schedules a task asynchronously; `SwingUtilities.invokeAndWait` schedules it synchronously and blocks the caller; `SwingUtilities.isEventDispatchThread` identifies whether the current thread is the EDT.
5. **Model access included.** The EDT restriction applies not only to visual components but also to the **models** attached to them (e.g., `TableModel`, `ListModel`, `Document`).
6. **Exception on unwind.** If a `Runnable` submitted to `invokeLater` throws an uncaught exception, the **EDT unwinds**, not the submitting thread. For `invokeAndWait`, the exception is caught and rethrown as an `InvocationTargetException` on the caller's thread.
7. **No blocking on `invokeAndWait` from EDT.** Calling `invokeAndWait` from the EDT itself causes a deadlock and must never be done.
8. **Automatic thread creation.** The EDT is started automatically and is tied closely to the system's windowing implementation.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JFrame`, `JPanel`, `JButton`, and at least one layout manager.
- Understanding of threads: the difference between the main thread, background threads, and the EDT.
- Familiarity with `Runnable` and the `Thread` class.
- A JDK (version 8 or later; the EDT infrastructure is unchanged in modern JDKs).

---

### Related Programming Areas

- **`java.awt.EventQueue`:** The underlying event queue that the EDT services. `SwingUtilities.invokeLater` is a cover for `EventQueue.invokeLater()`.
- **`javax.swing.SwingUtilities`:** The utility class that provides `invokeLater`, `invokeAndWait`, and `isEventDispatchThread`.
- **`javax.swing.SwingWorker`:** The standard mechanism for performing long-running work off the EDT and posting interim and final results back onto it.
- **`javax.swing.Timer`:** A timer whose `ActionEvent`s are dispatched on the EDT, making it safe for UI updates.
- **`java.awt.EventDispatchThread`:** The internal thread class that runs the EDT loop.
- **Swing's threading policy** (documented in `javax.swing` package summary): the authoritative specification of EDT rules.

---

## Core Concepts and Features

---

## 1. The Event Dispatch Thread (EDT)

### Definitions

**Core Definition.** The EDT is the dedicated thread that runs all Swing event handling and painting code, ensuring that component access is serialized.

**Technical Definition.** Swing event-handling and painting code executes in a single thread, called the event-dispatching thread. The EDT is started automatically by the AWT toolkit and runs an infinite loop that dequeues events from the system event queue and dispatches them to registered listeners. All user-generated events (mouse, keyboard, window, focus) are delivered on the EDT. Swing component methods are not thread-safe; invoking them from multiple threads "risks thread interference or memory consistency errors".

**Beginner-Friendly Explanation.** The EDT is the one and only thread that is allowed to touch Swing components. It processes events one at a time, in the order they arrive. Everything else in your program—file I/O, network calls, heavy computation—must either run on a background thread or be scheduled onto the EDT via `invokeLater`.

---

### Purposes

- To serialize all Swing component access, preventing race conditions and data corruption.
- To dispatch user-generated events (clicks, keystrokes, window events) to registered listeners in a predictable order.
- To perform repainting and layout updates consistently without interruption from other events.
- To provide a single, predictable execution context for event-handling code.
- To keep the GUI responsive by processing events one at a time in a dedicated thread.

---

### Syntax Rules and Structure

#### Complete General Syntax (Starting a Swing Application on the EDT)

```java
import javax.swing.SwingUtilities;

public class MyApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // All GUI construction and showing happens here, on the EDT
            createAndShowGUI();
        });
    }

    private static void createAndShowGUI() {
        // construct and show the GUI
    }
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `SwingUtilities.invokeLater(Runnable)` | Schedules the `Runnable` on the EDT and returns immediately. |
| `createAndShowGUI()` | A method containing all GUI construction logic, invoked on the EDT. |
| `main(String[])` | The entry point; runs on the **main thread**, not the EDT. |

#### Syntax Rules

- **All Swing component construction and modification must occur on the EDT**, except as explicitly documented otherwise.
- The `main` method must transfer control to the EDT before creating any GUI.
- Models attached to components (e.g., `TableModel`, `ListModel`, `Document`) must also be modified on the EDT.

#### Constraints and Limitations

- The EDT is a single thread; any long-running task on it blocks all event processing and freezes the GUI.
- Swing components are not thread-safe; accessing them from a non-EDT thread is unsafe even for reading.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Verifying the EDT

```java
import javax.swing.SwingUtilities;

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

#### Example 2: Standard Swing Application Startup

```java
import javax.swing.*;

public class StandardStartup {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(StandardStartup::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Standard Startup");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.add(new JButton("Hello"));
        frame.setSize(300, 150);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window appears with a “Hello” button. The GUI is constructed entirely on the EDT, avoiding race conditions and deadlocks.

**Why the code produces this result.** The `main` method delegates GUI construction to `createAndShowGUI` via `invokeLater`, ensuring all Swing component access occurs on the EDT.

---

### Real-World Cases

- **Every Swing application** uses `invokeLater` in `main` to transfer control to the EDT before constructing the GUI.
- **Background task completion** posts UI updates to the EDT via `invokeLater` or `SwingWorker.done()`.
- **Model updates** (e.g., `TableModel`, `ListModel`) are performed on the EDT to avoid display corruption.
- **Timers and animations** use `javax.swing.Timer`, which fires on the EDT.

---

## 2. SwingUtilities.invokeLater()

### Definitions

**Core Definition.** `SwingUtilities.invokeLater(Runnable)` causes the `Runnable`'s `run()` method to be executed **asynchronously** on the EDT, after all pending AWT events have been processed.

**Technical Definition.** `public static void invokeLater(Runnable doRun)`. The method queues the `Runnable` on the EDT's event queue and returns immediately. It can be called from **any thread**, including the EDT itself; if called from the EDT, the `Runnable` is still deferred until all pending events have been processed. As of JDK 1.3, it is a cover for `java.awt.EventQueue.invokeLater()`. If the `Runnable` throws an uncaught exception, the **EDT unwinds**, not the submitting thread.

**Beginner-Friendly Explanation.** `invokeLater` is the "drop off a note" method. You write a `Runnable` describing the GUI work you want done, drop it in the EDT's inbox, and walk away. The EDT eventually reads the note and executes it. You do not wait for it to finish.

---

### Purposes

- To transfer control from the main thread (or any other thread) to the EDT for safe GUI construction and mutation.
- To allow background threads to update the GUI safely after completing a task.
- To schedule GUI work without blocking the calling thread.
- To ensure that component state changes are serialized with event dispatch and repainting.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
SwingUtilities.invokeLater(new Runnable() {
    @Override
    public void run() {
        // GUI code here
    }
});

// or with a lambda:
SwingUtilities.invokeLater(() -> {
    // GUI code here
});
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `SwingUtilities.invokeLater(Runnable)` | Schedules the `Runnable` on the EDT. |
| `Runnable.run()` | Contains the GUI code to execute on the EDT. |

#### Syntax Rules

- The `Runnable` must not be `null`; passing `null` results in a `NullPointerException`.
- `invokeLater` returns immediately; it does not block the caller.
- If the EDT is not yet started, `invokeLater` starts it.
- If called from the EDT, the `Runnable` is queued and executed after the current event handler returns.

#### Constraints and Limitations

- `invokeLater` cannot return a result; for results, use `invokeAndWait` or a `Future`-based mechanism.
- Excessive use of `invokeLater` can flood the EDT; keep scheduled tasks small.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic `invokeLater` from Main

```java
import javax.swing.*;

public class InvokeLaterDemo {
    public static void main(String[] args) {
        System.out.println("Before invokeLater: " + Thread.currentThread());

        SwingUtilities.invokeLater(() -> {                          // 1. Schedule
            System.out.println("Inside invokeLater: " + Thread.currentThread());
            JFrame frame = new JFrame("invokeLater Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setVisible(true);
        });

        System.out.println("After invokeLater: " + Thread.currentThread());
    }
}
```

**Expected Output.**
```
Before invokeLater: Thread[main,5,main]
After invokeLater: Thread[main,5,main]
Inside invokeLater: Thread[AWT-EventQueue-0,6,main]
```

**Why the code produces this result.** The `main` thread prints its messages immediately. The `Runnable` is queued on the EDT and executed later, as shown by the different thread name (`AWT-EventQueue-0`).

---

#### Example 2: `invokeLater` from a Background Thread

```java
import javax.swing.*;

public class BackgroundUpdate {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Background Update");
            JLabel label = new JLabel("Loading...");
            frame.add(label);
            frame.setSize(300, 100);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);

            new Thread(() -> {                                    // 1. Background
                try { Thread.sleep(1500); } catch (InterruptedException e) {}
                SwingUtilities.invokeLater(() -> {                // 2. Post to EDT
                    label.setText("Loaded successfully.");
                });
            }).start();
        });
    }
}
```

**Expected Output.** The window shows “Loading...” for about 1.5 seconds, then changes to “Loaded successfully.”

**Why the code produces this result.** The background thread sleeps without blocking the EDT. After the sleep, it calls `invokeLater` to schedule the label update on the EDT, which is the only thread allowed to mutate the `JLabel`.

---

### Real-World Cases

- **Application startup:** every Swing application calls `invokeLater` in `main` to construct the GUI.
- **Background task completion:** after a file download or database query, `invokeLater` updates the GUI with the result.
- **Progress reporting:** worker threads post interim progress updates to the EDT via `invokeLater`.
- **Model listener deferral:** application listeners use `invokeLater` to update the UI after model changes have been processed.

---

## 3. SwingUtilities.invokeAndWait()

### Definitions

**Core Definition.** `SwingUtilities.invokeAndWait(Runnable)` causes the `Runnable`'s `run()` method to be executed **synchronously** on the EDT. The calling thread **blocks** until the `Runnable` completes.

**Technical Definition.** `public static void invokeAndWait(Runnable doRun) throws InterruptedException, InvocationTargetException`. The method queues the `Runnable` on the EDT and blocks the caller until all pending events have been processed and `doRun.run()` returns. It **must not** be called from the EDT itself; doing so causes a deadlock. If `doRun.run()` throws an exception, it is caught and rethrown as an `InvocationTargetException` on the caller's thread. As of JDK 1.3, it is a cover for `java.awt.EventQueue.invokeAndWait()`.

**Beginner-Friendly Explanation.** `invokeAndWait` is the "hand in a form and wait" method. You submit a `Runnable`, and your thread waits until the EDT has finished executing it. Use it when you need the result before continuing—but never call it from the EDT itself, or you will deadlock.

---

### Purposes

- To transfer control to the EDT and **wait** for the task to finish.
- To obtain results from EDT-executed code before continuing.
- To synchronize a background thread with GUI state changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
try {
    SwingUtilities.invokeAndWait(() -> {
        // GUI code here
    });
} catch (InterruptedException | InvocationTargetException e) {
    e.printStackTrace();
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `invokeAndWait(Runnable)` | Schedules the `Runnable` and blocks until it completes. |
| `InterruptedException` | Thrown if the caller is interrupted while waiting. |
| `InvocationTargetException` | Wraps an exception thrown inside `doRun.run()`. |

#### Syntax Rules

- **Must not be called from the EDT**; doing so throws an `Error` or deadlocks.
- The caller is blocked until the `Runnable` finishes.
- Exceptions from the `Runnable` are rethrown on the caller's thread as `InvocationTargetException`.

#### Constraints and Limitations

- Deadlocks easily if misused; `invokeLater` is generally safer.
- Blocks the calling thread, reducing concurrency.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: `invokeAndWait` with Result Retrieval

```java
import javax.swing.*;
import java.lang.reflect.InvocationTargetException;

public class InvokeAndWaitDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("invokeAndWait Demo");
            JLabel label = new JLabel("Waiting...");
            frame.add(label);
            frame.setSize(300, 100);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);

            new Thread(() -> {
                try {
                    SwingUtilities.invokeAndWait(() -> {           // 1. Schedule
                        label.setText("Updated via invokeAndWait");
                    });
                    System.out.println("Update completed before this line."); // 2. Waited
                } catch (InterruptedException | InvocationTargetException e) {
                    e.printStackTrace();
                }
            }).start();
        });
    }
}
```

**Expected Output.** The label updates to “Updated via invokeAndWait” before the background thread prints “Update completed before this line.”

**Why the code produces this result.** `invokeAndWait` blocks the background thread until the EDT has finished executing the label update, guaranteeing that the update is complete before the thread continues.

---

#### Example 2: Common Mistake — Calling `invokeAndWait` from the EDT

```java
import javax.swing.*;

public class BadInvokeAndWait {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // WRONG: invokeAndWait called from the EDT
            try {
                SwingUtilities.invokeAndWait(() -> {               // 1. Deadlock
                    System.out.println("This will deadlock");
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    }
}
```

**Expected Output.** The application deadlocks or throws an `Error` because the EDT is waiting for itself to finish.

**Why this is wrong.** As the documentation states, "It shouldn't be called from the event dispatching thread". Calling `invokeAndWait` from the EDT makes the EDT wait for a task that can only run after the current EDT task finishes—a classic deadlock. Use `invokeLater` instead if you are already on the EDT.

---

### Real-World Cases

- **Synchronizing background threads** with GUI state before proceeding.
- **Retrieving values** from EDT-executed code (e.g., reading the current selection).
- **Rarely used** in modern Swing code; `SwingWorker` is preferred for most background-to-EDT communication.

---

## 4. SwingUtilities.isEventDispatchThread()

### Definitions

**Core Definition.** `SwingUtilities.isEventDispatchThread()` returns `true` if the current thread is the EDT, and `false` otherwise.

**Technical Definition.** `public static boolean isEventDispatchThread()`. The method is a cover for `java.awt.EventQueue.isDispatchThread()`. It is used to determine whether the current execution context is the EDT, which is essential for writing methods that can be safely called from any thread—performing the work directly if already on the EDT, or scheduling it via `invokeLater` if not.

**Beginner-Friendly Explanation.** `isEventDispatchThread` is a "check the room" method. It tells you whether you are currently on the EDT. If you are, you can touch Swing components directly. If you are not, you must use `invokeLater` to schedule the work.

---

### Purposes

- To determine whether the current thread is the EDT.
- To write thread-neutral methods that work correctly from any thread.
- To avoid unnecessary `invokeLater` calls when already on the EDT.
- To debug threading issues by logging the current thread.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
if (SwingUtilities.isEventDispatchThread()) {
    // safe to touch Swing components directly
} else {
    SwingUtilities.invokeLater(() -> {
        // schedule on the EDT
    });
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `isEventDispatchThread()` | Returns `true` if the calling thread is the EDT. |

#### Syntax Rules

- Safe to call from any thread.
- Does not block or schedule anything; it simply reports the current thread's identity.

#### Constraints and Limitations

- A `true` result does not guarantee that the component is displayable; it only confirms the thread.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Thread-Neutral Method

```java
import javax.swing.*;

public class ThreadNeutral {
    public static void updateLabel(JLabel label, String text) {
        if (SwingUtilities.isEventDispatchThread()) {               // 1. Check
            label.setText(text);                                    // 2. Direct
        } else {
            SwingUtilities.invokeLater(() ->                        // 3. Schedule
                label.setText(text));
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Thread-Neutral");
            JLabel label = new JLabel("Initial");
            frame.add(label);
            frame.setSize(300, 100);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);

            new Thread(() ->
                updateLabel(label, "Updated from background")).start();
        });
    }
}
```

**Expected Output.** The label initially shows “Initial” and is updated to “Updated from background” via the thread-neutral method, which schedules the update on the EDT because the caller is a background thread.

**Why the code produces this result.** `updateLabel` checks `isEventDispatchThread()`; since the background thread is not the EDT, it uses `invokeLater` to schedule the update safely.

---

### Real-World Cases

- **Utility methods** that may be called from either the EDT or a background thread.
- **Debugging** to log whether the current thread is the EDT.
- **Guard clauses** in custom components that assert they are called on the EDT.

---

## 5. Thread Safety — Swing's Threading Policy

### Definitions

**Core Definition.** Swing's threading policy states that Swing components and their models are **not thread-safe** and must be accessed only on the EDT, unless a specific method is documented as thread-safe.

**Technical Definition.** "In general Swing is not thread safe. All Swing components and related classes, unless otherwise documented, must be accessed on the event dispatching thread". The restriction applies not only to visual components but also to the **models** attached to them (e.g., `TableModel`, `ListModel`, `Document`). Modifying a model on a separate thread "runs the risk of exceptions and possible display corruption". There is one subtle exception: an application listener that reacts to a model change may need to update the UI via `invokeLater` so that component-installed listeners can finish processing first.

**Beginner-Friendly Explanation.** Swing's thread safety rule is simple: if you are touching a Swing component or its model, you must be on the EDT. Breaking this rule causes symptoms that range from subtle painting glitches to hard-to-reproduce crashes. The good news is that most of your code already runs on the EDT—event listeners, `SwingWorker` callbacks, and `Timer` actions all execute there. The tricky part is anything that runs on a **background thread**: you must either avoid touching Swing entirely or use `invokeLater` to schedule the touch.

---

### Purposes

- To prevent race conditions, memory consistency errors, and data corruption.
- To ensure that painting and layout are consistent and deterministic.
- To provide a clear, enforceable rule for where GUI code can run.
- To enable safe interaction between background threads and the GUI via scheduling APIs.

---

### Syntax Rules and Structure

#### Complete General Syntax (Thread-Safe Access Pattern)

```java
// From a background thread:
SwingUtilities.invokeLater(() -> {
    // safe to access Swing components here
});

// Or use SwingWorker:
new SwingWorker<T, V>() {
    protected T doInBackground() { /* background work */ }
    protected void done() { /* EDT update */ }
}.execute();
```

#### Component Breakdown

| Rule | Implementation |
|------|----------------|
| Construct GUI on EDT | `SwingUtilities.invokeLater` in `main`. |
| Update GUI from background | `invokeLater` or `SwingWorker.done()`. |
| Modify models on EDT | Schedule model changes on the EDT. |
| Never block EDT | Use `SwingWorker` for long tasks. |
| Never `invokeAndWait` from EDT | Use `invokeLater` when already on the EDT. |

#### Syntax Rules

- **All Swing component and model access must occur on the EDT.**
- **Background threads must use `invokeLater`** to schedule GUI updates.
- **Long-running tasks must be moved off the EDT** using `SwingWorker` or a dedicated background thread.
- **Model listeners that need to update the UI** should do so via `invokeLater` to avoid rendering issues.

#### Constraints and Limitations

- There is no compile-time enforcement; violations produce runtime symptoms.
- Some methods (e.g., `repaint()`, `revalidate()`, `add/remove listener`) are documented as safe from any thread, but these are exceptions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Model Modification on the EDT

```java
import javax.swing.*;
import javax.swing.table.DefaultTableModel;

public class ModelThreadSafety {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Model Thread Safety");
            DefaultTableModel model = new DefaultTableModel(
                new Object[]{"Name"}, 0);
            JTable table = new JTable(model);
            frame.add(new JScrollPane(table));
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);

            new Thread(() -> {
                // WRONG: modifying model off the EDT
                // model.addRow(new Object[]{"Bad"});

                // CORRECT: schedule on the EDT
                SwingUtilities.invokeLater(() -> {                 // 1. Schedule
                    model.addRow(new Object[]{"Safe"});            // 2. EDT update
                });
            }).start();
        });
    }
}
```

**Expected Output.** A row containing “Safe” is added to the table without display corruption, because the model modification occurs on the EDT.

**Why the code produces this result.** The background thread uses `invokeLater` to schedule the model modification on the EDT. Modifying the `DefaultTableModel` directly from the background thread could corrupt the table's internal state and produce exceptions or painting glitches.

---

#### Example 2: Common Mistake — Accessing Swing from a Background Thread

```java
import javax.swing.*;

public class BadThreadAccess {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Bad Access");
            JLabel label = new JLabel("Initial");
            frame.add(label);
            frame.setSize(300, 100);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);

            new Thread(() -> {
                label.setText("Updated from background");          // WRONG
            }).start();
        });
    }
}
```

**Expected Output.** The label may update correctly, or it may not—the behavior is unpredictable. In more complex scenarios, this can cause painting artifacts, `ConcurrentModificationException`, or a frozen GUI.

**Why this is wrong.** `JLabel.setText` is not thread-safe. Modifying the label from a background thread violates Swing's threading policy and can lead to thread interference and memory consistency errors. The correct approach is to wrap the call in `SwingUtilities.invokeLater`.

---

### Real-World Cases

- **Table and list models** are updated on the EDT to avoid display corruption.
- **Document models** in text components are modified on the EDT to keep the view synchronized.
- **Background workers** post results via `SwingWorker.done()` or `invokeLater`.
- **Model listeners** that need to update the UI use `invokeLater` to defer the update until after component listeners have processed the change.

---

## 6. EDT Best Practices

### Definitions

**Core Definition.** EDT best practices are the conventions and patterns that ensure Swing applications are thread-safe, responsive, and maintainable by respecting the EDT.

**Technical Definition.** Best practices include: constructing and showing the GUI on the EDT via `SwingUtilities.invokeLater`; never blocking the EDT with I/O, sleeps, or heavy computation; using `SwingWorker` for long-running tasks; using `invokeLater` to post UI updates from background threads; never calling `invokeAndWait` from the EDT; using `isEventDispatchThread()` to write thread-neutral methods; modifying component models only on the EDT; and removing listeners when no longer needed.

**Beginner-Friendly Explanation.** Best practices for the EDT are the "rules of the road" that keep Swing apps responsive and stable. They include things like "build the GUI on the EDT," "don't sleep in an event handler," "use `SwingWorker` for long tasks," and "never call `invokeAndWait` from the EDT."

---

### Purposes

- To maintain thread safety by respecting the EDT for all component and model access.
- To keep the GUI responsive by offloading long-running work to background threads.
- To avoid deadlocks by never calling `invokeAndWait` from the EDT.
- To prevent memory leaks by removing listeners from long-lived sources.
- To write clean, maintainable threading code using `SwingWorker` and `invokeLater`.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
SwingUtilities.invokeLater(() -> {                                // 1. EDT startup
    JFrame frame = new JFrame("Best Practice");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

    JButton button = new JButton("Start Work");
    button.addActionListener(e -> startWork());                    // 2. Listener

    frame.add(button);
    frame.setSize(400, 200);
    frame.setLocationRelativeTo(null);
    frame.setVisible(true);
});

private static void startWork() {
    new SwingWorker<Void, Void>() {                                // 3. Background
        @Override
        protected Void doInBackground() {
            // long-running work here (not on EDT)
            return null;
        }
        @Override
        protected void done() {
            // UI update here (on EDT)
        }
    }.execute();
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Construct GUI on EDT | `SwingUtilities.invokeLater(...)` |
| Offload long tasks | `SwingWorker` or background thread |
| Never block EDT | No `Thread.sleep`, I/O, or heavy loops in listeners |
| Post UI updates | `invokeLater` or `SwingWorker.done()` |
| Never `invokeAndWait` from EDT | Use `invokeLater` when on the EDT |
| Model access on EDT | Schedule model changes via `invokeLater` |
| Remove listeners | `removeXxxListener` when no longer needed |
| Thread-neutral methods | `if (isEventDispatchThread()) { ... } else { invokeLater(...); }` |

#### Syntax Rules

- **Always construct and show the GUI on the EDT** via `invokeLater`.
- **Always offload long-running work** to `SwingWorker`; never block the EDT.
- **Always use `invokeLater` to post UI updates** from background threads.
- **Never call `invokeAndWait` from the EDT**; it causes deadlock.
- **Always modify component models on the EDT**; off-EDT model changes risk display corruption.
- **Always remove listeners** from long-lived sources when the listening component is disposed.

#### Constraints and Limitations

- The EDT is a single thread; blocking it freezes the GUI.
- `SwingWorker`'s `doInBackground` runs off the EDT; `process` and `done` run on the EDT.
- `javax.swing.Timer` fires on the EDT, making it safe for UI updates.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Background Task with Progress

```java
import javax.swing.*;
import java.util.List;

public class BestPracticeEdt {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeEdt::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice EDT");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        JProgressBar bar = new JProgressBar(0, 100);
        bar.setStringPainted(true);
        JLabel status = new JLabel("Ready");

        JButton start = new JButton("Start");
        start.addActionListener(e -> {
            new SwingWorker<Void, Integer>() {                        // 1. Worker
                @Override
                protected Void doInBackground() throws Exception {
                    for (int i = 1; i <= 100; i++) {
                        Thread.sleep(30);                             // 2. Background
                        publish(i);                                    // 3. Post
                    }
                    return null;
                }
                @Override
                protected void process(List<Integer> chunks) {
                    int latest = chunks.get(chunks.size() - 1);
                    bar.setValue(latest);                              // 4. EDT
                    status.setText("Progress: " + latest + "%");
                }
                @Override
                protected void done() {
                    status.setText("Complete!");                       // 5. EDT
                }
            }.execute();
        });

        frame.add(bar, java.awt.BorderLayout.CENTER);
        JPanel south = new JPanel();
        south.add(start);
        south.add(status);
        frame.add(south, java.awt.BorderLayout.SOUTH);
        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Clicking “Start” begins a background task; the progress bar fills from 0% to 100% over about three seconds, with the status label updating on the EDT, and finally showing “Complete!”—without freezing the UI.

**Why the code produces this result.** `SwingWorker.doInBackground` runs on a background thread, keeping the EDT responsive. `publish` queues progress chunks; `process` and `done` run on the EDT, where updating the bar and label is safe.

---

#### Example 2: Common Mistake — Blocking the EDT

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
                    Thread.sleep(5000);                               // WRONG
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

**Why this is wrong.** As Swing's threading policy states, "a long running task, such as network io or computational intensive processing, executed on the event dispatching thread blocks the event [queue]". The correct approach is to use `SwingWorker` or a background thread and post UI updates via `invokeLater`.

---

### Real-World Cases

- **Production Swing applications** use `SwingWorker` for file I/O, network access, and heavy computation, updating the UI via `process`/`done`.
- **IDE and editor plugins** follow the same EDT discipline and remove listeners when panels are disposed.
- **Educational examples** use the best-practice skeleton to demonstrate correct EDT usage from the start.

---

## References

Oracle. *Class SwingUtilities*. Java Development Kit Version 16 API Specification. https://docs.oracle.com/en/java/javase/16/docs/api/java.desktop/javax/swing/SwingUtilities.html

Oracle. *Package javax.swing — Swing's Threading Policy*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/en/java/javase/26/docs/api/java.desktop/javax/swing/package-summary.html

Oracle. *How to Use Threads*. The Java Tutorials (Concurrency in Swing). https://docs.oracle.com/javase/tutorial/uiswing/concurrency/index.html

Oracle. *Initial Threads*. Java Platform, Standard Edition Documentation. https://docs.oracle.com/en/java/javase/

Oracle. *Class EventQueue*. Java Platform, Standard Edition API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/EventQueue.html

Oracle. *Class SwingWorker*. Java Platform, Standard Edition API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingWorker.html

Oracle. *Class Timer (javax.swing.Timer)*. Java Platform, Standard Edition API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/Timer.html