# WindowListener — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `WindowListener` is a low-level event listener interface in the `java.awt.event` package for receiving window lifecycle events. It declares **seven methods** that are invoked when a window is opened, closing, closed, iconified, deiconified, activated, or deactivated. `WindowAdapter` is an abstract adapter class that implements all seven methods with empty bodies, allowing developers to override only the events of interest.

**Technical Definition.** `public interface WindowListener extends EventListener`. The interface declares: `void windowOpened(WindowEvent e)`, `void windowClosing(WindowEvent e)`, `void windowClosed(WindowEvent e)`, `void windowIconified(WindowEvent e)`, `void windowDeiconified(WindowEvent e)`, `void windowActivated(WindowEvent e)`, and `void windowDeactivated(WindowEvent e)`. Listeners are registered on a `Window` (or its subclasses `JFrame`, `JDialog`) via `addWindowListener(WindowListener)`. All window events are dispatched on the Event Dispatch Thread (EDT). The `WindowEvent` object carries the window that fired the event (`getWindow()`) and, for activation/deactivation events, the opposite window (`getOppositeWindow()`). `WindowAdapter` implements all seven methods with empty bodies, so subclasses can override only the methods they care about.

**Beginner-Friendly Explanation.** `WindowListener` is the "window lifecycle" listener. It tells you when a window is opened, when the user tries to close it, when it has actually closed, when it is minimised (iconified), restored (deiconified), and when it gains or loses focus. The most common use is to intercept the window-closing event—for example, to ask the user "Are you sure you want to quit?" before exiting, or to save data before the window disappears. Because `WindowListener` has seven methods, most developers extend `WindowAdapter` and override only `windowClosing`. Think of `WindowListener` as the flight recorder for a window's life: it logs every major state change, and you decide which of those changes should trigger actions in your application.

---

### Key Characteristics

1. **Seven-method interface.** Covers the complete window lifecycle: opening, closing (requested), closed (completed), iconified, deiconified, activated, and deactivated.
2. **Adapter class available.** `WindowAdapter` implements all seven methods with empty bodies, so you can override only `windowClosing` (the most common case) without writing six empty methods.
3. **Window events are low-level.** They are fired by the AWT windowing system, not by Swing components. They are delivered on the EDT.
4. **`windowClosing` vs. `windowClosed`.** `windowClosing` fires **when the user requests** that the window be closed (e.g., clicks the X button). To actually close the window, the listener must call `dispose()` or `setVisible(false)`. `windowClosed` fires **after** the window has been disposed.
5. **`DO_NOTHING_ON_CLOSE` for custom close logic.** To take full control of window closing, set `setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE)` and handle everything in `windowClosing`.
6. **`WindowAdapter` saves boilerplate.** If you implement `WindowListener` directly, you must implement all seven methods—even the six you do not care about.
7. **Iconification and deiconification for resource management.** `windowIconified` is the ideal place to stop animations and release resources; `windowDeiconified` is the place to restart them.
8. **Activation and deactivation for focus-aware behaviour.** A window is activated when it or one of its components has keyboard focus; deactivated when it loses focus.

---

### Prerequisites

- Java language fundamentals: classes, interfaces, inheritance, anonymous inner classes, and lambda expressions.
- Familiarity with `JFrame`, `JDialog`, and basic Swing containers.
- Understanding of the Event Dispatch Thread (EDT) and `SwingUtilities.invokeLater`.
- Familiarity with `WindowEvent` and the `java.awt.event` package.
- Knowledge of `JOptionPane` for confirmation dialogs.
- A JDK (version 8 or later; `WindowListener` and `WindowAdapter` are in `java.awt.event`).

---

### Related Programming Areas

