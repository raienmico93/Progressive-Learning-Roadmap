# Pandas Creating and Modifying Columns: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Creating and modifying columns in pandas is the process of adding new columns to a DataFrame or transforming existing ones, using vectorized operations, conditional logic, functional derivation, and method chaining.

**Technical Definition:** Column creation and modification in pandas operates on the DataFrame's block manager, where each column is a labeled Series backed by a NumPy array or extension array. New columns are assigned via `df['new_col'] = expression` or `df.assign(new_col=expression)`. Transformations are applied element-wise through vectorized operations (arithmetic operators, NumPy universal functions), conditional selection via `np.where()`/`np.select()`, or functional derivation via `Series.map()`, `Series.apply()`, and `Series.transform()`. Method chaining with `.assign()` enables memory-efficient, readable workflows by returning new DataFrames rather than mutating in place. As of pandas 3.0, Copy-on-Write (CoW) semantics ensure that any modification returns a new object, eliminating the ambiguity that caused the legacy `SettingWithCopyWarning`.

**Beginner-Friendly Explanation:** DataFrames are like spreadsheets. Creating a column means adding a new column of data, either from scratch or by calculating it from existing columns. You can do simple math (like adding two columns together), conditional logic (like labeling rows based on a condition), or complex transformations. pandas is designed to do this efficiently across entire columns at once, rather than row by row.

### Key Characteristics

- **Vectorized by default:** Operations on entire columns are hundreds of times faster than iterating row by row.
- **Label-based alignment:** When combining Series or DataFrames, pandas aligns on index labels, not positions.
- **Broadcasting:** Scalars and lower-dimensional objects are broadcast across higher-dimensional objects automatically.
- **CoW semantics (pandas 3.0):** Modifying a DataFrame always returns a new object; chained assignment no longer works and `SettingWithCopyWarning` has been removed.
- **`assign()` returns a copy:** The `assign()` method always returns a new DataFrame, leaving the original untouched.

### Prerequisites

