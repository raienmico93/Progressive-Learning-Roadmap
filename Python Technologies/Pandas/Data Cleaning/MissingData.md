# Pandas Missing Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Missing data in pandas refers to values absent from a dataset, represented by sentinel values such as `NaN` (Not a Number), `None`, `NaT` (Not a Time), or `pd.NA`. pandas provides methods to detect, count, visualize, and analyze these missing values across rows and columns.

**Technical Definition:** pandas represents missing values using a combination of NumPy's `np.nan` (a float), Python's `None` (a singleton), pandas' `NaT` (for datetime/timedelta), and `pd.NA` (the unified missing indicator for nullable extension types). Detection is performed via `isna()`/`isnull()` (returning a boolean mask where `True` indicates missingness) and their inverses `notna()`/`notnull()`. Missingness patterns are analyzed through boolean masking, summary metrics, visual diagnostics, and structural profiling across axes.

**Beginner-Friendly Explanation:** When you load real-world data, some cells are often empty. pandas marks these as "missing" in different ways depending on the data type. This cheat sheet shows you how to find those missing values, count how many there are, see if they follow patterns, and decide what to do about them.

### Key Characteristics

- **Multiple sentinel values:** `NaN`, `None`, `NaT`, and `pd.NA` all indicate missingness; `isna()` and `notna()` treat all of them as missing.
- **Empty strings are not missing:** `""` and `" "` are valid strings, not NA values, unless explicitly converted.
- **Boolean masking:** `isna()` returns a same-shaped boolean DataFrame or Series for filtering.
- **Axis-aware operations:** Missingness can be analyzed column-wise (`axis=0`) or row-wise (`axis=1`).
- **Visual diagnostics:** The `missingno` library provides matrix, bar, heatmap, and dendrogram visualizations of missingness.

### Prerequisites

- Basic Python knowledge (lists, functions, dictionaries).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of NumPy's `np.nan` and boolean masking.
- Installation of pandas (`pip install pandas`) and optionally `missingno` (`pip install missingno`) for visualizations.

### Related Programming Areas

- **Data Cleaning & Preprocessing:** Missing data detection is the first step before imputation or deletion.
- **Exploratory Data Analysis (EDA):** Missingness patterns reveal data collection issues and guide feature engineering.
- **Statistical Analysis:** Missing data mechanisms (MCAR, MAR, MNAR) affect the validity of statistical inferences.
- **Machine Learning:** Missing values must be handled before training models; detection informs strategy.

### Core Concepts / Features

1. Boolean Masking: Detecting Missing Values
2. Summary Metrics: Counting and Percentages
3. Pattern Diagnostics: Visualizing Missingness
4. Structural Profiling: Column-wise vs. Row-wise Analysis
5. Implicit vs. Explicit Missingness


## 1. Boolean Masking: Identifying Missing Nodes

### Definitions

**Core Definition:** Boolean masking uses `isna()`, `isnull()`, `notna()`, and `notnull()` to return boolean objects indicating whether each element in a Series or DataFrame is missing.

**Technical Definition:** `DataFrame.isna()` returns a same-sized boolean DataFrame where `True` indicates that the corresponding element is `NA`. NA values such as `None` and `np.nan` map to `True`; all other values map to `False`. `DataFrame.isnull()` is an alias of `isna()`. `DataFrame.notna()` is the boolean inverse. These methods are top-level functions (`pd.isna()`) and methods on both Series and DataFrame.

**Beginner-Friendly Explanation:** `isna()` asks “Is this cell empty?” and gives you a grid of True/False answers. `notna()` asks the opposite: “Is this cell filled?” You can use these answers to filter rows or columns.

### Purposes

- To create a boolean mask identifying which cells contain missing values.
- To filter rows or columns based on missingness.
- To count missing values by summing the boolean mask.
- To check whether any or all values in a Series or DataFrame are missing.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Detection functions
pd.isna(obj)          # Top-level function
pd.notna(obj)

