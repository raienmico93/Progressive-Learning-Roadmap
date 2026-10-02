# Pandas Duplicate Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Duplicate data in pandas refers to rows or index labels that appear more than once in a DataFrame or Series. pandas provides methods to detect, count, and remove duplicates, with configurable rules for which occurrence to retain.

**Technical Definition:** Duplicate detection is implemented via the `duplicated()` method on `DataFrame`, `Series`, and `Index` objects, returning a boolean mask where `True` indicates a duplicate occurrence. Removal is performed by `drop_duplicates()`, which returns a new object with duplicate rows or values eliminated. The `subset` parameter restricts the comparison to specific columns, the `keep` parameter controls which occurrence is retained (`'first'`, `'last'`, or `False` for none), and the `ignore_index` parameter controls whether the resulting axis is renumbered. Duplicate index labels are handled separately via `Index.duplicated()` and boolean filtering, as they affect indexing semantics differently from duplicate row values.

**Beginner-Friendly Explanation:** Duplicates are rows that look the same (or have the same values in certain columns). pandas can tell you where they are, count them, and remove them. You can choose which copy to keep—the first, the last, or none of them.

### Key Characteristics

- **Row-based by default:** `duplicated()` and `drop_duplicates()` compare entire rows across all columns unless `subset` is specified.
- **Index is ignored:** The index is not considered part of the row when comparing duplicates; only column values matter.
- **Configurable retention:** The `keep` parameter determines which occurrence is marked as non-duplicate: first, last, or none.
- **Index duplicates are separate:** Duplicate index labels are a different phenomenon from duplicate row values and require `Index.duplicated()` for detection.
- **Hashability requirement:** Columns used in `subset` must be hashable; unhashable types (lists, dicts) raise `TypeError`.

### Prerequisites

- Basic Python knowledge (lists, dictionaries, boolean indexing).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of boolean masking and the `.loc` accessor.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Cleaning & Preprocessing:** Duplicate removal is a standard step before analysis or modeling.
- **Data Integration:** When merging datasets from multiple sources, duplicates often arise and must be resolved.
- **Database Operations:** pandas duplicate handling mirrors SQL `DISTINCT` and `GROUP BY` semantics.
- **Data Quality Assurance:** Duplicate detection is a key metric in data validation pipelines.

### Core Concepts / Features

1. Boolean Indicators: `duplicated()` and Frequency Tracking
2. Record Elimination: `drop_duplicates()`
3. Multi-Column Subsetting
4. Conflict Resolution Strategy: `keep` Parameter
5. Handling Duplicates in Non-Indexed Rows vs. Duplicate Index Labels


## 1. Boolean Indicators: Identifying Redundant Records

### Definitions

**Core Definition:** `duplicated()` returns a boolean Series where `True` marks each duplicate occurrence and `False` marks the first (or last, or all, depending on `keep`) occurrence of each unique row.

**Technical Definition:** `DataFrame.duplicated(subset=None, keep='first')` returns a `Series` of boolean values aligned with the DataFrame's index. By default, for each set of duplicated values, the first occurrence is set to `False` and all others to `True`. When `keep='last'`, the last occurrence is `False` and all others are `True`. When `keep=False`, all occurrences are marked `True`. The `subset` parameter restricts the comparison to a specified column label or sequence of labels.

**Beginner-Friendly Explanation:** `duplicated()` gives you a True/False answer for each row: “Is this row a repeat of an earlier one?” By default, the first time a value appears it is not considered a duplicate; every subsequent appearance is.

### Purposes

- To create a boolean mask identifying which rows are duplicates of earlier rows.
- To count the total number of duplicate rows in a DataFrame.
- To filter rows based on duplicate status using boolean indexing.
- To track group frequencies by combining `duplicated()` with `value_counts()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.duplicated(subset=None, keep='first')
Series.duplicated(*, keep='first')
Index.duplicated(keep='first')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `subset` | Column label or sequence of labels to consider; default `None` (all columns) |
| `keep` | `'first'` (default): mark duplicates as `True` except first occurrence; `'last'`: mark duplicates as `True` except last occurrence; `False`: mark all duplicates as `True` |
| Returns | `Series` of booleans for each row (DataFrame/Series); `ndarray` of booleans for Index |

**Syntax Rules:**

- `duplicated()` returns a Series aligned with the original index.
- For `keep='first'`, the first occurrence of each unique row is `False`; all subsequent occurrences are `True`.
- For `keep='last'`, the last occurrence is `False`; all earlier occurrences are `True`.
- For `keep=False`, every occurrence in a duplicate group is `True`.
- `subset` must contain hashable column labels.

**Constraints and Limitations:**

- The index is ignored when comparing rows; two rows with identical values but different index labels are still considered duplicates.
- `duplicated()` does not modify the DataFrame; it only returns a boolean mask.
- Columns with unhashable types (lists, dicts) raise `TypeError` when used in `subset`.

### Annotated Code Examples

**Example 1: Basic Duplicate Detection**

```python
import pandas as pd

