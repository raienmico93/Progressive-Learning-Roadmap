# Comprehensive Programming Cheat Sheet: Pandas Series Inspection & Metadata Tracking

---

## Topic Overview

### Definitions

**Core Definition:** Series inspection and metadata tracking refers to the set of pandas methods and attributes used to examine the structure, contents, statistical properties, and memory footprint of a Series object without modifying its data.

**Technical Definition:** Pandas provides a suite of inspection methods that operate at different levels of granularity: `.head()` and `.tail()` for row-level scanning; `.info()` for macro-level structural summaries; `.describe()` for statistical summaries (with dtype-dependent output); `.value_counts()` for frequency distribution analysis; `.unique()` and `.nunique()` for distinct-value enumeration; and a set of attributes (`.memory_usage()`, `.dtype`, `.values`, `.to_numpy()`, `.array`) for interrogating the physical storage and data representation of the underlying array.

**Beginner-Friendly Explanation:** When you first receive a dataset, you want to understand what it looks like before doing any analysis. Pandas provides tools to peek at the first or last few rows, get a summary of the data types and memory usage, compute basic statistics, count how often each value appears, and inspect how the data is stored in memory. These inspection tools help you catch problems early and plan your analysis.

### Key Characteristics

- **Non-destructive:** Inspection methods return new objects or print summaries; they do not modify the original Series.
- **Dtype-aware:** `.describe()` and `.info()` adapt their output based on whether the Series is numeric, categorical, or datetime-typed.
- **Missing-value sensitive:** Most methods handle `NaN` according to configurable defaults (`skipna`, `dropna`).
- **Memory-aware:** `.memory_usage(deep=True)` provides accurate byte-level memory reporting.
- **Version-specific:** pandas 3.0 introduced the `str` dtype, changing dtype inference and inspection behavior.

### Prerequisites

- Basic Python syntax and familiarity with pandas Series construction.
- Understanding of NumPy dtypes and memory layout.
- Familiarity with Series indexing and basic operations.

### Related Programming Areas

- **Exploratory Data Analysis (EDA):** Initial data understanding and quality assessment.
- **Data Cleaning:** Identifying missing values, duplicates, and type inconsistencies.
- **Performance Optimization:** Memory profiling and dtype selection.
- **Feature Engineering:** Understanding value distributions before transformation.
- **Data Validation:** Verifying data integrity and structural expectations.

### Core Concepts / Features

1. head() & tail()
2. info()
3. describe()
4. value_counts()
5. unique() & nunique()
6. Memory & Storage Attributes

---

## Core Concept 1: head() & tail()

### Definitions

**Core Definition:** `head()` and `tail()` are Series methods that return the first or last `n` rows of a Series, respectively, enabling quick visual inspection of data format and structure.

**Technical Definition:** `Series.head(n=5)` returns a new Series containing the first `n` rows, preserving the original index. `Series.tail(n=5)` returns a new Series containing the last `n` rows. Both methods accept an integer `n` (positive or negative) and return the same dtype as the original Series. For very long Series, these methods provide a memory-efficient way to preview data without loading the entire object into a display context.

**Beginner-Friendly Explanation:** `head()` shows you the first few rows, and `tail()` shows you the last few rows. It's like flipping to the front or back of a book to see what's there before reading the whole thing.

### Purposes

- To quickly verify the format and structure of a newly created or loaded Series.
- To confirm that data was ingested correctly without displaying thousands of rows.
- To inspect the beginning and end of a time-ordered Series for anomalies.
- To preview data before applying transformations.
- To check that index labels and dtypes are as expected.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.head(n=5)
s.tail(n=5)
```

#### Component Breakdown

| Parameter | Description | Required? | Default |
|-----------|-------------|-----------|---------|
| `n` | Number of rows to return (can be negative) | No | `5` |

#### Syntax Rules

- `n` must be an integer; floats raise `TypeError`.
- Negative `n` returns all rows except the last (or first) `|n|` rows.
- The returned Series retains the original index labels.
- Both methods return a new Series; the original is unchanged.

#### Constraints and Limitations

- `head()` and `tail()` do not modify the original Series.
- For a Series with fewer than `n` elements, the entire Series is returned.
- Display truncation in Jupyter may limit visible output regardless of `n`.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic head() and tail()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create a Series
s = pd.Series(range(100), name='numbers')

# Step 3: Display first 3 rows
print("Head (3):")
print(s.head(3))

# Step 4: Display last 3 rows
print("\nTail (3):")
print(s.tail(3))
```

