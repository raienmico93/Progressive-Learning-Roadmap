# SQL Date and Time Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL date and time functions are built-in scalar functions that manipulate temporal data types (DATE, TIME, TIMESTAMP, INTERVAL), enabling retrieval of the current date/time, extraction of date parts, date arithmetic, formatting, parsing, time-zone conversion, and chronological construction.

**Technical Definition:** Per the ANSI/ISO SQL Standard (ISO/IEC 9075), temporal functions operate on the four core datetime data types — DATE (date only), TIME (time only), TIMESTAMP (date and time combined), and INTERVAL (the interval between two datetime values). These functions are evaluated once per query at the start of query execution and return values in the connection's current time zone. Core standard functions include CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP, EXTRACT, and INTERVAL arithmetic. Vendor implementations extend the standard with functions such as NOW(), GETDATE(), DATE_ADD(), DATE_TRUNC(), and STR_TO_DATE().

**Beginner-Friendly Explanation:** Databases store dates and times in special formats, but you rarely want to use them exactly as stored. Date and time functions are like the clock, calendar, and calculator tools built into your database. They let you ask "what is today's date?", "what month was this order placed in?", "add 30 days to this deadline", or "convert this timestamp to another time zone." Without them, you would have to do all this math manually.

### Key Characteristics

- **Standardized core, vendor-extended periphery:** CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP, and EXTRACT are ANSI standard; functions like NOW(), GETDATE(), and DATE_TRUNC are vendor-specific.
- **Time-zone aware:** Functions like FROM_UNIXTIME() and CURRENT_TIMESTAMP() return values in the connection's current time zone.
- **Implicit type promotion:** Adding a time part to a DATE value automatically converts the result to a TIMESTAMP (e.g., adding 1 HOUR to a DATE produces a datetime).
- **Calendar-aware arithmetic:** Month and year arithmetic adjusts for variable month lengths (e.g., adding 1 month to January 30 yields February 28).
- **Precision control:** Many functions accept a fractional seconds precision (fsp) argument, such as CURRENT_TIMESTAMP(6) for microsecond precision.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of DATE, TIME, TIMESTAMP, and INTERVAL data types
- Familiarity with string formatting and parsing
- Basic knowledge of time zones and UTC

### Related Programming Areas

- **Business Intelligence and Reporting:** Time-series aggregation using DATE_TRUNC for monthly, quarterly, and yearly trends.
- **Financial Systems:** Computing maturity dates, interest accrual periods, and settlement windows.
- **Log Analysis:** Filtering and grouping events by timestamp intervals.
- **Geospatial Applications:** Converting coordinates and computing time-zone-adjusted positions.
- **Application Development:** Displaying dates in user-local formats and handling time-zone conversions.
- **Data Engineering:** ETL pipelines rely heavily on date parsing and Unix epoch conversion.

---

## Core Concepts / Key Features

### 1. Current System Inputs

#### 1.1 Current Date: CURRENT_DATE / CURDATE()

**Core Definitions:**

- **Core Definition:** CURRENT_DATE returns the current system date without the time component.
- **Technical Definition:** CURRENT_DATE is an ANSI-standard keyword that returns the current system date. In MySQL, CURRENT_DATE and CURRENT_DATE() are synonyms for CURDATE().
- **Beginner-Friendly Explanation:** CURRENT_DATE gives you today's date — just the year, month, and day, no clock time.

**Purposes:**

- To filter records for "today" or "this date"
- To timestamp records with the current date only
- To compute date-based calculations (e.g., age, tenure)

**Syntax Structures and Rules:**

```sql
CURRENT_DATE
CURDATE()      -- MySQL synonym
```

**Annotated Code Examples:**

```sql
-- Standard SQL
SELECT CURRENT_DATE AS today;

-- MySQL
SELECT CURDATE() AS today;
```

**Expected Output:**

| today |
|-------|
| 2026-09-20 |

**Real-World Cases:**

- **Daily reports:** `WHERE order_date = CURRENT_DATE`.
- **Attendance systems:** Recording the date an employee clocks in.
- **Audit trails:** Storing the date a record was created.

**References:**

- Microsoft Learn: CURRENT_DATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/current-timestamp-transact-sql
- MySQL: Date and Time Functions - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 1.2 Current Time: CURRENT_TIME / CURTIME()

**Core Definitions:**

