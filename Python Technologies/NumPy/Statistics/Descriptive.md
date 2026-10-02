# Descriptive Statistics — A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Descriptive statistics are numerical summaries that characterize the central tendency, dispersion, and distributional shape of a dataset without making inferences about a larger population. NumPy provides a comprehensive suite of functions for computing these summaries on arrays of any dimension.

**Technical Definition:** NumPy's descriptive statistics functions operate on `ndarray` objects and follow the reduction paradigm: they collapse one or more axes of the input array into scalar or lower-dimensional outputs. Central tendency measures include `np.mean()` (arithmetic average) and `np.median()` (middle value). Dispersion measures include `np.var()` (variance) and `np.std()` (standard deviation), both controlled by the `ddof` (delta degrees of freedom) parameter for population vs. sample adjustments. Order statistics include `np.min()`, `np.max()`, `np.argmin()`, and `np.argmax()`. Relative standing measures include `np.percentile()` and `np.quantile()`. NaN-aware variants (`np.nanmean`, `np.nanstd`, `np.nanmedian`, etc.) handle missing data by ignoring `NaN` values during computation.

**Beginner-Friendly Explanation:** Descriptive statistics are like the "elevator pitch" for your data. Instead of looking at thousands of numbers, you summarize them with a few key values: the average (mean), the middle value (median), how spread out they are (variance and standard deviation), and where specific values fall (percentiles). NumPy makes these calculations fast and easy, even for huge datasets.

### Key Characteristics

- **Reduction Paradigm:** All functions collapse one or more axes, returning scalars or lower-dimensional arrays.
- **Axis-Aware:** The `axis` parameter controls which dimension(s) to reduce.
- **`ddof` Control:** Variance and standard deviation support population (`ddof=0`) and sample (`ddof=1`) adjustments.
- **NaN-Aware Variants:** Functions prefixed with `nan` ignore `NaN` values, enabling robust statistics on incomplete data.
- **Broadcasting Support:** Functions accept array-like inputs and support `keepdims` for broadcasting compatibility.

### Prerequisites

- Basic Python programming (variables, functions, imports)
- Familiarity with NumPy arrays, dtypes, and indexing
- Basic statistical concepts (mean, median, variance, standard deviation)
- Understanding of the `axis` parameter for multi-dimensional arrays

### Related Programming Areas

- Data Science & Exploratory Data Analysis (EDA)
- Machine Learning (feature scaling, normalization, outlier detection)
- Finance (risk metrics, return distributions)
- Quality Control (process monitoring, six-sigma)
- Biostatistics (clinical trial analysis)
- Signal Processing (noise characterization)

### Core Concepts / Features

1. Central Tendency: `np.mean()` and `np.median()`
2. Dispersion Measures: `np.var()` and `np.std()` (Population vs. Sample via `ddof`)
3. Order Statistics: `np.min()`, `np.max()`, `np.argmin()`, `np.argmax()`
4. Relative Standing: `np.percentile()` and `np.quantile()`
5. Missing Data Handling: Robust Alternatives (`np.nanmean`, `np.nanstd`, `np.nanmedian`, etc.)

---

## Core Concept 1: Central Tendency — `np.mean()` and `np.median()`

### Definitions

**Core Definition:** Central tendency measures identify a single value that represents the "center" of a dataset. The mean is the arithmetic average, and the median is the middle value when the data is sorted.

**Technical Definition:** `numpy.mean(a, axis=None, dtype=None, out=None, keepdims=False, *, where=)` computes the arithmetic mean \( \bar{a} = \frac{1}{N} \sum_{i=1}^{N} a_i \) along the specified axis. For integer inputs, the default computation uses `float64` intermediates and returns `float64`; for floating-point inputs, it uses the same dtype as the input. `numpy.median(a, axis=None, out=None, overwrite_input=False, keepdims=False)` computes the median: for odd \( N \), it is the middle value \( V_{\text{sorted}}[(N-1)/2] \); for even \( N \), it is the average of the two middle values.

**Beginner-Friendly Explanation:** The mean is what you get when you add up all the numbers and divide by how many there are — the "average." The median is the number that sits right in the middle when you line up all the values from smallest to largest. If you have an even number of values, the median is the average of the two middle ones. The median is more robust to extreme values (outliers) than the mean.

### Purposes

- To summarize a dataset with a single representative value.
- To compare central tendency across different groups or conditions.
- To detect skewness by comparing the mean and median (if they differ substantially, the data is skewed).
- To provide a baseline for measuring dispersion and relative standing.
- To support statistical inference procedures that rely on sample means (e.g., t-tests).

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Arithmetic mean
mean_value = np.mean(a, axis=None, dtype=None, out=None, keepdims=False, *, where=)

# Median
median_value = np.median(a, axis=None, out=None, overwrite_input=False, keepdims=False)
```

**Component Breakdown:**
- `a`: Input array or array-like object.
- `axis`: Axis or axes along which the mean/median is computed. Default `None` computes over the flattened array.
- `dtype`: Type to use in computing the mean. For integer inputs, default is `float64`.
- `out`: Alternative output array.
- `keepdims`: If `True`, retains reduced dimensions as size-1, enabling broadcasting.
- `where`: Array-like of bool, selecting elements to include (mean only).
- `overwrite_input`: If `True`, allows the input array to be modified (median only).

**Syntax Rules:**
- For integer inputs, `mean` returns `float64`; for floating-point inputs, it returns the input dtype.
- `median` returns `float64` for integer or float32 inputs; otherwise, it returns the input dtype.
- `axis` can be `None`, an integer, or a tuple of integers.
- `keepdims=True` preserves the reduced dimensions as size-1, useful for broadcasting.

**Constraints and Limitations:**
- `mean` is sensitive to outliers; `median` is robust.
- For `float32` inputs, `mean` can be inaccurate due to floating-point accumulation; specifying `dtype=np.float64` mitigates this.
- `median` with `overwrite_input=True` modifies the input array, which may be undesirable.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Mean and Median Computation

```python
import numpy as np

