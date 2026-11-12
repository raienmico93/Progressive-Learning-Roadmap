# Table Accessibility — Comprehensive Programming Cheat Sheet & Research

## Topic Overview

### Definitions

**Core Definition**
Table accessibility is the practice of building HTML tables so that their structure, headers, captions, and relationships are programmatically determinable by assistive technologies. It ensures that screen-reader users can navigate tables cell by cell, hear the correct header context for each data cell, understand the table's purpose, and skip tables that are not relevant to them.

**Technical Definition**
In WCAG 2.2, table accessibility is governed primarily by **Success Criterion 1.3.1 (Info and Relationships, Level A)** , which requires that information, structure, and relationships conveyed through presentation be programmatically determinable or available in text. The W3C Techniques for tables include **H39** (using `<caption>` to associate captions with data tables), **H43** (using `id` and `headers` attributes to associate data cells with header cells in complex tables), **H51** (using table markup to present tabular information), and **H63** (using the `scope` attribute to associate header cells and data cells in data tables). Failures include **F91** (not correctly marking up table headers) and **F46** (using `th` elements for layout tables). The HTML specification adds that tables must not be used as layout aids, and that a table's cells must completely cover the grid without overlap.

**Beginner-Friendly Explanation**
Table accessibility means making sure that when a screen reader reads a table, the user knows what the table is about (the caption), what each column and row means (the headers), and how each data cell relates to those headers. Without proper markup, a screen-reader user hears a stream of numbers with no context — “15%, 20%, 10%” — with no idea what those percentages refer to. With proper headers, captions, and associations, they hear “Homework, 15%; Exams, 15%; Projects, 10%” and can navigate the table intelligently.

---

### Key Characteristics

- **Header association is the core requirement**: Headers must be marked with `<th>`, not `<td>` with bold styling; failure F91 defines this as a WCAG 1.3.1 failure.
- **`scope` is the primary association mechanism**: `scope="col"`, `scope="row"`, `scope="colgroup"`, and `scope="rowgroup"` associate headers with cells directionally.
- **`id` and `headers` handle complex tables**: When a data cell belongs to multiple headers, `id` on the headers and `headers` on the cell make the relationships explicit (Technique H43).
- **Captions identify tables**: `<caption>` provides the table's accessible name and is navigable by screen readers (Technique H39).
- **Summaries are handled by `<caption>` now**: The deprecated `summary` attribute on `<table>` is obsolete; the `<caption>` element is the correct mechanism for table titles and descriptions.
- **Layout tables are prohibited**: The specification states “Tables must not be used as layout aids”; screen readers cannot distinguish layout tables from data tables without correct markup.
- **Table navigation commands are essential**: Screen readers provide commands to jump between tables, move cell by cell, and hear headers as context changes.

---

### Prerequisites

1. **Basic HTML table structure**: familiarity with `<table>`, `<tr>`, `<th>`, `<td>`, `<caption>`, `<thead>`, `<tbody>`, and `<tfoot>`.
2. **WCAG 1.3.1 awareness**: understanding that structure must be programmatically determinable.
3. **The table model**: awareness that rows and columns form a grid and that headers provide context for data cells.
4. **Screen-reader basics**: awareness of how NVDA, JAWS, and VoiceOver navigate tables and announce headers.
5. **CSS table styling**: understanding that styling must not remove semantic structure.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **WCAG conformance** | 1.3.1 is the core criterion; Techniques H39, H43, H51, H63, and failures F91/F46 apply. |
| **Screen readers** | NVDA, JAWS, and VoiceOver provide table navigation and header announcement. |
| **CSS** | `border-collapse`, sticky headers, and responsive layouts must not break semantic structure. |
| **PDF and print** | `scope` and `headers` associations carry over to tagged PDF exports. |
| **Data visualisation** | Tables are the accessible foundation for charts and dashboards. |
| **SEO** | Well-structured tables can be parsed for featured snippets and structured data. |

---

### Core Concepts / Features

1. **Table accessibility** — the discipline
2. **Table headers** — `<th>`, `scope`, `id`/`headers`
3. **Table captions** — `<caption>` and accessible names
4. **Table summaries** — the obsolete `summary` attribute and its replacement
5. **Table accessibility best practices** — consolidated guidelines

