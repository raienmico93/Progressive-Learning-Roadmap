# Time and Data — Comprehensive Programming Cheat Sheet & Research

---

## Topic Overview

### Definitions

**Core Definition**
The HTML `<time>` and `<data>` elements are semantic containers that link human-readable content to machine-readable values. `<time>` represents a specific period in time — a date, a time, or a duration — and can carry a `datetime` attribute that provides a machine-readable equivalent. `<data>` links a given piece of content with a machine-readable translation via its required `value` attribute, and is used for content that is not time- or date-related.

**Technical Definition**
In the WHATWG HTML Living Standard, both elements belong to the **text-level semantics** chapter. `<time>` and `<data>` both belong to the **flow content**, **phrasing content**, and **palpable content** categories. `<time>` accepts the `datetime` attribute, which must contain a valid date, time, time-zone offset, or duration string; if the `datetime` attribute is absent, the element must not have any element descendants, and its child text content becomes the datetime value. `<data>` requires the `value` attribute, whose value must be a machine-readable representation of the element's contents. If the content is time- or date-related, `<time>` must be used instead of `<data>`.

**Beginner-Friendly Explanation**
`<time>` is for dates and times. It lets you show something like "November 6, 2026" to readers while giving browsers and search engines the machine-readable version `2026-11-06`. `<data>` is for any other kind of data that has a human-readable form and a machine-readable form — for example, a product name shown as "Mini Ketchup" with a product code of `398`. These elements make your content understandable to both humans and computers.

---

### Key Characteristics

- **Dual representation**: Both elements pair visible, human-readable content with a machine-readable value.
- **`<time>` handles temporal data**: dates, times, time-zone offsets, durations, weeks, and years.
- **`<data>` handles non-temporal data**: product codes, numeric identifiers, currency values, and other machine-readable translations.
- **Mutual exclusivity**: If the content is time- or date-related, `<time>` must be used; `<data>` must not be used for temporal data.
- **Attribute-driven machine readability**: `<time>` uses the optional `datetime` attribute; `<data>` uses the required `value` attribute.
- **Accessibility value**: Semantic markup helps screen readers identify temporal data and machine-readable values, which raw `<span>` or `<p>` elements cannot do.
- **SEO value**: Search engines and browser extensions can parse structured temporal and product data from these elements.

---

### Prerequisites

1. **Basic HTML document structure**: familiarity with elements, tags, and attributes.
2. **Phrasing vs. flow content**: understanding which elements can appear inside a paragraph.
3. **The concept of machine-readable data**: understanding why visible text alone is insufficient for computers.
4. **ISO 8601 date formats**: familiarity with `YYYY-MM-DD` and related date/time notations.

---

### Related Programming Areas

| Area | Relationship |
|---|---|
| **Accessibility (WCAG)** | Semantic temporal markup helps screen readers identify timestamps; raw spans fail this requirement. |
| **SEO / Structured data** | `<time datetime>` enables search engines to understand publication dates and event times. |
| **JavaScript / DOM** | `HTMLTimeElement.dateTime` and `HTMLDataElement.value` reflect the attributes for scripting. |
| **Microdata / RDFa** | Both elements are commonly used with `itemprop` for structured data. |

---

### Core Concepts / Features

The cheat sheet treats the following concepts, each with the same uniform structure:

1. **`<time>`** — the date/time element
2. **`<data>`** — the machine-readable data element
3. **`datetime` attribute** — the machine-readable time value
4. **`value` attribute** — the machine-readable data value
5. **Time and data best practices** — authoring guidelines

---

## 1. `<time>` — The Date/Time Element

### Definitions

**Core Definition**
The `<time>` element represents a specific period in time. It may include the `datetime` attribute to translate dates into a machine-readable format, allowing for better search-engine results or custom features such as reminders.

