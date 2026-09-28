# Comprehensive Programming Cheat Sheet: Pandas `.iloc` (Position-Based Selection)

---

## Topic Overview

### Definitions

**Core Definition:** `.iloc` is a pandas indexer that performs strictly integer-position-based selection on a DataFrame or Series, allowing access to rows and columns by their zero-based physical offsets, with slicing that excludes the stop boundary in accordance with native Python list slicing semantics.

**Technical Definition:** `DataFrame.iloc[row_indexer, column_indexer]` is a property that returns an `_iLocIndexer` object, which accepts integers, lists/arrays of integers, integer slices, boolean arrays, callables, and tuples of row-and-column indexers. It raises `IndexError` for out-of-bounds scalar or list indexers, but allows out-of-bounds slicing (conforming with Python/NumPy slice semantics). `.iloc` never falls back to label-based access, even when the index contains integers. In pandas 3.0, callables that return a tuple are deprecated as input. 

**Beginner-Friendly Explanation:** `.iloc` lets you pick rows and columns by their position number, starting from 0. The first row is position 0, the second is 1, and so on—just like indexing a Python list. When you slice with `.iloc`, pandas stops one item before the number you give, just like Python lists. Negative numbers count from the end. This works regardless of what labels are in the index.

### Key Characteristics

- **Strictly position-based:** Uses zero-based integer offsets; never label-based.
- **Exclusive slicing:** `df.iloc[0:3]` returns positions 0, 1, and 2—not 3.
- **Dual-axis selection:** Accepts both row and column indexers: `df.iloc[rows, cols]`.
- **Rich indexer types:** Supports integers, lists, slices, boolean arrays, and callables.
- **Negative indexing:** Counts from the end (`-1` is last, `-2` is second-to-last).
- **Out-of-bounds behavior:** Scalar/list indexers raise `IndexError`; slices allow out-of-bounds.
- **Version-specific:** Callables returning tuples are deprecated in pandas 3.0.

### Prerequisites

- Basic Python syntax (lists, slicing, negative indices).
- Pandas Series and DataFrame construction.
- Understanding of DataFrame indexing basics.
- Familiarity with boolean logic for boolean array masking.
- Awareness of the distinction between `.loc` (label-based) and `.iloc` (position-based).

### Related Programming Areas

- **Data Analysis:** Positional subsetting for sampling and splitting.
- **Machine Learning:** Train/test splitting by position.
- **Data Cleaning:** Position-based row removal and inspection.
- **ETL Pipelines:** Deterministic extraction of records by order.
- **Exploratory Data Analysis:** Quick inspection of first/last records.

### Core Concepts / Features

1. Position-Based Selection
2. Row Ranges
3. Column Ranges
4. Scalar Access
5. Negative Position Handling

---

## Core Concept 1: Position-Based Selection

### Definitions

**Core Definition:** Position-based selection uses `.iloc` to access DataFrame or Series elements by their zero-based integer offsets, where slicing excludes the stop boundary, matching native Python list slice semantics.

**Technical Definition:** `DataFrame.iloc` is a property that returns an indexer object. When a scalar integer is passed, it returns a Series (if only a row indexer is passed) or a scalar (if both row and column indexers are scalars). When a list of integers is passed, it returns a DataFrame. When a slice is passed, it returns a DataFrame with the start included and the stop excluded. `.iloc` raises `IndexError` for out-of-bounds scalar or list indexers, but allows out-of-bounds slicing. 

**Beginner-Friendly Explanation:** `.iloc` is all about position numbers. You pass the position of the row you want (0 for first, 1 for second), and pandas gives you that row. When you pass a range, pandas includes the start but stops before the end—exactly like Python list slicing.

### Purposes

- To select data by physical order when labels are unknown or irrelevant.
- To avoid ambiguity when the index contains integer labels that could be confused with positions.
- To enable deterministic, order-based extraction in pipelines.
- To perform positional slicing and fancy indexing.
- To support negative indexing for accessing trailing records.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Scalar row position
df.iloc[position]

# List of row positions
df.iloc[[pos1, pos2, pos3]]

# Integer slice (exclusive stop)
df.iloc[start:stop]

# Row and column positions
df.iloc[row_pos, col_pos]

# Slice both axes
df.iloc[row_start:row_stop, col_start:col_stop]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `position` | Zero-based integer offset | Yes |
| `[pos1, pos2]` | List of integer offsets | Yes (for multiple) |
| `start:stop` | Integer slice (exclusive stop) | Yes |
| `col_pos` | Column position | No |

