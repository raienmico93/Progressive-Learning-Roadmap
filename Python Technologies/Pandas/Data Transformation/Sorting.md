# Pandas Sorting and Ordering: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Sorting and ordering in pandas is the process of rearranging the rows of a DataFrame or the elements of a Series into a specified sequence, either by the values in one or more columns (`sort_values()`) or by the index labels (`sort_index()`).

**Technical Definition:** Pandas provides two primary sorting methods: `DataFrame.sort_values(by, *, axis=0, ascending=True, inplace=False, kind='quicksort', na_position='last', ignore_index=False, key=None)` and `DataFrame.sort_index(*, axis=0, level=None, ascending=True, inplace=False, kind='quicksort', na_position='last', sort_remaining=True, ignore_index=False, key=None)`. Both methods return a new object under pandas 3.0's Copy-on-Write semantics, and both support configurable sorting algorithms (`'quicksort'`, `'mergesort'`, `'heapsort'`, `'stable'`), null placement (`na_position='first'` or `'last'`), and index resetting (`ignore_index=True`).

**Beginner-Friendly Explanation:** Sorting is how you arrange your data in a specific order—like sorting a spreadsheet by name, or by date, or by price. Pandas gives you two main tools: `sort_values()` to sort by the data inside the columns, and `sort_index()` to sort by the row labels.

### Key Characteristics

- **Two sorting axes:** `sort_values()` sorts by column values; `sort_index()` sorts by index labels.
- **Multi-column sorting:** Pass a list to `by` and a matching list to `ascending` to sort by multiple columns with different directions.
- **Stability:** Only `'mergesort'` and `'stable'` preserve the original relative order of rows with equal keys; `'quicksort'` (the default) does not.
- **Null handling:** The `na_position` parameter controls whether `NaN` values appear first or last, independent of the `ascending` parameter.
- **`inplace` is deprecated:** Since pandas 3.1.0, the `inplace` parameter is deprecated and will be removed in pandas 4.0; use reassignment instead.

### Prerequisites

- Basic Python knowledge (lists, dictionaries, functions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of `NaN` and missing data handling.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Analysis:** Sorting is a prerequisite for rank-based analysis, top-N queries, and time-series alignment.
- **Data Visualization:** Sorted data produces more readable bar charts and line plots.
- **ETL Pipelines:** Sorting ensures deterministic output before loading into databases or exporting to CSV.
- **Machine Learning:** Sorting by time or ID is often required before creating lag features or time-based train-test splits.

### Core Concepts / Features

1. Value Sorting: `sort_values()`
2. Coordinate Sorting: `sort_index()`
3. Custom Hierarchies: Mixed Ascending/Descending Conditions
4. Algorithmic Selection: Choosing the Right Sort Algorithm
5. Null Alignment: The `na_position` Parameter


## 1. Value Sorting: Ordering Rows by Column Values

### Definitions

**Core Definition:** `sort_values()` sorts the rows of a DataFrame (or the elements of a Series) by the values in one or more specified columns, in ascending or descending order.

**Technical Definition:** `DataFrame.sort_values(by, *, axis=0, ascending=True, inplace=False, kind='quicksort', na_position='last', ignore_index=False, key=None)` sorts by the values along either axis. The `by` parameter accepts a string (single column) or a list of strings (multiple columns). When `axis=0` (default), `by` may contain index levels and/or column labels. The `ascending` parameter accepts a single boolean or a list of booleans matching the length of `by`. The `key` parameter accepts a vectorized callable that is applied to the values before sorting.

**Beginner-Friendly Explanation:** `sort_values()` is how you sort a spreadsheet by a column—like sorting a list of employees by salary, or a list of products by price. You can sort by one column or several, and you can choose ascending or descending order for each.

### Purposes

- To sort rows by the values in a single column in ascending or descending order.
- To sort rows by multiple columns with independent ascending/descending directions.
- To sort by a computed key using the `key` parameter (e.g., case-insensitive string sorting).
- To reset the index after sorting using `ignore_index=True`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.sort_values(by, *, axis=0, ascending=True, inplace=False,
                      kind='quicksort', na_position='last',
                      ignore_index=False, key=None)
Series.sort_values(*, axis=0, ascending=True, inplace=False,
                   kind='quicksort', na_position='last',
                   ignore_index=False, key=None)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `by` | String or list of strings: column name(s) or index level(s) to sort by |
| `axis` | `0` or `'index'` (default): sort rows; `1` or `'columns'`: sort columns |
| `ascending` | `True` (default) or `False`, or a list of booleans matching `by` |
| `inplace` | Deprecated since pandas 3.1.0; use reassignment |
| `kind` | `'quicksort'` (default), `'mergesort'`, `'heapsort'`, or `'stable'` |
| `na_position` | `'last'` (default) or `'first'` |
| `ignore_index` | `True`: reset index to 0, 1, …, n−1 |
| `key` | Vectorized callable applied to values before sorting |

**Syntax Rules:**

- `by` must be a column label (string) or a list of column labels.
- When `ascending` is a list, it must match the length of `by`.
- The `key` function must expect a Series and return an array-like of the same shape.
- `inplace=True` is deprecated in pandas 3.1.0+ and will be removed in pandas 4.0.

**Constraints and Limitations:**

- Sorting by multiple columns with `kind` is only applied when sorting on a single column or label; for multi-column sorts, a stable algorithm is used internally.
- The `key` function is applied to each column in `by` independently.
- Sorting does not modify the original DataFrame unless `inplace=True` (deprecated).

### Annotated Code Examples

**Example 1: Basic Single-Column Sort**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [30, 25, 35, 28],
    'salary': [60000, 50000, 80000, 55000]
})

