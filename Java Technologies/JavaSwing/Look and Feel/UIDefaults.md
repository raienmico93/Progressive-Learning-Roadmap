# UI Defaults — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **UI Defaults** are the centralised table of values — colors, fonts, icons, borders, dimensions, insets, and UI delegate class names — that Swing components consult at construction time to determine their appearance. The table is managed by `UIManager` through the `UIDefaults` class, and it is the single source of truth for every component's default look across the entire application.

**Technical Definition.** `javax.swing.UIDefaults` extends `Hashtable<Object, Object>` and provides type‑checked accessors (`getColor`, `getFont`, `getIcon`, `getBorder`, `getInsets`, `getDimension`, `getInt`, `getBoolean`, etc.), lazy value resolution (`get` returns the value, resolving `LazyValue` and `ActiveValue` implementations on demand), and property‑change support for the `"UIDefaults"` property. `UIManager` maintains **three levels of defaults** — developer defaults, look and feel defaults, and system defaults — resolved in that order. Developer defaults (set via `UIManager.put`) take precedence over look and feel defaults (from `getLookAndFeelDefaults()`), which take precedence over system defaults. Component UI delegates read from these tables when installing themselves and repaint themselves when a UI default changes.

**Beginner-Friendly Explanation.** **UI Defaults** is the **shared settings library** for your entire Swing application. When you create a `JButton`, the button doesn't hard‑code its font or background — it looks them up in the UI Defaults table under keys like `"Button.font"` and `"Button.background"`. If you want to change the font of **every** button in your application, you change **one** entry in the UI Defaults table and every button updates. Think of UI Defaults as the **master style guide** for the whole app: one change ripples out to every component.

---

### Key Characteristics

1. **Three‑tier resolution.** Developer defaults override look and feel defaults, which override system defaults.
2. **Typed accessors.** `getColor`, `getFont`, `getIcon`, `getBorder`, `getInsets`, `getInt`, `getBoolean`, `getDimension`, and `getString` return the correct type.
3. **Lazy values.** Values may be `UIDefaults.LazyValue` or `UIDefaults.ActiveValue`; `get` resolves them on first access.
4. **Property‑change support.** Listeners registered on `UIManager` for the `"lookAndFeel"` or `"UIDefaults"` property are notified when defaults change.
5. **UIResource convention.** Values that implement `UIResource` (e.g., `ColorUIResource`, `FontUIResource`) are considered "owned" by the L&F and may be replaced when the L&F changes; values set by application code without `UIResource` are preserved.
6. **Installation‑time snapshot.** Components read defaults at `installUI` time; changing a default after a component is created does **not** automatically update that component.
7. **Change propagation.** `UIManager.put` fires a property‑change event; components listening to it can refresh their UI delegate — but you still typically need `SwingUtilities.updateComponentTreeUI` for structural changes.
8. **EDT discipline.** All defaults manipulation must occur on the Event Dispatch Thread (EDT).

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and generics.
- **Basic Swing:** `JComponent`, `JButton`, `JLabel`, and `SwingUtilities.invokeLater`.
- **Understanding of `UIManager` and the Look and Feel architecture.**
- **`UIResource` and its subclasses.**
- **The EDT and Swing's single‑thread rule.**
- **A JDK (version 8 or later).**

---

### Related Programming Areas

- **`javax.swing.UIManager`** — manages the defaults and the current L&F.
- **`javax.swing.UIDefaults`** — the defaults table itself.
- **`javax.swing.plaf.ColorUIResource`, `FontUIResource`, `IconUIResource`, `BorderUIResource`, `DimensionUIResource`, `InsetsUIResource`** — immutable resource wrappers.
- **`UIDefaults.LazyValue`, `UIDefaults.ActiveValue`, `UIDefaults.ProxyLazyValue`** — deferred value resolution.
- **`UIManager.LookAndFeelInfo`** — metadata for installed L&Fs.
- **`ComponentUI`** — the per‑component delegate that reads defaults.
- **`SwingUtilities.updateComponentTreeUI`** — refreshes components after L&F or default changes.

---

## 1. UIManager — The Defaults Manager

### Definitions

**Core Definition.** `UIManager` is the central Swing class that manages the current Look and Feel, the installed L&Fs, the `UIDefaults` table, and change notification for the whole application.

**Technical Definition.** `public class UIManager extends Object implements Serializable`. The class manages the following data: the current look and feel, the set of available L&Fs, `PropertyChangeListener`s that are notified when the look and feel is changed, the look and feel defaults, and the developer defaults. All accessors are static; the class is a global singleton facade.