# DataFrame methods
DataFrame.isna()      # Boolean mask, True = missing
DataFrame.isnull()    # Alias of isna
DataFrame.notna()     # Boolean inverse
DataFrame.notnull()   # Alias of notna

# Series methods
Series.isna()
Series.notna()
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `isna()` | Returns `True` for `NA` values (`None`, `np.nan`, `NaT`, `pd.NA`) |
| `isnull()` | Alias of `isna()` |
| `notna()` | Returns `True` for non-NA values |
| `notnull()` | Alias of `notna()` |
| `any(axis=...)` | Returns whether any value is True along axis |
| `all(axis=...)` | Returns whether all values are True along axis |

**Syntax Rules:**

- Empty strings (`""`) and whitespace strings (`" "`) are not considered NA values by `isna()`.
- `numpy.inf` is not considered NA unless `pandas.options.mode.use_inf_as_na = True`.
- `NaT` (Not a Time) is detected as missing by `isna()`.
- `pd.NA` is detected as missing by `isna()`.

**Constraints and Limitations:**

- `isna()` does not detect empty strings or whitespace strings; these must be converted explicitly.
- Very large DataFrames produce large boolean masks; consider using `any()` or `sum()` directly.
- In pandas < 1.0, `isna()` behaviour for `pd.NA` may differ; `pd.NA` was introduced in pandas 1.0.

### Annotated Code Examples

**Example 1: Basic Boolean Masking**

```python
import pandas as pd
import numpy as np

# Create a DataFrame with missing values
df = pd.DataFrame({
    'age' : [5, 6, np.nan],
    'born': [pd.NaT, pd.Timestamp('1939-05-27'), pd.Timestamp('1940-04-25')],
    'name': ['Alfred', 'Batman', ''],
    'toy' : [None, 'Batmobile', 'Joker']
})

print(df)
# Expected output:
#    age       born    name        toy
# 0  5.0        NaT  Alfred       None
# 1  6.0 1939-05-27  Batman  Batmobile
# 2  NaN 1940-04-25            Joker

# Detect missing values
print(df.isna())
# Expected output:
#      age   born   name    toy
# 0  False   True  False   True
# 1  False  False  False  False
# 2   True  False  False  False
```

**Why this output:** `np.nan` in the `age` column is detected as missing (`True`). `pd.NaT` in the `born` column is detected as missing. The empty string `''` in the `name` column is NOT detected as missing (`False`). `None` in the `toy` column is detected as missing.

**Example 2: notna() for Non-Missing Values**

```python
# Check for non-missing values
print(df.notna())
# Expected output:
#      age   born   name    toy
# 0   True  False   True  False
# 1   True   True   True   True
# 2  False   True   True   True

# Count non-missing values per column
print(df.notna().sum())
# Expected output:
# age     2
# born    2
# name    3
# toy     2
# dtype: int64
```

**Why this output:** `notna()` returns the inverse of `isna()`. Summing the boolean mask (where `True` = 1) counts non-missing values. The `name` column has 3 non-missing values because the empty string is not considered missing.

**Example 3: Using any() and all() for Quick Checks**

```python
# Check if any value in each column is missing
print(df.isna().any())
# Expected output:
# age      True
# born     True
# name    False
# toy      True
# dtype: bool

# Check if any value in the entire DataFrame is missing
print(df.isna().any().any())
# Expected output: True

# Check if all values in each column are non-missing
print(df.notna().all())
# Expected output:
# age     False
# born    False
# name     True
# toy     False
# dtype: bool
```

**Why this output:** `df.isna().any()` returns `True` for columns that contain at least one missing value. The `name` column returns `False` because it has no missing values (the empty string is not missing). `notna().all()` returns `True` only for columns with no missing values.

### Real-World Cases

- **CSV imports:** After `pd.read_csv()`, use `df.isna().sum()` to immediately assess data quality.
- **Data validation:** Filter rows with missing critical fields using `df[df['email'].isna()]`.
- **Feature engineering:** Create binary indicator columns for missingness (e.g., `df['age_missing'] = df['age'].isna()`).