- **`WindowAdapter`:** The abstract adapter class for `WindowListener`.
- **`WindowEvent`:** The event object carrying the window and opposite window.
- **`WindowStateListener` and `WindowFocusListener`:** Additional listener interfaces for window state (maximised, minimised) and focus changes.
- **`JFrame.setDefaultCloseOperation`:** The `DO_NOTHING_ON_CLOSE` constant used to enable custom close handling.
- **`JOptionPane.showConfirmDialog`:** The standard way to ask the user for confirmation before closing.
- **`AbstractAction`:** Used to share close logic between a menu item and a window listener.
- **`System.exit` vs. `dispose()`:** The two ways to end a window's lifecycle—terminate the JVM or release the window's resources.

---

## Core Concepts and Features

---

## 1. WindowListener — The Interface

### Definitions

**Core Definition.** `WindowListener` is an interface for receiving window lifecycle events on a `Window` (including `JFrame` and `JDialog`).

**Technical Definition.** `public interface WindowListener extends EventListener`. The interface declares seven methods, each accepting a `WindowEvent`. Components register listeners with `addWindowListener`. If multiple listeners are registered, each is invoked in registration order. All listener methods are invoked on the EDT.

**Beginner-Friendly Explanation.** `WindowListener` is the "window events" contract. Implement it (or extend `WindowAdapter`) and register it with a window to hear about its lifecycle.

---

### Purposes

- To intercept the window-closing request and implement custom close logic (confirmation, save, cleanup).
- To perform actions when the window is first opened.
- To release resources when the window is closed or iconified.
- To restart resources when the window is deiconified.
- To track window activation and deactivation for focus-aware behaviour.
- To coordinate multiple windows in multi-window applications.

---

### Syntax Rules and Structure

#### Complete General Syntax (Full Implementation)

```java
window.addWindowListener(new WindowListener() {
    public void windowOpened(WindowEvent e)      { /* window shown */ }
    public void windowClosing(WindowEvent e)     { /* close requested */ }
    public void windowClosed(WindowEvent e)      { /* window disposed */ }
    public void windowIconified(WindowEvent e)   { /* minimised */ }
    public void windowDeiconified(WindowEvent e) { /* restored */ }
    public void windowActivated(WindowEvent e)   { /* gained focus */ }
    public void windowDeactivated(WindowEvent e) { /* lost focus */ }
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `windowOpened` | The window has been shown for the first time. |
| `windowClosing` | The user has requested that the window be closed (e.g., clicked the X). |
| `windowClosed` | The window has been disposed. |
| `windowIconified` | The window has been minimised to an icon. |
| `windowDeiconified` | The window has been restored from an icon. |
| `windowActivated` | The window or one of its components has gained keyboard focus. |
| `windowDeactivated` | The window and all its components have lost keyboard focus. |

#### Syntax Rules

- All seven methods must be implemented when implementing the interface directly.
- Use `WindowAdapter` to override only the methods of interest.
- Register the listener with `addWindowListener`.
- Listener methods are invoked on the EDT.

#### Constraints and Limitations

- `windowClosing` is a **request**; the window is not automatically closed. You must call `dispose()` or `setVisible(false)` to actually close it.
- If you handle `windowClosing` yourself, set `DO_NOTHING_ON_CLOSE` as the default close operation to prevent the default behaviour from interfering.
- `windowClosed` does not fire if you call `setVisible(false)` instead of `dispose()`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full WindowListener Implementation

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.WindowEvent;
import java.awt.event.WindowListener;

public class FullWindowListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Full WindowListener");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);  // 1. Custom

            frame.addWindowListener(new WindowListener() {               // 2. Register
                @Override
                public void windowOpened(WindowEvent e) {
                    System.out.println("Window opened");
                }
                @Override
                public void windowClosing(WindowEvent e) {               // 3. Closing
                    System.out.println("Window closing");
                    e.getWindow().dispose();                              // 4. Dispose
                }
                @Override
                public void windowClosed(WindowEvent e) {
                    System.out.println("Window closed");
                }
                @Override
                public void windowIconified(WindowEvent e) {
                    System.out.println("Window iconified");
                }
                @Override
                public void windowDeiconified(WindowEvent e) {
                    System.out.println("Window deiconified");
                }
                @Override
                public void windowActivated(WindowEvent e) {
                    System.out.println("Window activated");
                }
                @Override
                public void windowDeactivated(WindowEvent e) {
                    System.out.println("Window deactivated");
                }
            });

            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints "Window opened", "Window activated", and other lifecycle messages. Clicking the close button prints "Window closing" and then "Window closed" after the window is disposed.

**Why the code produces this result.** All seven methods are implemented. `DO_NOTHING_ON_CLOSE` prevents the default close behaviour, so the listener's `windowClosing` is the only place where closing is handled. `dispose()` releases the window's resources and triggers `windowClosed`.

---

### Real-World Cases

- **Confirmation before exit:** Asking "Are you sure?" when the user clicks the close button.
- **Save-on-close:** Saving data before the window is disposed.
- **Resource cleanup:** Closing database connections, stopping threads, or releasing file handles when the window closes.
- **Animation control:** Stopping animation threads when the window is iconified and restarting them when deiconified.
- **Multi-window coordination:** Disposing all child windows when the main window closes.

---

## 2. WindowAdapter — The Adapter Class

### Definitions

**Core Definition.** `WindowAdapter` is an abstract class that implements `WindowListener` with empty method bodies, allowing subclasses to override only the methods of interest.

**Technical Definition.** `public abstract class WindowAdapter extends Object implements WindowListener`. The class provides empty implementations of all seven `WindowListener` methods. It exists to reduce boilerplate: implementing `WindowListener` directly requires all seven methods, while extending `WindowAdapter` lets you override only `windowClosing` (or any other method). Registration is done via `addWindowListener`.

**Beginner-Friendly Explanation.** `WindowAdapter` is the "pre-filled form" for window events. Instead of implementing seven empty methods just to handle `windowClosing`, you extend `WindowAdapter` and override only the method you need.

---

### Purposes

- To reduce boilerplate when implementing `WindowListener`.
- To override only `windowClosing` (the most common case) cleanly.
- To enable readable anonymous inner class implementations.
- To avoid implementing unused methods.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
window.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        // reaction code
    }
});
```

