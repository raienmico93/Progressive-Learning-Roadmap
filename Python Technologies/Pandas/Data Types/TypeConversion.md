# Pandas Type Conversion: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Type conversion in pandas is the process of transforming data from one dtype to another, either explicitly through casting methods or implicitly through parsing functions, to ensure data is stored in the most appropriate format for analysis.

**Technical Definition:** Type conversion encompasses explicit casting via `astype()` (which creates a new object with the specified `numpy.dtype`, `ExtensionDtype`, or Python type) and error-tolerant parsing via `pd.to_numeric()`, `pd.to_datetime()`, and `pd.to_timedelta()` (which use format-aware parsers and an `errors` parameter to control failure behaviour). Downcasting via the `downcast` parameter in `to_numeric` selects the smallest safe numerical dtype, while `convert_dtypes()` infers the best nullable extension dtype for each column.

**Beginner-Friendly Explanation:** Type conversion is how you tell pandas to change a column from one kind of data to another—like turning text that looks like numbers into actual numbers, or strings that look like dates into real datetime objects. It also lets you shrink memory usage by choosing smaller number types when your data doesn't need the full range.

### Key Characteristics

- **Explicit vs. error-tolerant:** `astype()` raises errors on invalid data unless `errors='ignore'` is set; parsing functions (`to_numeric`, `to_datetime`, `to_timedelta`) offer `errors='coerce'` to replace failures with `NaN`/`NaT`.
- **Per-column control:** `astype()` accepts a dictionary mapping column names to dtypes, enabling selective conversion across a DataFrame.
- **Memory optimization:** Downcasting reduces numeric columns to the smallest dtype that can safely hold all values.
- **Format-aware date parsing:** `to_datetime()` accepts strftime format strings for fast, unambiguous parsing; `utc=True` resolves mixed-timezone inputs.
- **Version-specific behaviours:** The `errors='ignore'` option is deprecated in pandas 2.2+ for `to_numeric`; the `copy` parameter of `astype` is ignored as of pandas 3.0.

### Prerequisites

- Basic Python knowledge (functions, dictionaries, string formatting).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of numpy dtypes (`int64`, `float64`, `object`, etc.).
- Installation of pandas (`pip install pandas`); PyArrow optional for Arrow-backed dtypes.

### Related Programming Areas

- **Data Cleaning & Preprocessing:** Converting strings to numbers or dates is a core step in ETL pipelines.
- **Memory Management:** Downcasting and nullable dtypes reduce RAM usage in large datasets.
- **Time Series Analysis:** Accurate datetime parsing is essential for resampling and rolling windows.
- **Data Validation:** `errors='coerce'` surfaces data quality issues as `NaN`/`NaT` for inspection.

---

## 1. Standard Conversion: Explicit Casting with `astype()`

### Definitions

**Core Definition:** `astype()` explicitly casts a pandas Series or DataFrame to a specified dtype, either globally or per-column via a dictionary mapping.

**Technical Definition:** `DataFrame.astype(dtype, copy=<no_default>, errors='raise')` accepts a `str`, `numpy.dtype`, `pandas.ExtensionDtype`, Python type, or a mapping of column label → dtype. It returns a new object with the cast dtypes; the `copy` parameter is ignored as of pandas 3.0 due to Copy-on-Write.

**Beginner-Friendly Explanation:** `astype()` is the “force the type” button. You can cast every column to the same type, or give different types to different columns using a dictionary.

### Purposes

- To convert a Series or entire DataFrame to a single dtype in one call.
- To apply different dtypes to different columns using a dictionary mapping.
- To explicitly control the storage type when inference produces an unwanted dtype.
- To enable type-specific operations (e.g., numeric arithmetic after casting from object).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Entire object to one dtype
DataFrame.astype(dtype, errors='raise')
Series.astype(dtype, errors='raise')

# Per-column mapping
DataFrame.astype({col1: dtype1, col2: dtype2, ...}, errors='raise')
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `dtype` | `str`, `numpy.dtype`, `ExtensionDtype`, Python type, or mapping `{col: dtype}` |
| `errors` | `'raise'` (default) raises on invalid cast; `'ignore'` returns the original object |
| `copy` | Deprecated since pandas 3.0; ignored. Method always returns a new object using lazy copy |

**Syntax Rules:**