# Step 1: Create a 1-D array with an outlier
data = np.array([12, 15, 18, 20, 22, 25, 30, 100])
print(f"Data: {data}")

# Step 2: Compute mean and median
mean_val = np.mean(data)
median_val = np.median(data)
print(f"\nMean:   {mean_val:.2f}")
print(f"Median: {median_val:.2f}")
print(f"Difference: {mean_val - median_val:.2f}")
print("(Large difference indicates right-skewness due to the outlier 100.)")

# Step 3: 2-D array with axis parameter
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
print(f"\nMatrix:\n{matrix}")

print(f"\nMean (flattened): {np.mean(matrix):.2f}")
print(f"Mean (axis=0):    {np.mean(matrix, axis=0)}")
print(f"Mean (axis=1):    {np.mean(matrix, axis=1)}")

print(f"\nMedian (flattened): {np.median(matrix):.2f}")
print(f"Median (axis=0):    {np.median(matrix, axis=0)}")
print(f"Median (axis=1):    {np.median(matrix, axis=1)}")

# Step 4: keepdims for broadcasting
mean_keepdims = np.mean(matrix, axis=1, keepdims=True)
print(f"\nMean axis=1 keepdims shape: {mean_keepdims.shape}")
print(f"Values:\n{mean_keepdims}")

# Centering the matrix by subtracting the row means
centered = matrix - mean_keepdims
print(f"\nCentered matrix:\n{centered}")
print(f"Row means after centering: {np.mean(centered, axis=1)}")
```

**Expected Output:**
```
Data: [ 12  15  18  20  22  25  30 100]

Mean:   30.25
Median: 21.00
Difference: 9.25
(Large difference indicates right-skewness due to the outlier 100.)

Matrix:
[[1 2 3]
 [4 5 6]
 [7 8 9]]

Mean (flattened): 5.00
Mean (axis=0):    [4. 5. 6.]
Mean (axis=1):    [2. 5. 8.]

Median (flattened): 5.00
Median (axis=0):    [4. 5. 6.]
Median (axis=1):    [2. 5. 8.]

Mean axis=1 keepdims shape: (3, 1)
Values:
[[2.]
 [5.]
 [8.]]

Centered matrix:
[[-1  0  1]
 [-1  0  1]
 [-1  0  1]]
Row means after centering: [0. 0. 0.]
```

**Why This Output Occurs:** The mean (30.25) is pulled upward by the outlier 100, while the median (21.00) remains close to the bulk of the data. For the 2-D matrix, `axis=0` computes means down columns `[4, 5, 6]`, and `axis=1` computes means across rows `[2, 5, 8]`. The `keepdims=True` version preserves the shape `(3, 1)` for broadcasting, allowing the row means to be subtracted from the matrix.

#### Example 2: Weighted Mean with `np.average()`

```python
import numpy as np

# Step 1: Create data and weights
data = np.array([10, 20, 30, 40])
weights = np.array([0.1, 0.2, 0.3, 0.4])

print(f"Data:    {data}")
print(f"Weights: {weights}")

# Step 2: Simple mean (unweighted)
simple_mean = np.mean(data)
print(f"\nSimple mean: {simple_mean}")

# Step 3: Weighted mean
weighted_mean = np.average(data, weights=weights)
print(f"Weighted mean: {weighted_mean:.2f}")

# Step 4: Verify weighted mean manually
manual = np.sum(data * weights) / np.sum(weights)
print(f"Manual weighted mean: {manual:.2f}")
print(f"Match: {np.isclose(weighted_mean, manual)}")

# Step 5: 2-D weighted average along axis
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
row_weights = np.array([0.7, 0.3])

weighted_rows = np.average(matrix, axis=0, weights=row_weights)
print(f"\nMatrix:\n{matrix}")
print(f"Row weights: {row_weights}")
print(f"Weighted average (axis=0): {weighted_rows}")
```

**Expected Output:**
```
Data:    [10 20 30 40]
Weights: [0.1 0.2 0.3 0.4]

Simple mean: 25.0
Weighted mean: 30.00
Manual weighted mean: 30.00
Match: True

Matrix:
[[1 2 3]
 [4 5 6]]
Row weights: [0.7 0.3]
Weighted average (axis=0): [1.9 2.9 3.9]
```

**Why This Output Occurs:** The simple mean of `[10, 20, 30, 40]` is 25.0. The weighted mean gives more influence to the larger values (weights 0.3 and 0.4), producing 30.0. The 2-D weighted average combines rows using the specified weights, producing `0.7*1 + 0.3*4 = 1.9` for the first column, and so on.

### Real-World Cases

- **Income Analysis:** Median income is preferred over mean income because a few billionaires skew the mean.
- **Quality Control:** Mean and median measurements of manufactured parts are compared to specifications.
- **Financial Returns:** Mean returns measure average performance; median returns measure typical performance.
- **Sensor Data:** Median filtering removes spikes from sensor readings while preserving the underlying signal.

---

## Core Concept 2: Dispersion Measures — `np.var()` and `np.std()` (Population vs. Sample via `ddof`)

### Definitions

**Core Definition:** Dispersion measures quantify how spread out a dataset is around its center. Variance is the average squared deviation from the mean; standard deviation is the square root of variance.

**Technical Definition:** `numpy.var(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)` computes the variance \( \text{var} = \frac{\sum_i |a_i - \bar{a}|^2}{N - \text{ddof}} \). `numpy.std(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)` computes the standard deviation \( \text{std} = \sqrt{\text{var}} \). The `ddof` parameter ("Delta Degrees of Freedom") controls the divisor: `ddof=0` (default) gives the population variance (divisor \( N \)), while `ddof=1` gives the sample variance (divisor \( N-1 \)), which is an unbiased estimator of the population variance.

**Beginner-Friendly Explanation:** Variance tells you how far, on average, your data points are from the mean — but squared (to avoid negative distances cancelling out). Standard deviation is the square root of variance, so it's in the same units as the data. The `ddof` parameter is a technical adjustment: use `ddof=0` when your data is the entire population, and `ddof=1` when your data is a sample from a larger population.

### Purposes

- To measure the spread or variability of a dataset.
- To compare the consistency of different datasets (lower variance = more consistent).
- To standardize data for machine learning (z-score normalization: \( z = (x - \mu) / \sigma \)).
- To construct confidence intervals and perform hypothesis tests.
- To detect outliers (values more than 2 or 3 standard deviations from the mean).

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Variance
var_value = np.var(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)

# Standard deviation
std_value = np.std(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)
```