- Basic Python knowledge (variables, functions, lambda expressions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of NumPy arrays and broadcasting.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Feature Engineering:** Creating new features from existing columns is central to ML pipelines.
- **Data Cleaning:** Transforming columns (e.g., scaling, encoding, normalizing) prepares data for analysis.
- **ETL Pipelines:** Column creation and modification are core steps in extract-transform-load workflows.
- **Time Series Analysis:** Deriving lag, rolling, and calendar features requires column creation.

### Core Concepts / Features

1. Vectorized Operations: Arithmetic, Broadcasting, and Scalars
2. Conditional Column Creation: `np.where()` and `np.select()`
3. Functional Derivation: `map()`, `apply()`, and Custom Expressions
4. Method Chaining: `df.assign()` Workflows
5. Data Integrity: Understanding and Avoiding `SettingWithCopyWarning`


## 1. Vectorized Operations: Arithmetic, Broadcasting, and Scalars

### Definitions

**Core Definition:** Vectorized operations apply arithmetic or logical operations to entire columns at once, using optimized C-level loops, rather than iterating row by row in Python.

**Technical Definition:** pandas leverages NumPy's vectorized operations under the hood. When you write `df['C'] = df['A'] + df['B']`, pandas aligns the two Series by index label and performs element-wise addition using NumPy's `add` ufunc. Scalar operations (e.g., `df['A'] * 2`) broadcast the scalar across all elements. Between a DataFrame and a Series, pandas aligns the Series index with the DataFrame columns and broadcasts across rows by default. The arithmetic methods `add()`, `sub()`, `mul()`, `div()` and their `r`-prefixed counterparts provide explicit control over broadcasting via the `axis` parameter.

**Beginner-Friendly Explanation:** Instead of writing a loop to add two columns together row by row, you just write `df['C'] = df['A'] + df['B']`. pandas handles the entire column at once, which is much faster and less error-prone.

### Purposes

- To perform arithmetic operations on entire columns without explicit loops.
- To broadcast scalar values across all elements of a column or DataFrame.
- To combine columns using element-wise operations with automatic index alignment.
- To leverage NumPy's optimized C loops for maximum performance.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Column from arithmetic on two existing columns
df['new_col'] = df['A'] + df['B']
df['new_col'] = df['A'] * df['B']
df['new_col'] = df['A'] / df['B']

# Scalar operations (broadcast)
df['new_col'] = df['A'] * 2
df['new_col'] = df['A'] + 100
df['new_col'] = df['A'] ** 2

# Explicit arithmetic methods with axis control
df['new_col'] = df['A'].add(df['B'])
df['new_col'] = df['A'].sub(df['B'])
df['new_col'] = df['A'].mul(df['B'], fill_value=0)

# Broadcasting a Series across a DataFrame
df['new_col'] = df['A'] + pd.Series([1, 2], index=['x', 'y'])
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `df['A'] + df['B']` | Element-wise addition, aligned by index |
| `df['A'] * 2` | Scalar broadcast: every element multiplied by 2 |
| `.add()`, `.sub()`, `.mul()`, `.div()` | Explicit arithmetic methods |
| `fill_value` | Value to use for missing entries during alignment |
| `axis` | `0` or `'index'`: broadcast across rows; `1` or `'columns'`: broadcast across columns |

**Syntax Rules:**

- Arithmetic operators (`+`, `-`, `*`, `/`, `**`) work element-wise on Series and DataFrames.
- Scalars are broadcast to every element.
- When combining Series with different indexes, pandas aligns on labels and introduces `NaN` for non-matching labels.
- The `.add()` method with `fill_value` replaces non-matching entries with the specified value instead of `NaN`.
- Broadcasting a Series across a DataFrame aligns the Series index with the DataFrame columns by default (row-wise broadcast).

**Constraints and Limitations:**

- Mismatched indexes produce `NaN` values; use `fill_value` to handle them.
- Operations between DataFrames with different column sets align on column labels.
- Very large DataFrames may consume significant memory when creating new columns; consider dropping intermediate columns.

### Annotated Code Examples

**Example 1: Basic Arithmetic Column Creation**

```python
import pandas as pd

# Sample DataFrame
df = pd.DataFrame({
    'price'   : [10.0, 25.0, 15.0],
    'quantity': [2   , 4   , 3]
})

# Create a new column from arithmetic
df['total'] = df['price'] * df['quantity']
print(df)

# Expected output:
#    price  quantity  total
# 0   10.0         2   20.0
# 1   25.0         4  100.0
# 2   15.0         3   45.0
```

**Why this output:** `df['price'] * df['quantity']` multiplies the two Series element-wise, producing `20.0`, `100.0`, and `45.0`. The result is assigned to the new column `total`.

**Example 2: Scalar Broadcasting**

```python
# Scalar operations
df['price_with_tax'] = df['price'] * 1.1
df['price_rounded']  = df['price'].round(0).astype(int)

print(df[['price', 'price_with_tax', 'price_rounded']])
# Expected output:
#    price  price_with_tax  price_rounded
# 0   10.0            11.0             10
# 1   25.0            27.5             25
# 2   15.0            16.5             15
```

**Why this output:** The scalar `1.1` is broadcast across every element of `price`. `round(0)` rounds to the nearest integer, and `astype(int)` converts the result to integer dtype.

**Example 3: Series-to-DataFrame Broadcasting with Index Alignment**

```python
# DataFrame with custom column labels
df2 = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6]
})

# Series with matching column labels
adjustment = pd.Series({'A': 10, 'B': 100})

# Broadcasting the Series across the DataFrame
df2['A_adjusted'] = df2['A'] + adjustment['A']
df2['B_adjusted'] = df2['B'] + adjustment['B']

print(df2)
# Expected output:
#    A  B  A_adjusted  B_adjusted
# 0  1  4          11         104
# 1  2  5          12         105
# 2  3  6          13         106
```

**Why this output:** The Series `adjustment` has index labels `'A'` and `'B'`, which match the DataFrame columns. Accessing `adjustment['A']` returns the scalar 10, which is broadcast across column `A`. This is a manual example of alignment-based broadcasting.

### Real-World Cases

- **Financial analysis:** Compute revenue as `price * quantity` across an entire sales DataFrame.
- **Unit conversion:** Convert temperatures from Celsius to Fahrenheit using `df['temp_f'] = df['temp_c'] * 9/5 + 32`.
- **Feature scaling:** Normalize a column by subtracting the mean and dividing by the standard deviation.

### References

- pandas.DataFrame.add — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.add.html
- pandas.Series.add — https://pandas.pydata.org/docs/reference/api/pandas.Series.add.html
- Binary operations — https://pandas.pydata.org/docs/user_guide/basics.html#binary-operation


## 2. Conditional Column Creation: Multi-Branch Conditional Labeling

### Definitions

**Core Definition:** Conditional column creation uses `np.where()` for binary choices and `np.select()` for multiple conditions to assign values to a new column based on logical conditions.

**Technical Definition:** `np.where(condition, x, y)` returns an array with elements from `x` where `condition` is `True` and from `y` otherwise. `np.select(condlist, choicelist, default=0)` evaluates each condition in `condlist` in order, returning the corresponding choice from `choicelist` for the first `True` condition, or `default` if none match. Both functions are vectorized and operate on NumPy arrays, making them significantly faster than `apply()` with a Python function.

**Beginner-Friendly Explanation:** `np.where()` is like an if-else statement for entire columns: “If the value is greater than 100, label it ‘High’, otherwise label it ‘Low’.” `np.select()` handles multiple if-elif-else branches: “If condition 1, use label A; if condition 2, use label B; otherwise use label C.”

### Purposes

- To create a binary label column based on a single condition using `np.where()`.
- To create a multi-class label column based on multiple conditions using `np.select()`.
- To avoid slow row-by-row `apply()` calls for conditional logic.
- To structure complex conditional labeling in a readable, maintainable way.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import numpy as np

# Binary conditional (np.where)
df['label'] = np.where(condition, value_if_true, value_if_false)

# Multi-branch conditional (np.select)
conditions = [cond1, cond2, cond3]
choices = [choice1, choice2, choice3]
df['label'] = np.select(conditions, choices, default=default_value)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | Boolean Series or array |
| `value_if_true` | Scalar or array to use where condition is `True` |
| `value_if_false` | Scalar or array to use where condition is `False` |
| `condlist` | List of boolean Series or arrays |
| `choicelist` | List of values (scalars or arrays) corresponding to each condition |
| `default` | Value to use when no condition matches (default `0`) |

**Syntax Rules:**

- `np.where()` requires exactly three arguments: condition, true-value, false-value.
- `np.select()` evaluates conditions in order; the first `True` condition wins.
- Both functions accept pandas Series and return NumPy arrays.
- Conditions can be combined using `&` (and), `|` (or), and `~` (not) with parentheses around each condition.

**Constraints and Limitations:**

- `np.where()` returns a NumPy array, which may lose the original index; assign back to the DataFrame to preserve index alignment.
- `np.select()` requires at least two conditions and two choices.
- For very complex logic with many conditions, consider `pd.cut()` or `pd.qcut()` for binning numeric data.

### Annotated Code Examples

**Example 1: Binary Conditional with np.where()**

```python
import pandas as pd
import numpy as np

# Sample data
df = pd.DataFrame({
    'score': [85, 42, 91, 67, 78]
})

# Binary classification: pass if score >= 60
df['result'] = np.where(df['score'] >= 60, 'Pass', 'Fail')
print(df)
# Expected output:
#    score result
# 0     85   Pass
# 1     42   Fail
# 2     91   Pass
# 3     67   Pass
# 4     78   Pass
```

**Why this output:** `df['score'] >= 60` returns a boolean Series. `np.where()` selects `'Pass'` where the condition is `True` and `'Fail'` where it is `False`.

**Example 2: Multi-Branch Conditional with np.select()**

```python
# Multi-class classification
conditions = [
    df['score'] >= 90,
    df['score'] >= 75,
    df['score'] >= 60
]
choices = ['A', 'B', 'C']
df['grade'] = np.select(conditions, choices, default='F')
print(df)
# Expected output:
#    score result grade
# 0     85   Pass     B
# 1     42   Fail     F
# 2     91   Pass     A
# 3     67   Pass     C
# 4     78   Pass     B
```

**Why this output:** `np.select()` evaluates conditions in order. Score 85 does not satisfy `>= 90`, so it checks `>= 75` (True) and returns `'B'`. Score 42 satisfies none, so it returns the default `'F'`.

**Example 3: Combining Conditions with np.select()**

```python
# Complex conditions with multiple columns
df2 = pd.DataFrame({
    'age': [25, 45, 65, 35, 55],
    'income': [30000, 80000, 60000, 120000, 50000]
})

conditions = [
    (df2['age'] >= 60) & (df2['income'] >= 50000),
    (df2['age'] >= 40) & (df2['income'] >= 80000),
    (df2['age'] >= 18) & (df2['income'] < 50000)
]
choices = ['Senior-Prime', 'Mid-Prime', 'Junior-Basic']
df2['segment'] = np.select(conditions, choices, default='Other')
print(df2)
# Expected output:
#    age  income       segment
# 0   25   30000  Junior-Basic
# 1   45   80000     Mid-Prime
# 2   65   60000  Senior-Prime
# 3   35  120000         Other
# 4   55   50000         Other
```

**Why this output:** Each condition combines two column comparisons with `&`. The `np.select()` function evaluates them in order and assigns the first matching label. Row 3 (age 35, income 120000) matches none, so it gets the default `'Other'`.

### Real-World Cases

- **Risk scoring:** Assign risk levels (Low, Medium, High) based on credit score and debt-to-income ratio.
- **Customer segmentation:** Label customers as “VIP”, “Regular”, or “New” based on purchase history and recency.
- **Grade assignment:** Convert numerical scores to letter grades using `np.select()`.

### References

- numpy.where — https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.select — https://numpy.org/doc/stable/reference/generated/numpy.select.html
- Indexing and selecting data (np.where) — https://pandas.pydata.org/docs/user_guide/indexing.html


## 3. Functional Derivation: Building Feature Columns via Transformations

### Definitions

**Core Definition:** Functional derivation creates new columns by applying mathematical transforms or custom Python functions to existing columns, using methods such as `Series.map()`, `Series.apply()`, and `Series.transform()`.

**Technical Definition:** `Series.map(arg, na_action=None)` applies a function, dictionary, or Series to each element of a Series. `Series.apply(func, convert_dtype=True, args=(), **kwargs)` applies a function along the Series axis. `Series.transform(func, axis=0, *args, **kwargs)` applies a function and returns an object with the same index. For grouped transformations, `groupby().transform()` applies a function within each group and returns a result aligned with the original index. These methods differ in performance and flexibility: `map()` is fastest for element-wise dictionary lookups; `apply()` is more flexible but slower; `transform()` is designed for group-wise operations that preserve shape.

**Beginner-Friendly Explanation:** Sometimes you need to apply a custom calculation to every value in a column. `map()` is best for simple lookups (e.g., mapping state codes to state names). `apply()` is best for custom functions that operate on each value. `transform()` is best when you want to compute something within groups (e.g., subtract the group mean from each value).

### Purposes

- To apply mathematical transforms (log, square root, exponential) to entire columns.
- To map values using a dictionary (e.g., code-to-label mapping).
- To apply custom Python functions element-wise to a Series.
- To perform group-wise transformations that preserve the original shape.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Mathematical transforms (vectorized)
df['log_col'] = np.log(df['col'])
df['sqrt_col'] = np.sqrt(df['col'])
df['exp_col'] = np.exp(df['col'])

# Dictionary mapping with map()
df['label'] = df['code'].map({1: 'A', 2: 'B', 3: 'C'})

# Custom function with apply()
df['new_col'] = df['col'].apply(lambda x: x ** 2 if x > 0 else 0)

# Group-wise transform
df['group_demeaned'] = df.groupby('group')['value'].transform(lambda x: x - x.mean())
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `np.log()`, `np.sqrt()`, `np.exp()` | NumPy universal functions for mathematical transforms |
| `Series.map(mapping)` | Dict, Series, or function applied element-wise |
| `Series.apply(func)` | Custom function applied element-wise |
| `Series.transform(func)` | Function applied, result aligned with original index |
| `groupby().transform(func)` | Function applied within each group, shape preserved |

**Syntax Rules:**

- `map()` accepts a dictionary, Series, or function.
- `apply()` passes each element (or row/column for DataFrames) to the function.
- `transform()` must return a result with the same length as the input.
- `groupby().transform()` is ideal for operations like group-wise mean centering, z-score normalization, or filling missing values.

**Constraints and Limitations:**

- `apply()` with a Python function is slower than vectorized operations; use NumPy ufuncs when possible.
- `map()` with a dictionary returns `NaN` for unmapped values unless a default is provided via `na_action`.
- `transform()` on a DataFrame applies the function column-wise by default; use `axis=1` for row-wise operations.

### Annotated Code Examples

**Example 1: Mathematical Transforms**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'value': [1, 10, 100, 1000]
})

