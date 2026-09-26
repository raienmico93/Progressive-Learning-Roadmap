# HTML Code and Technical Text: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML code and technical text elements are semantic inline and block-level elements that identify fragments of computer code, preformatted text, user input, program output, and variable names within an HTML document.

**Technical Definition**

Code and technical text encompasses the text-level semantic elements defined in the WHATWG HTML Living Standard that annotate content as computer code (`<code>`), preformatted text (`<pre>`), user input (`<kbd>`), sample program output (`<samp>`), and variable names (`<var>`). Each element carries a specific semantic meaning that user agents and assistive technologies can interpret independently of default visual presentation. The `<code>` element represents a fragment of computer code, which could be an XML element name, a filename, a computer program, or any other string that a computer would recognise. The `<pre>` element represents preformatted text that is presented exactly as written in the HTML file, typically rendered in a monospaced font. The `<kbd>` element represents user input, typically keyboard input. The `<samp>` element represents sample or quoted output from another program or computing system. The `<var>` element represents a variable in a mathematical expression or programming context.

**Beginner-Friendly Explanation**

When you write about programming on a webpage, you need to show code, keyboard shortcuts, program output, and variables in a way that makes sense. HTML gives you five special tags for this: `<code>` for code snippets, `<pre>` for text that needs exact spacing, `<kbd>` for keys the user should press, `<samp>` for what a program displays, and `<var>` for variable names. Using the right tag tells browsers and screen readers what kind of technical content you're showing.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Semantic precision** | Each element has a narrowly defined meaning in the HTML specification |
| **Monospace default** | All five elements render in a monospaced (fixed-width) font by default |
| **Whitespace handling** | `<pre>` preserves all whitespace; `<code>`, `<kbd>`, `<samp>`, and `<var>` collapse whitespace normally |
| **Nesting patterns** | `<code>` is often nested inside `<pre>`; `<kbd>` and `<samp>` can be nested in various combinations |
| **Accessibility relevance** | Screen readers interpret these elements to convey meaning beyond visual styling |
| **Escape requirements** | Inside `<pre>`, characters like `<` and `&` must be escaped using character references |

---

### Prerequisites

- Basic familiarity with HTML document structure and elements
- Understanding of inline (phrasing) vs. block-level (flow) content
- Awareness of HTML character references (entities)
- Basic knowledge of programming concepts (variables, code syntax)
- Basic knowledge of accessibility principles (helpful but not required)

---

### Related Programming Areas

- **Semantic HTML** – These elements are fundamental to meaningful technical markup
- **Web Accessibility (A11y)** – Screen readers interpret code and technical text elements to convey meaning
- **Syntax Highlighting** – Libraries like highlight.js and Prism.js target `<pre><code>` blocks
- **Technical Documentation** – API docs, tutorials, and code examples rely on these elements
- **CSS Typography** – Monospace fonts, line height, and tab size are controlled via CSS
- **HTML Character References** – Escaping `<`, `>`, and `&` is essential inside `<pre>` and `<code>`

---

## Core Concepts / Features

---

### 1. `<code>` — Code Fragment

#### Definitions

**Core Definition**

The `<code>` element represents a fragment of computer code, such as an XML element name, a filename, a computer program, or any other string that a computer would recognise.

**Technical Definition**

The `<code>` element represents a fragment of computer code. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. Both start and end tags are mandatory. It accepts only global attributes. Its DOM interface is `HTMLElement`. The element is strictly inline-level content and is typically rendered using the user agent's default monospaced font.

**Beginner-Friendly Explanation**

Use `<code>` when you want to show a piece of code inside a sentence or paragraph — like a function name, a variable, or a short command. It usually appears in a monospaced font. If you have a whole block of code, wrap it in `<pre><code>` instead.

#### Purposes

- To mark a fragment of computer code semantically
- To distinguish code from surrounding prose
- To enable syntax highlighting libraries to identify code content
- To provide a styling hook for monospaced font rendering

#### Syntax Rules and Structure

**General Syntax**