---

## 1. Table Accessibility — The Discipline

### Definitions

**Core Definition**
Table accessibility is the practice of marking up tabular data so that its structure, headers, and relationships are conveyed to assistive technologies, enabling screen-reader users to navigate and understand tables.

**Technical Definition**
Table accessibility is achieved by satisfying WCAG 1.3.1 through the table techniques: using `<table>`, `<tr>`, `<th>`, and `<td>` (Technique H51), providing a `<caption>` (Technique H39), associating headers with data cells via `scope` (Technique H63) or `id`/`headers` (Technique H43), and avoiding layout tables (Failure F46). The table must form a complete grid without overlap, and header cells must use `<th>` rather than `<td>` styled as a header (Failure F91).

**Beginner-Friendly Explanation**
Table accessibility means using the right table tags so that screen readers can tell users what the table is, what the headers are, and how each cell relates to them.

---

### Purposes

- **To satisfy WCAG 1.3.1 (Info and Relationships)** by making table structure programmatically determinable.
- **To give screen-reader users context** for every data cell by announcing the relevant headers.
- **To enable table navigation** so users can jump between tables, rows, and cells.
- **To make tables usable on small screens** through responsive techniques that preserve semantics.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<table>
  <caption>Table title</caption>
  <thead>
    <tr>
      <th scope="col">Header 1</th>
      <th scope="col">Header 2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Row header</th>
      <td>Data</td>
    </tr>
  </tbody>
</table>
```

**Component breakdown**

| Component | Role |
|---|---|
| `<caption>` | Table title and accessible name. |
| `<thead>` | Header row group. |
| `<tbody>` | Body row group. |
| `<tr>` | A table row. |
| `<th scope>` | A header cell associated directionally with data cells. |
| `<td>` | A data cell. |

#### Syntax Rules

1. **Use `<th>` for headers**, not `<td>` with bold styling; this is a WCAG failure (F91).
2. **Add `scope` to every header** whenever possible; use `id`/`headers` only for complex tables.
3. **Provide a `<caption>`** to identify the table.
4. **Avoid layout tables**; use CSS Grid or Flexbox for layout.
5. **Ensure the grid is complete**; cells must cover the grid without overlap.

#### Constraints and Limitations

- **Layout tables are non-conforming** and cannot be distinguished from data tables by screen readers.
- **Empty or overlapping cells break the table model** and confuse navigation.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Simple Accessible Table

```html
<table>
  <caption>Current employee directory</caption>
  <thead>
    <tr>
      <th scope="col">Employee</th>
      <th scope="col">Role</th>
      <th scope="col">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Alex Rivera</th>
      <td>Engineer</td>
      <td>Active</td>
    </tr>
    <tr>
      <th scope="row">Sam Chen</th>
      <td>Designer</td>
      <td>Active</td>
    </tr>
  </tbody>
</table>
```

**Expected output**: A screen reader announces the caption, then when navigating cells announces “Employee, Alex Rivera, Engineer, Active” and “Employee, Sam Chen, Designer, Active.”

**Why this output occurs**: The `<caption>` provides the table's accessible name, the `<th scope="col">` elements identify column headers, and the `<th scope="row">` elements identify row headers. Together they make the relationships programmatically determinable.

---

#### Example 2: A Layout Table (Anti-Pattern)

```html
<!-- ANTI-PATTERN: table used for page layout -->
<table>
  <tr>
    <td>Sidebar</td>
    <td>Main content</td>
  </tr>
