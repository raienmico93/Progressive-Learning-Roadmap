# Pandas Understanding Dtypes: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** In pandas, a **dtype** (data type) is an object that describes the type of data stored in a Series, Index, or DataFrame column, determining how values are stored in memory and which operations are permitted.

**Technical Definition:** A pandas dtype is an instance of `numpy.dtype` or a pandas extension dtype (subclass of `pandas.api.extensions.ExtensionDtype`). It specifies the in-memory representation (e.g., 64-bit integer, IEEE 754 double-precision float, nanosecond-resolution datetime), the set of allowed values, and the semantics of operations (arithmetic, comparison, aggregation). Pandas extends NumPy's type system to support timezone-aware datetimes, nullable integers, categoricals, and PyArrow-backed arrays.

**Beginner-Friendly Explanation:** Think of a dtype as a label on a column that tells pandas what kind of values live there—numbers, text, dates, or categories. This label matters because pandas uses it to decide how much memory to allocate, which mathematical operations make sense, and how to sort or group the data.

### Key Characteristics

- **Static typing per column:** Each column in a DataFrame has exactly one dtype; mixed-type columns are coerced to `object`.
- **Default dtypes:** Integer types default to `int64` and floats to `float64`, regardless of platform (32-bit or 64-bit).
- **Extension types:** Pandas extends NumPy with dtypes for nullable integers (`Int64`), strings (`str` in pandas 3.0+), categoricals (`category`), timezone-aware datetimes (`datetime64[ns, tz]`), and PyArrow-backed types (e.g., `int64[pyarrow]`).
- **Type propagation:** Numeric dtypes propagate and coexist in DataFrames; the `.values` attribute returns the lowest common denominator dtype that can accommodate all types.
- **Version-specific transition:** In pandas 3.0, string columns are inferred as `str` dtype instead of `object`, and PyArrow is used as the default backend when installed.

### Prerequisites

- Basic Python knowledge (variables, functions, lists).
- Familiarity with NumPy arrays and `numpy.dtype`.
- Installation of pandas (`pip install pandas`). Optional: `pyarrow` for Arrow-backed dtypes.
- Understanding of basic data structures: Series and DataFrame.

### Related Programming Areas

- **Data Analysis & Wrangling:** dtype selection affects groupby, pivot, and merge performance.
- **Time Series Analysis:** datetime64 and timedelta64 are foundational for resampling, rolling windows, and timezone conversion.
- **Memory Optimization:** Categorical and nullable types reduce memory footprint.
- **Interoperability:** PyArrow dtypes facilitate zero-copy exchange with Polars, DuckDB, and Apache Arrow.
- **Machine Learning:** Correct dtypes prevent silent bugs in scikit-learn pipelines (e.g., object vs. string).

### Core Concepts / Features

1. Core Type Categories (Numeric, Boolean, Object)
2. Specialized Time Series Types (datetime64[ns], timedelta64[ns], timezone-aware)
3. String Data Type Transition (object → str)
4. Categorical Data
5. PyArrow-Backed Dtypes
6. dtype Inspection and Conversion (`astype`, `select_dtypes`, `convert_dtypes`)

---

## 1. Core Type Categories: Numeric, Boolean, and Object

### Definitions

**Core Definition:** Pandas' three foundational dtype families are numeric (integers and floats), boolean (`bool`), and object (`object`, which holds arbitrary Python objects including strings).

**Technical Definition:** Numeric types are backed by NumPy fixed-width integer (`int8`, `int16`, `int32`, `int64`, `uint8`–`uint64`) and floating-point (`float16`, `float32`, `float64`) dtypes. Boolean is `numpy.bool_`. The `object` dtype is a NumPy array of pointers to Python objects, allowing heterogeneous types but sacrificing performance.

**Beginner-Friendly Explanation:** Numbers get fast numeric types. True/False values get `bool`. Anything else—like text or mixed values—ends up as `object`, which is flexible but slow.

### Purposes

- To store numeric data efficiently for arithmetic and statistical operations.
- To represent logical conditions and masks for filtering.
- To hold heterogeneous or text data when no specialized dtype applies.
- To provide a fallback that accommodates mixed types in a column.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Constructing with explicit dtype
pd.Series(data, dtype=dtype)
pd.DataFrame(data, dtype=dtype)

# Inspecting dtype
series.dtype
dataframe.dtypes
dataframe.dtypes.value_counts()
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `data` | List, array, dict, or scalar values |
| `dtype` | `int64`, `float64`, `bool`, `object`, or NumPy dtype object |
| `series.dtype` | Returns the dtype of a Series |
| `dataframe.dtypes` | Returns a Series of column names → dtypes |
| `.value_counts()` | Counts columns by dtype |

**Syntax Rules:**

- Numeric literals like `1` produce `int64`; `1.0` produces `float64`.
- Mixing ints and floats coerces to `float64`.
- Introducing a string into a numeric column coerces the entire column to `object`.
- `object` is the most general dtype and is chosen when no narrower type fits all values.

