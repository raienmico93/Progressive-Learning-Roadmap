# Comprehensive Programming Cheat Sheet: Pandas DataFrame Inspection

---

## Topic Overview

### Definitions

**Core Definition:** DataFrame inspection refers to the set of pandas methods and attributes used to examine the structure, contents, statistical properties, schema, and memory footprint of a DataFrame without modifying its data.

**Technical Definition:** Pandas provides a suite of inspection methods operating at different granularities: `.head()` and `.tail()` for row-level scanning; `.shape`, `.columns`, and `.index` for structural metadata; `.dtypes` for schema inspection; `.info()` for comprehensive structural summaries including non-null counts and memory usage; `.describe()` for statistical summaries (dtype-dependent); and `.memory_usage(deep=True)` for accurate byte-level memory profiling.

**Beginner-Friendly Explanation:** When you first load a dataset, you want to understand what it looks like before doing any analysis. Pandas provides tools to peek at the first or last rows, check the dimensions, list column names, see what data types each column holds, get a summary of missing values and memory usage, compute basic statistics, and measure memory consumption per column. These inspection tools help you catch problems early and plan your analysis.

### Key Characteristics

- **Non-destructive:** Inspection methods return new objects or print summaries; they do not modify the original DataFrame.
- **Dtype-aware:** `.describe()` and `.info()` adapt their output based on column dtypes.
- **Missing-value sensitive:** Most methods report or exclude `NaN` according to configurable defaults.
- **Memory-aware:** `.memory_usage(deep=True)` provides accurate byte-level reporting.
- **Version-specific:** pandas 3.0 introduced the `str` dtype, changing dtype inference and inspection behavior.

### Prerequisites

- Basic Python syntax and familiarity with pandas DataFrame construction.
- Understanding of NumPy dtypes and 2-D array memory layout.
- Familiarity with DataFrame indexing (`.loc`, `.iloc`) and basic operations.

### Related Programming Areas

- **Exploratory Data Analysis (EDA):** Initial data understanding and quality assessment.
- **Data Cleaning:** Identifying missing values, type inconsistencies, and misparsed columns.
- **Performance Optimization:** Memory profiling and dtype selection.
- **Data Validation:** Verifying schema and structural expectations.
- **ETL Pipeline Monitoring:** Tracking shape and dtype changes across stages.

### Core Concepts / Features

1. head() & tail()
2. shape, columns, & index
3. dtypes
4. info()
5. describe()
6. memory_usage()

---

## Core Concept 1: head() & tail()

### Definitions

**Core Definition:** `head()` and `tail()` are DataFrame methods that return the first or last `n` rows of a DataFrame, respectively, enabling quick visual inspection of data layout, column alignment, and parsing correctness.

**Technical Definition:** `DataFrame.head(n=5)` returns a new DataFrame containing the first `n` rows, preserving the original index and column labels. `DataFrame.tail(n=5)` returns the last `n` rows. Both methods accept an integer `n` (positive or negative) and return the same dtypes as the original DataFrame. For very large DataFrames, these methods provide a memory-efficient way to preview data without loading the entire object into a display context.

**Beginner-Friendly Explanation:** `head()` shows you the first few rows, and `tail()` shows you the last few rows. It's like flipping to the front or back of a book to see what's there before reading the whole thing. This is one of the first things you do when you load a new dataset.

### Purposes

- To quickly verify the format and structure of a newly created or loaded DataFrame.
- To confirm that column parsing and alignment are correct without displaying thousands of rows.
- To inspect the beginning and end of a time-ordered dataset for anomalies.
- To preview data before applying transformations.
- To check that column headers and index labels are as expected.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.head(n=5)
df.tail(n=5)
```

#### Component Breakdown

| Parameter | Description | Required? | Default |
|-----------|-------------|-----------|---------|
| `n` | Number of rows to return (can be negative) | No | `5` |

#### Syntax Rules

- `n` must be an integer; floats raise `TypeError`.
- Negative `n` returns all rows except the last (or first) `|n|` rows.
- The returned DataFrame retains the original index and column labels.
- Both methods return a new DataFrame; the original is unchanged.

#### Constraints and Limitations

- `head()` and `tail()` do not modify the original DataFrame.
- For a DataFrame with fewer than `n` rows, the entire DataFrame is returned.
- Display truncation in Jupyter may limit visible output regardless of `n`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic head() and tail()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create a DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve', 'Frank'],
    'age': [25, 30, 35, 28, 32, 40],
    'city': ['NYC', 'LA', 'Chicago', 'Boston', 'Seattle', 'Denver']
})

# Step 3: Display first 3 rows
print("Head (3):")
print(df.head(3))

# Step 4: Display last 3 rows
print("\nTail (3):")
print(df.tail(3))
```

