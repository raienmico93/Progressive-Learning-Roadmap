# C++ Time and Date (`<chrono>`) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The C++ `<chrono>` library is a flexible collection of types that track time with varying degrees of precision, providing type-safe abstractions for durations (time intervals), time points (instants in time), and clocks (sources of time).

**Technical Definition**
The chrono library, defined in the `<chrono>` header and namespace `std::chrono` (since C++11), provides three fundamental abstractions: **durations** (a span of time represented as a count of ticks of some time unit), **time points** (a duration of time that has passed since the epoch of a specific clock), and **clocks** (a starting point or epoch combined with a tick rate). C++20 extends the library with calendar types (`year`, `month`, `day`, `year_month_day`) and time zone support via `std::chrono::tzdb`, an interface to the IANA Time Zone Database. The library encodes dimensional analysis into the type system, preventing accidental mixing of incompatible time units and silently losing precision.

**Beginner-Friendly Explanation**
The chrono library is C++'s modern, type-safe way to work with time. Instead of using raw numbers (where you might accidentally add seconds to milliseconds), chrono gives you distinct types for durations (like `seconds`, `milliseconds`), time points (a specific moment), and clocks (sources of "now"). It is like having a unit-aware calculator for time: the compiler catches mistakes before they happen. C++20 adds real calendar dates and time zones, so you can say "December 25, 2024" or "3 PM in New York" directly in code.

### Key Characteristics

- **Type-safe** – durations encode their unit in the type; seconds and milliseconds are distinct types.
- **Compile-time ratios** – tick periods are represented as `std::ratio` compile-time fractions.
- **Three standard clocks** – `system_clock` (wall-clock), `steady_clock` (monotonic), and `high_resolution_clock` (shortest tick period).
- **Epoch-based** – every clock has an epoch (starting point); time points measure time since that epoch.
- **Extensible** – C++20 adds calendars, time zones, and formatting.
- **Header-only** – all functionality is available by including `<chrono>`.

### Prerequisites

- Basic C++ syntax (variables, types, templates).
- Familiarity with `std::ratio` and compile-time constants.
- Understanding of namespaces and `using` declarations.
- Knowledge of stream I/O (for printing) and `std::format` (C++20).

### Related Programming Areas

- **Concurrency** – `std::this_thread::sleep_for` and `sleep_until` take chrono durations/time points.
- **Filesystem** – `std::filesystem::file_time_type` is a chrono time point.
- **Logging and profiling** – measuring elapsed time with `steady_clock`.
- **Scheduling** – timeouts and deadlines using time points.
- **Internationalization** – time zones and calendar formatting.

### Core Concepts / Features

1. `<chrono>` (the library and header)
2. Duration
3. Time point
4. Clock
5. `std::chrono::system_clock`
6. `std::chrono::steady_clock`
7. `std::chrono::high_resolution_clock`
8. Time zones (C++20)
9. Calendars (C++20)
10. Date and time best practices

---

## 1. `<chrono>` — The Time Library

### Definitions

**Core Definition**
`<chrono>` is the standard library header that declares the chrono time library, providing clocks, time points, durations, and (since C++20) calendar and time zone support.

**Technical Definition**
The header defines the namespace `std::chrono` and the inline namespace `std::chrono::literals` (for duration literals such as `1s`, `500ms`). It provides the `Duration`, `TimePoint`, and `Clock` concepts, several standard clocks, duration aliases (`nanoseconds`, `microseconds`, `milliseconds`, `seconds`, `minutes`, `hours`, `days`, `weeks`, `months`, `years`), and C++20 calendar/time zone types.

**Beginner-Friendly Explanation**
`<chrono>` is the header you include to use all the time-related tools in C++. It brings in clocks, durations, time points, and (in C++20) calendar and time zone support.

### Purposes

