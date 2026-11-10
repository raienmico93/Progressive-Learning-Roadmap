# Custom Look and Feel — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** A **Custom Look and Feel (L&F)** is a user-written implementation of Swing's `LookAndFeel` abstract class that defines a unique visual appearance and behavioural style for all Swing components in an application. It allows developers to move beyond the built-in L&Fs (Metal, Nimbus, Windows, GTK+, etc.) and create a fully branded or specialised UI that can be installed and swapped at runtime like any other L&F.

**Technical Definition.** `public abstract class LookAndFeel extends Object`. A custom L&F is typically created by extending **`BasicLookAndFeel`** (the recommended base class) rather than implementing `LookAndFeel` directly. `BasicLookAndFeel` provides default `ComponentUI` implementations for every Swing component, backed by the **`UIDefaults`** table. A custom L&F overrides the necessary `initComponentDefaults`, `initClassDefaults`, and `initSystemColorDefaults` methods to supply its own colors, fonts, borders, icons, and UI delegate class mappings. The L&F is registered by `UIManager.setLookAndFeel()`, which triggers `initialize()` followed by `getDefaults()` to populate the application-wide defaults. All installation and GUI construction must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** A **Custom Look and Feel** is your **own theme** for a Swing application. Java ships with several built-in themes (Metal, Nimbus, Windows, etc.), but if you want your application to have a completely unique appearance—your own colors, fonts, button shapes, and icons—you write a custom L&F. The easiest way to do this is to **extend `BasicLookAndFeel`** and override only the parts you want to change; the base class provides working defaults for everything else. Think of it as creating a **custom skin** for your application: you don't have to rebuild the whole UI framework, just the visual and behavioural details that make your application unique.

---

### Key Characteristics

1. **Extend `BasicLookAndFeel`, not `LookAndFeel` directly.** `BasicLookAndFeel` provides complete default implementations for every component; overriding `LookAndFeel` directly would require implementing every UI delegate from scratch.
2. **`UIDefaults` is the configuration centre.** Colors, fonts, borders, icons, and UI delegate class names are all stored as key-value pairs in the `UIDefaults` table returned by `getDefaults()`.
3. **Selective overriding.** You can change only the properties you care about (e.g., button background) and inherit the rest from `BasicLookAndFeel`.
4. **`ColorUIResource` and `FontUIResource`.** These immutable resource wrappers ensure that default values set by the L&F cannot be accidentally modified by application code.
5. **`UIResource` convention.** As a general rule, if a `JComponent` property is `null` or implements `UIResource`, the UI delegate is free to change it; if the developer has set a non-`UIResource` value, the delegate must leave it alone.
6. **Runtime installable.** A custom L&F is installed via `UIManager.setLookAndFeel(new MyLookAndFeel())` and can be changed at runtime with `SwingUtilities.updateComponentTreeUI`.
7. **EDT discipline.** L&F installation and all component construction must occur on the EDT.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and exception handling.
- **Swing fundamentals:** `JComponent`, `UIDefaults`, `UIManager`, and `SwingUtilities.invokeLater`.
- **Understanding of `ComponentUI` and the separable model architecture.**
- **Knowledge of `UIResource` and its subclasses (`ColorUIResource`, `FontUIResource`, `IconUIResource`).**
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`javax.swing.LookAndFeel`** — the abstract base class for all L&Fs.
- **`javax.swing.plaf.basic.BasicLookAndFeel`** — the recommended base class for custom L&Fs.
- **`javax.swing.UIDefaults`** — the defaults table for colors, fonts, icons, and UI class mappings.
- **`javax.swing.plaf.ColorUIResource`, `FontUIResource`, `IconUIResource`** — immutable resource wrappers for defaults.
- **`javax.swing.plaf.ComponentUI`** — the per-component UI delegate base class.
- **`UIManager`** — installs and manages the L&F.
- **`SwingUtilities.updateComponentTreeUI`** — refreshes existing components after a runtime L&F change.

---

## 1. LookAndFeel — The Abstract Base Class

### Definitions

**Core Definition.** `LookAndFeel` is the abstract superclass that "completely characterizes a look and feel from the point of view of the pluggable look and feel components".

**Technical Definition.** `public abstract class LookAndFeel extends Object`. It declares `getDefaults()`, `getDescription()`, `getID()`, `getName()`, `initialize()`, `uninitialize()`, `isNativeLookAndFeel()`, `isSupportedLookAndFeel()`, and `toString()`. `getDefaults()` is called once by `UIManager.setLookAndFeel` to create the L&F-specific defaults table. `initialize()` and `uninitialize()` are called when the L&F is installed and uninstalled.

**Beginner-Friendly Explanation.** `LookAndFeel` is the **top-level contract** for a theme. It declares the methods that every theme must provide: a name, an ID, a description, whether it is native, whether it is supported, and most importantly a `UIDefaults` table.

