# Comprehensive Programming Cheat Sheet: Pandas `.loc` (Label-Based Selection)

---

## Topic Overview

### Definitions

**Core Definition:** `.loc` is a pandas indexer that performs strictly label-based selection on a DataFrame or Series, allowing access to rows and columns by their index and column labels, with inclusive slicing of both start and stop boundaries.

**Technical Definition:** `DataFrame.loc[row_indexer, column_indexer]` is a property that returns a `_LocIndexer` object, which accepts labels, lists of labels, label slices (inclusive), boolean arrays, callables, and `pd.IndexSlice` objects. It never falls back to positional access, even when the index contains integers. Missing labels raise `KeyError`. Under pandas 3.0, `.loc` is the canonical label-based accessor, and `[]` with integer keys also behaves label-based (aligning with `.loc` semantics).

**Beginner-Friendly Explanation:** `.loc` lets you pick rows and columns by their names or labels. If your DataFrame has row labels like dates or IDs and column names like "name" or "score," you use `.loc` to select exactly what you want. The special thing about `.loc` slicing is that it includes both the start and end labels—unlike Python lists, which stop before the end.

### Key Characteristics

- **Strictly label-based:** Never positional, even with integer indexes.
- **Inclusive slicing:** `df.loc['a':'c']` includes both `'a'` and `'c'`.
- **Dual-axis selection:** Accepts both row and column indexers: `df.loc[rows, cols]`.
- **Rich indexer types:** Supports labels, lists, slices, boolean arrays, callables, and tuples.
- **Missing labels raise KeyError:** No silent fallback.
- **MultiIndex-aware:** Supports tuple-based and `pd.IndexSlice` selection.

### Prerequisites

- Basic Python syntax (lists, dictionaries, tuples).
- Pandas Series and DataFrame construction.
- Understanding of DataFrame indexing basics.
- Familiarity with boolean logic and comparison operators.
- Awareness of MultiIndex concepts (for advanced sections).

### Related Programming Areas

- **Data Analysis:** Subsetting data by labels.
- **Time Series Analysis:** Date-range slicing.
- **Data Cleaning:** Conditional filtering.
- **Panel Data:** MultiIndex selection.
- **ETL Pipelines:** Deterministic data extraction.

### Core Concepts / Features

1. Label-Based Selection
2. Row Selection
3. Column Selection
4. Conditional Selection
5. Multiple Conditions
6. MultiIndex Slicing

---

## Core Concept 1: Label-Based Selection

### Definitions

**Core Definition:** Label-based selection uses `.loc` to access DataFrame or Series elements by their explicit index and column labels, with slicing that includes both the start and stop boundaries.

**Technical Definition:** `DataFrame.loc` is a property that returns an indexer object. When a scalar label is passed, it returns a scalar (if both row and column labels are scalars) or a Series (if only a row label is passed). When a list of labels is passed, it returns a DataFrame. When a label slice is passed, it returns a DataFrame with both endpoints included if present in the index. Unlike Python's list slicing, `.loc` slice endpoints are inclusive.

**Beginner-Friendly Explanation:** `.loc` is all about labels. You pass the name of the row you want, and pandas gives you that row. If you pass a range of labels, pandas includes both ends. This is different from positional slicing, which excludes the end.

### Purposes

- To access data by meaningful identifiers (dates, IDs, names).
- To select ranges of data using label-based slices.
- To avoid ambiguity when the index contains integer labels.
- To align selections with domain-specific row/column labels.
- To enable reliable lookups in production pipelines.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Scalar row label
df.loc[label]

# List of row labels
df.loc[[label1, label2]]

# Label slice (inclusive)
df.loc[start_label:stop_label]

# Row and column labels
df.loc[row_label, col_label]

# Slice both axes
df.loc[row_start:row_stop, col_start:col_stop]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `row_indexer` | Label, list, slice, boolean, callable | Yes |
| `col_indexer` | Label, list, slice, boolean, callable | No |
| `start:stop` | Inclusive label slice | No |

#### Syntax Rules

- `.loc` is strictly label-based; integer inputs are labels, not positions.
- Label slices include both endpoints if present.
- Missing labels raise `KeyError`.
- Boolean arrays passed to `.loc` are aligned by index before masking.

#### Constraints and Limitations

