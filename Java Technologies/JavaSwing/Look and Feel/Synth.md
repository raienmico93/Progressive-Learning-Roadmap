# Synth — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition.** **Synth** is a skinnable Look and Feel (L&F) for Java Swing in which **all painting is delegated** to a separate, configurable skin. Unlike Metal, Nimbus, or Windows — which ship with a fixed visual design — Synth provides **no default look of its own**. You must supply the entire visual definition, either through an **XML configuration file** or a **programmatic `SynthStyleFactory`**. This makes Synth the most flexible L&F in the Java platform, but also the most demanding: nothing appears unless you define it.

**Technical Definition.** `javax.swing.plaf.synth` is the package that implements Synth. Each `ComponentUI` implementation in Synth associates itself with one **`SynthStyle` per `Region`**. A `SynthStyle` is used to access all style-related properties: fonts, colors, and other component properties. In addition, `SynthStyle`s are used to obtain **`SynthPainter`s** for painting the background, border, focus, and other portions of a component. The `ComponentUI`s obtain `SynthStyle`s from a **`SynthStyleFactory`**. A `SynthStyleFactory` can be provided directly via `SynthLookAndFeel.setStyleFactory(SynthStyleFactory)` or indirectly via `SynthLookAndFeel.load(InputStream, Class<?>)`. Synth breaks many `JComponent`s down into smaller pieces identified by the type-safe `Region` enumeration (e.g., `JTabbedPane` contains `Region.TABBED_PANE`, `Region.TABBED_PANE_CONTENT`, `Region.TABBED_PANE_TAB`, etc.). `SynthLookAndFeel` extends `BasicLookAndFeel` and is the class installed via `UIManager.setLookAndFeel()`. All Synth configuration and GUI construction must occur on the Event Dispatch Thread (EDT).

**Beginner-Friendly Explanation.** **Synth** is Swing's **"build-your-own-skin"** Look and Feel. Whereas Metal and Nimbus come with pre-designed buttons, scroll bars, and text fields, Synth gives you **a blank canvas** — you define exactly how every component looks and behaves, down to the color of a pressed button, the font of a tab, and the image used for a border. You do this primarily through an **XML file** (the "synth file") that declares styles, binds them to component regions, and specifies painters, colors, fonts, icons, and properties. Think of Synth as a **professional skinning engine** for Swing: you draw the artwork (or write the XML), and Synth applies it consistently to every component in your application.

---

### Key Characteristics

1. **No default look.** "Synth does not provide a default look." You must specify a file or provide a `SynthStyleFactory`; otherwise, components render without styling.
2. **All painting is delegated.** Synth delegates painting to `SynthPainter`s obtained from `SynthStyle`s. You can supply painters programmatically or through XML (`<painter>`, `<imagePainter>`).
3. **Region-based styling.** Every component is broken into one or more `Region`s (e.g., `Region.BUTTON`, `Region.TEXT_FIELD`, `Region.TABBED_PANE_TAB`). Styles bind to regions via regular expressions.
4. **State-aware styles.** Styles can define different visual properties for component states: `ENABLED`, `DISABLED`, `PRESSED`, `MOUSE_OVER`, `FOCUSED`, `SELECTED`, and combinations like `SELECTED AND PRESSED`.
5. **XML-driven configuration.** The primary configuration mechanism is an XML file validated against `synth.dtd`. It supports styles, state elements, colors, fonts, images, painters, insets, opaque, and property elements.
6. **Programmatic alternative.** For dynamic or computed skins, you can implement `SynthStyleFactory` and set it directly on `SynthLookAndFeel`, bypassing XML entirely.
7. **Style merging and inheritance.** Styles can be cloned (`clone="..."`) and merged. Multiple styles can bind to the same region; later definitions take precedence.
8. **Performance considerations.** Complex painters and large images can affect repaint performance; caching and insets alignment are important best practices.
9. **EDT discipline.** All Synth configuration and GUI construction must occur on the EDT, as Swing is not thread-safe.
10. **`SynthLookAndFeel` extends `BasicLookAndFeel`.** It inherits the basic L&F infrastructure and replaces the default UI delegates with Synth-specific ones.

