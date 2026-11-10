# Setting Look and Feel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Setting Look and Feel** is the process of installing a chosen `LookAndFeel` implementation on a running Swing application via the `UIManager` class. It determines the visual appearance and behavioural style of every Swing component created after installation, and it can be changed at runtime by installing a new L&F and updating existing component hierarchies.

**Technical Definition.** `public class UIManager` (part of `javax.swing`) is the central manager for Swing's Look and Feel infrastructure. The primary installation methods are `UIManager.setLookAndFeel(String className)` and `UIManager.setLookAndFeel(LookAndFeel lookAndFeel)`. Once installed, `UIManager` uses the L&F to construct each component's `ComponentUI` delegate from the L&F's defaults table. The class also provides discovery methods (`getInstalledLookAndFeels()`, `getSystemLookAndFeelClassName()`, `getCrossPlatformLookAndFeelClassName()`) that make portable L&F selection possible without hard-coding class names. `setLookAndFeel` throws `UnsupportedLookAndFeelException`, `ClassNotFoundException`, `InstantiationException`, and `IllegalAccessException`. All installation and GUI construction must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** **Setting the Look and Feel** is how you tell your Swing application which theme to use. You call one of `UIManager`'s `setLookAndFeel` methods before creating any Swing components, and from that point on, every button, label, table, and dialog uses the chosen theme. `UIManager` provides helpers so you don't have to know the exact class names: `getSystemLookAndFeelClassName()` returns "use the native theme," `getCrossPlatformLookAndFeelClassName()` returns "use the classic Java theme," and `getInstalledLookAndFeels()` lets you discover all available themes for a theme selector. Think of `UIManager` as the **theme installer**: you tell it which theme, and it installs it for the whole application.

---

### Key Characteristics

1. **Multiple overloads.** `setLookAndFeel` accepts either a `String` class name or a `LookAndFeel` instance.
2. **First thing, always.** The L&F must be set **before** any Swing component is created, or the components will be initialised with the previous (usually Metal) L&F.
3. **Checked exceptions.** `setLookAndFeel` throws checked exceptions and must be wrapped in `try`/`catch`.
4. **Discovery API.** `getInstalledLookAndFeels()` returns `LookAndFeelInfo` metadata; `getSystemLookAndFeelClassName()` and `getCrossPlatformLookAndFeelClassName()` return portable class-name helpers.
5. **Runtime switchability.** The L&F can be changed while the GUI is visible, provided you call `SwingUtilities.updateComponentTreeUI` on every top-level container.
6. **Precedence of settings.** Programmatic setting overrides the command-line setting (`-Dswing.defaultlaf=...`), which overrides the `swing.properties` file.
7. **EDT discipline.** All L&F installation and component construction must occur on the EDT.
8. **ComponentUI delegation.** Every component gets a UI delegate from the L&F's defaults; the delegate paints and handles input for the component.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and exception handling.
- **Basic Swing:** `JFrame`, `JButton`, `JLabel`, and `SwingUtilities.invokeLater`.
- **Understanding of `UIManager` and `LookAndFeel` classes.**
- **Knowledge of the EDT and Swing's single-thread rule.**
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`javax.swing.UIManager`** — the theme manager.
- **`javax.swing.LookAndFeel`** — the abstract base class for all L&Fs.
- **`javax.swing.plaf.ComponentUI`** — the UI delegate for each component type.
- **`SwingUtilities.updateComponentTreeUI`** — updates existing components after an L&F change.
- **`LookAndFeelInfo`** — metadata describing an installed L&F (name + class name).
- **`UIDefaults`** — the default values table (fonts, colours, icons, insets) used by components.
- **`swing.defaultlaf` system property** — sets the default L&F at startup.

---

## 1. UIManager.setLookAndFeel() — Installing a Look and Feel

### Definitions

**Core Definition.** `setLookAndFeel` installs the specified Look and Feel on the application, affecting all components created afterwards.

**Technical Definition.** `public static void setLookAndFeel(String className) throws ClassNotFoundException, InstantiationException, IllegalAccessException, UnsupportedLookAndFeelException` and `public static void setLookAndFeel(LookAndFeel lookAndFeel) throws UnsupportedLookAndFeelException`. The method delegates to `LookAndFeel.initialize()` and then makes the L&F the current one. Existing components are **not** updated automatically; the caller must invoke `SwingUtilities.updateComponentTreeUI` on each top-level container.