**Expected Output:**
```
Head (3):
0    0
1    1
2    2
Name: numbers, dtype: int64

Tail (3):
97    97
98    98
99    99
Name: numbers, dtype: int64
```

**Why this output:** `.head(3)` returned the first three rows (positions 0–2), and `.tail(3)` returned the last three rows (positions 97–99). The `name` attribute was preserved.

#### Example 2: Negative n Parameter

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series
s = pd.Series([10, 20, 30, 40, 50])

# Step 3: head with negative n
print("head(-2):")
print(s.head(-2))

# Step 4: tail with negative n
print("\ntail(-2):")
print(s.tail(-2))
```

**Expected Output:**
```
head(-2):
0    10
1    20
2    30
dtype: int64

tail(-2):
2    30
3    40
4    50
dtype: int64
```

**Why this output:** `head(-2)` returned all rows except the last 2; `tail(-2)` returned all rows except the first 2.

#### Example 3: Inspecting a String Series

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create string Series
s = pd.Series(['alpha', 'beta', 'gamma', 'delta', 'epsilon'], name='greek')

# Step 3: Preview
print(s.head(2))
print("\n---")
print(s.tail(2))
```

**Expected Output:**
```
0    alpha
1     beta
Name: greek, dtype: str

---
3     delta
4    epsilon
Name: greek, dtype: str
```

**Why this output:** Under pandas 3.0, string data is inferred as the `str` dtype (not `object`). The `name` attribute `'greek'` is preserved.

### Real-World Cases with Explanation

**Case 1: CSV Ingestion Verification** — A data engineer loads a large CSV into a Series and uses `.head()` and `.tail()` to confirm that parsing was correct and that no trailing summary rows were included.

**Case 2: Time Series Boundary Check** — A financial analyst uses `.head()` and `.tail()` on a datetime-indexed stock price Series to verify that the date range covers the expected period.

**Case 3: ETL Pipeline Debugging** — A data pipeline developer inserts `.head()` calls at each transformation stage to verify that data shapes and values remain correct.

### References

- pandas.Series.head — https://pandas.pydata.org/docs/reference/api/pandas.Series.head.html
- pandas.Series.tail — https://pandas.pydata.org/docs/reference/api/pandas.Series.tail.html
- pandas Cheat Sheet — https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf

---

## Core Concept 2: info()

### Definitions

**Core Definition:** `info()` prints a concise summary of a Series, including the class type, index dtype, number of entries, non-null count, dtype, and memory usage.

**Technical Definition:** `Series.info(verbose=None, buf=None, max_cols=None, memory_usage=None, show_counts=True)` outputs a summary to `sys.stdout` (or a writable buffer). For a Series, the summary includes: `<class 'pandas.core.series.Series'>`, the index range and number of entries, the Series name, the non-null count, the dtype, and memory usage. The `memory_usage` parameter accepts `bool`, `str` (e.g., `'deep'`), or `None` to control whether and how memory usage is displayed.

**Beginner-Friendly Explanation:** `info()` is like a “report card” for your Series. It tells you what kind of data it is, how many entries it has, what data type it uses, and how much memory it takes up. This is one of the first commands you run when you encounter a new dataset.

### Purposes

- To obtain a macro-level structural summary of a Series in a single call.
- To verify the dtype and non-null count of a Series.
- To inspect memory usage without computing it separately.
- To check index dtype and range.
- To document the structural properties of a Series for reproducibility.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.info(verbose=None, buf=None, max_cols=None, memory_usage=None, show_counts=True)
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
- In pandas 1.4.0 and later, `info()` was added as a Series method.

#### Constraints and Limitations

- **Version-specific:** `Series.info()` was added in pandas 1.4.0. Earlier versions only had `DataFrame.info()`.
- **Performance:** Deep memory introspection can be slow for large Series.
- **Display truncation:** Very large Series may have output truncated based on display options.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic info() Output

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series with mixed non-null values
s = pd.Series([1, 2, 3, 4, 5], index=['a', 'b', 'c', 'd', 'e'], name='my_series')

