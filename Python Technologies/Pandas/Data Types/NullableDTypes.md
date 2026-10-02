# Pandas Nullable Data Types: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Nullable data types in pandas are extension dtypes that support missing values natively without forcing a change to the underlying data type, using `pd.NA` as the unified missing value indicator.

**Technical Definition:** Nullable data types are pandas `ExtensionDtype` subclasses (e.g., `Int64Dtype`, `Float64Dtype`, `BooleanDtype`) backed by `ExtensionArray` objects (`IntegerArray`, `FloatingArray`, `BooleanArray`) that store a data array and a separate boolean mask array to track missing entries. They are distinguished from their NumPy counterparts by capitalization: `Int64` vs `int64`, `Float64` vs `float64`, `boolean` vs `bool`.

**Beginner-Friendly Explanation:** Normally, if you have a column of whole numbers and one value is missing, pandas has to turn the entire column into decimals to store that missing value. Nullable types let you keep whole numbers as whole numbers even when some values are missing, using a special marker called `pd.NA` instead of the decimal `NaN`.

### Key Characteristics

- **Capitalization convention:** Nullable dtypes use capitalized names (`Int64`, `Float64`, `boolean`) to differentiate them from NumPy dtypes (`int64`, `float64`, `bool`).
- **Unified missing value:** All nullable types use `pd.NA` as their missing value indicator, replacing the inconsistent mix of `np.nan`, `None`, and `pd.NaT`.
- **Mask-based architecture:** Each nullable array stores a data array plus a boolean mask array indicating which entries are missing.
- **Three-valued logic:** Nullable boolean arrays implement Kleene logic (three-valued logic) for logical operations, where `NA` propagates only when the result cannot be determined.
- **Experimental status:** `IntegerArray` is currently experimental; its API or implementation may change without warning.

### Prerequisites

- Basic Python knowledge (variables, functions, data types).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of NumPy dtypes and the `NaN` missing value convention.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Cleaning:** Nullable types preserve original data types when missing values are present, preventing unwanted type coercion.
- **Memory Optimization:** Nullable integers and floats allow smaller bit-width dtypes to be used even with missing values.
- **Statistical Analysis:** Three-valued logic ensures correct propagation of missing values in logical and comparison operations.
- **Database Integration:** Nullable types mirror SQL NULL semantics, making pandas-to-database workflows more consistent.

### Core Concepts / Features

1. The Missing Data Problem: NumPy Type Coercion
2. Capitalized Nullable Extension Types: `Int64`, `Float64`, and `boolean`
3. The Missing Value Indicator `pd.NA` and Three-Valued Logic
4. The Missing Value Matrix: `None` vs `np.nan` vs `pd.NA`

---

## 1. The Missing Data Problem: How Legacy NumPy Types Upcast Integers to Floats

### Definitions

**Core Definition:** The missing data problem in legacy pandas arises because `NaN` is a floating-point value, and NumPy's integer and boolean arrays cannot store `NaN`, forcing pandas to upcast those columns to `float64` or `object` when missing values are introduced.

**Technical Definition:** NumPy's fixed-width integer dtypes (`int8`–`int64`, `uint8`–`uint64`) and boolean dtype (`bool`) have no representation for missing values. When pandas introduces `NaN` into such an array (e.g., via reindexing or assignment), it must promote the dtype according to a casting table: `integer` → `float64`, `boolean` → `object`. This upcasting can cause precision loss for large integers that cannot be exactly represented as floating-point numbers.

**Beginner-Friendly Explanation:** Imagine a column of whole numbers. If one value is missing, pandas writes `NaN` in that spot. But `NaN` is a decimal number, and whole-number columns cannot hold decimals. So pandas converts the whole column to decimals—even the values that were perfectly good whole numbers. This is wasteful and can even change the actual value of very large numbers.

### Purposes