- Non-unique index labels may return multiple rows.
- Unsorted indexes may behave unpredictably in slices (sort with `.sort_index()`).
- Cannot perform positional lookups; use `.iloc` for positions.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Scalar Label Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with custom index
df = pd.DataFrame(
    {'name': ['Alice', 'Bob', 'Charlie'], 'score': [85, 92, 78]},
    index=['a', 'b', 'c']
)

# Step 3: Select row by label
print("Row 'b':")
print(df.loc['b'])

# Step 4: Select specific cell
print(f"\ndf.loc['c', 'name'] = {df.loc['c', 'name']}")
```

**Expected Output:**
```
Row 'b':
name     Bob
score     92
Name: b, dtype: object

df.loc['c', 'name'] = Charlie
```

**Why this output:** `.loc['b']` returned a Series for the row labeled `'b'`. `.loc['c', 'name']` returned the scalar at row `'c'`, column `'name'`.

#### Example 2: Inclusive Label Slice

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
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

**Why this output:** `.loc['b':'d']` included both `'b'` and `'d'`, demonstrating inclusive slicing. This differs from `.iloc`, where the stop is exclusive.

#### Example 3: Integer Labels vs. Positions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with integer index
df = pd.DataFrame({'value': [10, 20, 30]}, index=[100, 200, 300])

# Step 3: .loc uses labels
print(f"df.loc[200] = {df.loc[200]}")

# Step 4: .loc with non-existent label
try:
    df.loc[1]
except KeyError as e:
    print(f"KeyError: {e}")
```

**Expected Output:**
```
df.loc[200] = value    20
Name: 200, dtype: int64
KeyError: 1
```

**Why this output:** `.loc[200]` retrieved the row with label 200. `.loc[1]` raised `KeyError` because `1` is not a label—`.loc` never falls back to positional access.

### Real-World Cases with Explanation

**Case 1: Time Series** — A financial analyst selects a date range with `df.loc['2024-01-01':'2024-03-31']`, which includes both endpoints.

**Case 2: Customer Records** — A CRM analyst selects a specific customer by ID with `df.loc[customer_id]`.

**Case 3: Panel Data** — A researcher selects a country's data with `df.loc['USA']` from a country-indexed DataFrame.

### References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label

---

## Core Concept 2: Row Selection

### Definitions

**Core Definition:** Row selection with `.loc` extracts single rows as a Series or multiple rows as a DataFrame, using scalar labels, lists of labels, or label slice ranges.

**Technical Definition:** `df.loc[label]` returns a Series when `label` is a scalar; `df.loc[[label1, label2]]` returns a DataFrame with the specified rows; `df.loc[start:stop]` returns a DataFrame with rows from `start` through `stop`, inclusive. The Series returned by scalar selection has the column labels as its index and the Series name set to the row label.

**Beginner-Friendly Explanation:** You can select one row by its label, multiple rows by passing a list of labels, or a range of rows by slicing with labels. A single row comes back as a Series; multiple rows come back as a DataFrame.

### Purposes

- To extract a single record for detailed inspection.
- To select a subset of rows for analysis.
- To select a range of rows by label.
- To create smaller DataFrames for downstream operations.
- To align rows for joins or merges.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single row (Series)
df.loc[label]

# Multiple rows (DataFrame)
df.loc[[label1, label2, label3]]

# Slice of rows (DataFrame)
df.loc[start_label:stop_label]

# Slice with step
df.loc[start_label:stop_label:step]
```

#### Component Breakdown

| Operation | Returns | Notes |
|-----------|---------|-------|
| `df.loc[label]` | Series | Scalar row label |
| `df.loc[[l1, l2]]` | DataFrame | List of labels |
| `df.loc[start:stop]` | DataFrame | Inclusive slice |
| `df.loc[start:stop:step]` | DataFrame | Step applied |

#### Syntax Rules

- Scalar label → Series; list or slice → DataFrame.
- Slices include both endpoints if present.
- The Series name equals the row label.
- Missing labels raise `KeyError`.

#### Constraints and Limitations

- Non-unique labels return multiple rows for scalar selection (as a DataFrame).
- Labels must exist in the index; no positional fallback.
- Slices on unsorted indexes may behave unexpectedly.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single Row as Series

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame(
    {'A': [1, 2, 3], 'B': [4, 5, 6]},
    index=['x', 'y', 'z']
)

# Step 3: Select single row
row = df.loc['y']
print(row)
print(f"\nType: {type(row).__name__}")
print(f"Name: {row.name}")
```

