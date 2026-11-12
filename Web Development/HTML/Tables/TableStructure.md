# Table Structure — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
Table structure is the system of HTML elements that organize tabular data into a grid of rows, columns, and cells. The `<table>` element is the container, and its child elements — `<caption>`, `<colgroup>`, `<col>`, `<thead>`, `<tbody>`, `<tfoot>`, `<tr>`, `<th>`, and `<td>` — define the table's title, column groups, header rows, body rows, footer rows, and individual header and data cells.

**Technical Definition**
In the WHATWG HTML Living Standard, a table is defined as “data with more than one dimension, in the form of a table”. The `<table>` element's content model is, **in this order**: an optional `<caption>` element, followed by zero or more `<colgroup>` elements, followed optionally by a `<thead>` element, followed by either zero or more `<tbody>` elements or one or more `<tr>` elements, followed optionally by a `<tfoot>` element, optionally intermixed with one or more script-supporting elements . The rows and columns form a **grid**, and a table's cells must completely cover that grid without overlap . The table model defines how cells are assigned to slots; a cell cannot cover slots from two or more row groups . The `<th>` element represents a header cell, and `<td>` represents a data cell; both share the `colspan` and `rowspan` attributes for spanning multiple columns or rows .

**Beginner-Friendly Explanation**
A table is a grid of information — like a spreadsheet. You build it from the outside in: `<table>` is the container, `<caption>` gives it a title, `<thead>` holds the header row, `<tbody>` holds the data rows, and `<tfoot>` holds the footer row. Each row is a `<tr>`, each header cell is a `<th>`, and each data cell is a `<td>`. If you want to style whole columns at once, you group them with `<colgroup>` and `<col>`. The structure makes the relationships between headers and data programmatically determinable, so screen readers can announce the correct context for every cell.

---

### Key Characteristics

- **Strict content-model order**: The child elements must appear in a specific sequence: caption → colgroup* → thead? → (tbody* | tr+) → tfoot? .
- **Grid completeness**: Cells must completely cover the grid without overlap; `colspan` and `rowspan` must be used correctly .
- **Header cells are mandatory**: `<th>` elements make headers programmatically determinable; using `<td>` with bold styling is a WCAG failure .
- **Caption provides the accessible name**: `<caption>` identifies the table and is the first child if present .
- **Row groups are optional but recommended**: `<thead>`, `<tbody>`, and `<tfoot>` add semantic structure and enable CSS styling and print repetition .
- **Column groups are for styling**: `<colgroup>` and `<col>` group columns for presentation; they do not create headers .
- **Tables are not for layout**: The specification explicitly states “Tables must not be used as layout aids” .

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements and content models.
2. **The distinction between data and layout**: understanding that tables are for tabular data, not page structure.
3. **CSS fundamentals**: `border-collapse`, `display: table-*`, and column styling.
4. **Accessibility awareness**: understanding WCAG 1.3.1 (Info and Relationships) and how screen readers navigate tables.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | 1.3.1 is the core criterion; Techniques H39, H43, H51, and failures F91/F46 apply. |
| **CSS** | `border-collapse`, `table-layout`, and `col` styling control presentation. |
| **JavaScript** | `HTMLTableElement` provides `caption`, `tHead`, `tFoot`, `insertRow()`, and `insertCell()`. |
| **Responsive design** | Sticky headers, horizontal scrolling, and stacked layouts adapt tables to small screens. |
| **Data visualisation** | Tables are the accessible foundation for charts and dashboards. |

---

### Core Concepts / Features

1. **`<caption>`** — the table title and accessible name
2. **`<colgroup>`** — the column group container
3. **`<col>`** — a column within a column group
4. **`<thead>`** — the header row group
5. **`<tbody>`** — the body row group
6. **`<tfoot>`** — the footer row group
7. **`<tr>`** — a table row
8. **`<th>`** — a header cell
9. **`<td>`** — a data cell
10. **Table structure best practices** — consolidated guidelines

---

## 1. `<caption>` — The Table Title

### Definitions

**Core Definition**
The `<caption>` element specifies the caption (or title) of a table, providing the table with an accessible name or description .