</table>
```

**Expected output**: The screen reader announces a data table with no headers, making the page confusing and difficult to navigate.

**Why this output occurs**: The specification prohibits layout tables, and Failure F46 defines using `<table>` for layout as a WCAG failure because screen readers cannot distinguish it from a data table.

---

### Real-World Cases

1. **Employee directories**: Names, roles, and statuses with row and column headers.
2. **Financial reports**: Quarterly revenue by product line.
3. **Comparison tables**: Product features compared across models.

---

### References

- W3C — Technique H51: Using table markup to present tabular information — https://www.w3.org/WAI/WCAG22/Techniques/html/H51.html
- W3C — Technique F91: Failure due to not correctly marking up table headers — https://www.w3.org/WAI/WCAG22/Techniques/failures/F91.html
- W3C — Technique F46: Failure due to using `th` elements for layout tables — https://www.w3.org/WAI/WCAG22/Techniques/failures/F46.html
- MDN Web Docs — HTML table accessibility — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility

---

## 2. Table Headers — `<th>`, `scope`, `id`/`headers`

### Definitions

**Core Definition**
Table headers are cells that label a row, column, row group, or column group. They are marked with `<th>` and associated with data cells via `scope` or `id`/`headers`.

**Technical Definition**
`<th>` elements are header cells; using `<td>` with bold styling instead is a WCAG failure (F91). The `scope` attribute (Technique H63) associates a header directionally: `row`, `col`, `rowgroup`, or `colgroup`. When a data cell is associated with more than one row and/or column header, `id` on the headers and `headers` on the data cell make the associations explicit (Technique H43). Screen readers use these associations to announce the correct headers when the user navigates to a data cell.

**Beginner-Friendly Explanation**
Headers tell users what each row and column means. `<th>` is the header tag. `scope` says “this header applies to this row or column.” `headers` is used for complex tables where a cell needs several headers.

---

### Purposes

- **To label rows and columns** so data cells have context.
- **To satisfy WCAG 1.3.1** by making header associations programmatically determinable.
- **To enable screen readers to announce the correct headers** for each cell.

---

### Syntax Rules and Structure

```html
<th scope="col">Product</th>
<th scope="row">Wireless Headphones</th>
```

| Attribute | Purpose |
|---|---|
| `scope="col"` | Applies to the cells in the header's column. |
| `scope="row"` | Applies to the cells in the header's row. |
| `scope="colgroup"` | Applies to the rest of the column group. |
| `scope="rowgroup"` | Applies to the rest of the row group. |
| `headers="id1 id2"` | Associates the cell with the listed header IDs. |

**Constraints**: `scope` and `abbr` on `<td>` are obsolete; use `<th>` instead. `scope="rowgroup"` requires the `<th>` to be anchored in a row group.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Simple Table with `scope`

```html
<table>
  <tr>
    <th scope="col">Name</th>
    <th scope="col">Age</th>
  </tr>
  <tr>
    <th scope="row">Chris</th>
    <td>22</td>
  </tr>
</table>
```

**Expected output**: A screen reader announces “Name, Chris, 22” — the column header and row header are associated with the data cell.

**Why this output occurs**: `scope="col"` tells the screen reader that “Name” and “Age” label the columns, and `scope="row"` tells it that “Chris” labels the row.

---

#### Example 2: Complex Table with `id`/`headers`

```html
<table>
  <tr>
    <th rowspan="2" id="h">Homework</th>
    <th colspan="3" id="e">Exams</th>
    <th colspan="3" id="p">Projects</th>
  </tr>
  <tr>
    <th id="e1" headers="e">1</th>
    <th id="e2" headers="e">2</th>
    <th id="ef" headers="e">Final</th>
    <th id="p1" headers="p">1</th>
    <th id="p2" headers="p">2</th>
    <th id="pf" headers="p">Final</th>
  </tr>
  <tr>
    <td headers="h">15%</td>
    <td headers="e e1">15%</td>
    <td headers="e e2">15%</td>
    <td headers="e ef">20%</td>
    <td headers="p p1">10%</td>
    <td headers="p p2">10%</td>
    <td headers="p pf">15%</td>
  </tr>
</table>
```

**Expected output**: Each data cell is associated with all relevant headers; the cell “15%” under Exams 1 is associated with both “Exams” and “1.”

**Why this output occurs**: The `id` attributes on the headers and the `headers` attributes on the data cells make explicit every header that applies to each cell, satisfying Technique H43 for complex tables.

---

### Real-World Cases

1. **Employee directories**: `<th scope="col">` for column headers and `<th scope="row">` for employee names.
2. **Financial reports**: Multi-level headers with `colspan` and `scope="colgroup"`.
3. **Grade breakdowns**: Complex tables with `id`/`headers` for category and assessment associations.

---

### References

- W3C — Technique H63: Using the `scope` attribute — https://w3c.github.io/wcag/techniques/html/H63
- W3C — Technique H43: Using `id` and `headers` attributes — https://w3c.github.io/wcag/techniques/html/H43
- MDN Web Docs — `<th>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/th

