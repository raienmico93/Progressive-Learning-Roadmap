# JFrame — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** `JFrame` is an extended version of `java.awt.Frame` that adds support for the JFC/Swing component architecture. It is a top-level window with native decorations—title bar, border, and window-management buttons—that can be minimized, maximized, and closed. It is the primary container for standalone Swing applications.

**Technical Definition.** `public class JFrame extends Frame implements WindowConstants, Accessible, RootPaneContainer`. A `JFrame` contains a `JRootPane` as its only child. The root pane's `contentPane`, which always has a non-null value and uses `BorderLayout` by default, should contain all non-menu components displayed by the frame. The `add`, `remove`, and `setLayout` methods of `JFrame` are overridden to delegate to the content pane. `JFrame` is not thread-safe; all construction and manipulation must occur on the Event Dispatch Thread (EDT) via `SwingUtilities.invokeLater`.

**Beginner-Friendly Explanation.** `JFrame` is the main window of a desktop application—the rectangular frame on your screen with a title bar, a minimize button, a maximize button, and a close button. You do not place buttons and labels directly on the frame itself; instead, you place them on the frame's **content pane**, which is the visible canvas inside the window's decorative border. Think of the `JFrame` as the physical picture frame and the content pane as the canvas where you paint your GUI.

---

### Key Characteristics

1. **Top-level container.** `JFrame` is one of four Swing top-level containers (alongside `JDialog`, `JWindow`, and the deprecated `JApplet`). Every Swing application that runs in a window needs at least one `JFrame`.
2. **Root pane architecture.** Every `JFrame` owns a `JRootPane` that manages the content pane, glass pane, layered pane, and optional menu bar.
3. **Heavyweight nature.** Unlike lightweight Swing components, `JFrame` depends on a native platform window for its existence and decorations.
4. **Explicit close semantics.** The default close operation is `HIDE_ON_CLOSE`, which hides the frame but does not terminate the JVM. Programmers must explicitly call `setDefaultCloseOperation` to define the desired behavior.
5. **Event-driven lifecycle.** Window-related events (opening, closing, iconifying, activating) are delivered to registered `WindowListener` objects on the EDT.
6. **Single-thread rule.** `JFrame` and all its components must be accessed on the EDT. GUI creation code placed on the main thread can cause race conditions and deadlocks.

---

### Prerequisites

- Java language fundamentals: classes, inheritance, anonymous inner classes, and lambda expressions.
- The `public static void main(String[] args)` entry-point signature.
- Basic understanding of the AWT container hierarchy (`Container`, `Window`, `Frame`).
- Familiarity with `SwingUtilities.invokeLater` and the Event Dispatch Thread.
- A JDK (version 8 or later; `JFrame` is available in all modern JDKs).

---

### Related Programming Areas

- **AWT (Abstract Window Toolkit):** The parent framework; `JFrame` extends `java.awt.Frame`.
- **Java 2D:** Used for painting lightweight components inside the frame.
- **Event handling:** `WindowListener`, `WindowAdapter`, and `WindowEvent` from `java.awt.event`.
- **Concurrency:** The EDT and `SwingWorker` for keeping the GUI responsive.
- **Layout management:** Layout managers arrange components inside the content pane.

---

## Core Concepts and Features

---

## 1. JFrame — Frame Creation

### Definitions

**Core Definition.** Frame creation is the process of instantiating a `JFrame` object, configuring its essential properties, and making it visible on the screen.

**Technical Definition.** A `JFrame` is created via one of its constructors: `JFrame()`, `JFrame(String title)`, `JFrame(GraphicsConfiguration gc)`, or `JFrame(String title, GraphicsConfiguration gc)`. The no-argument constructor creates a frame with an empty title. After construction, the frame is not visible until `setVisible(true)` is called. Creation must occur on the EDT.

**Beginner-Friendly Explanation.** Creating a frame means typing `new JFrame("My App")` to build the window in memory. At this point, the window exists but is invisible. You then configure it—giving it a size, a position, and close behavior—and finally call `setVisible(true)` to make it appear on the screen.

---

### Purposes

- To instantiate the top-level window that will host the application's GUI.
- To provide a container whose content pane can hold all visible Swing components.
- To establish the root of a containment hierarchy for a standalone desktop application.
- To serve as the owner/parent for dependent windows such as `JDialog` and `JWindow`.
- To anchor layout managers that arrange the application's interface.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JFrame frame = new JFrame("Application Title");
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setSize(500, 400);
frame.setLocationRelativeTo(null);
frame.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `new JFrame("Title")` | Creates a frame with the specified title bar text. |
| `setDefaultCloseOperation(int)` | Defines the action taken when the close button is clicked. |
| `setSize(int, int)` | Sets the frame's width and height in pixels. |
| `setLocationRelativeTo(Component)` | Centers the frame on the screen (`null`) or relative to another component. |
| `setVisible(true)` | Makes the frame visible. Must be the final call after configuration. |

#### Syntax Rules

- The `JFrame` constructor must be called before any other frame method.
- `setVisible(true)` should be the last operation after all components have been added and configured.
- Frame creation must occur on the EDT via `SwingUtilities.invokeLater`.
- The default close operation is `HIDE_ON_CLOSE`; it should be changed explicitly.
- A `JFrame` cannot be added to another container.

#### Constraints and Limitations

- Constructing a `JFrame` on the main thread violates Swing's threading policy and can cause unpredictable behavior.
- A `JFrame` created but never made visible will not appear and may not prevent the JVM from exiting.
- The frame's content pane is never `null`; attempting to set it to `null` throws an exception.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Frame Creation

```java
import javax.swing.JFrame;                     // 1. Import JFrame
import javax.swing.SwingUtilities;             // 2. Import invokeLater

public class MinimalCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {     // 3. Schedule on the EDT
            JFrame frame = new JFrame("Minimal App"); // 4. Create frame
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // 5
            frame.setSize(400, 300);            // 6. 400 x 300 pixels
            frame.setLocationRelativeTo(null);  // 7. Center on screen
            frame.setVisible(true);             // 8. Show the window
        });
    }
}
```

**Expected Output.** A 400×300 window titled "Minimal App" appears in the center of the screen. Clicking the close button terminates the JVM.

**Why the code produces this result.** `SwingUtilities.invokeLater` schedules the lambda on the EDT. The EDT constructs the `JFrame`, sets its close operation, size, and location, and calls `setVisible(true)`. Because `EXIT_ON_CLOSE` is set, the JVM exits when the window is closed.

---

#### Example 2: Frame Creation with a Helper Method

```java
import javax.swing.*;

public class HelperCreation {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(HelperCreation::createAndShow);
    }

    private static void createAndShow() {
        JFrame frame = new JFrame("Helper Method App");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(350, 250);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A 350×250 window titled "Helper Method App" appears. The `main` method delegates GUI construction to a dedicated helper method, which runs on the EDT.

**Why the code produces this result.** The method reference `HelperCreation::createAndShow` is a `Runnable` that `invokeLater` schedules on the EDT. Separating GUI creation into a helper method keeps `main` minimal and makes the code easier to read and maintain.

---

### Real-World Cases

- **Every standalone Swing application** begins with frame creation in its `main` method.
- **Launcher classes** that parse command-line arguments and then create the main window.
- **Development harnesses** that create a frame to display a single panel or dialog for testing.

---

## 2. Frame Properties — Title

### Definitions

**Core Definition.** The title is the text displayed in the frame's title bar. It is a property of `JFrame` that can be set at construction time or modified later.

**Technical Definition.** The title is managed by the inherited `Frame.setTitle(String)` method. It is a `String` property that may be `null` (resulting in an empty title bar). The title is displayed by the native window manager as part of the frame's decorations.

**Beginner-Friendly Explanation.** The title is the name shown at the top of the window—the text that appears in the title bar. It tells the user what the application or window is.

---

### Purposes

- To identify the application or document displayed in the window.
- To provide a human-readable label in the title bar and task bar.
- To allow dynamic updates (e.g., showing the current file name in an editor).
- To support accessibility tools that read the window title.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setTitle("New Title");
// or at construction:
JFrame frame = new JFrame("Initial Title");
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setTitle(String)` | Sets the title bar text. Passing `null` clears the title. |
| `new JFrame(String)` | Constructor that sets the initial title. |

#### Syntax Rules

- The title can be changed at any time after construction.
- `null` is a valid title value but results in an empty title bar.
- Title changes should occur on the EDT.

#### Constraints and Limitations

- The title is displayed by the native window manager; its appearance (font, truncation) is platform-dependent.
- Very long titles may be truncated in the title bar or task bar.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting and Updating the Title