**Expected Output:**
```
Head (3):
      name  age     city
0    Alice   25      NYC
1      Bob   30       LA
2  Charlie   35  Chicago

Tail (3):
    name  age     city
3  Diana   28   Boston
4    Eve   32  Seattle
5  Frank   40   Denver
```

**Why this output:** `.head(3)` returned the first three rows (positions 0–2), and `.tail(3)` returned the last three rows (positions 3–5). The index and column labels were preserved.

#### Example 2: Negative n Parameter

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4, 5], 'B': [10, 20, 30, 40, 50]})

# Step 3: head with negative n
print("head(-2):")
print(df.head(-2))

# Step 4: tail with negative n
print("\ntail(-2):")
print(df.tail(-2))
```

**Expected Output:**
```
head(-2):
   A  B
0  1  10
1  2  20
2  3  30

tail(-2):
   A  B
2  3  30
3  4  40
4  5  50
```

**Why this output:** `head(-2)` returned all rows except the last 2; `tail(-2)` returned all rows except the first 2.

#### Example 3: Detecting Misalignment in a CSV-Like DataFrame

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Simulate a misparsed DataFrame (shifted columns)
df = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35]
})

# Step 3: Inspect head and tail to verify alignment
print(df.head(2))
print("\n---")
print(df.tail(2))
```

**Expected Output:**
```
    id   name  age
0   1  Alice   25
1   2    Bob   30

---
    id     name  age
1   2      Bob   30
2   3  Charlie   35
```

**Why this output:** `.head()` and `.tail()` allowed verification that the `id`, `name`, and `age` columns were correctly aligned. If values were shifted, this inspection would reveal it.

### Real-World Cases with Explanation

**Case 1: CSV Ingestion Verification** — A data engineer loads a large CSV and uses `.head()` and `.tail()` to confirm that parsing was correct and that no trailing summary rows were included.

**Case 2: Time Series Boundary Check** — A financial analyst uses `.head()` and `.tail()` on a datetime-indexed stock price DataFrame to verify that the date range covers the expected period.

**Case 3: ETL Pipeline Debugging** — A data pipeline developer inserts `.head()` calls at each transformation stage to verify that data shapes and values remain correct.

### References

- pandas.DataFrame.head — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.head.html
- pandas.DataFrame.tail — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.tail.html
- pandas Cheat Sheet — https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf

---

## Core Concept 2: shape, columns, & index

### Definitions

**Core Definition:** The `.shape`, `.columns`, and `.index` attributes provide programmatic access to a DataFrame's structural metadata, including its row/column counts, column labels, and row labels.

**Technical Definition:** `DataFrame.shape` returns a tuple `(n_rows, n_columns)`. `DataFrame.columns` returns a `pd.Index` object containing the column labels. `DataFrame.index` returns a `pd.Index` object containing the row labels. These attributes are read-only and reflect the current state of the DataFrame.

**Beginner-Friendly Explanation:** These attributes tell you the "shape" of your table: how many rows and columns it has, what the column names are, and what the row labels are. They're essential for understanding the boundaries of your data programmatically.

### Purposes

- To programmatically fetch row and column counts.
- To retrieve column labels for iteration or selection.
- To retrieve row index labels for alignment or filtering.
- To validate structural expectations in automated pipelines.
- To compute dimensions for memory estimation or reshaping.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.shape          # (n_rows, n_columns)
df.columns        # Index of column labels
df.index          # Index of row labels