#### Component Breakdown

| Adapter | Implements | Methods |
|---------|-----------|---------|
| `WindowAdapter` | `WindowListener` | 7 (all empty) |

#### Syntax Rules

- Extend the adapter and override only the methods you need.
- Register via `addWindowListener`.
- Adapters are abstract classes; lambdas cannot replace them (the interface has seven methods).

#### Constraints and Limitations

- Extending an adapter prevents extending another class (single inheritance).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: WindowAdapter for Confirmation on Close

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class ConfirmCloseAdapter {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Confirm Close");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);  // 1. Custom

            frame.addWindowListener(new WindowAdapter() {               // 2. Adapter
                @Override
                public void windowClosing(WindowEvent e) {               // 3. Override
                    int choice = JOptionPane.showConfirmDialog(
                        frame,
                        "Are you sure you want to exit?",
                        "Confirm Exit",
                        JOptionPane.YES_NO_OPTION);                      // 4. Ask
                    if (choice == JOptionPane.YES_OPTION) {
                        frame.dispose();                                  // 5. Dispose
                    }
                }
            });

            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the close button shows a confirmation dialog. Choosing "Yes" disposes the frame; choosing "No" keeps it open.

**Why the code produces this result.** `WindowAdapter` overrides only `windowClosing`. `DO_NOTHING_ON_CLOSE` prevents the default close behaviour, so the confirmation dialog is the only path to closing. The `JOptionPane` dialog asks the user, and `dispose()` is called only on confirmation.

---

### Real-World Cases

- **Confirmation dialogs** for exit.
- **Save-before-close** prompts.
- **Single-purpose close handlers** in multi-window applications.
- **Resource cleanup** in `windowClosed`.

---

## 3. Window Events — The Seven Methods

### 3.1 windowOpened()

#### Definitions

**Core Definition.** `windowOpened(WindowEvent e)` is invoked just after the window has been shown for the first time.

**Technical Definition.** `public void windowOpened(WindowEvent e)`. Fired by the AWT just after the listened-to window has been shown for the first time.

**Beginner-Friendly Explanation.** `windowOpened` fires when the window first appears on screen.

---

#### Purposes

- To perform initialisation that requires the window to be visible.
- To start animations or timers when the window appears.
- To log the window's opening.

---

#### Syntax Rules and Structure

```java
@Override
public void windowOpened(WindowEvent e) {
    // initialisation code
}
```

#### Constraints and Limitations

- Fires only once per window display; not when the window is hidden and re-shown.

---

#### Annotated Complete Step-by-Step Code Examples

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowOpened(WindowEvent e) {
        System.out.println("Window is now visible");
        startAnimation();
    }
});
```

**Expected Output.** The console prints "Window is now visible" once when the window first appears.

---

### 3.2 windowClosing()

#### Definitions

**Core Definition.** `windowClosing(WindowEvent e)` is invoked in response to a user request that the window be closed (e.g., clicking the close button).

**Technical Definition.** `public void windowClosing(WindowEvent e)`. Fired when the user attempts to close the window. **To actually close the window, the listener must invoke `dispose()` or `setVisible(false)`**. If `DO_NOTHING_ON_CLOSE` is set, this is the only place where closing is handled.

**Beginner-Friendly Explanation.** `windowClosing` fires when the user tries to close the window. You decide whether to actually close it.

---

#### Purposes

- To intercept the close request and ask for confirmation.
- To save data before the window closes.
- To prevent the window from closing under certain conditions.
- To perform cleanup before disposal.

---

#### Syntax Rules and Structure

```java
@Override
public void windowClosing(WindowEvent e) {
    int choice = JOptionPane.showConfirmDialog(...);
    if (choice == JOptionPane.YES_OPTION) {
        e.getWindow().dispose();
    }
}
```

#### Constraints and Limitations

- Does not close the window automatically; the listener must call `dispose()` or `setVisible(false)`.
- `windowClosed` fires only after `dispose()`, not after `setVisible(false)`.

---

#### Annotated Complete Step-by-Step Code Examples

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        int choice = JOptionPane.showConfirmDialog(frame,
            "Save before closing?", "Unsaved Changes",
            JOptionPane.YES_NO_CANCEL_OPTION);
        if (choice == JOptionPane.YES_OPTION) {
            saveData();
            frame.dispose();
        } else if (choice == JOptionPane.NO_OPTION) {
            frame.dispose();
        }
        // CANCEL: do nothing
    }
});
```