---

### Prerequisites

- **Java language fundamentals:** classes, inheritance, interfaces, and XML basics.
- **Swing fundamentals:** `JComponent`, `UIManager`, `LookAndFeel`, and the EDT.
- **Understanding of the pluggable L&F architecture and `ComponentUI` delegates.**
- **Familiarity with `UIDefaults` and the `UIResource` convention.**
- **Basic Java 2D painting:** `Graphics2D`, `Image`, and `SynthPainter`.
- **A JDK (version 8 or later).** Synth has been available since Java 5.

---

### Related Programming Areas

- **`javax.swing.plaf.synth.SynthLookAndFeel`** — the L&F class.
- **`javax.swing.plaf.synth.SynthStyle`** — holds the style properties for a region.
- **`javax.swing.plaf.synth.SynthStyleFactory`** — creates `SynthStyle`s per component and region.
- **`javax.swing.plaf.synth.SynthPainter`** — paints portions of components.
- **`javax.swing.plaf.synth.Region`** — the type-safe enumeration of component regions.
- **`javax.swing.plaf.synth.SynthContext`** — carries the component, region, and state.
- **`javax.swing.plaf.synth.ColorType`** — background, foreground, focus, and other color types.
- **`javax.swing.plaf.synth.SynthConstants`** — state constants (`ENABLED`, `PRESSED`, etc.).
- **`SynthIcon`** — interface for icons that paint with a `SynthContext`.
- **`UIManager` and `BasicLookAndFeel`** — the installation and base-class infrastructure.

---

## 1. Synth — The Concept

### Definitions

**Core Definition.** Synth is a skinnable L&F in which all painting is delegated to external configuration. It provides the engine but not the look.

**Technical Definition.** Synth's architecture revolves around `SynthStyle`, `SynthPainter`, and `SynthStyleFactory`. Each `ComponentUI` asks the factory for a style, which in turn provides colors, fonts, and painters. The style is state-sensitive: the same style returns different properties depending on whether the component is enabled, pressed, hovered, focused, or selected.

**Beginner-Friendly Explanation.** Synth is the **engine** that applies your skin. You provide the skin (XML or `SynthStyleFactory`), and Synth makes every button, label, and table follow it.

---

### Purposes

- To create fully custom L&Fs without writing `ComponentUI` subclasses for every component.
- To enable branded, skin-driven UIs that can be changed by swapping an XML file.
- To support fine-grained, state-aware styling for every component region.
- To provide a declarative alternative to imperative `BasicLookAndFeel` subclassing.
- To allow programmatic skins for dynamic or computed visuals.

---

### Syntax Rules and Structure

#### Complete General Syntax (Loading a Synth Skin)

```java
SynthLookAndFeel laf = new SynthLookAndFeel();
laf.load(MyClass.class.getResourceAsStream("laf.xml"), MyClass.class);
UIManager.setLookAndFeel(laf);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `new SynthLookAndFeel()` | Creates the Synth L&F instance. |
| `load(InputStream, Class)` | Loads the XML skin, using the class as the resource base. |
| `UIManager.setLookAndFeel(laf)` | Installs Synth as the current L&F. |

#### Syntax Rules

- `load` must be called before installation.
- The `Class` argument is used to resolve relative resource paths in the XML.
- `load` throws `ParseException`; wrap in `try`/`catch`.
- All configuration must occur on the EDT.

#### Constraints and Limitations

- Synth provides no fallback look; unstyled regions may paint nothing.
- XML errors (missing required attributes, wrong types) cause `ParseException`.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal Synth Installation

```java
import javax.swing.*;
import javax.swing.plaf.synth.SynthLookAndFeel;
import java.text.ParseException;

