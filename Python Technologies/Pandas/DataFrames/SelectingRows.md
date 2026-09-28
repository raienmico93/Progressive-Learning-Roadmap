# Comprehensive Programming Cheat Sheet: Selecting Rows in Pandas DataFrames

---

## Topic Overview

### Definitions

**Core Definition:** Selecting rows in a pandas DataFrame refers to the set of operations for extracting, filtering, and reindexing row-level records using label-based indexing (`.loc`), position-based indexing (`.iloc`), boolean masks, slicing, conditional querying (`.query()`, `.isin()`), and index manipulation (`.reset_index()`, `.set_index()`).

**Technical Definition:** Row selection in pandas is performed through two primary indexer accessors: `.loc` (strictly label-based, inclusive-stop slicing) and `.iloc` (strictly integer-position-based, exclusive-stop slicing). Boolean masks filter rows by evaluating element-wise conditions. The `.query()` method evaluates string expressions using `numexpr` when available, and `.isin()` filters rows whose values are contained in an iterable. Index resets and sets transition between row labels and data columns via `.reset_index()` and `.set_index()`.

**Beginner-Friendly Explanation:** Rows are the horizontal records in your table. You can pick rows by their label (`.loc`), by their position number (`.iloc`), by a condition (boolean masks, `.query()`, `.isin()`), or by slicing. You can also move the row labels into a column or make a column the new row labels. These tools let you focus on exactly the records you need.

### Key Characteristics

- **Dual indexing systems:** `.loc` uses labels; `.iloc` uses integer positions.
- **Slice semantics differ:** `.loc` slices are inclusive of the stop; `.iloc` slices are exclusive.
- **Boolean masking:** Conditions produce boolean arrays used to filter rows.
- **Query optimization:** `.query()` uses `numexpr` for fast evaluation of string expressions.
- **Index flexibility:** `.reset_index()` and `.set_index()` convert between row labels and columns.
- **Copy-on-Write:** Under pandas 3.0, row selections behave as copies for modification purposes.

### Prerequisites

- Basic Python syntax (lists, dictionaries, functions).
- Pandas Series and DataFrame construction.
- Understanding of DataFrame indexing (`.loc`, `.iloc`).
- Familiarity with boolean logic and comparison operators.

### Related Programming Areas

- **Exploratory Data Analysis:** Filtering and subsetting data.
- **Data Cleaning:** Removing outliers and invalid records.
- **Feature Engineering:** Selecting rows for training/test splits.
- **Time Series Analysis:** Date-range slicing with `.loc`.
- **ETL Pipelines:** Row-level filtering and index management.

### Core Concepts / Features

1. .loc (Label-Based Selection)
2. .iloc (Position-Based Selection)
3. Boolean Masks
4. Slicing
5. Conditional Row Selection (.query() and .isin())
6. Index Resets (.reset_index() and .set_index())

---

## Core Concept 1: .loc (Label-Based Selection)

### Definitions

**Core Definition:** `.loc` is a strictly label-based indexer that selects rows (and optionally columns) by their index labels, supporting single labels, lists of labels, label slices, and boolean arrays.

**Technical Definition:** `DataFrame.loc[row_indexer, column_indexer]` selects data using labels. For a single label, it returns a Series (row) or scalar; for a list of labels, it returns a DataFrame; for a label slice (`start:stop`), it returns a DataFrame including both endpoints if they exist in the index. Missing labels raise `KeyError`. `.loc` never falls back to positional access, even when the index contains integers.

**Beginner-Friendly Explanation:** `.loc` lets you pick rows by their label—the name or number used in the index. If your DataFrame has a "name" index, you can select the row for "Alice" with `df.loc['Alice']`. You can also select multiple rows by passing a list or a slice.

### Purposes

- To select rows by meaningful labels (dates, IDs, names).
- To select multiple rows by a list of labels.
- To perform label-based slicing with inclusive endpoints.
- To select paired row-and-column intersections (`df.loc[row, col]`).
- To apply boolean masks with automatic index alignment.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single row
df.loc[row_label]

# Multiple rows by list
df.loc[[label1, label2]]

# Label slice (inclusive)
df.loc[start_label:stop_label]

# Row-and-column intersection
df.loc[row_label, col_label]