#### Syntax Rules

- Positions must be integers; floats raise `TypeError`.
- Slices follow Python's exclusive-stop convention.
- Negative positions count from the end (`-1` is last).
- `.iloc` ignores index labels entirely.
- Boolean arrays must match the length of the axis.
- Callables receiving the DataFrame and returning valid indexers are accepted.

#### Constraints and Limitations

- Cannot perform label-based lookups.
- Out-of-bounds scalar or list positions raise `IndexError`.
- Slices allow out-of-bounds indexing (conforming with Python/NumPy semantics).
- Non-contiguous arrays may require copies.
- Callables returning tuples are deprecated as of pandas 3.0.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Scalar Position Selection

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

# Step 4: Select third row by position
print("\nRow at position 2:")
print(df.iloc[2])
```

**Expected Output:**
```
Row at position 0:
name    Alice
score      85
Name: a, dtype: object

Row at position 2:
name    Charlie
score        78
Name: c, dtype: object
```

**Why this output:** `.iloc[0]` selected the first row (position 0), regardless of its label `'a'`. `.iloc[2]` selected the third row (position 2), ignoring its label `'c'`. 

#### Example 2: List of Positions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [1, 2, 3, 4], 'B': [10, 20, 30, 40]})

# Step 3: Select positions 0 and 2
subset = df.iloc[[0, 2]]
print(subset)
print(f"\nType: {type(subset).__name__}")
```

**Expected Output:**
```
   A   B
0  1  10
2  3  30

Type: DataFrame
```

**Why this output:** Passing a list `[0, 2]` selected rows at positions 0 and 2, returning a DataFrame. The original index labels (0 and 2) were preserved. 

#### Example 3: Integer Labels vs. Positions

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame with integer index
df = pd.DataFrame({'value': [10, 20, 30]}, index=[100, 200, 300])

# Step 3: .iloc uses positions
print(f"df.iloc[0] = {df.iloc[0].values[0]}")

# Step 4: .loc uses labels
print(f"df.loc[100] = {df.loc[100].values[0]}")
```

**Expected Output:**
```
df.iloc[0] = 10
df.loc[100] = 10
```

**Why this output:** `.iloc[0]` retrieved the value at position 0 (10). `.loc[100]` retrieved the value at label 100 (also 10). They happen to return the same value here, but for different reasons—one by position, one by label.

### Real-World Cases with Explanation

**Case 1: First/Last Records** — A data engineer uses `.iloc[0]` and `.iloc[-1]` to inspect the first and last rows of a DataFrame.

**Case 2: Train/Test Split** — A machine learning engineer uses positional slicing (`.iloc[:800]`, `.iloc[800:]`) to split data into training and testing sets.

**Case 3: Sampling** — A statistician uses `.iloc` with a random list of positions to draw a random sample from a DataFrame.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- pandas.Series.iloc — https://pandas.pydata.org/docs/reference/api/pandas.Series.iloc.html

---

## Core Concept 2: Row Ranges

### Definitions

**Core Definition:** Row ranges with `.iloc` slice specific vertical index bands using integer intervals (`df.iloc[0:10]`), where the start is included and the stop is excluded.

**Technical Definition:** `df.iloc[start:stop]` returns a DataFrame containing rows from position `start` up to (but not including) `stop`. The `start` defaults to 0 if omitted, and `stop` defaults to the length of the DataFrame if omitted. A step can be included: `df.iloc[start:stop:step]`. This matches Python's list slicing semantics exactly. 

**Beginner-Friendly Explanation:** You can grab a range of rows by their position numbers. `df.iloc[0:5]` gives you the first five rows (positions 0, 1, 2, 3, 4)—it stops before position 5. This is exactly how Python list slicing works.

### Purposes

- To extract contiguous subsets of rows by position.
- To partition data into segments (e.g., first half, second half).
- To process DataFrames in chunks.
- To select every Nth row using a step.
- To sample contiguous blocks of records.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Row range (exclusive stop)
df.iloc[start:stop]

# Row range with step
df.iloc[start:stop:step]

# From start to end
df.iloc[start:]

# From beginning to stop
df.iloc[:stop]

# Every other row
df.iloc[::2]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `start` | Starting position (inclusive) | No (default 0) |
| `stop` | Ending position (exclusive) | No (default length) |
| `step` | Step size | No (default 1) |

#### Syntax Rules

- The start bound is inclusive; the stop bound is exclusive.
- Slices allow out-of-bounds indexing (no `IndexError`).
- Negative step reverses direction.
- Omitting `start` defaults to 0; omitting `stop` defaults to the length.

#### Constraints and Limitations

- Slices always return a DataFrame, even for a single row.
- Non-integer slices raise `TypeError`.
- Out-of-bounds slices are silently clipped (no error).

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic Row Range

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eva'],
    'Age': [25, 30, 35, 40, 45]
})

# Step 3: Select first two rows
print("First two rows:")
print(df.iloc[0:2])

# Step 4: Select rows 1 through 3
print("\nRows 1 through 3:")
print(df.iloc[1:4])
```