- To provide a unified, type-safe interface for time manipulation.
- To replace the error-prone C `<ctime>` API.
- To enable portable time measurement and calendar operations.
- To support duration arithmetic and conversions.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <chrono>
using namespace std::chrono;           // or use std::chrono:: explicitly
using namespace std::chrono_literals;  // for 1s, 500ms, etc.
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `std::chrono::duration` | Time interval template. |
| `std::chrono::time_point` | Instant in time template. |
| `std::chrono::system_clock` | Wall-clock clock. |
| `std::chrono::steady_clock` | Monotonic clock. |
| `std::chrono::high_resolution_clock` | Highest-resolution clock (often an alias). |
| `std::chrono::year_month_day` | Calendar date (C++20). |
| `std::chrono::zoned_time` | Time in a specific time zone (C++20). |

**Syntax Rules**
- All chrono types are in `std::chrono`.
- Duration literals require `using namespace std::chrono_literals;`.
- C++20 calendar/time zone features require a C++20-compliant compiler and library.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    auto d = 5s;                           // duration literal
    std::cout << d.count() << " seconds\n";
}
```

**Expected Output**
```
5 seconds
```

**Why this output?**  
`5s` creates a `std::chrono::seconds` duration; `count()` returns the number of ticks.

### Real-World Cases

- **Profiling** – measuring function execution time.
- **Scheduling** – delays and timeouts in concurrent code.
- **Logging** – timestamping events with calendar dates and time zones.

---

## 2. Duration

### Definitions

**Core Definition**
A duration represents a span of time, defined as a number of ticks of some time unit.

**Technical Definition**
`std::chrono::duration<Rep, Period>` is a class template consisting of a tick count of type `Rep` and a tick period, where the tick period is a compile-time rational fraction (`std::ratio`) representing the time in seconds from one tick to the next. For example, `std::chrono::seconds` is `duration<int64_t, std::ratio<1>>`, and `std::chrono::milliseconds` is `duration<int64_t, std::ratio<1, 1000>>`. Duration supports arithmetic, comparison, and explicit conversion via `duration_cast`.

**Beginner-Friendly Explanation**
A duration is a "length of time" with a unit attached. `5 seconds`, `500 milliseconds`, and `2 hours` are all durations. Because the unit is part of the type, you cannot accidentally add seconds to milliseconds without an explicit conversion—the compiler will warn you.

### Purposes

- To represent time intervals with type-safe units.
- To perform arithmetic on time spans.
- To convert between units explicitly via `duration_cast`.
- To measure elapsed time using clocks.

### Syntax Rules and Structure

**General Syntax**
```cpp
std::chrono::duration<Rep, Period> d(count);
// Or use predefined aliases:
std::chrono::seconds s(5);
std::chrono::milliseconds ms(500);
```

**Component Breakdown**

| Alias | Definition |
|-------|------------|
| `nanoseconds` | `duration<int64_t, std::nano>` |
| `microseconds` | `duration<int64_t, std::micro>` |
| `milliseconds` | `duration<int64_t, std::milli>` |
| `seconds` | `duration<int64_t>` |
| `minutes` | `duration<int64_t, std::ratio<60>>` |
| `hours` | `duration<int64_t, std::ratio<3600>>` |

*Source: Dr-Sergey/learn_cpp duration aliases table*.

**Syntax Rules**
- `duration_cast<ToDuration>(d)` performs explicit conversion; truncates if the target has lower precision.
- `d.count()` returns the tick count.
- Duration arithmetic is type-safe; implicit conversions occur only when no precision is lost.

**Constraints and Limitations**
- `Rep` must be an arithmetic type or a type emulating one.
- `Period` must be a `std::ratio` specialization.
- Lossy conversions require `duration_cast`.

### Annotated Code Examples

**Example 1: Basic duration arithmetic**

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    auto total = 2h + 30min + 15s;
    std::cout << duration_cast<minutes>(total).count() << " minutes\n";
}
```

**Expected Output**
```
150 minutes
```

**Why this output?**  
Durations are converted to a common unit for addition; `2h 30min 15s` is 150.25 minutes, and `duration_cast<minutes>` truncates to 150.