- A Python type such as `int` maps to the corresponding dtype; `datetime.datetime` raises `TypeError`.
- Columns absent from the mapping are left unchanged.
- Casting timezone-naive datetimes to timezone-aware via `astype` raises an exception; use `.dt.tz_localize()` instead.

**Constraints and Limitations:**

- Casting `float` to `int` silently truncates fractional parts.
- `astype` does not parse strings to datetimes; use `pd.to_datetime` for that.
- `errors='ignore'` suppresses all exceptions, potentially hiding data issues.

### Annotated Code Examples

**Example 1: Casting an Entire DataFrame**

```python
import pandas as pd

d = {"col1": [1, 2], "col2": [3, 4]}
df = pd.DataFrame(data=d)
print(df.dtypes)
# Expected output:
# col1    int64
# col2    int64
# dtype: object

# Cast all columns to int32
df_cast = df.astype("int32")
print(df_cast.dtypes)
# Expected output:
# col1    int32
# col2    int32
# dtype: object
```

**Why this output:** The default integer dtype is `int64`. Passing `"int32"` casts both columns to the smaller 32-bit integer dtype, reducing memory usage for values that fit within ±2,147,483,647.

**Example 2: Per-Column Casting with a Dictionary**

```python
df = pd.DataFrame({"col1": [1, 2], "col2": [3, 4]})

# Cast only col1 to int32; col2 remains int64
df_typed = df.astype({"col1": "int32"})
print(df_typed.dtypes)
# Expected output:
# col1    int32
# col2    int64
# dtype: object
```

**Why this output:** The dictionary `{"col1": "int32"}` specifies the dtype for `col1` only. `col2` is not present in the mapping and therefore retains its original `int64` dtype.

**Example 3: Error Handling with `errors='ignore'`**

```python
# Create a Series with a non-numeric string
s = pd.Series(["1", "2", "foo"])

# Attempt to cast to int — raises ValueError
try:
    s.astype("int64")
except ValueError as e:
    print(f"Error: {e}")

# Use errors='ignore' to return the original object
s_original = s.astype("int64", errors="ignore")
print(s_original)
# Expected output:
# 0      1
# 1      2
# 2    foo
# dtype: object
```

**Why this output:** `"foo"` cannot be parsed as an integer, so the default `errors='raise'` raises a `ValueError`. With `errors='ignore'`, pandas suppresses the exception and returns the original Series unchanged.

### Real-World Cases

- **Survey data:** Likert-scale responses stored as strings (“1”, “2”, “3”) are cast to `int64` for statistical analysis.
- **ID columns:** Numeric IDs are cast to `str` to prevent arithmetic operations and preserve leading zeros.
- **Memory optimization:** Large DataFrames with small integer ranges are cast from `int64` to `int8` or `int16`.

### References

- pandas.DataFrame.astype — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html

---

## 2. Error-Tolerant Conversions: `pd.to_numeric()`, `pd.to_datetime()`, and `pd.to_timedelta()`

### Definitions

**Core Definition:** Error-tolerant conversion functions parse data into numeric, datetime, or timedelta types while providing an `errors` parameter to control what happens when parsing fails.

**Technical Definition:** `pd.to_numeric(arg, errors='raise', downcast=None, dtype_backend=...)` converts scalars, lists, or Series to numeric types. `pd.to_datetime(arg, errors='raise', utc=False, format=None, ...)` parses to `datetime64[ns]` or `DatetimeTZDtype`. `pd.to_timedelta(arg, unit=None, errors='raise')` converts to `timedelta64[ns]`. All three support `errors='coerce'` (invalid → `NaN`/`NaT`) and `errors='ignore'` (deprecated for `to_numeric` and `to_timedelta`).

**Beginner-Friendly Explanation:** These functions are “smart converters” that can deal with messy data. Instead of crashing on bad values, you can tell them to turn bad values into `NaN` or `NaT` so you can clean them up later.

### Purposes

- To convert string representations of numbers into actual numeric dtypes.
- To parse date and time strings into `datetime64[ns]` or timezone-aware datetimes.
- To convert duration strings or numbers into `timedelta64[ns]`.
- To handle malformed data gracefully using `errors='coerce'`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
pd.to_numeric(arg, errors='raise', downcast=None, dtype_backend='numpy_nullable')
pd.to_datetime(arg, errors='raise', dayfirst=False, yearfirst=False,
               utc=False, format=None, exact=True, unit=None, origin='unix')