**Expected Output:**
```
A    2
B    5
Name: y, dtype: int64

Type: Series
Name: y
```

**Why this output:** `.loc['y']` returned a Series with column labels as the index and the row label `'y'` as the Series name.

#### Example 2: Multiple Rows as DataFrame

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40]}, index=['a', 'b', 'c', 'd'])

# Step 3: Select multiple rows by list
subset = df.loc[['a', 'c']]
print(subset)
print(f"\nType: {type(subset).__name__}")
print(f"Shape: {subset.shape}")
```

**Expected Output:**
```
   value
a     10
c     30

Type: DataFrame
Shape: (2, 1)
```

**Why this output:** Passing a list `['a', 'c']` returned a DataFrame with those two rows in the specified order.

#### Example 3: Label Slice Range

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]}, index=list('abcde'))

# Step 3: Slice rows 'b' through 'd'
print(df.loc['b':'d'])

# Step 4: Slice with step
print("\nEvery other row from 'a' to 'e':")
print(df.loc['a':'e':2])
```

**Expected Output:**
```
   value
b     20
c     30
d     40

Every other row from 'a' to 'e':
   value
a     10
c     30
e     50
```

**Why this output:** `.loc['b':'d']` included rows `b`, `c`, `d`. Adding `:2` selected every other row from `a` to `e`, including both endpoints.

### Real-World Cases with Explanation

**Case 1: Snapshot Reports** — A business analyst selects a single month's record using `df.loc['2024-03']` for a report.

**Case 2: Multi-Period Analysis** — A financial analyst selects multiple quarters using `df.loc[['2024-Q1', '2024-Q3']]`.

**Case 3: Date-Range Filtering** — A data scientist selects a date range with `df.loc['2024-01':'2024-06']`, including both endpoints.

### References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label

---

## Core Concept 3: Column Selection

### Definitions

**Core Definition:** Column selection with `.loc` uses the second dimension argument to limit visible columns, accessed via `df.loc[row_indexer, col_indexer]`, avoiding chained indexing patterns like `df.loc[...][...]`.

**Technical Definition:** `DataFrame.loc[row_indexer, col_indexer]` accepts both row and column indexers simultaneously. The column indexer can be a scalar label, a list of labels, a label slice, or a boolean array. Passing both indexers in a single `.loc` call is more efficient and avoids the pitfalls of chained indexing, which may produce `SettingWithCopyWarning` in legacy pandas and unnecessary copies under CoW.

**Beginner-Friendly Explanation:** `.loc` lets you pick both rows and columns at once. You write `df.loc[rows, columns]`—the first part selects rows, the second selects columns. Doing it in one call is better than doing it in two, because chained selection is slower and can cause bugs.

### Purposes

- To select specific columns for a subset of rows.
- To avoid chained indexing and its associated warnings.
- To align rows and columns in a single, atomic operation.
- To select column ranges alongside row ranges.
- To ensure predictable behavior across pandas versions.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single column for all rows
df.loc[:, 'col']

# Single column for specific rows
df.loc[row_label, 'col']

# Multiple columns
df.loc[:, ['col1', 'col2']]

# Column slice (inclusive)
df.loc[:, 'col_start':'col_stop']

# Row and column selection
df.loc[row_label, ['col1', 'col2']]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `row_indexer` | `:` for all rows, or label/list/slice | Yes |
| `col_indexer` | Label, list, slice, boolean | Yes (for column selection) |

#### Syntax Rules

- Use `:` to select all rows.
- Column labels follow the same semantics as row labels.
- Combining row and column indexers is a single atomic operation.
- Avoid chained indexing: prefer `df.loc[rows, cols]` over `df.loc[rows][cols]`.

#### Constraints and Limitations

- Non-unique column labels may return multiple columns.
- Column slices require the columns to be sortable or in order.
- Chained indexing is discouraged but still works; under CoW, no warning is raised.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single Column Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['NYC', 'LA', 'Chicago']
})

# Step 3: Select single column
print(df.loc[:, 'name'])

# Step 4: Select single column, specific rows
print("\nNames for rows 0 and 2:")
print(df.loc[[0, 2], 'name'])
```

**Expected Output:**
```
0      Alice
1        Bob
2    Charlie
Name: name, dtype: object