# Sort by age (ascending)
print(df.sort_values('age'))
# Expected output:
#       name  age  salary
# 1      Bob   25   50000
# 3    Diana   28   55000
# 0    Alice   30   60000
# 2  Charlie   35   80000

# Sort by salary (descending)
print(df.sort_values('salary', ascending=False))
# Expected output:
#       name  age  salary
# 2  Charlie   35   80000
# 0    Alice   30   60000
# 3    Diana   28   55000
# 1      Bob   25   50000
```

**Why this output:** `sort_values('age')` sorts rows by the `age` column in ascending order. `sort_values('salary', ascending=False)` sorts in descending order.

**Example 2: Multi-Column Sort with Mixed Directions**

```python
# Sort by age ascending, then salary descending
print(df.sort_values(['age', 'salary'], ascending=[True, False]))
# Expected output:
#       name  age  salary
# 1      Bob   25   50000
# 3    Diana   28   55000
# 0    Alice   30   60000
# 2  Charlie   35   80000
```

**Why this output:** The list `[True, False]` applies ascending order to `age` and descending to `salary`. Since all ages are unique in this example, the secondary sort on `salary` does not change the order, but it would if there were ties in `age`.

**Example 3: Using the key Parameter for Case-Insensitive Sorting**

```python
df2 = pd.DataFrame({
    'name': ['alice', 'Bob', 'Charlie', 'diana']
})

# Case-sensitive sort (default)
print(df2.sort_values('name'))
# Expected output:
#       name
# 2  Charlie
# 1      Bob
# 3    diana
# 0    alice

