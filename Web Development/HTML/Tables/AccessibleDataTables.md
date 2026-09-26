# Accessible Data Tables: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Accessible data tables are HTML tables whose structure, headers, captions, and relationships are marked up so that screen readers and other assistive technologies can programmatically determine the meaning and organisation of the tabular data.

**Technical Definition**

Accessible data tables conform to WCAG 2.1 Success Criterion 1.3.1 (Info and Relationships, Level A), which requires that “information, structure, and relationships conveyed through presentation can be programmatically determined or are available in text.” This is achieved through proper use of the `<table>`, `<caption>`, `<thead>`, `<tbody>`, `<tfoot>`, `<tr>`, `<th>`, and `<td>` elements, combined with the `scope` attribute on header cells, or the `id` and `headers` attributes for complex tables. The HTML Accessibility API Mappings (HTML-AAM) define how these elements map to platform accessibility APIs, enabling screen readers to announce header-data relationships correctly.

**Beginner-Friendly Explanation**

A table on a webpage is like a spreadsheet: it has rows and columns, and you find information by matching a row with a column. But if you can‘t see the table, how do you know which header belongs to which data cell? Accessible data tables solve this problem. They use special HTML tags and attributes to tell screen readers “this header goes with this data.” That way, when someone using a screen reader navigates to a cell containing “95,” the screen reader can announce “Math, Alice, 95” — giving the user the full context they need.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Programmatic relationships** | Header-data associations must be determinable by assistive technology, not just visually apparent |
| **`scope` for simple tables** | The `scope` attribute on `<th>` elements defines row and column relationships |
| **`id`/`headers` for complex tables** | Explicit programmatic association for tables with multiple header levels |
| **Caption provides context** | The `<caption>` element gives the table an accessible name |
| **No layout tables** | Tables must be used only for tabular data, never for page layout |
| **WCAG 1.3.1 Level A** | This is a minimum conformance requirement for all websites |
| **Screen reader navigation** | Accessible tables enable screen readers to announce headers before data cells |

---

### Prerequisites

- Basic familiarity with HTML table elements (`<table>`, `<tr>`, `<th>`, `<td>`)
- Understanding of the `<thead>`, `<tbody>`, and `<tfoot>` elements
- Awareness of how screen readers interpret tables
- Basic knowledge of WCAG guidelines

---

### Related Programming Areas

- **Web Accessibility (A11y)** – Table accessibility is a core component of accessible web design
- **Semantic HTML** – Proper table markup is a semantic requirement
- **WCAG Compliance** – Success Criterion 1.3.1 requires programmatically determinable relationships
- **Screen Reader Navigation** – Table headers determine how screen readers announce data cells
- **CSS Layout** – CSS Grid and Flexbox replace tables for page layout
- **ARIA** – `aria-describedby`, `aria-label`, and ARIA table roles supplement native HTML semantics

---

## Core Concepts / Features

---

### 1. Proper Header Association

#### Definitions

**Core Definition**

Proper header association is the practice of programmatically linking header cells (`<th>`) to the data cells (`<td>`) they describe, using the `scope` attribute for simple tables and the `id` and `headers` attributes for complex tables.

**Technical Definition**

Header association in HTML tables is established through two mechanisms. For simple tables — those with at most one row and/or one column of headers — the `scope` attribute on `<th>` elements explicitly defines whether the header applies to a row (`scope="row"`), a column (`scope="col"`), a group of rows (`scope="rowgroup"`), or a group of columns (`scope="colgroup"`). For complex tables — those with more than one level of headers — each `<th>` element is given a unique `id`, and each `<td>` (or `<th>`) element references the relevant header IDs using the `headers` attribute, which contains a space-separated list of ID values. This creates a complete programmatic association that screen readers can interpret. Technique H63 of the W3C Techniques for WCAG 2.1 documents the `scope` approach, while Technique H43 documents the `id`/`headers` approach.

**Beginner-Friendly Explanation**

A header cell is like a label on a spreadsheet column. “Math” is the header for a column of math scores. Proper header association means telling the browser and screen readers “this header belongs to this data.” For simple tables, you use `scope` — like saying “this header is for the column below it.” For complex tables with multiple levels of headers, you give each header a name (`id`) and then on each data cell you list which headers apply to it (`headers`). Screen readers use this information to announce, for example, “Math, Alice, 95” so users know exactly what the data means.

