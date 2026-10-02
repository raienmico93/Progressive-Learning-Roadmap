# Pandas Categorical Data: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Categorical data is a pandas data type that represents variables which can take on only a limited, and usually fixed, number of possible values called **categories**.

**Technical Definition:** `pandas.Categorical` is a pandas extension array type backed by two NumPy arrays: a `categories` array (an `Index` of unique, non-null values) and an integer `codes` array where each code is the positional index of the corresponding value in `categories`. The `CategoricalDtype` class wraps these two attributes (`categories` and `ordered`) into a dtype object that can be used anywhere pandas expects a dtype.

**Beginner-Friendly Explanation:** Think of a categorical column as a multiple-choice question: there are only a few possible answers (e.g., “Red”, “Green”, “Blue”), and each row picks one. Instead of storing the full text “Red” a million times, pandas stores the unique answers once and uses small integer codes for the rest, saving memory and making operations like grouping and sorting much faster.

### Key Characteristics

- **Finite, fixed set of values:** All values in a categorical are either in `categories` or `np.nan`. No other values are allowed.
- **Order is defined by category order, not lexical order:** You can define a logical order (e.g., “Low” < “Medium” < “High”) that differs from alphabetical order.
- **Numerical operations are not possible:** Addition, division, and similar operations raise `TypeError` on categorical data.
- **Memory proportional to categories + data length:** For low-cardinality columns, categoricals use dramatically less memory than `object` dtype.
- **`observed` parameter in groupby:** Controls whether unobserved categories appear in results; default is changing from `False` to `True` in a future pandas version.

### Prerequisites

- Basic Python knowledge (lists, strings, dictionaries).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of pandas dtypes and the `astype()` method.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Analysis & Statistics:** Categoricals map directly to statistical categorical variables (nominal and ordinal scales).
- **Memory Optimization:** Categorical encoding reduces memory footprint for repetitive string columns by up to 95%.
- **Performance Engineering:** Groupby, sorting, and filtering operations run significantly faster on categorical columns.
- **Machine Learning:** Categoricals signal to libraries (e.g., plotting, scikit-learn) that a variable should be treated as categorical for encoding or visualization.

### Core Concepts / Features

1. Instantiation: Creating Categoricals
2. Structural Ordering: Ordered vs. Unordered Categories
3. Computational Profiling: Memory and Speed Optimization
4. Advanced Interactions: Grouping by Categorical Columns
5. Categorical Management: The `.cat` Accessor


## 1. Instantiation: Creating Categoricals

### Definitions

**Core Definition:** Categoricals can be instantiated in three primary ways: directly via `pd.Categorical()`, by casting an existing Series with `astype('category')`, or by specifying a `pd.CategoricalDtype()` as the dtype parameter.

**Technical Definition:** `pd.Categorical(values, categories=None, ordered=None, dtype=None, copy=True)` constructs a `Categorical` object from array-like data. `Series.astype('category')` converts an existing Series in-place (returning a new object). `pd.CategoricalDtype(categories=None, ordered=False)` creates a reusable dtype object that defines the categories and ordering independently of the values.

**Beginner-Friendly Explanation:** There are three ways to make a categorical column: you can create one from scratch with `pd.Categorical()`, convert an existing column with `.astype('category')`, or define a “recipe” (CategoricalDtype) first and apply it to data.

### Purposes

- To create a categorical Series from a list, array, or existing Series.
- To convert an existing column in a DataFrame to categorical dtype.
- To define a reusable dtype with explicit categories and ordering for consistent application across multiple columns or datasets.
- To specify categories that may not all be present in the data.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Direct construction
pd.Categorical(values, categories=None, ordered=None, dtype=None)

# Series construction
pd.Series(data, dtype="category")
pd.Series(data, dtype=pd.CategoricalDtype(categories, ordered))

# Casting existing Series
series.astype("category")
series.astype(pd.CategoricalDtype(categories, ordered))
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `values` | Array-like data to convert |
| `categories` | Optional sequence of unique, non-null values |
| `ordered` | Boolean or `None`; whether categories have a logical order |
| `dtype` | A `CategoricalDtype` instance; cannot be used with `categories` or `ordered` |
| `"category"` | String alias for an unordered categorical with inferred categories |