# Log transform
df['log_value'] = np.log(df['value'])
# Square root transform
df['sqrt_value'] = np.sqrt(df['value'])
print(df)
# Expected output:
#    value  log_value  sqrt_value
# 0      1   0.000000    1.000000
# 1     10   2.302585    3.162278
# 2    100   4.605170   10.000000
# 3   1000   6.907755   31.622777
```

**Why this output:** `np.log()` computes the natural logarithm (base e), and `np.sqrt()` computes the square root. Both are vectorized NumPy operations applied to the entire column at once.

**Example 2: Dictionary Mapping with map()**

```python
# Map numeric codes to labels
df2 = pd.DataFrame({
    'status_code': [1, 2, 3, 1, 2]
})

status_map = {1: 'Active', 2: 'Inactive', 3: 'Pending'}
df2['status_label'] = df2['status_code'].map(status_map)
print(df2)
# Expected output:
#    status_code status_label
# 0            1       Active
# 1            2     Inactive
# 2            3      Pending
# 3            1       Active
# 4            2     Inactive
```

**Why this output:** `.map(status_map)` looks up each value in the dictionary and returns the corresponding label. Values not in the dictionary would become `NaN`.

**Example 3: Group-Wise Transform**

```python
# Group-wise mean centering
df3 = pd.DataFrame({
    'group': ['A', 'A', 'A', 'B', 'B', 'B'],
    'value': [10, 20, 30, 100, 200, 300]
})

