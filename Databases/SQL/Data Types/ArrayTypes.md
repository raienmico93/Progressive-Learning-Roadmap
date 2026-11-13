# Comprehensive Programming Cheat Sheet: PostgreSQL Array Types

## Topic Overview

### Definitions

**Core Definition**
Array types are PostgreSQL data types that allow a single column to store an ordered, indexed collection of values of the same element type—including integers, text, enums, composite types, ranges, and even other arrays—enabling multiple values to be held in one row without creating a separate child table.

**Technical Definition**
PostgreSQL allows columns of a table to be defined as variable-length multidimensional arrays. Arrays of any built-in or user-defined base type, enum type, composite type, range type, or domain can be created. An array data type is named by appending square brackets (`[]`) to the data type name of the array elements. The syntax for `CREATE TABLE` allows exact sizes to be specified, but the current implementation ignores supplied array size limits, and the declared number of dimensions is not enforced—arrays of a particular element type are all considered to be of the same type, regardless of size or number of dimensions. PostgreSQL arrays are **1-indexed** by default, though custom lower bounds can be specified in array literals. The `array` type is a first-class citizen: it supports dedicated operators (`@>`, `<@`, `&&`, `||`), a rich function library, GIN indexing, and the `unnest()` function that expands an array into a set of rows.

**Beginner-Friendly Explanation**
An array is a container that holds multiple values in a single database column. Instead of creating a separate `tags` table with a row for every tag on every post, you can store all of a post's tags in one array column. Think of an array as a numbered list inside a single cell: the first item is at position 1, the second at position 2, and so on. Arrays are convenient for small, tightly coupled collections that are always read and written together—but they are not a replacement for proper relational design when you need to search, join, or aggregate on individual elements at scale.

### Key Characteristics

- **1-indexed by default**: PostgreSQL arrays start at subscript 1, not 0. Custom lower bounds are possible (e.g., `'[0:2]={1,2,3}'`), but 1-based indexing is the norm.
- **Dimensions are not enforced**: Declaring `integer[3][3]` or `integer ARRAY[4]` is purely documentation; PostgreSQL does not enforce the declared size or number of dimensions at runtime. Arrays of the same element type are considered the same type regardless of dimensionality.
- **Variable-length and multidimensional**: Arrays can grow and shrink, and can have multiple dimensions. Multidimensional arrays are compared in row-major order (the last subscript varies most rapidly).
- **Rich operator and function support**: Containment (`@>`, `<@`), overlap (`&&`), concatenation (`||`), and a comprehensive function library including `array_agg`, `array_append`, `array_remove`, and `unnest`.
- **GIN indexing enables fast element search**: A GIN index on an array column accelerates containment (`@>`) and overlap (`&&`) queries, making arrays viable for certain search workloads—but GIN indexes carry higher write overhead than B-tree indexes.

### Prerequisites

- A running PostgreSQL instance (version 9.4+ for most array features; `array_sample` and `array_shuffle` require PostgreSQL 16+).
- Familiarity with `CREATE TABLE` and SQL data types.
- Understanding of when to use an array versus a normalized child table or a `JSONB` column.
- Awareness of GIN indexing and its operator support for array containment and overlap queries.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations with `[]` or `ARRAY`.
- Data Manipulation Language (DML) – inserting, updating, and querying array values.
- Aggregate functions – `array_agg` for building arrays from query results.
- Query optimisation – GIN indexes for array containment and overlap queries.
- Data modelling – arrays versus junction tables versus `JSONB` for multi-valued attributes.

---

## Core Concepts and Features

### 1. Arrays: Creation and Declaration

#### Definitions

**Core Definition**
Array creation is the process of declaring an array-typed column, constructing array literals, and inserting array values into a table.

**Technical Definition**
An array data type is named by appending square brackets (`[]`) to the data type name of the array elements. For example, `integer[]` is a one-dimensional integer array, and `text[][]` is a two-dimensional text array. The SQL standard keyword `ARRAY` can also be used for one-dimensional arrays (e.g., `integer ARRAY[4]`). Array literals are written by enclosing element values within curly braces and separating them with commas: `'{val1,val2,...}'`. Multidimensional arrays nest curly braces: `'{{1,2,3},{4,5,6},{7,8,9}}'`. To set an element to `NULL`, write `NULL`; to store the literal string `"NULL"`, enclose it in double quotes. The `ARRAY` constructor syntax (`ARRAY[1,2,3]`) is the modern, unambiguous alternative to curly-brace literals.

**Beginner-Friendly Explanation**
Creating an array column is like telling the database “this column holds a list of values, not a single value.” You declare it by adding `[]` after the element type. You can then insert lists using either curly-brace literals (`'{1,2,3}'`) or the `ARRAY` constructor (`ARRAY[1,2,3]`), and PostgreSQL will store the whole list in one cell.

#### Purposes