**Expected Output.** The dialog asks "Save before closing?" with Yes/No/Cancel. Yes saves and closes; No closes without saving; Cancel keeps the window open.

---

### 3.3 windowClosed()

#### Definitions

**Core Definition.** `windowClosed(WindowEvent e)` is invoked just after the window has been disposed.

**Technical Definition.** `public void windowClosed(WindowEvent e)`. Fired by the AWT just after the listened-to window has closed. This event fires only after `dispose()` has been called; it does not fire after `setVisible(false)`.

**Beginner-Friendly Explanation.** `windowClosed` fires after the window has actually been disposed—its resources are released and it can no longer be shown.

---

#### Purposes

- To perform final cleanup after the window is disposed.
- To release resources that were held by the window.
- To log the window's closure.

---

#### Syntax Rules and Structure

```java
@Override
public void windowClosed(WindowEvent e) {
    // cleanup code
}
```

#### Constraints and Limitations

- Does not fire if the window is hidden with `setVisible(false)`.
- The window is no longer usable when this method is called.

---

#### Annotated Complete Step-by-Step Code Examples

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosed(WindowEvent e) {
        System.out.println("Window resources released");
        databaseConnection.close();
    }
});
```

**Expected Output.** When the window is disposed, the console prints "Window resources released" and the database connection is closed.

---

### 3.4 windowIconified()

#### Definitions

**Core Definition.** `windowIconified(WindowEvent e)` is invoked just after the window has been minimised to an icon.

**Technical Definition.** `public void windowIconified(WindowEvent e)`. Fired by the AWT just after the window is iconified.

**Beginner-Friendly Explanation.** `windowIconified` fires when the window is minimised.

---

#### Purposes

- To stop animations and timers when the window is not visible.
- To release resources (buffers, threads) while minimised.
- To log minimise events.

---

#### Syntax Rules and Structure

```java
@Override
public void windowIconified(WindowEvent e) {
    animationTimer.stop();
}
```

#### Constraints and Limitations

- May not fire on platforms that do not support iconification.

---

#### Annotated Complete Step-by-Step Code Examples

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowIconified(WindowEvent e) {
        System.out.println("Window minimised — stopping animation");
        animationTimer.stop();
    }
    @Override
    public void windowDeiconified(WindowEvent e) {
        System.out.println("Window restored — resuming animation");
        animationTimer.start();
    }
});
```

