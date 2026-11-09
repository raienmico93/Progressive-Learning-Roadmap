# Top-Level Containers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** Top-level containers are the root containers of every Swing-based graphical user interface. They are the only Swing components that can exist independently on the desktop and are the entry points through which all other Swing components become visible. Swing provides four top-level container classes: `JFrame`, `JDialog`, `JWindow`, and (historically) `JApplet`.

**Technical Definition.** A top-level container is a subclass of `java.awt.Window` (or, in the case of `JApplet`, `java.applet.Applet`) that implements `RootPaneContainer`. Every top-level container holds a single `JRootPane` as its immediate child, and the root pane manages the `contentPane`, `glassPane`, `layeredPane`, and optional `menuBar`. The `add`, `remove`, and `setLayout` methods of top-level containers are overridden to delegate to the content pane, so that `frame.add(component)` is functionally equivalent to `frame.getContentPane().add(component)`.

**Beginner-Friendly Explanation.** Imagine you are building a house. The top-level container is the land and the outer walls—it is the thing that exists on the “desktop” (the physical ground). Everything else—rooms, furniture, decorations—must be placed inside it. Swing gives you different types of “land”: a main house with a front door and address (`JFrame`), a temporary guest house that pops up when needed (`JDialog`), a simple shed with no doors or windows (`JWindow`), and an old-style house that used to sit inside a web browser (`JApplet`). You cannot put a table (a `JButton`) directly on the street; it must go inside one of these structures.

---

### Key Characteristics

1. **Root of containment hierarchies.** Every Swing component that appears on screen must be part of a containment hierarchy whose root is a top-level container. An application with one main window and two dialogs therefore has three containment hierarchies and three top-level containers.
2. **Root pane management.** Each top-level container owns a `JRootPane`, which in turn manages the `contentPane` (where visible non-menu components go), the `glassPane` (for intercepting mouse events or drawing overlays), the `layeredPane` (for menu bars and popups), and an optional `JMenuBar`.
3. **Heavyweight nature.** Top-level containers are the only heavyweight components in Swing. They depend on native platform windows for their existence, while their child components are lightweight and painted by Swing itself.
4. **Single-thread rule.** Like all Swing components, top-level containers are not thread-safe. They must be constructed, shown, and manipulated exclusively on the Event Dispatch Thread (EDT).
5. **Explicit close semantics.** `JFrame` and `JDialog` support `setDefaultCloseOperation`, which determines whether closing the window disposes it, hides it, or exits the JVM. `JWindow` has no decorations and therefore no built-in close behavior.
6. **Owner–dependent relationships.** `JDialog` and `JWindow` can be associated with an owner window. When the owner is minimized or destroyed, its dependent dialogs and windows follow suit.
7. **Deprecation of `JApplet`.** The Applet API, including `JApplet`, was deprecated in JDK 9 and marked for removal in JDK 17 (`forRemoval = true`). It was removed in JDK 26 (JEP 504). New development should not use `JApplet`; migration paths include Java Web Start and self-contained applications, though Java Web Start itself has also been discontinued in modern JDKs.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, anonymous classes, lambda expressions, and the `public static void main(String[] args)` signature.
- **AWT concepts:** `Container`, `Window`, `Frame`, `Dialog`, `LayoutManager`, and the component hierarchy.
- **Swing threading model:** understanding that `main` runs on the main thread and that Swing components must be accessed on the EDT via `SwingUtilities.invokeLater`.
- **Event handling:** `ActionListener`, `WindowListener`, and the event delegation model.
- **A JDK (version 8 or later for `JFrame`, `JDialog`, and `JWindow`; `JApplet` requires JDK 8 or earlier for practical use, though it is present but deprecated in JDK 9–25 and removed in JDK 26).**

---

### Related Programming Areas

- **AWT (Abstract Window Toolkit):** The parent framework of Swing. `JFrame` extends `java.awt.Frame`, `JDialog` extends `java.awt.Dialog`, and `JWindow` extends `java.awt.Window`.
- **Java 2D:** Used for painting the contents of lightweight components inside top-level containers.
- **Model-View-Controller (MVC):** Swing’s separable model architecture applies to components hosted inside top-level containers, though the containers themselves are primarily view/controller shells.
- **Concurrency in Java:** The EDT is a single thread; `SwingWorker` and background threads are used to keep the GUI responsive.
- **Accessibility:** All top-level containers implement the `Accessible` interface, enabling assistive technologies to interrogate the GUI.
- **Java Web Start / Deployment:** The historical successor to applets; relevant when migrating `JApplet`-based code.

---

