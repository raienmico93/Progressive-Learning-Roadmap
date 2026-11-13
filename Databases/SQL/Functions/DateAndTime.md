# SQL Date and Time Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL date and time functions are built-in routines provided by relational database management systems (RDBMS) that accept date, time, timestamp, or interval values as input and return a date/time value, an extracted component (e.g., year, month, day), or a numeric result, enabling temporal data manipulation, extraction, arithmetic, and formatting directly within SQL statements.

**Technical Definition**
In SQL, date/time functions operate on values of types `DATE`, `TIME`, `TIMESTAMP`, `TIMESTAMP WITH TIME ZONE` (`TIMESTAMPTZ`), and `INTERVAL`. They are partially standardised in SQL:1999, SQL:2003, and SQL:2016, but each RDBMS—PostgreSQL, MySQL, and SQL Server—implements a distinct set of functions with vendor-specific names, return types, and semantics. These functions are governed by the database's time zone settings, calendar rules (Gregorian vs. fiscal), and precision (fractional seconds). Arithmetic on temporal types follows the rules of interval addition and subtraction, with results depending on the operand types and the database's handling of daylight saving time (DST) transitions.

**Beginner-Friendly Explanation**
Think of date and time functions as a calendar and clock toolkit built into your database. Just as a calendar app lets you find today's date, calculate how many days until an event, or convert times between zones, SQL date/time functions let you work with dates and times stored in database columns—without exporting data to another program. You can find the current timestamp, extract the year from a date, add intervals, format dates for reports, and handle time zones, all inside SQL.

---

### Key Characteristics

1. **Vendor-Specific Naming and Semantics**: The same operation may be called `NOW()` in PostgreSQL and MySQL but `GETDATE()` in SQL Server; `EXTRACT` exists in PostgreSQL and MySQL but SQL Server uses `DATEPART`. `DATE_TRUNC` is PostgreSQL-specific, while MySQL uses `DATE_FORMAT` for truncation-like operations.
2. **Time Zone Awareness**: PostgreSQL's `TIMESTAMPTZ` and `AT TIME ZONE` operator, MySQL's `CONVERT_TZ()`, and SQL Server's `AT TIME ZONE` (2016+) provide time zone conversion, but their semantics differ significantly. PostgreSQL's `CURRENT_TIMESTAMP` returns `TIMESTAMPTZ`, while MySQL's `NOW()` returns a value in the session time zone.
3. **Transaction vs. Statement vs. Clock Time**: PostgreSQL distinguishes `NOW()` / `CURRENT_TIMESTAMP` (start of transaction), `STATEMENT_TIMESTAMP()` (start of statement), and `CLOCK_TIMESTAMP()` (actual clock time). MySQL's `NOW()` and `SYSDATE()` differ in evaluation timing. SQL Server's `GETDATE()` and `SYSDATETIME()` are evaluated at statement execution.
4. **Interval Arithmetic**: Adding an `INTERVAL` to a `DATE` or `TIMESTAMP` produces a new temporal value; subtracting two dates yields an integer number of days; subtracting two timestamps yields an `INTERVAL`. Month-based intervals are calendar-aware (e.g., adding 1 month to January 31 may yield February 28 or 29).
5. **Formatting and Parsing Are Asymmetric**: MySQL provides `DATE_FORMAT()` for formatting and `STR_TO_DATE()` for parsing; PostgreSQL uses `TO_CHAR()` and `TO_TIMESTAMP()` / `TO_DATE()`; SQL Server uses `FORMAT()` (2012+) and `CONVERT()` / `PARSE()`.
6. **Business Day and Fiscal Calendar Logic**: No RDBMS provides a built-in "business day" function; calculations require `GENERATE_SERIES` (PostgreSQL), recursive CTEs (MySQL 8.0+), or calendar tables. Fiscal calendars are application-defined and must be modelled explicitly.
7. **NULL Propagation**: Most date/time functions return `NULL` when any required argument is `NULL`, consistent with SQL's three-valued logic.

---

### Prerequisites

Before working with SQL date/time functions, you should understand:

- **Temporal data types**: `DATE` (date only), `TIME` (time only), `TIMESTAMP` (date + time, without time zone), `TIMESTAMPTZ` / `DATETIMEOFFSET` (with time zone), and `INTERVAL` (a duration).
- **Time zone concepts**: UTC, local time zones, IANA time zone names (e.g., `'America/New_York'`), and daylight saving time (DST) transitions.
- **Interval syntax**: `INTERVAL '1 day'`, `INTERVAL '2 hours 30 minutes'`, and vendor-specific interval units.
- **Date arithmetic operators**: `+`, `-`, and comparison operators on temporal types.
- **NULL semantics**: How `NULL` propagates through temporal expressions.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Reporting & Analytics** | `DATE_TRUNC`, `EXTRACT`, and formatting functions group data by day, month, quarter, or year. |
| **Financial Systems** | Interval arithmetic and date differences support interest accrual, aging reports, and period-end calculations. |
| **Scheduling & Logistics** | Business day calculations, weekday determination, and time zone conversion are essential for delivery and appointment systems. |
| **Audit & Logging** | `CURRENT_TIMESTAMP` and `NOW()` provide transaction-consistent timestamps for audit trails. |
| **Data Warehousing** | Date dimensions, fiscal calendars, and date parsing are core ETL operations. |
| **Internationalisation** | Time zone handling (`AT TIME ZONE`, `CONVERT_TZ`) is required for global applications. |

---

### Core Concepts / Features

The cheat sheet is organised into the following conceptual clusters:

1. **Current Date & Time** — `CURRENT_DATE`, `CURRENT_TIME`, `CURRENT_TIMESTAMP`, `NOW`
2. **Extraction & Truncation** — `EXTRACT`, `DATE_PART`, `DATE_TRUNC`
3. **Intervals & Arithmetic** — `INTERVAL`, `AGE`, Date Arithmetic, Date Differences, Date Ranges
4. **Time Zones** — `AT TIME ZONE`, Time Zone Handling
5. **Formatting & Parsing** — Date Formatting, Date Parsing
6. **Calendar & Business Logic** — Weekday Calculations, Business Day Calculations, Fiscal Calendars

Each feature is documented with definitions, purposes beginning with "To", syntax rules, constraints, annotated code examples with expected outputs, and real-world cases.

---

## 1. Current Date & Time

### 1.1 `CURRENT_DATE`

#### Definitions

**Core Definition**
`CURRENT_DATE` returns the current date (without a time component) according to the database server's time zone.

**Technical Definition**
`CURRENT_DATE` is a niladic SQL-standard keyword that returns a `DATE` value representing the current date at the start of the current transaction (in PostgreSQL) or the current statement (in MySQL and SQL Server). It is a synonym for `CURDATE()` in MySQL. In SQL Server, `CAST(GETDATE() AS date)` is the conventional equivalent.

**Beginner-Friendly Explanation**
`CURRENT_DATE` simply tells you today's date—no time, just the year, month, and day.

#### Purposes

- To obtain today's date for filtering or comparison.
- To calculate date differences relative to today (e.g., age, days until an event).
- To stamp records with the date of creation.
- To generate date-based reports for the current day.

#### Syntax Rules and Structure