# Create a DataFrame with duplicate rows
df = pd.DataFrame({
    'brand': ['Yum Yum', 'Yum Yum', 'Indomie', 'Indomie', 'Indomie'],
    'style': ['cup', 'cup', 'cup', 'pack', 'pack'],
    'rating': [4, 4, 3.5, 15, 5]
})

print(df)
# Expected output:
#      brand style  rating
# 0  Yum Yum   cup     4.0
# 1  Yum Yum   cup     4.0
# 2  Indomie   cup     3.5
# 3  Indomie  pack    15.0
# 4  Indomie  pack     5.0

# Detect duplicates (default keep='first')
print(df.duplicated())
# Expected output:
# 0    False
# 1     True
# 2    False
# 3    False
# 4    False
# dtype: bool
```

**Why this output:** Rows 0 and 1 are identical across all columns. With `keep='first'`, row 0 (the first occurrence) is marked `False`, and row 1 (the duplicate) is marked `True`. Rows 2, 3, and 4 are unique, so they are all `False`.

**Example 2: Counting Duplicates**

```python
# Count total duplicate rows
print(f"Total duplicate rows: {df.duplicated().sum()}")
# Expected output: Total duplicate rows: 1

# Track frequency with value_counts
print(df.duplicated().value_counts())
# Expected output:
# False    4
# True     1
# dtype: int64
```

**Why this output:** `duplicated().sum()` counts the number of `True` values (1). `value_counts()` shows the distribution: 4 non-duplicate rows and 1 duplicate row.

**Example 3: Using keep=False for All Duplicates**

```python
# Mark all duplicates as True
print(df.duplicated(keep=False))
# Expected output:
# 0     True
# 1     True
# 2    False
# 3    False
# 4    False
# dtype: bool
```

**Why this output:** With `keep=False`, both row 0 and row 1 are marked `True` because both participate in the duplicate group. This is useful for isolating all rows that belong to any duplicate group.

### Real-World Cases

- **Data quality checks:** Before analysis, run `df.duplicated().sum()` to quantify duplicate records and decide on a cleaning strategy.
- **Merge validation:** After merging two DataFrames, check for duplicates introduced by many-to-many joins using `duplicated(keep=False)`.
- **Survey data:** Identify respondents who submitted the same answers multiple times using `duplicated(subset=['respondent_id'])`.

### References

- pandas.DataFrame.duplicated — https://pandas.pydata.org/pandas-docs/version/1.5/reference/api/pandas.DataFrame.duplicated.html
- pandas.Series.duplicated — https://pandas.pydata.org/docs/reference/api/pandas.Series.duplicated.html
- pandas.Index.duplicated — https://pandas.pydata.org/pandas-docs/version/1.1.1/reference/api/pandas.Index.duplicated.html


## 2. Record Elimination: Dropping Duplicates with `drop_duplicates()`

### Definitions

**Core Definition:** `drop_duplicates()` returns a new DataFrame or Series with duplicate rows or values removed, retaining one occurrence per unique combination based on the `keep` parameter.

**Technical Definition:** `DataFrame.drop_duplicates(subset=None, *, keep='first', inplace=False, ignore_index=False)` removes duplicate rows from a DataFrame. The index, including time indexes, is ignored when determining duplicates. `Series.drop_duplicates(*, keep='first', inplace=False, ignore_index=False)` removes duplicate values from a Series. `Index.drop_duplicates(*, keep='first')` removes duplicate labels from an Index. The `ignore_index` parameter (pandas 2.0+) renumbers the resulting axis from 0 to n−1.

**Beginner-Friendly Explanation:** `drop_duplicates()` is the “delete the repeats” button. It keeps one copy of each unique row and throws away the rest. You can choose which copy to keep.

### Purposes

- To remove exact duplicate rows from a DataFrame in one operation.
- To remove duplicate values from a Series while preserving order.
- To remove duplicate index labels from an Index object.
- To clean a DataFrame and optionally reset the index with `ignore_index=True`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.drop_duplicates(subset=None, *, keep='first', inplace=False, ignore_index=False)
Series.drop_duplicates(*, keep='first', inplace=False, ignore_index=False)
Index.drop_duplicates(*, keep='first')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `subset` | Column label or sequence of labels to consider; default `None` (all columns) |
| `keep` | `'first'` (default): keep first occurrence; `'last'`: keep last occurrence; `False`: drop all duplicates |
| `inplace` | If `True`, modifies the DataFrame in place; default `False` |
| `ignore_index` | If `True`, the resulting axis is labeled 0, 1, …, n−1; default `False` |
| Returns | DataFrame or Series with duplicates removed; `None` if `inplace=True` |

**Syntax Rules:**

- By default, `drop_duplicates()` uses all columns to determine duplicates.
- The index is not considered part of the row; duplicate index labels do not affect row duplicate detection.
- `inplace=True` is deprecated in pandas 3.0; prefer reassignment.
- `ignore_index=True` is only available in pandas 2.0 and later.

**Constraints and Limitations:**

- Dropping duplicates removes data; ensure the loss is acceptable for your analysis.
- `inplace=True` modifies the original object and returns `None`, which can break method chaining.
- `Index.drop_duplicates()` returns a new Index; it does not modify in place.

### Annotated Code Examples

**Example 1: Basic Drop Duplicates**

```python
import pandas as pd

