# HTML Table Fundamentals: Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

HTML tables are a semantic structure for organising and presenting two-dimensional tabular data — information arranged in a grid of rows and columns — using a set of interconnected elements that define the table‘s container, rows, header cells, and data cells.

**Technical Definition**

The `<table>` element represents tabular data — that is, information presented in a two-dimensional table comprised of rows and columns of cells containing data. It is categorised as flow content and palpable content. Its permitted content model consists of, in order: an optional `<caption>` element, zero or more `<colgroup>` elements, an optional `<thead>` element, either zero or more `<tbody>` elements or one or more `<tr>` elements, and an optional `<tfoot>` element. Its implicit ARIA role is `table`, and its DOM interface is `HTMLTableElement`. The `<tr>` element represents a row of cells in a table. Its permitted content is zero or more `<td>` or `<th>` elements, or a mixture of them. Its permitted parents are `<table>`, `<thead>`, `<tbody>`, or `<tfoot>`. The `<th>` element defines a cell as header of a group of table cells, with the exact nature of the group defined by the `scope` and `headers` attributes. The `<td>` element defines a cell of a table that contains data and may be used as a child of the `<tr>` element.

**Beginner-Friendly Explanation**

A table on a webpage is exactly like a table in a spreadsheet or a printed document — it organises information into rows and columns so you can quickly look up values. Think of a bus timetable: the rows are different routes, the columns are different times, and you find the information you need by matching a row with a column. HTML gives you four main tags to build tables: `<table>` wraps the whole thing, `<tr>` creates each row, `<th>` creates header cells (for labels), and `<td>` creates data cells (for actual values). The browser then arranges everything into a grid.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Tabular data** | Tables are for data with relationships between rows and columns — not for page layout |
| **Nested structure** | `<td>` and `<th>` go inside `<tr>`, which go inside `<table>` |
| **Header cells** | `<th>` defines headings for rows or columns; `<td>` contains data |
| **Scope attribute** | The `scope` attribute on `<th>` defines whether a header applies to a row, column, or group |
| **Colspan and rowspan** | Cells can span multiple columns or rows using `colspan` and `rowspan` |
| **Accessibility-critical** | Proper table markup enables screen readers to announce header-data relationships |
| **Deprecated attributes** | Presentational attributes like `align`, `bgcolor`, and `border` are obsolete; use CSS instead |

---

### Prerequisites

- Basic familiarity with HTML document structure (`<html>`, `<head>`, `<body>`)
- Understanding of HTML elements, tags, and attributes
- Awareness of block-level vs. inline-level elements
- Basic knowledge of accessibility principles (helpful but not required)
- Basic knowledge of CSS (helpful for styling tables)

---

### Related Programming Areas

- **Semantic HTML** – Tables are a core semantic grouping element for tabular data
- **Web Accessibility (A11y)** – Tables are one of the most common sources of accessibility problems when misused
- **CSS** – Table styling, borders, spacing, and responsive behaviour are controlled via CSS
- **Data Presentation** – Tables are used for financial data, schedules, comparison charts, and statistics
- **WCAG Compliance** – Tables must be properly structured to satisfy WCAG Success Criterion 1.3.1

---

## Core Concepts / Features

---

### 1. The `<table>` Element

#### Definitions

**Core Definition**

The `<table>` element is the wrapper element that defines the start and end of a table, containing all rows, cells, and related structural elements.

**Technical Definition**

The `<table>` HTML element represents tabular data — information presented in a two-dimensional table comprised of rows and columns of cells containing data. It is categorised as flow content and palpable content. Its permitted content, in order, is: an optional `<caption>` element, zero or more `<colgroup>` elements, an optional `<thead>` element, either zero or more `<tbody>` elements or one or more `<tr>` elements, and an optional `<tfoot>` element. The tag omission is none — both start and end tags are mandatory. Its implicit ARIA role is `table`, and its DOM interface is `HTMLTableElement`.

**Beginner-Friendly Explanation**