#### Purposes

- To enable screen readers to announce the correct header context for each data cell
- To satisfy WCAG Success Criterion 1.3.1 (Info and Relationships)
- To provide programmatically determinable relationships between headers and data
- To support efficient table navigation for assistive technology users
- To ensure that data tables are understandable regardless of visual presentation

#### Syntax Rules and Structure

**General Syntax for Simple Tables (Scope)**

```html
<table>
    <thead>
        <tr>
            <th scope="col">Column Header 1</th>
            <th scope="col">Column Header 2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">Row Header</th>
            <td>Data</td>
            <td>Data</td>
        </tr>
    </tbody>
</table>
```

**General Syntax for Complex Tables (ID and Headers)**

```html
<table>
    <thead>
        <tr>
            <th rowspan="2" id="name">Name</th>
            <th colspan="2" id="height">Height</th>
        </tr>
        <tr>
            <th id="feet" headers="height">Feet</th>
            <th id="inches" headers="height">Inches</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th id="walter" headers="name">Walter</th>
            <td headers="height feet walter">6</td>
            <td headers="height inches walter">4</td>
        </tr>
    </tbody>
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `scope="col"` | Header applies to the column below it |
| `scope="row"` | Header applies to the row beside it |
| `scope="colgroup"` | Header applies to a group of columns |
| `scope="rowgroup"` | Header applies to a group of rows |
| `id` | Unique identifier on a `<th>` element |
| `headers` | Space-separated list of header IDs on a `<td>` or `<th>` element |

**Syntax Rules**

- The `scope` attribute is only valid on `<th>` elements
- The value must be one of: `row`, `col`, `rowgroup`, or `colgroup`
- For `id`/`headers`, each `id` must be unique within the document
- The `headers` attribute can reference multiple header IDs, separated by spaces
- Use `scope` whenever possible; use `id`/`headers` only when `scope` cannot express the relationship

**Constraints and Limitations**

- The `scope` attribute is not valid on `<td>` elements
- For tables with non-adjacent headers, `id` and `headers` are required
- WCAG Technique F90 describes the failure condition when `headers` and `id` are incorrectly associated
- Some older screen readers may not fully support `scope="rowgroup"` or `scope="colgroup"`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Simple Table with Scope**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple Table with Scope</title>
</head>
<body>
    <table>
        <caption>User's Height and Age</caption>
        <thead>
            <tr>
                <th scope="col">Name</th>
                <th scope="col">Height</th>
                <th scope="col">Age</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th scope="row">Walter</th>
                <td>6'4"</td>
                <td>34</td>
            </tr>
            <tr>
                <th scope="row">Steve</th>
                <td>5'4"</td>
                <td>30</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

A table with column headers (“Name,” “Height,” “Age”) and row headers (“Walter,” “Steve”). When a screen reader user navigates to the cell containing “6‘4“,” the screen reader announces “Height, Walter, 6’4”.”

**Why This Output Occurs**

The `scope="col"` attribute on the first-row headers tells screen readers that these headers apply to the columns below them. The `scope="row"` attribute on the first-column headers tells screen readers that these headers apply to the rows beside them. The screen reader combines both headers to provide full context for each data cell.

---

**Example 2: Complex Table with ID and Headers**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Complex Table with ID and Headers</title>
</head>
<body>
    <table>
        <caption>User's Height and Weight</caption>
        <thead>
            <tr>
                <th rowspan="2" id="name" scope="col">Name</th>
                <th colspan="2" id="height" scope="col">Height</th>
                <th rowspan="2" id="age" scope="col">Age</th>
            </tr>
            <tr>
                <th id="feet" headers="height" scope="col">Feet</th>
                <th id="inches" headers="height" scope="col">Inches</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th headers="name" scope="row" id="walter">Walter</th>
                <td headers="height feet walter">6</td>
                <td headers="height inches walter">4</td>
                <td headers="age walter">34</td>
            </tr>
            <tr>
                <th headers="name" scope="row" id="steve">Steve</th>
                <td headers="height feet steve">5</td>
                <td headers="height inches steve">4</td>
                <td headers="age steve">30</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

A complex table with a two-level header. The “Height” header spans two columns, with “Feet” and “Inches” as sub-headers. Each data cell is explicitly associated with its headers.

**Why This Output Occurs**

The `id` attributes on the header cells provide unique identifiers. The `headers` attributes on the data cells list all applicable header IDs. When a screen reader user navigates to the cell containing “6,” it announces “Height, Feet, Walter, 6” — combining the group header, the specific column header, and the row header. This is the pattern recommended by the 18F Accessibility Guide.

---

#### Real-World Cases

**Case 1: Financial Reports**

Financial reports use tables with row headers for line items (Revenue, Expenses, Net Income) and column headers for quarters or years. The `scope` attribute ensures that screen readers announce “Revenue, Q1, $1.2M.”

**Case 2: Sports League Tables**

Sports standings tables use `scope="col"` for statistical categories (Played, Won, Lost, Points) and `scope="row"` for team names.

**Case 3: Product Comparison Tables**

E-commerce comparison tables use `scope="col"` for product names and `scope="row"` for feature names, enabling screen reader users to understand which product has which feature.

---

### 2. Table Captions and Summaries

#### Definitions

**Core Definition**

A table caption is a visible title or description of a table, provided using the `<caption>` element; a table summary is an extended description of the table‘s structure or purpose, provided using `aria-describedby` (since the `summary` attribute is obsolete).

**Technical Definition**

The `<caption>` element provides the table with an accessible name. It must be the first child of the `<table>` element. WCAG Technique H39 states that “the `caption` for a table is a table identifier and acts like a title or heading for the table.” The `caption` element ensures that the table identifier remains associated with the table, both visually and programmatically. The `summary` attribute, formerly used to provide extended descriptions, is **obsolete in the HTML Living Standard**. The modern replacement is to provide a description element (a paragraph or `<div>`) and reference it from the `<table>` using the `aria-describedby` attribute. The accessible description text should be short (one or two sentences, no lists or links), because screen readers announce it as a single string.

**Beginner-Friendly Explanation**

Think of a caption like the title of a table — “Quarterly Sales by Region.” It goes right after the opening `<table>` tag and tells everyone what the table is about. Sometimes a table needs more explanation — like “This table shows sales figures broken down by region and quarter. Each cell represents the sales in thousands of dollars.” You put that explanation in a paragraph and use `aria-describedby` to point the table to it. Screen readers will announce both the caption and the description, giving users full context.

#### Purposes

- To provide a visible, programmatically associated title for the table
- To give the table an accessible name for screen readers
- To describe the table’s structure or purpose for users who need additional context
- To satisfy WCAG Success Criterion 1.3.1
- To help all users understand the table‘s content before navigating it

#### Syntax Rules and Structure

**General Syntax for Caption**

```html
<table>
    <caption>Table Title</caption>
    <!-- table rows and cells -->