# Boolean mask
df.loc[boolean_array]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `row_label` | Index label | Yes |
| `[label1, label2]` | List of index labels | Yes (for multiple) |
| `start:stop` | Label slice (inclusive) | Yes |
| `col_label` | Column label | No |

#### Syntax Rules

- `.loc` is strictly label-based; missing labels raise `KeyError`.
- Label slices include both endpoints if present.
- A single integer passed to `.loc` is interpreted as a **label**, not a position.
- Boolean arrays passed to `.loc` are aligned by index before masking.

#### Constraints and Limitations

- Non-unique index labels may return multiple rows.
- Requires the index to be sorted for reliable slice behavior in some versions.
- Cannot perform positional lookups.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single and Multiple Row Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with custom index
df = pd.DataFrame(
    {'name': ['Alice', 'Bob', 'Charlie'], 'score': [85, 92, 78]},
    index=['a', 'b', 'c']
)

# Step 3: Select single row by label
print("Row 'b':")
print(df.loc['b'])

# Step 4: Select multiple rows by list
print("\nRows 'a' and 'c':")
print(df.loc[['a', 'c']])
```

**Expected Output:**
```
Row 'b':
name     Bob
score     92
Name: b, dtype: object

Rows 'a' and 'c':
      name  score
a    Alice     85
c  Charlie     78
```

**Why this output:** `.loc['b']` returned a Series for the row labeled `'b'`. `.loc[['a', 'c']]` returned a DataFrame with rows `'a'` and `'c'`.

#### Example 2: Label Slice (Inclusive)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame(
    {'value': [10, 20, 30, 40, 50]},
    index=list('abcde')
)

# Step 3: Slice from 'b' to 'd' (inclusive)
print(df.loc['b':'d'])
```

**Expected Output:**
```
   value
b     20
c     30
d     40
```

**Why this output:** `.loc['b':'d']` included both endpoints (`'b'` and `'d'`), demonstrating inclusive-stop semantics for label-based slices.

#### Example 3: Row-and-Column Intersection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame(
    {'A': [1, 2, 3], 'B': [4, 5, 6]},
    index=['x', 'y', 'z']
)

# Step 3: Select specific cell
print(f"df.loc['y', 'B'] = {df.loc['y', 'B']}")

# Step 4: Select row 'y' columns A and B
print("\nRow 'y', columns A and B:")
print(df.loc['y', ['A', 'B']])
```

**Expected Output:**
```
df.loc['y', 'B'] = 5

Row 'y', columns A and B:
A    2
B    5
Name: y, dtype: int64
```

**Why this output:** `.loc['y', 'B']` selected the scalar value at row `'y'` and column `'B'`. `.loc['y', ['A', 'B']]` returned a Series for row `'y'` with columns `A` and `B`.

### Real-World Cases with Explanation

**Case 1: Time Series Selection** — A financial analyst uses `df.loc['2024-01-01':'2024-03-31']` to select a quarter of daily records by date labels.

**Case 2: Customer Lookup** — A CRM analyst selects a specific customer's record by ID label using `df.loc[customer_id]`.

**Case 3: Panel Data** — A social scientist selects a country's data from a MultiIndex DataFrame using `df.loc['USA']`.

### References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label

---

## Core Concept 2: .iloc (Position-Based Selection)

### Definitions

**Core Definition:** `.iloc` is a strictly integer-position-based indexer that selects rows (and optionally columns) by their zero-based physical position in the DataFrame, ignoring custom index labels entirely.

**Technical Definition:** `DataFrame.iloc[row_indexer, column_indexer]` selects data using integer positions from `0` to `length-1`. For a single integer, it returns a Series (row) or scalar; for a list of integers, it returns a DataFrame; for an integer slice, it returns a DataFrame with exclusive stop. Out-of-bounds positions raise `IndexError`; non-integer inputs raise `TypeError`.

**Beginner-Friendly Explanation:** `.iloc` lets you pick rows by their position number, starting from 0. The first row is position 0, the second is 1, and so on. This works regardless of what labels are in the index.

### Purposes

- To select rows by their physical order in the DataFrame.
- To avoid ambiguity when the index contains integer labels.
- To perform position-based slicing and fancy indexing.
- To sample data by position.
- To access rows in DataFrames with non-integer indexes.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single row
df.iloc[position]

# Multiple rows by list
df.iloc[[pos1, pos2]]

# Integer slice (exclusive)
df.iloc[start:stop]

# Row-and-column intersection
df.iloc[row_pos, col_pos]

# Boolean array (positional)
df.iloc[boolean_array]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `position` | Integer offset (0-based) | Yes |
| `[pos1, pos2]` | List of integer offsets | Yes (for multiple) |
| `start:stop` | Integer slice (exclusive stop) | Yes |
| `col_pos` | Column position | No |

#### Syntax Rules

- Positions must be integers; floats raise `TypeError`.
- Negative positions count from the end (`-1` is last).
- Slices follow Python's exclusive-stop convention.
- `.iloc` ignores index labels entirely.

#### Constraints and Limitations

- Cannot perform label-based lookups.
- Out-of-bounds positions raise `IndexError`.
- Non-contiguous arrays may require copies.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single and Multiple Row Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with custom index
df = pd.DataFrame(
    {'name': ['Alice', 'Bob', 'Charlie'], 'score': [85, 92, 78]},
    index=['a', 'b', 'c']
)

# Step 3: Select first row by position
print("Row at position 0:")
print(df.iloc[0])

# Step 4: Select positions 0 and 2
print("\nRows at positions 0 and 2:")
print(df.iloc[[0, 2]])
```

