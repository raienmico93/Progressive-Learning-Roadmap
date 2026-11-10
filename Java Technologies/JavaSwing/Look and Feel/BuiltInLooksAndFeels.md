# Built-in Look and Feels — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Built-in Look and Feels** are the `LookAndFeel` implementations that ship with the Java platform, providing ready-made visual styles and behavioural conventions for Swing applications. Each L&F defines how every `JComponent` is painted and how it responds to user input, and each can be installed via `UIManager` without changing application code.

**Technical Definition.** Every built-in L&F is a concrete subclass of `javax.swing.LookAndFeel`. The Java platform ships with several L&Fs: **CrossPlatformLookAndFeel** (Metal, the “Java L&F”), **SystemLookAndFeel** (the platform-native L&F), **Synth**, **Multiplexing**, and platform-specific L&Fs including **Windows**, **Windows Classic**, **GTK+**, **Motif**, and **Aqua** (on macOS). Swing ships with four primary look and feels: Java (also called Metal), Microsoft Windows, GTK+, and CDE/Motif, with additional L&Fs such as Nimbus and Windows Classic available in modern JDKs. The available L&Fs can be enumerated via `UIManager.getInstalledLookAndFeels()`, which returns `LookAndFeelInfo` metadata including the display name and fully qualified class name of each L&F. **Not all L&Fs are available on all platforms**: the Windows L&F runs only on Windows, GTK+ only on UNIX/Linux with GTK+ 2.2 or later, and Aqua only on macOS. Metal and Motif run on any platform. All L&F installation and GUI construction must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** Built-in Look and Feels are the **themes that come pre-installed with Java**. They are the ready-made “skins” for your Swing application. Java gives you several to choose from: **Metal** (the classic cross-platform theme), **Nimbus** (the modern cross-platform theme), **Windows** (matches the current Windows theme), **Windows Classic** (forces the old Windows 95/98/2000 look), **GTK+** (matches the Linux desktop theme), **Motif** (an older UNIX-style theme), and **Aqua** (the macOS theme). You pick one by calling `UIManager.setLookAndFeel(...)` before you build your GUI. Think of it as choosing a filter for your application: the components are the same, but the colours, fonts, borders, and behaviours change to match the chosen theme.

---

### Key Characteristics

1. **Platform independence vs. native fidelity.** Metal, Nimbus, and Motif look the same on every platform; Windows, GTK+, and Aqua mimic the host platform.
2. **Availability varies.** Windows L&F runs only on Windows; GTK+ on Linux/UNIX; Aqua on macOS. Metal and Motif run everywhere.
3. **Enumerability.** `UIManager.getInstalledLookAndFeels()` returns metadata about every installed L&F, allowing applications to discover and offer available themes.
4. **Default L&F.** Metal is the historical default, but modern JVMs may use the system L&F. Nimbus is available from JDK 7 onward.
5. **Synth-based customisation.** Nimbus is built on Synth and is highly customisable via `UIDefaults` and painters.
6. **Motif is deprecated.** The Motif L&F is deprecated since JDK 13 with intent to remove and should not be used in new applications.
7. **EDT discipline.** All L&F installation and component construction must occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and exception handling.
- **Basic Swing:** `JFrame`, `JButton`, `JLabel`, and the `SwingUtilities.invokeLater` pattern.
- **`UIManager` and `LookAndFeel` basics.**
- **The EDT and Swing’s single-thread rule.**
- **A JDK (version 8 or later).** Nimbus is available from JDK 7 onward.

---

### Related Programming Areas

- **`javax.swing.UIManager`** — manages the current L&F and provides class-name helpers.
- **`javax.swing.LookAndFeel`** — the abstract base class for all L&Fs.
- **`javax.swing.plaf.metal`** — the Metal L&F and its themes (Ocean, Steel).
- **`javax.swing.plaf.nimbus`** — the Nimbus L&F (JDK 7+).
- **`javax.swing.plaf.synth`** — the Synth L&F framework, on which Nimbus is built.
- **`com.sun.java.swing.plaf.windows`** — the Windows and Windows Classic L&Fs.
- **`com.sun.java.swing.plaf.gtk`** — the GTK+ L&F.
- **`com.sun.java.swing.plaf.motif`** — the Motif L&F (deprecated).
- **`com.apple.laf.AquaLookAndFeel`** — the macOS Aqua L&F.
- **`SwingUtilities.updateComponentTreeUI`** — updates a hierarchy after an L&F change.