### References

- pandas.DataFrame.isna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isna.html
- pandas.DataFrame.notna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.notna.html
- Missing data (Working with missing data) — https://pandas.pydata.org/docs/user_guide/missing_data.html


## 2. Summary Metrics: Counting and Percentages of Missing Values

### Definitions

**Core Definition:** Summary metrics for missing data quantify the extent of missingness by counting missing values and calculating their percentage relative to the total number of observations per column.

**Technical Definition:** Chaining `df.isna().sum()` returns the count of missing values per column. Dividing this count by `len(df)` and multiplying by 100 yields the percentage of missing values. These metrics can be assembled into a summary DataFrame for systematic reporting.

**Beginner-Friendly Explanation:** After you know where the missing values are, the next question is “How many are there?” Counting them tells you the scale of the problem; percentages tell you which columns are most affected.

### Purposes

- To count the total number of missing values in each column.
- To calculate the percentage of missing values per column for prioritization.
- To build a summary report of missingness across features.
- To identify columns with excessive missingness that may require dropping or imputation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Count missing values per column
df.isna().sum()

# Total missing values in the DataFrame
df.isna().sum().sum()

# Percentage of missing values per column
(df.isna().sum() / len(df) * 100).round(2)

# Summary DataFrame
pd.DataFrame({
    'missing_count': df.isna().sum(),
    'missing_percent': (df.isna().sum() / len(df) * 100).round(2),
    'dtype': df.dtypes
})
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `df.isna()` | Boolean mask of missing values |
| `.sum()` | Sums `True` values (1) per column (axis=0 default) |
| `.sum().sum()` | Total missing values across the entire DataFrame |
| `/ len(df)` | Divides by number of rows |
| `* 100` | Converts to percentage |
| `.round(2)` | Rounds to 2 decimal places |

**Syntax Rules:**

- `sum()` with no axis argument sums along `axis=0` (down rows), giving per-column counts.
- `sum(axis=1)` sums across columns, giving per-row counts.
- Percentage is calculated as `(missing_count / total_rows) * 100`.
- Using `isna().mean()` directly gives the proportion (0 to 1) without multiplying by 100.

**Constraints and Limitations:**

- Percentage calculations assume the DataFrame length is the total number of observations; if the DataFrame contains duplicate indices, percentages may be misleading.
- Empty strings are not counted as missing; additional cleaning is required.

### Annotated Code Examples

**Example 1: Counting Missing Values**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['Alice', 'Bob', None, 'Diana'],
    'age': [25, np.nan, 35, 28],
    'salary': [50000, 60000, np.nan, np.nan],
    'department': ['Eng', '', 'Eng', 'Sales']
})

# Count missing values per column
print(df.isna().sum())
# Expected output:
# name          1
# age           1
# salary        2
# department    0
# dtype: int64