**PostgreSQL**
```
CURRENT_DATE → date
```
**MySQL**
```
CURRENT_DATE() → date
CURRENT_DATE → date   (synonym for CURDATE())
```
**SQL Server**
```
CAST(GETDATE() AS date) → date
CURRENT_TIMESTAMP → datetime   (includes time; cast to date if needed)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `CURRENT_DATE` | SQL-standard keyword; no parentheses required in PostgreSQL. |
| `→ date` | Current date in the session's time zone. |

**Syntax Rules**

1. `CURRENT_DATE` takes no arguments.
2. In PostgreSQL, parentheses are optional and rarely used.
3. The result is based on the start of the current transaction (PostgreSQL) or statement (MySQL/SQL Server).
4. The date is determined by the session's `TimeZone` setting in PostgreSQL, the session time zone in MySQL, and the server's local time in SQL Server.

**Constraints and Limitations**

- Not portable in syntax: SQL Server does not support `CURRENT_DATE` directly; use `CAST(GETDATE() AS date)`.
- Transaction-time semantics may not reflect the actual clock time if a long transaction spans midnight.

#### Annotated Code Examples

**Example 1: PostgreSQL `CURRENT_DATE`**

```sql
SELECT CURRENT_DATE AS today;
```

**Expected Output (illustrative)**

| today      |
|------------|
| 2026-11-13 |

**Why This Output Occurs**: `CURRENT_DATE` returns the date at the start of the current transaction in the session's time zone.

**Example 2: MySQL `CURRENT_DATE` and `CURDATE()`**

```sql
SELECT CURRENT_DATE AS today, CURDATE() AS same_today;
```

**Expected Output (illustrative)**

| today      | same_today |
|------------|------------|
| 2026-11-13 | 2026-11-13 |

**Why This Output Occurs**: `CURRENT_DATE` is a synonym for `CURDATE()` in MySQL.

**Example 3: SQL Server Equivalent**

```sql
SELECT CAST(GETDATE() AS date) AS today;
```

**Expected Output (illustrative)**

| today      |
|------------|
| 2026-11-13 |

**Why This Output Occurs**: SQL Server uses `GETDATE()` for the current timestamp; casting to `date` removes the time portion.

#### Real-World Cases

- **Daily sales report**: `SELECT SUM(amount) FROM sales WHERE sale_date = CURRENT_DATE;`.
- **Age calculation**: `SELECT FLOOR(DATEDIFF(CURRENT_DATE, birth_date) / 365.25) AS age FROM users;`.
- **Expiry checks**: `SELECT * FROM subscriptions WHERE expiry_date < CURRENT_DATE;`.

---

### 1.2 `CURRENT_TIME`

#### Definitions

**Core Definition**
`CURRENT_TIME` returns the current time of day (without a date component) according to the database server's time zone.

**Technical Definition**
`CURRENT_TIME` is an SQL-standard keyword that returns a `TIME WITH TIME ZONE` value (PostgreSQL) or a `TIME` value (MySQL, SQL Server equivalent via `CAST(GETDATE() AS time)`). In PostgreSQL, it reflects the start of the current transaction; in MySQL, it is a synonym for `CURTIME()`.

**Beginner-Friendly Explanation**
`CURRENT_TIME` tells you what time it is right now—hours, minutes, and seconds, without the date.

#### Purposes

- To obtain the current time for scheduling or logging.
- To compare times of day across records.
- To stamp events with the time of occurrence.
- To calculate elapsed time within a day.

#### Syntax Rules and Structure

**PostgreSQL**
```
CURRENT_TIME → time with time zone
```
**MySQL**
```
CURRENT_TIME() → time
CURRENT_TIME → time   (synonym for CURTIME())
```
**SQL Server**
```
CAST(GETDATE() AS time) → time
```

**Syntax Rules**

1. `CURRENT_TIME` takes no arguments.
2. Precision can be specified in PostgreSQL: `CURRENT_TIME(precision)`.
3. In MySQL, `CURRENT_TIME(precision)` accepts a fractional seconds precision from 0 to 6.
4. The value is determined by the session time zone.

**Constraints and Limitations**

- SQL Server does not support `CURRENT_TIME` directly; use `CAST(GETDATE() AS time)`.
- Time zone offset is included in PostgreSQL's result; MySQL returns a time without offset.

#### Annotated Code Examples

**Example 1: PostgreSQL `CURRENT_TIME`**

```sql
SELECT CURRENT_TIME AS now_time;
```

**Expected Output (illustrative)**

| now_time          |
|-------------------|
| 14:39:53.662522-05 |

**Why This Output Occurs**: `CURRENT_TIME` returns the time with time zone offset at the start of the transaction.

**Example 2: MySQL `CURTIME()`**

```sql
SELECT CURTIME() AS current_time;
```

**Expected Output (illustrative)**

| current_time |
|--------------|
| 14:39:53     |

#### Real-World Cases

- **Shift scheduling**: Comparing `CURRENT_TIME` against shift start and end times.
- **Time-based access control**: Granting access only during certain hours.
- **Elapsed time tracking**: Calculating the difference between `CURRENT_TIME` and a recorded start time.

---

### 1.3 `CURRENT_TIMESTAMP`

#### Definitions

**Core Definition**
`CURRENT_TIMESTAMP` returns the current date and time (with or without time zone, depending on the RDBMS).

**Technical Definition**
`CURRENT_TIMESTAMP` is an SQL-standard keyword that returns a `TIMESTAMP WITH TIME ZONE` value in PostgreSQL, a `DATETIME` value in MySQL, and a `DATETIME` value in SQL Server. In PostgreSQL and MySQL, it is equivalent to `NOW()`; in SQL Server, it is equivalent to `GETDATE()`.

**Beginner-Friendly Explanation**
`CURRENT_TIMESTAMP` gives you the complete current date and time in one value—useful for recording exactly when something happened.

#### Purposes

- To stamp rows with the exact moment of insertion or update.
- To record transaction times for auditing.
- To compare timestamps across records.
- To calculate time differences with precision.

#### Syntax Rules and Structure

**PostgreSQL**
```
CURRENT_TIMESTAMP → timestamp with time zone
CURRENT_TIMESTAMP(precision) → timestamp with time zone
```
**MySQL**
```
CURRENT_TIMESTAMP([fsp]) → datetime
```
**SQL Server**
```
CURRENT_TIMESTAMP → datetime
```

**Syntax Rules**

1. `CURRENT_TIMESTAMP` takes no required arguments; optional precision (fractional seconds) is supported in PostgreSQL and MySQL.
2. In PostgreSQL, `CURRENT_TIMESTAMP` is equivalent to `NOW()` and returns the transaction start time.
3. In MySQL, `CURRENT_TIMESTAMP` is a synonym for `NOW()` and returns the statement start time.
4. In SQL Server, `CURRENT_TIMESTAMP` is an alias for `GETDATE()`.

**Constraints and Limitations**

- Return type differs: PostgreSQL returns `TIMESTAMPTZ`; MySQL and SQL Server return a time-zone-naive `DATETIME`.
- Transaction-time semantics in PostgreSQL mean all calls within a transaction return the same value.

#### Annotated Code Examples

**Example 1: PostgreSQL `CURRENT_TIMESTAMP`**

```sql
SELECT CURRENT_TIMESTAMP AS now;
```

**Expected Output (illustrative)**

| now                          |
|------------------------------|
| 2026-11-13 14:39:53.662522-05 |

**Why This Output Occurs**: `CURRENT_TIMESTAMP` returns the transaction start time with time zone.

**Example 2: MySQL `CURRENT_TIMESTAMP`**

```sql
SELECT CURRENT_TIMESTAMP AS now;
```

**Expected Output (illustrative)**

| now                 |
|---------------------|
| 2026-11-13 14:39:53 |

**Example 3: SQL Server `CURRENT_TIMESTAMP`**

```sql
SELECT CURRENT_TIMESTAMP AS now;
```

**Expected Output (illustrative)**

| now                     |
|-------------------------|
| 2026-11-13 14:39:53.360 |

**Why This Output Occurs**: `CURRENT_TIMESTAMP` is an alias for `GETDATE()` in SQL Server.

#### Real-World Cases

- **Audit logging**: `INSERT INTO audit_log (event, created_at) VALUES ('login', CURRENT_TIMESTAMP);`.
- **Order tracking**: Recording the exact time an order was placed.
- **Session expiry**: `SELECT * FROM sessions WHERE last_activity < CURRENT_TIMESTAMP - INTERVAL '30 minutes';`.

---

### 1.4 `NOW`

#### Definitions

**Core Definition**
`NOW()` returns the current date and time, typically as a timestamp with time zone.

**Technical Definition**
`NOW()` is a PostgreSQL function that returns the current date and time as `TIMESTAMP WITH TIME ZONE`, based on the start of the current transaction. It is equivalent to `CURRENT_TIMESTAMP` and `TRANSACTION_TIMESTAMP()`. In MySQL, `NOW()` returns the current date and time as `DATETIME` at the start of the statement; it is a synonym for `CURRENT_TIMESTAMP` and `LOCALTIMESTAMP`.

**Beginner-Friendly Explanation**
`NOW()` is the classic way to get "right now" in SQL—it returns the current date and time in one call.

#### Purposes

- To obtain a transaction-consistent timestamp.
- To stamp records with the moment of creation.
- To calculate time differences relative to the present.
- To generate dynamic date ranges for queries.

#### Syntax Rules and Structure

**PostgreSQL**
```
NOW() → timestamp with time zone
```
**MySQL**
```
NOW([fsp]) → datetime
```

**Syntax Rules**

1. `NOW()` takes an optional fractional seconds precision argument in MySQL (0–6); PostgreSQL's `NOW()` takes no arguments.
2. In PostgreSQL, `NOW()` is transaction-time; all calls within a transaction return the same value.
3. In MySQL, `NOW()` is statement-time and may differ from `SYSDATE()` if the statement takes time to execute.

**Constraints and Limitations**

- SQL Server does not have `NOW()`; use `GETDATE()` or `SYSDATETIME()`.
- Transaction-time semantics may not reflect the actual clock at the moment a specific row is processed.

#### Annotated Code Examples

**Example 1: PostgreSQL `NOW()`**

```sql
SELECT NOW() AS current_moment;
```

**Expected Output (illustrative)**

| current_moment               |
|------------------------------|
| 2026-11-13 14:39:53.662522-05 |

**Why This Output Occurs**: `NOW()` returns the transaction start time with time zone.

**Example 2: MySQL `NOW()` with Precision**

```sql
SELECT NOW(6) AS precise_now;
```

**Expected Output (illustrative)**

| precise_now                 |
|-----------------------------|
| 2026-11-13 14:39:53.662522 |

**Why This Output Occurs**: The `6` argument requests six digits of fractional seconds.

#### Real-World Cases

- **Dynamic filtering**: `SELECT * FROM orders WHERE order_date >= NOW() - INTERVAL '7 days';`.
- **Record stamping**: `UPDATE users SET last_login = NOW() WHERE user_id = 42;`.
- **Transaction auditing**: Ensuring all rows in a batch share the same timestamp.

---

## 2. Extraction & Truncation

### 2.1 `EXTRACT`

#### Definitions

**Core Definition**
`EXTRACT` retrieves a single component (e.g., year, month, day, hour) from a date, time, timestamp, or interval value.

**Technical Definition**
`EXTRACT(field FROM source)` returns a numeric value representing the specified field of the source temporal value. The `field` is a keyword or string (e.g., `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`, `SECOND`, `DOW`, `DOY`, `EPOCH`, `QUARTER`, `WEEK`). PostgreSQL and MySQL support this SQL-standard syntax. In PostgreSQL, the return type is `NUMERIC`; in MySQL, it is an integer or double depending on the field.

**Beginner-Friendly Explanation**
`EXTRACT` lets you pull out one piece of a date or time—like getting just the year from a full date, or just the hour from a timestamp.

#### Purposes

- To group or filter data by year, month, quarter, or day of the week.
- To calculate age or tenure using extracted components.
- To analyse temporal patterns (e.g., sales by hour of day).
- To build custom date labels for reports.

#### Syntax Rules and Structure

**PostgreSQL**
```
EXTRACT ( field FROM source ) → numeric
```
**MySQL**
```
EXTRACT ( unit FROM date ) → integer/double
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `field` / `unit` | `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`, `SECOND`, `DOW`, `DOY`, `EPOCH`, `QUARTER`, `WEEK`, etc. |
| `source` / `date` | A `DATE`, `TIME`, `TIMESTAMP`, or `INTERVAL` value. |
| `→ numeric` | Extracted component. |