**Example 2: Explicit unit conversion**

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    milliseconds ms(1500);
    auto s = duration_cast<seconds>(ms);
    std::cout << s.count() << " seconds\n"; // 1 (truncated)
}
```

**Expected Output**
```
1 seconds
```

**Why this output?**  
`duration_cast<seconds>` truncates the fractional part.

### Real-World Cases

- **Timeouts** – `std::this_thread::sleep_for(500ms)`.
- **Profiling** – measuring elapsed time in milliseconds or microseconds.
- **Scheduling** – representing deadlines and intervals.

---

## 3. Time Point

### Definitions

**Core Definition**
A time point represents a specific instant in time, defined as a duration since the epoch of a specific clock.

**Technical Definition**
`std::chrono::time_point<Clock, Duration = typename Clock::duration>` is a class template implemented as if it stores a value of type `Duration` indicating the time interval from the start of the `Clock`'s epoch. Different clocks have different epochs; time points from different clocks are generally not directly comparable.

**Beginner-Friendly Explanation**
A time point is a timestamp—a precise moment in time, like "2024-12-25 10:30:00 UTC." It is measured as the amount of time that has passed since the clock's starting point (epoch).

### Purposes

- To represent instants in time.
- To compare and order events chronologically.
- To compute durations between time points (by subtraction).
- To convert between clocks via `clock_cast` (C++20).

### Syntax Rules and Structure

**General Syntax**
```cpp
std::chrono::time_point<Clock> tp = Clock::now();
auto d = tp.time_since_epoch();        // duration since epoch
```

**Component Breakdown**

| Operation | Description |
|-----------|-------------|
| `Clock::now()` | Returns the current time point. |
| `tp.time_since_epoch()` | Returns the duration since the epoch. |
| `tp + duration` | Returns a new time point shifted by the duration. |
| `tp1 - tp2` | Returns the duration between two time points. |

**Syntax Rules**
- Time points from the same clock can be compared and subtracted.
- `time_point_cast<Duration>(tp)` converts to a different duration precision.
- `clock_cast` (C++20) converts between clocks.

**Constraints and Limitations**
- Time points from different clocks are not directly comparable (before C++20 `clock_cast`).
- The epoch is implementation-defined for most clocks.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    auto start = steady_clock::now();
    // ... do work ...
    auto end = steady_clock::now();
    auto elapsed = duration_cast<milliseconds>(end - start);
    std::cout << "Elapsed: " << elapsed.count() << " ms\n";
}
```

**Expected Output** (value depends on execution)
```
Elapsed: 0 ms
```

**Why this output?**  
Subtracting two time points yields a duration; `duration_cast` converts it to milliseconds.

### Real-World Cases

- **Benchmarking** – recording start and end time points.
- **Deadlines** – representing a future instant for timeout checks.
- **Logging** – timestamping events.

---

## 4. Clock

### Definitions

**Core Definition**
A clock consists of a starting point (epoch) and a tick rate, and provides a way to obtain the current time point.

**Technical Definition**
The `Clock` concept (since C++11) requires a clock type to provide `Clock::duration`, `Clock::time_point`, `Clock::is_steady`, and a static `now()` function returning a `time_point`. The standard defines several clocks: `system_clock`, `steady_clock`, `high_resolution_clock`, `utc_clock`, `tai_clock`, `gps_clock`, `file_clock`, and `local_t` (C++20).

**Beginner-Friendly Explanation**
A clock is a source of time—something that can tell you "what time is it now?" Different clocks have different properties: some can be adjusted (like a wall clock), some are monotonic (never go backward), and some have very fine precision.

### Purposes

- To obtain the current time point.
- To provide a reference epoch for time points.
- To define the tick rate for durations.
- To support different timing needs (wall-clock vs. interval measurement).

### Syntax Rules and Structure

**General Syntax**
```cpp
auto now = Clock::now();               // current time point
using Dur = Clock::duration;           // clock's duration type
```

**Component Breakdown**

| Clock | Properties | Use For |
|-------|------------|---------|
| `system_clock` | Wall clock, adjustable, Unix epoch | Calendar dates, file timestamps |
| `steady_clock` | Monotonic, never adjusted | Measuring intervals, timeouts |
| `high_resolution_clock` | Shortest tick period; alias for one of the above | Avoid—use `steady_clock` explicitly |