df3['group_mean'] = df3.groupby('group')['value'].transform('mean')
df3['demeaned'] = df3['value'] - df3['group_mean']
print(df3)
# Expected output:
#   group  value  group_mean  demeaned
# 0     A     10        20.0     -10.0
# 1     A     20        20.0       0.0
# 2     A     30        20.0      10.0
# 3     B    100       200.0    -100.0
# 4     B    200       200.0       0.0
# 5     B    300       200.0     100.0
```

**Why this output:** `groupby('group')['value'].transform('mean')` computes the mean for each group and broadcasts it back to the original rows. Subtracting the group mean from each value yields the demeaned values.

### Real-World Cases

- **Feature engineering:** Create log-transformed features for skewed data, or polynomial features for regression models.
- **Data encoding:** Map categorical codes to human-readable labels using `.map()`.
- **Group-wise normalization:** Subtract group means or divide by group standard deviations for hierarchical data.

### References

- pandas.Series.map — https://pandas.pydata.org/docs/reference/api/pandas.Series.map.html
- pandas.Series.apply — https://pandas.pydata.org/docs/reference/api/pandas.Series.apply.html
- pandas.Series.transform — https://pandas.pydata.org/docs/reference/api/pandas.Series.transform.html
- pandas.core.groupby.DataFrameGroupBy.transform — https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.DataFrameGroupBy.transform.html


## 4. Method Chaining: Clean, Memory-Efficient Workflows with df.assign()

### Definitions

**Core Definition:** Method chaining with `df.assign()` creates new columns within a single, readable chain of operations, returning a new DataFrame at each step without modifying the original.

**Technical Definition:** `DataFrame.assign(**kwargs)` returns a new DataFrame with all original columns plus the newly assigned columns. Column names are provided as keyword arguments. If the value is callable, it is computed on the DataFrame and assigned to the new column. If the value is not callable (e.g., a Series, scalar, or array), it is simply assigned. Later items in `**kwargs` may refer to newly created or modified columns in the same `assign()` call, enabling dependent column creation. `assign()` always returns a copy, leaving the original DataFrame untouched.

**Beginner-Friendly Explanation:** `assign()` lets you add multiple columns in one go, and you can chain it with other pandas methods to build a clean, readable pipeline. It never modifies the original DataFrame, so your data stays safe.

### Purposes

- To create multiple new columns in a single, readable statement.
- To chain column creation with filtering, grouping, and other operations in a fluent workflow.
- To avoid intermediate variables and mutating the original DataFrame.
- To enable dependent column creation where one column is derived from another created earlier in the same call.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Single column
df.assign(new_col=expression)

# Multiple columns
df.assign(col1=expr1, col2=expr2, ...)

# Using callable (lambda) to reference the DataFrame
df.assign(new_col=lambda x: x['A'] + x['B'])

# Dependent column creation (later columns can reference earlier ones)
df.assign(
    col1=lambda x: x['A'] * 2,
    col2=lambda x: x['col1'] + 10
)

# Method chaining
(df
 .assign(new_col=lambda x: x['A'] + x['B'])
 .query('new_col > 100')
 .groupby('category')['new_col'].mean()
)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `**kwargs` | Keyword arguments where keys are column names and values are expressions or callables |
| `lambda x: ...` | Callable that receives the DataFrame as `x` |
| `x['col_name']` | Reference to an existing or newly created column within the same `assign()` |
| Returns | A new DataFrame with the new columns added |

**Syntax Rules:**

- Column names must be valid Python identifiers (no spaces or special characters) unless passed as a dictionary.
- Callables receive the DataFrame as their first argument.
- Later columns in the same `assign()` can reference columns created earlier in the same call.
- `assign()` always returns a new object; the original DataFrame is never modified.
- The `pd.col()` expression API (pandas 3.0+) can be used in place of lambdas: `df.assign(c=pd.col("a") + pd.col("b"))`.

**Constraints and Limitations:**

- Column names with spaces or special characters cannot be passed as keyword arguments; use dictionary unpacking (`df.assign(**{'my col': ...})`).
- `assign()` creates a copy of the DataFrame, which may be memory-intensive for very large DataFrames.
- Chaining too many operations can make debugging harder; break long chains into logical steps.

### Annotated Code Examples

**Example 1: Basic assign() with Lambda**

```python
import pandas as pd