**Component Breakdown:**
- `a`: Input array.
- `axis`: Axis or axes along which variance/std is computed.
- `dtype`: Type to use in computing. For integer inputs, default is `float64`.
- `ddof`: "Delta Degrees of Freedom." Divisor is `N - ddof`. Default 0 (population).
- `keepdims`: If `True`, retains reduced dimensions as size-1.
- `where`: Array-like of bool, selecting elements.
- `mean`: Precomputed mean (avoids recalculation; NumPy 2.0+).
- `correction`: Array API compatible name for `ddof` (NumPy 2.0+).

**Syntax Rules:**
- `ddof=0` gives population variance; `ddof=1` gives sample variance.
- For integer inputs, `var` and `std` return `float64`.
- `std` is always the square root of `var`.
- `ddof` can be a float (e.g., `ddof=0.5`), though integers are typical.

**Constraints and Limitations:**
- `ddof` must be less than `N` to avoid division by zero.
- For `float32` inputs, numerical precision can be an issue; use `dtype=np.float64`.
- The population variance (`ddof=0`) underestimates the true variance when computed from a sample.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Population vs. Sample Variance and Standard Deviation

```python
import numpy as np

# Step 1: Create a sample dataset
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
N = len(data)
print(f"Data: {data}")
print(f"Sample size N: {N}")

# Step 2: Compute mean
mean_val = np.mean(data)
print(f"\nMean: {mean_val}")

# Step 3: Population variance (ddof=0)
pop_var = np.var(data, ddof=0)
pop_std = np.std(data, ddof=0)
print(f"\nPopulation variance (ddof=0): {pop_var:.4f}")
print(f"Population std (ddof=0):      {pop_std:.4f}")

# Step 4: Sample variance (ddof=1)
sample_var = np.var(data, ddof=1)
sample_std = np.std(data, ddof=1)
print(f"\nSample variance (ddof=1):     {sample_var:.4f}")
print(f"Sample std (ddof=1):          {sample_std:.4f}")

# Step 5: Verify the difference
print(f"\nRatio (sample/population variance): {sample_var / pop_var:.4f}")
print(f"Expected ratio N/(N-1):            {N/(N-1):.4f}")
print(f"Match: {np.isclose(sample_var / pop_var, N/(N-1))}")

# Step 6: Manual calculation for verification
manual_pop_var = np.sum((data - mean_val)**2) / N
manual_sample_var = np.sum((data - mean_val)**2) / (N - 1)
print(f"\nManual population variance: {manual_pop_var:.4f}")
print(f"Manual sample variance:     {manual_sample_var:.4f}")

# Step 7: 2-D array with axis parameter
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
print(f"\nMatrix:\n{matrix}")
print(f"Variance (axis=0, ddof=0): {np.var(matrix, axis=0)}")
print(f"Variance (axis=1, ddof=0): {np.var(matrix, axis=1)}")
print(f"Std (axis=0, ddof=1):      {np.std(matrix, axis=0, ddof=1)}")
```

**Expected Output:**
```
Data: [2 4 4 4 5 5 7 9]
Sample size N: 8

Mean: 5.0

Population variance (ddof=0): 4.0000
Population std (ddof=0):      2.0000

Sample variance (ddof=1):     4.5714
Sample std (ddof=1):          2.1381

Ratio (sample/population variance): 1.1429
Expected ratio N/(N-1):            1.1429
Match: True

Manual population variance: 4.0000
Manual sample variance:     4.5714

Matrix:
[[1 2 3]
 [4 5 6]]
Variance (axis=0, ddof=0): [2.25 2.25 2.25]
Variance (axis=1, ddof=0): [0.6667 0.6667]
Std (axis=0, ddof=1):      [2.1213 2.1213 2.1213]
```

**Why This Output Occurs:** The population variance (divisor \( N = 8 \)) is 4.0, while the sample variance (divisor \( N-1 = 7 \)) is 4.5714. The ratio is exactly \( N/(N-1) = 8/7 = 1.1429 \). For the 2-D matrix, `axis=0` computes variance down columns, and `axis=1` computes variance across rows. The standard deviation is always the square root of the corresponding variance.

#### Example 2: Z-Score Standardization

```python
import numpy as np

# Step 1: Create a dataset
data = np.array([10, 12, 14, 16, 18, 20])
print(f"Original data: {data}")

# Step 2: Compute mean and standard deviation
mean_val = np.mean(data)
std_val = np.std(data)
print(f"Mean: {mean_val}")
print(f"Std:  {std_val:.4f}")

# Step 3: Compute z-scores
z_scores = (data - mean_val) / std_val
print(f"\nZ-scores: {z_scores.round(4)}")

# Step 4: Verify properties of z-scores
print(f"\nZ-score mean: {np.mean(z_scores):.10f}")
print(f"Z-score std:  {np.std(z_scores):.10f}")
print("(Z-scores always have mean 0 and std 1.)")

# Step 5: Detect outliers (|z| > 2)
outlier_mask = np.abs(z_scores) > 2
print(f"\nOutlier mask (|z| > 2): {outlier_mask}")
print(f"Outliers: {data[outlier_mask]}")
```

**Expected Output:**
```
Original data: [10 12 14 16 18 20]
Mean: 15.0
Std:  3.4157

Z-scores: [-1.4639 -0.8783 -0.2928  0.2928  0.8783  1.4639]

Z-score mean: 0.0000000000
Z-score std:  1.0000000000
(Z-scores always have mean 0 and std 1.)

Outlier mask (|z| > 2): [False False False False False False]
Outliers: []
```