**Expected Output:**
```
First two rows:
    Name  Age
0  Alice   25
1    Bob   30

Rows 1 through 3:
      Name  Age
1      Bob   30
2  Charlie   35
3    David   40
```

**Why this output:** `.iloc[0:2]` returned positions 0 and 1 (stop 2 excluded). `.iloc[1:4]` returned positions 1, 2, and 3 (stop 4 excluded). 

#### Example 2: Row Range with Step

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

**Why this output:** The step of 2 selected positions 1, 3, 5, and 7, skipping every other row within the range. 

#### Example 3: Open-Ended Row Ranges

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]})

# Step 3: From position 2 to end
print("From position 2:")
print(df.iloc[2:])

# Step 4: From beginning to position 3
print("\nFirst 3 rows:")
print(df.iloc[:3])

# Step 5: Reversed
print("\nReversed:")
print(df.iloc[::-1])
```

**Expected Output:**
```
From position 2:
   value
2     30
3     40
4     50

First 3 rows:
   value
0     10
1     20
2     30

Reversed:
   value
4     50
3     40
2     30
1     20
0     10
```

**Why this output:** `.iloc[2:]` selected from position 2 to the end. `.iloc[:3]` selected the first three rows. `.iloc[::-1]` reversed the DataFrame.

### Real-World Cases with Explanation

**Case 1: Batch Processing** — A data engineer processes a large DataFrame in chunks using positional slices (`df.iloc[i:i+1000]`).

**Case 2: Head and Tail** — A data scientist uses `df.iloc[:5]` and `df.iloc[-5:]` to inspect the first and last five records.

**Case 3: Sampling Every Nth Record** — A scientist extracts every 10th measurement using `df.iloc[::10]`.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- Pandas Slicing Guide — https://github.com/Compile-N-Run/Compile-N-Run/blob/5e277f39f3e5b8b740aba0789e01e388911932ce/docs/library/pandas/5-pandas-data-selection/7-pandas-slicing.mdx

---

## Core Concept 3: Column Ranges

### Definitions

**Core Definition:** Column ranges with `.iloc` isolate horizontal column blocks using precise slice coordinates or index lists in the second axis slot (`df.iloc[:, 0:3]`), where the start is included and the stop is excluded.

**Technical Definition:** `df.iloc[row_indexer, col_indexer]` accepts a column indexer that can be a scalar integer, a list of integers, or an integer slice. The `:` symbol selects all rows or columns. Column slices follow the same exclusive-stop semantics as row slices. To select specific columns by position, pass a list in the second slot: `df.iloc[:, [0, 2]]`. 

**Beginner-Friendly Explanation:** You can pick columns by their position too. `df.iloc[:, 0:3]` gives you the first three columns for all rows. The `:` before the comma means "all rows," and `0:3` means "columns at positions 0, 1, and 2."

### Purposes

- To select specific columns by position when column labels are unknown.
- To isolate feature subsets for analysis.
- To reorder columns by position.
- To apply column-wise operations on a subset.
- To extract a single column as a Series or DataFrame.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Single column (Series)
df.iloc[:, col_pos]

# Multiple columns (DataFrame)
df.iloc[:, [col1, col2]]

# Column slice (exclusive)
df.iloc[:, col_start:col_stop]

# Rows and columns
df.iloc[row_slice, col_slice]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `:` | All rows | Yes (for column-only selection) |
| `col_pos` | Integer column position | Yes |
| `[col1, col2]` | List of column positions | Yes (for multiple) |
| `col_start:col_stop` | Integer column slice (exclusive) | Yes |

#### Syntax Rules

- Use `:` to select all rows.
- Column positions are zero-based integers.
- Column slices follow the same exclusive-stop semantics.
- Lists of column positions preserve the order specified.

#### Constraints and Limitations

- Column positions must be integers.
- Out-of-bounds column positions raise `IndexError`.
- Column slices allow out-of-bounds indexing.
- The result of a single-column selection with a scalar is a Series.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Single Column by Position

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'Salary': [50000, 60000, 70000],
    'Department': ['HR', 'IT', 'Finance']
})

# Step 3: Select column at position 1 (Age)
print(df.iloc[:, 1])

# Step 4: Select column at position 0 (Name) as DataFrame
print("\nColumn at position 0:")
print(df.iloc[:, [0]])
```

