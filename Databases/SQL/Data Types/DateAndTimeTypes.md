# Comprehensive Programming Cheat Sheet: SQL Date and Time Types

## Topic Overview

### Definitions

**Core Definition**
Date and time types are the SQL data types used to store calendar dates, clock times, timestamps (date plus time), time-zone-aware instants, and durations, each with defined ranges, resolution, storage requirements, and time-zone semantics.

**Technical Definition**
SQL date and time types fall into several categories: **date-only types** (`DATE`) store a calendar date without a time component; **time-only types** (`TIME`, `TIME WITH TIME ZONE`, `TIMETZ`) store a time of day, optionally with a UTC offset; **timestamp types** (`TIMESTAMP`, `TIMESTAMP WITH TIME ZONE`, `TIMESTAMPTZ`) store a date and time, where the “with time zone” variants store an absolute instant in UTC and render it in the session's time zone; and **duration types** (`INTERVAL`) store a span of time expressed in months, days, and microseconds. PostgreSQL supports the full set of SQL date and time types, including `timestamp [ (p) ] [ without time zone ]`, `timestamp [ (p) ] with time zone`, `date`, `time [ (p) ] [ without time zone ]`, `time [ (p) ] with time zone`, and `interval [ fields ] [ (p) ]`, with resolution down to 1 microsecond and storage sizes ranging from 4 bytes (`date`) to 16 bytes (`interval`). MySQL provides `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`, and `YEAR`, with `TIMESTAMP` converting values to UTC for storage and back to the session time zone for retrieval. SQL Server provides `date`, `time`, `datetime`, `datetime2`, `smalldatetime`, and `datetimeoffset`, where `datetimeoffset` adds time-zone awareness based on UTC.

**Beginner-Friendly Explanation**
Date and time types are the clocks and calendars of a database. `DATE` is a calendar page. `TIME` is a clock face. `TIMESTAMP` is a calendar page and a clock face together. `TIMESTAMP WITH TIME ZONE` is a calendar page, a clock face, and a time-zone label—it records exactly when something happened, anywhere on Earth. `INTERVAL` is a stopwatch: it measures how long something lasted. Choosing the right type is like choosing the right instrument: a calendar for a birthday, a clock for a daily alarm, a stopwatch for a race, and a world clock for a global conference call.

### Key Characteristics