**Why This Output Occurs:** Z-scores measure how many standard deviations each data point is from the mean. The z-score mean is exactly 0 and the z-score standard deviation is exactly 1 (by construction). None of the values in this dataset exceed 2 standard deviations, so no outliers are detected.

### Real-World Cases

- **Finance:** Standard deviation of returns measures investment risk (volatility).
- **Manufacturing:** Process standard deviation determines quality control limits (Six Sigma).
- **Machine Learning:** Feature standardization (z-score) is a prerequisite for many algorithms (SVM, PCA, neural networks).
- **Biostatistics:** Sample variance is used to compute standard errors and confidence intervals.

---

## Core Concept 3: Order Statistics — `np.min()`, `np.max()`, `np.argmin()`, `np.argmax()`

### Definitions

**Core Definition:** Order statistics are values that characterize the position of elements within a dataset when sorted. The minimum and maximum are the extreme values, while `argmin` and `argmax` return the indices of these extremes.

**Technical Definition:** `numpy.min(a, axis=None, out=None, keepdims=False, initial=, where=)` returns the minimum value along the specified axis. `numpy.max(a, axis=None, out=None, keepdims=False, initial=, where=)` returns the maximum. `numpy.argmin(a, axis=None, out=None, keepdims=False)` returns the indices of the minimum values, and `numpy.argmax(a, axis=None, out=None, keepdims=False)` returns the indices of the maximum values. For multi-dimensional arrays, the `axis` parameter determines the direction of reduction. The `argmin` and `argmax` functions return the index of the first occurrence if there are multiple minima/maxima.

**Beginner-Friendly Explanation:** `min` and `max` tell you the smallest and largest values in your data. `argmin` and `argmax` tell you where those values are located (their index positions). This is useful when you need to know not just the extreme value but also which data point produced it.

### Purposes

- To identify the range of a dataset (max - min).
- To locate the position of extreme values for further analysis.
- To detect outliers and anomalies.
- To implement ranking and selection algorithms.
- To find the best or worst performing item in a collection.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Minimum and maximum values
min_val = np.min(a, axis=None, out=None, keepdims=False, initial=, where=)
max_val = np.max(a, axis=None, out=None, keepdims=False, initial=, where=)

# Indices of minimum and maximum
min_idx = np.argmin(a, axis=None, out=None, keepdims=False)
max_idx = np.argmax(a, axis=None, out=None, keepdims=False)
```

**Component Breakdown:**
- `a`: Input array.
- `axis`: Axis or axes along which the operation is performed. Default `None` operates on the flattened array.
- `out`: Alternative output array.
- `keepdims`: If `True`, retains reduced dimensions as size-1.
- `initial`: The maximum value of an output element (for `min`) or minimum value (for `max`). Used to handle empty arrays.
- `where`: Array-like of bool, selecting elements.

**Syntax Rules:**
- `argmin`/`argmax` return integer indices; for `axis=None`, the index is into the flattened array.
- For multi-dimensional arrays with `axis`, the result has one fewer dimension.
- `min`/`max` support `initial` and `where` parameters; `argmin`/`argmax` do not.

**Constraints and Limitations:**
- `argmin`/`argmax` return the first occurrence in case of ties.
- For empty arrays, `min`/`max` raise `ValueError` unless `initial` is provided.
- `argmin`/`argmax` on empty arrays raise `ValueError`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Min, Max, Argmin, Argmax

```python
import numpy as np

# Step 1: Create a 1-D array
data = np.array([45, 12, 78, 34, 90, 23, 67])
print(f"Data: {data}")

# Step 2: Min and max values
min_val = np.min(data)
max_val = np.max(data)
print(f"\nMinimum: {min_val}")
print(f"Maximum: {max_val}")
print(f"Range:   {max_val - min_val}")

# Step 3: Indices of min and max
min_idx = np.argmin(data)
max_idx = np.argmax(data)
print(f"\nIndex of minimum: {min_idx} (value: {data[min_idx]})")
print(f"Index of maximum: {max_idx} (value: {data[max_idx]})")

# Step 4: 2-D array with axis parameter
matrix = np.array([[5, 2, 9],
                   [1, 8, 3],
                   [7, 4, 6]])
print(f"\nMatrix:\n{matrix}")

print(f"\nMin (flattened): {np.min(matrix)}")
print(f"Max (flattened): {np.max(matrix)}")
print(f"Argmin (flattened): {np.argmin(matrix)}")
print(f"Argmax (flattened): {np.argmax(matrix)}")

print(f"\nMin (axis=0): {np.min(matrix, axis=0)}")
print(f"Max (axis=0): {np.max(matrix, axis=0)}")
print(f"Argmin (axis=0): {np.argmin(matrix, axis=0)}")
print(f"Argmax (axis=0): {np.argmax(matrix, axis=0)}")

print(f"\nMin (axis=1): {np.min(matrix, axis=1)}")
print(f"Max (axis=1): {np.max(matrix, axis=1)}")
print(f"Argmin (axis=1): {np.argmin(matrix, axis=1)}")
print(f"Argmax (axis=1): {np.argmax(matrix, axis=1)}")

# Step 5: Verify argmin/argmax indices
flat_idx = np.argmin(matrix)
row, col = np.unravel_index(flat_idx, matrix.shape)
print(f"\nFlat argmin index: {flat_idx} → position ({row}, {col}) = {matrix[row, col]}")
```

**Expected Output:**
```
Data: [45 12 78 34 90 23 67]

Minimum: 12
Maximum: 90
Range:   78

Index of minimum: 1 (value: 12)
Index of maximum: 4 (value: 90)

Matrix:
[[5 2 9]
 [1 8 3]
 [7 4 6]]

Min (flattened): 1
Max (flattened): 9
Argmin (flattened): 3
Argmax (flattened): 2

Min (axis=0): [1 2 3]
Max (axis=0): [7 8 9]
Argmin (axis=0): [1 0 1]
Argmax (axis=0): [2 1 0]

Min (axis=1): [2 1 4]
Max (axis=1): [9 8 7]
Argmin (axis=1): [1 0 1]
Argmax (axis=1): [2 2 0]