df = pd.DataFrame({
    'temp_c': [17.0, 25.0]
}, index=['Portland', 'Berkeley'])

# Assign a new column using a lambda
df2 = df.assign(temp_f=lambda x: x.temp_c * 9 / 5 + 32)
print(df2)
# Expected output:
#           temp_c  temp_f
# Portland    17.0    62.6
# Berkeley    25.0    77.0

# Original DataFrame is unchanged
print(df)
# Expected output:
#           temp_c
# Portland    17.0
# Berkeley    25.0
```

**Why this output:** The lambda receives the DataFrame as `x` and computes `temp_c * 9/5 + 32`. The result is assigned to `temp_f`. The original `df` remains unchanged because `assign()` returns a new object.

**Example 2: Dependent Column Creation**

```python
# Create multiple columns where later columns depend on earlier ones
df3 = df.assign(
    temp_f=lambda x: x['temp_c'] * 9 / 5 + 32,
    temp_k=lambda x: (x['temp_f'] + 459.67) * 5 / 9
)
print(df3)
# Expected output:
#           temp_c  temp_f  temp_k
# Portland    17.0    62.6  290.15
# Berkeley    25.0    77.0  298.15
```

**Why this output:** The `temp_k` lambda references `x['temp_f']`, which was created earlier in the same `assign()` call. pandas evaluates the kwargs in order, so `temp_f` is available when `temp_k` is computed.

**Example 3: Method Chaining with assign()**

```python
# Sample sales data
sales = pd.DataFrame({
    'product': ['A', 'B', 'A', 'B', 'A'],
    'unit_price': [10.0, 20.0, 15.0, 25.0, 12.0],
    'quantity': [5, 3, 8, 2, 6]
})

