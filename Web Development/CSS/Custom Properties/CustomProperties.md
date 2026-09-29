# CSS Custom Properties — Comprehensive Cheat Sheet Research

---

## Topic Overview

### Definitions

**Core Definition:** CSS Custom Properties (commonly called CSS Variables) are author-defined properties that allow developers to store and reuse values throughout a stylesheet. They are declared using a two-dash prefix (`--`) and accessed using the `var()` function, enabling dynamic, cascading, and runtime-modifiable values in CSS.

**Technical Definition:** Custom properties are a CSS feature defined in the CSS Custom Properties for Cascading Variables Module Level 1. They are special properties that allow authors to assign arbitrary values to a property name for reuse in other properties via the `var()` function. Unlike preprocessor variables (Sass, Less), custom properties are live, cascading, and can be modified at runtime using JavaScript or scoped by selector. The `@property` at-rule (CSS Properties and Values API Level 1) extends custom properties with type checking (`syntax`), initial values (`initial-value`), and inheritance control (`inherits`). The CSS Typed Object Model allows programmatic registration via `CSS.registerProperty()`.

**Beginner-Friendly Explanation:** A CSS custom property is like a labelled box where you store a value you want to reuse. You give the box a name starting with `--` (like `--primary-color`) and put a value inside. Then, anywhere in your CSS, you can pull that value out using `var(--primary-color)`. The magic is that you can change the value in one place, and every element that references it updates automatically. You can even change the value at runtime with JavaScript, making it perfect for theming, dark mode, and dynamic styling.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Two-dash prefix** | Custom properties must begin with `--`; any other name is a regular CSS property. |
| **Case-sensitive** | Custom property names are case-sensitive (`--color` and `--Color` are different). |
| **Cascading** | Custom properties cascade and inherit like regular CSS properties. |
| **Runtime-modifiable** | Values can be changed via JavaScript using `setProperty()`. |
| **Fallback support** | `var()` accepts one or more fallback values. |
| **Type-safe with `@property`** | The `@property` at-rule adds type checking, initial values, and inheritance control. |
| **Invalid at computed-value time** | Invalid var() usage causes the property to become `unset` (inherit or initial). |

---

### Prerequisites

Before studying CSS Custom Properties, you should understand:

- **CSS Syntax** — selectors, properties, values, and the cascade.
- **CSS Inheritance** — how properties inherit from parent to child.
- **CSS Functions** — basic function syntax like `calc()` and `rgb()`.
- **CSS At-Rules** — `@media`, `@supports`, and `@property`.
- **Basic JavaScript** — for runtime modification and `CSS.registerProperty()`.

---

### Related Programming Areas

- **Theming Systems** — dark mode, brand switching, and dynamic themes.
- **Design Tokens** — storing colour, spacing, typography, and motion values.
- **Responsive Design** — using custom properties with media queries.
- **CSS Architecture** — reducing repetition and improving maintainability.
- **Web Animations API** — animating custom properties with `@property`.

---

### Core Concepts / Features

1. Variable Declaration: `--` Prefix, Case-Sensitivity, and Permitted Value Types
2. The `var()` Function: Consumption, Nesting, and Fallback Values
3. The `@property` At-Rule: Type Checking, Initial Defaults, and Inheritance
4. CSS Typed OM and Registration: `CSS.registerProperty()`
5. Guaranteed Invalid Values: "Invalid at Computed-Value Time"

---

## 1. Variable Declaration: Syntax Patterns Using the `--` Prefix

### Definitions

**Core Definition:** A custom property is declared by writing a name that begins with `--`, followed by a colon and a value. The declaration must be inside a CSS rule (a selector block), and the value can be almost any CSS value.

