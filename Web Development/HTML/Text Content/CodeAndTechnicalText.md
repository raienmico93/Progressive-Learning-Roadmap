# Code and Preformatted Text — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Code and preformatted text elements are the HTML elements used to mark up computer code, user input, program output, variables, and text where whitespace and line breaks are significant. They include `<code>`, `<pre>`, `<kbd>`, `<samp>`, and `<var>`.

**Technical Definition**
In the WHATWG HTML Living Standard, these elements belong to the **text-level semantics** chapter. `<code>` represents a fragment of computer code; `<pre>` represents preformatted text where structure is conveyed by typographic conventions rather than elements; `<kbd>` represents user input; `<samp>` represents sample output from a program; and `<var>` represents a variable in a mathematical or programming context. All belong to the **flow content**, **phrasing content**, and **palpable content** categories (except `<pre>`, which is flow content and palpable content but not phrasing content). Their content models are phrasing content, with `<pre>` accepting phrasing content and the others accepting phrasing content as well.

**Beginner-Friendly Explanation**
These elements are how you tell the browser “this text is code,” “this is what the user types,” “this is what the program prints,” or “this is a variable.” They also include `<pre>`, which preserves spaces and line breaks so that code and text-based art display exactly as written.

---

### Key Characteristics

- **Semantic distinction**: Each element carries a specific meaning. `<code>` is for code fragments, `<kbd>` for user input, `<samp>` for program output, `<var>` for variables, and `<pre>` for whitespace-sensitive text.
- **Monospace default**: Browsers typically render these elements in a monospaced font, but the visual presentation is not the semantics.
- **Whitespace preservation**: `<pre>` is the only element in this group that preserves whitespace and line breaks exactly as written.
- **Combination patterns**: `<pre>` commonly contains `<code>`, `<samp>`, and `<kbd>` to create richly marked-up code blocks and terminal transcripts.
- **Escaping requirement**: Inside `<pre>`, characters such as `<` and `&` must be escaped as character references to avoid being parsed as HTML.
- **Accessibility considerations**: Scrollable `<pre>` blocks need keyboard access, and preformatted diagrams need alternative descriptions.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **Phrasing vs. flow content**: understanding which elements can appear inside a paragraph.
3. **HTML character escaping**: knowing how to use `&lt;`, `&gt;`, and `&amp;`.
4. **CSS fundamentals**: `font-family`, `white-space`, `overflow`, and `line-height`.
5. **Accessibility awareness**: understanding that scrollable content and visual-only information need accessible alternatives.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Technical documentation** | Code and preformatted text elements are essential for API references, tutorials, and guides. |
| **Accessibility (WCAG)** | Scrollable code blocks require keyboard access; ASCII art requires text alternatives. |
| **CSS** | Styling controls monospace fonts, syntax highlighting, wrapping, and scrolling. |
| **Copy-to-clipboard features** | Code blocks are often paired with copy buttons that need accessible labels. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<code>`** — code fragment
2. **`<pre>`** — preformatted text
3. **`<kbd>`** — user input
4. **`<samp>`** — sample output
5. **`<var>`** — variable
6. **Formatting source code**
7. **Representing user input**
8. **Representing program output**
9. **Code best practices**
10. **Preformatted text best practices**

---

## 1. `<code>` — The Code Fragment Element

### Definitions

**Core Definition**
The `<code>` element displays its contents styled in a fashion intended to indicate that the text is a short fragment of computer code.

**Technical Definition**
`<code>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. It accepts only global attributes. By default, content is displayed in the user agent’s default monospace font. The element by itself represents a single phrase of code or line of code; to represent multiple lines, it must be wrapped inside a `<pre>` element.

**Beginner-Friendly Explanation**
`<code>` marks a piece of code inside text — a variable name, function, or command. It renders in a monospaced font and tells screen readers that the text is code.

---

### Purposes

- **To mark inline code fragments** within prose.
- **To support syntax highlighting** by providing a semantic hook for stylesheets and scripts.
- **To distinguish code from surrounding text** for assistive technologies.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Use <code>console.log()</code> to print output.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<code>` | The code wrapper. |
| Content | Phrasing content (the code fragment). |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **For multi-line code**, wrap `<code>` inside `<pre>`.
3. **Escape `<` and `>` characters** as `&lt;` and `&gt;` when showing HTML code.

#### Constraints and Limitations

- `<code>` is a semantic marker, not a syntax highlighter; styling must be applied with CSS or JavaScript.
- It does not preserve whitespace on its own; use `<pre>` for that.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Inline Code

**Setup Guide**: Save as `code-inline.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Code Example</title>
</head>
<body>
  <p>The <code>querySelector</code> method selects elements.</p>