**Syntax Rules**

1. The `FROM` keyword is required in standard syntax.
2. `DOW` (day of week) returns 0 (Sunday) to 6 (Saturday) in PostgreSQL; MySQL's `DAYOFWEEK()` returns 1 (Sunday) to 7 (Saturday), but `EXTRACT(DOW ...)` in MySQL also returns 0–6? Check vendor documentation.
3. `EPOCH` returns the number of seconds since 1970-01-01 00:00:00 UTC.
4. `EXTRACT` on an `INTERVAL` returns the corresponding field value.

**Constraints and Limitations**

- Field names and return values vary by RDBMS (e.g., `DOW` semantics differ).
- SQL Server does not support `EXTRACT`; use `DATEPART()` instead.
- `EXTRACT` from a `TIME` value with time zone may require special handling.

#### Annotated Code Examples

**Example 1: Extracting Year and Month (PostgreSQL)**

```sql
SELECT
  EXTRACT(YEAR FROM TIMESTAMP '2001-02-16 20:38:40') AS yr,
  EXTRACT(MONTH FROM TIMESTAMP '2001-02-16 20:38:40') AS mon;
```

**Expected Output**

| yr   | mon |
|------|-----|
| 2001 | 2   |

**Why This Output Occurs**: `EXTRACT` retrieves the year and month components from the timestamp.

**Example 2: Extracting Hour from a Timestamp (MySQL)**

```sql
SELECT EXTRACT(HOUR FROM '2026-11-13 14:39:53') AS hr;
```

**Expected Output**

| hr |
|----|
| 14 |

**Why This Output Occurs**: `EXTRACT(HOUR ...)` returns the hour component.

**Example 3: Extracting Day of Week (PostgreSQL)**

```sql
SELECT EXTRACT(DOW FROM DATE '2026-11-13') AS day_of_week;
```

**Expected Output**

| day_of_week |
|-------------|
| 4           |

**Why This Output Occurs**: 2026-11-13 is a Friday; PostgreSQL's `DOW` returns 4 for Friday (0 = Sunday).

#### Real-World Cases

- **Sales by quarter**: `SELECT EXTRACT(QUARTER FROM sale_date) AS qtr, SUM(amount) FROM sales GROUP BY qtr;`.
- **Peak hours analysis**: `SELECT EXTRACT(HOUR FROM order_time) AS hr, COUNT(*) FROM orders GROUP BY hr;`.
- **Age cohort analysis**: `SELECT EXTRACT(YEAR FROM age(birth_date)) AS cohort, COUNT(*) FROM users GROUP BY cohort;`.

---

### 2.2 `DATE_PART`

#### Definitions

**Core Definition**
`DATE_PART` is a PostgreSQL function that returns a subfield of a timestamp or interval, equivalent to `EXTRACT`.

**Technical Definition**
`DATE_PART(field, source)` returns a `DOUBLE PRECISION` value representing the specified field of the source. It is functionally equivalent to `EXTRACT(field FROM source)` but uses a comma-separated argument style and always returns `DOUBLE PRECISION` (whereas `EXTRACT` returns `NUMERIC` for some fields in PostgreSQL).

**Beginner-Friendly Explanation**
`DATE_PART` does the same job as `EXTRACT` but with a different calling style—both let you pull out one piece of a date or time.

#### Purposes

- To extract temporal components when a `DOUBLE PRECISION` result is desired.
- To provide a comma-separated alternative to `EXTRACT` in PostgreSQL code.
- To maintain compatibility with legacy PostgreSQL code that uses `DATE_PART`.

#### Syntax Rules and Structure

**PostgreSQL**
```
DATE_PART ( field text, source timestamp ) → double precision
DATE_PART ( field text, source interval ) → double precision
```

**Syntax Rules**

1. `field` is a string literal (e.g., `'hour'`, `'month'`).
2. `source` can be a `TIMESTAMP`, `TIMESTAMPTZ`, `TIME`, or `INTERVAL`.
3. Returns `DOUBLE PRECISION`.

**Constraints and Limitations**

- **PostgreSQL-specific**; not available in MySQL or SQL Server.
- Returns floating-point values even for integer-like fields.

#### Annotated Code Examples

**Example 1: `DATE_PART` for Hour**

```sql
SELECT DATE_PART('hour', TIMESTAMP '2001-02-16 20:38:40') AS hr;
```

**Expected Output**

| hr |
|----|
| 20 |

**Why This Output Occurs**: The hour component of the timestamp is 20.