```html
<code>Code fragment here</code>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<code>` | Opening tag; indicates a code fragment |
| `Content` | Phrasing content; the code itself |
| `</code>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- Content model is phrasing content
- Only global attributes are permitted
- Use `<pre><code>` for multi-line code blocks
- Escape `<`, `>`, and `&` inside `<code>` when they are part of the code

**Constraints and Limitations**

- The `<code>` element is inline-level by default; for block-level code display, nest it inside `<pre>`
- It does not provide syntax highlighting on its own; CSS or JavaScript libraries are required
- Very long code strings may overflow; use CSS `overflow-wrap` or `white-space` to control wrapping

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Code Fragment**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Inline Code Demo</title>
    <style>
        code {
            background-color: #f4f4f4;
            padding: 2px 6px;
            border-radius: 4px;
            font-family: "Courier New", monospace;
        }
    </style>
</head>
<body>
    <p>
        Use the <code>console.log()</code> function to output a message
        to the browser console.
    </p>
    <p>
        The <code>document.getElementById()</code> method returns an
        element by its <code>id</code> attribute.
    </p>
</body>
</html>
```

**Expected Output**

The code fragments appear in a monospaced font with a light grey background and rounded corners.

**Why This Output Occurs**

The `<code>` element semantically marks the fragments as computer code. The CSS provides the visual styling. The browser's default monospaced font is overridden with “Courier New” for consistency.

---

**Example 2: Code Block with `<pre><code>`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Code Block Demo</title>
    <style>
        pre {
            background-color: #2d2d2d;
            color: #f8f8f2;
            padding: 1em;
            border-radius: 6px;
            overflow-x: auto;
        }
        code {
            font-family: "Fira Code", "Courier New", monospace;
        }
    </style>
</head>
<body>
    <p>The following function calculates the factorial of a number:</p>
    <pre><code>function factorial(n) {
    if (n === 0 || n === 1) {
        return 1;
    }
    return n * factorial(n - 1);
}</code></pre>
</body>
</html>
```

**Expected Output**

The code block appears with a dark background, light text, and preserved indentation and line breaks.

**Why This Output Occurs**

The `<pre>` element preserves whitespace and line breaks, while `<code>` semantically marks the content as code. The CSS provides the visual styling. The `overflow-x: auto` ensures the code scrolls horizontally rather than wrapping if it exceeds the container width.

#### Real-World Cases

**Case 1: API Documentation**

API documentation uses `<code>` for endpoint names, function signatures, and parameter names: “Call the `getUser()` function with the `userId` parameter.”

**Case 2: Tutorial Websites**

Coding tutorials use `<pre><code>` blocks to show complete code examples with syntax highlighting.

**Case 3: Developer Blogs**

Technical blogs use `<code>` for inline code references and `<pre><code>` for code blocks.

---

### 2. `<pre>` — Preformatted Text

#### Definitions

**Core Definition**

The `<pre>` element represents preformatted text that is presented exactly as written in the HTML file, with whitespace preserved.

**Technical Definition**

The `<pre>` element represents preformatted text which is to be presented exactly as written in the HTML file. The text is typically rendered using a non-proportional, or monospaced, font. Whitespace inside this element is displayed as written, with one exception: if one or more leading newline characters are included immediately following the opening `<pre>` tag, the first newline character is stripped by the HTML parser. The `<pre>` element is a block-level element with a default `display` value of `block`. Its permitted content is phrasing content.

**Beginner-Friendly Explanation**

Use `<pre>` when you need text to appear exactly how you typed it — with all the spaces, tabs, and line breaks preserved. This is essential for code blocks, ASCII art, and any text where formatting matters. Unlike normal HTML, where multiple spaces collapse into one, `<pre>` keeps everything as-is.

#### Purposes

- To preserve whitespace and line breaks exactly as written
- To display code blocks with correct indentation
- To render ASCII art or text-based diagrams
- To show preformatted text where structural formatting is significant

#### Syntax Rules and Structure

**General Syntax**

```html
<pre>
Preformatted text here
</pre>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<pre>` | Opening tag; indicates preformatted text |
| `Content` | Phrasing content; the preformatted text |
| `</pre>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- Content model is phrasing content
- Whitespace is preserved exactly as written
- A leading newline immediately after the opening tag is stripped
- Escape `<`, `>`, and `&` using character references
- Commonly used with `<code>`, `<samp>`, and `<kbd>` elements