</table>
```

**General Syntax for Summary with aria-describedby**

```html
<p id="table-description">This table shows quarterly sales figures in thousands of dollars.</p>
<table aria-describedby="table-description">
    <caption>Quarterly Sales</caption>
    <!-- table rows and cells -->
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<caption>` | First child of `<table>`; provides the accessible name |
| `aria-describedby` | References the `id` of a description element |
| `id` on description | Unique identifier for the summary element |

**Syntax Rules**

- The `<caption>` must be the first child of the `<table>` element
- Only one `<caption>` is permitted per table
- The `summary` attribute is obsolete and must not be used
- The `aria-describedby` attribute references the `id` of a description element
- The description text should be short and simple (no lists, links, or interactive elements)

**Constraints and Limitations**

- The `summary` attribute is obsolete in HTML5; assistive technologies may or may not continue to support it
- The description text is announced as a single string, so it must be concise
- If the description is longer than one or two sentences, provide a short description that points to a longer description elsewhere

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Table with Caption Only**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Table with Caption</title>
</head>
<body>
    <table>
        <caption>Quarterly Sales by Region</caption>
        <thead>
            <tr>
                <th scope="col">Region</th>
                <th scope="col">Q1</th>
                <th scope="col">Q2</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th scope="row">North</th>
                <td>$45,000</td>
                <td>$52,000</td>
            </tr>
            <tr>
                <th scope="row">South</th>
                <td>$38,000</td>
                <td>$41,000</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

The caption “Quarterly Sales by Region” appears above the table. Screen readers announce it as the table‘s name.

**Why This Output Occurs**

The `<caption>` element provides the table’s accessible name. Screen readers announce it before reading the table content, giving users context about what the table contains.

---

**Example 2: Table with Caption and aria-describedby**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Table with Caption and Description</title>
</head>
<body>
    <p id="sales-description">
        This table shows quarterly sales figures for each region.
        All values are in thousands of dollars.
    </p>

    <table aria-describedby="sales-description">
        <caption>Quarterly Sales by Region</caption>
        <thead>
            <tr>
                <th scope="col">Region</th>
                <th scope="col">Q1</th>
                <th scope="col">Q2</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th scope="row">North</th>
                <td>$45,000</td>
                <td>$52,000</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

The caption appears above the table, and the description paragraph appears above the table. Screen readers announce the caption as the table‘s name and the description as additional context.

**Why This Output Occurs**

The `aria-describedby` attribute creates a programmatic association between the table and the description paragraph. Screen readers announce both the accessible name (from `<caption>`) and the accessible description (from `aria-describedby`).

---

#### Real-World Cases

**Case 1: Financial Statements**

Financial reports use captions like “Consolidated Balance Sheet” and descriptions that explain accounting conventions.

**Case 2: Scientific Data Tables**

Scientific papers use captions like “Table 1: Demographic Characteristics of Study Participants” with descriptions explaining abbreviations and statistical methods.

**Case 3: Government Data Tables**

Government statistics tables use captions and descriptions to explain data sources, collection methods, and units of measurement.

---

### 3. Avoiding Tables for Layout

#### Definitions

**Core Definition**

Avoiding tables for layout means using CSS Grid or Flexbox for page layout instead of HTML tables, reserving `<table>` elements exclusively for tabular data.

**Technical Definition**

Layout tables use table markup for visual positioning rather than data relationships. This practice violates the semantic meaning of the `<table>` element and confuses assistive technologies. The WHATWG HTML Living Standard defines the `<table>` element as representing “tabular data — that is, information presented in a two-dimensional table comprised of rows and columns of cells containing data.” Using tables for layout fails WCAG Success Criterion 1.3.1 because there is no logical row-column relationship in the content. If a layout table is absolutely unavoidable, it must be marked with `role="presentation"` to hide its table semantics from assistive technology. The modern approach is to use CSS Grid or Flexbox for all page layout.

**Beginner-Friendly Explanation**

A table is for data, not for arranging things on a page. If you want a sidebar and a main content area, don‘t use a table — use CSS Grid or Flexbox. Tables used for layout confuse screen readers because they announce “table, 3 columns, 2 rows” when there’s no actual data relationship. CSS Grid and Flexbox are designed for layout, are responsive, and don‘t confuse assistive technology.

#### Purposes

- To preserve the semantic meaning of the `<table>` element for actual tabular data
- To prevent screen readers from announcing non-existent table structures
- To enable responsive layouts that adapt to different screen sizes
- To comply with WCAG Success Criterion 1.3.1
- To separate content (HTML) from presentation (CSS)

#### Syntax Rules and Structure

**Correct: CSS Grid for Layout**

```html
<div class="page-layout">
    <header>Header content</header>
    <aside>Sidebar content</aside>
    <main>Main content</main>
    <footer>Footer content</footer>
