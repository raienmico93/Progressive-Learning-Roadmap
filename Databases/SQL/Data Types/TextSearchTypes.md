# Comprehensive Programming Cheat Sheet: SQL Text Search Types

## Topic Overview

### Definitions

**Core Definition**
Text search types are the PostgreSQL data types—`TSVECTOR` and `TSQUERY`—designed to support full‑text search, the activity of searching through a collection of natural‑language documents to locate those that best match a query. The `tsvector` type represents a document in a form optimised for text search; the `tsquery` type similarly represents a text query. A `tsvector` value is a sorted list of distinct lexemes—words normalised to merge different variants of the same word—with sorting and duplicate elimination performed automatically during input. A `tsquery` value stores lexemes to be searched for and combines them using Boolean operators (`&` for AND, `|` for OR, `!` for NOT) as well as the phrase‑search operator `<->` (FOLLOWED BY) and its variant `<N>`, which specifies the distance between two lexemes.

**Technical Definition**
PostgreSQL provides two data types designed to support full‑text search: `tsvector` and `tsquery`. The `tsvector` type represents a document in a form optimised for text search, while the `tsquery` type represents a text query. A `tsvector` is a sorted list of distinct lexemes, where lexemes are words normalised to merge different variants of the same word; sorting and duplicate elimination are performed automatically during input. Lexemes may carry integer positions (ranging from 1 to 16383) indicating the source word's location, which enables proximity ranking, and they may further be labelled with weights `A`, `B`, `C`, or `D`, where `D` is the default and is not shown on output. A `tsquery` stores lexemes combined with Boolean operators `&` (AND), `|` (OR), and `!` (NOT), and the phrase‑search operator `<->` (FOLLOWED BY); in the absence of parentheses, `!` binds most tightly, `<->` next most tightly, then `&`, with `|` binding least tightly. Lexemes in a `tsquery` may be restricted to specific weights (e.g., `'fat':ab & cat'`) or may use prefix matching with `*` (e.g., `'super:*'`).

**Beginner-Friendly Explanation**
Full‑text search is how a database finds documents that are relevant to a query, not just rows that contain an exact string. PostgreSQL gives you two specialised containers: `TSVECTOR` is the processed, search‑ready version of a document—a tidy list of normalised words with their positions and optional importance labels—and `TSQUERY` is the processed, search‑ready version of a query, with Boolean logic and phrase matching. Think of `TSVECTOR` as an index card for each document, listing every meaningful word and where it appears, and `TSQUERY` as the search instruction that tells PostgreSQL which words to look for and how they must be combined.

### Key Characteristics

- **Lexeme‑based matching**: Both types operate on lexemes—normalised word forms—rather than raw text. This means a search for “friend” can match “friends” and “friendly” because all are reduced to the same lexeme.
- **Positional information enables ranking**: `tsvector` lexemes can carry positions (1–16383) that record where each word appeared in the document; these positions support proximity ranking and phrase queries.
- **Weights support relevance boosting**: Lexemes can be labelled `A`, `B`, `C`, or `D`, allowing titles to be ranked above body text. Weighting is typically applied with `setweight()` before concatenation.
- **Boolean and phrase operators**: `tsquery` supports `&` (AND), `|` (OR), `!` (NOT), and `<->` (FOLLOWED BY), with `<N>` allowing a specified distance between lexemes.
- **GIN indexes are the preferred acceleration structure**: GIN indexes store only the words (lexemes) of `tsvector` values, not their weight labels, and are the preferred text search index type; GiST indexes offer smaller size and faster updates but are lossy and may require row rechecks.

### Prerequisites

- A running PostgreSQL instance (text search types have been available since PostgreSQL 8.3; the features described here are current through PostgreSQL 16/18 documentation).
- Familiarity with `CREATE TABLE` syntax and SQL data types.
- Understanding of the difference between raw text and normalised lexemes.
- For indexing: awareness of GIN vs. GiST trade‑offs and the two‑argument form of `to_tsvector()` required for expression indexes.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations, generated columns, and expression indexes.
- Data Manipulation Language (DML) – inserting and updating `tsvector` values, querying with `@@`.
- Information retrieval – relevance ranking (`ts_rank`, `ts_rank_cd`), highlighting (`ts_headline`), and query rewriting (`ts_rewrite`).
- Indexing and query optimisation – GIN and GiST indexes, `maintenance_work_mem`, and expression indexes.
- Natural‑language processing – parsers, dictionaries, stemming, stop words, and custom text search configurations.

---

## Core Concepts and Features

### 1. `TSVECTOR`

#### Definitions

**Core Definition**
`TSVECTOR` is a PostgreSQL data type that represents a document in a form optimised for text search: a sorted list of distinct lexemes, optionally annotated with positions and weights.

**Technical Definition**
A `tsvector` value is a sorted list of distinct lexemes—words that have been normalised to merge different variants of the same word—with sorting and duplicate elimination performed automatically during input. Lexemes may optionally carry integer positions, normally indicating the source word's location in the document, with valid positions ranging from 1 to 16383; larger numbers are silently set to 16383, and duplicate positions for the same lexeme are discarded. Lexemes with positions may further be labelled with a weight (`A`, `B`, `C`, or `D`), where `D` is the default and is not shown on output; weights are typically used to reflect document structure, such as marking title words differently from body words. Critically, the `tsvector` type itself does **not** perform any word normalisation—it assumes the words given to it are already normalised appropriately for the application—so raw document text should usually be passed through `to_tsvector()` to normalise words appropriately.

**Beginner-Friendly Explanation**
`TSVECTOR` is the “search‑ready” version of a document. If you give raw text to `to_tsvector()`, PostgreSQL breaks it into words, reduces each word to its root form, removes stop words, and produces a compact list of the remaining words with their positions and optional importance labels. That list is what the database searches against. You can also build a `tsvector` by hand, but then you are responsible for normalising the words yourself.

#### Purposes

- To store a document in a form that can be searched efficiently with `tsquery`.
- To capture word positions for proximity ranking and phrase queries.
- To attach weights to words so that titles and headings can outrank body text.
- To serve as the target of a GIN or GiST index for fast full‑text search.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name TSVECTOR
```