**Beginner-Friendly Explanation.** `setLookAndFeel` is the method you call to install a theme. You give it either the class name of the L&F or a `LookAndFeel` instance, and it becomes the active theme for the application.

---

### Purposes

- To install a chosen L&F before the GUI is created.
- To switch to a different L&F at runtime.
- To provide a platform-native or cross-platform appearance.
- To enforce a corporate theme (via a custom L&F).
- To support user-selectable themes in a settings dialog.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// By class name:
UIManager.setLookAndFeel(String className);

// By instance:
UIManager.setLookAndFeel(LookAndFeel lookAndFeel);
```

#### Component Breakdown

| Overload | Meaning |
|----------|---------|
| `setLookAndFeel(String)` | Installs the L&F identified by the fully qualified class name. |
| `setLookAndFeel(LookAndFeel)` | Installs the given L&F instance. |

#### Syntax Rules

- Both overloads throw checked exceptions; wrap in `try`/`catch` or propagate.
- Call `setLookAndFeel` **before** creating components.
- After runtime changes, call `SwingUtilities.updateComponentTreeUI`.
- All calls should occur on the EDT.

#### Constraints and Limitations

- `setLookAndFeel` does not update existing components.
- `setLookAndFeel(String)` throws `ClassNotFoundException` if the class is not on the classpath.
- `setLookAndFeel(LookAndFeel)` may still throw `UnsupportedLookAndFeelException` on unsupported platforms.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing a L&F by Class Name

```java
import javax.swing.*;

public class SetByClassName {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. Install
                "javax.swing.plaf.nimbus.NimbusLookAndFeel");
        } catch (ClassNotFoundException | InstantiationException
                | IllegalAccessException
                | UnsupportedLookAndFeelException e) {
            e.printStackTrace();
        }
        SwingUtilities.invokeLater(SetByClassName::createAndShowGUI); // 2. EDT
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Set by Class Name");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.add(new JButton("Nimbus Button"));
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window with a button rendered in the Nimbus L&F.

**Why the code produces this result.** `setLookAndFeel` installs Nimbus. Components created afterwards (on the EDT) use Nimbus's UI delegates.

---

#### Example 2: Installing a L&F by Instance

```java
import javax.swing.*;
import javax.swing.plaf.metal.MetalLookAndFeel;

public class SetByInstance {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(new MetalLookAndFeel());        // 1. Instance
        } catch (UnsupportedLookAndFeelException e) {
            e.printStackTrace();
        }
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Set by Instance");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Metal Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the Metal L&F.

**Why the code produces this result.** Passing a `LookAndFeel` instance allows configurations (e.g., a specific theme) to be set on the instance before installation.

---

#### Example 3: Runtime L&F Switch

```java
import javax.swing.*;
import java.awt.*;

public class RuntimeSwitch {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Runtime Switch");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());

            JButton button = new JButton("Sample");
            JComboBox<String> combo = new JComboBox<>(new String[]{
                "Metal", "Nimbus", "System"
            });

            combo.addActionListener(e -> {
                try {
                    String choice = (String) combo.getSelectedItem();
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
                    }
                    SwingUtilities.updateComponentTreeUI(frame);     // 1. Update
                    frame.pack();                                      // 2. Pack
                } catch (Exception ex) { ex.printStackTrace(); }
            });

            frame.add(combo);
            frame.add(button);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Selecting a different L&F from the combo box immediately changes the appearance of the window.

**Why the code produces this result.** `setLookAndFeel` installs the new L&F and `SwingUtilities.updateComponentTreeUI(frame)` applies the new UI delegates to every component in the frame.

---

#### Example 4: Common Mistake — Setting the L&F After Component Creation

```java
public static void main(String[] args) {
    JFrame frame = new JFrame("Wrong Order");                        // WRONG
    try {
        UIManager.setLookAndFeel(
            "javax.swing.plaf.nimbus.NimbusLookAndFeel");            // Too late
    } catch (Exception e) { e.printStackTrace(); }
    frame.add(new JButton("Click"));
    frame.setVisible(true);
}
```

**Expected Output.** The frame and button use the previous L&F (usually Metal), not Nimbus.