**Syntax Rules:**

- If `categories` is not provided, it is inferred as the unique values of `values`, sorted lexically.
- All values must appear in `categories`; values not present become `NaN`.
- Categories cannot contain null values.
- When `dtype` is provided, `categories` and `ordered` must not be passed.

**Constraints and Limitations:**

- Categorical is not a NumPy dtype; `np.dtype("category")` raises `TypeError`.
- Categorical data is implemented as a Python object, not a low-level NumPy array, which can cause issues with some NumPy functions.
- Unused categories increase memory usage; use `remove_unused_categories()` to clean up.

### Annotated Code Examples

**Example 1: Three Ways to Create a Categorical Series**

```python
import pandas as pd

# Method 1: Direct construction with pd.Categorical()
cat1 = pd.Categorical(["a", "b", "c", "a"])
print(cat1)
# Expected output:
# ['a', 'b', 'c', 'a']
# Categories (3, str): ['a', 'b', 'c']

# Method 2: Series with dtype="category"
s2 = pd.Series(["a", "b", "c", "a"], dtype="category")
print(s2)
# Expected output:
# 0    a
# 1    b
# 2    c
# 3    a
# dtype: category
# Categories (3, str): ['a', 'b', 'c']

# Method 3: astype('category') on existing Series
s3 = pd.Series(["a", "b", "c", "a"])
s3_cat = s3.astype("category")
print(s3_cat)
# Expected output:
# 0    a
# 1    b
# 2    c
# 3    a
# dtype: category
# Categories (3, str): ['a', 'b', 'c']
```

**Why this output:** All three methods produce the same categorical Series with categories `['a', 'b', 'c']`. The categories are inferred as the unique values in sorted order.

**Example 2: Explicit Categories with CategoricalDtype**

```python
# Define a reusable dtype with explicit categories
cat_type = pd.CategoricalDtype(categories=["Low", "Medium", "High"], ordered=True)

# Apply to a Series containing a subset of categories
s = pd.Series(["Medium", "High", "Low", "Medium"], dtype=cat_type)
print(s)
# Expected output:
# 0    Medium
# 1      High
# 2       Low
# 3    Medium
# dtype: category
# Categories (3, object): ['Low' < 'Medium' < 'High']

# Values not in categories become NaN
s2 = pd.Series(["Low", "Unknown"], dtype=cat_type)
print(s2)
# Expected output:
# 0      Low
# 1      NaN
# dtype: category
# Categories (3, object): ['Low' < 'Medium' < 'High']
```

**Why this output:** The `CategoricalDtype` defines the categories and their order. Values must be in the categories; `"Unknown"` is not, so it becomes `NaN`. The ordered flag enables logical comparison (`Low < Medium < High`).

**Example 3: Using pd.Categorical with Explicit Categories**

```python
# Create with explicit categories that include unobserved values
cat = pd.Categorical(["a", "b", "a"], categories=["a", "b", "c"])
print(cat)
# Expected output:
# ['a', 'b', 'a']
# Categories (3, str): ['a', 'b', 'c']

# The code for 'c' is not used, but 'c' remains a category
print(cat.categories.tolist())  # ['a', 'b', 'c']
print(cat.codes)                # [0, 1, 0]
```

**Why this output:** The `categories` parameter explicitly includes `"c"` even though no value is `"c"`. The codes array only contains 0 and 1, but the categories array still has three entries. This is useful when you know the full set of possible categories in advance.

### Real-World Cases

- **Survey data:** Likert-scale responses (“Strongly Agree” to “Strongly Disagree”) are created as ordered categoricals for correct ordinal analysis.
- **E-commerce:** Product categories (Electronics, Clothing, Food) are converted to categoricals for memory-efficient storage and fast grouping.
- **Time series:** Observation periods (Q1, Q2, Q3, Q4) are defined as ordered categoricals to ensure correct chronological sorting.

### References