# Step 3: Call info()
s.info()
```

**Expected Output:**
```
<class 'pandas.core.series.Series'>
Index: 5 entries, a to e
Series name: my_series
Non-Null Count  Dtype
--------------  -----
5 non-null      int64
dtypes: int64(1)
memory usage: 120.0+ bytes
```

**Why this output:** `info()` reported the class type, index range (`a` to `e`), Series name (`my_series`), non-null count (5), dtype (`int64`), and estimated memory usage.

#### Example 2: info() with Missing Values

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create Series with NaN
s = pd.Series([1, 2, np.nan, 4, np.nan])

# Step 3: Call info()
s.info()
```

**Expected Output:**
```
<class 'pandas.core.series.Series'>
Index: 5 entries, 0 to 4
Series name: None
Non-Null Count  Dtype
--------------  -----
3 non-null      float64
dtypes: float64(1)
memory usage: 160.0 bytes
```

**Why this output:** The non-null count is 3 (positions 0, 1, and 3), and the dtype is `float64` because `NaN` forces floating-point representation.

#### Example 3: info() with Deep Memory Introspection

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create string Series
s = pd.Series(['alpha', 'beta', 'gamma'])

# Step 3: Call info() with deep memory
s.info(memory_usage='deep')
```

**Expected Output:**
```
<class 'pandas.core.series.Series'>
Index: 3 entries, 0 to 2
Series name: None
Non-Null Count  Dtype
--------------  -----
3 non-null      str
dtypes: str(1)
memory usage: 180.0 bytes
```

**Why this output:** With `memory_usage='deep'`, pandas performed actual memory introspection, accounting for the full memory consumed by the string objects. Under pandas 3.0, the dtype is `str`.

### Real-World Cases with Explanation

**Case 1: Dataset Onboarding** — A data scientist loads a new dataset and calls `.info()` first to understand the structure, dtypes, and missing-value situation before any analysis.

**Case 2: Memory Audit** — A data engineer calls `.info(memory_usage='deep')` on a large Series to identify memory bottlenecks and decide whether to convert to a more efficient dtype.

**Case 3: Pipeline Logging** — A machine learning engineer logs `.info()` output at each stage of a preprocessing pipeline to track dtype changes and missing-value counts.

### References

- pandas.Series.info — https://pandas.pydata.org/docs/reference/api/pandas.Series.info.html
- Frequently Asked Questions: Memory Usage — https://pandas.pydata.org/docs/user_guide/gotchas.html

---

## Core Concept 3: describe()

### Definitions

**Core Definition:** `describe()` generates rapid summary statistics for a Series, producing central tendency and dispersion measures for numeric data, and frequency summaries for categorical or object data.

**Technical Definition:** `Series.describe(percentiles=None, include=None, exclude=None)` returns a Series of summary statistics. For numeric dtypes, the output includes: `count`, `mean`, `std`, `min`, the 25th, 50th (median), and 75th percentiles, and `max`. For object dtypes (e.g., timestamps or strings), the output includes `count`, `unique`, `top` (most common value), and `freq` (frequency of the most common value). The `include` and `exclude` parameters are ignored for Series. The output excludes `NaN` values by default.

**Beginner-Friendly Explanation:** `describe()` gives you a quick statistical snapshot of your data. For numbers, it tells you the average, how spread out the data is, and the range. For text or categorical data, it tells you how many unique values there are and which one appears most often.

### Purposes

- To generate summary statistics in a single method call.
- To compare the distributional properties of different Series.
- To identify outliers via min/max and percentiles.
- To verify data quality (e.g., unexpected counts or frequencies).
- To document the statistical properties of a dataset for reports.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.describe(percentiles=None, include=None, exclude=None)
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `percentiles` | List of percentiles to include (in [0, 1]) | `[.25, .5, .75]` |
| `include` | Dtypes to include (ignored for Series) | `None` |
| `exclude` | Dtypes to exclude (ignored for Series) | `None` |

#### Syntax Rules

- For numeric Series, output includes `count`, `mean`, `std`, `min`, percentiles, and `max`.
- For object/string Series, output includes `count`, `unique`, `top`, and `freq`.
- The `include` and `exclude` parameters are ignored for Series (they apply to DataFrames).
- Custom percentiles can be specified as a list in the `[0, 1]` interval.

#### Constraints and Limitations

- `NaN` values are excluded from all computations.
- For mixed-dtype Series, output may be limited.
- If multiple values have the highest count, the `top` value is arbitrarily chosen.
- Timestamp Series include first and last items in the output.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Numeric Series describe()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create numeric Series
s = pd.Series([10, 20, 30, 40, 50])

# Step 3: Call describe()
print(s.describe())
```