- To store a small, tightly coupled collection of values in a single column without a separate child table.
- To model attributes that are always read and written as a unit (e.g., a product's tags, a user's roles).
- To support multidimensional data such as matrices or schedules.
- To provide a compact representation for values that are frequently retrieved together.

#### Syntax Rules and Structure

**Column declaration (square-bracket syntax):**

```sql
column_name data_type[]
column_name data_type[][]
column_name data_type[size]  -- size is documentation only; not enforced
```

**Column declaration (SQL-standard ARRAY syntax):**

```sql
column_name data_type ARRAY
column_name data_type ARRAY[size]  -- size is documentation only; not enforced
```

**Array literal (curly braces):**

```sql
'{val1,val2,val3}'
'{{1,2,3},{4,5,6}}'  -- 2D array
```

**ARRAY constructor:**

```sql
ARRAY[1, 2, 3]
ARRAY[[1,2,3],[4,5,6]]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `data_type[]` | Appends `[]` to the element type to declare an array. |
| `ARRAY` | SQL-standard keyword for one-dimensional arrays. |
| `'{...}'` | Curly-brace literal; elements separated by commas. |
| `ARRAY[...]` | Constructor syntax; unambiguous for expressions and nested arrays. |
| `NULL` | Unquoted `NULL` inside a literal represents a null element. |

**Syntax rules:**

- Array size and dimensionality in `CREATE TABLE` are documentation only and are not enforced.
- Curly-brace literals must be quoted as strings; the `ARRAY` constructor does not require quoting.
- In multidimensional arrays, all sub-arrays at the same level must have matching dimensions.

**Constraints and limitations:**

- Arrays of the same element type are considered the same type regardless of size or number of dimensions.
- `NULL` elements are allowed; use `IS NOT DISTINCT FROM` semantics when searching for them.
- Multidimensional arrays must be rectangular (no ragged arrays).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Declaring and populating array columns

```sql
-- Step 1: Create a table with one-dimensional and two-dimensional arrays.
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[],
    schedule        text[][]
);

-- Step 2: Insert rows using curly-brace literals.
INSERT INTO sal_emp VALUES (
    'Bill',
    '{10000, 10000, 10000, 10000}',
    '{{"meeting", "lunch"}, {"training", "presentation"}}'
);
INSERT INTO sal_emp VALUES (
    'Carol',
    '{20000, 25000, 25000, 25000}',
    '{{"breakfast", "consulting"}, {"meeting", "lunch"}}'
);

-- Step 3: Insert using the ARRAY constructor.
INSERT INTO sal_emp VALUES (
    'Dave',
    ARRAY[30000, 30000, 30000, 30000],
    ARRAY[['breakfast', 'consulting'], ['meeting', 'lunch']]
);

-- Step 4: Query the data.
SELECT name, pay_by_quarter FROM sal_emp ORDER BY name;
```

**Expected output:**

```text
 name  |     pay_by_quarter
-------+-------------------------
 Bill  | {10000,10000,10000,10000}
 Carol | {20000,25000,25000,25000}
 Dave  | {30000,30000,30000,30000}
(3 rows)
```

**Why this output appears**: The `integer[]` and `text[][]` declarations create array columns. The curly-brace literals and `ARRAY` constructors both produce the same array values. PostgreSQL renders arrays in curly-brace notation on output, with all rows present.

#### Real-World Cases

- **Product tags**: A `products` table stores `tags text[]` instead of a separate `product_tags` junction table when tags are a small, fixed-size set always read with the product.
- **Weekly schedules**: A `staff` table stores `work_days text[]` to represent which days of the week an employee works, avoiding seven boolean columns.
- **Matrix data**: A scientific table stores `measurements numeric[][]` for a two-dimensional grid of sensor readings.

---

### 2. Array Operations (Operators)

#### Definitions

**Core Definition**
Array operations are the operators that compare, combine, and test arrays, including containment (`@>`), reverse containment (`<@`), overlap (`&&`), and concatenation (`||`).

**Technical Definition**
PostgreSQL provides specialized operators for array types. The containment operators `@>` and `<@` test whether one array contains or is contained by another; `&&` tests whether two arrays have any elements in common; `||` concatenates two arrays or an array and an element. The usual comparison operators (`=`, `<>`, `<`, `>`, `<=`, `>=`) are also available and compare array contents element-by-element using the default B-tree comparison function for the element type. In multidimensional arrays, elements are visited in row-major order (last subscript varies most rapidly), and if two arrays have equal contents but different dimensionality, the first difference in dimensionality information determines the sort order.

**Beginner-Friendly Explanation**
Array operators let you ask questions like “does this list contain that item?” (`@>`), “do these two lists share any items?” (`&&`), and “what happens if I stick these two lists together?” (`||`). They are the tools for filtering and combining array values without unpacking them into rows first.

#### Purposes

- To filter rows based on whether an array contains a specific value or set of values.
- To test whether two arrays share any elements.
- To combine arrays or add elements to arrays in a single expression.
- To compare and sort arrays using their element values.

#### Syntax Rules and Structure

| Operator | Description | Example |
|----------|-------------|---------|
| `@>` | First array contains the second | `ARRAY[1,4,3] @> ARRAY[3,1,3]` → `t` |
| `<@` | First array is contained by the second | `ARRAY[2,2,7] <@ ARRAY[1,7,4,2,6]` → `t` |
| `&&` | Arrays overlap (share any element) | `ARRAY[1,4,3] && ARRAY[2,1]` → `t` |
| `\|\|` | Concatenate two arrays or an array and an element | `ARRAY[1,2,3] \|\| ARRAY[4,5,6]` → `{1,2,3,4,5,6}` |
| `=`, `<>`, `<`, `>`, `<=`, `>=` | Element-wise comparison | `ARRAY[1,2,3] = ARRAY[1,2,3]` → `t` |

**Syntax rules:**

- Containment (`@>`, `<@`) does not treat duplicates specially: `ARRAY[1]` and `ARRAY[1,1]` each contain the other.
- Concatenation with a `NULL` or empty array is a no-op; otherwise the arrays must have the same number of dimensions or differ by one.
- Element types are coerced to a common type for concatenation.

**Constraints and limitations:**

- `@>` and `&&` can use GIN indexes; the standard comparison operators cannot.
- Containment semantics do not consider element order or multiplicity.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Containment, overlap, and concatenation

```sql
-- Step 1: Create a table with a text array column.
CREATE TABLE products (id int, tags text[]);
INSERT INTO products VALUES
    (1, ARRAY['electronics', 'sale']),
    (2, ARRAY['electronics', 'new']),
    (3, ARRAY['clothing', 'sale']);