---

### Purposes

- To define the contract for all L&F implementations.
- To provide the `UIDefaults` table that drives component appearance and behaviour.
- To identify the L&F by name, ID, and description.
- To report native and platform support.
- To manage L&F initialization and cleanup.

---

### Syntax Rules and Structure

#### Complete General Syntax (Minimal `LookAndFeel` Implementation)

```java
public class MyLookAndFeel extends BasicLookAndFeel {               // 1. Base class
    @Override public String getName() { return "My L&F"; }          // 2. Metadata
    @Override public String getID() { return "MyLookAndFeel"; }
    @Override public String getDescription() { return "A custom L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }

    @Override
    protected void initComponentDefaults(UIDefaults table) {        // 3. Defaults
        super.initComponentDefaults(table);
        table.put("Button.background", new ColorUIResource(Color.GRAY));
    }
}
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `getName()` | Short display name (e.g., `"My L&F"`). |
| `getID()` | Unique identifier string. |
| `getDescription()` | One-to-two sentence description. |
| `isNativeLookAndFeel()` | Whether this L&F emulates the native platform. |
| `isSupportedLookAndFeel()` | Whether this L&F is supported on the current platform. |
| `getDefaults()` | Returns the `UIDefaults` table. |
| `initialize()` / `uninitialize()` | Called when installed/uninstalled. |
| `initComponentDefaults(UIDefaults)` | Populates component defaults. |
| `initClassDefaults(UIDefaults)` | Populates `uiClassID` → class name mappings. |
| `initSystemColorDefaults(UIDefaults)` | Populates system color defaults. |

#### Syntax Rules

- Extend `BasicLookAndFeel` rather than `LookAndFeel` directly.
- Override `initComponentDefaults` to set colors, fonts, and borders.
- Use `ColorUIResource` and `FontUIResource` for immutable defaults.
- Call `super.initComponentDefaults(table)` before adding your own defaults.

#### Constraints and Limitations

- `BasicLookAndFeel` is abstract; you must subclass it.
- Failing to provide a `uiClassID` mapping for a component produces random exceptions when that component is created.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: A Minimal Custom L&F

```java
import javax.swing.*;
import javax.swing.plaf.ColorUIResource;
import javax.swing.plaf.basic.BasicLookAndFeel;
import java.awt.*;