Flat argmin index: 3 → position (1, 0) = 1
```

**Why This Output Occurs:** For the 1-D array, the minimum is 12 at index 1 and the maximum is 90 at index 4. For the 2-D matrix, `axis=0` reduces down columns (min of column 0 is 1, max is 7), and `axis=1` reduces across rows (min of row 0 is 2, max is 9). The flattened argmin index 3 corresponds to position (1, 0) using `np.unravel_index`.

#### Example 2: Using Argmin/Argmax for Selection

```python
import numpy as np

# Step 1: Simulate student exam scores
scores = np.array([78, 92, 65, 88, 95, 72, 85])
students = np.array(['Alice', 'Bob', 'Charlie', 'Diana', 'Eve', 'Frank', 'Grace'])

print("Student scores:")
for s, sc in zip(students, scores):
    print(f"  {s}: {sc}")

# Step 2: Find highest and lowest scoring students
best_idx = np.argmax(scores)
worst_idx = np.argmin(scores)

print(f"\nHighest score: {students[best_idx]} with {scores[best_idx]}")
print(f"Lowest score:  {students[worst_idx]} with {scores[worst_idx]}")

# Step 3: Find top 3 students using argsort
top3_indices = np.argsort(scores)[::-1][:3]
print(f"\nTop 3 students:")
for rank, idx in enumerate(top3_indices, 1):
    print(f"  Rank {rank}: {students[idx]} ({scores[idx]})")

# Step 4: Ties handling — argmax returns first occurrence
tied_scores = np.array([90, 85, 90, 88, 90])
print(f"\nTied scores: {tied_scores}")
print(f"argmax (first max): {np.argmax(tied_scores)}")
print(f"argmin (first min): {np.argmin(tied_scores)}")
```

**Expected Output:**
```
Student scores:
  Alice: 78
  Bob: 92
  Charlie: 65
  Diana: 88
  Eve: 95
  Frank: 72
  Grace: 85

Highest score: Eve with 95
Lowest score:  Charlie with 65

Top 3 students:
  Rank 1: Eve (95)
  Rank 2: Bob (92)
  Rank 3: Diana (88)

Tied scores: [90 85 90 88 90]
argmax (first max): 0
argmin (first min): 1
```

**Why This Output Occurs:** `argmax` returns the index of the highest score (Eve, index 4), and `argmin` returns the index of the lowest (Charlie, index 2). The `argsort` with `[::-1]` gives descending order for ranking. For tied scores, `argmax` returns the first occurrence (index 0), and `argmin` returns the first minimum (index 1).

### Real-World Cases

- **Sports Analytics:** Finding the highest-scoring player and their position on the roster.
- **Finance:** Identifying the best and worst performing stocks in a portfolio.
- **Quality Control:** Locating the defective item in a production batch.
- **Image Processing:** Finding the brightest pixel and its coordinates.

---

## Core Concept 4: Relative Standing — `np.percentile()` and `np.quantile()`

### Definitions

**Core Definition:** Percentiles and quantiles indicate the relative standing of a value within a dataset. The \( q \)-th percentile is the value below which \( q \) percent of the data falls. Quantiles are the same concept but use the range [0, 1] instead of [0, 100].

**Technical Definition:** `numpy.percentile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)` computes the \( q \)-th percentile of the data, where \( q \) is in [0, 100]. `numpy.quantile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False, *, weights=None)` computes the \( q \)-th quantile, where \( q \) is in [0, 1]. Given a vector \( V \) of length \( N \), the \( q \)-th percentile is the value \( q/100 \) of the way from the minimum to the maximum in a sorted copy of \( V \). The `method` parameter (default `'linear'`) determines how interpolation is performed.

**Beginner-Friendly Explanation:** Percentiles tell you where a particular value stands relative to the rest of the data. If your score is at the 90th percentile, you scored better than 90% of the people. The median is the 50th percentile. The 25th and 75th percentiles (quartiles) divide the data into four equal parts.

### Purposes

- To determine the relative standing of individual values within a dataset.
- To compute quartiles, deciles, and other quantile-based summaries.
- To construct box plots and detect outliers using the interquartile range (IQR).
- To set thresholds for classification and decision-making.
- To compare distributions across different groups.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# Percentile (q in [0, 100])
p = np.percentile(a, q, axis=None, out=None, overwrite_input=False,
                  method='linear', keepdims=False, *, weights=None)

# Quantile (q in [0, 1])
q_val = np.quantile(a, q, axis=None, out=None, overwrite_input=False,
                    method='linear', keepdims=False, *, weights=None)
```

**Component Breakdown:**
- `a`: Input array.
- `q`: Percentage (0–100 for `percentile`) or probability (0–1 for `quantile`).
- `axis`: Axis or axes along which the operation is performed.
- `method`: Interpolation method. Default `'linear'`. Options include `'lower'`, `'higher'`, `'midpoint'`, `'nearest'`, `'inverted_cdf'`, etc.
- `keepdims`: If `True`, retains reduced dimensions as size-1.
- `weights`: Array of weights (NumPy 2.0+). Only `method='inverted_cdf'` supports weights.

**Syntax Rules:**
- `percentile(a, 50)` is equivalent to `median(a)`.
- `quantile(a, 0.5)` is equivalent to `percentile(a, 50)`.
- The `method` parameter controls interpolation between data points.
- For multiple `q` values, the first axis of the result corresponds to the percentiles.

**Constraints and Limitations:**
- Different interpolation methods can give different results; `'linear'` is the default and most common.
- Weights are only supported with `method='inverted_cdf'`.
- Percentiles are sensitive to the number of data points; small datasets may not have meaningful percentiles.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Percentile and Quantile Computation