The `<table>` tag is the container for your entire table. It tells the browser “everything inside me is part of a table.” You put rows (`<tr>`) inside it, and each row contains cells (`<td>` or `<th>`). Without the `<table>` wrapper, the browser wouldn‘t know that the rows belong together.

#### Purposes

- To represent tabular data with relationships between rows and columns
- To provide a semantic container for table rows and cells
- To enable screen readers to identify and navigate tabular data
- To serve as the root element for table structure

#### Syntax Rules and Structure

**General Syntax**

```html
<table>
    <!-- optional caption -->
    <!-- optional colgroup elements -->
    <!-- optional thead -->
    <!-- zero or more tbody elements OR one or more tr elements -->
    <!-- optional tfoot -->
</table>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<table>` | Opening tag; indicates the start of a table |
| `Content` | Table rows (`<tr>`) and optional structural elements |
| `</table>` | Closing tag; required |

**Syntax Rules**

- Both start and end tags are mandatory
- The content model is: optional `<caption>`, zero or more `<colgroup>`, optional `<thead>`, either zero or more `<tbody>` or one or more `<tr>`, optional `<tfoot>`
- Only global attributes are permitted on `<table>` (the `align`, `bgcolor`, and `border` attributes are deprecated)
- The element must not appear as a descendant of a `<caption>` element

**Constraints and Limitations**

- Tables must not be used for page layout; use CSS Grid or Flexbox instead
- The `align` attribute is deprecated; use CSS `margin-left` and `margin-right` for alignment
- The `bgcolor` attribute is deprecated; use CSS `background-color`
- The `border` attribute is deprecated; use CSS `border`

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Table Structure**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Basic Table Demo</title>
</head>
<body>
    <!-- table wraps the entire table -->
    <table>
        <!-- tr creates a row -->
        <tr>
            <!-- th creates header cells -->
            <th>Name</th>
            <th>Age</th>
            <th>City</th>
        </tr>
        <!-- second row -->
        <tr>
            <!-- td creates data cells -->
            <td>Alice</td>
            <td>30</td>
            <td>New York</td>
        </tr>
        <!-- third row -->
        <tr>
            <td>Bob</td>
            <td>25</td>
            <td>London</td>
        </tr>
    </table>
</body>
</html>
```

**Expected Output**

A table with three columns (Name, Age, City) and two data rows (Alice and Bob). The header cells are bold and centred by default.

**Why This Output Occurs**

The `<table>` element creates the table container. The `<tr>` elements create rows. The `<th>` elements create header cells, which browsers render with `font-weight: bold` and `text-align: center` by default. The `<td>` elements create data cells, which are left-aligned by default.

---

**Example 2: Table with Caption**

```html
<table>
    <caption>Employee Directory</caption>
    <tr>
        <th scope="col">Name</th>
        <th scope="col">Department</th>
        <th scope="col">Extension</th>
    </tr>
    <tr>
        <th scope="row">Sarah Chen</th>
        <td>Engineering</td>
        <td>4201</td>
    </tr>
    <tr>
        <th scope="row">Marcus Johnson</th>
        <td>Marketing</td>
        <td>3105</td>
    </tr>
