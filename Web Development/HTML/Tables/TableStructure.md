# HTML Table Structure: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML table structure refers to the semantic elements that divide a table into logical sections — a title, a header, a body, and a footer — enabling better organisation, styling, and accessibility of tabular data.

**Technical Definition**

The `<table>` element represents data with more than one dimension, in the form of a table. Its content model, in order, is: an optional `<caption>` element, zero or more `<colgroup>` elements, an optional `<thead>` element, either zero or more `<tbody>` elements or one or more `<tr>` elements, and an optional `<tfoot>` element. The `<caption>` element provides the table‘s accessible name or description and must be the first child of its parent `<table>`. The `<thead>` element defines a block of rows consisting of column labels, the `<tbody>` element encapsulates the main data rows, and the `<tfoot>` element defines rows summarising the columns.

**Beginner-Friendly Explanation**

A table is like a spreadsheet: it has a title, column headings, rows of data, and sometimes a summary row at the bottom. HTML lets you mark up each of these parts with its own tag: `<caption>` for the title, `<thead>` for the header rows, `<tbody>` for the main data, and `<tfoot>` for the summary. This makes your tables easier for everyone to read — and for screen readers to understand.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Semantic separation** | Each table section has a distinct semantic meaning and role |
| **Strict order** | `<caption>` first, then `<thead>`, then `<tbody>`, then `<tfoot>` |
| **Single instances** | Only one `<caption>`, one `<thead>`, and one `<tfoot>` are permitted per table |
| **Multiple bodies** | Zero or more `<tbody>` elements may be used to group related rows |
| **Accessibility-critical** | Screen readers use section elements to navigate and announce table structure |
| **Styling hooks** | Each section provides a CSS styling target for consistent visual design |
| **Printing behaviour** | `<thead>` and `<tfoot>` repeat on each printed page when the table spans multiple pages |

---

### Prerequisites

- Basic familiarity with HTML document structure
- Understanding of the `<table>`, `<tr>`, `<th>`, and `<td>` elements
- Awareness of block-level vs. inline-level elements
- Basic knowledge of accessibility principles

---

### Related Programming Areas

- **Semantic HTML** – Table sections are core semantic grouping elements
- **Web Accessibility (A11y)** – Proper table structure enables screen reader navigation
- **CSS** – Table sections provide styling hooks for borders, backgrounds, and typography
- **Print Media** – `<thead>` and `<tfoot>` repeat across printed pages
- **WCAG Compliance** – Tables must be properly structured to satisfy WCAG 1.3.1

---

## Core Concepts / Features

---

### 1. The `<caption>` Element

#### Definitions

**Core Definition**

The `<caption>` element specifies the title or description of a table and must be placed immediately after the opening `<table>` tag.

**Technical Definition**

The `<caption>` HTML element specifies the caption (or title) of a table, providing the table with an accessible name or accessible description. It must be the first child of its parent `<table>` element. The element has no content categories. Its permitted content is flow content. It accepts only global attributes. Its DOM interface is `HTMLTableCaptionElement`.

**Beginner-Friendly Explanation**

The `<caption>` tag is like a title for your table. It goes right after the opening `<table>` tag and tells readers what the table is about. Screen readers announce it as the table‘s name, so it‘s important for accessibility.

#### Purposes

- To provide a visible title or description for the table
- To give the table an accessible name for screen readers
- To describe the table‘s purpose before users navigate its contents
- To enable styling of the table title via CSS

#### Syntax Rules and Structure

**General Syntax**

```html
<table>
    <caption>Table Title</caption>
    <!-- table rows and cells -->
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<caption>` | Opening tag; indicates the table caption |
| `Content` | Flow content; the caption text |
| `</caption>` | Closing tag; required |

**Syntax Rules**

- The `<caption>` must be the first child of the `<table>` element
- Only one `<caption>` is permitted per table
- Both start and end tags are mandatory
- The element accepts only global attributes
- When a `<table>` is the only descendant of a `<figure>`, use `<figcaption>` instead of `<caption>`

**Constraints and Limitations**