---

## 3. Table Captions — `<caption>` and Accessible Names

### Definitions

**Core Definition**
The `<caption>` element provides a title or short description for a table, serving as the table's accessible name.

**Technical Definition**
`<caption>` must be the first child of its parent `<table>`. It is announced by screen readers as the table's name and can be navigated to using table navigation commands. Technique H39 uses `<caption>` to associate data table captions with the table, and is a sufficient technique for WCAG 1.3.1. The caption is particularly helpful when a table is preceded by introductory text that describes it, because the caption remains associated with the table when copied, bookmarked, or printed.

**Beginner-Friendly Explanation**
`<caption>` is the table's title. It tells the user what the table is about and stays with the table wherever it goes.

---

### Purposes

- **To provide the table's accessible name**.
- **To describe the table's overall topic**.
- **To satisfy WCAG 1.3.1** through Technique H39.

---

### Syntax Rules and Structure

```html
<table>
  <caption>Table title</caption>
  <!-- rest of the table -->
</table>
```

**Constraints**: `<caption>` must be the first child of `<table>`. At most one `<caption>` per table. The `summary` attribute on `<table>` is obsolete; use `<caption>` instead.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Table with a Caption

```html
<table>
  <caption>Dinosaurs in the Jurassic period</caption>
  <tr>
    <th scope="col">Dinosaur</th>
    <th scope="col">Period</th>
  </tr>
  <tr>
    <td>Stegosaurus</td>
    <td>Late Jurassic</td>
  </tr>
</table>
```

**Expected output**: The caption “Dinosaurs in the Jurassic period” appears above the table and is announced as the table's name.

**Why this output occurs**: The `<caption>` element is the first child of `<table>`, so it provides the accessible name and is navigable by screen readers.

---

### Real-World Cases

1. **Financial reports**: “Quarterly Revenue by Product Line”.
2. **Timetables**: “St. Winnifred's weekly language lesson timetable”.
3. **Comparison tables**: “Product Feature Comparison”.

---

### References

- W3C — Technique H39: Using caption elements to associate data table captions with data tables — https://w3c.github.io/wcag/techniques/html/H39
- MDN Web Docs — `<caption>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/caption

---

## 4. Table Summaries — The Obsolete `summary` Attribute and Its Replacement

### Definitions

**Core Definition**
A table summary is a longer description of a table's structure or purpose, beyond the title. Historically, the `summary` attribute on `<table>` provided this description; it is now obsolete and replaced by the `<caption>` element.

**Technical Definition**
The `summary` attribute on `<table>` was deprecated in HTML5 and is obsolete in the current WHATWG HTML Living Standard. The correct mechanism for a table's title or short description is the `<caption>` element. For longer descriptions of complex tables, the WAI guidance and HTML specification recommend providing the description in the surrounding text (e.g. a preceding paragraph) or, when the table is inside a `<figure>`, using `<figcaption>`. The `summary` attribute is not announced by modern screen readers consistently and must not be used in new documents.

**Beginner-Friendly Explanation**
A table summary is a longer explanation of what a table contains and how it is structured. The old way to add one was the `summary` attribute, but that is now obsolete. Use `<caption>` for the title and put longer descriptions in the text around the table.

---

### Purposes

- **To describe the table's structure or purpose** in more detail than a caption.
- **To help screen-reader users understand complex tables** before navigating them.
- **To replace the obsolete `summary` attribute** with modern, conforming markup.

---

### Syntax Rules and Structure

```html
<!-- GOOD: caption provides the title; preceding text provides the summary -->
<p>The table below compares Q1–Q4 revenue by product line; totals are in the final row.</p>
<table>
  <caption>Quarterly Revenue by Product Line</caption>
  <!-- rest of the table -->