public class MinimalCustomLaf extends BasicLookAndFeel {             // 1. Extend
    @Override public String getName() { return "Minimal Custom"; }   // 2. Metadata
    @Override public String getID() { return "MinimalCustomLaf"; }
    @Override public String getDescription() { return "A minimal custom L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }

    @Override
    protected void initComponentDefaults(UIDefaults table) {         // 3. Defaults
        super.initComponentDefaults(table);
        table.put("Button.background",
                  new ColorUIResource(new Color(70, 130, 180)));
        table.put("Button.foreground", new ColorUIResource(Color.WHITE));
        table.put("Panel.background",
                  new ColorUIResource(new Color(245, 245, 245)));
    }

    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(new MinimalCustomLaf());       // 4. Install
        } catch (UnsupportedLookAndFeelException e) {
            e.printStackTrace();
        }
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Minimal Custom L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Custom Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a steel-blue button with white text and a light-gray panel background.

**Why the code produces this result.** `BasicLookAndFeel` provides default UI delegates for all components; overriding `initComponentDefaults` replaces specific defaults (button background, button foreground, panel background) while inheriting everything else.

---

## 2. BasicLookAndFeel — The Recommended Base Class

### Definitions

**Core Definition.** `BasicLookAndFeel` is an abstract class that serves as the base for creating a L&F for Swing, providing default `ComponentUI` implementations for every standard component.

**Technical Definition.** `public abstract class BasicLookAndFeel extends LookAndFeel implements Serializable`. "A base class to use in creating a look and feel for Swing. Each of the `ComponentUI`s provided by `BasicLookAndFeel` derives its behavior from the defaults table". It populates `UIDefaults` via `initClassDefaults`, `initSystemColorDefaults`, and `initComponentDefaults`.

**Beginner-Friendly Explanation.** `BasicLookAndFeel` is the **foundation** for custom L&Fs. It already knows how to paint every Swing component; you just tell it to paint with your colors, fonts, and icons.

---

### Purposes

- To provide a complete, working baseline for custom L&Fs.
- To supply default UI delegates for every component.
- To populate the `UIDefaults` table with sensible defaults.
- To reduce the effort of customisation to overriding only the properties you want to change.

---

### Syntax Rules and Structure

#### Complete General Syntax (Overriding Defaults)

```java
public class MyLaf extends BasicLookAndFeel {
    @Override
    protected void initComponentDefaults(UIDefaults table) {
        super.initComponentDefaults(table);
        table.put("Button.background", new ColorUIResource(...));
        table.put("Label.font", new FontUIResource(...));
        table.put("Tree.closedIcon", new IconUIResource(...));
    }
}
```

#### Component Breakdown

| Method | Responsibility |
|--------|----------------|
| `initClassDefaults(UIDefaults)` | Maps `uiClassID` keys to UI delegate class names. |
| `initSystemColorDefaults(UIDefaults)` | Populates system color defaults. |
| `initComponentDefaults(UIDefaults)` | Populates component-specific defaults. |
| `getDefaults()` | Calls all three `init` methods and returns the table. |

#### Syntax Rules

- Override `initComponentDefaults` for colors, fonts, and borders.
- Override `initClassDefaults` to replace UI delegates with custom ones.
- Always call `super.initComponentDefaults(table)` first.

#### Constraints and Limitations

- `BasicLookAndFeel` provides **basic** delegates; advanced effects may require custom `ComponentUI` implementations.
- Some defaults are used by the UI delegate during `installUI`; setting them in `initComponentDefaults` is the correct approach.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Custom L&F with Icons and Fonts

```java
import javax.swing.*;
import javax.swing.plaf.*;
import javax.swing.plaf.basic.BasicLookAndFeel;
import java.awt.*;

public class FullCustomLaf extends BasicLookAndFeel {                // 1. Extend
    @Override public String getName() { return "Full Custom"; }
    @Override public String getID() { return "FullCustomLaf"; }
    @Override public String getDescription() { return "Full custom L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }

    @Override
    protected void initComponentDefaults(UIDefaults table) {
        super.initComponentDefaults(table);
        Color accent = new Color(0, 120, 215);                       // 2. Colors
        Font baseFont = new Font("SansSerif", Font.PLAIN, 13);       // 3. Fonts

        table.put("Button.background", new ColorUIResource(accent));
        table.put("Button.foreground", new ColorUIResource(Color.WHITE));
        table.put("Button.font", new FontUIResource(baseFont));
        table.put("Label.font", new FontUIResource(baseFont));
        table.put("Panel.background",
                  new ColorUIResource(new Color(250, 250, 250)));
        table.put("TextField.background", new ColorUIResource(Color.WHITE));
        table.put("TextField.foreground", new ColorUIResource(Color.BLACK));
    }

    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(new FullCustomLaf());
        } catch (UnsupportedLookAndFeelException e) { e.printStackTrace(); }
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Full Custom L&F");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setLayout(new FlowLayout());
            frame.add(new JButton("Button"));
            frame.add(new JLabel("Label"));
            frame.add(new JTextField(10));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window where buttons have a blue background with white text, and all components use the custom 13-point SansSerif font.

---

## 3. Custom L&F Best Practices

### Definitions

**Core Definition.** Custom L&F best practices are the conventions and patterns that ensure custom L&Fs are correct, maintainable, portable, and performant.

**Technical Definition.** Best practices include: extending `BasicLookAndFeel`; using `UIResource` wrappers for defaults; overriding only `initComponentDefaults` for simple customisation; falling back to `BasicLookAndFeel` for components not customised; respecting the `UIResource` convention for property installation; and testing the L&F across multiple component types.

**Beginner-Friendly Explanation.** Best practices are the **rules of thumb** that keep custom L&Fs simple and safe. They include things like "extend `BasicLookAndFeel`," "use `ColorUIResource`," and "only override what you need."

---

### Purposes

- To minimise the amount of code needed for a custom L&F.
- To ensure defaults are immutable and safe from accidental modification.
- To maintain compatibility with all Swing components.
- To make the L&F portable and future-proof.
- To respect the `UIResource` contract for property installation.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```java
public class BestPracticeLaf extends BasicLookAndFeel {              // 1. Basic
    @Override public String getName() { return "Best Practice"; }   // 2. Metadata
    @Override public String getID() { return "BestPracticeLaf"; }
    @Override public String getDescription() { return "Best practice L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }

    @Override
    protected void initComponentDefaults(UIDefaults table) {
        super.initComponentDefaults(table);                          // 3. Super
        table.put("Button.background",
                  new ColorUIResource(new Color(0, 120, 215)));    // 4. UIResource
        table.put("Label.font",
                  new FontUIResource(new Font("SansSerif", Font.PLAIN, 13)));
    }
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Extend `BasicLookAndFeel` | `extends BasicLookAndFeel` |
| Metadata methods | `getName`, `getID`, `getDescription`, `isNative`, `isSupported` |
| Call `super.initComponentDefaults` | Before adding custom defaults |
| Use `UIResource` wrappers | `ColorUIResource`, `FontUIResource`, `IconUIResource` |
| Override only what you need | Do not reimplement everything |
| Respect `UIResource` convention | Do not overwrite developer-set properties |

#### Syntax Rules

- **Always extend `BasicLookAndFeel`**, not `LookAndFeel` directly.
- **Always wrap defaults in `UIResource` subclasses** to protect them from accidental modification.
- **Always call `super.initComponentDefaults(table)`** before adding your own defaults.
- **Always implement the metadata methods** (`getName`, `getID`, `getDescription`, `isNativeLookAndFeel`, `isSupportedLookAndFeel`).
- **Always test with all component types** used in the application.

#### Constraints and Limitations

- Custom `ComponentUI` implementations are required for effects beyond default customisation (e.g., custom painting of buttons).
- Changes to the L&F must be followed by `SwingUtilities.updateComponentTreeUI` on all top-level containers.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Custom L&F with Fallback

```java
import javax.swing.*;
import javax.swing.plaf.ColorUIResource;
import javax.swing.plaf.FontUIResource;
import javax.swing.plaf.basic.BasicLookAndFeel;
import java.awt.*;

public class BestPracticeLaf extends BasicLookAndFeel {              // 1. Basic
    @Override public String getName() { return "Best Practice"; }   // 2. Metadata
    @Override public String getID() { return "BestPracticeLaf"; }
    @Override public String getDescription() { return "Best practice L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }

    @Override
    protected void initComponentDefaults(UIDefaults table) {
        super.initComponentDefaults(table);                          // 3. Super
        Color accent = new Color(0, 120, 215);
        Font base = new Font("SansSerif", Font.PLAIN, 13);            // 4. Font

        table.put("Button.background", new ColorUIResource(accent));
        table.put("Button.foreground", new ColorUIResource(Color.WHITE));
        table.put("Button.font", new FontUIResource(base));
        table.put("Label.font", new FontUIResource(base));
        table.put("Panel.background",
                  new ColorUIResource(new Color(245, 245, 245)));
    }

    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(new BestPracticeLaf());        // 5. Install
        } catch (UnsupportedLookAndFeelException e) {
            try {
                UIManager.setLookAndFeel(
                    UIManager.getCrossPlatformLookAndFeelClassName());
            } catch (Exception ex) { ex.printStackTrace(); }
        }
        SwingUtilities.invokeLater(BestPracticeLaf::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Best Practice L&F");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new FlowLayout());
        frame.add(new JButton("Custom Button"));
        frame.add(new JLabel("Custom Label"));
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** A window with a blue button and light-gray panel, using the custom font, with a safe fallback if installation fails.

---

#### Example 2: Common Mistake — Extending `LookAndFeel` Directly

```java
public class BadCustomLaf extends LookAndFeel {                     // WRONG
    @Override public String getName() { return "Bad"; }
    @Override public String getID() { return "BadLaf"; }
    @Override public String getDescription() { return "Bad L&F"; }
    @Override public boolean isNativeLookAndFeel() { return false; }
    @Override public boolean isSupportedLookAndFeel() { return true; }
    // No getDefaults() implementation → components fail to initialise
}
```

**Expected Output.** Components may throw exceptions or render incorrectly because no `UIDefaults` table is provided.

**Why this is wrong.** Extending `LookAndFeel` directly requires implementing the entire `UIDefaults` table and every `uiClassID` mapping from scratch. `BasicLookAndFeel` provides this baseline automatically.

---

## References

Oracle. *Class BasicLookAndFeel*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/plaf/basic/BasicLookAndFeel.html

Oracle. *Class LookAndFeel*. Java Platform, Standard Edition 25 API Specification. https://javadoc.scijava.org/Java25/java.desktop/javax/swing/LookAndFeel.html

Oracle. *Class LookAndFeel*. Java 2 Platform Standard Edition 1.3 API Specification. https://resources.mpi-inf.mpg.de/departments/d5/teaching/ss05/is05/java/IRDMSearch/javadoc/javax/swing/LookAndFeel.html

Oracle. *Class BasicLookAndFeel*. Java Platform, Standard Edition 27 API Specification (Japanese). https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/plaf/basic/BasicLookAndFeel.html

Swing Chapter 21 — Pluggable Look & Feel. http://dev.cs.ovgu.de/java/Books/SwingBook/Chapter21html/index.html

Oracle. *Class UIManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *Class SwingUtilities — updateComponentTreeUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html#updateComponentTreeUI-java.awt.Component-

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/

CSDN. *Java外观皮肤包定制与UI美化实战*. https://blog.csdn.net/weixin_42355400/article/details/152283020

CSDN. *swing学习-look and feel设置篇*. https://blog.csdn.net/netliving/article/details/83640830

GNU Classpath. *GNULookAndFeel.java — Example of a custom BasicLookAndFeel implementation*. https://cgit.git.savannah.gnu.org/cgit/classpath.git/plain/examples/gnu/classpath/examples/swing/GNULookAndFeel.java