**Constraints and Limitations**

- The `width` attribute is obsolete and has no visual effect; use CSS `width` instead
- The `wrap` attribute is ignored in modern browsers; use CSS `white-space` instead
- Content inside `<pre>` is parsed as HTML, so special characters must be escaped
- Long lines may cause horizontal scrolling; use `overflow-x: auto` in CSS

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Preserved Whitespace**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Pre Demo</title>
</head>
<body>
    <p>Without pre,    multiple     spaces    collapse.</p>
    <pre>With pre,    multiple     spaces    are preserved.</pre>
</body>
</html>
```

**Expected Output**

The first paragraph shows single spaces between words. The `<pre>` block shows the exact spacing as typed.

**Why This Output Occurs**

The `<pre>` element sets `white-space: pre` by default, which preserves all whitespace characters and prevents line wrapping.

---

**Example 2: Escaping Special Characters**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Escaping in Pre</title>
</head>
<body>
    <pre><code>&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;title&gt;Example&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;p&gt;Hello, world!&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;</code></pre>
</body>
</html>
```

**Expected Output**

The browser displays the literal HTML code with `<` and `>` characters, rather than interpreting them as tags.

**Why This Output Occurs**

The `<` and `>` characters are escaped using `&lt;` and `&gt;` character references. Without escaping, the browser would interpret them as HTML tags and the code would not be displayed correctly.

#### Real-World Cases

**Case 1: Code Repositories**

GitHub, GitLab, and similar platforms render code files using `<pre>` equivalents to preserve indentation.

**Case 2: ASCII Art**

Websites displaying ASCII art use `<pre>` to maintain the exact character positioning.

**Case 3: Terminal Emulators**

Web-based terminal emulators use `<pre>` to display command output with correct formatting.

---

### 3. `<kbd>` — Keyboard Input

#### Definitions

**Core Definition**

The `<kbd>` element represents user input, typically keyboard input, but it may also represent other input such as voice commands.

**Technical Definition**

The `<kbd>` element represents a span of inline text denoting textual user input from a keyboard, voice input, or any other text entry device. By default, the content text is displayed using the user agent's default monospaced font. The `<kbd>` element may be nested in various combinations with the `<samp>` element to represent various forms of input or output based on visual cues. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content.

**Beginner-Friendly Explanation**

Use `<kbd>` when you want to show what keys a user should press. For example, “Press `Ctrl` + `C` to copy.” It usually appears in a monospaced font. You can nest `<kbd>` inside `<kbd>` to show key combinations, and nest `<samp>` inside `<kbd>` to show menu options.

#### Purposes

- To mark user input, typically keyboard input
- To represent key combinations and shortcuts
- To distinguish input from surrounding prose
- To enable styling of keyboard shortcuts via CSS

#### Syntax Rules and Structure

**General Syntax**

```html
<kbd>Key or command</kbd>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<kbd>` | Opening tag; indicates user input |
| `Content` | Phrasing content; the input text |
| `</kbd>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- Content model is phrasing content
- Only global attributes are permitted
- Nest `<kbd>` inside `<kbd>` for key combinations
- Nest `<samp>` inside `<kbd>` for onscreen input options

**Constraints and Limitations**

- The `<kbd>` element is inline-level
- It does not provide a visual “key cap” style by default; CSS is required
- Overuse can make text harder to read

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Keyboard Input**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Kbd Demo</title>
    <style>
        kbd {
            background-color: #eeeeee;
            border-radius: 3px;
            border: 1px solid #b4b4b4;
            box-shadow: 0 1px 1px rgba(0,0,0,0.2);
            color: #333333;
            display: inline-block;
            font-size: 0.85em;
            font-weight: bold;
            line-height: 1;
            padding: 2px 4px;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <p>Please press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd> to re-render the page.</p>
</body>
</html>
```

**Expected Output**

Each key appears as a styled “key cap” with a light grey background, border, and subtle shadow.