# Total missing values
print(df.isna().sum().sum())
# Expected output: 4
```

**Why this output:** The `name` column has 1 missing value (`None`), `age` has 1 (`np.nan`), `salary` has 2, and `department` has 0 (the empty string is not missing). The total is 4.

**Example 2: Percentage of Missing Values**

```python
# Percentage of missing values per column
missing_pct = (df.isna().sum() / len(df) * 100).round(2)
print(missing_pct)
# Expected output:
# name          25.0
# age           25.0
# salary        50.0
# department     0.0
# dtype: float64
```

**Why this output:** With 4 rows, 1 missing value equals 25% and 2 missing values equal 50%. The `department` column has 0% missing because empty strings are not counted.

**Example 3: Comprehensive Missing Value Report**

```python
# Build a summary DataFrame
report = pd.DataFrame({
    'missing_count': df.isna().sum(),
    'missing_percent': (df.isna().sum() / len(df) * 100).round(2),
    'dtype': df.dtypes
})
print(report)
# Expected output:
#             missing_count  missing_percent     dtype
# name                    1             25.0    object
# age                     1             25.0   float64
# salary                  2             50.0   float64
# department              0              0.0    object
```

**Why this output:** The summary DataFrame consolidates count, percentage, and dtype into a single view. This is a standard first step in exploratory data analysis.

### Real-World Cases

- **Data quality dashboards:** Display missing-value percentages per column to monitor data pipeline health.
- **Feature selection:** Drop columns with more than 50% missing values before model training.
- **Imputation prioritization:** Focus imputation efforts on columns with moderate missingness (e.g., 5–30%).

### References

- pandas.DataFrame.isna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isna.html
- Missing data — https://pandas.pydata.org/docs/user_guide/missing_data.html


## 3. Pattern Diagnostics: Visualizing Missingness

### Definitions

**Core Definition:** Pattern diagnostics use visualizations to reveal the structure of missingness in a dataset, including whether missing values are randomly distributed or clustered in specific rows, columns, or combinations.

**Technical Definition:** The `missingno` library provides four primary visualizations: `matrix()` (nullity matrix showing missingness patterns across rows), `bar()` (bar chart of non-null counts per column), `heatmap()` (nullity correlation between columns), and `dendrogram()` (hierarchical clustering of missingness correlations). These tools help identify whether missingness is MCAR (Missing Completely at Random), MAR (Missing at Random), or MNAR (Missing Not at Random).

**Beginner-Friendly Explanation:** Visualizations make it easy to see if missing values happen randomly or follow patterns—like certain rows always missing certain columns. This helps you decide how to handle them.

### Purposes

- To visually identify whether missingness is randomly distributed or clustered.
- To detect correlations between missingness in different columns.
- To determine whether missingness patterns are systematic (e.g., all rows with missing blood pressure also have missing cholesterol).
- To inform the choice of missing data handling strategy (deletion, imputation, or modeling).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import missingno as msno

msno.matrix(df)        # Nullity matrix
msno.bar(df)           # Bar chart of non-null counts
msno.heatmap(df)       # Nullity correlation heatmap
msno.dendrogram(df)    # Hierarchical clustering dendrogram
```

**Component Breakdown:**

| Visualization | Description |
|---------------|-------------|
| `matrix()` | Data-dense display showing missing (white) vs. present (grey) values; sparkline on right summarizes row completeness |
| `bar()` | Bar chart of non-null counts per column; absolute counts on top, percentage on y-axis |
| `heatmap()` | Nullity correlation matrix; values range from −1 to 1 |
| `dendrogram()` | Hierarchical clustering; variables linked at distance 0 have perfectly correlated missingness |

**Syntax Rules:**

- `missingno` requires `matplotlib` and `scipy` for dendrogram.
- Install with `pip install missingno`.
- All functions accept a pandas DataFrame as input.
- Visualizations are displayed via `matplotlib`; use `plt.show()` if needed.

**Constraints and Limitations:**

- `missingno` is a third-party library, not part of pandas core.
- Visualizations are most effective with larger datasets; small datasets may not reveal clear patterns.
- Heatmap correlation is pairwise; higher-order interactions require the dendrogram or custom analysis.

### Annotated Code Examples

**Example 1: Nullity Matrix**

```python
import pandas as pd
import numpy as np
import missingno as msno
import matplotlib.pyplot as plt

# Create a DataFrame with a pattern of missingness
np.random.seed(42)
df = pd.DataFrame({
    'A': [1, 2, np.nan, 4, 5, np.nan, 7, 8, 9, 10],
    'B': [np.nan, 2, 3, 4, np.nan, 6, 7, 8, np.nan, 10],
    'C': [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    'D': [np.nan, np.nan, 3, 4, 5, 6, 7, np.nan, 9, 10]
})

# Generate nullity matrix
msno.matrix(df)
plt.show()
```