**Technical Definition**
`<time>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content** if the `datetime` attribute is present; otherwise, it must contain **text only** (no element descendants). The `datetime` attribute, when present, must be a valid date, time, time-zone offset, duration, week, or year string. The `dateTime` IDL attribute reflects the content attribute. The element should not be used for dates before the introduction of the Gregorian calendar.

**Beginner-Friendly Explanation**
`<time>` marks dates, times, and durations. You show readers something friendly like "July 7" or "20:00", and you put the machine-readable version in the `datetime` attribute so browsers and search engines can understand it.

---

### Purposes

- **To mark up dates and times machine-readably** for search engines and browsers.
- **To enable calendar and reminder features** that depend on parseable timestamps.
- **To provide structured temporal data** for microdata and RDFa vocabularies.
- **To improve accessibility** by allowing screen readers to identify temporal content.

---

### Syntax Rules and Structure

#### Complete General Syntax (with `datetime`)

```html
<time datetime="2026-11-06">6 November 2026</time>
```

#### Complete General Syntax (without `datetime`)

```html
<time>2026-11-06</time>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<time>` | The temporal wrapper. |
| `datetime` | Optional. A machine-readable date/time string. |
| Content | Human-readable text; must be text-only if `datetime` is absent. |

#### Valid `datetime` formats

| Type | Example |
|---|---|
| Valid month string | `2011-11` |
| Valid date string | `2011-11-18` |
| Valid yearless date string | `11-18` |
| Valid time string | `09:54`, `09:54:39`, `09:54:39.929` |
| Valid local date and time string | `2011-11-18T14:54`, `2011-11-18 14:54` |
| Valid time-zone offset string | `Z`, `+0000`, `+00:00`, `-0800`, `-08:00` |
| Valid global date and time string | `2011-11-18T14:54Z`, `2011-11-18T14:54+0000` |
| Valid week string | `2011-W47` |
| Valid duration string | `PT4H18M3S`, `4h 18m 3s` |
| Four or more ASCII digits | `2011`, `0001` |

#### Syntax Rules

1. **If `datetime` is absent, the element must not have element descendants**; the datetime value is the child text content.
2. **The `datetime` value must match one of the valid syntaxes** listed above.
3. **Use a floating date and time string (without time zone) for events that occur at the same local time in each time zone** (e.g. New Year celebrations).
4. **Use a global date and time string (with time zone) for events that occur at the same precise moment worldwide** (e.g. a videoconference).
5. **Do not use `<time>` for dates before the Gregorian calendar**.

#### Constraints and Limitations

- **The `pubdate` attribute is obsolete** and must not be used.
- **`<time>` must not be a descendant of another `<time>` element**.
- **Invalid `datetime` values produce no associated timestamp**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: A Human-Readable Date with Machine-Readable Value

**Setup Guide**: Save as `time-date.html` and inspect with a structured-data testing tool.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Time Example</title>
</head>
<body>
  <p>
    The event is on
    <time datetime="2026-11-06">6 November 2026</time>.
  </p>
</body>
</html>
```

**Expected output**: The text displays "6 November 2026", and the machine-readable value is `2026-11-06`.

**Why this output occurs**: The `<time>` element provides a machine-readable date via the `datetime` attribute, while the content displays a human-readable version. Search engines and scripts can parse the `datetime` value.

---

#### Example 2: A Duration and a Time

**Setup Guide**: Save as `time-duration.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Time and Duration</title>
</head>
<body>
  <p>
    The concert starts at <time datetime="20:00">20:00</time>
    and lasts for at least <time datetime="PT2H30M">2h 30m</time>.
  </p>
</body>
</html>
```

**Expected output**: "20:00" is marked as a time, and "2h 30m" is marked as a duration.

**Why this output occurs**: The `datetime` attribute accepts both time strings (`20:00`) and duration strings (`PT2H30M`), allowing machines to parse both values correctly.

---

#### Example 3: A Date Without `datetime`

**Setup Guide**: Save as `time-nodatetime.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Time Without Datetime</title>
</head>
<body>
  <p>Our first date was <time>2006-09-23</time>.</p>
</body>
</html>
```

**Expected output**: "2006-09-23" is marked as a machine-readable date.

**Why this output occurs**: The text content itself is a valid date string, so the `datetime` attribute is not required. The element contains only text, satisfying the no-element-descendants rule.

---

### Real-World Cases

1. **Blog posts**: Publication and update timestamps marked with `<time datetime>`.
2. **Event listings**: Start and end times with machine-readable values.
3. **Dashboards**: Relative timestamps like "5 minutes ago" backed by an ISO 8601 `datetime` value.

---

### References