</div>

<style>
    .page-layout {
        display: grid;
        grid-template-columns: 200px 1fr;
        grid-template-areas:
            "header header"
            "sidebar main"
            "footer footer";
    }
</style>
```

**Incorrect: Table for Layout**

```html
<table>
    <tr>
        <td colspan="2">Header content</td>
    </tr>
    <tr>
        <td width="200">Sidebar content</td>
        <td>Main content</td>
    </tr>
    <tr>
        <td colspan="2">Footer content</td>
    </tr>
</table>
```

**Syntax Rules**

- Use `<table>` only for data with a logical row-column relationship
- Use CSS Grid or Flexbox for page layout
- If a layout table is unavoidable, add `role="presentation"` to the `<table>`
- Layout tables must not contain `<th>`, `<caption>`, `summary`, `aria-describedby`, `aria-labelledby`, or `aria-label`

**Constraints and Limitations**

- Layout tables confuse screen readers by announcing table structure where none exists
- The `role="presentation"` attribute is the only acceptable mitigation for legacy layout tables
- Layout tables are not responsive and require horizontal scrolling on mobile

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct — CSS Grid for Layout**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS Grid Layout</title>
    <style>
        .layout {
            display: grid;
            grid-template-columns: 200px 1fr;
            grid-template-rows: auto 1fr auto;
            grid-template-areas:
                "header header"
                "sidebar main"
                "footer footer";
            gap: 1em;
            min-height: 100vh;
        }
        .header { grid-area: header; background: #333; color: white; padding: 1em; }
        .sidebar { grid-area: sidebar; background: #f0f0f0; padding: 1em; }
        .main { grid-area: main; padding: 1em; }
        .footer { grid-area: footer; background: #333; color: white; padding: 1em; }
    </style>
</head>
<body>
    <div class="layout">
        <header class="header">Header</header>
        <aside class="sidebar">Sidebar</aside>
        <main class="main">Main content</main>
        <footer class="footer">Footer</footer>
    </div>
</body>
</html>
```