# Accessing specific components
n_rows, n_cols = df.shape
col_list = df.columns.tolist()
index_list = df.index.tolist()
```

#### Component Breakdown

| Attribute | Description | Return Type |
|-----------|-------------|-------------|
| `.shape` | (rows, columns) | Tuple |
| `.columns` | Column labels | `pd.Index` |
| `.index` | Row labels | `pd.Index` |

#### Syntax Rules

- `.shape` returns a tuple; unpack as `n_rows, n_cols = df.shape`.
- `.columns` and `.index` return `pd.Index` objects, which support `.tolist()`, `.values`, and set operations.
- Column and index labels need not be unique.
- Both `.columns` and `.index` are immutable; use `df.rename()` or `df.set_axis()` to change labels.

#### Constraints and Limitations

- `.shape` does not include the index or column labels in its count.
- An empty DataFrame has shape `(0, 0)` or `(n, 0)`.
- Non-unique column labels can cause ambiguity in selection.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic Shape and Labels

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['NYC', 'LA', 'Chicago']
})

# Step 3: Inspect shape
print(f"Shape: {df.shape}")
print(f"Rows: {df.shape[0]}, Columns: {df.shape[1]}")

# Step 4: Inspect columns and index
print(f"\nColumns: {df.columns.tolist()}")
print(f"Index: {df.index.tolist()}")
```

**Expected Output:**
```
Shape: (3, 3)
Rows: 3, Columns: 3

Columns: ['name', 'age', 'city']
Index: [0, 1, 2]
```

**Why this output:** The DataFrame had 3 rows and 3 columns. The default `RangeIndex` produced row labels `[0, 1, 2]`, and the column labels were the dictionary keys.

#### Example 2: Custom Index and Column Metadata

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with custom index
df = pd.DataFrame(
    {'A': [1, 2], 'B': [3, 4]},
    index=['row1', 'row2']
)

# Step 3: Inspect metadata
print(f"Shape: {df.shape}")
print(f"Columns: {df.columns.tolist()}")
print(f"Index: {df.index.tolist()}")
print(f"Index name: {df.index.name}")
print(f"Columns name: {df.columns.name}")
```

**Expected Output:**
```
Shape: (2, 2)
Columns: ['A', 'B']
Index: ['row1', 'row2']
Index name: None
Columns name: None
```

**Why this output:** The custom index replaced the default integer labels. Both `index.name` and `columns.name` default to `None` unless explicitly set.

#### Example 3: Programmatic Validation in a Pipeline

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'id': [1, 2, 3], 'value': [10, 20, 30]})

# Step 3: Validate expected columns
expected_cols = ['id', 'value']
actual_cols = df.columns.tolist()

if actual_cols == expected_cols:
    print("Column validation passed")
else:
    print(f"Column mismatch: expected {expected_cols}, got {actual_cols}")

# Step 4: Validate row count
if df.shape[0] > 0:
    print(f"Row count OK: {df.shape[0]} rows")
```

**Expected Output:**
```
Column validation passed
Row count OK: 3 rows
```

**Why this output:** The column list matched expectations, and the row count was greater than 0, so both validations passed.

### Real-World Cases with Explanation

**Case 1: Pipeline Validation** — A data engineer checks `df.shape` and `df.columns` after each ETL step to ensure row/column counts and schema match expectations.

**Case 2: Dynamic Column Selection** — A data scientist uses `df.columns` to programmatically select columns matching a pattern (e.g., `[c for c in df.columns if c.startswith('feature_')]`).

**Case 3: Index Alignment** — An analyst inspects `df.index` before a join to verify that both DataFrames share compatible index labels.

### References

- pandas.DataFrame.shape — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.shape.html
- pandas.DataFrame.columns — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.columns.html
- pandas.DataFrame.index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.index.html

---

## Core Concept 3: dtypes

### Definitions

**Core Definition:** The `.dtypes` attribute returns a Series mapping each column name to its data type, enabling inspection of the DataFrame's schema and detection of misparsed columns.

**Technical Definition:** `DataFrame.dtypes` returns a `pd.Series` indexed by column labels, with values being dtype objects (NumPy dtypes, ExtensionDtypes, or ArrowDtypes). Common dtypes include `int64`, `float64`, `bool`, `datetime64[ns]`, `object`, `category`, `Int64` (nullable integer), `string`, and PyArrow-backed variants like `int64[pyarrow]`. Columns with mixed types are stored as `object`. Under pandas 3.0, string data is inferred as the dedicated `str` dtype rather than `object`.

**Beginner-Friendly Explanation:** Every column has a data type. `.dtypes` tells you what type each column is. This is crucial because a column of numbers accidentally parsed as text (`object`) won't support numerical operations. Checking `.dtypes` helps you catch such issues early.