## Core Concepts and Features

The following sections use a uniform structure for each top-level container: **Definitions**, **Purposes**, **Syntax Rules and Structure**, **Annotated Code Examples**, and **Real-World Cases**.

---

## 1. JFrame — The Main Application Window

### Definitions

**Core Definition.** `JFrame` is an extended version of `java.awt.Frame` that adds support for the JFC/Swing component architecture. It is a top-level window with a title bar, border, and window-management buttons (minimize, maximize, close) that can be minimized to an icon.

**Technical Definition.** `public class JFrame extends Frame implements WindowConstants, Accessible, RootPaneContainer`. A `JFrame` contains a `JRootPane` as its only child. The root pane’s `contentPane`, which always has a non-null value and uses `BorderLayout` by default, should contain all non-menu components displayed by the frame. The `add`, `remove`, and `setLayout` methods of `JFrame` are overridden to delegate to the content pane.

**Beginner-Friendly Explanation.** `JFrame` is the main window of a desktop application—the thing with a title bar, a minimize button, a maximize button, and a close button. You do not put buttons and labels directly into the `JFrame`; instead, you put them into the frame’s content area. Think of `JFrame` as the picture frame and the content pane as the canvas inside it.

---

### Purposes

- To provide a top-level window with native decorations (title bar, border, close button, iconify button) for a standalone Swing application.
- To serve as the root of the primary containment hierarchy, through its `JRootPane`, so that menus and content can be managed consistently.
- To define the application’s lifecycle through `setDefaultCloseOperation(int)`, allowing the programmer to choose between hiding the window, disposing it, or exiting the JVM.
- To support multi-screen environments by allowing a `JFrame` to be created on a specific `GraphicsDevice`.
- To act as the owner/parent for secondary windows such as `JDialog` and `JWindow`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JFrame frame = new JFrame("Title");
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setSize(500, 400);
frame.setLocationRelativeTo(null);
frame.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `new JFrame("Title")` | Creates a frame with the specified title bar text. |
| `setDefaultCloseOperation(int)` | Defines the action taken when the user clicks the close button. Constants: `EXIT_ON_CLOSE`, `DISPOSE_ON_CLOSE`, `HIDE_ON_CLOSE`, `DO_NOTHING_ON_CLOSE`. |
| `setSize(int, int)` | Sets the frame’s width and height in pixels. |
| `setLocationRelativeTo(Component)` | Centers the frame relative to the specified component; `null` centers it on the screen. |
| `setVisible(true)` | Makes the frame visible. Should be the final call after construction is complete. |
| `getContentPane().add(Component)` | Adds a component to the content pane (the visible area inside the frame). `frame.add(component)` delegates to this. |

#### Syntax Rules

- `setDefaultCloseOperation` should be called explicitly. The default behavior is `HIDE_ON_CLOSE`, which hides the frame but does not terminate the JVM.
- The content pane is never `null`; attempting to set it to `null` throws an exception.
- `setVisible(true)` should be called only after all components have been added and the layout has been set.
- `JFrame` is not thread-safe; construct and show it on the EDT via `SwingUtilities.invokeLater`.
- A `JFrame` cannot be added to another container; it is a top-level container.

#### Constraints and Limitations

- `JFrame` is heavyweight and depends on native window decorations provided by the platform.
- Closing a `JFrame` with `EXIT_ON_CLOSE` terminates the entire JVM, including other windows and background threads.
- Serialized `JFrame` objects are not guaranteed to be compatible with future Swing releases.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal JFrame with Explicit Close Behavior

**Setup.** Save as `MinimalFrame.java`, compile with `javac MinimalFrame.java`, and run with `java MinimalFrame`.

```java
import javax.swing.JFrame;                     // 1. Import JFrame
import javax.swing.SwingUtilities;             // 2. Import invokeLater

public class MinimalFrame {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {     // 3. Schedule on the EDT
            JFrame frame = new JFrame("Minimal Frame"); // 4. Create frame
            frame.setDefaultCloseOperation(     // 5. Exit JVM on close
                JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 300);            // 6. Width 400, height 300
            frame.setLocationRelativeTo(null);  // 7. Center on screen
            frame.setVisible(true);             // 8. Show the window
        });
    }
}
```

**Expected Output.** A 400×300 window titled “Minimal Frame” appears in the center of the screen. Clicking the close button terminates the Java process.

**Why the code produces this result.** `SwingUtilities.invokeLater` schedules the lambda on the EDT. The EDT constructs the `JFrame`, sets its title, size, close operation, and location, and finally makes it visible. Because `EXIT_ON_CLOSE` is set, the JVM exits when the window is closed.

