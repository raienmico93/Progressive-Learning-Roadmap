# Comprehensive Programming Cheat Sheet: Window Frames

## Topic Overview

### Definitions

**Core Definition**  
A window frame is the subset of rows within a partition that a window function actually uses to compute a value for a given row.

**Technical Definition**  
A window frame is defined by the frame clause of a window specification. It specifies a start bound and an end bound relative to the current row, expressed in one of three frame units: `ROWS` (physical row offsets), `RANGE` (logical value offsets based on the `ORDER BY` column), or `GROUPS` (peer-group offsets). The frame determines which rows participate in the window function’s calculation for each row. Optional frame exclusion clauses (`EXCLUDE`) remove rows from the frame after the bounds are applied. Default frames depend on the presence of `ORDER BY`: without `ORDER BY`, the entire partition is the frame; with `ORDER BY` and no explicit frame, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which includes all rows from the partition start through the current row’s last peer.

**Beginner-Friendly Explanation**  
Imagine you are looking at a long receipt and want to calculate a running total. The “frame” tells you which lines of the receipt to include: from the very first line up to the line you are currently on, or perhaps just the previous two lines and the next one. The frame is the slice of rows the calculation actually sees.

### Key Characteristics

- **Current-row-relative**: Frames are defined relative to the current row and move as the current row advances through the partition.
- **Three frame units**: `ROWS` (physical rows), `RANGE` (value-based), and `GROUPS` (peer-group-based).
- **Two bound positions**: Frame start and frame end, each of which can be `UNBOUNDED PRECEDING`, `n PRECEDING`, `CURRENT ROW`, `n FOLLOWING`, or `UNBOUNDED FOLLOWING`.
- **Default framing**: With `ORDER BY`, the default is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`; without `ORDER BY`, the entire partition is the frame.
- **Optional exclusion**: `EXCLUDE` removes the current row, its peer group, or its ties from the frame after bounds are applied.
- **Engine variation**: MySQL 8.0 supports only `ROWS` and `RANGE` (not `GROUPS` or `EXCLUDE`); SQL Server supports `ROWS` and `RANGE` but not `GROUPS` or `EXCLUDE`; PostgreSQL supports all three units and all exclusion modes.
- **Performance sensitivity**: `ROWS` is typically faster than `RANGE`, especially in SQL Server, because `RANGE` may require spooling for peer detection.

### Prerequisites

- Understanding of window function fundamentals, `OVER`, `PARTITION BY`, and `ORDER BY` in windows.
- Familiarity with ranking, aggregate, and value functions.
- Awareness of the target engine’s frame support matrix.
- Knowledge of the logical processing order of `SELECT` statements.

### Related Programming Areas

- Running totals and cumulative aggregates.
- Moving averages and rolling calculations.
- Time-series analysis and gap filling.
- Ranking with peer awareness.
- Query optimisation and execution-plan analysis.

---

## Core Concepts and Features

### 1. `ROWS`

#### Definitions

**Core Definition**  
`ROWS` is a frame unit that defines the window in terms of physical row positions relative to the current row.

**Technical Definition**  
In `ROWS` mode, offsets are integers counting physical rows before (`PRECEDING`) or after (`FOLLOWING`) the current row. `CURRENT ROW` means the current physical row. `ROWS` does not consider peer groups: rows are treated as individual physical entities in the order specified by the window `ORDER BY`. The `ROWS` unit can produce non-deterministic results if the `ORDER BY` does not uniquely order rows.

**Beginner-Friendly Explanation**  
`ROWS` means “count actual lines.” If you say `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`, the calculation looks at the current line and the two lines physically above it, regardless of whether those lines have the same values.

#### Purposes

- To compute running totals with strict row-by-row accumulation.
- To compute moving averages over a fixed number of rows.
- To avoid peer-inclusion surprises when `ORDER BY` has duplicates.
- To achieve faster performance compared to `RANGE` in many engines.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS } BETWEEN frame_start AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `ROWS` | Frame unit keyword. |
| `BETWEEN ... AND ...` | Defines start and end bounds. |
| `UNBOUNDED PRECEDING` | First row of the partition. |
| `n PRECEDING` | `n` physical rows before the current row. |
| `CURRENT ROW` | The current physical row. |
| `n FOLLOWING` | `n` physical rows after the current row. |
| `UNBOUNDED FOLLOWING` | Last row of the partition. |

**Syntax rules**

- Offsets must be non-negative integers.
- `frame_start` cannot be `UNBOUNDED FOLLOWING`; `frame_end` cannot be `UNBOUNDED PRECEDING`.
- `ROWS` ignores peers; each physical row is distinct.
- `ROWS` is supported by PostgreSQL, MySQL 8.0, SQL Server, Oracle, and SQLite.

**Constraints and limitations**

- Non-unique `ORDER BY` can make `ROWS` results non-deterministic because “which row is third” may vary.
- `ROWS` is generally faster than `RANGE` in SQL Server because it avoids spooling for peer detection.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Strict row-by-row running total

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rows_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | rows_sum
--------+----------
   3500 |     3500
   3900 |     7400
   4200 |    11600
   4500 |    16100
   4800 |    20900
   4800 |    25700
   5000 |    30700
   5200 |    35900
   5200 |    41100
   6000 |    47100
(10 rows)
```