- **Core Definition:** CURRENT_TIME returns the current system time without the date component.
- **Technical Definition:** CURRENT_TIME is an ANSI-standard keyword that returns the current system time. In MySQL, CURRENT_TIME and CURRENT_TIME() are synonyms for CURTIME().
- **Beginner-Friendly Explanation:** CURRENT_TIME gives you the clock time right now — just hours, minutes, and seconds.

**Annotated Code Examples:**

```sql
SELECT CURRENT_TIME AS now_time;
-- Returns: 14:35:22
```

**Real-World Cases:**

- **Shift scheduling:** Determining which shift a worker is on.
- **Real-time systems:** Logging the exact time of an event.

**References:**

- MySQL: Date and Time Functions - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 1.3 Current Timestamp: CURRENT_TIMESTAMP, NOW(), GETDATE()

**Core Definitions:**

- **Core Definition:** CURRENT_TIMESTAMP returns the full unified date and clock time.
- **Technical Definition:** CURRENT_TIMESTAMP is ANSI-standard. NOW() is a MySQL synonym. GETDATE() is SQL Server's equivalent. These functions return the current date and time in the connection's current time zone.
- **Beginner-Friendly Explanation:** CURRENT_TIMESTAMP gives you the complete picture — both the date and the exact time.

**Purposes:**

- To timestamp records with the exact moment of creation or modification
- To measure elapsed time between events
- To filter records by precise datetime ranges

**Annotated Code Examples:**

```sql
-- Standard SQL
SELECT CURRENT_TIMESTAMP AS right_now;

-- MySQL
SELECT NOW() AS right_now;

-- SQL Server
SELECT GETDATE() AS right_now;
```

**Expected Output:**

| right_now |
|-----------|
| 2026-09-20 14:35:22 |

**Real-World Cases:**

- **Audit columns:** `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`.
- **Session tracking:** Recording when a user logged in.
- **ETL pipelines:** Marking the start and end of batch jobs.

**References:**

- Microsoft Learn: GETDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/getdate-transact-sql
- MySQL: Date and Time Functions - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 1.4 UTC/GMT Tracking: SYSUTCDATETIME() / UTC_TIMESTAMP()

**Core Definitions:**

- **Core Definition:** SYSUTCDATETIME (SQL Server) or UTC_TIMESTAMP (MySQL) returns the current date and time in Coordinated Universal Time (UTC), bypassing local server offset biases.
- **Technical Definition:** SYSUTCDATETIME returns a datetime2 value containing the date and time of the computer running SQL Server, expressed as UTC. MySQL's UTC_TIMESTAMP returns the current UTC date and time.
- **Beginner-Friendly Explanation:** These functions give you the "world clock" time — the same time regardless of which time zone the server is in.

**Purposes:**

- To store timestamps in a time-zone-neutral format
- To avoid daylight saving time complications
- To synchronize data across distributed systems

**Annotated Code Examples:**

```sql
-- SQL Server
SELECT SYSUTCDATETIME() AS utc_now;

-- MySQL
SELECT UTC_TIMESTAMP() AS utc_now;
```

**Expected Output:**

| utc_now |
|---------|
| 2026-09-20 06:35:22 |

**Real-World Cases:**

- **Global applications:** Storing all timestamps in UTC and converting to local time at display.
- **Distributed databases:** Ensuring consistent ordering across servers in different regions.
- **Log aggregation:** Normalizing timestamps from multiple time zones.

**References:**