- The `align` attribute is deprecated; use CSS `caption-side` instead
- A table can have at most one caption
- The caption is not a substitute for a table summary; the `summary` attribute is deprecated

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Table Caption**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Table Caption Demo</title>
</head>
<body>
    <table>
        <caption>Monthly Sales Summary</caption>
        <tr>
            <th scope="col">Month</th>
            <th scope="col">Sales</th>
        </tr>
        <tr>
            <td>January</td>
            <td>$12,000</td>
        </tr>
        <tr>
            <td>February</td>
            <td>$15,500</td>
        </tr>
    </table>
</body>
</html>
```

**Expected Output**

The caption “Monthly Sales Summary” appears above the table by default. Screen readers announce it as the table‘s name.

**Why This Output Occurs**

The `<caption>` element provides the table’s accessible name. Browsers render it above the table by default, and screen readers announce it before the table content.

---

**Example 2: Caption Styled with CSS**

```html
<style>
    caption {
        caption-side: bottom;
        font-weight: bold;
        font-size: 1.1em;
        padding: 0.5em;
        text-align: left;
    }
</style>

<table>
    <caption>Figure 1: Quarterly Revenue</caption>
    <tr>
        <th scope="col">Quarter</th>
        <th scope="col">Revenue</th>
    </tr>
    <tr>
        <td>Q1</td>
        <td>$1.2M</td>
    </tr>
</table>
```

**Expected Output**

The caption appears below the table, bold and left-aligned.

**Why This Output Occurs**

The CSS `caption-side: bottom` moves the caption below the table. The other styles control its appearance. The semantic role of the caption remains unchanged regardless of its visual position.

#### Real-World Cases

**Case 1: Financial Reports**

Financial tables use `<caption>` for titles like “Table 3: Annual Revenue by Region.”

**Case 2: Academic Papers**

Scientific papers use `<caption>` for numbered table titles like “Table 2: Experimental Results.”

**Case 3: Data Dashboards**

Dashboard tables use `<caption>` to describe the data source and time period.

---

### 2. The `<thead>` Element (Table Head)

#### Definitions

**Core Definition**

The `<thead>` element groups the header rows of a table, containing the column labels that identify what each column of data represents.

**Technical Definition**

The `<thead>` HTML element defines a set of rows defining the head of the columns of the table. It represents the block of rows that consist of the column labels (headings) for its parent `<table>` element. Its permitted content is zero or more `<tr>` elements. Its permitted parents are `<table>` elements, and it must appear after any `<caption>` and `<colgroup>` elements and before any `<tbody>`, `<tfoot>`, and `<tr>` elements. The end tag may be omitted if the `<thead>` element is immediately followed by a `<tbody>` or `<tfoot>` element. Its DOM interface is `HTMLTableSectionElement`.

**Beginner-Friendly Explanation**

The `<thead>` tag wraps the header row (or rows) of your table — the row that contains the column labels like “Name,” “Price,” or “Quantity.” It tells the browser and screen readers “this part is the header.” When a table is printed across multiple pages, the browser repeats the `<thead>` on each page.

#### Purposes

- To group header rows into a single semantic unit
- To enable screen readers to identify and announce column headers
- To repeat headers on each printed page when tables span multiple pages
- To provide a styling hook for header row styling

#### Syntax Rules and Structure

**General Syntax**

```html
<table>
    <caption>Caption</caption>
    <thead>
        <tr>
            <th scope="col">Header 1</th>
            <th scope="col">Header 2</th>
        </tr>
    </thead>
    <tbody>
        <!-- data rows -->
    </tbody>
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<thead>` | Opening tag; indicates the header section |
| `<tr>` | One or more header rows |
| `</thead>` | Closing tag; may be omitted if followed by `<tbody>` or `<tfoot>` |

**Syntax Rules**

- The `<thead>` must appear after any `<caption>` and `<colgroup>` elements
- It must appear before any `<tbody>`, `<tfoot>`, or `<tr>` elements
- Only one `<thead>` element is permitted per table
- It must contain at least one `<tr>` element
- The end tag may be omitted if immediately followed by `<tbody>` or `<tfoot>`

**Constraints and Limitations**

- The `align`, `char`, and `valign` attributes are deprecated; use CSS instead
- A `<thead>` without rows is invalid
- Screen readers use `<thead>` to identify column headers, so proper `scope` attributes on `<th>` elements within it are essential

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Thead with Column Headers**

```html
<table>
    <caption>Student Grades</caption>
    <thead>
        <tr>
            <th scope="col">Student</th>
            <th scope="col">Math</th>
            <th scope="col">Science</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alice</td>
            <td>95</td>
            <td>88</td>
        </tr>
        <tr>
            <td>Bob</td>
            <td>87</td>
            <td>92</td>
        </tr>
    </tbody>