---

#### Example 2: JFrame with `DISPOSE_ON_CLOSE` and a Child Dialog

```java
import javax.swing.*;
import java.awt.*;

public class DisposeFrame {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Main Window");
            frame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE); // 1
            frame.setSize(400, 250);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JDialog dialog = new JDialog(frame, "Child Dialog", false); // 2
            dialog.setSize(250, 150);
            dialog.setLocationRelativeTo(frame);
            dialog.setVisible(true);
        });
    }
}
```

**Expected Output.** A main window and a smaller non-modal dialog appear. Closing the main window disposes it (releasing its native resources), but the JVM continues running if other non-daemon threads exist. Closing the dialog does not affect the main window.

**Why the code produces this result.** `DISPOSE_ON_CLOSE` calls `Window.dispose()` rather than `System.exit(0)`. The child dialog is non-modal (`false`), so both windows remain interactive simultaneously. The dialog’s owner is the frame, so when the frame is disposed, the dialog is also disposed.

---

### Real-World Cases

- **Single-window desktop applications.** A calculator, text editor, or unit converter uses one `JFrame` as its main window.
- **Multi-window applications.** A parent `JFrame` opens secondary `JDialog` or `JFrame` windows; `DISPOSE_ON_CLOSE` prevents closing a child from killing the application.
- **Dashboard-style applications.** A maximized `JFrame` with a menu bar and a content pane containing tabbed panes or split panes.
- **Splash-screen launchers.** A `JFrame` can display a splash image before the main window appears.

---

## 2. JDialog — The Dialog Window

### Definitions

**Core Definition.** `JDialog` is a top-level window that serves as a temporary sub-window for carrying notices, collecting input, or presenting options apart from the main application window. It is a subclass of `java.awt.Dialog` that adds Swing’s `JRootPane` architecture.

**Technical Definition.** `public class JDialog extends Dialog implements WindowConstants, Accessible, RootPaneContainer`. A `JDialog` contains a `JRootPane` as its only child, and the `add`, `remove`, and `setLayout` methods are overridden to delegate to the content pane. Dialogs can be **modal** or **modeless**. A modal dialog blocks input to all other windows in the application while it is visible; a modeless dialog allows interaction with other windows.

**Beginner-Friendly Explanation.** A `JDialog` is a smaller window that pops up on top of the main window to ask you something or tell you something. Think of it as a sticky note: it appears when needed, and when you are done, it goes away. Some dialogs are “modal,” meaning you must deal with them before you can click anything else in the application. Others are “modeless,” meaning you can ignore them and keep working.

---

### Purposes

- To present temporary messages, warnings, errors, or confirmations without replacing the main application window.
- To collect user input in a dedicated, focused window (e.g., a login dialog, a settings dialog).
- To implement modal interactions that require the user to respond before continuing.
- To implement modeless interactions that allow the user to refer to the dialog while working elsewhere.
- To serve as the underlying window for `JOptionPane`-based standard dialogs (message, confirm, input, option).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JDialog dialog = new JDialog(ownerFrame, "Dialog Title", true); // modal
dialog.setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
dialog.setSize(300, 200);
dialog.setLocationRelativeTo(ownerFrame);
dialog.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `new JDialog(Frame owner, String title, boolean modal)` | Creates a dialog owned by `owner`, with the given title. `modal` determines whether the dialog blocks other windows. |
| `setDefaultCloseOperation(int)` | Defines behavior when the dialog is closed. `DISPOSE_ON_CLOSE` is typical. |
| `setModalityType(Dialog.ModalityType)` | An alternative to the `modal` constructor argument; supports `APPLICATION_MODAL`, `DOCUMENT_MODAL`, `MODELESS`, and `TOOLKIT_MODAL`. |
| `setLocationRelativeTo(Component)` | Centers the dialog relative to its owner. |
| `setVisible(true)` | Shows the dialog. For a modal dialog, this call blocks the calling thread (which must not be the EDT) until the dialog is closed. |

#### Syntax Rules

- A dialog should have an owner (`Frame` or `Dialog`). When the owner is destroyed, its dependent dialogs are also destroyed.
- Modal dialogs block input to other application windows. `setVisible(true)` on a modal dialog called from the EDT does not block the EDT itself; instead, it starts a secondary event loop.
- `DISPOSE_ON_CLOSE` is the recommended default close operation for dialogs to release native resources.
- Dialogs do not appear in the operating system’s task bar (unlike `JFrame`).
- `JDialog` is not thread-safe; construct and show it on the EDT.

#### Constraints and Limitations