**Expected Output.** Minimising the window stops the animation; restoring it resumes the animation.

---

### 3.5 windowDeiconified()

#### Definitions

**Core Definition.** `windowDeiconified(WindowEvent e)` is invoked just after the window has been restored from a minimised state.

**Technical Definition.** `public void windowDeiconified(WindowEvent e)`. Fired by the AWT just after the window is deiconified.

**Beginner-Friendly Explanation.** `windowDeiconified` fires when the window is restored from being minimised.

---

#### Purposes

- To restart animations and timers that were stopped in `windowIconified`.
- To refresh data when the window becomes visible again.
- To recreate buffers that were released while minimised.

---

#### Syntax Rules and Structure

```java
@Override
public void windowDeiconified(WindowEvent e) {
    animationTimer.start();
}
```

---

#### Annotated Complete Step-by-Step Code Examples

See the combined iconified/deiconified example above.

---

### 3.6 windowActivated()

#### Definitions

**Core Definition.** `windowActivated(WindowEvent e)` is invoked just after the window has been activated (i.e., it or one of its components has gained keyboard focus).

**Technical Definition.** `public void windowActivated(WindowEvent e)`. Fired by the AWT just after the window is activated. A window is activated if it or a component it contains has the keyboard focus.

**Beginner-Friendly Explanation.** `windowActivated` fires when the window gains focus—when it becomes the "active" window.

---

#### Purposes

- To update the window's appearance when it gains focus.
- To resume paused activities when the window becomes active.
- To log focus events for usage analytics.

---

#### Syntax Rules and Structure

```java
@Override
public void windowActivated(WindowEvent e) {
    statusLabel.setText("Active");
}
```

---