**Example 2: Contrast with `EXTRACT`**

```sql
SELECT
  EXTRACT(MONTH FROM TIMESTAMP '2001-02-16 20:38:40') AS extract_month,
  DATE_PART('month', TIMESTAMP '2001-02-16 20:38:40') AS date_part_month;
```

**Expected Output**

| extract_month | date_part_month |
|---------------|-----------------|
| 2             | 2               |

Both return the same value; only the syntax and return type differ.

#### Real-World Cases

- **Legacy PostgreSQL code**: Maintaining compatibility with older applications that use `DATE_PART`.
- **Floating-point calculations**: When extracted components feed into calculations requiring `DOUBLE PRECISION`.

---

### 2.3 `DATE_TRUNC`

#### Definitions

**Core Definition**
`DATE_TRUNC` truncates a timestamp or interval to a specified precision (e.g., hour, day, month, year).

**Technical Definition**
`DATE_TRUNC(field, source)` returns a `TIMESTAMP` or `INTERVAL` with the value truncated to the specified precision. For example, truncating `2001-02-16 20:38:40` to `'hour'` yields `2001-02-16 20:00:00`. PostgreSQL supports truncation to `microseconds`, `milliseconds`, `second`, `minute`, `hour`, `day`, `week`, `month`, `quarter`, `year`, `decade`, `century`, and `millennium`.

**Beginner-Friendly Explanation**
`DATE_TRUNC` chops off the smaller parts of a date or time—like turning `14:39:53` into `14:00:00` (truncating to the hour) or `2026-11-13` into `2026-11-01` (truncating to the month).

#### Purposes

- To group time-series data into buckets (hourly, daily, monthly, etc.).
- To standardise timestamps for comparison by removing lower-order components.
- To calculate period boundaries (start of day, start of month).
- To align data to calendar or fiscal periods.

#### Syntax Rules and Structure