Names for rows 0 and 2:
0      Alice
2    Charlie
Name: name, dtype: object
```

**Why this output:** `.loc[:, 'name']` selected column `'name'` for all rows. `.loc[[0, 2], 'name']` restricted both rows and column.

#### Example 2: Multiple Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9]
})

# Step 3: Select multiple columns
print(df.loc[:, ['A', 'C']])
```

**Expected Output:**
```
   A  C
0  1  7
1  2  8
2  3  9
```

**Why this output:** The list `['A', 'C']` selected columns `A` and `C` in that order, for all rows.

#### Example 3: Row and Column Combined vs. Chained

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'score': [85, 92, 78]
}, index=['a', 'b', 'c'])

# Step 3: Combined selection (preferred)
combined = df.loc[['a', 'c'], ['name', 'score']]
print("Combined:")
print(combined)

# Step 4: Chained selection (discouraged)
chained = df.loc[['a', 'c']].loc[:, ['name', 'score']]
print("\nChained (same result but less efficient):")
print(chained)
```

**Expected Output:**
```
Combined:
      name  score
a    Alice     85
c  Charlie     78

Chained (same result but less efficient):
      name  score
a    Alice     85
c  Charlie     78
```

**Why this output:** Both approaches return the same result, but the combined form `df.loc[rows, cols]` is a single atomic operation, whereas the chained form `df.loc[rows].loc[:, cols]` creates an intermediate DataFrame.

### Real-World Cases with Explanation

**Case 1: Report Subsetting** — A business analyst selects specific columns for specific rows in a report using `df.loc[['row1', 'row2'], ['name', 'revenue']]`.

**Case 2: Model Feature Selection** — A data scientist selects a feature matrix with `df.loc[:, ['f1', 'f2', 'f3']]`.

**Case 3: Time-Series Column Slice** — A financial analyst selects columns from `'price'` to `'volume'` using `df.loc[:, 'price':'volume']`.

### References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label
- Returning a View versus Copy — https://pandas.pydata.org/docs/user_guide/indexing.html#returning-a-view-versus-a-copy

---

## Core Concept 4: Conditional Selection

### Definitions

**Core Definition:** Conditional selection uses `.loc` with boolean arrays or callable functions in the row indexer slot to filter rows dynamically based on evaluations of the DataFrame's data.

**Technical Definition:** When a boolean array or Series is passed to `.loc`, pandas aligns it by index and returns rows where the value is `True`. Callables (lambda functions or named functions) may also be passed; pandas calls the function with the calling DataFrame (or Series) and uses the boolean result for filtering. `NA` values in boolean arrays are treated as `False`.

**Beginner-Friendly Explanation:** You can filter rows by a condition, such as "all rows where score > 80." You write the condition, which becomes a `True`/`False` array, and `.loc` uses it to pick only the `True` rows. You can also pass a function that takes the DataFrame and returns a boolean array.

### Purposes

- To filter rows based on data-driven conditions.
- To dynamically select records that meet criteria.
- To combine selection with column projection in one call.
- To apply reusable filter functions across DataFrames.
- To handle complex conditional logic concisely.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Boolean array
df.loc[df['col'] > threshold]

# Boolean array with column selection
df.loc[df['col'] > threshold, ['col1', 'col2']]

# Callable
df.loc[lambda df: df['col'] > threshold]

# Callable with column selection
df.loc[lambda df: df['col'] > threshold, ['col1', 'col2']]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `boolean_array` | Boolean Series or array | Yes (for mask form) |
| `callable` | Function taking DataFrame, returning boolean | Yes (for callable form) |
| `col_indexer` | Column selection | No |

#### Syntax Rules

- Boolean arrays must align with the DataFrame's index.
- Callables receive the calling DataFrame as their argument.
- Use `&`, `|`, `~` for combined conditions; wrap each in parentheses.
- `NA` values in boolean arrays are treated as `False`.

#### Constraints and Limitations

- Boolean arrays from a different index are aligned before masking.
- Callables are invoked once per `.loc` call.
- For complex filters, consider `.query()` for readability.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Boolean Array Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'score': [85, 45, 92, 38]
})

# Step 3: Boolean mask with .loc
result = df.loc[df['score'] > 80]
print(result)
```

**Expected Output:**
```
      name  score
0    Alice     85
2  Charlie     92
```

**Why this output:** The mask `df['score'] > 80` produced `True` for rows 0 and 2, so `.loc` returned only those rows.

