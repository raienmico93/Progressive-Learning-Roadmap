# Pandas Numerical Transformations and Preprocessing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Numerical transformations and preprocessing in pandas is the set of techniques used to rescale, discretize, round, clip, and otherwise modify continuous numerical data to prepare it for analysis or machine learning.

**Technical Definition:** Numerical preprocessing encompasses range formatting (Min-Max scaling, Z-score standardization, robust scaling), discretization (`pd.cut()` for equal-width bins and `pd.qcut()` for equal-frequency quantile bins), precision controls (`.round()`, `np.floor()`, `np.ceil()`, and floating-point constraints), outlier clamping (`.clip()` with lower and upper thresholds), and sign conversions (binary masks and multi-interval flags). These operations are vectorized, operating on entire Series or DataFrames at C-level speed, and are typically applied during feature engineering or data cleaning pipelines.

**Beginner-Friendly Explanation:** Raw numerical data often needs to be reshaped before it's useful. For example, features on wildly different scales (age in years vs. income in dollars) can confuse machine learning models. Preprocessing techniques like scaling make them comparable. Discretization turns continuous numbers into categories (like “Low”, “Medium”, “High”). Rounding removes unnecessary decimal places, clipping caps extreme outliers, and sign conversions turn numbers into binary flags or interval indicators.

### Key Characteristics

- **Vectorized execution:** All transformations operate on entire columns at C-level speed.
- **Multiple scaling paradigms:** Min-Max scaling compresses to a fixed range; Z-score standardization centers around mean zero with unit variance; robust scaling uses median and IQR to resist outliers.
- **Two discretization strategies:** `pd.cut()` creates equal-width bins; `pd.qcut()` creates equal-frequency (quantile) bins.
- **Rounding half to even:** Pandas `.round()` uses banker's rounding (round half to even), not the “round half up” taught in schools.
- **Clip is vectorized and element-wise:** `.clip()` supports scalar thresholds or array-like thresholds for element-wise truncation.

### Prerequisites

- Basic Python knowledge (variables, functions, lists).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of descriptive statistics (mean, median, standard deviation, IQR).
- Installation of pandas (`pip install pandas`); NumPy for mathematical functions.

### Related Programming Areas

- **Feature Engineering:** Creating scaled, binned, and clipped features for ML models.
- **Data Cleaning:** Removing the impact of outliers and extreme values.
- **Exploratory Data Analysis (EDA):** Binning continuous variables for visualisation and cross-tabulation.
- **Financial Modeling:** Normalising returns, capping drawdowns, and creating binary signal flags.

### Core Concepts / Features

1. Range Formatting: Min-Max Scaling, Z-Score Normalization, and Robust Scaling
2. Discretization Strategies: `pd.cut()` vs. `pd.qcut()`
3. Precision Controls: Rounding, Floors, Ceilings, and Floating-Point Constraints
4. Outlier Clamping: `.clip()` and Custom Capping Functions
5. Sign Conversions and Transformations: Binary Masks and Multi-Interval Flags


## 1. Range Formatting: Min-Max Scaling, Z-Score Normalization, and Robust Scaling

### Definitions

**Core Definition:** Range formatting transforms numerical features to a common scale: Min-Max scaling compresses values to a fixed range (typically [0, 1]), Z-score normalization centres values around mean zero with unit variance, and robust scaling uses median and interquartile range to resist outliers.

**Technical Definition:** Min-Max scaling is computed as `(x - min) / (max - min)`, mapping values to [0, 1]. Z-score standardization is computed as `(x - mean) / std`, producing a distribution with mean 0 and standard deviation 1. Robust scaling is computed as `(x - median) / IQR`, where IQR is the interquartile range (Q3 − Q1). All three can be implemented with pandas arithmetic operations or via scikit-learn's `MinMaxScaler`, `StandardScaler`, and `RobustScaler`. The choice depends on the data's distribution and the presence of outliers.