**Expected Output:**
```
0    25
1    30
2    35
Name: Age, dtype: int64

Column at position 0:
      Name
0    Alice
1      Bob
2  Charlie
```

**Why this output:** `.iloc[:, 1]` selected the column at position 1 (`Age`) as a Series. `.iloc[:, [0]]` selected the column at position 0 (`Name`) as a DataFrame because the list wrapping makes it 2-D. 

#### Example 2: Column Slice (Range)

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9],
    'D': [10, 11, 12]
})

# Step 3: Select columns 1 through 3 (exclusive)
print("Columns 1 to 3 (exclusive):")
print(df.iloc[:, 1:3])

# Step 4: Select columns 0 and 2
print("\nColumns 0 and 2:")
print(df.iloc[:, [0, 2]])
```

**Expected Output:**
```
Columns 1 to 3 (exclusive):
   B  C
0  4  7
1  5  8
2  6  9

Columns 0 and 2:
   A  C
0  1  7
1  2  8
2  3  9
```

**Why this output:** `.iloc[:, 1:3]` returned columns at positions 1 and 2 (`B` and `C`), excluding position 3 (`D`). `.iloc[:, [0, 2]]` returned columns `A` and `C` in that order.

#### Example 3: Row and Column Ranges Combined

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eva'],
    'Age': [25, 30, 35, 40, 45],
    'Salary': [50000, 60000, 70000, 80000, 90000],
    'Department': ['HR', 'IT', 'Finance', 'IT', 'Marketing']
})

# Step 3: Select rows 1-3 and columns 0-1
subset = df.iloc[1:4, 0:2]
print(subset)
```

**Expected Output:**
```
      Name  Age
1      Bob   30
2  Charlie   35
3    David   40
```

**Why this output:** `.iloc[1:4, 0:2]` selected rows at positions 1, 2, and 3 (stop 4 excluded) and columns at positions 0 and 1 (stop 2 excluded). 

### Real-World Cases with Explanation

**Case 1: Feature Matrix Construction** — A data scientist selects a subset of feature columns by position using `df.iloc[:, 2:10]` when the feature columns are known to be in a specific positional range.

**Case 2: Report Subsetting** — A business analyst selects specific columns by position for a report using `df.iloc[:, [0, 3, 5]]`.

**Case 3: Time Series Subsetting** — A financial analyst selects columns from position 1 to 4 using `df.iloc[:, 1:5]`.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- Pandas Slicing Guide — https://github.com/Compile-N-Run/Compile-N-Run/blob/5e277f39f3e5b8b740aba0789e01e388911932ce/docs/library/pandas/5-pandas-data-selection/7-pandas-slicing.mdx

---

## Core Concept 4: Scalar Access

### Definitions

**Core Definition:** Scalar access with `.iloc` extracts a single primitive data point from the DataFrame matrix grid by feeding raw coordinate integer pairs (`df.iloc[row_idx, col_idx]`).

**Technical Definition:** `df.iloc[row_pos, col_pos]` returns a scalar value (NumPy scalar, Python scalar, or pandas extension scalar) when both indexers are scalar integers. This is the equivalent of the deprecated `.ix` scalar access. For fast scalar access, `DataFrame.iat` is the recommended method, as `.iloc` may have additional overhead. 

**Beginner-Friendly Explanation:** You can pull out a single value from your table by giving its row position and column position. For example, `df.iloc[0, 1]` gives you the value in the first row, second column.

### Purposes

- To extract a single primitive value from a DataFrame.
- To access a specific cell in a grid without creating intermediate Series or DataFrames.
- To obtain scalar values for use in calculations or conditions.
- To read individual data points in a matrix-like structure.
- To enable fast scalar access when performance is critical.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Scalar access (row, column)
df.iloc[row_pos, col_pos]