**PostgreSQL**
```
DATE_TRUNC ( field text, source timestamp ) → timestamp
DATE_TRUNC ( field text, source timestamptz [, timezone text] ) → timestamptz
DATE_TRUNC ( field text, source interval ) → interval
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `field` | Truncation precision (e.g., `'hour'`, `'day'`, `'month'`). |
| `source` | `TIMESTAMP`, `TIMESTAMPTZ`, or `INTERVAL`. |
| `timezone` | Optional time zone for `TIMESTAMPTZ` truncation. |

**Syntax Rules**

1. `field` must be a valid precision string.
2. Truncating to `'week'` sets the date to the Monday of that week in PostgreSQL.
3. The optional time zone argument (PostgreSQL 16+) allows truncation in a specific zone.

**Constraints and Limitations**

- **PostgreSQL-specific**; MySQL and SQL Server use different techniques (e.g., `DATE_FORMAT` for month truncation in MySQL, `DATEFROMPARTS` in SQL Server).
- Truncating a `TIMESTAMPTZ` without a time zone uses the session time zone.

#### Annotated Code Examples

**Example 1: Truncating to the Hour**

```sql
SELECT DATE_TRUNC('hour', TIMESTAMP '2001-02-16 20:38:40') AS truncated_hour;
```

**Expected Output**

| truncated_hour     |
|--------------------|
| 2001-02-16 20:00:00 |

**Why This Output Occurs**: The minutes and seconds are set to zero.

**Example 2: Truncating to the Month**

```sql
SELECT DATE_TRUNC('month', DATE '2026-11-13') AS start_of_month;
```

**Expected Output**

| start_of_month |
|----------------|
| 2026-11-01     |

**Why This Output Occurs**: The day and time are set to the first of the month at midnight.

**Example 3: MySQL Equivalent for Month Truncation**

```sql
SELECT DATE_FORMAT('2026-11-13', '%Y-%m-01') AS start_of_month;
```

**Expected Output**

| start_of_month |
|----------------|
| 2026-11-01     |

**Why This Output Occurs**: MySQL lacks `DATE_TRUNC`; `DATE_FORMAT` constructs the first day of the month.

#### Real-World Cases

- **Daily active users**: `SELECT DATE_TRUNC('day', login_time) AS day, COUNT(DISTINCT user_id) FROM logins GROUP BY day;`.
- **Monthly revenue**: `SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) FROM sales GROUP BY month;`.
- **Hourly traffic**: Bucketing web server logs by hour for analysis.

---

## 3. Intervals & Arithmetic

### 3.1 `INTERVAL`

#### Definitions

**Core Definition**
`INTERVAL` represents a duration or span of time, such as `'1 day'`, `'2 hours 30 minutes'`, or `'3 months'`.

**Technical Definition**
The `INTERVAL` data type stores a period of time expressed in years, months, days, hours, minutes, and seconds. In PostgreSQL, intervals are stored internally as months, days, and microseconds (three separate fields), which allows calendar-aware arithmetic (e.g., adding 1 month to January 31 respects month length). MySQL uses a simpler interval representation. SQL Server does not have a dedicated `INTERVAL` type; intervals are expressed via `DATEADD` and `DATEDIFF`.

**Beginner-Friendly Explanation**
An `INTERVAL` is a length of time—like "3 days" or "2 hours"—that you can add to or subtract from a date or timestamp.

#### Purposes

- To perform date arithmetic (adding or subtracting durations).
- To represent time spans in scheduling and billing.
- To calculate differences between timestamps as intervals.
- To model recurring periods (e.g., subscription lengths).

#### Syntax Rules and Structure

**PostgreSQL Interval Literals**
```
INTERVAL '1 day'
INTERVAL '2 hours 30 minutes'
INTERVAL '3 months'
INTERVAL '1 year 2 months 3 days 04:05:06'
```
**MySQL Interval Syntax**
```
INTERVAL expr unit
```
where `unit` is `DAY`, `HOUR`, `MINUTE`, `SECOND`, `MONTH`, `YEAR`, etc.

**Syntax Rules**

1. PostgreSQL intervals can combine multiple fields in a single literal.
2. MySQL intervals use a numeric expression followed by a unit keyword.
3. Interval arithmetic is calendar-aware for month and year components.
4. Adding an interval to a `DATE` yields a `TIMESTAMP` (unless the interval is day-only, in which case it may yield a `DATE`).

**Constraints and Limitations**

- SQL Server has no `INTERVAL` type; use `DATEADD` for arithmetic.
- Interval comparison and ordering rules vary by RDBMS.
- Month-based intervals are not fixed-length (28–31 days).

#### Annotated Code Examples

**Example 1: Adding an Interval to a Timestamp (PostgreSQL)**

```sql
SELECT TIMESTAMP '2026-11-13 10:00:00' + INTERVAL '1 day 2 hours' AS future_time;
```

**Expected Output**

| future_time        |
|--------------------|
| 2026-11-14 12:00:00 |

**Why This Output Occurs**: The interval `'1 day 2 hours'` is added to the timestamp.

**Example 2: MySQL `DATE_ADD` with Interval**

```sql
SELECT DATE_ADD('2026-11-13', INTERVAL 31 DAY) AS future_date;
```

**Expected Output**

| future_date |
|-------------|
| 2026-12-14  |

**Why This Output Occurs**: MySQL uses `DATE_ADD` with an interval to add 31 days.

#### Real-World Cases

- **Subscription expiry**: `SELECT start_date + INTERVAL '1 year' AS expiry_date FROM subscriptions;`.
- **SLA deadlines**: `SELECT created_at + INTERVAL '48 hours' AS deadline FROM tickets;`.
- **Recurring billing**: Calculating the next billing date by adding an interval.

---

### 3.2 `AGE`

#### Definitions

**Core Definition**
`AGE` returns the difference between two timestamps (or between a timestamp and the current date) as a symbolic interval using years, months, and days.

**Technical Definition**
`AGE(timestamp, timestamp)` subtracts the second argument from the first and returns an `INTERVAL` expressed in years, months, and days, rather than just total days. `AGE(timestamp)` subtracts the argument from `CURRENT_DATE` (at midnight). This symbolic representation is useful for human-readable age calculations (e.g., "43 years 8 months 15 days").

**Beginner-Friendly Explanation**
`AGE` calculates how much time has passed between two dates and expresses it in a way humans understand—years, months, and days—rather than just a total number of days.

#### Purposes

- To calculate a person's age in years, months, and days.
- To compute tenure or duration in a human-readable format.
- To measure the time elapsed since an event.
- To produce symbolic intervals for display.

#### Syntax Rules and Structure

**PostgreSQL**
```
AGE ( timestamp, timestamp ) → interval
AGE ( timestamp ) → interval
```

**Syntax Rules**

1. The two-argument form subtracts the second argument from the first.
2. The one-argument form subtracts the argument from `CURRENT_DATE` (midnight).
3. The result is a symbolic interval (e.g., `43 years 8 mons 15 days`).
4. The result can be negative if the end timestamp is earlier than the start timestamp.

**Constraints and Limitations**

- **PostgreSQL-specific**; MySQL and SQL Server use `TIMESTAMPDIFF` or `DATEDIFF` for similar calculations but return numeric differences, not symbolic intervals.
- The symbolic interval is not suitable for precise arithmetic; use timestamp subtraction for exact durations.

#### Annotated Code Examples

**Example 1: Calculating Age**

```sql
SELECT AGE(TIMESTAMP '2001-04-10', TIMESTAMP '1957-06-13') AS age_interval;
```

**Expected Output**

| age_interval          |
|-----------------------|
| 43 years 9 mons 27 days |

**Why This Output Occurs**: `AGE` computes the symbolic difference between the two timestamps.

**Example 2: Age from Current Date**

```sql
SELECT AGE(TIMESTAMP '1957-06-13') AS age_since_birth;
```

**Expected Output (illustrative)**

| age_since_birth       |
|-----------------------|
| 69 years 5 mons 0 days |

**Why This Output Occurs**: The one-argument form subtracts the timestamp from `CURRENT_DATE`.

#### Real-World Cases

- **Customer age**: `SELECT AGE(birth_date) AS age FROM customers;`.
- **Employee tenure**: `SELECT AGE(hire_date) AS tenure FROM employees;`.
- **Asset age**: Calculating how long equipment has been in service.

---

### 3.3 Date Arithmetic

#### Definitions

**Core Definition**
Date arithmetic refers to the use of operators (`+`, `-`) to add or subtract intervals and integers from date/time values, or to subtract two temporal values to obtain a difference.

**Technical Definition**
SQL supports a family of arithmetic operators on temporal types: `date + integer` adds days; `date + interval` yields a timestamp; `timestamp - timestamp` yields an interval; `date - date` yields an integer number of days. The exact operators and return types are documented in the SQL standard and vendor-specific references.

**Beginner-Friendly Explanation**
Date arithmetic lets you do math with dates—like adding 7 days to today, subtracting two dates to find the gap, or adding 2 hours to a timestamp.

#### Purposes

- To compute future or past dates (e.g., due dates, renewal dates).
- To calculate elapsed time between events.
- To shift timestamps by a fixed duration.
- To derive date ranges for queries.

#### Syntax Rules and Structure

**PostgreSQL Operators**
```
date + integer → date
date + interval → timestamp
timestamp + interval → timestamp
timestamp - timestamp → interval
date - date → integer
interval + interval → interval
interval - interval → interval
```

**MySQL Operators**
```
date + INTERVAL expr unit
date - INTERVAL expr unit
DATEDIFF(date1, date2) → integer days
TIMESTAMPDIFF(unit, datetime1, datetime2) → integer
```

**SQL Server Operators**
```
DATEADD(datepart, number, date) → datetime
DATEDIFF(datepart, startdate, enddate) → integer
```

**Syntax Rules**

1. Adding an integer to a `DATE` adds that many days.
2. Adding an `INTERVAL` to a `DATE` or `TIMESTAMP` yields a `TIMESTAMP`.
3. Subtracting two `DATE` values yields an integer number of days.
4. Subtracting two `TIMESTAMP` values yields an `INTERVAL`.
5. Month and year intervals are calendar-aware.

**Constraints and Limitations**

- Operator behaviour varies by RDBMS; SQL Server uses functions (`DATEADD`, `DATEDIFF`) instead of operators.
- DST transitions can affect time arithmetic when time zones are involved.

#### Annotated Code Examples

**Example 1: Adding Days to a Date (PostgreSQL)**

```sql
SELECT DATE '2001-09-28' + 7 AS new_date;
```

**Expected Output**

| new_date   |
|------------|
| 2001-10-05 |

**Why This Output Occurs**: Adding an integer to a date adds that many days.

**Example 2: Subtracting Dates (PostgreSQL)**

```sql
SELECT DATE '2001-10-01' - DATE '2001-09-28' AS days_elapsed;
```

**Expected Output**

| days_elapsed |
|--------------|
| 3            |

**Why This Output Occurs**: Subtracting two dates yields the number of days between them.

**Example 3: SQL Server `DATEADD`**

```sql
SELECT DATEADD(day, 7, '2001-09-28') AS new_date;
```

**Expected Output**

| new_date   |
|------------|
| 2001-10-05 |

**Why This Output Occurs**: `DATEADD` adds the specified number of datepart units to the date.

#### Real-World Cases

- **Due date calculation**: `SELECT order_date + INTERVAL '30 days' AS due_date FROM orders;`.
- **Days since event**: `SELECT CURRENT_DATE - event_date AS days_since FROM events;`.
- **Renewal reminders**: Filtering records where `expiry_date - CURRENT_DATE <= 7`.

---

### 3.4 Date Differences

#### Definitions

**Core Definition**
Date differences refer to the computation of the elapsed time between two temporal values, typically expressed as an integer number of days, months, or other units.

**Technical Definition**
Different RDBMS provide different functions for date differences: PostgreSQL uses the `-` operator (`date - date → integer`, `timestamp - timestamp → interval`); MySQL provides `DATEDIFF(later, earlier)` for day differences and `TIMESTAMPDIFF(unit, earlier, later)` for other units; SQL Server provides `DATEDIFF(datepart, startdate, enddate)` which returns the number of datepart boundaries crossed.

**Beginner-Friendly Explanation**
Date differences tell you how far apart two dates are—like "5 days between Monday and Saturday" or "3 months between January and April."

#### Purposes

- To calculate elapsed time or age.
- To measure durations for SLA or performance analysis.
- To filter records within a date range.
- To compute intervals for scheduling and billing.

#### Syntax Rules and Structure

**PostgreSQL**
```
date1 - date2 → integer   (days)
timestamp1 - timestamp2 → interval
```
**MySQL**
```
DATEDIFF(expr1, expr2) → integer days
TIMESTAMPDIFF(unit, datetime_expr1, datetime_expr2) → integer
```
**SQL Server**
```
DATEDIFF(datepart, startdate, enddate) → integer
```

**Syntax Rules**

1. MySQL's `DATEDIFF(later, earlier)` returns `later - earlier` in days; argument order matters.
2. SQL Server's `DATEDIFF` counts boundaries crossed, so `DATEDIFF(year, '2025-12-31', '2026-01-01')` returns `1`.
3. PostgreSQL's `timestamp - timestamp` returns an interval, which can be converted to days with `EXTRACT(DAY FROM interval)`.

**Constraints and Limitations**

- SQL Server's boundary-counting semantics can produce non-intuitive results for partial periods.
- DST transitions can cause `TIMESTAMPDIFF` to return unexpected values when time zones are involved.

#### Annotated Code Examples

**Example 1: MySQL `DATEDIFF`**

```sql
SELECT DATEDIFF('2026-11-13', '2026-11-01') AS days_diff;
```

**Expected Output**

| days_diff |
|-----------|
| 12        |

**Why This Output Occurs**: `DATEDIFF` subtracts the second date from the first.

**Example 2: SQL Server `DATEDIFF`**

```sql
SELECT DATEDIFF(day, '2026-11-01', '2026-11-13') AS days_diff;
```

**Expected Output**

| days_diff |
|-----------|
| 12        |

**Why This Output Occurs**: `DATEDIFF` counts the number of day boundaries between the two dates.

**Example 3: PostgreSQL Timestamp Difference**

```sql
SELECT TIMESTAMP '2026-11-13 15:00:00' - TIMESTAMP '2026-11-13 10:00:00' AS time_diff;
```

**Expected Output**

| time_diff |
|-----------|
| 05:00:00  |

**Why This Output Occurs**: Subtracting timestamps yields an interval.

#### Real-World Cases

- **Age in days**: `SELECT DATEDIFF(CURRENT_DATE, birth_date) AS age_days FROM users;`.
- **SLA compliance**: `SELECT DATEDIFF(hour, created_at, resolved_at) AS resolution_hours FROM tickets;`.
- **Subscription length**: Calculating the number of days between start and end dates.

---

### 3.5 Date Ranges

#### Definitions

**Core Definition**
Date ranges refer to queries and expressions that operate on a span of dates, often using `BETWEEN`, `>=`, `<=`, or overlap operators.

**Technical Definition**
SQL provides range comparison operators (`BETWEEN`, `>=`, `<=`, `>`, `<`) and the `OVERLAPS` operator for determining whether two time periods overlap. PostgreSQL supports `OVERLAPS` with date/time pairs or intervals. Range queries are fundamental to temporal data filtering and are subject to index optimisation considerations.

**Beginner-Friendly Explanation**
Date ranges let you ask questions like "show me all sales between January and March" or "find appointments that overlap with this time slot."

#### Purposes

- To filter records within a start and end date.
- To detect overlapping time periods (e.g., booking conflicts).
- To aggregate data over a range.
- To generate series of dates for reporting.

#### Syntax Rules and Structure

**Range Filtering**
```
column BETWEEN start AND end
column >= start AND column <= end
```

**PostgreSQL `OVERLAPS`**
```
(start1, end1) OVERLAPS (start2, end2)
(start1, length1) OVERLAPS (start2, length2)
```

**Syntax Rules**

1. `BETWEEN` is inclusive of both endpoints.
2. `OVERLAPS` treats periods as half-open intervals `[start, end)`, so periods sharing only an endpoint do not overlap.
3. `GENERATE_SERIES(start, end, interval)` in PostgreSQL produces a series of timestamps or dates.

**Constraints and Limitations**

- `OVERLAPS` is not universally supported; MySQL lacks a direct equivalent.
- Range queries on functions (e.g., `DATE_TRUNC`) may prevent index usage.

#### Annotated Code Examples

**Example 1: Filtering a Date Range**

```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2026-01-01' AND '2026-03-31';
```

**Expected Output**: All orders placed in Q1 2026.

**Example 2: PostgreSQL `OVERLAPS`**

```sql
SELECT (DATE '2026-11-01', DATE '2026-11-10') OVERLAPS (DATE '2026-11-05', DATE '2026-11-15') AS overlap;
```

**Expected Output**

| overlap |
|---------|
| t       |

**Why This Output Occurs**: The two periods share dates from November 5 to November 10.

**Example 3: Generating a Date Series (PostgreSQL)**

```sql
SELECT GENERATE_SERIES('2026-11-01'::date, '2026-11-05'::date, '1 day') AS day;
```

**Expected Output**

| day        |
|------------|
| 2026-11-01 |
| 2026-11-02 |
| 2026-11-03 |
| 2026-11-04 |
| 2026-11-05 |

#### Real-World Cases

- **Booking systems**: Detecting overlapping reservations.
- **Financial reporting**: Aggregating transactions within a fiscal period.
- **Calendar generation**: Producing a list of dates for a report.

---

## 4. Time Zones

### 4.1 `AT TIME ZONE`

#### Definitions

**Core Definition**
`AT TIME ZONE` converts a timestamp to a different time zone.

**Technical Definition**
`AT TIME ZONE` is an operator that converts a `TIMESTAMP` or `TIMESTAMPTZ` expression to the specified time zone. In PostgreSQL, `timestamp AT TIME ZONE zone` interprets a timestamp without time zone as being in the specified zone and returns a `TIMESTAMPTZ`; `timestamptz AT TIME ZONE zone` converts a `TIMESTAMPTZ` to a timestamp without time zone in the specified zone. In SQL Server (2016+), `input_date AT TIME ZONE time_zone_value` converts a date to a `DATETIMEOFFSET` in the target time zone.

**Beginner-Friendly Explanation**
`AT TIME ZONE` lets you say "what time is it in Tokyo?" when you have a timestamp recorded in New York—or convert a local time to UTC.

#### Purposes

- To convert timestamps between time zones for reporting.
- To store timestamps in UTC and display them in local time.
- To compare events that occurred in different time zones.
- To handle daylight saving time transitions correctly.

#### Syntax Rules and Structure

**PostgreSQL**
```
timestamp AT TIME ZONE zone → timestamptz
timestamptz AT TIME ZONE zone → timestamp
```
**SQL Server**
```
input_date AT TIME ZONE time_zone_value → datetimeoffset
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `timestamp` / `input_date` | `TIMESTAMP`, `TIMESTAMPTZ`, or `DATETIME`. |
| `zone` / `time_zone_value` | IANA zone name (e.g., `'America/New_York'`) or offset. |
| `→` | Converted value. |