### Purposes

- To inspect the schema of a DataFrame at a glance.
- To detect columns misparsed as `object` when they should be numeric or datetime.
- To identify columns that need type conversion.
- To verify dtype consistency after data loading or transformation.
- To plan memory optimization by choosing efficient dtypes.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.dtypes

# Count dtype occurrences
df.dtypes.value_counts()

# Select columns by dtype
df.select_dtypes(include=['int64', 'float64'])
df.select_dtypes(exclude=['object'])
```

#### Component Breakdown

| Attribute/Method | Description |
|------------------|-------------|
| `df.dtypes` | Series of column dtypes |
| `df.dtypes.value_counts()` | Count of each dtype |
| `df.select_dtypes()` | Filter columns by dtype |

#### Syntax Rules

- `df.dtypes` returns a Series indexed by column names.
- Mixed types in a column → `object` dtype.
- Nullable integer dtypes use capital `I` (e.g., `Int64` vs NumPy's `int64`).
- `select_dtypes()` accepts `include` and `exclude` lists of dtype strings.

#### Constraints and Limitations

- **Version-specific:** pandas 3.0 introduces a default `str` dtype for strings, replacing `object`.
- **Performance:** `object` dtype is slower and uses more memory than specialized dtypes.
- **Conversion cost:** `astype()` creates a copy; `convert_dtypes()` may be expensive for large DataFrames.
- **Nullable types:** `Int64` uses `pd.NA`; mixing with NumPy `NaN` requires care.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Inspecting dtypes

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with mixed types
df = pd.DataFrame({
    'int_col': [1, 2, 3],
    'float_col': [1.1, 2.2, 3.3],
    'str_col': ['a', 'b', 'c'],
    'bool_col': [True, False, True]
})

# Step 3: Inspect dtypes
print(df.dtypes)
```

**Expected Output:**
```
int_col        int64
float_col    float64
str_col       object
bool_col        bool
dtype: object
```

**Why this output:** `.dtypes` returned a Series mapping each column to its inferred dtype. The string column was `object` (pre-pandas 3.0 behavior).

#### Example 2: Detecting Misparsed Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Simulate a DataFrame where a numeric column was parsed as string
df = pd.DataFrame({
    'id': ['1', '2', '3'],        # Accidentally string
    'value': [10, 20, 30]          # Correctly numeric
})

# Step 3: Inspect dtypes
print(df.dtypes)

# Step 4: Detect object columns
object_cols = df.dtypes[df.dtypes == 'object'].index.tolist()
print(f"\nColumns with object dtype: {object_cols}")

# Step 5: Fix by converting
df['id'] = df['id'].astype(int)
print(f"\nAfter conversion:\n{df.dtypes}")
```

**Expected Output:**
```
id       object
value     int64
dtype: object

Columns with object dtype: ['id']

After conversion:
id       int64
value    int64
dtype: object
```

**Why this output:** The `id` column was parsed as `object` (string) instead of `int64`. The detection logic identified it, and `.astype(int)` corrected it.

#### Example 3: Counting and Selecting by Dtype

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'A': [1, 2],
    'B': [1.1, 2.2],
    'C': ['x', 'y'],
    'D': [True, False]
})

# Step 3: Count dtypes
print("Dtype counts:")
print(df.dtypes.value_counts())

# Step 4: Select numeric columns only
numeric_df = df.select_dtypes(include=['int64', 'float64'])
print(f"\nNumeric columns:\n{numeric_df}")
```

**Expected Output:**
```
Dtype counts:
object    1
int64     1
float64   1
bool      1
dtype: int64

Numeric columns:
   A    B
0  1  1.1
1  2  2.2
```

**Why this output:** `value_counts()` counted one column per dtype. `select_dtypes(include=['int64', 'float64'])` returned only the numeric columns `A` and `B`.

### Real-World Cases with Explanation

**Case 1: Data Quality Assessment** — A data scientist checks `.dtypes` after loading a CSV to detect columns that should be numeric but were parsed as `object` (e.g., due to currency symbols or commas).

**Case 2: Memory Optimization** — A data engineer uses `.dtypes` to identify `int64` columns that can be downcast to `int8` or `int16`, reducing memory usage.