**Beginner-Friendly Explanation:** Scaling puts all your numbers on the same ruler. Min-Max scaling squishes everything between 0 and 1. Z-score normalization shifts and stretches the data so the average is 0 and most values fall between -3 and +3. Robust scaling uses the middle 50% of the data (the interquartile range) instead of the full range, so extreme outliers don't distort the scale.

### Purposes

- To bring features with different units and scales to a comparable range.
- To improve the convergence and performance of gradient-based machine learning algorithms.
- To reduce the influence of outliers using robust statistics (median and IQR).
- To satisfy algorithms that assume normally distributed input (Z-score).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Min-Max scaling (pandas arithmetic)
df_scaled = (df - df.min()) / (df.max() - df.min())

# Z-score normalization (pandas arithmetic)
df_standardized = (df - df.mean()) / df.std()

# Robust scaling (pandas arithmetic)
df_robust = (df - df.median()) / (df.quantile(0.75) - df.quantile(0.25))

# Using scikit-learn
from sklearn.preprocessing import MinMaxScaler, StandardScaler, RobustScaler
scaler = MinMaxScaler()
df_scaled = pd.DataFrame(scaler.fit_transform(df), columns=df.columns)
```

**Component Breakdown:**

| Method | Formula | Range | Outlier Sensitivity |
|--------|---------|-------|---------------------|
| Min-Max | `(x - min) / (max - min)` | [0, 1] | High |
| Z-score | `(x - mean) / std` | Unbounded | Moderate |
| Robust | `(x - median) / IQR` | Unbounded | Low |

**Syntax Rules:**

- Min-Max scaling requires `max != min`; otherwise, division by zero occurs.
- Z-score uses sample standard deviation (`ddof=1`) by default in pandas.
- Robust scaling uses `quantile(0.75) - quantile(0.25)` for the IQR.
- Scikit-learn scalers return NumPy arrays; wrap in `pd.DataFrame()` to preserve column names.

**Constraints and Limitations:**

- Min-Max scaling is highly sensitive to outliers because it uses the full range.
- Z-score assumes approximately normal distribution; skewed data may benefit from log transformation first.
- Robust scaling is resistant to outliers but does not produce a fixed range.

### Annotated Code Examples

**Example 1: Min-Max Scaling with Pandas**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'age': [20, 30, 40, 50, 60],
    'income': [30000, 50000, 70000, 90000, 110000]
})

# Min-Max scaling
df_minmax = (df - df.min()) / (df.max() - df.min())
print(df_minmax)
# Expected output:
#    age  income
# 0  0.0     0.0
# 1  0.25    0.25
# 2  0.5     0.5
# 3  0.75    0.75
# 4  1.0     1.0
```

**Why this output:** The age column spans 20–60 with a range of 40. Each value is scaled as `(x - 20) / 40`. The income column spans 30,000–110,000 and is scaled similarly. Both columns now fall in [0, 1].

**Example 2: Z-Score Normalization**

```python
# Z-score normalization
df_zscore = (df - df.mean()) / df.std()
print(df_zscore)
# Expected output:
#         age    income
# 0 -1.264911 -1.264911
# 1 -0.632456 -0.632456
# 2  0.000000  0.000000
# 3  0.632456  0.632456
# 4  1.264911  1.264911
```

**Why this output:** The mean age is 40, and the standard deviation is approximately 15.81. Each value is standardised as `(x - 40) / 15.81`. The result has mean 0 and standard deviation 1.

**Example 3: Robust Scaling**

```python
# Robust scaling
df_robust = (df - df.median()) / (df.quantile(0.75) - df.quantile(0.25))
print(df_robust)
# Expected output:
#    age  income
# 0 -1.0    -1.0
# 1 -0.5    -0.5
# 2  0.0     0.0
# 3  0.5     0.5
# 4  1.0     1.0
```

**Why this output:** The median age is 40 and the IQR is 20 (Q3=50, Q1=30). Each value is scaled as `(x - 40) / 20`. Robust scaling is not affected by extreme values in the tails.

### Real-World Cases

- **Machine learning pipelines:** Scale features before training neural networks or SVMs.
- **Financial data:** Standardise returns before computing correlation matrices.
- **Healthcare:** Normalise patient measurements (blood pressure, cholesterol) for clustering.