df = pd.DataFrame({
    'brand': ['Yum Yum', 'Yum Yum', 'Indomie', 'Indomie', 'Indomie'],
    'style': ['cup', 'cup', 'cup', 'pack', 'pack'],
    'rating': [4, 4, 3.5, 15, 5]
})

# Drop duplicate rows (keep first)
print(df.drop_duplicates())
# Expected output:
#      brand style  rating
# 0  Yum Yum   cup     4.0
# 2  Indomie   cup     3.5
# 3  Indomie  pack    15.0
# 4  Indomie  pack     5.0
```

**Why this output:** Row 1 is an exact duplicate of row 0, so it is removed. Rows 2, 3, and 4 are unique and retained. The original index labels (0, 2, 3, 4) are preserved.

**Example 2: Dropping with keep='last' and keep=False**

```python
# Keep last occurrence
print(df.drop_duplicates(keep='last'))
# Expected output:
#      brand style  rating
# 1  Yum Yum   cup     4.0
# 2  Indomie   cup     3.5
# 3  Indomie  pack    15.0
# 4  Indomie  pack     5.0

# Drop all duplicates (keep neither)
print(df.drop_duplicates(keep=False))
# Expected output:
#      brand style  rating
# 2  Indomie   cup     3.5
# 3  Indomie  pack    15.0
# 4  Indomie  pack     5.0
```

**Why this output:** `keep='last'` retains row 1 (the last occurrence of the Yum Yum/cup/4.0 combination) and removes row 0. `keep=False` removes both row 0 and row 1 because they form a duplicate group, leaving only the unique rows.

**Example 3: Using ignore_index**

```python
# Drop duplicates and reset index
print(df.drop_duplicates(ignore_index=True))
# Expected output:
#      brand style  rating
# 0  Yum Yum   cup     4.0
# 1  Indomie   cup     3.5
# 2  Indomie  pack    15.0
# 3  Indomie  pack     5.0
```

**Why this output:** `ignore_index=True` renumbers the resulting rows from 0 to 3, discarding the original index labels (0, 2, 3, 4).

### Real-World Cases

- **Customer databases:** Remove duplicate customer records before analysis using `drop_duplicates(subset=['email'])`.
- **Transaction logs:** Remove duplicate transaction entries caused by system retries using `drop_duplicates(keep='last')` to keep the most recent record.
- **Survey data:** Remove duplicate responses using `drop_duplicates(subset=['respondent_id', 'question_id'])`.

### References

- pandas.DataFrame.drop_duplicates — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.drop_duplicates.html
- pandas.Series.drop_duplicates — https://pandas.pydata.org/docs/reference/api/pandas.Series.drop_duplicates.html
- pandas.Index.drop_duplicates — https://pandas.pydata.org/docs/reference/api/pandas.Index.drop_duplicates.html


## 3. Multi-Column Subsetting: Defining Distinct Definitions of a Duplicate

### Definitions

**Core Definition:** The `subset` parameter restricts duplicate detection to a specified column or sequence of columns, allowing you to define what constitutes a duplicate based on a subset of features rather than the entire row.

**Technical Definition:** When `subset` is provided to `duplicated()` or `drop_duplicates()`, pandas compares only the values in the specified columns. Rows with identical values in those columns are considered duplicates, even if other columns differ. The `subset` parameter accepts a single column label or a list of column labels; columns must be hashable.

**Beginner-Friendly Explanation:** Sometimes two rows are not exactly the same, but they represent the same entity (e.g., same customer but different order dates). The `subset` parameter lets you say “consider these rows duplicates if they have the same customer ID, even if other columns differ.”

### Purposes

- To define duplicates based on a subset of identifying columns rather than the entire row.
- To detect partial duplicates where only key columns match.
- To remove duplicates while preserving rows that differ in non-key columns.
- To validate uniqueness constraints on a combination of columns.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.duplicated(subset=['col1', 'col2'], keep='first')
DataFrame.drop_duplicates(subset=['col1', 'col2'], keep='first')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `subset` | Single label or list of labels to consider for duplicate detection |
| `keep` | Retention rule (`'first'`, `'last'`, `False`) |

**Syntax Rules:**

- `subset` must contain labels that exist in the DataFrame's columns.
- All columns in `subset` must be hashable (no lists or dicts).
- If `subset` is a single label, it can be passed as a string rather than a list.
- Columns not in `subset` are ignored when determining duplicates.

**Constraints and Limitations:**

- Using `subset` with unhashable columns raises `TypeError`.
- Partial duplicates (rows that match on `subset` but differ in other columns) are considered duplicates under `subset` semantics; choose `keep` carefully to retain the desired row.

### Annotated Code Examples

**Example 1: Subset on a Single Column**

```python
import pandas as pd