**Case 3: Feature Selection** — A machine learning engineer uses `select_dtypes(include=['number'])` to isolate numeric features for model training.

### References

- pandas.DataFrame.dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.dtypes.html
- pandas.DataFrame.select_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html
- Nullable Integer Data Type — https://pandas.pydata.org/docs/user_guide/integer_na.html
- PyArrow Backend — https://pandas.pydata.org/docs/user_guide/pyarrow.html

---

## Core Concept 4: info()

### Definitions

**Core Definition:** `info()` prints a comprehensive summary of a DataFrame, including the class type, index range, column names, non-null counts, dtypes, and memory usage.

**Technical Definition:** `DataFrame.info(verbose=None, buf=None, max_cols=None, memory_usage=None, show_counts=True)` outputs a summary to `sys.stdout` (or a writable buffer). The summary includes: `<class 'pandas.core.frame.DataFrame'>`, the index range and number of entries, the total number of columns, a per-column listing of non-null counts and dtypes, and the total memory usage. The `memory_usage` parameter accepts `bool`, `str` (e.g., `'deep'`), or `None` to control whether and how memory usage is displayed.

**Beginner-Friendly Explanation:** `info()` is like a "report card" for your DataFrame. It tells you how many rows and columns you have, what data type each column is, how many non-null values each column has, and how much memory the whole thing uses. This is one of the first commands you run when you encounter a new dataset.

### Purposes

- To obtain a macro-level structural summary of a DataFrame in a single call.
- To verify dtypes and non-null counts across all columns.
- To inspect memory usage without computing it separately.
- To identify columns with missing values quickly.
- To document the structural properties of a DataFrame for reproducibility.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.info(verbose=None, buf=None, max_cols=None, memory_usage=None, show_counts=True)
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `verbose` | Whether to print the full summary | Follows `pandas.options.display.max_info_columns` |
| `buf` | Writable buffer for output | `sys.stdout` |
| `max_cols` | Maximum number of columns to display | Follows display options |
| `memory_usage` | Whether to show memory usage; `'deep'` for deep introspection | Follows `pandas.options.display.memory_usage` |
| `show_counts` | Whether to show non-null counts | `True` |

#### Syntax Rules

- `info()` prints to `sys.stdout` by default; returns `None`.
- `memory_usage='deep'` performs actual memory introspection, which is computationally expensive.
- Without deep introspection, memory is estimated based on dtype and row count.
- For DataFrames with many columns, `verbose=True` forces the full column listing.

#### Constraints and Limitations

- **Performance:** Deep memory introspection can be slow for large DataFrames.
- **Display truncation:** Very wide DataFrames may have column output truncated based on display options.
- **Non-null counts:** Columns with all-`NaN` values show `0 non-null`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic info() Output

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['NYC', 'LA', 'Chicago']
})

# Step 3: Call info()
df.info()
```

**Expected Output:**
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 3 entries, 0 to 2
Data columns (total 3 columns):
 #   Column  Non-Null Count  Dtype 
---  ------  --------------  ----- 
 0   name    3 non-null      object
 1   age     3 non-null      int64 
 2   city    3 non-null      object
dtypes: int64(1), object(2)
memory usage: 200.0+ bytes
```

**Why this output:** `info()` reported the class type, index range (3 entries, 0 to 2), per-column non-null counts and dtypes, a dtype summary (`int64(1), object(2)`), and estimated memory usage.

#### Example 2: info() with Missing Values

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create DataFrame with NaN
df = pd.DataFrame({
    'A': [1, 2, np.nan],
    'B': ['x', None, 'z'],
    'C': [1.1, 2.2, 3.3]
})

# Step 3: Call info()
df.info()
```

**Expected Output:**
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 3 entries, 0 to 2
Data columns (total 3 columns):
 #   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   A       2 non-null      float64
 1   B       2 non-null      object 
 2   C       3 non-null      float64
dtypes: float64(2), object(1)
memory usage: 200.0+ bytes
```

**Why this output:** The non-null counts revealed that columns `A` and `B` each had 2 non-null values (one `NaN` each), while column `C` had 3. Column `A` was `float64` because `NaN` forces floating-point representation.

#### Example 3: info() with Deep Memory Introspection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with strings
df = pd.DataFrame({'text': ['hello', 'world', 'pandas']})