```java
import javax.swing.*;
import java.awt.*;

public class TitleDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Initial Title"); // 1. Initial title
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);

            JButton button = new JButton("Change Title");
            button.addActionListener(e ->
                frame.setTitle("Updated Title"));       // 2. Dynamic update

            frame.add(button);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window appears with the title "Initial Title." Clicking the button changes the title to "Updated Title."

**Why the code produces this result.** The `JFrame` constructor sets the initial title. The `ActionListener` calls `setTitle` on the EDT, and the native window manager updates the title bar accordingly.

---

### Real-World Cases

- **Text editors** display the current file name in the title bar and update it when the file is saved or renamed.
- **Web browsers** show the page title and the browser name.
- **Database applications** show the connected database name in the title.

---

## 3. Frame Properties — Size

### Definitions

**Core Definition.** Size refers to the width and height of the frame in pixels, either as explicitly set dimensions or as the preferred size computed from the frame's components.

**Technical Definition.** The frame's size is managed by `setSize(int width, int height)` (inherited from `java.awt.Window`) and `pack()`. `setSize` sets the outer dimensions of the frame, including decorations. `pack()` resizes the frame so that its content pane is at its preferred size.

**Beginner-Friendly Explanation.** Size is how big the window is—how many pixels wide and tall. You can either tell the window exactly how big to be with `setSize`, or you can let it figure out the right size based on what is inside it using `pack()`.

---

### Purposes

- To control the visible dimensions of the application window.
- To ensure the window is large enough to display its contents comfortably.
- To adapt the window to different screen resolutions.
- To allow the user to resize the window if `setResizable(true)` is set.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setSize(600, 400);   // explicit size
// or
frame.pack();               // size to fit preferred sizes
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setSize(int, int)` | Sets the frame's width and height in pixels, including decorations. |
| `pack()` | Sizes the frame so that its content pane is at its preferred size. |
| `setPreferredSize(Dimension)` | Sets the preferred size of the content pane or components inside it. |

#### Syntax Rules

- `setSize` overrides any size previously set by `pack()`.
- `pack()` should be called after all components have been added to the content pane.
- If `setSize` is used, the frame does not automatically adjust to its content.
- The actual visible area inside the frame is smaller than the set size because decorations consume space.

#### Constraints and Limitations

- `setSize` sets the outer dimensions; the content area is smaller by the size of the window borders and title bar.
- `pack()` may produce a very small or very large window if preferred sizes are not configured correctly.
- On multi-monitor setups, `setSize` may produce a window larger than a single screen.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Explicit Size vs. Pack

```java
import javax.swing.*;
import java.awt.*;

public class SizeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // Frame with explicit size
            JFrame sizedFrame = new JFrame("SetSize Demo");
            sizedFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
            sizedFrame.setSize(400, 300); // 1. Explicit 400 x 300
            sizedFrame.setLocationRelativeTo(null);
            sizedFrame.setVisible(true);

            // Frame sized with pack()
            JFrame packedFrame = new JFrame("Pack Demo");
            packedFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
            packedFrame.add(new JLabel("This label determines the size."));
            packedFrame.pack(); // 2. Size to fit content
            packedFrame.setLocationRelativeTo(null);
            packedFrame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two windows appear. The first is exactly 400×300 pixels. The second is just large enough to contain the label and its decorations.

**Why the code produces this result.** `setSize` forces the outer dimensions to 400×300. `pack()` queries the preferred sizes of the content pane and its children and sets the frame's size accordingly.

---

### Real-World Cases

- **Data entry forms** use `pack()` so the window fits all fields and labels.
- **Image viewers** use `setSize` to match the image dimensions.
- **Responsive applications** compute `setSize` based on screen resolution.

---

## 4. Frame Properties — Location

### Definitions

**Core Definition.** Location refers to the position of the frame's top-left corner on the screen, expressed as (x, y) coordinates in pixels.

**Technical Definition.** The location is managed by `setLocation(int x, int y)` and `setLocationRelativeTo(Component c)`, both inherited from `java.awt.Window`. `setLocationRelativeTo` centers the window relative to the specified component or, if `null`, relative to the screen.

**Beginner-Friendly Explanation.** Location is where the window sits on your screen. You can place it at exact coordinates, or you can tell it to center itself on the screen or over another window.

---

### Purposes

- To position the window at a specific screen coordinate.
- To center the window on the screen for a balanced appearance.
- To align a dialog or secondary window relative to its owner.
- To restore a window to a previously saved position.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setLocation(100, 200);           // absolute position
// or
frame.setLocationRelativeTo(null);     // center on screen
// or
frame.setLocationRelativeTo(owner);    // center over owner
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setLocation(int x, int y)` | Places the frame's top-left corner at the given screen coordinates. |
| `setLocationRelativeTo(Component)` | Centers the frame relative to the given component; `null` centers on the screen. |

#### Syntax Rules

- `setLocationRelativeTo` should be called after the frame's size has been set or `pack()` has been called.
- Absolute coordinates are measured from the top-left corner of the primary screen.
- On multi-monitor setups, coordinates may span multiple screens.

#### Constraints and Limitations

- Absolute positioning is fragile across different screen resolutions and monitor configurations.
- `setLocationRelativeTo(null)` centers on the primary screen; centering on a secondary screen requires additional logic.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Centering a Frame

```java
import javax.swing.*;

public class LocationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Centered Frame");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(500, 300);
            frame.setLocationRelativeTo(null); // 1. Center on screen
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 500×300 window appears in the exact center of the screen.

**Why the code produces this result.** `setLocationRelativeTo(null)` computes the screen dimensions and the frame dimensions and positions the frame so that its center coincides with the screen's center.

---

### Real-World Cases

- **Splash screens** are centered on the screen.
- **Dialogs** are centered over their parent frames.
- **Application windows** remember their last position and restore it on next launch.

---

## 5. Frame Properties — Resizable

### Definitions

**Core Definition.** Resizable is a boolean property that determines whether the user can change the frame's size by dragging its borders or corners.

**Technical Definition.** The resizable state is controlled by `setResizable(boolean resizable)`, inherited from `java.awt.Frame`. By default, frames are resizable. When set to `false`, the native window manager prevents the user from resizing the window.

**Beginner-Friendly Explanation.** Resizable means whether the user can stretch or shrink the window by dragging its edges. If it is resizable, the user can make the window bigger or smaller. If not, the window stays at its fixed size.

---

### Purposes

- To allow the user to adjust the window size to their preference.
- To lock the window size for dialogs or splash screens that should not be resized.
- To prevent layout problems when the GUI is designed for a fixed size.
- To support responsive layouts that adapt to different sizes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setResizable(false); // prevent resizing
// or
frame.setResizable(true);  // allow resizing (default)
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setResizable(boolean)` | Enables or disables user resizing of the frame. |

#### Syntax Rules

- Must be called before `setVisible(true)` for the setting to take effect reliably.
- The default value is `true` for `JFrame`.
- Even when `setResizable(false)`, the programmer can still call `setSize` programmatically.

#### Constraints and Limitations

- On some platforms, the native window manager may ignore or partially honor the resizable setting.
- A non-resizable frame with content larger than its size may clip the content.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Non-Resizable Frame

```java
import javax.swing.*;

public class FixedSizeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Fixed Size");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(350, 200);
            frame.setResizable(false); // 1. Lock the size
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 350×200 window appears. The user cannot resize it by dragging its borders; the resize cursor does not appear.

**Why the code produces this result.** `setResizable(false)` instructs the native window manager to remove resize handles and prevent border-drag resizing.

---

### Real-World Cases

- **Splash screens** are typically non-resizable.
- **Fixed-size utility windows** (e.g., a calculator) use `setResizable(false)`.
- **Dialogs with rigid layouts** prevent resizing to avoid component overlap.

---

## 6. Frame Properties — Undecorated

### Definitions

**Core Definition.** Undecorated is a boolean property that removes the frame's native decorations—title bar, borders, and window-management buttons—from the window.

**Technical Definition.** `setUndecorated(boolean undecorated)` is inherited from `java.awt.Frame`. This method can only be called while the frame is not displayable (i.e., before `pack()` or `setVisible(true)` has been called). An undecorated frame has no native title bar or resize handles.

**Beginner-Friendly Explanation.** Undecorated means the window has no frame around it—no title bar, no close button, no borders. It is just a bare rectangle on the screen. You can use this for splash screens or custom-shaped windows.

---

### Purposes

- To create splash screens and splash-style windows without title bars.
- To implement custom window decorations drawn by the application itself.
- To create shaped or transparent windows with custom chrome.
- To eliminate native window borders for a seamless full-screen experience.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JFrame frame = new JFrame();
frame.setUndecorated(true); // must be before pack() or setVisible()
frame.setSize(400, 300);
frame.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setUndecorated(boolean)` | Removes or restores native decorations. Must be called before the frame is displayable. |

#### Syntax Rules

- `setUndecorated` must be called **before** `pack()` or `setVisible(true)`; otherwise, it throws `IllegalComponentStateException`.
- An undecorated frame cannot be moved or resized by the user through native decorations.
- The programmer must provide alternative mechanisms for closing or moving the window.

#### Constraints and Limitations

- Once the frame is displayable, `setUndecorated` cannot be called.
- Undecorated frames do not appear in the task bar on some platforms.
- Accessibility and keyboard navigation may require additional configuration.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Undecorated Splash-Style Frame