```python
import numpy as np

# Step 1: Create a dataset
data = np.array([12, 15, 18, 20, 22, 25, 28, 30, 35, 40, 45, 50])
print(f"Data: {data}")
print(f"N = {len(data)}")

# Step 2: Compute percentiles
p25 = np.percentile(data, 25)
p50 = np.percentile(data, 50)
p75 = np.percentile(data, 75)
p90 = np.percentile(data, 90)

print(f"\n25th percentile (Q1): {p25}")
print(f"50th percentile (median): {p50}")
print(f"75th percentile (Q3): {p75}")
print(f"90th percentile: {p90}")

# Step 3: Compute quantiles (equivalent)
q25 = np.quantile(data, 0.25)
q50 = np.quantile(data, 0.50)
print(f"\nquantile(0.25): {q25}")
print(f"quantile(0.50): {q50}")
print(f"percentile(25) == quantile(0.25): {np.isclose(p25, q25)}")
print(f"percentile(50) == median: {np.isclose(p50, np.median(data))}")

# Step 4: IQR and outlier detection
iqr = p75 - p25
lower_bound = p25 - 1.5 * iqr
upper_bound = p75 + 1.5 * iqr
print(f"\nIQR: {iqr}")
print(f"Lower bound (Q1 - 1.5*IQR): {lower_bound}")
print(f"Upper bound (Q3 + 1.5*IQR): {upper_bound}")

outliers = data[(data < lower_bound) | (data > upper_bound)]
print(f"Outliers: {outliers}")

# Step 5: Multiple percentiles at once
percentiles = np.percentile(data, [10, 25, 50, 75, 90])
print(f"\n10th, 25th, 50th, 75th, 90th percentiles: {percentiles}")

# Step 6: 2-D array with axis
matrix = np.array([[10, 20, 30],
                   [40, 50, 60],
                   [70, 80, 90]])
print(f"\nMatrix:\n{matrix}")
print(f"50th percentile (axis=0): {np.percentile(matrix, 50, axis=0)}")
print(f"50th percentile (axis=1): {np.percentile(matrix, 50, axis=1)}")
print(f"75th percentile (axis=1): {np.percentile(matrix, 75, axis=1)}")
```

**Expected Output:**
```
Data: [12 15 18 20 22 25 28 30 35 40 45 50]
N = 12

25th percentile (Q1): 19.5
50th percentile (median): 26.5
75th percentile (Q3): 33.75
90th percentile: 43.0

quantile(0.25): 19.5
quantile(0.50): 26.5
percentile(25) == quantile(0.25): True
percentile(50) == median: True

IQR: 14.25
Lower bound (Q1 - 1.5*IQR): -1.875
Upper bound (Q3 + 1.5*IQR): 55.125
Outliers: []

10th, 25th, 50th, 75th, 90th percentiles: [16.1  19.5  26.5  33.75 43.  ]

Matrix:
[[10 20 30]
 [40 50 60]
 [70 80 90]]
50th percentile (axis=0): [40. 50. 60.]
50th percentile (axis=1): [20. 50. 80.]
75th percentile (axis=1): [25. 55. 85.]
```

**Why This Output Occurs:** The 25th percentile (Q1) is 19.5, the median (50th percentile) is 26.5, and the 75th percentile (Q3) is 33.75. The IQR is `33.75 - 19.5 = 14.25`. The outlier bounds are `-1.875` to `55.125`, and no data points fall outside this range. For the 2-D matrix, `axis=0` computes percentiles down columns, and `axis=1` computes across rows.

#### Example 2: Interpolation Methods Comparison

```python
import numpy as np

# Step 1: Create a small dataset where interpolation matters
data = np.array([1, 2, 3, 4, 5])
print(f"Data: {data}")

# Step 2: Compute 35th percentile with different methods
methods = ['linear', 'lower', 'higher', 'midpoint', 'nearest']
print(f"\n35th percentile with different methods:")
for method in methods:
    result = np.percentile(data, 35, method=method)
    print(f"  {method:10s}: {result}")

# Step 3: Compute 65th percentile with different methods
print(f"\n65th percentile with different methods:")
for method in methods:
    result = np.percentile(data, 65, method=method)
    print(f"  {method:10s}: {result}")

# Step 4: Explain the linear interpolation
# For q=35, position = (N-1) * q/100 = 4 * 0.35 = 1.4
# Value at index 1 = 2, index 2 = 3
# Linear interpolation: 2 + 0.4 * (3 - 2) = 2.4
position = (len(data) - 1) * 0.35
print(f"\nLinear interpolation for 35th percentile:")
print(f"  Position = (N-1) * q/100 = {position}")
print(f"  Value at index 1 = {data[1]}, index 2 = {data[2]}")
print(f"  Interpolated: {data[1]} + {position - 1:.1f} * ({data[2]} - {data[1]}) = {data[1] + (position - 1) * (data[2] - data[1]):.1f}")
```

**Expected Output:**
```
Data: [1 2 3 4 5]

35th percentile with different methods:
  linear    : 2.4
  lower     : 2
  higher    : 3
  midpoint  : 2.5
  nearest   : 2

65th percentile with different methods:
  linear    : 3.6
  lower     : 3
  higher    : 4
  midpoint  : 3.5
  nearest   : 4

Linear interpolation for 35th percentile:
  Position = (N-1) * q/100 = 1.4
  Value at index 1 = 2, index 2 = 3
  Interpolated: 2 + 0.4 * (3 - 2) = 2.4
```

**Why This Output Occurs:** The `'linear'` method interpolates between adjacent values, giving 2.4 for the 35th percentile (since the position 1.4 falls between index 1 and index 2). The `'lower'` method returns the value at index 1 (2), the `'higher'` method returns the value at index 2 (3), the `'midpoint'` method averages them (2.5), and the `'nearest'` method rounds to the nearest index (2).

### Real-World Cases

- **Education:** Reporting test scores in percentiles (e.g., "90th percentile") to compare students nationally.
- **Growth Charts:** Pediatricians use percentiles to track children's height and weight against population norms.
- **Finance:** Value-at-Risk (VaR) uses percentiles to estimate potential losses.
- **Quality Control:** Percentiles define acceptable ranges for manufacturing tolerances.

---

## Core Concept 5: Missing Data Handling — Robust Alternatives (`np.nanmean`, `np.nanstd`, `np.nanmedian`, etc.)

### Definitions