- To understand why integer columns become `float64` when missing values are present.
- To recognize when type coercion causes precision loss or logical errors.
- To motivate the use of nullable extension types as a solution.
- To diagnose unexpected `float64` columns after data loading or reindexing.

### Syntax Rules and Structure

**Demonstrating the Problem:**

```python
# Creating an integer Series
s = pd.Series([1, 2, 3], dtype="int64")
print(s.dtype)  # int64

# Introducing a missing value forces upcast to float64
s_with_na = pd.Series([1, 2, None])
print(s_with_na.dtype)  # float64

# Reindexing introduces NaN and upcasts
s_reindexed = s.reindex([0, 1, 2, 3])
print(s_reindexed.dtype)  # float64
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `pd.Series([1, 2, None])` | List containing integers and `None` |
| `.dtype` | Returns `float64` because `None` is coerced to `NaN` |
| `.reindex([0,1,2,3])` | Introduces a missing index label, creating `NaN` |

**Syntax Rules:**

- Any `NaN` or `None` in an integer list forces the Series dtype to `float64`.
- Reindexing to include missing labels upcasts integer and boolean dtypes.
- Once upcast to `float64`, the original integer type information is lost.

**Constraints and Limitations:**

- Integers larger than 2^53 cannot be represented exactly in `float64`, causing silent precision loss.
- Boolean columns with `NaN` become `object` dtype, which is memory-inefficient and breaks vectorized operations.
- The upcast is not reversible without dropping or filling missing values.

### Annotated Code Examples

**Example 1: Integer Upcast to Float**

```python
import pandas as pd

# Integer Series without missing values
s_int = pd.Series([10, 20, 30])
print(f"Original dtype: {s_int.dtype}")
# Expected output: Original dtype: int64

# Introduce a missing value by reindexing
s_na = s_int.reindex([0, 1, 2, 3])
print(s_na)
print(f"After reindex dtype: {s_na.dtype}")
# Expected output:
# 0    10.0
# 1    20.0
# 2    30.0
# 3     NaN
# dtype: float64
# After reindex dtype: float64
```

**Why this output:** Reindexing to include index label `3` introduces a missing value. Since `int64` cannot store `NaN`, pandas upcasts the entire Series to `float64`. The original integers `10`, `20`, `30` are now stored as `10.0`, `20.0`, `30.0`.

**Example 2: Boolean Upcast to Object**

```python
# Boolean Series without missing values
s_bool = pd.Series([True, False, True])
print(f"Original dtype: {s_bool.dtype}")
# Expected output: Original dtype: bool

# Introduce a missing value
s_bool_na = s_bool.reindex([0, 1, 2, 3])
print(s_bool_na)
print(f"After reindex dtype: {s_bool_na.dtype}")
# Expected output:
# 0     True
# 1    False
# 2     True
# 3     None
# dtype: object
# After reindex dtype: object
```

**Why this output:** NumPy's `bool` dtype cannot store `NaN` or `None`. When a missing value is introduced, pandas upcasts the Series to `object`, storing Python `True`/`False` and `None` as arbitrary Python objects. This is memory-inefficient and loses vectorized boolean operations.

**Example 3: Precision Loss with Large Integers**

```python
# A large integer that exceeds float64 precision
large_int = 2**53 + 1  # 9007199254740993
print(f"Original integer: {large_int}")