public class SynthMinimal {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            SynthLookAndFeel laf = new SynthLookAndFeel();               // 1. Create
            try {
                laf.load(SynthMinimal.class.getResourceAsStream("synth.xml"),
                         SynthMinimal.class);                             // 2. Load
                UIManager.setLookAndFeel(laf);                            // 3. Install
            } catch (ParseException e) {
                e.printStackTrace();
            }
            JFrame frame = new JFrame("Synth Minimal");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Skinned Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window whose button appearance is entirely determined by `synth.xml`. If the XML is missing or incomplete, the button may appear unstyled.

**Why the code produces this result.** Synth reads the XML, builds a `SynthStyleFactory`, and applies the resulting styles to the components.

---

### Real-World Cases

- **Branded desktop applications** with a unique corporate skin.
- **Themed utilities** where the skin is shipped as an external XML file.
- **Games and kiosk UIs** requiring pixel-perfect custom controls.

---

## 2. Synth L&F — SynthLookAndFeel

### Definitions

**Core Definition.** `SynthLookAndFeel` is the concrete L&F class that provides the basis for creating a customized look and feel through delegated painting.

**Technical Definition.** `public class SynthLookAndFeel extends BasicLookAndFeel`. It is the entry point for Synth. You must either provide a configuration file via `load(InputStream, Class)` or provide your own `SynthStyleFactory` via `setStyleFactory(SynthStyleFactory)`. `SynthLookAndFeel` extends `BasicLookAndFeel` and is the direct superclass of `NimbusLookAndFeel`.

**Beginner-Friendly Explanation.** `SynthLookAndFeel` is the **class you install** to enable Synth. You load a skin into it, then give it to `UIManager`.

---

### Purposes

- To provide the installable L&F object for Synth.
- To load XML skins and build the internal style factory.
- To allow programmatic `SynthStyleFactory` injection for dynamic skins.
- To serve as the base class for derived L&Fs such as Nimbus.

---

### Syntax Rules and Structure

#### Complete General Syntax (Key Methods)

```java
SynthLookAndFeel laf = new SynthLookAndFeel();
laf.load(InputStream, Class<?>);
laf.setStyleFactory(SynthStyleFactory);
UIManager.setLookAndFeel(laf);
```

#### Component Breakdown

| Method | Purpose |
|--------|---------|
| `load(InputStream, Class<?>)` | Loads an XML skin file. |
| `load(URL)` | Loads an XML skin from a URL. |
| `setStyleFactory(SynthStyleFactory)` | Provides a programmatic style factory. |
| `getStyleFactory()` | Returns the current factory. |

#### Constraints and Limitations

- `SynthLookAndFeel` is serializable only as a side effect of extending `BasicLookAndFeel`; serialization is not intended.

---

### Annotated Code Example

```java
SynthLookAndFeel laf = new SynthLookAndFeel();
laf.load(new URL("file:///C:/java/synth/laf/laf.xml"));              // 1. URL
UIManager.setLookAndFeel(laf);
```

**Expected Output.** Synth is installed with the skin loaded from the given URL.

---

## 3. Synth XML — The File Format

### Definitions

**Core Definition.** The Synth XML file format (validated against `synth.dtd`) is a declarative language for specifying every aspect of a Synth-based L&F: styles, states, colors, fonts, painters, images, icons, insets, and component properties.

**Technical Definition.** `<!ELEMENT synth ((%beansPersistance;) | style | bind | font | color | imagePainter | imageIcon | inputMap | defaultsProperty)*>`. The root `<synth>` element contains `<style>`, `<bind>`, `<font>`, `<color>`, `<imagePainter>`, `<imageIcon>`, and `<property>` elements. A `<style>` element corresponds to a `SynthStyle` and may contain `<state>` children that override properties for specific component states. A `<bind>` element associates a style with a region or component name via a regular expression.

**Beginner-Friendly Explanation.** The Synth XML file is the **skin definition language**. You declare a style, give it an ID, fill it with colors, fonts, and painters, and then bind it to a component region (e.g., "apply this style to all buttons"). You can override properties for different states (enabled, pressed, hovered, etc.).

---

### Purposes

- To define the complete appearance of every component region.
- To separate the skin from Java code for easier maintenance and swapping.
- To enable designers to modify the UI without recompiling.
- To support state-aware styling declaratively.

---

### Syntax Rules and Structure

#### Complete General Syntax (Minimal Style and Bind)

```xml
<synth>
    <style id="buttonStyle">
        <opaque value="true"/>
        <insets top="4" left="6" bottom="4" right="6"/>
        <font name="Dialog" size="12"/>
        <state>
            <color value="#D8D987" type="BACKGROUND"/>
            <color value="RED" type="FOREGROUND"/>
        </state>
        <state value="PRESSED">
            <color value="#B8B967" type="BACKGROUND"/>
        </state>
    </style>
    <bind style="buttonStyle" type="region" key="Button"/>
</synth>
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `<style id="...">` | Defines a style with a unique ID. |
| `<opaque value="true"/>` | Makes the region paint its background. |
| `<insets .../>` | Defines padding around the content. |
| `<font .../>` | Sets the font (name, size, style). |
| `<state value="...">` | Overrides properties for a specific state. |
| `<color value="..." type="..."/>` | Sets a color (BACKGROUND, FOREGROUND, FOCUS, etc.). |
| `<bind style="..." type="region" key="..."/>` | Binds a style to a region via a regex. |

#### Syntax Rules

- Every style should ideally be bound to a region; unbounded styles paint nothing.
- `type="region"` uses the region name; `type="name"` uses `component.getName()`.
- State values include `ENABLED`, `DISABLED`, `PRESSED`, `MOUSE_OVER`, `FOCUSED`, `SELECTED`, `DEFAULT`, combined with `AND`.
- Later styles and states take precedence during merging.

#### Constraints and Limitations

- The DTD is not validating; only missing required attributes or wrong types cause failures.
- Not all components support all states (e.g., `JPanel` only supports `ENABLED` and `DISABLED`).

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Minimal XML Skin

**File: `synth.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<synth>
    <style id="basic">
        <opaque value="true"/>
        <font name="Dialog" size="14"/>
        <state>
            <color value="#F0F0F0" type="BACKGROUND"/>
            <color value="#333333" type="FOREGROUND"/>
        </state>
        <state value="PRESSED">
            <color value="#C0C0C0" type="BACKGROUND"/>
        </state>
    </style>
    <bind style="basic" type="region" key=".*"/>
</synth>
```

**Expected Output.** Every region gets a light-gray background, dark text, and a darker background when pressed.

**Why the code produces this result.** The `.*` regex binds the style to **all** regions, providing a fallback look.

---

#### Example 2: Image Painter for a Button

**File: `synth-button.xml`**

```xml
<synth>
    <style id="button">
        <insets top="4" left="8" bottom="4" right="8"/>
        <state>
            <imagePainter path="button-bg.png" sourceInsets="4 8 4 8"/>
        </state>
    </style>
    <bind style="button" type="region" key="Button"/>
</synth>
```

**Expected Output.** The button's background is painted using the 9-slice image `button-bg.png`, scaled to fit the button.

---

### Real-World Cases

- **Corporate branding skins** with logo-based buttons.
- **Game UIs** with custom-drawn panels and controls.
- **Accessibility themes** with high-contrast colors and enlarged fonts.

---

## 4. Synth — Programmatic Configuration

### Definitions

**Core Definition.** Programmatic Synth configuration uses `SynthStyleFactory` and `SynthStyle` implementations to supply styles without an XML file.

**Technical Definition.** `public abstract class SynthStyleFactory`. A custom factory overrides `getStyle(JComponent c, Region id)` to return a `SynthStyle` for a given component and region. `SynthStyle` provides `getColor`, `getFont`, `getInsets`, and `getPainter`, and may be built with a `SynthStyle` builder or by subclassing.

**Beginner-Friendly Explanation.** Instead of XML, you can write Java code that tells Synth how to style each region. This is useful when styles are computed dynamically or when you prefer code over XML.

---

### Purposes

- To supply styles computed at runtime (e.g., from a database or theme engine).
- To avoid XML parsing overhead.
- To support conditional styling based on application state.
- To integrate Synth with existing programmatic theming systems.

---

### Syntax Rules and Structure

#### Complete General Syntax

```java
class MyStyleFactory extends SynthStyleFactory {
    @Override
    public SynthStyle getStyle(JComponent c, Region id) {
        if (id == Region.BUTTON) return buttonStyle;
        return defaultStyle;
    }
}
SynthLookAndFeel laf = new SynthLookAndFeel();
laf.setStyleFactory(new MyStyleFactory());
UIManager.setLookAndFeel(laf);
```

#### Component Breakdown

| Element | Meaning |
|---------|---------|
| `SynthStyleFactory` | Abstract factory for `SynthStyle`s. |
| `getStyle(JComponent, Region)` | Returns the style for the component and region. |
| `SynthStyle` | Holds colors, fonts, and painters. |

#### Constraints and Limitations

- Programmatic styles require more Java code than XML.
- You must handle every region you want to style; unstyled regions will have no appearance.

---

### Annotated Code Example

```java
class MyStyleFactory extends SynthStyleFactory {
    private final SynthStyle buttonStyle = new SynthStyle() { ... };
    @Override
    public SynthStyle getStyle(JComponent c, Region id) {
        return id == Region.BUTTON ? buttonStyle : null;              // 1. Region
    }
}
```

**Expected Output.** Buttons receive `buttonStyle`; other regions are unstyled.

---

## 5. Synth Best Practices

### Definitions

**Core Definition.** Synth best practices are the conventions and patterns that ensure Synth-based skins are complete, performant, maintainable, and thread-safe.

**Technical Definition.** Best practices include: providing a **backing style** bound to `.*` so unstyled regions still paint; wrapping XML loading in `try`/`catch` for `ParseException`; aligning `<insets>` with `sourceInsets` for image painters; caching painters and images; loading and configuring Synth on the EDT; and preferring one-style-per-component until inheritance models are well understood.

**Beginner-Friendly Explanation.** Best practices are the rules of thumb that prevent blank components, parsing errors, and sluggish repaints. They include things like "always provide a fallback style," "match insets to image slices," and "do all Synth work on the EDT."

---

### Purposes

- To ensure every component has a visible appearance.
- To avoid runtime `ParseException` crashes.
- To maintain smooth repaint performance.
- To keep skins maintainable and easy to evolve.
- To respect Swing's thread-safety rules.

---

### Syntax Rules and Structure

#### Complete General Best-Practice Skeleton

```xml
<synth>
    <!-- 1. Backing style for all regions -->
    <style id="backing">
        <opaque value="true"/>
        <font name="Dialog" size="14"/>
        <state>
            <color value="#F0F0F0" type="BACKGROUND"/>
            <color value="#333333" type="FOREGROUND"/>
        </state>
    </style>
    <bind style="backing" type="region" key=".*"/>