-- Step 2: Containment: products containing 'electronics'.
SELECT id FROM products WHERE tags @> ARRAY['electronics'];

-- Step 3: Overlap: products sharing any tag with ['sale', 'clearance'].
SELECT id FROM products WHERE tags && ARRAY['sale', 'clearance'];

-- Step 4: Concatenation: add 'featured' to each product's tags.
SELECT id, tags || ARRAY['featured'] AS updated_tags FROM products WHERE id = 1;
```

**Expected output:**

```text
 id
----
  1
  2
(2 rows)

 id
----
  1
  3
(2 rows)

    updated_tags
-----------------------
 {electronics,sale,featured}
(1 row)
```

**Why this output appears**: `@>` returns the rows whose `tags` array contains `'electronics'` (products 1 and 2). `&&` returns the rows that share at least one tag with `['sale', 'clearance']` (products 1 and 3). `||` appends `'featured'` to the array, producing the combined array.

#### Real-World Cases

- **Content filtering**: Filtering articles whose `topics` array contains a requested topic, using `@>` with a GIN index.
- **Recommendation overlap**: Finding users whose `interests` array shares any element with a target user's interests, using `&&`.
- **Tag aggregation**: Appending a new tag to a product's `tags` array with `||` without reading and rewriting the whole array in application code.

---

### 3. `array_agg()`

#### Definitions

**Core Definition**
`array_agg()` is an aggregate function that collects all input values, including nulls, into an array.

**Technical Definition**
The PostgreSQL documentation defines two forms: `array_agg(anynonarray ORDER BY input_sort_columns) → anyarray` collects all input values, including nulls, into an array; and `array_agg(anyarray ORDER BY input_sort_columns) → anyarray` concatenates all input arrays into an array of one higher dimension, where the inputs must all have the same dimensionality and cannot be empty or null. An optional `ORDER BY` clause controls the order of elements in the resulting array.

**Beginner-Friendly Explanation**
`array_agg()` is the “collect rows into a list” function. If you have a table of comments and you want one row per post with an array of all its comment IDs, `array_agg` does that. It is the inverse of `unnest()`.

#### Purposes

- To aggregate multiple rows into a single array for denormalised output or API responses.
- To collect related values into a compact representation for further processing.
- To concatenate arrays from multiple rows into a higher-dimensional array.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_agg(expression [ ORDER BY sort_expression [, ...] ])
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to aggregate; can be a scalar or an array. |
| `ORDER BY` | Optional; controls element order in the result. |

**Syntax rules:**

- `array_agg` is an aggregate function and is used with `GROUP BY` or as a window function.
- The scalar form includes `NULL` values in the array; the array form concatenates arrays into one higher dimension.
- The array form requires all input arrays to have the same dimensionality and to be non-null and non-empty.

**Constraints and limitations:**

- The resulting array is 1-indexed and has no declared size limit.
- Very large aggregations can consume significant memory.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Aggregating values into an array

```sql
-- Step 1: Create a table with related rows.
CREATE TABLE post_comments (
    post_id int,
    comment_id int,
    body text
);
INSERT INTO post_comments VALUES
    (1, 100, 'First!'),
    (1, 101, 'Nice post'),
    (2, 200, 'Interesting');