# Upcast to float64
s = pd.Series([large_int, None])
print(f"Stored as float: {s.iloc[0]}")
print(f"Are they equal? {large_int == s.iloc[0]}")
# Expected output:
# Original integer: 9007199254740993
# Stored as float: 9007199254740992.0
# Are they equal? False
```

**Why this output:** The integer `2^53 + 1` cannot be represented exactly in IEEE 754 double-precision floating point. The nearest representable value is `2^53`, causing silent precision loss. This demonstrates why upcasting integers to floats is problematic for identifier columns.

### Real-World Cases

- **Database IDs:** A column of user IDs stored as `int64` becomes `float64` when a row is missing, causing IDs like `9007199254740993` to be silently corrupted.
- **Boolean flags:** A `bool` column of feature flags becomes `object` when a single `None` is introduced, breaking downstream filtering and aggregation.
- **Financial data:** Account balances stored as integers (cents) lose precision when upcast to `float64` after reindexing.

### References

- Nullable integer data type — https://pandas.pydata.org/pandas-docs/version/2.0/user_guide/integer_na.html
- Missing data casting rules and indexing — https://pandas.pydata.org/docs/user_guide/missing_data.html

---

## 2. Capitalized Nullable Extension Types: Int64, Float64, and boolean

### Definitions

**Core Definition:** Pandas provides nullable extension types `Int64` (and `Int8`, `Int16`, `Int32`), `Float64` (and `Float32`), and `boolean` that store a data array plus a boolean mask array to track missing values, using `pd.NA` as the missing indicator.

**Technical Definition:** These types are instances of `pandas.api.extensions.ExtensionDtype` subclasses: `Int64Dtype`, `Float64Dtype`, and `BooleanDtype`. Each is paired with an `ExtensionArray`: `IntegerArray`, `FloatingArray`, and `BooleanArray` respectively. The data array stores values, and the mask array stores `True` where values are missing. The string aliases are `"Int64"`, `"Float64"`, and `"boolean"` (note the capitalization).

**Beginner-Friendly Explanation:** These are special column types that can hold both numbers (or `True`/`False`) and a special missing marker called `pd.NA`. They are named with a capital letter—`Int64` instead of `int64`—to remind you they are the nullable versions.

### Purposes

- To preserve integer and boolean dtypes when missing values are present.
- To avoid the precision loss and memory overhead of upcasting to `float64` or `object`.
- To provide a consistent `pd.NA` missing value indicator across all data types.
- To enable correct three-valued logic in boolean operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Integer nullable types
pd.Series([1, 2, None], dtype="Int64")
pd.array([1, 2, None], dtype=pd.Int64Dtype())
pd.Series([1, 2, None], dtype="Int8")   # also Int16, Int32

# Float nullable types
pd.Series([1.5, 2.5, None], dtype="Float64")
pd.Series([1.5, 2.5, None], dtype="Float32")

# Boolean nullable type
pd.Series([True, False, None], dtype="boolean")
pd.array([True, False, pd.NA], dtype="boolean")
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `"Int64"` | String alias for nullable 64-bit integer |
| `pd.Int64Dtype()` | Full constructor for nullable integer dtype |
| `"Float64"` | String alias for nullable 64-bit float |
| `"boolean"` | String alias for nullable boolean (all lowercase) |
| `pd.array()` | Creates a standalone ExtensionArray |

**Syntax Rules:**

- The capitalization is mandatory: `"Int64"` is nullable; `"int64"` is NumPy-backed.
- `"boolean"` is all lowercase; `"Boolean"` raises an error.
- All NA-like values (`None`, `np.nan`, `pd.NA`) are converted to `pd.NA` in nullable arrays.
- Nullable types support the same operations as their NumPy counterparts, plus `pd.NA`-aware behaviour.

**Constraints and Limitations:**

- `IntegerArray` is currently experimental; its API may change without warning.
- `pandas.array()` and `pandas.Series()` use different rules for dtype inference; `pandas.array()` will infer a nullable dtype while `pandas.Series()` may not.
- Not all pandas operations are optimized for nullable dtypes; some may be slower than NumPy-backed equivalents.

### Annotated Code Examples

**Example 1: Creating Nullable Integer Series**

```python
import pandas as pd
import numpy as np

# Construct with Int64Dtype
arr = pd.array([1, 2, None], dtype=pd.Int64Dtype())
print(arr)
# Expected output:
# <IntegerArray>
# [1, 2, <NA>]
# Length: 3, dtype: Int64