*Source: Dr-Sergey/learn_cpp clock table*.

**Syntax Rules**
- Every clock provides `now()`.
- `is_steady` is `true` if the clock is monotonic.
- Clock epochs are implementation-defined except where specified.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    std::cout << "system_clock steady: "
              << std::chrono::system_clock::is_steady << '\n';
    std::cout << "steady_clock steady: "
              << std::chrono::steady_clock::is_steady << '\n';
}
```

**Expected Output** (typical)
```
system_clock steady: false
steady_clock steady: true
```

**Why this output?**  
`system_clock` is adjustable (not steady); `steady_clock` is monotonic (steady).

---

## 5. `std::chrono::system_clock`

### Definitions

**Core Definition**
`std::chrono::system_clock` is the wall-clock time source representing the system-wide real-time clock.

**Technical Definition**
`system_clock` is a `Clock` that represents wall-clock time. Its epoch is typically the Unix epoch (January 1, 1970). It is adjustable (not steady) because the system administrator or NTP can change it. It provides `to_time_t` and `from_time_t` conversions to/from `std::time_t`.

**Beginner-Friendly Explanation**
`system_clock` is the clock that tells you the actual date and time—what you would see on your computer's clock. It is the right clock for timestamps and calendar operations.

### Purposes

- To obtain the current wall-clock time.
- To create calendar dates and timestamps.
- To convert to/from `std::time_t` for C interoperability.
- To log events with real-world dates and times.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto now = std::chrono::system_clock::now();
std::time_t t = std::chrono::system_clock::to_time_t(now);
auto tp = std::chrono::system_clock::from_time_t(t);
```

**Constraints and Limitations**
- Not monotonic; system time adjustments can cause it to jump backward.
- Not suitable for measuring intervals.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>
#include <ctime>

int main() {
    auto now = std::chrono::system_clock::now();
    std::time_t t = std::chrono::system_clock::to_time_t(now);
    std::cout << std::ctime(&t);
}
```

**Expected Output** (example)
```
Wed Nov 12 14:30:00 2026
```

**Why this output?**  
`to_time_t` converts the time point to `time_t`; `ctime` formats it as a human-readable string.

### Real-World Cases

- **File timestamps** – recording when a file was modified.
- **Logging** – adding real-world dates to log entries.
- **Scheduling** – setting calendar-based reminders.

---

## 6. `std::chrono::steady_clock`

### Definitions

**Core Definition**
`std::chrono::steady_clock` is a monotonic clock that will never be adjusted.

**Technical Definition**
`steady_clock` is a `Clock` whose `is_steady` is `true`. Its `now()` never returns a value earlier than a previous call. It is the reliable choice for measuring intervals and implementing timeouts.

**Beginner-Friendly Explanation**
`steady_clock` is a clock that always moves forward at a constant rate. It cannot be changed by the user or the system, so it is perfect for measuring how long something takes.

### Purposes

- To measure elapsed time reliably.
- To implement timeouts and deadlines.
- To benchmark code without interference from system clock adjustments.
- To provide monotonic time for algorithms and data structures.

### Syntax Rules and Structure

**General Syntax**
```cpp
auto start = std::chrono::steady_clock::now();
// ... work ...
auto end = std::chrono::steady_clock::now();
auto elapsed = end - start;
```

**Constraints and Limitations**
- The epoch is implementation-defined; not meaningful for calendar dates.
- Not guaranteed to have the highest resolution (use `high_resolution_clock` only if needed).

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>
#include <thread>

int main() {
    using namespace std::chrono;
    auto start = steady_clock::now();
    std::this_thread::sleep_for(100ms);
    auto end = steady_clock::now();
    std::cout << duration_cast<milliseconds>(end - start).count() << " ms\n";
}
```

**Expected Output** (example)
```
100 ms
```

**Why this output?**  
`steady_clock` measures the actual elapsed time of the sleep.

### Real-World Cases

- **Profiling** – measuring function execution time.
- **Game loops** – computing frame times.
- **Network timeouts** – detecting elapsed time for retries.

---