</table>
```

**Expected Output**

A table with a header row containing “Student,” “Math,” and “Science,” followed by two data rows.

**Why This Output Occurs**

The `<thead>` groups the header row. Screen readers announce the column headers when navigating to data cells. The `<tbody>` groups the data rows.

---

**Example 2: Thead with Multiple Header Rows**

```html
<table>
    <thead>
        <tr>
            <th scope="col" rowspan="2">Name</th>
            <th scope="colgroup" colspan="2">Scores</th>
        </tr>
        <tr>
            <th scope="col">Math</th>
            <th scope="col">Science</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alice</td>
            <td>95</td>
            <td>88</td>
        </tr>
    </tbody>
</table>
```

**Expected Output**

A two-level header: the first row has “Name” spanning two rows and “Scores” spanning two columns. The second row has “Math” and “Science.”

**Why This Output Occurs**

The `<thead>` can contain multiple `<tr>` elements. The `rowspan` and `colspan` attributes create the multi-level header structure. Screen readers announce the header hierarchy.

#### Real-World Cases

**Case 1: Financial Statements**

Financial statements use `<thead>` for column headers like “2024,” “2025,” and “Change.”

**Case 2: Sports League Tables**

League tables use `<thead>` for column headers like “Team,” “Played,” “Won,” “Drawn,” “Lost,” and “Points.”

**Case 3: Comparison Tables**

Product comparison pages use `<thead>` for product names as column headers.

---

### 3. The `<tbody>` Element (Table Body)

#### Definitions

**Core Definition**

The `<tbody>` element encapsulates a set of table rows that comprise the main body of the table‘s data.

**Technical Definition**

The `<tbody>` HTML element encapsulates a set of table rows (`<tr>` elements), indicating that they comprise the body of a table’s (main) data. It represents a block of rows that consist of a body of data for its parent `<table>` element. Its permitted content is zero or more `<tr>` elements. Its permitted parents are `<table>` elements, and it must appear after any `<caption>`, `<colgroup>`, and `<thead>` elements and before any `<tfoot>` element. A `<tbody>` element‘s start tag may be omitted if the first thing inside it is a `<tr>` element and the element is not preceded by another `<tbody>`, `<thead>`, or `<tfoot>` element. Its DOM interface is `HTMLTableSectionElement`.

**Beginner-Friendly Explanation**

The `<tbody>` tag wraps the main data rows of your table — all the rows that contain the actual information, like student names and their grades. It tells the browser “this is the body of the table.” You can have more than one `<tbody>` if you want to group related rows together.

#### Purposes

- To group the main data rows into a semantic unit
- To enable independent scrolling of the table body in some contexts
- To allow multiple `<tbody>` elements for grouping related data
- To provide a styling hook for data row styling

#### Syntax Rules and Structure

**General Syntax**

```html
<table>
    <thead>
        <tr><th>Header</th></tr>
    </thead>
    <tbody>
        <tr><td>Data 1</td></tr>
        <tr><td>Data 2</td></tr>
    </tbody>
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<tbody>` | Opening tag; indicates a body section |
| `<tr>` | One or more data rows |
| `</tbody>` | Closing tag; may be omitted in specific circumstances |

**Syntax Rules**

- The `<tbody>` must appear after any `<caption>`, `<colgroup>`, and `<thead>` elements
- It must appear before any `<tfoot>` element
- Zero or more `<tbody>` elements are permitted per table
- If no `<tbody>` is used, `<tr>` elements are direct children of `<table>`
- The end tag may be omitted if immediately followed by another `<tbody>` or `<tfoot>`

**Constraints and Limitations**

- The `align`, `char`, and `valign` attributes are deprecated; use CSS instead
- An empty `<tbody>` is technically valid but serves no purpose
- Multiple `<tbody>` elements can be used to group rows, but each must have at least one row

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Tbody with Data Rows**

```html
<table>
    <caption>Employee Directory</caption>
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Department</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Jane Doe</td>
            <td>Engineering</td>
        </tr>
        <tr>
            <td>John Smith</td>
            <td>Marketing</td>
        </tr>
    </tbody>