**Technical Definition**
`<caption>` must be the **first child** of its parent `<table>` element . Its content model is flow content, but with no descendant `<table>` elements . The caption is displayed as a title for the table and is announced by screen readers as the table's name . If a `<table>` is the only content in a `<figure>` other than a `<figcaption>`, the `<figcaption>` should be used instead of `<caption>` .

**Beginner-Friendly Explanation**
`<caption>` is the table's title. It goes right after the opening `<table>` tag, and it tells everyone — sighted users and screen-reader users — what the table is about.

---

### Purposes

- **To provide the table's accessible name** .
- **To describe the table's overall topic** .
- **To give users a quick idea of whether the table is useful** before reading its contents .

---

### Syntax Rules and Structure

```html
<table>
  <caption>Table title</caption>
  <!-- rest of the table -->
</table>
```

| Rule | Requirement |
|---|---|
| Position | Must be the first child of `<table>` . |
| Content model | Flow content, no descendant `<table>` . |
| Uniqueness | At most one `<caption>` per table. |

**Constraints**: The `summary` attribute on `<table>` is deprecated; use `<caption>` instead . The `<figcaption>` is preferred when the table is inside a `<figure>` .

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

**Expected output**: The caption “Dinosaurs in the Jurassic period” appears above the table, and a screen reader announces it as the table's name.

**Why this output occurs**: The `<caption>` element is the first child of `<table>`, so the browser renders it as the table's title and exposes it as the accessible name .

---

### Real-World Cases

1. **Financial tables**: “Quarterly Revenue by Product Line”.
2. **Timetables**: “St. Winnifred's weekly language lesson timetable” .
3. **Comparison tables**: “Product Feature Comparison”.

---

### References

- MDN Web Docs — `<caption>`: The Table Caption element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/caption
- WHATWG — The `caption` element — https://html.spec.whatwg.org/dev/tables.html#the-caption-element

---

## 2. `<colgroup>` — The Column Group Container

### Definitions

**Core Definition**
The `<colgroup>` element represents a group of one or more columns in the table that is its parent .

**Technical Definition**
`<colgroup>` must be a child of `<table>`, appearing after any `<caption>` and before any `<thead>`, `<tbody>`, `<tfoot>`, or `<tr>` . Its content model is zero or more `<col>` elements, optionally intermixed with script-supporting elements. The `span` attribute specifies the number of consecutive columns the group spans . Column groups are used for applying styling to whole columns at once, not for creating headers .

**Beginner-Friendly Explanation**
`<colgroup>` groups columns together so you can style them as a unit. It sits between the caption and the header rows.

---

### Purposes

- **To group columns for styling** .
- **To apply column-based defaults** to multiple columns at once .
- **To define the number of columns** via the `span` attribute .

---

### Syntax Rules and Structure

```html
<table>
  <caption>Table title</caption>
  <colgroup>
    <col span="2">
    <col class="highlight">
  </colgroup>
  <thead>…</thead>
</table>
```

| Rule | Requirement |
|---|---|
| Position | After `<caption>`, before `<thead>`/`<tbody>`/`<tfoot>`/`<tr>` . |
| Content model | Zero or more `<col>` elements. |
| `span` attribute | Number of columns the group spans. |

**Constraints**: The `align`, `char`, `charoff`, `valign`, and `width` attributes on `<colgroup>` are obsolete; use CSS instead.

---

### Real-World Cases

1. **Timetables**: Grouping morning and afternoon columns for different background colours.
2. **Financial tables**: Grouping quarterly columns under a yearly heading.
3. **Comparison tables**: Grouping feature columns by category.

---

### References

- MDN Web Docs — `<colgroup>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/colgroup
- WHATWG — The `colgroup` element — https://html.spec.whatwg.org/dev/tables.html#the-colgroup-element

---

## 3. `<col>` — A Column Within a Column Group

### Definitions

**Core Definition**
The `<col>` element represents one or more columns in the column group represented by its `<colgroup>` parent .

**Technical Definition**
`<col>` is a void element; it has no content and no end tag. It must be a child of `<colgroup>`. The `span` attribute specifies the number of consecutive columns the `<col>` element spans . Column styling applied to a `<col>` affects all cells in the columns it spans .

**Beginner-Friendly Explanation**
`<col>` is a single column (or a span of columns) inside a `<colgroup>`. You use it to target specific columns for styling.

---

### Purposes

- **To style individual columns** within a column group .
- **To apply column-specific backgrounds, widths, or borders** .

---

### Syntax Rules and Structure

```html
<colgroup>
  <col span="2" class="left-columns">
  <col class="highlight">