**Why This Output Occurs**

The `<kbd>` element marks each key as user input. The CSS creates the key cap appearance. The `white-space: nowrap` prevents keys from wrapping.

---

**Example 2: Nested Kbd for Key Combinations**

```html
<p>
    You can also create a new document using the
    <kbd><kbd>Ctrl</kbd>+<kbd>N</kbd></kbd> keyboard shortcut.
</p>
```

**Expected Output**

The entire shortcut “Ctrl+N” is wrapped in an outer `<kbd>`, with each individual key in its own `<kbd>`.

**Why This Output Occurs**

The specification supports nesting `<kbd>` elements, with the outer element representing the overall input and each inner element representing an individual keystroke.

#### Real-World Cases

**Case 1: Software Documentation**

Software manuals use `<kbd>` for keyboard shortcuts: “Press `<kbd>Ctrl</kbd>+<kbd>S</kbd>` to save.”

**Case 2: Game Guides**

Video game guides use `<kbd>` for controller button combinations.

**Case 3: Web Applications**

Web apps use `<kbd>` for keyboard navigation hints: “Press `<kbd>Esc</kbd>` to close.”

---

### 4. `<samp>` — Sample Program Output

#### Definitions

**Core Definition**

The `<samp>` element represents sample or quoted output from another program or computing system.

**Technical Definition**

The `<samp>` element represents sample or quoted output from another program or computing system. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. Both start and end tags are mandatory. It accepts only global attributes. Its DOM interface is `HTMLElement`. The element can be contrasted with the `<output>` element, which is used to provide immediate output in a web application.

**Beginner-Friendly Explanation**

Use `<samp>` when you want to show what a program displayed — like an error message, a command-line response, or any output the computer produced. It usually appears in a monospaced font.

#### Purposes

- To mark sample output from a program or computing system
- To distinguish program output from surrounding prose
- To enable styling of output text via CSS
- To work with `<kbd>` and `<pre>` for terminal-style displays

#### Syntax Rules and Structure

**General Syntax**

```html
<samp>Program output here</samp>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<samp>` | Opening tag; indicates sample output |
| `Content` | Phrasing content; the output text |
| `</samp>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- Content model is phrasing content
- Only global attributes are permitted
- Can be nested inside `<pre>` for block-level output
- Can be nested with `<kbd>` for echoed input

**Constraints and Limitations**

- The `<samp>` element is inline-level
- It does not provide syntax highlighting on its own
- For immediate output in a web application, use `<output>` instead

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Sample Output**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Samp Demo</title>
    <style>
        samp {
            background-color: #f0f0f0;
            padding: 2px 6px;
            border-radius: 4px;
            font-family: monospace;
        }
    </style>
</head>
<body>
    <p>The computer said <samp>Too much cheese in tray two</samp> but I didn't know what that meant.</p>
</body>
</html>
```

**Expected Output**

The output message appears in a monospaced font with a light background.

**Why This Output Occurs**

The `<samp>` element semantically marks the text as sample output. The CSS provides the visual styling.

---

**Example 2: Block-Level Sample Output with `<pre><samp>`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Terminal Output</title>
    <style>
        pre {
            background-color: #1e1e1e;
            color: #d4d4d4;
            padding: 1em;
            border-radius: 6px;
            overflow-x: auto;
        }
        .prompt {
            color: #4ec9b0;
        }
        samp {
            font-family: "Courier New", monospace;
        }
    </style>
</head>
<body>
    <pre><samp><span class="prompt">jdoe@mowmow:~$</span> ssh demo.example.com