---

## 1. Metal (Cross-Platform)

### Definitions

**Core Definition.** Metal, also called the **Java Look and Feel (JLF)** or **CrossPlatformLookAndFeel**, is the default cross-platform L&F that looks the same on every platform.

**Technical Definition.** `javax.swing.plaf.metal.MetalLookAndFeel` extends `BasicLookAndFeel`. It obtains its colour palette and fonts from a `MetalTheme`. The default theme is **OceanTheme** (since Java 5.0); the older **DefaultMetalTheme** (Steel) can be restored via the `swing.metalTheme=steel` system property or programmatically with `MetalLookAndFeel.setCurrentTheme(new DefaultMetalTheme())`. Metal runs on any platform.

**Beginner-Friendly Explanation.** Metal is the **classic Java theme**. It looks the same whether you run on Windows, macOS, or Linux. It has been the default Swing L&F for most of Java’s history.

---

### Purposes

- To provide a consistent appearance across all platforms.
- To serve as the default L&F when no other L&F is specified.
- To offer a base for themed customisation via `MetalTheme`.
- To guarantee availability on every Java runtime.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// By class name:
UIManager.setLookAndFeel("javax.swing.plaf.metal.MetalLookAndFeel");

// By helper (Cross-platform):
UIManager.setLookAndFeel(UIManager.getCrossPlatformLookAndFeelClassName());

// With a specific theme:
MetalLookAndFeel.setCurrentTheme(new OceanTheme());
UIManager.setLookAndFeel(new MetalLookAndFeel());
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `MetalLookAndFeel` | The L&F class. |
| `OceanTheme` | The default theme (Java 5.0+). |
| `DefaultMetalTheme` | The legacy Steel theme. |
| `MetalLookAndFeel.setCurrentTheme` | Changes the theme before installation. |

#### Syntax Rules

- Metal is available on every platform.
- The theme must be set **before** `setLookAndFeel` or component creation.
- `swing.metalTheme=steel` restores the pre-Java 5 appearance.

#### Constraints and Limitations

- Metal may appear dated compared to native L&Fs or Nimbus.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing Metal with the Ocean Theme

```java
import javax.swing.*;
import javax.swing.plaf.metal.*;

public class MetalDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                MetalLookAndFeel.setCurrentTheme(new OceanTheme());   // 1. Ocean
                UIManager.setLookAndFeel(new MetalLookAndFeel());     // 2. Install
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Metal L&F");                   // 3. GUI
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Metal Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the Metal Ocean theme.

---

### Real-World Cases

- **Cross-platform applications** that want a consistent look on every OS.
- **Legacy applications** that have not migrated to Nimbus.
- **Applications requiring guaranteed L&F availability.**

---

## 2. Nimbus

### Definitions

**Core Definition.** Nimbus is a polished **cross-platform** L&F introduced in Java SE 6 Update 10 and bundled with JDK 7 onward.

**Technical Definition.** `javax.swing.plaf.nimbus.NimbusLookAndFeel`. Nimbus uses **Java 2D vector graphics** to draw the UI rather than static bitmaps, so it renders crisply at any resolution. It is built on the **Synth** L&F framework and is highly customisable via `UIDefaults` and painters. For backwards compatibility, Metal is still the default Swing L&F, but Nimbus can be enabled programmatically, via the command line (`-Dswing.defaultlaf=javax.swing.plaf.nimbus.NimbusLookAndFeel`), or via `swing.properties`.

**Beginner-Friendly Explanation.** Nimbus is the **modern Java theme**. It looks clean and sharp on any screen because it uses vector graphics instead of fixed-size images. It is the recommended cross-platform L&F for modern Swing applications.

---

### Purposes

- To provide a modern, high-resolution cross-platform appearance.
- To offer a smoother alternative to Metal.
- To support customisation through Synth painters and `UIDefaults`.
- To render crisply on high-DPI displays.

---

### Syntax Rules and Structure

#### Complete General Syntax (Robust Installation)

```java
try {
    for (UIManager.LookAndFeelInfo info : UIManager.getInstalledLookAndFeels()) {
        if ("Nimbus".equals(info.getName())) {
            UIManager.setLookAndFeel(info.getClassName());
            break;
        }
    }
} catch (Exception e) {
    // Nimbus not available; fall back to default
}
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `UIManager.getInstalledLookAndFeels()` | Enumerates available L&Fs. |
| `info.getName()` | Display name (e.g., `"Nimbus"`). |
| `info.getClassName()` | Fully qualified class name. |