pd.to_timedelta(arg, unit=None, errors='raise')
```

**Component Breakdown:**

| Function | Key Parameters |
|----------|----------------|
| `to_numeric` | `arg` (scalar/list/Series), `errors` (`'raise'`/`'coerce'`/`'ignore'`), `downcast` (`'integer'`/`'signed'`/`'unsigned'`/`'float'`) |
| `to_datetime` | `arg`, `errors` (`'raise'`/`'coerce'`/`'ignore'`), `utc` (bool), `format` (strftime string or `'mixed'`/`'ISO8601'`), `dayfirst`, `yearfirst`, `unit` |
| `to_timedelta` | `arg` (str/timedelta/list-like/Series), `unit` (`'D'`/`'s'`/`'ms'`/`'us'`/`'ns'`), `errors` (`'raise'`/`'coerce'`) |

**Syntax Rules:**

- `errors='coerce'` replaces invalid parsing with `NaN` (numeric) or `NaT` (datetime/timedelta).
- `errors='ignore'` returns the input unchanged on failure; deprecated for `to_numeric` in pandas 2.2+.
- `to_datetime` with `format` specified is significantly faster than inference (6–7× faster in benchmarks).
- `utc=True` converts timezone-aware inputs to UTC and localizes naive inputs as UTC.

**Constraints and Limitations:**

- `to_numeric` may convert very large integers to `float64` due to `ndarray` limits.
- `to_datetime` with mixed timezones and `utc=False` raises `ValueError` in pandas 2.0+.
- `to_timedelta` with strings containing `'M'`, `'Y'`, or `'y'` raises an exception in pandas 2.0+ as these are ambiguous.

### Annotated Code Examples

**Example 1: `pd.to_numeric` with `errors='coerce'`**

```python
import pandas as pd

# Series with mixed numeric and non-numeric values
s = pd.Series(["1.0", "2", "-3.14", "apple"])

# errors='raise' (default) raises ValueError
try:
    pd.to_numeric(s)
except ValueError as e:
    print(f"Error: {e}")

# errors='coerce' turns invalid values into NaN
s_num = pd.to_numeric(s, errors="coerce")
print(s_num)
# Expected output:
# 0    1.00
# 1    2.00
# 2   -3.14
# 3     NaN
# dtype: float64
```

**Why this output:** `"apple"` cannot be parsed as a number, so `errors='coerce'` replaces it with `NaN`. The remaining values are converted to `float64`, the default numeric dtype.

**Example 2: `pd.to_datetime` with Format and `errors='coerce'`**

```python
# Date strings with a consistent format
dates = pd.Series(["2025-01-15", "2025-02-20", "not-a-date"])

# Specify format for fast, unambiguous parsing; coerce errors to NaT
parsed = pd.to_datetime(dates, format="%Y-%m-%d", errors="coerce")
print(parsed)
# Expected output:
# 0   2025-01-15
# 1   2025-02-20
# 2          NaT
# dtype: datetime64[ns]
```

**Why this output:** The `format="%Y-%m-%d"` directive tells pandas the exact expected pattern. `"not-a-date"` does not match, so `errors='coerce'` converts it to `NaT`.

**Example 3: `pd.to_timedelta` with Unit Specification**

```python
import numpy as np

# Convert numeric values with unit='D' (days)
td = pd.to_timedelta(np.arange(3), unit="D")
print(td)
# Expected output:
# TimedeltaIndex(['0 days', '1 days', '2 days'],
#                dtype='timedelta64[s]', freq=None)