df = pd.DataFrame({
    'brand': ['Yum Yum', 'Yum Yum', 'Indomie', 'Indomie', 'Indomie'],
    'style': ['cup', 'cup', 'cup', 'pack', 'pack'],
    'rating': [4, 4, 3.5, 15, 5]
})

# Detect duplicates based on 'brand' only
print(df.duplicated(subset=['brand']))
# Expected output:
# 0    False
# 1     True
# 2    False
# 3     True
# 4     True
# dtype: bool

# Drop duplicates based on 'brand' only
print(df.drop_duplicates(subset=['brand']))
# Expected output:
#      brand style  rating
# 0  Yum Yum   cup     4.0
# 2  Indomie   cup     3.5
```

**Why this output:** With `subset=['brand']`, rows 0 and 1 share the brand `'Yum Yum'`; rows 2, 3, and 4 share the brand `'Indomie'`. With `keep='first'`, the first occurrence of each brand is retained (rows 0 and 2), and all other occurrences are marked as duplicates and dropped.

**Example 2: Subset on Multiple Columns**

```python
# Detect duplicates based on 'brand' and 'style'
print(df.duplicated(subset=['brand', 'style']))
# Expected output:
# 0    False
# 1     True
# 2    False
# 3    False
# 4     True
# dtype: bool