# Step 3: Call info() with deep memory
df.info(memory_usage='deep')
```

**Expected Output:**
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 3 entries, 0 to 2
Data columns (total 1 columns):
 #   Column  Non-Null Count  Dtype 
---  ------  --------------  ----- 
 0   text    3 non-null      object
dtypes: object(1)
memory usage: 195.0 bytes
```

**Why this output:** With `memory_usage='deep'`, pandas performed actual memory introspection, accounting for the full memory consumed by the string objects in the `text` column.

### Real-World Cases with Explanation

**Case 1: Dataset Onboarding** — A data scientist loads a new dataset and calls `.info()` first to understand the structure, dtypes, and missing-value situation before any analysis.

**Case 2: Memory Audit** — A data engineer calls `.info(memory_usage='deep')` on a large DataFrame to identify memory bottlenecks and decide whether to convert to more efficient dtypes.

**Case 3: Pipeline Logging** — A machine learning engineer logs `.info()` output at each stage of a preprocessing pipeline to track dtype changes and missing-value counts.

### References

- pandas.DataFrame.info — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.info.html
- Frequently Asked Questions: Memory Usage — https://pandas.pydata.org/docs/user_guide/gotchas.html

---

## Core Concept 5: describe()

### Definitions

**Core Definition:** `describe()` generates rapid summary statistics for a DataFrame, producing central tendency and dispersion measures for numeric columns, and frequency summaries for object, string, or categorical columns.

**Technical Definition:** `DataFrame.describe(percentiles=None, include=None, exclude=None)` returns a DataFrame of summary statistics. For numeric columns, the output includes `count`, `mean`, `std`, `min`, the 25th, 50th (median), and 75th percentiles, and `max`. For object columns, the output includes `count`, `unique`, `top` (most common value), and `freq` (frequency of the most common value). The `include` and `exclude` parameters control which columns are summarized; `include='all'` includes all columns.

**Beginner-Friendly Explanation:** `describe()` gives you a quick statistical snapshot of your data. For numeric columns, it tells you the average, how spread out the data is, and the range. For text or categorical columns, it tells you how many unique values there are and which one appears most often.

### Purposes

- To generate summary statistics for all or selected columns in a single call.
- To compare the distributional properties of different columns.
- To identify outliers via min/max and percentiles.
- To verify data quality (e.g., unexpected counts or frequencies).
- To document the statistical properties of a dataset for reports.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.describe(percentiles=None, include=None, exclude=None)

# Include all columns
df.describe(include='all')

# Include only numeric
df.describe(include='number')

# Include only object
df.describe(include='object')
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `percentiles` | List of percentiles to include (in [0, 1]) | `[.25, .5, .75]` |
| `include` | Dtypes to include | `None` (numeric only) |
| `exclude` | Dtypes to exclude | `None` |

#### Syntax Rules

- For numeric columns, output includes `count`, `mean`, `std`, `min`, percentiles, and `max`.
- For object/string columns, output includes `count`, `unique`, `top`, and `freq`.
- `include='all'` includes all columns, mixing numeric and categorical output.
- Custom percentiles can be specified as a list in the `[0, 1]` interval.

#### Constraints and Limitations

- `NaN` values are excluded from all computations.
- For mixed-dtype DataFrames, output rows differ between numeric and object columns.
- If multiple values have the highest count, the `top` value is arbitrarily chosen.
- Datetime columns produce first/last items in the output.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic describe() on Numeric DataFrame

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create numeric DataFrame
df = pd.DataFrame({
    'age': [25, 30, 35, 40, 45],
    'salary': [50000, 60000, 70000, 80000, 90000]
})

# Step 3: Call describe()
print(df.describe())
```

**Expected Output:**
```
             age       salary
count    5.00000      5.00000
mean    35.00000  70000.00000
std     15.81139  15811.38830
min     25.00000  50000.00000
25%     30.00000  60000.00000
50%     35.00000  70000.00000
75%     40.00000  80000.00000
max     45.00000  90000.00000
```

**Why this output:** The numeric DataFrame produced the standard seven-statistic summary for each column: count, mean, standard deviation, minimum, 25th percentile, median, 75th percentile, and maximum.

#### Example 2: describe() with Mixed Dtypes

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create mixed DataFrame
df = pd.DataFrame({
    'age': [25, 30, 35],
    'name': ['Alice', 'Bob', 'Alice'],
    'city': ['NYC', 'LA', 'NYC']
})

# Step 3: Describe with include='all'
print(df.describe(include='all'))
```