**Why this is wrong.** Components are bound to their UI delegate at construction time. Installing a new L&F afterward does not retroactively change existing components unless `updateComponentTreeUI` is called.

---

### Real-World Cases

- **Settings dialogs** that let the user pick a theme.
- **Corporate applications** installing a branded L&F at startup.
- **Cross-platform applications** installing the system L&F for native appearance.

---

## 2. UIManager.getInstalledLookAndFeels() — Discovering Available L&Fs

### Definitions

**Core Definition.** `getInstalledLookAndFeels()` returns metadata about the L&Fs currently installed in the JVM, allowing applications to discover and offer available themes.

**Technical Definition.** `public static UIManager.LookAndFeelInfo[] getInstalledLookAndFeels()`. Each `LookAndFeelInfo` object provides `getName()` (display name, e.g., "Nimbus") and `getClassName()` (fully qualified class name). The returned array lists the L&Fs that the JVM has registered as available on the current platform.

**Beginner-Friendly Explanation.** `getInstalledLookAndFeels()` is the **theme discovery method**. It tells you which themes are available, so you can build a theme selector without hard-coding class names.

---

### Purposes

- To discover available L&Fs without hard-coding class names.
- To build L&F-selection UIs (combo boxes, menus, settings dialogs).
- To check whether a specific L&F (e.g., Nimbus) is available before trying to install it.
- To fall back gracefully when a preferred L&F is unavailable.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.LookAndFeelInfo[] infos = UIManager.getInstalledLookAndFeels();
for (UIManager.LookAndFeelInfo info : infos) {
    System.out.println(info.getName() + " -> " + info.getClassName());
}
```

#### Component Breakdown

| Method | Meaning |
|--------|---------|
| `getInstalledLookAndFeels()` | Returns an array of `LookAndFeelInfo`. |
| `LookAndFeelInfo.getName()` | The display name (e.g., `"Nimbus"`). |
| `LookAndFeelInfo.getClassName()` | The fully qualified class name. |

#### Syntax Rules

- The returned array is a copy; modifying it does not affect `UIManager`.
- The list includes platform-appropriate L&Fs only.
- Use the display name for UI labels and the class name for installation.

#### Constraints and Limitations

- The specific L&Fs listed vary by JDK version and platform.
- Some L&Fs listed may still fail to install on the current platform.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Listing Installed L&Fs

```java
import javax.swing.*;

public class ListInstalledLafs {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            UIManager.LookAndFeelInfo[] infos =                       // 1. Get all
                UIManager.getInstalledLookAndFeels();