**Expected Output**

A page layout with a header, sidebar, main content, and footer, arranged in a grid. Screen readers see a logical document structure with no table semantics.

**Why This Output Occurs**

The CSS Grid layout uses semantic HTML elements (`<header>`, `<aside>`, `<main>`, `<footer>`) that convey meaning without any table structure. Screen readers announce the landmarks correctly.

---

**Example 2: Incorrect — Table for Layout (with Mitigation)**

```html
<!-- INCORRECT: Table used for layout -->
<table role="presentation">
    <tr>
        <td colspan="2">Header</td>
    </tr>
    <tr>
        <td width="200">Sidebar</td>
        <td>Main content</td>
    </tr>
</table>
```

**Expected Output**

The layout looks like a table visually, but the `role="presentation"` attribute tells screen readers to ignore the table semantics. This is a legacy mitigation, not a recommended approach.

**Why This Output Occurs**

The `role="presentation"` attribute removes the implicit table role from the `<table>` element, so screen readers treat it as a generic container. However, this is a workaround for legacy content, not a recommended practice. The correct approach is to use CSS Grid or Flexbox.

---

#### Real-World Cases

**Case 1: Government Website Modernisation**

Government websites that used tables for layout in the 2000s have been modernised to use CSS Grid, improving accessibility and responsiveness.

**Case 2: Email Templates**

HTML email templates historically used tables for layout because email clients had poor CSS support. Modern email templates use CSS with table fallbacks where necessary.

**Case 3: Legacy Enterprise Applications**

Older enterprise applications often use tables for layout. These are being progressively refactored to use CSS Grid as part of accessibility remediation.

---

### 4. Complex-Table Accessibility

#### Definitions

**Core Definition**

Complex-table accessibility is the practice of ensuring that tables with multiple header levels, merged cells, or non-adjacent headers remain understandable to screen reader users through the use of `id` and `headers` attributes, and careful responsive design that preserves semantic integrity.

**Technical Definition**

A complex table is one that has more than one row and/or one column of headings, or that includes merged or split cells. For such tables, the `scope` attribute alone is insufficient because a data cell may be associated with multiple headers that are not strictly horizontal or vertical. The `id` and `headers` attributes provide the explicit programmatic association required. The `id` attribute on each `<th>` element provides a unique identifier, and the `headers` attribute on each `<td>` (or `<th>`) element lists the IDs of all headers associated with that cell. When tables are made responsive on mobile screens — for example, by converting rows into cards or hiding columns — the semantic integrity of the table must be preserved. Hiding columns with `display: none` can cause screen readers to miss data; the recommended approach is to provide alternative layouts that maintain header associations, or to use `aria-label` on cells to describe their content in the mobile view.

**Beginner-Friendly Explanation**