-- Step 2: Aggregate comment IDs per post, ordered by comment_id.
SELECT post_id, array_agg(comment_id ORDER BY comment_id) AS comment_ids
FROM post_comments
GROUP BY post_id
ORDER BY post_id;
```

**Expected output:**

```text
 post_id | comment_ids
---------+-------------
       1 | {100,101}
       2 | {200}
(2 rows)
```

**Why this output appears**: `array_agg(comment_id ORDER BY comment_id)` groups the rows by `post_id` and collects each group's `comment_id` values into an array, ordered by `comment_id`. The result has one row per post with an array of its comment IDs.

#### Real-World Cases

- **API responses**: Building a JSON-friendly array of IDs or names for each parent record.
- **Denormalisation**: Storing a snapshot of related IDs in an array column for read-heavy use cases.
- **Batch processing**: Collecting a set of IDs into an array for a subsequent `WHERE id = ANY(array)` query.

---

### 4. `array_append()`

#### Definitions

**Core Definition**
`array_append()` appends a single element to the end of an array and returns the resulting array.

**Technical Definition**
The function signature is `array_append(anycompatiblearray, anycompatible) → anycompatiblearray`. It is equivalent to the `||` operator when the right-hand operand is a scalar element. PostgreSQL documents the example `array_append(ARRAY[1,2], 3) → {1,2,3}`.

**Beginner-Friendly Explanation**
`array_append` adds an item to the end of a list. It is the functional equivalent of the `||` operator with an element on the right.

#### Purposes

- To add an element to the end of an array without rewriting the array manually.
- To build up an array incrementally in queries or procedural code.
- To provide a functional alternative to the `||` operator for readability.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_append(array, element)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The input array (`anycompatiblearray`). |
| `element` | The value to append (`anycompatible`). |

**Syntax rules:**

- The element type must be compatible with the array's element type.
- The array must be one-dimensional or empty for the result to be well-defined.

**Constraints and limitations:**

- Appending to a multidimensional array is not supported in the same way; use `||` with another array instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Appending an element

```sql
-- Step 1: Append an element to an array.
SELECT array_append(ARRAY[1, 2], 3) AS result;
```

**Expected output:**

```text
 result
--------
 {1,2,3}
(1 row)
```

**Why this output appears**: `array_append(ARRAY[1,2], 3)` adds `3` to the end of `{1,2}`, producing `{1,2,3}`.

#### Real-World Cases

- **Building arrays in PL/pgSQL**: Appending to an array inside a loop to accumulate results.
- **Readable expressions**: Using `array_append` instead of `array || element` for clarity in complex queries.

---

### 5. `array_prepend()`

#### Definitions

**Core Definition**
`array_prepend()` prepends a single element to the beginning of an array and returns the resulting array.

**Technical Definition**
The function signature is `array_prepend(anycompatible, anycompatiblearray) → anycompatiblearray`. It is equivalent to the `||` operator when the left-hand operand is a scalar element. PostgreSQL documents the example `array_prepend(1, ARRAY[2,3]) → {1,2,3}`.

**Beginner-Friendly Explanation**
`array_prepend` adds an item to the front of a list. It is the functional equivalent of `element || array`.

#### Purposes

- To add an element to the beginning of an array.
- To build up an array in reverse order or with priority elements first.
- To provide a functional alternative to the `||` operator.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_prepend(element, array)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `element` | The value to prepend (`anycompatible`). |
| `array` | The input array (`anycompatiblearray`). |

**Syntax rules:**

- The element type must be compatible with the array's element type.
- The array must be one-dimensional or empty.

**Constraints and limitations:**

- Prepending to a multidimensional array may not produce the expected shape; use `||` with another array instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Prepending an element

```sql
-- Step 1: Prepend an element to an array.
SELECT array_prepend(1, ARRAY[2, 3]) AS result;
```

**Expected output:**

```text
 result
--------
 {1,2,3}
(1 row)
```

**Why this output appears**: `array_prepend(1, ARRAY[2,3])` adds `1` to the front of `{2,3}`, producing `{1,2,3}`.

#### Real-World Cases

- **Priority ordering**: Prepending a high-priority item to a list of tasks.
- **Reverse accumulation**: Building an array where later elements appear first.

---

### 6. `array_cat()`

#### Definitions

**Core Definition**
`array_cat()` concatenates two arrays into a single array.

**Technical Definition**
The function signature is `array_cat(anycompatiblearray, anycompatiblearray) → anycompatiblearray`. It is equivalent to the `||` operator when both operands are arrays. PostgreSQL documents the example `array_cat(ARRAY[1,2,3], ARRAY[4,5]) → {1,2,3,4,5}`. Concatenating with `NULL` or an empty array is a no-op; otherwise the arrays must have the same number of dimensions or differ by one, and element types are coerced to a common type.

**Beginner-Friendly Explanation**
`array_cat` joins two lists together into one longer list. It is the functional equivalent of `array1 || array2`.

#### Purposes

- To combine two arrays into a single array.
- To merge a stored array with a newly computed array.
- To provide a functional alternative to the `||` operator for array-array concatenation.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_cat(array1, array2)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array1` | The first array (`anycompatiblearray`). |
| `array2` | The second array (`anycompatiblearray`). |