</table>
```

**Expected Output**

A table with a header row and two data rows grouped in a `<tbody>`.

**Why This Output Occurs**

The `<tbody>` groups the data rows. Screen readers announce the number of rows in the body and allow navigation between them.

---

**Example 2: Multiple Tbody Elements for Grouping**

```html
<table>
    <thead>
        <tr>
            <th scope="col">Product</th>
            <th scope="col">Price</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>Laptop</td><td>$999</td></tr>
        <tr><td>Mouse</td><td>$29</td></tr>
    </tbody>
    <tbody>
        <tr><td>Keyboard</td><td>$79</td></tr>
        <tr><td>Monitor</td><td>$299</td></tr>
    </tbody>
</table>
```

**Expected Output**

Two groups of data rows under the same header. Screen readers may announce the presence of two table body groups.

**Why This Output Occurs**

Multiple `<tbody>` elements group related rows. This can be useful for styling or for separating different categories of data within a single table.

#### Real-World Cases

**Case 1: Invoice Line Items**

Invoices use `<tbody>` to group all line items, with `<tfoot>` for the total.

**Case 2: Multi-Category Data Tables**

Tables that display data across multiple categories use multiple `<tbody>` elements to separate the groups.

**Case 3: Timetables**

Weekly timetables use `<tbody>` for each day‘s schedule rows.

---

### 4. The `<tfoot>` Element (Table Footer)

#### Definitions

**Core Definition**

The `<tfoot>` element groups the footer rows of a table, typically containing summary information such as totals, averages, or footnotes.

**Technical Definition**

The `<tfoot>` HTML element defines a set of rows summarizing the columns of the table. It represents the block of rows that consist of the column summaries (footers) for its parent `<table>` element. Its permitted content is zero or more `<tr>` elements. Its permitted parents are `<table>` elements, and it must appear after any `<caption>`, `<colgroup>`, `<thead>`, `<tbody>`, and `<tr>` elements. The `tfoot` element‘s end tag may be omitted if the element is immediately followed by a `<tbody>` element. Its DOM interface is `HTMLTableSectionElement`.

**Beginner-Friendly Explanation**

The `<tfoot>` tag wraps the footer row of your table — the row at the bottom that usually contains totals or summaries, like “Total: $1,500.” It tells the browser and screen readers “this row is a summary of the data above.” When a table is printed across multiple pages, the `<tfoot>` repeats on each page.

#### Purposes

- To group summary or footer rows into a semantic unit
- To provide totals, averages, or other aggregate information
- To repeat the footer on each printed page when tables span multiple pages
- To provide a styling hook for footer row styling

#### Syntax Rules and Structure

**General Syntax**

```html
<table>
    <thead>
        <tr><th>Item</th><th>Cost</th></tr>
    </thead>
    <tbody>
        <tr><td>Item 1</td><td>$100</td></tr>
    </tbody>
    <tfoot>
        <tr><td>Total</td><td>$100</td></tr>
    </tfoot>
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<tfoot>` | Opening tag; indicates the footer section |
| `<tr>` | One or more footer rows |
| `</tfoot>` | Closing tag; may be omitted if followed by `<tbody>` |

**Syntax Rules**

- The `<tfoot>` must appear after any `<caption>`, `<colgroup>`, `<thead>`, `<tbody>`, and `<tr>` elements
- Only one `<tfoot>` element is permitted per table
- It must contain at least one `<tr>` element
- The end tag may be omitted if immediately followed by a `<tbody>` element

**Constraints and Limitations**

- The `align`, `char`, and `valign` attributes are deprecated; use CSS instead
- In HTML5, the `<tfoot>` element must appear after the `<tbody>` elements (not before them, as was required in HTML4)
- An empty `<tfoot>` is technically valid but serves no purpose

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Tfoot with Totals**

```html
<table>
    <caption>Monthly Expenses</caption>
    <thead>
        <tr>
            <th scope="col">Category</th>
            <th scope="col">Amount</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>Rent</td><td>$1,200</td></tr>
        <tr><td>Utilities</td><td>$300</td></tr>
        <tr><td>Groceries</td><td>$500</td></tr>
    </tbody>
    <tfoot>
        <tr>
            <th scope="row">Total</th>
            <td>$2,000</td>
        </tr>
    </tfoot>