</table>
```

**Constraints**: The `summary` attribute is obsolete and must not be used. Use `<caption>` for the title and surrounding text for longer descriptions.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Caption and Surrounding Description

```html
<p>The table below compares quarterly revenue across three product lines. The final row shows the annual total.</p>
<table>
  <caption>Quarterly Revenue by Product Line</caption>
  <thead>
    <tr>
      <th scope="col">Product</th>
      <th scope="col">Q1</th>
      <th scope="col">Q2</th>
      <th scope="col">Q3</th>
      <th scope="col">Q4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Headphones</th>
      <td>$10k</td>
      <td>$12k</td>
      <td>$15k</td>
      <td>$18k</td>
    </tr>
  </tbody>
</table>
```

**Expected output**: The screen reader announces the table's caption and the preceding paragraph provides a longer description of the table's structure.

**Why this output occurs**: The `<caption>` provides the accessible name, and the preceding paragraph provides the longer summary that the obsolete `summary` attribute used to carry.

---

### Real-World Cases

1. **Financial reports**: A caption plus a preceding paragraph describing the columns and totals.
2. **Complex tables**: A `<figure>` with `<figcaption>` when the table is inside a figure.
3. **Data dashboards**: A caption plus a link to a full description elsewhere on the page.

---

### References

- MDN Web Docs — `<table>`: `summary` (obsolete) — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/table#summary
- W3C — HTML 5.2: Obsolete features (`summary`) — https://www.w3.org/TR/html52/obsolete.html#the-summary-attribute

---

## 5. Table Accessibility Best Practices

### Definitions

**Core Definition**
Table accessibility best practices are the consolidated guidelines for building tables that are semantically correct, navigable, and understandable by assistive technologies — covering headers, captions, summaries, complex associations, layout-table avoidance, and responsive techniques.

**Technical Definition**
Best practices are informed by WCAG 2.2 SC 1.3.1, Techniques H39, H43, H51, H63, and failures F91/F46. They encompass using semantic table markup, marking headers with `<th>` and `scope`, providing a `<caption>`, using `id`/`headers` for complex tables, replacing the obsolete `summary` attribute with a caption and surrounding text, never using tables for layout, and ensuring responsive tables remain keyboard-accessible.

**Beginner-Friendly Explanation**
Best practices help you build tables that everyone can use: use real table tags, mark headers with `<th>`, add `scope`, give every table a caption, describe complex tables in the surrounding text, avoid layout tables, and make tables work on phones.

---

### Purposes

- **To satisfy WCAG 1.3.1** through correct header associations and captions.
- **To enable screen-reader navigation** of tables and their headers.
- **To prevent common failures** such as `<td>` headers, layout tables, and missing captions.
- **To keep tables usable on small screens** through responsive techniques.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use semantic table markup** | `<table>`, `<tr>`, `<th>`, `<td>`; not `<div>` grids. |
| **Mark headers with `<th>`** | Never use `<td>` with bold styling as a header (F91). |
| **Add `scope` to every header** | `col`, `row`, `colgroup`, or `rowgroup`. |
| **Use `id`/`headers` for complex tables** | When a data cell belongs to multiple headers. |
| **Provide a `<caption>`** | Identifies the table and satisfies H39. |
| **Replace `summary` with caption + surrounding text** | The `summary` attribute is obsolete. |
| **Never use tables for layout** | The specification prohibits it; Failure F46. |
| **Make tables responsive and keyboard-accessible** | Wrap in a scrollable container with `tabindex="0"` and `role="region"`. |

#### Constraints and Limitations

- **Layout tables cannot be distinguished from data tables** by screen readers; avoid them entirely.
- **The `summary` attribute is obsolete** and must not be used in new documents.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Fully Accessible Responsive Table

```html
<div class="table-wrapper" tabindex="0" role="region" aria-labelledby="employee-caption">
  <table>
    <caption id="employee-caption">Current employee directory</caption>
    <thead>
      <tr>
        <th scope="col">Employee</th>
        <th scope="col">Role</th>
        <th scope="col">Status</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">Alex Rivera</th>
        <td>Engineer</td>
        <td>Active</td>
      </tr>
      <tr>
        <th scope="row">Sam Chen</th>
        <td>Designer</td>
        <td>Active</td>
      </tr>
    </tbody>
  </table>