**Expected Output:**
```
count     5.000000
mean     30.000000
std      15.811388
min      10.000000
25%      20.000000
50%      30.000000
75%      40.000000
max      50.000000
dtype: float64
```

**Why this output:** The numeric Series produced the standard seven-statistic summary: count, mean, standard deviation, minimum, 25th percentile, median, 75th percentile, and maximum.

#### Example 2: Object Series describe()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create string Series
s = pd.Series(['a', 'a', 'b', 'c'])

# Step 3: Call describe()
print(s.describe())
```

**Expected Output:**
```
count     4
unique    3
top       a
freq      2
dtype: object
```

**Why this output:** For object/string Series, `describe()` returned count (4), unique values (3: `a`, `b`, `c`), the most common value (`a`), and its frequency (2).

#### Example 3: describe() with Custom Percentiles

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series
s = pd.Series([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# Step 3: Describe with custom percentiles
print(s.describe(percentiles=[.1, .5, .9]))
```

**Expected Output:**
```
count    10.0
mean      5.5
std       3.02765
min       1.0
10%       1.9
50%       5.5
90%       9.1
max      10.0
dtype: float64
```

**Why this output:** The custom percentiles `[.1, .5, .9]` replaced the default `[.25, .5, .75]`, showing the 10th, 50th, and 90th percentiles.

### Real-World Cases with Explanation

**Case 1: Salary Analysis** — An HR analyst uses `.describe()` on a salary Series to understand the distribution, identify the median, and detect potential outliers via min/max.

**Case 2: Survey Response Analysis** — A researcher uses `.describe()` on a categorical Series to identify the most common response and the number of unique responses.

**Case 3: Sensor Calibration** — An engineer uses `.describe()` on sensor readings to verify that the measurement range is within expected bounds.

### References

- pandas.Series.describe — https://pandas.pydata.org/docs/reference/api/pandas.Series.describe.html
- Descriptive Statistics — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics

---

## Core Concept 4: value_counts()

### Definitions

**Core Definition:** `value_counts()` computes the frequency of each unique value in a Series, returning a new Series sorted in descending order of frequency, with the original values as the index and counts as the data.

**Technical Definition:** `Series.value_counts(normalize=False, sort=True, ascending=False, bins=None, dropna=True)` returns a Series containing counts of unique values. When `normalize=True`, the result contains relative frequencies (proportions) obtained by dividing each count by the total number of non-null values. The `bins` parameter groups numeric values into half-open bins (using `pd.cut`), and `dropna=False` includes `NaN` in the counts.

**Beginner-Friendly Explanation:** `value_counts()` tells you how many times each distinct value appears in your Series. It's like tallying votes—each unique value gets a count of how many times it showed up, and the results are sorted with the most frequent value first.

### Purposes

- To compute frequency distributions of categorical or discrete data.
- To identify the most and least common values in a Series.
- To compute proportions using `normalize=True`.
- To bin continuous data into categorical intervals using `bins`.
- To detect data quality issues such as unexpected value frequencies.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.value_counts(normalize=False, sort=True, ascending=False, bins=None, dropna=True)
```

#### Component Breakdown

| Parameter | Description | Default |
|-----------|-------------|---------|
| `normalize` | Return proportions instead of counts | `False` |
| `sort` | Sort by frequency | `True` |
| `ascending` | Sort in ascending order | `False` |
| `bins` | Number of half-open bins (numeric only) | `None` |
| `dropna` | Include `NaN` counts | `True` |

#### Syntax Rules

- The result is a Series indexed by unique values, with counts as data.
- When `normalize=True`, values are divided by the sum of all counts (proportions sum to 1.0).
- `bins` only works with numeric data and uses `pd.cut` semantics.
- `dropna=False` includes `NaN` as a separate category in the output.
- By default, `NaN` values are excluded from the counts.

#### Constraints and Limitations

- `bins` is incompatible with `normalize=True` and `dropna=False` simultaneously in some versions.
- For very high-cardinality Series, output may be large.
- `sort=True` sorts by count, not by value; use `sort_index()` afterward if value-order sorting is needed.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic value_counts()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series with repeated values
s = pd.Series(['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'])

# Step 3: Compute value counts
print(s.value_counts())
```