# Fast scalar access
df.iat[row_pos, col_pos]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `row_pos` | Zero-based row position | Yes |
| `col_pos` | Zero-based column position | Yes |

#### Syntax Rules

- Both indexers must be scalar integers.
- Returns a scalar value (not a Series or DataFrame).
- Out-of-bounds positions raise `IndexError`.
- `df.iat` is the faster alternative for scalar access.

#### Constraints and Limitations

- Does not return a Series or DataFrame; only a single value.
- Cannot be used to select multiple cells.
- Float positions raise `TypeError`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic Scalar Access

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9]
})

# Step 3: Access cell at row 1, column 2
value = df.iloc[1, 2]
print(f"df.iloc[1, 2] = {value}")
print(f"Type: {type(value).__name__}")
```

**Expected Output:**
```
df.iloc[1, 2] = 8
Type: int64
```

**Why this output:** `.iloc[1, 2]` returned the scalar at row position 1, column position 2 (value 8). The type is `int64` (NumPy scalar). 

#### Example 2: Scalar Access with Negative Indices

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Score': [85, 92, 78]
})

# Step 3: Access last row, last column
value = df.iloc[-1, -1]
print(f"df.iloc[-1, -1] = {value}")

# Step 4: Access second row, first column
value2 = df.iloc[1, 0]
print(f"df.iloc[1, 0] = {value2}")
```

**Expected Output:**
```
df.iloc[-1, -1] = 78
df.iloc[1, 0] = Bob
```

**Why this output:** `.iloc[-1, -1]` retrieved the last row's last column (78). `.iloc[1, 0]` retrieved the second row's first column (`'Bob'`).

#### Example 3: iat vs. iloc for Scalar Access

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'A': [10, 20], 'B': [30, 40]})

# Step 3: Scalar access with .iloc
print(f"iloc: {df.iloc[0, 1]}")

# Step 4: Scalar access with .iat (faster)
print(f"iat: {df.iat[0, 1]}")
```

**Expected Output:**
```
iloc: 30
iat: 30
```

**Why this output:** Both methods returned the same scalar value. `.iat` is recommended for fast scalar access because it bypasses the indexer machinery overhead. 

### Real-World Cases with Explanation

**Case 1: Specific Measurement Lookup** — A scientist retrieves a single sensor reading from a specific row and column using `df.iloc[42, 3]`.

**Case 2: Configuration Value** — A data engineer reads a specific configuration value from a settings DataFrame by position.

**Case 3: Loop Optimization** — A developer uses `.iat` inside a loop for fast scalar access when iterating over a DataFrame by position.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- pandas.DataFrame.iat — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iat.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position

---

## Core Concept 5: Negative Position Handling

### Definitions

**Core Definition:** Negative position handling with `.iloc` steps backward from the trailing edges of the matrix coordinates using negative integers (e.g., `-1` for the last row or column), mirroring Python's negative indexing semantics.

**Technical Definition:** `.iloc[-1]` selects the last row; `.iloc[-2]` selects the second-to-last row; `.iloc[:, -1]` selects the last column. Negative indices in slices also work: `.iloc[-3:]` selects the last three rows. Negative indices count from the end of the axis, where `-1` is the last element, `-2` is the second-to-last, and so on. 

**Beginner-Friendly Explanation:** Negative numbers let you count from the end. `-1` means the last item, `-2` means the second-to-last, and so on. This is convenient when you want the last row or column without knowing the exact length.

### Purposes

- To access trailing records without knowing the DataFrame length.
- To select the last N rows or columns.
- To reverse the order of rows or columns.
- To avoid computing explicit lengths for end-relative selection.
- To support Pythonic slicing idioms.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Last row
df.iloc[-1]

# Second-to-last row
df.iloc[-2]

# Last column
df.iloc[:, -1]

# Last N rows
df.iloc[-N:]

# Last N columns
df.iloc[:, -N:]

# Reverse all rows
df.iloc[::-1]
```

#### Component Breakdown

| Component | Description | Required? |
|-----------|-------------|-----------|
| `-1` | Last element | Yes |
| `-N` | Nth from end | Yes |
| `[::-1]` | Reverse | Yes |

#### Syntax Rules

- Negative indices count from the end of the axis.
- `-1` is the last element, `-2` is the second-to-last, etc.
- Slices with negative indices follow Python semantics.
- `[::-1]` reverses the order of the axis.

#### Constraints and Limitations