</table>
```

**Expected Output**

A table with a caption “Employee Directory” displayed above the table. The first column contains row headers (bold), and the first row contains column headers.

**Why This Output Occurs**

The `<caption>` element provides a visible title for the table. The `scope="col"` attribute on `<th>` elements in the first row identifies them as column headers. The `scope="row"` attribute on `<th>` elements in the first column identifies them as row headers.

#### Real-World Cases

**Case 1: Financial Data Tables**

Bank statements and financial reports use tables to display transactions, balances, and dates in a structured grid.

**Case 2: Comparison Tables**

Product comparison pages use tables to show features across different products, with features as rows and products as columns.

**Case 3: Timetables and Schedules**

Bus, train, and flight schedules use tables with routes as rows and times as columns.

---

### 2. The `<tr>` Element (Table Row)

#### Definitions

**Core Definition**

The `<tr>` element is the container element used to group table cells into horizontal rows.

**Technical Definition**

The `<tr>` HTML element defines a row of cells in a table. The row‘s cells can then be established with a mix of `<td>` (data cell) and `<th>` (header cell) elements. Its content model is zero or more `<td>` or `<th>` elements, or a mixture of them. Its permitted parents are `<table>`, `<thead>`, `<tbody>`, or `<tfoot>`. The start tag is mandatory; the end tag may be omitted if the `<tr>` element is immediately followed by another `<tr>` element, or if the parent table group has no more content. Its implicit ARIA role is `row`, and its DOM interface is `HTMLTableRowElement`.

**Beginner-Friendly Explanation**

The `<tr>` tag creates one horizontal row in your table. You put all the cells for that row inside it — some may be header cells (`<th>`) and some may be data cells (`<td>`). If you have a table with 5 rows, you‘ll have 5 `<tr>` elements.

#### Purposes

- To group table cells into horizontal rows
- To provide the structural unit for table rows
- To enable row-level styling and scripting
- To define the horizontal axis of the table grid

#### Syntax Rules and Structure

**General Syntax**

```html
<tr>
    <td>Cell 1</td>
    <td>Cell 2</td>
    <td>Cell 3</td>
</tr>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<tr>` | Opening tag; indicates a row |
| `<td>` or `<th>` | Cells within the row |
| `</tr>` | Closing tag; optional in specific circumstances |

**Syntax Rules**

- The start tag is mandatory; the end tag may be omitted if immediately followed by another `<tr>` or if the parent table group has no more content
- The content model is zero or more `<td>` or `<th>` elements
- Only global attributes are permitted (the `align` attribute is deprecated)
- The `<tr>` element must be a child of `<table>`, `<thead>`, `<tbody>`, or `<tfoot>`

**Constraints and Limitations**

- The `align` attribute is deprecated; use CSS `text-align` instead
- The `<tr>` element cannot contain text directly; all content must be inside cells
- Rows without cells are technically valid but serve no purpose

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Multiple Rows**

```html
<table>
    <!-- First row: headers -->
    <tr>
        <th>Product</th>
        <th>Price</th>
        <th>Stock</th>
    </tr>

    <!-- Second row: data -->
    <tr>
        <td>Laptop</td>
        <td>$999</td>
        <td>15</td>
    </tr>

    <!-- Third row: data -->
    <tr>
        <td>Mouse</td>
        <td>$29</td>
        <td>50</td>
    </tr>

    <!-- Fourth row: data -->
    <tr>
        <td>Keyboard</td>
        <td>$79</td>
        <td>30</td>
    </tr>
</table>
```

**Expected Output**

A table with one header row and three data rows.

**Why This Output Occurs**

Each `<tr>` element creates a distinct horizontal row. The browser stacks the rows vertically, creating the table grid. The `<th>` cells in the first row are rendered as bold, centred headers; the `<td>` cells in subsequent rows are rendered as standard data cells.

---

**Example 2: Row with Mixed Header and Data Cells**

```html
<table>
    <tr>
        <th scope="row">January</th>
        <td>$1,200</td>
        <td>$800</td>
        <td>$400</td>
    </tr>
    <tr>
        <th scope="row">February</th>
        <td>$1,500</td>
        <td>$900</td>
        <td>$600</td>
    </tr>