**Technical Definition:** Custom properties are ordinary properties, so they can be declared on any element, are resolved with the normal inheritance and cascade rules, can be made conditional with `@media` and other conditional rules, can be used in HTML's `style` attribute, can be read or set using the CSSOM, and so on. The `--` prefix distinguishes custom properties from standard CSS properties. Any property name that starts with `--` is treated as a custom property. Custom properties are case-sensitive and accept almost any token sequence as their value, including values that would normally be invalid for other properties.

**Beginner-Friendly Explanation:** You declare a custom property just like any other CSS property, but the name must start with `--`. For example, `--main-color: #3498db;` creates a variable called `--main-color`. The value can be anything: a colour, a length, a number, a whole rule, even an empty value. Because custom property names are case-sensitive, `--Main-Color` is a different variable from `--main-color`. You can declare the same variable on different selectors, and each scope gets its own value.

---

### Purposes

- To store reusable values that can be referenced throughout a stylesheet.
- To enable runtime theming and dynamic value changes.
- To reduce repetition and improve maintainability of CSS.
- To create scoped values that cascade and inherit through the DOM.
- To provide a foundation for design token systems.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    --custom-property-name: <declaration-value>;
}
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `--` | The required prefix for custom properties. | `--` |
| `<custom-property-name>` | A case-sensitive name (can contain letters, digits, hyphens, underscores). | `--main-color`, `--spacing-md` |
| `<declaration-value>` | Almost any CSS value, including empty values. | `#3498db`, `16px`, `1fr 2fr` |

#### Permitted Value Types

| Value Type | Example | Notes |
|---|---|---|
| Colour | `#3498db`, `rgb(52, 152, 219)` | Any valid colour value. |
| Length | `16px`, `1.5rem`, `50%` | Any valid length or percentage. |
| Number | `1.5`, `42` | Any unitless number. |
| Keyword | `auto`, `flex-start` | Any valid keyword. |
| Multiple values | `1fr 2fr`, `0 2px 4px rgba(0,0,0,0.1)` | Space-separated lists. |
| Empty value | `--empty: ;` | Valid but unusual; resolves to the guaranteed-invalid value. |

#### Syntax Rules

1. Custom property names are **case-sensitive**: `--color` and `--Color` are distinct.
2. Custom properties must be declared inside a CSS rule block (or the `style` attribute).
3. The value can be almost any token sequence, including values invalid for other properties.
4. Custom properties inherit by default from parent to child.
5. Custom properties can be declared on any element, including the `:root` pseudo-class.
6. The `var()` function is used to reference the value of a custom property.
7. Custom properties are resolved at computed-value time, not at parse time.

#### Constraints and Limitations

- **No preprocessor-style math** — custom properties store values; they do not perform calculations unless wrapped in `calc()`.
- **Empty value ambiguity** — an empty custom property value is treated as invalid at computed-value time.
- **Case-sensitivity trap** — typos in case create distinct variables that silently fail.
- **No `!important` in declaration value** — the `!important` flag is not allowed inside a custom property declaration.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Declaration and Usage

**HTML File (`custom-props-basic.html`):**

```html
<!DOCTYPE html>
<!-- Declares the document as HTML5 -->
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- Ensures proper character encoding -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Properties Basic</title>
    <!-- Links the external CSS file -->
    <link rel="stylesheet" href="custom-props-basic.css">
</head>
<body>
    <!-- Two cards that inherit the same custom property values -->
    <div class="card">Card 1</div>
    <div class="card card-dark">Card 2 (overrides variables)</div>
</body>
</html>
```

**CSS File (`custom-props-basic.css`):**

```css
/* Declare custom properties on :root for global access */
:root {
    --primary-color: #3498db;
    --text-color: #1a1a1a;
    --card-bg: #ffffff;
    --spacing: 1.5rem;
    --radius: 12px;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.card {
    /* Consume the custom properties using var() */
    background-color: var(--card-bg);
    color: var(--text-color);
    padding: var(--spacing);
    border-radius: var(--radius);
    margin-bottom: 16px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    font-weight: bold;
    text-align: center;
}

/* Override custom properties on a specific element */
.card-dark {
    --card-bg: #1a1a1a;
    --text-color: #e8e8e8;
    /* The card inherits these new values for its subtree */
}
```