#### Example 2: Boolean Mask with Column Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 35, 30, 45],
    'score': [85, 45, 92, 38]
})

# Step 3: Filter rows and select columns
result = df.loc[df['score'] > 50, ['name', 'score']]
print(result)
```

**Expected Output:**
```
      name  score
0    Alice     85
2  Charlie     92
```

**Why this output:** `.loc` applied the boolean mask to rows and restricted columns to `name` and `score` in a single atomic operation.

#### Example 3: Callable Filtering

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4], 'B': [10, 20, 30, 40]})

# Step 3: Callable filter
result = df.loc[lambda d: d['A'] > 2]
print(result)

# Step 4: Callable with column selection
result2 = df.loc[lambda d: d['B'] < 30, ['A']]
print("\nCallable with column selection:")
print(result2)
```

**Expected Output:**
```
   A   B
2  3  30
3  4  40

Callable with column selection:
   A
0  1
1  2
```

**Why this output:** The lambda `lambda d: d['A'] > 2` was called with the DataFrame and returned a boolean Series used for filtering. The second callable restricted both rows and columns.

### Real-World Cases with Explanation

**Case 1: Outlier Filtering** — A data scientist uses `df.loc[df['value'] < upper_bound]` to remove extreme outliers.

**Case 2: Active User Filtering** — An analyst uses `df.loc[lambda d: d['last_login'] > cutoff_date]` to select active users.

**Case 3: Dynamic Thresholds** — A DevOps engineer uses `df.loc[df['latency'] > df['latency'].quantile(0.95)]` to identify tail latency.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html

---

## Core Concept 5: Multiple Conditions

### Definitions

**Core Definition:** Multiple conditions in `.loc` combine several boolean expressions using logical operators (`&`, `|`, `~`) to filter rows that satisfy all, any, or none of the given criteria, while preserving index alignment.

**Technical Definition:** When multiple boolean Series are combined with `&` (and), `|` (or), or `~` (not), pandas evaluates the resulting boolean Series element-wise. Each condition must be wrapped in parentheses due to Python's operator precedence. Indexes are aligned across all conditions, and rows where the combined result is `True` are returned.

**Beginner-Friendly Explanation:** You can filter with more than one condition by combining them with `&` (both true), `|` (either true), or `~` (not true). Always wrap each condition in parentheses. Pandas will align everything by index and return rows that satisfy the combined condition.

### Purposes

- To filter rows that satisfy multiple criteria simultaneously.
- To construct complex, data-driven subsets.
- To combine inclusion and exclusion filters.
- To align multiple condition Series by index automatically.
- To express nuanced business logic in selections.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# AND condition
df.loc[(df['col1'] > a) & (df['col2'] < b)]

# OR condition
df.loc[(df['col1'] > a) | (df['col2'] < b)]

# NOT condition
df.loc[~(df['col'] == value)]

# Combined with column selection
df.loc[(df['col1'] > a) & (df['col2'] < b), ['col1', 'col2']]
```

#### Component Breakdown

| Operator | Meaning | Example |
|----------|---------|---------|
| `&` | Logical AND | `(cond1) & (cond2)` |
| `|` | Logical OR | `(cond1) | (cond2)` |
| `~` | Logical NOT | `~(cond1)` |
| Parentheses | Required around each condition | — |

#### Syntax Rules

- Use `&`, `|`, `~` instead of Python's `and`, `or`, `not`.
- Each condition must be wrapped in parentheses.
- Indexes are aligned across all condition Series.
- `NA` values in boolean conditions are treated as `False`.

#### Constraints and Limitations

- Python operator precedence makes parentheses mandatory.
- Combining many conditions can be slow on large DataFrames.
- For very complex filters, `.query()` may be more readable.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: AND Condition

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 35, 30, 45],
    'score': [85, 45, 92, 38]
})

# Step 3: Combine conditions with AND
result = df.loc[(df['age'] > 28) & (df['score'] > 50)]
print(result)
```

**Expected Output:**
```
      name  age  score
2  Charlie   30     92
```

**Why this output:** The combined condition required both age > 28 **and** score > 50. Only Charlie satisfied both.

#### Example 2: OR and NOT Conditions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'score': [85, 45, 92, 38]
})

# Step 3: OR condition
print("Score > 90 OR score < 40:")
print(df.loc[(df['score'] > 90) | (df['score'] < 40)])