- Dialogs are dependent on their owner. If the owner is iconified, the dialog disappears; if the owner is destroyed, the dialog is destroyed.
- Modal dialogs can cause deadlocks if `setVisible(true)` is called from the wrong thread or if the dialog’s construction blocks the EDT.
- Custom dialogs built directly with `JDialog` require manual layout and component management; `JOptionPane` is preferred for standard dialogs.
- Dialogs do not have maximize or minimize buttons (platform-dependent, but generally true).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Modal Confirmation Dialog

```java
import javax.swing.*;
import java.awt.*;

public class ModalDialogDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Main");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);

            JButton openButton = new JButton("Open Dialog");
            openButton.addActionListener(e -> {
                JDialog dialog = new JDialog(frame, "Confirm", true); // 1
                dialog.setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
                dialog.setLayout(new FlowLayout());

                JLabel message = new JLabel("Are you sure?");
                JButton yes = new JButton("Yes");
                JButton no  = new JButton("No");

                yes.addActionListener(ev -> dialog.dispose()); // 2
                no.addActionListener(ev -> dialog.dispose());  // 3

                dialog.add(message);
                dialog.add(yes);
                dialog.add(no);
                dialog.pack();
                dialog.setLocationRelativeTo(frame);
                dialog.setVisible(true);                       // 4
            });

            frame.add(openButton);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The main window appears with an “Open Dialog” button. Clicking it opens a small modal dialog with “Are you sure?”, “Yes”, and “No” buttons. While the dialog is visible, the main window cannot be clicked. Clicking either button disposes the dialog, returning control to the main window.

**Why the code produces this result.** The `JDialog` constructor receives `true` for the `modal` parameter, making it application-modal. `setVisible(true)` starts the dialog’s secondary event loop, which blocks input to other windows. Both buttons call `dispose()`, which closes the dialog and releases its resources.

---

#### Example 2: Modeless Dialog with `ModalityType`

```java
import javax.swing.*;
import java.awt.*;

public class ModelessDialogDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Main");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(400, 200);
            frame.setLocationRelativeTo(null);

            JButton openButton = new JButton("Open Modeless Dialog");
            openButton.addActionListener(e -> {
                JDialog dialog = new JDialog(frame, "Modeless", false); // 1
                dialog.setModalityType(Dialog.ModalityType.MODELESS);    // 2
                dialog.setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
                dialog.add(new JLabel("This dialog does not block."));
                dialog.setSize(250, 120);
                dialog.setLocationRelativeTo(frame);
                dialog.setVisible(true);
            });

            frame.add(openButton);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The main window remains fully interactive while the modeless dialog is open. The user can click the main window’s button again to open additional dialogs, and can close each dialog independently.

**Why the code produces this result.** The constructor’s `false` argument and the explicit `setModalityType(MODELESS)` ensure that the dialog does not block input to other windows. `DISPOSE_ON_CLOSE` ensures that closing the dialog releases its resources without terminating the JVM.

---

### Real-World Cases

- **Error and warning messages.** `JOptionPane.showMessageDialog` creates a modal `JDialog` behind the scenes to display an error or confirmation.
- **Login prompts.** A modal `JDialog` collects credentials before allowing access to the main application.
- **Settings and preferences.** A custom `JDialog` with form fields (text fields, checkboxes, combo boxes) lets users adjust application settings.
- **Progress indicators.** A modeless `JDialog` can display progress while the user continues to work elsewhere.

---

## 3. JWindow — The Undecorated Window

### Definitions

**Core Definition.** `JWindow` is a top-level container that has no system-provided borders, title bar, or window-management controls. It is the Swing version of `java.awt.Window`.

**Technical Definition.** `public class JWindow extends Window implements Accessible, RootPaneContainer`. A `JWindow` contains a `JRootPane` as its only child, and the `add`, `remove`, and `setLayout` methods are overridden to delegate to the content pane. It can be displayed anywhere on the user’s desktop but cannot be moved or resized by the user through native decorations.

**Beginner-Friendly Explanation.** A `JWindow` is like a floating rectangle on the screen with no frame around it. It has no title bar, no close button, and no border. You cannot drag it by its (nonexistent) title bar. It is used for things like splash screens that appear briefly at startup, or floating tool palettes that the program itself moves and controls.

---

### Purposes