**Literal form (raw, unnormalised):**

```sql
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector;
```

**Normalised form via `to_tsvector()`:**

```sql
SELECT to_tsvector('english', 'The Fat Rats');
```

**Lexemes with positions:**

```sql
SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:9 a:10 fat:11 rat:12'::tsvector;
```

**Lexemes with weights:**

```sql
SELECT 'a:1A fat:2B,4C cat:5D'::tsvector;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Lexeme | A normalised word, surrounded by quotes if it contains whitespace or punctuation. |
| Position | An integer (1–16383) indicating the source word's location. |
| Weight | One of `A`, `B`, `C`, or `D`; `D` is the default and is omitted on output. |
| `to_tsvector(config, text)` | Converts raw text to a normalised `tsvector` using the named configuration. |

**Syntax rules:**

- Positions are optional; without positions, the `tsvector` cannot be used for proximity ranking or phrase queries.
- Weights are optional and require positions.
- Duplicate positions for the same lexeme are discarded.
- Raw `tsvector` literals do not normalise words; use `to_tsvector()` for raw text.

**Constraints and limitations:**

- The `tsvector` type does not perform stemming, stop‑word removal, or any other normalisation on its own.
- Position values larger than 16383 are silently set to 16383.
- GIN indexes store only lexemes, not weights, so weight‑filtered queries may require row rechecks.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating a normalised `tsvector` with positions and weights

```sql
-- Step 1: Convert raw text to a tsvector using the English configuration.
SELECT to_tsvector('english', 'The Fat Rats') AS doc;

-- Step 2: Build a tsvector with explicit positions.
SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:9 a:10 fat:11 rat:12'::tsvector AS positioned;

-- Step 3: Attach weights to selected lexemes.
SELECT setweight(to_tsvector('english', 'The Fat Rats'), 'A') AS weighted;
```

**Expected output:**

```text
         doc
---------------------
 'fat':2 'rat':3
(1 row)

                          positioned
---------------------------------------------------------------
 'a':1,6,10 'and':8 'ate':9 'cat':3 'fat':2,11 'mat':7 'on':5 'rat':12 'sat':4
(1 row)

        weighted
-------------------------
 'fat':2A 'rat':3A