#### Syntax Rules

- **Do not set Nimbus by class name directly**; not all JDK 6 implementations include it. Iterate installed L&Fs instead.
- Nimbus is available from JDK 7 onward.
- Customisation is done via `UIDefaults` and Synth painters.

#### Constraints and Limitations

- Nimbus may not perfectly match native platform conventions.
- Some custom painters require understanding of Synth’s painter architecture.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Robust Nimbus Installation

```java
import javax.swing.*;

public class NimbusDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                for (UIManager.LookAndFeelInfo info :                 // 1. Enumerate
                        UIManager.getInstalledLookAndFeels()) {
                    if ("Nimbus".equals(info.getName())) {            // 2. Find
                        UIManager.setLookAndFeel(info.getClassName()); // 3. Install
                        break;
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

            JFrame frame = new JFrame("Nimbus L&F");                  // 4. GUI
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Nimbus Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the Nimbus style.

---

### Real-World Cases

- **Modern cross-platform applications** that want a polished default appearance.
- **High-DPI applications** that benefit from vector rendering.
- **Themed applications** that customise Nimbus via Synth.

---

## 3. Motif (CDE/Motif)

### Definitions

**Core Definition.** Motif is a cross-platform L&F that mimics the **CDE/Motif** UNIX desktop style. It is **deprecated**.

**Technical Definition.** `com.sun.java.swing.plaf.motif.MotifLookAndFeel` extends `BasicLookAndFeel`. Its display name is `"CDE/Motif"`. It is deprecated since JDK 13 with intent to remove and should not be used in new applications. It runs on any platform.

**Beginner-Friendly Explanation.** Motif is the **old UNIX theme**. It looks dated and is officially deprecated. Use it only for maintaining legacy applications.

---

### Purposes

- **Legacy compatibility:** to support older applications that expect the Motif appearance.
- **Historical reference:** to understand Swing’s L&F history.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.setLookAndFeel("com.sun.java.swing.plaf.motif.MotifLookAndFeel");
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `MotifLookAndFeel` | The deprecated L&F class. |
| `"CDE/Motif"` | Its display name. |

#### Constraints and Limitations

- **Deprecated**; may be removed in a future JDK.
- Does not support modern visual conventions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing Motif

```java
import javax.swing.*;

public class MotifDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(
                    "com.sun.java.swing.plaf.motif.MotifLookAndFeel"); // 1. Deprecated
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Motif L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Motif Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the Motif style.

---

### Real-World Cases

- **Legacy application maintenance.**
- **Historical research** on Swing L&Fs.

---

## 4. Windows

### Definitions

**Core Definition.** The Windows L&F mimics the **current Windows theme** applied by the user.

**Technical Definition.** `com.sun.java.swing.plaf.windows.WindowsLookAndFeel`. It is available **only on Windows**. It adapts to the user’s current Windows theme, so the application looks native regardless of whether the user has chosen Windows Classic, Windows XP, or a modern Windows theme.

**Beginner-Friendly Explanation.** The Windows theme makes your application look like a **native Windows application**. It follows whatever theme the user has chosen in Windows settings.