# Case-insensitive sort using key
print(df2.sort_values('name', key=lambda x: x.str.lower()))
# Expected output:
#       name
# 0    alice
# 1      Bob
# 2  Charlie
# 3    diana
```

**Why this output:** Without the key, uppercase letters sort before lowercase letters. With `key=lambda x: x.str.lower()`, all names are lowercased before sorting, producing alphabetical order regardless of case.

### Real-World Cases

- **Employee records:** Sort by department (ascending) then salary (descending) to find the highest-paid employee in each department.
- **E-commerce:** Sort products by price or customer rating to display the most relevant items first.
- **Financial data:** Sort transactions by date to prepare time-series analysis.

### References

- pandas.DataFrame.sort_values — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_values.html
- pandas.Series.sort_values — https://pandas.pydata.org/docs/reference/api/pandas.Series.sort_values.html


## 2. Coordinate Sorting: sort_index()

### Definitions

**Core Definition:** `sort_index()` sorts a DataFrame or Series by its index labels (row labels or column labels), rather than by the values inside the columns.

**Technical Definition:** `DataFrame.sort_index(*, axis=0, level=None, ascending=True, inplace=False, kind='quicksort', na_position='last', sort_remaining=True, ignore_index=False, key=None)` sorts the object by labels along the specified axis. The `level` parameter allows sorting on a specific level of a MultiIndex. The `sort_remaining` parameter controls whether other levels are sorted after the specified level.

**Beginner-Friendly Explanation:** `sort_index()` is for when you want to sort by the row labels (the index) rather than by the data inside the columns. For example, if your rows are labeled with dates, `sort_index()` arranges them chronologically.

### Purposes

- To sort a DataFrame or Series by its index labels in ascending or descending order.
- To restore chronological order after operations that shuffle the index.
- To sort specific levels of a MultiIndex.
- To align data before time-series operations that require a monotonic index.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.sort_index(*, axis=0, level=None, ascending=True,
                     inplace=False, kind='quicksort',
                     na_position='last', sort_remaining=True,
                     ignore_index=False, key=None)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `axis` | `0` or `'index'` (default): sort row labels; `1` or `'columns'`: sort column labels |
| `level` | Integer, level name, or list: sort on specific MultiIndex level(s) |
| `ascending` | `True` (default) or `False`, or a list for MultiIndex levels |
| `sort_remaining` | `True` (default): sort other levels after the specified level |
| `na_position` | `'last'` (default) or `'first'`; not implemented for MultiIndex |
| `ignore_index` | `True`: reset index to 0, 1, …, n−1 |
| `key` | Vectorized callable applied to index values before sorting |

**Syntax Rules:**

- When the index is a MultiIndex, `ascending` can be a list to control each level independently.
- `na_position` is not implemented for MultiIndex.
- `level` accepts level names if the MultiIndex levels are named.
- The `key` function is applied per level for MultiIndex inputs.

**Constraints and Limitations:**

- `sort_index()` does not sort by column values; use `sort_values()` for that.
- Sorting a MultiIndex by one level while `sort_remaining=True` may reorder the other levels, which can affect slicing operations.
- `inplace=True` is deprecated in pandas 3.1.0+.

### Annotated Code Examples

**Example 1: Basic Index Sorting**

```python
import pandas as pd

# Create a Series with unsorted index
s = pd.Series(['a', 'b', 'c', 'd'], index=[3, 2, 1, 4])
print("Original:")
print(s)
# Expected output:
# 3    a
# 2    b
# 1    c
# 4    d
# dtype: object

# Sort by index
print("\nSorted:")
print(s.sort_index())
# Expected output:
# 1    c
# 2    b
# 3    a
# 4    d
# dtype: object
```

**Why this output:** `sort_index()` rearranges the rows so that the index labels are in ascending order: 1, 2, 3, 4. The values (`c`, `b`, `a`, `d`) move with their labels.

**Example 2: Sorting a MultiIndex**

```python
# Create a DataFrame with a MultiIndex
df = pd.DataFrame({
    'value': [10, 20, 30, 40]
}, index=pd.MultiIndex.from_tuples([
    ('B', 2), ('A', 1), ('B', 1), ('A', 2)
], names=['letter', 'number']))

print("Original:")
print(df)
# Expected output:
#               value
# letter number
# B      2         10
# A      1         20
# B      1         30
# A      2         40

