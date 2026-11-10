# Look and Feel Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Look and Feel (L&F)** is the combination of the **appearance** (“look”) of Swing components and the **behaviour** (“feel”) of those components. The “look” refers to the visual appearance of GUI widgets—their colours, fonts, borders, and rendering—while the “feel” refers to how the widgets behave—how they respond to clicks, keyboard input, and state changes. **Pluggable Look and Feel** is Swing’s architectural capability that allows an application to replace its entire visual and behavioural style at runtime without changing application code.

**Technical Definition.** Look and Feel in Swing is implemented through the **separable model architecture**, which splits every Swing component into two distinct classes: a **`JComponent` subclass** (the component itself) and a corresponding **`ComponentUI` subclass** (the UI delegate). The `ComponentUI` subclass is referred to by various names in Swing’s documentation—“the UI,” “component UI,” “UI delegate,” and “look and feel delegate”. All painting in `JComponent` subclasses is delegated to the UI delegate, so the “look” can vary depending upon the installed L&F. The **`UIManager`** class manages the current look and feel, the set of available look and feels, `PropertyChangeListener`s that are notified when the look and feel changes, the look and feel defaults, and convenience methods for obtaining various default values. The **`LookAndFeel`** abstract class is the root of all L&F implementations; it provides the L&F-specific defaults table through `getDefaults()`, identifies itself through `getName()` and `getDescription()`, and reports platform support through `isNativeLookAndFeel()` and `isSupportedLookAndFeel()`.

**Beginner-Friendly Explanation.** A **Look and Feel** is the **“skin and personality”** of a Swing application. The **look** is what you see—the colours, the shape of buttons, the font used in labels, the style of scroll bars. The **feel** is how things behave—what happens when you click a button, how a combo box opens, how focus moves between fields. **Pluggable** means you can **swap** the entire skin and personality without rewriting your application. Swing ships with several L&Fs (Metal, Nimbus, Windows, GTK+, Motif) and lets you write your own. Think of it as choosing between different themes for the same application: the buttons still work, the labels still show text, but the whole interface looks and behaves differently. The **`UIManager`** is the **“theme manager”** that decides which theme is active and supplies the default colours, fonts, and sizes to every component.

---

### Key Characteristics

1. **Separation of appearance and logic.** Swing components delegate all painting and much of their behaviour to a `ComponentUI` delegate, so the same `JButton` can look like a Windows button, a Metal button, or a Nimbus button without changing its API.
2. **Runtime replaceability.** The look and feel can be changed programmatically, via the command line, or via a properties file, even after the GUI is visible.
3. **Pluggable architecture.** Any L&F that implements the `LookAndFeel` abstract class and provides the required UI delegates can be installed, including third-party L&Fs distributed as JAR files.
4. **Centralised defaults.** `UIManager` maintains three levels of defaults—developer defaults, look and feel defaults, and system defaults—resolved in that order.
5. **Component-specific delegates.** Each component type has its own UI delegate (e.g., `ButtonUI`, `LabelUI`, `TableUI`), and each L&F provides concrete implementations of these delegates.
6. **Initialisation-order sensitivity.** The L&F should be set as the very first step of the application; otherwise, Swing components may be initialised with the default L&F before the desired L&F is installed.
7. **Thread rule.** All L&F installation and GUI construction should occur on the Event Dispatch Thread (EDT), typically via `SwingUtilities.invokeLater`.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and exception handling (`try`/`catch`).
- **Basic Swing:** `JFrame`, `JPanel`, `JButton`, `JLabel`, and the `SwingUtilities.invokeLater` pattern.
- **The EDT and Swing’s single-thread rule.**
- **Basic understanding of `UIManager` and `LookAndFeel` class names.**
- **A JDK (version 8 or later).** Nimbus is available from JDK 7 onward.

---

### Related Programming Areas

- **`javax.swing.UIManager`** — manages the current L&F and its defaults.
- **`javax.swing.LookAndFeel`** — the abstract base class for all L&Fs.
- **`javax.swing.plaf.ComponentUI`** — the base class for UI delegates.
- **`javax.swing.plaf.basic`** — the Basic L&F, which provides the foundation for many other L&Fs.
- **`javax.swing.plaf.metal`** — the Metal (CrossPlatform) L&F, including the Ocean theme.
- **`javax.swing.plaf.nimbus`** — the Nimbus L&F (since JDK 7), built on Synth.
- **`javax.swing.plaf.synth`** — the Synth L&F, configurable via XML files.
- **`javax.swing.plaf.multi`** — the Multiplexing L&F, which delegates to multiple L&Fs simultaneously.
- **`SwingUtilities.updateComponentTreeUI`** — updates a component hierarchy after an L&F change.
- **`UIDefaults`** — the table of default values (fonts, colours, icons, insets) used by components.