---

### Purposes

- To provide a native Windows appearance.
- To follow the user’s current Windows theme automatically.
- To integrate seamlessly with the Windows desktop.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.setLookAndFeel(
    "com.sun.java.swing.plaf.windows.WindowsLookAndFeel");
```

#### Constraints and Limitations

- Available **only on Windows**; falls back to Metal elsewhere.
- Follows the user’s theme; use **Windows Classic** to force the classic look.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing the Windows L&F

```java
import javax.swing.*;

public class WindowsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(
                    "com.sun.java.swing.plaf.windows.WindowsLookAndFeel"); // 1. Native
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Windows L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Native Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button that looks native to the current Windows theme.

---

### Real-World Cases

- **Windows-only enterprise applications.**
- **Applications that want native Windows integration.**

---

## 5. Windows Classic

### Definitions

**Core Definition.** Windows Classic is a variant of the Windows L&F that **forces the Windows 95/98/ME/NT/2000 classic theme**, regardless of the user’s current Windows theme.

**Technical Definition.** `com.sun.java.swing.plaf.windows.WindowsClassicLookAndFeel` extends `WindowsLookAndFeel`. Its display name is `"Windows Classic"`. It runs only on Windows. While `WindowsLookAndFeel` follows the user’s theme, `WindowsClassicLookAndFeel` forces the classic appearance even if the user has selected a modern theme.

**Beginner-Friendly Explanation.** Windows Classic forces the **old grey Windows 95/98 style**, no matter what the user has chosen in their Windows settings.

---

### Purposes

- To force a consistent classic Windows appearance.
- To support legacy applications designed for the classic theme.
- To avoid theme-dependent inconsistencies across Windows versions.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.setLookAndFeel(
    "com.sun.java.swing.plaf.windows.WindowsClassicLookAndFeel");
```

#### Constraints and Limitations

- Available **only on Windows**.
- Visually dated; not recommended for new applications.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Forcing Windows Classic

```java
import javax.swing.*;

public class WindowsClassicDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 1. Force classic
                    "com.sun.java.swing.plaf.windows.WindowsClassicLookAndFeel");
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Windows Classic L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Classic Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the classic Windows 95/98 style.

---

### Real-World Cases

- **Legacy Windows applications.**
- **Testing** classic Windows appearance.

---

## 6. GTK+

### Definitions

**Core Definition.** The GTK+ L&F mimics the **native GNOME/GTK+ desktop theme** on Linux and UNIX systems.

**Technical Definition.** `com.sun.java.swing.plaf.gtk.GTKLookAndFeel`. It is available **only on UNIX/Linux** systems with **GTK+ 2.2 or later** installed. On Linux, `UIManager.getSystemLookAndFeelClassName()` typically returns the GTK+ L&F class name. It can be forced via the `swing.defaultlaf` or `swing.systemlaf` system properties.

**Beginner-Friendly Explanation.** The GTK+ theme makes your application look like a **native Linux application**, matching the desktop theme.

---

### Purposes

- To provide a native Linux/UNIX appearance.
- To integrate with the GTK+ desktop environment.
- To follow the user’s desktop theme.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
UIManager.setLookAndFeel(
    "com.sun.java.swing.plaf.gtk.GTKLookAndFeel");
```

#### Constraints and Limitations

- Available **only on UNIX/Linux** with GTK+ 2.2+.
- Linked against GTK2; some modern desktops use GTK3, which may cause visual mismatches.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing GTK+

```java
import javax.swing.*;