**Expected Output:**
```
Row at position 0:
name    Alice
score      85
Name: a, dtype: object

Rows at positions 0 and 2:
      name  score
a    Alice     85
c  Charlie     78
```

**Why this output:** `.iloc[0]` selected the first row (position 0), regardless of its label `'a'`. `.iloc[[0, 2]]` selected positions 0 and 2.

#### Example 2: Positional Slice (Exclusive)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]})

# Step 3: Slice positions 1 to 3 (exclusive)
print(df.iloc[1:3])
```

**Expected Output:**
```
   value
1     20
2     30
```

**Why this output:** `.iloc[1:3]` returned positions 1 and 2, excluding position 3—standard Python slice behavior.

#### Example 3: Row-and-Column Positional Intersection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame(
    {'A': [1, 2, 3], 'B': [4, 5, 6], 'C': [7, 8, 9]},
    index=['x', 'y', 'z']
)

# Step 3: Select cell at row 1, column 2
print(f"df.iloc[1, 2] = {df.iloc[1, 2]}")

# Step 4: Select first 2 rows, first 2 columns
print("\nFirst 2 rows, first 2 columns:")
print(df.iloc[0:2, 0:2])
```

**Expected Output:**
```
df.iloc[1, 2] = 8

First 2 rows, first 2 columns:
   A  B
x  1  4
y  2  5
```

**Why this output:** `.iloc[1, 2]` selected the scalar at position (row 1, column 2). `.iloc[0:2, 0:2]` selected the top-left 2×2 submatrix.

### Real-World Cases with Explanation

**Case 1: First/Last Records** — A data engineer uses `.iloc[0]` and `.iloc[-1]` to inspect the first and last rows of a DataFrame.

**Case 2: Train/Test Split** — A machine learning engineer uses positional slicing (`.iloc[:800]`, `.iloc[800:]`) to split data into training and testing sets.

**Case 3: Sampling** — A statistician uses `.iloc` with a random list of positions to draw a random sample.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position

---

## Core Concept 3: Boolean Masks

### Definitions

**Core Definition:** Boolean masks filter DataFrame rows by applying a boolean array (or Series) of `True`/`False` values, where `True` indicates inclusion and `False` indicates exclusion.

**Technical Definition:** A boolean mask is generated by applying a comparison or logical operation to a DataFrame, producing a boolean Series or DataFrame. When used as a row indexer, pandas aligns the mask by index labels before filtering. Compound conditions are constructed with `&` (and), `|` (or), and `~` (not), with parentheses required around each condition. `NA` values in boolean masks are treated as `False`.

**Beginner-Friendly Explanation:** Boolean masks let you say "give me all the rows that satisfy this condition." You write a condition (like `df['score'] > 80`), which produces `True`/`False` for each row, and pandas returns only the rows where the condition is `True`.

### Purposes