**Syntax rules:**

- If the arrays are not of identical element types, they are coerced to a common type.
- Concatenating a `NULL` or empty array is a no-op.
- Multidimensional arrays must have compatible dimensionality (same number of dimensions or differ by one).

**Constraints and limitations:**

- Concatenating a multidimensional array with a one-dimensional array produces a higher-dimensional result in some cases, depending on the shape.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Concatenating two arrays

```sql
-- Step 1: Concatenate two arrays.
SELECT array_cat(ARRAY[1, 2, 3], ARRAY[4, 5]) AS result;
```

**Expected output:**

```text
 result
-----------
 {1,2,3,4,5}
(1 row)
```

**Why this output appears**: `array_cat` appends the elements of the second array to the first, producing `{1,2,3,4,5}`.

#### Real-World Cases

- **Merging tag lists**: Combining an existing tag array with a new set of tags in an update.
- **Batch accumulation**: Concatenating per-batch arrays into a single result array.

---

### 7. `array_length()`

#### Definitions

**Core Definition**
`array_length()` returns the length of a specified dimension of an array, or `NULL` if the dimension is empty or does not exist.

**Technical Definition**
The function signature is `array_length(anyarray, integer) → integer`. It returns the length of the requested array dimension. The documentation notes that it produces `NULL` instead of `0` for empty or missing array dimensions. For example, `array_length(array[1,2,3], 1) → 3`, `array_length(array[]::int[], 1) → NULL`, and `array_length(array['text'], 2) → NULL`.

**Beginner-Friendly Explanation**
`array_length` tells you how many elements are in a particular dimension of an array. You give it the array and the dimension number (1 for the first dimension, 2 for the second, etc.), and it returns the length—or `NULL` if that dimension does not exist or is empty.

#### Purposes

- To determine the number of elements in an array dimension.
- To validate array sizes before processing.
- To drive loops or conditional logic based on array length.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_length(array, dimension)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The input array (`anyarray`). |
| `dimension` | The dimension number (1-based). |

**Syntax rules:**

- Returns `NULL` for empty or missing dimensions, not `0`.
- Use `cardinality(array)` to get the total number of elements across all dimensions.

**Constraints and limitations:**

- For multidimensional arrays, you must specify the correct dimension number.
- Empty arrays return `NULL` for any dimension.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Array length of different dimensions

```sql
-- Step 1: Check lengths of a one-dimensional and a two-dimensional array.
SELECT array_length(ARRAY[1,2,3], 1) AS len_1d,
       array_length(ARRAY[[1,2,3],[4,5,6]], 1) AS len_2d_dim1,
       array_length(ARRAY[[1,2,3],[4,5,6]], 2) AS len_2d_dim2,
       array_length(ARRAY[]::int[], 1) AS len_empty;
```

**Expected output:**

```text
 len_1d | len_2d_dim1 | len_2d_dim2 | len_empty
--------+-------------+-------------+-----------
      3 |           2 |           3 |
(1 row)
```

**Why this output appears**: `array_length(ARRAY[1,2,3], 1)` returns 3. For the 2×3 array, dimension 1 has 2 rows and dimension 2 has 3 columns. The empty array returns `NULL`, as documented.

#### Real-World Cases

- **Validation**: Checking that an array has the expected number of elements before processing.
- **Loop bounds**: Using `array_length` to control a `FOR` loop over array subscripts.
- **Reporting**: Including array cardinality in query output.

---

### 8. `array_position()`

#### Definitions

**Core Definition**
`array_position()` returns the subscript of the first occurrence of a value in a one-dimensional array, or `NULL` if the value is not present.

**Technical Definition**
The function signature is `array_position(anycompatiblearray, anycompatible [, integer]) → integer`. It returns the subscript of the first occurrence of the second argument in the array, or `NULL` if it is not present. If the third argument is given, the search begins at that subscript. The array must be one-dimensional. Comparisons use `IS NOT DISTINCT FROM` semantics, so it is possible to search for `NULL`. PostgreSQL documents the example `array_position(ARRAY['sun','mon','tue','wed','thu','fri','sat'], 'mon') → 2`.

**Beginner-Friendly Explanation**
`array_position` finds where a value appears in a list. You give it an array and a value, and it returns the position of the first match (starting from 1), or `NULL` if the value is not found.

#### Purposes

- To locate the position of a value within an array.
- To check for membership and retrieve the index in one operation.
- To support logic that depends on where an element appears.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_position(array, value [, start])
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The input array (`anycompatiblearray`); must be one-dimensional. |
| `value` | The value to search for (`anycompatible`). |
| `start` | Optional starting subscript for the search. |

**Syntax rules:**

- Returns `NULL` if the value is not found.
- Uses `IS NOT DISTINCT FROM`, so `NULL` can be searched for.
- The array must be one-dimensional.