### References

- MinMaxScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html
- StandardScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html
- RobustScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html
- Data Normalization and Standardization — https://notebooks.githubusercontent.com/view/ipynb


## 2. Discretization Strategies: pd.cut() vs. pd.qcut()

### Definitions

**Core Definition:** Discretization (binning) converts continuous numerical values into discrete categorical intervals. `pd.cut()` creates bins of equal width (equal value range), while `pd.qcut()` creates bins of equal frequency (equal number of observations per bin).

**Technical Definition:** `pd.cut(x, bins, right=True, labels=None, retbins=False, precision=3, include_lowest=False)` bins values into discrete intervals based on the numerical edges of `bins`. `pd.qcut(x, q, labels=None, retbins=False, precision=3, duplicates='raise')` bins values based on sample quantiles, where `q` is an integer (number of quantiles) or an array of quantiles. `pd.cut()` produces equal-width bins but unequal frequencies; `pd.qcut()` produces unequal-width bins but equal frequencies.

**Beginner-Friendly Explanation:** Discretization turns numbers into categories. `pd.cut()` divides the range from the smallest to the largest value into equal-sized chunks—like slicing a loaf of bread into equal-thickness slices. `pd.qcut()` divides the data so that each category has the same number of observations—like dealing a deck of cards into equal piles, even if some piles have a wider range of card values.

### Purposes

- To convert continuous features into categorical ones for models that require categorical input.
- To reduce the impact of outliers and noisy data by grouping values into bins.
- To create interpretable features like “Low”, “Medium”, “High” for reporting and visualisation.
- To perform equal-frequency binning for quantile-based analysis.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Equal-width binning
pd.cut(x, bins, right=True, labels=None, retbins=False, precision=3, include_lowest=False)

# Equal-frequency binning
pd.qcut(x, q, labels=None, retbins=False, precision=3, duplicates='raise')
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `x` | 1-D array-like (Series) to bin |
| `bins` | Int (number of equal-width bins) or sequence of scalars (bin edges) |
| `q` | Int (number of quantiles) or array of quantiles (e.g., [0, 0.25, 0.5, 0.75, 1]) |
| `right` | `True` (default): bins include the right edge; `False`: include the left edge |
| `labels` | Labels for the resulting bins; `None` returns interval notation |
| `retbins` | `True`: return the bin edges as a second return value |
| `duplicates` | `'raise'` (default) or `'drop'`: how to handle duplicate bin edges in `qcut` |

**Syntax Rules:**

- `pd.cut()` with an integer `bins` creates that many equal-width bins.
- `pd.qcut()` with an integer `q` creates that many equal-frequency bins.
- `right=True` means bins are `(a, b]`; `right=False` means `[a, b)`.
- `labels` can be a list of strings or `False` (returns integer indicators).
- `qcut` may raise an error if duplicate bin edges occur; use `duplicates='drop'` to handle.

**Constraints and Limitations:**

- `pd.cut()` can produce bins with zero observations if the data is clustered.
- `pd.qcut()` can fail if there are too few unique values to create the requested number of quantiles.
- The resulting Categorical can be converted to numeric codes with `.cat.codes`.

### Annotated Code Examples

**Example 1: Equal-Width Binning with pd.cut()**

```python
import pandas as pd

df = pd.DataFrame({
    'age': [22, 25, 30, 35, 40, 45, 50, 55, 60, 65]
})

# 3 equal-width bins
df['age_bin'] = pd.cut(df['age'], bins=3)
print(df)
# Expected output:
#    age         age_bin
# 0   22  (21.956, 36.333]
# 1   25  (21.956, 36.333]
# 2   30  (21.956, 36.333]
# 3   35  (21.956, 36.333]
# 4   40  (36.333, 50.667]
# 5   45  (36.333, 50.667]
# 6   50  (36.333, 50.667]
# 7   55  (50.667, 65.0]
# 8   60  (50.667, 65.0]
# 9   65  (50.667, 65.0]
```