- To display a borderless, decoration-free area on the desktop for custom-drawn content.
- To implement splash screens that appear while an application is initializing.
- To create floating palettes, tool tips, or overlays that should not look like standard application windows.
- To serve as a lightweight container for content that requires no native window decorations.
- To support transparency and “always on top” behavior when combined with `Window.setAlwaysOnTop(boolean)`.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
JWindow window = new JWindow(ownerFrame);
window.setSize(400, 300);
window.setLocationRelativeTo(null);
window.setVisible(true);
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `new JWindow(Frame owner)` | Creates an undecorated window owned by `owner`. The no-argument constructor creates an ownerless window. |
| `setSize(int, int)` | Sets the window’s dimensions. |
| `setLocationRelativeTo(Component)` | Centers the window relative to the screen (`null`) or another component. |
| `setAlwaysOnTop(boolean)` | Controls whether the window floats above all other windows. |
| `setVisible(true)` | Makes the window visible. |

#### Syntax Rules

- `JWindow` has no title bar or close button. The programmer is responsible for providing a way to close or dismiss it (e.g., a button, a timer, or a mouse listener).
- `JWindow` is not focusable by default in some contexts; content that requires keyboard input may need additional configuration.
- The content pane is never `null`; `window.add(component)` delegates to the content pane, which uses `BorderLayout` by default.
- `JWindow` is not thread-safe; manipulate it on the EDT.

#### Constraints and Limitations

- The user cannot move or resize a `JWindow` using native window decorations because there are none.
- `JWindow` does not appear in the task bar and cannot be minimized or maximized through standard window controls.
- Serialized `JWindow` objects are not compatible with future Swing releases.
- Because `JWindow` is undecorated, accessibility and keyboard navigation may require additional attention.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Splash-Style JWindow

```java
import javax.swing.*;
import java.awt.*;

public class SplashWindowDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JWindow window = new JWindow();               // 1. Undecorated
            window.setSize(350, 200);
            window.setLocationRelativeTo(null);           // 2. Center
            window.setAlwaysOnTop(true);                  // 3. Float on top

            JPanel panel = new JPanel(new BorderLayout());
            panel.setBackground(Color.DARK_GRAY);
            JLabel label = new JLabel("Loading...", SwingConstants.CENTER);
            label.setForeground(Color.WHITE);
            label.setFont(new Font("SansSerif", Font.BOLD, 24));
            panel.add(label, BorderLayout.CENTER);

            window.setContentPane(panel);
            window.setVisible(true);                       // 4. Show

            // Simulate loading and close after 3 seconds
            new Timer(3000, e -> window.dispose()).start(); // 5. Auto-close
        });
    }
}
```

**Expected Output.** A 350×200 dark-gray rectangle with the white text “Loading...” appears in the center of the screen and floats above other windows. After three seconds, it disappears.

**Why the code produces this result.** `JWindow` has no decorations, so only the content pane is visible. `setAlwaysOnTop(true)` keeps it above other windows. `javax.swing.Timer` fires an `ActionEvent` on the EDT after 3000 ms, and the listener calls `dispose()` to close the window.

---

#### Example 2: JWindow Owned by a JFrame

```java
import javax.swing.*;
import java.awt.*;

public class OwnedWindowDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Owner Frame");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setSize(500, 300);
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);

            JWindow window = new JWindow(frame);           // 1. Owned
            window.setSize(200, 100);
            window.setLocationRelativeTo(frame);           // 2. Center on owner
            window.getContentPane().setBackground(Color.YELLOW);
            window.getContentPane().add(
                new JLabel("Owned Window", SwingConstants.CENTER));
            window.setVisible(true);
        });
    }
}
```

**Expected Output.** A main frame and a smaller yellow undecorated window appear. The yellow window is centered over the frame. When the frame is minimized or closed, the `JWindow` follows the owner’s visibility state (it disappears when the owner is iconified and reappears when the owner is deiconified).

**Why the code produces this result.** Passing `frame` as the owner establishes a dependent relationship. The AWT window management system hides owned windows when the owner is iconified and shows them when the owner is restored.

---

### Real-World Cases

- **Splash screens.** Many desktop applications display a `JWindow` with a logo and “Loading…” text while the main application initializes.
- **Floating tool palettes.** Graphics editors and CAD tools use undecorated windows for color pickers, brush selectors, or layer panels that the application positions programmatically.
- **Custom notifications.** A `JWindow` can display a temporary toast-style notification in a corner of the screen.
- **Drag-and-drop feedback.** During a drag operation, a `JWindow` can display a “ghost” image or a drop-target indicator.

---

## 4. JApplet — The Deprecated Browser-Embedded Container

### Definitions

**Core Definition.** `JApplet` is an extended version of `java.applet.Applet` that adds support for the JFC/Swing component architecture. It was designed to host Swing GUIs inside a web browser’s Java plug-in.