            JTextArea area = new JTextArea(10, 45);
            area.setEditable(false);
            StringBuilder sb = new StringBuilder();
            for (UIManager.LookAndFeelInfo info : infos) {            // 2. Iterate
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

**Expected Output.** A text area listing each installed L&F's display name and class name (e.g., "Metal -> javax.swing.plaf.metal.MetalLookAndFeel").

---

#### Example 2: Robust Nimbus Installation

```java
import javax.swing.*;

public class RobustNimbus {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                for (UIManager.LookAndFeelInfo info :                 // 1. Discover
                        UIManager.getInstalledLookAndFeels()) {
                    if ("Nimbus".equals(info.getName())) {            // 2. Match
                        UIManager.setLookAndFeel(info.getClassName()); // 3. Install
                        break;
                    }
                }
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Robust Nimbus");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Nimbus Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** Nimbus is installed if available; otherwise, the default L&F is used without throwing an exception.

**Why the code produces this result.** The loop searches the installed L&Fs by display name and installs the matching class. If Nimbus is not available, the loop completes without installing it and the application continues with the default L&F.

---

#### Example 3: Populating a Theme Selector

```java
import javax.swing.*;

public class ThemeSelector {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JComboBox<UIManager.LookAndFeelInfo> combo =             // 1. Typed
                new JComboBox<>(UIManager.getInstalledLookAndFeels());

            JFrame frame = new JFrame("Theme Selector");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            combo.addActionListener(e -> {
                UIManager.LookAndFeelInfo info =
                    (UIManager.LookAndFeelInfo) combo.getSelectedItem();
                try {
                    UIManager.setLookAndFeel(info.getClassName());   // 2. Install
                    SwingUtilities.updateComponentTreeUI(frame);      // 3. Update
                    frame.pack();
                } catch (Exception ex) { ex.printStackTrace(); }
            });

            frame.add(combo);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A combo box listing every installed L&F by display name; selecting one changes the window's appearance immediately.

---

### Real-World Cases

- **Theme selection dialogs** in settings windows.
- **First-run configuration** where the user picks a preferred theme.
- **Fallback logic** that installs Nimbus if available.

---

## 3. UIManager.getSystemLookAndFeelClassName() — Native L&F Class Name

### Definitions

**Core Definition.** `getSystemLookAndFeelClassName()` returns the class name of the **platform-native** L&F (or the cross-platform L&F if no native L&F is available).

**Technical Definition.** `public static String getSystemLookAndFeelClassName()`. On Windows, it returns the Windows L&F class name. On macOS, it returns the Aqua L&F class name. On Linux/UNIX, it returns the GTK+ L&F class name (or Metal if GTK+ is unavailable). The value can be overridden via the `swing.systemlaf` system property.

**Beginner-Friendly Explanation.** `getSystemLookAndFeelClassName()` tells you the **native theme** for the current platform, so you can install it without hard-coding platform-specific class names.

---

### Purposes

- To install the platform-native L&F without knowing its class name.
- To make the application look like a native application on every OS.
- To avoid hard-coding Windows/GTK+/Aqua class names.
- To support deployment across multiple platforms with the same code.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
try {
    UIManager.setLookAndFeel(
        UIManager.getSystemLookAndFeelClassName());
} catch (Exception e) {
    e.printStackTrace();
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getSystemLookAndFeelClassName()` | Returns the native L&F class name. |
| `setLookAndFeel(...)` | Installs the returned L&F. |

#### Syntax Rules

- The result varies by platform; do not hard-code assumptions about which L&F it is.
- Must still be called before component creation.
- If the native L&F cannot be installed, catch the exception and fall back to the cross-platform L&F.

#### Constraints and Limitations

- Some platforms may lack a native L&F, in which case the cross-platform L&F is returned.
- The native L&F's appearance may differ across OS versions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing the Native L&F

```java
import javax.swing.*;

public class NativeLafDemo {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. Native
                UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            System.err.println("Could not set system L&F: " + e);
        }
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Native L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Native Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the platform's native L&F.

---

#### Example 2: Fallback on Failure

```java
try {
    UIManager.setLookAndFeel(
        UIManager.getSystemLookAndFeelClassName());
} catch (Exception e) {
    try {
        UIManager.setLookAndFeel(                                // 1. Fallback
            UIManager.getCrossPlatformLookAndFeelClassName());
    } catch (Exception e2) {
        e2.printStackTrace();
    }
}
```

**Expected Output.** If the system L&F fails to install, the application falls back to the cross-platform L&F.

---

#### Example 3: Checking Which Native L&F Is Used

```java
System.out.println(UIManager.getSystemLookAndFeelClassName());
// Windows: "com.sun.java.swing.plaf.windows.WindowsLookAndFeel"
// Linux:   "com.sun.java.swing.plaf.gtk.GTKLookAndFeel"
// macOS:   "com.apple.laf.AquaLookAndFeel"
// Other:   "javax.swing.plaf.metal.MetalLookAndFeel"
```

---

### Real-World Cases

- **Cross-platform applications** defaulting to native appearance.
- **Desktop applications** deployed on unknown platforms.
- **Fallback chains** ensuring a valid L&F is always installed.

---

## 4. UIManager.getCrossPlatformLookAndFeelClassName() — Cross-Platform L&F Class Name

### Definitions

**Core Definition.** `getCrossPlatformLookAndFeelClassName()` returns the class name of the **default cross-platform** L&F, which is **Metal** (the "Java L&F").

**Technical Definition.** `public static String getCrossPlatformLookAndFeelClassName()`. The method returns the value of the `swing.crossplatformlaf` system property if set, or `"javax.swing.plaf.metal.MetalLookAndFeel"` by default.

**Beginner-Friendly Explanation.** `getCrossPlatformLookAndFeelClassName()` returns the class name of the theme that looks the same on every platform. By default, that's Metal.

---

### Purposes

- To install a consistent L&F on every platform.
- To fall back gracefully when a native L&F is unavailable.
- To avoid hard-coding "Metal" or another cross-platform L&F class name.
- To allow deployment to override the cross-platform L&F via a system property.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.setLookAndFeel(
    UIManager.getCrossPlatformLookAndFeelClassName());
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `getCrossPlatformLookAndFeelClassName()` | Returns the cross-platform L&F class name. |
| `setLookAndFeel(...)` | Installs it. |

#### Syntax Rules

- Returns Metal by default; may return a different class if `swing.crossplatformlaf` is set.
- Must be called before component creation.

#### Constraints and Limitations

- Cross-platform L&Fs may not feel native on any platform.
- The class name may change if the system property is overridden.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing the Cross-Platform L&F

```java
import javax.swing.*;

public class CrossPlatformDemo {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. Cross
                UIManager.getCrossPlatformLookAndFeelClassName());
        } catch (Exception e) { e.printStackTrace(); }

        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Cross-Platform L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Consistent Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in Metal, identical on every platform.

---

#### Example 2: Verifying the Cross-Platform Class Name

```java
System.out.println(UIManager.getCrossPlatformLookAndFeelClassName());
// Usually: javax.swing.plaf.metal.MetalLookAndFeel
```

---

### Real-World Cases

- **Cross-platform consistency** in enterprise applications.
- **Fallback L&F** when native is unavailable.
- **Deterministic appearance** for tests and screenshots.

---

## 5. Setting L&F Best Practices

### Definitions

**Core Definition.** Setting L&F best practices are the conventions and patterns that ensure L&F installation is correct, portable, and maintainable.

**Technical Definition.** Best practices include: setting the L&F **before** creating any Swing component; using `getSystemLookAndFeelClassName()` for native appearance and `getCrossPlatformLookAndFeelClassName()` for consistency; using `getInstalledLookAndFeels()` for discovery rather than hard-coded class names; wrapping `setLookAndFeel` in `try`/`catch` with a fallback; calling `SwingUtilities.updateComponentTreeUI` after runtime changes; and respecting the EDT.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that prevent L&F bugs: set the theme first, use the discovery helpers instead of hard-coding class names, handle exceptions with a fallback, and update all windows after switching.

---

### Purposes

- To avoid initialising components with the wrong L&F.
- To ensure portable behaviour across platforms.
- To handle missing L&Fs gracefully.
- To support runtime theme switching safely.
- To keep application code independent of any single L&F.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public static void main(String[] args) {
    try {
        UIManager.setLookAndFeel(                                // 1. First
            UIManager.getSystemLookAndFeelClassName());
    } catch (Exception e) {
        try {
            UIManager.setLookAndFeel(                            // 2. Fallback
                UIManager.getCrossPlatformLookAndFeelClassName());
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    SwingUtilities.invokeLater(() -> createAndShowGUI());        // 3. EDT
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Set L&F first | Before any `JComponent` is created. |
| Use helpers | `getSystemLookAndFeelClassName`, `getCrossPlatformLookAndFeelClassName`. |
| Handle exceptions | `try`/`catch` with fallback. |
| Discover for selectors | `getInstalledLookAndFeels()`. |
| Update after runtime change | `SwingUtilities.updateComponentTreeUI(frame)`. |
| EDT | `SwingUtilities.invokeLater`. |
| Test across L&Fs | Verify layout with Metal, Nimbus, System. |

#### Syntax Rules

- **Always set the L&F before creating components.**
- **Always handle the checked exceptions** from `setLookAndFeel`.
- **Always use the discovery helpers** rather than hard-coded class names when you can.
- **Always call `updateComponentTreeUI`** after switching L&Fs at runtime.
- **Always construct and show the GUI on the EDT.**

#### Constraints and Limitations

- The system L&F may be unavailable on some platforms; always provide a fallback.
- Some L&Fs are platform-specific and may fail to install elsewhere.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Initialisation with Fallback

```java
import javax.swing.*;

public class BestPracticeLaf {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                                // 1. Native
                UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            try {
                UIManager.setLookAndFeel(                            // 2. Cross-platform
                    UIManager.getCrossPlatformLookAndFeelClassName());
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
        SwingUtilities.invokeLater(BestPracticeLaf::createAndShowGUI);
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

**Expected Output.** The window appears with the platform's native L&F, or with Metal if the native L&F is unavailable.

**Why the code produces this result.** The `try`/`catch` chain first attempts the system L&F and falls back to the cross-platform L&F. The GUI is built after the L&F is installed and on the EDT.

---

#### Example 2: Best-Practice Runtime Switch

```java
import javax.swing.*;
import java.awt.*;

public class BestPracticeRuntime {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Best Practice Runtime");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            JComboBox<UIManager.LookAndFeelInfo> combo =             // 1. Discover
                new JComboBox<>(UIManager.getInstalledLookAndFeels());

            combo.addActionListener(e -> {
                UIManager.LookAndFeelInfo info =
                    (UIManager.LookAndFeelInfo) combo.getSelectedItem();
                try {
                    UIManager.setLookAndFeel(info.getClassName());   // 2. Install
                    SwingUtilities.updateComponentTreeUI(frame);      // 3. Update
                    frame.pack();                                      // 4. Pack
                } catch (Exception ex) { ex.printStackTrace(); }
            });

            frame.add(combo);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A combo box lists available L&Fs; selecting one immediately restyles the window without restarting.

---

#### Example 3: Common Mistake — Hard-Coding a Platform-Specific L&F

```java
UIManager.setLookAndFeel(
    "com.sun.java.swing.plaf.windows.WindowsLookAndFeel");        // WRONG
```

**Expected Output.** On non-Windows platforms, the Windows L&F is unsupported and falls back to Metal (or throws in older JDKs). The application's appearance is inconsistent across platforms.

**Why this is wrong.** Platform-specific class names should not be hard-coded when portability matters. The correct approach is `getSystemLookAndFeelClassName()` (native) or `getCrossPlatformLookAndFeelClassName()` (consistent).

---

#### Example 4: Common Mistake — Not Updating After a Runtime Switch

```java
UIManager.setLookAndFeel("javax.swing.plaf.nimbus.NimbusLookAndFeel");
// Missing: SwingUtilities.updateComponentTreeUI(frame);
```

**Expected Output.** Existing components may not refresh with the new L&F, producing inconsistent or broken appearance.

**Why this is wrong.** `setLookAndFeel` does not update existing components; `updateComponentTreeUI` must be called on each top-level container.

---

### Real-World Cases

- **Production applications** set the L&F in `main` before any Swing class is touched and provide a fallback.
- **Settings dialogs** list installed L&Fs, install the selected one, and update the component tree.
- **Cross-platform applications** default to the system L&F and fall back to the cross-platform L&F.
- **Educational examples** demonstrate the discovery helpers, exception handling, and update cycle from the start.

---

## References

Oracle. *Class UIManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *Class UIManager*. Java Development Kit Version 23 API Specification. https://docs.oracle.com/en/java/javase/23/docs/api/java.desktop/javax/swing/UIManager.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Nimbus Look and Feel*. The Java Tutorials. http://www-lehre.informatik.uni-osnabrueck.de/manuals/java_tutorial_2012/uiswing/lookandfeel/nimbus.html

Oracle. *Class UIManager — getSystemLookAndFeelClassName*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html#getSystemLookAndFeelClassName--

Oracle. *Class UIManager — getCrossPlatformLookAndFeelClassName*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html#getCrossPlatformLookAndFeelClassName--

Oracle. *Class UIManager — getInstalledLookAndFeels*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html#getInstalledLookAndFeels--

Oracle. *Class UIManager.LookAndFeelInfo*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.LookAndFeelInfo.html

Oracle. *Class SwingUtilities — updateComponentTreeUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html#updateComponentTreeUI-java.awt.Component-

Oracle. *Swing's Threading Policy*. Java Platform, Standard Edition 8 API Specification (javax.swing package summary). https://docs.oracle.com/javase/8/docs/api/javax/swing/package-summary.html#threading

Oracle. *A Swing Architecture Overview*. https://www.oracle.com/technetwork/java/architecture-142923.html

Oracle. *Pluggable Look and Feel — Wikipedia*. https://en.wikipedia.org/wiki/Pluggable_look_and_feel

Stack Overflow. *How to set Nimbus L&F robustly*. https://stackoverflow.com/questions/ (community consensus on iterating `getInstalledLookAndFeels`).

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/