</div>
```

**Expected output**: A table with a caption, column and row headers, and a keyboard-accessible scrollable container with an accessible name.

**Why this output occurs**: The `<caption>` provides the table's name; the `<th scope>` attributes associate headers with cells; the wrapper's `tabindex="0"` and `role="region"` make the scrollable area keyboard-accessible, and `aria-labelledby` gives it an accessible name.

---

#### Example 2: Common Accessibility Failures

```html
<!-- FAILURE 1: headers as td without scope -->
<!--
<table>
  <tr><td>Name</td><td>Age</td></tr>
  <tr><td>Linda</td><td>33</td></tr>
</table>
-->

<!-- FAILURE 2: no caption -->
<!--
<table>
  <tr><th scope="col">Name</th><th scope="col">Age</th></tr>
  <tr><td>Linda</td><td>33</td></tr>
</table>
-->

<!-- FAILURE 3: layout table -->
<!--
<table>
  <tr><td>Sidebar</td><td>Main content</td></tr>
</table>
-->

<!-- FAILURE 4: obsolete summary attribute -->
<!-- <table summary="This table shows employee data"> -->
```

**Expected output**: Each failure violates a WCAG rule: `<td>` headers fail F91, missing captions fail H39, layout tables fail F46, and the `summary` attribute is obsolete.

**Why this output occurs**: The failures break the header associations, remove the table's accessible name, or use obsolete/non-conforming markup.

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 audits check for `<th>`, `scope`, captions, and keyboard access.
2. **E-commerce comparison tables**: `<caption>`, `<th scope>`, and responsive scrollable wrappers.
3. **Financial reports**: Complex tables with `id`/`headers` and a caption plus preceding description.

---

### References

- W3C — Technique H39: Using caption elements — https://w3c.github.io/wcag/techniques/html/H39
- W3C — Technique H43: Using `id` and `headers` attributes — https://w3c.github.io/wcag/techniques/html/H43
- W3C — Technique H51: Using table markup — https://www.w3.org/WAI/WCAG22/Techniques/html/H51.html
- W3C — Technique H63: Using the `scope` attribute — https://w3c.github.io/wcag/techniques/html/H63
- W3C — Technique F91: Failure due to not correctly marking up table headers — https://www.w3.org/WAI/WCAG22/Techniques/failures/F91.html
- W3C — Technique F46: Failure due to using `th` elements for layout tables — https://www.w3.org/WAI/WCAG22/Techniques/failures/F46.html
- MDN Web Docs — HTML table accessibility — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility

---

## Summary Table of Table Accessibility Concepts

| Concept | WCAG Criterion | Key Technique | Common Failure |
|---|---|---|---|
| **Table accessibility** | 1.3.1 | Semantic table markup (H51) | Layout tables (F46) |
| **Table headers** | 1.3.1 | `<th>` + `scope` (H63) or `id`/`headers` (H43) | `<td>` headers (F91) |
| **Table captions** | 1.3.1 | `<caption>` (H39) | Missing caption |
| **Table summaries** | 1.3.1 | Caption + surrounding text | Obsolete `summary` attribute |
| **Best practices** | Multiple | Combined techniques | Layout tables, missing scope, missing captions |

---

## Final Notes

- **Headers must be `<th>`**: `<td>` with bold styling is a WCAG failure, not a header.
- **`scope` is the first tool**: Add `scope="col"` or `scope="row"` to every header; use `id`/`headers` only for complex tables.
- **Captions identify tables**: `<caption>` provides the accessible name and satisfies Technique H39.
- **Summaries belong in captions and surrounding text**: The `summary` attribute is obsolete and must not be used.
- **Layout tables are prohibited**: Screen readers cannot distinguish them from data tables, and Failure F46 defines them as a WCAG failure.
- **Responsive tables need keyboard access**: Wrap scrollable tables in a container with `tabindex="0"`, `role="region"`, and an accessible name.
- **Test with real screen readers**: NVDA, JAWS, and VoiceOver all navigate tables differently; manual testing is essential.

---

This cheat sheet has covered table accessibility — the discipline, headers, captions, summaries, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate table markup against WCAG 2.2 Success Criterion 1.3.1 (Info and Relationships) and the W3C table techniques H39, H43, H51, H63, and failures F91/F46.