**Expected Output:** A matrix plot where white lines represent missing values and grey represents present values. A sparkline on the right summarizes row completeness. The visualization reveals that row 0 has A missing, row 4 has B missing, and rows 0 and 7 have D missing.

**Why this output:** The nullity matrix displays the entire dataset as a grid, making it easy to spot patterns. If missing values cluster in certain rows or columns, they appear as consistent white bands.

**Example 2: Nullity Bar Chart**

```python
# Bar chart of non-null counts
msno.bar(df)
plt.show()
```

**Expected Output:** A bar chart showing the number of non-null values per column (A, B, C, D) with the absolute count on top of each bar and the percentage of non-null values on the left y-axis. Column C has 100% completeness; columns A and B have 80%; column D has 70%.

**Why this output:** The bar chart is a simplification of the matrix, providing a quick per-column summary of data completeness.

**Example 3: Nullity Correlation Heatmap**

```python
# Heatmap of nullity correlations
msno.heatmap(df)
plt.show()
```

**Expected Output:** A heatmap showing correlations between missingness in columns. Values close to 1 indicate that when one column is missing, the other is also likely missing; values close to −1 indicate that when one is missing, the other is likely present. Values near 0 indicate no relationship.

**Why this output:** The heatmap measures nullity correlation using the `.corr()` function. Strong correlations suggest a common cause of missingness, which may inform imputation strategies (e.g., imputing one column from another).

**Example 4: Dendrogram**

```python
# Hierarchical clustering dendrogram
msno.dendrogram(df)
plt.show()
```

**Expected Output:** A dendrogram showing hierarchical clustering of missingness patterns. Variables linked at distance 0 have perfectly correlated missingness (e.g., if A and D are always missing together, they cluster at 0).

**Why this output:** The dendrogram groups variables by the similarity of their missingness patterns. Variables that branch close to zero predict each other's missingness well.

### Real-World Cases

- **Clinical data:** Missing blood pressure readings are often correlated with missing cholesterol readings, revealing a systematic data collection issue.
- **Survey data:** Missing responses to sensitive questions may cluster together, suggesting respondent discomfort.
- **Sensor data:** Missing sensor readings may correlate with specific time periods (e.g., maintenance windows).

### References

- Visualising Missingness — The Turing Way — https://book.the-turing-way.org/project-design/missing-data/missing-data-visualising-missingness/
- missingno: Missing data visualization module for Python — https://github.com/ResidentMario/missingno


## 4. Structural Profiling: Column-wise vs. Row-wise Analysis

### Definitions

**Core Definition:** Structural profiling analyzes missing values along different axes: column-wise (`axis=0`) to assess feature completeness, and row-wise (`axis=1`) to assess record integrity.

**Technical Definition:** In pandas, `df.isna().sum(axis=0)` returns the count of missing values per column (default). `df.isna().sum(axis=1)` returns the count of missing values per row. `df.dropna(axis=0)` drops rows with missing values; `df.dropna(axis=1)` drops columns. The `how` parameter (`'any'` or `'all'`) and `thresh` parameter control the strictness.

**Beginner-Friendly Explanation:** You can look at missing data from two directions: down columns (which features have the most missing values?) or across rows (which records are incomplete?). Each view answers a different question.

### Purposes

- To assess which columns (features) have the most missing values.
- To assess which rows (records) have the most missing values.
- To drop rows or columns with missing values based on thresholds.
- To filter rows that meet minimum completeness criteria.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Column-wise missing counts (default)
df.isna().sum(axis=0)

# Row-wise missing counts
df.isna().sum(axis=1)

# Drop rows with any missing value
df.dropna(axis=0, how='any')

# Drop rows where all values are missing
df.dropna(axis=0, how='all')

# Drop rows with fewer than n non-null values
df.dropna(axis=0, thresh=n)

# Drop columns with any missing value
df.dropna(axis=1, how='any')

# Filter rows with no missing values
df[df.notna().all(axis=1)]