</colgroup>
```

| Rule | Requirement |
|---|---|
| Element type | Void element (no end tag, no content). |
| Parent | `<colgroup>` . |
| `span` attribute | Number of columns spanned . |

**Constraints**: The `width`, `align`, `char`, `charoff`, and `valign` attributes are obsolete; use CSS instead.

---

### Real-World Cases

1. **Zebra-striping columns**: Alternating column background colours.
2. **Highlighting a data column**: Emphasising the “Total” column.
3. **Fixed-width columns**: Setting a consistent width for date columns.

---

### References

- MDN Web Docs — `<col>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/col
- WHATWG — The `col` element — https://html.spec.whatwg.org/dev/tables.html#the-col-element

---

## 4. `<thead>` — The Header Row Group

### Definitions

**Core Definition**
The `<thead>` element defines a set of rows that form the header of the table .

**Technical Definition**
`<thead>` must appear after any `<caption>` and `<colgroup>` elements and before any `<tbody>`, `<tfoot>`, or `<tr>` elements . Its content model is zero or more `<tr>` elements. The `<thead>` element does not affect the table model directly; it groups header rows for semantic clarity and styling. When printing a long table, the `<thead>` content can be repeated on each page .

**Beginner-Friendly Explanation**
`<thead>` wraps the header row(s) of the table — usually the row with the column titles.

---

### Purposes

- **To group header rows semantically** .
- **To enable CSS styling of the header section** .
- **To allow header rows to repeat on printed pages** .
- **To provide a navigable landmark** for screen readers.

---

### Syntax Rules and Structure

```html
<thead>
  <tr>
    <th scope="col">Name</th>
    <th scope="col">Age</th>
  </tr>
</thead>
```

| Rule | Requirement |
|---|---|
| Position | After `<caption>` and `<colgroup>`, before `<tbody>`/`<tfoot>`/`<tr>` . |
| Content model | Zero or more `<tr>` elements. |
| Uniqueness | At most one `<thead>` per table. |

**Constraints**: The `<thead>` must not be empty; it should contain at least one `<tr>` with header cells.

---

### Real-World Cases

1. **Data tables**: The row containing column headings.
2. **Long tables**: Headers that repeat on every printed page.
3. **Sticky headers**: CSS `position: sticky` on `<thead>` for scrolling tables.

---

### References

- MDN Web Docs — `<thead>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/thead
- WHATWG — The `thead` element — https://html.spec.whatwg.org/dev/tables.html#the-thead-element

---

## 5. `<tbody>` — The Body Row Group

### Definitions

**Core Definition**
The `<tbody>` element encapsulates a set of table rows that form the body of the table .

**Technical Definition**
`<tbody>` must appear after any `<caption>`, `<colgroup>`, and `<thead>` elements, and before any `<tfoot>` element . Its content model is zero or more `<tr>` elements. The `<tbody>` element is **implicitly created by the browser** if you use `<tr>` elements directly inside `<table>` without a `<tbody>` wrapper . Using an explicit `<tbody>` gives you more control over structure and styling .

**Beginner-Friendly Explanation**
`<tbody>` wraps the main data rows of the table — everything that is not the header or the footer.

---

### Purposes

- **To group body rows semantically** .
- **To enable CSS styling of the body section** .
- **To provide explicit structure** that the browser would otherwise add implicitly .
- **To support multiple `<tbody>` sections** for grouping related rows.

---

### Syntax Rules and Structure

```html
<tbody>
  <tr>
    <td>Chris</td>
    <td>22</td>
  </tr>
  <tr>
    <td>Dennis</td>
    <td>45</td>
  </tr>