**Why this output appears**: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` accumulates one physical row at a time. The two 4800 rows produce different running totals (20900 and 25700), because `ROWS` does not group peers.

##### Example 2: MySQL 8.0 – Moving average with `ROWS`

```sql
SELECT time, subject, val,
       AVG(val) OVER (PARTITION BY subject ORDER BY time ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS running_average
FROM observations;
```

**Expected output** (excerpt):

```text
+----------+---------+------+-----------------+
| time     | subject | val  | running_average |
+----------+---------+------+-----------------+
| 07:00:00 | st113   |   10 |          9.5000 |
| 07:15:00 | st113   |    9 |         14.6667 |
| 07:30:00 | st113   |   25 |         18.0000 |
| 07:45:00 | st113   |   20 |         22.5000 |
+----------+---------+------+-----------------+
```

**Why this output appears**: The frame includes the current row and one row before and after it; the average is computed over at most three rows. The first row has no preceding row, so the average uses only available rows.

#### Real-World Cases

- **Running totals**: Cumulative sums where each row must add to the previous total exactly once.
- **Moving averages**: Fixed-window smoothing over the previous `n` rows.
- **Row-based analytics**: Calculations that must not be affected by duplicate values.

---

### 2. `RANGE`

#### Definitions

**Core Definition**  
`RANGE` is a frame unit that defines the window in terms of logical value ranges relative to the current row’s `ORDER BY` value.

**Technical Definition**  
In `RANGE` mode, bounds are defined by values of the window `ORDER BY` column. `CURRENT ROW` means the current row and all its peers (rows with equal `ORDER BY` values). `n PRECEDING` and `n FOLLOWING` define value offsets: rows whose `ORDER BY` value is within `n` units of the current row’s value are included. `RANGE` with offsets requires exactly one `ORDER BY` column in most engines. The offset data type matches the ordering column (integer for numeric, interval for datetime).

**Beginner-Friendly Explanation**  
`RANGE` means “include rows whose values are close enough.” If you order by date and say `RANGE BETWEEN '7 days' PRECEDING AND CURRENT ROW`, you get all rows from a week ago up to today, regardless of how many physical rows that involves.

#### Purposes

- To compute value-based windows (e.g., last 7 days, last 500 units).
- To include peers automatically when `ORDER BY` has duplicates.
- To produce running totals where tied values are treated consistently.
- To support time-series and financial calculations with value offsets.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ RANGE } BETWEEN frame_start AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `RANGE` | Frame unit keyword. |
| `UNBOUNDED PRECEDING` | Start of the partition. |
| `n PRECEDING` | Rows with `ORDER BY` value ≥ current value − `n`. |
| `CURRENT ROW` | Current row and all its peers. |
| `n FOLLOWING` | Rows with `ORDER BY` value ≤ current value + `n`. |
| `UNBOUNDED FOLLOWING` | End of the partition. |

**Syntax rules**

- Offset `n` must be non-null and non-negative.
- `RANGE` with offsets requires exactly one `ORDER BY` column in PostgreSQL and most engines.
- Offset cannot contain variables, aggregate functions, or window functions.
- `RANGE` is supported by PostgreSQL, MySQL 8.0, SQL Server, and Oracle.

**Constraints and limitations**

- `RANGE` may require peer detection and spooling, making it slower than `ROWS` in some engines.
- `RANGE` with offsets is not supported by all engines (e.g., SQL Server does not support `RANGE` with numeric offsets in window frames; only `UNBOUNDED` and `CURRENT ROW` bounds are allowed).
- Without `ORDER BY`, the entire partition is the frame, and `RANGE` has no value basis for offsets.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `RANGE` with a date interval

```sql
SELECT date, revenue,
       SUM(revenue) OVER (ORDER BY date RANGE BETWEEN '7 days' PRECEDING AND CURRENT ROW) AS week_revenue
FROM daily_sales;
```

**Expected output** (illustrative):

```text
    date    | revenue | week_revenue
------------+---------+--------------
 2025-01-01 |     100 |          100
 2025-01-03 |     200 |          300
 2025-01-05 |     150 |          450
 2025-01-08 |     300 |          650
 2025-01-10 |     250 |          900
```

**Why this output appears**: The frame includes all rows whose `date` is within 7 days before the current row’s date. The offset type is an interval because the ordering column is a date.

##### Example 2: PostgreSQL – Default `RANGE` frame includes peers

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary) AS default_range_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | default_range_sum
--------+-------------------
   3500 |              3500
   3900 |              7400
   4200 |             11600
   4500 |             16100
   4800 |             25700
   4800 |             25700
   5000 |             30700
   5200 |             41100
   5200 |             41100
   6000 |             47100
(10 rows)
```

**Why this output appears**: The default frame with `ORDER BY` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which includes all rows from the start through the current row’s last peer. The two 4800 rows both see the same cumulative sum (25700), unlike the `ROWS` example.

#### Real-World Cases

- **Sliding time windows**: Revenue over the last 7 or 30 days.
- **Peer-aware running totals**: Cumulative sums where ties must be treated equally.
- **Financial calculations**: Value-range windows around a price or quantity.

---

### 3. `GROUPS`

#### Definitions

**Core Definition**  
`GROUPS` is a frame unit that defines the window in terms of peer groups, counting groups of rows that are equivalent under the window `ORDER BY`.

**Technical Definition**  
In `GROUPS` mode, offsets count peer groups rather than physical rows or value ranges. `CURRENT ROW` means the current row’s entire peer group. `n PRECEDING` means the frame starts `n` peer groups before the current row’s group; `n FOLLOWING` means it ends `n` peer groups after. `GROUPS` ensures that all rows in a peer group are treated uniformly: all are included or all are excluded. It is defined in SQL:2011 and supported by PostgreSQL, Oracle, and some other engines.

**Beginner-Friendly Explanation**  
`GROUPS` means “count distinct values.” If three rows share the same salary, they form one peer group. `GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW` includes the current peer group and the one distinct value before it, no matter how many physical rows those groups contain.

#### Purposes

- To compute windows that respect peer groups rather than physical row counts.
- To avoid splitting a peer group across frame boundaries.
- To create value-group-based moving calculations (e.g., previous `n` distinct categories).
- To provide a middle ground between `ROWS` and `RANGE`.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ GROUPS } BETWEEN frame_start AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `GROUPS` | Frame unit keyword. |
| `n PRECEDING` | `n` peer groups before the current group. |
| `CURRENT ROW` | The current row’s peer group. |
| `n FOLLOWING` | `n` peer groups after the current group. |