**Constraints and Limitations:**

- `object` columns cannot participate in vectorized numeric operations without explicit conversion.
- Arithmetic on `object` columns containing numbers may work element-wise but is slow and error-prone.
- Boolean columns cannot hold `NaN` natively in NumPy-backed form; use nullable `boolean` dtype for missing values.

### Annotated Code Examples

**Example 1: Numeric Type Propagation**

```python
import pandas as pd
import numpy as np

# Integer and float columns retain their dtypes
df = pd.DataFrame({
    'int_col': [1, 2, 3],           # int64
    'float_col': [1.0, 2.5, 3.7],   # float64
    'bool_col': [True, False, True] # bool
})

print(df.dtypes)
# Expected output:
# int_col        int64
# float_col    float64
# bool_col        bool
# dtype: object
```

**Why this output:** Each column is created from homogeneous literals. Pandas infers the narrowest compatible dtype per column. `int_col` contains only integers → `int64`. `float_col` contains floats → `float64`. `bool_col` contains booleans → `bool`.

**Example 2: Mixed Types Force Object**

```python
# Introducing a string into a numeric column coerces to object
s = pd.Series([1, 2, 3, 6.0, 'foo'])
print(s.dtype)
# Expected output: object

# Ints coerced to floats when combined with floats
s2 = pd.Series([1, 2, 3, 4, 5, 6.0])
print(s2.dtype)
# Expected output: float64
```

**Why this output:** In `s`, the string `'foo'` cannot be represented as a numeric type, so pandas falls back to `object`—the most general dtype. In `s2`, integers are safely cast to `float64` because float can represent all integers in the list without loss.

**Example 3: dtype Inspection on a DataFrame**

```python
df = pd.DataFrame({
    'A': np.random.rand(3),
    'B': 1,
    'C': 'foo',
    'D': pd.Timestamp('20010102'),
    'E': pd.Series([1.0] * 3).astype('float32'),
    'F': False,
    'G': pd.Series([1] * 3, dtype='int8')
})

print(df.dtypes)
print("\nColumn dtype counts:")
print(df.dtypes.value_counts())
```

**Expected Output:**
```
A           float64
B             int64
C            object
D    datetime64[ns]
E           float32
F              bool
G              int8
dtype: object

Column dtype counts:
float64         1
int64           1
object          1
datetime64[ns]  1
float32         1
bool            1
int8            1
dtype: int64
```

**Why this output:** Each column is constructed with a different literal or dtype specification. `df.dtypes` returns a Series indexed by column names. `.value_counts()` counts how many columns share each dtype.

### Real-World Cases

- **Financial data:** Revenue columns use `float64`; transaction counts use `int64`; transaction IDs may use `object` if alphanumeric.
- **Survey data:** Likert-scale responses may be stored as `int64` initially, then converted to `category` for analysis.
- **Mixed CSV imports:** A column with numbers and “N/A” strings becomes `object` until cleaned and converted with `pd.to_numeric()`.

### References

- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2/reference/arrays.html
- pandas.DataFrame.astype — https://pandas.pydata.org/pandas-docs/stable/dev/reference/api/pandas.DataFrame.astype.html

---

## 2. Specialized Time Series Types: datetime64[ns] and timedelta64[ns]

### Definitions

**Core Definition:** Pandas provides two NumPy-backed time-series dtypes: `datetime64[ns]` for points in time and `timedelta64[ns]` for durations, plus a timezone-aware extension dtype `datetime64[ns, tz]`.

**Technical Definition:** `datetime64[ns]` stores timestamps as 64-bit integers counting nanoseconds since the Unix epoch (1970-01-01 UTC). `timedelta64[ns]` stores durations in nanoseconds. Timezone-aware datetimes use `DatetimeTZDtype`, an extension dtype that pairs a `datetime64[ns]` array with a `pytz`/`zoneinfo` timezone object.

**Beginner-Friendly Explanation:** These dtypes handle dates, times, and durations. `datetime64[ns]` is a specific moment; `timedelta64[ns]` is a length of time. Timezone-aware versions also remember which timezone the moment belongs to.

### Purposes

- To represent timestamps for time-series indexing and resampling.
- To compute durations between events.
- To perform timezone conversions and DST-aware arithmetic.
- To enable frequency-based operations like `resample('D')` and `rolling('7D')`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Creating datetime Series
pd.to_datetime(arg, utc=False, format=None)
pd.date_range(start, end, freq, tz=None)

# Creating timedelta Series
pd.to_timedelta(arg, unit='ns')

# Accessing datetime properties
series.dt.year, series.dt.month, series.dt.tz