(1 row)
```

**Why this output appears**: `to_tsvector('english', 'The Fat Rats')` normalises “The” (a stop word, removed), “Fat” → `fat`, and “Rats” → `rat`, producing a sorted list with positions 2 and 3. The explicit literal preserves all lexemes and aggregates duplicate positions for `'a'` into `1,6,10`. `setweight(..., 'A')` labels each lexeme with weight `A`, which is shown on output because it is not the default `D`.

#### Real-World Cases

- **Article search**: A `documents` table stores a `tsvector` column generated from the title and body, enabling fast full‑text search with ranking.
- **Product catalogues**: A `products` table stores a weighted `tsvector` where the product name receives weight `A` and the description receives weight `B`, so name matches rank higher than description matches.
- **Log analysis**: A `logs` table stores a `tsvector` of each log message, enabling searches for error patterns across millions of entries.

---

### 2. `TSQUERY`

#### Definitions

**Core Definition**
`TSQUERY` is a PostgreSQL data type that represents a text query as a combination of lexemes with Boolean operators, phrase operators, weight restrictions, and prefix matching.

**Technical Definition**
A `tsquery` value stores lexemes that are to be searched for and combines them using the Boolean operators `&` (AND), `|` (OR), and `!` (NOT), as well as the phrase‑search operator `<->` (FOLLOWED BY). There is also a variant `<N>` of the FOLLOWED BY operator, where `N` is an integer constant specifying the distance between the two lexemes being searched for; `<->` is equivalent to `<1>`. Parentheses can be used to enforce grouping, and in the absence of parentheses, `!` binds most tightly, `<->` next most tightly, then `&`, with `|` binding least tightly. Lexemes in a `tsquery` can be labelled with one or more weight letters, restricting them to match only `tsvector` lexemes with one of those weights, and they can be labelled with `*` to specify prefix matching. As with `tsvector`, any required normalisation of words must be done before converting to the `tsquery` type; the `to_tsquery` function is convenient for performing such normalisation.

**Beginner-Friendly Explanation**
`TSQUERY` is the search instruction. It says which words to look for and how they must be combined: “this AND that,” “this OR that,” “this but NOT that,” or “this immediately followed by that.” You can also say “only match this word if it has weight A” or “match any word starting with these letters.” Just like `TSVECTOR`, `TSQUERY` expects normalised words, so you usually build it with `to_tsquery()`, `plainto_tsquery()`, `phraseto_tsquery()`, or `websearch_to_tsquery()`.

#### Purposes

- To express complex search conditions with Boolean logic and phrase matching.
- To restrict matches to specific weights (e.g., only title words).
- To support prefix matching for type‑ahead or partial‑word search.
- To serve as the query operand of the `@@` match operator against a `tsvector`.

#### Syntax Rules and Structure

**Literal form:**

```sql
SELECT 'fat & rat'::tsquery;
SELECT 'fat & (rat | cat)'::tsquery;
SELECT 'fat & rat & ! cat'::tsquery;
```

**Weight‑restricted lexemes:**

```sql
SELECT 'fat:ab & cat'::tsquery;
```

**Prefix matching:**

```sql
SELECT 'super:*'::tsquery;
```

**Query‑building functions:**

```sql
SELECT to_tsquery('english', 'The & Fat & Rats');
SELECT plainto_tsquery('english', 'The Fat Rats');
SELECT phraseto_tsquery('english', 'The Fat Rats');
SELECT websearch_to_tsquery('english', '"fat rat" or cat dog');
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `&` | AND: both operands must match. |
| `\|` | OR: either operand must match. |
| `!` | NOT: the operand must not match. |
| `<->` / `<N>` | FOLLOWED BY: operands must appear at successive lexemes, or exactly `N` lexemes apart. |
| `:AB` | Restricts the lexeme to weights `A` and `B`. |
| `:*` | Prefix matching: matches any word beginning with the given stem. |

**Syntax rules:**

- Operator precedence: `!` > `<->` > `&` > `|`.
- The two‑argument form of `to_tsquery` normalises words using the named configuration.
- `plainto_tsquery` converts plain text to an AND query, ignoring punctuation.
- `phraseto_tsquery` inserts `<->` between surviving words instead of `&`.
- `websearch_to_tsquery` understands quoted phrases, `or`, and `-` (NOT), approximating web search syntax.

**Constraints and limitations:**