---

## 1. Look and Feel (L&F) — The Concept

### Definitions

**Core Definition.** Look and Feel is the combined visual appearance and behavioural style of a Swing application’s user interface.

**Technical Definition.** The L&F determines the `ComponentUI` delegate used by each `JComponent`, the default values in `UIDefaults`, and the rendering and event-handling behaviour of every widget. It is selected by `UIManager` and can be changed at runtime.

**Beginner-Friendly Explanation.** The L&F is the **overall theme** of your application: how it looks and how it behaves.

---

### Purposes

- To provide a consistent visual identity across all components.
- To adapt the application to the host platform’s native appearance (Windows, macOS, Linux).
- To allow developers to choose between cross-platform consistency and platform fidelity.
- To enable accessibility and behavioural customisation through alternative L&Fs.
- To support branding and theming.

---

### Syntax Rules and Structure

#### Complete General Syntax (Conceptual)

```java
// The L&F is installed via UIManager before creating the GUI:
UIManager.setLookAndFeel(lookAndFeelClassName);
// Then components are created:
JFrame frame = new JFrame("App");
JButton button = new JButton("Click");
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `lookAndFeelClassName` | The fully qualified class name of a `LookAndFeel` implementation. |
| `UIManager.setLookAndFeel` | Installs the L&F. |
| `JComponent` creation | Components query `UIManager` for their UI delegate. |

#### Syntax Rules

- The L&F must be set before components are created, or `updateComponentTreeUI` must be called afterwards.
- The L&F affects all components created after installation.

#### Constraints and Limitations

- Some L&Fs are platform-specific (Windows, GTK+, Aqua).
- Changing the L&F at runtime requires updating existing components.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Observing L&F Differences

```java
import javax.swing.*;
import java.awt.*;