- Microsoft Learn: SYSUTCDATETIME (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sysutcdatetime-transact-sql


### 2. Extraction & Parsing

#### 2.1 Part Extraction: EXTRACT(YEAR FROM date) and Shortcut Functions

**Core Definitions:**

- **Core Definition:** EXTRACT retrieves a specific component (year, month, day, hour, minute, second) from a datetime or interval value.
- **Technical Definition:** EXTRACT(field FROM source) is ANSI-standard. Valid fields include YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, TIMEZONE_HOUR, and TIMEZONE_MINUTE. Oracle requires the source to be DATE, TIMESTAMP, or INTERVAL depending on the field requested.
- **Beginner-Friendly Explanation:** EXTRACT is like a pair of tweezers — it pulls out just the part of the date you need.

**Purposes:**

- To extract individual date components for grouping or filtering
- To build custom date formats
- To perform calculations based on specific date parts

**Syntax Structures and Rules:**

```sql
-- ANSI standard
EXTRACT(field FROM source)

-- Vendor shortcuts
YEAR(date), MONTH(date), DAY(date), HOUR(time), MINUTE(time), SECOND(time)
```

**Annotated Code Examples:**

```sql
SELECT EXTRACT(YEAR FROM DATE '2024-03-15') AS yr,
       EXTRACT(MONTH FROM DATE '2024-03-15') AS mo,
       EXTRACT(DAY FROM DATE '2024-03-15') AS dy,
       YEAR('2024-03-15') AS yr_shortcut;
```

**Expected Output:**

| yr | mo | dy | yr_shortcut |
|----|----|----|-------------|
| 2024 | 3 | 15 | 2024 |

**Real-World Cases:**

- **Seasonal analysis:** Grouping sales by quarter.
- **Birthday reminders:** Extracting month and day from birth dates.
- **Time tracking:** Extracting hours from login timestamps.

**References:**

- Oracle: EXTRACT (datetime) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/EXTRACT-datetime.html
- PostgreSQL: EXTRACT - https://www.postgresql.org/docs/current/functions-datetime.html


#### 2.2 Day-of-Week & Metadata: WEEK(), QUARTER(), DAYOFWEEK()

**Core Definitions:**

- **Core Definition:** These functions isolate structural calendar metrics such as the week number, quarter, and day-of-week index.
- **Technical Definition:** QUARTER(date) returns 1–4. WEEK(date) returns the week number (0–52 or 1–53 depending on the mode). DAYOFWEEK(date) returns the weekday index (1=Sunday, 2=Monday, …, 7=Saturday in MySQL, following ODBC standards).
- **Beginner-Friendly Explanation:** These functions tell you things like "this date is in Q2" or "this date falls on a Tuesday."

**Annotated Code Examples:**

```sql
SELECT QUARTER('2024-05-15') AS qtr,
       WEEK('2024-05-15') AS wk,
       DAYOFWEEK('2024-05-15') AS dow,
       DAYNAME('2024-05-15') AS day_name;
```

**Expected Output:**

| qtr | wk | dow | day_name |
|-----|----|----|----------|
| 2 | 20 | 4 | Wednesday |

**Real-World Cases:**

- **Retail analytics:** Comparing sales across quarters.
- **Scheduling:** Determining which weekday has the highest traffic.
- **Payroll:** Computing week numbers for pay periods.

**References:**

- MySQL: Date and Time Functions (DAYOFWEEK, WEEK, QUARTER) - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


### 3. Date Arithmetic & Intervals

#### 3.1 Addition & Subtraction: DATE_ADD() / DATE_SUB()

**Core Definitions:**

- **Core Definition:** DATE_ADD adds a specified time interval to a date; DATE_SUB subtracts one.
- **Technical Definition:** MySQL's DATE_ADD(date, INTERVAL expr unit) and DATE_SUB(date, INTERVAL expr unit) perform calendar-aware arithmetic. The INTERVAL keyword is required. You can also use the + and - operators with INTERVAL.
- **Beginner-Friendly Explanation:** DATE_ADD is like fast-forwarding a date; DATE_SUB is like rewinding it.

**Purposes:**

- To compute future or past dates (due dates, expiration dates)
- To implement date-based reminders and alerts
- To shift dates for reporting periods

**Syntax Structures and Rules:**

```sql
DATE_ADD(date, INTERVAL expr unit)
DATE_SUB(date, INTERVAL expr unit)
date + INTERVAL expr unit
date - INTERVAL expr unit
```

**Annotated Code Examples:**

```sql
SELECT DATE_ADD('2024-03-15', INTERVAL 1 MONTH) AS next_month,
       DATE_SUB('2024-03-15', INTERVAL 7 DAY) AS last_week,
       '2024-03-15' + INTERVAL 1 YEAR AS next_year;
```

**Expected Output:**

| next_month | last_week | next_year |
|------------|-----------|-----------|
| 2024-04-15 | 2024-03-08 | 2025-03-15 |

**Important Note:** Month-end adjustments occur automatically. Adding 1 month to January 30 yields February 28 (or 29 in a leap year).

**Real-World Cases:**

- **Subscriptions:** Computing renewal dates (`start_date + INTERVAL 1 YEAR`).
- **Project management:** Setting task deadlines (`start_date + INTERVAL 14 DAY`).
- **Warranty tracking:** Determining expiration dates.

**References:**

- MySQL: Date and Time Functions (DATE_ADD, DATE_SUB) - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 3.2 Explicit Intervals

**Core Definitions:**

- **Core Definition:** INTERVAL declares a strict, human-readable duration that can be added to or subtracted from a datetime value.
- **Technical Definition:** The INTERVAL keyword is followed by a numeric expression and a unit (DAY, MONTH, YEAR, HOUR, MINUTE, SECOND, etc.). MySQL supports compound units like DAY_SECOND and YEAR_MONTH.
- **Beginner-Friendly Explanation:** INTERVAL is how you tell the database "add exactly this much time" in plain language.

**Annotated Code Examples:**

```sql
SELECT DATE_ADD('2024-01-01', INTERVAL 3 MONTH) AS plus_3_months,
       DATE_ADD('2024-01-01 12:00:00', INTERVAL '1 2' DAY_HOUR) AS plus_1_day_2_hours;
```

**Expected Output:**

| plus_3_months | plus_1_day_2_hours |
|---------------|-------------------|
| 2024-04-01 | 2024-01-02 14:00:00 |

**Real-World Cases:**

- **Billing cycles:** `invoice_date + INTERVAL 30 DAY`.
- **Grace periods:** `due_date + INTERVAL 5 DAY`.
- **Retention policies:** `created_at + INTERVAL 7 YEAR`.

**References:**

- MySQL: Date and Time Functions (INTERVAL) - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 3.3 Date Differences: DATEDIFF() / TIMEDIFF()

**Core Definitions:**

- **Core Definition:** DATEDIFF calculates the number of date-part boundaries crossed between two dates; TIMEDIFF returns the time difference between two time values.
- **Technical Definition:** In SQL Server and MySQL, DATEDIFF(datepart, startdate, enddate) returns the count of the specified datepart boundaries crossed between the two dates. It counts boundaries, not whole units — for example, DATEDIFF(day, '2036-03-01', '2036-02-28') returns -2.
- **Beginner-Friendly Explanation:** DATEDIFF tells you "how many days/months/years are between these two dates?"

**Purposes:**

- To compute elapsed durations (age, tenure, time since last event)
- To measure performance or processing time
- To implement date-based business rules

**Annotated Code Examples:**

```sql
-- SQL Server / MySQL
SELECT DATEDIFF(day, '2024-01-01', '2024-03-15') AS days_diff,
       DATEDIFF(month, '2024-01-01', '2024-03-15') AS months_diff,
       DATEDIFF(year, '2020-06-15', '2024-06-15') AS years_diff;
```

**Expected Output:**

| days_diff | months_diff | years_diff |
|-----------|-------------|------------|
| 74 | 2 | 4 |

**Real-World Cases:**

- **HR:** Computing employee tenure.
- **Finance:** Calculating days past due.
- **E-commerce:** Measuring time between order and shipment.

**References:**

- Microsoft Learn: DATEDIFF (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datediff-transact-sql


### 4. Truncation & Rounding

#### 4.1 Timeline Truncation: DATE_TRUNC() / DATETRUNC()

**Core Definitions:**

- **Core Definition:** DATE_TRUNC snaps a timestamp down to the start of its parent container (e.g., the first day of the month, the first hour of the day).
- **Technical Definition:** DATE_TRUNC('datepart', timestamp) truncates a timestamp to the precision of the specified datepart. Valid dateparts include microsecond, millisecond, second, minute, hour, day, week, month, quarter, year, decade, century, and millennium. SQL Server 2022+ provides DATETRUNC with the same behavior.
- **Beginner-Friendly Explanation:** DATE_TRUNC is like "flattening" a timestamp to the beginning of its period. All days in January become January 1st.

**Purposes:**

- To group data by month, quarter, or year for trend analysis
- To normalize timestamps for comparison
- To create time buckets for aggregation

**Annotated Code Examples:**

```sql
SELECT DATE_TRUNC('month', TIMESTAMP '2024-04-30 14:35:22') AS month_start,
       DATE_TRUNC('quarter', TIMESTAMP '2024-04-30 14:35:22') AS quarter_start,
       DATE_TRUNC('year', TIMESTAMP '2024-04-30 14:35:22') AS year_start,
       DATE_TRUNC('week', TIMESTAMP '2024-04-30 14:35:22') AS week_start;
```

**Expected Output:**

| month_start | quarter_start | year_start | week_start |
|-------------|---------------|------------|------------|
| 2024-04-01 | 2024-04-01 | 2024-01-01 | 2024-04-29 |

**Explanation:** DATE_TRUNC('month', ...) collapses all days and hours to the first day of the month at midnight.

**Real-World Cases:**

- **Sales dashboards:** Monthly revenue trends using `DATE_TRUNC('month', sale_date)`.
- **User analytics:** Weekly active users using `DATE_TRUNC('week', login_time)`.
- **Financial reporting:** Quarterly earnings using `DATE_TRUNC('quarter', transaction_date)`.

**References:**

- Amazon Redshift: DATE_TRUNC - https://docs.aws.amazon.com/redshift/latest/dg/r_DATE_TRUNC.html
- Microsoft Learn: DATETRUNC (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datetrunc-transact-sql


### 5. Conversions, Formats, & Epochs

#### 5.1 String Conversion: DATE_FORMAT() / TO_CHAR() / FORMAT()

**Core Definitions:**

- **Core Definition:** These functions format a date object into a custom string pattern.
- **Technical Definition:** MySQL uses DATE_FORMAT(date, format). Oracle and PostgreSQL use TO_CHAR(date, format). SQL Server uses FORMAT(date, format) (2012+) or CONVERT with style codes. Format strings differ significantly between databases.
- **Beginner-Friendly Explanation:** DATE_FORMAT turns a date into text in whatever pattern you choose — "2024-03-15", "15/03/2024", or "March 15, 2024".

**Annotated Code Examples:**

```sql
-- MySQL
SELECT DATE_FORMAT('2024-03-15', '%Y-%m-%d') AS iso_format,
       DATE_FORMAT('2024-03-15', '%M %d, %Y') AS long_format;

-- Oracle / PostgreSQL
SELECT TO_CHAR(DATE '2024-03-15', 'YYYY-MM-DD') AS iso_format,
       TO_CHAR(DATE '2024-03-15', 'Month DD, YYYY') AS long_format;
```

**Expected Output:**

| iso_format | long_format |
|------------|-------------|
| 2024-03-15 | March 15, 2024 |

**Real-World Cases:**

- **Reports:** Formatting dates for PDF or Excel export.
- **User interfaces:** Displaying dates in locale-specific formats.
- **APIs:** Returning dates in ISO 8601 format.

**References:**

- MySQL: DATE_FORMAT - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- Oracle: TO_CHAR (datetime) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_CHAR-datetime.html


#### 5.2 String Parsing: STR_TO_DATE() / TO_DATE()

**Core Definitions:**

- **Core Definition:** STR_TO_DATE (MySQL) or TO_DATE (Oracle, PostgreSQL) converts unstructured text into a structured date data type.
- **Technical Definition:** STR_TO_DATE(str, format) returns a DATETIME value if the format contains both date and time parts, or a DATE or TIME value if the string contains only date or time parts. TO_DATE(char, format) is the Oracle/PostgreSQL equivalent.
- **Beginner-Friendly Explanation:** STR_TO_DATE is the reverse of DATE_FORMAT — it takes text and turns it into a date the database can work with.

**Annotated Code Examples:**

```sql
-- MySQL
SELECT STR_TO_DATE('15,5,2013', '%d,%m,%Y') AS parsed_date;

-- Oracle / PostgreSQL
SELECT TO_DATE('2024-03-15', 'YYYY-MM-DD') AS parsed_date;
```

**Expected Output:**

| parsed_date |
|-------------|
| 2013-05-15 |

**Real-World Cases:**

- **CSV imports:** Parsing date columns from flat files.
- **API integration:** Converting date strings from external systems.
- **Data cleaning:** Normalizing inconsistent date formats.

**References:**

- MySQL: STR_TO_DATE - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- Oracle: TO_DATE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_DATE.html


#### 5.3 Unix Epoch Time: FROM_UNIXTIME() / UNIX_TIMESTAMP()

**Core Definitions:**

- **Core Definition:** FROM_UNIXTIME converts a Unix timestamp (seconds since January 1, 1970) into a readable date; UNIX_TIMESTAMP does the reverse.
- **Technical Definition:** FROM_UNIXTIME(unix_timestamp) returns a representation of the Unix timestamp as a value in 'YYYY-MM-DD HH:MM:SS' format. FROM_UNIXTIME(unix_timestamp, format) returns a formatted string. These functions return values in the connection's current time zone.
- **Beginner-Friendly Explanation:** Unix time is a number like 1637258854 that represents a moment in time. FROM_UNIXTIME turns that number into a date like "2021-11-18 10:47:34."

**Annotated Code Examples:**

```sql
SELECT FROM_UNIXTIME(1637258854) AS readable_date,
       UNIX_TIMESTAMP('2021-11-18 10:47:34') AS epoch_seconds;
```

**Expected Output:**

| readable_date | epoch_seconds |
|---------------|---------------|
| 2021-11-18 10:47:34 | 1637258854 |

**Real-World Cases:**

- **Log analysis:** Converting Unix timestamps in log files.
- **API responses:** Many APIs return Unix epoch time.
- **Data migration:** Converting between epoch-based and native datetime systems.

**References:**

- MySQL: FROM_UNIXTIME - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


### 6. Time-Zone Mechanics

#### 6.1 Offset Transformations: AT TIME ZONE / CONVERT_TZ()

**Core Definitions:**

- **Core Definition:** AT TIME ZONE (PostgreSQL, SQL Server) and CONVERT_TZ() (MySQL) translate a timestamp from one time-zone region to another safely.
- **Technical Definition:** The AT TIME ZONE expression converts a datetime value to a different time zone while managing daylight saving adjustments. MySQL's CONVERT_TZ(dt, from_tz, to_tz) requires the time-zone tables to be loaded for named zones.
- **Beginner-Friendly Explanation:** These functions let you say "what time was it in Tokyo when this event happened in New York?"

**Purposes:**

- To display timestamps in the user's local time zone
- To convert between time zones for scheduling and coordination
- To handle daylight saving time transitions automatically

**Annotated Code Examples:**

```sql
-- PostgreSQL
SELECT TIMESTAMP '2024-03-15 12:00:00' AT TIME ZONE 'UTC' AT TIME ZONE 'America/New_York' AS ny_time;

-- MySQL
SELECT CONVERT_TZ('2024-03-15 12:00:00', 'UTC', 'America/New_York') AS ny_time;

-- SQL Server
SELECT '2024-03-15 12:00:00' AT TIME ZONE 'UTC' AT TIME ZONE 'Eastern Standard Time' AS ny_time;
```

**Expected Output:**

| ny_time |
|---------|
| 2024-03-15 08:00:00 |

**Real-World Cases:**

- **Global scheduling:** Converting meeting times across time zones.
- **E-commerce:** Displaying order times in the customer's local time.
- **Distributed systems:** Normalizing timestamps from servers in different regions.

**References:**

- PostgreSQL: AT TIME ZONE - https://www.postgresql.org/docs/current/functions-datetime.html
- MySQL: CONVERT_TZ - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


### 7. Chronological Edge Cases & Building

#### 7.1 Date Construction: MAKEDATE() / DATEFROMPARTS()

**Core Definitions:**

- **Core Definition:** These functions build a valid date object from separate raw numbers.
- **Technical Definition:** MySQL's MAKEDATE(year, dayofyear) creates a date from a year and a day-of-year value. SQL Server's DATEFROMPARTS(year, month, day) creates a date from year, month, and day integers. DATEFROMPARTS raises an error for invalid arguments and returns NULL if any argument is NULL.
- **Beginner-Friendly Explanation:** If you have the year, month, and day stored separately, DATEFROMPARTS assembles them into a proper date.

**Annotated Code Examples:**

```sql
-- SQL Server
SELECT DATEFROMPARTS(2024, 3, 15) AS constructed_date;

-- MySQL
SELECT MAKEDATE(2024, 75) AS constructed_date;
-- 75th day of 2024 = March 15
```

**Expected Output:**

| constructed_date |
|------------------|
| 2024-03-15 |

**Real-World Cases:**

- **Data migration:** Building dates from separate year/month/day columns.
- **Report generation:** Constructing date ranges dynamically.
- **ETL:** Assembling dates from normalized source data.

**References:**

- Microsoft Learn: DATEFROMPARTS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datefromparts-transact-sql
- MySQL: MAKEDATE - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


#### 7.2 Month-End Traps: LAST_DAY(date)

**Core Definitions:**

- **Core Definition:** LAST_DAY returns the date of the last day of the month that contains the given date.
- **Technical Definition:** LAST_DAY(date) returns the date of the last day of the month that contains date. The return type is always DATE, regardless of the input data type. The last day of the month is defined by the session parameter NLS_CALENDAR (Oracle).
- **Beginner-Friendly Explanation:** LAST_DAY tells you "what is the final date of this month?" — it automatically handles leap years and months of different lengths.

**Purposes:**

- To find the end of a month for billing, reporting, or scheduling
- To avoid hardcoding month lengths (28, 29, 30, 31)
- To compute month-end balances and accruals

**Annotated Code Examples:**

```sql
-- Oracle / MySQL
SELECT LAST_DAY('2024-02-15') AS end_of_feb,
       LAST_DAY('2024-04-15') AS end_of_april,
       LAST_DAY('2023-02-15') AS end_of_feb_2023;
```

**Expected Output:**

| end_of_feb | end_of_april | end_of_feb_2023 |
|------------|--------------|-----------------|
| 2024-02-29 | 2024-04-30 | 2023-02-28 |

**Explanation:** 2024 is a leap year, so LAST_DAY('2024-02-15') returns February 29. 2023 is not, so it returns February 28.

**Real-World Cases:**

- **Finance:** Computing month-end balances and interest accruals.
- **Billing:** Generating invoices due on the last day of the month.
- **Reporting:** Setting the end date for monthly reports.

**References:**

- Oracle: LAST_DAY - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LAST_DAY.html
- MySQL: LAST_DAY - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html


## Summary Tables

### Current Date/Time Functions Across Databases

| Function | ANSI Standard | PostgreSQL | MySQL | SQL Server | Oracle |
|----------|--------------|------------|-------|------------|--------|
| Current Date | CURRENT_DATE | CURRENT_DATE | CURDATE() | CAST(GETDATE() AS DATE) | CURRENT_DATE |
| Current Time | CURRENT_TIME | CURRENT_TIME | CURTIME() | CAST(GETDATE() AS TIME) | CURRENT_TIME |
| Current Timestamp | CURRENT_TIMESTAMP | CURRENT_TIMESTAMP | NOW() | GETDATE() | CURRENT_TIMESTAMP |
| UTC Timestamp | — | now() AT TIME ZONE 'UTC' | UTC_TIMESTAMP() | SYSUTCDATETIME() | SYS_EXTRACT_UTC(SYSTIMESTAMP) |

### Date Arithmetic Comparison

| Operation | MySQL | PostgreSQL | SQL Server | Oracle |
|-----------|-------|------------|------------|--------|
| Add days | DATE_ADD(d, INTERVAL n DAY) | d + n | DATEADD(day, n, d) | d + n |
| Add months | DATE_ADD(d, INTERVAL n MONTH) | d + INTERVAL 'n months' | DATEADD(month, n, d) | ADD_MONTHS(d, n) |
| Subtract | DATE_SUB(d, INTERVAL n unit) | d - INTERVAL 'n unit' | DATEADD(-n, ...) | d - n |
| Difference (days) | DATEDIFF(d1, d2) | d1 - d2 | DATEDIFF(day, d1, d2) | d1 - d2 |

### Date Formatting Comparison

| Database | Date → String | String → Date |
|----------|--------------|---------------|
| MySQL | DATE_FORMAT(date, format) | STR_TO_DATE(str, format) |
| Oracle | TO_CHAR(date, format) | TO_DATE(str, format) |
| PostgreSQL | TO_CHAR(date, format) | TO_DATE(str, format) |
| SQL Server | FORMAT(date, format) or CONVERT | CONVERT or PARSE |

---

## References

- Microsoft Learn: GETDATE (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/getdate-transact-sql
- Microsoft Learn: DATEDIFF (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datediff-transact-sql
- Microsoft Learn: DATEFROMPARTS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datefromparts-transact-sql
- Microsoft Learn: SYSUTCDATETIME (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sysutcdatetime-transact-sql
- Microsoft Learn: DATETRUNC (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/datetrunc-transact-sql
- MySQL: Date and Time Functions - https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
- Oracle: EXTRACT (datetime) - https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/EXTRACT-datetime.html
- Oracle: LAST_DAY - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/LAST_DAY.html
- Oracle: TO_CHAR (datetime) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_CHAR-datetime.html
- Oracle: TO_DATE - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TO_DATE.html
- PostgreSQL: Date/Time Functions and Operators - https://www.postgresql.org/docs/current/functions-datetime.html
- Amazon Redshift: DATE_TRUNC - https://docs.aws.amazon.com/redshift/latest/dg/r_DATE_TRUNC.html
- ScienceDirect: Date and Time Manipulation - https://www.sciencedirect.com/topics/computer-science/column-data-type