**Beginner‑Friendly Explanation.** `UIManager` is the **configuration manager** for Swing. It knows which theme is active, which themes are available, and what the default fonts, colors, and icons are. You call its static methods to install themes, read defaults, or override them.

---

### Purposes

- To provide a single point of access to the application‑wide defaults table.
- To manage the current Look and Feel.
- To enumerate and install available L&Fs.
- To provide typed accessor helpers (`getColor`, `getFont`, `getIcon`, etc.).
- To notify listeners when the L&F changes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
// Get the developer defaults table:
UIDefaults defaults = UIManager.getDefaults();

// Get the L&F defaults table:
UIDefaults lafDefaults = UIManager.getLookAndFeelDefaults();

// Add or override a value:
UIManager.put("Button.background", new Color(0, 120, 215));

// Typed accessors:
Color color = UIManager.getColor("Button.background");
Font font = UIManager.getFont("Button.font");
Icon icon = UIManager.getIcon("OptionPane.informationIcon");

// Get the current L&F:
LookAndFeel laf = UIManager.getLookAndFeel();
```

#### Component Breakdown

| Method | Returns | Meaning |
|--------|---------|---------|
| `getDefaults()` | `UIDefaults` | The developer defaults table (mutable, top priority). |
| `getLookAndFeelDefaults()` | `UIDefaults` | The L&F defaults table (read‑only for applications). |
| `put(Object, Object)` | `void` | Adds or overrides a developer default. |
| `getColor(Object)` | `Color` | Typed accessor for color defaults. |
| `getFont(Object)` | `Font` | Typed accessor for font defaults. |
| `getIcon(Object)` | `Icon` | Typed accessor for icon defaults. |
| `getBorder(Object)` | `Border` | Typed accessor for border defaults. |
| `getInt(Object)` | `int` | Typed accessor for integer defaults. |
| `getBoolean(Object)` | `boolean` | Typed accessor for boolean defaults. |
| `getDimension(Object)` | `Dimension` | Typed accessor for dimension defaults. |
| `getInsets(Object)` | `Insets` | Typed accessor for insets defaults. |
| `getString(Object)` | `String` | Typed accessor for string defaults. |

#### Syntax Rules

- `UIManager.put` should be called **before** component creation for the value to be picked up.
- `getDefaults()` returns the mutable developer table; `getLookAndFeelDefaults()` returns the L&F table.
- Typed accessors return `null` if the key is not found or has the wrong type.
- All defaults manipulation should occur on the EDT.

#### Constraints and Limitations

- Changing a default after a component is created does **not** retroactively update that component.
- Some keys are L&F‑specific; not all keys exist in all L&Fs.

---

### Annotated Complete Step-by‑Step Code Examples

#### Example 1: Reading and Overriding Defaults

```java
import javax.swing.*;
import javax.swing.plaf.ColorUIResource;
import java.awt.*;

public class DefaultsDemo {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            // 1. Read an existing default.
            Color defaultButtonBg = UIManager.getColor("Button.background");
            System.out.println("Default button background: " + defaultButtonBg);

            // 2. Override the default before creating components.
            UIManager.put("Button.background",
                          new ColorUIResource(new Color(0, 120, 215)));
            UIManager.put("Button.foreground",
                          new ColorUIResource(Color.WHITE));

            // 3. Create the component; it picks up the new defaults.
            JButton button = new JButton("Custom Button");

