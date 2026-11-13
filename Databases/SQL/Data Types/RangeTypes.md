# Comprehensive Programming Cheat Sheet: PostgreSQL Range Types

## Topic Overview

### Definitions

**Core Definition**
Range types are PostgreSQL data types that represent a continuous or discrete span of values of some element type (called the range's subtype), enabling a single column to store an entire interval—such as a time period, a numeric band, or an integer sequence—and to express concepts like overlap, containment, and adjacency directly in SQL.

**Technical Definition**
A range type is a data type representing a range of values of some element type, where the subtype must have a total order so that it is well-defined whether element values are within, before, or after a range of values. Every non-empty range has a lower bound and an upper bound; bounds may be inclusive (closed) or exclusive (open), represented by square brackets `[` `]` and parentheses `(` `)` respectively. PostgreSQL provides six built-in range types—`int4range`, `int8range`, `numrange`, `tsrange`, `tstzrange`, and `daterange`—and each has a corresponding multirange type that stores an ordered list of non-contiguous, non-empty, non-null ranges. Range types support specialized operators (`&&`, `@>`, `<@`, `<<`, `>>`, `-|-`, `+`, `*`, `-`) and functions (`lower`, `upper`, `isempty`, `lower_inc`, `upper_inc`, `lower_inf`, `upper_inf`, `range_merge`, `range_agg`). GiST indexes accelerate range queries, and `EXCLUDE` constraints use GiST to prevent overlapping values at the database level.

**Beginner-Friendly Explanation**
A range type is a container for a span of values. Instead of storing a start time and an end time in two separate columns—and then writing complicated `WHERE` clauses to detect overlaps—you store the entire time period in one column. PostgreSQL then gives you operators that speak the language of intervals: “does this range overlap that range?”, “is this timestamp inside that period?”, “do these two ranges touch?”. Think of a range as a highlighted section of a ruler: you can ask whether two highlighted sections overlap, whether one is completely inside another, or whether they meet exactly at the edges. The built-in range types cover the most common rulers—integers, big integers, decimals, timestamps (with and without time zones), and dates—but you can also define your own range types for any subtype with a total order.

### Key Characteristics

- **Bounds are inclusive or exclusive**: Every range has a lower and upper bound, each of which may be inclusive (the boundary value is part of the range) or exclusive (the boundary value is not part of the range). The notation `[a, b)` means “from a inclusive to b exclusive”.
- **Half-open canonical form for discrete types**: The built-in discrete range types (`int4range`, `int8range`, `daterange`) use a canonical form that includes the lower bound and excludes the upper bound—`[)`—so equivalent representations are normalised to the same value.
- **Infinite (unbounded) ranges are supported**: Either bound can be omitted, meaning the range extends to negative or positive infinity. An omitted bound is treated as beyond any value of the element type, including explicit “infinity” values.
- **Empty ranges are valid**: A range like `[4,4)` contains no points and is normalised to `empty`; empty ranges are treated as the additive identity for union and difference operations.
- **Multiranges extend the model**: Since PostgreSQL 14, every range type has a corresponding multirange type that stores an ordered list of disjoint ranges, enabling union and difference operations that would otherwise fail when the result is not a single contiguous range.
- **GiST indexes and EXCLUDE constraints are the performance and integrity backbone**: GiST indexes accelerate overlap (`&&`) and containment (`@>`) queries, while `EXCLUDE USING GIST` constraints prevent overlapping values at write time, generalising `UNIQUE` from equality to any operator.

### Prerequisites

- A running PostgreSQL instance (range types were introduced in PostgreSQL 9.2; multiranges in PostgreSQL 14).
- Familiarity with `CREATE TABLE` and SQL data types.
- For `EXCLUDE` constraints and GiST indexes: the `btree_gist` extension if equality comparisons on non-range columns are combined with range overlap in the same constraint.
- Understanding of the application's interval semantics: whether the upper bound should be inclusive or exclusive, whether ranges are discrete or continuous, and whether overlaps must be prevented.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations, `EXCLUDE` constraints, and GiST indexes.
- Data Manipulation Language (DML) – inserting, querying, and updating range values.
- Query optimisation – GiST and SP-GiST indexes for range operators.
- Scheduling and booking systems – preventing double bookings with `EXCLUDE` constraints.
- Temporal data modelling – validity periods, version ranges, and time-series intervals.

---

## Core Concepts and Features

### 1. Range Types: Built-in Types

#### Definitions

**Core Definition**
Built-in range types are the six predefined PostgreSQL types that store ranges over integers, big integers, numerics, timestamps (with and without time zone), and dates.

**Technical Definition**
PostgreSQL provides `int4range` (range of `integer`), `int8range` (range of `bigint`), `numrange` (range of `numeric`), `tsrange` (range of `timestamp without time zone`), `tstzrange` (range of `timestamp with time zone`), and `daterange` (range of `date`). Each range type has a constructor function with the same name, and a corresponding multirange type (`int4multirange`, `int8multirange`, `nummultirange`, `tsmultirange`, `tstzmultirange`, `datemultirange`). The subtype must have a total order, and discrete subtypes (integer, bigint, date) have a well-defined “step,” enabling canonicalisation to the `[)` form.

**Beginner-Friendly Explanation**
PostgreSQL comes with six ready-made rulers: one for ordinary integers (`int4range`), one for big integers (`int8range`), one for decimals (`numrange`), two for timestamps (`tsrange` without time zone, `tstzrange` with time zone), and one for dates (`daterange`). Each ruler has a matching “multirange” container that can hold several disjoint pieces.

#### Purposes

- To store a span of values in a single column without a separate start/end column pair.
- To express overlap, containment, and adjacency queries directly with range operators.
- To prevent overlapping values with `EXCLUDE` constraints.
- To support scheduling, pricing, validity periods, and measurement-band applications.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name int4range
column_name int8range
column_name numrange
column_name tsrange
column_name tstzrange
column_name daterange
```

**Literal notation:**

```sql
'[lower, upper)'  -- half-open: lower inclusive, upper exclusive
'(lower, upper)'  -- open: both exclusive
'[lower, upper]'  -- closed: both inclusive
'(lower, upper]'  -- lower exclusive, upper inclusive
'empty'           -- empty range
```

**Constructor functions:**

```sql
int4range(lower, upper [, bound])
numrange(lower, upper [, bound])
tsrange(lower, upper [, bound])
```

**Component breakdown:**

| Type | Subtype | Discrete/Continuous | Canonical form |
|------|---------|---------------------|----------------|
| `int4range` | `integer` | Discrete | `[)` |
| `int8range` | `bigint` | Discrete | `[)` |
| `numrange` | `numeric` | Continuous | Preserves bounds |
| `tsrange` | `timestamp without time zone` | Continuous | Preserves bounds |
| `tstzrange` | `timestamp with time zone` | Continuous | Preserves bounds |
| `daterange` | `date` | Discrete | `[)` |

**Syntax rules:**

- Bounds may be omitted to indicate infinity: `(,3]` includes all values up to and including 3; `[3,)` includes all values from 3 onward.
- Specifying a missing bound as inclusive is automatically converted to exclusive: `[,]` becomes `(,)`.
- For discrete types, equivalent representations are canonicalised: `[1,6)`, `[1,5]`, `(0,6)`, and `(0,5]` all represent the same integer range and are stored as `[1,6)`.
- The constructor function’s third argument specifies the bound type (`'[]'`, `'[)'`, `'(]'`, `'()'`); if omitted, `[)` is assumed.

**Constraints and limitations:**

- The subtype must have a total order; types without a well-defined ordering cannot be used as range subtypes.
- Continuous ranges preserve inclusive/exclusive bounds as specified; discrete ranges canonicalise to `[)`.
- Multiranges are an ordered list of non-contiguous, non-empty, non-null ranges.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating a reservation table with `tsrange`

```sql
-- Step 1: Create a table with a tsrange column.
CREATE TABLE reservation (
    room   int,
    during tsrange
);

-- Step 2: Insert a half-open time range.
INSERT INTO reservation VALUES
    (1108, '[2010-01-01 14:30, 2010-01-01 15:30)');

-- Step 3: Query the inserted range.
SELECT room, during FROM reservation;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 room |                    during
------+-----------------------------------------------
 1108 | ["2010-01-01 14:30:00","2010-01-01 15:30:00")
(1 row)
```

**Why this output appears**: The `tsrange` column stores the time interval from 14:30 inclusive to 15:30 exclusive. PostgreSQL renders the range in its canonical text form, with double quotes around the timestamp values.

##### Example 2: Using constructor functions and canonicalisation

```sql
-- Step 1: Create an integer range with explicit exclusive bounds.
SELECT int4range(10, 20) AS canonical_range;

-- Step 2: Create an integer range with a closed upper bound.
SELECT int4range(1, 5, '[]') AS closed_range;
```

**Expected output:**

```text
 canonical_range
-----------------
 [10,20)
(1 row)

 closed_range
-------------
 [1,6)
(1 row)
```

**Why this output appears**: `int4range` is a discrete type, so `[1,5]` is canonicalised to `[1,6)` because the integer 5 is the last value before 6, and the exclusive upper bound 6 represents the same set of integers.

#### Real-World Cases

- **Conference room booking**: A `reservation` table stores each booking as a `tsrange` and uses an `EXCLUDE` constraint to prevent double-booking.
- **Price validity periods**: A `pricing` table stores `numrange` values representing price bands or discount tiers.
- **Employment history**: A `job_history` table stores `daterange` values for each employee’s tenure, enabling queries for “who worked here on a given date?”

---

### 2. Range Operations (Operators)

#### Definitions

**Core Definition**
Range operators are the specialized comparison and set operators that test relationships between ranges—overlap, containment, strict ordering, adjacency—and combine ranges through union, intersection, and difference.

**Technical Definition**
PostgreSQL provides the following range operators: containment (`@>` for range-contains-range, range-contains-element; `<@` for reverse containment), overlap (`&&`), strict left/right (`<<`, `>>`), “does not extend to the right/left” (`&<`, `&>`), adjacency (`-|-`), union (`+`), intersection (`*`), and difference (`-`). Comparison operators (`=`, `<>`, `<`, `>`, `<=`, `>=`) are also available and order first by lower bounds, then by upper bounds.

**Beginner-Friendly Explanation**
Range operators are the questions you can ask about intervals. `&&` asks “do these two ranges overlap?”. `@>` asks “does this range contain that value or that other range?”. `<<` asks “is this range entirely to the left of that one?”. `-|-` asks “do these ranges touch at the edges?”. `+`, `*`, and `-` perform set operations: union, intersection, and difference.

#### Purposes

- To detect overlapping bookings, schedules, or validity periods.
- To test whether a point lies within a range.
- To order and compare ranges for sorting and unique constraints.
- To combine or subtract intervals for scheduling and availability calculations.

#### Syntax Rules and Structure

**Range operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `@>` | First range contains the second, or range contains element | `int4range(2,4) @> 3` → `t` |
| `<@` | First range is contained by the second, or element is in range | `3 <@ int4range(2,4)` → `t` |
| `&&` | Ranges overlap (share any element) | `int4range(3,7) && int4range(4,12)` → `t` |
| `<<` | First range strictly left of the second | `int4range(1,10) << int4range(100,110)` → `t` |
| `>>` | First range strictly right of the second | `int4range(50,60) >> int4range(20,30)` → `t` |
| `&<` | First range does not extend to the right of the second | `int4range(1,20) &< int4range(18,20)` → `t` |
| `&>` | First range does not extend to the left of the second | `int4range(7,20) &> int4range(5,10)` → `t` |
| `-|-` | Ranges are adjacent | `numrange(1.1,2.2) -|- numrange(2.2,3.3)` → `t` |
| `+` | Union (ranges must overlap or be adjacent) | `numrange(5,15) + numrange(10,20)` → `[5,20)` |
| `*` | Intersection | `int4range(5,15) * int4range(10,20)` → `[10,15)` |
| `-` | Difference (second must not be contained in a way that splits the first) | `int4range(5,15) - int4range(10,20)` → `[5,10)` |

**Syntax rules:**

- Containment tests element membership when the right operand is a scalar: `int4range(10,20) @> 3` returns `true` if `3` is in the range.
- Union (`+`) fails if the result would be two disjoint ranges; cast to multirange and use multirange union instead.
- The left-of/right-of/adjacent operators always return `false` when an empty range is involved.

**Constraints and limitations:**

- The `+` operator cannot produce a disjoint result; use multirange union for that.
- Comparison ordering of ranges is not necessarily useful for all applications, but it allows unique indexes to be constructed on range columns.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Containment, overlap, and intersection

```sql
-- Step 1: Test containment of an element.
SELECT int4range(10, 20) @> 3 AS contains_3;

-- Step 2: Test overlap.
SELECT numrange(11.1, 22.2) && numrange(20.0, 30.0) AS overlaps;

-- Step 3: Compute intersection.
SELECT int4range(10, 20) * int4range(15, 25) AS intersection;

-- Step 4: Extract the upper bound.
SELECT upper(int8range(15, 25)) AS upper_bound;
```

**Expected output:**

```text
 contains_3
------------
 f
(1 row)
 overlaps
----------
 t
(1 row)
 intersection
--------------
 [15,20)
(1 row)
 upper_bound
-------------
          25
(1 row)
```

**Why this output appears**: `3` is not in `[10,20)`, so containment is `false`. The two numeric ranges share the interval `[20.0, 22.2)`, so overlap is `true`. The intersection of `[10,20)` and `[15,25)` is `[15,20)`. The upper bound of `[15,25)` is `25`.

#### Real-World Cases

- **Double-booking prevention**: An overlap query `WHERE during && '[requested_start, requested_end)'` finds conflicts before insert.
- **Availability search**: A containment query `WHERE availability @> requested_slot` finds resources available for an entire requested period.
- **Shift scheduling**: Adjacency checks (`-|-`) ensure that consecutive shifts do not leave gaps or overlap.

---

### 3. Range Functions

#### Definitions

**Core Definition**
Range functions are the built-in functions that extract bounds, test inclusivity, test emptiness, and merge ranges, including `lower()`, `upper()`, `isempty()`, `lower_inc()`, `upper_inc()`, `lower_inf()`, `upper_inf()`, `range_merge()`, and the aggregate `range_agg()`.

**Technical Definition**
`lower(anyrange)` returns the lower bound (or `NULL` if empty or infinite); `upper(anyrange)` returns the upper bound (or `NULL` if empty or infinite); `isempty(anyrange)` tests whether a range is empty; `lower_inc(anyrange)` and `upper_inc(anyrange)` test inclusivity; `lower_inf(anyrange)` and `upper_inf(anyrange)` test for infinite bounds. `range_merge(anyrange, anyrange)` computes the smallest range that includes both inputs, and the aggregate `range_agg(anyrange)` collects ranges into a multirange. The lower_inc, upper_inc, lower_inf, and upper_inf functions all return `false` for an empty range.

**Beginner-Friendly Explanation**
Range functions let you inspect and combine intervals. `lower()` and `upper()` give you the endpoints; `isempty()` tells you if there is nothing inside; `lower_inc()` and `upper_inc()` tell you whether the endpoints themselves are included; `lower_inf()` and `upper_inf()` tell you whether the range extends to infinity. `range_merge()` combines two ranges into the smallest range that covers both, and `range_agg()` collects many ranges into a multirange.

#### Purposes

- To extract the start or end of a range for display or computation.
- To test whether a range includes its boundary points.
- To detect empty or unbounded ranges for validation.
- To merge adjacent or overlapping ranges into a single continuous range.

#### Syntax Rules and Structure

| Function | Return type | Description |
|----------|-------------|-------------|
| `lower(anyrange)` | `anyelement` | Lower bound, or `NULL` if empty/infinite |
| `upper(anyrange)` | `anyelement` | Upper bound, or `NULL` if empty/infinite |
| `isempty(anyrange)` | `boolean` | True if the range contains no points |
| `lower_inc(anyrange)` | `boolean` | True if lower bound is inclusive |
| `upper_inc(anyrange)` | `boolean` | True if upper bound is inclusive |
| `lower_inf(anyrange)` | `boolean` | True if lower bound is infinite |
| `upper_inf(anyrange)` | `boolean` | True if upper bound is infinite |
| `range_merge(anyrange, anyrange)` | `anyrange` | Smallest range covering both inputs |
| `range_agg(anyrange)` | `anymultirange` | Aggregate of ranges into a multirange |

**Syntax rules:**

- `lower()` and `upper()` return `NULL` for empty ranges or infinite bounds.
- `lower_inc()`, `upper_inc()`, `lower_inf()`, and `upper_inf()` all return `false` for empty ranges.
- `range_merge()` can merge disjoint ranges into the smallest covering range.

**Constraints and limitations:**

- `range_merge()` may produce a range that covers gaps between the inputs if the inputs are disjoint.
- `range_agg()` requires PostgreSQL 14+ (multirange support).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Inspecting and merging ranges

```sql
-- Step 1: Extract bounds and inclusivity.
SELECT lower(int4range(1, 10)) AS low,
       upper(int4range(1, 10)) AS high,
       lower_inc(int4range(1, 10)) AS low_inc,
       upper_inc(int4range(1, 10)) AS high_inc;

-- Step 2: Test emptiness.
SELECT isempty(numrange(1, 5)) AS empty_range;

-- Step 3: Merge two ranges.
SELECT range_merge(numrange(1, 5), numrange(3, 10)) AS merged;
```

**Expected output:**

```text
 low | high | low_inc | high_inc
-----+------+---------+----------
   1 |   10 | t       | f
(1 row)

 empty_range
-------------
 f
(1 row)

  merged
-----------
 [1,10)
(1 row)
```

**Why this output appears**: `[1,10)` has lower bound 1 (inclusive) and upper bound 10 (exclusive). `isempty(numrange(1,5))` returns `false` because the range contains points. `range_merge` produces the smallest range covering both `[1,5)` and `[3,10)`, which is `[1,10)`.

#### Real-World Cases

- **Reporting**: Extracting `lower()` and `upper()` to display start and end dates in a report.
- **Data validation**: Using `isempty()` to reject empty ranges in a `CHECK` constraint.
- **Availability consolidation**: Using `range_agg()` and `range_merge()` to combine multiple booked slots into the overall occupied period.

---

### 4. GiST Indexes and Performance

#### Definitions

**Core Definition**
GiST (Generalised Search Tree) indexes are the index type PostgreSQL uses to accelerate range overlap, containment, and other range-operator queries.

**Technical Definition**
GiST indexes are PostgreSQL’s standard index type for range overlap queries—a single GiST index on a range column efficiently answers queries with `&&`, `@>`, and other range operators. SP-GiST indexes are also supported for range types. When range overlap queries also filter on equality predicates (e.g., `room_id = 5 AND during && '[10:00, 11:00)'`), a composite GiST index including both columns via the `btree_gist` extension outperforms separate indexes.

**Beginner-Friendly Explanation**
A GiST index is a special index that understands intervals. A normal B-tree index is good for looking up exact values, but a GiST index is good for asking “what overlaps this interval?”. Without a GiST index, an overlap query has to scan every row. With one, PostgreSQL can jump straight to the relevant intervals, often producing a dramatic speedup.

#### Purposes

- To accelerate overlap (`&&`), containment (`@>`), and other range-operator queries.
- To support `EXCLUDE` constraints that prevent overlapping values.
- To combine range queries with equality filters on other columns via `btree_gist`.

#### Syntax Rules and Structure

**Creating a GiST index on a range column:**

```sql
CREATE INDEX idx_reservation_during ON reservation USING GIST (during);
```

**Creating a composite GiST index with `btree_gist`:**

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE INDEX idx_reservation_room_during ON reservation USING GIST (room, during);
```

**Component breakdown:**

| Index type | Use case |
|------------|----------|
| `GIST` on range column | Overlap and containment queries on the range alone |
| `GIST` with `btree_gist` | Range queries combined with equality filters on other columns |

**Syntax rules:**

- GiST indexes support `&&`, `@>`, `<@`, `<<`, `>>`, `&<`, `&>`, and `-|-`.
- B-tree indexes on range columns support equality and ordering comparisons but not overlap queries.
- `btree_gist` is required to include non-range columns (e.g., `room`) in a GiST index.

**Constraints and limitations:**

- GiST indexes have higher write overhead than B-tree indexes.
- Without a GiST index, overlap queries perform sequential scans.
- GiST indexes are not a substitute for proper query design; they accelerate the operators they support, not arbitrary expressions.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: GiST index for overlap queries

```sql
-- Step 1: Create a table without an index.
CREATE TABLE bookings (
    id      int PRIMARY KEY,
    room    int,
    during  tsrange
);

-- Step 2: Insert sample data.
INSERT INTO bookings VALUES
    (1, 101, '[2025-01-01 09:00, 2025-01-01 10:00)'),
    (2, 101, '[2025-01-01 10:00, 2025-01-01 11:00)'),
    (3, 102, '[2025-01-01 09:30, 2025-01-01 10:30)');

-- Step 3: Create a GiST index on the range column.
CREATE INDEX idx_bookings_during ON bookings USING GIST (during);

-- Step 4: Query for overlapping bookings.
SELECT id, room FROM bookings
WHERE during && '[2025-01-01 09:45, 2025-01-01 10:15)';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
CREATE INDEX
 id | room
----+------
  1 |  101
  2 |  101
  3 |  102
(3 rows)
```

**Why this output appears**: The query range `[09:45, 10:15)` overlaps with booking 1 (ends at 10:00), booking 2 (starts at 10:00), and booking 3 (09:30–10:30). The GiST index accelerates the `&&` operator, allowing PostgreSQL to find all overlapping ranges efficiently.

#### Real-World Cases

- **Booking systems**: A GiST index on `during` makes overlap checks fast even with millions of reservations.
- **Temporal databases**: A GiST index on `validity` accelerates “as-of” queries that find rows valid at a given timestamp.
- **Composite filters**: A composite GiST index on `(room_id, during)` with `btree_gist` speeds up queries like “find bookings for room 101 that overlap this time slot”.

---

### 5. `EXCLUDE` Constraints for Non-Overlapping Ranges

#### Definitions

**Core Definition**
An `EXCLUDE` constraint is a PostgreSQL constraint that prevents overlapping values in a range column—or overlapping ranges within a group defined by equality on other columns—by rejecting inserts and updates that would create a conflict.

**Technical Definition**
`EXCLUDE USING GIST (column WITH operator)` creates a constraint backed by a GiST index. The classic example is `EXCLUDE USING GIST (during WITH &&)`, which prevents any two rows from having overlapping `during` ranges. For per-room non-overlap, the constraint combines equality on the room with overlap on the range: `EXCLUDE USING GIST (room WITH =, during WITH &&)`, requiring the `btree_gist` extension. The constraint is enforced at write time under any isolation level, by the same mechanism that makes `UNIQUE` reliable.

**Beginner-Friendly Explanation**
An `EXCLUDE` constraint is a “no overlaps allowed” rule. You tell PostgreSQL: “For this table, no two rows may have overlapping time ranges.” If someone tries to insert a booking that overlaps an existing one, the database rejects it automatically—no application code needed. You can also say “no overlapping ranges for the same room,” which allows different rooms to have overlapping bookings but not the same room.

#### Purposes

- To enforce non-overlap at the database level, independent of application code.
- To prevent double bookings in scheduling and reservation systems.
- To enforce non-overlapping validity periods for prices, rates, or versions.
- To generalise `UNIQUE` from equality to arbitrary operators like `&&`.

#### Syntax Rules and Structure

**Table-level non-overlap:**

```sql
CREATE TABLE reservation (
    during tsrange,
    EXCLUDE USING GIST (during WITH &&)
);
```

**Per-group non-overlap:**

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

CREATE TABLE room_reservation (
    room_id int,
    during  tsrange,
    EXCLUDE USING GIST (room_id WITH =, during WITH &&)
);
```

**Component breakdown:**

| Clause | Description |
|--------|-------------|
| `EXCLUDE USING GIST` | Specifies the constraint and its backing index type |
| `(column WITH operator)` | The operator that defines the conflict |
| `(room_id WITH =, during WITH &&)` | Per-room non-overlap: equality on room, overlap on range |

**Syntax rules:**

- `btree_gist` is required when mixing equality operators (`=`) with range operators (`&&`) in the same `EXCLUDE` constraint.
- The constraint creates an implicit GiST index; additional GiST indexes may be redundant.
- The order of expressions in the constraint matters for performance.

**Constraints and limitations:**

- `EXCLUDE` constraints do not fire triggers; they are enforced at the index level.
- They can only be used with operators supported by the index method (GiST or SP-GiST for ranges).
- Inserting a conflicting row raises an `exclusion_violation` error (SQLSTATE `23P01`).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Preventing double bookings with `EXCLUDE`

```sql
-- Step 1: Enable btree_gist for the composite constraint.
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Step 2: Create a table with a per-room non-overlap constraint.
CREATE TABLE bookings (
    booking_id serial PRIMARY KEY,
    room_id    int NOT NULL,
    during     tstzrange NOT NULL,
    EXCLUDE USING GIST (room_id WITH =, during WITH &&)
);

-- Step 3: Insert a valid booking.
INSERT INTO bookings (room_id, during)
VALUES (101, '[2025-01-01 09:00, 2025-01-01 10:00)');

-- Step 4: Attempt to insert an overlapping booking for the same room.
INSERT INTO bookings (room_id, during)
VALUES (101, '[2025-01-01 09:30, 2025-01-01 10:30)');
```

**Expected output:**

```text
CREATE EXTENSION
CREATE TABLE
INSERT 0 1
ERROR:  conflicting key value violates exclusion constraint "bookings_room_id_during_excl"
DETAIL:  Key (room_id, during)=(101, ["2025-01-01 09:30:00+00","2025-01-01 10:30:00+00")) conflicts with existing key (room_id, during)=(101, ["2025-01-01 09:00:00+00","2025-01-01 10:00:00+00")).
```

**Why this output appears**: The `EXCLUDE` constraint combines `room_id WITH =` and `during WITH &&`. The second insert conflicts with the existing booking for room 101 because the ranges overlap. PostgreSQL raises an `exclusion_violation` and rejects the insert, ensuring no double booking occurs.

#### Real-World Cases

- **Conference room booking**: A `bookings` table with `EXCLUDE USING GIST (room_id WITH =, during WITH &&)` ensures no two bookings for the same room overlap.
- **Price validity**: A `pricing` table with `EXCLUDE USING GIST (product_id WITH =, validity WITH &&)` prevents overlapping price periods for the same product.
- **Podcast scheduling**: A `episodes` table with `EXCLUDE USING GIST (channel_id WITH =, air_time WITH &&)` prevents two episodes from airing at the same time on the same channel.

---

### 6. Multiranges

#### Definitions

**Core Definition**
A multirange is an ordered list of non-contiguous, non-empty, non-null ranges, introduced in PostgreSQL 14 to represent sets of intervals that cannot be merged into a single contiguous range.

**Technical Definition**
Every range type has a corresponding multirange type: `int4multirange`, `int8multirange`, `nummultirange`, `tsmultirange`, `tstzmultirange`, and `datemultirange`. Multiranges support many of the same operators as ranges (`@>`, `<@`, `&&`, `<<`, `>>`, `&<`, `&>`, `-|-`, `+`, `*`, `-`) and add multirange-specific functions and aggregates. Multirange union and difference operators do not fail when the result is disjoint, unlike range union and difference, which require a single contiguous result.

**Beginner-Friendly Explanation**
A multirange is a container for several separate intervals. If a room is booked from 9–10, free from 10–11, and booked again from 11–12, the occupied time is two separate ranges—9–10 and 11–12. A multirange can hold both pieces in one value, and operations like union and difference work even when the result is not one continuous block.

#### Purposes

- To represent multiple disjoint intervals in a single column.
- To perform union and difference operations without errors when ranges are disjoint.
- To aggregate many ranges into a consolidated multirange using `range_agg()`.
- To support complex scheduling and availability models with gaps.

#### Syntax Rules and Structure

**Multirange literal:**

```sql
'{}'::int4multirange                    -- empty multirange
'{[3,7)}'::int4multirange               -- one range
'{[3,7), [8,9)}'::int4multirange        -- two disjoint ranges
```

**Multirange constructor:**

```sql
nummultirange()                          -- empty
nummultirange(numrange(1.0, 14.0))       -- one range
nummultirange(numrange(1.0, 14.0), numrange(20.0, 25.0))
```

**Component breakdown:**

| Feature | Description |
|---------|-------------|
| Literal syntax | Curly braces `{...}` containing zero or more comma-separated ranges |
| Constructor | Same name as the multirange type, accepting zero or more ranges |
| Aggregate | `range_agg(anyrange)` collects ranges into a multirange |
| Union/difference | Multirange operators do not fail on disjoint results |

**Syntax rules:**

- Multiranges are written with curly braces: `'{[3,7), [8,9)}'`.
- Multirange operators mirror range operators but operate on the whole collection.
- `range_agg()` aggregates ranges into a multirange, which can then be passed to `range_merge()` to compute the overall outer range.

**Constraints and limitations:**

- Multiranges require PostgreSQL 14 or later.
- They are always one-dimensional; there is no nesting of multiranges.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Multirange union and aggregation

```sql
-- Step 1: Create an empty multirange and a multirange with two disjoint ranges.
SELECT '{}'::int4multirange AS empty_mr,
       '{[3,7), [8,9)}'::int4multirange AS two_ranges;

-- Step 2: Aggregate ranges into a multirange.
SELECT range_agg(r) AS aggregated
FROM (VALUES
    (int4range(1, 5)),
    (int4range(10, 15)),
    (int4range(20, 25))
) AS t(r);

-- Step 3: Merge the aggregated multirange into the smallest covering range.
SELECT range_merge(range_agg(r)) AS outer_range
FROM (VALUES
    (int4range(1, 5)),
    (int4range(10, 15))
) AS t(r);
```

**Expected output:**

```text
 empty_mr |   two_ranges
----------+-----------------
 {}       | {[3,7),[8,9)}
(1 row)

    aggregated
-------------------
 {[1,5),[10,15),[20,25)}
(1 row)

 outer_range
-------------
 [1,15)
(1 row)
```

**Why this output appears**: The empty multirange `{}` contains no ranges. The multirange `{[3,7),[8,9)}` contains two disjoint ranges. `range_agg` collects the three input ranges into a multirange with three disjoint components. `range_merge` then computes the smallest single range that covers `[1,5)` and `[10,15)`, which is `[1,15)`—including the gap between them.

#### Real-World Cases

- **Availability windows**: A resource’s free time is stored as a multirange, combining multiple free slots into one value.
- **Occupied periods**: `range_agg()` aggregates individual bookings into a multirange, and `range_merge()` computes the overall outer range for reporting.
- **Subscription gaps**: A customer’s active subscription periods are stored as a multirange, allowing queries for gaps in coverage.

---

### 7. Range Best Practices

#### Definitions

**Core Definition**
Range best practices are the guidelines for choosing the right range type, using canonical bounds, indexing correctly, preventing overlaps with `EXCLUDE` constraints, and knowing when to use multiranges instead of ranges.

**Technical Definition**
Best practices include: (1) **use the most specific built-in range type** (`int4range` for integers, `tstzrange` for time-zone-aware timestamps, etc.) rather than creating custom types unless necessary; (2) **use half-open `[)` bounds** for scheduling and temporal data, because this matches SQL standard semantics and avoids boundary ambiguity; (3) **create GiST indexes on range columns** for overlap and containment queries; (4) **use `EXCLUDE USING GIST` constraints** to prevent overlapping values at the database level, combining equality on a group key with `&&` on the range when non-overlap is per-group; (5) **use multiranges** when the data genuinely consists of multiple disjoint intervals, and use `range_agg()` plus `range_merge()` to compute outer bounds; and (6) **avoid B-tree indexes for overlap queries**—they cannot accelerate `&&` or `@>` operators.

**Beginner-Friendly Explanation**
Range best practices are the habits that keep range data correct and fast. Use `[)` bounds for schedules so that “10:00 to 11:00” and “11:00 to 12:00” do not overlap at the boundary. Add a GiST index so overlap queries don’t scan the whole table. Use `EXCLUDE` constraints so the database itself refuses double bookings. And when you have multiple separate intervals, use a multirange instead of trying to force them into a single range.

#### Purposes

- To avoid boundary bugs caused by mismatched inclusive/exclusive bounds.
- To ensure that overlap queries are fast at scale.
- To enforce non-overlap integrity without relying on application code.
- To model multi-interval data correctly with multiranges.

#### Syntax Rules and Structure

**Best-practice schema pattern:**

```sql
-- Step 1: Enable btree_gist if combining equality with range overlap.
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Step 2: Create a table with a range column and a GiST index.
CREATE TABLE reservations (
    reservation_id serial PRIMARY KEY,
    room_id        int NOT NULL,
    during         tstzrange NOT NULL
);

-- Step 3: Prevent overlapping bookings per room.
ALTER TABLE reservations
    ADD CONSTRAINT no_overlap
    EXCLUDE USING GIST (room_id WITH =, during WITH &&);

-- Step 4: Add a GiST index for overlap queries (the EXCLUDE constraint already creates one,
-- but a separate index may be useful if the constraint is dropped or for other columns).
CREATE INDEX idx_reservations_during ON reservations USING GIST (during);
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Half-open bounds | Use `[start, end)` for scheduling |
| GiST index | `CREATE INDEX ... USING GIST (range_column)` |
| Per-group non-overlap | `EXCLUDE USING GIST (group_key WITH =, range_column WITH &&)` |
| Multirange for disjoint sets | `range_agg()` and multirange columns |

**Syntax rules:**

- For scheduling, use `[start, end)` so that adjacent bookings do not overlap.
- `EXCLUDE` constraints create an implicit GiST index; a separate GiST index may be redundant but can be useful if the constraint is conditional.
- Multirange union and difference operators do not fail on disjoint results, unlike range operators.

**Constraints and limitations:**

- GiST indexes have higher write overhead than B-tree indexes; use them only when range-operator queries are common.
- `EXCLUDE` constraints with `btree_gist` require the extension to be installed.
- Multiranges require PostgreSQL 14+.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Best-practice reservation table with half-open bounds and per-room non-overlap

```sql
-- Step 1: Enable btree_gist.
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Step 2: Create a reservation table with a half-open range and an EXCLUDE constraint.
CREATE TABLE reservations (
    reservation_id serial PRIMARY KEY,
    room_id        int NOT NULL,
    during         tstzrange NOT NULL,
    EXCLUDE USING GIST (room_id WITH =, during WITH &&)
);

-- Step 3: Insert a valid booking.
INSERT INTO reservations (room_id, during)
VALUES (101, '[2025-06-01 09:00, 2025-06-01 10:00)');

-- Step 4: Insert an adjacent booking that should succeed (half-open bounds).
INSERT INTO reservations (room_id, during)
VALUES (101, '[2025-06-01 10:00, 2025-06-01 11:00)');

-- Step 5: Verify both bookings exist.
SELECT reservation_id, room_id, during FROM reservations ORDER BY reservation_id;
```

**Expected output:**

```text
CREATE EXTENSION
CREATE TABLE
INSERT 0 1
INSERT 0 1
 reservation_id | room_id |                    during
----------------+---------+-----------------------------------------------
              1 |     101 | ["2025-06-01 09:00:00+00","2025-06-01 10:00:00+00")
              2 |     101 | ["2025-06-01 10:00:00+00","2025-06-01 11:00:00+00")
(2 rows)
```

**Why this output appears**: Both bookings use half-open `[)` bounds. The first ends at 10:00 (exclusive), and the second starts at 10:00 (inclusive). Because the bounds are half-open, the ranges do not overlap, so the `EXCLUDE` constraint permits both inserts. This is the canonical scheduling pattern.

#### Real-World Cases

- **Meeting room scheduling**: `EXCLUDE USING GIST (room_id WITH =, during WITH &&)` with `tstzrange` and half-open bounds prevents double-booking while allowing back-to-back meetings.
- **Rate validity periods**: A `rates` table uses `EXCLUDE USING GIST (product_id WITH =, validity WITH &&)` with `daterange` to ensure that a product never has two overlapping price periods.
- **Resource availability with gaps**: A `resources` table stores `available_during tsmultirange` to represent multiple free windows, and queries use `@>` to test whether a requested slot falls entirely within the available multirange.

---

## References

- PostgreSQL Documentation – Range Types – https://www.postgresql.org/docs/current/rangetypes.html
- PostgreSQL Documentation – Range/Multirange Functions and Operators – https://www.postgresql.org/docs/current/functions-range.html
- PostgreSQL Documentation – Range Types (PostgreSQL 13) – https://www.postgresql.org/docs/13/rangetypes.html
- PostgreSQL Documentation – Range/Multirange Functions and Operators (development) – https://www.postgresql.org/docs/devel/functions-range.html
- Red Gate – PostgreSQL Range Overlap Queries: GiST Indexes, EXCLUDE Constraints, and tstzrange Semantics – https://www.red-gate.com/simple-talk/databases/postgresql/overlapping-ranges-in-subsets-in-postgresql/
- Alibaba Cloud – Range Types in PostgreSQL and GiST Indexes – https://www.alibabacloud.com/blog/range-types-in-postgresql-and-gist-indexes_595128
- PostgreSQL Git – Range Type Canonicalization – https://git.postgresql.org/gitweb/?p=postgresql.git
- PostgreSQL Documentation – CREATE TYPE (Range Types) – https://www.postgresql.org/docs/current/sql-createtype.html
- PostgreSQL Documentation – CREATE INDEX (GiST) – https://www.postgresql.org/docs/current/sql-createindex.html
- PostgreSQL Documentation – ALTER TABLE (EXCLUDE Constraints) – https://www.postgresql.org/docs/current/sql-altertable.html
- PostgreSQL Documentation – Multirange Types – https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-BUILTIN
- PostgreSQL Wiki – Range Types – https://wiki.postgresql.org/wiki/RangeTypes