</body>
</html>
```

**Expected output**: “querySelector” appears in a monospaced font.

**Why this output occurs**: `<code>` marks the term as code. The browser applies a monospaced font by default, and screen readers may announce it as code.

---

#### Example 2: Code Block with Pre

**Setup Guide**: Save as `code-block.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Code Block</title>
</head>
<body>
  <pre><code>for (let i = 0; i &lt; 3; i++) {
  console.log(i);
}</code></pre>
</body>
</html>
```

**Expected output**: A multi-line code block with preserved whitespace and monospaced font.

**Why this output occurs**: `<pre>` preserves whitespace and line breaks, and `<code>` marks the text as code. The `<` and `>` are escaped as HTML entities so they render as literal characters.

---

### Real-World Cases

1. **API documentation**: Function names and parameters.
2. **Tutorials**: Sample code blocks.
3. **Technical blogs**: Inline code references.

---

### References

- MDN Web Docs — `<code>`: The Inline Code element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code
- WHATWG — The `code` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-code-element

---

## 2. `<pre>` — The Preformatted Text Element

### Definitions

**Core Definition**
The `<pre>` element represents preformatted text, which is to be presented exactly as written in the HTML file. Whitespace and line breaks are preserved.

**Technical Definition**
`<pre>` belongs to the **flow content** and **palpable content** categories. Its content model is **phrasing content**. The text is typically rendered in a monospaced font, and whitespace inside the element is displayed as written, with one exception: if one or more leading newline characters are included immediately following the opening `<pre>` tag, the first newline character is stripped. The element is block-level by default. The `width` attribute is obsolete and has no visual effect; the `wrap` attribute is ignored in modern browsers. CSS `white-space` and `width` should be used for presentation instead.

**Beginner-Friendly Explanation**
`<pre>` shows text exactly as you typed it — spaces, tabs, and line breaks are all preserved. It is used for code blocks, ASCII art, and any text where the exact formatting matters.

---

### Purposes

- **To preserve whitespace and line breaks** in text where formatting is meaningful.
- **To display code blocks** in technical documentation, often combined with `<code>`.
- **To present ASCII art** or other text-based visual layouts.
- **To show preformatted output** from programs or systems.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<pre>
  function greet(name) {
    return "Hello, " + name + "!";
  }
</pre>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<pre>` | The preformatted text container. |
| Phrasing content | Text, `<code>`, `<samp>`, `<kbd>`, and other phrasing elements. |

#### Syntax Rules

1. **Content model is phrasing content**; block-level elements are not permitted inside `<pre>`.
2. **Whitespace is preserved**; spaces, tabs, and newlines are rendered literally.
3. **A leading newline after `<pre>` is ignored** by the parser.
4. **Escape `<` and `&`** as `&lt;` and `&amp;` to prevent them from being parsed as markup.

#### Constraints and Limitations

- `<pre>` content must not contain block-level elements.
- Very long lines can cause horizontal scrolling; use CSS `white-space` and `overflow` to manage.
- Leading newline stripping can affect ASCII art and diagrams; test the output.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Code Block with `<pre>` and `<code>`

**Setup Guide**: Save as `pre-code.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Preformatted Code</title>
</head>
<body>
  <p>Example JavaScript function:</p>

  <pre><code>function add(a, b) {
  return a + b;
}</code></pre>
</body>
</html>
```

**Expected output**: A monospaced code block in which the indentation and line break are preserved exactly as written.

**Why this output occurs**: The `<pre>` element preserves whitespace and line breaks, and the `<code>` element indicates that the text is computer code. The combination is the standard pattern for code listings.

---

#### Example 2: ASCII Art Preserved with `<pre>`

**Setup Guide**: Save as `pre-ascii.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>ASCII Art</title>
</head>
<body>
  <pre>
    /\_/\
   ( o.o )
    > ^ <
  </pre>
</body>
</html>
```

**Expected output**: A small ASCII cat rendered with the exact spacing and line breaks from the source.