            JFrame frame = new JFrame("Defaults Demo");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(button);
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** The console prints the default button background; the button appears with the custom blue background and white text.

**Why the code produces this result.** `UIManager.getColor` reads the current value; `UIManager.put` overrides it with a `ColorUIResource`. The `JButton` constructor reads the new value when it installs its UI delegate.

---

#### Example 2: Inspecting the Defaults Table

```java
import javax.swing.*;
import java.util.Enumeration;

public class InspectDefaults {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            UIDefaults defaults = UIManager.getDefaults();          // 1. Table
            Enumeration<Object> keys = defaults.keys();               // 2. Keys
            int buttonKeys = 0;
            while (keys.hasMoreElements()) {
                Object key = keys.nextElement();
                if (key.toString().startsWith("Button.")) {           // 3. Filter
                    System.out.println(key + " = " + defaults.get(key));
                    buttonKeys++;
                }
            }
            System.out.println("Total Button.* keys: " + buttonKeys);
        });
    }
}
```

**Expected Output.** The console prints every `Button.*` default (font, background, border, margin, etc.) and the total count.

---

### Real-World Cases

- **Theming:** changing default fonts and colors application‑wide from a settings dialog.
- **Branding:** applying a corporate color palette via `UIManager.put`.
- **Debugging:** inspecting defaults to understand why a component looks different than expected.
- **Accessibility:** increasing default font sizes for readability.

---

## 2. UIManager.getDefaults() — The Developer Defaults Table

### Definitions

**Core Definition.** `getDefaults()` returns the developer defaults `UIDefaults` table — the highest‑priority, mutable table for application‑level overrides.

**Technical Definition.** `public static UIDefaults getDefaults()`. The returned table is the **developer defaults** table, which is chained above the L&F defaults and the system defaults. Values placed here via `put` or `putAll` take precedence over L&F defaults. The method returns a live reference; modifications affect the application‑wide defaults.

**Beginner‑Friendly Explanation.** `getDefaults()` gives you the **top‑level settings table**. Anything you put here overrides the L&F defaults. It's the table you modify to customise the application's appearance.

---

### Purposes

- To access the mutable developer defaults table for bulk manipulation.
- To enumerate all keys and values currently in effect.
- To query individual keys with type‑safe access.
- To install application‑wide overrides.

---

### Syntax Rules and Structure

```java
UIDefaults defaults = UIManager.getDefaults();
defaults.put("Button.background", new ColorUIResource(...));
defaults.putAll(otherDefaultsTable);
Object value = defaults.get("Button.font");
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `getDefaults()` | Returns the developer defaults table. |
| `put(Object, Object)` | Adds or overrides a value. |
| `get(Object)` | Returns the value, resolving lazy values. |
| `remove(Object)` | Removes a key. |
| `keys()` | Returns an `Enumeration` of keys. |
| `keySet()` | Returns a `Set` of keys. |

#### Constraints and Limitations

- Values added via `getDefaults().put` do not automatically fire component repaints.

---

### Annotated Code Example

```java
UIDefaults defaults = UIManager.getDefaults();
defaults.put("Label.font", new FontUIResource("SansSerif", Font.BOLD, 14));
defaults.put("Panel.background", new ColorUIResource(Color.WHITE));
```

**Expected Output.** All labels use bold 14‑point SansSerif; panels have a white background.

---

## 3. UIManager.put() — Overriding a Default

### Definitions

**Core Definition.** `UIManager.put(Object key, Object value)` adds or overrides a developer default in the developer defaults table.

**Technical Definition.** `public static void put(Object key, Object value)`. The method delegates to `getDefaults().put(key, value)` and fires a property‑change event for the `"UIDefaults"` property. Components that listen for changes (many `ComponentUI` implementations do) may refresh themselves. Values should be wrapped in `UIResource` subclasses for L&F‑managed properties.

**Beginner‑Friendly Explanation.** `UIManager.put` is the **setter** for application‑wide defaults. You give it a key (e.g., `"Button.font"`) and a value, and every component created afterward uses that value.

---

### Purposes

- To override a single default application‑wide.
- To install custom fonts, colors, icons, and borders.
- To support dark mode, branding, and accessibility themes.
- To tweak L&F‑specific defaults without replacing the L&F.

---

### Syntax Rules and Structure

```java
UIManager.put("Button.background", new ColorUIResource(...));
UIManager.put("Label.font", new FontUIResource(...));
UIManager.put("Tree.collapsedIcon", new IconUIResource(icon));
```

#### Component Breakdown

| Parameter | Meaning |
|-----------|---------|
| `Object key` | The default's key (a `String` following the `"ComponentName.propertyName"` convention). |
| `Object value` | The value (typically a `UIResource` subclass). |

#### Syntax Rules

- Call **before** creating components for the value to take effect.
- Use `UIResource` subclasses to allow the L&F to override when it changes.
- `put` fires a `PropertyChangeEvent` for the `"UIDefaults"` property.

#### Constraints and Limitations

- Existing components may not refresh automatically; `updateComponentTreeUI` may be needed.

---

### Annotated Code Example

```java
UIManager.put("OptionPane.informationIcon",
              new IconUIResource(new ImageIcon("custom-info.png")));
UIManager.put("OptionPane.errorIcon",
              new IconUIResource(new ImageIcon("custom-error.png")));
```

**Expected Output.** Standard `JOptionPane` dialogs use the custom icons.

---

## 4. UIManager.getColor() — Reading Color Defaults

### Definitions

**Core Definition.** `getColor(Object key)` returns the `Color` value associated with the key, or `null` if the key does not exist or is not a `Color`.

**Technical Definition.** `public static Color getColor(Object key)`. The method looks up the key in the defaults and casts the result to `Color`. It does **not** throw a `ClassCastException`; it returns `null` if the value is not a `Color`.

**Beginner‑Friendly Explanation.** `getColor` fetches a color default like `"Button.background"` or `"Panel.background"`.

---

### Purposes

- To read the current color default for a key.
- To apply L&F‑consistent colors in custom painting.
- To check whether a default exists before using it.

---

### Syntax Rules and Structure

```java
Color bg = UIManager.getColor("Panel.background");
if (bg != null) { g.setColor(bg); }
```

#### Component Breakdown

| Return | Meaning |
|--------|---------|
| `Color` | The color default. |
| `null` | Key not found or wrong type. |

#### Constraints and Limitations

- Returns `null`, not a `ClassCastException`, for type mismatches.

---

### Annotated Code Example

```java
public class CustomButton extends JButton {
    @Override
    protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        try {
            Color accent = UIManager.getColor("Button.background");
            if (accent != null) {
                g2.setColor(accent);
                g2.fillRoundRect(0, 0, getWidth(), getHeight(), 10, 10);
            }
        } finally {
            g2.dispose();
        }
        super.paintComponent(g);
    }
}
```

**Expected Output.** A `JButton` subclass that paints a rounded background using the L&F's `"Button.background"` color.

---

## 5. UIManager.getFont() — Reading Font Defaults

### Definitions

**Core Definition.** `getFont(Object key)` returns the `Font` value associated with the key, or `null`.

**Technical Definition.** `public static Font getFont(Object key)`. Returns the font stored under the key. Fonts are typically `FontUIResource` instances to allow L&F overriding.

**Beginner‑Friendly Explanation.** `getFont` fetches a font default like `"Button.font"` or `"Label.font"`.

---

### Purposes

- To read the current font for a component type.
- To apply consistent typography in custom components.
- To derive larger or bolder fonts from L&F fonts.

---

### Syntax Rules and Structure

```java
Font labelFont = UIManager.getFont("Label.font");
if (labelFont != null) {
    Font larger = labelFont.deriveFont(16f);
    g.setFont(larger);
}
```

---

### Annotated Code Example

```java
Font baseFont = UIManager.getFont("Label.font");
if (baseFont != null) {
    Font heading = baseFont.deriveFont(Font.BOLD, 20f);
    JLabel title = new JLabel("Heading");
    title.setFont(heading);
}
```

**Expected Output.** A `JLabel` using a bold 20‑point version of the L&F's label font.

---

## 6. UIManager.getIcon() — Reading Icon Defaults

### Definitions

**Core Definition.** `getIcon(Object key)` returns the `Icon` value associated with the key, or `null`.

**Technical Definition.** `public static Icon getIcon(Object key)`. Returns the icon stored under the key. Common keys include `"OptionPane.informationIcon"`, `"OptionPane.warningIcon"`, `"OptionPane.errorIcon"`, `"OptionPane.questionIcon"`, `"Tree.expandedIcon"`, `"Tree.collapsedIcon"`, `"FileView.directoryIcon"`, `"FileView.fileIcon"`.

**Beginner‑Friendly Explanation.** `getIcon` fetches an icon default, useful for `JOptionPane`, `JTree`, `JFileChooser`, and other components.

---

### Purposes

- To read the current icon for a component.
- To build custom components that share the L&F's icons.
- To replace icons application‑wide with `UIManager.put`.

---

### Syntax Rules and Structure

```java
Icon info = UIManager.getIcon("OptionPane.informationIcon");
if (info != null) {
    label.setIcon(info);
}
```

---

### Annotated Code Example

```java
Icon errorIcon = UIManager.getIcon("OptionPane.errorIcon");
JLabel errorLabel = new JLabel("Error", errorIcon, SwingConstants.LEFT);
```

**Expected Output.** A label displaying "Error" next to the L&F's error icon.

---

## 7. UI Defaults Best Practices

### Definitions

**Core Definition.** UI Defaults best practices are the conventions and patterns that ensure defaults manipulation is correct, portable, and maintainable.

**Technical Definition.** Best practices include: setting defaults **before** component creation; using `UIResource` subclasses for L&F‑managed values; preferring `UIManager.put` over per‑component setters for application‑wide changes; calling `SwingUtilities.updateComponentTreeUI` after changing defaults that affect existing components; avoiding hard‑coded keys for L&F‑specific internals; and respecting the EDT.

**Beginner‑Friendly Explanation.** Best practices are the **rules of thumb** that keep defaults manipulation safe: set them first, wrap them in `UIResource`, and update the component tree if necessary.

---

### Purposes

- To ensure defaults changes take effect for components created afterward.
- To protect defaults from accidental modification by application code.
- To keep the application consistent with L&F expectations.
- To avoid errors from missing or L&F‑specific keys.
- To respect the EDT.

---

### Syntax Rules and Structure

#### Complete General Best‑Practice Skeleton

```java
public static void main(String[] args) {
    try {
        UIManager.setLookAndFeel(                                // 1. L&F first
            UIManager.getSystemLookAndFeelClassName());
    } catch (Exception e) { e.printStackTrace(); }

    UIManager.put("Button.background",                           // 2. Defaults
                  new ColorUIResource(new Color(0, 120, 215)));
    UIManager.put("Label.font",
                  new FontUIResource(new Font("SansSerif", Font.PLAIN, 13)));

    SwingUtilities.invokeLater(() -> createAndShowGUI());        // 3. GUI
}
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Set L&F first | `setLookAndFeel` before defaults |
| Set defaults second | `UIManager.put` before components |
| Use `UIResource` | `ColorUIResource`, `FontUIResource`, `IconUIResource` |
| Prefer application‑wide changes | `UIManager.put` over per‑component setters |
| Update tree after changes | `SwingUtilities.updateComponentTreeUI` |
| Respect EDT | `SwingUtilities.invokeLater` |