# Convert a duration string
td2 = pd.to_timedelta("1 days 06:05:01.00003")
print(td2)
# Expected output: Timedelta('1 days 06:05:01.000030')
```

**Why this output:** When `arg` contains numbers, `unit` specifies the time unit. `np.arange(3)` with `unit="D"` produces 0, 1, and 2 days. String inputs are parsed directly without needing `unit`.

### Real-World Cases

- **Financial data:** CSV columns with currency strings (“$1,234.56”) are cleaned and converted with `to_numeric(errors='coerce')`.
- **Log analysis:** Timestamp columns with occasional malformed entries are parsed with `to_datetime(errors='coerce')` to isolate bad rows.
- **IoT sensor data:** Duration columns stored as seconds are converted with `to_timedelta(unit='s')` for time-series analysis.

### References

- pandas.to_numeric — https://pandas.pydata.org/docs/reference/api/pandas.to_numeric.html
- pandas.to_datetime — https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html
- pandas.to_timedelta — https://pandas.pydata.org/docs/reference/api/pandas.to_timedelta.html

---

## 3. Downcasting Numerical Collections to Minimize Memory Footprint

### Definitions

**Core Definition:** Downcasting is the process of converting a numeric column to the smallest dtype that can safely represent all its values, reducing memory usage without data loss.

**Technical Definition:** The `downcast` parameter in `pd.to_numeric()` accepts `'integer'`, `'signed'`, `'unsigned'`, or `'float'`. For integers, it selects the smallest signed (`int8`–`int64`) or unsigned (`uint8`–`uint64`) dtype based on the data's min/max. For floats, it selects `float32` if the data can be represented without precision loss. Downcasting only occurs if the resulting dtype is strictly smaller than the original.

**Beginner-Friendly Explanation:** If your numbers are all between 0 and 100, there's no reason to store them in a type that can handle billions. Downcasting picks the smallest box that still fits all your numbers.

### Purposes

- To reduce the memory footprint of large DataFrames by 50–80% without losing information.
- To speed up computation by using smaller, cache-friendly data types.
- To prepare data for memory-constrained environments (e.g., embedded systems, mobile devices).
- To complement other memory-saving techniques such as categorical encoding.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
pd.to_numeric(series, downcast='integer')   # smallest signed int
pd.to_numeric(series, downcast='unsigned')  # smallest unsigned int
pd.to_numeric(series, downcast='float')     # smallest float
```

**Component Breakdown:**

| `downcast` Value | Behaviour |
|------------------|-----------|
| `'integer'` or `'signed'` | Smallest signed int dtype (min. `int8`) |
| `'unsigned'` | Smallest unsigned int dtype (min. `uint8`) |
| `'float'` | Smallest float dtype (min. `float32`) |

**Syntax Rules:**

- Downcasting occurs only if the resulting dtype is strictly smaller than the original.
- Errors during downcasting are surfaced regardless of the `errors` parameter.
- For DataFrames, apply `to_numeric(downcast=...)` column by column using `select_dtypes()` to target numeric columns.

**Constraints and Limitations:**

- Downcasting `float64` to `float32` may introduce precision loss for values requiring more than ~7 significant digits.
- Downcasting integers does not handle missing values (`NaN`); use nullable integer dtypes if NA values are present.
- Very large values outside the selected dtype's range cause overflow errors.

### Annotated Code Examples

**Example 1: Downcasting Integer Columns**

```python
import pandas as pd

# Create a DataFrame with int64 columns
df = pd.DataFrame({
    "small_vals": [1, 2, 3, 4, 5],
    "large_vals": [100000, 200000, 300000, 400000, 500000]
})

print("Before:", df.dtypes.tolist())
# Expected output: ['int64', 'int64']

for col in df.select_dtypes(include=["int"]).columns:
    df[col] = pd.to_numeric(df[col], downcast="integer")

print("After:", df.dtypes.tolist())
# Expected output: ['int8', 'int32']
```

**Why this output:** `small_vals` has a range of 1–5, which fits in `int8` (range −128 to 127). `large_vals` has a range of 100,000–500,000, which requires `int32`. Downcasting selects the smallest dtype per column.

**Example 2: Downcasting Float Columns**

```python
df = pd.DataFrame({
    "precise": [1.123456789, 2.987654321, 3.141592653],
    "rough": [1.1, 2.2, 3.3]
})

for col in df.select_dtypes(include=["float"]).columns:
    df[col] = pd.to_numeric(df[col], downcast="float")

print(df.dtypes)
# Expected output:
# precise    float64
# rough      float32
# dtype: object
```

**Why this output:** `rough` contains values with few significant digits, so `float32` is sufficient. `precise` contains values with many decimal places that would lose precision in `float32`, so it remains `float64`.

**Example 3: Memory Impact Measurement**