**Technical Definition.** `public class JApplet extends Applet implements Accessible, RootPaneContainer`. `JApplet` contains a `JRootPane` as its only child, and its `add`, `remove`, and `setLayout` methods are overridden to delegate to the content pane. **`JApplet` is deprecated.** It was marked `@Deprecated(since="9")` and, in JDK 17, `forRemoval=true` was added. The Applet API was removed entirely in JDK 26 (JEP 504).

**Beginner-Friendly Explanation.** An applet was a Java program that ran inside a web browser, like a tiny application embedded in a webpage. `JApplet` was the Swing-enabled version of that idea. However, browsers stopped supporting the Java plug-in years ago, and modern Java versions no longer include the Applet API at all. You should **not** write new `JApplet` code. If you encounter old applet code, the recommended path is to migrate it to a standalone Swing application or, historically, to Java Web Start (though JWS itself is now also discontinued in modern JDKs).

---

### Purposes

- **Historical only:** To host a Swing-based GUI inside a web browser via the Java plug-in.
- **Historical only:** To provide a top-level container with a `JRootPane` for applets that needed Swing’s content pane, menu bar, and layered pane architecture.
- **For legacy maintenance:** To understand and modify existing applet code that still uses `JApplet` (before removal).
- **For migration planning:** To recognize the structural similarities between `JApplet` and `JFrame`/`JDialog` so that applet code can be refactored into standalone windows.

---

### Syntax Rules and Structure

#### Complete General Syntax (Historical)

```java
import javax.swing.JApplet;
import javax.swing.JLabel;

public class MyApplet extends JApplet {
    @Override
    public void init() {
        getContentPane().add(new JLabel("Hello from JApplet"));
    }
}
```

#### Component Breakdown

| Component | Meaning |
|-----------|---------|
| `extends JApplet` | Declares the class as a Swing applet. |
| `init()` | The lifecycle method called by the browser/plug-in after the applet is loaded. GUI construction belongs here. |
| `getContentPane()` | Returns the content pane where visible components are added. |
| `add(Component)` | Delegates to the content pane, like `JFrame.add`. |

#### Syntax Rules

- `JApplet` uses the Applet lifecycle methods (`init`, `start`, `stop`, `destroy`) rather than a `main` method.
- GUI construction should occur in `init()`, and `setVisible` is not called directly (the browser/plug-in controls visibility).
- `JApplet` is not thread-safe; the same EDT rules apply, but the applet container’s threading model adds additional constraints.

#### Constraints and Limitations

- **Deprecated and removed.** `JApplet` is not available in modern JDKs (removed in JDK 26). It should not be used for new development.
- **No browser support.** Modern web browsers do not support the Java plug-in, making `JApplet` non-functional in contemporary environments.
- **No direct replacement.** Oracle’s documentation states: “The Applet API is deprecated, no replacement”.
- **Migration required.** Existing applet code must be migrated to standalone Swing applications, Java Web Start (historically), or other deployment technologies.

---

### Annotated Complete Step-by-Step Code Examples

Because `JApplet` is deprecated and removed, the following example is presented for **legacy understanding only**. It illustrates the historical structure and is **not** intended for new development.

#### Example 1: Historical JApplet with a Label and Button (Legacy)

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

// LEGACY CODE — DO NOT USE IN NEW PROJECTS
public class LegacyApplet extends JApplet {
    @Override
    public void init() {                          // 1. Lifecycle method
        SwingUtilities.invokeLater(() -> {        // 2. Still use EDT
            getContentPane().setLayout(new FlowLayout());
            JLabel label = new JLabel("Legacy Applet");
            JButton button = new JButton("Click");
            JLabel output = new JLabel("");

            button.addActionListener(e ->
                output.setText("Button clicked!"));

            getContentPane().add(label);
            getContentPane().add(button);
            getContentPane().add(output);
        });
    }
}
```

**Expected Output (historical).** When embedded in a browser with the Java plug-in, the applet displays a label, a button, and an output label. Clicking the button updates the output label. Modern browsers will not run this code.

**Why the code produces this result (historically).** The browser invoked `init()`, which scheduled GUI construction on the EDT. `getContentPane().add(...)` placed components into the applet’s content pane, and the `ActionListener` updated the output label when the button was clicked.

---

#### Example 2: Migrating JApplet Structure to JFrame

```java
import javax.swing.*;
import java.awt.*;