**Why this output occurs**: The `<pre>` element preserves every space and newline, so the ASCII art appears exactly as authored.

---

### Real-World Cases

1. **Code documentation and tutorials**: `<pre><code>` blocks are the standard way to display source code.
2. **Terminal output and logs**: Preformatted text preserves line breaks and alignment.
3. **ASCII art and text-based diagrams**: `<pre>` is the only way to reliably preserve the layout.

---

### References

- MDN Web Docs — `<pre>`: The Preformatted Text element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/pre
- WHATWG — The `pre` element — https://html.spec.whatwg.org/multipage/grouping-content.html#the-pre-element

---

## 3. `<kbd>` — The Keyboard Input Element

### Definitions

**Core Definition**
The `<kbd>` element represents user input, typically keyboard input, though it may also represent other input such as voice commands.

**Technical Definition**
`<kbd>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. By default, content is displayed in a monospaced font. Nesting rules allow `<kbd>` inside `<kbd>` to represent individual keys within a larger input, `<kbd>` inside `<samp>` to represent echoed input, and `<samp>` inside `<kbd>` to represent on-screen input options such as menu names.

**Beginner-Friendly Explanation**
`<kbd>` marks what the user should type or press — a keyboard shortcut like `Ctrl+C`, or a command like `ls -la`.

---

### Purposes

- **To mark user input** in instructions.
- **To signal keyboard shortcuts**.
- **To style keyboard commands** in documentation.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Press <kbd>Ctrl</kbd> + <kbd>C</kbd> to copy.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<kbd>` | The user-input wrapper. |
| Content | Phrasing content (the key or command). |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Nest `<kbd>` inside `<kbd>`** for multi-key sequences.
3. **Combine with `<samp>`** for input/output transcripts.

#### Constraints and Limitations

- `<kbd>` is for user input; do not use it for program output (that is `<samp>`).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Keyboard Shortcut

**Setup Guide**: Save as `kbd-shortcut.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Keyboard Input</title>
</head>
<body>
  <p>To save the file, press <kbd>Ctrl</kbd> + <kbd>S</kbd>.</p>
</body>
</html>
```

**Expected output**: “Ctrl” and “S” each appear in a monospaced font, indicating keyboard keys.

**Why this output occurs**: `<kbd>` marks the key names. The browser applies a monospaced font, and additional CSS can add key-cap styling.

---

#### Example 2: Multi-Key Sequence with Nested `<kbd>`

**Setup Guide**: Save as `kbd-nested.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nested Kbd</title>
</head>
<body>
  <p>
    Press <kbd><kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>R</kbd></kbd>
    to re-render the page.
  </p>
</body>
</html>
```

**Expected output**: The entire key sequence is wrapped in an outer `<kbd>`, with each individual key in its own `<kbd>`.

**Why this output occurs**: The nesting pattern represents an actual key sequence as a single unit of input, with each key marked individually.

---

### Real-World Cases

1. **Software documentation**: Keyboard shortcuts and commands.
2. **Tutorials**: Command-line instructions.
3. **Help systems**: Key combinations for actions.

---

### References

- MDN Web Docs — `<kbd>`: The Keyboard Input element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/kbd
- WHATWG — The `kbd` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-kbd-element

---

## 4. `<samp>` — The Sample Output Element

### Definitions

**Core Definition**
The `<samp>` element is used to enclose inline text that represents sample (or quoted) output from a computer program.

**Technical Definition**
`<samp>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. Its contents are typically rendered using the browser’s default monospaced font. It is distinct from `<output>`, which is for output generated by the page’s own JavaScript code.

**Beginner-Friendly Explanation**
`<samp>` marks the output of a program — what you would see in a terminal or a console. It is often combined with `<kbd>` to show “type this, see that”.

---

### Purposes

- **To mark program output** as sample output.
- **To distinguish system responses** from surrounding prose.
- **To support technical documentation** with clear output examples.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>When you run the program, you will see: <samp>Hello, World!</samp></p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<samp>` | The sample-output wrapper. |
| Content | Phrasing content (the output text). |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<samp>` for program output**, not for arbitrary text.
3. **Combine with `<kbd>`** to show input and output together.

#### Constraints and Limitations

- `<samp>` should not be used for user input; that is `<kbd>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Program Output

**Setup Guide**: Save as `samp-output.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Sample Output</title>
</head>
<body>
  <p>The program will display: <samp>Hello, World!</samp></p>
</body>
</html>
```