**Syntax rules**

- Offsets must be non-negative integers.
- `GROUPS` requires an `ORDER BY` to define peer groups.
- `GROUPS` is supported by PostgreSQL, Oracle, and some other engines.
- MySQL 8.0 parses `GROUPS` but raises an error; only `ROWS` and `RANGE` are supported.

**Constraints and limitations**

- Not universally supported (MySQL does not support it).
- Peer groups are determined by the `ORDER BY` expressions; adding columns changes group boundaries.
- `GROUPS` can be less intuitive than `ROWS` or `RANGE` for users unfamiliar with peer-group semantics.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `GROUPS` with peer groups

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW) AS group_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | group_sum
--------+-----------
   3500 |      3500
   3900 |      7400
   4200 |     11600
   4500 |     16100
   4800 |     25700
   4800 |     25700
   5000 |     30700
   5200 |     41100
   5200 |     41100
   6000 |     47100
(10 rows)
```

**Why this output appears**: `GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW` includes the current peer group and the immediately preceding peer group. The two 4800 rows form one peer group and both see the same sum (25700), because `GROUPS` does not split peer groups.

#### Real-World Cases

- **Distinct-value windows**: Calculations over the previous `n` distinct categories or levels.
- **Peer-group-aware reporting**: Ensuring that tied rows are treated identically.
- **Hierarchical analytics**: Windows that respect grouping levels.

---

### 4. `BETWEEN`

#### Definitions

**Core Definition**  
`BETWEEN` is the keyword that separates the frame start bound from the frame end bound in a window frame specification.

**Technical Definition**  
The `BETWEEN ... AND ...` syntax defines a closed interval of rows from `frame_start` to `frame_end`. Both bounds are inclusive. `BETWEEN` is optional in some engines for the short form (where only a start bound is given and the end defaults to `CURRENT ROW`), but the explicit `BETWEEN` form is required when both start and end are specified. The start bound must not be logically after the end bound.

**Beginner-Friendly Explanation**  
`BETWEEN` means “from this starting point to this ending point,” inclusive of both ends.

#### Purposes

- To define an explicit start and end for the frame.
- To enable symmetric windows (e.g., 2 preceding to 2 following).
- To override the default end bound (`CURRENT ROW`) when a different end is needed.
- To make frame boundaries unambiguous and readable.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `BETWEEN` | Separator between start and end bounds. |
| `frame_start` | Start bound (`UNBOUNDED PRECEDING`, `n PRECEDING`, `CURRENT ROW`, etc.). |
| `AND` | Separator keyword. |
| `frame_end` | End bound (`n FOLLOWING`, `UNBOUNDED FOLLOWING`, `CURRENT ROW`, etc.). |

**Syntax rules**

- `frame_start` cannot be `UNBOUNDED FOLLOWING`; `frame_end` cannot be `UNBOUNDED PRECEDING`.
- The frame start must not be after the frame end in the logical order (e.g., `RANGE BETWEEN CURRENT ROW AND n PRECEDING` is invalid).
- Without `BETWEEN`, only a start bound is given, and the end defaults to `CURRENT ROW` in most engines.
- `BETWEEN` is supported by all engines that support window frames.

**Constraints and limitations**

- The bound order must be logically valid; engines reject invalid combinations.
- Some engines restrict which bound combinations are allowed (e.g., SQL Server does not allow `RANGE` with numeric offsets).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Symmetric `BETWEEN` frame

```sql
SELECT val,
       AVG(val) OVER (ORDER BY id ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) AS centered_avg
FROM measurements;
```

**Expected output** (illustrative):

```text
 val | centered_avg
-----+--------------
  10 |        12.50
  12 |        14.00
  15 |        16.20
  20 |        18.50
  18 |        20.00