- MDN Web Docs — `<time>`: The (Date) Time element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time
- W3C — HTML 5.2: The `time` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-time-element
- WHATWG — The `time` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element

---

## 2. `<data>` — The Machine-Readable Data Element

### Definitions

**Core Definition**
The `<data>` element links a given piece of content with a machine-readable translation. If the content is time- or date-related, the `<time>` element must be used instead.

**Technical Definition**
`<data>` belongs to the **flow content**, **phrasing content**, and **palpable content** categories. Its content model is **phrasing content**. The `value` attribute is **required**; its value must be a representation of the element's contents in a machine-readable format. The `value` IDL attribute reflects the content attribute. The element is intended for cases where the human-readable content is not already in a machine-readable format, and it can be used with Microdata or as a hook for scripts, similar to `data-*` attributes.

**Beginner-Friendly Explanation**
`<data>` pairs something people can read with something machines can read. For example, you display "Mini Ketchup" but store the product code `398`. If you were working with a date or time, you would use `<time>` instead.

---

### Purposes

- **To provide a machine-readable translation** for non-temporal human-readable content.
- **To support scripts** that need to read structured values from the page.
- **To avoid abusing `data-*` attributes** for content that has a visible human-readable representation.
- **To enable structured-data vocabularies** when combined with Microdata or RDFa.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<data value="398">Mini Ketchup</data>
```

**Component breakdown**

| Component | Description |
|---|---|
| `<data>` | The data wrapper. |
| `value` | **Required.** The machine-readable value. |
| Content | Human-readable content (phrasing content). |

#### Syntax Rules

1. **The `value` attribute is required** and must be a machine-readable representation of the content.
2. **Content model is phrasing content**.
3. **If the content is time- or date-related, `<time>` must be used instead**.
4. **The human-readable content and the machine-readable `value` should correspond** to the same thing.

#### Constraints and Limitations

- **`<data>` must not be used for temporal data**; use `<time>`.
- **The `value` attribute is not a substitute for `data-*` attributes** when there is no human-readable content.
- **Browser support is broad** (Baseline widely available since October 2017).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Names with Codes

**Setup Guide**: Save as `data-product.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Example</title>
</head>
<body>
  <p>New Products:</p>
  <ul>
    <li><data value="398">Mini Ketchup</data></li>
    <li><data value="399">Jumbo Ketchup</data></li>
    <li><data value="400">Mega Jumbo Ketchup</data></li>
  </ul>
</body>
</html>
```

**Expected output**: A list of product names, each with a machine-readable product code in the `value` attribute.

**Why this output occurs**: The `<data>` element stores the product code in `value` while displaying the human-readable name. Scripts can read `element.value` to obtain the code.

---

#### Example 2: A Numeric Value with a Human-Readable Label

**Setup Guide**: Save as `data-number.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Data Numeric</title>
</head>
<body>
  <p>
    The temperature is
    <data value="-5">minus five</data> degrees Celsius.
  </p>
</body>
</html>
```

**Expected output**: "minus five" is displayed, and the machine-readable value is `-5`.

**Why this output occurs**: The `<data>` element separates the human-readable display ("minus five") from the machine-readable numeric value (`-5`), allowing scripts to parse the number directly.

---

### Real-World Cases

1. **E-commerce**: Product codes alongside product names.
2. **Catalogs**: Unique identifiers for items.
3. **Dashboards**: Numeric values with human-friendly labels.

---

### References

- MDN Web Docs — `<data>`: The Data element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/data
- WHATWG — The `data` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-data-element
- WHATWG Blog — Now it's `<time>` for `<data>` — https://blog.whatwg.org/weekly-time-data

---

## 3. `datetime` Attribute — The Machine-Readable Time Value

### Definitions

**Core Definition**
The `datetime` attribute of the `<time>` element specifies the machine-readable form of the element's contents. It must be a valid date, time, time-zone offset, duration, week, or year string.

**Technical Definition**
The `datetime` attribute is optional. When present, its value must be a representation of the element's contents in a machine-readable format. When absent, the element must contain only text, and the child text content becomes the datetime value. The `dateTime` IDL attribute reflects the content attribute. Valid syntaxes are defined in the WHATWG specification and include month strings, date strings, yearless date strings, time strings, local and global date-and-time strings, time-zone offset strings, week strings, year strings, and duration strings.

**Beginner-Friendly Explanation**
The `datetime` attribute is where you put the machine-readable version of a date or time. If the text inside `<time>` is already in a standard format like `2026-11-06`, you can skip the attribute. If it is something like "next Friday", you need to provide the machine-readable version in `datetime`.

---

### Purposes

- **To provide a machine-readable equivalent** when the visible text is not in a standard format.
- **To enable search engines and scripts** to parse temporal data.
- **To support calendar and reminder features** that depend on parseable timestamps.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<time datetime="2026-11-06T14:54Z">November 6, 2026 at 2:54 PM UTC</time>
```