    <!-- 2. Specific component styles -->
    <style id="button" clone="backing">
        <state value="PRESSED">
            <color value="#C0C0C0" type="BACKGROUND"/>
        </state>
    </style>
    <bind style="button" type="region" key="Button"/>
</synth>
```

#### Component Breakdown

| Best Practice | Implementation |
|---------------|----------------|
| Backing style | `<bind style="backing" type="region" key=".*"/>` |
| Exception handling | `try { laf.load(...); } catch (ParseException e) { ... }` |
| Insets alignment | `<insets>` matches `sourceInsets` |
| Image caching | Reuse `imageIcon` elements; avoid regenerating images in painters |
| EDT | `SwingUtilities.invokeLater` for load and install |
| One-style-per-component | Start simple; add inheritance later |

#### Syntax Rules

- **Always provide a backing style** bound to `.*` so that every region paints something.
- **Always wrap `load` in `try`/`catch`** for `ParseException`.
- **Always align `<insets>` with `sourceInsets`** when using `<imagePainter>`.
- **Always load and configure Synth on the EDT.**
- **Prefer one-style-per-component** until you are comfortable with style merging.

#### Constraints and Limitations

- Synth XML parsing is not validating; only missing required attributes fail.
- Complex painters can degrade repaint performance; cache where possible.

---

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Best-Practice Synth Installation

```java
import javax.swing.*;
import javax.swing.plaf.synth.SynthLookAndFeel;
import java.text.ParseException;

public class BestPracticeSynth {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            SynthLookAndFeel laf = new SynthLookAndFeel();               // 1. Create
            try {
                laf.load(BestPracticeSynth.class.getResourceAsStream(
                         "best-practice-synth.xml"), BestPracticeSynth.class); // 2. Load
                UIManager.setLookAndFeel(laf);                            // 3. Install
            } catch (ParseException e) {
                System.err.println("Failed to load skin: " + e);        // 4. Handle
            }
            JFrame frame = new JFrame("Best Practice Synth");
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.add(new JButton("Skinned Button"));
            frame.pack();
            frame.setLocationRelativeTo(null);
            frame.setVisible(true);
        });
    }
}
```

**Expected Output.** A window with a button styled by `best-practice-synth.xml`, with a backing style ensuring every region is painted.

---

#### Example 2: Common Mistake — Missing Backing Style

```xml
<synth>
    <style id="button">
        <state>
            <color value="BLUE" type="BACKGROUND"/>
        </state>
    </style>
    <bind style="button" type="region" key="Button"/>