public class LookAndFeelConcept {
    public static void main(String[] args) {
        // 1. The L&F determines how this button will look and behave.
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("L&F Concept");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Same Button, Different L&F"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button whose appearance depends on the installed L&F (default is Metal/Ocean if no L&F is set).

**Why the code produces this result.** The `JButton` queries `UIManager` for its UI delegate at construction time; the delegate paints the button according to the current L&F.

---

### Real-World Cases

- **Cross-platform desktop applications** choosing Metal/Nimbus for a consistent look.
- **Platform-native applications** choosing the System L&F for a familiar feel.
- **Branded applications** using a custom Synth L&F to match corporate identity.

---

## 2. Pluggable Look and Feel — The Architecture

### Definitions

**Core Definition.** Pluggable Look and Feel is Swing’s architecture that separates each component into a `JComponent` subclass and a corresponding `ComponentUI` delegate, allowing the delegate to be replaced to change the component’s appearance and behaviour.

**Technical Definition.** Swing’s architecture enables multiple L&Fs by separating every component into two distinct classes: a `JComponent` subclass and a corresponding `ComponentUI` subclass. For example, every `JList` instance has a concrete implementation of `ListUI` (`ListUI` extends `ComponentUI`). The `ComponentUI` subclass is referred to as “the UI,” “component UI,” “UI delegate,” or “look and feel delegate”. All painting in `JComponent` subclasses is delegated to the UI delegate, so the “look” can vary depending upon the L&F. Each L&F is responsible for providing a concrete implementation for each of the `ComponentUI` subclasses defined by Swing.

**Beginner-Friendly Explanation.** Pluggable L&F means that **every component has a “painter” and “behaviour handler” that can be swapped out**. The button itself is a thin shell; the UI delegate does the actual drawing and event handling. Different L&Fs provide different delegates.

---

### Purposes

- To allow the same component API to produce different appearances and behaviours.
- To enable runtime L&F changes without changing application code.
- To support platform-native and cross-platform L&Fs.
- To enable third-party and custom L&Fs.
- To keep component logic separate from rendering and input handling.

---

### Syntax Rules and Structure

#### Complete General Syntax (Conceptual Architecture)

```java
// Component delegates to its UI delegate:
public class JButton extends AbstractButton {
    // Painting and behaviour are delegated to ButtonUI.
}

// L&F provides the delegate:
public class MetalButtonUI extends BasicButtonUI { ... }
```

#### Component Breakdown

| Layer | Responsibility |
|-------|----------------|
| `JComponent` | Component API, model, listeners, layout. |
| `ComponentUI` | Painting, sizing, event handling for the component. |
| `LookAndFeel` | Supplies the defaults and the UI delegates. |
| `UIManager` | Installs the L&F and resolves defaults. |

#### Syntax Rules

- Each component type has a UI delegate class (e.g., `ButtonUI`, `LabelUI`).
- The L&F provides concrete subclasses of these delegates.
- The component asks `UIManager` for its delegate at construction time.

#### Constraints and Limitations

- UI delegates are L&F-specific; replacing a delegate directly is fragile and should generally be done through an L&F.
- Custom delegates must implement the full `ComponentUI` contract.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Inspecting the UI Delegate

```java
import javax.swing.*;
import java.awt.*;

public class UIDelegateDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JButton button = new JButton("Inspect Me");              // 1. Create
            System.out.println("UI delegate: " + button.getUI().getClass().getName()); // 2. Inspect

            JFrame frame = new JFrame("UI Delegate");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints the class name of the current `ButtonUI` delegate (e.g., `javax.swing.plaf.metal.MetalButtonUI`).

**Why the code produces this result.** `JButton.getUI()` returns the installed `ButtonUI` delegate, which is chosen by the current L&F.

---

### Real-World Cases

- **Custom L&F development:** creating new `ComponentUI` subclasses.
- **Synth L&F:** configuring delegates via XML rather than code.
- **Multiplexing L&F:** delegating to multiple L&Fs for augmented behaviour.

---

## 3. UIManager — The L&F Manager

### Definitions

**Core Definition.** `UIManager` is the central class that manages the current look and feel, the set of available look and feels, property-change listeners, look and feel defaults, and convenience methods for obtaining default values.

**Technical Definition.** `public class UIManager extends Object implements Serializable`. The look and feel can be specified in two distinct ways: by specifying the fully qualified class name of the L&F class, or by creating an instance of `LookAndFeel` and passing it to `setLookAndFeel`. Once the look and feel has been changed, it is imperative to invoke `updateUI` on all `JComponent`s; `SwingUtilities.updateComponentTreeUI(Component)` makes this easy. `UIManager` manages three sets of `UIDefaults`: developer defaults, look and feel defaults, and system defaults, resolved in that order.

**Beginner-Friendly Explanation.** `UIManager` is the **theme manager**. It knows which L&F is active, which L&Fs are available, and what the default fonts, colours, and sizes are. You call its static methods to install an L&F or to change defaults.

---

### Purposes

- To install and manage the current look and feel.
- To enumerate and install available look and feels.
- To provide default values (fonts, colours, icons, insets) to components.
- To allow developers to override defaults at three levels.
- To notify listeners when the L&F changes.

---

### Syntax Rules and Structure

#### Complete General Syntax (Key Methods)

```java
// Set the L&F by class name:
UIManager.setLookAndFeel(String className);

// Set the L&F by instance:
UIManager.setLookAndFeel(LookAndFeel lookAndFeel);

// Get the system L&F class name:
String systemLaf = UIManager.getSystemLookAndFeelClassName();

// Get the cross-platform L&F class name:
String crossLaf = UIManager.getCrossPlatformLookAndFeelClassName();

// Get all installed L&Fs:
UIManager.LookAndFeelInfo[] infos = UIManager.getInstalledLookAndFeels();

// Get the current L&F:
LookAndFeel current = UIManager.getLookAndFeel();

// Get the look and feel defaults:
UIDefaults lafDefaults = UIManager.getLookAndFeelDefaults();

// Put a developer default:
UIManager.put("Button.font", new Font("SansSerif", Font.BOLD, 14));
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `setLookAndFeel(String)` | Installs the L&F identified by the class name. |
| `setLookAndFeel(LookAndFeel)` | Installs the given L&F instance. |
| `getSystemLookAndFeelClassName()` | Returns the class name of the platform-native L&F. |
| `getCrossPlatformLookAndFeelClassName()` | Returns the class name of the cross-platform L&F (Metal). |
| `getInstalledLookAndFeels()` | Returns an array of `LookAndFeelInfo` describing available L&Fs. |
| `getLookAndFeel()` | Returns the current `LookAndFeel`. |
| `getLookAndFeelDefaults()` | Returns the L&F-specific defaults table. |
| `put(Object key, Object value)` | Adds or overrides a developer default. |

#### Syntax Rules

- `setLookAndFeel` throws `UnsupportedLookAndFeelException`, `ClassNotFoundException`, `InstantiationException`, or `IllegalAccessException`.
- After changing the L&F, call `SwingUtilities.updateComponentTreeUI(...)` on each top-level container.
- Developer defaults set via `UIManager.put` take precedence over L&F and system defaults.

#### Constraints and Limitations

- Changing the L&F after components are created without updating the component tree produces unspecified behaviour, including painting problems and exceptions.
- Some L&Fs (Windows, GTK+, Aqua) are platform-specific.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Listing Installed Look and Feels

```java
import javax.swing.*;
import java.awt.*;

public class ListLookAndFeels {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            UIManager.LookAndFeelInfo[] infos =                         // 1. Get all
                UIManager.getInstalledLookAndFeels();

            JTextArea area = new JTextArea(10, 40);
            area.setEditable(false);
            StringBuilder sb = new StringBuilder();
            for (UIManager.LookAndFeelInfo info : infos) {              // 2. Iterate
                sb.append(info.getName())
                  .append(" -> ")
                  .append(info.getClassName())
                  .append("\n");
            }
            area.setText(sb.toString());

            JFrame frame = new JFrame("Installed L&Fs");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JScrollPane(area));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A text area listing each installed L&F’s display name and fully qualified class name (e.g., “Metal -> javax.swing.plaf.metal.MetalLookAndFeel”).

**Why the code produces this result.** `UIManager.getInstalledLookAndFeels()` returns metadata about each installed L&F, including its name and class name.

---

#### Example 2: Setting the System Look and Feel

```java
import javax.swing.*;

public class SystemLafDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 1. Set system
                    UIManager.getSystemLookAndFeelClassName());
            } catch (Exception e) {
                e.printStackTrace();
            }
            JFrame frame = new JFrame("System L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Native Look"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window whose button looks like a native platform button (Windows, GTK+, or Aqua, depending on the OS).

**Why the code produces this result.** `getSystemLookAndFeelClassName()` returns the platform-specific L&F class name, and `setLookAndFeel` installs it.

---

### Real-World Cases

- **Applications that want native appearance** use `getSystemLookAndFeelClassName`.
- **Applications that want consistent cross-platform appearance** use `getCrossPlatformLookAndFeelClassName`.
- **L&F selection dialogs** use `getInstalledLookAndFeels` to populate a combo box.

---

## 4. Available Look and Feels

### Definitions

**Core Definition.** Available Look and Feels are the L&F implementations supplied with the Java platform (and by vendors) that can be installed in a Swing application.

**Technical Definition.** The Java platform provides the following L&Fs: **CrossPlatformLookAndFeel** (also called Metal, the “Java L&F”), **SystemLookAndFeel** (the platform-native L&F), **Synth** (an XML-configurable L&F), **Multiplexing** (delegates to multiple L&Fs simultaneously), and platform-specific L&Fs such as **Windows**, **GTK+**, **Motif**, and **Aqua** (on macOS). The GTK+ L&F runs only on UNIX/Linux with GTK+ 2.2 or later; the Windows L&F runs only on Windows; the Motif and Metal L&Fs run on any platform.

**Beginner-Friendly Explanation.** Java ships with several ready-made themes. **Metal** is the classic cross-platform theme. **Nimbus** is a modern cross-platform theme introduced in JDK 7. **Windows**, **GTK+**, and **Aqua** match the native platform. **Motif** is an older cross-platform theme. **Synth** lets you define your own theme in XML.

---

### Purposes

- To provide ready-to-use L&Fs for common scenarios.
- To allow platform-native appearance.
- To provide a consistent cross-platform appearance.
- To serve as a base for custom L&Fs (Basic, Synth).
- To support accessibility and specialised rendering.

---

### Syntax Rules and Structure

#### Complete General Syntax (Class Names of Common L&Fs)

```java
// Cross-platform (Metal):
UIManager.setLookAndFeel("javax.swing.plaf.metal.MetalLookAndFeel");

// Nimbus (JDK 7+):
UIManager.setLookAndFeel("javax.swing.plaf.nimbus.NimbusLookAndFeel");

// System L&F:
UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());

// Windows (Windows only):
UIManager.setLookAndFeel("com.sun.java.swing.plaf.windows.WindowsLookAndFeel");

// Motif (any platform):
UIManager.setLookAndFeel("com.sun.java.swing.plaf.motif.MotifLookAndFeel");

// GTK+ (Linux/UNIX with GTK+ 2.2+):
UIManager.setLookAndFeel("com.sun.java.swing.plaf.gtk.GTKLookAndFeel");
```

#### Component Breakdown

| L&F | Class Name | Platform |
|-----|------------|----------|
| Metal (CrossPlatform) | `javax.swing.plaf.metal.MetalLookAndFeel` | Any |
| Nimbus | `javax.swing.plaf.nimbus.NimbusLookAndFeel` | Any (JDK 7+) |
| System | `UIManager.getSystemLookAndFeelClassName()` | Current platform |
| Windows | `com.sun.java.swing.plaf.windows.WindowsLookAndFeel` | Windows |
| Motif | `com.sun.java.swing.plaf.motif.MotifLookAndFeel` | Any |
| GTK+ | `com.sun.java.swing.plaf.gtk.GTKLookAndFeel` | Linux/UNIX |
| Synth | `javax.swing.plaf.synth.SynthLookAndFeel` | Any (XML-configured) |
| Multiplexing | `javax.swing.plaf.multi.MultiLookAndFeel` | Any |

#### Syntax Rules

- Platform-specific L&Fs fall back to Metal if installed on an unsupported platform.
- External L&Fs must be on the classpath and are used like built-in ones.

#### Constraints and Limitations

- Windows L&F only runs on Windows.
- GTK+ L&F only runs on UNIX/Linux with GTK+ 2.2+.
- Nimbus is built on Synth and supports customisation via `UIDefaults`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Switching Between Available L&Fs

```java
import javax.swing.*;
import java.awt.*;

public class SwitchLafDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Switch L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JComboBox<String> combo = new JComboBox<>(new String[]{
                "Metal", "Nimbus", "System", "Motif"
            });
            frame.add(combo);
            frame.add(new JButton("Sample"));
            frame.add(new JLabel("Sample Label"));

            combo.addActionListener(e -> {
                String choice = (String) combo.getSelectedItem();
                try {
                    switch (choice) {
                        case "Metal":
                            UIManager.setLookAndFeel(
                                "javax.swing.plaf.metal.MetalLookAndFeel");
                            break;
                        case "Nimbus":
                            UIManager.setLookAndFeel(
                                "javax.swing.plaf.nimbus.NimbusLookAndFeel");
                            break;
                        case "System":
                            UIManager.setLookAndFeel(
                                UIManager.getSystemLookAndFeelClassName());
                            break;
                        case "Motif":
                            UIManager.setLookAndFeel(
                                "com.sun.java.swing.plaf.motif.MotifLookAndFeel");
                            break;
                    }
                    SwingUtilities.updateComponentTreeUI(frame);     // 1. Update
                    frame.pack();                                      // 2. Re-pack
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            });

            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a combo box, a button, and a label. Selecting a different L&F immediately changes the appearance of the entire window.

**Why the code produces this result.** `UIManager.setLookAndFeel` installs the new L&F, and `SwingUtilities.updateComponentTreeUI(frame)` applies the new delegates to every component in the frame.

---

### Real-World Cases

- **Cross-platform applications** defaulting to Nimbus or Metal.
- **Windows-only applications** defaulting to the Windows L&F.
- **Linux applications** defaulting to GTK+.

---

## 5. Setting the Look and Feel

### Definitions

**Core Definition.** Setting the Look and Feel is the process of installing a chosen L&F before or during application startup.

**Technical Definition.** The L&F can be set programmatically via `UIManager.setLookAndFeel`, via the `swing.defaultlaf` system property on the command line, or via the `swing.properties` file. The L&F should be set as the **very first step** in the application; otherwise, components may be initialised with the default L&F before the requested L&F is installed.

**Beginner-Friendly Explanation.** You can set the L&F in **three ways**: in code, on the command line, or in a properties file. Set it **before** creating any Swing components.

---

### Purposes

- To select the application’s visual style.
- To match the host platform’s native appearance.
- To enforce a consistent cross-platform appearance.
- To allow end users or administrators to override the L&F without code changes.

---

### Syntax Rules and Structure

#### Complete General Syntax (Three Approaches)

**1. Programmatic:**

```java
public static void main(String[] args) {
    try {
        UIManager.setLookAndFeel(
            UIManager.getSystemLookAndFeelClassName());
    } catch (Exception e) { e.printStackTrace(); }
    SwingUtilities.invokeLater(() -> createAndShowGUI());
}
```

**2. Command line:**

```bash
java -Dswing.defaultlaf=javax.swing.plaf.nimbus.NimbusLookAndFeel MyApp
```

**3. `swing.properties` file:**

```properties
swing.defaultlaf=com.sun.java.swing.plaf.windows.WindowsLookAndFeel
```

#### Component Breakdown

| Approach | When to Use |
|----------|-------------|
| Programmatic | Application controls the L&F. |
| Command line | Deployment-time override. |
| `swing.properties` | Per-installation default. |

#### Syntax Rules

- Programmatic setting must occur before component creation.
- Command line and properties file settings affect the **default** L&F used when no programmatic setting is made.
- Command line takes precedence over `swing.properties`.

#### Constraints and Limitations

- Static field initialisation of Swing classes can trigger premature L&F loading; set the L&F before any Swing class is touched.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Programmatic Setting with `invokeLater`

```java
import javax.swing.*;

public class ProgrammaticLaf {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. First
                UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            e.printStackTrace();
        }
        SwingUtilities.invokeLater(() -> {                            // 2. Then GUI
            JFrame frame = new JFrame("Programmatic L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Native Look"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with the system L&F applied.

---

#### Example 2: Command-Line Setting

```bash
java -Dswing.defaultlaf=javax.swing.plaf.nimbus.NimbusLookAndFeel MyApp
```

**Expected Output.** The application starts with the Nimbus L&F, unless the application programmatically overrides it.

---

### Real-World Cases

- **Enterprise deployments** use `swing.properties` to enforce a corporate L&F.
- **User-selectable themes** use programmatic setting with a preferences dialog.
- **Testing** uses command-line overrides to compare L&Fs.

---

## 6. Changing the Look and Feel at Runtime

### Definitions

**Core Definition.** Runtime L&F changing is the process of installing a different L&F while the application is running and updating existing components to reflect the new style.

**Technical Definition.** `UIManager.setLookAndFeel` can be called after the GUI is visible. To make existing components reflect the new L&F, invoke `SwingUtilities.updateComponentTreeUI` once per top-level container. Without this call, the behaviour is unspecified and may produce exceptions or painting problems.

**Beginner-Friendly Explanation.** You can **switch themes while the application is running**, but after switching you must tell every window to refresh its components with `updateComponentTreeUI`.

---

### Purposes

- To allow users to choose their preferred theme at runtime.
- To support live previews of L&F options.
- To enable accessibility or platform-adaptation features.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
try {
    UIManager.setLookAndFeel(newLafClassName);
    SwingUtilities.updateComponentTreeUI(topLevelContainer);
    topLevelContainer.pack();   // optional, if sizes changed
} catch (Exception e) {
    e.printStackTrace();
}
```

#### Component Breakdown

| Step | Meaning |
|------|---------|
| `setLookAndFeel` | Installs the new L&F. |
| `updateComponentTreeUI` | Updates all components in the hierarchy. |
| `pack()` | Re-sizes the window to fit the new L&F. |

#### Syntax Rules

- Call `updateComponentTreeUI` on **each** top-level container.
- Call `pack()` or `revalidate()` if the L&F changes component sizes.

#### Constraints and Limitations

- Custom components that cache L&F-specific resources may need manual refresh.
- Some L&Fs may not support all component types gracefully.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Runtime L&F Switcher

```java
import javax.swing.*;
import java.awt.*;

public class RuntimeSwitch {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Runtime Switch");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JComboBox<String> combo = new JComboBox<>(new String[]{
                "Metal", "Nimbus", "Motif"
            });
            JButton button = new JButton("Sample");
            JLabel label = new JLabel("Sample Label");

            combo.addActionListener(e -> {
                String name = (String) combo.getSelectedItem();
                try {
                    UIManager.setLookAndFeel(classNameFor(name));    // 1. Set
                    SwingUtilities.updateComponentTreeUI(frame);     // 2. Update
                    frame.pack();                                      // 3. Re-pack
                } catch (Exception ex) { ex.printStackTrace(); }
            });

            frame.setLayout(new FlowLayout());
            frame.add(combo);
            frame.add(button);
            frame.add(label);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }

    private static String classNameFor(String name) {
        switch (name) {
            case "Metal":  return "javax.swing.plaf.metal.MetalLookAndFeel";
            case "Nimbus": return "javax.swing.plaf.nimbus.NimbusLookAndFeel";
            case "Motif":  return "com.sun.java.swing.plaf.motif.MotifLookAndFeel";
            default:       return UIManager.getCrossPlatformLookAndFeelClassName();
        }
    }
}
```

**Expected Output.** The window immediately changes its appearance when a new L&F is selected from the combo box.

---

### Real-World Cases

- **Theme selectors** in application settings.
- **Live L&F previews** for design tools.
- **Accessibility modes** that switch to a high-contrast L&F.

---

## 7. Customizing Look and Feel Defaults

### Definitions

**Core Definition.** Customizing L&F defaults is the process of overriding the default fonts, colours, icons, insets, and other `UIDefaults` entries used by Swing components.

**Technical Definition.** `UIManager` manages three sets of `UIDefaults`: **developer defaults**, **look and feel defaults**, and **system defaults**, resolved in that order. `UIManager.put(key, value)` adds or overrides a developer default; `UIManager.getLookAndFeelDefaults()` returns the L&F-specific defaults table for direct modification.

**Beginner-Friendly Explanation.** You can **change the default font, colours, and sizes** without writing a custom L&F. Developer defaults via `UIManager.put` are the simplest way; they apply to all components that use the corresponding key.

---

### Purposes

- To apply a consistent font or colour scheme without writing a custom L&F.
- To adjust spacing, insets, and sizes globally.
- To override specific L&F defaults for branding or accessibility.
- To customise Nimbus’s painters and derived colours.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// Global font override (developer default):
UIManager.put("Button.font", new Font("SansSerif", Font.BOLD, 14));
UIManager.put("Label.font", new Font("SansSerif", Font.PLAIN, 14));

// L&F-specific override:
UIDefaults defaults = UIManager.getLookAndFeelDefaults();
defaults.put("Table.foreground", Color.DARK_GRAY);
```

#### Component Breakdown

| Key | Meaning |
|-----|---------|
| `"Button.font"` | Font used by `JButton`. |
| `"Label.font"` | Font used by `JLabel`. |
| `"Table.foreground"` | Foreground colour of `JTable`. |
| `"Panel.background"` | Background colour of `JPanel`. |

#### Syntax Rules

- Developer defaults take precedence over L&F and system defaults.
- Keys are L&F-specific; consult the L&F documentation for the exact key names.
- Changes should be made **before** components are created.

#### Constraints and Limitations

- Some L&Fs (e.g., Nimbus) may compute values dynamically and require specific keys or painters.
- Not all keys are honoured by every L&F.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Global Font Override

```java
import javax.swing.*;
import java.awt.*;

public class FontOverride {
    public static void main(String[] args) {
        UIManager.put("Button.font", new Font("Serif", Font.BOLD, 16)); // 1. Override
        UIManager.put("Label.font", new Font("Serif", Font.PLAIN, 16));

        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Font Override");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());
            frame.add(new JButton("Bold Button"));
            frame.add(new JLabel("Serif Label"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The button and label use the specified Serif font.

---

### Real-World Cases

- **Corporate branding** applies a company font globally.
- **Accessibility** increases font sizes for readability.
- **Dark mode** overrides background and foreground defaults.

---

## 8. Look and Feel Best Practices

### Definitions

**Core Definition.** Look and Feel best practices are the conventions and patterns that ensure L&F selection, installation, and customisation are correct, portable, and maintainable.

**Technical Definition.** Best practices include: setting the L&F **before** any Swing components are created; calling `SwingUtilities.updateComponentTreeUI` after runtime L&F changes; using `UIManager.getSystemLookAndFeelClassName()` for native appearance; using `UIManager.getCrossPlatformLookAndFeelClassName()` for consistency; caching and reusing L&F class names; and avoiding hard-coded L&F-specific assumptions in application code.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that prevent L&F bugs: set the theme first, update all windows when switching, use the standard class-name helpers, and don’t assume a particular L&F’s look.

---

### Purposes

- To avoid initialising components with the wrong L&F.
- To prevent painting problems and exceptions after L&F changes.
- To ensure portability across platforms.
- To keep application code independent of any single L&F.
- To make L&F customisation maintainable.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public static void main(String[] args) {
    // 1. Set L&F FIRST, before any Swing class is touched.
    try {
        UIManager.setLookAndFeel(
            UIManager.getSystemLookAndFeelClassName());
    } catch (Exception e) {
        e.printStackTrace();
    }

    // 2. Then construct and show the GUI on the EDT.
    SwingUtilities.invokeLater(() -> createAndShowGUI());
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Set L&F first | Before any `JComponent` is created. |
| Update after runtime change | `SwingUtilities.updateComponentTreeUI(frame)`. |
| Prefer standard class-name helpers | `getSystemLookAndFeelClassName`, `getCrossPlatformLookAndFeelClassName`. |
| Avoid L&F-specific assumptions | Do not hard-code colours, sizes, or class names. |
| EDT | `SwingUtilities.invokeLater` for GUI construction. |
| Test across L&Fs | Verify layout and behaviour with Metal, Nimbus, and System L&Fs. |

#### Syntax Rules

- **Always set the L&F before creating components.**
- **Always call `updateComponentTreeUI` after changing the L&F at runtime.**
- **Always handle the checked exceptions** from `setLookAndFeel`.
- **Never rely on a specific L&F’s default colours or fonts** in layout calculations.

#### Constraints and Limitations

- Some L&Fs are platform-specific; test on each target platform.
- Custom L&Fs may not support all components or may have performance implications.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Initialisation

```java
import javax.swing.*;

public class BestPracticeLaf {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. First
                UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            System.err.println("Could not set L&F: " + e.getMessage());
        }
        SwingUtilities.invokeLater(BestPracticeLaf::createAndShowGUI); // 2. EDT
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Best Practice L&F");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.add(new JButton("Click Me"));
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window using the system L&F, constructed entirely after the L&F is installed.

---

#### Example 2: Common Mistake — Setting L&F After Creating Components

```java
public static void main(String[] args) {
    JFrame frame = new JFrame("Wrong Order");                        // WRONG
    try {
        UIManager.setLookAndFeel(
            "javax.swing.plaf.nimbus.NimbusLookAndFeel");            // Too late!
    } catch (Exception e) { e.printStackTrace(); }
    frame.add(new JButton("Click"));
    frame.setVisible(true);
}
```

**Expected Output.** The frame and button may retain the previous L&F’s appearance, or may exhibit painting problems.

**Why this is wrong.** Components created before the L&F is installed use the old L&F’s delegates. The L&F must be set first.

---

#### Example 3: Common Mistake — Forgetting `updateComponentTreeUI`

```java
UIManager.setLookAndFeel("javax.swing.plaf.nimbus.NimbusLookAndFeel");
// Missing: SwingUtilities.updateComponentTreeUI(frame);
```

**Expected Output.** Existing components may keep their old appearance or produce unspecified behaviour, including exceptions.

---

### Real-World Cases

- **Production applications** set the L&F in `main` before any Swing class is touched.
- **Settings dialogs** update the component tree after an L&F change.
- **Cross-platform testing** verifies layout with Metal, Nimbus, and System L&Fs.

---

## References

Oracle. *Class UIManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *Class UIManager*. Java Development Kit Version 23 API Specification. https://docs.oracle.com/en/java/javase/23/docs/api/java.desktop/javax/swing/UIManager.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Lesson: Modifying the Look and Feel*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/lookandfeel/

Oracle. *Class LookAndFeel*. Java Platform, Standard Edition 8 API Specification. https://resources.mpi-inf.mpg.de/d5/teaching/ss05/is05/java/GoToJava2.2/html/k100236.html

Oracle. *Class LookAndFeel*. Java 2 Platform Standard Edition v1.3 API Specification. https://resources.mpi-inf.mpg.de/d5/teaching/ss05/is05/java/GoToJava2.2/html/k100236.html

Oracle. *Package javax.swing.plaf.nimbus*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/nimbus/package-summary.html

Oracle. *Synth File Format*. Java Platform, Standard Edition Documentation. https://docs.oracle.com/en/java/javase/...

Oracle. *Class UIManager — getInstalledLookAndFeels*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html#getInstalledLookAndFeels--

Oracle. *Class UIManager — getCrossPlatformLookAndFeelClassName*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html#getCrossPlatformLookAndFeelClassName--

Oracle. *Class UIManager — getLookAndFeelDefaults and put*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *A Swing Architecture Overview*. https://www.oracle.com/technetwork/java/architecture-142923.html

Oracle. *Class SwingUtilities — updateComponentTreeUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html#updateComponentTreeUI-java.awt.Component-

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/