#### Annotated Complete Step-by-Step Code Examples

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowActivated(WindowEvent e) {
        statusLabel.setText("Active");
    }
    @Override
    public void windowDeactivated(WindowEvent e) {
        statusLabel.setText("Inactive");
    }
});
```

**Expected Output.** The status label shows "Active" when the window has focus and "Inactive" when it does not.

---

### 3.7 windowDeactivated()

#### Definitions

**Core Definition.** `windowDeactivated(WindowEvent e)` is invoked just after the window has been deactivated (i.e., it and all its components have lost keyboard focus).

**Technical Definition.** `public void windowDeactivated(WindowEvent e)`. Fired by the AWT just after the window is deactivated.

**Beginner-Friendly Explanation.** `windowDeactivated` fires when the window loses focus—when another window becomes active.

---

#### Purposes

- To pause activities when the window loses focus.
- To update the window's appearance to indicate it is inactive.
- To save state when the user switches away.

---

#### Syntax Rules and Structure

```java
@Override
public void windowDeactivated(WindowEvent e) {
    statusLabel.setText("Inactive");
}
```

---

#### Annotated Complete Step-by-Step Code Examples

See the combined activated/deactivated example above.

---

## 4. Window Listener Best Practices

### Definitions

**Core Definition.** Window listener best practices are the conventions and patterns that ensure window lifecycle handling is safe, responsive, and maintainable.

**Technical Definition.** Best practices include: using `WindowAdapter` to override only the methods of interest; setting `DO_NOTHING_ON_CLOSE` when handling `windowClosing` manually; calling `dispose()` (not `System.exit`) to close individual windows; using `AbstractAction` to share close logic between menus and the window listener; stopping resources in `windowIconified` and restarting them in `windowDeiconified`; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices for window listeners are the "rules of thumb" that prevent accidental JVM termination, resource leaks, and inconsistent close behaviour. They include things like "use `WindowAdapter`," "set `DO_NOTHING_ON_CLOSE`," "use `dispose()` not `System.exit`," and "stop and restart resources when minimised and restored."

---

### Purposes

- To avoid boilerplate by using `WindowAdapter`.
- To take full control of the close process with `DO_NOTHING_ON_CLOSE`.
- To avoid terminating the JVM when only one window should close.
- To share close logic between menu items, buttons, and window listeners.
- To manage resources efficiently during iconification and deiconification.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);       // 1. Custom
Action closeAction = new AbstractAction("Exit") {                  // 2. Shared
    @Override
    public void actionPerformed(ActionEvent e) {
        confirmClose();
    }
};
frame.addWindowListener(new WindowAdapter() {                       // 3. Adapter
    @Override
    public void windowClosing(WindowEvent e) {
        closeAction.actionPerformed(null);                          // 4. Delegate
    }
});
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Use `WindowAdapter` | Extend and override only what you need. |
| `DO_NOTHING_ON_CLOSE` for custom close | `setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE)` |
| Use `dispose()` for individual windows | `e.getWindow().dispose()` |
| `System.exit(0)` only for whole application | Use in `CloseAction` when appropriate. |
| Share close logic | `AbstractAction` used by menu, button, and listener. |
| Resource management | Stop in `windowIconified`, restart in `windowDeiconified`. |
| EDT | All window events fire on the EDT. |

#### Syntax Rules

- **Always use `WindowAdapter`** when you only need one or two methods.
- **Always set `DO_NOTHING_ON_CLOSE`** when you handle `windowClosing` manually.
- **Always call `dispose()`** to close an individual window; use `System.exit(0)` only when the entire application should terminate.
- **Always share close logic** via `AbstractAction` when the same close operation is available from menus, buttons, and the window listener.
- **Always stop resources in `windowIconified`** and restart them in `windowDeiconified` to avoid wasted CPU and memory.

#### Constraints and Limitations

- `windowClosed` does not fire if you use `setVisible(false)` instead of `dispose()`.
- `windowClosing` does not close the window automatically; you must call `dispose()` or `setVisible(false)`.
- `System.exit(0)` terminates the entire JVM, including other windows and background threads.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Close with Shared Action

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class BestPracticeWindowListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeWindowListener::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Best Practice WindowListener");
        frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);  // 1. Custom

        Action closeAction = new AbstractAction("Exit") {            // 2. Shared
            @Override
            public void actionPerformed(ActionEvent e) {
                int confirmed = JOptionPane.showConfirmDialog(
                    frame,
                    "Are you sure you want to quit?",
                    "Confirm quit",
                    JOptionPane.YES_NO_OPTION);
                if (confirmed == JOptionPane.YES_OPTION) {
                    // cleanup code
                    frame.dispose();                                  // 3. Dispose
                }
            }
        };

        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        file.add(new JMenuItem(closeAction));
        bar.add(file);
        frame.setJMenuBar(bar);

        frame.addWindowListener(new WindowAdapter() {                // 4. Adapter
            @Override
            public void windowClosing(WindowEvent e) {
                closeAction.actionPerformed(null);                   // 5. Delegate
            }
        });

        frame.setSize(400, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** Clicking the window's close button or choosing "File → Exit" both show the same confirmation dialog. Choosing "Yes" disposes the frame; choosing "No" keeps it open.

**Why the code produces this result.** The `AbstractAction` encapsulates the close logic and is shared between the menu item and the window listener. `DO_NOTHING_ON_CLOSE` ensures the window listener is the only close handler. `dispose()` releases the frame's resources without terminating the JVM.

---

#### Example 2: Common Mistake — Using `System.exit` for a Single Window

```java
import javax.swing.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class BadSystemExit {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame main = new JFrame("Main");
            main.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JFrame secondary = new JFrame("Secondary");
            secondary.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
            secondary.addWindowListener(new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    System.exit(0);                                  // WRONG
                }
            });

            main.setSize(300, 200);
            secondary.setSize(300, 200);
            main.setLocationRelativeTo(null);
            secondary.setLocationRelativeTo(null);
            main.setVisible(true);
            secondary.setVisible(true);
        });
    }
}
```

**Expected Output.** Closing the "Secondary" window terminates the entire JVM, closing the "Main" window as well—even though the user only intended to close the secondary window.

**Why this is wrong.** `System.exit(0)` terminates the JVM, including all windows and background threads. For individual windows, use `dispose()` instead. `System.exit(0)` should be reserved for the main application window or a deliberate "Exit application" command.

---

#### Example 3: Resource Management with Iconification

```java
import javax.swing.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;