# Construct with string alias
s = pd.Series([1, 2, np.nan], dtype="Int64")
print(s)
print(s.dtype)
# Expected output:
# 0       1
# 1       2
# 2    <NA>
# dtype: Int64
# Int64
```

**Why this output:** The nullable integer array stores `1` and `2` as integers and `None`/`np.nan` as `pd.NA` (displayed as `<NA>`). The dtype is `Int64` (capital I), not `int64`.

**Example 2: Nullable Boolean with Three-Valued Logic**

```python
# Create nullable boolean Series
s = pd.Series([True, False, pd.NA], dtype="boolean")
print(s)
# Expected output:
# 0     True
# 1    False
# 2     <NA>
# dtype: boolean

# Logical AND with True
print(s & True)
# Expected output:
# 0     True
# 1    False
# 2     <NA>
# dtype: boolean

# Logical OR with True
print(s | True)
# Expected output:
# 0    True
# 1    True
# 2    True
# dtype: boolean
```

**Why this output:** `True & NA` is `NA` because the result depends on whether `NA` is really `True` or `False`. `True | NA` is `True` because regardless of the `NA` value, the result is `True`. This is Kleene three-valued logic.

**Example 3: Nullable Float**

```python
# Create nullable float Series
s = pd.Series([2.25, pd.NA, 3.75], dtype=pd.Float64Dtype())
print(s)
print(s.dtype)
# Expected output:
# 0    2.25
# 1    <NA>
# 2    3.75
# dtype: Float64
# Float64

# Arithmetic operations propagate NA
print(s + 1)
# Expected output:
# 0    3.25
# 1    <NA>
# 2    4.75
# dtype: Float64
```

**Why this output:** `Float64` uses `pd.NA` instead of `np.nan` for missing values. Arithmetic operations propagate `pd.NA` to the result, maintaining the `Float64` dtype.

### Real-World Cases

- **Database-like data:** Columns read from SQL databases with nullable integer columns are mapped to pandas `Int64` to preserve NULL semantics.
- **Survey data:** Boolean columns (“Did you answer question 5?”) with missing responses use `boolean` dtype to distinguish “No” from “Did not answer”.
- **Financial data:** Cent-denominated integer columns with occasional missing values use `Int64` to avoid float precision loss.

### References

- Nullable integer data type — https://pandas.pydata.org/pandas-docs/version/2.0/user_guide/integer_na.html
- Nullable Boolean data type — https://pandas.pydata.org/docs/dev/_sources/user_guide/boolean.rst.txt
- pandas.Float64Dtype — https://pandas.pydata.org/docs/reference/api/pandas.Float64Dtype.html

---

## 3. The Missing Value Indicator pd.NA and Three-Valued Logic

### Definitions

**Core Definition:** `pd.NA` is pandas' native singleton missing value indicator, used consistently across all nullable extension types to represent missing or unknown values.

**Technical Definition:** `pd.NA` is an instance of `pandas._libs.missing.NAType`. It propagates through most operations (arithmetic, comparison, logical) as “unknown”, implementing Kleene three-valued logic for boolean operations. Unlike `np.nan`, which is a float and behaves unpredictably in comparisons (`np.nan != np.nan` is `True`), `pd.NA` returns `pd.NA` for equality comparisons, enabling consistent missing-value semantics.

**Beginner-Friendly Explanation:** `pd.NA` is a special marker that means “this value is missing.” It behaves like the SQL NULL: if you compare anything to it, you get back `pd.NA` (not `True` or `False`), because the answer is unknown.

### Purposes

- To provide a single, consistent missing value indicator across all pandas data types.
- To implement three-valued logic (True, False, Unknown) for boolean operations.
- To avoid the confusion of `np.nan` (which is a float) and `None` (which is a Python object).
- To enable correct propagation of missing values in logical and comparison operations.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Accessing pd.NA
pd.NA

# Using in arrays
pd.array([1, 2, pd.NA], dtype="Int64")
pd.Series([True, False, pd.NA], dtype="boolean")

# Checking for NA
pd.isna(value)
series.isna()
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `pd.NA` | Singleton missing value indicator |
| `pd.isna(x)` | Returns `True` if `x` is `pd.NA`, `None`, or `NaN` |
| `series.isna()` | Boolean mask of missing values |

**Syntax Rules:**

- `pd.NA` is a singleton; use `pd.isna(x)` rather than `x == pd.NA` to check for missing values.
- Arithmetic operations with `pd.NA` return `pd.NA`.
- Comparison operations (`==`, `<`, `>`) with `pd.NA` return `pd.NA`, not `True` or `False`.
- Logical operations (`&`, `|`, `^`) follow Kleene three-valued logic.

**Constraints and Limitations:**

- `pd.NA` cannot be used in NumPy-backed dtypes; it is only valid for nullable extension types.
- Assigning `pd.NA` to a NumPy-backed column may upcast the dtype to `object`.
- Some third-party libraries may not recognize `pd.NA` and may require conversion to `np.nan`.

### Annotated Code Examples

**Example 1: pd.NA in Comparisons**

```python
import pandas as pd