**Why this output:** `pd.cut(bins=3)` divides the range from 22 to 65 (range = 43) into three equal-width bins of approximately 14.33 each. The bins are `(21.956, 36.333]`, `(36.333, 50.667]`, and `(50.667, 65.0]`.

**Example 2: Equal-Frequency Binning with pd.qcut()**

```python
# 3 equal-frequency bins (quantiles)
df['age_qcut'] = pd.qcut(df['age'], q=3)
print(df[['age', 'age_qcut']])
# Expected output:
#    age         age_qcut
# 0   22  (21.999, 31.667]
# 1   25  (21.999, 31.667]
# 2   30  (21.999, 31.667]
# 3   35  (31.667, 45.0]
# 4   40  (31.667, 45.0]
# 5   45  (31.667, 45.0]
# 6   50  (45.0, 65.0]
# 7   55  (45.0, 65.0]
# 8   60  (45.0, 65.0]
# 9   65  (45.0, 65.0]
```

**Why this output:** `pd.qcut(q=3)` divides the data into three bins of equal frequency. With 10 observations, the first bin contains the 4 smallest values (22–35), the second contains the next 3 (35–45), and the third contains the largest 3 (50–65). The bin edges are determined by the quantiles.

**Example 3: Custom Bin Edges and Labels**

```python
# Custom bin edges and labels
bins = [0, 30, 50, 100]
labels = ['Young', 'Middle', 'Senior']
df['age_cat'] = pd.cut(df['age'], bins=bins, labels=labels)
print(df[['age', 'age_cat']])
# Expected output:
#    age age_cat
# 0   22   Young
# 1   25   Young
# 2   30   Young
# 3   35  Middle
# 4   40  Middle
# 5   45  Middle
# 6   50  Middle
# 7   55  Senior
# 8   60  Senior
# 9   65  Senior
```

**Why this output:** Custom bin edges `[0, 30, 50, 100]` define three intervals: `(0, 30]`, `(30, 50]`, and `(50, 100]`. The labels replace the interval notation with readable category names.

### Real-World Cases

- **Credit scoring:** Discretize income into “Low”, “Medium”, “High” for risk bands.
- **Marketing:** Segment customers by age groups for targeted campaigns.
- **Healthcare:** Bin BMI values into underweight, normal, overweight, and obese categories.

### References