</table>
```

**Expected Output**

A table with a header row, three data rows, and a footer row showing the total.

**Why This Output Occurs**

The `<tfoot>` groups the summary row. Screen readers announce the footer separately and can identify it as containing summary information.

---

**Example 2: Tfoot with Colspan for a Single Summary Cell**

```html
<table>
    <thead>
        <tr>
            <th scope="col">Product</th>
            <th scope="col">Q1</th>
            <th scope="col">Q2</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>Product A</td><td>$10,000</td><td>$12,000</td></tr>
        <tr><td>Product B</td><td>$8,000</td><td>$9,500</td></tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">Total revenue: $39,500</td>
        </tr>
    </tfoot>
</table>
```

**Expected Output**

A footer row with a single cell spanning all three columns.

**Why This Output Occurs**

The `colspan="3"` attribute makes the footer cell span the entire table width, which is common for summary statements.

#### Real-World Cases

**Case 1: Financial Spreadsheets**

Spreadsheets use `<tfoot>` for totals, subtotals, and grand totals.

**Case 2: Grade Books**

Grade books use `<tfoot>` for class averages and highest/lowest scores.

**Case 3: Inventory Reports**

Inventory reports use `<tfoot>` for total quantities and values.

---

### 5. Table Sections

#### Definitions

**Core Definition**

Table sections are the semantic divisions of a table — caption, header, body, and footer — that allow for structured data presentation, independent scrolling, and consistent rendering across different media.

**Technical Definition**

Table sections refer to the semantic separation of tabular data into logical groups using `<caption>`, `<thead>`, `<tbody>`, and `<tfoot>`. This separation enables user agents to apply different rendering behaviours: the `<thead>` and `<tfoot>` elements are repeated on each page when printing a table that spans multiple pages; the `<tbody>` may be scrolled independently of the header and footer in certain contexts; and each section provides a distinct styling target for CSS. The WHATWG specification defines the content model of `<table>` as an ordered sequence of these sections, with strict rules governing their placement and repetition.

**Beginner-Friendly Explanation**

Think of a table like a book: the caption is the title, the header is the table of contents, the body is the chapters, and the footer is the index. Splitting a table into these sections lets the browser treat each part differently. For example, when you print a long table, the header and footer are repeated on every page so you always know what the columns mean.

#### Purposes

- To provide a clear semantic structure for tabular data
- To enable independent scrolling of the table body while keeping headers visible
- To repeat headers and footers across printed pages
- To offer distinct CSS styling targets for each table section
- To improve screen reader navigation by announcing section boundaries

#### Syntax Rules and Structure

**Full Table Structure**

```html
<table>
    <caption>Table Title</caption>
    <colgroup>
        <col span="3">
    </colgroup>
    <thead>
        <tr>
            <th scope="col">Column 1</th>
            <th scope="col">Column 2</th>
            <th scope="col">Column 3</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>Data 1</td><td>Data 2</td><td>Data 3</td></tr>
        <tr><td>Data 4</td><td>Data 5</td><td>Data 6</td></tr>
    </tbody>
    <tfoot>
        <tr><td colspan="3">Summary row</td></tr>
    </tfoot>