```

**Why this output appears**: The frame includes two rows before and two rows after the current row; the average is computed over up to five rows centered on the current row.

##### Example 2: MySQL – Short form vs `BETWEEN`

```sql
-- Short form: start bound only; end defaults to CURRENT ROW.
SUM(val) OVER (ORDER BY time ROWS UNBOUNDED PRECEDING)

-- Explicit BETWEEN with the same effect.
SUM(val) OVER (ORDER BY time ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

**Expected output**: Identical results for both expressions.

**Why this output appears**: The short form `ROWS UNBOUNDED PRECEDING` is equivalent to `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` in engines that support the short form.

#### Real-World Cases

- **Centered moving averages**: Smoothing over `n` rows before and after the current row.
- **Explicit window bounds**: Making frame intent clear in complex analytical queries.
- **Cross-engine compatibility**: Using `BETWEEN` explicitly avoids reliance on short-form defaults.

---

### 5. `UNBOUNDED PRECEDING`

#### Definitions

**Core Definition**  
`UNBOUNDED PRECEDING` is a frame bound that extends the frame to the very first row of the partition.

**Technical Definition**  
When used as `frame_start`, `UNBOUNDED PRECEDING` means the frame begins at the first row of the current partition. It is valid in `ROWS`, `RANGE`, and `GROUPS` modes. When used as `frame_end`, it is invalid because a frame cannot end before its start. `UNBOUNDED PRECEDING` is the most common start bound for running totals and cumulative aggregates.

**Beginner-Friendly Explanation**  
`UNBOUNDED PRECEDING` means “start from the very beginning of the group, no matter how far back that is.”

#### Purposes

- To compute cumulative aggregates from the start of the partition.
- To define the default frame start when `ORDER BY` is present.
- To ensure that all preceding rows are included in the calculation.
- To combine with `CURRENT ROW` for classic running totals.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN UNBOUNDED PRECEDING AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `UNBOUNDED PRECEDING` | Start at the first row of the partition. |
| `frame_end` | End bound (e.g., `CURRENT ROW`, `n FOLLOWING`, `UNBOUNDED FOLLOWING`). |

**Syntax rules**

- Only valid as `frame_start`, never as `frame_end`.
- Can be used with any frame unit (`ROWS`, `RANGE`, `GROUPS`).
- The default frame with `ORDER BY` uses `UNBOUNDED PRECEDING` as the start.

**Constraints and limitations**

- Must be paired with a valid end bound that does not precede it.
- Cannot be used as the end bound of a frame.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Cumulative sum from partition start

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative
FROM empsalary;
```

**Expected output**:

```text
 salary | cumulative
--------+------------
   3500 |       3500
   3900 |       7400
   4200 |      11600
   4500 |      16100
   4800 |      20900
   4800 |      25700
   5000 |      30700
   5200 |      35900
   5200 |      41100
   6000 |      47100
(10 rows)
```

**Why this output appears**: The frame starts at the first row of the partition and ends at the current row; the sum accumulates as rows are processed.

##### Example 2: MySQL – Short-form `UNBOUNDED PRECEDING`

```sql
SUM(val) OVER (PARTITION BY subject ORDER BY time ROWS UNBOUNDED PRECEDING) AS running_total
```

**Expected output** (excerpt):

```text
+----------+---------+------+---------------+
| time     | subject | val  | running_total |
+----------+---------+------+---------------+
| 07:00:00 | st113   |   10 |            10 |
| 07:15:00 | st113   |    9 |            19 |
| 07:30:00 | st113   |   25 |            44 |
| 07:45:00 | st113   |   20 |            64 |
+----------+---------+------+---------------+
```

**Why this output appears**: The short form `ROWS UNBOUNDED PRECEDING` is equivalent to `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, producing a running total.

#### Real-World Cases

- **Running totals**: Cumulative revenue, balance, or count from the start.
- **Year-to-date calculations**: Accumulating values from the beginning of the period.
- **Cumulative distributions**: Summing probabilities or weights from the first row.

---

### 6. `UNBOUNDED FOLLOWING`

#### Definitions

**Core Definition**  
`UNBOUNDED FOLLOWING` is a frame bound that extends the frame to the very last row of the partition.

**Technical Definition**  
When used as `frame_end`, `UNBOUNDED FOLLOWING` means the frame ends at the last row of the current partition. It is valid in `ROWS`, `RANGE`, and `GROUPS` modes. When used as `frame_start`, it is invalid. It is typically paired with `CURRENT ROW` or `UNBOUNDED PRECEDING` to compute totals that include all rows from the current row (or the start) to the end of the partition.

**Beginner-Friendly Explanation**  
`UNBOUNDED FOLLOWING` means “go all the way to the end of the group, no matter how far forward that is.”

#### Purposes

- To compute totals that include all rows from the current row to the end of the partition.
- To define the frame end for “remaining” or “look-ahead” calculations.
- To compute whole-partition aggregates when combined with `UNBOUNDED PRECEDING`.
- To support reverse cumulative calculations.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND UNBOUNDED FOLLOWING
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `frame_start` | Start bound (e.g., `CURRENT ROW`, `n PRECEDING`). |
| `UNBOUNDED FOLLOWING` | End at the last row of the partition. |

**Syntax rules**

- Only valid as `frame_end`, never as `frame_start`.
- Can be used with any frame unit.
- Must be paired with a valid start bound that does not follow it.

**Constraints and limitations**

- Cannot be used as the start bound.
- Without an explicit start, the end would be the only bound, which is not valid syntax in most engines.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Sum from current row to the end

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS remaining_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | remaining_sum
--------+---------------
   3500 |         47100
   3900 |         43600
   4200 |         39700
   4500 |         35500
   4800 |         31000
   4800 |         26200
   5000 |         21400
   5200 |         16200
   5200 |         11000
   6000 |          6000
(10 rows)
```

**Why this output appears**: The frame starts at the current row and extends to the last row; the sum decreases as the current row advances because fewer rows remain.

##### Example 2: PostgreSQL – Whole-partition aggregate with both unbounded bounds

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total
FROM empsalary;
```

**Expected output**:

```text
 salary | total
--------+-------
   3500 | 47100
   3900 | 47100
   4200 | 47100
   4500 | 47100
   4800 | 47100
   4800 | 47100
   5000 | 47100
   5200 | 47100
   5200 | 47100
   6000 | 47100
(10 rows)
```

**Why this output appears**: Both bounds are unbounded, so the frame is the entire partition; the total is the same on every row.

#### Real-World Cases

- **Remaining balance**: Sum of amounts from the current transaction to the end.
- **Reverse cumulative metrics**: Totals that decrease as the current row advances.
- **Whole-partition totals**: Computing a constant total across all rows.

---

### 7. `CURRENT ROW`

#### Definitions

**Core Definition**  
`CURRENT ROW` is a frame bound that refers to the current row (or, in `RANGE` and `GROUPS` modes, the current row’s peer group).

**Technical Definition**  
The meaning of `CURRENT ROW` depends on the frame unit. In `ROWS` mode, it refers to the current physical row. In `RANGE` and `GROUPS` mode, it refers to the current row’s peer group: the frame starts or ends with the first or last peer of the current row, respectively. As a start bound, `CURRENT ROW` means the frame begins at the current row (or its first peer); as an end bound, it means the frame ends at the current row (or its last peer).

**Beginner-Friendly Explanation**  
`CURRENT ROW` means “this row” in `ROWS` mode, but in `RANGE` and `GROUPS` mode it means “all rows tied with this row.”

#### Purposes

- To define the current row as the frame boundary.
- To compute running totals from the start to the current row.
- To compute remaining totals from the current row to the end.
- To ensure peer-group inclusion in `RANGE` and `GROUPS` modes.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND CURRENT ROW
{ ROWS | RANGE | GROUPS } BETWEEN CURRENT ROW AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `CURRENT ROW` | Current row (ROWS) or current peer group boundary (RANGE/GROUPS). |
| `frame_start` | May be `UNBOUNDED PRECEDING`, `n PRECEDING`, etc. |
| `frame_end` | May be `n FOLLOWING`, `UNBOUNDED FOLLOWING`, etc. |

**Syntax rules**

- `CURRENT ROW` is valid as both start and end bound.
- In `ROWS` mode, it refers to the physical current row.
- In `RANGE` and `GROUPS` mode, it refers to the current row’s first or last peer, depending on whether it is the start or end bound.
- The default frame end with `ORDER BY` is `CURRENT ROW` (in `RANGE` mode).

**Constraints and limitations**

- In `ROWS` mode with non-unique ordering, `CURRENT ROW` may produce non-deterministic results because “which row is current” is stable but peers are not grouped.
- In `RANGE` and `GROUPS` mode, `CURRENT ROW` includes all peers, which can surprise users expecting strict row-by-row behavior.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `CURRENT ROW` in `ROWS` vs `RANGE`

```sql
-- ROWS: CURRENT ROW is the physical row.
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS rows_sum
FROM empsalary;

-- RANGE: CURRENT ROW includes all peers.
SELECT salary,
       SUM(salary) OVER (ORDER BY salary RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS range_sum
FROM empsalary;
```

**Expected output** (`ROWS`):

```text
 salary | rows_sum
--------+----------
   4800 |    20900
   4800 |    25700
```

**Expected output** (`RANGE`):

```text
 salary | range_sum
--------+-----------
   4800 |     25700
   4800 |     25700
```

**Why this output appears**: In `ROWS` mode, the two 4800 rows are distinct physical rows and produce different sums; in `RANGE` mode, they are peers and both see the same sum including both of them.

##### Example 2: MySQL – `CURRENT ROW` as the end bound

```sql
SUM(val) OVER (ORDER BY time ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
```

**Expected output**:

```text
+----------+---------+------+---------------+
| time     | subject | val  | running_total |
+----------+---------+------+---------------+
| 07:00:00 | st113   |   10 |            10 |
| 07:15:00 | st113   |    9 |            19 |
| 07:30:00 | st113   |   25 |            44 |
| 07:45:00 | st113   |   20 |            64 |
+----------+---------+------+---------------+
```

**Why this output appears**: The frame ends at the current physical row in `ROWS` mode, producing a classic running total.

#### Real-World Cases

- **Running totals**: From the start to the current row.
- **Remaining calculations**: From the current row to the end.
- **Peer-aware totals**: In `RANGE` and `GROUPS` mode, including all tied rows.

---

### 8. `n PRECEDING`

#### Definitions

**Core Definition**  
`n PRECEDING` is a frame bound that starts or ends the frame `n` units before the current row, where the unit depends on the frame mode.

**Technical Definition**  
In `ROWS` mode, `n PRECEDING` means `n` physical rows before the current row. In `RANGE` mode, it means rows whose `ORDER BY` value is no more than `n` less than the current row’s value. In `GROUPS` mode, it means `n` peer groups before the current row’s peer group. The offset `n` must be a non-negative, non-null expression that does not contain variables, aggregates, or window functions.

**Beginner-Friendly Explanation**  
`n PRECEDING` means “go back `n` steps.” What a “step” is depends on the mode: rows in `ROWS`, value units in `RANGE`, or peer groups in `GROUPS`.

#### Purposes

- To define a moving window of a fixed size.
- To compute averages or sums over the previous `n` rows or units.
- To create trailing windows for trend analysis.
- To limit the frame to recent data relative to the current row.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN n PRECEDING AND frame_end
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `n` | Non-negative integer (ROWS/GROUPS) or value/interval (RANGE). |
| `PRECEDING` | Direction keyword. |
| `frame_end` | End bound. |

**Syntax rules**

- `n` must be non-negative and non-null.
- In `RANGE` mode, `n` must be compatible with the `ORDER BY` column type.
- `n PRECEDING` is valid as both start and end bound, but as an end bound it must not precede the start.
- `RANGE` with offsets requires exactly one `ORDER BY` column in most engines.

**Constraints and limitations**

- `n` cannot be negative.
- `RANGE` offsets require a single ordering column and a compatible type.
- `GROUPS` offsets are not supported in MySQL 8.0.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`

```sql
SELECT val,
       AVG(val) OVER (ORDER BY id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM measurements;
```

**Expected output** (illustrative):

```text
 val | moving_avg
-----+------------
  10 |      10.00
  12 |      11.00
  15 |      12.33
  20 |      15.67
  18 |      17.67
```

**Why this output appears**: The frame includes the current row and the two physical rows before it; the average is computed over up to three rows.

##### Example 2: PostgreSQL – `RANGE BETWEEN 500 PRECEDING AND CURRENT ROW`

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary RANGE BETWEEN 500 PRECEDING AND CURRENT ROW) AS range_sum
FROM empsalary;
```

**Expected output** (illustrative):

```text
 salary | range_sum
--------+-----------
   3500 |      3500
   3900 |      7400
   4200 |     11600
   4500 |     16100
   4800 |     20900
   4800 |     20900
   5000 |     30700
```

**Why this output appears**: The frame includes rows whose salary is within 500 of the current row’s salary; it operates on value proximity rather than physical row counts.

#### Real-World Cases

- **Trailing moving averages**: Average of the last `n` observations.
- **Rolling sums**: Cumulative totals over a fixed lookback window.
- **Value-band analytics**: Aggregates over rows within a value tolerance.

---

### 9. `n FOLLOWING`

#### Definitions

**Core Definition**  
`n FOLLOWING` is a frame bound that ends or starts the frame `n` units after the current row, where the unit depends on the frame mode.

**Technical Definition**  
In `ROWS` mode, `n FOLLOWING` means `n` physical rows after the current row. In `RANGE` mode, it means rows whose `ORDER BY` value is no more than `n` greater than the current row’s value. In `GROUPS` mode, it means `n` peer groups after the current row’s peer group. The offset must be non-negative and non-null.

**Beginner-Friendly Explanation**  
`n FOLLOWING` means “go forward `n` steps.” It is the mirror image of `n PRECEDING`.

#### Purposes

- To create forward-looking windows that include future rows.
- To compute centered moving averages that look ahead.
- To compute remaining totals from a point forward.
- To define symmetric frames around the current row.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND n FOLLOWING
```

**Component breakdown**

| Component | Description |
|-----------|-------------|
| `n` | Non-negative integer (ROWS/GROUPS) or value/interval (RANGE). |
| `FOLLOWING` | Direction keyword. |
| `frame_start` | Start bound. |

**Syntax rules**

- `n` must be non-negative and non-null.
- In `RANGE` mode, `n` must be compatible with the `ORDER BY` column type.
- `n FOLLOWING` is valid as both end and start bound.
- The start bound must not logically follow the end bound.

**Constraints and limitations**

- `n` cannot be negative.
- `RANGE` offsets require a single ordering column.
- `GROUPS` offsets are not supported in MySQL 8.0.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: MySQL – Centered moving average with `1 PRECEDING AND 1 FOLLOWING`

```sql
SELECT time, val,
       AVG(val) OVER (ORDER BY time ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS centered_avg
FROM observations;
```

**Expected output** (excerpt):

```text
+----------+------+-------------+
| time     | val  | centered_avg|
+----------+------+-------------+
| 07:00:00 |   10 |      9.5000 |
| 07:15:00 |    9 |     14.6667 |
| 07:30:00 |   25 |     18.0000 |
| 07:45:00 |   20 |     22.5000 |
+----------+------+-------------+
```

**Why this output appears**: The frame includes the current row, one row before, and one row after; the average is centered on the current row.

##### Example 2: PostgreSQL – `RANGE BETWEEN CURRENT ROW AND '7 days' FOLLOWING`

```sql
SELECT date, revenue,
       SUM(revenue) OVER (ORDER BY date RANGE BETWEEN CURRENT ROW AND '7 days' FOLLOWING) AS next_week_revenue
FROM daily_sales;
```

**Expected output** (illustrative):

```text
    date    | revenue | next_week_revenue
------------+---------+------------------
 2025-01-01 |     100 |               550
 2025-01-03 |     200 |               650
 2025-01-05 |     150 |               550
 2025-01-08 |     300 |               550
 2025-01-10 |     250 |               250
```

**Why this output appears**: The frame includes the current row and all rows within 7 days after it; the sum looks forward in time.

#### Real-World Cases

- **Centered smoothing**: Averages that use future and past data equally.
- **Forward-looking totals**: Remaining amounts or counts.
- **Symmetric rolling windows**: Windows defined as `n PRECEDING` to `n FOLLOWING`.

---

### 10. Default Frames

#### Definitions

**Core Definition**  
A default frame is the frame the database uses when no explicit frame clause is provided.

**Technical Definition**  
When a window specification contains no frame clause:
- If `ORDER BY` is absent, the default frame is the entire partition.
- If `ORDER BY` is present, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which includes all rows from the partition start through the current row’s last peer.
This default is consistent across PostgreSQL, SQL Server, MySQL, and Oracle.

**Beginner-Friendly Explanation**  
If you do not specify a frame, the database uses a sensible default: all rows from the beginning of the group up to the current row (including any rows tied with it). If there is no ordering, it uses the whole group.

#### Purposes

- To provide predictable behavior when frames are not explicitly specified.
- To simplify common running-total and cumulative queries.
- To define a baseline against which explicit frames can be compared.
- To understand why `SUM(...) OVER (ORDER BY ...)` behaves like a running total.

#### Syntax Rules and Structure

**Implicit default**:

```sql
-- With ORDER BY, default frame is RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW.
SUM(salary) OVER (ORDER BY salary)

-- Without ORDER BY, default frame is the entire partition.
SUM(salary) OVER (PARTITION BY depname)
```

**Component breakdown**

| Scenario | Default frame |
|----------|---------------|
| `ORDER BY` present, no frame clause | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` |
| No `ORDER BY`, no frame clause | Entire partition |
| Frame clause present | As specified |

**Syntax rules**

- The default applies only when no frame clause is given.
- The default is the same across major engines for the `ORDER BY` case.
- Without `ORDER BY`, ranking functions are still valid but produce arbitrary order-based results.

**Constraints and limitations**

- The default `RANGE` frame includes peers, which may not be desired for strict row-by-row running totals; use `ROWS` to avoid peer inclusion.
- The default without `ORDER BY` computes over the whole partition, which may be expensive for large partitions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Default frame with `ORDER BY`

```sql
SELECT salary,
       sum(salary) OVER (ORDER BY salary) AS default_sum
FROM empsalary;
```

**Expected output**:

```text
 salary | default_sum
--------+-------------
   3500 |        3500
   3900 |        7400
   4200 |       11600
   4500 |       16100
   4800 |       25700
   4800 |       25700
   5000 |       30700
   5200 |       41100
   5200 |       41100
   6000 |       47100
(10 rows)
```

**Why this output appears**: The default `RANGE` frame includes peers; the two 4800 rows both see 25700 because both are in the same peer group.

##### Example 2: SQL Server – Explicit `ROWS` overrides the default

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS UNBOUNDED PRECEDING) AS rows_sum
FROM Employees;
```

**Expected output** (illustrative):

```text
salary rows_sum
------ --------
3500   3500
3900   7400
4200   11600
...
4800   20900
4800   25700
```

**Why this output appears**: `ROWS` overrides the default `RANGE` frame and produces strict row-by-row accumulation, so the two 4800 rows have different running totals.

#### Real-World Cases

- **Running totals**: The default frame makes `SUM(...) OVER (ORDER BY ...)` behave as a running total (with peers).
- **Whole-partition aggregates**: Omitting `ORDER BY` makes the default frame the entire partition.
- **Understanding plan differences**: Knowing the default helps explain why `RANGE` may be slower than `ROWS` when peers exist.

---

### 11. Frame Exclusion

#### Definitions

**Core Definition**  
Frame exclusion is an optional clause that removes specific rows from the frame after the start and end bounds have been applied.

**Technical Definition**  
`EXCLUDE` is an SQL standard extension supported by PostgreSQL and Oracle. It can take four forms: `EXCLUDE CURRENT ROW` (remove the current row), `EXCLUDE GROUP` (remove the current row and all its peers), `EXCLUDE TIES` (remove peers of the current row but keep the current row), and `EXCLUDE NO OTHERS` (the default, which excludes nothing). The exclusion is applied after the frame bounds are evaluated; it does not change the bounds themselves.

**Beginner-Friendly Explanation**  
Frame exclusion lets you say “include this range of rows, but leave out this specific row (or the rows tied with it)”—useful when you want a total that does not include the current row itself.

#### Purposes

- To compute totals that exclude the current row (e.g., “other than this transaction”).
- To exclude an entire peer group from a calculation.
- To keep the current row but exclude its ties.
- To fine-tune frame contents without changing the start and end bounds.

#### Syntax Rules and Structure

**Syntax**:

```sql
{ ROWS | RANGE | GROUPS } BETWEEN frame_start AND frame_end
[ EXCLUDE { CURRENT ROW | GROUP | TIES | NO OTHERS } ]
```

**Component breakdown**

| Exclusion | Effect |
|-----------|--------|
| `EXCLUDE CURRENT ROW` | Removes only the current row from the frame. |
| `EXCLUDE GROUP` | Removes the current row and all its peers. |
| `EXCLUDE TIES` | Removes peers of the current row but keeps the current row. |
| `EXCLUDE NO OTHERS` | Excludes nothing (default behavior). |

**Syntax rules**

- `EXCLUDE` is supported by PostgreSQL and Oracle.
- MySQL 8.0 parses `EXCLUDE` but raises an error; only `ROWS` and `RANGE` are supported.
- SQL Server does not support `EXCLUDE`.
- The exclusion is applied after the frame bounds are determined.

**Constraints and limitations**

- Not universally supported (MySQL and SQL Server lack support).
- `EXCLUDE GROUP` and `EXCLUDE TIES` require an `ORDER BY` to define peers.
- Exclusion does not affect the bounds; it only removes rows from the resulting frame.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – `EXCLUDE CURRENT ROW`

```sql
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE CURRENT ROW) AS sum_without_current
FROM empsalary;
```

**Expected output**:

```text
 salary | sum_without_current
--------+---------------------
   3500 |                   0
   3900 |                3500
   4200 |                7400
   4500 |               11600
   4800 |               16100
   4800 |               20900
   5000 |               25700
   5200 |               30700
   5200 |               35900
   6000 |               41100
(10 rows)
```

**Why this output appears**: The frame normally includes the current row, but `EXCLUDE CURRENT ROW` removes it; the running sum for each row is the sum of all preceding rows only.

##### Example 2: PostgreSQL – `EXCLUDE GROUP` vs `EXCLUDE TIES`

```sql
-- EXCLUDE GROUP: remove current row and its peers.
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE GROUP) AS without_group
FROM empsalary;