**Step-by-Step Setup Guide:**

1. Create a project folder.
2. Save the HTML code as `custom-props-basic.html`.
3. Save the CSS code as `custom-props-basic.css` in the same folder.
4. Open `custom-props-basic.html` in a web browser.
5. Observe that Card 1 has a white background with dark text, and Card 2 has a dark background with light text.

**Expected Output:** Two cards. The first uses the default values from `:root`. The second overrides `--card-bg` and `--text-color` to create a dark theme for itself.

**Why This Works:** The `:root` selector declares global custom properties. Each `.card` uses `var()` to consume them. The `.card-dark` class overrides the values locally, and because custom properties inherit, all descendants of that card see the new values. This demonstrates the cascade and scope of custom properties.

---

### Real-World Cases

- **Design tokens:** Declaring `--colour-primary`, `--spacing-md`, and `--font-size-base` globally.
- **Dark mode:** Overriding colour custom properties inside a `@media (prefers-color-scheme: dark)` block.
- **Component scoping:** Declaring component-specific custom properties on the component root.
- **Responsive spacing:** Changing `--spacing` values at different breakpoints.

---

## 2. The `var()` Function: Consuming Properties, Nesting Variable Lookups, and Defining Strict Multi-Tiered Fallback Values

### Definitions

**Core Definition:** The `var()` function is used to reference the value of a custom property in a CSS declaration. It accepts the custom property name and an optional fallback value that is used if the custom property is not defined.

**Technical Definition:** The `var()` function can be used to insert the value of a custom property instead of any part of a value of another property. The function accepts two arguments: the first is the custom property name (prefixed with `--`), and the optional second argument is a fallback value. If the custom property is not defined or is invalid, the fallback is used. If no fallback is provided and the custom property is invalid, the property becomes invalid at computed-value time (equivalent to `unset`). The fallback can itself be a `var()` function, enabling multi-tiered fallback chains.

**Beginner-Friendly Explanation:** The `var()` function is how you pull a value out of a custom property. You write `var(--my-variable)`, and the browser substitutes the value stored in `--my-variable`. If that variable is not defined, the browser uses whatever you provide as the second argument: `var(--my-variable, 16px)`. You can even nest fallbacks: `var(--a, var(--b, 16px))` means "use `--a`, but if it is missing, use `--b`, and if that is also missing, use `16px`."

---

### Purposes

- To consume the value of a custom property in any CSS declaration.
- To provide fallback values when custom properties are undefined.
- To create multi-tiered fallback chains for robust theming.
- To nest variable lookups for complex value resolution.
- To enable runtime value changes without modifying the consuming property.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
selector {
    property: var(--custom-property-name);
    property: var(--custom-property-name, <fallback-value>);
    property: var(--a, var(--b, <fallback>));
}
```

#### Component Breakdown

| Component | Description | Example |
|---|---|---|
| `var()` | The function keyword. | `var()` |
| `--custom-property-name` | The custom property to look up. | `--primary-color` |
| `<fallback-value>` | Optional fallback if the property is invalid. | `#3498db` |

#### Fallback Behaviour

| Scenario | Result |
|---|---|
| Custom property is defined and valid. | Uses the custom property value. |
| Custom property is undefined. | Uses the fallback value (if provided). |
| Custom property is undefined and no fallback. | Property becomes invalid at computed-value time. |
| Custom property value is invalid for the property. | Property becomes invalid at computed-value time. |

#### Syntax Rules

1. The first argument to `var()` must be a custom property name starting with `--`.
2. The fallback is everything after the first comma; it can contain commas itself.
3. Fallbacks can be nested: `var(--a, var(--b, fallback))`.
4. If the custom property is defined but its value is empty, the fallback is used.
5. `var()` can be used in any property value, including shorthand properties.
6. `var()` cannot be used in property names or selectors.
7. The fallback is only used if the custom property is invalid at computed-value time.