- **Time-zone awareness is the critical distinction**: `TIMESTAMP WITHOUT TIME ZONE` stores a literal wall-clock value with no UTC anchor; `TIMESTAMP WITH TIME ZONE` (and SQL Server's `datetimeoffset`) stores an absolute instant (internally in UTC) and renders it in the session's zone. MySQL's `DATETIME` is time-zone-agnostic, while `TIMESTAMP` converts to UTC on storage.
- **Ranges and resolutions vary significantly**: PostgreSQL's `date` spans 4713 BC to 5874897 AD with 1-day resolution; `timestamp` spans 4713 BC to 294276 AD with 1-microsecond resolution; `interval` spans ±178,000,000 years. MySQL's `TIMESTAMP` is limited to 1970-01-01 to 2038-01-19 UTC (the 32-bit UNIX epoch limit). SQL Server's `datetime2` spans 0001-01-01 to 9999-12-31 with 100-nanosecond accuracy, and `datetimeoffset` adds a -14:00 to +14:00 offset range.
- **DST and ambiguous times are handled differently**: PostgreSQL assigns invalid timestamps during a DST “spring forward” the offset that existed before the transition, and ambiguous timestamps during a “fall back” the offset that exists after the transition. MySQL's `TIMESTAMP` conversion depends on the session time zone; `DATETIME` does not convert at all.
- **`TIMETZ` (time with time zone) is of questionable usefulness**: PostgreSQL's own documentation notes that the `time with time zone` type “exhibits properties which lead to questionable usefulness” and recommends using `date`, `time`, `timestamp`, and `timestamptz` combinations instead.
- **Best practice is unambiguous: use `TIMESTAMPTZ` (PostgreSQL) or `datetimeoffset` (SQL Server) for absolute instants**. Reserve `DATE`, `TIME`, and `TIMESTAMP WITHOUT TIME ZONE` for civil (wall-clock) values such as birthdays, opening hours, and scheduled local events.

### Prerequisites

- A running RDBMS instance (PostgreSQL, MySQL, or SQL Server) and a client connection.
- Familiarity with `CREATE TABLE` syntax and column definitions.
- Understanding of the application's temporal requirements: whether values represent absolute instants (e.g., transaction times) or civil values (e.g., birthdates), whether time zones matter, and what range of dates must be supported.
- Awareness of the session time zone setting (`TimeZone` in PostgreSQL, `time_zone` in MySQL, `AT TIME ZONE` in SQL Server).

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data integrity – DST transitions, leap seconds, and range overflow.
- Query performance – date/time columns are frequently indexed and filtered; type choice affects index efficiency and sargability.
- Application mapping – SQL temporal types correspond to language-level types such as `Instant`, `OffsetDateTime`, `LocalDate`, and `ZonedDateTime`.
- Data warehousing – time-series partitioning and date dimension tables.

---

## Core Concepts and Features

### 1. `DATE`

#### Definitions

**Core Definition**
`DATE` is a type that stores a calendar date (year, month, day) without a time-of-day component.

**Technical Definition**
PostgreSQL's `date` type occupies **4 bytes**, covers **4713 BC to 5874897 AD** with **1-day resolution**, and is stored as a calendar date without a time zone. MySQL's `DATE` occupies **3 bytes**, covers **1000-01-01 to 9999-12-31**, and stores only the calendar date in `YYYY-MM-DD` format. SQL Server's `date` occupies **3 bytes**, covers **0001-01-01 through 9999-12-31**, and has a **1-day** accuracy.

**Beginner-Friendly Explanation**
`DATE` is a calendar page. It records the year, month, and day—nothing more. It is perfect for birthdays, anniversaries, and due dates, where the exact time of day does not matter.

#### Purposes

- To store civil dates that have no meaningful time component (birthdays, holidays, contract dates).
- To filter or group records by day without the overhead of a full timestamp.
- To reduce storage consumption compared with timestamp types when only a date is needed.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name DATE
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DATE` | A calendar date with year, month, and day. |

**Syntax rules:**

- `DATE` accepts ISO 8601 format (`YYYY-MM-DD`) and, in PostgreSQL, many other formats depending on the `DateStyle` setting.
- `DATE` has no time zone; it is a civil value.

**Constraints and limitations:**

- `DATE` cannot store a time of day; use `TIME` or `TIMESTAMP` for that.
- MySQL's `DATE` range is limited to 1000-01-01 through 9999-12-31.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – DATE for birthdates

```sql
-- Step 1: Create a table with a DATE column.
CREATE TABLE people (
    person_id serial PRIMARY KEY,
    name      text NOT NULL,
    birthdate date NOT NULL
);

-- Step 2: Insert a birthdate.
INSERT INTO people (name, birthdate) VALUES ('Alice', '1990-05-15');

-- Step 3: Query by date.
SELECT name, birthdate FROM people WHERE birthdate = '1990-05-15';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 name  | birthdate
-------+------------
 Alice | 1990-05-15
(1 row)
```

**Why this output appears**: The `birthdate` column stores only the calendar date `1990-05-15`. The `WHERE` clause matches the date exactly, returning the row.

#### Real-World Cases

- **HR systems**: Employee birthdates and hire dates are stored as `DATE` because the time of day is irrelevant.
- **Billing**: Invoice due dates and contract start dates are civil dates that do not change across time zones.

---

### 2. `TIME`

#### Definitions

**Core Definition**
`TIME` (or `TIME WITHOUT TIME ZONE`) is a type that stores a time of day (hour, minute, second) without a date or time zone.

**Technical Definition**
PostgreSQL's `time [ (p) ] without time zone` occupies **8 bytes**, covers **00:00:00 to 24:00:00** with **1-microsecond resolution**, and stores a time of day with no date and no time zone. MySQL's `TIME` occupies **3 bytes**, covers **-838:59:59 to 838:59:59** (allowing negative durations), and supports fractional seconds up to 6 digits. SQL Server's `time` occupies **3 to 5 bytes**, covers **00:00:00.0000000 through 23:59:59.9999999**, and has **100-nanosecond accuracy**.

**Beginner-Friendly Explanation**
`TIME` is a clock face without a date. It records the hour, minute, and second—nothing more. It is useful for opening hours, daily schedules, and recurring events where the date is handled separately.

#### Purposes

- To store a time of day when the date is irrelevant or stored elsewhere.
- To represent recurring daily schedules (e.g., a store opens at 09:00).
- To represent durations when the number of hours may exceed 24 (MySQL's extended `TIME` range).

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name TIME [ (p) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TIME` | A time of day (hour, minute, second). |
| `(p)` | Optional fractional seconds precision (0–6 in PostgreSQL). |

**Syntax rules:**

- `time` alone is equivalent to `time without time zone` in PostgreSQL.
- MySQL's `TIME` accepts values outside the 00:00–24:00 range, using the sign to indicate negative durations.

**Constraints and limitations:**

- `TIME` does not store a date; combining it with a separate `DATE` column requires application logic.
- In PostgreSQL, if a time zone is specified in the input for `time without time zone`, it is silently ignored.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TIME for daily schedules

```sql
-- Step 1: Create a table with a TIME column.
CREATE TABLE store_hours (
    store_id  int,
    open_time time,
    close_time time
);

-- Step 2: Insert a schedule.
INSERT INTO store_hours VALUES (1, '09:00', '17:00');

-- Step 3: Query stores open at a given time.
SELECT store_id FROM store_hours WHERE open_time <= '10:00' AND close_time > '10:00';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
 store_id
----------
        1
(1 row)
```

**Why this output appears**: The `TIME` columns store only the time of day. The query compares the current time against the opening and closing times, returning the store that is open at 10:00.

#### Real-World Cases

- **Retail**: Store opening and closing hours are stored as `TIME` because they recur daily and are independent of the date.
- **Transportation**: Train and bus schedules use `TIME` for departure and arrival times on a given route.

---

### 3. `TIME WITH TIME ZONE` / `TIMETZ`

#### Definitions

**Core Definition**
`TIME WITH TIME ZONE` (abbreviated `TIMETZ`) is a type that stores a time of day together with a UTC offset, but without a date.

**Technical Definition**
PostgreSQL's `time [ (p) ] with time zone` occupies **12 bytes**, covers **00:00:00+1559 to 24:00:00-1559** with **1-microsecond resolution**, and stores a time of day with a time-zone offset. The appropriate offset is recorded in the value and is output as stored; it is not adjusted to the active time zone. PostgreSQL's documentation explicitly notes that this type “exhibits properties which lead to questionable usefulness” because without a date, the offset cannot account for daylight-saving transitions correctly, and it recommends using combinations of `date`, `time`, `timestamp`, and `timestamptz` instead.

**Beginner-Friendly Explanation**
`TIMETZ` is a clock face with a time-zone label, but no calendar. It tells you “it is 10:00 AM in New York,” but because there is no date, it cannot tell you whether New York is on standard time or daylight-saving time on that day. This makes it rarely useful in practice.

#### Purposes

- To store a time of day with an explicit UTC offset when the date is not needed and DST is not a concern.
- To represent fixed-offset times such as those used in some scheduling systems.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name TIME [ (p) ] WITH TIME ZONE
-- or
column_name TIMETZ [ (p) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TIME WITH TIME ZONE` | A time of day with a UTC offset. |
| `TIMETZ` | PostgreSQL abbreviation for the same type. |

**Syntax rules:**

- The offset is stored as part of the value and is returned as stored.
- In PostgreSQL, the offset is not adjusted to the session time zone on output.

**Constraints and limitations:**

- Without a date, `TIMETZ` cannot correctly handle daylight-saving transitions.
- PostgreSQL's own documentation warns against its use in most applications.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TIMETZ stores and returns the offset as given

```sql
-- Step 1: Create a table with a TIMETZ column.
CREATE TABLE events (
    event_id int,
    local_time time with time zone
);

-- Step 2: Insert a value with an explicit offset.
INSERT INTO events VALUES (1, '10:00:00+02');

-- Step 3: Query the stored value.
SELECT local_time FROM events;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
  local_time
---------------
 10:00:00+02
(1 row)
```

**Why this output appears**: The `TIMETZ` value is stored with its offset (`+02`) and returned with the same offset. Unlike `timestamptz`, it does not convert to the session time zone because there is no date to anchor the conversion.

#### Real-World Cases

- **Legacy systems**: Some legacy applications use `TIMETZ` for fixed-offset schedules, but modern designs generally prefer full timestamps.
- **Rare scheduling edge cases**: A system that records “10:00 AM in a specific fixed-offset zone” may use `TIMETZ`, accepting that DST is not handled.

---

### 4. `TIMESTAMP` / `TIMESTAMP WITHOUT TIME ZONE`

#### Definitions

**Core Definition**
`TIMESTAMP` (or `TIMESTAMP WITHOUT TIME ZONE`) is a type that stores a date and time together, without any time-zone information.

**Technical Definition**
PostgreSQL's `timestamp [ (p) ] [ without time zone ]` occupies **8 bytes**, covers **4713 BC to 294276 AD** with **1-microsecond resolution**, and stores a date and time with no time-zone conversion. The SQL standard requires that writing just `timestamp` be equivalent to `timestamp without time zone`, and PostgreSQL honors that behaviour. MySQL's `DATETIME` occupies **8 bytes**, covers **1000-01-01 00:00:00 to 9999-12-31 23:59:59**, and stores the literal date and time with no time-zone conversion. SQL Server's `datetime2` occupies **6 to 8 bytes**, covers **0001-01-01 00:00:00.0000000 through 9999-12-31 23:59:59.9999999**, and has **100-nanosecond accuracy**.

**Beginner-Friendly Explanation**
`TIMESTAMP WITHOUT TIME ZONE` is a calendar page and a clock face with no time-zone label. It says “2025-06-15 10:00:00,” but it does not say where in the world that time applies. If you move the database to a different time zone, the value does not change—it still says 10:00:00, even though the actual moment it represents has shifted.

#### Purposes

- To store civil date-time values where the local wall-clock time is what matters (e.g., a scheduled appointment at a clinic).
- To store literal values that should not shift when the session time zone changes.
- To represent dates and times in systems that are explicitly single-time-zone and do not require cross-zone consistency.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name TIMESTAMP [ (p) ] [ WITHOUT TIME ZONE ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TIMESTAMP` | Date and time without time zone. |
| `(p)` | Optional fractional seconds precision. |
| `WITHOUT TIME ZONE` | Explicit spelling of the default behaviour. |

**Syntax rules:**

- In PostgreSQL, `timestamp` without a qualifier means `timestamp without time zone`.
- Input strings with a time-zone offset are silently accepted but the offset is ignored; the value is stored as the literal fields.

**Constraints and limitations:**

- `TIMESTAMP WITHOUT TIME ZONE` cannot answer questions like “how many events happened in the last hour?” across DST transitions or across users in different zones.
- Storing UTC in a `timestamp without time zone` column is a known anti-pattern; use `timestamptz` or `datetimeoffset` instead.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TIMESTAMP ignores input offsets

```sql
-- Step 1: Create a table with a timestamp without time zone column.
CREATE TABLE appointments (
    appointment_id serial PRIMARY KEY,
    scheduled_at   timestamp
);

-- Step 2: Insert a value with an explicit offset.
INSERT INTO appointments (scheduled_at) VALUES ('2025-06-15 10:00:00+02');

-- Step 3: Query the stored value.
SELECT scheduled_at FROM appointments;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
   scheduled_at
---------------------
 2025-06-15 10:00:00
(1 row)
```

**Why this output appears**: PostgreSQL silently ignores the `+02` offset when the target column is `timestamp without time zone`. The stored value is the literal wall-clock time `10:00:00`, with no adjustment.

#### Real-World Cases

- **Clinic scheduling**: An appointment is scheduled for 10:00 local time at the clinic. The clinic is in one time zone, so `DATETIME` or `timestamp without time zone` is appropriate.
- **Legacy systems**: Older applications that predate time-zone-aware types often store literal timestamps in `DATETIME` or `timestamp` columns.

---

### 5. `TIMESTAMP WITH TIME ZONE` / `TIMESTAMPTZ`

#### Definitions

**Core Definition**
`TIMESTAMP WITH TIME ZONE` (abbreviated `TIMESTAMPTZ`) is a type that stores an absolute instant in time, internally normalised to UTC, and rendered in the session's time zone on output.

**Technical Definition**
PostgreSQL's `timestamp [ (p) ] with time zone` occupies **8 bytes**, covers **4713 BC to 294276 AD** with **1-microsecond resolution**, and stores an absolute instant. An input string with an explicit time zone is converted to UTC using the appropriate offset for that zone; if no time zone is stated, it is assumed to be in the time zone indicated by the system's `TimeZone` parameter and converted to UTC. The originally stated or assumed time zone is **not retained**. On output, the value is always converted from UTC to the current `timezone` setting and displayed as local time in that zone. `timestamptz` is accepted as an abbreviation for `timestamp with time zone` (a PostgreSQL extension). SQL Server's `datetimeoffset` is the equivalent type, storing the local date and time plus the offset from UTC, with a **-14:00 to +14:00** offset range.

**Beginner-Friendly Explanation**
`TIMESTAMPTZ` is a world clock. It records exactly when something happened, regardless of where you are. When you insert “2025-06-15 10:00:00+02,” it remembers that as a specific instant. When a user in Tokyo queries it, they see the same instant rendered in Tokyo time; a user in New York sees it in New York time. The original offset is not preserved—only the absolute instant matters.

#### Purposes

- To store absolute instants such as transaction times, event timestamps, and audit logs that must be consistent across time zones.
- To enable correct time-based queries (e.g., “events in the last hour”) regardless of DST transitions or session time zones.
- To provide a single, unambiguous representation of a moment in time for multi-time-zone applications.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name TIMESTAMP [ (p) ] WITH TIME ZONE
-- or
column_name TIMESTAMPTZ [ (p) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `TIMESTAMP WITH TIME ZONE` | An absolute instant stored in UTC. |
| `TIMESTAMPTZ` | PostgreSQL abbreviation. |

**Syntax rules:**

- Input without an explicit offset is interpreted in the session `TimeZone`.
- Output is always rendered in the session `TimeZone`.
- The `AT TIME ZONE` construct converts a `timestamptz` to a `timestamp` in a specified zone, or vice versa.

**Constraints and limitations:**

- `TIMESTAMPTZ` does **not** store the original time zone; it stores the UTC instant. If you need to preserve the original zone, store it in a separate column.
- PostgreSQL handles invalid and ambiguous DST timestamps with specific rules: invalid times during “spring forward” receive the offset that existed before the transition, and ambiguous times during “fall back” receive the offset that exists after the transition.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – TIMESTAMPTZ converts to UTC and renders in session zone

```sql
-- Step 1: Create a table with a TIMESTAMPTZ column.
CREATE TABLE events (
    event_id   serial PRIMARY KEY,
    happened_at timestamptz
);

-- Step 2: Insert a value with an explicit offset.
INSERT INTO events (happened_at) VALUES ('2025-06-15 10:00:00+02');

-- Step 3: Set the session time zone to Tokyo and query.
SET timezone = 'Asia/Tokyo';
SELECT happened_at FROM events;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
SET
      happened_at
------------------------
 2025-06-15 17:00:00+09
(1 row)
```

**Why this output appears**: The input `10:00:00+02` represents the same instant as `08:00:00 UTC`. When the session time zone is set to `Asia/Tokyo` (UTC+9), the value is rendered as `17:00:00+09`, preserving the absolute instant while displaying it in the session zone.

#### Real-World Cases

- **Audit logs**: Transaction timestamps must record the exact instant regardless of where the server or user is located; `timestamptz` or `datetimeoffset` is the correct choice.
- **Multi-region SaaS**: Events generated in different regions are stored as absolute instants and rendered in each user's local time zone on retrieval.

---

### 6. `INTERVAL`

#### Definitions

**Core Definition**
`INTERVAL` is a type that stores a duration or span of time expressed in months, days, and microseconds.

**Technical Definition**
PostgreSQL's `interval [ fields ] [ (p) ]` occupies **16 bytes**, covers **-178,000,000 years to 178,000,000 years** with **1-microsecond resolution**, and stores the duration as three independent fields—**months**, **days**, and **microseconds**. The `fields` option restricts the stored fields (e.g., `INTERVAL YEAR TO MONTH` or `INTERVAL DAY TO SECOND`), and `p` specifies fractional-second precision (0–6). MySQL's `TIME` type can represent durations from `-838:59:59` to `838:59:59`, but for date arithmetic MySQL also supports `INTERVAL` expressions (e.g., `DATE_ADD(date, INTERVAL 1 DAY)`). SQL Server provides `DATEADD` for date arithmetic but does not have a standalone `INTERVAL` data type.

**Beginner-Friendly Explanation**
`INTERVAL` is a stopwatch. It measures how long something lasts—two hours, three days, one month. Unlike a timestamp, it does not say *when* something happened; it says *how long* it took. PostgreSQL's `INTERVAL` is smart about calendar arithmetic: one month is not always 30 days, and one day is not always 24 hours during a DST transition, so `INTERVAL` keeps months, days, and seconds separate.

#### Purposes

- To store durations such as subscription lengths, rental periods, or timeouts.
- To perform date arithmetic (`date + interval`) that respects calendar rules and DST.
- To express time spans in a human-readable and calendar-aware way.

#### Syntax Rules and Structure

**General syntax:**

```sql
column_name INTERVAL [ fields ] [ (p) ]
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `fields` | Optional restriction (e.g., `YEAR TO MONTH`, `DAY TO SECOND`). |
| `(p)` | Optional fractional-seconds precision (0–6). |

**Syntax rules:**

- PostgreSQL stores intervals as three independent fields (months, days, microseconds) to handle variable-length months and DST transitions correctly.
- `1 month`, `30 days`, and `2592000 seconds` are different values because months vary in length and days can be 23, 24, or 25 hours under DST.

**Constraints and limitations:**

- MySQL does not have a first-class `INTERVAL` data type; it uses `TIME` for durations up to 838 hours and `INTERVAL` expressions for date arithmetic.
- SQL Server uses `DATEADD` rather than an `INTERVAL` type.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – INTERVAL for subscription duration

```sql
-- Step 1: Create a table with an INTERVAL column.
CREATE TABLE subscriptions (
    subscription_id serial PRIMARY KEY,
    started_at      timestamptz,
    duration        interval
);

-- Step 2: Insert a subscription that lasts 3 months.
INSERT INTO subscriptions (started_at, duration)
VALUES ('2025-01-15 00:00:00+00', '3 months');

-- Step 3: Compute the end date.
SELECT started_at, duration, started_at + duration AS ends_at FROM subscriptions;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
      started_at      | duration |       ends_at
----------------------+----------+---------------------
 2025-01-15 00:00:00+00 | 3 mons  | 2025-04-15 00:00:00+00
(1 row)
```

**Why this output appears**: The interval `'3 months'` is added to the timestamp `2025-01-15`, producing `2025-04-15`. Because the interval stores months as a separate field, the result respects the varying lengths of January, February, and March.

#### Real-World Cases

- **Subscription billing**: A `duration` column stores the length of a subscription (e.g., `1 month`, `1 year`), and the end date is computed as `started_at + duration`.
- **SLA tracking**: An `INTERVAL` column stores the promised response time (e.g., `4 hours`), and queries compute `deadline = created_at + sla_interval`.

---

### 7. Time Zones

#### Definitions

**Core Definition**
Time zones are geographic regions that share the same local time, defined by their offset from UTC and their daylight-saving rules.

**Technical Definition**
Time-zone handling in SQL depends on the type and the RDBMS. **PostgreSQL** stores `timestamptz` values as UTC and renders them in the session's `TimeZone` parameter; `AT TIME ZONE` converts between `timestamptz` and `timestamp`. PostgreSQL interprets invalid timestamps during DST “spring forward” with the offset that existed before the transition, and ambiguous timestamps during “fall back” with the offset that exists after the transition. **MySQL** converts `TIMESTAMP` values from the current time zone to UTC for storage and back from UTC to the current time zone for retrieval; `DATETIME` does not convert at all. **SQL Server** provides `datetimeoffset`, which stores the local date and time plus the UTC offset, with a **-14:00 to +14:00** range; `datetime2` has no time-zone awareness.

**Beginner-Friendly Explanation**
Time zones are the reason “what time is it?” has more than one answer. A time-zone-aware type records the moment in a way that every clock on Earth agrees on—the absolute instant—while still letting each user see it on their own clock. A time-zone-ignorant type records the wall-clock reading and hopes everyone is in the same zone.

#### Purposes

- To ensure that events are ordered correctly regardless of the user's location.
- To display timestamps in each user's local time.
- To handle daylight-saving transitions without losing or duplicating hours.

#### Syntax Rules and Structure

**PostgreSQL `AT TIME ZONE`:**

```sql
-- Convert a timestamptz to a timestamp in a specific zone.
SELECT happened_at AT TIME ZONE 'America/New_York' FROM events;

-- Convert a timestamp (assumed to be in a zone) to timestamptz.
SELECT scheduled_at AT TIME ZONE 'Europe/London' FROM appointments;
```

**MySQL session time zone:**

```sql
SET time_zone = 'America/New_York';
```

**SQL Server `AT TIME ZONE`:**

```sql
SELECT happened_at AT TIME ZONE 'Tokyo Standard Time' FROM events;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AT TIME ZONE` | Converts between time-zone-aware and time-zone-ignorant types. |
| `timezone` / `time_zone` | Session parameter controlling the display zone. |
| `datetimeoffset` | SQL Server's time-zone-aware type. |

**Syntax rules:**

- In PostgreSQL, `AT TIME ZONE` on a `timestamptz` yields a `timestamp` in the specified zone; on a `timestamp`, it yields a `timestamptz`.
- MySQL's `TIMESTAMP` conversion depends on the session `time_zone`; `DATETIME` is never converted.

**Constraints and limitations:**

- Storing UTC in a `timestamp without time zone` column is an anti-pattern; use `timestamptz` or `datetimeoffset`.
- `TIMESTAMP` in MySQL has a range limited to 2038-01-19 UTC; `DATETIME` and `TIMESTAMPTZ` do not have this limitation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – AT TIME ZONE conversion

```sql
-- Step 1: Create a table with a timestamptz column.
CREATE TABLE meetings (meeting_id int, starts_at timestamptz);
INSERT INTO meetings VALUES (1, '2025-06-15 14:00:00+00');

-- Step 2: Convert to New York local time.
SELECT starts_at AT TIME ZONE 'America/New_York' AS ny_time FROM meetings;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
       ny_time
---------------------
 2025-06-15 10:00:00
(1 row)
```

**Why this output appears**: The `timestamptz` value `14:00:00 UTC` is converted to `America/New_York` time, which is UTC−4 during daylight-saving time, yielding `10:00:00`. The result is a `timestamp` (without time zone) representing the local wall-clock time in New York.

#### Real-World Cases

- **Global event platforms**: A conference scheduled for 14:00 UTC is displayed to attendees in their local time zones using `AT TIME ZONE`.
- **Audit compliance**: Regulators require that transaction timestamps be stored as absolute instants (UTC) and rendered in the relevant jurisdiction's local time for reporting.

---

### 8. Date and Time Best Practices

#### Definitions

**Core Definition**
Date and time best practices are the guidelines for selecting, storing, and manipulating temporal data to ensure correctness, consistency, and performance across time zones and DST transitions.

**Technical Definition**
Best practices include: (1) **use `TIMESTAMPTZ` (PostgreSQL) or `datetimeoffset` (SQL Server) for absolute instants**, and reserve `DATE`, `TIME`, and `TIMESTAMP WITHOUT TIME ZONE` for civil values; (2) **never store UTC in a `timestamp without time zone` column**; (3) **use `DATE` for birthdates and anniversaries**, where the time of day and time zone are irrelevant; (4) **use MySQL's `TIMESTAMP` for time-zone-aware logging** and `DATETIME` for time-zone-agnostic civil values; (5) **avoid `TIMETZ`** because it cannot correctly handle DST without a date; (6) **use `'infinity'::timestamptz` for unbounded future values** rather than `NULL` or sentinel dates like `'9999-12-31'`; (7) **use `now()` for transaction-start timestamps** and `clock_timestamp()` for statement-execution timestamps; and (8) **be aware of the 2038 problem** when using MySQL's `TIMESTAMP`.

**Beginner-Friendly Explanation**
Best practices are the habits that keep date and time data correct. Use time-zone-aware types for anything that represents a real moment, use plain dates for birthdays, avoid ambiguous types like `TIMETZ`, and never store UTC in a column that ignores time zones. These habits prevent the classic bugs where events appear to happen in the future, reports show the wrong day, and schedules break during daylight-saving changes.

#### Purposes

- To prevent silent data corruption caused by time-zone mismatches.
- To ensure that time-based queries return correct results across DST transitions and user locations.
- To avoid range limitations and overflow errors (e.g., the 2038 problem).
- To improve performance by using the appropriate type for the query pattern.

#### Syntax Rules and Structure

**Best-practice guidance:**

| Scenario | Recommended type | Rationale |
|----------|------------------|-----------|
| Absolute instant (transaction, event, audit log) | `timestamptz` (PostgreSQL), `datetimeoffset` (SQL Server) | Unambiguous; renders in any session zone |
| Civil date (birthday, holiday) | `DATE` | No time zone; no time component |
| Civil time (opening hours) | `TIME` | No date; no time zone |
| Civil date-time (clinic appointment) | `timestamp without time zone` / `DATETIME` | Local wall-clock time matters |
| MySQL time-zone-aware logging | `TIMESTAMP` | Converts to UTC; 4-byte storage |
| Duration / span | `INTERVAL` (PostgreSQL), `TIME` (MySQL), `DATEADD` (SQL Server) | Calendar-aware or fixed-duration arithmetic |
| Unbounded future | `'infinity'::timestamptz` (PostgreSQL) | Sorts correctly; no NULL branches |

**Syntax rules:**

- Use `AT TIME ZONE` to convert between time-zone-aware and time-zone-ignorant types.
- Set the session time zone (`SET timezone` in PostgreSQL, `SET time_zone` in MySQL) appropriately for display.
- Use `now()` for the transaction-start time and `clock_timestamp()` for the actual statement-execution time.

**Constraints and limitations:**

- `TIMESTAMP` in MySQL has a hard upper limit of 2038-01-19 03:14:07 UTC; use `DATETIME` for dates beyond 2038.
- `TIMETZ` cannot correctly resolve DST without a date; avoid it unless the use case is extremely constrained.
- Changing a column's type later can require a table rewrite and may be disruptive on large tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: PostgreSQL – Best-practice temporal schema

```sql
-- Step 1: Create a table applying best practices.
CREATE TABLE orders (
    order_id    bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    placed_at   timestamptz NOT NULL DEFAULT now(),
    promised_on date NOT NULL,
    sla         interval NOT NULL DEFAULT '2 days',
    deadline_at timestamptz GENERATED ALWAYS AS (placed_at + sla) STORED
);

-- Step 2: Insert an order.
INSERT INTO orders (promised_on) VALUES ('2025-06-20') RETURNING placed_at, deadline_at;
```

**Expected output:**

```text
CREATE TABLE
      placed_at       |      deadline_at
----------------------+---------------------
 2026-11-13 10:00:00+00 | 2026-11-15 10:00:00+00
(1 row)
```

**Why this output appears**: The `placed_at` column uses `timestamptz` to record the exact instant of order placement. The `promised_on` column uses `date` for the civil delivery date. The `sla` column uses `interval` for a calendar-aware duration. The `deadline_at` generated column computes the absolute deadline as `placed_at + sla`, preserving time-zone correctness.

#### Real-World Cases

- **Global e-commerce**: Orders use `timestamptz` for `placed_at`, `date` for `promised_delivery_date`, and `interval` for return windows, ensuring correctness across time zones and DST.
- **HR systems**: Birthdates and hire dates use `DATE`; shift start and end times use `TIME`; actual clock-in and clock-out events use `timestamptz` or `datetimeoffset`.

---

## References

- PostgreSQL Documentation – Date/Time Types – https://www.postgresql.org/docs/current/datatype-datetime.html
- PostgreSQL Documentation – Date/Time Types (version 15) – https://www.postgresql.org/docs/15/datatype-datetime.html
- MySQL 8.0 Reference Manual – The DATE, DATETIME, and TIMESTAMP Types – https://dev.mysql.com/doc/refman/8.0/en/datetime.html
- MySQL 8.0 Reference Manual – The TIME Type – https://dev.mysql.com/doc/refman/8.0/en/time.html
- Microsoft Learn – Date and Time Data Types and Functions (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql
- Microsoft Learn – datetimeoffset (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/datetimeoffset-transact-sql
- Microsoft Learn – datetime2 (Transact-SQL) – https://learn.microsoft.com/en-us/sql/t-sql/data-types/datetime2-transact-sql
- PostgreSQL Mailing List – Please clarify that Timestamp with Timezone doesn't preserve the timezone – https://www.postgresql.org/message-id/...
- PostgreSQL Mailing List – Re: timestamp and timestamptz – https://www.postgresql.org/message-id/...
- GitHub – Timestamps, Time Zones, Intervals (PostgreSQL reference) – https://github.com/damusix/skills/blob/main/postgres/references/19-timestamp-timezones.md
- Devart – MySQL Date Format Guide: Queries, Functions & Examples – https://www.devart.com/blog/mysql-date-format.html
- OneUptime Blog – How to Choose Between DATETIME and TIMESTAMP in MySQL – https://github.com/OneUptime/blog/blob/master/posts/2026-03-31-mysql-choose-datetime-vs-timestamp/README.md
- Rockdata – Caveats for Some Data Types in PostgreSQL – https://www.rockdata.net/postgresql-caveats-for-some-data-types/
- Microsoft Learn – Time Zone Support in mssql-django – https://learn.microsoft.com/en-us/sql/connect/python/mssql-django/time-zone-support
- Red Gate – MySQL vs. PostgreSQL: Date and Time Data Types – https://www.red-gate.com/blog/mysql-vs-postgresql-date-and-time-data-types