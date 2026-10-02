# Pandas Statistical Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Statistical operations in pandas are vectorized computational methods that summarise, aggregate, and analyse the distribution of numerical data within Series and DataFrame objects.

**Technical Definition:** Pandas implements statistical operations as reduction methods (`.mean()`, `.median()`, `.var()`, `.std()`, etc.) that operate along a specified axis, collapsing one dimension of the data and returning a lower-dimensional result. These methods leverage NumPy's statistical functions under the hood and automatically exclude `NaN` values by default (`skipna=True`). Windowed statistics (`.rolling()`, `.expanding()`) return objects that compute statistics over a sliding or growing window of observations. Pandas uses `ddof=1` (Delta Degrees of Freedom) as the default for variance and standard deviation, providing the unbiased sample estimator normalized by N-1.

**Beginner-Friendly Explanation:** Statistical operations let you summarise your data—like finding the average, the middle value, how spread out the values are, or the running total. Instead of calculating these manually, you call methods like `.mean()` or `.std()` on your columns and pandas does the math for you, automatically skipping empty cells.

### Key Characteristics

- **Vectorized execution:** Statistical methods operate on entire columns or rows in a single call at C-level speed.
- **NA-safe by default:** `NaN` values are automatically excluded from calculations (`skipna=True`); the result is `NaN` only if all values are `NaN`.
- **Axis-aware:** All reduction methods accept an `axis` parameter (`0` for column-wise, `1` for row-wise).
- **Sample vs. population statistics:** Variance and standard deviation default to sample statistics (`ddof=1`), normalized by N-1.
- **Windowed computations:** `.rolling()` and `.expanding()` provide a two-step API: first define the window, then call an aggregation method.

### Prerequisites

- Basic Python knowledge (variables, functions, lists).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of descriptive statistics (mean, median, variance, standard deviation).
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Exploratory Data Analysis (EDA):** Computing summary statistics is the first step in understanding any dataset.
- **Feature Engineering:** Creating rolling averages, expanding sums, and volatility measures for ML models.
- **Time Series Analysis:** Moving averages, exponential smoothing, and rolling volatility.
- **Financial Analysis:** Sharpe ratios, drawdowns, and cumulative returns.

### Core Concepts / Features

1. Central Tendency Profiling: `.mean()`, `.median()`, `.mode()`
2. Spread Indicators: `.var()`, `.std()` with `ddof` Adjustment
3. Extreme Boundary Analysis: `.min()`, `.max()`, `.idxmin()`, `.idxmax()`
4. Distribution Splits: `.quantile()` and Percentiles
5. Windowed Statistics: `.rolling()` and `.expanding()`


## 1. Central Tendency Profiling: Calculating Average Dimensions

### Definitions

**Core Definition:** Central tendency measures summarise a dataset with a single value representing the "centre" of the distribution: `.mean()` computes the arithmetic average, `.median()` finds the middle value, and `.mode()` identifies the most frequent value(s).

**Technical Definition:** `DataFrame.mean(axis=0, skipna=True, numeric_only=False)` returns the arithmetic mean of the values over the requested axis. `DataFrame.median(axis=0, skipna=True, numeric_only=False)` returns the median (50th percentile) of the values. `DataFrame.mode(axis=0, numeric_only=False, dropna=True)` returns the mode(s) of each column or row, and can return multiple values if the distribution is multimodal. All methods exclude `NaN` values by default.

**Beginner-Friendly Explanation:** The mean is the average—add up all the numbers and divide by how many there are. The median is the middle number when the data is sorted. The mode is the number that appears most often. If your data has outliers, the median is often more representative than the mean.

### Purposes