# Chain: assign total, filter, group
result = (sales
    .assign(total=lambda x: x['unit_price'] * x['quantity'])
    .query('total > 60')
    .groupby('product')['total'].sum()
)
print(result)
# Expected output:
# product
# A    156.0
# B     50.0
# Name: total, dtype: float64
```

**Why this output:** The chain assigns a `total` column, filters to rows where `total > 60`, then groups by `product` and sums `total`. Each step returns a new object, and the chain reads top-to-bottom.

### Real-World Cases

- **Feature engineering pipelines:** Create multiple derived features (ratios, differences, log transforms) in a single `assign()` call.
- **ETL workflows:** Chain `assign()` with `query()`, `groupby()`, and `sort_values()` to build a complete transformation pipeline.
- **Reproducible analysis:** Method chaining makes data transformation steps explicit and readable, improving code review and reproducibility.

### References

- pandas.DataFrame.assign — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.assign.html
- pandas.col — https://pandas.pydata.org/docs/reference/api/pandas.col.html
- Assigning new columns in method chains — https://pandas.pydata.org/docs/user_guide/dsintro.html#assigning-new-columns-in-method-chains


## 5. Data Integrity: Understanding and Avoiding SettingWithCopyWarning

### Definitions

**Core Definition:** `SettingWithCopyWarning` was a pandas warning raised when a value was assigned to a DataFrame slice that might be a copy rather than a view, potentially leaving the original DataFrame unmodified. In pandas 3.0, this warning has been removed due to the adoption of Copy-on-Write semantics.

**Technical Definition:** `SettingWithCopyWarning` was raised when pandas detected chained assignment—an operation like `df[df['A'] > 2]['B'] = 0` where the intermediate object (`df[df['A'] > 2]`) might be a view or a copy, making the assignment ambiguous. The `pd.options.mode.chained_assignment` option controlled the behaviour: `'warn'` (default), `'raise'`, or `None`. With Copy-on-Write (CoW) as the default in pandas 3.0, every operation returns a new object, and chained assignment never modifies the original DataFrame. The `SettingWithCopyWarning` has been removed entirely, and defensive `.copy()` calls are no longer necessary.

**Beginner-Friendly Explanation:** In older pandas, modifying a slice of a DataFrame sometimes worked and sometimes didn't, and pandas warned you about the ambiguity. In pandas 3.0, this is no longer a problem: every operation returns a new DataFrame, so the original is never accidentally modified. You don't need to worry about `SettingWithCopyWarning` anymore.

### Purposes

- To understand why chained assignment was problematic in pandas 2.x and earlier.
- To adopt CoW-compatible coding patterns that work correctly in pandas 3.0.
- To recognise deprecated patterns that relied on chained assignment.
- To write code that is forward-compatible with pandas 3.0's CoW semantics.

### Syntax Rules and Structure

**Deprecated Pattern (pandas 2.x and earlier):**

```python
# Chained assignment — ambiguous behaviour, now deprecated
df[df['A'] > 2]['B'] = 0  # May not modify df