</table>
```

**Expected Output**

A table where the first column contains row headers (bold) and the remaining columns contain data.

**Why This Output Occurs**

The `<th scope="row">` elements identify the first cell in each row as a row header. This creates a semantic relationship between the row header and the data cells in the same row, which screen readers can announce.

#### Real-World Cases

**Case 1: Invoice Line Items**

Invoices use table rows for each line item, with columns for description, quantity, unit price, and total.

**Case 2: Sports League Standings**

Sports standings tables use rows for each team, with columns for wins, losses, and points.

**Case 3: Grade Books**

Teachers use table rows for each student, with columns for assignment scores and final grades.

---

### 3. The `<th>` Element (Table Header)

#### Definitions

**Core Definition**

The `<th>` element defines a cell as a header for a group of table cells, providing a label or heading for a row or column of data.

**Technical Definition**

The `<th>` HTML element defines a cell as header of a group of table cells. The exact nature of this group is defined by the `scope` and `headers` attributes. It has no content categories. Its permitted content is flow content, but with no header, footer, sectioning content, or heading content descendants. The start tag is mandatory; the end tag may be omitted if immediately followed by a `<th>` or `<td>` element or if there is no more data in its parent element. Its permitted parent is a `<tr>` element. Its implicit ARIA role is `columnheader` or `rowheader`, and its DOM interface is `HTMLTableCellElement`.

**Beginner-Friendly Explanation**

The `<th>` tag creates a header cell — a cell that labels other cells. For example, in a table of student grades, the top row might have `<th>` cells saying “Name,” “Math,” and “Science.” These labels help everyone understand what the data in each column means. Browsers display `<th>` cells in bold and centred by default.

#### Purposes

- To define header cells that label rows or columns of data
- To provide semantic structure for screen readers
- To enable the `scope` attribute to define header-data relationships
- To distinguish labels from actual data

#### Syntax Rules and Structure

**General Syntax**

```html
<th scope="col">Column Header</th>
<th scope="row">Row Header</th>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<th>` | Opening tag; indicates a header cell |
| `scope` | Defines whether the header applies to a column, row, or group |
| `Content` | Flow content; the header text |
| `</th>` | Closing tag; optional in specific circumstances |

**Scope Attribute Values**

| Value | Description |
|---|---|
| `col` | Header applies to the column below it |
| `row` | Header applies to the row beside it |
| `colgroup` | Header applies to a group of columns |
| `rowgroup` | Header applies to a group of rows |

**Syntax Rules**

- The `<th>` element must be a child of a `<tr>` element
- The `scope` attribute defines the header‘s relationship to data cells
- The `abbr` attribute can provide an abbreviated description for screen readers
- The `colspan` and `rowspan` attributes allow headers to span multiple columns or rows

**Constraints and Limitations**

- The `align`, `bgcolor`, `char`, and `valign` attributes are deprecated; use CSS instead
- Without `scope`, screen readers may not correctly associate headers with data cells
- The `<th>` element should not be used for data that is not a header

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Column Headers with Scope**

```html
<table>
    <thead>
        <tr>
            <th scope="col">Name</th>
            <th scope="col">Email</th>
            <th scope="col">Role</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Jane Doe</td>
            <td>jane@example.com</td>
            <td>Admin</td>
        </tr>
        <tr>
            <td>John Smith</td>
            <td>john@example.com</td>
            <td>Editor</td>
        </tr>
    </tbody>
</table>
```

**Expected Output**

A table with bold column headers and two data rows.

**Why This Output Occurs**

The `scope="col"` attribute tells screen readers that these headers apply to the columns below them. When a screen reader user navigates to a data cell, it can announce the associated column header.

---

**Example 2: Row Headers with Scope**

```html
<table>
    <tr>
        <th scope="row">Monday</th>
        <td>Math</td>
        <td>Science</td>
        <td>English</td>
    </tr>
    <tr>
        <th scope="row">Tuesday</th>
        <td>History</td>
        <td>Art</td>
        <td>Music</td>
    </tr>