**Component breakdown**

| Component | Description |
|---|---|
| `datetime` | A valid date/time string. |
| Content | Human-readable text. |

#### Syntax Rules

1. **The value must match one of the valid syntaxes** (month, date, yearless date, time, local date and time, time-zone offset, global date and time, week, year, or duration).
2. **If `datetime` is absent, the element must contain only text**, and the text becomes the datetime value.
3. **Use floating date and time strings for events observed at the same local time in each time zone**.
4. **Use global date and time strings for events that occur at the same precise moment worldwide**.

#### Constraints and Limitations

- **Invalid `datetime` values produce no associated timestamp**.
- **The `pubdate` attribute is obsolete** and must not be used.
- **`<time>` must not be nested inside another `<time>`**.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Floating vs Global Date and Time

**Setup Guide**: Save as `datetime-formats.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Datetime Formats</title>
</head>
<body>
  <!-- Floating: same local time in each time zone -->
  <p>New Year: <time datetime="2027-01-01T00:00">2027-01-01 00:00</time></p>

  <!-- Global: same precise moment worldwide -->
  <p>Meeting: <time datetime="2027-01-01T14:00Z">2027-01-01 14:00 UTC</time></p>
</body>
</html>
```

**Expected output**: Both times are marked with valid `datetime` values. The floating time represents local midnight in each time zone; the global time represents a single moment in UTC.

**Why this output occurs**: The first `datetime` lacks a time-zone offset (floating), meaning the event is observed at that local time in each time zone. The second includes `Z` (UTC), meaning a single global moment.

---

#### Example 2: Duration String

**Setup Guide**: Save as `datetime-duration.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Duration</title>
</head>
<body>
  <p>The film runs for <time datetime="PT2H30M">2 hours and 30 minutes</time>.</p>
</body>
</html>
```

**Expected output**: "2 hours and 30 minutes" is marked as a duration of 2 hours and 30 minutes.

**Why this output occurs**: The `datetime` attribute accepts duration strings such as `PT2H30M`, allowing machines to parse the duration correctly.

---

### Real-World Cases

1. **Event pages**: Floating times for local events; global times for international events.
2. **Media metadata**: Durations of films, songs, and videos.
3. **Logs and timestamps**: Precise global date and time strings.

---

### References

- W3C — HTML 5.2: The `time` element (datetime syntaxes) — https://www.w3.org/TR/html52/textlevel-semantics.html#the-time-element
- MDN Web Docs — `<time>`: Valid datetime values — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time#valid_datetime_values
- Stack Overflow — Accessibility: datetime attribute in time element — https://stackoverflow.com/questions/22545621/accessibility-datetime-attribute-in-time-element

---

## 4. `value` Attribute — The Machine-Readable Data Value

### Definitions

**Core Definition**
The `value` attribute of the `<data>` element specifies the machine-readable translation of the element's contents. It is required.

**Technical Definition**
The `value` attribute is a required content attribute on `<data>`. Its value must be a representation of the element's contents in a machine-readable format. The `value` IDL attribute reflects the content attribute. The element represents its contents along with the machine-readable form in the `value` attribute.

**Beginner-Friendly Explanation**
The `value` attribute is where you put the machine-readable version of the data. If you display "Mini Ketchup", you put `398` in the `value` attribute.

---

### Purposes

- **To provide the machine-readable form** of human-readable content.
- **To enable scripts and structured-data parsers** to read the value directly.
- **To separate display text from data** without using non-standard attributes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```html
<data value="398">Mini Ketchup</data>
```

**Component breakdown**