# Correct pattern in pandas 2.x: use .loc
df.loc[df['A'] > 2, 'B'] = 0
```

**CoW-Compatible Pattern (pandas 3.0+):**

```python
# Assign a new column (always returns a new object)
df = df.assign(B=0)

# Or, if modifying in place is intended:
df.loc[df['A'] > 2, 'B'] = 0

# Boolean masking with .loc
df.loc[df['A'] > 2, 'B'] = df['A'] * 2

# Using .copy() explicitly if a standalone copy is needed
df_subset = df[df['A'] > 2].copy()
df_subset['B'] = 0
```

**Component Breakdown:**

| Pattern | Status |
|---------|--------|
| `df[df['A'] > 2]['B'] = 0` | Deprecated; ambiguous in pandas 2.x, never works in pandas 3.0 |
| `df.loc[df['A'] > 2, 'B'] = 0` | Recommended; explicit and unambiguous |
| `df = df.assign(B=...)` | Recommended; returns a new object |
| `df_subset = df[...].copy()` | Defensive copy; no longer necessary in pandas 3.0 but harmless |

**Syntax Rules:**

- In pandas 3.0, all operations that modify data return new objects; the original is never modified.
- Chained assignment (`df[...][...] = value`) never modifies the original DataFrame.
- Use `df.loc[row_indexer, col_indexer] = value` for explicit modification.
- Use `df.assign()` to create new columns without modifying the original.
- The `pd.options.mode.chained_assignment` option and `SettingWithCopyWarning` are removed in pandas 3.0.

**Constraints and Limitations:**

- **Version-specific:** The removal of `SettingWithCopyWarning` applies to pandas 3.0+. In pandas 2.x, the warning still exists and CoW can be enabled with `pd.options.mode.copy_on_write = True`.
- Code that relied on chained assignment modifying the original DataFrame will silently fail to do so in pandas 3.0; this is intentional.
- Defensive `.copy()` calls are no longer necessary to suppress warnings but may still be used for clarity.

### Annotated Code Examples

**Example 1: The Deprecated Chained Assignment Pattern**

```python
import pandas as pd