</tbody>
```

| Rule | Requirement |
|---|---|
| Position | After `<thead>`, before `<tfoot>` . |
| Content model | Zero or more `<tr>` elements. |
| Implicit creation | The browser adds `<tbody>` if `<tr>` is used directly inside `<table>` . |

**Constraints**: You may have multiple `<tbody>` elements to group related rows (e.g. by category).

---

### Real-World Cases

1. **Grouped data**: Multiple `<tbody>` sections for different categories.
2. **Styling**: Applying alternating row colours via `tbody tr:nth-child()`.
3. **Scripting**: Targeting specific body sections for dynamic updates.

---

### References

- MDN Web Docs — `<tbody>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tbody
- WHATWG — The `tbody` element — https://html.spec.whatwg.org/dev/tables.html#the-tbody-element

---

## 6. `<tfoot>` — The Footer Row Group

### Definitions

**Core Definition**
The `<tfoot>` element defines a set of rows that form the footer of the table .

**Technical Definition**
`<tfoot>` must appear after any `<caption>`, `<colgroup>`, `<thead>`, and `<tbody>` elements. In earlier HTML versions, `<tfoot>` had to come before `<tbody>`, but the current standard allows it to come after . Its content model is zero or more `<tr>` elements. The footer often contains summary rows, such as totals .

**Beginner-Friendly Explanation**
`<tfoot>` wraps the footer row of the table — usually a row with totals or a summary.

---

### Purposes

- **To group footer rows semantically** .
- **To provide summary or totals rows** .
- **To enable CSS styling of the footer section** .

---

### Syntax Rules and Structure

```html
<tfoot>
  <tr>
    <td>Total</td>
    <td>$1,234.56</td>
  </tr>
</tfoot>
```

| Rule | Requirement |
|---|---|
| Position | After `<tbody>` elements . |
| Content model | Zero or more `<tr>` elements. |
| Uniqueness | At most one `<tfoot>` per table. |

**Constraints**: The `<tfoot>` must not be empty; it should contain at least one `<tr>`.

---

### Real-World Cases

1. **Financial tables**: A total row at the bottom.
2. **Sports standings**: A summary row with league averages.
3. **Inventory tables**: A footer row with total stock counts.

---

### References

- MDN Web Docs — `<tfoot>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tfoot
- WHATWG — The `tfoot` element — https://html.spec.whatwg.org/dev/tables.html#the-tfoot-element

---

## 7. `<tr>` — The Table Row

### Definitions

**Core Definition**
The `<tr>` element defines a row of cells in a table .

**Technical Definition**
`<tr>` may be a child of `<table>` (if no `<tbody>` is used), `<thead>`, `<tbody>`, or `<tfoot>`. Its content model is zero or more `<td>` and `<th>` elements, optionally intermixed with script-supporting elements . The cells within a row form the columns of the table. The `tr` end tag may be omitted if the row is immediately followed by another `<tr>` or the parent has no more content .

**Beginner-Friendly Explanation**
`<tr>` is a table row. It contains the cells for that row — header cells (`<th>`) or data cells (`<td>`).

---

### Purposes

- **To define a single row** in the table .
- **To contain the cells** that form the columns of that row .

---

### Syntax Rules and Structure

```html
<tr>
  <th scope="col">Name</th>
  <td>Chris</td>
  <td>22</td>
</tr>
```

| Rule | Requirement |
|---|---|
| Parent | `<table>`, `<thead>`, `<tbody>`, or `<tfoot>` . |
| Content model | Zero or more `<td>` and `<th>` elements . |
| Tag omission | End tag may be omitted before another `<tr>` . |

**Constraints**: A `<tr>` must not be empty; it should contain at least one cell.

---

### Real-World Cases

1. **Every table row**: Header rows, data rows, and footer rows.
2. **Dynamic tables**: JavaScript `insertRow()` adds a `<tr>` to a table.

---

### References

- MDN Web Docs — `<tr>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tr
- WHATWG — The `tr` element — https://html.spec.whatwg.org/dev/tables.html#the-tr-element

---

## 8. `<th>` — The Header Cell

### Definitions

**Core Definition**
The `<th>` element defines a cell that serves as a header for a group of table cells .