</table>
```

**Expected Output**

A table where the first column contains bold row headers.

**Why This Output Occurs**

The `scope="row"` attribute tells screen readers that these headers apply to the data cells in the same row. When navigating to “Math,” the screen reader announces “Monday, Math.”

#### Real-World Cases

**Case 1: Financial Statements**

Financial statements use `<th scope="col">` for years and `<th scope="row">` for line items like revenue and expenses.

**Case 2: Scientific Data Tables**

Scientific papers use `<th>` for variable names and units, with `scope` attributes defining the relationships.

**Case 3: Sports Statistics**

Sports statistics tables use `<th>` for player names and statistical categories.

---

### 4. The `<td>` Element (Table Data)

#### Definitions

**Core Definition**

The `<td>` element defines a standard data cell that contains the actual data or content of the table.

**Technical Definition**

The `<td>` HTML element defines a cell of a table that contains data and may be used as a child of the `<tr>` element. It supports the `colspan`, `headers`, and `rowspan` attributes. The `colspan` attribute contains a non-negative integer indicating how many columns the cell spans; its default value is 1, and values higher than 1000 are treated as 1. The `rowspan` attribute indicates how many rows the cell spans; its default value is 1, and if set to 0, it extends to the end of the table section. Its DOM interface is `HTMLTableCellElement`.

**Beginner-Friendly Explanation**

The `<td>` tag creates a data cell — a cell that contains actual information, like a number, a name, or a date. In a table of student grades, the cells containing the actual grades would be `<td>` elements. Browsers display `<td>` cells left-aligned by default.

#### Purposes

- To contain the actual data or content of a table
- To provide the grid cells that hold values
- To support spanning multiple columns or rows via `colspan` and `rowspan`
- To enable association with header cells via the `headers` attribute

#### Syntax Rules and Structure

**General Syntax**

```html
<td>Cell content</td>
<td colspan="2">Cell spanning two columns</td>
<td rowspan="3">Cell spanning three rows</td>
```

**Component Breakdown**

| Component | Description |
|---|---|
| `<td>` | Opening tag; indicates a data cell |
| `Content` | Flow content; the data or content |
| `</td>` | Closing tag; optional in specific circumstances |

**Spanning Attributes**

| Attribute | Description | Default |
|---|---|---|
| `colspan` | Number of columns the cell spans | 1 |
| `rowspan` | Number of rows the cell spans | 1 |
| `headers` | Space-separated list of header cell IDs | — |

**Syntax Rules**

- The `<td>` element must be a child of a `<tr>` element
- The `colspan` and `rowspan` attributes accept non-negative integers
- The `headers` attribute associates the cell with specific header cells by ID
- The `abbr`, `align`, `bgcolor`, `char`, and `valign` attributes are deprecated

**Constraints and Limitations**

- The `<td>` element cannot contain block-level elements like `<div>` or `<p>` in strict HTML4, but HTML5 permits flow content
- Values higher than 1000 for `colspan` are treated as 1
- Values higher than 65534 for `rowspan` are clipped to 65534

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Data Cells**

```html
<table>
    <tr>
        <th>Name</th>
        <th>Score</th>
    </tr>
    <tr>
        <td>Alice</td>
        <td>95</td>
    </tr>
    <tr>
        <td>Bob</td>
        <td>87</td>
    </tr>
</table>
```

**Expected Output**

A table with two columns and three rows (one header row, two data rows).

**Why This Output Occurs**

Each `<td>` element creates a data cell. The browser arranges the cells into a grid based on the number of cells in each row.

---

**Example 2: Cells Spanning Columns and Rows**

```html
<table border="1">
    <tr>
        <th>Name</th>
        <th colspan="2">Contact Information</th>
    </tr>
    <tr>
        <td>Jane</td>
        <td>jane@example.com</td>
        <td>555-0101</td>
    </tr>
    <tr>
        <td rowspan="2">John</td>
        <td>john@example.com</td>
        <td>555-0102</td>
    </tr>
    <tr>
        <td>john.work@example.com</td>
        <td>555-0103</td>
    </tr>