# Drop duplicates keeping last occurrence
print(df.drop_duplicates(subset=['brand', 'style'], keep='last'))
# Expected output:
#      brand style  rating
# 1  Yum Yum   cup     4.0
# 2  Indomie   cup     3.5
# 4  Indomie  pack     5.0
```

**Why this output:** With `subset=['brand', 'style']`, rows 0 and 1 are duplicates (same brand and style), rows 2 is unique, and rows 3 and 4 are duplicates. With `keep='last'`, row 1 is retained over row 0, and row 4 is retained over row 3.

**Example 3: Subset for Uniqueness Validation**

```python
# Check if a combination of columns is unique
is_unique = not df.duplicated(subset=['brand', 'style']).any()
print(f"Brand+style combination is unique: {is_unique}")
# Expected output: Brand+style combination is unique: False
```

**Why this output:** `duplicated(subset=['brand', 'style']).any()` returns `True` if any duplicate exists. Negating it gives `False`, indicating that the combination is not unique.

### Real-World Cases

- **Customer deduplication:** Use `subset=['email']` to remove duplicate customer records, keeping the most recent entry with `keep='last'`.
- **Transaction validation:** Use `subset=['transaction_id']` to verify that transaction IDs are unique before processing.
- **Survey analysis:** Use `subset=['respondent_id', 'question_id']` to detect duplicate responses to the same question.

### References

- pandas.DataFrame.duplicated (subset parameter) — https://pandas.pydata.org/pandas-docs/version/1.5/reference/api/pandas.DataFrame.duplicated.html
- pandas.DataFrame.drop_duplicates (subset parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.drop_duplicates.html


## 4. Conflict Resolution Strategy: The keep Parameter

### Definitions

**Core Definition:** The `keep` parameter determines which occurrence within each set of duplicates is retained: `'first'` keeps the first, `'last'` keeps the last, and `False` drops all occurrences.

**Technical Definition:** In `duplicated()`, `keep` controls which occurrences are marked as `True` (duplicates): `'first'` marks all except the first as `True`; `'last'` marks all except the last as `True`; `False` marks all occurrences as `True`. In `drop_duplicates()`, `keep` controls which occurrence is retained: `'first'` keeps the first occurrence; `'last'` keeps the last; `False` drops all duplicates, retaining none.

**Beginner-Friendly Explanation:** When you have multiple copies of the same row, which one do you want to keep? `keep='first'` keeps the top one, `keep='last'` keeps the bottom one, and `keep=False` keeps none of them.

### Purposes

- To retain the first occurrence of each duplicate group (default behaviour).
- To retain the most recent occurrence of each duplicate group using `keep='last'`.
- To remove all occurrences of duplicated records using `keep=False`.
- To isolate all rows that belong to any duplicate group for inspection.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.duplicated(keep='first' | 'last' | False)
DataFrame.drop_duplicates(keep='first' | 'last' | False)
```

**Component Breakdown:**

| `keep` Value | `duplicated()` Behaviour | `drop_duplicates()` Behaviour |
|--------------|--------------------------|-------------------------------|
| `'first'` (default) | Mark duplicates as `True` except first occurrence | Keep first occurrence |
| `'last'` | Mark duplicates as `True` except last occurrence | Keep last occurrence |
| `False` | Mark all duplicates as `True` | Drop all duplicates |

**Syntax Rules:**

- `keep` must be one of `'first'`, `'last'`, or `False`.
- The default is `'first'` for both methods.
- `keep=False` is useful for isolating all duplicate rows for inspection before deciding on a retention strategy.

**Constraints and Limitations:**

- `keep='last'` assumes the last occurrence is the most recent or authoritative; this may not always be true if the data is not ordered by time.
- `keep=False` removes all duplicate rows entirely, which may result in significant data loss.

### Annotated Code Examples

**Example 1: Comparing keep='first' and keep='last'**

```python
import pandas as pd

df = pd.DataFrame({
    'id': [1, 2, 1, 2, 1],
    'value': ['a', 'b', 'c', 'd', 'e']
})

print("Original:")
print(df)
# Expected output:
#    id value
# 0   1     a
# 1   2     b
# 2   1     c
# 3   2     d
# 4   1     e

# keep='first': keep first occurrence of each id
print("\nkeep='first':")
print(df.drop_duplicates(subset=['id'], keep='first'))
# Expected output:
#    id value
# 0   1     a
# 1   2     b

# keep='last': keep last occurrence of each id
print("\nkeep='last':")
print(df.drop_duplicates(subset=['id'], keep='last'))
# Expected output:
#    id value
# 3   2     d
# 4   1     e
```

**Why this output:** With `keep='first'`, the first occurrence of each `id` is retained (rows 0 and 1). With `keep='last'`, the last occurrence of each `id` is retained (rows 3 and 4).

**Example 2: Using keep=False to Identify All Duplicate Rows**

```python
# keep=False: mark all duplicates as True
print(df.duplicated(subset=['id'], keep=False))
# Expected output:
# 0     True
# 1     True
# 2     True
# 3     True
# 4     True
# dtype: bool

# Filter to see all rows that belong to any duplicate group
duplicates = df[df.duplicated(subset=['id'], keep=False)]
print(duplicates)
# Expected output:
#    id value
# 0   1     a
# 1   2     b
# 2   1     c
# 3   2     d
# 4   1     e
```