</table>
```

**Component Breakdown**

| Element | Position | Purpose |
|---|---|---|
| `<caption>` | First child of `<table>` | Title or description |
| `<colgroup>` | After caption | Column grouping |
| `<thead>` | After colgroup | Header rows |
| `<tbody>` | After thead | Data rows |
| `<tfoot>` | After tbody | Summary rows |

**Syntax Rules**

- The order is strict: caption → colgroup → thead → tbody → tfoot
- Only one `<caption>`, `<thead>`, and `<tfoot>` per table
- Zero or more `<tbody>` elements are permitted
- Each section must contain at least one `<tr>` element (except `<colgroup>`)

**Constraints and Limitations**

- The strict ordering means you cannot place `<tfoot>` before `<tbody>` in HTML5 (unlike HTML4)
- Not all browsers support independent scrolling of `<tbody>` out of the box; CSS may be required
- Screen reader support for table sections varies; testing with assistive technology is recommended

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Complete Table with All Sections**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Complete Table Structure</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        caption { font-weight: bold; font-size: 1.2em; padding: 0.5em; }
        thead { background-color: #e0e0e0; }
        tfoot { background-color: #f0f0f0; font-weight: bold; }
        th, td { border: 1px solid #ccc; padding: 0.5em; text-align: left; }
        tbody tr:nth-child(even) { background-color: #f9f9f9; }
    </style>
</head>
<body>
    <table>
        <caption>Quarterly Sales Report</caption>
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
                <th scope="row">Widget A</th>
                <td>$10,000</td>
                <td>$12,000</td>
                <td>$11,500</td>
                <td>$14,000</td>
            </tr>
            <tr>
                <th scope="row">Widget B</th>
                <td>$8,000</td>
                <td>$9,500</td>
                <td>$10,200</td>
                <td>$11,800</td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <th scope="row">Total</th>
                <td>$18,000</td>
                <td>$21,500</td>
                <td>$21,700</td>
                <td>$25,800</td>
            </tr>
        </tfoot>
    </table>
</body>
</html>
```

**Expected Output**

A fully structured table with a caption, a header row, two data rows, and a footer row of totals. The header and footer have distinct background colours.

**Why This Output Occurs**

Each section element provides a styling hook. The `<caption>` provides the title. The `<thead>` groups the header row with its background colour. The `<tbody>` contains the data rows with alternating row colours via `nth-child(even)`. The `<tfoot>` contains the totals with its own background colour.

---

**Example 2: Independent Scrolling of Tbody**

```html
<style>
    .scrollable-table {
        display: block;
        max-height: 200px;
        overflow-y: auto;
    }
    .scrollable-table thead {
        position: sticky;
        top: 0;
        background: white;
        z-index: 1;
    }
</style>

<table class="scrollable-table">
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Score</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>Alice</td><td>95</td></tr>
        <tr><td>Bob</td><td>87</td></tr>
        <tr><td>Charlie</td><td>92</td></tr>
        <tr><td>Diana</td><td>88</td></tr>
        <tr><td>Eve</td><td>91</td></tr>
        <tr><td>Frank</td><td>85</td></tr>
        <tr><td>Grace</td><td>93</td></tr>
    </tbody>
</table>
```

**Expected Output**

A table with a fixed height where the body scrolls independently and the header remains visible at the top.

**Why This Output Occurs**

The CSS `max-height` and `overflow-y: auto` on the table container enable scrolling. The `position: sticky` on the `<thead>` keeps the header visible while the body scrolls. This demonstrates one of the benefits of separating the header from the body.

#### Real-World Cases

**Case 1: Financial Dashboards**

Dashboards use scrollable tables with sticky headers to display large datasets without losing column context.

**Case 2: Printed Reports**

Financial reports use `<thead>` and `<tfoot>` to repeat headers and totals on every printed page.

**Case 3: Data Grids**

Web-based data grids use `<thead>` for column headers and `<tbody>` for data rows, enabling features like sorting, filtering, and pagination.

---

### 6. Choosing Semantic Elements Rather Than Purely Visual Elements

#### Definitions

**Core Definition**

Choosing semantic table structure means using `<caption>`, `<thead>`, `<tbody>`, and `<tfoot>` for their intended semantic purposes, rather than using `<div>` or `<span>` elements to visually simulate a table structure.

**Technical Definition**

Semantic table structure requires using the native HTML table elements for tabular data. WCAG Success Criterion 1.3.1 (Info and Relationships) requires that information, structure, and relationships conveyed through presentation can be programmatically determined. Using `<div>` elements with CSS to create a table-like appearance fails this criterion because there is no semantic row-column relationship. The W3C‘s Techniques for Accessibility Guidelines explicitly recommend using `<thead>`, `<tfoot>`, and `<tbody>` to group rows and identify structural divisions.

**Beginner-Friendly Explanation**

Don’t use `<div>` tags to fake a table. Use the real `<table>`, `<thead>`, `<tbody>`, and `<tfoot>` tags. That way, screen readers can announce the table structure correctly, and the browser knows how to handle printing and scrolling. CSS can style the table however you like — the HTML provides the meaning.