- To compute the arithmetic average for a quick summary of typical values.
- To find the middle value for a robust measure of central tendency resistant to outliers.
- To identify the most frequent value(s), especially useful for categorical or discrete data.
- To fill missing values with the mean, median, or mode as a simple imputation strategy.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.mean(axis=0, skipna=True, numeric_only=False)
DataFrame.median(axis=0, skipna=True, numeric_only=False)
DataFrame.mode(axis=0, numeric_only=False, dropna=True)
Series.mean(axis=None, skipna=True, numeric_only=False)
Series.median(axis=None, skipna=True, numeric_only=False)
Series.mode(dropna=True)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `axis` | `0` or `'index'`: column-wise (default); `1` or `'columns'`: row-wise |
| `skipna` | `True` (default): exclude `NaN` values; `False`: include `NaN` |
| `numeric_only` | `False` (default): include all columns; `True`: include only numeric columns |
| `dropna` | `True` (default, mode only): exclude `NaN` before computing the mode |

**Syntax Rules:**

- `skipna=True` (default) excludes `NaN` values; if all values are `NaN`, the result is `NaN`.
- `mode()` returns a DataFrame (not a Series) because it can return multiple values per column.
- `mode()` fills empty slots with `NaN` when some columns have fewer modes than others.
- For Series, the `axis` parameter is unused and defaults to 0.

**Constraints and Limitations:**

- The mean is sensitive to outliers; the median is not.
- `mode()` can return multiple rows if the distribution is multimodal.
- `numeric_only=True` is recommended for mixed-type DataFrames to avoid `TypeError`.

### Annotated Code Examples

**Example 1: Mean, Median, and Mode**

```python
import pandas as pd

df = pd.DataFrame({
    'scores': [85, 92, 78, 92, 88, 70, 92, 85]
})

print(f"Mean:   {df['scores'].mean():.2f}")
print(f"Median: {df['scores'].median():.2f}")
print(f"Mode:   {df['scores'].mode().tolist()}")
# Expected output:
# Mean:   85.25
# Median: 86.50
# Mode:   [92]
```

**Why this output:** The mean is (85+92+78+92+88+70+92+85)/8 = 85.25. The median of the sorted values [70, 78, 85, 85, 88, 92, 92, 92] is (85+88)/2 = 86.50. The mode is 92 (appears three times).

**Example 2: Central Tendency with Missing Values**

```python
import numpy as np

df2 = pd.DataFrame({
    'values': [10, np.nan, 20, 30, np.nan, 40]
})

print(f"Mean (skipna=True):  {df2['values'].mean():.2f}")
print(f"Mean (skipna=False): {df2['values'].mean(skipna=False)}")
print(f"Median:              {df2['values'].median():.2f}")
# Expected output:
# Mean (skipna=True):  25.00
# Mean (skipna=False): nan
# Median:              25.00
```

**Why this output:** With `skipna=True` (default), the two `NaN` values are excluded, and the mean of [10, 20, 30, 40] is 25.00. With `skipna=False`, the presence of `NaN` causes the result to be `NaN`.

**Example 3: Multimodal Distribution**

```python
# Multimodal data: both 1 and 2 appear twice
df3 = pd.DataFrame({
    'A': [1, 1, 2, 2, 3],
    'B': [5, 5, 5, 6, 6]
})

print(df3.mode())
# Expected output:
#    A    B
# 0  1  5.0
# 1  2  NaN
```

**Why this output:** Column A has two modes (1 and 2, each appearing twice), so they occupy rows 0 and 1. Column B has one mode (5, appearing three times), so row 1 is filled with `NaN` for column B.

### Real-World Cases

- **Student grades:** Compute the mean and median exam scores to assess class performance.
- **E-commerce:** Find the most frequent product rating (mode) to understand customer satisfaction.
- **Sensor data:** Use the median to summarise temperature readings in the presence of sensor glitches.

### References