**Expected Output:**
```
apple     3
banana    2
cherry    1
dtype: int64
```

**Why this output:** The Series was sorted by frequency in descending order. `'apple'` appeared 3 times, `'banana'` 2 times, and `'cherry'` 1 time.

#### Example 2: Normalized value_counts()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series
s = pd.Series([3, 1, 2, 3, 4])

# Step 3: Compute normalized counts
print(s.value_counts(normalize=True))
```

**Expected Output:**
```
3    0.4
1    0.2
2    0.2
4    0.2
dtype: float64
```

**Why this output:** With `normalize=True`, each count was divided by the total (5), producing proportions that sum to 1.0. Value `3` appeared twice (0.4), and the others appeared once each (0.2).

#### Example 3: value_counts() with Bins and dropna=False

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create Series with NaN
s = pd.Series([1, 2, 2, 3, 3, 3, np.nan])

# Step 3: Binned counts
print("Binned:")
print(s.value_counts(bins=3))

# Step 4: Include NaN
print("\nWith NaN:")
print(s.value_counts(dropna=False))
```

**Expected Output:**
```
Binned:
(2.0, 3.0]    3
(0.996, 2.0]  2
(3.0, 3.0]    0
dtype: int64

With NaN:
3.0    3
2.0    2
NaN    1
1.0    1
dtype: int64
```

**Why this output:** The `bins=3` parameter grouped numeric values into three half-open intervals. With `dropna=False`, the `NaN` value appeared as a separate category with count 1.

### Real-World Cases with Explanation

**Case 1: E-commerce Product Analysis** — An analyst uses `.value_counts()` on a product category Series to identify the most popular categories and inform inventory decisions.

**Case 2: Data Quality Assessment** — A data engineer uses `.value_counts(dropna=False)` to quantify the proportion of missing values in a Series.

**Case 3: Survey Distribution** — A researcher uses `.value_counts(normalize=True)` to compute the percentage of respondents in each category.

### References

- pandas.Series.value_counts — https://pandas.pydata.org/docs/reference/api/pandas.Series.value_counts.html
- pandas.cut — https://pandas.pydata.org/docs/reference/api/pandas.cut.html

---

## Core Concept 5: unique() & nunique()

### Definitions

**Core Definition:** `unique()` returns an array of distinct values present in a Series in order of appearance, while `nunique()` returns the count of distinct values, with optional inclusion or exclusion of `NaN` values.

**Technical Definition:** `Series.unique()` returns a NumPy ndarray (or ExtensionArray) of unique values in order of appearance, significantly faster than `numpy.unique`. `Series.nunique(dropna=True)` returns the number of unique elements, excluding `NaN` by default. Setting `dropna=False` includes `NaN` in the count. Note that `unique()` does not have a `dropna` parameter; it includes `NaN` in the returned array if present.

**Beginner-Friendly Explanation:** `unique()` shows you all the different values that appear in your Series (without duplicates), and `nunique()` tells you how many different values there are. If your Series has `NaN` values, `nunique()` won't count them unless you explicitly ask it to.

### Purposes

- To extract a list of distinct values for categorical analysis.
- To count the cardinality of a Series.
- To verify data quality by checking for unexpected unique values.
- To support encoding decisions (e.g., one-hot encoding, label encoding).
- To compare cardinality across different Series.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.unique()
s.nunique(dropna=True)
```

#### Component Breakdown

| Method | Parameter | Description | Default |
|--------|-----------|-------------|---------|
| `.unique()` | — | Returns array of distinct values (includes `NaN`) | — |
| `.nunique()` | `dropna` | Exclude `NaN` from count | `True` |

#### Syntax Rules

- `.unique()` returns values in order of first appearance.
- `.nunique()` counts distinct values; `dropna=True` excludes `NaN`.
- `.unique()` does not accept `dropna`; `NaN` is always included if present.
- `.unique()` is significantly faster than `numpy.unique` for Series objects.

#### Constraints and Limitations

- **Inconsistency:** `.unique()` includes `NaN`, but `.nunique(dropna=True)` excludes it. This can lead to confusion when comparing results.
- **GitHub issue #61209** discusses making `unique()` and `nunique()` consistent with respect to `NaN` handling.
- `.unique()` returns a NumPy array; for extension dtypes, it returns an ExtensionArray.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Basic unique() and nunique()

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create Series with duplicates
s = pd.Series(['c', 'a', 'd', 'a', 'b', 'c'])

# Step 3: Get unique values
print("Unique values:")
print(s.unique())

# Step 4: Count unique values
print(f"\nNumber of unique values: {s.nunique()}")
```