Last login: Fri Mar 19 11:12:56 2026 from 192.168.1.100
<samp>Welcome to the demo server.</samp></samp></pre>
</body>
</html>
```

**Expected Output**

A terminal-style output block with a coloured prompt, command echo, login message, and welcome message.

**Why This Output Occurs**

The `<pre>` element preserves the formatting, `<samp>` marks the output semantically, and `<span>` with CSS provides colour for the prompt. This is the pattern recommended by the WHATWG specification for styling specific elements of sample output.

#### Real-World Cases

**Case 1: Command-Line Tutorials**

Tutorials use `<samp>` to show expected terminal output: “You should see `<samp>Build successful</samp>`.”

**Case 2: Error Documentation**

Error message documentation uses `<samp>` to show exact error text: `<samp>Error 404: File not found</samp>`.

**Case 3: API Documentation**

API docs use `<samp>` to show example JSON responses or console output.

---

### 5. `<var>` — Variable

#### Definitions

**Core Definition**

The `<var>` element represents the name of a variable in a mathematical expression or programming context.

**Technical Definition**

The `<var>` element represents a variable. This could be an actual variable in a mathematical expression or programming context, an identifier representing a constant, a symbol identifying a physical quantity, a function parameter, or just a term used as a placeholder in prose. It is categorised as flow content, phrasing content, and palpable content. Its permitted content is phrasing content. Both start and end tags are mandatory. It accepts only global attributes. Its DOM interface is `HTMLElement`. For mathematics, particularly for anything beyond the simplest of expressions, MathML is more appropriate. However, the `<var>` element can still be used to refer to specific variables that are then mentioned in MathML expressions.

**Beginner-Friendly Explanation**

Use `<var>` when you refer to a variable by name — like `x` in a math equation, or `userId` in a programming context. It usually appears in italics. It helps readers distinguish variable names from regular words.

#### Purposes

- To mark the name of a variable in programming or mathematical context
- To distinguish variable names from surrounding prose
- To provide a styling hook for variable rendering
- To mark function parameters and placeholder terms

#### Syntax Rules and Structure

**General Syntax**

```html
<var>variableName</var>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<var>` | Opening tag; indicates a variable |
| `Content` | Phrasing content; the variable name |
| `</var>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- Content model is phrasing content
- Only global attributes are permitted
- Can contain `<sub>` for subscripted variables
- Use MathML for complex mathematical expressions

**Constraints and Limitations**

- The `<var>` element is inline-level
- It renders in italic monospace by default, but this varies by browser
- Overuse for non-variable content is discouraged
- For mathematics beyond simple expressions, MathML is recommended

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Mathematical Variable**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Var Demo</title>
    <style>
        var {
            font-style: italic;
            font-family: "Times New Roman", serif;
            color: #c7254e;
        }
    </style>
</head>
<body>
    <p>
        If there are <var>n</var> pipes leading to the ice cream factory,
        then I expect at least <var>n</var> flavors of ice cream to be available.
    </p>
    <p>
        Then she turned to the blackboard and wrote
        <var>E</var> = <var>m</var><var>c</var><sup>2</sup>.
    </p>
</body>
</html>
```

**Expected Output**

The variables *n*, *E*, *m*, and *c* appear in italic serif font with a reddish colour.

**Why This Output Occurs**

The `<var>` element semantically marks the variable names. The CSS provides the visual styling. The `<sup>` element marks the exponent 2.

---

**Example 2: Programming Variable**

```html
<p>
    The function accepts a parameter <var>userId</var> and returns
    the corresponding user object. If <var>userId</var> is null,
    an error is thrown.