- To filter rows based on one or more conditions.
- To create subsets for focused analysis.
- To combine multiple conditions with logical operators.
- To identify and handle outliers or invalid records.
- To prepare data for machine learning.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single condition
df[df['col'] > threshold]

# Multiple conditions
df[(df['col1'] > a) & (df['col2'] < b)]

# Using .loc with boolean mask
df.loc[df['col'] > threshold]

# Using .iloc with boolean array
df.iloc[boolean_array]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `condition` | Comparison producing boolean Series | Yes |
| `&`, `|`, `~` | Logical operators | No |
| Parentheses | Required around each condition | Yes (when combining) |

#### Syntax Rules

- The boolean mask must align with the DataFrame's index.
- Use `&`, `|`, `~` instead of `and`, `or`, `not`.
- Each condition must be wrapped in parentheses when combined.
- `NA` values in boolean masks are treated as `False`.

#### Constraints and Limitations

- Boolean masks created from a different index are aligned before filtering.
- Chained boolean operations can be slow for very large DataFrames.
- The `[]` operator with a boolean mask returns a copy, not a view.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single Condition Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'name': ['Alice', 'Bob', 'Charlie', 'Diana'], 'score': [85, 45, 92, 38]})

# Step 3: Create mask
mask = df['score'] > 80
print("Mask:")
print(mask)

# Step 4: Apply mask
print("\nFiltered:")
print(df[mask])
```

**Expected Output:**
```
Mask:
0     True
1    False
2     True
3    False
Name: score, dtype: bool

Filtered:
      name  score
0    Alice     85
2  Charlie     92
```

**Why this output:** The mask `df['score'] > 80` produced `True` for rows 0 and 2. When applied, only those rows were returned.

#### Example 2: Combining Conditions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 35, 30, 45],
    'score': [85, 45, 92, 38]
})

# Step 3: Combine conditions
result = df[(df['age'] > 28) & (df['score'] > 50)]
print(result)
```

**Expected Output:**
```
      name  age  score
2  Charlie   30     92
```

**Why this output:** The combined condition selected rows where age > 28 **and** score > 50. Only Charlie satisfied both.

#### Example 3: Using .loc with Boolean Mask

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4], 'B': [10, 20, 30, 40]})

# Step 3: Apply boolean mask via .loc
result = df.loc[df['A'] > 2, ['A', 'B']]
print(result)
```

**Expected Output:**
```
   A   B
2  3  30
3  4  40
```

**Why this output:** `.loc` combined the boolean mask with explicit column selection, returning rows where A > 2 and only columns A and B.

### Real-World Cases with Explanation

**Case 1: Outlier Detection** — A data analyst filters sensor readings beyond a threshold (`df[df['reading'] > 100]`) to identify outliers.

**Case 2: Customer Segmentation** — A marketing analyst selects high-value customers using `df[(df['spend'] > 1000) & (df['visits'] > 10)]`.

**Case 3: Missing Data** — A data scientist filters rows with non-missing values using `df[df['col'].notna()]`.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Indexing with Boolean Arrays — https://pandas.pydata.org/docs/user_guide/indexing.html#indexing-with-boolean-arrays

---

## Core Concept 4: Slicing

### Definitions

**Core Definition:** Slicing extracts a contiguous or stepped interval of rows from a DataFrame, using either integer-position slices (via `.iloc`, exclusive stop) or label-based slices (via `.loc`, inclusive stop).

**Technical Definition:** Positional slices (`df.iloc[start:stop:step]`) follow Python's exclusive-stop convention. Label slices (`df.loc[start_label:stop_label:step]`) include both endpoints if they exist in the index. The `[]` operator with integer slices behaves positionally; with label slices, it behaves label-based. Slicing returns a DataFrame (or Series for single-row slices).

**Beginner-Friendly Explanation:** Slicing lets you grab a range of rows. If you slice by position, pandas stops one row before the number you give. If you slice by label, pandas includes both the start and end labels. This difference is a common source of confusion, so always use `.loc` or `.iloc` to make your intent clear.

### Purposes

- To extract contiguous subsets of rows for analysis.
- To select time ranges in time-series data.
- To partition data into segments (e.g., first half, second half).
- To apply operations to a subset of a DataFrame.
- To sample every Nth row using step slices.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Positional slice (exclusive stop)
df.iloc[start:stop:step]

# Label slice (inclusive stop)
df.loc[start_label:stop_label:step]

# Bracket slice (positional for integer slices)
df[start:stop]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `start` | Starting position or label | No |
| `stop` | Ending position (exclusive) or label (inclusive) | No |
| `step` | Step size | No |

#### Syntax Rules

- Positional slices use exclusive stop; label slices use inclusive stop.
- If a label in a slice is not found, `.loc` raises `KeyError`.
- Step can be negative to reverse direction.
- The `[]` operator with integer slices behaves positionally; with label slices, it behaves label-based.

#### Constraints and Limitations

- Mixing positional and label-based slicing in `[]` can produce unexpected results.
- Label-based slicing requires the index to be sorted for reliable behavior.
- Non-unique indexes may produce ambiguous slice results.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Positional Slice (Exclusive)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]})

# Step 3: Slice positions 1 to 3 (exclusive)
print(df.iloc[1:3])
```