// MIGRATION TARGET: standalone JFrame replacement
public class MigratedApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Migrated Applet");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JLabel label = new JLabel("Migrated Applet");
            JButton button = new JButton("Click");
            JLabel output = new JLabel("");

            button.addActionListener(e ->
                output.setText("Button clicked!"));

            frame.add(label);
            frame.add(button);
            frame.add(output);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A standalone window appears with the same components and behavior as the historical applet. Clicking the button updates the output label.

**Why the code produces this result.** The `JApplet` lifecycle is replaced by a `main` method that schedules GUI construction on the EDT. The content pane structure is preserved (`frame.add` delegates to the content pane), and the same components and listeners are used. This demonstrates that most `JApplet` GUI code can be mechanically migrated to `JFrame`.

---

### Real-World Cases

- **Legacy enterprise intranet applications.** Some organizations still maintain internal applet-based tools that must be migrated to standalone applications before the Applet API is removed from their JDK.
- **Educational applets.** Many older Java tutorials and demonstrations were written as applets; understanding `JApplet` helps when reading historical code.
- **Migration audits.** Development teams auditing their codebases for deprecated APIs need to identify and replace `JApplet` usage.

---

## 5. Top-Level Container Best Practices

### Definitions

**Core Definition.** Best practices for top-level containers are the conventions and patterns that ensure Swing applications are thread-safe, resource-efficient, visually consistent, and maintainable from the moment the first window is created.

**Technical Definition.** Best practices include: constructing and showing all top-level containers on the EDT via `SwingUtilities.invokeLater`; choosing the correct container type for each window role; setting explicit close operations; managing ownership relationships; avoiding mixing heavyweight and lightweight components improperly; and disposing of unneeded windows to release native resources.

**Beginner-Friendly Explanation.** Best practices are the “rules of the road” for building windows in Swing. They help you avoid crashes, frozen interfaces, and windows that refuse to close. They include things like “always build the window on the special GUI thread,” “use a `JFrame` for your main window and a `JDialog` for pop-ups,” and “tell the window what to do when the user clicks the X button.”

---

### Purposes

- To prevent thread-related bugs such as race conditions, painting artifacts, and deadlocks.
- To ensure the correct window type is used for each role (main window, temporary dialog, splash screen, overlay).
- To manage the application lifecycle correctly through explicit close operations and ownership relationships.
- To avoid resource leaks by disposing of windows that are no longer needed.
- To maintain a responsive GUI by keeping the EDT free of long-running tasks.

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
        frame.setSize(600, 400);
        frame.setLocationRelativeTo(null);                        // 4
        frame.setVisible(true);
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| EDT transfer | `SwingUtilities.invokeLater(...)` in `main`. |
| Explicit close operation | `setDefaultCloseOperation(EXIT_ON_CLOSE)` for the main frame; `DISPOSE_ON_CLOSE` for dialogs and secondary windows. |
| Correct container type | `JFrame` for the main window; `JDialog` for pop-ups; `JWindow` for splash screens. |
| Ownership | Pass the owner frame to `JDialog` and `JWindow` constructors so that dependent windows follow the owner’s lifecycle. |
| Dispose unneeded windows | Call `dispose()` on dialogs and `JWindow` instances when they are no longer needed. |
| No long tasks on EDT | Use `SwingWorker` or background threads for I/O, computation, and network access. |
| Avoid `JApplet` | Migrate legacy applet code to `JFrame`-based standalone applications. |

#### Syntax Rules

- **Never construct a top-level container on the main thread.** Always use `invokeLater`.
- **Always set a close operation** on `JFrame` and `JDialog`; never rely on the default `HIDE_ON_CLOSE` for the main window.
- **Use `DISPOSE_ON_CLOSE` for dialogs** to release native resources; use `EXIT_ON_CLOSE` only for the main frame when the application should terminate.
- **Do not mix heavyweight and lightweight components improperly.** Top-level containers are heavyweight; their children should be lightweight Swing components. Mixing AWT heavyweight components inside Swing containers can cause z-order and painting problems.
- **Dispose windows you no longer need.** A `JWindow` splash screen, for example, should be disposed after the main frame is ready.
- **Avoid creating multiple `JFrame` instances for what should be a single-window application.** Use `CardLayout` or `JDialog` instead.

#### Constraints and Limitations

- `invokeLater` does not guarantee immediate execution; it only guarantees eventual execution on the EDT.
- Modal dialogs can cause deadlocks if `setVisible(true)` is called from the wrong thread.
- `JWindow` has no built-in close mechanism; the programmer must provide one.
- `JApplet` is removed from modern JDKs; best-practice code should not include it.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Main Frame with a Modal Dialog and a Splash Screen