# Timezone operations
series.dt.tz_localize(tz)
series.dt.tz_convert(tz)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `pd.to_datetime` | Converts strings, ints, or lists to datetime64 |
| `utc` | If True, returns UTC-aware; if False, naive |
| `pd.date_range` | Generates fixed-frequency DatetimeIndex |
| `tz` | Timezone string (e.g., `'US/Eastern'`, `'UTC'`) |
| `pd.to_timedelta` | Converts to timedelta64[ns] |
| `unit` | Base unit for numeric timedelta input |
| `.dt.tz_localize` | Attaches timezone to naive datetimes |
| `.dt.tz_convert` | Converts between timezones |

**Syntax Rules:**

- `pd.to_datetime` infers format; use `format` for speed when format is known.
- Timezone-naive datetimes have no timezone info; attaching one requires `tz_localize`.
- Converting a timezone-naive Series with `astype` to a timezone-aware dtype raises an exception; use `.dt.tz_localize()` instead.
- Arithmetic between datetime and timedelta yields datetime; subtraction of two datetimes yields timedelta.
- `NaT` (Not a Time) represents missing datetime/timedelta values.

**Constraints and Limitations:**

- NumPy cannot natively represent timezone-aware datetimes; pandas uses `DatetimeArray` extension array.
- `datetime64[ns]` range is limited to approximately 1677–2262 due to nanosecond resolution.
- Deprecated: `'datetimetz'` and `'datetime64tz'` string aliases; use `pd.DatetimeTZDtype` instead.

### Annotated Code Examples

**Example 1: Creating and Inspecting Datetime Data**

```python
import pandas as pd

# Create naive datetime Series
s = pd.Series(pd.to_datetime(['2024-01-01', '2024-06-15', '2024-12-31']))
print(s.dtype)
# Expected output: datetime64[ns]

# Create timezone-aware DatetimeIndex
dti = pd.date_range('2018-03-01 09:00', periods=3, freq='D', tz='US/Eastern')
print(dti)
# Expected output:
# DatetimeIndex(['2018-03-01 09:00:00-05:00',
#                '2018-03-02 09:00:00-05:00',
#                '2018-03-03 09:00:00-05:00'],
#               dtype='datetime64[ns, US/Eastern]', freq='D')
```

**Why this output:** `pd.to_datetime` on date strings produces `datetime64[ns]` by default. `pd.date_range` with `tz='US/Eastern'` creates a `DatetimeTZDtype` array where each timestamp carries the UTC offset for that date (EST = UTC−5 in March before DST).

**Example 2: Timedelta Arithmetic**

```python
# Create a DatetimeIndex and add a Timedelta
tdi = pd.DatetimeIndex(['2013-01-01', 'NaT', '2013-01-03'])
result = tdi + pd.Timedelta('10 days')
print(result)
# Expected output:
# DatetimeIndex(['2013-01-11', 'NaT', '2013-01-13'], dtype='datetime64[ns]', freq=None)

# Subtraction of two datetimes yields timedelta
delta = pd.Timestamp('20130101') - pd.Timestamp('20121222')
print(delta)
# Expected output: Timedelta('10 days 00:00:00')

# Timedelta divided by integer yields Timedelta
print(pd.Timedelta('10 days') / 2)
# Expected output: Timedelta('5 days 00:00:00')
```

**Why this output:** Adding a `Timedelta` to a `DatetimeIndex` performs element-wise date arithmetic. `NaT` propagates (any operation with `NaT` yields `NaT`). Subtracting two timestamps produces a `Timedelta` object. Division by a scalar preserves the timedelta type.

**Example 3: Timezone Localization and Conversion**

```python
# Start with naive datetimes
naive = pd.Series(pd.to_datetime(['2024-01-01 12:00', '2024-07-01 12:00']))

# Localize to US/Eastern
aware = naive.dt.tz_localize('US/Eastern')
print(aware)
# Expected output:
# 0   2024-01-01 12:00:00-05:00
# 1   2024-07-01 12:00:00-04:00
# dtype: datetime64[ns, US/Eastern]

# Convert to UTC
utc = aware.dt.tz_convert('UTC')
print(utc)
# Expected output:
# 0   2024-01-01 17:00:00+00:00
# 1   2024-07-01 16:00:00+00:00
# dtype: datetime64[ns, UTC]
```

**Why this output:** `tz_localize` attaches the timezone, adjusting the UTC offset based on DST (EST = UTC−5 in January; EDT = UTC−4 in July). `tz_convert` changes the representation to UTC without changing the absolute moment in time.

### Real-World Cases

- **Log analysis:** Server logs with timestamps in UTC are converted to local time for reporting.
- **Financial markets:** Trade timestamps are timezone-aware to handle cross-market sessions.
- **IoT sensor data:** Durations between sensor readings are computed as `timedelta64[ns]` for anomaly detection.

### References

- Time series / date functionality — https://pandas.pydata.org/docs/user_guide/timeseries.html
- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2/reference/arrays.html

---

## 3. String Data Type Transition: From object to str

### Definitions