#### Correct vs. Incorrect Patterns

| If you want to… | Use… | Not… |
|---|---|---|
| Create a data table | `<table><thead><tbody><tfoot>` | `<div>` with CSS Grid |
| Group header rows | `<thead>` | `<div class="header">` |
| Group data rows | `<tbody>` | `<div class="body">` |
| Group summary rows | `<tfoot>` | `<div class="footer">` |
| Provide a table title | `<caption>` | `<h2>` above the table |

**Syntax Rules**

- Use `<table>` only for tabular data
- Use `<thead>`, `<tbody>`, and `<tfoot>` to group rows semantically
- Use `<caption>` for the table title
- Use CSS for all visual styling
- Never use tables or table elements for layout purposes

**Constraints and Limitations**

- Screen readers may not correctly interpret table structure if semantic elements are not used
- Tables used for layout violate accessibility guidelines
- The deprecated `align`, `bgcolor`, and `border` attributes must not be used

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct vs. Incorrect Table Structure**

```html
<!-- CORRECT: Semantic table structure -->
<table>
    <caption>Employee Salaries</caption>
    <thead>
        <tr><th scope="col">Name</th><th scope="col">Salary</th></tr>
    </thead>
    <tbody>
        <tr><td>Jane</td><td>$75,000</td></tr>
        <tr><td>John</td><td>$82,000</td></tr>
    </tbody>
    <tfoot>
        <tr><td>Average</td><td>$78,500</td></tr>
    </tfoot>
</table>

<!-- INCORRECT: Div-based fake table -->
<div class="table">
    <div class="caption">Employee Salaries</div>
    <div class="header">
        <div class="cell">Name</div>
        <div class="cell">Salary</div>
    </div>
    <div class="body">
        <div class="cell">Jane</div>
        <div class="cell">$75,000</div>
    </div>
</div>
```

**Expected Output**

Both may look similar visually, but only the first is semantically correct. Screen readers announce the first as a table with headers and data. The second is just a group of `<div>` elements with no table semantics.

**Why This Output Occurs**

The `<table>`, `<thead>`, `<tbody>`, and `<tfoot>` elements carry semantic meaning that assistive technology can interpret. The `<div>` elements carry no table semantics, so screen readers cannot identify the content as a table.

#### Real-World Cases

**Case 1: Government Accessibility Compliance**

Government websites must use semantic table structure to comply with WCAG 2.1 Success Criterion 1.3.1.

**Case 2: Screen Reader Navigation**

Screen readers use `<thead>` and `<tbody>` to announce table structure and allow navigation between header and data cells.

**Case 3: Print Stylesheets**

Print stylesheets rely on `<thead>` and `<tfoot>` to repeat headers and footers on each printed page.

---

## References

- MDN Web Docs – `<table>`: The Table element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/table
- MDN Web Docs – `<caption>`: The Table Caption element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/caption
- MDN Web Docs – `<thead>`: The Table Head element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/thead
- MDN Web Docs – `<tbody>`: The Table Body element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tbody
- MDN Web Docs – `<tfoot>`: The Table Foot element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tfoot
- WHATWG HTML Living Standard – Tabular data – https://html.spec.whatwg.org/multipage/tables.html
- WHATWG HTML Living Standard – The table element – https://html.spec.whatwg.org/multipage/tables.html#the-table-element
- WHATWG HTML Living Standard – The caption element – https://html.spec.whatwg.org/multipage/tables.html#the-caption-element
- WHATWG HTML Living Standard – The thead element – https://html.spec.whatwg.org/multipage/tables.html#the-thead-element
- WHATWG HTML Living Standard – The tbody element – https://html.spec.whatwg.org/multipage/tables.html#the-tbody-element
- WHATWG HTML Living Standard – The tfoot element – https://html.spec.whatwg.org/multipage/tables.html#the-tfoot-element
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – H51: Using table markup to present tabular information – https://www.w3.org/WAI/WCAG21/Techniques/html/H51
- W3C – H63: Using the scope attribute to associate header cells and data cells in data tables – https://www.w3.org/WAI/WCAG21/Techniques/html/H63
- MDN Web Docs – Styling tables – https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Tables
- LogRocket – Creating responsive data tables with CSS – https://blog.logrocket.com/creating-responsive-data-tables-css/