**Why this output:** With `keep=False`, every row whose `id` appears more than once is marked `True`. All five rows belong to duplicate groups (id 1 appears three times, id 2 appears twice), so all are marked `True`.

**Example 3: keep=False with drop_duplicates**

```python
# Drop all duplicates
print(df.drop_duplicates(subset=['id'], keep=False))
# Expected output:
# Empty DataFrame
# Columns: [id, value]
# Index: []
```

**Why this output:** With `keep=False`, `drop_duplicates()` removes all rows that belong to any duplicate group. Since every row has a duplicate `id`, the result is an empty DataFrame.

### Real-World Cases

- **Transaction processing:** Keep the last occurrence of a transaction ID using `keep='last'` to retain the most recent status.
- **Data inspection:** Use `keep=False` to isolate all duplicate rows before deciding on a cleaning strategy.
- **Customer records:** Keep the first occurrence of a customer ID using `keep='first'` if the data is ordered by entry time.

### References

- pandas.DataFrame.duplicated (keep parameter) — https://pandas.pydata.org/pandas-docs/version/1.5/reference/api/pandas.DataFrame.duplicated.html
- pandas.DataFrame.drop_duplicates (keep parameter) — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.drop_duplicates.html


## 5. Handling Duplicates in Non-Indexed Rows vs. Duplicate Index Labels

### Definitions

**Core Definition:** Duplicate row values (non-indexed duplicates) are rows with identical column values regardless of their index labels; duplicate index labels are repeated labels in the index itself, which affect indexing and alignment operations.

**Technical Definition:** Row-level duplicates are detected by `DataFrame.duplicated()`, which ignores the index and compares column values. Index-level duplicates are detected by `Index.duplicated()`, which returns a boolean array indicating whether a label is repeated. Duplicate index labels change the behaviour of indexing operations: scalar indexing may return a Series instead of a scalar, and `reindex()` raises `ValueError: cannot reindex from a duplicate axis`. pandas 1.2.0 introduced the `allows_duplicate_labels` flag to disallow duplicate labels entirely.

**Beginner-Friendly Explanation:** Duplicate rows are about the data inside the columns. Duplicate index labels are about the row names. They are different problems. Duplicate row values can be removed with `drop_duplicates()`, but duplicate index labels need to be handled separately using `Index.duplicated()` and boolean filtering.

### Purposes

- To distinguish between duplicate data values and duplicate index labels.
- To detect duplicate index labels using `Index.duplicated()`.
- To filter out duplicate index labels using boolean indexing.
- To understand how duplicate index labels affect indexing, alignment, and reindexing.
- To prevent duplicate labels from arising using `allows_duplicate_labels=False`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Detect duplicate index labels
Index.duplicated(keep='first')
Index.is_unique

# Filter out duplicate index labels
df.loc[~df.index.duplicated(), :]
df.loc[~df.index.duplicated(keep='last'), :]
df.loc[~df.index.duplicated(keep=False), :]

# Prevent duplicate labels
df.flags.allows_duplicate_labels = False
pd.DataFrame(data, index=..., columns=...).set_flags(allows_duplicate_labels=False)
```

**Component Breakdown:**

| Method/Property | Description |
|-----------------|-------------|
| `Index.duplicated()` | Returns boolean ndarray indicating duplicate index labels |
| `Index.is_unique` | Returns `True` if all index labels are unique |
| `df.loc[~df.index.duplicated()]` | Boolean filter to drop duplicate index labels |
| `allows_duplicate_labels` | Flag to disallow duplicate labels (pandas 1.2.0+) |

**Syntax Rules:**

- `Index.duplicated()` returns an ndarray, not a Series.
- Boolean filtering with `~` inverts the mask to keep non-duplicate labels.
- `keep` parameter works the same as in `DataFrame.duplicated()`.
- `allows_duplicate_labels=False` raises `DuplicateLabelError` when an operation would introduce duplicates.

**Constraints and Limitations:**

- Duplicate index labels cannot be resolved by `drop_duplicates()`; they require `Index.duplicated()` filtering.
- `reindex()` raises `ValueError` when the index has duplicate labels.
- Setting `allows_duplicate_labels=False` may cause some operations to raise, so it should be set after cleaning.

### Annotated Code Examples

**Example 1: Detecting Duplicate Index Labels**

```python
import pandas as pd