Some tables are complicated — they have headers that span multiple rows and columns, or headers that don‘t sit directly above or beside the data. For these tables, simple `scope` isn‘t enough. You need to give each header a name (`id`) and then on each data cell, list which headers apply to it (`headers`). When you make the table responsive for mobile — for example, by stacking cells into cards — you need to make sure the header information still makes sense. Don‘t just hide columns with CSS; that can hide data from screen readers too.

#### Purposes

- To provide correct header associations for tables with multiple header levels
- To ensure that merged cells and spanning headers are announced correctly
- To maintain semantic integrity when tables are made responsive
- To satisfy WCAG Success Criterion 1.3.1 for complex tables
- To prevent screen readers from announcing incorrect or missing header context

#### Syntax Rules and Structure

**General Syntax for Complex Tables**

```html
<table>
    <caption>Complex Table Example</caption>
    <thead>
        <tr>
            <th rowspan="2" id="name">Name</th>
            <th colspan="2" id="contact">Contact</th>
        </tr>
        <tr>
            <th id="email">Email</th>
            <th id="phone">Phone</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th id="jane" headers="name">Jane</th>
            <td headers="contact email jane">jane@example.com</td>
            <td headers="contact phone jane">555-0101</td>
        </tr>
    </tbody>
</table>
```

**Responsive Table Pattern**

```html
<style>
    @media (max-width: 600px) {
        table, thead, tbody, tr, th, td {
            display: block;
        }
        thead {
            display: none;
        }
        td::before {
            content: attr(data-label) ": ";
            font-weight: bold;
        }
    }
</style>

<table>
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Email</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td data-label="Name">Jane</td>
            <td data-label="Email">jane@example.com</td>
        </tr>
    </tbody>
</table>
```

**Syntax Rules**

- Use `id` on every `<th>` in complex tables
- Use `headers` on every `<td>` and `<th>` that is associated with headers
- The `headers` attribute contains a space-separated list of header IDs
- For responsive tables, use `data-label` attributes on `<td>` elements and CSS `::before` to display labels in the mobile view
- Never hide columns with `display: none` without providing alternative header context

**Constraints and Limitations**

- WCAG Technique F90 describes the failure condition when `headers` and `id` are incorrectly associated
- Responsive tables that hide columns incorrectly are a common accessibility failure
- Screen reader support for complex tables varies; testing is essential
- The `role="grid"` pattern should be used only for interactive tables that require spreadsheet-like keyboard navigation

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Complex Table with Merged Headers**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Complex Table with Merged Headers</title>
</head>
<body>
    <table>
        <caption>Employee Contact Information</caption>
        <thead>
            <tr>
                <th rowspan="2" id="name">Name</th>
                <th colspan="2" id="contact">Contact</th>
            </tr>
            <tr>
                <th id="email" headers="contact">Email</th>
                <th id="phone" headers="contact">Phone</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th id="jane" headers="name">Jane Doe</th>
                <td headers="contact email jane">jane@example.com</td>
                <td headers="contact phone jane">555-0101</td>
            </tr>
            <tr>
                <th id="john" headers="name">John Smith</th>
                <td headers="contact email john">john@example.com</td>
                <td headers="contact phone john">555-0102</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

A table with a two-level header. “Contact” spans two columns, with “Email” and “Phone” as sub-headers. When a screen reader user navigates to “jane@example.com,” it announces “Contact, Email, Jane Doe, jane@example.com.”

**Why This Output Occurs**

The `id` attributes on the header cells provide unique identifiers. The `headers` attributes on the data cells list all applicable header IDs. The screen reader combines the group header (“Contact”), the specific column header (“Email”), and the row header (“Jane Doe”) to provide full context.

---

**Example 2: Responsive Table with Data Labels**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Responsive Table</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ccc; padding: 0.5em; text-align: left; }

        @media (max-width: 600px) {
            table, thead, tbody, tr, th, td { display: block; }
            thead { display: none; }
            td { position: relative; padding-left: 50%; }
            td::before {
                content: attr(data-label);
                position: absolute;
                left: 0;
                width: 45%;
                padding-left: 0.5em;
                font-weight: bold;
            }
        }
    </style>