# Sort by both levels
print("\nSorted by both levels:")
print(df.sort_index())
# Expected output:
#               value
# letter number
# A      1         20
#        2         40
# B      1         30
#        2         10
```

**Why this output:** `sort_index()` sorts the MultiIndex first by `letter` (A before B), then by `number` (1 before 2) within each letter.

**Example 3: Sorting by a Specific MultiIndex Level**

```python
# Sort by the 'number' level only
print(df.sort_index(level='number'))
# Expected output:
#               value
# letter number
# A      1         20
# B      1         30
# B      2         10
# A      2         40
```

**Why this output:** `level='number'` sorts by the `number` level first (1 then 2), and `sort_remaining=True` (default) sorts by `letter` within each number group.

### Real-World Cases

- **Time series:** After resampling or slicing, restore chronological order with `sort_index()` before computing rolling statistics.
- **Database exports:** Sort by primary key index to ensure deterministic output.
- **MultiIndex DataFrames:** Sort by a specific level (e.g., “Year”) before slicing or grouping.

### References

- pandas.DataFrame.sort_index — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_index.html
- pandas.Series.sort_index — https://pandas.pydata.org/docs/reference/api/pandas.Series.sort_index.html


## 3. Custom Hierarchies: Mixed Ascending/Descending Conditions

### Definitions

**Core Definition:** Custom hierarchy sorting allows different sort directions for different columns by passing a list of booleans to the `ascending` parameter, enabling mixed ascending/descending sorts in a single operation.

**Technical Definition:** When `ascending` is a list, it must match the length of the `by` list. Each boolean corresponds to the sort direction for the column at the same position in `by`. For example, `ascending=[True, False]` sorts the first column ascending and the second column descending.

**Beginner-Friendly Explanation:** Sometimes you want to sort one column from A to Z and another column from highest to lowest. Instead of sorting twice, you pass a list like `[True, False]` to `ascending` to control each column separately.

### Purposes

- To sort by multiple columns with different directions in a single operation.
- To implement complex ranking rules (e.g., sort by department ascending, then by salary descending).
- To prioritise the most important sort criterion while using secondary criteria for ties.
- To control the sort direction of each level in a MultiIndex sort.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Mixed ascending/descending for multiple columns
df.sort_values(by=['col1', 'col2'], ascending=[True, False])

# Mixed ascending/descending for MultiIndex levels
df.sort_index(ascending=[True, False])
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `by` | List of column names |
| `ascending` | List of booleans matching the length of `by` |
| `True` | Ascending order (smallest to largest, A to Z) |
| `False` | Descending order (largest to smallest, Z to A) |

**Syntax Rules:**

- The length of `ascending` must match the length of `by` when it is a list.
- A single boolean applies to all columns in `by`.
- For MultiIndex `sort_index()`, `ascending` can be a list to control each level independently.

**Constraints and Limitations:**

- The `kind` parameter (sorting algorithm) is only applied when sorting on a single column or label; for multi-column sorts, a stable algorithm is used internally.
- Mixed directions require a list; passing a single boolean to a multi-column sort applies the same direction to all columns.

### Annotated Code Examples

**Example 1: Mixed Ascending/Descending Sort**

```python
import pandas as pd

df = pd.DataFrame({
    'department': ['Sales', 'Sales', 'Eng', 'Eng', 'Sales'],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'salary': [60000, 50000, 80000, 75000, 55000]
})

# Sort by department ascending, then salary descending
result = df.sort_values(
    by=['department', 'salary'],
    ascending=[True, False]
)
print(result)
# Expected output:
#   department     name  salary
# 2        Eng  Charlie   80000
# 3        Eng    Diana   75000
# 0      Sales    Alice   60000
# 4      Sales      Eve   55000
# 1      Sales      Bob   50000
```

**Why this output:** The `department` column is sorted ascending (Eng before Sales), and within each department, the `salary` column is sorted descending (highest first). This is a classic “top earner per department” sort.

**Example 2: Three-Column Mixed Sort**

```python
df2 = pd.DataFrame({
    'year': [2024, 2024, 2024, 2025, 2025],
    'quarter': ['Q1', 'Q2', 'Q1', 'Q1', 'Q2'],
    'revenue': [100, 200, 150, 300, 250]
})

# Sort by year ascending, quarter descending, revenue ascending
result = df2.sort_values(
    by=['year', 'quarter', 'revenue'],
    ascending=[True, False, True]
)
print(result)
# Expected output:
#    year quarter  revenue
# 1  2024      Q2      200
# 0  2024      Q1      100
# 2  2024      Q1      150
# 3  2025      Q1      300
# 4  2025      Q2      250
```

**Why this output:** Year is sorted ascending (2024 before 2025). Within each year, quarter is sorted descending (Q2 before Q1). Within each year-quarter, revenue is sorted ascending.

**Example 3: Mixed MultiIndex Sort**

```python
# MultiIndex with mixed ascending/descending
df3 = pd.DataFrame({
    'value': [10, 20, 30, 40]
}, index=pd.MultiIndex.from_tuples([
    ('B', 2), ('A', 1), ('B', 1), ('A', 2)
], names=['letter', 'number']))