# Step 4: NOT condition
print("\nNot Alice or Bob:")
print(df.loc[~df['name'].isin(['Alice', 'Bob'])])
```

**Expected Output:**
```
Score > 90 OR score < 40:
      name  score
2  Charlie     92
3    Diana     38

Not Alice or Bob:
      name  score
2  Charlie     92
3    Diana     38
```

**Why this output:** The OR condition matched Charlie (92) and Diana (38). The NOT condition excluded Alice and Bob, leaving Charlie and Diana.

#### Example 3: Complex Combined Conditions with Column Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 35, 30, 45, 28],
    'score': [85, 45, 92, 38, 75],
    'city': ['NYC', 'LA', 'NYC', 'LA', 'NYC']
})

# Step 3: Complex filter with column selection
result = df.loc[
    (df['score'] > 50) & (df['age'] < 40) & (df['city'] == 'NYC'),
    ['name', 'score']
]
print(result)
```

**Expected Output:**
```
    name  score
0  Alice     85
4    Eve     75
```

**Why this output:** The combined condition selected rows where score > 50 **and** age < 40 **and** city == 'NYC'. Alice and Eve satisfied all three conditions.

### Real-World Cases with Explanation

**Case 1: Multi-Criteria Filtering** — A marketing analyst uses `df.loc[(df['spend'] > 1000) & (df['visits'] > 10)]` to select high-value, engaged customers.

**Case 2: Exclusion Logic** — A data engineer uses `df.loc[~df['status'].isin(['deleted', 'archived'])]` to exclude inactive records.

**Case 3: Combined Range Filtering** — A financial analyst uses `df.loc[(df['price'] > low) & (df['price'] < high)]` to select prices within a range.

### References

- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Indexing with Boolean Arrays — https://pandas.pydata.org/docs/user_guide/indexing.html#indexing-with-boolean-arrays

---

## Core Concept 6: MultiIndex Slicing

### Definitions

**Core Definition:** MultiIndex slicing navigates hierarchical index spaces using tuples or the `pd.IndexSlice` utility to select data across multiple index tiers simultaneously.

**Technical Definition:** A `MultiIndex` has multiple levels (e.g., country and year). Selection via `.loc` uses tuples to specify values per level: `df.loc[('USA', 2024)]`. To slice across multiple levels, use `pd.IndexSlice`: `df.loc[pd.IndexSlice[:, '2024'], :]` selects all entries in 2024 across all outer levels. Partial tuples are allowed—`df.loc['USA']` returns all entries for the USA across all inner levels. Missing outer levels in slices must use `slice(None)` or `pd.IndexSlice[:]`.

**Beginner-Friendly Explanation:** When your DataFrame has multiple index levels (like country + year), you can select data using tuples. For example, `df.loc[('USA', 2024)]` picks the row for USA in 2024. To slice across levels (like all years for all countries), you use `pd.IndexSlice` to fill in the levels you don't want to restrict.

### Purposes

- To select data from hierarchical indexes with multiple levels.
- To slice across one or more levels of a MultiIndex.
- To extract clean subsets of panel or grouped data.
- To enable cross-sectional and time-series selections simultaneously.
- To support advanced reporting and aggregation.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Full tuple
df.loc[(level0_val, level1_val)]

# Partial tuple (all inner levels)
df.loc[level0_val]

# Slice with IndexSlice
df.loc[pd.IndexSlice[level0_start:level0_stop, level1_val], :]

# Cross-section with xs
df.xs(key, level=level_name)
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| Tuple | Values for each level | Yes (for full selection) |
| `pd.IndexSlice` | Cross-level slicing utility | For multi-level slices |
| `:` | All values for a level | No |

#### Syntax Rules

- Tuples must match the number of index levels when fully specified.
- Partial tuples are allowed for outer levels.
- `pd.IndexSlice` is required for slicing across levels.
- Missing levels require `slice(None)` or `:` in `IndexSlice`.

#### Constraints and Limitations

- Slicing with `pd.IndexSlice` requires the MultiIndex to be sorted for reliable performance.
- Non-unique MultiIndex entries may cause ambiguity.
- `pd.IndexSlice` is a utility object, not a function; use square brackets.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Full Tuple Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create MultiIndex DataFrame
df = pd.DataFrame({
    'sales': [100, 200, 150, 250, 300],
    'region': ['North', 'North', 'South', 'South', 'East'],
    'year': [2023, 2024, 2023, 2024, 2024]
}).set_index(['region', 'year'])