#### Syntax Rules

- **Always set the L&F before applying defaults.**
- **Always wrap defaults in `UIResource` subclasses** so the L&F can override them when it changes.
- **Always call `updateComponentTreeUI`** if defaults change after components are created.
- **Never rely on L&F‑specific internal keys** that are not part of the documented public defaults.

#### Constraints and Limitations

- Some defaults are only read at `installUI` time; changing them afterward requires `updateComponentTreeUI`.
- Not all L&Fs honour all keys; check platform documentation.

---

### Annotated Code Example

```java
import javax.swing.*;
import javax.swing.plaf.ColorUIResource;
import javax.swing.plaf.FontUIResource;
import java.awt.*;

public class BestPracticeDefaults {
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(                             // 1. L&F
                UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) { e.printStackTrace(); }

        UIManager.put("Button.background",                        // 2. Colors
                      new ColorUIResource(new Color(0, 120, 215)));
        UIManager.put("Button.foreground",
                      new ColorUIResource(Color.WHITE));
        UIManager.put("Label.font",                               // 3. Fonts
                      new FontUIResource(new Font("SansSerif", Font.PLAIN, 13)));

        SwingUtilities.invokeLater(BestPracticeDefaults::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Best Practice Defaults");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new FlowLayout());
        frame.add(new JButton("Custom Button"));
        frame.add(new JLabel("Label"));
        frame.pack();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }
}
```