- Raw `tsquery` literals do not normalise words; use the query‑building functions for user input.
- `to_tsquery` is strict about operators; `plainto_tsquery` is more forgiving.
- Prefix matching with `:*` applies after stemming, so `'postgres:*'` matches the stemmed form of “postgraduate”.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Building and using `tsquery` values

```sql
-- Step 1: Build a Boolean tsquery.
SELECT 'fat & rat'::tsquery AS q1,
       'fat & (rat | cat)'::tsquery AS q2,
       'fat & rat & ! cat'::tsquery AS q3;

-- Step 2: Build a phrase query with distance.
SELECT 'fat' <-> 'rat'::tsquery AS phrase,
       'fat' <3> 'rat'::tsquery AS distance_3;

-- Step 3: Build a weight-restricted query.
SELECT 'fat:ab & cat'::tsquery AS weighted_query;

-- Step 4: Build a prefix query.
SELECT 'super:*'::tsquery AS prefix_query;

-- Step 5: Use query-building functions.
SELECT to_tsquery('english', 'The & Fat & Rats') AS to_query,
       plainto_tsquery('english', 'The Fat Rats') AS plain_query,
       phraseto_tsquery('english', 'The Fat Rats') AS phrase_query,
       websearch_to_tsquery('english', '"fat rat" or cat dog') AS web_query;
```

**Expected output:**

```text
 q1        | q2              | q3
-----------+-----------------+----------------
 'fat' & 'rat' | 'fat' & ( 'rat' | 'cat' ) | 'fat' & 'rat' & !'cat'
(1 row)

 phrase        | distance_3
---------------+-----------------
 'fat' <-> 'rat' | 'fat' <3> 'rat'
(1 row)

 weighted_query
----------------
 'fat':AB & 'cat'
(1 row)

 prefix_query
--------------
 'super':*
(1 row)

 to_query        | plain_query   | phrase_query     | web_query
-----------------+---------------+------------------+--------------------------
 'fat' & 'rat'   | 'fat' & 'rat' | 'fat' <-> 'rat'  | 'fat' <-> 'rat' | 'cat' & 'dog'
(1 row)
```

**Why this output appears**: The literal `tsquery` values preserve the operators as written. `to_tsquery` normalises “The” (stop word, removed) and produces `'fat' & 'rat'`. `plainto_tsquery` also produces an AND query. `phraseto_tsquery` uses `<->` to require adjacency. `websearch_to_tsquery` interprets the quoted phrase `"fat rat"` as a phrase query and the word “or” as OR, producing a combined query.

#### Real-World Cases

- **Search boxes**: A user types “fat rat” into a search box; `plainto_tsquery` converts it to `'fat' & 'rat'`, matching documents containing both words anywhere.
- **Phrase search**: A user types `"full text search"` into a quoted search; `phraseto_tsquery` or `websearch_to_tsquery` produces a phrase query that matches only when the words appear adjacent and in order.
- **Weight‑aware search**: A search application restricts matches to title words by building `tsquery` values with weight labels (e.g., `'postgres':A`) and combining them with `@@` against a weighted `tsvector`.

---

### 3. Text Search Operations

#### Definitions

**Core Definition**
Text search operations are the operators and functions that match `tsvector` values against `tsquery` values, combine and modify vectors and queries, rank results, highlight matches, and rewrite queries.

**Technical Definition**
The central operator is `@@`, which returns `true` if a `tsvector` matches a `tsquery`; the arguments can be given in either order, and a `text` left operand is implicitly converted to `tsvector`. The `||` operator concatenates two `tsvector` values, adjusting the second input's positions accordingly. The `&&` and `||` operators combine `tsquery` values with AND and OR respectively, and `!!` negates a query. The `<->` operator constructs a phrase query, and `@>` and `<@` test whether one query contains or is contained in another (considering only lexemes, ignoring operators). Functions include `to_tsvector`, `to_tsquery`, `plainto_tsquery`, `phraseto_tsquery`, `websearch_to_tsquery`, `setweight`, `strip`, `length`, `numnode`, `querytree`, `ts_rank`, `ts_rank_cd`, `ts_headline`, `ts_rewrite`, `ts_delete`, `ts_filter`, `tsvector_to_array`, and `unnest`.