**Core Definition:** Pandas 3.0 introduces a dedicated `str` dtype (alias `"str"`) for string data, replacing the historical use of `object` as the default string container.

**Technical Definition:** The new `str` dtype is an instance of `pandas.StringDtype` with `na_value=np.nan`. When PyArrow is installed, it is backed by `pyarrow.string()` arrays (contiguous memory, zero-copy interoperability); otherwise it falls back to Python objects in a NumPy array. It can only hold strings or missing values.

**Beginner-Friendly Explanation:** In older pandas, text columns were labeled `object`, which was confusing because `object` can hold anything. Starting with pandas 3.0, text columns get their own `str` label, which is clearer and faster.

### Purposes

- To provide a dedicated, self-documenting dtype for text data.
- To improve memory efficiency through Arrow-backed contiguous storage.
- To enable type safety by rejecting non-string values in string columns.
- To align pandas string handling with Apache Arrow standards for interoperability.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Explicit construction
pd.Series(data, dtype="str")
pd.Series(data, dtype=pd.StringDtype(na_value=np.nan))

# Enabling future behavior in pandas 2.3+
pd.options.future.infer_string = True

# String accessor
series.str.contains(pattern)
series.str.len()
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype="str"` | Short alias for the new StringDtype |
| `pd.StringDtype(na_value=...)` | Full constructor with NA value control |
| `future.infer_string` | Opt-in option to preview pandas 3.0 behavior |
| `.str` accessor | Vectorized string methods |

**Syntax Rules:**

- In pandas ≥ 3.0, string data is inferred as `str` by default in constructors and IO functions.
- The `str` dtype can only hold strings or missing values; setting a non-string value raises an error.
- `StringDtype` methods returning numeric output always return nullable integer dtype, not `int` or `float`.
- In pandas < 3.0, use `dtype="string"` to opt into the nullable string dtype (uses `pd.NA`).

**Constraints and Limitations:**

- **Version-specific:** The automatic inference of `str` is pandas 3.0+. In pandas 2.3, it must be enabled via `pd.options.future.infer_string = True`.
- String operations on `str` dtype may differ subtly from `object` dtype (e.g., NA propagation).
- PyArrow backend requires `pyarrow` to be installed for optimal performance.

### Annotated Code Examples

**Example 1: Default String Inference in pandas 3.0**

```python
import pandas as pd

# In pandas 3.0, string columns are inferred as str dtype
s = pd.Series(["a", "b", None])
print(s)
# Expected output:
# 0      a
# 1      b
# 2    NaN
# dtype: str

print(s.dtype)
# Expected output: str
```

**Why this output:** Pandas 3.0 infers `str` for columns containing string literals and `None`. The `None` becomes `NaN`, consistent with other default dtypes. This replaces the old `object` dtype.

**Example 2: Explicit StringDtype Construction**

```python
# Explicitly request the new string dtype
s = pd.Series(["a", "b", None], dtype="str")
print(s.dtype)
# Expected output: str

# In pandas < 3.0, use "string" for the nullable string dtype
s_old = pd.Series(["a", "b", None], dtype="string")
print(s_old.dtype)
# Expected output: string
print(s_old)
# 0       a
# 1       b
# 2    <NA>
# dtype: string
```

**Why this output:** `dtype="str"` is the new alias in pandas 3.0. `dtype="string"` is the older nullable string dtype that uses `pd.NA` instead of `NaN` and has been available since pandas 1.0.

**Example 3: String vs Object Type Safety**

```python
import pandas as pd

# Object dtype can hold mixed types
obj = pd.Series(["a", "b", 1], dtype=object)
print(obj.dtype)
# Expected output: object

# str dtype rejects non-string values
try:
    s = pd.Series(["a", "b"], dtype="str")
    s.iloc[0] = 123  # Attempt to set an integer
except Exception as e:
    print(f"Error: {e}")