- Categorical data — https://pandas.pydata.org/docs/dev/user_guide/categorical.html
- pandas.Categorical — https://pandas.pydata.org/docs/reference/api/pandas.Categorical.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html


## 2. Structural Ordering: Ordered vs. Unordered Categories

### Definitions

**Core Definition:** A categorical is **ordered** if its categories have a meaningful logical sequence; otherwise it is **unordered**. Ordering enables comparison operations like `<`, `>`, `min()`, and `max()`.

**Technical Definition:** The `ordered` attribute of `CategoricalDtype` is a boolean (or `None`). When `True`, the categories array defines a total order, and pandas enables ordering operations (comparison, min, max, sort) based on the position of each category in the categories array. When `False`, such operations raise `TypeError`. The `ordered` parameter defaults to `False` and is not inferred automatically.

**Beginner-Friendly Explanation:** A “Low/Medium/High” column is ordered because “Low” comes before “Medium” before “High”. A “Red/Green/Blue” column is unordered because there is no natural order. Ordered categoricals let you sort logically (Low, Medium, High) instead of alphabetically (High, Low, Medium).

### Purposes

- To enable logical sorting and comparison based on a domain-specific order.
- To compute `min()`, `max()`, and `mode()` on categorical data.
- To ensure that visualizations and reports present categories in the correct logical sequence.
- To distinguish between nominal (unordered) and ordinal (ordered) variables in statistical analysis.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Ordered categorical
pd.Categorical(values, categories=cats, ordered=True)
pd.CategoricalDtype(categories=cats, ordered=True)

# Unordered categorical
pd.Categorical(values, categories=cats, ordered=False)

# Converting between ordered and unordered
series.cat.as_ordered()
series.cat.as_unordered()
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ordered=True` | Enables comparison and min/max based on category order |
| `ordered=False` | Disables ordering operations (default) |
| `.cat.as_ordered()` | Returns a copy with `ordered=True` |
| `.cat.as_unordered()` | Returns a copy with `ordered=False` |

**Syntax Rules:**

- `ordered` is not inferred; you must explicitly pass `ordered=True` to create an ordered categorical.
- Comparisons (`<`, `>`, `<=`, `>=`) between ordered categoricals and scalars work if the scalar is a category.
- `min()` and `max()` on unordered categoricals raise `TypeError`.
- The order of categories is defined by the order of the `categories` array, not by lexical sorting.

**Constraints and Limitations:**

- Comparing two unordered categoricals with the same categories does not consider order.
- An ordered categorical cannot be compared to a differently ordered categorical without explicit conversion.
- Reordering categories changes the logical order but does not reorder the data values.

### Annotated Code Examples

**Example 1: Ordered Categorical in Action**

```python
import pandas as pd

# Define an ordered categorical for sizes
sizes = pd.Categorical(
    ["Medium", "Small", "Large", "Medium"],
    categories=["Small", "Medium", "Large"],
    ordered=True
)
s = pd.Series(sizes)
print(s)
# Expected output:
# 0    Medium
# 1     Small
# 2     Large
# 3    Medium
# dtype: category
# Categories (3, object): ['Small' < 'Medium' < 'Large']

# Sorting uses logical order
print(s.sort_values())
# Expected output:
# 1     Small
# 3    Medium
# 0    Medium
# 2     Large
# dtype: category
# Categories (3, object): ['Small' < 'Medium' < 'Large']

# min and max work on ordered categoricals
print(f"Min: {s.min()}, Max: {s.max()}")
# Expected output: Min: Small, Max: Large
```

**Why this output:** The categories array defines the order `Small < Medium < Large`. `sort_values()` respects this order, placing Small first and Large last. `min()` and `max()` return the first and last categories respectively.

**Example 2: Unordered Categorical Raises on min/max**

```python
# Unordered categorical (default)
s_unordered = pd.Series(["a", "b", "c", "a"], dtype="category")
print(s_unordered.cat.ordered)  # False

# min() raises TypeError
try:
    s_unordered.min()
except TypeError as e:
    print(f"Error: {e}")