**Beginner-Friendly Explanation**
The `@@` operator is the heart of full‑text search: it asks “does this document match this query?”. Around it, PostgreSQL provides tools to combine documents (`||`), combine queries (`&&`, `||`, `!!`), build phrases (`<->`), rank results by relevance (`ts_rank`, `ts_rank_cd`), show snippets with matches highlighted (`ts_headline`), and clean up vectors or queries (`strip`, `ts_delete`, `ts_filter`).

#### Purposes

- To determine whether a document matches a query using `@@`.
- To combine multiple documents into a single searchable vector with `||`.
- To combine or negate queries with `&&`, `||`, and `!!`.
- To build phrase queries with `<->` and its distance variant.
- To rank results by relevance using `ts_rank` or `ts_rank_cd`.
- To highlight matched terms in displayed results with `ts_headline`.
- To rewrite queries for synonym expansion or query relaxation with `ts_rewrite`.

#### Syntax Rules and Structure

**Match operator:**

```sql
tsvector @@ tsquery → boolean
tsquery @@ tsvector → boolean
```

**Vector concatenation:**

```sql
tsvector || tsvector → tsvector
```

**Query combination:**

```sql
tsquery && tsquery → tsquery   -- AND
tsquery || tsquery → tsquery   -- OR
!! tsquery → tsquery           -- NOT
```

**Phrase query:**

```sql
tsquery <-> tsquery → tsquery
tsquery <N> tsquery → tsquery
```

**Query containment:**

```sql
tsquery @> tsquery → boolean
tsquery <@ tsquery → boolean
```

**Ranking:**

```sql
ts_rank([weights real[],] vector tsvector, query tsquery [, normalization integer]) → real
ts_rank_cd([weights real[],] vector tsvector, query tsquery [, normalization integer]) → real
```

**Highlighting:**

```sql
ts_headline([config regconfig,] document text, query tsquery [, options text]) → text
```

**Component breakdown:**

| Operator / Function | Description |
|---------------------|-------------|
| `@@` | Match: document matches query. |
| `\|\|` (vectors) | Concatenate documents, adjusting positions. |
| `&&`, `\|\|`, `!!` (queries) | Combine queries with AND, OR, NOT. |
| `<->`, `<N>` | Phrase query with adjacency or distance. |
| `@>`, `<@` (queries) | Query containment (lexeme‑only). |
| `ts_rank` | Relevance score based on frequency. |
| `ts_rank_cd` | Relevance score using cover density. |
| `ts_headline` | Abbreviated document with matched terms highlighted. |

**Syntax rules:**

- The `@@` operator accepts arguments in either order; a `text` left operand is implicitly converted with `to_tsvector`.
- The `@@@` operator is a deprecated synonym for `@@` and should not be used in new code.
- Query containment operators consider only whether all lexemes in one query appear in the other, ignoring the combining operators.
- `ts_rank` and `ts_rank_cd` accept optional weight arrays and normalization integers.

**Constraints and limitations:**

- `@@` requires `tsvector` and `tsquery` operands; raw text on the left is implicitly converted, but raw text on the right is not.
- `ts_rank` and `ts_rank_cd` can be slow on very large result sets; consider limiting candidates with `@@` first.
- `ts_headline` operates on raw text, not on `tsvector` values.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Matching, combining, and ranking

```sql
-- Step 1: Match a tsvector against a tsquery.
SELECT to_tsvector('fat cats ate rats') @@ to_tsquery('cat & rat') AS matches;

-- Step 2: Combine two documents.
SELECT 'a:1 b:2'::tsvector || 'c:1 d:2 b:3'::tsvector AS combined;

-- Step 3: Combine two queries with AND.
SELECT 'fat | rat'::tsquery && 'cat'::tsquery AS and_query;

-- Step 4: Build a phrase query.
SELECT to_tsquery('fat') <-> to_tsquery('rat') AS phrase_query;

-- Step 5: Rank a document against a query.
SELECT ts_rank(to_tsvector('raining cats and dogs'), 'cat') AS rank;

-- Step 6: Highlight matches in a document.
SELECT ts_headline('The fat cat ate the rat.', 'cat') AS headline;
```

**Expected output:**