df = pd.DataFrame({
    'A': [1, 2, 3, 4],
    'B': [10, 20, 30, 40]
})

# Deprecated chained assignment (pandas 2.x with default options)
# This would emit SettingWithCopyWarning
df[df['A'] > 2]['B'] = 0
print(df)
# Expected output (pandas 2.x): the original may not be modified
# Expected output (pandas 3.0): the original is never modified
#    A   B
# 0  1  10
# 1  2  20
# 2  3  30
# 3  4  40
```

**Why this output:** The expression `df[df['A'] > 2]` creates a new object. Assigning to `['B']` on that temporary object modifies the temporary, not the original `df`. In pandas 2.x, this ambiguity triggered `SettingWithCopyWarning`. In pandas 3.0, the behaviour is consistent: the original is never modified.

**Example 2: The Recommended .loc Pattern**

```python
# Correct pattern: use .loc for explicit assignment
df.loc[df['A'] > 2, 'B'] = 0
print(df)
# Expected output:
#    A   B
# 0  1  10
# 1  2  20
# 2  3   0
# 3  4   0
```

**Why this output:** `.loc[row_indexer, col_indexer]` explicitly selects the rows and columns to modify. This pattern works correctly in both pandas 2.x and 3.0.

**Example 3: CoW-Compatible Pattern with assign()**

```python
# CoW-compatible: create a new DataFrame with a modified column
df_new = df.assign(B=lambda x: x['B'].where(x['A'] <= 2, 0))
print(df_new)
# Expected output:
#    A   B
# 0  1  10
# 1  2  20
# 2  3   0
# 3  4   0

# Original is unchanged
print(df)
# Expected output:
#    A   B
# 0  1  10
# 1  2  20
# 2  3  30
# 3  4  40
```

**Why this output:** `assign()` creates a new DataFrame where column `B` is replaced using `where()`. The original `df` remains unchanged because `assign()` never modifies in place. This pattern is fully compatible with pandas 3.0's CoW semantics.

### Real-World Cases

- **Data cleaning pipelines:** Use `.loc` for explicit row-and-column modifications to avoid ambiguity.
- **Feature engineering:** Use `assign()` to create new columns without mutating the original DataFrame.
- **Migration to pandas 3.0:** Audit codebases for chained assignment patterns and replace them with `.loc` or `assign()`.

### References

- Copy-on-Write (CoW) — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- pandas 3.0 release notes — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html
- Indexing and selecting data — https://pandas.pydata.org/docs/user_guide/indexing.html
- PDEP-7: Consistent copy/view semantics in pandas with Copy-on-Write — https://pandas.pydata.org/pdeps/0007-copy-on-write.html


## References

- pandas.DataFrame.assign — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.assign.html
- pandas.DataFrame.add — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.add.html
- pandas.Series.map — https://pandas.pydata.org/docs/reference/api/pandas.Series.map.html
- pandas.Series.apply — https://pandas.pydata.org/docs/reference/api/pandas.Series.apply.html
- pandas.Series.transform — https://pandas.pydata.org/docs/reference/api/pandas.Series.transform.html
- pandas.core.groupby.DataFrameGroupBy.transform — https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.DataFrameGroupBy.transform.html
- numpy.where — https://numpy.org/doc/stable/reference/generated/numpy.where.html
- numpy.select — https://numpy.org/doc/stable/reference/generated/numpy.select.html
- Copy-on-Write (CoW) — https://pandas.pydata.org/docs/user_guide/copy_on_write.html
- pandas 3.0 release notes — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html
- PDEP-7: Consistent copy/view semantics in pandas with Copy-on-Write — https://pandas.pydata.org/pdeps/0007-copy-on-write.html
- Indexing and selecting data — https://pandas.pydata.org/docs/user_guide/indexing.html
- Binary operations — https://pandas.pydata.org/docs/user_guide/basics.html#binary-operation
- Assigning new columns in method chains — https://pandas.pydata.org/docs/user_guide/dsintro.html#assigning-new-columns-in-method-chains