```python
df = pd.DataFrame({
    "id": range(1_000_000),
    "score": [i % 100 for i in range(1_000_000)]
})

before = df.memory_usage(deep=True).sum()
for col in df.select_dtypes(include=["int"]).columns:
    df[col] = pd.to_numeric(df[col], downcast="integer")
after = df.memory_usage(deep=True).sum()

print(f"Before: {before / 1e6:.2f} MB")
print(f"After:  {after / 1e6:.2f} MB")
print(f"Reduction: {(1 - after/before) * 100:.1f}%")
# Expected output (approximate):
# Before: 16.00 MB
# After:  3.00 MB
# Reduction: 81.2%
```

**Why this output:** `id` (0–999,999) fits in `int32`, and `score` (0–99) fits in `int8`. Both columns shrink from 8 bytes per element (`int64`) to 1–4 bytes, dramatically reducing memory usage.

### Real-World Cases

- **Large CSV datasets:** Downcasting after `read_csv` can reduce a 10 GB DataFrame to 2–3 GB, making it fit in RAM.
- **Machine learning pipelines:** Smaller dtypes reduce GPU memory usage when transferring data to accelerators.
- **Embedded analytics:** Downcasting enables pandas DataFrames to run on devices with limited memory.

### References

- pandas.to_numeric (downcast parameter) — https://pandas.pydata.org/docs/reference/api/pandas.to_numeric.html
- 7 Pandas Tricks to Handle Large Datasets — https://machinelearningmastery.com/7-pandas-tricks-to-handle-large-datasets/

---

## 4. Infinitely Structured Date Parsing: Format Strings and Mixed-Offset Timezones

### Definitions

**Core Definition:** Structured date parsing uses explicit strftime format strings in `pd.to_datetime()` for fast, unambiguous parsing, and `utc=True` to handle mixed-timezone inputs by converting them to a common UTC representation.

**Technical Definition:** The `format` parameter accepts strftime directives (e.g., `%Y`, `%m`, `%d`, `%H`, `%M`, `%S`, `%z`, `%Z`) and, in pandas 2.0+, the special values `'mixed'` (infer format per element) and `'ISO8601'`. The `utc=True` parameter localizes naive inputs as UTC and converts timezone-aware inputs to UTC, resolving mixed-offset ambiguities that would otherwise raise `ValueError`.

**Beginner-Friendly Explanation:** If you know exactly how your dates look (e.g., “day/month/year”), you can give pandas a format string so it parses correctly and quickly. If your data has dates from different timezones, `utc=True` lines them all up on the same clock.

### Purposes

- To parse date strings quickly and unambiguously by specifying the exact format.
- To resolve date ambiguity (e.g., “01/02/2025” as January 2nd vs. February 1st).
- To handle mixed-timezone data by converting all timestamps to UTC.
- To parse non-standard date representations using custom strftime directives.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
pd.to_datetime(
    arg,
    errors='raise',
    dayfirst=False,
    yearfirst=False,
    utc=False,
    format=None,       # strftime string, 'mixed', or 'ISO8601'
    exact=True,
    unit=None,
    origin='unix'
)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `format` | strftime-compatible format string; `None` infers; `'mixed'` infers per element; `'ISO8601'` requires ISO format |
| `utc` | If `True`, returns UTC-localized `DatetimeIndex`; converts aware inputs to UTC |
| `dayfirst` | If `True`, parses “01/02/2025” as February 1st |
| `yearfirst` | If `True`, parses “25/01/02” as 2025-01-02; takes precedence over `dayfirst` |
| `exact` | If `True` (default), format must match exactly; if `False`, allows substring matching |

**Syntax Rules:**

- Common strftime directives: `%Y` (4-digit year), `%m` (month), `%d` (day), `%H` (hour), `%M` (minute), `%S` (second), `%z` (UTC offset like `+0000`), `%Z` (timezone name like `UTC`).
- For mixed timezones, `utc=True` is required in pandas 2.0+; otherwise a `ValueError` is raised.
- A mix of timezone-aware and timezone-naive inputs also raises `ValueError` unless `utc=True` is set.
- Specifying `format` is 6–7× faster than letting pandas infer the format for large datasets.

**Constraints and Limitations:**

- `format='mixed'` is risky and may produce incorrect results if formats overlap ambiguously.
- `utc=True` converts all timestamps to UTC, losing the original local timezone information (though the moment in time is preserved).
- `%z` must match the exact offset format in the data (e.g., `+0000` vs. `+00:00`).