**Syntax Rules**

1. In PostgreSQL, `AT TIME ZONE` reverses the type: `TIMESTAMP` → `TIMESTAMPTZ` and `TIMESTAMPTZ` → `TIMESTAMP`.
2. In SQL Server, the result is `DATETIMEOFFSET` (includes the offset).
3. Zone names should be IANA time zone identifiers for DST accuracy.

**Constraints and Limitations**

- SQL Server's `AT TIME ZONE` requires SQL Server 2016 or later.
- MySQL does not have `AT TIME ZONE`; use `CONVERT_TZ()` instead.
- Time zone data must be installed and up to date.

#### Annotated Code Examples

**Example 1: PostgreSQL `AT TIME ZONE`**

```sql
SELECT TIMESTAMP '2001-02-16 20:38:40' AT TIME ZONE 'America/Denver' AS denver_time;
```

**Expected Output (illustrative)**

| denver_time               |
|---------------------------|
| 2001-02-16 20:38:40-07   |

**Why This Output Occurs**: The timestamp without time zone is interpreted as being in `America/Denver` and returned as `TIMESTAMPTZ`.

**Example 2: SQL Server `AT TIME ZONE`**

```sql
SELECT '2026-11-13 14:00:00' AT TIME ZONE 'Central European Standard Time' AS cet_time;
```

**Expected Output (illustrative)**

| cet_time                         |
|----------------------------------|
| 2026-11-13 14:00:00.000 +01:00   |

**Why This Output Occurs**: The input date is converted to a `DATETIMEOFFSET` in the target time zone.

#### Real-World Cases

- **Global dashboards**: Converting UTC-stored timestamps to each user's local time.
- **Log analysis**: Normalising logs from servers in different time zones.
- **Scheduling**: Converting meeting times between participant time zones.

---

### 4.2 Time Zone Handling

#### Definitions

**Core Definition**
Time zone handling encompasses the practices and functions used to store, convert, and compare temporal data across different time zones.

**Technical Definition**
Time zone handling involves the use of time-zone-aware data types (`TIMESTAMPTZ`, `DATETIMEOFFSET`), conversion functions (`AT TIME ZONE`, `CONVERT_TZ`, `SWITCHOFFSET`), and configuration settings (`TimeZone` in PostgreSQL, session time zone in MySQL, `AT TIME ZONE` in SQL Server). Correct handling requires understanding the distinction between wall-clock time and absolute time (UTC), and the impact of daylight saving time transitions.

**Beginner-Friendly Explanation**
Time zone handling is about making sure that a time recorded in one part of the world is correctly understood and displayed in another—without getting confused by daylight saving changes.

#### Purposes

- To store timestamps consistently in UTC.
- To display timestamps in the user's local time zone.
- To compare events across time zones accurately.
- To avoid DST-related bugs in scheduling and billing.

#### Syntax Rules and Structure

**PostgreSQL**
```
SET TIME ZONE 'America/New_York';
SHOW TIME ZONE;
timestamptz AT TIME ZONE 'UTC';
```
**MySQL**
```
SET time_zone = '+00:00';
CONVERT_TZ(dt, from_tz, to_tz);
```
**SQL Server**
```
AT TIME ZONE 'UTC';
SWITCHOFFSET(datetimeoffset, '-05:00');
```