# Filter rows with at least one missing value
df[df.isna().any(axis=1)]
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `axis=0` | Operate on columns (drop rows) |
| `axis=1` | Operate on rows (drop columns) |
| `how='any'` | Drop if any value is missing (default) |
| `how='all'` | Drop only if all values are missing |
| `thresh=n` | Keep rows/columns with at least `n` non-null values |

**Syntax Rules:**

- `dropna()` with `axis=0` (default) drops rows; `axis=1` drops columns.
- `how='any'` is the default and most aggressive.
- `thresh` specifies the minimum number of non-null values required to keep the row/column.
- `subset` allows dropping based on specific columns only.

**Constraints and Limitations:**

- Dropping rows with any missing value can result in significant data loss.
- `dropna()` on a column with all missing values drops the entire column.
- In-place dropping (`inplace=True`) is deprecated; prefer reassignment.

### Annotated Code Examples

**Example 1: Column-wise vs. Row-wise Missing Counts**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'A': [1, np.nan, 3, np.nan, 5],
    'B': [np.nan, 2, np.nan, 4, 5],
    'C': [1, 2, 3, 4, 5],
    'D': [np.nan, np.nan, np.nan, np.nan, 5]
})

# Column-wise missing counts
print("Column-wise missing counts:")
print(df.isna().sum(axis=0))
# Expected output:
# A    2
# B    2
# C    0
# D    4
# dtype: int64

# Row-wise missing counts
print("\nRow-wise missing counts:")
print(df.isna().sum(axis=1))
# Expected output:
# 0    2
# 1    2
# 2    2
# 3    2
# 4    0
# dtype: int64
```

**Why this output:** Column-wise counts show that column D has 4 missing values (the most), while column C has none. Row-wise counts show that row 4 is complete (0 missing) and all other rows have exactly 2 missing values each.

**Example 2: Dropping Rows Based on Missingness**

```python
# Drop rows with any missing value
print("Drop rows with any missing:")
print(df.dropna(axis=0, how='any'))
# Expected output:
#      A    B  C    D
# 4  5.0  5.0  5  5.0

# Drop rows where all values are missing
df_all_na = pd.DataFrame({
    'A': [1, np.nan, np.nan],
    'B': [2, np.nan, np.nan]
})
print("\nDrop rows where all missing:")
print(df_all_na.dropna(axis=0, how='all'))
# Expected output:
#      A    B
# 0  1.0  2.0

# Keep rows with at least 2 non-null values
print("\nKeep rows with at least 2 non-null:")
print(df.dropna(axis=0, thresh=2))
# Expected output:
#      A    B  C    D
# 0  1.0  NaN  1  NaN
# 1  NaN  2.0  2  NaN
# 2  3.0  NaN  3  NaN
# 3  NaN  4.0  4  NaN
# 4  5.0  5.0  5  5.0
```

**Why this output:** `how='any'` drops all rows with at least one missing value, leaving only row 4. `how='all'` drops only rows where every value is missing. `thresh=2` keeps rows with at least 2 non-null values, which all rows satisfy (each row has at least 2 non-null values).

**Example 3: Filtering Rows by Completeness**

```python
# Filter rows with no missing values
complete_rows = df[df.notna().all(axis=1)]
print("Complete rows:")
print(complete_rows)
# Expected output:
#      A    B  C    D
# 4  5.0  5.0  5  5.0