</table>
```

**Expected Output**

A table where the “Contact Information” header spans two columns, and the “John” cell spans two rows.

**Why This Output Occurs**

The `colspan="2"` attribute on the “Contact Information” header makes it span two columns. The `rowspan="2"` attribute on the “John” cell makes it span two rows. The browser merges the cells accordingly.

#### Real-World Cases

**Case 1: Price Comparison Tables**

E-commerce sites use `<td>` for prices, product names, and availability across different vendors.

**Case 2: Sports Scoreboards**

Sports scoreboards use `<td>` for scores, team names, and game times.

**Case 3: Inventory Lists**

Warehouse systems use `<td>` for item names, quantities, and locations.

---

### 5. Rows and Columns (The Grid Matrix)

#### Definitions

**Core Definition**

Rows and columns are the two axes of a table grid, formed by nesting `<td>` and `<th>` cells inside `<tr>` rows; the table model builds a grid of cells from these rows and columns.

**Technical Definition**

A table is a structured set of data made up of rows and columns (tabular data). The table model consists of a grid of cells. Each `<tr>` element defines a row, and the cells within that row (`<td>` and `<th>`) occupy columns. The browser‘s table model determines the column positions based on the order of cells, accounting for `colspan` and `rowspan` attributes. The point of a table is that it is rigid — information is easily interpreted by making visual associations between row and column headers.

**Beginner-Friendly Explanation**

A table is like a grid. The rows go across (left to right), and the columns go down (top to bottom). When you put cells inside a row, they line up to form columns. The browser figures out which column each cell belongs to based on where it is in the row. This grid structure is what makes tables useful for looking up information — you find the row you want, then find the column you want, and the cell where they meet has your answer.

#### Purposes

- To provide the two-dimensional structure for tabular data
- To enable quick lookups by matching row and column headers
- To create visual associations between headers and data
- To support complex table structures with spanning cells

#### Syntax Rules and Structure

**Row-Column Relationship**

| Concept | Description |
|---|---|
| Row | Defined by a `<tr>` element; a horizontal group of cells |
| Column | The vertical alignment of cells across multiple rows |
| Cell | A `<td>` or `<th>` element; the intersection of a row and column |

**Syntax Rules**

- Cells are placed in columns in the order they appear in the row
- The `colspan` attribute makes a cell occupy multiple column positions
- The `rowspan` attribute makes a cell occupy multiple row positions
- The table model calculates the grid based on the sequence and spanning of cells

**Constraints and Limitations**

- Irregular tables (with cells spanning multiple rows/columns) are harder for screen readers to navigate
- The table model algorithm handles implicit cell placement based on available slots
- Tables with complex spanning should include `scope` and `headers` attributes for accessibility

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Grid Structure**

```html
<table>
    <tr>
        <th>Name</th>
        <th>Math</th>
        <th>Science</th>
    </tr>
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
</table>
```

**Expected Output**

A 3×3 grid: 3 columns (Name, Math, Science) and 3 rows (header row, Alice, Bob).

**Why This Output Occurs**

The browser creates a grid with 3 columns based on the 3 cells in the first row. The second and third rows each have 3 cells that align with the columns defined by the header row.

---

**Example 2: Column Association with Headers Attribute**

```html
<table>
    <tr>
        <th id="name">Name</th>
        <th id="math">Math</th>
        <th id="science">Science</th>
    </tr>
    <tr>
        <td headers="name">Alice</td>
        <td headers="math">95</td>
        <td headers="science">88</td>
    </tr>