**Core Definition:** NaN-aware functions compute descriptive statistics while ignoring `NaN` (Not a Number) values. This enables analysis of datasets with missing or invalid entries without removing entire rows or columns.

**Technical Definition:** NumPy provides `nan`-prefixed variants of its statistical functions: `np.nanmean`, `np.nanmedian`, `np.nanstd`, `np.nanvar`, `np.nanmin`, `np.nanmax`, `np.nanargmin`, `np.nanargmax`, `np.nanpercentile`, and `np.nanquantile`. These functions compute the statistic over the non-NaN elements along the specified axis. For slices where all elements are NaN, the result is NaN, and a `RuntimeWarning` is raised. The number of non-NaN elements is used as the denominator for mean and variance calculations.

**Beginner-Friendly Explanation:** Real-world data often has missing values (represented as `NaN`). If you use regular `np.mean()` on data with `NaN`, the result is `NaN` — one missing value ruins the whole calculation. The `nan`-prefixed functions ignore the missing values and compute the statistic using only the available data. This is like averaging test scores while skipping absent students.

### Purposes

- To compute statistics on datasets with missing values without deleting rows or columns.
- To preserve as much data as possible when analyzing incomplete datasets.
- To implement robust data cleaning pipelines that handle `NaN` gracefully.
- To compute statistics on slices that may be entirely `NaN` (returning `NaN`).
- To avoid the propagation of `NaN` through statistical computations.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import numpy as np

# NaN-aware mean
nan_mean = np.nanmean(a, axis=None, dtype=None, out=None, keepdims=False, *, where=)

# NaN-aware median
nan_median = np.nanmedian(a, axis=None, out=None, overwrite_input=False, keepdims=False)

# NaN-aware standard deviation
nan_std = np.nanstd(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)

# NaN-aware variance
nan_var = np.nanvar(a, axis=None, dtype=None, out=None, ddof=0, keepdims=False, *, where=, mean=, correction=)

# NaN-aware min/max/argmin/argmax
nan_min = np.nanmin(a, axis=None, out=None, keepdims=False, initial=, where=)
nan_max = np.nanmax(a, axis=None, out=None, keepdims=False, initial=, where=)
nan_argmin = np.nanargmin(a, axis=None, out=None, keepdims=False)
nan_argmax = np.nanargmax(a, axis=None, out=None, keepdims=False)

# NaN-aware percentile/quantile
nan_p = np.nanpercentile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False)
nan_q = np.nanquantile(a, q, axis=None, out=None, overwrite_input=False, method='linear', keepdims=False)
```

**Component Breakdown:**
- `a`: Input array, possibly containing `NaN` values.
- `axis`: Axis or axes along which the statistic is computed.
- `ddof`: Delta degrees of freedom (for `nanstd` and `nanvar`).
- `keepdims`: If `True`, retains reduced dimensions as size-1.
- `method`: Interpolation method (for `nanpercentile` and `nanquantile`).

**Syntax Rules:**
- All `nan` functions ignore `NaN` values in the computation.
- For slices that contain only `NaN`, the result is `NaN` (and a `RuntimeWarning` is raised).
- The denominator for `nanmean` is the number of non-NaN elements.
- `nanargmin` and `nanargmax` return the index of the minimum/maximum non-NaN value.

**Constraints and Limitations:**
- If all values in a slice are `NaN`, the result is `NaN`.
- `nanargmin`/`nanargmax` raise `ValueError` if all values are `NaN`.
- `nan` functions may be slower than their regular counterparts due to the masking operation.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Comparing Regular vs. NaN-Aware Functions

```python
import numpy as np
import warnings

# Step 1: Create an array with NaN values
data = np.array([12.0, 15.0, np.nan, 20.0, 22.0, np.nan, 28.0, 30.0])
print(f"Data: {data}")
print(f"Number of non-NaN values: {np.sum(~np.isnan(data))}")

# Step 2: Regular functions propagate NaN
print(f"\n=== Regular Functions (NaN propagates) ===")
print(f"np.mean:   {np.mean(data)}")
print(f"np.median: {np.median(data)}")
print(f"np.std:    {np.std(data)}")
print(f"np.min:    {np.min(data)}")
print(f"np.max:    {np.max(data)}")

# Step 3: NaN-aware functions ignore NaN
print(f"\n=== NaN-Aware Functions (NaN ignored) ===")
with warnings.catch_warnings():
    warnings.simplefilter("ignore", RuntimeWarning)
    print(f"np.nanmean:   {np.nanmean(data):.4f}")
    print(f"np.nanmedian: {np.nanmedian(data):.4f}")
    print(f"np.nanstd:    {np.nanstd(data):.4f}")
    print(f"np.nanmin:    {np.nanmin(data):.4f}")
    print(f"np.nanmax:    {np.nanmax(data):.4f}")
    print(f"np.nanvar:    {np.nanvar(data):.4f}")

# Step 4: Verify by manually excluding NaN
clean_data = data[~np.isnan(data)]
print(f"\nClean data: {clean_data}")
print(f"np.mean(clean_data): {np.mean(clean_data):.4f}")
print(f"np.nanmean matches: {np.isclose(np.nanmean(data), np.mean(clean_data))}")

# Step 5: 2-D array with NaN along an axis
matrix = np.array([[1.0, 2.0, np.nan],
                   [4.0, np.nan, 6.0],
                   [7.0, 8.0, 9.0]])
print(f"\nMatrix with NaN:\n{matrix}")

with warnings.catch_warnings():
    warnings.simplefilter("ignore", RuntimeWarning)
    print(f"\nnp.nanmean (axis=0): {np.nanmean(matrix, axis=0)}")
    print(f"np.nanmean (axis=1): {np.nanmean(matrix, axis=1)}")
    print(f"np.nanmedian (axis=0): {np.nanmedian(matrix, axis=0)}")
    print(f"np.nanstd (axis=1): {np.nanstd(matrix, axis=1)}")
```

**Expected Output:**
```
Data: [12. 15. nan 20. 22. nan 28. 30.]
Number of non-NaN values: 6