```java
import javax.swing.*;
import java.awt.*;

public class UndecoratedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame();
            frame.setUndecorated(true); // 1. Remove decorations
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);

            JPanel panel = new JPanel(new BorderLayout());
            panel.setBackground(Color.DARK_GRAY);
            JLabel label = new JLabel("Splash Screen", SwingConstants.CENTER);
            label.setForeground(Color.WHITE);
            label.setFont(new Font("SansSerif", Font.BOLD, 24));
            panel.add(label, BorderLayout.CENTER);

            frame.setContentPane(panel);
            frame.setVisible(true); // 2. Show undecorated window

            // Auto-close after 2 seconds
            new Timer(2000, e -> frame.dispose()).start();
        });
    }
}
```

**Expected Output.** A dark-gray rectangle with the white text "Splash Screen" appears in the center of the screen with no title bar or borders. After two seconds, it disappears.

**Why the code produces this result.** `setUndecorated(true)` is called before `setVisible(true)`, removing all native decorations. The content pane fills the entire window area. The `Timer` disposes the frame after 2000 ms.

---

### Real-World Cases

- **Splash screens** that display a logo during application startup.
- **Custom-shaped windows** that use transparency and non-rectangular shapes.
- **Full-screen media players** that hide all chrome.

---

## 7. Frame Properties — Always on Top

### Definitions

**Core Definition.** Always-on-top is a boolean property that, when enabled, keeps the frame above all other windows in the window manager's z-order.

**Technical Definition.** `setAlwaysOnTop(boolean alwaysOnTop)` is inherited from `java.awt.Window`. When set to `true`, the window manager attempts to keep the frame above other windows, even when it is not focused. Support for this property is platform-dependent.

**Beginner-Friendly Explanation.** Always-on-top means the window stays visible above other windows. It floats on top of everything else on your screen, like a sticky note that refuses to be covered.

---

### Purposes

- To keep important notifications or status windows visible above other applications.
- To implement floating tool palettes that should not be obscured.
- To display media player controls that remain accessible while other work is done.
- To create “heads-up” displays for monitoring or alerts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setAlwaysOnTop(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setAlwaysOnTop(boolean)` | Enables or disables the always-on-top behavior. |

#### Syntax Rules

- Can be called before or after the frame is visible.
- Support depends on the platform's window manager.
- Some platforms may require additional permissions or security manager approval.

#### Constraints and Limitations

- Not all platforms support always-on-top; the property may be silently ignored.
- Always-on-top windows can be disruptive to the user experience and should be used sparingly.
- On some systems, other always-on-top windows may still appear above the frame.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Always-on-Top Status Window

```java
import javax.swing.*;
import java.awt.*;

public class AlwaysOnTopDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Always on Top");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            frame.setAlwaysOnTop(true); // 1. Float above others
            frame.setLocationRelativeTo(null);

            frame.add(new JLabel("This window stays on top.",
                SwingConstants.CENTER));
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 300×150 window appears. When another application window is clicked and brought to the foreground, the always-on-top frame remains visible above it.

**Why the code produces this result.** `setAlwaysOnTop(true)` requests the native window manager to keep the frame in the topmost z-order layer. The behavior is platform-dependent but generally works on modern desktop systems.

---

### Real-World Cases

- **Video conferencing controls** that remain visible during a call.
- **System monitors** that display CPU or memory usage above other windows.
- **Reminder pop-ups** that should not be missed.

---

## 8. Frame Methods — setTitle()

### Definitions

**Core Definition.** `setTitle(String title)` sets the text displayed in the frame's title bar.

**Technical Definition.** `public void setTitle(String title)` is inherited from `java.awt.Frame`. It updates the native window title and may trigger a property change event (`title` property).

**Beginner-Friendly Explanation.** `setTitle` changes the name shown at the top of the window.

---

### Purposes

- To set or change the window's title after construction.
- To reflect dynamic state (e.g., current file name, connection status).
- To support internationalization by updating the title based on locale.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setTitle("New Title");
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setTitle(String)` | Sets the title bar text. `null` clears the title. |

#### Syntax Rules

- Can be called at any time after construction.
- Should be called on the EDT.

#### Constraints and Limitations

- The native window manager controls the title's font and truncation.
- Very long titles may be truncated.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Dynamic Title Update

```java
import javax.swing.*;

public class SetTitleDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Untitled");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);

            JButton saveButton = new JButton("Save");
            saveButton.addActionListener(e ->
                frame.setTitle("document.txt - Saved")); // 1. Update title

            frame.add(saveButton);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The window initially shows "Untitled." After clicking "Save," the title changes to "document.txt - Saved."

**Why the code produces this result.** The `ActionListener` calls `setTitle` on the EDT. The native window manager repaints the title bar with the new text.

---

### Real-World Cases

- **Text editors** update the title when a file is saved.
- **Browsers** update the title when a page loads.
- **Chat applications** update the title when a new message arrives.

---

## 9. Frame Methods — setSize()

### Definitions

**Core Definition.** `setSize(int width, int height)` sets the frame's width and height in pixels.

**Technical Definition.** `public void setSize(int width, int height)` is inherited from `java.awt.Window`. It sets the outer dimensions of the frame, including native decorations. The method fires a `componentResized` event.

**Beginner-Friendly Explanation.** `setSize` tells the window how big to be.

---

### Purposes

- To explicitly control the window's dimensions.
- To restore a window to a saved size.
- To adapt the window to the screen resolution.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setSize(600, 400);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setSize(int, int)` | Sets width and height in pixels. |

#### Syntax Rules

- The dimensions include window decorations.
- `setSize` overrides any size previously set by `pack()`.
- Should be called before `setVisible(true)` for reliable results.

#### Constraints and Limitations

- The content area is smaller than the set size due to decorations.
- Does not respect preferred sizes of components.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Explicit Sizing

```java
import javax.swing.*;

public class SetSizeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetSize Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(800, 600); // 1. 800 x 600 pixels
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** An 800×600 window appears centered on the screen.

**Why the code produces this result.** `setSize(800, 600)` sets the outer frame dimensions. The native window manager creates a window of exactly that size.

---

### Real-World Cases

- **Image viewers** set the window size to match the image.
- **Games** set a fixed window size for consistent gameplay.
- **Applications** restore a saved window size from preferences.

---

## 10. Frame Methods — setLocation()

### Definitions

**Core Definition.** `setLocation(int x, int y)` positions the frame's top-left corner at the specified screen coordinates.

**Technical Definition.** `public void setLocation(int x, int y)` is inherited from `java.awt.Window`. The coordinates are in pixels relative to the top-left corner of the virtual screen (which may span multiple monitors).

**Beginner-Friendly Explanation.** `setLocation` tells the window exactly where to appear on the screen.

---

### Purposes

- To place the window at a precise screen position.
- To restore a window to a saved location.
- To align multiple windows in a specific arrangement.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setLocation(100, 200);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setLocation(int x, int y)` | Sets the top-left corner coordinates. |

#### Syntax Rules

- Coordinates are measured from the top-left of the primary screen.
- Should be called after setting the size.

#### Constraints and Limitations

- Absolute positioning is not portable across different screen resolutions.
- Negative coordinates may place the window partially off-screen.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Positioning at Specific Coordinates

```java
import javax.swing.*;

public class SetLocationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Location Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 300);
            frame.setLocation(150, 100); // 1. Top-left at (150, 100)
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 400×300 window appears with its top-left corner at screen coordinates (150, 100).

**Why the code produces this result.** `setLocation(150, 100)` instructs the window manager to place the frame's top-left corner at pixel (150, 100) from the screen's top-left corner.

---

### Real-World Cases

- **Multi-window layouts** position windows at specific screen regions.
- **Restoring saved positions** from user preferences.
- **Testing** places windows at known coordinates for automation.

---

## 11. Frame Methods — setBounds()

### Definitions

**Core Definition.** `setBounds(int x, int y, int width, int height)` sets both the location and size of the frame in a single call.

**Technical Definition.** `public void setBounds(int x, int y, int width, int height)` is inherited from `java.awt.Window`. It combines `setLocation` and `setSize` into one operation.

**Beginner-Friendly Explanation.** `setBounds` is a shortcut that sets both where the window is and how big it is at the same time.

---

### Purposes

- To set location and size together without separate method calls.
- To restore a window's geometry from saved state.
- To position and size a window in a single, atomic operation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setBounds(100, 50, 800, 600);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setBounds(int x, int y, int w, int h)` | Sets top-left corner at (x, y) and dimensions to w × h. |

#### Syntax Rules

- The four arguments are (x, y, width, height).
- Can be called at any time after construction.

#### Constraints and Limitations

- Same portability concerns as `setLocation` and `setSize`.
- Does not respect preferred sizes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting Geometry in One Call

```java
import javax.swing.*;