**Constraints and limitations:**

- Multidimensional arrays are not supported.
- Only the first occurrence is returned; use `array_positions()` for all occurrences.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Finding an element's position

```sql
-- Step 1: Find the position of 'mon' in an array of days.
SELECT array_position(ARRAY['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'], 'mon') AS pos;

-- Step 2: Search for a NULL value.
SELECT array_position(ARRAY[1, NULL, 3], NULL) AS null_pos;

-- Step 3: Search for a missing value.
SELECT array_position(ARRAY[1, 2, 3], 5) AS missing_pos;
```

**Expected output:**

```text
 pos
-----
   2
(1 row)
 null_pos
----------
        2
(1 row)
 missing_pos
-------------
(1 row)
```

**Why this output appears**: `'mon'` is at position 2. `NULL` is at position 2 and is found because `IS NOT DISTINCT FROM` is used. `5` is not present, so the result is `NULL` (displayed as an empty cell).

#### Real-World Cases

- **Membership testing with index**: Finding whether a user's role appears in a roles array and which position it occupies.
- **Array search**: Locating a specific value in a stored list without unnesting.

---

### 9. `array_remove()`

#### Definitions

**Core Definition**
`array_remove()` removes all elements equal to a given value from a one-dimensional array.

**Technical Definition**
The function signature is `array_remove(anycompatiblearray, anycompatible) → anycompatiblearray`. It removes all elements equal to the given value from the array. The array must be one-dimensional. Comparisons use `IS NOT DISTINCT FROM` semantics, so it is possible to remove `NULL` values. PostgreSQL documents the example `array_remove(ARRAY[1,2,3,2], 2) → {1,3}`.

**Beginner-Friendly Explanation**
`array_remove` takes a list and a value and returns a new list with every occurrence of that value removed.

#### Purposes

- To remove all occurrences of a specific value from an array.
- To clean up arrays by removing unwanted or invalid entries.
- To support updates that need to delete a value from a stored list.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_remove(array, value)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The input array (`anycompatiblearray`); must be one-dimensional. |
| `value` | The value to remove (`anycompatible`). |

**Syntax rules:**

- Removes all occurrences, not just the first.
- Uses `IS NOT DISTINCT FROM`, so `NULL` can be removed.
- The array must be one-dimensional.

**Constraints and limitations:**

- Multidimensional arrays are not supported.
- The result may be empty if all elements equal the removed value.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Removing elements from an array

```sql
-- Step 1: Remove all occurrences of 2.
SELECT array_remove(ARRAY[1, 2, 3, 2], 2) AS result;

-- Step 2: Remove NULL values.
SELECT array_remove(ARRAY[1, NULL, 3, NULL], NULL) AS result_no_nulls;
```

**Expected output:**

```text
 result
--------
 {1,3}
(1 row)
 result_no_nulls
-----------------
 {1,3}
(1 row)
```

**Why this output appears**: `array_remove(ARRAY[1,2,3,2], 2)` removes both `2` values, leaving `{1,3}`. `array_remove(ARRAY[1,NULL,3,NULL], NULL)` removes both `NULL` values, leaving `{1,3}`.

#### Real-World Cases

- **Tag removal**: Removing a specific tag from a product's `tags` array.
- **Data cleaning**: Removing `NULL` or sentinel values from stored arrays.

---

### 10. `array_replace()`

#### Definitions

**Core Definition**
`array_replace()` replaces each array element equal to a specified value with a new value.

**Technical Definition**
The function signature is `array_replace(anycompatiblearray, anycompatible, anycompatible) → anycompatiblearray`. It replaces each array element equal to the second argument with the third argument. PostgreSQL documents the example `array_replace(ARRAY[1,2,5,4], 5, 3) → {1,2,3,4}`.

**Beginner-Friendly Explanation**
`array_replace` is a find-and-replace for arrays. It finds every occurrence of a value and swaps it for a new value.

#### Purposes

- To replace all occurrences of a value in an array.
- To update stored arrays when a value is renamed or recoded.
- To perform bulk substitutions without unnesting and re-aggregating.

#### Syntax Rules and Structure

**General syntax:**

```sql
array_replace(array, old_value, new_value)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The input array (`anycompatiblearray`). |
| `old_value` | The value to replace (`anycompatible`). |
| `new_value` | The replacement value (`anycompatible`). |

**Syntax rules:**

- All occurrences of `old_value` are replaced.
- The array must be one-dimensional for predictable results.

**Constraints and limitations:**

- Multidimensional arrays may produce unexpected results; prefer one-dimensional arrays.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Replacing an element

```sql
-- Step 1: Replace 5 with 3 in an array.
SELECT array_replace(ARRAY[1, 2, 5, 4], 5, 3) AS result;
```

**Expected output:**

```text
 result
-----------
 {1,2,3,4}