| Component | Description |
|---|---|
| `value` | **Required.** The machine-readable value. |
| Content | Human-readable content. |

#### Syntax Rules

1. **The `value` attribute is required**.
2. **The value must be a machine-readable representation** of the element's contents.
3. **The human-readable content and the `value` should correspond** to the same thing.

#### Constraints and Limitations

- **`<data>` must not be used for time- or date-related content**; use `<time>` with `datetime`.
- **The `value` attribute is not a replacement for `data-*` attributes** when there is no visible human-readable content.

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Product Code

**Setup Guide**: Save as `value-product.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Value Attribute</title>
</head>
<body>
  <ul>
    <li><data value="398">Mini Ketchup</data></li>
    <li><data value="399">Jumbo Ketchup</data></li>
  </ul>
</body>
</html>
```

**Expected output**: The product names are displayed, and the `value` attributes hold the machine-readable product codes.

**Why this output occurs**: The `value` attribute provides the machine-readable translation, while the content provides the human-readable display.

---

#### Example 2: Numeric Value

**Setup Guide**: Save as `value-numeric.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Numeric Value</title>
</head>
<body>
  <p>Rating: <data value="4.5">four and a half stars</data></p>
</body>
</html>
```

**Expected output**: "four and a half stars" is displayed, and the machine-readable value is `4.5`.

**Why this output occurs**: The `value` attribute allows scripts to parse the numeric rating directly, while the content provides a human-friendly label.

---

### Real-World Cases

1. **Product catalogs**: Product codes and identifiers.
2. **Ratings and reviews**: Numeric ratings with human-readable labels.
3. **Financial data**: Currency values with formatted display text.

---

### References

- WHATWG — The `data` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-data-element
- MDN Web Docs — `<data>`: The Data element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/data
- W3C — HTML 5.2: The `data` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-data-element

---

## 5. Time and Data Best Practices

### Definitions

**Core Definition**
Time and data best practices are guidelines for using `<time>` and `<data>` semantically, accessibly, and correctly — choosing the right element for temporal versus non-temporal content, providing machine-readable values, and ensuring that visible text and machine-readable data correspond.

**Technical Definition**
Best practices are informed by the WHATWG HTML Living Standard, WCAG accessibility guidelines, and structured-data guidance. They encompass using `<time>` for all temporal data, using `<data>` only for non-temporal content, providing `datetime` when the visible text is not a standard date format, ensuring `value` corresponds to the content, and avoiding obsolete attributes such as `pubdate`.

**Beginner-Friendly Explanation**
Best practices help you use these elements correctly: use `<time>` for dates and times, use `<data>` for everything else, make sure the machine-readable value matches what users see, and avoid old attributes that no longer work.

---

### Purposes

- **To ensure correct element selection**: `<time>` for temporal data, `<data>` for non-temporal data.
- **To maximise machine readability**: provide valid `datetime` and `value` attributes.
- **To improve accessibility**: help screen readers identify temporal data and machine-readable values.
- **To support SEO and structured data**: enable search engines to parse dates and values.

---

### Syntax Rules and Structure

| Practice | Rule |
|---|---|
| **Use `<time>` for all temporal data** | Dates, times, durations, weeks, and years. |
| **Use `<data>` only for non-temporal data** | Product codes, numeric values, and other machine-readable translations. |
| **Provide `datetime` when the visible text is not a standard date format** | e.g. "next Friday" → `datetime="2026-11-13"`. |
| **Provide `value` on `<data>`** | The attribute is required. |
| **Ensure visible text and machine-readable value correspond** | The human-readable content and the `value` or `datetime` should describe the same thing. |
| **Use floating times for local events** | Omit the time-zone offset when the event occurs at the same local time in each time zone. |
| **Use global times for worldwide events** | Include `Z` or a time-zone offset for events at a single precise moment. |
| **Do not use `pubdate`** | It is obsolete. |
| **Do not nest `<time>` inside `<time>`** | The element must not appear as a descendant of another `<time>`. |

#### Syntax Rules

1. **Semantic correctness**: `<time>` for temporal data, `<data>` for non-temporal data.
2. **Machine readability**: always provide valid `datetime` or `value` attributes.
3. **Correspondence**: the human-readable content and the machine-readable value must describe the same thing.

#### Constraints and Limitations