### Annotated Code Examples

**Example 1: Custom Format String Parsing**

```python
import pandas as pd

# Dates in dd/mm/yyyy format
dates = pd.Series(["15/01/2025", "20/06/2025", "31/12/2025"])

# Parse with explicit format
parsed = pd.to_datetime(dates, format="%d/%m/%Y")
print(parsed)
# Expected output:
# 0   2025-01-15
# 1   2025-06-20
# 2   2025-12-31
# dtype: datetime64[ns]

# Without format, dayfirst=True is needed for correct parsing
ambiguous = pd.Series(["01/02/2025"])
print(pd.to_datetime(ambiguous))                # January 2nd (US default)
print(pd.to_datetime(ambiguous, dayfirst=True)) # February 1st
# Expected output:
# 0   2025-01-02
# dtype: datetime64[ns]
# 0   2025-02-01
# dtype: datetime64[ns]
```

**Why this output:** The format string `%d/%m/%Y` explicitly tells pandas that the first component is the day, the second is the month, and the third is the year. Without it, pandas defaults to month-first (US convention).

**Example 2: Mixed Timezones with `utc=True`**

```python
# Timestamps with different UTC offsets
mixed = pd.Series([
    "2020-10-25 02:00 +0200",
    "2020-10-25 04:00 +0100"
])

# Without utc=True, pandas raises ValueError
try:
    pd.to_datetime(mixed, format="%Y-%m-%d %H:%M %z")
except ValueError as e:
    print(f"Error: {e}")

# With utc=True, all timestamps are converted to UTC
result = pd.to_datetime(mixed, format="%Y-%m-%d %H:%M %z", utc=True)
print(result)
# Expected output:
# 0   2020-10-25 00:00:00+00:00
# 1   2020-10-25 03:00:00+00:00
# dtype: datetime64[ns, UTC]
```

**Why this output:** The two timestamps represent the same wall-clock moment but different UTC offsets. Without `utc=True`, pandas cannot reconcile them and raises `ValueError`. With `utc=True`, both are converted to UTC: `02:00+02:00` becomes `00:00 UTC`, and `04:00+01:00` becomes `03:00 UTC`.

**Example 3: Parsing ISO8601 with `format='ISO8601'`**

```python
iso_dates = pd.Series([
    "2025-01-15T14:30:00+00:00",
    "2025-06-20T09:00:00+02:00"
])

result = pd.to_datetime(iso_dates, format="ISO8601", utc=True)
print(result)
# Expected output:
# 0   2025-01-15 14:30:00+00:00
# 1   2025-06-20 07:00:00+00:00
# dtype: datetime64[ns, UTC]
```

**Why this output:** `format='ISO8601'` tells pandas to require ISO 8601 compliance. `utc=True` converts the `+02:00` offset to UTC, shifting `09:00` to `07:00 UTC`. This is the fastest parsing mode for ISO-formatted data.

### Real-World Cases

- **International datasets:** European date formats (“dd/mm/yyyy”) are parsed with `format='%d/%m/%Y'` or `dayfirst=True`.
- **Distributed systems:** Logs from servers in different timezones are normalized to UTC with `utc=True` for consistent analysis.
- **API responses:** ISO 8601 timestamps are parsed with `format='ISO8601'` for maximum speed and correctness.
- **Financial data:** Market timestamps with exchange-specific offsets are converted to UTC for cross-market comparison.

### References

- pandas.to_datetime — https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html
- Time series / date functionality — https://pandas.pydata.org/docs/user_guide/timeseries.html

---

## References

- pandas.DataFrame.astype — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html
- pandas.to_numeric — https://pandas.pydata.org/docs/reference/api/pandas.to_numeric.html
- pandas.to_datetime — https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html
- pandas.to_timedelta — https://pandas.pydata.org/docs/reference/api/pandas.to_timedelta.html
- pandas.DataFrame.convert_dtypes — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.convert_dtypes.html
- Time series / date functionality — https://pandas.pydata.org/docs/user_guide/timeseries.html
- 7 Pandas Tricks to Handle Large Datasets — https://machinelearningmastery.com/7-pandas-tricks-to-handle-large-datasets/
- Pandas to_datetime: Convert Strings, Timestamps, and Mixed Formats — https://docs.kanaries.net/topics/Pandas/pandas-to-datetime