public class SetBoundsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("SetBounds Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setBounds(200, 100, 600, 400); // 1. x, y, w, h
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A 600×400 window appears with its top-left corner at (200, 100).

**Why the code produces this result.** `setBounds` applies both the location and size in one operation.

---

### Real-World Cases

- **Window state persistence** stores the bounds rectangle and restores it.
- **Programmatic window arrangement** positions and sizes windows in one call.
- **Animation** moves and resizes windows smoothly.

---

## 12. Frame Methods — setResizable()

### Definitions

**Core Definition.** `setResizable(boolean resizable)` controls whether the user can resize the frame by dragging its borders.

**Technical Definition.** `public void setResizable(boolean resizable)` is inherited from `java.awt.Frame`. When `false`, the native window manager removes resize handles.

**Beginner-Friendly Explanation.** `setResizable` locks or unlocks the window's size for the user.

---

### Purposes

- To lock the window size for dialogs or splash screens.
- To allow or prevent user resizing based on application state.
- To simplify layout management by enforcing a fixed size.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setResizable(false);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setResizable(boolean)` | `false` locks the size; `true` allows resizing. |

#### Syntax Rules

- Default is `true`.
- Call before `setVisible(true)` for reliable behavior.

#### Constraints and Limitations

- Platform-dependent behavior.
- Content may be clipped if the frame is too small.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Toggling Resizability

```java
import javax.swing.*;

public class ToggleResizable {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Resizable Toggle");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 300);
            frame.setResizable(false); // 1. Start locked
            frame.setLocationRelativeTo(null);

            JButton toggle = new JButton("Toggle Resizable");
            toggle.addActionListener(e ->
                frame.setResizable(!frame.isResizable())); // 2. Toggle

            frame.add(toggle);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The window starts non-resizable. Clicking the button toggles the resizable state, allowing or preventing the user from dragging the borders.

**Why the code produces this result.** `setResizable(!frame.isResizable())` inverts the current state. The native window manager updates the resize handles accordingly.

---

### Real-World Cases

- **Dialogs** that should not be resized.
- **Full-screen applications** that lock the window size.
- **Games** that require a fixed viewport.

---

## 13. Frame Methods — setDefaultCloseOperation()

### Definitions

**Core Definition.** `setDefaultCloseOperation(int operation)` defines what happens when the user clicks the frame's close button. The operation is specified by one of four constants: `DO_NOTHING_ON_CLOSE`, `HIDE_ON_CLOSE`, `DISPOSE_ON_CLOSE`, or `EXIT_ON_CLOSE`.

**Technical Definition.** `public void setDefaultCloseOperation(int operation)` is part of the `WindowConstants` interface. The default value is `HIDE_ON_CLOSE`. `EXIT_ON_CLOSE` is recommended for applications, while `DISPOSE_ON_CLOSE` releases resources without terminating the JVM.

**Beginner-Friendly Explanation.** `setDefaultCloseOperation` tells the window what to do when the user clicks the X button. Should the window hide? Should it close and release memory? Should the whole program quit?

---

### Purposes

- To define the application's termination behavior.
- To prevent accidental data loss by intercepting the close event.
- To release native resources when a window is no longer needed.
- To distinguish between hiding a window and quitting the application.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `DO_NOTHING_ON_CLOSE` | The window remains open; the programmer must handle the close event. |
| `HIDE_ON_CLOSE` | The window is hidden but not disposed (default). |
| `DISPOSE_ON_CLOSE` | The window is disposed, releasing native resources. |
| `EXIT_ON_CLOSE` | The JVM is terminated via `System.exit(0)`. |

#### Syntax Rules

- Should be called explicitly; the default is `HIDE_ON_CLOSE`.
- `EXIT_ON_CLOSE` should be used only for the main application frame.
- `DO_NOTHING_ON_CLOSE` requires a `WindowListener` to handle `windowClosing`.

#### Constraints and Limitations

- `EXIT_ON_CLOSE` terminates the JVM, including other windows and background threads.
- `DISPOSE_ON_CLOSE` may allow the JVM to exit if no other non-daemon threads exist.
- `HIDE_ON_CLOSE` can leave the JVM running invisibly.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Close Operations

```java
import javax.swing.*;

public class CloseOperationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // EXIT_ON_CLOSE
            JFrame exitFrame = new JFrame("EXIT_ON_CLOSE");
            exitFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            exitFrame.setSize(250, 150);
            exitFrame.setLocationRelativeTo(null);
            exitFrame.setVisible(true);

            // DISPOSE_ON_CLOSE
            JFrame disposeFrame = new JFrame("DISPOSE_ON_CLOSE");
            disposeFrame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
            disposeFrame.setSize(250, 150);
            disposeFrame.setLocationRelativeTo(null);
            disposeFrame.setVisible(true);
        });
    }
}
```

**Expected Output.** Two windows appear. Closing the "EXIT_ON_CLOSE" window terminates the entire JVM, closing both windows. Closing the "DISPOSE_ON_CLOSE" window disposes only that window; the other window remains open.

**Why the code produces this result.** `EXIT_ON_CLOSE` calls `System.exit(0)`, terminating all threads. `DISPOSE_ON_CLOSE` calls `dispose()` on the frame, releasing its resources without affecting other windows.

---

### Real-World Cases

- **Main application windows** use `EXIT_ON_CLOSE`.
- **Secondary windows and dialogs** use `DISPOSE_ON_CLOSE`.
- **Windows with unsaved data** use `DO_NOTHING_ON_CLOSE` and a listener to prompt the user.

---

## 14. Frame Methods — setVisible()

### Definitions

**Core Definition.** `setVisible(boolean visible)` makes the frame visible or invisible on the screen.

**Technical Definition.** `public void setVisible(boolean visible)` is inherited from `java.awt.Window`. When set to `true`, the frame is displayed and its native peer is created (if not already created). When set to `false`, the frame is hidden.

**Beginner-Friendly Explanation.** `setVisible` is the “on/off switch” for the window. Before this is called with `true`, the window exists in memory but is not shown.

---

### Purposes

- To display the frame after it has been fully configured.
- To hide the frame without disposing it.
- To control the frame's visibility programmatically.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setVisible(boolean)` | `true` shows the frame; `false` hides it. |

#### Syntax Rules

- Should be the last call after all configuration.
- Can be called multiple times to toggle visibility.

#### Constraints and Limitations

- Calling `setVisible(true)` on a frame with no components shows an empty window.
- Hiding a frame with `setVisible(false)` does not dispose it; resources remain allocated.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Toggling Visibility

```java
import javax.swing.*;

public class VisibilityDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Visibility Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);

            JButton hideButton = new JButton("Hide");
            hideButton.addActionListener(e ->
                frame.setVisible(false)); // 1. Hide

            frame.add(hideButton);
            frame.setVisible(true); // 2. Show initially
        });
    }
}
```

**Expected Output.** A window appears with a "Hide" button. Clicking the button hides the window (but the JVM continues running). The window can be shown again programmatically.

**Why the code produces this result.** `setVisible(true)` displays the frame. The `ActionListener` calls `setVisible(false)`, which hides the frame without disposing it.

---

### Real-World Cases

- **Splash screens** become invisible when the main window appears.
- **Wizard dialogs** hide and show panels based on the current step.
- **Parental controls** temporarily hide windows when a timeout occurs.

---

## 15. Frame Methods — pack()

### Definitions

**Core Definition.** `pack()` resizes the frame so that its content pane is at its preferred size, taking into account the preferred sizes and layouts of all its children.

**Technical Definition.** `public void pack()` is inherited from `java.awt.Window`. It calls `validate()` and then sets the frame's size to the preferred size of the content pane plus the frame's insets (decorations).

**Beginner-Friendly Explanation.** `pack()` is like shrink-wrapping the window around its contents. It makes the window exactly big enough to fit everything inside it comfortably.

---

### Purposes

- To automatically size the window to fit its contents.
- To avoid manual size calculations.
- To ensure that all components are fully visible.
- To create compact, well-fitted windows.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.pack();
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `pack()` | Sizes the frame to fit the preferred sizes of its contents. |

#### Syntax Rules

- Must be called after all components have been added.
- Overrides any previously set size.
- Should be called before `setVisible(true)`.

#### Constraints and Limitations

- May produce a very small window if components have small preferred sizes.
- Does not account for dynamic content changes after packing.
- On some platforms, the calculated size may be slightly off due to insets.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Packing a Frame with Components

```java
import javax.swing.*;
import java.awt.*;

public class PackDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Pack Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JPanel panel = new JPanel(new GridLayout(3, 2, 5, 5));
            panel.add(new JLabel("Name:"));
            panel.add(new JTextField(15));
            panel.add(new JLabel("Email:"));
            panel.add(new JTextField(15));
            panel.add(new JLabel("Phone:"));
            panel.add(new JTextField(15));

            frame.add(panel);
            frame.pack(); // 1. Size to fit contents
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A compact window appears with a 3×2 grid of labels and text fields. The window is sized exactly to fit the grid and its decorations.

**Why the code produces this result.** `pack()` queries the preferred sizes of the panel and its children, computes the required frame size, and applies it. The window is neither too small nor too large for its content.

---

### Real-World Cases

- **Forms** with fixed-size fields use `pack()` to fit everything neatly.
- **About dialogs** that display version information use `pack()`.
- **Tool windows** that contain a small number of controls use `pack()`.

---

## 16. Frame Methods — dispose()

### Definitions

**Core Definition.** `dispose()` releases the native screen resources, native window, and any child components of the frame, making it no longer displayable.

**Technical Definition.** `public void dispose()` is inherited from `java.awt.Window`. After disposal, the frame and its children cannot be made displayable again without re-creating them. `DISPOSE_ON_CLOSE` calls `dispose()` automatically when the close button is clicked.

**Beginner-Friendly Explanation.** `dispose()` is like throwing away the window and all its resources. The window disappears from the screen, and the memory it used is freed.

---

### Purposes

- To release native resources when a window is no longer needed.
- To prevent memory leaks in applications that create and destroy multiple windows.
- To implement the `DISPOSE_ON_CLOSE` behavior.
- To clean up before opening a new window.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.dispose();
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `dispose()` | Releases native resources and makes the frame undisplayable. |

#### Syntax Rules

- Once disposed, the frame cannot be made visible again.
- Calling `dispose()` on an already disposed frame has no effect.
- Should be called on the EDT.

#### Constraints and Limitations

- Disposing the last displayable window may cause the JVM to exit.
- Resources held by child components may not be immediately released.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Disposing a Splash Screen

```java
import javax.swing.*;
import java.awt.*;

public class DisposeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JWindow splash = new JWindow();
            splash.setSize(300, 150);
            splash.setLocationRelativeTo(null);
            splash.add(new JLabel("Loading...", SwingConstants.CENTER));
            splash.setVisible(true);

            new Timer(2000, e -> {
                splash.dispose(); // 1. Release splash resources
                showMainFrame();
            }).start();
        });
    }

    private static void showMainFrame() {
        JFrame frame = new JFrame("Main App");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(500, 350);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A splash window appears for two seconds, then is disposed. The main application window appears immediately afterward.

**Why the code produces this result.** `dispose()` releases the splash window's native resources. The main frame is then created and shown on the EDT.

---

### Real-World Cases

- **Splash screens** are disposed after the main window loads.
- **Progress dialogs** are disposed when the task completes.
- **Multi-window applications** dispose secondary windows when they are closed.

---

## 17. Frame Methods — setIconImage()

### Definitions

**Core Definition.** `setIconImage(Image image)` sets the image displayed as the frame's icon in the title bar, task bar, or window switcher.

**Technical Definition.** `public void setIconImage(Image image)` is inherited from `java.awt.Frame`. The image is displayed in the native window decorations and may be scaled by the platform.

**Beginner-Friendly Explanation.** `setIconImage` changes the little picture that appears next to the window's title or in the task bar. It is the application's “face” in the operating system.

---

### Purposes

- To provide a visual identity for the application.
- To make the application easily recognizable in the task bar.
- To replace the default Java coffee-cup icon.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
Image icon = Toolkit.getDefaultToolkit().getImage("icon.png");
frame.setIconImage(icon);
// or with ImageIcon:
frame.setIconImage(new ImageIcon("icon.png").getImage());
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setIconImage(Image)` | Sets the frame's icon. |
| `Toolkit.getDefaultToolkit().getImage(String)` | Loads an image from a file. |
| `ImageIcon(String).getImage()` | Loads an image and returns its `Image`. |

#### Syntax Rules

- The image should be loaded before calling `setIconImage`.
- Supported formats depend on the platform (typically PNG, GIF, JPEG).
- Should be called before `setVisible(true)`.

#### Constraints and Limitations

- Image loading may fail silently if the file is not found.
- The platform may scale or crop the icon.
- Multi-resolution icons are not directly supported; a single image is used.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Setting a Custom Icon

```java
import javax.swing.*;
import java.awt.*;

public class IconDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Icon Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 250);

            ImageIcon icon = new ImageIcon("app-icon.png"); // 1. Load icon
            frame.setIconImage(icon.getImage());            // 2. Set icon

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window appears with the custom icon “app-icon.png” displayed in the title bar and task bar.