**Expected Output:**
```
        age   name  city
count   3.0      3     3
unique  NaN      2     2
top     NaN  Alice   NYC
freq    NaN      2     2
mean   30.0    NaN   NaN
std     5.0    NaN   NaN
min    25.0    NaN   NaN
25%    27.5    NaN   NaN
50%    30.0    NaN   NaN
75%    32.5    NaN   NaN
max    35.0    NaN   NaN
```

**Why this output:** With `include='all'`, the output combined numeric statistics (for `age`) and categorical statistics (for `name` and `city`). Non-applicable cells are `NaN`.

#### Example 3: describe() on Object Columns Only

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with categorical data
df = pd.DataFrame({
    'product': ['A', 'B', 'A', 'C', 'A', 'B'],
    'region': ['North', 'South', 'North', 'East', 'North', 'South']
})

# Step 3: Describe object columns
print(df.describe(include='object'))
```

**Expected Output:**
```
        product region
count         6      6
unique        3      3
top           A  North
freq          3      3
```

**Why this output:** For object columns, `describe()` returned count (6), unique values (3 each), the most common value (`A` for product, `North` for region), and its frequency (3 each).

### Real-World Cases with Explanation

**Case 1: Salary Analysis** — An HR analyst uses `.describe()` on a salary DataFrame to understand the distribution, identify the median, and detect potential outliers via min/max.

**Case 2: Survey Response Analysis** — A researcher uses `.describe(include='object')` on categorical survey columns to identify the most common responses and the number of unique responses.

**Case 3: Sensor Calibration** — An engineer uses `.describe()` on sensor readings to verify that the measurement range is within expected bounds.

### References

- pandas.DataFrame.describe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html
- Descriptive Statistics — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics

---

## Core Concept 6: memory_usage()

### Definitions

**Core Definition:** `memory_usage()` returns the memory usage of each column in a DataFrame in bytes, with the `deep=True` option performing accurate byte-level introspection of object-typed data (e.g., strings).

**Technical Definition:** `DataFrame.memory_usage(index=True, deep=False)` returns a `pd.Series` indexed by column names, with values being the memory usage of each column in bytes. When `deep=True`, pandas performs actual memory introspection for object dtypes, accounting for the full memory consumed by the objects (e.g., Python string objects) rather than just the pointer array. When `deep=False` (default), memory is estimated based on dtype and row count.

**Beginner-Friendly Explanation:** `memory_usage()` tells you how much RAM each column uses. For columns containing text or mixed data (object dtype), the default estimate can be very wrong—it only counts the pointers, not the actual text. Setting `deep=True` gives you the true memory footprint.

### Purposes

- To measure the memory footprint of each column in a DataFrame.
- To identify memory bottlenecks and optimize dtype usage.
- To accurately measure the cost of object-typed columns via `deep=True`.
- To compare memory usage before and after dtype conversions.
- To plan memory-constrained data processing.

### Syntax Rules and Structure

#### Complete General Syntax

```python
df.memory_usage(index=True, deep=False)

# Total memory usage
df.memory_usage(deep=True).sum()

# Per-column memory
df.memory_usage(deep=True)
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `index` | Include index memory in the total | `True` |
| `deep` | Perform deep introspection for object dtypes | `False` |

#### Syntax Rules

- Returns a Series indexed by column names (plus an `Index` entry if `index=True`).
- `deep=True` is computationally expensive for large DataFrames with many object columns.
- Without deep introspection, object columns are estimated as `n_rows * 8` bytes (pointer size).
- `.sum()` gives the total memory usage in bytes.

#### Constraints and Limitations

- **Performance:** `deep=True` can be slow for large DataFrames with many strings.
- **Accuracy:** Without `deep=True`, object column memory is significantly underestimated.
- **Version-specific:** Under pandas 3.0, string columns use the `str` dtype, and `.array` returns a `StringArray`, affecting memory reporting.
- **Index overhead:** The index itself consumes memory; set `index=False` to exclude it.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Shallow vs. Deep Memory Usage

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with strings
df = pd.DataFrame({'text': ['hello', 'world', 'pandas']})