public class GtkDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 1. GTK+
                    "com.sun.java.swing.plaf.gtk.GTKLookAndFeel");
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("GTK+ L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("GTK Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the GTK+ desktop style (on Linux with GTK+).

---

### Real-World Cases

- **Linux desktop applications** that want native GTK+ integration.

---

## 7. Mac (Aqua)

### Definitions

**Core Definition.** Aqua is the **default macOS L&F**, providing the native Macintosh appearance.

**Technical Definition.** `com.apple.laf.AquaLookAndFeel` extends `BasicLookAndFeel`. It is the default L&F on macOS. It differs from other L&Fs in several ways: menu bars can be moved to the screen menu bar via the `apple.laf.useScreenMenuBar` system property; tabbed panes use a pull-down menu for overflow tabs instead of stacking; and component sizing follows Aqua’s strict guidelines. Dark mode support in Aqua has historically been limited.

**Beginner-Friendly Explanation.** Aqua is the **native macOS theme**. On a Mac, Swing applications use Aqua by default and look like native Mac applications.

---

### Purposes

- To provide a native macOS appearance.
- To integrate with the Mac menu bar and Aqua guidelines.
- To support Mac-specific behaviours (screen menu bar, tab overflow menu).

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// Default on macOS:
UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());

// Screen menu bar:
System.setProperty("apple.laf.useScreenMenuBar", "true");
```

#### Constraints and Limitations

- Available **only on macOS**.
- Aqua’s strict sizing guidelines may differ from other L&Fs.
- Dark mode support is limited in older JDK versions.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Enabling the Mac Screen Menu Bar

```java
import javax.swing.*;

public class AquaDemo {
    public static void main(String[] args) {
        System.setProperty("apple.laf.useScreenMenuBar", "true");    // 1. Screen bar
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 2. Aqua
                    UIManager.getSystemLookAndFeelClassName());
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("Aqua L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Mac Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in Aqua, and the application’s menu bar moved to the macOS screen menu bar.

---

### Real-World Cases

- **macOS applications** that want native appearance.
- **Cross-platform applications** that adapt automatically on macOS.

---

## 8. Cross-Platform

### Definitions

**Core Definition.** Cross-platform is a **category** of L&Fs that look identical on every platform, providing consistent appearance regardless of the host OS.

**Technical Definition.** `UIManager.getCrossPlatformLookAndFeelClassName()` returns the class name of the default cross-platform L&F, which is **Metal** (the “Java L&F”). The value can be overridden via the `swing.crossplatformlaf` system property.

**Beginner-Friendly Explanation.** Cross-platform means the application **looks the same everywhere**. Metal is the default cross-platform L&F; Nimbus is also cross-platform.

---

### Purposes

- To ensure a consistent appearance across all platforms.
- To avoid platform-specific visual inconsistencies.
- To simplify testing and support.

---

### Syntax Rules and Structure

```java
UIManager.setLookAndFeel(
    UIManager.getCrossPlatformLookAndFeelClassName());
```

#### Constraints and Limitations

- Cross-platform L&Fs may not feel native to any platform.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Installing the Cross-Platform L&F

```java
import javax.swing.*;

public class CrossPlatformDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 1. Cross-platform
                    UIManager.getCrossPlatformLookAndFeelClassName());
            } catch (Exception e) { e.printStackTrace(); }

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

## 9. System

### Definitions

**Core Definition.** System is a **category** representing the platform-native L&F. `UIManager.getSystemLookAndFeelClassName()` returns the native L&F class name for the current platform.

**Technical Definition.** `UIManager.getSystemLookAndFeelClassName()` returns the class name of the native L&F if one exists, otherwise the cross-platform L&F. On Windows, it returns `WindowsLookAndFeel`; on macOS, `AquaLookAndFeel`; on Linux, typically `GTKLookAndFeel`. The value can be overridden via the `swing.systemlaf` system property.

**Beginner-Friendly Explanation.** System means “use whatever looks native on this computer.” It automatically picks the right L&F for the OS.

---

### Purposes

- To automatically match the host platform’s native appearance.
- To avoid hard-coding platform-specific L&F class names.
- To provide a consistent native experience on every platform.

---

### Syntax Rules and Structure

```java
UIManager.setLookAndFeel(
    UIManager.getSystemLookAndFeelClassName());
```

#### Constraints and Limitations

- The result varies by platform; test on each target OS.
- On unsupported platforms, it falls back to the cross-platform L&F.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Using the System L&F

```java
import javax.swing.*;

public class SystemLafDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(                            // 1. Native
                    UIManager.getSystemLookAndFeelClassName());
            } catch (Exception e) { e.printStackTrace(); }