</head>
<body>
    <table>
        <caption>Employee Directory</caption>
        <thead>
            <tr>
                <th scope="col">Name</th>
                <th scope="col">Department</th>
                <th scope="col">Extension</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td data-label="Name">Sarah Chen</td>
                <td data-label="Department">Engineering</td>
                <td data-label="Extension">4201</td>
            </tr>
            <tr>
                <td data-label="Name">Marcus Johnson</td>
                <td data-label="Department">Marketing</td>
                <td data-label="Extension">3105</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

**Expected Output**

On desktop, the table displays normally with headers. On mobile (under 600px), each row becomes a card-like block, with the column header displayed as a label before each data cell using `data-label` and CSS `::before`.

**Why This Output Occurs**

The `data-label` attributes on the `<td>` elements provide the header text for the mobile view. The CSS `::before` pseudo-element displays the label before the cell content. The `thead` is hidden visually but the semantic association remains in the DOM. Screen readers announce the table structure correctly.

---

#### Real-World Cases

**Case 1: Financial Statements with Quarterly Breakdowns**

Financial statements use `id` and `headers` for tables with annual headers spanning quarterly sub-headers.

**Case 2: Scientific Data Tables**

Scientific papers use complex tables with treatment groups, time points, and measurement types, requiring `id` and `headers` for correct associations.

**Case 3: Responsive E-Commerce Product Tables**

E-commerce comparison tables use `data-label` attributes for responsive mobile views, ensuring that product features remain associated with their values when the table stacks vertically.

---

### 5. Choosing Semantic Elements Rather Than Purely Visual Elements

#### Definitions

**Core Definition**

Choosing semantic elements for tables means using the native HTML table elements (`<table>`, `<caption>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`) for tabular data, and using CSS Grid or Flexbox for layout, rather than using `<div>` or `<span>` elements to simulate tables.

**Technical Definition**

Semantic table markup requires using the native HTML table elements, which carry inherent accessibility semantics that map to platform accessibility APIs. WCAG Success Criterion 1.3.1 requires that information, structure, and relationships conveyed through presentation can be programmatically determined. Using `<div>` elements with CSS to create a table-like appearance fails this criterion because there is no semantic row-column relationship. The W3C’s ARIA Authoring Practices Guide states that “authors are strongly encouraged to use a native HTML `table` element whenever possible.” The `role="grid"` pattern should be reserved for interactive tables that require spreadsheet-like keyboard navigation, not for read-only data tables.

**Beginner-Friendly Explanation**

Don‘t use `<div>` tags to fake a table. Use the real `<table>`, `<th>`, and `<td>` tags. That way, screen readers can announce the table structure correctly, and users can navigate between cells using table navigation commands. CSS can style the table however you like — the HTML provides the meaning.

#### Purposes

- To ensure content is accessible to screen readers and other assistive technologies
- To make HTML meaningful and understandable to search engines
- To separate content structure (HTML) from visual presentation (CSS)
- To comply with web standards and accessibility guidelines
- To improve code maintainability and consistency

#### Correct vs. Incorrect Patterns

| If you want to… | Use… | Not… |
|---|---|---|
| Create a data table | `<table>`, `<th>`, `<td>` | `<div>` with CSS Grid |
| Provide a table title | `<caption>` | `<h2>` above the table |
| Associate headers with data | `scope` or `id`/`headers` | `aria-label` on each cell |
| Create an interactive grid | `role="grid"` with full keyboard support | `<table>` with click handlers |
| Create a page layout | CSS Grid or Flexbox | `<table>` |

**Syntax Rules**

- Use `<table>` only for tabular data
- Use `<th>` for headers and `<td>` for data
- Use `<caption>` for the table title
- Use CSS for all visual styling
- Never use tables or table elements for layout purposes

**Constraints and Limitations**

- Screen readers may not correctly interpret table structure if semantic elements are not used
- Tables used for layout violate accessibility guidelines
- The `role="grid"` pattern adds complexity and should be used only when interactive behaviour is required

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct vs. Incorrect Table Markup**