# Expected output (pandas 3.0): TypeError or similar
```

**Why this output:** The `object` dtype is permissive and can store strings, integers, and other objects. The `str` dtype is strict: it only accepts strings or missing values. This enforces type consistency and catches data-quality issues early.

### Real-World Cases

- **Data cleaning:** A column of product names inferred as `str` ensures that accidental numeric codes are flagged.
- **Memory optimization:** Arrow-backed `str` arrays reduce memory usage by 50–80% for large text columns compared to `object`.
- **Interoperability:** Reading Parquet files with pandas 3.0 yields `str` columns that can be passed to Polars or DuckDB without conversion.

### References

- Migration guide for the new string data type (pandas 3.0) — https://pandas.pydata.org/docs/user_guide/migration-3-strings.html
- What's new in 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html

---

## 4. Categorical Data

### Definitions

**Core Definition:** The `category` dtype represents data with a limited, usually fixed, set of possible values, storing each value as an integer code pointing to a category label.

**Technical Definition:** `CategoricalDtype` is a pandas extension dtype defined by two attributes: `categories` (a unique, non-null sequence of values) and `ordered` (a boolean). Internally, the data is stored as an integer array of codes plus a categories array, making memory usage proportional to the number of categories plus the data length.

**Beginner-Friendly Explanation:** If a column can only take a few different values—like “Red”, “Green”, “Blue”—categorical stores each value once and uses short codes for the rest, saving memory and enabling logical ordering.

### Purposes

- To reduce memory usage for low-cardinality string columns.
- To enforce a logical order on values that is not lexical (e.g., “Low”, “Medium”, “High”).
- To signal to libraries (plotting, statistics) that a variable is categorical.
- To speed up groupby and sorting operations on repeated values.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Creating categorical data
pd.Series(data, dtype="category")
pd.Series(data, dtype=pd.CategoricalDtype(categories, ordered))

# Casting to category
series.astype("category")
series.astype(pd.CategoricalDtype(categories=['a','b','c'], ordered=True))

# Accessing categorical properties
series.cat.categories
series.cat.ordered
series.cat.codes

# Modifying categories
series.cat.add_categories(new_cats)
series.cat.remove_categories(old_cats)
series.cat.rename_categories(new_names)
series.cat.reorder_categories(new_order)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype="category"` | Creates an unordered categorical from observed values |
| `CategoricalDtype` | Full constructor with `categories` and `ordered` |
| `.cat` accessor | Access to categorical methods |
| `.cat.categories` | Returns the Index of categories |
| `.cat.codes` | Returns integer codes (−1 for NaN) |
| `.cat.ordered` | Boolean indicating orderedness |

**Syntax Rules:**

- When `dtype="category"` is used without explicit categories, categories are the unique observed values in sorted order.
- All values must be in `categories` or `NaN`; values not in categories become `NaN`.
- Order is defined by the order of categories, not lexical order of values.
- Numerical operations (addition, division) are not possible on categorical data.

**Constraints and Limitations:**

- Cannot perform numerical operations directly; must convert to numeric first.
- Memory savings only significant for low-cardinality columns; high-cardinality columns can use more memory than `object`.
- Categorical data is implemented as a Python object, not a low-level NumPy dtype, which can cause issues with some NumPy functions.

### Annotated Code Examples

**Example 1: Creating Categorical Data**

```python
import pandas as pd

# Create from Series with dtype
s = pd.Series(["a", "b", "c", "a"], dtype="category")
print(s)
# Expected output:
# 0    a
# 1    b
# 2    c
# 3    a
# dtype: category
# Categories (3, object): ['a', 'b', 'c']

# Create with explicit categories and ordering
from pandas.api.types import CategoricalDtype
cat_type = CategoricalDtype(categories=["Low", "Medium", "High"], ordered=True)
s2 = pd.Series(["Low", "High", "Medium", "Low"], dtype=cat_type)
print(s2)
# Expected output:
# 0       Low
# 1      High
# 2    Medium
# 3       Low
# dtype: category
# Categories (3, object): ['Low' < 'Medium' < 'High']
```

**Why this output:** When `dtype="category"` is used without explicit categories, pandas infers unique values and sorts them. When `CategoricalDtype` is provided with `ordered=True`, the categories are ordered as specified, and comparisons and sorting respect this logical order.

**Example 2: Memory Comparison**

```python
import pandas as pd
import numpy as np

# Simulate a low-cardinality column
np.random.seed(42)
values = np.random.choice(["Red", "Green", "Blue"], size=1_000_000)

# Object dtype
s_obj = pd.Series(values, dtype=object)
print(f"Object memory: {s_obj.memory_usage(deep=True) / 1e6:.2f} MB")

# Category dtype
s_cat = pd.Series(values, dtype="category")
print(f"Category memory: {s_cat.memory_usage(deep=True) / 1e6:.2f} MB")

# Expected output (approximate):
# Object memory: 56.00 MB
# Category memory: 1.00 MB
```

**Why this output:** The `object` dtype stores a Python string object for every element, each with overhead. The `category` dtype stores 1 million 1-byte integer codes plus a small categories array, dramatically reducing memory usage for low-cardinality data.

**Example 3: Categorical Sorting and Grouping**

```python
# Logical ordering with categorical
size_order = ["Small", "Medium", "Large"]
sizes = pd.Series(["Medium", "Small", "Large", "Small"], dtype=CategoricalDtype(size_order, ordered=True))

print(sizes.sort_values())
# Expected output:
# 1     Small
# 3     Small
# 0    Medium
# 2     Large
# dtype: category
# Categories (3, object): ['Small' < 'Medium' < 'Large']

# Groupby on categorical
df = pd.DataFrame({"size": sizes, "count": [1, 2, 3, 4]})
print(df.groupby("size", observed=False)["count"].sum())
# Expected output:
# size
# Small     6
# Medium    1
# Large     3
# Name: count, dtype: int64
```