- **Invalid `datetime` values produce no timestamp**.
- **`<data>` must not be used for temporal content**.
- **`<time>` without `datetime` must contain only text** (no element descendants).

---

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Correct Use of `<time>` and `<data>`

**Setup Guide**: Save as `best-practice-time-data.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Time and Data Best Practices</title>
</head>
<body>
  <!-- GOOD: time for a date -->
  <p>Published on <time datetime="2026-11-06">6 November 2026</time>.</p>

  <!-- GOOD: data for a product code -->
  <p><data value="398">Mini Ketchup</data></p>

  <!-- BAD: using data for a date -->
  <!-- <p><data value="2026-11-06">6 November 2026</data></p> -->

  <!-- BAD: time without datetime for non-standard text -->
  <!-- <p><time>next Friday</time></p> -->
</body>
</html>
```

**Expected output**: The good examples correctly mark up a date with `<time>` and a product code with `<data>`. The bad examples misuse the elements, losing machine readability and violating the specification.

**Why this output occurs**: `<time>` is the correct element for temporal data, and `<data>` is the correct element for non-temporal machine-readable translations. Using the wrong element or omitting the required attribute breaks machine readability.

---

#### Example 2: Corresponding Display and Machine Values

**Setup Guide**: Save as `best-practice-correspondence.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Correspondence</title>
</head>
<body>
  <!-- GOOD: visible text and datetime correspond -->
  <p>Event date: <time datetime="2026-11-06">6 November 2026</time></p>

  <!-- GOOD: visible text and value correspond -->
  <p>Rating: <data value="4.5">four and a half stars</data></p>

  <!-- BAD: visible text and datetime do not correspond -->
  <!-- <p>Event date: <time datetime="2026-11-06">December 25</time></p> -->
</body>
</html>
```

**Expected output**: The good examples have consistent human-readable and machine-readable values. The bad example has a mismatch that confuses users and machines.

**Why this output occurs**: Best practice requires that the visible text and the machine-readable value describe the same thing. A mismatch undermines trust and breaks structured-data parsing.

---

### Real-World Cases

1. **News and publishing**: `<time datetime>` for publication dates; `<data value>` for article IDs.
2. **E-commerce**: `<data value>` for product codes; `<time datetime>` for sale end dates.
3. **Dashboards**: `<time datetime>` for timestamps; `<data value>` for numeric KPIs.

---

### References

- MDN Web Docs — `<time>`: Usage notes — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/time#usage_notes
- MDN Web Docs — `<data>`: The Data element — https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/data
- W3C — HTML 5.2: The `time` element — https://www.w3.org/TR/html52/textlevel-semantics.html#the-time-element
- WHATWG — The `data` element — https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-data-element
- GitHub — Proposal: Semantic time elements — https://github.com/hivemoot/colony/issues/61

---

## Summary Table of Time and Data Elements

| Element | Primary Role | Required Attribute | Content Model | When to Use |
|---|---|---|---|---|
| `<time>` | Machine-readable date/time | `datetime` (optional) | Phrasing content (or text only if no `datetime`) | Dates, times, durations, weeks, years. |
| `<data>` | Machine-readable data | `value` (required) | Phrasing content | Product codes, numeric values, non-temporal translations. |

---

## Final Notes

- **`<time>` is for temporal data, `<data>` is for everything else**: If the content is time- or date-related, `<time>` must be used; `<data>` is for non-temporal machine-readable translations.
- **The `datetime` attribute is optional but recommended**: If the visible text is already a valid date string, `datetime` can be omitted; otherwise, it is required for machine readability.
- **The `value` attribute is mandatory on `<data>`**: There is no fallback to text content.
- **Correspondence matters**: The human-readable content and the machine-readable value must describe the same thing.
- **Avoid obsolete attributes**: `pubdate` on `<time>` must not be used.
- **Validate and test**: Use HTML validators and structured-data testing tools to confirm that dates and values are parseable.

---

This cheat sheet has covered the `<time>` and `<data>` elements, the `datetime` and `value` attributes, and time and data best practices — with uniform definitions, purposes, syntax rules, annotated examples, real-world cases, and authoritative references. Use it as a reference and a teaching aid, and validate markup against the WHATWG HTML Living Standard and accessibility guidelines.