**Expected Output:**
```
   value
1     20
2     30
```

**Why this output:** `.iloc[1:3]` returned positions 1 and 2, excluding position 3.

#### Example 2: Label Slice (Inclusive)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with string index
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]}, index=list('abcde'))

# Step 3: Slice from 'b' to 'd' (inclusive)
print(df.loc['b':'d'])
```

**Expected Output:**
```
   value
b     20
c     30
d     40
```

**Why this output:** `.loc['b':'d']` included both endpoints (`'b'` and `'d'`), demonstrating inclusive-stop semantics.

#### Example 3: Slice with Step

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': range(10)})

# Step 3: Slice every other row from positions 1 to 8
print(df.iloc[1:8:2])
```

**Expected Output:**
```
   value
1      1
3      3
5      5
7      7
```

**Why this output:** The step of 2 selected positions 1, 3, 5, and 7, skipping every other row.

### Real-World Cases with Explanation

**Case 1: Time Window** — An analyst selects a six-month window of daily sales data using `df.loc['2024-01-01':'2024-06-30']`.

**Case 2: Batch Processing** — A data engineer processes a large DataFrame in chunks using positional slices (`df.iloc[i:i+1000]`).

**Case 3: Sampling Every Nth Record** — A scientist extracts every 10th measurement using `df.iloc[::10]`.

### References

- Slicing with Labels — https://pandas.pydata.org/docs/user_guide/indexing.html#slicing-with-labels
- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html

---

## Core Concept 5: Conditional Row Selection (.query() and .isin())

### Definitions

**Core Definition:** Conditional row selection uses optimized querying filters—`.query()` for readable string-expression syntax and `.isin()` for filtering values against an iterable of choices—to extract rows matching specific criteria.

**Technical Definition:** `DataFrame.query(expr, inplace=False, **kwargs)` evaluates a boolean expression string against the DataFrame's columns, using the `numexpr` engine when available for performance. It supports variable interpolation via `@` (e.g., `@external_var`). `Series.isin(values)` and `DataFrame.isin(values)` return a boolean mask indicating whether each element is contained in the provided iterable. These methods are particularly useful for complex, readable filters.

**Beginner-Friendly Explanation:** `.query()` lets you filter rows using a readable string, like `df.query('age > 30 and score > 80')`. `.isin()` lets you filter rows where a column's value is in a list, like `df[df['city'].isin(['NYC', 'LA'])]`. Both make complex filters easier to read.

### Purposes

- To write readable, SQL-like filtering expressions.
- To filter rows where values match a list of choices.
- To leverage `numexpr` for faster evaluation on large DataFrames.
- To interpolate external variables into queries via `@`.
- To simplify compound conditions in pipelines.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Query with string expression
df.query('col1 > 10 and col2 < 20')
df.query('col1 > @threshold')