**Expected output**: “Hello, World!” appears in a monospaced font, signalling that it is program output.

**Why this output occurs**: `<samp>` indicates sample output. The browser renders it in a monospace font by default.

---

#### Example 2: Combined `kbd` and `samp` Transcript

**Setup Guide**: Save as `samp-kbd.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Kbd and Samp Transcript</title>
</head>
<body>
  <p>
    <samp>
      $ <kbd>ls -la</kbd><br>
      total 12<br>
      drwxr-xr-x  2 user  staff  64 Jan  1 12:00 .
    </samp>
  </p>
</body>
</html>
```

**Expected output**: A terminal-style transcript where the command `ls -la` is marked as keyboard input and the output is marked as sample output.

**Why this output occurs**: Nesting `<kbd>` inside `<samp>` represents input that has been echoed back to the user by the system, creating a realistic transcript.

---

### Real-World Cases

1. **Terminal documentation**: Showing output.
2. **Tutorials**: Displaying program results.
3. **CLI references**: Illustrating command output.

---

### References

- MDN Web Docs — `<samp>`: The Sample Output element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/samp
- WHATWG — The `samp` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-samp-element

---

## 5. `<var>` — The Variable Element

### Definitions

**Core Definition**
The `<var>` element represents the name of a variable in a mathematical or programming context.

**Technical Definition**
`<var>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. Most browsers default to rendering `<var>` in both monospace and italic.

**Beginner-Friendly Explanation**
`<var>` marks a variable name in code or math — like `x` in an equation or `count` in a program. It is usually rendered in italics and monospace.

---

### Purposes

- **To mark variable names** in technical content.
- **To distinguish variables** from surrounding prose.
- **To support mathematical and programming notation**.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Let <var>x</var> equal 5.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<var>` | The variable wrapper. |
| Content | Phrasing content (the variable name). |

#### Syntax Rules

1. **Content model is phrasing content**.
2. **Use `<var>` for mathematical or programming variables**.
3. **Do not use `<var>` for arbitrary italic text**.

#### Constraints and Limitations

- `<var>` is not a generic emphasis element; use `<em>` for emphasis.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Mathematical Variable

**Setup Guide**: Save as `var-math.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Variable Example</title>
</head>
<body>
  <p>If <var>x</var> = 3 and <var>y</var> = 4, then <var>x</var> + <var>y</var> = 7.</p>
</body>
</html>
```

**Expected output**: The variables `x` and `y` appear in italic monospace.

**Why this output occurs**: `<var>` marks the letters as variables. Most browsers render them in italic monospace, distinguishing them from surrounding text.

---

#### Example 2: Programming Variable

**Setup Guide**: Save as `var-code.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Program Variable</title>
</head>
<body>
  <p>Set <var>count</var> to 0 before the loop.</p>
</body>
</html>
```

**Expected output**: “count” appears in italic monospace.

**Why this output occurs**: `<var>` marks the programming variable name, distinguishing it from function names or literal values.

---

### Real-World Cases

1. **Mathematics**: Equations and formulas.
2. **Programming tutorials**: Variable and parameter names.
3. **Scientific documentation**: Variables in formulas.

---

### References

- MDN Web Docs — `<var>`: The Variable element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/var
- WHATWG — The `var` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-var-element

---

## 6. Formatting Source Code

### Definitions

**Core Definition**
Formatting source code in HTML means using `<pre>` and `<code>` together to present code blocks with preserved whitespace, monospaced fonts, and semantic markup.

**Technical Definition**
The standard pattern is `<pre><code>…</code></pre>`. The `<pre>` element preserves whitespace and line breaks; the `<code>` element marks the content as computer code. Inside `<pre>`, characters such as `<` and `&` must be escaped as `&lt;` and `&amp;`. The `class` attribute on `<code>` can be used to indicate the language for syntax highlighting (e.g. `class="language-javascript"`).

**Beginner-Friendly Explanation**
To show a block of code on a web page, wrap it in `<pre><code>`. `<pre>` keeps the formatting, and `<code>` says “this is code.” You also need to escape `<` as `&lt;` so the browser does not treat it as a tag.

---

### Purposes