# Filter rows with at least one missing value
incomplete_rows = df[df.isna().any(axis=1)]
print("\nIncomplete rows:")
print(incomplete_rows)
# Expected output:
#      A    B  C    D
# 0  1.0  NaN  1  NaN
# 1  NaN  2.0  2  NaN
# 2  3.0  NaN  3  NaN
# 3  NaN  4.0  4  NaN
```

**Why this output:** `df.notna().all(axis=1)` returns `True` for rows where every value is non-missing. `df.isna().any(axis=1)` returns `True` for rows with at least one missing value.

### Real-World Cases

- **Customer records:** Drop rows where critical fields (e.g., customer ID, email) are missing using `dropna(subset=['customer_id', 'email'])`.
- **Sensor data:** Drop columns with more than 50% missing values using `dropna(axis=1, thresh=len(df) * 0.5)`.
- **Survey analysis:** Filter to complete responses only using `df[df.notna().all(axis=1)]` for per-protocol analysis.

### References

- pandas.DataFrame.dropna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.dropna.html
- pandas.DataFrame.isna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isna.html


## 5. Implicit vs. Explicit Missingness

### Definitions

**Core Definition:** Explicit missingness is represented by pandas' NA sentinel values (`NaN`, `None`, `NaT`, `pd.NA`), while implicit missingness refers to values that are absent from the data structure entirely (e.g., missing rows in a time series) or represented as empty/whitespace strings.

**Technical Definition:** Explicit missing values are detected by `isna()`. Implicit missing values—such as empty strings (`""`), whitespace-only strings (`" "`), or placeholder values like `"N/A"`, `"null"`, `"-"`—are not detected by `isna()` and must be converted to `NaN` explicitly using `replace()`, `str.strip()`, or the `na_values` parameter in `read_csv()`. Structural missingness refers to entire rows or index labels absent from the DataFrame.

**Beginner-Friendly Explanation:** Some missing values are clearly marked as missing (like `NaN`). Others hide in plain sight—empty strings, spaces, or “N/A” text. pandas won't automatically know these are missing, so you have to convert them.

### Purposes

- To distinguish between genuinely missing data and placeholder values that look like data.
- To convert placeholder strings (e.g., `"N/A"`, `" "`, `"-"`) into proper NA values.
- To detect structural missingness (missing rows/index labels) using reindexing.
- To ensure consistent missing value representation before analysis.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Replace empty strings with NaN
df.replace('', np.nan)
df.replace(r'^\s*$', np.nan, regex=True)

# Replace multiple placeholder values
df.replace(['', 'N/A', 'null', 'None', '-'], np.nan)

# Strip whitespace and replace empty results
df['col'] = df['col'].str.strip().replace('', np.nan)

# Detect whitespace strings
df['col'].str.strip() == ''

# na_values during read_csv
pd.read_csv('file.csv', na_values=['', 'N/A', 'null', 'None', '-'])

# Detect structural missingness (missing index labels)
df.reindex(range(df.index.min(), df.index.max() + 1))
```

**Component Breakdown:**

| Approach | Description |
|----------|-------------|
| `replace('', np.nan)` | Replaces empty strings with NaN |
| `replace(r'^\s*$', np.nan, regex=True)` | Replaces whitespace-only strings with NaN |
| `.str.strip()` | Removes leading/trailing whitespace |
| `na_values` in `read_csv` | Treats specified strings as NA during import |
| `reindex()` | Makes implicit missing rows explicit |

**Syntax Rules:**

- Empty strings (`""`) are valid strings in pandas and are not detected by `isna()`.
- Whitespace strings (`" "`) require regex or `str.strip()` to detect.
- `na_values` accepts a list of strings to treat as NA during file reading.
- `reindex()` is used to make structurally missing rows explicit.

**Constraints and Limitations:**

- Replacing empty strings with `NaN` requires the column to be `object` dtype; after replacement, the column may be upcast.
- `str.strip()` on non-string columns raises `AttributeError`.
- Placeholder values vary by dataset; no universal list covers all cases.

### Annotated Code Examples

**Example 1: Converting Empty Strings to NaN**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['Alice', 'Bob', '', 'Diana'],
    'city': ['NYC', ' ', 'LA', 'SF'],
    'notes': ['', 'N/A', 'null', '-']
})

print("Before cleaning:")
print(df.isna().sum())
# Expected output:
# name     0
# city     0
# notes    0
# dtype: int64

# Replace empty strings and whitespace with NaN
df_clean = df.replace(r'^\s*$', np.nan, regex=True)