# Isin with iterable
df[df['col'].isin(['A', 'B', 'C'])]
df[~df['col'].isin(['A', 'B'])]  # Negation
```

#### Component Breakdown

| Method | Description | Key Parameters |
|--------|-------------|----------------|
| `.query()` | Evaluate string expression | `expr`, `inplace` |
| `.isin()` | Check membership in iterable | `values` |

#### Syntax Rules

- `.query()` uses `and`/`or`/`not` (not `&`/`|`/`~`).
- Column names with spaces must be wrapped in backticks (`` `col name` ``).
- External variables are referenced with `@` prefix.
- `.isin()` accepts lists, sets, Series, and other iterables.

#### Constraints and Limitations

- **`numexpr` dependency:** Performance benefit requires `numexpr` installed.
- **Column name conflicts:** Columns named `index` or Python keywords need backticks.
- **Version-specific:** Some `query()` expressions behave differently across versions.
- **`.isin()` performance:** For very large iterables, may be slower than alternatives.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic .query()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name' : ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age'  : [25, 35, 30, 45],
    'score': [85, 45, 92, 38]
})

# Step 3: Query with condition
result = df.query('age > 28 and score > 50')
print(result)
```

**Expected Output:**
```
      name  age  score
2  Charlie   30     92
```

**Why this output:** `.query('age > 28 and score > 50')` filtered rows where both conditions were true. Only Charlie satisfied both.

#### Example 2: .query() with External Variable

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4], 'B': [10, 20, 30, 40]})

# Step 3: Define threshold variable
threshold = 25

# Step 4: Query using @ interpolation
result = df.query('B > @threshold')
print(result)
```

**Expected Output:**
```
   A   B
2  3  30
3  4  40
```

**Why this output:** The `@threshold` syntax interpolated the external variable `threshold = 25` into the query, filtering rows where B > 25.

#### Example 3: .isin() for Membership Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'city': ['NYC', 'LA', 'Chicago', 'NYC']
})

# Step 3: Filter with isin
result = df[df['city'].isin(['NYC', 'LA'])]
print(result)

# Step 4: Negation with ~
print("\nNot in NYC or LA:")
print(df[~df['city'].isin(['NYC', 'LA'])])
```

**Expected Output:**
```
      name     city
0    Alice      NYC
1      Bob       LA
3    Diana      NYC

Not in NYC or LA:
      name     city
2  Charlie  Chicago
```

**Why this output:** `.isin(['NYC', 'LA'])` returned `True` for rows where city was NYC or LA. The `~` operator negated the mask, returning rows not in those cities.

### Real-World Cases with Explanation

**Case 1: Log Filtering** — A DevOps engineer uses `df.query('level == "ERROR" and timestamp > @cutoff')` to filter error logs after a specific time.

**Case 2: Customer Segmentation** — A marketing analyst uses `df[df['segment'].isin(['premium', 'gold'])]` to select high-value customers.

**Case 3: Multi-Criteria Filtering** — A data scientist uses `.query()` with multiple conditions to create readable, maintainable filters in a pipeline.

### References

- pandas.DataFrame.query — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.query.html
- pandas.DataFrame.isin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isin.html
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html

---

## Core Concept 6: Index Resets (.reset_index() and .set_index())

### Definitions

**Core Definition:** Index resets refer to the operations that transition between row labels and data columns: `.reset_index()` converts the index into a regular column and restores a default `RangeIndex`, while `.set_index()` promotes one or more columns to become the new row index.

**Technical Definition:** `DataFrame.reset_index(level=None, drop=False, inplace=False, col_level=0, col_fill='')` returns a new DataFrame with the index reset to a `RangeIndex`, and the old index inserted as column(s) unless `drop=True`. `DataFrame.set_index(keys, drop=True, append=False, inplace=False, verify_integrity=False)` returns a new DataFrame with the specified column(s) as the index, removing them from the columns unless `drop=False`. Both methods support `inplace=True` (discouraged under CoW).

**Beginner-Friendly Explanation:** The index is the row labels. Sometimes you want to move those labels into a regular column (reset) or make a column become the new row labels (set). These operations let you switch between different ways of identifying rows.

### Purposes

- To convert an index into a regular column for analysis.
- To promote a column to the index for label-based selection.
- To restore a default `RangeIndex` after filtering.
- To create MultiIndex structures for hierarchical data.
- To prepare data for joins or merges on specific keys.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Reset index
df.reset_index(level=None, drop=False, inplace=False)