# Step 3: Select with full tuple
print(df.loc[('North', 2024)])
```

**Expected Output:**
```
sales    200
Name: (North, 2024), dtype: int64
```

**Why this output:** The tuple `('North', 2024)` matched a single row in the MultiIndex, returning it as a Series.

#### Example 2: Partial Tuple Selection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create MultiIndex DataFrame
df = pd.DataFrame({
    'sales': [100, 200, 150, 250, 300],
    'region': ['North', 'North', 'South', 'South', 'East'],
    'year': [2023, 2024, 2023, 2024, 2024]
}).set_index(['region', 'year'])

# Step 3: Partial tuple (all years for North)
print("North (all years):")
print(df.loc['North'])
```

**Expected Output:**
```
North (all years):
      sales
year       
2023    1002024    200
```

**Why this output:** Passing `'North'` (a partial tuple) selected all entries for the North region across all years.

#### Example 3: pd.IndexSlice for Cross-Level Slicing

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create MultiIndex DataFrame
df = pd.DataFrame({
    'sales'  : [100, 200, 150, 250, 300, 400],
    'region' : ['North', 'North', 'South', 'South', 'East', 'East'],
    'year'   : [2023, 2024, 2023, 2024, 2023, 2024]
}).set_index(['region', 'year'])

# Step 3: Slice across levels with IndexSlice
result = df.loc[pd.IndexSlice[:, 2024], :]
print("All regions, year 2024:")
print(result)
```

**Expected Output:**
```
All regions, year 2024:
             sales
region year       
North  2024    200
South  2024    250
East   2024    400
```

**Why this output:** `pd.IndexSlice[:, 2024]` selected all values of the outer level (`region`) and specifically 2024 in the inner level (`year`).

### Real-World Cases with Explanation

**Case 1: Panel Data Analysis** — A social scientist selects a specific country-year combination with `df.loc[('USA', 2024)]` from a country-year MultiIndex.

**Case 2: Regional Reports** — A business analyst selects all years for a specific region with `df.loc['North']`.

**Case 3: Cross-Sectional Slicing** — A financial analyst selects all regions for a specific year with `df.loc[pd.IndexSlice[:, 2024], :]`.

### References

- MultiIndex / Advanced Indexing — https://pandas.pydata.org/docs/user_guide/advanced.html
- pandas.IndexSlice — https://pandas.pydata.org/docs/reference/api/pandas.IndexSlice.html
- pandas.DataFrame.xs — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.xs.html

---

## Summary Table: .loc Selection Patterns

| Pattern | Syntax | Returns | Notes |
|---------|--------|---------|-------|
| Single row | `df.loc[label]` | Series | Row label → Series |
| Multiple rows | `df.loc[[l1, l2]]` | DataFrame | List of labels |
| Row slice | `df.loc[start:stop]` | DataFrame | Inclusive stop |
| Single cell | `df.loc[row, col]` | Scalar | Both scalars |
| Row + column | `df.loc[rows, cols]` | DataFrame | Atomic operation |
| Boolean mask | `df.loc[df['col'] > t]` | DataFrame | Conditional filter |
| Callable | `df.loc[lambda d: ...]` | DataFrame | Function-based filter |
| MultiIndex tuple | `df.loc[(a, b)]` | Series/DataFrame | Hierarchical selection |
| IndexSlice | `df.loc[pd.IndexSlice[...], :]` | DataFrame | Cross-level slicing |

---

## References

- pandas.DataFrame.loc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html
- Selection by Label — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-label
- Boolean Indexing — https://pandas.pydata.org/docs/user_guide/indexing.html#boolean-indexing
- Indexing with Boolean Arrays — https://pandas.pydata.org/docs/user_guide/indexing.html#indexing-with-boolean-arrays
- Returning a View versus Copy — https://pandas.pydata.org/docs/user_guide/indexing.html#returning-a-view-versus-a-copy
- MultiIndex / Advanced Indexing — https://pandas.pydata.org/docs/user_guide/advanced.html
- pandas.IndexSlice — https://pandas.pydata.org/docs/reference/api/pandas.IndexSlice.html
- pandas.DataFrame.xs — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.xs.html
- Copy-on-Write — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- What's New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html