- **To display multi-line code blocks** with exact formatting.
- **To provide a semantic hook** for syntax highlighting.
- **To distinguish code from prose** for assistive technologies.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<pre><code class="language-javascript">
function greet(name) {
  return `Hello, ${name}!`;
}
</code></pre>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<pre>` | Preserves whitespace and line breaks. |
| `<code>` | Marks the content as code. |
| `class="language-…"` | Optional; used by syntax highlighters. |

#### Syntax Rules

1. **Always escape `<` and `&`** inside `<pre>`.
2. **Use `<pre><code>`** as the standard combination for code blocks.
3. **Place the opening `<code>` on the same line as `<pre>`** or account for the leading newline stripping.

#### Constraints and Limitations

- Very long lines may require horizontal scrolling; consider CSS `white-space: pre-wrap` or `overflow-x: auto`.
- Leading newline after `<pre>` is stripped, which can affect the first line of code.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: JavaScript Code Block

**Setup Guide**: Save as `code-block-js.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>JavaScript Code Block</title>
</head>
<body>
  <pre><code class="language-javascript">
function add(a, b) {
  return a + b;
}
</code></pre>
</body>
</html>
```

**Expected output**: A monospaced code block with the JavaScript function preserved exactly, ready for syntax highlighting if a library is added.

**Why this output occurs**: `<pre>` preserves the indentation and line breaks, while `<code>` marks the content as code and provides a language class for highlighting.

---

### Real-World Cases

1. **Documentation sites**: Code examples in API references.
2. **Blog posts**: Tutorial code snippets.
3. **Educational platforms**: Interactive code examples.

---

### References

- MDN Web Docs — `<code>`: Notes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/code#notes
- Google Style Guide — Code samples — https://developers.google.com/style/code-samples

---

## 7. Representing User Input

### Definitions

**Core Definition**
Representing user input means using `<kbd>` to mark text that the user is expected to type or press, such as keyboard shortcuts, commands, or voice input.

**Technical Definition**
`<kbd>` is the semantic element for user input. It can be nested to represent multi-key sequences, and it can be combined with `<samp>` to represent input that has been echoed back by a system.

**Beginner-Friendly Explanation**
When you tell someone to “press Ctrl+S” or “type `ls -la`,” you wrap the keys or command in `<kbd>`. This tells screen readers and browsers that the text is something the user should enter.

---

### Purposes

- **To clearly mark user input** in instructions.
- **To support keyboard shortcut documentation**.
- **To distinguish input from output** in transcripts.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<p>Press <kbd>Ctrl</kbd> + <kbd>S</kbd> to save.</p>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<kbd>` | Marks the user input. |
| Content | The key or command. |

#### Syntax Rules

1. **Use `<kbd>` for user input only**.
2. **Nest `<kbd>` inside `<kbd>`** for key sequences.
3. **Combine with `<samp>`** for terminal transcripts.

#### Constraints and Limitations

- Do not use `<kbd>` for program output; use `<samp>`.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Keyboard Shortcut

**Setup Guide**: Save as `user-input.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>User Input</title>
</head>
<body>
  <p>Press <kbd>Ctrl</kbd> + <kbd>Z</kbd> to undo.</p>
</body>
</html>
```

**Expected output**: “Ctrl” and “Z” are marked as keyboard input.

**Why this output occurs**: `<kbd>` provides the semantic meaning of user input, and the browser renders it in a monospaced font.

---

### Real-World Cases

1. **Help documentation**: Keyboard shortcuts.
2. **Tutorials**: Command-line instructions.
3. **Software manuals**: Key combinations.

---

### References

- MDN Web Docs — `<kbd>`: The Keyboard Input element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/kbd

---

## 8. Representing Program Output

### Definitions

**Core Definition**
Representing program output means using `<samp>` to mark text that is produced by a computer program, such as terminal output, console logs, or error messages.

**Technical Definition**
`<samp>` is the semantic element for sample output. It can contain `<kbd>` to show input within a transcript, and it is distinct from `<output>`, which is for output generated by the page’s own JavaScript.

**Beginner-Friendly Explanation**
When a program prints something — like “Hello, World!” or an error message — you wrap that text in `<samp>`. This tells the browser and screen readers that the text is output, not input or prose.

---

### Purposes