**Technical Definition**
`<th>` may be a child of `<tr>`. Its content model is flow content, but with no `<header>`, `<footer>`, sectioning content, or heading content descendants . The `scope` attribute specifies whether the header applies to a row, column, row group, or column group . The `abbr` attribute provides an abbreviated description for the header . The `colspan` and `rowspan` attributes allow the header to span multiple columns or rows .

**Beginner-Friendly Explanation**
`<th>` is a header cell. It labels a row or column so users — and screen readers — know what the data in that row or column represents.

---

### Purposes

- **To identify row and column headers** .
- **To make the table's structure programmatically determinable** (WCAG 1.3.1) .
- **To provide abbreviated headers** for long column titles .

---

### Syntax Rules and Structure

```html
<tr>
  <th scope="col">Product</th>
  <th scope="col">Price</th>
</tr>
```

| Attribute | Purpose |
|---|---|
| `scope` | `col`, `row`, `colgroup`, or `rowgroup` . |
| `abbr` | Abbreviated header text for screen readers. |
| `colspan` | Number of columns spanned. |
| `rowspan` | Number of rows spanned. |

**Constraints**: Using `<td>` with bold styling instead of `<th>` is a WCAG failure (Technique F91) .

---

### Real-World Cases

1. **Column headers**: The top row of a data table.
2. **Row headers**: The first cell in each row (e.g. a product name).
3. **Complex headers**: Multi-level headers with `colspan` and `scope` .

---

### References

- MDN Web Docs — `<th>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/th
- W3C — Technique H51: Using table markup to present tabular information — https://www.w3.org/WAI/WCAG22/Techniques/html/H51.html

---

## 9. `<td>` — The Data Cell

### Definitions

**Core Definition**
The `<td>` element defines a cell of a table that contains data .

**Technical Definition**
`<td>` may be a child of `<tr>`. Its content model is flow content . The `colspan` and `rowspan` attributes allow the cell to span multiple columns or rows . The `headers` attribute associates the cell with one or more header cells by `id` .

**Beginner-Friendly Explanation**
`<td>` is a data cell. It holds the actual data values in the table.

---

### Purposes

- **To hold data values** in the table grid .
- **To span multiple rows or columns** when a single value applies to several cells .
- **To associate with headers** via the `headers` attribute in complex tables .

---

### Syntax Rules and Structure

```html
<tr>
  <td>Chris</td>
  <td>22</td>
</tr>
```

| Attribute | Purpose |
|---|---|
| `colspan` | Number of columns spanned. |
| `rowspan` | Number of rows spanned. |
| `headers` | Space-separated IDs of associated header cells. |

**Constraints**: A `<td>` must be inside a `<tr>`. Empty `<td>` elements are allowed but should be used for genuinely empty data slots.

---

### Real-World Cases

1. **Data grids**: Every value cell in the table.
2. **Spanning cells**: A “Total” cell that spans multiple columns.
3. **Complex tables**: `headers` associations for multi-level headers.

---

### References

- MDN Web Docs — `<td>` — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/td
- W3C — Technique H43: Using id and headers attributes — https://w3c.github.io/wcag/techniques/html/H43

---

## 10. Table Structure Best Practices

### Definitions

**Core Definition**
Table structure best practices are the consolidated guidelines for building tables that are semantically correct, accessible, and maintainable — covering element order, header markup, caption usage, column grouping, and layout-table avoidance.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships), and W3C Techniques H39, H43, and H51. They encompass following the strict content-model order, using `<th>` for headers with `scope`, providing a `<caption>`, using `<colgroup>`/`<col>` for column styling, avoiding layout tables, and using CSS for all presentation.

**Beginner-Friendly Explanation**
Best practices help you build tables that work for everyone: use the right elements in the right order, mark headers as headers, give the table a caption, group columns for styling, and never use tables for page layout.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Follow the content-model order** | caption → colgroup* → thead? → (tbody* \| tr+) → tfoot? . |
| **Use `<th>` for headers** | Never use `<td>` with bold styling as a header (WCAG failure) . |
| **Provide a `<caption>`** | Identifies the table and gives it an accessible name . |
| **Use `scope` for simple tables** | `scope="col"` and `scope="row"` associate headers with cells . |
| **Use `id` and `headers` for complex tables** | When a data cell needs multiple headers . |
| **Use `<colgroup>` and `<col>` for column styling** | Group columns for CSS; do not use them for headers . |
| **Never use tables for layout** | The specification explicitly prohibits it . |
| **Use CSS for presentation** | Obsolete attributes like `align`, `bgcolor`, `border`, `cellpadding`, and `cellspacing` must not be used . |

#### Constraints and Limitations

- **Layout tables are non-conforming** and cause serious navigation problems for screen-reader users .
- **Cells must completely cover the grid without overlap** .
- **A cell cannot cover slots from two or more row groups** .

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Fully Structured Accessible Table

```html
<table>
  <caption>Employee Directory</caption>
  <colgroup>
    <col span="1" class="name-column">
    <col span="2" class="data-columns">
  </colgroup>
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
  <tfoot>
    <tr>
      <td colspan="3">2 employees</td>
    </tr>
  </tfoot>