**Expected Output:**
```
Unique values:
['c' 'a' 'd' 'b']

Number of unique values: 4
```

**Why this output:** `.unique()` returned the distinct values in order of first appearance (`c`, `a`, `d`, `b`). `.nunique()` counted 4 distinct values.

#### Example 2: unique() and nunique() with NaN

```python
# Step 1: Import pandas and numpy
import pandas as pd
import numpy as np

# Step 2: Create Series with NaN
s = pd.Series([1, 2, np.nan, 2, 1])

# Step 3: Unique values (includes NaN)
print("Unique (includes NaN):")
print(s.unique())

# Step 4: nunique with default dropna=True
print(f"\nnunique (dropna=True): {s.nunique()}")

# Step 5: nunique with dropna=False
print(f"nunique (dropna=False): {s.nunique(dropna=False)}")
```

**Expected Output:**
```
Unique (includes NaN):
[ 1.  2. nan]

nunique (dropna=True): 2
nunique (dropna=False): 3
```

**Why this output:** `.unique()` included `NaN` in the array. `.nunique()` with `dropna=True` counted only 2 (excluding `NaN`), while `dropna=False` counted 3 (including `NaN`).

#### Example 3: Cardinality Check for Encoding

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create categorical Series
s = pd.Series(['red', 'blue', 'red', 'green', 'blue', 'red'], name='color')

# Step 3: Check cardinality
n = s.nunique()
print(f"Cardinality: {n}")
print(f"Values: {s.unique().tolist()}")

# Step 4: Conditional logic
if n < 10:
    print("Suitable for one-hot encoding")
else:
    print("Consider target encoding or hashing")