-- EXCLUDE TIES: remove peers but keep current row.
SELECT salary,
       SUM(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE TIES) AS without_ties
FROM empsalary;
```

**Expected output** (`EXCLUDE GROUP`, excerpt):

```text
 salary | without_group
--------+---------------
   4800 |         16100
   4800 |         16100
```

**Expected output** (`EXCLUDE TIES`, excerpt):

```text
 salary | without_ties
--------+--------------
   4800 |        20900
   4800 |        20900
```

**Why this output appears**: `EXCLUDE GROUP` removes both 4800 rows, so both see only the rows before them (16100). `EXCLUDE TIES` keeps the current row but removes the other peer, so each 4800 row sees 20900 (the preceding total plus its own value).

#### Real-World Cases

- **Excluding the current transaction**: Computing a balance that does not include the current row.
- **Peer-aware deduplication**: Keeping the current row but excluding duplicates.
- **Comparative analysis**: Excluding an entire group to compare against “others.”

---

## References

- PostgreSQL Documentation – Window Functions (Tutorial) – https://www.postgresql.org/docs/current/tutorial-window.html
- PostgreSQL Documentation – Window Functions (Reference, Frame Specification) – https://www.postgresql.org/docs/current/functions-window.html
- PostgreSQL Documentation – `SELECT` (Window Frame Syntax) – https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- PostgreSQL Git – Default framing option is `RANGE UNBOUNDED PRECEDING` – https://git.postgresql.org/gitweb/?p=postgresql.git;a=blob_plain;f=doc/src/sgml/syntax.sgml;hb=HEAD
- MySQL 8.0 Reference Manual – Window Function Frame Specification – https://dev.mysql.com/doc/refman/8.0/en/window-functions-frames.html
- MySQL 8.0 Reference Manual – Window Function Restrictions – https://dev.mysql.com/doc/refman/8.0/en/window-function-restrictions.html
- Microsoft Learn – `OVER` Clause (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql
- Oracle Database SQL Language Reference – Windowing Clause – https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/img_text/windowing_clause.html
- FreeBSD Man Pages – `SELECT` (frame exclusion) – https://man.freebsd.org/cgi/man.cgi?query=SELECT&sektion=7
- Erik Darling Data – A Little About Row & Range Window Functions vs Batch Mode – https://erikdarling.com/a-little-about-row-range-window-functions-vs-batch-mode-in-sql-server/
- SQL Authority – T-SQL Window Function Framing and Performance – https://blog.sqlauthority.com/2015/11/19/sql-server-t-sql-window-function-framing-and-performance-notes-from-the-field-103/
- PostgreSQL Documentation – `RANGE` with interval example – https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS
- SQL:2011 Standard – Window Frame Specification (ISO/IEC 9075-2:2011)
- PostgreSQL Wiki – What's new in PostgreSQL 9.0 (window frames) – https://wiki.postgresql.org/wiki/What%27s_new_in_PostgreSQL_9.0