(1 row)
```

**Why this output appears**: `array_replace` finds `5` at position 3 and replaces it with `3`, producing `{1,2,3,4}`.

#### Real-World Cases

- **Value migration**: Replacing a deprecated status code with a new one in stored arrays.
- **Tag renaming**: Updating a tag name across all rows that store it in an array.

---

### 11. `unnest()`

#### Definitions

**Core Definition**
`unnest()` expands an array into a set of rows, one row per array element.

**Technical Definition**
The function signature is `unnest(anyarray) → setof anyelement`. It expands an array into a set of rows; the array's elements are read out in storage order. A multi-array form `unnest(anyarray, anyarray [, ...]) → setof anyelement, anyelement [, ...]` expands multiple arrays into a set of rows, padding shorter arrays with `NULL`s, and is allowed only in a query's `FROM` clause. PostgreSQL documents the examples `unnest(ARRAY[1,2]) → 1 2` and `unnest(ARRAY[['foo','bar'],['baz','quux']]) → foo bar baz quux`.

**Beginner-Friendly Explanation**
`unnest()` is the inverse of `array_agg()`. It takes an array and turns it into a normal table of rows, so you can use standard SQL operations—joins, filters, aggregates—on the individual elements.

#### Purposes

- To expand an array into rows for joining, filtering, or aggregating.
- To normalize array data for downstream processing.
- To avoid positional access and `generate_subscripts` by working set-wise instead.

#### Syntax Rules and Structure

**General syntax:**

```sql
unnest(array)  -- in SELECT list
unnest(array1, array2, ...)  -- in FROM clause
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `array` | The array to expand (`anyarray`). |
| Multi-array form | Expands multiple arrays in parallel, padding shorter arrays with `NULL`. |

**Syntax rules:**

- The single-array form can appear in the `SELECT` list or `FROM` clause.
- The multi-array form is allowed only in the `FROM` clause.
- Elements are returned in storage order.

**Constraints and limitations:**

- Multidimensional arrays are expanded into a single column of elements in row-major order, losing the dimensional structure.
- The multi-array form requires all arrays to have compatible element counts; shorter arrays are padded with `NULL`.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Expanding an array into rows

```sql
-- Step 1: Expand a simple array.
SELECT unnest(ARRAY[1, 2, 3]) AS element;

-- Step 2: Expand a multidimensional array.
SELECT unnest(ARRAY[['foo','bar'],['baz','quux']]) AS element;

-- Step 3: Expand two arrays in parallel in the FROM clause.
SELECT * FROM unnest(ARRAY[1,2], ARRAY['foo','bar','baz']) AS x(a, b);
```

**Expected output:**

```text
 element
---------
       1
       2
       3
(3 rows)
 element
---------
 foo
 bar
 baz
 quux
(4 rows)
 a |   b
---+-------
 1 | foo
 2 | bar
   | baz
(3 rows)
```

**Why this output appears**: The single-array form produces one row per element. The multidimensional array is flattened in row-major order. The multi-array form pairs elements by position and pads the shorter array (`ARRAY[1,2]`) with `NULL` for the third row.

#### Real-World Cases

- **Joining array elements**: Expanding a `tags` array to join against a `tags` table and filter by tag metadata.
- **Aggregating elements**: Unnesting an array, applying `count()` or `sum()`, and re-aggregating the results.
- **Avoiding positional loops**: Replacing `generate_subscripts` or `array_length` loops with set-based `unnest()` operations for better performance.

---

### 12. Array Best Practices

#### Definitions

**Core Definition**
Array best practices are the guidelines for deciding when to use arrays, how to index them, how to query them efficiently, and when to prefer normalized child tables or `JSONB` instead.

**Technical Definition**
Best practices include: (1) **use arrays for small, tightly coupled collections** that are always read and written together; (2) **use GIN indexes for containment and overlap queries** (`@>`, `&&`), not for equality or `ANY` tests; (3) **avoid searching inside arrays without a GIN index**—performance degrades significantly on large unindexed arrays; (4) **prefer `unnest()` over positional access** in queries to work set-wise rather than element-by-element; (5) **consider a junction table when individual elements are filtered, joined, or aggregated heavily**, because arrays are not a replacement for relational design in those cases; and (6) **be aware that GIN indexes increase write overhead** compared with B-tree indexes.

**Beginner-Friendly Explanation**
Arrays are great for small lists that travel together—like a product's tags or a user's roles. But they are not a silver bullet. If you need to search inside them often, add a GIN index. If you need to join, filter, or aggregate on individual elements heavily, use a proper child table instead. And avoid accessing array elements by position in loops; use `unnest()` to turn the array into rows and let the database do set-based work.

#### Purposes

- To choose arrays appropriately and avoid the performance pitfalls of misusing them.
- To index array columns correctly for containment and overlap queries.
- To write efficient array queries that leverage set-based operations.
- To know when to refactor an array into a junction table.

#### Syntax Rules and Structure

**GIN index on an array column:**

```sql
CREATE INDEX idx_products_tags ON products USING GIN (tags);
```