- pandas.cut — https://pandas.pydata.org/docs/reference/api/pandas.cut.html
- pandas.qcut — https://pandas.pydata.org/docs/reference/api/pandas.qcut.html
- Discretization (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#discretization
- cut vs. qcut — Stack Overflow — https://stackoverflow.com/revisions/a13dd594-2aa3-419a-a8f5-aa09b05e7156


## 3. Precision Controls: Rounding, Floors, Ceilings, and Floating-Point Constraints

### Definitions

**Core Definition:** Precision controls reduce the number of decimal places in numerical values: `.round()` rounds to the nearest value, `np.floor()` rounds down to the nearest integer, and `np.ceil()` rounds up to the nearest integer.

**Technical Definition:** `DataFrame.round(decimals=0, *args, **kwargs)` rounds numerical columns to the specified number of decimal places. Pandas uses **round half to even** (banker's rounding): values exactly halfway between two rounded values are rounded to the nearest even number (e.g., 0.5 → 0.0, 1.5 → 2.0). NumPy's `np.floor()` rounds toward negative infinity, and `np.ceil()` rounds toward positive infinity. Floating-point representation (IEEE 754) means some decimal values cannot be represented exactly, causing rounding anomalies (e.g., `round(2.675, 2)` may return `2.67` instead of `2.68`).

**Beginner-Friendly Explanation:** Rounding reduces the number of digits after the decimal point. `.round()` does normal rounding, but with a twist: when a number is exactly halfway, pandas rounds to the nearest even number (so 2.5 becomes 2, not 3). `np.floor()` always rounds down, and `np.ceil()` always rounds up. Because computers store decimals in binary, some numbers like 2.675 can't be stored exactly, which can cause surprising rounding results.

### Purposes

- To reduce the precision of numerical data for presentation or storage.
- To convert floating-point numbers to integers using floor or ceiling.
- To control the number of decimal places for consistent output.
- To handle floating-point representation issues by understanding the limits of binary storage.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Rounding to decimal places
df.round(decimals=2)
df['col'].round(0)

# Floor (round down)
import numpy as np
np.floor(df['col'])

# Ceiling (round up)
np.ceil(df['col'])

# Truncation (round toward zero)
np.trunc(df['col'])
```

**Component Breakdown:**

| Function | Description | Example (2.7 → ) |
|----------|-------------|-------------------|
| `.round(0)` | Round to nearest even | 3.0 |
| `np.floor()` | Round toward −∞ | 2.0 |
| `np.ceil()` | Round toward +∞ | 3.0 |
| `np.trunc()` | Round toward zero | 2.0 |

**Syntax Rules:**

- `round(decimals)` accepts an integer or a dictionary/Series for per-column decimals.
- Pandas `.round()` uses round-half-to-even; NumPy's `np.round()` uses the same strategy.
- `np.floor()` and `np.ceil()` return floats; cast to int with `.astype(int)` if needed.
- **Version-specific:** In pandas 3.0, `.round()` with `decimals=0` and a `frac` parameter is deprecated.

**Constraints and Limitations:**

- Floating-point representation means some values cannot be represented exactly (e.g., 0.1, 0.2).
- Rounding a large dataset can introduce cumulative bias if not using a consistent strategy.
- `np.floor()` and `np.ceil()` on negative numbers: `floor(-2.5) = -3`, `ceil(-2.5) = -2`.

### Annotated Code Examples

**Example 1: Basic Rounding**

```python
import pandas as pd

df = pd.DataFrame({
    'value': [1.234, 2.567, 3.891, 4.123, 5.456]
})

# Round to 1 decimal place
df['rounded_1'] = df['value'].round(1)
df['rounded_0'] = df['value'].round(0)
print(df)
# Expected output:
#    value  rounded_1  rounded_0
# 0  1.234        1.2        1.0
# 1  2.567        2.6        3.0
# 2  3.891        3.9        4.0
# 3  4.123        4.1        4.0
# 4  5.456        5.5        5.0
```

**Why this output:** `.round(1)` rounds to one decimal place. `.round(0)` rounds to the nearest integer. 2.567 rounds to 2.6; 3.891 rounds to 3.9.

**Example 2: Banker's Rounding**

```python
# Banker's rounding: halfway values round to nearest even
s = pd.Series([0.5, 1.5, 2.5, 3.5, 4.5])
print(s.round(0))
# Expected output:
# 0    0.0
# 1    2.0
# 2    2.0
# 3    4.0
# 4    4.0
# dtype: float64
```

**Why this output:** 0.5 rounds to 0 (even), 1.5 rounds to 2 (even), 2.5 rounds to 2 (even), 3.5 rounds to 4 (even), 4.5 rounds to 4 (even). This minimises rounding bias in large datasets.

**Example 3: Floor and Ceiling**

```python
import numpy as np

# Floor and ceiling
df2 = pd.DataFrame({'value': [2.3, 2.7, -2.3, -2.7]})
df2['floor'] = np.floor(df2['value'])
df2['ceil'] = np.ceil(df2['value'])
print(df2)
# Expected output:
#    value  floor  ceil
# 0    2.3    2.0   3.0
# 1    2.7    2.0   3.0
# 2   -2.3   -3.0  -2.0
# 3   -2.7   -3.0  -2.0
```

**Why this output:** `floor(2.3) = 2.0`, `floor(2.7) = 2.0`, `floor(-2.3) = -3.0`, `floor(-2.7) = -3.0`. `ceil(2.3) = 3.0`, `ceil(2.7) = 3.0`, `ceil(-2.3) = -2.0`, `ceil(-2.7) = -2.0`.

### Real-World Cases

- **Financial reporting:** Round monetary values to two decimal places.
- **Inventory management:** Use `ceil()` to determine the number of packages needed.
- **Data visualisation:** Round axis labels to reduce clutter.

### References

- pandas.DataFrame.round — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.round.html
- numpy.floor — https://numpy.org/doc/stable/reference/generated/numpy.floor.html
- numpy.ceil — https://numpy.org/doc/stable/reference/generated/numpy.ceil.html
- How to Round Numbers in Python — https://realpython.com/python-rounding/


## 4. Outlier Clamping: Truncating Extreme Tails and Bounding Distribution Edges

### Definitions

**Core Definition:** Outlier clamping caps extreme values to specified lower and upper thresholds using `.clip()`, preserving the shape of the distribution while eliminating the influence of extreme tails.

**Technical Definition:** `DataFrame.clip(lower=None, upper=None, *, axis=None, inplace=False, **kwargs)` assigns values outside the boundary to the boundary values. Thresholds can be scalars or array-like, and in the latter case clipping is performed element-wise in the specified axis. A missing threshold (e.g., `NA`) will not clip the value. `clip()` is equivalent to but more concise than `np.where(df < lower, lower, np.where(df > upper, upper, df))`.

**Beginner-Friendly Explanation:** Clipping is like putting a speed limit on your data. If the speed limit is 100 and a value is 150, it gets reduced to 100. If a value is -20 and the lower limit is 0, it gets raised to 0. This keeps extreme values from distorting your analysis without deleting any rows.

### Purposes

- To cap outliers at a reasonable boundary without removing rows.
- To implement Winsorization (capping extreme values at a percentile).
- To ensure values fall within a valid domain (e.g., probabilities in [0, 1]).
- To prevent extreme values from dominating calculations like mean and standard deviation.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.clip(lower=None, upper=None, *, axis=None, inplace=False, **kwargs)
Series.clip(lower=None, upper=None, *, axis=None, inplace=False, **kwargs)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `lower` | Minimum threshold; values below are set to this value |
| `upper` | Maximum threshold; values above are set to this value |
| `axis` | Alignment axis for array-like thresholds |
| `inplace` | `True`: modify in place; `False` (default): return a new object |

**Syntax Rules:**

- `lower` and `upper` can be scalars or array-like of the same shape.
- If only `lower` is specified, there is no upper bound (and vice versa).
- `clip()` is vectorized and element-wise.
- For array-like thresholds with `axis=None`, the shape must match the object.

**Constraints and Limitations:**

- Clipping does not remove rows; it only changes values at the boundaries.
- Clipping can distort distributions if too many values are affected.
- The choice of lower and upper bounds should be justified by domain knowledge or statistical analysis (e.g., 1st and 99th percentiles).

### Annotated Code Examples

**Example 1: Basic Clipping**

```python
import pandas as pd

df = pd.DataFrame({
    'value': [-50, -10, 0, 10, 50, 100, 200]
})

# Clip values to [-20, 80]
df['clipped'] = df['value'].clip(lower=-20, upper=80)
print(df)
# Expected output:
#    value  clipped
# 0    -50      -20
# 1    -10      -10
# 2      0        0
# 3     10       10
# 4     50       50
# 5    100       80
# 6    200       80
```

**Why this output:** Values below -20 are set to -20 (row 0). Values above 80 are set to 80 (rows 5 and 6). Values within the range are unchanged.

**Example 2: Winsorization at Percentiles**

```python
# Winsorize at 5th and 95th percentiles
lower = df['value'].quantile(0.05)
upper = df['value'].quantile(0.95)
df['winsorized'] = df['value'].clip(lower=lower, upper=upper)
print(df[['value', 'winsorized']])
# Expected output (approximate):
#    value  winsorized
# 0    -50        -50.0
# 1    -10        -10.0
# 2      0          0.0
# 3     10         10.0
# 4     50         50.0
# 5    100        100.0
# 6    200        100.0
```

**Why this output:** The 5th percentile is -50 and the 95th percentile is 100. Values above 100 are clipped to 100. The lower bound is already satisfied, so no lower clipping occurs.

**Example 3: Clipping with Array-Like Thresholds**

```python
df2 = pd.DataFrame({
    'A': [1, 3, 5],
    'B': [2, 4, 6]
})

# Clip each column with different thresholds
df2_clipped = df2.clip(lower=[2, 3], upper=[5, 5], axis=1)
print(df2_clipped)
# Expected output:
#    A  B
# 0  2  3
# 1  3  4
# 2  5  5
```

**Why this output:** Column A has lower=2 and upper=5; column B has lower=3 and upper=5. Values in column A below 2 are raised to 2, and values above 5 are capped at 5. Column B is clipped similarly.

### Real-World Cases

- **Sensor data:** Cap erroneous sensor readings (e.g., temperature beyond physical limits) at plausible boundaries.
- **Financial returns:** Clip extreme daily returns to ±10% to reduce the impact of flash crashes.
- **Survey data:** Cap Likert-scale responses at the valid range (e.g., 1–5).

### References

- pandas.DataFrame.clip — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.clip.html
- pandas.Series.clip — https://pandas.pydata.org/docs/reference/api/pandas.Series.clip.html
- numpy.clip — https://numpy.org/doc/stable/reference/generated/numpy.clip.html
- Discretize and Clip Numerical Data with pandas — https://www.educative.io


## 5. Sign Conversions and Transformations: Binary Masks and Multi-Interval Flags

### Definitions

**Core Definition:** Sign conversion transforms continuous numerical values into binary (0/1) indicators or multi-interval flags based on thresholds or conditions, using boolean masking and type conversion.

**Technical Definition:** A binary mask is created by evaluating a boolean condition (e.g., `df['value'] > 0`) and converting the result to integers with `.astype(int)`. Multi-interval flags assign an integer code to each interval defined by `pd.cut()` or `pd.qcut()`, accessed via `.cat.codes`. These transformations are commonly used to create indicator features for machine learning models, risk flags, or signal indicators.

**Beginner-Friendly Explanation:** Sign conversion turns numbers into simple flags. For example, you can turn a column of temperatures into a binary flag “is it freezing?” (1 if temperature ≤ 0, 0 otherwise). Multi-interval flags assign a number to each range—like 0 for “Low”, 1 for “Medium”, 2 for “High”.

### Purposes

- To create binary indicator features from continuous variables (e.g., positive/negative, above/below threshold).
- To convert multi-interval bins into integer codes for machine learning.
- To implement risk flags (e.g., “high risk” = 1 if score > 80).
- To simplify complex numerical data into categorical signals.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Binary mask (sign flag)
df['is_positive'] = (df['value'] > 0).astype(int)

# Binary mask with multiple conditions
df['flag'] = ((df['value'] > 0) & (df['value'] < 100)).astype(int)

# Multi-interval flag via cut
df['bin_code'] = pd.cut(df['value'], bins=3, labels=False)

# Multi-interval flag via cat.codes
df['bin_code'] = pd.cut(df['value'], bins=3).cat.codes
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `(condition).astype(int)` | Converts boolean to 0/1 |
| `pd.cut(..., labels=False)` | Returns integer bin indicators |
| `.cat.codes` | Returns integer codes for categorical bins |
| `np.where(condition, 1, 0)` | Alternative binary mask creation |

**Syntax Rules:**

- Boolean Series can be converted to integers with `.astype(int)`.
- `pd.cut(labels=False)` returns integers starting from 0.
- `.cat.codes` returns -1 for NaN values.
- Multiple conditions are combined with `&` (and) and `|` (or), with parentheses around each condition.

**Constraints and Limitations:**

- Binary masks lose information about magnitude; only the sign or threshold comparison is retained.
- `cat.codes` returns -1 for missing values, which may need handling.
- Multi-interval flags assume a natural ordering of intervals.

### Annotated Code Examples

**Example 1: Binary Sign Mask**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'returns': [0.05, -0.02, 0.03, -0.01, 0.04]
})

# Binary flag: 1 if positive, 0 if negative
df['is_positive'] = (df['returns'] > 0).astype(int)
print(df)
# Expected output:
#    returns  is_positive
# 0     0.05            1
# 1    -0.02            0
# 2     0.03            1
# 3    -0.01            0
# 4     0.04            1
```

**Why this output:** The boolean Series `(df['returns'] > 0)` is `True` for positive returns and `False` for negative returns. `.astype(int)` converts `True` to 1 and `False` to 0.

**Example 2: Multi-Interval Flags with pd.cut()**

```python
# Multi-interval flags for age groups
df2 = pd.DataFrame({'age': [22, 35, 48, 61, 75]})

# 3 bins with integer labels
df2['age_bin'] = pd.cut(df2['age'], bins=3, labels=False)
print(df2)
# Expected output:
#    age  age_bin
# 0   22        0
# 1   35        0
# 2   48        1
# 3   61        2
# 4   75        2
```

**Why this output:** `labels=False` returns integer bin indicators starting from 0. The three bins correspond to the ranges `(21.947, 39.667]`, `(39.667, 57.333]`, and `(57.333, 75.0]`, coded as 0, 1, and 2.

**Example 3: Multi-Interval Flags with cat.codes**

```python
# Using cat.codes for custom labels
df2['age_cat'] = pd.cut(df2['age'], bins=3, labels=['Young', 'Middle', 'Senior'])
df2['age_code'] = df2['age_cat'].cat.codes
print(df2[['age', 'age_cat', 'age_code']])
# Expected output:
#    age age_cat  age_code
# 0   22   Young         0
# 1   35   Young         0
# 2   48  Middle         1
# 3   61  Senior         2
# 4   75  Senior         2
```

**Why this output:** The categorical column `age_cat` is created with custom labels. `.cat.codes` maps each label to its integer position in the categories array: “Young” → 0, “Middle” → 1, “Senior” → 2.

### Real-World Cases

- **Credit risk:** Create a binary flag for “high risk” based on a credit score threshold.
- **Medical diagnosis:** Flag patients as “hypertensive” if blood pressure exceeds 140/90.
- **Marketing:** Create multi-interval flags for customer value tiers (Bronze, Silver, Gold).

### References

- pandas.cut — https://pandas.pydata.org/docs/reference/api/pandas.cut.html
- pandas.Categorical.cat.codes — https://pandas.pydata.org/docs/reference/api/pandas.Categorical.cat.codes.html
- pandas.DataFrame.mask — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mask.html
- pandas.DataFrame.where — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.where.html


## References

- pandas.DataFrame.round — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.round.html
- pandas.DataFrame.clip — https://pandas.pydata.org/pandas-docs/version/2.2/reference/api/pandas.DataFrame.clip.html
- pandas.Series.clip — https://pandas.pydata.org/docs/reference/api/pandas.Series.clip.html
- pandas.cut — https://pandas.pydata.org/docs/reference/api/pandas.cut.html
- pandas.qcut — https://pandas.pydata.org/docs/reference/api/pandas.qcut.html
- pandas.DataFrame.mask — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mask.html
- pandas.DataFrame.where — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.where.html
- pandas.Categorical.cat.codes — https://pandas.pydata.org/docs/reference/api/pandas.Categorical.cat.codes.html
- numpy.clip — https://numpy.org/doc/stable/reference/generated/numpy.clip.html
- numpy.floor — https://numpy.org/doc/stable/reference/generated/numpy.floor.html
- numpy.ceil — https://numpy.org/doc/stable/reference/generated/numpy.ceil.html
- MinMaxScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html
- StandardScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html
- RobustScaler — https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.RobustScaler.html
- Discretization (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#discretization
- How to Round Numbers in Python — https://realpython.com/python-rounding/
- Data Normalization and Standardization — https://notebooks.githubusercontent.com/view/ipynb
- cut vs. qcut — Stack Overflow — https://stackoverflow.com/revisions/a13dd594-2aa3-419a-a8f5-aa09b05e7156
- Discretize and Clip Numerical Data with pandas — https://www.educative.io