</p>
```

**Expected Output**

“userId” appears in italic monospace, distinguishing it from the surrounding prose.

**Why This Output Occurs**

The `<var>` element marks the parameter name as a variable, following the specification's description of function parameters as a valid use case.

#### Real-World Cases

**Case 1: Programming Tutorials**

Tutorials use `<var>` for variable names in explanations: “Assign a value to `<var>count</var>` before using it.”

**Case 2: Mathematical Documentation**

Math textbooks and papers use `<var>` for variables in simple expressions.

**Case 3: API Reference**

API documentation uses `<var>` for parameter names: “The `<var>timeout</var>` parameter controls how long the request waits.”

---

### 6. Formatting Source Code

#### Definitions

**Core Definition**

Formatting source code is the practice of structuring HTML markup to display code fragments and code blocks clearly, using `<code>` and `<pre>` together with proper escaping and styling.

**Technical Definition**

Formatting source code in HTML requires the combined use of `<pre>` for whitespace preservation and `<code>` for semantic code marking. Inside the `<pre>` element, characters that have special meaning in HTML (`<`, `>`, `&`) must be escaped using character references (`&lt;`, `&gt;`, `&amp;`). Additional formatting such as syntax highlighting, line numbers, and code annotations is achieved through CSS and JavaScript libraries.

**Beginner-Friendly Explanation**

To show code properly on a webpage, you usually wrap it in `<pre><code>`. The `<pre>` tag keeps your line breaks and indentation, while `<code>` tells the browser “this is code.” You also need to replace `<` with `&lt;` and `>` with `&gt;` so the browser doesn't think your code is actual HTML tags. For nicer display, you can add syntax highlighting with a library like highlight.js.

#### Purposes

- To display code fragments with correct indentation and line breaks
- To semantically mark code content for screen readers and search engines
- To enable syntax highlighting and other visual enhancements
- To prevent HTML tags in code from being interpreted as markup

#### Syntax Rules and Structure

**General Syntax**

```html
<pre><code class="language-name">
Escaped code content here
</code></pre>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<pre>` | Preserves whitespace and line breaks |
| `<code>` | Marks content as code |
| `class="language-..."` | Optional; hints the programming language for syntax highlighting |
| `Escaped content` | Code with `<`, `>`, `&` escaped |

**Syntax Rules**

- Always escape `<`, `>`, and `&` inside `<pre>` and `<code>`
- Use `<pre><code>` for multi-line code blocks
- Use `<code>` alone for inline code fragments
- Add a `class` attribute with a language identifier for syntax highlighting libraries
- Use CSS to control overflow, background, and font

**Constraints and Limitations**

- The HTML specification does not define a syntax highlighting mechanism; third-party libraries are required
- Code inside `<pre>` is parsed as HTML, so escaping is mandatory
- Very long code blocks may require horizontal scrolling

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Complete Code Block with Syntax Highlighting**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Code Formatting Demo</title>
    <!-- Load highlight.js theme and library -->
    <link rel="stylesheet"
          href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/github-dark.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    <script>hljs.highlightAll();</script>
    <style>
        pre {
            border-radius: 8px;
            overflow-x: auto;
        }
        code {
            font-family: "Fira Code", "Courier New", monospace;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <p>Here is a JavaScript function:</p>
    <pre><code class="language-javascript">
function greet(name) {
    return "Hello, " + name + "!";
}
    </code></pre>
</body>
</html>
```

**Expected Output**

The code block appears with syntax highlighting (keywords in one colour, strings in another), a dark background, and preserved indentation.

**Why This Output Occurs**

The `<pre>` element preserves the formatting, `<code>` marks the content as code, and the `language-javascript` class tells highlight.js which language to use. The library applies syntax highlighting automatically.

---

**Example 2: Escaping Special Characters**

```html
<pre><code class="language-html">
&lt;div class="container"&gt;
    &lt;p&gt;Hello, world!&lt;/p&gt;
&lt;/div&gt;
</code></pre>
```

**Expected Output**

The browser displays the literal HTML code with `<div>`, `<p>`, and `</div>` visible as text.

**Why This Output Occurs**

The `<` and `>` characters are escaped using `&lt;` and `&gt;`. Without escaping, the browser would interpret them as actual HTML tags and the code would not be displayed.

#### Real-World Cases

**Case 1: Documentation Websites**

MDN Web Docs, DevDocs, and similar platforms use `<pre><code>` with syntax highlighting for all code examples.

**Case 2: Blog Platforms**

Dev.to, Medium, and Hashnode use `<pre><code>` for code blocks in technical articles.

**Case 3: Learning Platforms**

freeCodeCamp, Codecademy, and similar platforms use `<pre><code>` for code examples and exercises.

---

### 7. Representing User Input

#### Definitions

**Core Definition**

Representing user input involves using the `<kbd>` element to mark text that the user should enter, typically through a keyboard, along with nesting patterns for key combinations and onscreen options.

**Technical Definition**