# Create a Series with duplicate index labels
s = pd.Series([0, 1, 2], index=["a", "b", "b"])
print("Original:")
print(s)
# Expected output:
# a    0
# b    1
# b    2
# dtype: int64

# Check uniqueness
print(f"Index is unique: {s.index.is_unique}")
# Expected output: Index is unique: False

# Detect duplicate index labels
print(s.index.duplicated())
# Expected output: array([False, False,  True])
```

**Why this output:** The index labels are `["a", "b", "b"]`. The first `"b"` is `False` (first occurrence), and the second `"b"` is `True` (duplicate).

**Example 2: Filtering Duplicate Index Labels**

```python
# Keep first occurrence of each label
print(s.loc[~s.index.duplicated()])
# Expected output:
# a    0
# b    1
# dtype: int64

# Keep last occurrence
print(s.loc[~s.index.duplicated(keep='last')])
# Expected output:
# a    0
# b    2
# dtype: int64

# Drop all duplicates
print(s.loc[~s.index.duplicated(keep=False)])
# Expected output:
# a    0
# dtype: int64
```

**Why this output:** `~s.index.duplicated()` inverts the boolean mask, keeping non-duplicate labels. `keep='first'` retains the first `"b"` (value 1); `keep='last'` retains the last `"b"` (value 2); `keep=False` removes all `"b"` labels, leaving only `"a"`.

**Example 3: Duplicate Row Values vs. Duplicate Index Labels**

```python
# DataFrame with duplicate row values and duplicate index labels
df = pd.DataFrame({
    'A': [1, 1, 2],
    'B': ['x', 'x', 'y']
}, index=['row1', 'row1', 'row2'])

print(df)
# Expected output:
#       A  B
# row1  1  x
# row1  1  x
# row2  2  y

# Duplicate row values (ignores index)
print(f"Row duplicates: {df.duplicated().tolist()}")
# Expected output: Row duplicates: [False, True, False]

# Duplicate index labels
print(f"Index duplicates: {df.index.duplicated().tolist()}")
# Expected output: Index duplicates: [False, True, False]
```

**Why this output:** Rows 0 and 1 have identical values in columns A and B, so `df.duplicated()` marks row 1 as `True`. The index labels are `['row1', 'row1', 'row2']`, so `df.index.duplicated()` also marks row 1 as `True`. In this case both coincide, but they are independent concepts: you could have duplicate row values with unique index labels, or unique row values with duplicate index labels.

### Real-World Cases

- **Time series data:** Sensor readings with duplicate timestamps as index labels require `Index.duplicated()` filtering before resampling.
- **Merged data:** After merging DataFrames, duplicate index labels can arise; use `allows_duplicate_labels=False` to catch them early.
- **Database exports:** When exporting to SQL, duplicate index labels cause primary key violations; filter them before export.

### References

- Duplicate Labels — https://pandas.pydata.org/docs/dev/user_guide/duplicates.html
- pandas.Index.duplicated — https://pandas.pydata.org/pandas-docs/version/1.1.1/reference/api/pandas.Index.duplicated.html
- pandas.Flags.allows_duplicate_labels — https://pandas.pydata.org/docs/reference/api/pandas.Flags.allows_duplicate_labels.html


## References

- pandas.DataFrame.duplicated — https://pandas.pydata.org/pandas-docs/version/1.5/reference/api/pandas.DataFrame.duplicated.html
- pandas.Series.duplicated — https://pandas.pydata.org/docs/reference/api/pandas.Series.duplicated.html
- pandas.Index.duplicated — https://pandas.pydata.org/pandas-docs/version/1.1.1/reference/api/pandas.Index.duplicated.html
- pandas.DataFrame.drop_duplicates — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.drop_duplicates.html
- pandas.Series.drop_duplicates — https://pandas.pydata.org/docs/reference/api/pandas.Series.drop_duplicates.html
- pandas.Index.drop_duplicates — https://pandas.pydata.org/docs/reference/api/pandas.Index.drop_duplicates.html
- Duplicate Labels — https://pandas.pydata.org/docs/dev/user_guide/duplicates.html
- pandas.Flags.allows_duplicate_labels — https://pandas.pydata.org/docs/reference/api/pandas.Flags.allows_duplicate_labels.html