# Expected output: Error: Categorical is not ordered for operation min
```

**Why this output:** Without an explicit order, pandas cannot determine which category is “smallest”. `min()` and `max()` require `ordered=True`.

**Example 3: Converting Between Ordered and Unordered**

```python
# Start with unordered
s = pd.Series(["Low", "High", "Medium"], dtype="category")

# Convert to ordered
s_ordered = s.cat.as_ordered()
print(s_ordered.cat.ordered)  # True
print(s_ordered)
# Expected output:
# 0       Low
# 1      High
# 2    Medium
# dtype: category
# Categories (3, object): ['High' < 'Low' < 'Medium']

# Convert back to unordered
s_unordered = s_ordered.cat.as_unordered()
print(s_unordered.cat.ordered)  # False
```

**Why this output:** `as_ordered()` enables ordering using the existing category order (which is lexical because it was inferred). To define a custom order, you must specify `categories` explicitly when creating or reordering.

### Real-World Cases

- **Education levels:** “High School” < “Bachelor” < “Master” < “PhD” requires an ordered categorical for correct analysis.
- **Customer satisfaction:** “Very Dissatisfied” < “Dissatisfied” < “Neutral” < “Satisfied” < “Very Satisfied” is a classic ordered categorical.
- **T-shirt sizes:** XS < S < M < L < XL < XXL requires ordered categories to sort correctly.

### References

- Categorical data (Ordered or not) — https://pandas.pydata.org/docs/user_guide/categorical.html#ordered-or-not
- pandas.CategoricalDtype.ordered — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.ordered.html


## 3. Computational Profiling: Memory Consumption and Execution Speed

### Definitions

**Core Definition:** Computational profiling of categorical data measures the memory footprint and execution speed of categorical dtypes compared to `object` dtype, revealing that categoricals are more memory-efficient for low-cardinality columns and faster for grouping, sorting, and filtering operations.

**Technical Definition:** The memory usage of a `Categorical` is proportional to the number of categories plus the length of the data. In contrast, an `object` dtype is a constant times the length of the data. The `.nbytes` attribute returns the total bytes consumed by the underlying arrays. For speed, categorical operations leverage the integer codes array (which is contiguous and cache-friendly) instead of repeatedly hashing strings.

**Beginner-Friendly Explanation:** Categoricals save memory because they store each unique value only once. They are also faster because pandas works with small integer codes instead of comparing long strings. But if almost every value is unique, categoricals can use more memory than `object`.

### Purposes

- To quantify the memory savings of converting `object` columns to `category`.
- To measure the speed improvement of groupby, sorting, and filtering on categoricals.
- To determine when categorical encoding is beneficial (low cardinality) vs. detrimental (high cardinality).
- To justify dtype conversion decisions in data pipelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Memory measurement
series.memory_usage(deep=True)
dataframe.memory_usage(deep=True).sum()
categorical_series.nbytes

# Structure inspection
series.cat.categories
series.cat.codes

# Conversion
series.astype("category")
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `memory_usage(deep=True)` | Returns actual memory including object overhead |
| `.nbytes` | Total bytes of the underlying arrays |
| `.cat.categories` | The unique category values |
| `.cat.codes` | Integer codes pointing to categories |

**Syntax Rules:**

- `memory_usage(deep=True)` is required for `object` dtype to account for Python string overhead; without it, only pointer sizes are counted.
- The memory of a categorical is `len(codes) * itemsize + len(categories) * itemsize_of_category_dtype`.
- If the number of categories approaches the data length, categorical memory approaches or exceeds `object` memory.

**Constraints and Limitations:**

- Categorical conversion requires a full pass over the data (factorization), which has a one-time cost.
- Very high-cardinality columns (e.g., unique IDs) should not be converted to categorical.
- Categorical data is not a NumPy dtype; some NumPy functions do not work on it.

### Annotated Code Examples

**Example 1: Memory Comparison — Object vs. Category**

```python
import pandas as pd
import numpy as np

# Create a Series with 1 million rows and 3 unique values
np.random.seed(42)
values = np.random.choice(["Red", "Green", "Blue"], size=1_000_000)