```

**Expected Output:**
```
Cardinality: 3
Values: ['red', 'blue', 'green']
Suitable for one-hot encoding
```

**Why this output:** The Series had 3 unique values, which is below the threshold of 10, making it suitable for one-hot encoding.

### Real-World Cases with Explanation

**Case 1: Categorical Encoding Decisions** — A data scientist checks `.nunique()` on all categorical Series to decide between one-hot encoding (low cardinality) and target encoding (high cardinality).

**Case 2: Data Quality Validation** — A data engineer uses `.unique()` to verify that a categorical column contains only expected values and no typos or unexpected categories.

**Case 3: Deduplication Verification** — A data analyst uses `.nunique()` to confirm that a supposedly unique identifier column truly has no duplicates.

### References

- pandas.Series.unique — https://pandas.pydata.org/docs/reference/api/pandas.Series.unique.html
- pandas.Series.nunique — https://pandas.pydata.org/docs/reference/api/pandas.Series.nunique.html
- GitHub Issue: Consistent NA handling in unique() and nunique() — https://github.com/pandas-dev/pandas/issues/61209

---

## Core Concept 6: Memory & Storage Attributes

### Definitions

**Core Definition:** Memory and storage attributes in pandas Series provide direct access to the physical storage characteristics of the Series, including its memory footprint (`.memory_usage()`), data type (`.dtype`), and the underlying array representation (`.values`, `.to_numpy()`, `.array`).

**Technical Definition:** `Series.memory_usage(index=True, deep=False)` returns the memory usage of the Series in bytes. `Series.dtype` returns the NumPy dtype or ExtensionDtype of the underlying data. `Series.values` returns a NumPy representation of the data (deprecated in favor of `.array` and `.to_numpy()`). `Series.to_numpy(dtype=None, copy=False, na_value=lib.no_default)` returns a NumPy ndarray, potentially copying or coercing values. `Series.array` returns the underlying ExtensionArray, never copying data. Under pandas 3.0, string data is inferred as the `str` dtype (backed by PyArrow if available), changing how dtypes and underlying arrays are represented.

**Beginner-Friendly Explanation:** These attributes let you look “under the hood” of a Series. `.memory_usage()` tells you how much RAM it uses. `.dtype` tells you what kind of data it holds. `.values`, `.to_numpy()`, and `.array` give you the raw data in different forms, depending on whether you want a NumPy array or a pandas array.

### Purposes

- To measure the memory footprint of a Series for optimization.
- To verify or enforce the data type of a Series.
- To extract the underlying array for interoperability with NumPy or other libraries.
- To choose between view and copy semantics for performance.
- To understand the storage representation of extension dtypes.

### Syntax Rules and Structure

#### Complete General Syntax

```python
s.memory_usage(index=True, deep=False)
s.dtype
s.values
s.to_numpy(dtype=None, copy=False, na_value=lib.no_default)
s.array
```

#### Component Breakdown

| Attribute/Method | Description | Key Parameters |
|------------------|-------------|----------------|
| `.memory_usage()` | Memory usage in bytes | `index` (include index), `deep` (introspect objects) |
| `.dtype` | Data type of the Series | — |
| `.values` | NumPy representation (deprecated) | — |
| `.to_numpy()` | NumPy ndarray | `dtype`, `copy`, `na_value` |
| `.array` | ExtensionArray (never copies) | — |

#### Syntax Rules

- `.memory_usage(deep=True)` performs actual memory introspection for object dtypes; without it, memory is estimated.
- `.values` is deprecated in favor of `.array` or `.to_numpy()`.
- `.to_numpy()` may copy or coerce data; use `.array` to strictly avoid a copy.
- `.array` always returns an ExtensionArray, which may be a NumPy wrapper for native dtypes.

#### Constraints and Limitations

- **Deprecation:** `.values` is considered old; `.array` and `.to_numpy()` are recommended.
- **Performance:** `deep=True` is computationally expensive for large Series with object dtypes.
- **Version-specific:** Under pandas 3.0, string Series have `dtype='str'`, and `.array` returns a `StringArray` (PyArrow-backed if installed).
- **Copy semantics:** `.to_numpy()` may return a copy if the dtype is not homogeneous or not a NumPy dtype.

### Multiple Annotated Complete Step-by-Step Code Examples and Their Expected Outputs

#### Example 1: Memory Usage Comparison

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create a small string Series
s = pd.Series(['a', 'b'])

# Step 3: Memory usage without deep
print(f"memory_usage(): {s.memory_usage()}")

# Step 4: Memory usage with deep
print(f"memory_usage(deep=True): {s.memory_usage(deep=True)}")
```

**Expected Output:**
```
memory_usage(): 144
memory_usage(deep=True): 164
```

**Why this output:** Without deep introspection, pandas estimated memory based on dtype and length. With `deep=True`, pandas interrogated the actual string objects, revealing a higher memory footprint (164 bytes) due to the overhead of Python string objects.

#### Example 2: dtype and array Representation

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create integer Series
s = pd.Series([1, 2, 3])

# Step 3: Check dtype
print(f"dtype: {s.dtype}")

# Step 4: Get NumPy array
print(f"to_numpy(): {s.to_numpy()}")
print(f"type: {type(s.to_numpy())}")

# Step 5: Get ExtensionArray
print(f"array: {s.array}")
print(f"type: {type(s.array)}")
```

**Expected Output:**
```
dtype: int64
to_numpy(): [1 2 3]
type: <class 'numpy.ndarray'>
array: <NumpyExtensionArray>
[1, 2, 3]
Length: 3, dtype: int64
type: <class 'pandas.core.arrays.numpy_.NumpyExtensionArray'>
```

**Why this output:** `.to_numpy()` returned a NumPy ndarray, while `.array` returned a `NumpyExtensionArray` wrapper. For NumPy native dtypes, `.array` is a thin (no-copy) wrapper around the ndarray.

#### Example 3: String dtype Under pandas 3.0

```python
# Step 1: Import pandas
import pandas as pd

# Step 2: Create string Series
s = pd.Series(['alpha', 'beta', 'gamma'])

# Step 3: Check dtype
print(f"dtype: {s.dtype}")