```java
import javax.swing.*;
import java.awt.*;

public class TopLevelBestPractice {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(TopLevelBestPractice::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        // 1. Splash screen as a JWindow
        JWindow splash = new JWindow();
        splash.setSize(300, 150);
        splash.setLocationRelativeTo(null);
        JLabel splashLabel = new JLabel("Starting...", SwingConstants.CENTER);
        splashLabel.setOpaque(true);
        splashLabel.setBackground(Color.LIGHT_GRAY);
        splash.add(splashLabel);
        splash.setVisible(true);

        // 2. Simulate initialization delay
        new Timer(1000, e -> {
            splash.dispose();                                    // 3. Dispose
            showMainFrame();
        }).start();
    }

    private static void showMainFrame() {
        JFrame frame = new JFrame("Main Application");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);   // 4
        frame.setSize(500, 350);
        frame.setLocationRelativeTo(null);

        JButton dialogButton = new JButton("Open Settings");
        dialogButton.addActionListener(e -> {
            JDialog dialog = new JDialog(frame, "Settings", true); // 5
            dialog.setDefaultCloseOperation(JDialog.DISPOSE_ON_CLOSE);
            dialog.add(new JLabel("Settings go here.", SwingConstants.CENTER));
            dialog.setSize(250, 150);
            dialog.setLocationRelativeTo(frame);
            dialog.setVisible(true);
        });

        frame.add(dialogButton);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A splash `JWindow` appears for one second, then is disposed. The main `JFrame` appears with an “Open Settings” button. Clicking the button opens a modal `JDialog`. Closing the dialog disposes it; closing the main frame exits the JVM.

**Why the code produces this result.** The splash screen is a `JWindow` that is explicitly disposed after initialization. The main frame uses `EXIT_ON_CLOSE` and owns a modal `JDialog` with `DISPOSE_ON_CLOSE`. The `Timer` ensures that the splash is dismissed before the main frame is shown, and all GUI work occurs on the EDT.

---

#### Example 2: Common Mistake — Mixing Window Types Incorrectly

```java
import javax.swing.*;

public class BadWindowUsage {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // WRONG: Creating multiple JFrames for pop-up content
            JFrame popup = new JFrame("Popup");             // 1
            popup.setDefaultCloseOperation(JFrame.HIDE_ON_CLOSE);
            popup.add(new JLabel("This should be a dialog."));
            popup.setSize(200, 100);
            popup.setVisible(true);

            // WRONG: Forgetting to set close operation on main frame
            JFrame main = new JFrame("Main");                // 2
            main.setSize(400, 200);
            main.setVisible(true);
        });
    }
}
```

**Expected Output.** Two `JFrame` windows appear. Closing the “Popup” frame hides it but does not dispose it, leaking resources. Closing the “Main” frame also hides it (the default `HIDE_ON_CLOSE`), leaving the JVM running invisibly.

**Why this is wrong.** A pop-up should use `JDialog` (or `JOptionPane`), not a second `JFrame`. The main frame lacks an explicit close operation, so the application does not terminate when the user expects it to. Correct usage: use `JDialog` for the pop-up with `DISPOSE_ON_CLOSE`, and set `EXIT_ON_CLOSE` on the main frame.

---

### Real-World Cases

- **Enterprise desktop applications.** Large Swing applications use one `JFrame` as the main window, multiple `JDialog` instances for settings and confirmations, and occasionally a `JWindow` for splash screens.
- **IDE and editor plugins.** Swing-based panels embedded in IDEs often follow the same containment hierarchy patterns, though the top-level container may be provided by the host application.
- **Educational tools.** Teaching examples that follow best practices demonstrate the correct way to structure a Swing application from the first window onward.

---

## References

Oracle. *Using Top-Level Containers*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/toplevel.html.

Oracle. *How to Make Dialogs*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/components/dialog.html.

Oracle. *Class JWindow*. Java Platform, Standard Edition 25 API Specification. https://docs.oracle.com/javase/jp/25/docs/api/java.desktop/javax/swing/JWindow.html.

Oracle. *Class JApplet (Java SE 9 & JDK 9)*. https://docs.oracle.com/javase/9/docs/api/javax/swing/JApplet.html.

Oracle. *Class JApplet (Java SE 17 API Specification)*. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/javax/swing/JApplet.html.

Oracle. *Migrating Java Applets to Java Web Start and JNLP*. https://docs.oracle.com/javase/tutorial/deployment/applet/migration.html.

Oracle. *Lesson: Using Swing Components*. https://docs.oracle.com/javase/tutorial/uiswing/components/index.html.

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/.

JEP 504: Remove the Applet API. OpenJDK. https://openjdk.org/jeps/504.