</table>
```

**Expected output**: A table with a caption, column groups, header row, body rows, and a footer row. Screen readers announce the caption, the column headers, and the row headers as context for each cell.

**Why this output occurs**: The `<caption>` provides the table's name, the `<colgroup>` groups columns for styling, the `<thead>`/`<tbody>`/`<tfoot>` structure groups rows semantically, and the `<th scope>` attributes make the header relationships programmatically determinable .

---

### Real-World Cases

1. **Government and public-sector sites**: WCAG 1.3.1 audits check for `<th>`, `scope`, and captions.
2. **E-commerce comparison tables**: Structured with `<thead>`, `<tbody>`, and `<tfoot>` for clarity.
3. **Financial reports**: Column groups and row groups for multi-level data.

---

### References

- W3C — Technique H51: Using table markup to present tabular information — https://www.w3.org/WAI/WCAG22/Techniques/html/H51.html
- W3C — Technique H39: Using caption elements — https://w3c.github.io/wcag/techniques/html/H39
- W3C — Technique H43: Using id and headers attributes — https://w3c.github.io/wcag/techniques/html/H43
- WHATWG — Tabular data — https://html.spec.whatwg.org/dev/tables.html
- MDN Web Docs — HTML table accessibility — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility

---

## Summary Table of Table Structure Elements

| Element | Role | Position in Content Model |
|---|---|---|
| `<caption>` | Table title and accessible name | First child of `<table>` . |
| `<colgroup>` | Column group container | After `<caption>`, before `<thead>` . |
| `<col>` | Individual column within a group | Child of `<colgroup>` . |
| `<thead>` | Header row group | After `<colgroup>`, before `<tbody>` . |
| `<tbody>` | Body row group | After `<thead>`, before `<tfoot>` . |
| `<tfoot>` | Footer row group | After `<tbody>` . |
| `<tr>` | Table row | Child of `<table>`, `<thead>`, `<tbody>`, or `<tfoot>` . |
| `<th>` | Header cell | Child of `<tr>` . |
| `<td>` | Data cell | Child of `<tr>` . |

---

## Final Notes

- **Order matters**: The content model is strict; `<caption>` → `<colgroup>` → `<thead>` → `<tbody>` → `<tfoot>` must be respected .
- **Headers are mandatory**: `<th>` with `scope` (or `id`/`headers` for complex tables) makes the table accessible .
- **Captions identify tables**: `<caption>` is the correct element for a table title and provides the accessible name .
- **Column groups are for styling**: `<colgroup>` and `<col>` do not create headers; they apply presentation to columns .
- **Layout tables are prohibited**: The specification explicitly states “Tables must not be used as layout aids” .
- **Validate and test**: Use HTML validators to catch content-model violations and accessibility checkers to verify header associations and captions.

---

This cheat sheet has covered table structure — `<caption>`, `<colgroup>`, `<col>`, `<thead>`, `<tbody>`, `<tfoot>`, `<tr>`, `<th>`, `<td>`, and best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate table markup against the WHATWG HTML Living Standard, WCAG 1.3.1 (Info and Relationships), and the W3C table techniques.