# Object dtype
s_obj = pd.Series(values, dtype=object)
obj_mem = s_obj.memory_usage(deep=True)

# Category dtype
s_cat = s_obj.astype("category")
cat_mem = s_cat.memory_usage(deep=True)

print(f"Object memory:  {obj_mem / 1e6:.2f} MB")
print(f"Category memory: {cat_mem / 1e6:.2f} MB")
print(f"Reduction: {(1 - cat_mem / obj_mem) * 100:.1f}%")
# Expected output (approximate):
# Object memory:  56.00 MB
# Category memory: 1.00 MB
# Reduction: 98.2%
```

**Why this output:** The `object` dtype stores a Python string object for every element, each with significant overhead (approximately 56 bytes per element). The categorical stores 1 million 1-byte integer codes plus a small categories array, reducing memory by over 98%.

**Example 2: High-Cardinality Warning**

```python
# Create a Series with 2000 unique values
s_unique = pd.Series([f"value_{i}" for i in range(2000)])

obj_mem = s_unique.memory_usage(deep=True)
cat_mem = s_unique.astype("category").memory_usage(deep=True)

print(f"Object memory:  {obj_mem} bytes")
print(f"Category memory: {cat_mem} bytes")
# Expected output (approximate):
# Object memory:  16000 bytes
# Category memory: 20000 bytes
```

**Why this output:** When every value is unique, the categorical must store all 2000 categories plus 2000 codes, using more memory than the `object` representation. Categoricals are only beneficial when cardinality is low relative to data length.

**Example 3: Speed Comparison — Groupby on Category vs. Object**

```python
import pandas as pd
import numpy as np
import time

# Create a large DataFrame with a low-cardinality column
np.random.seed(42)
n = 5_000_000
df = pd.DataFrame({
    "category_col": np.random.choice(["A", "B", "C", "D"], size=n),
    "value": np.random.randn(n)
})

# Convert to categorical
df["category_col"] = df["category_col"].astype("category")

# Time groupby on categorical
start = time.time()
result_cat = df.groupby("category_col", observed=True)["value"].mean()
cat_time = time.time() - start

# Convert back to object for comparison
df["object_col"] = df["category_col"].astype(object)

# Time groupby on object
start = time.time()
result_obj = df.groupby("object_col")["value"].mean()
obj_time = time.time() - start

print(f"Category groupby time: {cat_time:.3f}s")
print(f"Object groupby time:   {obj_time:.3f}s")
print(f"Speedup: {obj_time / cat_time:.1f}x")
# Expected output (approximate):
# Category groupby time: 0.045s
# Object groupby time:   0.320s
# Speedup: 7.1x
```

**Why this output:** Groupby on categorical uses the integer codes array for grouping, which is contiguous and cache-friendly. Groupby on object requires hashing strings, which is significantly slower. The speedup depends on cardinality and data size.

### Real-World Cases

- **Large CSV datasets:** Converting low-cardinality string columns to categorical after `read_csv` reduces memory from gigabytes to megabytes.
- **Dashboard performance:** Categorical columns speed up interactive filtering and grouping in visualization dashboards.
- **Data pipelines:** Profiling memory usage before and after conversion helps engineers decide which columns to encode.

### References

- Categorical data (Memory usage) — https://pandas.pydata.org/docs/user_guide/categorical.html#memory-usage
- Pandas Categories Optimization — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/library/pandas/14-pandas-advanced-topics/3-pandas-categories-optimization.mdx


## 4. Advanced Interactions: Grouping by Categorical Columns

### Definitions

**Core Definition:** Grouping by a categorical column uses the categories as group keys, with the `observed` parameter controlling whether unobserved categories appear in the result.

**Technical Definition:** When a `Categorical` is used as a grouper in `groupby()`, the `observed` parameter determines the output: `observed=False` returns a Cartesian product of all categories (including those not present in the data), while `observed=True` returns only categories that actually appear. The default of `observed=False` is deprecated and will change to `True` in a future pandas version.

**Beginner-Friendly Explanation:** If you group by “Color” but your data only has “Red” and “Blue” (not “Green”), `observed=False` will show a row for “Green” with zero/NaN values. `observed=True` will hide “Green” because it never appeared. The default is changing from showing everything to showing only what's there.

### Purposes

- To aggregate data by categorical groups while controlling whether empty categories appear.
- To generate complete category combinations for reporting (e.g., ensuring all product categories appear even if sales are zero).
- To improve performance by skipping unobserved categories when they are not needed.
- To handle multiple categorical groupers and their Cartesian product.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
df.groupby("categorical_col", observed=True/False)["value"].agg()
df.groupby(["cat_col1", "cat_col2"], observed=True/False)["value"].sum()
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `observed=True` | Only show categories present in the data |
| `observed=False` | Show all categories (Cartesian product for multiple groupers) |
| Default | `False` but deprecated; will become `True` in future versions |

**Syntax Rules:**

- `observed` only applies when at least one grouper is categorical.
- With multiple categorical groupers and `observed=False`, the result includes all combinations of categories.
- Passing `observed=True` silences the `FutureWarning` and adopts the future default.

**Constraints and Limitations:**

- `observed=False` can produce very large results if there are many categorical groupers with many categories.
- **Deprecated:** The default `observed=False` is deprecated and emits a `FutureWarning`; explicitly pass `observed=True` or `observed=False` to avoid the warning.
- Unobserved categories in the result have `NaN` or zero values depending on the aggregation function.

### Annotated Code Examples

**Example 1: observed=True vs. observed=False**

```python
import pandas as pd