**Why this output:** `sort_values` respects the logical category order (“Small” < “Medium” < “Large”) rather than alphabetical order. `groupby` with `observed=False` includes all categories, even those not present in the data.

### Real-World Cases

- **Survey analysis:** Likert-scale responses (“Strongly Agree” to “Strongly Disagree”) stored as ordered categoricals enable correct ordinal analysis.
- **E-commerce:** Product sizes (XS, S, M, L, XL) use ordered categoricals for correct sorting.
- **Memory-constrained environments:** Categorical encoding of country codes reduces DataFrame memory footprint by orders of magnitude.

### References

- Categorical data — https://pandas.pydata.org/docs/user_guide/categorical.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html

---

## 5. PyArrow-Backed Dtypes

### Definitions

**Core Definition:** PyArrow-backed dtypes are pandas extension dtypes whose arrays are stored as `pyarrow.ChunkedArray` objects instead of NumPy arrays, enabling Arrow-native data types and missing-value support.

**Technical Definition:** The `ArrowDtype` class wraps a `pyarrow.DataType` and is backed by `ArrowExtensionArray`. String aliases like `"int64[pyarrow]"` are accepted in `dtype` parameters. The `"string[pyarrow]"` alias maps to `pd.StringDtype("pyarrow")`, which is distinct from `pd.ArrowDtype(pa.string())`.

**Beginner-Friendly Explanation:** PyArrow dtypes let pandas use Apache Arrow's efficient memory format, which is great for big data and exchanging data with other tools without copying.

### Purposes

- To access a broader set of data types than NumPy provides (e.g., `decimal128`, `list`, `struct`).
- To enable first-class missing value support (`NA`) for all data types.
- To improve performance for IO operations (Parquet, Feather, ORC).
- To facilitate zero-copy interoperability with Polars, DuckDB, and other Arrow-based tools.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
import pyarrow as pa

# String alias
pd.Series(data, dtype="int64[pyarrow]")
pd.Series(data, dtype="float32[pyarrow]")
pd.Series(data, dtype="bool[pyarrow]")

# ArrowDtype constructor
pd.Series(data, dtype=pd.ArrowDtype(pa.int64()))
pd.Series(data, dtype=pd.ArrowDtype(pa.decimal128(10, 2)))
pd.Series(data, dtype=pd.ArrowDtype(pa.list_(pa.int64())))

# StringDtype with pyarrow backend
pd.Series(data, dtype="string[pyarrow]")
pd.StringDtype("pyarrow")
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `"int64[pyarrow]"` | String alias for Arrow int64 |
| `pd.ArrowDtype(pa.int64())` | Full constructor using PyArrow type |
| `pa.decimal128(10, 2)` | Parameterized PyArrow type |
| `pd.StringDtype("pyarrow")` | String dtype with Arrow backend |

**Syntax Rules:**

- The `[pyarrow]` suffix in a string alias tells pandas to use the Arrow backend.
- For parameterized types (e.g., `decimal128`, `timestamp`), use `pd.ArrowDtype` with a `pyarrow.DataType` instance.
- `"string[pyarrow]"` is not equivalent to `pd.ArrowDtype(pa.string())`; the former can return NumPy-backed nullable types, while the latter always returns Arrow-backed arrays.
- PyArrow must be installed; minimum version requirements apply.

**Constraints and Limitations:**

- **Experimental:** The PyArrow backend is experimental; the API may change without warning in future releases.
- Some pandas operations may not fully support Arrow-backed arrays; behavior can differ from NumPy-backed dtypes.
- Requires `pyarrow` installation (`pip install pyarrow`).

### Annotated Code Examples

**Example 1: Creating PyArrow-Backed Series**

```python
import pandas as pd

# Float with Arrow backend
ser = pd.Series([-1.5, 0.2, None], dtype="float32[pyarrow]")
print(ser)
# Expected output:
# 0    -1.5
# 1     0.2
# 2    <NA>
# dtype: float[pyarrow]

# Boolean with Arrow backend
idx = pd.Index([True, None], dtype="bool[pyarrow]")
print(idx)
# Expected output:
# Index([True, <NA>], dtype='bool[pyarrow]')

# DataFrame with Arrow uint64
df = pd.DataFrame([[1, 2], [3, 4]], dtype="uint64[pyarrow]")
print(df)
# Expected output:
#    0  1
# 0  1  2
# 1  3  4

print(df.dtypes)
# Expected output:
# 0    uint64[pyarrow]
# 1    uint64[pyarrow]
# dtype: object
```