# Sort letter ascending, number descending
print(df3.sort_index(ascending=[True, False]))
# Expected output:
#               value
# letter number
# A      2         40
#        1         20
# B      2         10
#        1         30
```

**Why this output:** The `letter` level is sorted ascending (A before B), and the `number` level is sorted descending (2 before 1) within each letter.

### Real-World Cases

- **Sales reporting:** Sort by region ascending, then revenue descending to show top-performing products per region.
- **HR analytics:** Sort by department ascending, then tenure descending to identify the most senior employees per department.
- **Financial data:** Sort by year ascending, quarter descending to show the most recent quarters first within each year.

### References

- pandas.DataFrame.sort_values (ascending parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_values.html
- pandas.DataFrame.sort_index (ascending parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_index.html


## 4. Algorithmic Selection: Choosing the Right Sort Algorithm

### Definitions

**Core Definition:** The `kind` parameter selects the sorting algorithm used by pandas, determining performance characteristics and whether the sort is stable (preserving the original relative order of rows with equal keys).

**Technical Definition:** The `kind` parameter accepts `'quicksort'` (default), `'mergesort'`, `'heapsort'`, or `'stable'`. Only `'mergesort'` and `'stable'` are stable algorithms; the others do not guarantee that ties preserve their original order. For DataFrames, the algorithm choice is only applied when sorting on a single column or label; multi-column sorts use a stable algorithm internally.

**Beginner-Friendly Explanation:** A “stable” sort keeps rows with the same sort value in their original order. If you sort by age and two people are both 25, a stable sort will keep them in the order they appeared originally. A non-stable sort might swap them. Use `kind='mergesort'` or `kind='stable'` when you need this guarantee.

### Purposes

- To preserve the original relative order of rows with equal sort keys using a stable algorithm.
- To choose a faster algorithm (quicksort) when stability is not required.
- To ensure deterministic output when rows have tied sort values.
- To maintain the order of secondary criteria when performing multi-stage sorts.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
df.sort_values(by='col', kind='mergesort')   # stable
df.sort_values(by='col', kind='stable')      # alias for mergesort
df.sort_values(by='col', kind='quicksort')   # default, not stable
df.sort_values(by='col', kind='heapsort')    # not stable
```

**Component Breakdown:**

| Algorithm | Stable? | Description |
|-----------|---------|-------------|
| `'quicksort'` | No | Default; fast on average but not stable |
| `'mergesort'` | Yes | Stable; preserves order of equal keys |
| `'heapsort'` | No | Not stable; O(n log n) worst-case |
| `'stable'` | Yes | Alias for `'mergesort'` |

**Syntax Rules:**

- The `kind` parameter is only applied when sorting on a single column or label for DataFrames.
- For MultiIndex sorting, `kind` is not applied.
- `'stable'` and `'mergesort'` are equivalent in pandas.

**Constraints and Limitations:**

- Quicksort is faster than mergesort on average but not stable.
- For multi-column sorts, pandas uses a stable algorithm internally regardless of the `kind` parameter.
- The `kind` parameter does not affect the correctness of the sort, only the tie-breaking behaviour and performance.

### Annotated Code Examples

**Example 1: Stable vs. Non-Stable Sort**

```python
import pandas as pd

df = pd.DataFrame({
    'grade': ['A', 'B', 'A', 'B', 'A'],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve']
})

print("Original:")
print(df)
# Expected output:
#   grade     name
# 0     A    Alice
# 1     B      Bob
# 2     A  Charlie
# 3     B    Diana
# 4     A      Eve

# Non-stable sort (default quicksort)
print("\nQuicksort (not stable):")
print(df.sort_values('grade', kind='quicksort'))
# Expected output (order of A's may vary):
#   grade     name
# 0     A    Alice
# 2     A  Charlie
# 4     A      Eve
# 1     B      Bob
# 3     B    Diana

# Stable sort (mergesort)
print("\nMergesort (stable):")
print(df.sort_values('grade', kind='mergesort'))
# Expected output:
#   grade     name
# 0     A    Alice
# 2     A  Charlie
# 4     A      Eve
# 1     B      Bob
# 3     B    Diana
```