User input representation uses the `<kbd>` element, which may be nested in various combinations with `<samp>` to represent different forms of input and output. The specification defines three key nesting patterns: nesting `<kbd>` within `<kbd>` represents an actual key or unit of input as a portion of a larger input; nesting `<kbd>` inside `<samp>` represents input that has been echoed back to the user by the system; nesting `<samp>` inside `<kbd>` represents input based on text presented by the system, such as menu names or button labels.

**Beginner-Friendly Explanation**

You can combine `<kbd>` and `<samp>` in different ways to show different things. Put `<kbd>` inside `<kbd>` to show a key combination. Put `<samp>` inside `<kbd>` to show a menu option the user should choose. Put `<kbd>` inside `<samp>` to show what the user typed that the computer displayed back.

#### Purposes

- To mark keyboard input and key combinations
- To represent onscreen input options (menus, buttons)
- To show echoed input (what the user typed, as displayed by the system)
- To distinguish different types of user input through semantic nesting

#### Syntax Rules and Structure

**Nesting Patterns**

| Pattern | Meaning | Example |
|---|---|---|
| `<kbd><kbd>Ctrl</kbd>+<kbd>S</kbd></kbd>` | Key combination | Ctrl+S |
| `<kbd><samp>File</samp></kbd>` | Onscreen input option | File menu |
| `<samp><kbd>help</kbd></samp>` | Echoed input | help |

**Component Breakdown**

| Component | Description |
|---|---|
| Outer `<kbd>` | Represents the overall input |
| Inner `<kbd>` | Represents individual keystrokes |
| `<samp>` inside `<kbd>` | Represents onscreen option |
| `<kbd>` inside `<samp>` | Represents echoed input |

**Syntax Rules**

- Nest `<kbd>` inside `<kbd>` for key combinations
- Nest `<samp>` inside `<kbd>` for onscreen options
- Nest `<kbd>` inside `<samp>` for echoed input
- Escape special characters as needed

**Constraints and Limitations**

- These nesting patterns are visual and semantic cues, not strict rules
- Browser default styling may not distinguish nested elements without CSS
- Overuse of nesting can make text hard to read

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Key Combination**

```html
<p>
    To make George eat an apple, press
    <kbd><kbd>Shift</kbd>+<kbd>F3</kbd></kbd>.
</p>
```

**Expected Output**

The shortcut “Shift+F3” is displayed with the outer `<kbd>` containing the full combination and each key individually marked.

**Why This Output Occurs**

The nesting pattern follows the specification's recommendation for representing keystrokes within an input.

---

**Example 2: Onscreen Input Option**

```html
<p>
    To make George eat an apple, select
    <kbd><samp>File</samp></kbd> >
    <kbd><samp>Eat Apple...</samp></kbd>.
</p>
```

**Expected Output**

The menu path “File > Eat Apple...” is displayed with each menu option wrapped in `<kbd><samp>`.

**Why This Output Occurs**

The `<samp>` inside `<kbd>` represents input based on text presented by the system, such as menu names.

#### Real-World Cases

**Case 1: Software Manuals**

Manuals use the `<kbd><kbd>` pattern for keyboard shortcuts and `<kbd><samp>` for menu navigation.

**Case 2: Game Guides**

Game guides use `<kbd>` nesting for controller button combinations.

**Case 3: Help Documentation**

Help docs use `<kbd><samp>` for menu paths and `<kbd>` for keyboard shortcuts.

---

### 8. Representing Program Output

#### Definitions

**Core Definition**

Representing program output involves using the `<samp>` element to mark sample or quoted output from a program or computing system, often in combination with `<pre>` and `<kbd>`.

**Technical Definition**

Program output representation uses the `<samp>` element, which represents sample or quoted output from another program or computing system. The element may be nested with `<kbd>` to show user input within output, and with `<span>` for styling specific parts of the output. When used inside `<pre>`, `<samp>` can represent block-level terminal output with preserved formatting.

**Beginner-Friendly Explanation**

Use `<samp>` to show what a program displayed. If you're showing a terminal session where the user typed commands and the computer responded, you can nest `<kbd>` inside `<samp>` to show the typed parts. You can also use `<span>` with CSS to colour different parts of the output.

#### Purposes