# Create a DataFrame with a categorical column
df = pd.DataFrame({
    "size": pd.Categorical(
        ["Small", "Medium", "Small", "Large"],
        categories=["Small", "Medium", "Large", "X-Large"]
    ),
    "count": [1, 2, 3, 4]
})

# observed=False: includes unobserved "X-Large"
print("observed=False:")
print(df.groupby("size", observed=False)["count"].sum())
# Expected output:
# size
# Small      4
# Medium     2
# Large      4
# X-Large    0
# Name: count, dtype: int64

# observed=True: only observed categories
print("\nobserved=True:")
print(df.groupby("size", observed=True)["count"].sum())
# Expected output:
# size
# Small     4
# Medium    2
# Large     4
# Name: count, dtype: int64
```

**Why this output:** With `observed=False`, the result includes `X-Large` even though it never appears in the data, showing 0 for the sum. With `observed=True`, `X-Large` is omitted entirely.

**Example 2: Multiple Categorical Groupers**

```python
# Two categorical columns
df = pd.DataFrame({
    "size": pd.Categorical(["S", "M", "S"], categories=["S", "M", "L"]),
    "color": pd.Categorical(["Red", "Blue", "Red"], categories=["Red", "Blue", "Green"]),
    "value": [1, 2, 3]
})

# Cartesian product with observed=False
result = df.groupby(["size", "color"], observed=False)["value"].sum()
print(result)
# Expected output:
# size  color
# S     Red      4
#       Blue     0
#       Green    0
# M     Red      0
#       Blue     2
#       Green    0
# L     Red      0
#       Blue     0
#       Green    0
# Name: value, dtype: int64
```

**Why this output:** With `observed=False`, all 9 combinations (3 sizes × 3 colors) are included, even those with no data. This is useful for reports that need to show zero values for missing combinations.

**Example 3: Silencing the FutureWarning**

```python
import warnings

# This will emit a FutureWarning
with warnings.catch_warnings(record=True) as w:
    warnings.simplefilter("always")
    result = df.groupby("size")["value"].sum()
    if w:
        print(f"Warning: {w[0].message}")
# Expected output (approx.): Warning: The default of observed=False is deprecated...