**Why this output:** With a stable sort, the A's and B's preserve their original relative order (Alice, Charlie, Eve for A; Bob, Diana for B). With quicksort, the order within each grade group may differ.

**Example 2: Preserving Secondary Sort Order with Stable Sort**

```python
df2 = pd.DataFrame({
    'team': ['Red', 'Blue', 'Red', 'Blue', 'Red'],
    'score': [10, 20, 10, 20, 10],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve']
})

# Sort by score using stable sort to preserve team order within ties
result = df2.sort_values('score', kind='mergesort')
print(result)
# Expected output:
#    team  score     name
# 0   Red     10    Alice
# 2   Red     10  Charlie
# 4   Red     10      Eve
# 1  Blue     20      Bob
# 3  Blue     20    Diana
```

**Why this output:** Within each score group (10 and 20), the original row order is preserved because mergesort is stable. If quicksort were used, the order within each score group could be different.

**Example 3: Multi-Column Sort Always Uses Stable Algorithm**

```python
# Multi-column sort ignores kind parameter for stability
result = df.sort_values(['grade', 'name'], kind='quicksort')
print(result)
# Expected output:
#   grade     name
# 0     A    Alice
# 2     A  Charlie
# 4     A      Eve
# 1     B      Bob
# 3     B    Diana
```

**Why this output:** Even though `kind='quicksort'` is specified, pandas uses a stable algorithm internally for multi-column sorts, so the result is deterministic and preserves the original order of ties.

### Real-World Cases

- **Financial transactions:** Sort by transaction amount using a stable sort to preserve the original time-order of equal-value transactions.
- **Rankings:** When computing ranks with tied values, use a stable sort to ensure consistent tie-breaking.
- **Multi-stage sorting:** Sort by a primary criterion, then by a secondary criterion using stable sort to preserve the primary sort order.

### References