- To mark sample output from a program or computing system
- To distinguish program output from user input and surrounding prose
- To enable styling of output text via CSS
- To show complete terminal sessions with input and output

#### Syntax Rules and Structure

**General Syntax**

```html
<samp>Program output here</samp>
```

**Terminal Session Syntax**

```html
<pre><samp>
<span class="prompt">user@host:~$</span> <kbd>command</kbd>
Program output line 1
Program output line 2
</samp></pre>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<samp>` | Marks program output |
| `<kbd>` inside `<samp>` | Marks user input within output |
| `<span>` inside `<samp>` | Styles specific parts of output |
| `<pre>` | Preserves formatting for block output |

**Syntax Rules**

- Use `<samp>` alone for inline output
- Use `<pre><samp>` for block-level terminal output
- Nest `<kbd>` inside `<samp>` for echoed input
- Use `<span>` with CSS for styling prompts and highlights

**Constraints and Limitations**

- The `<samp>` element does not provide syntax highlighting on its own
- Styling nested elements requires CSS
- For immediate output in a web application, use `<output>` instead

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Inline Program Output**

```html
<p>
    When the program finishes, it displays
    <samp>Build successful</samp> in the console.
</p>
```

**Expected Output**

“Build successful” appears in a monospaced font within the sentence.

**Why This Output Occurs**

The `<samp>` element semantically marks the output text.

---

**Example 2: Terminal Session with Echoed Input**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Terminal Session</title>
    <style>
        pre { background: #1e1e1e; color: #d4d4d4; padding: 1em; border-radius: 6px; }
        .prompt { color: #4ec9b0; }
        kbd { color: #ce9178; }
        samp { font-family: monospace; }
    </style>
</head>
<body>
    <pre><samp><span class="prompt">user@host:~$</span> <kbd>npm install</kbd>
added 150 packages in 12s

<span class="prompt">user@host:~$</span> <kbd>npm test</kbd>
<samp>All 42 tests passed.</samp></samp></pre>
</body>
</html>
```

**Expected Output**

A terminal session showing the prompt, the typed commands (`npm install`, `npm test`), and the program output.

**Why This Output Occurs**

The `<pre>` preserves formatting, `<samp>` marks the overall output, `<kbd>` marks the typed commands (echoed input), and `<span>` with CSS colours the prompt.

#### Real-World Cases

**Case 1: Command-Line Tutorials**

Tutorials show complete terminal sessions with prompts, commands, and output.

**Case 2: Error Documentation**

Error docs show exact program output with `<samp>` for the error message.

**Case 3: API Documentation**

API docs show example responses and console output using `<samp>`.

---

## References

- MDN Web Docs – `<code>`: The Inline Code element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code
- MDN Web Docs – `<pre>`: The Preformatted Text element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/pre
- MDN Web Docs – `<kbd>`: The Keyboard Input element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/kbd
- MDN Web Docs – `<samp>`: The Sample Output element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/samp
- MDN Web Docs – `<var>`: The Variable element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/var
- WHATWG HTML Living Standard – Text-level semantics: The code element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-code-element
- WHATWG HTML Living Standard – Text-level semantics: The var element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-var-element
- WHATWG HTML Living Standard – Text-level semantics: The samp element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-samp-element
- WHATWG HTML Living Standard – Text-level semantics: The kbd element – https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-kbd-element
- WHATWG HTML Living Standard – Grouping content: The pre element – https://html.spec.whatwg.org/multipage/grouping-content.html#the-pre-element
- W3C – HTML 5: The pre element – https://dev.w3.org/html5/spec-author-view/grouping-content.html#the-pre-element
- W3C – CSS Text Module Level 3: White Space Processing – https://www.w3.org/TR/css-text-3/#white-space-processing
- MDN Web Docs – HTML character reference – https://developer.mozilla.org/en-US/docs/Glossary/Character_reference
- MDN Web Docs – Guidelines for styling HTML code examples – https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Code_style_guide/HTML
- MDN Web Docs – `white-space` CSS property – https://developer.mozilla.org/en-US/docs/Web/CSS/white-space
- Highlight.js – Getting Started – https://highlightjs.org/