</table>
```

**Expected Output**

A table where each data cell is explicitly associated with its column header.

**Why This Output Occurs**

The `headers` attribute on each `<td>` references the `id` of the corresponding `<th>` element, creating an explicit programmatic association. This is especially useful in complex tables where `scope` alone is insufficient.

#### Real-World Cases

**Case 1: NASA Planetary Fact Sheet**

The MDN table basics tutorial uses NASA‘s Planetary Fact Sheet as an example, demonstrating a complex table with rows for each planet and columns for mass, diameter, density, gravity, and other properties.

**Case 2: Census Data**

Government census data uses large tables with rows for geographic areas and columns for population statistics.

**Case 3: Sports League Tables**

Football league tables use rows for teams and columns for played, won, drawn, lost, and points.

---

### 6. Choosing Semantic Elements Rather Than Purely Visual Elements

#### Definitions

**Core Definition**

Choosing semantic table elements means using `<table>`, `<tr>`, `<th>`, and `<td>` for actual tabular data, and using CSS Grid or Flexbox for page layout, rather than using tables to achieve visual layouts.

**Technical Definition**

Semantic table markup requires using the `<table>` element only for tabular data — information that has a logical relationship between rows and columns. For page layout, CSS Grid and Flexbox are the appropriate tools. WCAG Success Criterion 1.3.1 (Info and Relationships) requires that information, structure, and relationships conveyed through presentation can be programmatically determined. Using tables for layout fails this criterion because there is no logical row-column relationship in the content.

**Beginner-Friendly Explanation**

Use tables for data, not for layout. If you‘re arranging items on a page — like a sidebar and a main content area — use CSS, not a table. Tables are rigid grids that are perfect for data but terrible for flexible page layouts. Screen readers also get confused when tables are used for layout because they try to announce non-existent row-column relationships.

#### Correct vs. Incorrect Patterns

| If you want to… | Use… | Not… |
|---|---|---|
| Display tabular data | `<table>`, `<tr>`, `<th>`, `<td>` | `<div>` with CSS Grid |
| Create a page layout | CSS Grid or Flexbox | `<table>` |
| Show a comparison | `<table>` with `<th>` headers | `<div>` with columns |
| Display a schedule | `<table>` with time rows | `<ul>` with CSS |

**Syntax Rules**

- Use `<table>` only for data that has a logical row-column relationship
- Use `<th>` for headers and `<td>` for data
- Use CSS for all visual styling (borders, spacing, colours)
- Never use tables for layout; use CSS Grid or Flexbox

**Constraints and Limitations**

- Tables used for layout confuse screen readers and violate accessibility guidelines
- Tables are rigid and do not adapt well to responsive design
- The deprecated `align`, `bgcolor`, and `border` attributes must not be used

#### Annotated Complete Step-by-Step Code Examples

**Example 1: Correct vs. Incorrect Table Usage**

```html
<!-- CORRECT: Table for tabular data -->
<h2>Employee Directory</h2>
<table>
    <tr>
        <th scope="col">Name</th>
        <th scope="col">Department</th>
    </tr>
    <tr>
        <td>Jane Doe</td>
        <td>Engineering</td>
    </tr>
</table>

<!-- INCORRECT: Table for page layout -->
<table>
    <tr>
        <td width="200">Sidebar content</td>
        <td>Main content</td>
    </tr>
</table>
```

**Expected Output**

The correct example displays a data table with clear headers. The incorrect example may look like a layout, but it confuses screen readers.

**Why This Output Occurs**

The correct example uses a table for actual tabular data — information with a row-column relationship. The incorrect example uses a table purely for visual layout, which has no semantic row-column relationship. Screen readers will announce the layout table as a table with rows and columns, which is misleading.

#### Real-World Cases

**Case 1: Government Accessibility Compliance**

Government websites must not use tables for layout to comply with WCAG 2.1 Success Criterion 1.3.1.

**Case 2: Screen Reader Navigation**

Screen readers announce table structure, including row and column counts. Layout tables create confusing announcements.

**Case 3: Responsive Design**

CSS Grid and Flexbox are responsive and adapt to different screen sizes; tables are rigid and require horizontal scrolling on mobile.

---

## References

- MDN Web Docs – `<table>`: The Table element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/table
- MDN Web Docs – `<tr>`: The Table Row element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/tr
- MDN Web Docs – `<th>`: The Table Header element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/th
- MDN Web Docs – `<td>`: The Table Data Cell element – https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/td
- MDN Web Docs – HTML table basics – https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics
- WHATWG HTML Living Standard – The table element – https://html.spec.whatwg.org/multipage/tables.html#the-table-element
- WHATWG HTML Living Standard – The tr element – https://html.spec.whatwg.org/multipage/tables.html#the-tr-element
- WHATWG HTML Living Standard – The th element – https://html.spec.whatwg.org/multipage/tables.html#the-th-element
- WHATWG HTML Living Standard – The td element – https://html.spec.whatwg.org/multipage/tables.html#the-td-element
- W3C – HTML 5: The table element – https://dev.w3.org/html5/spec-author-view/tabular-data.html#the-table-element
- W3C – WCAG 2.1 Understanding Success Criterion 1.3.1: Info and Relationships – https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html
- W3C – H51: Using table markup to present tabular information – https://www.w3.org/WAI/WCAG21/Techniques/html/H51
- W3C – H63: Using the scope attribute to associate header cells and data cells in data tables – https://www.w3.org/WAI/WCAG21/Techniques/html/H63
- web.dev – HTML tables – https://web.dev/learn/html/tables