            JFrame frame = new JFrame("System L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Native Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button rendered in the platform’s native L&F.

---

### Real-World Cases

- **Cross-platform applications** that want native appearance everywhere.
- **Applications deployed on unknown platforms.**

---

## 10. Built-in L&F Best Practices

### Definitions

**Core Definition.** Built-in L&F best practices are the conventions and patterns that ensure L&F selection, installation, and customisation are correct, portable, and maintainable.

**Technical Definition.** Best practices include: setting the L&F **before** any Swing component is created; using `UIManager.getSystemLookAndFeelClassName()` for native appearance; using `UIManager.getCrossPlatformLookAndFeelClassName()` for consistency; enumerating installed L&Fs for runtime selection; calling `SwingUtilities.updateComponentTreeUI` after L&F changes; and preferring Nimbus or Metal for cross-platform applications.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that prevent L&F bugs: set the theme first, use the standard class-name helpers, update all windows when switching, and don’t assume a particular L&F’s look.

---

### Purposes

- To avoid initialising components with the wrong L&F.
- To ensure portability across platforms.
- To support runtime theme switching safely.
- To keep application code independent of any single L&F.

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
| Use class-name helpers | `getSystemLookAndFeelClassName`, `getCrossPlatformLookAndFeelClassName`. |
| Enumerate for robustness | `UIManager.getInstalledLookAndFeels()`. |
| Update after runtime change | `SwingUtilities.updateComponentTreeUI(frame)`. |
| Avoid deprecated L&Fs | Do not use Motif in new code. |
| Test across L&Fs | Verify layout with Metal, Nimbus, and System L&Fs. |

#### Syntax Rules

- **Always set the L&F before creating components.**
- **Always call `updateComponentTreeUI` after changing the L&F at runtime.**
- **Never rely on a specific L&F’s default colours or fonts** in layout calculations.
- **Handle the checked exceptions** from `setLookAndFeel`.

#### Constraints and Limitations

- Some L&Fs are platform-specific; test on each target platform.
- Motif is deprecated and should not be used in new applications.

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

**Why this is wrong.** After an L&F change, existing components must be updated via `SwingUtilities.updateComponentTreeUI`.

---

### Real-World Cases

- **Production applications** set the L&F in `main` before any Swing class is touched.
- **Settings dialogs** update the component tree after an L&F change.
- **Cross-platform testing** verifies layout with Metal, Nimbus, and System L&Fs.

---

## References

Oracle. *Class UIManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Nimbus Look and Feel*. The Java Tutorials. http://www-lehre.informatik.uni-osnabrueck.de/manuals/java_tutorial_2012/uiswing/lookandfeel/nimbus.html

Oracle. *Class MetalLookAndFeel*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/jp/8/docs/api/javax/swing/plaf/metal/MetalLookAndFeel.html

Oracle. *Pluggable look and feel — Wikipedia*. https://en.wikipedia.org/wiki/Pluggable_look_and_feel

Stack Overflow. *How to use Windows XP theme in Java/Swing applications?*. https://stackoverflow.com/questions/16459758/how-to-use-windows-xp-theme-in-java-swing-applications/16460742

Apple Developer. *User Interface Toolkits for Java — Java Development Guide for Mac*. https://developer.apple.com/library/archive/documentation/Java/Conceptual/Java14Development/04-JavaUIToolkits/JavaUIToolkits.html

OpenJDK. *MotifLookAndFeel.java source*. https://cr.openjdk.org/

OpenJDK. *Swing Dev mailing list — Native look and feel on Linux*. https://mail.openjdk.org/

Oracle. *Class UIManager — getSystemLookAndFeelClassName and getCrossPlatformLookAndFeelClassName*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *Class SwingUtilities — updateComponentTreeUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html#updateComponentTreeUI-java.awt.Component-

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/