- pandas.DataFrame.sort_values (kind parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_values.html
- numpy.sort — https://numpy.org/doc/stable/reference/generated/numpy.sort.html


## 5. Null Alignment: Controlling na_position

### Definitions

**Core Definition:** The `na_position` parameter controls whether `NaN` values are placed at the beginning (`'first'`) or end (`'last'`) of the sorted result, independent of the `ascending` parameter.

**Technical Definition:** `na_position` accepts `'first'` or `'last'` (default `'last'`). It works independently of `ascending`: whether you sort ascending or descending, `na_position` determines where `NaN` values land. This parameter is not implemented for MultiIndex sorting.

**Beginner-Friendly Explanation:** When you sort data that has missing values, you can choose whether the missing values go at the top or the bottom of the result. By default, they go at the bottom.

### Purposes

- To control the placement of `NaN` values in sorted output.
- To ensure missing data appears at the beginning of a sorted list (e.g., for data quality inspection).
- To keep missing data at the end of a sorted list for clean presentation.
- To separate the treatment of missing values from the sort direction.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
df.sort_values(by='col', na_position='first')
df.sort_values(by='col', na_position='last')
df.sort_values(by='col', ascending=False, na_position='first')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `na_position='last'` | Default; `NaN` values at the end |
| `na_position='first'` | `NaN` values at the beginning |
| Independent of `ascending` | `na_position` controls NaN placement regardless of sort direction |

**Syntax Rules:**

- `na_position` must be `'first'` or `'last'`.
- It is independent of `ascending`: changing `ascending` does not change where `NaN` values appear.
- Not implemented for MultiIndex sorting.

**Constraints and Limitations:**

- `na_position` only affects `NaN` values in the sort column; it does not affect `NaN` values in other columns.
- For MultiIndex sorting, `na_position` is not supported.
- The default `na_position='last'` places `NaN` at the end regardless of ascending or descending order.

### Annotated Code Examples

**Example 1: Default na_position='last'**

```python
import pandas as pd
import numpy as np

s = pd.Series([np.nan, 1, 3, 10, 5])
print("Original:")
print(s)
# Expected output:
# 0     NaN
# 1     1.0
# 2     3.0
# 3    10.0
# 4     5.0
# dtype: float64

# Ascending sort with default na_position='last'
print("\nAscending (na_position='last'):")
print(s.sort_values(ascending=True))
# Expected output:
# 1     1.0
# 2     3.0
# 4     5.0
# 3    10.0
# 0     NaN
# dtype: float64

# Descending sort with default na_position='last'
print("\nDescending (na_position='last'):")
print(s.sort_values(ascending=False))
# Expected output:
# 3    10.0
# 4     5.0
# 2     3.0
# 1     1.0
# 0     NaN
# dtype: float64
```

**Why this output:** In both ascending and descending sorts, the `NaN` value appears at the end because `na_position='last'` is the default. The `ascending` parameter only affects the ordering of non-NaN values.

**Example 2: na_position='first'**

```python
# Ascending sort with na_position='first'
print("Ascending (na_position='first'):")
print(s.sort_values(ascending=True, na_position='first'))
# Expected output:
# 0     NaN
# 1     1.0
# 2     3.0
# 4     5.0
# 3    10.0
# dtype: float64

# Descending sort with na_position='first'
print("\nDescending (na_position='first'):")
print(s.sort_values(ascending=False, na_position='first'))
# Expected output:
# 0     NaN
# 3    10.0
# 4     5.0
# 2     3.0
# 1     1.0
# dtype: float64
```

**Why this output:** With `na_position='first'`, the `NaN` value appears at the beginning in both ascending and descending sorts. The non-NaN values are sorted according to the `ascending` parameter.

**Example 3: na_position with a DataFrame**

```python
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'score': [85, np.nan, 92, np.nan]
})

# Sort by score with NaN at the end
print(df.sort_values('score', na_position='last'))
# Expected output:
#       name  score
# 0    Alice   85.0
# 2  Charlie   92.0
# 1      Bob    NaN
# 3    Diana    NaN

# Sort by score with NaN at the beginning
print("\n" + df.sort_values('score', na_position='first').to_string())
# Expected output:
#       name  score
# 1      Bob    NaN
# 3    Diana    NaN
# 0    Alice   85.0
# 2  Charlie   92.0
```

**Why this output:** In the first sort, rows with `NaN` scores (Bob and Diana) appear at the end. In the second sort, they appear at the beginning. The rows with valid scores are sorted ascending in both cases.

### Real-World Cases

- **Data quality inspection:** Use `na_position='first'` to quickly see which rows have missing values in a sorted column.
- **Report generation:** Use `na_position='last'` (default) to keep reports clean with missing data at the bottom.
- **Financial analysis:** Sort by price ascending with `na_position='last'` to show the cheapest items first, with missing prices at the end.

### References

- pandas.DataFrame.sort_values (na_position parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_values.html
- pandas.Series.sort_values (na_position parameter) — https://pandas.pydata.org/docs/reference/api/pandas.Series.sort_values.html


## References

- pandas.DataFrame.sort_values — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_values.html
- pandas.Series.sort_values — https://pandas.pydata.org/docs/reference/api/pandas.Series.sort_values.html
- pandas.DataFrame.sort_index — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.sort_index.html
- pandas.Series.sort_index — https://pandas.pydata.org/docs/reference/api/pandas.Series.sort_index.html
- pandas.Index.sort_values — https://pandas.pydata.org/docs/reference/api/pandas.Index.sort_values.html
- numpy.sort — https://numpy.org/doc/stable/reference/generated/numpy.sort.html
- PDEP-8 In-place methods in pandas — https://pandas.pydata.org/pdeps/0008-inplace-methods-in-pandas.html
- Pandas Sort Values: Complete Guide to Sorting DataFrames in Python — https://docs.kanaries.net/articles/pandas-sort-values
- How to sort by multiple columns? | LabEx — https://labex.io/tutorials/python-how-to-sort-by-multiple-columns-421233