## 7. `std::chrono::high_resolution_clock`

### Definitions

**Core Definition**
`std::chrono::high_resolution_clock` is the clock with the shortest tick period available in the implementation.

**Technical Definition**
`high_resolution_clock` is often a type alias for either `system_clock` or `steady_clock`, depending on the standard library implementation (e.g., libstdc++ uses `system_clock`, MSVC uses `steady_clock`, libc++ depends on configuration). It is not guaranteed to be steady, even though its name suggests high precision.

**Beginner-Friendly Explanation**
`high_resolution_clock` sounds like the best clock for precision, but it is usually just another name for `system_clock` or `steady_clock`. It is generally better to use `steady_clock` explicitly for timing and `system_clock` for wall-clock time.

### Purposes

- To provide the finest available tick resolution (implementation-defined).
- To serve as a placeholder when maximum precision is needed (rarely the right choice).

### Syntax Rules and Structure

**General Syntax**
```cpp
auto now = std::chrono::high_resolution_clock::now();
```

**Constraints and Limitations**
- Not guaranteed to be steady.
- Often an alias for `system_clock` or `steady_clock`.
- Its resolution may not translate to accuracy.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    std::cout << std::boolalpha;
    std::cout << "high_resolution_clock steady: "
              << std::chrono::high_resolution_clock::is_steady << '\n';
}
```

**Expected Output** (implementation-dependent)
```
high_resolution_clock steady: true
```

**Why this output?**  
`is_steady` depends on the underlying clock type the implementation aliases.

### Real-World Cases

- **Rarely recommended** – prefer `steady_clock` for timing and `system_clock` for wall-clock time.

---

## 8. Time Zones (C++20)

### Definitions

**Core Definition**
C++20 time zone support provides types and functions to represent and convert times in specific IANA time zones, handling UTC offsets and Daylight Saving Time (DST) automatically.

**Technical Definition**
The `<chrono>` header provides `std::chrono::tzdb`, a complete interface to the IANA Time Zone Database. Key entry points are `get_tzdb()` (access the database), `locate_zone()` (look up a zone by name), and `current_zone()` (obtain the system's local time zone). The `std::chrono::zoned_time` class template represents a time point in a given time zone. The database can be reloaded at runtime via `reload_tzdb()`.

**Beginner-Friendly Explanation**
Time zone support lets you say "what time is it in New York?" or "convert 3 PM Tokyo time to London time" directly in C++. It knows about daylight saving time and historical time zone changes.

### Purposes

- To represent time in a specific time zone.
- To convert between time zones.
- To handle DST transitions automatically.
- To access the IANA Time Zone Database.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <chrono>
auto zone = std::chrono::locate_zone("America/New_York");
std::chrono::zoned_time zt{zone, std::chrono::system_clock::now()};
auto local = zt.get_local_time();
auto sys = zt.get_sys_time();
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `tzdb` | Time zone database. |
| `locate_zone(name)` | Looks up a time zone by IANA name. |
| `current_zone()` | Returns the system's local time zone. |
| `zoned_time` | Pairs a time point with a time zone. |
| `get_local_time()` | Returns the local time point. |
| `get_sys_time()` | Returns the system time point. |

**Syntax Rules**
- Time zone names use IANA format (e.g., `"America/New_York"`, `"Asia/Tokyo"`).
- `zoned_time` handles DST and offset arithmetic automatically.
- Requires a C++20-compliant library with time zone database support.

**Constraints and Limitations**
- Library support varies; some implementations may require linking or environment configuration.
- The tzdb must be available on the system.

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    auto now = system_clock::now();

    auto ny = locate_zone("America/New_York");
    auto local = zoned_time{ny, now};
    std::cout << "NYC: " << local << '\n';

    auto tokyo = zoned_time{"Asia/Tokyo", now};
    std::cout << "Tokyo: " << tokyo << '\n';
}
```

**Expected Output** (example)
```
NYC: 2026-11-12 09:30:00 EST
Tokyo: 2026-11-12 23:30:00 JST
```

**Why this output?**  
`zoned_time` converts the same system time point to the local time in each time zone, applying the correct offset and DST rules.