# pd.NA in equality comparison
print(pd.NA == pd.NA)    # <NA>
print(pd.NA == 1)        # <NA>
print(pd.NA != 1)        # <NA>

# np.nan behaviour for comparison
import numpy as np
print(np.nan == np.nan)  # False
print(np.nan != np.nan)  # True
```

**Why this output:** `pd.NA` follows SQL-style three-valued logic: comparing anything to `pd.NA` yields `pd.NA` (unknown). In contrast, `np.nan` is not equal to itself, which is often surprising and inconsistent.

**Example 2: Kleene Three-Valued Logic Table**

```python
import pandas as pd

# Create a DataFrame demonstrating all combinations
left = pd.Series([True, True, True, False, False, False, pd.NA, pd.NA, pd.NA], dtype="boolean")
right = pd.Series([True, False, pd.NA, True, False, pd.NA, True, False, pd.NA], dtype="boolean")

print("AND (&):")
print(pd.DataFrame({"left": left, "right": right, "result": left & right}))
# Expected output:
#     left  right result
# 0   True   True   True
# 1   True  False  False
# 2   True   <NA>   <NA>
# 3  False   True  False
# 4  False  False  False
# 5  False   <NA>  False
# 6   <NA>   True   <NA>
# 7   <NA>  False  False
# 8   <NA>   <NA>   <NA>

print("\nOR (|):")
print(pd.DataFrame({"left": left, "right": right, "result": left | right}))
# Expected output:
#     left  right result
# 0   True   True   True
# 1   True  False   True
# 2   True   <NA>   True
# 3  False   True   True
# 4  False  False  False
# 5  False   <NA>   <NA>
# 6   <NA>   True   True
# 7   <NA>  False   <NA>
# 8   <NA>   <NA>   <NA>
```

**Why this output:** In Kleene logic, `True & NA` is `NA` because the result depends on the unknown value. `False & NA` is `False` because regardless of the unknown value, `False & anything` is `False`. `True | NA` is `True` because `True | anything` is `True`. This is the core of three-valued logic.

**Example 3: Using pd.isna vs Equality**

```python
s = pd.Series([1, pd.NA, 3], dtype="Int64")

# Correct way to check for NA
print(pd.isna(s))
# Expected output:
# 0    False
# 1     True
# 2    False
# dtype: bool