```text
 matches
---------
 t
(1 row)

          combined
----------------------------
 'a':1 'b':2,5 'c':3 'd':4
(1 row)

    and_query
-----------------
 ( 'fat' | 'rat' ) & 'cat'
(1 row)

 phrase_query
--------------
 'fat' <-> 'rat'
(1 row)

   rank
----------
 0.06079271
(1 row)

           headline
-------------------------------
 The fat <b>cat</b> ate the rat.
(1 row)
```

**Why this output appears**: `@@` returns `true` because the document contains both `cat` and `rat`. The `||` concatenation adjusts the positions of the second vector so that `b` appears at positions 2 and 5. The `&&` operator combines the two queries into a single AND query. The `<->` operator builds a phrase query requiring `fat` immediately followed by `rat`. `ts_rank` computes a relevance score, and `ts_headline` wraps the matched word `cat` in `<b>` tags.

#### Real-World Cases

- **Search result ranking**: A web application uses `@@` to filter matching documents and `ts_rank` to order results by relevance.
- **Snippet generation**: `ts_headline` produces a short excerpt with matched terms highlighted for display in search results.
- **Synonym expansion**: `ts_rewrite` replaces query terms with a list of synonyms before matching.

---

### 4. Text Search Best Practices

#### Definitions

**Core Definition**
Text search best practices are the guidelines for designing, indexing, configuring, and querying full‑text search in PostgreSQL to achieve accurate, fast, and maintainable search behaviour.

**Technical Definition**
Best practices encompass: (1) **use GIN indexes for static or read‑heavy data**, because GIN index lookups are about three times faster than GiST, while GiST is preferred for dynamic data because it updates faster and builds smaller indexes; (2) **use the two‑argument form of `to_tsvector` in expression indexes**, because only configuration‑aware functions can be used in expression indexes—otherwise the index contents could be inconsistent with the query configuration; (3) **consider a stored generated column for `tsvector`** to avoid re‑computing `to_tsvector` at query time and to simplify queries, at the cost of additional storage; (4) **use `setweight` to boost titles over bodies**, assigning weight `A` to titles and `B` to descriptions, and pass custom weight arrays to `ts_rank`; (5) **customise the text search configuration** by copying the built‑in configuration and adding domain‑specific dictionaries, placing the most specific dictionaries first and the most general last; and (6) **choose the right query‑building function**—`plainto_tsquery` for simple user input, `phraseto_tsquery` for phrase search, `websearch_to_tsquery` for web‑style syntax, and `to_tsquery` only when the input is already in valid `tsquery` syntax.

**Beginner-Friendly Explanation**
Best practices for full‑text search are about making searches fast and relevant. GIN indexes make lookups fast but are slower to update; GiST indexes are better for data that changes constantly. If you index `to_tsvector`, always use the two‑argument form so the index and the query use the same configuration. Give titles more weight than body text so title matches rank higher. And when building queries from user input, use the friendly functions (`plainto_tsquery`, `phraseto_tsquery`, `websearch_to_tsquery`) rather than the strict `to_tsquery`.

#### Purposes

- To achieve sub‑second search response times on large document collections.
- To ensure that indexed and queried `tsvector` values use the same configuration.
- To rank results so that the most relevant documents appear first.
- To support domain‑specific vocabulary and synonyms.
- To keep search behaviour predictable and maintainable as data changes.

#### Syntax Rules and Structure

**GIN index on an expression:**

```sql
CREATE INDEX pgweb_idx ON pgweb USING GIN (to_tsvector('english', body));
```

**Stored generated column:**

```sql
ALTER TABLE pgweb
ADD COLUMN textsearchable_index_col tsvector
GENERATED ALWAYS AS (to_tsvector('english', coalesce(title, '') || ' ' || coalesce(body, ''))) STORED;
CREATE INDEX textsearch_idx ON pgweb USING GIN (textsearchable_index_col);
```

**Weighted vector:**

```sql
SELECT setweight(to_tsvector('english', title), 'A') ||
       setweight(to_tsvector('english', coalesce(description, '')), 'B') AS weighted_vector;
```

**Custom configuration:**