- **To mark program output** semantically.
- **To distinguish system responses** from user input and prose.
- **To support terminal transcripts** with clear roles for input and output.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<samp>Hello, World!</samp>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<samp>` | Marks the program output. |
| Content | The output text. |

#### Syntax Rules

1. **Use `<samp>` for program output only**.
2. **Nest `<kbd>` inside `<samp>`** for echoed input.
3. **Do not use `<samp>` for page-generated output**; use `<output>`.

#### Constraints and Limitations

- `<samp>` is for sample output, not for user input.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Program Output

**Setup Guide**: Save as `program-output.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Program Output</title>
</head>
<body>
  <p>The program prints: <samp>Build succeeded.</samp></p>
</body>
</html>
```

**Expected output**: “Build succeeded.” is marked as program output in a monospaced font.

**Why this output occurs**: `<samp>` indicates that the text is output from a program, distinguishing it from the surrounding prose.

---

### Real-World Cases

1. **Terminal documentation**: Showing command output.
2. **Tutorials**: Displaying program results.
3. **CLI references**: Illustrating output messages.

---

### References

- MDN Web Docs — `<samp>`: The Sample Output element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/samp

---

## 9. Code Best Practices

### Definitions

**Core Definition**
Code best practices are guidelines for marking up code semantically, escaping correctly, formatting consistently, and ensuring accessibility.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, Google’s documentation style guide, and WCAG requirements. They encompass using `<pre><code>` for blocks, `<code>` for inline fragments, escaping special characters, wrapping lines at reasonable lengths, and providing keyboard access for scrollable code blocks.

**Beginner-Friendly Explanation**
Best practices help you show code correctly and accessibly: use the right elements, escape `<` and `&`, keep lines readable, and make sure keyboard users can scroll long code blocks.

---

### Purposes

- **To ensure semantic correctness** of code markup.
- **To prevent rendering errors** from unescaped characters.
- **To improve readability** through consistent formatting.
- **To satisfy accessibility requirements** for scrollable code.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<pre><code>` for blocks** | `<pre>` preserves whitespace; `<code>` marks the content as code. |
| **Use `<code>` for inline fragments** | Wrap short code snippets inside paragraphs. |
| **Escape `<` and `&`** | Use `&lt;` and `&amp;` inside `<pre>`. |
| **Wrap lines at ~80 characters** | Improves readability in narrow windows. |
| **Make scrollable code keyboard-accessible** | Add `tabindex="0"` and `role="region"` with `aria-label` for scrollable `<pre>` blocks. |
| **Provide language classes** | Use `class="language-…"` on `<code>` for syntax highlighting. |

#### Syntax Rules

1. **Escape all ambiguous characters** inside `<pre>`.
2. **Use the `<pre><code>` combination** for multi-line code.
3. **Ensure keyboard access** for horizontally scrollable code blocks.

#### Constraints and Limitations

- Wrapping long lines must not change the meaning of the code.
- Scrollable code blocks without keyboard access fail WCAG 2.1.1 (Keyboard).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible Scrollable Code Block

**Setup Guide**: Save as `code-accessible.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible Code Block</title>
  <style>
    pre { overflow-x: auto; }
    pre:focus-visible { outline: 2px solid blue; }
  </style>
</head>
<body>
  <pre tabindex="0" role="region" aria-label="JavaScript example">
<code>function veryLongFunctionName(argumentOne, argumentTwo) {
  return argumentOne + argumentTwo;
}</code></pre>
</body>
</html>
```

**Expected output**: The code block is focusable via keyboard and can be scrolled horizontally with the arrow keys, and it has a distinct accessible name.

**Why this output occurs**: `tabindex="0"` makes the scrollable region keyboard-focusable, `role="region"` identifies it as a landmark, and `aria-label` provides a distinct name for screen-reader navigation.

---

### Real-World Cases

1. **Documentation platforms**: Code blocks with copy buttons and keyboard scrolling.
2. **Technical blogs**: Syntax-highlighted examples.
3. **Educational tools**: Interactive code editors.

---

### References

- Google Style Guide — Code samples — https://developers.google.com/style/code-samples
- Accessibility notes on scrollable pre blocks — https://raw.githubusercontent.com/berkayturanci/keel/refs/heads/main/.jules/palette.md
- WCAG 2.1.1 Keyboard — https://www.w3.org/WAI/WCAG21/Understanding/keyboard

---

## 10. Preformatted Text Best Practices

### Definitions

**Core Definition**
Preformatted text best practices are guidelines for using `<pre>` correctly, handling whitespace, escaping characters, and ensuring accessibility for text-based diagrams and scrollable content.