**Constraints and Limitations**

- MySQL requires time zone tables to be populated for named zones.
- Storing timestamps without time zone can lead to ambiguity during DST transitions.
- Different RDBMS interpret `TIMESTAMP` (without time zone) differently.

#### Annotated Code Examples

**Example 1: MySQL `CONVERT_TZ`**

```sql
SELECT CONVERT_TZ('2026-11-13 14:00:00', 'UTC', 'America/New_York') AS ny_time;
```

**Expected Output (illustrative)**

| ny_time            |
|--------------------|
| 2026-11-13 09:00:00 |

**Why This Output Occurs**: `CONVERT_TZ` converts from UTC to New York time (UTC-5 in November).

#### Real-World Cases

- **Multi-region applications**: Storing all timestamps in UTC and converting for display.
- **Financial trading**: Ensuring trade timestamps are comparable across exchanges.
- **Travel booking**: Converting flight times between departure and arrival zones.

---

## 5. Formatting & Parsing

### 5.1 Date Formatting

#### Definitions

**Core Definition**
Date formatting converts a date/time value into a human-readable string according to a specified format pattern.

**Technical Definition**
Formatting functions transform temporal values into `TEXT`/`VARCHAR` using format models. MySQL uses `DATE_FORMAT(date, format)`; PostgreSQL uses `TO_CHAR(timestamp, format)`; SQL Server uses `FORMAT(value, format)` (2012+) or `CONVERT(varchar, value, style)`. Format models consist of specifiers such as `%Y` (four-digit year), `%m` (two-digit month), `%d` (two-digit day), and `%H` (24-hour hour).

**Beginner-Friendly Explanation**
Date formatting turns a date like `2026-11-13` into a custom string like `13 November 2026` or `11/13/26`, depending on what you need.

#### Purposes

- To produce reports with locale-specific date formats.
- To generate file names or labels containing dates.
- To display dates in a user-friendly format.
- To prepare data for export to other systems.

#### Syntax Rules and Structure

**MySQL**
```
DATE_FORMAT(date, format)
```
**PostgreSQL**
```
TO_CHAR(timestamp, format)
```
**SQL Server**
```
FORMAT(value, format [, culture])
CONVERT(varchar, value, style)
```

**Syntax Rules**

1. MySQL format specifiers are prefixed with `%` (e.g., `%Y-%m-%d`).
2. PostgreSQL uses template patterns (e.g., `YYYY-MM-DD`).
3. SQL Server's `FORMAT` uses .NET format strings (e.g., `'yyyy-MM-dd'`); `CONVERT` uses style numbers (e.g., `112` for `yyyymmdd`).

**Constraints and Limitations**

- Format specifiers are not portable across RDBMS.
- `FORMAT` in SQL Server is slower than `CONVERT` for large datasets.

#### Annotated Code Examples

**Example 1: MySQL `DATE_FORMAT`**

```sql
SELECT DATE_FORMAT('2026-11-13', '%W, %M %d, %Y') AS formatted_date;
```

**Expected Output**

| formatted_date        |
|-----------------------|
| Friday, November 13, 2026 |

**Why This Output Occurs**: `%W` = weekday name, `%M` = month name, `%d` = day, `%Y` = year.

**Example 2: PostgreSQL `TO_CHAR`**

```sql
SELECT TO_CHAR(TIMESTAMP '2026-11-13 14:39:53', 'Day, DD Month YYYY') AS formatted;
```

**Expected Output**

| formatted               |
|-------------------------|
| Friday   , 13 November 2026 |

**Why This Output Occurs**: PostgreSQL's template patterns produce the formatted string.

**Example 3: SQL Server `FORMAT`**

```sql
SELECT FORMAT(GETDATE(), 'yyyy-MM-dd HH:mm:ss') AS formatted;
```

**Expected Output (illustrative)**

| formatted           |
|---------------------|
| 2026-11-13 14:39:53 |

#### Real-World Cases

- **Invoice headers**: Displaying dates in the customer's locale.
- **File naming**: `SELECT 'report_' || TO_CHAR(CURRENT_DATE, 'YYYYMMDD') || '.csv';`.
- **UI display**: Formatting timestamps for web interfaces.

---

### 5.2 Date Parsing

#### Definitions

**Core Definition**
Date parsing converts a string into a date/time value according to a specified format pattern.

**Technical Definition**
Parsing functions interpret a formatted string and return a temporal value. MySQL uses `STR_TO_DATE(str, format)`; PostgreSQL uses `TO_DATE(text, format)` and `TO_TIMESTAMP(text, format)`; SQL Server uses `CONVERT` / `PARSE` with style numbers or culture-aware parsing.

**Beginner-Friendly Explanation**
Date parsing takes a text like `'13/11/2026'` and turns it into a proper date value that the database can work with.

#### Purposes

- To import date data from external files.
- To convert user input into a date value.
- To standardise dates stored as strings.
- To parse legacy data with non-standard formats.

#### Syntax Rules and Structure

**MySQL**
```
STR_TO_DATE(str, format)
```
**PostgreSQL**
```
TO_DATE(text, format)
TO_TIMESTAMP(text, format)
```
**SQL Server**
```
CONVERT(date, string, style)
PARSE(string AS date USING culture)
```

**Syntax Rules**

1. The format string must match the input string exactly (including separators).
2. Unmatched or invalid input returns `NULL` (MySQL) or raises an error (PostgreSQL/SQL Server, depending on settings).
3. `STR_TO_DATE` returns a `DATE`, `TIME`, or `DATETIME` depending on the format components present.

**Constraints and Limitations**

- Format specifiers are vendor-specific.
- Ambiguous dates (e.g., `01/02/2026`) depend on the format mask or locale.

#### Annotated Code Examples

**Example 1: MySQL `STR_TO_DATE`**

```sql
SELECT STR_TO_DATE('07-Jul-2026', '%d-%b-%Y') AS parsed_date;
```

**Expected Output**

| parsed_date |
|-------------|
| 2026-07-07  |

**Why This Output Occurs**: The format mask `%d-%b-%Y` matches the input string and produces a `DATE`.

**Example 2: PostgreSQL `TO_DATE`**

```sql
SELECT TO_DATE('13/11/2026', 'DD/MM/YYYY') AS parsed_date;
```

**Expected Output**

| parsed_date |
|-------------|
| 2026-11-13  |

#### Real-World Cases

- **CSV import**: Parsing date strings from uploaded files.
- **API integration**: Converting date strings from external APIs.
- **Data migration**: Converting legacy date formats to standard types.

---

## 6. Calendar & Business Logic

### 6.1 Weekday Calculations

#### Definitions

**Core Definition**
Weekday calculations determine the day of the week for a given date, or count the number of weekdays (Monday–Friday) within a date range.

**Technical Definition**
RDBMS provide functions to extract the day of the week: PostgreSQL's `EXTRACT(DOW ...)` returns 0 (Sunday) to 6 (Saturday); MySQL's `DAYOFWEEK()` returns 1 (Sunday) to 7 (Saturday), and `WEEKDAY()` returns 0 (Monday) to 6 (Sunday); SQL Server's `DATEPART(weekday, date)` depends on the `DATEFIRST` setting. Business day calculations require enumerating dates and filtering out weekends and holidays.

**Beginner-Friendly Explanation**
Weekday calculations tell you what day of the week a date falls on (e.g., "is this a Saturday?") and let you count only the working days between two dates.

#### Purposes

- To determine whether a date is a weekend.
- To count business days between two dates.
- To schedule recurring events on specific weekdays.
- To analyse data by day of the week.

#### Syntax Rules and Structure

**PostgreSQL**
```
EXTRACT(DOW FROM date) → 0–6 (0 = Sunday)
```
**MySQL**
```
DAYOFWEEK(date) → 1–7 (1 = Sunday)
WEEKDAY(date) → 0–6 (0 = Monday)
```
**SQL Server**
```
DATEPART(weekday, date) → 1–7 (depends on DATEFIRST)
DATENAME(weekday, date) → string name
```