```sql
CREATE TEXT SEARCH CONFIGURATION english_custom (COPY = english);
CREATE TEXT SEARCH DICTIONARY domain_terms (TEMPLATE = simple, STOPWORDS = english);
ALTER TEXT SEARCH CONFIGURATION english_custom
    ALTER MAPPING FOR asciiword, asciihword, word WITH domain_terms;
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Index type | GIN for static/read‑heavy data; GiST for dynamic data |
| Expression index | `CREATE INDEX ... USING GIN (to_tsvector('english', body))` |
| Generated column | `GENERATED ALWAYS AS (to_tsvector(...)) STORED` |
| Weighting | `setweight(to_tsvector(title), 'A')` |
| Custom configuration | `CREATE TEXT SEARCH CONFIGURATION` and `CREATE TEXT SEARCH DICTIONARY` |

**Syntax rules:**

- Only the two‑argument form of `to_tsvector` can be used in expression indexes.
- Queries must use the same configuration as the index to use the index.
- GIN indexes store only lexemes, not weight labels, so weight‑filtered queries may require row rechecks.

**Constraints and limitations:**

- GIN indexes have slower build times and updates than GiST; increase `maintenance_work_mem` to speed up GIN builds.
- Stored `tsvector` columns increase disk usage but eliminate re‑computation at query time.
- Custom dictionaries add complexity and must be maintained as vocabulary changes.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Best‑practice index and query with weights and ranking

```sql
-- Step 1: Create a table with a stored generated tsvector column.
CREATE TABLE documents (
    doc_id   serial PRIMARY KEY,
    title    text,
    body     text,
    search_vector tsvector GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(body, '')), 'B')
    ) STORED
);

-- Step 2: Create a GIN index on the generated column.
CREATE INDEX documents_search_idx ON documents USING GIN (search_vector);

-- Step 3: Insert a document.
INSERT INTO documents (title, body) VALUES ('PostgreSQL Text Search', 'Full text search with weights');

-- Step 4: Query with ranking and weight boosting.
SELECT doc_id, title,
       ts_rank(search_vector, to_tsquery('english', 'postgresql')) AS rank
FROM documents
WHERE search_vector @@ to_tsquery('english', 'postgresql')
ORDER BY rank DESC;
```

**Expected output:**

```text
CREATE TABLE
CREATE INDEX
INSERT 0 1
 doc_id |          title          |   rank
--------+-------------------------+----------
      1 | PostgreSQL Text Search  | 0.607927
(1 row)
```

**Why this output appears**: The generated column `search_vector` concatenates the weighted title (`A`) and body (`B`) vectors. The GIN index accelerates the `@@` match. `ts_rank` computes a relevance score that reflects the weight labels, so a title match receives a higher score than a body match. The query uses the same `'english'` configuration as the index, ensuring the index is used.

#### Real-World Cases

- **Content management**: Articles are indexed with weighted vectors (title `A`, body `B`) and a GIN index; search results are ranked with `ts_rank` so title matches appear first.
- **E‑commerce search**: Product names and descriptions are stored in a generated `tsvector` column with a GIN index; `websearch_to_tsquery` handles user input with quoted phrases and `-` exclusions.
- **Multi‑language search**: A `config_name` column identifies the language of each document, and an expression index `to_tsvector(config_name, body)` supports mixed‑language collections.

---

## References

- PostgreSQL Documentation – Text Search Types – https://www.postgresql.org/docs/current/datatype-textsearch.html
- PostgreSQL Documentation – Text Search Functions and Operators – https://www.postgresql.org/docs/16/functions-textsearch.html
- PostgreSQL Documentation – Tables and Indexes – https://www.postgresql.org/docs/current/textsearch-tables.html
- PostgreSQL Documentation – Preferred Index Types for Text Search – https://www.postgresql.org/docs/current/textsearch-indexes.html
- PostgreSQL Documentation – Configuration Example – https://www.postgresql.org/docs/current/textsearch-configuration.html
- PostgreSQL Documentation – Dictionaries – https://www.postgresql.org/docs/current/textsearch-dictionaries.html
- PostgreSQL Documentation – Ranking Search Results – https://www.postgresql.org/docs/current/textsearch-controls.html#TEXTSEARCH-RANKING
- PostgreSQL Documentation – Highlighting Results – https://www.postgresql.org/docs/current/textsearch-controls.html#TEXTSEARCH-HEADLINE
- PostgreSQL Documentation – Text Search Debugging Functions – https://www.postgresql.org/docs/current/textsearch-debugging.html