**Why the code produces this result.** `ImageIcon` loads the image from the file system, and `setIconImage` passes the underlying `Image` to the native window manager, which displays it as the window icon.

---

### Real-World Cases

- **Branded applications** use a custom icon for recognition.
- **Multi-document applications** change the icon based on the document type.
- **Games** use a game logo as the window icon.

---

## 18. Frame Methods — setExtendedState()

### Definitions

**Core Definition.** `setExtendedState(int state)` sets the frame's extended state, which can be `NORMAL`, `ICONIFIED`, `MAXIMIZED_HORIZ`, `MAXIMIZED_VERT`, or `MAXIMIZED_BOTH`.

**Technical Definition.** `public void setExtendedState(int state)` is inherited from `java.awt.Frame`. The state is a bitwise mask. `MAXIMIZED_BOTH` concatenates `MAXIMIZED_HORIZ` and `MAXIMIZED_VERT`. Platform support is determined by `Toolkit.isFrameStateSupported(int)`.

**Beginner-Friendly Explanation.** `setExtendedState` is how you tell the window to maximize, minimize, or return to normal size programmatically.

---

### Purposes

- To programmatically maximize the frame to fill the screen.
- To minimize the frame to the task bar.
- To restore the frame to its normal state.
- To create applications that start maximized.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setExtendedState(JFrame.MAXIMIZED_BOTH); // maximize
frame.setExtendedState(JFrame.ICONIFIED);      // minimize
frame.setExtendedState(JFrame.NORMAL);         // restore
```

#### Component Breakdown

| Constant | Meaning |
|----------|---------|
| `NORMAL` | No state bits set (restored). |
| `ICONIFIED` | Minimized to task bar. |
| `MAXIMIZED_HORIZ` | Maximized horizontally. |
| `MAXIMIZED_VERT` | Maximized vertically. |
| `MAXIMIZED_BOTH` | Maximized in both directions. |

#### Syntax Rules

- The state is a bitwise mask; multiple states can be combined.
- Should be called after the frame is visible for reliable results.
- Platform support should be verified with `Toolkit.isFrameStateSupported`.

#### Constraints and Limitations

- Not all states are supported on all platforms.
- When the frame is not visible, state changes may not generate events.
- `ICONIFIED` combined with other states may produce unexpected results.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Starting Maximized

```java
import javax.swing.*;

public class MaximizedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Maximized App");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
            frame.setExtendedState(JFrame.MAXIMIZED_BOTH); // 1. Maximize
        });
    }
}
```

**Expected Output.** The window appears and then immediately maximizes to fill the entire screen.

**Why the code produces this result.** `setExtendedState(JFrame.MAXIMIZED_BOTH)` requests the window manager to maximize the frame horizontally and vertically. Calling it after `setVisible(true)` ensures the native window exists and can respond to the state change.

---

### Real-World Cases

- **Dashboard applications** start maximized for maximum data visibility.
- **Kiosk applications** use `MAXIMIZED_BOTH` and `setUndecorated(true)`.
- **Restoring window state** from saved preferences (maximized vs. normal).

---

## 19. Frame Methods — setUndecorated()

### Definitions

**Core Definition.** `setUndecorated(boolean undecorated)` removes or restores the frame's native decorations.

**Technical Definition.** `public void setUndecorated(boolean undecorated)` is inherited from `java.awt.Frame`. It can only be called while the frame is not displayable (before `pack()` or `setVisible(true)`). Calling it on a displayable frame throws `IllegalComponentStateException`.

**Beginner-Friendly Explanation.** `setUndecorated(true)` strips the window of its title bar and borders, leaving a bare rectangle.

---

### Purposes

- To create splash screens without chrome.
- To implement custom window decorations.
- To create full-screen or shaped windows.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JFrame frame = new JFrame();
frame.setUndecorated(true); // before displayable
frame.setSize(400, 300);
frame.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setUndecorated(boolean)` | `true` removes decorations; `false` restores them (before display). |

#### Syntax Rules

- Must be called before `pack()` or `setVisible(true)`.
- Cannot be changed after the frame is displayable.

#### Constraints and Limitations

- Throws `IllegalComponentStateException` if called after display.
- Undecorated frames cannot be moved or resized by the user through native decorations.
- May not appear in the task bar on some platforms.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Undecorated Window (Correct Order)

```java
import javax.swing.*;

public class UndecoratedCorrect {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Undecorated");
            frame.setUndecorated(true); // 1. BEFORE display
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A bare 400×250 rectangle appears with no title bar or borders.

**Why the code produces this result.** `setUndecorated(true)` is called before `setVisible(true)`, so the native peer is created without decorations.

---

### Real-World Cases

- **Splash screens** use undecorated frames.
- **Custom window shapes** use undecorated frames with transparency.
- **Full-screen games** use undecorated maximized frames.

---

## 20. Frame Methods — setAlwaysOnTop()

### Definitions

**Core Definition.** `setAlwaysOnTop(boolean alwaysOnTop)` requests the window manager to keep the frame above all other windows.

**Technical Definition.** `public void setAlwaysOnTop(boolean alwaysOnTop)` is inherited from `java.awt.Window`. Support is platform-dependent and may require the window to be visible.

**Beginner-Friendly Explanation.** `setAlwaysOnTop(true)` makes the window float above everything else on the screen.

---

### Purposes

- To keep critical notifications visible.
- To implement floating palettes.
- To create monitoring dashboards that should not be obscured.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setAlwaysOnTop(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setAlwaysOnTop(boolean)` | Enables or disables always-on-top behavior. |