# Incorrect way (returns NA, not True/False)
print(s == pd.NA)
# Expected output:
# 0    False
# 1     <NA>
# 2    False
# dtype: boolean
```

**Why this output:** `pd.isna()` returns a proper boolean mask. `s == pd.NA` returns `pd.NA` for the missing entry because comparing to `NA` yields `NA` (unknown). Always use `pd.isna()` for missing value detection.

### Real-World Cases

- **SQL integration:** Data read from databases with NULL values is mapped to `pd.NA`, preserving SQL NULL semantics.
- **Data validation:** Checking `pd.isna()` on a column identifies missing values for imputation or removal.
- **Logical filtering:** Boolean masks with `pd.NA` values are treated as `False` during indexing, allowing safe filtering without errors.

### References

- Nullable Boolean data type (Kleene logic) — https://pandas.pydata.org/docs/dev/_sources/user_guide/boolean.rst.txt
- pandas.NA — https://pandas.pydata.org/docs/reference/api/pandas.NA.html
- Missing data — https://pandas.pydata.org/docs/user_guide/missing_data.html

---

## 4. The Missing Value Matrix: None vs np.nan vs pd.NA

### Definitions

**Core Definition:** The Missing Value Matrix compares the three primary missing value representations in pandas—Python `None`, NumPy `np.nan`, and pandas `pd.NA`—across dimensions of type, equality behaviour, propagation, and dtype compatibility.

**Technical Definition:** `None` is a Python singleton of type `NoneType`. `np.nan` is a Python float (`float`) representing IEEE 754 “Not a Number”. `pd.NA` is a singleton of type `NAType`. Each has distinct behaviour in comparisons, arithmetic, logical operations, and dtype coercion. Pandas treats `None` and `np.nan` as largely interchangeable for missing value indication in NumPy-backed dtypes, while `pd.NA` is the unified missing indicator for nullable extension dtypes.

**Beginner-Friendly Explanation:** Think of these as three different “missing” stickers. `None` is Python's way of saying “nothing.” `np.nan` is a decimal number that means “not a number.” `pd.NA` is pandas' own sticker that means “missing.” They look similar but behave differently—especially when you compare them.

### Purposes

- To understand which missing value to use in which context.
- To predict how missing values behave in comparisons, arithmetic, and logic.
- To choose the right missing value indicator for nullable vs. NumPy-backed dtypes.
- To debug unexpected behaviour in missing-value handling.

### Syntax Rules and Structure

**Behavioural Matrix:**

| Dimension | `None` (Python) | `np.nan` (NumPy) | `pd.NA` (Pandas) |
|-----------|-----------------|-------------------|-------------------|
| **Type** | `NoneType` | `float` | `NAType` |
| **Singleton** | Yes | No (multiple NaN values possible) | Yes |
| **`x == x`** | `True` | `False` | `pd.NA` |
| **`x != x`** | `False` | `True` | `pd.NA` |
| **Arithmetic** | Raises `TypeError` | Returns `nan` | Returns `pd.NA` |
| **Logical `&`** | Works (treated as `False`) | Works (treated as `False`) | Kleene logic (`NA` propagates) |
| **Used in `int64`** | Upcasts to `float64` | Upcasts to `float64` | Not allowed (use `Int64`) |
| **Used in `boolean`** | Upcasts to `object` | Upcasts to `object` | Native |
| **Used in `object`** | Yes | Yes | Yes |
| **Detection** | `is None` | `math.isnan()` / `pd.isna()` | `pd.isna()` |

**Syntax Rules:**

- In NumPy-backed dtypes, `None` and `np.nan` are largely interchangeable; pandas converts `None` to `NaN` in numeric contexts.
- In nullable extension dtypes, all NA-like values (`None`, `np.nan`, `pd.NA`) are converted to `pd.NA`.
- `pd.NA` cannot be stored in NumPy-backed dtypes without upcasting to `object` or a nullable type.

**Constraints and Limitations:**

- `None` in a numeric context raises `TypeError` for arithmetic; `np.nan` propagates silently.
- `np.nan` is not equal to itself, which breaks dictionary lookups and set membership tests.
- `pd.NA` is not supported by all third-party libraries; may need conversion to `np.nan` for interoperability.

### Annotated Code Examples

**Example 1: Equality Behaviour Comparison**

```python
import pandas as pd
import numpy as np

# None
print(f"None == None: {None == None}")          # True
print(f"None != None: {None != None}")          # False