print("\nAfter cleaning whitespace:")
print(df_clean.isna().sum())
# Expected output:
# name     1
# city     1
# notes    0
# dtype: int64

# Replace multiple placeholder values
df_clean = df_clean.replace(['N/A', 'null', '-'], np.nan)
print("\nAfter cleaning placeholders:")
print(df_clean.isna().sum())
# Expected output:
# name     1
# city     1
# notes    3
# dtype: int64
```

**Why this output:** Initially, `isna()` detects 0 missing values because empty strings, spaces, and placeholder strings are valid strings. After regex replacement, empty strings and whitespace become `NaN`, increasing counts. After replacing placeholder values, `notes` has 3 missing values.

**Example 2: Using na_values During Import**

```python
# Simulate reading a CSV with placeholder values
from io import StringIO

csv_data = """name,age,city
Alice,25,NYC
Bob,N/A,
Charlie,,LA
Diana,30,-
"""

df = pd.read_csv(StringIO(csv_data), na_values=['', 'N/A', '-'])
print(df)
print(df.isna().sum())
# Expected output:
#       name   age city
# 0    Alice  25.0  NYC
# 1      Bob   NaN  NaN
# 2  Charlie   NaN   LA
# 3    Diana  30.0  NaN
# name    0
# age     2
# city    2
# dtype: int64
```

**Why this output:** The `na_values` parameter tells `read_csv` to treat `''`, `'N/A'`, and `'-'` as missing. These values are converted to `NaN` during import, and the `age` column is upcast to `float64` to accommodate the missing values.

**Example 3: Detecting Structural Missingness**

```python
# Create a time series with a missing date
dates = pd.to_datetime(['2024-01-01', '2024-01-02', '2024-01-04'])
s = pd.Series([1, 2, 4], index=dates)
print("Original:")
print(s)
# Expected output:
# 2024-01-01    1
# 2024-01-02    2
# 2024-01-04    4
# dtype: int64

# Reindex to full date range
full_range = pd.date_range('2024-01-01', '2024-01-04')
s_reindexed = s.reindex(full_range)
print("\nAfter reindex:")
print(s_reindexed)
print(f"\nMissing values: {s_reindexed.isna().sum()}")
# Expected output:
# 2024-01-01    1.0
# 2024-01-02    2.0
# 2024-01-03    NaN
# 2024-01-04    4.0
# Freq: D, dtype: float64
# Missing values: 1
```

**Why this output:** The original Series is missing the date `2024-01-03` entirely (structural missingness). `reindex()` makes this implicit missingness explicit by introducing a `NaN` for the missing date, and the dtype is upcast from `int64` to `float64`.

### Real-World Cases

- **CSV imports:** Use `na_values=['', 'N/A', 'null', 'None', '-', ' ']` to handle common placeholder values during import.
- **Survey data:** Empty responses stored as `""` or `" "` are converted to `NaN` before analysis.
- **Time series:** `reindex()` is used to fill in missing dates in financial or sensor data, making gaps explicit for imputation.

### References

- Missing data (Implicit vs. explicit) — https://pandas.pydata.org/docs/user_guide/missing_data.html
- pandas.read_csv (na_values parameter) — https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html
- pandas.DataFrame.replace — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.replace.html
- pandas.Series.str.strip — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.strip.html


## References

- pandas.DataFrame.isna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.isna.html
- pandas.DataFrame.notna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.notna.html
- pandas.DataFrame.dropna — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.dropna.html
- pandas.DataFrame.replace — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.replace.html
- pandas.read_csv (na_values parameter) — https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html
- Missing data — https://pandas.pydata.org/docs/user_guide/missing_data.html
- Visualising Missingness — The Turing Way — https://book.the-turing-way.org/project-design/missing-data/missing-data-visualising-missingness/
- missingno: Missing data visualization module for Python — https://github.com/ResidentMario/missingno
- pandas.Series.str.strip — https://pandas.pydata.org/docs/reference/api/pandas.Series.str.strip.html