#### Syntax Rules

- Can be called before or after visibility.
- Platform support varies.

#### Constraints and Limitations

- May be ignored on some platforms.
- Can be disruptive to the user experience.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Always-on-Top Palette

```java
import javax.swing.*;

public class AlwaysOnTopPalette {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Palette");
            frame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
            frame.setSize(200, 300);
            frame.setAlwaysOnTop(true); // 1. Float
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A small 200×300 window that remains above other windows even when they are focused.

**Why the code produces this result.** `setAlwaysOnTop(true)` instructs the window manager to keep the frame in the topmost layer.

---

### Real-World Cases

- **Tool palettes** in graphics editors.
- **System monitors** and performance dashboards.
- **Reminder pop-ups** and alerts.

---

## 21. Frame Methods — setLocationRelativeTo()

### Definitions

**Core Definition.** `setLocationRelativeTo(Component c)` centers the frame relative to the specified component, or centers it on the screen if `c` is `null`.

**Technical Definition.** `public void setLocationRelativeTo(Component c)` is inherited from `java.awt.Window`. It computes the appropriate location based on the component's bounds and the frame's size.

**Beginner-Friendly Explanation.** `setLocationRelativeTo` is a smart way to center a window. Pass `null` to center on the screen, or pass another window to center over it.

---

### Purposes

- To center the frame on the screen.
- To center a dialog or secondary window over its owner.
- To avoid manual coordinate calculations.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.setLocationRelativeTo(null);    // center on screen
frame.setLocationRelativeTo(owner);   // center over owner
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `setLocationRelativeTo(Component)` | Centers relative to the given component; `null` means screen center. |

#### Syntax Rules

- Should be called after the frame's size has been set or `pack()` has been called.
- If the component is not showing, the behavior may be undefined.

#### Constraints and Limitations

- Centers on the primary screen when `null`; multi-monitor centering requires additional logic.
- If the frame is larger than the screen, it may be positioned off-screen.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Centering a Dialog over Its Parent

```java
import javax.swing.*;
import java.awt.*;

public class RelativeLocationDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame parent = new JFrame("Parent");
            parent.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            parent.setSize(600, 400);
            parent.setLocationRelativeTo(null);
            parent.setVisible(true);

            JDialog dialog = new JDialog(parent, "Child", false);
            dialog.setSize(250, 150);
            dialog.setLocationRelativeTo(parent); // 1. Center over parent
            dialog.setVisible(true);
        });
    }
}
```

**Expected Output.** A parent window appears centered on the screen. A smaller dialog appears centered over the parent window.

**Why the code produces this result.** `setLocationRelativeTo(parent)` computes the dialog's location so that its center aligns with the parent's center.

---

### Real-World Cases

- **Dialogs** are centered over their owner frames.
- **Splash screens** are centered on the screen.
- **About boxes** are centered over the main application window.

---

## 22. Frame Events — WindowListener

### Definitions

**Core Definition.** `WindowListener` is an interface for receiving window events. It declares seven methods, one for each type of window state change: opened, closing, closed, iconified, deiconified, activated, and deactivated.

**Technical Definition.** `public interface WindowListener extends EventListener`. A class interested in window events either implements all seven methods or extends `WindowAdapter` and overrides only the methods of interest. The listener is registered with `window.addWindowListener(listener)`. When the window's status changes, the relevant method is invoked on the EDT.

**Beginner-Friendly Explanation.** `WindowListener` is an “ears” for the window. It listens for things like the window opening, closing, being minimized, being restored, or gaining/losing focus. You attach it to the frame with `addWindowListener`, and it tells you when these things happen.

---

### Purposes

- To react to the window being opened or closed.
- To intercept the window-closing event (e.g., to prompt for unsaved changes).
- To respond to minimize, restore, activate, and deactivate events.
- To perform cleanup when the window is closed.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.addWindowListener(new WindowListener() {
    public void windowOpened(WindowEvent e) {}
    public void windowClosing(WindowEvent e) {}
    public void windowClosed(WindowEvent e) {}
    public void windowIconified(WindowEvent e) {}
    public void windowDeiconified(WindowEvent e) {}
    public void windowActivated(WindowEvent e) {}
    public void windowDeactivated(WindowEvent e) {}
});
```

#### Component Breakdown

| Method | Invoked When |
|--------|-------------|
| `windowOpened` | The window is first made visible. |
| `windowClosing` | The user attempts to close the window. |
| `windowClosed` | The window has been disposed. |
| `windowIconified` | The window is minimized. |
| `windowDeiconified` | The window is restored from minimized state. |
| `windowActivated` | The window becomes the active window. |
| `windowDeactivated` | The window is no longer the active window. |

#### Syntax Rules

- All seven methods must be implemented when implementing `WindowListener` directly.
- Use `WindowAdapter` to override only the methods of interest.
- The listener is registered with `addWindowListener`.

#### Constraints and Limitations

- `WindowListener` has no built-in support for window state changes like maximization; use `WindowStateListener` for those.
- `windowClosing` is called before the window closes; `windowClosed` is called after disposal.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full WindowListener Implementation

```java
import javax.swing.*;
import java.awt.event.*;

public class FullWindowListener {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Window Listener");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
            frame.setSize(400, 250);

            frame.addWindowListener(new WindowListener() { // 1. Register
                public void windowOpened(WindowEvent e) {
                    System.out.println("Window opened");
                }
                public void windowClosing(WindowEvent e) {
                    System.out.println("Window closing"); // 2. Intercept
                    frame.dispose();
                }
                public void windowClosed(WindowEvent e) {
                    System.out.println("Window closed");
                }
                public void windowIconified(WindowEvent e) {
                    System.out.println("Window iconified");
                }
                public void windowDeiconified(WindowEvent e) {
                    System.out.println("Window deiconified");
                }
                public void windowActivated(WindowEvent e) {
                    System.out.println("Window activated");
                }
                public void windowDeactivated(WindowEvent e) {
                    System.out.println("Window deactivated");
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints "Window opened" when the frame appears. Minimizing prints "Window iconified"; restoring prints "Window deiconified". Clicking the close button prints "Window closing" and then "Window closed" after the frame is disposed.

**Why the code produces this result.** Each state change fires the corresponding `WindowEvent`, and the listener prints a message. `DO_NOTHING_ON_CLOSE` allows the listener to handle the closing event manually.

---

### Real-World Cases

- **Unsaved changes prompts** intercept `windowClosing` to ask the user to save.
- **Resource cleanup** releases database connections in `windowClosed`.
- **Analytics** track window activation and deactivation for usage metrics.

---

## 23. Frame Events — WindowAdapter

### Definitions

**Core Definition.** `WindowAdapter` is an abstract adapter class for receiving window events. It provides empty implementations of all seven `WindowListener` methods, allowing subclasses to override only the methods of interest.

**Technical Definition.** `public abstract class WindowAdapter implements WindowListener, WindowStateListener, WindowFocusListener`. It is the recommended way to handle window events when not all seven methods are needed.

**Beginner-Friendly Explanation.** `WindowAdapter` is a convenience class that saves you from writing seven empty methods every time you only care about one or two window events.

---

### Purposes

- To simplify window event handling by providing default empty implementations.
- To allow selective overriding of only the events of interest.
- To reduce boilerplate code in event-handling classes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        // handle closing
    }
});
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `WindowAdapter` | Abstract adapter with empty implementations of all `WindowListener` methods. |
| Overridden method | The specific event handler you want to implement. |

#### Syntax Rules

- Extend `WindowAdapter` and override only the desired methods.
- Register with `addWindowListener`.

#### Constraints and Limitations

- Only one method can be overridden per anonymous class instance (unless multiple are needed).
- Does not handle `WindowStateListener` events unless explicitly extended.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: WindowAdapter for Close Confirmation

```java
import javax.swing.*;
import java.awt.event.*;