# Explicit observed=True silences the warning
result = df.groupby("size", observed=True)["value"].sum()
print(result)
# No warning emitted
```

**Why this output:** Pandas emits a `FutureWarning` when `observed` is not specified because the default is changing from `False` to `True`. Explicitly passing `observed=True` or `observed=False` silences the warning and future-proofs the code.

### Real-World Cases

- **Sales reporting:** Grouping by product category and region with `observed=False` ensures all category-region combinations appear, even with zero sales.
- **Survey analysis:** Grouping by demographic categories with `observed=True` avoids cluttering results with categories that no respondent selected.
- **Inventory management:** Grouping by warehouse and product with `observed=False` reveals stock gaps (products with zero inventory in a warehouse).

### References

- pandas.DataFrame.groupby (observed parameter) — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.groupby.html
- DEPR: observed=False default in groupby — https://github.com/pandas-dev/pandas/pull/51811
- Group by: split-apply-combine (Categorical groupers) — https://pandas.pydata.org/docs/user_guide/groupby.html


## 5. Categorical Management: The `.cat` Accessor

### Definitions

**Core Definition:** The `.cat` accessor provides methods and properties for inspecting and modifying the categories of a categorical Series, including renaming, reordering, adding, and removing categories.

**Technical Definition:** `Series.cat` is a `CategoricalAccessor` object that delegates to the underlying `Categorical` array. Properties include `categories`, `ordered`, and `codes`. Methods include `rename_categories()`, `reorder_categories()`, `add_categories()`, `remove_categories()`, `remove_unused_categories()`, `set_categories()`, `as_ordered()`, and `as_unordered()`. All methods return new objects by default; `inplace=True` is available for some operations.

**Beginner-Friendly Explanation:** The `.cat` accessor is like a toolbox for managing your categorical column. You can rename the categories, change their order, add new ones, or remove unused ones.

### Purposes

- To inspect the current categories and their order.
- To rename categories without changing the underlying data values.
- To reorder categories to change the logical order.
- To add new categories that may not yet appear in the data.
- To remove categories that are no longer needed or unused.
- To change between ordered and unordered representations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Inspection
series.cat.categories
series.cat.ordered
series.cat.codes

# Modification
series.cat.rename_categories(new_categories, inplace=False)
series.cat.reorder_categories(new_order, ordered=None, inplace=False)
series.cat.add_categories(new_categories, inplace=False)
series.cat.remove_categories(removals, inplace=False)
series.cat.remove_unused_categories(inplace=False)
series.cat.set_categories(new_categories, ordered=None, rename=False, inplace=False)
series.cat.as_ordered()
series.cat.as_unordered()
```

**Component Breakdown:**

| Method/Property | Description |
|-----------------|-------------|
| `.cat.categories` | Returns the Index of categories |
| `.cat.ordered` | Returns whether the categorical is ordered |
| `.cat.codes` | Returns integer codes (−1 for NaN) |
| `.cat.rename_categories()` | Renames categories (values are re-labeled) |
| `.cat.reorder_categories()` | Reorders categories (changes logical order) |
| `.cat.add_categories()` | Adds new categories |
| `.cat.remove_categories()` | Removes categories (matching values become NaN) |
| `.cat.remove_unused_categories()` | Removes categories not present in the data |
| `.cat.set_categories()` | Sets categories in one operation (faster than add/remove) |

**Syntax Rules:**

- **Renaming vs. reordering:** `rename_categories()` changes the labels; `reorder_categories()` changes the order. They are different operations.
- **Adding categories:** New categories are added at the end of the categories array by default.
- **Removing categories:** Values matching removed categories become `NaN`.
- **set_categories:** Can add, remove, and reorder simultaneously; faster than sequential operations.
- **In-place operations:** Assigning to `.cat.categories` is an in-place operation; methods return new objects by default.

**Constraints and Limitations:**

- Categories must be unique and cannot contain null values.
- Removing a category that is present in the data sets those values to `NaN`.
- `reorder_categories()` requires the new order to contain exactly the same categories as the current one.
- The `.cat` accessor is only available on Series with categorical dtype.

### Annotated Code Examples

**Example 1: Inspecting and Renaming Categories**