**Technical Definition**
Best practices are informed by MDN, WCAG, and practical accessibility guidance. They encompass escaping reserved characters, accounting for leading-newline stripping, providing alternative descriptions for ASCII art, and making scrollable preformatted regions keyboard-accessible.

**Beginner-Friendly Explanation**
Best practices help you use `<pre>` without breaking your layout or excluding users: escape `<` and `&`, watch out for the first line, describe any diagrams, and make sure long lines can be scrolled with a keyboard.

---

### Purposes

- **To prevent rendering errors** from unescaped characters.
- **To preserve intended layout** for ASCII art and diagrams.
- **To ensure accessibility** for users of screen readers and keyboards.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Escape `<` and `&`** | Use `&lt;` and `&amp;` inside `<pre>`. |
| **Account for leading newline stripping** | The first newline after `<pre>` is removed; adjust diagrams accordingly. |
| **Provide alternate descriptions for ASCII art** | Use `<figure>` with `<figcaption>` or ARIA labels. |
| **Make scrollable regions keyboard-accessible** | Add `tabindex="0"` and `role="region"` with `aria-label`. |
| **Use CSS for width and wrapping** | The `width` and `wrap` attributes are obsolete; use `width` and `white-space` in CSS. |

#### Syntax Rules

1. **Escape all reserved characters** inside `<pre>`.
2. **Provide text alternatives** for visual preformatted content.
3. **Ensure keyboard scrolling** for overflow content.

#### Constraints and Limitations

- Leading newline stripping can shift ASCII art; always test the rendered output.
- Scrollable preformatted content without keyboard access is an accessibility failure.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Accessible ASCII Art with Figure

**Setup Guide**: Save as `pre-ascii-accessible.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Accessible ASCII Art</title>
</head>
<body>
  <figure>
    <pre role="img" aria-label="ASCII art of a cat">
    /\_/\
   ( o.o )
    > ^ <
    </pre>
    <figcaption>A simple ASCII cat.</figcaption>
  </figure>
</body>
</html>
```

**Expected output**: The ASCII art is announced as an image with the label “ASCII art of a cat,” and the caption provides additional context.

**Why this output occurs**: The `role="img"` and `aria-label` on `<pre>` give the preformatted diagram an accessible name, and the `<figure>`/`<figcaption>` provide a visible caption.

---

### Real-World Cases

1. **Technical documentation**: Code blocks and terminal output.
2. **Educational content**: Diagrams and charts made with text.
3. **Accessibility-critical sites**: Preformatted content that must be usable by all.

---

### References

- MDN Web Docs — `<pre>`: Accessibility — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/pre#accessibility
- Preformatted text best practices — https://raw.githubusercontent.com/serpuniversity/learn-docs/refs/heads/main/html/HTML%20pre%20The%20Preformatted%20Text%20Element.md
- WCAG 1.1.1 Non-text Content — https://www.w3.org/WAI/WCAG21/Understanding/non-text-content

---

## Summary Table of Code and Preformatted Text Elements

| Element | Primary Role | Content Model | Default Rendering |
|---|---|---|---|
| `<code>` | Code fragment | Phrasing content | Monospace |
| `<pre>` | Preformatted text | Phrasing content | Monospace, whitespace preserved |
| `<kbd>` | User input | Phrasing content | Monospace |
| `<samp>` | Program output | Phrasing content | Monospace |
| `<var>` | Variable | Phrasing content | Monospace, italic |

---

## Final Notes

- **Semantic distinction matters**: `<code>`, `<kbd>`, `<samp>`, and `<var>` each convey different meanings; do not use them interchangeably.
- **`<pre>` is for whitespace preservation**: Use it whenever line breaks and spaces must be displayed exactly.
- **Escape ambiguous characters**: Inside `<pre>`, always escape `<` and `&` to avoid parser errors.
- **Accessibility is essential**: Scrollable code blocks need keyboard access, and ASCII art needs text alternatives.
- **Validate and test**: Check rendered output for leading-newline stripping and test scrollability with a keyboard.

---

This cheat sheet has covered code and preformatted text elements — `<code>`, `<pre>`, `<kbd>`, `<samp>`, and `<var>` — plus formatting source code, representing user input and program output, and best practices, with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate markup against the WHATWG HTML Living Standard, WCAG accessibility guidelines, and documentation style guides.