- pandas.DataFrame.mean — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mean.html
- pandas.DataFrame.median — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.median.html
- pandas.DataFrame.mode — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mode.html
- pandas.Series.mode — https://pandas.pydata.org/docs/reference/api/pandas.Series.mode.html
- Summary statistics (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics


## 2. Spread Indicators: Measuring Structural Variance

### Definitions

**Core Definition:** Spread indicators measure how dispersed or variable the data is around the centre: `.var()` computes the variance (average squared deviation from the mean), and `.std()` computes the standard deviation (square root of the variance).

**Technical Definition:** `DataFrame.var(axis=0, skipna=True, ddof=1, numeric_only=False)` returns the unbiased variance over the requested axis. `DataFrame.std(axis=0, skipna=True, ddof=1, numeric_only=False)` returns the sample standard deviation. Both methods normalize by N-1 by default (`ddof=1`), providing the unbiased sample estimator. Setting `ddof=0` normalizes by N, giving the population variance/standard deviation.

**Beginner-Friendly Explanation:** Variance measures how spread out the numbers are. A small variance means all values are close to the mean; a large variance means they are spread far apart. Standard deviation is the square root of variance and is in the same units as the original data, making it easier to interpret.

### Purposes

- To quantify the dispersion of data around the mean.
- To compare the variability of different datasets or columns.
- To compute volatility measures in financial analysis.
- To standardise data (z-score normalization) using mean and standard deviation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.var(axis=0, skipna=True, ddof=1, numeric_only=False)
DataFrame.std(axis=0, skipna=True, ddof=1, numeric_only=False)
Series.var(axis=None, skipna=True, ddof=1, numeric_only=False)
Series.std(axis=None, skipna=True, ddof=1, numeric_only=False)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `axis` | `0` or `'index'`: column-wise (default); `1` or `'columns'`: row-wise |
| `skipna` | `True` (default): exclude `NaN` values |
| `ddof` | Delta Degrees of Freedom; divisor is N − ddof; default `1` (sample) |
| `numeric_only` | `False` (default): include all columns; `True`: numeric only |

**Syntax Rules:**

- `ddof=1` (default) gives the sample variance/std; `ddof=0` gives the population variance/std.
- The divisor used in calculations is N − ddof, where N is the number of non-NA elements.
- NumPy's `np.var()` and `np.std()` default to `ddof=0`, so results may differ from pandas unless `ddof` is set explicitly.

**Constraints and Limitations:**

- Variance and standard deviation require at least two non-NA observations; otherwise, the result is `NaN`.
- The `axis=None` behaviour for `Series.var()` is deprecated; pass `axis=0` to retain the old behaviour.
- Standard deviation is more interpretable than variance because it is in the same units as the data.

### Annotated Code Examples

**Example 1: Variance and Standard Deviation**

```python
import pandas as pd

df = pd.DataFrame({
    'A': [10, 20, 30, 40, 50],
    'B': [5, 5, 5, 5, 5]
})

print(f"A mean: {df['A'].mean():.1f}, A std: {df['A'].std():.2f}")
print(f"B mean: {df['B'].mean():.1f}, B std: {df['B'].std():.2f}")
# Expected output:
# A mean: 30.0, A std: 15.81
# B mean: 5.0, B std: 0.00
```

**Why this output:** Column A has values spread from 10 to 50, so its standard deviation is 15.81. Column B has all identical values, so its standard deviation is 0.00 (no variability).

**Example 2: ddof Comparison (Sample vs. Population)**

```python
# Compare ddof=1 (sample) with ddof=0 (population)
df2 = pd.DataFrame({'values': [2, 4, 4, 4, 5, 5, 7, 9]})

print(f"Sample std (ddof=1):     {df2['values'].std(ddof=1):.4f}")
print(f"Population std (ddof=0): {df2['values'].std(ddof=0):.4f}")
print(f"Sample var (ddof=1):     {df2['values'].var(ddof=1):.4f}")
print(f"Population var (ddof=0): {df2['values'].var(ddof=0):.4f}")
# Expected output:
# Sample std (ddof=1):     2.1381
# Population std (ddof=0): 2.0000
# Sample var (ddof=1):     4.5714
# Population var (ddof=0): 4.0000
```

**Why this output:** The sample standard deviation (ddof=1) divides by N−1 = 7, while the population standard deviation (ddof=0) divides by N = 8. The sample values are larger because dividing by a smaller number increases the result.

**Example 3: Comparing Spread Across Columns**

```python
df3 = pd.DataFrame({
    'low_variance': [100, 101, 99, 100, 101],
    'high_variance': [50, 150, 80, 120, 100]
})

print(df3.std())
# Expected output:
# low_variance      0.894427
# high_variance    39.051248
# dtype: float64
```

**Why this output:** The `low_variance` column has values tightly clustered around 100, while `high_variance` has values spread from 50 to 150. The standard deviation quantifies this difference: 0.89 vs. 39.05.

### Real-World Cases

- **Financial analysis:** Compute the standard deviation of daily returns as a measure of volatility (risk).
- **Quality control:** Monitor the standard deviation of product dimensions to detect manufacturing variability.
- **A/B testing:** Compare the variance of conversion rates between control and treatment groups.

### References

- pandas.DataFrame.var — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.var.html
- pandas.DataFrame.std — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.std.html
- pandas.Series.var — https://pandas.pydata.org/docs/reference/api/pandas.Series.var.html
- pandas.Series.std — https://pandas.pydata.org/docs/reference/api/pandas.Series.std.html
- Summary statistics (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics


## 3. Extreme Boundary Analysis: Spotting Absolute Data Points

### Definitions

**Core Definition:** Extreme boundary analysis identifies the minimum and maximum values in a dataset and their corresponding index labels: `.min()` and `.max()` return the extreme values, while `.idxmin()` and `.idxmax()` return the index labels of their first occurrence.

**Technical Definition:** `DataFrame.min(axis=0, skipna=True, numeric_only=False)` returns the minimum value over the requested axis. `DataFrame.max()` returns the maximum. `DataFrame.idxmin(axis=0, skipna=True)` returns the index of the first occurrence of the minimum value; `DataFrame.idxmax()` returns the index of the first occurrence of the maximum. NA/null values are excluded by default. For a DataFrame, the result is a Series indexed by column name.

**Beginner-Friendly Explanation:** `.min()` and `.max()` tell you the smallest and largest values. `.idxmin()` and `.idxmax()` tell you where those values are—which row label holds the minimum or maximum. This is useful for finding the best or worst performing item.

### Purposes

- To identify the smallest and largest values in each column.
- To find the index label (e.g., date, product name) associated with extreme values.
- To detect outliers and boundary violations in data validation.
- To locate the best or worst performing entity in a dataset.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.min(axis=0, skipna=True, numeric_only=False)
DataFrame.max(axis=0, skipna=True, numeric_only=False)
DataFrame.idxmin(axis=0, skipna=True)
DataFrame.idxmax(axis=0, skipna=True)
Series.min(axis=None, skipna=True, numeric_only=False)
Series.max(axis=None, skipna=True, numeric_only=False)
Series.idxmin(axis=None, skipna=True)
Series.idxmax(axis=None, skipna=True)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `axis` | `0` or `'index'`: column-wise (default); `1` or `'columns'`: row-wise |
| `skipna` | `True` (default): exclude `NaN` values; if an entire row/column is `NA`, the result is `NA` |
| `numeric_only` | `False` (default): include all columns; `True`: numeric only |

**Syntax Rules:**

- `idxmin()` and `idxmax()` return the index **label**, not the integer position.
- If multiple rows share the minimum/maximum value, the index of the **first** occurrence is returned.
- If an entire row or column is `NA`, the result is `NA`.
- For DataFrames, `idxmin()`/`idxmax()` return a Series indexed by column name.

**Constraints and Limitations:**

- `idxmin()`/`idxmax()` are slower than `min()`/`max()` because they must track the index.
- For very large DataFrames, `idxmin()`/`idxmax()` may be memory-intensive.
- If the index contains duplicate labels, the returned label may be ambiguous.

### Annotated Code Examples

**Example 1: Min, Max, and Their Index Labels**

```python
import pandas as pd

df = pd.DataFrame({
    'month': ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
    'sales': [100, 150, 80, 200, 120]
})

print(f"Min sales:  {df['sales'].min()}")
print(f"Max sales:  {df['sales'].max()}")
print(f"Month of min: {df['sales'].idxmin()}")
print(f"Month of max: {df['sales'].idxmax()}")
# Expected output:
# Min sales:  80
# Max sales:  200
# Month of min: 2
# Month of max: 3
```

**Why this output:** The minimum sales value is 80 (at index 2, “Mar”), and the maximum is 200 (at index 3, “Apr”). `idxmin()` and `idxmax()` return the integer index labels.

**Example 2: Using a Custom Index for Meaningful Labels**

```python
# Set month as index for meaningful labels
df_indexed = df.set_index('month')
print(f"Best month:  {df_indexed['sales'].idxmax()}")
print(f"Worst month: {df_indexed['sales'].idxmin()}")
# Expected output:
# Best month:  Apr
# Worst month: Mar
```

**Why this output:** After setting `'month'` as the index, `idxmax()` returns the index label `'Apr'` instead of the integer position, making the result more interpretable.

**Example 3: Column-Wise Extremes**

```python
df2 = pd.DataFrame({
    'A': [10, 5, 8],
    'B': [3, 12, 7],
    'C': [9, 4, 11]
})

print(df2.idxmax())
# Expected output:
# A    0
# B    1
# C    2
# dtype: int64
```

**Why this output:** For column A, the maximum is 10 at index 0. For column B, the maximum is 12 at index 1. For column C, the maximum is 11 at index 2. The result is a Series indexed by column name.

### Real-World Cases

- **Stock market analysis:** Find the date of the highest and lowest stock prices using `idxmax()` and `idxmin()`.
- **Sales performance:** Identify the top-selling and worst-selling products in a catalogue.
- **Quality control:** Detect the maximum deviation from target specifications.

### References

- pandas.DataFrame.min — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.min.html
- pandas.DataFrame.max — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.max.html
- pandas.DataFrame.idxmin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.idxmin.html
- pandas.DataFrame.idxmax — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.idxmax.html
- pandas.Series.idxmin — https://pandas.pydata.org/docs/reference/api/pandas.Series.idxmin.html
- pandas.Series.idxmax — https://pandas.pydata.org/docs/reference/api/pandas.Series.idxmax.html


## 4. Distribution Splits: Parsing Distribution Spreads via Quantiles

### Definitions

**Core Definition:** Quantiles divide a dataset into equal-sized groups. `.quantile(q)` returns the value below which a given proportion `q` of the data falls. The 0.5 quantile is the median; 0.25 and 0.75 are the first and third quartiles.

**Technical Definition:** `DataFrame.quantile(q=0.5, axis=0, numeric_only=False, interpolation='linear')` returns values at the given quantile(s) over the requested axis. The `q` parameter accepts a float or array-like of floats between 0 and 1. If `q` is an array, the result is a DataFrame indexed by `q`, with columns matching the original DataFrame. The `interpolation` parameter controls how the quantile is computed when it falls between two data points; `'linear'` (default) performs linear interpolation.

**Beginner-Friendly Explanation:** Quantiles split your data into chunks. The median (0.5 quantile) splits it in half. Quartiles (0.25, 0.5, 0.75) split it into four equal groups. Percentiles are quantiles multiplied by 100—the 25th percentile is the same as the 0.25 quantile. Quantiles help you understand where a value stands relative to the rest of the data.

### Purposes

- To understand the distribution of data beyond just the mean and median.
- To identify outliers using the interquartile range (IQR = Q3 − Q1).
- To compute percentiles for ranking and benchmarking.
- To create equal-sized bins for discretisation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.quantile(q=0.5, axis=0, numeric_only=False, interpolation='linear')
Series.quantile(q=0.5, interpolation='linear')
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `q` | Float or array-like between 0 and 1; default 0.5 (median) |
| `axis` | `0` or `'index'`: column-wise (default); `1` or `'columns'`: row-wise |
| `numeric_only` | `False` (default): include all columns; `True`: numeric only |
| `interpolation` | `'linear'` (default), `'lower'`, `'higher'`, `'midpoint'`, `'nearest'` |

**Syntax Rules:**

- `q` must be between 0 and 1. Unlike NumPy's `percentile()`, which uses 0–100, pandas uses 0–1.
- If `q` is an array, a DataFrame is returned where the index is `q` and the columns are the original columns.
- The `interpolation` parameter determines the value when the quantile falls between two data points.
- `DataFrame.describe()` uses `interpolation='linear'` and only supports linear interpolation.

**Constraints and Limitations:**

- Different interpolation methods can produce different quantile values for the same data.
- For small datasets, quantiles may not be meaningful.
- `numeric_only=True` may be needed for mixed-type DataFrames.

### Annotated Code Examples

**Example 1: Basic Quantile Calculation**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({'values': [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]})

# Single quantile (median)
print(f"Median (q=0.5): {df['values'].quantile(0.5)}")

# Multiple quantiles
print(df['values'].quantile([0.25, 0.5, 0.75]))
# Expected output:
# Median (q=0.5): 55.0
# 0.25    32.5
# 0.50    55.0
# 0.75    77.5
# Name: values, dtype: float64
```

**Why this output:** The median of 10 evenly spaced values from 10 to 100 is 55.0. The first quartile (0.25) is 32.5, and the third quartile (0.75) is 77.5.

**Example 2: Quantiles with Different Interpolation Methods**

```python
# Compare interpolation methods
s = pd.Series([1, 2, 3, 4])
print("Linear (default):", s.quantile(0.25, interpolation='linear'))
print("Lower:           ", s.quantile(0.25, interpolation='lower'))
print("Higher:          ", s.quantile(0.25, interpolation='higher'))
print("Midpoint:        ", s.quantile(0.25, interpolation='midpoint'))
print("Nearest:         ", s.quantile(0.25, interpolation='nearest'))
# Expected output:
# Linear (default): 1.75
# Lower:            1
# Higher:           2
# Midpoint:         1.5
# Nearest:          2
```

**Why this output:** With four values [1, 2, 3, 4], the 0.25 quantile falls between 1 and 2. The interpolation method determines the exact value returned. `'linear'` returns 1.75, `'lower'` returns 1, `'higher'` returns 2, `'midpoint'` returns 1.5, and `'nearest'` returns 2.

**Example 3: Quantiles Across a DataFrame**

```python
df2 = pd.DataFrame({
    'A': [1, 2, 3, 4, 5],
    'B': [10, 20, 30, 40, 50]
})

print(df2.quantile([0.25, 0.5, 0.75]))
# Expected output:
#         A     B
# 0.25  2.0  20.0
# 0.50  3.0  30.0
# 0.75  4.0  40.0
```

**Why this output:** When `q` is an array, the result is a DataFrame with one row per quantile and one column per original column. Column A and column B have the same relative distribution, so their quantiles are proportional.

### Real-World Cases

- **Income analysis:** Compute income percentiles to understand economic inequality.
- **Performance benchmarking:** Determine the 90th percentile response time to assess system performance.
- **Outlier detection:** Use the IQR (Q3 − Q1) to identify values outside [Q1 − 1.5×IQR, Q3 + 1.5×IQR].

### References

- pandas.DataFrame.quantile — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.quantile.html
- pandas.Series.quantile — https://pandas.pydata.org/docs/reference/api/pandas.Series.quantile.html
- numpy.percentile — https://numpy.org/doc/stable/reference/generated/numpy.percentile.html
- pandas.DataFrame.describe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html


## 5. Windowed Statistics: Generating Moving Trends and Historical Windows

### Definitions

**Core Definition:** Windowed statistics compute aggregate measures over a sliding (rolling) or growing (expanding) window of observations, enabling trend analysis and smoothing of time-series data.

**Technical Definition:** `DataFrame.rolling(window, min_periods=None, center=False, win_type=None, on=None, closed=None)` returns a `Rolling` object that provides rolling window calculations. `DataFrame.expanding(min_periods=1, center=None, axis=0)` returns an `Expanding` object that computes cumulative statistics from the start of the series. Both objects expose aggregation methods (`.sum()`, `.mean()`, `.std()`, etc.) that return a Series or DataFrame of the same shape, with the first `window-1` (or `min_periods-1`) values set to `NaN`.

**Beginner-Friendly Explanation:** A rolling window computes a statistic (like the average) over a fixed number of recent observations. For example, a 7-day rolling average of daily temperatures gives you the average of the last 7 days at each point. An expanding window computes a statistic over all observations from the start up to the current point—like a running total that grows.

### Purposes

- To smooth noisy time-series data for trend identification.
- To compute moving averages for technical analysis in finance.
- To calculate rolling volatility or rolling correlation.
- To track running totals, running averages, or cumulative maximums.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Rolling window
DataFrame.rolling(window, min_periods=None, center=False, win_type=None, on=None, closed=None)
Series.rolling(window, min_periods=None, center=False, win_type=None, on=None, closed=None)

# Expanding window
DataFrame.expanding(min_periods=1, center=None, axis=0)
Series.expanding(min_periods=1, center=None, axis=0)

# Aggregation methods (applied to the window object)
.rolling(window).mean()
.rolling(window).sum()
.rolling(window).std()
.expanding().mean()
.expanding().sum()
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `window` | Size of the moving window (int for fixed size, offset for time-based) |
| `min_periods` | Minimum number of observations in window required to have a value; default = window size for rolling, 1 for expanding |
| `center` | `False` (default): label at the right edge; `True`: label at the center |
| `win_type` | Window type for weighted windows (e.g., `'boxcar'`, `'triang'`, `'gaussian'`) |
| `on` | For DataFrames, the column to use as the rolling index instead of the index |
| `closed` | Make the interval closed on `'right'`, `'left'`, `'both'`, or `'neither'` |

**Syntax Rules:**

- The `window` parameter can be an integer (fixed number of observations) or a time offset string (e.g., `'3D'`, `'2H'`) for time-based windows.
- `min_periods` controls the minimum number of non-NaN observations required; if fewer, the result is `NaN`.
- The first `window-1` values are `NaN` when `min_periods` equals `window`.
- `.rolling()` and `.expanding()` return window objects; aggregation methods must be called on them.

**Constraints and Limitations:**

- Rolling windows require at least `min_periods` observations to produce a value.
- The result has the same length as the original Series/DataFrame.
- Time-based windows require a `DatetimeIndex` or the `on` parameter.
- Rolling operations on DataFrames with non-numeric columns may raise `TypeError`.

### Annotated Code Examples

**Example 1: Rolling Mean**

```python
import pandas as pd
import numpy as np

# Daily temperatures over 10 days
s = pd.Series([22, 24, 25, 23, 26, 28, 27, 29, 30, 28],
              index=pd.date_range('2024-01-01', periods=10, freq='D'))

# 3-day rolling mean
rolling_mean = s.rolling(window=3).mean()
print(rolling_mean)
# Expected output:
# 2024-01-01          NaN
# 2024-01-02          NaN
# 2024-01-03    23.666667
# 2024-01-04    24.000000
# 2024-01-05    24.666667
# 2024-01-06    25.666667
# 2024-01-07    27.000000
# 2024-01-08    28.000000
# 2024-01-09    28.666667
# 2024-01-10    29.000000
# Freq: D, dtype: float64
```

**Why this output:** The first two values are `NaN` because there are not enough observations to fill a 3-day window. From the third day onward, the value is the average of the current and previous two days.

**Example 2: Expanding Sum**

```python
# Expanding (cumulative) sum
expanding_sum = s.expanding().sum()
print(expanding_sum)
# Expected output:
# 2024-01-01     22.0
# 2024-01-02     46.0
# 2024-01-03     71.0
# 2024-01-04     94.0
# 2024-01-05    120.0
# 2024-01-06    148.0
# 2024-01-07    175.0
# 2024-01-08    204.0
# 2024-01-09    234.0
# 2024-01-10    262.0
# Freq: D, dtype: float64
```

**Why this output:** The expanding sum includes all observations from the start up to the current point. The first value is 22, the second is 22+24=46, the third is 46+25=71, and so on.

**Example 3: Rolling Standard Deviation**

```python
# 3-day rolling standard deviation
rolling_std = s.rolling(window=3).std()
print(rolling_std)
# Expected output:
# 2024-01-01         NaN
# 2024-01-02         NaN
# 2024-01-03    1.527525
# 2024-01-04    1.000000
# 2024-01-05    1.527525
# 2024-01-06    2.516611
# 2024-01-07    1.000000
# 2024-01-08    1.527525
# 2024-01-09    1.527525
# 2024-01-10    1.000000
# Freq: D, dtype: float64
```

**Why this output:** The rolling standard deviation measures volatility over each 3-day window. For example, the standard deviation of [22, 24, 25] is 1.527525. The first two values are `NaN` because the window is not full.

### Real-World Cases

- **Stock analysis:** Compute the 50-day and 200-day moving averages to identify trends and crossovers.
- **Sensor smoothing:** Apply a rolling mean to reduce noise in sensor readings.
- **Financial risk:** Calculate rolling volatility (standard deviation of returns) over a 30-day window.
- **Running totals:** Use expanding sum to compute cumulative sales or revenue.

### References

- pandas.DataFrame.rolling — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rolling.html
- pandas.DataFrame.expanding — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.expanding.html
- pandas.Series.rolling — https://pandas.pydata.org/docs/reference/api/pandas.Series.rolling.html
- pandas.Series.expanding — https://pandas.pydata.org/docs/reference/api/pandas.Series.expanding.html
- Window functions (User Guide) — https://pandas.pydata.org/docs/user_guide/window.html
- Rolling window (API reference) — https://pandas.pydata.org/docs/reference/window.html


## References

- pandas.DataFrame.mean — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mean.html
- pandas.DataFrame.median — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.median.html
- pandas.DataFrame.mode — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mode.html
- pandas.Series.mode — https://pandas.pydata.org/docs/reference/api/pandas.Series.mode.html
- pandas.DataFrame.var — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.var.html
- pandas.DataFrame.std — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.std.html
- pandas.Series.var — https://pandas.pydata.org/docs/reference/api/pandas.Series.var.html
- pandas.Series.std — https://pandas.pydata.org/docs/reference/api/pandas.Series.std.html
- pandas.DataFrame.min — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.min.html
- pandas.DataFrame.max — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.max.html
- pandas.DataFrame.idxmin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.idxmin.html
- pandas.DataFrame.idxmax — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.idxmax.html
- pandas.Series.idxmin — https://pandas.pydata.org/docs/reference/api/pandas.Series.idxmin.html
- pandas.Series.idxmax — https://pandas.pydata.org/docs/reference/api/pandas.Series.idxmax.html
- pandas.DataFrame.quantile — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.quantile.html
- pandas.Series.quantile — https://pandas.pydata.org/docs/reference/api/pandas.Series.quantile.html
- numpy.percentile — https://numpy.org/doc/stable/reference/generated/numpy.percentile.html
- pandas.DataFrame.describe — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html
- pandas.DataFrame.rolling — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rolling.html
- pandas.DataFrame.expanding — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.expanding.html
- pandas.Series.rolling — https://pandas.pydata.org/docs/reference/api/pandas.Series.rolling.html
- pandas.Series.expanding — https://pandas.pydata.org/docs/reference/api/pandas.Series.expanding.html
- Summary statistics (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics
- Window functions (User Guide) — https://pandas.pydata.org/docs/user_guide/window.html
- Rolling window (API reference) — https://pandas.pydata.org/docs/reference/window.html