public class AdapterDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Adapter Demo");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
            frame.setSize(400, 200);

            frame.addWindowListener(new WindowAdapter() { // 1. Extend adapter
                @Override
                public void windowClosing(WindowEvent e) {
                    int result = JOptionPane.showConfirmDialog(
                        frame, "Are you sure?");
                    if (result == JOptionPane.YES_OPTION) {
                        frame.dispose(); // 2. Close if confirmed
                    }
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the close button shows a confirmation dialog. Clicking "Yes" disposes the frame; clicking "No" or "Cancel" keeps the frame open.

**Why the code produces this result.** `WindowAdapter` allows overriding only `windowClosing`. The confirmation dialog blocks until the user responds, and `dispose()` is called only if confirmed.

---

### Real-World Cases

- **Close confirmation dialogs** in editors and data-entry applications.
- **Logging window lifecycle events** with minimal code.
- **Cleanup operations** in `windowClosed` without implementing all seven methods.

---

## 24. Frame Events — windowOpened()

### Definitions

**Core Definition.** `windowOpened(WindowEvent e)` is invoked the first time the window is made visible.

**Technical Definition.** This method is part of `WindowListener`. It is called once per window display, after the window becomes visible.

**Beginner-Friendly Explanation.** `windowOpened` fires when the window first appears on the screen.

---

### Purposes

- To perform initialization that requires the window to be visible.
- To start animations or timers when the window appears.
- To log the window's opening.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowOpened(WindowEvent e) {
    // initialization code
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowOpened(WindowEvent)` | Invoked once when the window is first shown. |

#### Syntax Rules

- Called only once per window display.
- Not called when the window is hidden and re-shown (only the first time).

#### Constraints and Limitations

- May not be called if the window is already visible when the listener is added.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Starting a Timer on Window Open

```java
import javax.swing.*;
import java.awt.event.*;

public class OpenedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Opened Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowOpened(WindowEvent e) {
                    System.out.println("Window is now visible."); // 1
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints "Window is now visible." once when the window first appears.

**Why the code produces this result.** `windowOpened` is invoked after the frame becomes visible for the first time.

---

### Real-World Cases

- **Starting background tasks** once the GUI is ready.
- **Animating splash screens** that fade in.
- **Logging application startup** for diagnostics.

---

## 25. Frame Events — windowClosing()

### Definitions

**Core Definition.** `windowClosing(WindowEvent e)` is invoked when the user attempts to close the window from the system menu (e.g., by clicking the close button).

**Technical Definition.** This method is part of `WindowListener`. It is called before the window is actually closed, allowing the programmer to intercept the close operation, prompt the user, or perform cleanup.

**Beginner-Friendly Explanation.** `windowClosing` fires when the user tries to close the window. You can use it to ask “Are you sure?” before actually closing.

---

### Purposes

- To intercept the close operation and prompt for confirmation.
- To save data before the window closes.
- To prevent the window from closing under certain conditions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowClosing(WindowEvent e) {
    // handle closing
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowClosing(WindowEvent)` | Invoked when the user attempts to close the window. |

#### Syntax Rules

- Must set `setDefaultCloseOperation(DO_NOTHING_ON_CLOSE)` to prevent the default close behavior.
- Call `dispose()` or `System.exit(0)` to actually close the window after handling.

#### Constraints and Limitations

- Not called if `EXIT_ON_CLOSE` or `DISPOSE_ON_CLOSE` handles the close automatically.
- The programmer is responsible for actually closing the window.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Save Confirmation on Close

```java
import javax.swing.*;
import java.awt.event.*;

public class ClosingDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Closing Demo");
            frame.setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
            frame.setSize(400, 200);

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowClosing(WindowEvent e) {
                    int choice = JOptionPane.showConfirmDialog(
                        frame, "Save before closing?");
                    if (choice == JOptionPane.YES_OPTION) {
                        // save data
                        frame.dispose();
                    } else if (choice == JOptionPane.NO_OPTION) {
                        frame.dispose();
                    }
                    // CANCEL: do nothing
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Clicking the close button shows a "Save before closing?" dialog. "Yes" and "No" both dispose the frame; "Cancel" keeps it open.

**Why the code produces this result.** `DO_NOTHING_ON_CLOSE` prevents automatic closing. `windowClosing` handles the confirmation logic and explicitly disposes the frame when appropriate.

---

### Real-World Cases

- **Text editors** prompt to save unsaved changes.
- **Data-entry forms** confirm discard of unsaved data.
- **Games** confirm exit during active gameplay.

---

## 26. Frame Events — windowClosed()

### Definitions

**Core Definition.** `windowClosed(WindowEvent e)` is invoked after the window has been closed as a result of calling `dispose()`.

**Technical Definition.** This method is part of `WindowListener`. It is called after the window is no longer displayable and its native resources have been released.

**Beginner-Friendly Explanation.** `windowClosed` fires after the window is completely gone.

---

### Purposes

- To perform cleanup after the window is closed.
- To release resources that were held by the window.
- To log the window's closure.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowClosed(WindowEvent e) {
    // cleanup code
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowClosed(WindowEvent)` | Invoked after the window is disposed. |

#### Syntax Rules

- Called after `dispose()`.
- Not called if the window is hidden with `setVisible(false)`.

#### Constraints and Limitations

- The window is no longer usable when this method is called.
- May not be called if the JVM exits before disposal completes.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cleanup After Close

```java
import javax.swing.*;
import java.awt.event.*;

public class ClosedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Closed Demo");
            frame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
            frame.setSize(300, 150);

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowClosed(WindowEvent e) {
                    System.out.println("Window resources released."); // 1
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the window is closed and disposed, the console prints "Window resources released."

**Why the code produces this result.** `DISPOSE_ON_CLOSE` calls `dispose()`, which triggers `windowClosed` after the native resources are released.

---

### Real-World Cases

- **Database connection cleanup** after a window closes.
- **Temporary file deletion** when an editor window closes.
- **Analytics logging** of window closure events.

---

## 27. Frame Events — windowIconified()

### Definitions

**Core Definition.** `windowIconified(WindowEvent e)` is invoked when the window is minimized (iconified) to the task bar.

**Technical Definition.** This method is part of `WindowListener`. It is called when the window's state changes to `ICONIFIED`.

**Beginner-Friendly Explanation.** `windowIconified` fires when the user minimizes the window.

---

### Purposes

- To pause animations or timers when the window is minimized.
- To reduce resource usage while the window is not visible.
- To log minimize events for usage analytics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowIconified(WindowEvent e) {
    // handle minimize
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowIconified(WindowEvent)` | Invoked when the window is minimized. |

#### Syntax Rules

- Fired when the window is iconified, regardless of how (user action or programmatic).
- Pairs with `windowDeiconified` for restore events.

#### Constraints and Limitations

- May not fire on platforms that do not support iconification.
- Some window managers may handle minimization differently.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Pausing a Timer on Minimize

```java
import javax.swing.*;
import java.awt.event.*;

public class IconifiedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Iconified Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);

            Timer timer = new Timer(1000, e ->
                System.out.println("Tick"));
            timer.start();

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowIconified(WindowEvent e) {
                    timer.stop(); // 1. Pause on minimize
                    System.out.println("Timer paused.");
                }
                @Override
                public void windowDeiconified(WindowEvent e) {
                    timer.start(); // 2. Resume on restore
                    System.out.println("Timer resumed.");
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The timer prints "Tick" every second. When the window is minimized, the timer pauses and "Timer paused." is printed. When restored, "Timer resumed." is printed and ticking continues.

**Why the code produces this result.** `windowIconified` stops the `Timer` to conserve resources. `windowDeiconified` restarts it when the window becomes visible again.

---

### Real-World Cases

- **Games** pause when minimized.
- **Animations** stop rendering when the window is not visible.
- **Polling applications** reduce network activity when minimized.

---

## 28. Frame Events — windowDeiconified()

### Definitions

**Core Definition.** `windowDeiconified(WindowEvent e)` is invoked when the window is restored from a minimized state to a normal state.

**Technical Definition.** This method is part of `WindowListener`. It is called when the window's state changes from `ICONIFIED` back to `NORMAL`.

**Beginner-Friendly Explanation.** `windowDeiconified` fires when the user restores a minimized window.

---

### Purposes

- To resume animations or timers when the window is restored.
- To refresh data displayed in the window.
- To log restore events.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowDeiconified(WindowEvent e) {
    // handle restore
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowDeiconified(WindowEvent)` | Invoked when the window is restored from minimized state. |

#### Syntax Rules

- Pairs with `windowIconified`.
- Fired when the window becomes visible again after being minimized.

#### Constraints and Limitations

- May not fire on platforms that do not support iconification.
- Restoration behavior varies across window managers.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Refreshing Data on Restore

```java
import javax.swing.*;
import java.awt.event.*;

public class DeiconifiedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Deiconified Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            JLabel status = new JLabel("Data loaded.");
            frame.add(status);

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowDeiconified(WindowEvent e) {
                    status.setText("Data refreshed at " +
                        new java.util.Date()); // 1. Refresh on restore
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the window is minimized and then restored, the label updates to show the current time, indicating that data has been refreshed.

**Why the code produces this result.** `windowDeiconified` is triggered when the window is restored, and the listener updates the label with the current timestamp.

---

### Real-World Cases

- **Email clients** check for new messages when the window is restored.
- **Dashboard applications** refresh data when brought back from minimization.
- **Games** resume rendering when restored.

---

## 29. Frame Events — windowActivated()

### Definitions

**Core Definition.** `windowActivated(WindowEvent e)` is invoked when the window is set to be the active window (i.e., it gains focus).

**Technical Definition.** This method is part of `WindowListener`. Only a `Frame` or `Dialog` can be the active window. The active window is typically denoted by a highlighted title bar.

**Beginner-Friendly Explanation.** `windowActivated` fires when the window becomes the one you are currently using—the one with the highlighted title bar.

---

### Purposes

- To update the window's appearance when it gains focus.
- To resume paused activities when the window becomes active.
- To log focus events for usage analytics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowActivated(WindowEvent e) {
    // handle activation
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowActivated(WindowEvent)` | Invoked when the window becomes active. |

#### Syntax Rules

- Only frames and dialogs can be active windows.
- Pairs with `windowDeactivated`.

#### Constraints and Limitations

- The native windowing system determines which window is active.
- Activation does not necessarily mean the window has keyboard focus.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Updating Status on Activation

```java
import javax.swing.*;
import java.awt.event.*;

public class ActivatedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Activated Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(300, 150);
            JLabel label = new JLabel("Inactive", SwingConstants.CENTER);
            frame.add(label);

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowActivated(WindowEvent e) {
                    label.setText("Active"); // 1. On activation
                }
                @Override
                public void windowDeactivated(WindowEvent e) {
                    label.setText("Inactive"); // 2. On deactivation
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The label shows "Active" when the window is focused and "Inactive" when it loses focus.

**Why the code produces this result.** `windowActivated` updates the label when the window gains focus, and `windowDeactivated` updates it when focus is lost.

---

### Real-World Cases

- **Chat applications** mark messages as read when the window is activated.
- **Editors** show the cursor and selection when activated.
- **Games** pause rendering when deactivated to save resources.

---

## 30. Frame Events — windowDeactivated()

### Definitions

**Core Definition.** `windowDeactivated(WindowEvent e)` is invoked when the window is no longer the active window.

**Technical Definition.** This method is part of `WindowListener`. It is called when another window becomes active, causing this window to lose its active status.

**Beginner-Friendly Explanation.** `windowDeactivated` fires when you click on another window and your window is no longer the one in focus.

---

### Purposes

- To pause activities when the window loses focus.
- To update the window's appearance to indicate it is inactive.
- To save state when the user switches away.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
@Override
public void windowDeactivated(WindowEvent e) {
    // handle deactivation
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `windowDeactivated(WindowEvent)` | Invoked when the window loses active status. |

#### Syntax Rules

- Pairs with `windowActivated`.
- Fired when another window becomes active.

#### Constraints and Limitations

- Deactivation does not necessarily mean the window is minimized or hidden.
- The native windowing system determines activation order.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Saving State on Deactivation

```java
import javax.swing.*;
import java.awt.event.*;

public class DeactivatedDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Deactivated Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 250);
            JTextArea area = new JTextArea();
            frame.add(new JScrollPane(area));

            frame.addWindowListener(new WindowAdapter() {
                @Override
                public void windowDeactivated(WindowEvent e) {
                    System.out.println("Auto-saving: " +
                        area.getText()); // 1. Save on deactivation
                }
            });

            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** When the user clicks another window, the console prints "Auto-saving: " followed by the current text in the text area.

**Why the code produces this result.** `windowDeactivated` triggers the save logic whenever the window loses focus, implementing an auto-save-on-blur behavior.

---

### Real-World Cases

- **Auto-save** in editors when the user switches away.
- **Pausing media playback** when the media player loses focus.
- **Status updates** in collaboration tools when the user switches windows.

---

## 31. JFrame Best Practices

### Definitions

**Core Definition.** JFrame best practices are the conventions and patterns that ensure Swing applications are thread-safe, resource-efficient, visually consistent, and maintainable from the moment the first window is created.

**Technical Definition.** Best practices include: constructing and showing frames on the EDT via `SwingUtilities.invokeLater`; setting explicit close operations; using `pack()` instead of arbitrary sizes; disposing of unneeded frames; avoiding heavy work on the EDT; and choosing `DISPOSE_ON_CLOSE` for secondary windows while reserving `EXIT_ON_CLOSE` for the main application frame.

**Beginner-Friendly Explanation.** Best practices are the “rules of the road” for building windows in Swing. They help you avoid crashes, frozen interfaces, and windows that refuse to close. They include things like “always build the window on the special GUI thread,” “tell the window what to do when the user clicks the X,” and “do not do heavy work inside a button click.”

---

### Purposes

- To prevent thread-related bugs such as race conditions, painting artifacts, and deadlocks.
- To ensure the application terminates correctly when the main window is closed.
- To avoid resource leaks by disposing of windows that are no longer needed.
- To maintain a responsive GUI by keeping the EDT free of long-running tasks.
- To create layouts that adapt gracefully to different screen sizes and font settings.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
import javax.swing.*;

public class BestPracticeApp {

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeApp::createAndShowGUI); // 1
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("App");                        // 2
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);     // 3
        frame.pack();                                             // 4
        frame.setLocationRelativeTo(null);                        // 5
        frame.setVisible(true);                                   // 6
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT transfer | `SwingUtilities.invokeLater(...)` in `main`. |
| Explicit close operation | `setDefaultCloseOperation(EXIT_ON_CLOSE)` for the main frame; `DISPOSE_ON_CLOSE` for secondary windows. |
| Pack, don't guess | Use `pack()` to size the frame to its content instead of arbitrary `setSize`. |
| Center the frame | `setLocationRelativeTo(null)` centers on the screen. |
| Visibility last | `setVisible(true)` should be the final call. |
| No heavy work on EDT | Use `SwingWorker` or background threads for I/O, computation, and network access. |
| Dispose unneeded frames | Call `dispose()` on secondary frames and dialogs when they are no longer needed. |

#### Syntax Rules

- **Never construct a `JFrame` on the main thread.** Always use `invokeLater`.
- **Always set a close operation.** The default `HIDE_ON_CLOSE` can leave the JVM running invisibly.
- **Use `pack()` before `setVisible(true)`** when possible, so the window is sized to its content.
- **Dispose windows you no longer need** to release native resources.
- **Do not block the EDT** with long-running tasks.

#### Constraints and Limitations

- `invokeLater` does not guarantee immediate execution; it only guarantees eventual execution on the EDT.
- `pack()` may produce a very small or very large window if preferred sizes are not set correctly.
- `setLocationRelativeTo(null)` centers on the primary screen; multi-screen setups may require additional logic.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Application

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(BestPracticeDemo::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        // 1. Create frame with title
        JFrame frame = new JFrame("Best Practice App");

        // 2. Explicit close operation
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // 3. Build content
        JPanel panel = new JPanel(new FlowLayout());
        JLabel label = new JLabel("Enter name:");
        JTextField field = new JTextField(15);
        JButton button = new JButton("Greet");
        JLabel output = new JLabel("");

        button.addActionListener(e ->
            output.setText("Hello, " + field.getText() + "!"));

        panel.add(label);
        panel.add(field);
        panel.add(button);
        panel.add(output);

        // 4. Add to frame
        frame.add(panel);

        // 5. Size to content
        frame.pack();

        // 6. Center and show
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A compact window appears with a label, text field, button, and output label. Typing a name and clicking "Greet" displays "Hello, [name]!". Closing the window terminates the JVM.

**Why the code produces this result.** The GUI is constructed on the EDT, the close operation is explicitly set to `EXIT_ON_CLOSE`, and `pack()` sizes the frame to its content. The event handler is short and updates the label directly.

---

#### Example 2: Common Mistake — Direct Construction in `main` (What Not to Do)

```java
import javax.swing.*;

public class BadMain {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Bad Main"); // WRONG: not on EDT
        JButton button = new JButton("Click"); // WRONG
        frame.add(button);
        frame.setSize(300, 100);
        frame.setVisible(true);
    }
}
```

**Expected Output.** The window may appear, but it may also exhibit intermittent painting artifacts, unresponsive buttons, or race conditions. Behavior is not guaranteed.

**Why this is wrong.** `main` runs on the main thread, not the EDT. Swing components are not thread-safe, and constructing them off the EDT violates Swing's threading policy. The correct approach is to wrap the GUI construction in `SwingUtilities.invokeLater`.

---

### Real-World Cases

- **Production Swing applications** follow these practices to ensure stability and maintainability.
- **Educational examples** use the best-practice skeleton to teach students the correct pattern from the start.
- **Legacy code migration** moves GUI construction into `invokeLater` and sets explicit close operations.

---

## References

Oracle. *Class JFrame*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/en/java/javase/26/docs/api/java.desktop/javax/swing/JFrame.html.

Oracle. *Interface WindowConstants*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/javase/jp/26/docs/api/java.desktop/javax/swing/WindowConstants.html.

Oracle. *Interface WindowListener*. JavaTM 2 Platform Standard Edition 5.0 API Specification. https://docs.oracle.com/javase/1.5.0/docs/api/java/awt/event/WindowListener.html.

Oracle. *Class Frame (Java SE 11 & JDK 11)*. https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/Frame.html.

Oracle. *Creating the Demo Application (Step 1)*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/painting/step1.html.

Oracle. *How to Make Frames (Main Windows)*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/frame.html.

Oracle. *Package javax.swing*. Java Platform, Standard Edition 26 API Specification. https://docs.oracle.com/en/java/javase/26/docs/api/java.desktop/javax/swing/package-summary.html.