### Real-World Cases

- **International scheduling** – scheduling meetings across time zones.
- **Logging** – recording events in local time.
- **Travel applications** – converting flight times.

---

## 9. Calendars (C++20)

### Definitions

**Core Definition**
C++20 calendar types provide type-safe representations of Gregorian calendar dates, including year, month, day, weekday, and combinations thereof.

**Technical Definition**
C++20 adds calendar types to `<chrono>`: `year`, `month`, `day`, `weekday`, `year_month`, `month_day`, `year_month_day`, `year_month_weekday`, and others. The `operator/` is overloaded to build calendar dates (e.g., `2024y/December/25`). `year_month_day` represents a specific year, month, and day, with `std::chrono::days` resolution; it is a `TriviallyCopyable` `StandardLayoutType`. The `.ok()` member checks validity (e.g., February 30 is invalid).

**Beginner-Friendly Explanation**
Calendars let you write dates naturally: `2024y/December/25` is Christmas 2024. The compiler checks validity—February 30 is caught as an error. You can also compute weekdays and find the last day of a month.

### Purposes

- To represent calendar dates type-safely.
- To perform date arithmetic (add days, months, years).
- To validate dates.
- To format dates with `std::format`.

### Syntax Rules and Structure

**General Syntax**
```cpp
#include <chrono>
using namespace std::chrono;
auto date = 2024y/December/25;         // year_month_day
auto date2 = year{2024}/12/25d;        // alternative syntax
bool valid = date.ok();                 // validity check
```

**Component Breakdown**

| Type | Represents |
|------|------------|
| `year` | Gregorian year. |
| `month` | Month of year. |
| `day` | Day of month. |
| `weekday` | Day of week. |
| `year_month_day` | Full date (year, month, day). |
| `year_month_weekday` | Nth weekday of a month. |

**Syntax Rules**
- `operator/` constructs calendar dates.
- `.ok()` checks validity.
- `year_month_day` can be converted to/from `sys_days` for time point interoperability.
- Date arithmetic: `date + days{10}`, `date + months{1}`, `date + years{1}`.

**Constraints and Limitations**
- Only the Gregorian calendar is supported.
- Date arithmetic with months/years saturates at the last valid day (e.g., Jan 31 + 1 month = Feb 28/29).

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    auto today = 2024y/December/25;
    std::cout << today << '\n';            // 2024-12-25

    auto bad = 2024y/February/30d;
    std::cout << std::boolalpha << bad.ok() << '\n'; // false

    auto last_feb = 2024y/February/last;
    std::cout << year_month_day{last_feb} << '\n'; // 2024-02-29
}
```

**Expected Output**
```
2024-12-25
false
2024-02-29
```

**Why this output?**  
`2024y/December/25` creates a valid date; `February/30` is invalid (`.ok()` returns `false`); `February/last` yields the last day of February 2024, which is the 29th (leap year).

### Real-World Cases

- **Event scheduling** – representing dates for calendars.
- **Data validation** – ensuring dates are valid.
- **Financial applications** – date arithmetic for interest calculations.

---

## 10. Date and Time Best Practices

### Definitions

**Core Definition**
Date and time best practices are guidelines for using `<chrono>` correctly, safely, and efficiently, avoiding common pitfalls such as clock misuse, unit confusion, and time zone errors.

**Technical Definition**
These practices are informed by the C++ Core Guidelines, community consensus, and library documentation: use `steady_clock` for measuring intervals (never `system_clock`), use `system_clock` for wall-clock time and calendar dates, avoid `high_resolution_clock` in favour of explicit choices, prefer `duration_cast` for explicit conversions, and use C++20 calendar/time zone types over the C `<ctime>` API.

**Beginner-Friendly Explanation**
Best practices for time are simple: use the right clock for the job (`steady_clock` for timing, `system_clock` for dates), be explicit about units, and use the modern C++20 calendar and time zone features instead of old C functions.

### Key Guidelines

| Guideline | Summary |
|-----------|---------|
| **Use `steady_clock` for measuring intervals** | `system_clock` can jump, corrupting measurements. |
| **Use `system_clock` for wall-clock time** | Calendar dates, timestamps, logging. |
| **Avoid `high_resolution_clock`** | It is often an alias; use `steady_clock` or `system_clock` explicitly. |
| **Use `duration_cast` for unit conversion** | Makes lossy conversions visible. |
| **Prefer duration literals** | `500ms` is clearer than `milliseconds(500)`. |
| **Use C++20 calendars over `<ctime>`** | Type-safe, no global buffers, compiler catches mistakes. |
| **Use `zoned_time` for time zone conversions** | Handles DST and offsets automatically. |
| **Validate calendar dates with `.ok()`** | Catches invalid dates like February 30. |
| **Be aware of epoch differences** | `steady_clock` epoch is not meaningful for dates. |
| **Use `std::format` (C++20) for formatting** | Type-safe, locale-aware formatting. |

### Annotated Code Example

```cpp
#include <chrono>
#include <iostream>
#include <thread>