**Why this output:** Passing `"[pyarrow]"` aliases to `dtype` instructs pandas to create `ArrowExtensionArray` objects. Missing values are represented as `<NA>` (Arrow's null indicator), not `NaN`. The dtype string includes the `[pyarrow]` suffix.

**Example 2: StringDtype vs ArrowDtype for Strings**

```python
import pandas as pd
import pyarrow as pa

data = list("abc")

# StringDtype with pyarrow backend
ser_sd = pd.Series(data, dtype="string[pyarrow]")
print(ser_sd.dtype)
# Expected output: string[pyarrow]

# ArrowDtype with pyarrow string
ser_ad = pd.Series(data, dtype=pd.ArrowDtype(pa.string()))
print(ser_ad.dtype)
# Expected output: string[pyarrow]

# They are not equal
print(ser_ad.dtype == ser_sd.dtype)
# Expected output: False

# String operation results differ
print(ser_sd.str.contains("a"))
# Expected output:
# 0     True
# 1    False
# 2    False
# dtype: boolean

print(ser_ad.str.contains("a"))
# Expected output:
# 0     True
# 1    False
# 2    False
# dtype: bool[pyarrow]
```

**Why this output:** `pd.StringDtype("pyarrow")` and `pd.ArrowDtype(pa.string())` are distinct dtype classes even though both use Arrow storage. Their string accessor methods return different nullable dtypes: `boolean` vs `bool[pyarrow]`.

**Example 3: Parameterized Arrow Types**

```python
import pandas as pd
import pyarrow as pa

# Decimal128 with precision 10, scale 2
decimal_type = pd.ArrowDtype(pa.decimal128(10, 2))
ser_decimal = pd.Series(["123.45", "678.90"], dtype=decimal_type)
print(ser_decimal)
# Expected output:
# 0    123.45
# 1    678.90
# dtype: decimal128(10, 2)[pyarrow]

# List type
list_type = pd.ArrowDtype(pa.list_(pa.int64()))
ser_list = pd.Series([[1, 2], [3, None]], dtype=list_type)
print(ser_list)
# Expected output:
# 0    [1, 2]
# 1    [3, None]
# dtype: list<item: int64>[pyarrow]
```

**Why this output:** `pd.ArrowDtype` wraps a parameterized PyArrow type (decimal with precision and scale, or list with element type). The dtype string includes the parameters in parentheses followed by `[pyarrow]`.

### Real-World Cases

- **Large-scale analytics:** Reading Parquet files directly into Arrow-backed DataFrames avoids conversion overhead.
- **Decimal precision:** Financial calculations require `decimal128` to avoid floating-point rounding errors.
- **Nested data:** Columns containing lists or structs (e.g., JSON-like data) are naturally supported by Arrow types.
- **Multi-engine workflows:** DataFrames passed between pandas, Polars, and DuckDB using Arrow dtypes avoid serialization costs.

### References

- PyArrow Functionality — https://pandas.pydata.org/docs/user_guide/pyarrow.html
- pandas.ArrowDtype — https://pandas.pydata.org/docs/reference/api/pandas.ArrowDtype.html

---

## 6. dtype Inspection and Conversion

### Definitions

**Core Definition:** dtype inspection and conversion refer to reading the dtype of pandas objects and transforming data from one dtype to another using methods like `astype`, `convert_dtypes`, and `select_dtypes`.

**Technical Definition:** `Series.dtype` returns the `numpy.dtype` or `ExtensionDtype` of the underlying array. `DataFrame.dtypes` returns a Series of column dtypes. `astype` casts to a specified dtype; `convert_dtypes` infers the best nullable dtype; `select_dtypes` filters columns by dtype.

**Beginner-Friendly Explanation:** You can check what type a column is, change it to another type, or pick out only the columns of a certain type—like selecting all numeric columns.

### Purposes

- To verify that data was loaded with the expected types.
- To fix type mismatches (e.g., string numbers → numeric).
- To select subsets of columns for analysis (e.g., only numeric columns).
- To optimize memory by converting to more efficient types.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Inspection
series.dtype
dataframe.dtypes
dataframe.dtypes.value_counts()

# Conversion
series.astype(dtype)
dataframe.astype(dtype)  # entire DataFrame
dataframe.astype({col: dtype, ...})  # per-column

# Automatic conversion
series.convert_dtypes()
dataframe.convert_dtypes()

# Selection
dataframe.select_dtypes(include=None, exclude=None)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `.dtype` | Returns dtype of a Series |
| `.dtypes` | Returns Series of column dtypes |
| `astype(dtype)` | Casts to specified dtype |
| `convert_dtypes()` | Converts to best nullable dtype |
| `select_dtypes(include=[...])` | Selects columns by dtype |
| `select_dtypes(exclude=[...])` | Excludes columns by dtype |

**Syntax Rules:**

- `astype` with `errors='raise'` (default) raises on invalid casts; `errors='ignore'` suppresses and returns the original object.
- Using `astype` to convert timezone-naive to timezone-aware raises an exception; use `.dt.tz_localize()` instead.
- `select_dtypes` accepts dtype strings like `'number'`, `'int64'`, `'object'`, `'category'`, `'datetime'`, and `pd.DatetimeTZDtype`.
- `convert_dtypes` returns nullable extension types (e.g., `Int64`, `string`, `boolean`) that support `pd.NA`.

**Constraints and Limitations:**

- `astype` creates a new object (Copy-on-Write since pandas 3.0); the `copy` parameter is deprecated and ignored.
- Casting from `float` to `int` silently truncates; check for data loss.
- `select_dtypes` with `pd.DatetimeTZDtype` selects all timezone-aware columns; string aliases like `'datetimetz'` are deprecated.

### Annotated Code Examples

**Example 1: dtype Inspection**

```python
import pandas as pd

df = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [1.0, 2.5, 3.7],
    'C': ['x', 'y', 'z'],
    'D': pd.date_range('2024-01-01', periods=3),
    'E': pd.Categorical(['low', 'mid', 'high'])
})

print(df.dtypes)
# Expected output:
# A             int64
# B           float64
# C            object
# D    datetime64[ns]
# E          category
# dtype: object

print(df.dtypes.value_counts())
# Expected output:
# int64           1
# float64         1
# object          1
# datetime64[ns]  1
# category        1
# dtype: int64
```

**Why this output:** `df.dtypes` returns a Series indexed by column name with the dtype of each column. `.value_counts()` counts how many columns share each dtype.

**Example 2: Conversion with astype**

```python
# Convert float column to integer
df = pd.DataFrame({'A': [1.0, 2.5, 3.7], 'B': ['1', '2', '3']})
df['A'] = df['A'].astype('int64')  # Truncates decimals
print(df)

# Expected output:
#    A  B
# 0  1  1
# 1  2  2
# 2  3  3

# Convert string numbers to numeric
df['B'] = pd.to_numeric(df['B'])
print(df.dtypes)

# Expected output:
# A     int64
# B     int64
# dtype: object

# Per-column dtype mapping
df2 = pd.DataFrame({'x': [1, 2], 'y': [3.0, 4.0]})
df2 = df2.astype({'x': 'float64', 'y': 'int64'})
print(df2.dtypes)

# Expected output:
# x    float64
# y      int64
# dtype: object
```

**Why this output:** `astype('int64')` truncates `2.5` to `2`. `pd.to_numeric` safely converts strings to integers. Per-column mapping allows different dtypes for different columns in one call.

**Example 3: select_dtypes**

```python
df = pd.DataFrame({
    'num1': [1, 2, 3],
    'num2': [1.5, 2.5, 3.5],
    'text': ['a', 'b', 'c'],
    'date': pd.date_range('2024-01-01', periods=3),
    'tz_date': pd.date_range('2024-01-01', periods=3, tz='UTC')
})

# Select only numeric columns
print(df.select_dtypes(include='number').columns.tolist())
# Expected output: ['num1', 'num2']

# Select only datetime columns (including tz-aware)
print(df.select_dtypes(include='datetime').columns.tolist())
# Expected output: ['date', 'tz_date']

# Select tz-aware only
print(df.select_dtypes(include=pd.DatetimeTZDtype).columns.tolist())
# Expected output: ['tz_date']

# Exclude numeric columns
print(df.select_dtypes(exclude='number').columns.tolist())
# Expected output: ['text', 'date', 'tz_date']
```

**Why this output:** `select_dtypes` filters columns by dtype category. `'number'` matches all numeric types. `'datetime'` matches both naive and aware datetimes. `pd.DatetimeTZDtype` matches only timezone-aware columns.

### Real-World Cases

- **Data validation:** Check that a loaded CSV has the expected numeric and datetime columns before analysis.
- **Feature engineering:** Select only numeric columns for correlation matrices.
- **Memory optimization:** Convert `object` columns to `category` or `str` to reduce memory.

### References

- pandas.DataFrame.astype — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html
- pandas.DataFrame.select_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.select_dtypes.html
- pandas.Series.convert_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.Series.convert_dtypes.html

---

## References

- pandas arrays, scalars, and data types — https://pandas.pydata.org/pandas-docs/version/2.2/reference/arrays.html
- Time series / date functionality — https://pandas.pydata.org/docs/user_guide/timeseries.html
- Migration guide for the new string data type (pandas 3.0) — https://pandas.pydata.org/docs/user_guide/migration-3-strings.html
- Categorical data — https://pandas.pydata.org/docs/user_guide/categorical.html
- PyArrow Functionality — https://pandas.pydata.org/docs/user_guide/pyarrow.html
- pandas.DataFrame.astype — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html
- pandas.CategoricalDtype — https://pandas.pydata.org/docs/reference/api/pandas.CategoricalDtype.html
- pandas.ArrowDtype — https://pandas.pydata.org/docs/reference/api/pandas.ArrowDtype.html
- What's new in 3.0.0 — https://pandas.pydata.org/docs/whatsnew/v3.0.0.html
- PDEP-14: Dedicated string data type for pandas 3.0 — https://pandas.pydata.org/pdeps/0014-string-dtype.html