=== Regular Functions (NaN propagates) ===
np.mean:   nan
np.median: nan
np.std:    nan
np.min:    nan
np.max:    nan

=== NaN-Aware Functions (NaN ignored) ===
np.nanmean:   21.1667
np.nanmedian: 21.0000
np.nanstd:    6.3470
np.nanmin:    12.0000
np.nanmax:    30.0000
np.nanvar:    40.2847

Clean data: [12. 15. 20. 22. 28. 30.]
np.mean(clean_data): 21.1667
np.nanmean matches: True

Matrix with NaN:
[[ 1.  2. nan]
 [ 4. nan  6.]
 [ 7.  8.  9.]]

np.nanmean (axis=0): [4. 5. 7.5]
np.nanmean (axis=1): [1.5 5.  8. ]
np.nanmedian (axis=0): [4. 5. 7.5]
np.nanstd (axis=1): [0.7071 1.4142 1.     ]
```

**Why This Output Occurs:** 
- Regular functions return `NaN` because `NaN` propagates through arithmetic operations. 
- NaN-aware functions ignore the `NaN` values and compute the statistic using only the 6 non-NaN values. 
- For the 2-D matrix, `np.nanmean(axis=0)` computes the mean down each column, ignoring `NaN` in the first two columns. 
- The `np.nanstd(axis=1)` computes the standard deviation across each row, ignoring `NaN` in the first two rows.

#### Example 2: Handling All-NaN Slices

```python
import numpy as np
import warnings

# Step 1: Create an array with an all-NaN slice
data = np.array([[1.0, 2.0, 3.0],
                 [np.nan, np.nan, np.nan],
                 [4.0, 5.0, 6.0]])
print(f"Matrix:\n{data}")
print("(Row 1 contains all NaN values.)")

# Step 2: NaN-aware functions on all-NaN slice
with warnings.catch_warnings(record=True) as w:
    warnings.simplefilter("always")
    row_means = np.nanmean(data, axis=1)
    print(f"\nnp.nanmean (axis=1): {row_means}")
    print(f"Warning raised: {len(w) > 0}")
    if w:
        print(f"Warning message: {w[0].message}")

# Step 3: Check for NaN in results
print(f"\nIs NaN in result: {np.isnan(row_means).any()}")
print(f"NaN index: {np.where(np.isnan(row_means))[0]}")

# Step 4: Handle all-NaN slices with a fallback
def safe_nanmean(arr, axis=None, fill_value=0.0):
    """Compute nanmean, replacing all-NaN results with fill_value."""
    with warnings.catch_warnings():
        warnings.simplefilter("ignore", RuntimeWarning)
        result = np.nanmean(arr, axis=axis)
    if axis is None:
        return result if not np.isnan(result) else fill_value
    return np.where(np.isnan(result), fill_value, result)

safe_means = safe_nanmean(data, axis=1, fill_value=0.0)
print(f"\nSafe nanmean (fill_value=0.0): {safe_means}")

# Step 5: nanargmin/nanargmax behavior with all-NaN
try:
    np.nanargmin(np.array([np.nan, np.nan, np.nan]))
except ValueError as e:
    print(f"\nnanargmin on all-NaN array raises: {e}")

# Step 6: nanpercentile with all-NaN slice
with warnings.catch_warnings():
    warnings.simplefilter("ignore", RuntimeWarning)
    p50 = np.nanpercentile(data, 50, axis=1)
print(f"\nnanpercentile(50, axis=1): {p50}")
```

**Expected Output:**
```
Matrix:
[[ 1.  2.  3.]
 [nan nan nan]
 [ 4.  5.  6.]]
(Row 1 contains all NaN values.)

np.nanmean (axis=1): [2.  nan 5.]
Warning raised: True
Warning message: Mean of empty slice

Is NaN in result: True
NaN index: [1]

Safe nanmean (fill_value=0.0): [2. 0. 5.]

nanargmin on all-NaN array raises: All-NaN slice encountered

nanpercentile(50, axis=1): [2.  nan 5.]
```

**Why This Output Occurs:** 
- The second row contains only `NaN` values, so `np.nanmean` returns `NaN` for that row and issues a `RuntimeWarning` ("Mean of empty slice"). 
- The custom `safe_nanmean` function replaces the `NaN` result with a fallback value. 
- `np.nanargmin` raises a `ValueError` when all values are `NaN` because there is no minimum to find. 
- `np.nanpercentile` also returns `NaN` for the all-NaN slice.

### Real-World Cases

- **Survey Data:** Handling missing responses in questionnaires without discarding entire respondents.
- **Sensor Networks:** Dealing with malfunctioning sensors that report `NaN` values.
- **Financial Data:** Handling missing stock prices due to market closures or data gaps.
- **Clinical Trials:** Analyzing patient outcomes when some measurements are missing.
- **Machine Learning:** Preprocessing datasets with missing features before imputation.

---

## References

1. **NumPy Statistics — Official Documentation** — https://numpy.org/doc/stable/reference/routines.statistics.html
2. **numpy.mean — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.mean.html
3. **numpy.median — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.median.html
4. **numpy.var — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.var.html
5. **numpy.std — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.std.html
6. **numpy.percentile — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.percentile.html
7. **numpy.quantile — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.quantile.html
8. **numpy.nanmean — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.nanmean.html
9. **numpy.nanstd — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.nanstd.html
10. **numpy.nanmedian — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.nanmedian.html
11. **numpy.nanpercentile — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.nanpercentile.html
12. **numpy.argmin — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.argmin.html
13. **numpy.argmax — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.argmax.html
14. **numpy.average — Official Documentation** — https://numpy.org/doc/stable/reference/generated/numpy.average.html
15. **Python Statistics Fundamentals (Real Python)** — https://realpython.com/python-statistics/
16. **Hyndman, R. J., & Fan, Y. (1996). Sample Quantiles in Statistical Packages. The American Statistician, 50(4), 361–365** — Referenced by NumPy's percentile/quantile documentation.
17. **IEEE 754 Floating-Point Standard** — https://standards.ieee.org/standard/754-2019.html