</synth>
```

**Expected Output.** Buttons are blue, but labels, panels, and text fields may paint nothing or appear broken because no style is bound to their regions.

**Why this is wrong.** Without a backing style bound to `.*`, unstyled regions have no visual definition. The fix is to add a backing style and bind it first.

---

## References

Oracle. *Package javax.swing.plaf.synth*. Java Platform, Standard Edition 11 API Specification. https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/plaf/synth/package-summary.html

Oracle. *File Format*. Java Platform, Standard Edition 27 API Specification. https://download.java.net/java/early_access/jdk27/docs/api/java.desktop/javax/swing/plaf/synth/doc-files/synthFileFormat.html

Oracle. *Class SynthLookAndFeel*. Java Platform, Standard Edition 17 API Specification. https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/javax/swing/plaf/synth/SynthLookAndFeel.html

Oracle. *A Synth Example*. The Java Tutorials. https://docs.oracle.com/javase/tutorial/uiswing/lookandfeel/synthExample.html

Oracle. *Class SynthStyleFactory*. Java Platform, Standard Edition API Specification. https://download.java.net/java/early_access/jdk27/docs/api/java.desktop/javax/swing/plaf/synth/SynthStyleFactory.html

Oracle. *Class SynthPainter*. Java Platform, Standard Edition API Specification. https://download.java.net/java/early_access/jdk27/docs/api/java.desktop/javax/swing/plaf/synth/SynthPainter.html

Oracle. *Synth Look and Feel — Grokipedia*. https://grokipedia.com/page/Synth_Look_and_Feel

OpenJDK. *Synth File Format — OpenJDK 9 Source*. https://android.googlesource.com/platform/toolchain/jdk/jdk9_jdk/

Stack Overflow. *Set JLabel icon in Synth XML*. https://stackoverflow.com/questions/

Stack Overflow. *Java-LAF: How to create Decoration using Synth-XML?*. https://stackoverflow.com/questions/