```html
<!-- CORRECT: Native HTML table -->
<table>
    <caption>Employee Salaries</caption>
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Salary</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">Jane</th>
            <td>$75,000</td>
        </tr>
        <tr>
            <th scope="row">John</th>
            <td>$82,000</td>
        </tr>
    </tbody>
</table>

<!-- INCORRECT: Div-based fake table -->
<div class="table">
    <div class="caption">Employee Salaries</div>
    <div class="header">
        <div class="cell">Name</div>
        <div class="cell">Salary</div>
    </div>
    <div class="row">
        <div class="cell">Jane</div>
        <div class="cell">$75,000</div>
    </div>
</div>
```

**Expected Output**

Both may look similar visually, but only the first is semantically correct. Screen readers announce the first as a table with headers and data. The second is just a group of `<div>` elements with no table semantics.

**Why This Output Occurs**

The `<table>`, `<thead>`, `<tbody>`, `<th>`, and `<td>` elements carry semantic meaning that assistive technology can interpret. The `<div>` elements carry no table semantics, so screen readers cannot identify the content as a table or navigate it with table commands.

---

**Example 2: Layout with CSS Grid Instead of Table**

```html
<!-- CORRECT: CSS Grid for layout -->
<div class="page">
    <header>Site Header</header>
    <nav>Navigation</nav>
    <main>Main Content</main>
    <footer>Site Footer</footer>
</div>

<style>
    .page {
        display: grid;
        grid-template-areas:
            "header header"
            "nav main"
            "footer footer";
        grid-template-columns: 200px 1fr;
        gap: 1em;
    }
    header { grid-area: header; }
    nav { grid-area: nav; }
    main { grid-area: main; }
    footer { grid-area: footer; }
</style>
```

**Expected Output**

A page layout with header, navigation, main content, and footer. Screen readers see a logical document structure with no table semantics.

**Why This Output Occurs**

The CSS Grid layout uses semantic HTML elements (`<header>`, `<nav>`, `<main>`, `<footer>`) that convey meaning without any table structure. Screen readers announce the landmarks correctly.

---

#### Real-World Cases

**Case 1: Government Accessibility Compliance**

Government websites must use semantic table markup to comply with WCAG 2.1 Success Criterion 1.3.1.

**Case 2: Screen Reader Navigation**

Screen readers use table navigation commands (e.g., Ctrl+Alt+Arrow keys) to move between cells. These commands only work with native HTML tables.

**Case 3: Responsive Design**

CSS Grid and Flexbox are responsive and adapt to different screen sizes; tables are rigid and require horizontal scrolling on mobile.

---

## References

- W3C – H39: Using caption elements to associate data table captions with data tables – https://www.w3.org/WAI/WCAG21/Techniques/html/H39
- W3C – H43: Using id and headers attributes to associate data cells with header cells in data tables – https://www.w3.org/WAI/WCAG21/Techniques/html/H43
- W3C – H63: Using the scope attribute to associate header cells and data cells in data tables – https://www.w3.org/WAI/WCAG21/Techniques/html/H63
- W3C – H73: Using the summary attribute of the table element to give an overview of data tables (Obsolete) – https://www.w3.org/WAI/WCAG21/Techniques/html/H73
- W3C – F90: Failure of Success Criterion 1.3.1 for incorrectly associating table headers and content via the headers and id attributes – https://www.w3.org/WAI/WCAG21/Techniques/failures/F90
- W3C – F91: Failure of Success Criterion 1.3.1 for not correctly marking up table headers – https://www.w3.org/WAI/WCAG21/Techniques/failures/F91
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – ARIA Authoring Practices Guide: Table Pattern – https://www.w3.org/WAI/ARIA/apg/patterns/table/
- WHATWG HTML Living Standard – Tabular data – https://html.spec.whatwg.org/multipage/tables.html
- 18F – Accessibility Guide: Tables – https://guides.18f.org/accessibility/tables/
- WebAIM – Creating Accessible Tables – https://webaim.org/techniques/tables/
- MDN Web Docs – HTML table accessibility – https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Table_accessibility
- Microsoft Learn – ARIA data table warning – https://learn.microsoft.com/en-us/windows/win32/winauto/aria-data-table-description
- TestParty – Table Accessibility Guide: Data Tables and WCAG Compliance – https://testparty.ai/blog/wcag-tables-accessibility
- RGAA Checker – Tableau de données accessible : caption, th et scope – https://rgaa-checker.com/guide/composants/tableau