#### Constraints and Limitations

- **Invalid at computed-value time** — if a custom property is invalid for the property it is used in, the entire declaration becomes invalid, not just the var().
- **No fallback for invalid values** — the fallback is only used if the custom property is undefined or empty, not if its value is invalid for the consuming property.
- **Comma ambiguity** — commas inside the fallback are treated as part of the fallback value.
- **No `var()` in media queries** — `var()` cannot be used inside media query conditions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Tiered Fallbacks

**HTML File (`var-fallbacks.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>var() Fallbacks</title>
    <link rel="stylesheet" href="var-fallbacks.css">
</head>
<body>
    <div class="box box-a">Uses --color-a</div>
    <div class="box box-b">Uses --color-b (fallback)</div>
    <div class="box box-c">Uses fallback colour</div>
</body>
</html>
```

**CSS File (`var-fallbacks.css`):**

```css
:root {
    /* --color-a is defined; --color-b is NOT defined */
    --color-a: #3498db;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    padding: 30px;
    border-radius: 12px;
    margin-bottom: 16px;
    color: white;
    font-weight: bold;
    text-align: center;
}

.box-a {
    /* Uses --color-a, which is defined */
    background-color: var(--color-a, #e74c3c);
}

.box-b {
    /* --color-b is not defined, so the fallback #e74c3c is used */
    background-color: var(--color-b, #e74c3c);
}

.box-c {
    /* Multi-tiered: --color-a is defined, so it wins */
    background-color: var(--color-a, var(--color-b, #27ae60));
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `var-fallbacks.html` and CSS as `var-fallbacks.css`.
2. Open in a browser.
3. Observe: Box A is blue (`--color-a`), Box B is red (fallback), and Box C is blue (`--color-a` wins in the nested fallback).

**Expected Output:** Three boxes demonstrating different fallback scenarios. Box A uses a defined variable. Box B uses a fallback because the variable is undefined. Box C uses the first defined variable in a nested fallback chain.

**Why This Works:** The `var()` function checks if the custom property is defined. If it is, that value is used. If not, the fallback is evaluated. In nested fallbacks, each `var()` is evaluated in order until a defined value is found.

---

### Real-World Cases

- **Theme systems:** `var(--brand-color, #3498db)` for components that work with or without a brand theme.
- **Spacing scales:** `var(--spacing-md, 1rem)` for components with optional spacing overrides.
- **Typography:** `var(--font-heading, var(--font-body, sans-serif))` for multi-tiered font fallbacks.
- **Dark mode:** `var(--bg-color, white)` with overrides in a dark-mode media query.

---

## 3. The `@property` At-Rule: Defining Type Checking, Initial Defaults, and Inheritance Settings

### Definitions

**Core Definition:** The `@property` at-rule is a CSS feature that allows authors to register custom properties with explicit type definitions, initial values, and inheritance behaviour. It transforms an untyped custom property into a typed, animatable, and predictable one.

**Technical Definition:** The `@property` CSS at-rule is part of the CSS Properties and Values API Level 1. It allows developers to explicitly define their CSS custom properties, allowing for property type checking and constraining, setting default values, and defining whether a custom property can inherit values or not. The at-rule accepts three descriptors: `syntax` (the type of the property), `inherits` (whether the property inherits by default), and `initial-value` (the default value if the property is not set). Registered custom properties can be animated with transitions and keyframes, unlike unregistered custom properties.

**Beginner-Friendly Explanation:** Normally, a custom property is just a "box of text" — the browser does not know if it contains a colour, a length, or a number. The `@property` rule lets you tell the browser exactly what type of value it should hold. You write `@property --my-color { syntax: "<color>"; inherits: false; initial-value: #3498db; }`. Now the browser knows `--my-color` is a colour, it does not inherit by default, and its default value is blue. This makes the property safer, animatable, and more predictable.

---

### Purposes

- To define the type of a custom property for validation and animation.
- To set an initial value that is used when the property is undefined.
- To control whether the property inherits from parent to child.
- To enable animation and transition of custom properties.
- To prevent "invalid at computed-value time" errors by providing a valid default.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
@property --custom-property-name {
    syntax: "<syntax-string>";
    inherits: true | false;
    initial-value: <value>;
}
```

#### Component Breakdown

| Descriptor | Description | Values |
|---|---|---|
| `syntax` | The type of the custom property. | `"<color>"`, `"<length>"`, `"<number>"`, `"*"`, etc. |
| `inherits` | Whether the property inherits by default. | `true` or `false` |
| `initial-value` | The default value when the property is not set. | Any valid value for the syntax type. |

#### Common Syntax Strings

| Syntax String | Description | Example Values |
|---|---|---|
| `"<color>"` | Colour values. | `#3498db`, `rgb(0,0,0)` |
| `"<length>"` | Length values. | `16px`, `1.5rem` |
| `"<percentage>"` | Percentage values. | `50%` |
| `"<length-percentage>"` | Length or percentage. | `16px`, `50%` |
| `"<number>"` | Unitless numbers. | `1.5`, `42` |
| `"<integer>"` | Whole numbers. | `1`, `2`, `-3` |
| `"*"` | Any value (universal syntax). | Anything |

#### Syntax Rules

1. `syntax` is required when registering a property.
2. `inherits` is required and must be `true` or `false`.
3. `initial-value` is required unless the syntax is `"*"`.
4. Registered properties can be animated with `transition` and `@keyframes`.
5. Unregistered custom properties cannot be animated (the browser treats them as discrete).
6. The `initial-value` is used when the property is not defined in the cascade.
7. Registered properties still cascade and can be overridden like any custom property.

#### Constraints and Limitations

- **`initial-value` required** — for typed syntax, an `initial-value` must be provided or the registration fails.
- **`"*"` syntax** — the universal syntax does not support `initial-value` and does not enable animation.
- **Browser support** — `@property` is Baseline widely available (Chrome 85+, Safari 16.4+, Firefox 128+).
- **No fallback for invalid values** — if the assigned value does not match the syntax, the property falls back to the `initial-value` (or inherits if `inherits: true`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Registering and Animating a Custom Property

**HTML File (`property-rule.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@property Rule</title>
    <link rel="stylesheet" href="property-rule.css">
</head>
<body>
    <div class="gradient-box"></div>
</body>
</html>
```

**CSS File (`property-rule.css`):**

```css
/* Register a custom property for angle values */
@property --angle {
    syntax: "<angle>";
    inherits: false;
    initial-value: 0deg;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.gradient-box {
    width: 300px;
    height: 300px;
    border-radius: 20px;
    /* Use the custom property in a gradient */
    background: conic-gradient(
        from var(--angle),
        #3498db,
        #e74c3c,
        #3498db
    );
    /* Animate the custom property */
    animation: rotate-gradient 3s linear infinite;
}

@keyframes rotate-gradient {
    from { --angle: 0deg; }
    to   { --angle: 360deg; }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `property-rule.html` and CSS as `property-rule.css`.
2. Open in a modern browser that supports `@property`.
3. Observe the gradient box continuously rotating its conic gradient.

**Expected Output:** A square box with a rotating conic gradient that sweeps through blue, red, and back to blue.

**Why This Works:** The `@property --angle` rule registers `--angle` as an `<angle>` type with an initial value of `0deg`. Because the property is registered with a type, the browser can interpolate it smoothly during the animation. The `@keyframes rotate-gradient` animates `--angle` from `0deg` to `360deg`, which drives the rotation of the conic gradient.

---

### Real-World Cases

- **Animated gradients:** Registering `--angle` for rotating gradient backgrounds.
- **Progress indicators:** Registering `--progress` as a `<number>` for animated progress bars.
- **Theme tokens:** Registering `--brand-hue` for animatable colour themes.
- **Smooth transitions:** Registering colour properties for smooth hover transitions.

---

## 4. CSS Typed OM and Registration: Registering Type-Safe Variables Programmatically

### Definitions

**Core Definition:** The `CSS.registerProperty()` method is a JavaScript API that registers custom properties with type definitions, initial values, and inheritance behaviour — the programmatic equivalent of the `@property` at-rule.

**Technical Definition:** The `CSS.registerProperty()` static method is part of the CSS Properties and Values API. It allows developers to register custom properties programmatically with a `PropertyDefinition` dictionary containing the `name`, `syntax`, `inherits`, and `initialValue` properties. Once registered, the custom property becomes type-safe, can be animated, and is included in the CSS Typed OM, which provides typed access to CSS property values. Registration can also be done declaratively using the `@property` at-rule; both approaches achieve the same result.

**Beginner-Friendly Explanation:** `CSS.registerProperty()` is the JavaScript way to do what `@property` does in CSS. You call it with the property name, its type (like `"<color>"`), whether it inherits, and its default value. This is useful when you want to register properties dynamically, or when you are building a library and want to register properties from JavaScript. Once registered, the property behaves just like one registered with `@property`.

---

### Purposes

- To register custom properties programmatically from JavaScript.
- To enable type checking and animation for custom properties.
- To provide a fallback for browsers that support `@property` but not the CSS syntax.
- To integrate custom property registration into JavaScript build processes.
- To access typed custom property values via the CSS Typed OM.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
CSS.registerProperty({
    name: "--custom-property-name",
    syntax: "<syntax-string>",
    inherits: true | false,
    initialValue: <value>
});
```

#### Component Breakdown

| Property | Description | Example |
|---|---|---|
| `name` | The custom property name (must start with `--`). | `"--my-color"` |
| `syntax` | The type of the property. | `"<color>"` |
| `inherits` | Whether the property inherits. | `false` |
| `initialValue` | The default value. | `"#3498db"` |

#### Syntax Rules

1. `name` must be a string starting with `--`.
2. `syntax` must be a valid syntax string.
3. `inherits` must be a boolean.
4. `initialValue` is required for typed syntax.
5. The method returns `undefined`; registration is a side effect.
6. If the property is already registered, the call has no effect.
7. Browser support: Chrome 78+, Safari 16.4+, Firefox 128+.

#### Constraints and Limitations

- **No un-registration** — once registered, a property cannot be unregistered.
- **Global scope** — registration is global, not scoped to a document.
- **No dynamic syntax changes** — the syntax cannot be changed after registration.
- **Browser support** — not supported in Internet Explorer.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Registering a Custom Property from JavaScript

**HTML File (`register-property.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS.registerProperty()</title>
    <link rel="stylesheet" href="register-property.css">
</head>
<body>
    <div class="animated-box"></div>

    <script>
        // Register a custom property for rotation
        CSS.registerProperty({
            name: '--rotation',
            syntax: '<angle>',
            inherits: false,
            initialValue: '0deg'
        });
    </script>
</body>
</html>
```

**CSS File (`register-property.css`):**

```css
body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 60px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
}

.animated-box {
    width: 200px;
    height: 200px;
    background: linear-gradient(135deg, #3498db, #e74c3c);
    border-radius: 20px;
    /* Use the registered custom property */
    transform: rotate(var(--rotation));
    /* Animate it */
    animation: spin 2s linear infinite;
}

@keyframes spin {
    from { --rotation: 0deg; }
    to   { --rotation: 360deg; }
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `register-property.html` and CSS as `register-property.css`.
2. Open in a modern browser that supports `CSS.registerProperty()`.
3. Observe the gradient box continuously rotating.

**Expected Output:** A square box with a diagonal gradient that rotates 360 degrees continuously.

**Why This Works:** The JavaScript calls `CSS.registerProperty()` to register `--rotation` as an `<angle>` type with an initial value of `0deg`. This makes the property type-safe and animatable. The CSS animation drives the rotation by changing `--rotation` from `0deg` to `360deg`, which is applied via `transform: rotate()`.

---

### Real-World Cases

- **JavaScript libraries:** Registering custom properties for animations and themes.
- **Build tools:** Automating property registration during build.
- **Dynamic theming:** Registering colour properties from a theme configuration.
- **Web Components:** Registering properties for component-scoped animations.

---

## 5. Guaranteed Invalid Values: Understanding "Invalid at Computed-Value Time"

### Definitions

**Core Definition:** "Invalid at computed-value time" is the state of a CSS property when a `var()` reference cannot be resolved — either because the custom property is undefined and no fallback is provided, or because the custom property's value is invalid for the property it is used in. When this happens, the property is set to its inherited or initial value.

**Technical Definition:** If a property contains a `var()` function and the substitution cannot be performed (because the custom property is invalid or undefined without a fallback), the declaration is invalid at computed-value time. The property's value is then set to the property's inherited value if the property is inherited, or its initial value if not. This is different from a parse-time error: the declaration is valid at parse time (because `var()` is syntactically valid) but becomes invalid when the custom property is resolved. The guaranteed-invalid value is the value that results from resolving a `var()` with no fallback when the custom property is not defined.

**Beginner-Friendly Explanation:** When you write `color: var(--undefined-variable)`, the browser does not know at parse time that `--undefined-variable` does not exist. It only finds out when it computes the value. At that point, the declaration becomes "invalid at computed-value time," and the browser falls back to the property's inherited or initial value. This is why the fallback argument in `var()` is so important — it prevents the invalid state. Without a fallback, the property resets to its default, which can cause unexpected styling.

---

### Purposes

- To understand why undefined custom properties cause unexpected default styling.
- To provide fallbacks that prevent the invalid state.
- To debug custom property errors in DevTools.
- To use `@property` with `initial-value` to guarantee a valid default.
- To design robust theming systems that degrade gracefully.

---

### Syntax Rules and Structure

#### Complete General Syntax

```css
/* Invalid: --undefined is not defined and no fallback */
selector {
    color: var(--undefined); /* Invalid at computed-value time */
}

/* Valid: fallback provided */
selector {
    color: var(--undefined, #000000); /* Fallback used */
}

/* Valid: @property provides initial-value */
@property --defined {
    syntax: "<color>";
    inherits: false;
    initial-value: #000000;
}
selector {
    color: var(--defined); /* Uses #000000 */
}
```

#### Component Breakdown

| Scenario | Result |
|---|---|
| Custom property defined, value valid for property. | Property uses the custom property value. |
| Custom property defined, value invalid for property. | Property becomes invalid at computed-value time. |
| Custom property undefined, no fallback. | Property becomes invalid at computed-value time. |
| Custom property undefined, fallback provided. | Property uses the fallback. |
| Custom property registered with `@property`, undefined. | Property uses the `initial-value`. |

#### Syntax Rules

1. Invalid at computed-value time occurs when a `var()` reference cannot be resolved.
2. The property is set to its inherited value (if inherited) or initial value (if not).
3. A fallback in `var()` prevents the invalid state.
4. An `@property` registration with `initial-value` prevents the invalid state.
5. The `var()` function itself is always valid at parse time.
6. Unregistered custom properties with invalid values cause the consuming property to become invalid.
7. Registered custom properties with invalid values fall back to the `initial-value`.

#### Constraints and Limitations

- **Fallback only for undefined** — the fallback is not used if the custom property is defined but its value is invalid for the consuming property.
- **Cascade complexity** — determining which custom property is invalid can be difficult in large stylesheets.
- **DevTools support** — DevTools shows the computed value as "invalid" or the inherited/initial value.
- **No runtime error** — invalid at computed-value time does not throw a JavaScript error; it silently resets the property.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Invalid at Computed-Value Time Demonstration

**HTML File (`invalid-values.html`):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Invalid at Computed-Value Time</title>
    <link rel="stylesheet" href="invalid-values.css">
</head>
<body>
    <div class="box no-fallback">No fallback (invalid)</div>
    <div class="box with-fallback">With fallback (valid)</div>
    <div class="box registered">Registered property (valid)</div>
</body>
</html>
```

**CSS File (`invalid-values.css`):**

```css
/* Register a property with an initial-value */
@property --registered-color {
    syntax: "<color>";
    inherits: false;
    initial-value: #27ae60;
}

body {
    font-family: system-ui, sans-serif;
    margin: 0;
    padding: 40px;
    background-color: #f5f5f5;
}

.box {
    padding: 30px;
    border-radius: 12px;
    margin-bottom: 16px;
    color: white;
    font-weight: bold;
    text-align: center;
}

.no-fallback {
    /* --undefined-color is NOT defined and no fallback is provided.
       The property becomes invalid at computed-value time,
       so the background resets to transparent (initial value). */
    background-color: var(--undefined-color);
    color: #1a1a1a;
    border: 2px dashed #e74c3c;
}

.with-fallback {
    /* Fallback provided: uses #3498db */
    background-color: var(--undefined-color, #3498db);
}

.registered {
    /* Registered property with initial-value: uses #27ae60 */
    background-color: var(--registered-color);
}
```

**Step-by-Step Setup Guide:**

1. Save the HTML as `invalid-values.html` and CSS as `invalid-values.css`.
2. Open in a browser.
3. Observe:
   - Box 1: transparent background (invalid at computed-value time resets to initial).
   - Box 2: blue background (fallback used).
   - Box 3: green background (`@property` initial-value used).

**Expected Output:** Three boxes demonstrating the different outcomes of custom property resolution. Box 1 shows the invalid state (transparent background with a dashed border to highlight the issue). Box 2 uses a fallback. Box 3 uses a registered initial value.

**Why This Works:** Box 1 uses `var(--undefined-color)` without a fallback. Because the custom property is undefined, the declaration becomes invalid at computed-value time, and the `background-color` resets to its initial value (`transparent`). Box 2 provides a fallback. Box 3 uses a registered property with an `initial-value`, which prevents the invalid state.

---

### Real-World Cases

- **Debugging theming issues:** Understanding why a component is transparent when a theme variable is missing.
- **Robust design systems:** Providing fallbacks for all custom property references.
- **Graceful degradation:** Using `@property` initial values for critical properties.
- **DevTools inspection:** Checking the Computed panel to see if a property is invalid.

---

## References

- MDN Web Docs — Using CSS custom properties - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties
- MDN Web Docs — `var()` - https://developer.mozilla.org/en-US/docs/Web/CSS/var
- MDN Web Docs — `@property` - https://developer.mozilla.org/en-US/docs/Web/CSS/@property
- MDN Web Docs — `CSS.registerProperty()` - https://developer.mozilla.org/en-US/docs/Web/API/CSS/registerProperty_static
- MDN Web Docs — Invalid at computed-value time - https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties#invalid_at_computed-value_time
- W3C — CSS Custom Properties for Cascading Variables Module Level 1 - https://www.w3.org/TR/css-variables-1/
- W3C — CSS Properties and Values API Level 1 - https://www.w3.org/TR/css-properties-values-api-1/
- CSS-Tricks — A Complete Guide to Custom Properties - https://css-tricks.com/a-complete-guide-to-custom-properties/
- web.dev — `@property`: giving superpowers to CSS variables - https://web.dev/articles/at-property
- Can I Use — CSS Custom Properties - https://caniuse.com/css-variables
- Can I Use — `@property` - https://caniuse.com/mdn-css_at-rules_property