int main() {
    using namespace std::chrono;

    // Good: steady_clock for timing
    auto start = steady_clock::now();
    std::this_thread::sleep_for(50ms);
    auto elapsed = duration_cast<milliseconds>(steady_clock::now() - start);
    std::cout << "Elapsed: " << elapsed.count() << " ms\n";

    // Good: system_clock for calendar time
    auto now = system_clock::now();
    auto today = floor<days>(now);
    year_month_day ymd{today};
    std::cout << "Today: " << ymd << '\n';
}
```

**Expected Output** (example)
```
Elapsed: 50 ms
Today: 2026-11-12
```

**Why this output?**  
`steady_clock` measures the sleep accurately; `system_clock` and `year_month_day` provide the current calendar date.

### Real-World Cases

- **Code reviews** – checking for `system_clock` used for timing.
- **Profiling tools** – using `steady_clock` for reliable measurements.
- **International applications** – using `zoned_time` for local times.

---

## References

- Date and time library — cppreference – https://en.cppreference.com/w/cpp/chrono
- `std::chrono::duration` — cppreference – https://en.cppreference.com/w/cpp/chrono/duration
- `std::chrono::time_point` — cppreference – https://en.cppreference.com/w/cpp/chrono/time_point
- `std::chrono::system_clock` — cppreference – https://en.cppreference.com/w/cpp/chrono/system_clock
- `std::chrono::steady_clock` — cppreference – https://en.cppreference.com/w/cpp/chrono/steady_clock
- `std::chrono::high_resolution_clock` — cppreference – https://en.cppreference.com/w/cpp/chrono/high_resolution_clock
- Time zone database (libstdc++ manual) — GCC – https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/doc/html/manual/time.html
- `std::chrono::year_month_day` — cppreference – https://en.cppreference.com/w/cpp/chrono/year_month_day
- `std::chrono::zoned_time` — cppreference – https://en.cppreference.com/w/cpp/chrono/zoned_time
- Time in C++: `std::chrono::high_resolution_clock`: Myths and Realities — Sandor Dargo (ISO C++) – https://isocpp.org/blog/2026/01/time-in-cpp-stdchronohigh-resolution-clock-myths-and-realities-sandor-dargo
- Use `std::chrono` for time measurement and duration arithmetic — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/65967af19ad6654c18150f8bf4907ed128f756a7/09_Standard_Library_Utilities/Use_stdchrono_for_time_measurement_and_duration_arithmetic.md
- Use `std::chrono` calendar and timezone features (C++20) — Dr-Sergey/learn_cpp – https://github.com/Dr-Sergey/learn_cpp/blob/1de392465d3cbf56761a34f2b7283a79849e6bba/09_Standard_Library_Utilities/Use_stdstdchrono_calendar_and_timezone_features_Cpp20.md
- C++ Core Guidelines — https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

---

This cheat sheet provides a comprehensive, structured reference for C++ time and date utilities, covering `<chrono>`, durations, time points, clocks, the three standard clocks, C++20 time zones and calendars, and best practices, with annotated code examples, expected outputs, real-world cases, and authoritative references.