**Containment query (uses GIN index):**

```sql
SELECT * FROM products WHERE tags @> ARRAY['electronics'];
```

**Overlap query (uses GIN index):**

```sql
SELECT * FROM products WHERE tags && ARRAY['electronics', 'sale'];
```

**Set-based expansion instead of positional loops:**

```sql
SELECT p.id, tag
FROM products p,
LATERAL unnest(p.tags) AS tag
WHERE tag = 'electronics';
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Index for containment | `CREATE INDEX ... USING GIN (array_column)` |
| Indexable operators | `@>`, `<@`, `&&` |
| Non-indexable patterns | `= ANY(array)`, positional access in loops |
| Set-based expansion | `unnest()` in `FROM` or `LATERAL` |

**Syntax rules:**

- GIN indexes support `@>`, `<@`, and `&&`; they do not support `= ANY(array)` or equality comparisons.
- `unnest()` is the preferred way to expand arrays into rows for joins and aggregates.

**Constraints and limitations:**

- GIN indexes incur higher write overhead than B-tree indexes.
- `= ANY(array)` expands to equality and typically forces a sequential scan even with a GIN index.
- Arrays with heavy element-level filtering and joining are better modelled as junction tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – GIN index for array containment and overlap

```sql
-- Step 1: Create a table with an array column and a GIN index.
CREATE TABLE products (
    id   int PRIMARY KEY,
    tags text[]
);
CREATE INDEX idx_products_tags ON products USING GIN (tags);

-- Step 2: Insert rows.
INSERT INTO products VALUES
    (1, ARRAY['electronics', 'sale']),
    (2, ARRAY['electronics', 'new']),
    (3, ARRAY['clothing', 'sale']);

-- Step 3: Query using containment (indexable).
SELECT id FROM products WHERE tags @> ARRAY['electronics'];

-- Step 4: Query using overlap (indexable).
SELECT id FROM products WHERE tags && ARRAY['sale', 'clearance'];
```

**Expected output:**

```text
 id
----
  1
  2
(2 rows)

 id
----
  1
  3
(2 rows)
```

**Why this output appears**: The GIN index on `tags` accelerates the `@>` and `&&` queries, quickly locating the rows whose arrays contain `'electronics'` or share at least one tag with `['sale', 'clearance']`.

##### Example 2: PostgreSQL – `unnest()` versus positional access

```sql
-- Step 1: Create a table with an array column.
CREATE TABLE items (id int, tags text[]);
INSERT INTO items VALUES (1, ARRAY['a','b','c']), (2, ARRAY['b','d']);

-- Step 2: Expand with unnest() and filter.
SELECT id, tag
FROM items, LATERAL unnest(tags) AS tag
WHERE tag = 'b';
```

**Expected output:**

```text
 id | tag
----+-----
  1 | b
  2 | b
(2 rows)
```

**Why this output appears**: `unnest(tags)` expands each array into rows, and the `WHERE tag = 'b'` filter keeps only the elements equal to `'b'`. This set-based approach is more efficient than iterating over positions with `array_length` and `generate_subscripts`.

#### Real-World Cases

- **Product tags**: A `products` table uses a `tags text[]` array with a GIN index for containment queries, avoiding a junction table for a small, read-heavy tag set.
- **User roles**: A `users` table stores `roles text[]` and uses `&&` to check whether a user has any of the required roles for access control.
- **When to refactor**: If queries frequently join tags to a `tag_metadata` table or aggregate counts per tag, the array should be replaced by a `product_tags` junction table for better join performance and relational integrity.

---

## References

- PostgreSQL Documentation – Arrays – https://www.postgresql.org/docs/current/arrays.html
- PostgreSQL Documentation – Array Functions and Operators – https://www.postgresql.org/docs/current/functions-array.html
- PostgreSQL Documentation – Aggregate Functions (array_agg) – https://www.postgresql.org/docs/current/functions-aggregate.html
- PostgreSQL Documentation – GIN Indexes – https://www.postgresql.org/docs/current/gin.html
- PostgreSQL Documentation – Index Types (GIN for arrays) – https://www.postgresql.org/docs/current/indexes-types.html
- PostgreSQL Wiki – Arrays – https://wiki.postgresql.org/wiki/Arrays
- Tiger Data – Optimizing Array Queries With GIN Indexes in PostgreSQL – https://www.tigerdata.com/blog/optimizing-array-queries-with-gin-indexes-in-postgresql
- Heap – Using PostgreSQL Arrays The Right Way – https://www.heap.io/blog/using-postgresql-arrays-the-right-way
- PostgreSQL Mailing List – Arrays vs separate tables – https://www.postgresql.org/message-id/flat/...
- The Art of PostgreSQL, Vol. 1 – Ch 23: Arrays, Composite Types, Enums – https://github.com/BrewHubPHL/supabase-specialist/blob/master/references/book-summaries/taop-vol1-ch23-arrays-composite-enum.md