**Syntax Rules**

1. `DOW` semantics differ: PostgreSQL 0 = Sunday, MySQL `DAYOFWEEK` 1 = Sunday, MySQL `WEEKDAY` 0 = Monday.
2. SQL Server's `DATEPART(weekday)` depends on `SET DATEFIRST`.
3. `DATENAME` returns the weekday name as a string.

**Constraints and Limitations**

- Non-portable return values require careful handling.
- Business day calculations must account for holidays in addition to weekends.

#### Annotated Code Examples

**Example 1: Checking for Weekend (PostgreSQL)**

```sql
SELECT EXTRACT(DOW FROM DATE '2026-11-14') IN (0, 6) AS is_weekend;
```

**Expected Output**

| is_weekend |
|------------|
| t          |

**Why This Output Occurs**: 2026-11-14 is a Saturday; `DOW` returns 6, which is in `(0, 6)`.

**Example 2: MySQL `DAYOFWEEK`**

```sql
SELECT DAYOFWEEK('2026-11-13') AS day_num;
```

**Expected Output**

| day_num |
|---------|
| 6       |

**Why This Output Occurs**: Friday is 6 in MySQL's `DAYOFWEEK` (1 = Sunday).

#### Real-World Cases

- **Weekend surcharge**: Applying different pricing on weekends.
- **Staff scheduling**: Ensuring sufficient weekend coverage.
- **Marketing analysis**: Comparing weekday vs. weekend sales.

---

### 6.2 Business Day Calculations

#### Definitions

**Core Definition**
Business day calculations count the number of working days (typically Monday–Friday, excluding holidays) between two dates.

**Technical Definition**
No RDBMS provides a native business-day function. PostgreSQL uses `GENERATE_SERIES` to enumerate dates and filter out weekends and holidays. MySQL 8.0+ uses recursive CTEs for the same purpose. SQL Server commonly uses a calendar table with an `is_weekday` flag and a `holiday` table. The calculation must account for the `DATEFIRST` setting in SQL Server and the different weekday numbering conventions across systems.

**Beginner-Friendly Explanation**
Business day calculations tell you how many actual working days there are between two dates—ignoring weekends and holidays.

#### Purposes

- To calculate SLA deadlines in business days.
- To compute interest accrual on business-day conventions.
- To determine project durations excluding non-working days.
- To schedule deliveries or appointments on working days.

#### Syntax Rules and Structure

**PostgreSQL Approach**
```
SELECT COUNT(*)
FROM GENERATE_SERIES(start_date, end_date, '1 day') AS d
WHERE EXTRACT(DOW FROM d) NOT IN (0, 6)
  AND d NOT IN (SELECT holiday_date FROM holidays);
```

**MySQL 8.0+ Approach**
```
WITH RECURSIVE dates AS (
  SELECT start_date AS d
  UNION ALL
  SELECT d + INTERVAL 1 DAY FROM dates WHERE d < end_date
)
SELECT COUNT(*) FROM dates
WHERE WEEKDAY(d) < 5
  AND d NOT IN (SELECT holiday_date FROM holidays);
```

**SQL Server Approach**
```
SELECT COUNT(*)
FROM calendar c
WHERE c.date BETWEEN @start AND @end
  AND c.is_weekday = 1
  AND c.is_holiday = 0;
```

**Constraints and Limitations**

- Requires a holiday table or calendar dimension for accuracy.
- Recursive CTEs may be slow for very large ranges.
- SQL Server's `DATEFIRST` setting affects `DATEPART(weekday)`.

#### Annotated Code Examples

**Example 1: Business Days in PostgreSQL**

```sql
SELECT COUNT(*) AS business_days
FROM GENERATE_SERIES('2026-11-01'::date, '2026-11-30'::date, '1 day') AS d
WHERE EXTRACT(DOW FROM d) NOT IN (0, 6);
```

**Expected Output (illustrative)**

| business_days |
|---------------|
| 21            |

**Why This Output Occurs**: November 2026 has 30 days; weekends are excluded, leaving 21 weekdays (assuming no holidays).

#### Real-World Cases

- **Loan interest**: Calculating interest accrual on business days only.
- **Project management**: Estimating delivery dates excluding weekends.
- **Customer support**: Measuring response times in business hours.

---

### 6.3 Fiscal Calendars

#### Definitions

**Core Definition**
A fiscal calendar defines an organisation's financial year, which may not align with the Gregorian calendar (e.g., starting in July or October).

**Technical Definition**
SQL does not provide built-in fiscal calendar functions in PostgreSQL, MySQL, or SQL Server. Fiscal calendars are implemented by defining the fiscal year start month/day and using date arithmetic to map calendar dates to fiscal periods. For example, if the fiscal year starts on July 1, the fiscal year for a date `d` is `YEAR(d) + (MONTH(d) >= 7 ? 0 : -1)`. Oracle provides `FISCAL_YEAR` and related functions, but these are Oracle-specific.

**Beginner-Friendly Explanation**
A fiscal calendar lets a company define its own financial year—like starting the year in July instead of January—so that reports match how the business actually operates.

#### Purposes

- To align financial reporting with the organisation's fiscal year.
- To calculate fiscal quarter and fiscal month for transactions.
- To compare performance across fiscal periods.
- To support budgeting and forecasting cycles.

#### Syntax Rules and Structure

Fiscal calendar logic is application-defined and typically implemented via:

1. **CASE expressions**:
   ```
   CASE WHEN MONTH(date) >= 7 THEN YEAR(date) ELSE YEAR(date) - 1 END AS fiscal_year
   ```
2. **Date arithmetic**:
   ```
   DATE_TRUNC('year', date - INTERVAL '6 months') + INTERVAL '6 months'
   ```
3. **Calendar dimension table** with pre-computed fiscal periods.

**Constraints and Limitations**

- No standard SQL fiscal functions in PostgreSQL, MySQL, or SQL Server.
- Fiscal year start varies by organisation and country.
- 4-4-5 and 4-5-4 retail calendars require more complex modelling.

#### Annotated Code Examples

**Example 1: Fiscal Year with July Start (PostgreSQL)**

```sql
SELECT
  date,
  CASE WHEN EXTRACT(MONTH FROM date) >= 7
       THEN EXTRACT(YEAR FROM date)
       ELSE EXTRACT(YEAR FROM date) - 1
  END AS fiscal_year
FROM transactions;
```

**Expected Output (illustrative)**

| date       | fiscal_year |
|------------|-------------|
| 2026-08-15 | 2026        |
| 2026-03-10 | 2025        |

**Why This Output Occurs**: August is in the fiscal year starting July 2026; March 2026 belongs to the fiscal year that started July 2025.

#### Real-World Cases

- **Retail**: 4-5-4 fiscal calendars for merchandising.
- **Government**: Fiscal years starting October 1.
- **Non-profits**: Fiscal years aligned with grant cycles.

---

## References

1. **PostgreSQL 16 Documentation — Date/Time Functions and Operators** — https://www.postgresql.org/docs/16/functions-datetime.html
2. **PostgreSQL 16 Documentation — Date/Time Types** — https://www.postgresql.org/docs/16/datatype-datetime.html
3. **PostgreSQL 16 Documentation — Pattern Matching (TO_CHAR, TO_DATE)** — https://www.postgresql.org/docs/16/functions-formatting.html
4. **MySQL 8.0 Reference Manual — Date and Time Functions** — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
5. **MySQL 8.0 Reference Manual — Date and Time Data Types** — https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html
6. **Microsoft SQL Server — Date and Time Data Types and Functions (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql
7. **Microsoft SQL Server — AT TIME ZONE (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/queries/at-time-zone-transact-sql
8. **Microsoft SQL Server — DATEADD (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/dateadd-transact-sql
9. **Microsoft SQL Server — DATEDIFF (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/datediff-transact-sql
10. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL date and time functions across PostgreSQL, MySQL, and SQL Server, with standard SQL notes where applicable. Always consult your specific RDBMS documentation for version-specific behaviour and extensions, particularly for time zone data, fiscal calendar modelling, and business day calculations.