# np.nan
print(f"np.nan == np.nan: {np.nan == np.nan}")  # False
print(f"np.nan != np.nan: {np.nan != np.nan}")  # True

# pd.NA
print(f"pd.NA == pd.NA: {pd.NA == pd.NA}")      # <NA>
print(f"pd.NA != pd.NA: {pd.NA != pd.NA}")      # <NA>
```

**Why this output:** `None` is a singleton, so it equals itself. `np.nan` is defined by IEEE 754 as not equal to itself. `pd.NA` follows three-valued logic, so equality is “unknown” (`pd.NA`).

**Example 2: Arithmetic Behaviour Comparison**

```python
# None in arithmetic
try:
    print(None + 1)
except TypeError as e:
    print(f"None + 1 raises: {e}")
# Expected output: None + 1 raises: unsupported operand type(s) for +: 'NoneType' and 'int'

# np.nan in arithmetic
print(f"np.nan + 1 = {np.nan + 1}")  # nan

# pd.NA in arithmetic
print(f"pd.NA + 1 = {pd.NA + 1}")    # <NA>
```

**Why this output:** `None` is not a number, so arithmetic raises `TypeError`. `np.nan` is a float, so arithmetic propagates `nan`. `pd.NA` propagates itself through arithmetic.

**Example 3: dtype Coercion Comparison**

```python
# Integer column with None
s_none = pd.Series([1, 2, None])
print(f"With None: {s_none.dtype}")   # float64

# Integer column with np.nan
s_nan = pd.Series([1, 2, np.nan])
print(f"With np.nan: {s_nan.dtype}")  # float64

# Integer column with pd.NA (nullable)
s_na = pd.Series([1, 2, pd.NA], dtype="Int64")
print(f"With pd.NA: {s_na.dtype}")    # Int64

# Boolean column with None
b_none = pd.Series([True, False, None])
print(f"Bool with None: {b_none.dtype}")   # object

# Boolean column with pd.NA
b_na = pd.Series([True, False, pd.NA], dtype="boolean")
print(f"Bool with pd.NA: {b_na.dtype}")    # boolean
```

**Why this output:** In NumPy-backed Series, both `None` and `np.nan` force integer upcast to `float64` and boolean upcast to `object`. Only `pd.NA` in a nullable dtype preserves the underlying type (`Int64`, `boolean`).

### Real-World Cases

- **Data cleaning pipelines:** Using `pd.isna()` to detect missing values works uniformly across `None`, `np.nan`, and `pd.NA`.
- **Database exports:** When exporting to CSV, `pd.NA` and `None` are written as empty strings; `np.nan` may be written as “NaN” unless `na_rep` is specified.
- **Interoperability:** When passing data to scikit-learn, `pd.NA` must be converted to `np.nan` because scikit-learn does not recognize `pd.NA`.

### References

- Missing data — https://pandas.pydata.org/docs/user_guide/missing_data.html
- pandas.NA — https://pandas.pydata.org/docs/reference/api/pandas.NA.html
- PDEP-16: Consistent missing value handling — https://github.com/pandas-dev/pandas/pull/58988

---

## References

- Nullable integer data type — https://pandas.pydata.org/pandas-docs/version/2.0/user_guide/integer_na.html
- Nullable Boolean data type (Kleene logic) — https://pandas.pydata.org/docs/dev/_sources/user_guide/boolean.rst.txt
- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2.3/reference/arrays.html
- pandas.Float64Dtype — https://pandas.pydata.org/docs/reference/api/pandas.Float64Dtype.html
- pandas.NA — https://pandas.pydata.org/docs/reference/api/pandas.NA.html
- Missing data — https://pandas.pydata.org/docs/user_guide/missing_data.html
- pandas.DataFrame.convert_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.convert_dtypes.html
- PDEP-16: Consistent missing value handling — https://github.com/pandas-dev/pandas/pull/58988