# Step 4: Get array representation
print(f"array type: {type(s.array)}")

# Step 5: Memory usage
print(f"memory_usage(deep=True): {s.memory_usage(deep=True)}")
```

**Expected Output:**
```
dtype: str
array type: <class 'pandas.core.arrays.string_.StringArray'>
memory_usage(deep=True): 185
```

**Why this output:** Under pandas 3.0, the string Series was inferred as the `str` dtype (not `object`). The `.array` attribute returned a `StringArray` (PyArrow-backed if installed). Memory usage reflects the overhead of the string storage.

### Real-World Cases with Explanation

**Case 1: Memory Optimization** — A data engineer working with a large string Series uses `.memory_usage(deep=True)` to identify the overhead of object dtypes and converts to `category` or `str` dtype for significant memory savings.

**Case 2: NumPy Interoperability** — A machine learning engineer extracts the underlying NumPy array via `.to_numpy()` to pass data to scikit-learn or PyTorch, accepting the potential copy for compatibility.

**Case 3: ExtensionArray Integration** — A developer building custom pandas extensions uses `.array` to access the ExtensionArray directly without triggering a copy, preserving performance for large datasets.

### References

- pandas.Series.memory_usage — https://pandas.pydata.org/docs/reference/api/pandas.Series.memory_usage.html
- pandas.Series.dtype — https://pandas.pydata.org/docs/reference/api/pandas.Series.dtype.html
- pandas.Series.to_numpy — https://pandas.pydata.org/docs/reference/api/pandas.Series.to_numpy.html
- pandas.Series.array — https://pandas.pydata.org/docs/reference/api/pandas.Series.array.html
- What’s New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html
- Accessing the values in a Series or Index — https://pandas.pydata.org/docs/whatsnew/v0.24.0.html#accessing-the-values-in-a-series-or-index

---

## Summary Table: Inspection & Metadata Methods

| Method/Attribute | Returns | Dtype-Dependent | Missing-Value Default | Memory-Related |
|------------------|---------|-----------------|----------------------|----------------|
| `.head(n)` / `.tail(n)` | Series | No | Includes NaN | No |
| `.info()` | None (prints) | Yes | Shows non-null count | Yes |
| `.describe()` | Series | Yes | Excludes NaN | No |
| `.value_counts()` | Series | No | Excludes NaN (`dropna=True`) | No |
| `.unique()` | ndarray/ExtensionArray | No | Includes NaN | No |
| `.nunique()` | int | No | Excludes NaN (`dropna=True`) | No |
| `.memory_usage()` | int (bytes) | Yes | — | Yes |
| `.dtype` | dtype object | Yes | — | No |
| `.to_numpy()` | ndarray | Yes | — | May copy |
| `.array` | ExtensionArray | Yes | — | No copy |

---

## References

- pandas.Series.head — https://pandas.pydata.org/docs/reference/api/pandas.Series.head.html
- pandas.Series.tail — https://pandas.pydata.org/docs/reference/api/pandas.Series.tail.html
- pandas.Series.info — https://pandas.pydata.org/docs/reference/api/pandas.Series.info.html
- pandas.Series.describe — https://pandas.pydata.org/docs/reference/api/pandas.Series.describe.html
- pandas.Series.value_counts — https://pandas.pydata.org/docs/reference/api/pandas.Series.value_counts.html
- pandas.Series.unique — https://pandas.pydata.org/docs/reference/api/pandas.Series.unique.html
- pandas.Series.nunique — https://pandas.pydata.org/docs/reference/api/pandas.Series.nunique.html
- pandas.Series.memory_usage — https://pandas.pydata.org/docs/reference/api/pandas.Series.memory_usage.html
- pandas.Series.to_numpy — https://pandas.pydata.org/docs/reference/api/pandas.Series.to_numpy.html
- pandas.Series.array — https://pandas.pydata.org/docs/reference/api/pandas.Series.array.html
- What’s New in pandas 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html
- pandas Cheat Sheet — https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf
- GitHub Issue: Consistent NA handling in unique() and nunique() — https://github.com/pandas-dev/pandas/issues/61209
- Accessing the values in a Series or Index — https://pandas.pydata.org/docs/whatsnew/v0.24.0.html#accessing-the-values-in-a-series-or-index