- Negative positions beyond the length raise `IndexError`.
- Mixing positive and negative indices in slices works as expected.
- `.iloc[:, :-1]` selects all columns except the last.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Last Row and Column

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Score': [85, 92, 78]
})

# Step 3: Select last row
print("Last row:")
print(df.iloc[-1])

# Step 4: Select last column
print("\nLast column:")
print(df.iloc[:, -1])
```

**Expected Output:**
```
Last row:
Name     Charlie
Score         78
Name: 2, dtype: object

Last column:
0    85
1    92
2    78
Name: Score, dtype: int64
```

**Why this output:** `.iloc[-1]` selected the last row (position -1 = position 2). `.iloc[:, -1]` selected the last column (`Score`). 

#### Example 2: Last N Rows and Columns

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({
    'A': [1, 2, 3, 4, 5],
    'B': [10, 20, 30, 40, 50],
    'C': [100, 200, 300, 400, 500]
})

# Step 3: Last 2 rows
print("Last 2 rows:")
print(df.iloc[-2:])

# Step 4: Last 2 columns
print("\nLast 2 columns:")
print(df.iloc[:, -2:])
```

**Expected Output:**
```
Last 2 rows:
   A   B    C
3  4  40  400
4  5  50  500

Last 2 columns:
    B    C
0  10  100
1  20  200
2  30  300
3  40  400
4  50  500
```

**Why this output:** `.iloc[-2:]` selected the last two rows (positions 3 and 4). `.iloc[:, -2:]` selected the last two columns (`B` and `C`).

#### Example 3: Reversing Rows

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create DataFrame
df = pd.DataFrame({'value': [10, 20, 30, 40, 50]})

# Step 3: Reverse row order
print("Reversed:")
print(df.iloc[::-1])
```

**Expected Output:**
```
Reversed:
   value
4     50
3     40
2     30
1     20
0     10
```

**Why this output:** The slice `[::-1]` with step -1 reversed the order of rows, starting from the last and going to the first.

### Real-World Cases with Explanation

**Case 1: Latest Record** — A data engineer uses `.iloc[-1]` to retrieve the most recent record from a time-ordered DataFrame.

**Case 2: Last Column Metrics** — A financial analyst uses `.iloc[:, -1]` to access the most recent metric column when column order represents time.

**Case 3: Reversing for Display** — A business analyst uses `.iloc[::-1]` to reverse the row order for a bottom-up report display.

### References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- Negative Indexing — https://stackoverflow.com/questions/25229593/why-cant-i-use-negative-indexing-for-this-pandas-dataframe-column

---

## Summary Table: .iloc Selection Patterns

| Pattern | Syntax | Returns | Notes |
|---------|--------|---------|-------|
| Single row | `df.iloc[pos]` | Series | Position-based |
| Multiple rows | `df.iloc[[p1, p2]]` | DataFrame | List of positions |
| Row slice | `df.iloc[start:stop]` | DataFrame | Exclusive stop |
| Single cell | `df.iloc[r, c]` | Scalar | Both scalars |
| Row + column | `df.iloc[rows, cols]` | DataFrame | Atomic operation |
| Boolean array | `df.iloc[bool_array]` | DataFrame | Positional boolean |
| Callable | `df.iloc[lambda d: ...]` | DataFrame | Function-based |
| Negative last row | `df.iloc[-1]` | Series | Last row |
| Negative last column | `df.iloc[:, -1]` | Series | Last column |
| Reverse | `df.iloc[::-1]` | DataFrame | Reversed rows |

---

## References

- pandas.DataFrame.iloc — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html
- pandas.Series.iloc — https://pandas.pydata.org/docs/reference/api/pandas.Series.iloc.html
- pandas.DataFrame.iat — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iat.html
- Selection by Position — https://pandas.pydata.org/docs/user_guide/indexing.html#selection-by-position
- Indexing and Selecting Data — https://pandas.pydata.org/docs/user_guide/indexing.html
- Pandas Slicing Guide — https://github.com/Compile-N-Run/Compile-N-Run/blob/5e277f39f3e5b8b740aba0789e01e388911932ce/docs/library/pandas/5-pandas-data-selection/7-pandas-slicing.mdx
- Negative Indexing in pandas — https://stackoverflow.com/questions/25229593/why-cant-i-use-negative-indexing-for-this-pandas-dataframe-column
- pandas.DataFrame.iloc (Chinese mirror) — https://pandas.ac.cn/pandas-docs/stable/reference/api/pandas.DataFrame.iloc.html
- What's New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html