```python
import pandas as pd

# Create a categorical Series
s = pd.Series(["a", "b", "c", "a"], dtype="category")
print(s.cat.categories.tolist())  # ['a', 'b', 'c']
print(s.cat.ordered)              # False
print(s.cat.codes.tolist())       # [0, 1, 2, 0]

# Rename categories (values are re-labeled)
s_renamed = s.cat.rename_categories(["X", "Y", "Z"])
print(s_renamed)
# Expected output:
# 0    X
# 1    Y
# 2    Z
# 3    X
# dtype: category
# Categories (3, object): ['X', 'Y', 'Z']
```

**Why this output:** `rename_categories()` replaces the category labels without changing the underlying codes. The data values `a`, `b`, `c` become `X`, `Y`, `Z`.

**Example 2: Adding and Removing Categories**

```python
# Add new categories
s_added = s.cat.add_categories(["d", "e"])
print(s_added.cat.categories.tolist())
# Expected output: ['a', 'b', 'c', 'd', 'e']

# Remove categories (matching values become NaN)
s_removed = s.cat.remove_categories(["b"])
print(s_removed)
# Expected output:
# 0      a
# 1    NaN
# 2      c
# 3      a
# dtype: category
# Categories (2, object): ['a', 'c']

# Remove unused categories
s_with_unused = pd.Series(["a", "b"], dtype=pd.CategoricalDtype(["a", "b", "c"]))
s_clean = s_with_unused.cat.remove_unused_categories()
print(s_clean.cat.categories.tolist())
# Expected output: ['a', 'b']
```

**Why this output:** `add_categories()` adds new labels without affecting existing data. `remove_categories()` replaces matching values with `NaN` and removes the category. `remove_unused_categories()` removes categories that do not appear in the data.

**Example 3: Reordering and Setting Categories**

```python
# Reorder categories to define logical order
s = pd.Series(["Low", "High", "Medium"], dtype="category")
s_reordered = s.cat.reorder_categories(["Low", "Medium", "High"], ordered=True)
print(s_reordered)
# Expected output:
# 0       Low
# 1      High
# 2    Medium
# dtype: category
# Categories (3, object): ['Low' < 'Medium' < 'High']

# set_categories: add, remove, and reorder in one call
s_set = s.cat.set_categories(["High", "Medium", "Low", "Unknown"])
print(s_set.cat.categories.tolist())
# Expected output: ['High', 'Medium', 'Low', 'Unknown']
```

**Why this output:** `reorder_categories()` changes the logical order of categories, enabling comparison operations. `set_categories()` is a one-step operation that can add, remove, and reorder categories simultaneously.

### Real-World Cases

- **Data cleaning:** Renaming categories to correct typos (e.g., “Medum” → “Medium”).
- **Report preparation:** Reordering categories to match a desired presentation sequence (e.g., “Q1, Q2, Q3, Q4”).
- **Memory cleanup:** Removing unused categories after filtering to free memory.
- **Standardization:** Using `set_categories()` to align categories across multiple datasets before concatenation.

### References

- pandas.Series.cat — https://pandas.pydata.org/docs/reference/api/pandas.Series.cat.html
- Categorical data (Categorical accessor) — https://pandas.pydata.org/docs/user_guide/categorical.html#categorical-accessor


## References

- Categorical data — https://pandas.pydata.org/docs/dev/user_guide/categorical.html
- pandas.Categorical — https://pandas.pydata.org/docs/reference/api/pandas.Categorical.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html
- pandas.Series.cat — https://pandas.pydata.org/docs/reference/api/pandas.Series.cat.html
- pandas.DataFrame.groupby — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.groupby.html
- DEPR: observed=False default in groupby — https://github.com/pandas-dev/pandas/pull/51811
- Pandas Categories Optimization — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/library/pandas/14-pandas-advanced-topics/3-pandas-categories-optimization.mdx
- Categorical data (Memory usage) — https://pandas.pydata.org/docs/user_guide/categorical.html#memory-usage
- Categorical data (Ordered or not) — https://pandas.pydata.org/docs/user_guide/categorical.html#ordered-or-not
- Categorical data (Categorical accessor) — https://pandas.pydata.org/docs/user_guide/categorical.html#categorical-accessor