public class ResourceManagement {
    private static Timer animationTimer;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Resource Management");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            animationTimer = new Timer(100, e -> {
                // animation tick
            });
            animationTimer.start();

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowIconified(WindowEvent e) {
                    System.out.println("Stopping animation");
                    animationTimer.stop();                           // 1. Stop
                }
                @Override
                public void windowDeiconified(WindowEvent e) {
                    System.out.println("Restarting animation");
                    animationTimer.start();                          // 2. Restart
                }
            });

            frame.setSize(300, 200);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Minimising the window stops the animation timer; restoring the window restarts it. The console prints the corresponding messages.

**Why the code produces this result.** `windowIconified` stops the timer to avoid wasting CPU while the window is minimised. `windowDeiconified` restarts it when the window becomes visible again. This pattern is recommended for animation and other resource-intensive activities.

---

### Real-World Cases

- **Production Swing applications** use `WindowAdapter` with `DO_NOTHING_ON_CLOSE` and a shared `AbstractAction` for consistent close behaviour across menus, buttons, and the window listener.
- **Multi-window applications** use `dispose()` to close individual windows without terminating the JVM.
- **Animation-heavy applications** stop timers in `windowIconified` and restart them in `windowDeiconified` to conserve resources.
- **Educational examples** use the best-practice skeleton to demonstrate correct window lifecycle handling from the start.

---

## References

Oracle. *How to Write a Window Listener*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/events/windowlistener.html

Oracle. *Interface WindowListener*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/WindowListener.html

Oracle. *Class WindowAdapter*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/WindowAdapter.html

Oracle. *Class WindowEvent*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/java/awt/event/WindowEvent.html

Oracle. *How to Make Frames (Main Windows)*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/frame.html

Oracle. *Writing a Window Listener*. https://www.cs.fsu.edu/~jtbauer/cis3931/tutorial/ui/components/windowlistener.html

Oracle. *Examples that Use Window Listeners*. https://www.comp.hkbu.edu.hk/docs/j/tutorial/uiswing/events/windowlistener.html

Stack Overflow. *Keep your JFrame's default close operation to JFrame.DO_NOTHING_ON_CLOSE*. https://stackoverflow.com/revisions/5f633c7b-4016-4cf3-b2eb-77f2c8a255eb/view-source

Stack Overflow. *WindowListener to be called on close*. https://stackoverflow.com/revisions/190b1098-e010-4c30-a31c-2a645a4c00f7/view-source

Oracle. *Class JOptionPane*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/JOptionPane.html

Oracle. *Class AbstractAction*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/AbstractAction.html