# Set index
df.set_index(keys, drop=True, append=False, inplace=False)
```

#### Component Breakdown

| Method | Description | Key Parameters |
|--------|-------------|----------------|
| `.reset_index()` | Convert index to column | `drop`, `inplace` |
| `.set_index()` | Promote column(s) to index | `keys`, `drop`, `append` |

#### Syntax Rules

- `.reset_index()` returns a new DataFrame unless `inplace=True`.
- `drop=True` discards the old index instead of converting it to a column.
- `.set_index()` removes the column(s) from the data unless `drop=False`.
- `append=True` in `.set_index()` adds to an existing MultiIndex.

#### Constraints and Limitations

- `inplace=True` is deprecated under CoW; prefer reassignment.
- `.set_index()` with non-unique keys may cause ambiguity in lookups.
- Resetting a MultiIndex produces multiple columns.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic reset_index()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with custom index
df = pd.DataFrame({'value': [10, 20, 30]}, index=['a', 'b', 'c'])
print("Original:")
print(df)

# Step 3: Reset index
df_reset = df.reset_index()
print("\nAfter reset_index:")
print(df_reset)
```

**Expected Output:**
```
Original:
   value
a     10
b     20
c     30

After reset_index:
  index  value
0     a     10
1     b     20
2     c     30
```

**Why this output:** `.reset_index()` moved the index labels (`'a'`, `'b'`, `'c'`) into a new column named `'index'` and restored a default `RangeIndex`.

#### Example 2: set_index() to Promote a Column

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'id': [101, 102, 103],
    'name': ['Alice', 'Bob', 'Charlie'],
    'score': [85, 92, 78]
})

# Step 3: Set 'id' as index
df_indexed = df.set_index('id')
print(df_indexed)

# Step 4: Verify index
print(f"\nIndex: {df_indexed.index.tolist()}")
```

**Expected Output:**
```
       name  score
id                
101   Alice     85
102     Bob     92
103  Charlie     78

Index: [101, 102, 103]
```

**Why this output:** `.set_index('id')` promoted the `'id'` column to the index, removing it from the data columns.

#### Example 3: Round-Trip reset_index() and set_index()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'id': [1, 2], 'value': [10, 20]}).set_index('id')
print("After set_index:")
print(df)

# Step 3: Reset index
df_reset = df.reset_index()
print("\nAfter reset_index:")
print(df_reset)
```

**Expected Output:**
```
After set_index:
    value
id       
1      10
2      20

After reset_index:
   id  value
0   1     10
1   2     20
```

**Why this output:** `.set_index('id')` promoted `id` to the index. `.reset_index()` restored `id` as a regular column and created a default `RangeIndex`.

### Real-World Cases with Explanation

**Case 1: Time Series Preparation** — A financial analyst uses `.set_index('date')` to make a date column the index, enabling date-range slicing with `.loc`.

**Case 2: After Filtering** — A data engineer uses `.reset_index(drop=True)` after filtering to restore a clean sequential index for export.

**Case 3: MultiIndex Creation** — A social scientist uses `.set_index(['country', 'year'])` to create a hierarchical index for panel data analysis.

### References

- pandas.DataFrame.reset_index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.reset_index.html
- pandas.DataFrame.set_index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.set_index.html
- Reshaping and Pivot Tables — https://pandas.pydata.org/docs/user_guide/reshaping.html

---

## Summary Table: Row Selection Methods

| Method | Key Type | Slice Stop | Ambiguity | Best For |
|--------|----------|------------|-----------|----------|
| `.loc` | Label | Inclusive | None | Label-based selection |
| `.iloc` | Integer position | Exclusive | None | Position-based selection |
| Boolean mask | Boolean | N/A | None | Conditional filtering |
| `df[start:stop]` | Positional (int) or label | Depends | Medium | Simple slices |
| `.query()` | String expression | N/A | None | Readable complex filters |
| `.isin()` | Iterable | N/A | None | Membership filtering |
| `.reset_index()` | N/A | N/A | None | Index → column |
| `.set_index()` | Column(s) | N/A | None | Column → index |

---

## References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- pandas.DataFrame.query — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.query.html
- pandas.DataFrame.isin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isin.html
- pandas.DataFrame.reset_index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.reset_index.html
- pandas.DataFrame.set_index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.set_index.html
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html
- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- Slicing with Labels — https://pandas.pydata.org/docs/user_guide/indexing.html#slicing-with-labels
- Reshaping and Pivot Tables — https://pandas.pydata.org/docs/user_guide/reshaping.html
- MultiIndex / Advanced Indexing — https://pandas.pydata.org/docs/user_guide/advanced.html