# Step 3: Shallow memory usage
print(f"Shallow: {df.memory_usage(deep=False).sum()} bytes")

# Step 4: Deep memory usage
print(f"Deep: {df.memory_usage(deep=True).sum()} bytes")
```

**Expected Output:**
```
Shallow: 148 bytes
Deep: 195 bytes
```

**Why this output:** The shallow estimate only counted the pointer array (8 bytes × 3 rows = 24 bytes for the data, plus index overhead). The deep introspection accounted for the actual string objects, revealing a higher true memory footprint.

#### Example 2: Per-Column Memory Breakdown

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create mixed DataFrame
df = pd.DataFrame({
    'id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'score': [85.5, 92.3, 78.9]
})

# Step 3: Per-column memory usage
print(df.memory_usage(deep=True))
```

**Expected Output:**
```
Index     132
id         24
name      180
score      24
dtype: int64
```

**Why this output:** The `name` column (object dtype) consumed the most memory (180 bytes) due to the string objects. The `id` and `score` columns used 24 bytes each (8 bytes × 3 rows). The index consumed 132 bytes.

#### Example 3: Memory Optimization Comparison

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with object dtype
df = pd.DataFrame({'category': ['A', 'B', 'A', 'C', 'B'] * 1000})

# Step 3: Memory before conversion
before = df.memory_usage(deep=True).sum()
print(f"Before (object): {before} bytes")

# Step 4: Convert to category dtype
df['category'] = df['category'].astype('category')
after = df.memory_usage(deep=True).sum()
print(f"After (category): {after} bytes")
print(f"Savings: {(before - after) / before * 100:.1f}%")
```

**Expected Output:**
```
Before (object): 80080 bytes
After (category): 10080 bytes
Savings: 87.4%
```

**Why this output:** Converting a low-cardinality string column to `category` dtype dramatically reduced memory usage by storing integer codes and a lookup table instead of full strings.

### Real-World Cases with Explanation

**Case 1: Memory Optimization** — A data engineer working with a large DataFrame uses `.memory_usage(deep=True)` to identify columns consuming excessive memory and converts low-cardinality strings to `category` dtype.

**Case 2: Dataset Sizing** — A data scientist estimates whether a dataset will fit in available RAM by computing `.memory_usage(deep=True).sum()` before loading it fully.

**Case 3: Before/After Profiling** — A machine learning engineer measures memory usage before and after dtype optimization to quantify the impact of their changes.

### References

- pandas.DataFrame.memory_usage — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.memory_usage.html
- Categorical Data — https://pandas.pydata.org/docs/user_guide/categorical.html
- Frequently Asked Questions: Memory Usage — https://pandas.pydata.org/docs/user_guide/gotchas.html

---

## Summary Table: DataFrame Inspection Methods

| Method/Attribute | Returns | Dtype-Dependent | Missing-Value Default | Memory-Related |
|------------------|---------|-----------------|----------------------|----------------|
| `.head(n)` / `.tail(n)` | DataFrame | No | Includes NaN | No |
| `.shape` | Tuple (rows, cols) | No | — | No |
| `.columns` | `pd.Index` | No | — | No |
| `.index` | `pd.Index` | No | — | No |
| `.dtypes` | Series | Yes | — | No |
| `.info()` | None (prints) | Yes | Shows non-null count | Yes |
| `.describe()` | DataFrame | Yes | Excludes NaN | No |
| `.memory_usage()` | Series (bytes) | Yes | — | Yes |

---

## References

- pandas.DataFrame.head — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.head.html
- pandas.DataFrame.tail — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.tail.html
- pandas.DataFrame.shape — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.shape.html
- pandas.DataFrame.columns — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.columns.html
- pandas.DataFrame.index — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.index.html
- pandas.DataFrame.dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.dtypes.html
- pandas.DataFrame.select_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html
- pandas.DataFrame.info — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.info.html
- pandas.DataFrame.describe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html
- pandas.DataFrame.memory_usage — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.memory_usage.html
- Descriptive Statistics — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics
- Categorical Data — https://pandas.pydata.org/docs/user_guide/categorical.html
- Frequently Asked Questions: Memory Usage — https://pandas.pydata.org/docs/user_guide/gotchas.html
- pandas Cheat Sheet — https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf
- What's New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html