**Expected Output.** All buttons use the custom blue background with white text; all labels use 13‑point SansSerif.

---

### Real‑World Cases

- **Application theming** that changes default colors and fonts across the entire UI.
- **Accessibility mode** that increases default font sizes.
- **Branding** that applies a corporate color palette.
- **Dark mode** that overrides panel and text backgrounds.

---

## References

Oracle. *Class UIManager*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIManager.html

Oracle. *Class UIDefaults*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/UIDefaults.html

Oracle. *Class ColorUIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/ColorUIResource.html

Oracle. *Class FontUIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/FontUIResource.html

Oracle. *Class IconUIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/IconUIResource.html

Oracle. *Interface UIResource*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/plaf/UIResource.html

Oracle. *How to Set the Look and Feel*. The Java Tutorials. http://dev.cs.ovgu.de/java/Tutorials/java/uiswing/lookandfeel/plaf.html

Oracle. *Class BasicLookAndFeel*. Java Platform, Standard Edition 27 API Specification. https://docs.oracle.com/en/java/javase/27/docs/api/java.desktop/javax/swing/plaf/basic/BasicLookAndFeel.html

Oracle. *Class LookAndFeel*. Java Platform, Standard Edition 25 API Specification. https://javadoc.scijava.org/Java25/java.desktop/javax/swing/LookAndFeel.html

Swing Chapter 21 — Pluggable Look & Feel. http://dev.cs.ovgu.de/java/Books/SwingBook/Chapter21html/index.html

Oracle. *Class SwingUtilities — updateComponentTreeUI*. Java Platform, Standard Edition 8 API Specification. https://docs.oracle.com/javase/8/docs/api/javax/swing/SwingUtilities.html#updateComponentTreeUI-java.awt.Component-

Cleverence. *Javax.swing in Java SE 7: Complete Guide to Building Modern Desktop UIs*. https://www.cleverence.com/articles/oracle-documentation/javax-swing-java-platform-se-7-4827/

CSDN. *Java外观皮肤包定制与UI美化实战*. https://blog.csdn.net/weixin_42355400/article/details/152283020