# Comprehensive Programming Cheat Sheet: PostgreSQL Geometric Types

## Topic Overview

### Definitions

**Core Definition**
Geometric types are the built-in PostgreSQL data types—`POINT`, `LINE`, `LSEG`, `BOX`, `PATH`, `POLYGON`, and `CIRCLE`—designed to represent two-dimensional spatial objects directly in a database column, enabling storage and querying of locations, shapes, and their spatial relationships.

**Technical Definition**
PostgreSQL provides a family of geometric data types that represent two-dimensional spatial objects. These types store individual coordinates as **double precision** (`float8`) numbers, and PostgreSQL supplies a rich set of native functions and operators for operations such as scaling, translation, rotation, distance calculation, and intersection detection. The seven types form a hierarchy: `point` is the fundamental building block; `line` is an infinite line represented by the linear equation `Ax + By + C = 0`; `lseg` is a finite line segment between two points; `box` is a rectangular box defined by two opposite corners; `path` is a list of connected points that can be open or closed; `polygon` is a closed shape similar to a path; and `circle` is defined by a center point and a radius. These are **flat-plane, non-geographic** types—they do not understand Earth's curvature, coordinate reference systems (SRIDs), or projections. For real-world GIS applications, the PostGIS extension is the recommended choice.

**Beginner-Friendly Explanation**
Geometric types are the database's shapes and coordinates toolbox. They let you store a location (`POINT`), a line (`LINE` or `LSEG`), a rectangle (`BOX`), a connected trail of points (`PATH`), a closed shape like a state boundary (`POLYGON`), or a circular area (`CIRCLE`) directly in a column. Once the data is stored, PostgreSQL can answer spatial questions: “Do these two shapes overlap?”, “How far apart are these points?”, “What is the centre of this circle?”. The key caveat is that these built-in types treat the world as a flat sheet of paper. They are excellent for simple geometry on a plane—game boards, floor plans, diagram coordinates—but they are not a substitute for PostGIS when you need to work with real-world latitude and longitude on the curved surface of the Earth.

### Key Characteristics

- **Flat-plane, non-geographic geometry**: The built-in geometric types operate on a Cartesian plane with straight-line math. They have no concept of Earth's curvature, latitude/longitude, or coordinate reference systems. For geographic data, PostGIS is the correct tool.
- **Points are the fundamental building block**: Every other geometric type is constructed from points. Coordinates are stored as `double precision` (`float8`) numbers, providing approximately 15 significant digits of precision.
- **Distinct equality semantics**: The `~=` operator represents the usual notion of equality for `point`, `box`, `polygon`, and `circle` types. Some of these types also have an `=` operator, but `=` compares for **equal areas only**; the other scalar comparison operators (`<=` and so on) likewise compare areas for these types. This is a critical subtlety that can lead to incorrect query results if misunderstood.
- **GiST indexes are the spatial accelerator**: GiST (Generalized Search Tree) indexes are the primary index type for geometric data, supporting overlap (`&&`), containment, and nearest-neighbour queries. Without a GiST index, spatial queries perform full table scans.
- **Limited to 2D geometry**: The built-in types support only two-dimensional geometry. There are no native 3D or higher-dimensional geometric types in core PostgreSQL.

### Prerequisites

- A running PostgreSQL instance and a client connection.
- Familiarity with `CREATE TABLE` syntax and SQL data types.
- Basic understanding of coordinate geometry (x, y axes, points, lines, shapes).
- For spatial indexing: awareness of GiST indexes and, when combining geometric filters with scalar equality filters, the `btree_gist` extension.
- For real-world geographic applications: knowledge of PostGIS and the distinction between `geometry` (planar) and `geography` (spheroidal) types.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations and GiST indexes.
- Data Manipulation Language (DML) – inserting, querying, and updating geometric values.
- Spatial indexing and query optimisation – GiST, SP-GiST, and BRIN indexes.
- Geographic Information Systems (GIS) – PostGIS extension for real-world spatial data.
- Game development and diagramming – flat-plane geometry for game boards, floor plans, and schematic layouts.

---

## Core Concepts and Features

### 1. `POINT`

#### Definitions

**Core Definition**
`POINT` is a PostgreSQL type that represents a single location on a two-dimensional plane, stored as an (x, y) coordinate pair.

**Technical Definition**
A `point` occupies **16 bytes** and is represented as `(x, y)`. Points are the fundamental two-dimensional building block for geometric types. Values of type `point` are specified using either `(x, y)` or `x, y` syntax, and are output using the first syntax. Coordinates are stored as `double precision` (`float8`) numbers.

**Beginner-Friendly Explanation**
`POINT` is a dot on a map. It records exactly where something is: “x = 5, y = 10.” Every other shape is built from points, so understanding `POINT` is the key to understanding all the geometric types.

#### Purposes

- To store a single location (e.g., a marker on a game board, a vertex of a shape).
- To serve as the building block for all other geometric types.
- To support distance calculations and nearest-neighbour queries.
- To anchor translations and rotations of other shapes.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name POINT
```

**Input syntaxes:**

```sql
'(x, y)'
'(x,y)'
'x, y'
'x,y'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `(x, y)` | Coordinate pair; x and y are floating-point numbers. |
| Storage | 16 bytes. |

**Syntax rules:**

- Points are output using the `(x, y)` syntax.
- Coordinates are stored as `double precision`.
- Points are used as anchors for translation (`+`, `-`) and scaling/rotation (`*`, `/`) operations on other types.

**Constraints and limitations:**

- `point` supports the usual comparison operators, but the `~=` operator represents true geometric equality, while `=` may compare differently depending on the type involved.
- Points have no concept of geographic coordinates; latitude/longitude values are just numbers on a plane.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing and querying points

```sql
-- Step 1: Create a table with a point column.
CREATE TABLE locations (
    id   int PRIMARY KEY,
    name text NOT NULL,
    pos  point NOT NULL
);

-- Step 2: Insert points using different input syntaxes.
INSERT INTO locations VALUES
    (1, 'origin',    '(0,0)'),
    (2, 'northeast', '(3,4)'),
    (3, 'southwest', '-1,-1');

-- Step 3: Query the points.
SELECT id, name, pos FROM locations ORDER BY id;
```

**Expected output:**

```text
 id |    name    |  pos
----+------------+-------
  1 | origin     | (0,0)
  2 | northeast  | (3,4)
  3 | southwest  | (-1,-1)
(3 rows)
```

**Why this output appears**: Each point is stored as a pair of floating-point coordinates and rendered in the canonical `(x,y)` format. The input syntax `-1,-1` is accepted and normalized to `(-1,-1)` on output.

#### Real-World Cases

- **Game development**: A `tiles` table stores each tile's position on a 2D game board as a `point`.
- **Floor plans**: A `fixtures` table stores the location of lights, outlets, and furniture as points in a floor plan coordinate system.
- **Diagramming tools**: A `nodes` table stores the position of each node in a flowchart or network diagram.

---

### 2. `LINE`

#### Definitions

**Core Definition**
`LINE` is a PostgreSQL type that represents an infinite straight line on a two-dimensional plane, defined by the linear equation `Ax + By + C = 0`.

**Technical Definition**
A `line` occupies **24 bytes** and is represented by the coefficients `{A, B, C}` of the linear equation `Ax + By + C = 0`, where `A` and `B` are not both zero. Values can be input as `{A, B, C}` or as two points on the line using any of the forms `[(x1,y1),(x2,y2)]`, `((x1,y1),(x2,y2))`, `(x1,y1),(x2,y2)`, or `x1,y1,x2,y2`. Lines are infinite in both directions.

**Beginner-Friendly Explanation**
`LINE` is an infinitely long straight line. You can describe it either by its equation coefficients (A, B, C) or by giving two points that the line passes through. It extends forever in both directions—unlike `LSEG`, which is finite.

#### Purposes

- To represent infinite lines for geometric computations (e.g., axes, asymptotes, reference lines).
- To compute intersections with other geometric objects.
- To support perpendicular and parallel line calculations.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name LINE
```

**Input syntaxes:**

```sql
'{A, B, C}'
'[(x1,y1),(x2,y2)]'
'((x1,y1),(x2,y2))'
'(x1,y1),(x2,y2)'
'x1,y1,x2,y2'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `{A, B, C}` | Coefficients of `Ax + By + C = 0`. |
| Two points | Any two distinct points on the line. |
| Storage | 24 bytes. |

**Syntax rules:**

- `A` and `B` must not both be zero.
- Two distinct points are required when using point-based input.
- Lines are output in `{A, B, C}` format.

**Constraints and limitations:**

- Lines are infinite; they are not clipped to any bounding region.
- Lines have no notion of direction or orientation in the way a segment does.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating and querying lines

```sql
-- Step 1: Create a table with a line column.
CREATE TABLE reference_lines (
    id   int PRIMARY KEY,
    label text,
    l    line
);

-- Step 2: Insert lines using coefficients and two points.
INSERT INTO reference_lines VALUES
    (1, 'diagonal', '{1, -1, 0}'),          -- y = x
    (2, 'horizontal', '((0,5),(1,5))');      -- y = 5

-- Step 3: Query the lines.
SELECT id, label, l FROM reference_lines ORDER BY id;
```

**Expected output:**

```text
 id |   label    |    l
----+------------+----------
  1 | diagonal   | {1,-1,0}
  2 | horizontal | {0,1,-5}
(2 rows)
```

**Why this output appears**: The line `y = x` is represented as `{1, -1, 0}` because `1x + (-1)y + 0 = 0`. The horizontal line through `(0,5)` and `(1,5)` is normalized to `{0, 1, -5}` because `0x + 1y - 5 = 0` (i.e., `y = 5`).

#### Real-World Cases

- **Geometry education**: A `lines` table stores reference lines for teaching coordinate geometry.
- **CAD tools**: Infinite construction lines are stored as `line` types for alignment and reference.
- **Ray tracing**: A `rays` table stores ray directions as lines for intersection computations.

---

### 3. `LSEG`

#### Definitions

**Core Definition**
`LSEG` is a PostgreSQL type that represents a finite line segment between two points.

**Technical Definition**
A `lseg` occupies **32 bytes** and is represented by its two endpoints as `((x1,y1),(x2,y2))`. It is a finite segment, not an infinite line. Values can be input using any of the forms `[(x1,y1),(x2,y2)]`, `((x1,y1),(x2,y2))`, `(x1,y1),(x2,y2)`, or `x1,y1,x2,y2`. Line segments are output using the first syntax.

**Beginner-Friendly Explanation**
`LSEG` is a line segment—a straight line with a clear start and end point. Unlike `LINE`, it does not extend forever; it stops at its two endpoints.

#### Purposes

- To represent finite straight edges or edges of shapes.
- To compute distances between points and segments.
- To determine intersections between segments and other geometric objects.
- To build paths and polygons from connected segments.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name LSEG
```

**Input syntaxes:**

```sql
'[(x1,y1),(x2,y2)]'
'((x1,y1),(x2,y2))'
'(x1,y1),(x2,y2)'
'x1,y1,x2,y2'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `((x1,y1),(x2,y2))` | The two endpoints of the segment. |
| Storage | 32 bytes. |

**Syntax rules:**

- The two endpoints must be distinct.
- Segments are output using the `[(x1,y1),(x2,y2)]` syntax.

**Constraints and limitations:**

- Segments are straight; they cannot represent curves.
- Segments have no width or area.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Working with line segments

```sql
-- Step 1: Create a table with a lseg column.
CREATE TABLE edges (
    id int PRIMARY KEY,
    e  lseg
);

-- Step 2: Insert segments.
INSERT INTO edges VALUES
    (1, '[(0,0),(3,4)]'),
    (2, '((1,1),(1,5))');

-- Step 3: Query the segments.
SELECT id, e FROM edges ORDER BY id;
```

**Expected output:**

```text
 id |       e
----+----------------
  1 | [(0,0),(3,4)]
  2 | [(1,1),(1,5)]
(2 rows)
```

**Why this output appears**: Each segment is stored as its two endpoints and rendered in the canonical `[(x1,y1),(x2,y2)]` format. The input `((1,1),(1,5))` is normalized to `[(1,1),(1,5)]` on output.

#### Real-World Cases

- **Floor plans**: Walls and edges are stored as `lseg` values, enabling distance and intersection calculations.
- **Network diagrams**: Connections between nodes are represented as `lseg` values.
- **Mesh generation**: Edges of a 2D mesh are stored as line segments for geometric processing.

---

### 4. `BOX`

#### Definitions

**Core Definition**
`BOX` is a PostgreSQL type that represents an axis-aligned rectangular box on a two-dimensional plane, defined by two opposite corners.

**Technical Definition**
A `box` occupies **32 bytes** and is represented by two opposite corners as `((x1,y1),(x2,y2))`. On input, any two opposite corners can be supplied, but PostgreSQL reorders them as needed to store the **upper right** and **lower left** corners, in that order. Boxes are output using the `(x1,y1),(x2,y2)` syntax.

**Beginner-Friendly Explanation**
`BOX` is a rectangle aligned with the axes. You give it two opposite corners, and PostgreSQL stores them in a standard order (upper right and lower left). It is useful for bounding boxes, regions of interest, and simple area calculations.

#### Purposes

- To represent rectangular regions, bounding boxes, and axis-aligned areas.
- To support overlap (`&&`) and containment queries for spatial filtering.
- To serve as a bounding box for more complex shapes.
- To compute areas and intersections of rectangles.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name BOX
```

**Input syntaxes:**

```sql
'((x1,y1),(x2,y2))'
'(x1,y1),(x2,y2)'
'x1,y1,x2,y2'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `((x1,y1),(x2,y2))` | Two opposite corners. |
| Storage | 32 bytes. |

**Syntax rules:**

- Any two opposite corners are accepted on input.
- Values are reordered to store the upper right and lower left corners.
- Boxes are output using the second syntax form.

**Constraints and limitations:**

- Boxes are axis-aligned; they cannot be rotated.
- Boxes have area and can be compared by area using `=` and the scalar comparison operators.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating and querying boxes

```sql
-- Step 1: Create a table with a box column.
CREATE TABLE regions (
    id  int PRIMARY KEY,
    name text,
    b   box
);

-- Step 2: Insert boxes with corners in different orders.
INSERT INTO regions VALUES
    (1, 'A', '((0,0),(2,2))'),
    (2, 'B', '((3,1),(1,3))');   -- corners supplied in non-canonical order

-- Step 3: Query the boxes.
SELECT id, name, b FROM regions ORDER BY id;
```

**Expected output:**

```text
 id | name |    b
----+------+---------
  1 | A    | (2,2),(0,0)
  2 | B    | (3,3),(1,1)
(2 rows)
```

**Why this output appears**: The first box is already in canonical order (upper right `(2,2)`, lower left `(0,0)`). The second box’s corners were supplied as `(3,1)` and `(1,3)`; PostgreSQL reordered them to upper right `(3,3)` and lower left `(1,1)`.

#### Real-World Cases

- **Image processing**: Bounding boxes around detected objects are stored as `box` values.
- **Map tile systems**: Rectangular tile boundaries are stored as boxes for spatial indexing.
- **Game collision detection**: Axis-aligned bounding boxes (AABBs) are stored as `box` values for fast overlap tests.

---

### 5. `PATH`

#### Definitions

**Core Definition**
`PATH` is a PostgreSQL type that represents a list of connected points, which can be either **open** (first and last points not connected) or **closed** (first and last points connected).

**Technical Definition**
A `path` occupies **16 + 16n bytes**, where `n` is the number of points. An open path is represented as `[(x1,y1),...,(xn,yn)]` and a closed path as `((x1,y1),...,(xn,yn))`. Paths are lists of connected points; the distinction between open and closed is whether the first and last points are considered connected.

**Beginner-Friendly Explanation**
`PATH` is a trail of connected dots. If the trail is open, it starts at the first dot and ends at the last dot without looping back. If it is closed, the last dot connects back to the first, forming a loop. Paths are useful for representing routes, outlines, and sequences of connected points.

#### Purposes

- To represent connected point sequences such as routes, trails, or outlines.
- To support open paths (polylines) and closed paths (loops) with a single type.
- To compute lengths and intersections of connected segments.
- To serve as an intermediate representation between individual segments and closed polygons.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name PATH
```

**Input syntaxes:**

```sql
'[(x1,y1),...,(xn,yn)]'       -- open path
'((x1,y1),...,(xn,yn))'       -- closed path
'(x1,y1),...,(xn,yn)'         -- open path
'(x1,y1),...,(xn,yn)'         -- can be interpreted as closed depending on context
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[(...)]` | Open path (square brackets). |
| `((...))` | Closed path (parentheses). |
| Storage | `16 + 16n` bytes for `n` points. |

**Syntax rules:**

- Square brackets denote an open path; parentheses denote a closed path.
- A path must have at least one point.

**Constraints and limitations:**

- Paths are sequences of straight segments; they cannot represent curves.
- A path with fewer than two points has no length.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Open and closed paths

```sql
-- Step 1: Create a table with a path column.
CREATE TABLE trails (
    id   int PRIMARY KEY,
    name text,
    p    path
);

-- Step 2: Insert an open path and a closed path.
INSERT INTO trails VALUES
    (1, 'open trail',  '[(0,0),(1,1),(2,0)]'),
    (2, 'closed loop', '((0,0),(1,1),(2,0))');

-- Step 3: Query the paths.
SELECT id, name, p FROM trails ORDER BY id;
```

**Expected output:**

```text
 id |    name     |        p
----+-------------+------------------
  1 | open trail  | [(0,0),(1,1),(2,0)]
  2 | closed loop | ((0,0),(1,1),(2,0))
(2 rows)
```

**Why this output appears**: The square-bracket syntax produces an open path, and the parenthesized syntax produces a closed path. The point lists are identical, but the connectivity differs: the closed path connects the last point back to the first.

#### Real-World Cases

- **Route planning**: A `routes` table stores the sequence of waypoints along a route as a path.
- **Drawing tools**: A `strokes` table stores freehand drawing strokes as paths.
- **Game AI**: Patrol routes for NPCs are stored as closed or open paths.

---

### 6. `POLYGON`

#### Definitions

**Core Definition**
`POLYGON` is a PostgreSQL type that represents a closed, filled shape defined by a sequence of connected points, where the last point connects back to the first.

**Technical Definition**
A `polygon` occupies **40 + 16n bytes**, where `n` is the number of points. It is similar to a closed path but represents a filled area rather than just an outline. Values are input as `((x1,y1),...,(xn,yn))`, with the first and last points implicitly connected. Polygons support area calculations, intersection tests, and containment checks.

**Beginner-Friendly Explanation**
`POLYGON` is a closed shape with an inside and an outside—like a triangle, a rectangle, or a state boundary. It is defined by a list of corner points, and the shape is filled in. You can ask whether a point is inside a polygon, whether two polygons overlap, and what the area of a polygon is.

#### Purposes

- To represent filled regions and closed shapes (e.g., state boundaries, property parcels, game zones).
- To support containment queries (`@>`, `<@`) and overlap tests (`&&`).
- To compute areas and intersections of shapes.
- To serve as a building block for spatial analysis on a flat plane.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name POLYGON
```

**Input syntax:**

```sql
'((x1,y1),...,(xn,yn))'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `((...))` | Sequence of points defining the polygon boundary. |
| Storage | `40 + 16n` bytes for `n` points. |

**Syntax rules:**

- The first and last points are implicitly connected to close the shape.
- The order of points matters; polygons should be simple (non-self-intersecting) for predictable results.
- Polygons support the `~=` operator for true geometric equality and `=` for area equality.

**Constraints and limitations:**

- Polygons cannot have holes (no interior rings).
- Self-intersecting polygons may produce unexpected results for area and containment calculations.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating and querying polygons

```sql
-- Step 1: Create a table with a polygon column.
CREATE TABLE zones (
    id   int PRIMARY KEY,
    name text,
    z    polygon
);

-- Step 2: Insert a triangle and a square.
INSERT INTO zones VALUES
    (1, 'triangle', '((0,0),(4,0),(2,3))'),
    (2, 'square',   '((0,0),(2,0),(2,2),(0,2))');

-- Step 3: Compute the area of each polygon.
SELECT id, name, area(z) AS area FROM zones ORDER BY id;
```

**Expected output:**

```text
 id |   name   | area
----+----------+-------
  1 | triangle |     6
  2 | square   |     4
(2 rows)
```

**Why this output appears**: The triangle has base 4 and height 3, so its area is `(4 × 3) / 2 = 6`. The square has side length 2, so its area is `2 × 2 = 4`. The `area()` function computes the filled area of each polygon.

#### Real-World Cases

- **Property management**: Land parcels are stored as polygons, and queries find parcels that contain a given point.
- **Game zone control**: Capture-the-flag zones are stored as polygons, and overlap tests determine which team controls an area.
- **Retail trade areas**: Geographic trade areas are approximated as polygons for simple spatial analysis.

---

### 7. `CIRCLE`

#### Definitions

**Core Definition**
`CIRCLE` is a PostgreSQL type that represents a circle on a two-dimensional plane, defined by a center point and a radius.

**Technical Definition**
A `circle` occupies **24 bytes** and is represented as `<(x,y),r>`, where `(x,y)` is the center point and `r` is the radius. The center is stored as a `point` (16 bytes) and the radius as a `double precision` number (8 bytes), totaling 24 bytes. Circles support distance calculations, containment tests, and overlap checks.

**Beginner-Friendly Explanation**
`CIRCLE` is a round shape defined by its center and radius. It is perfect for representing things like coverage areas, zones of influence, or circular regions on a flat plane.

#### Purposes

- To represent circular regions and coverage areas.
- To support distance and nearest-neighbour queries.
- To test whether points or other shapes fall within a circle.
- To compute intersections and overlaps between circles and other geometric objects.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name CIRCLE
```

**Input syntax:**

```sql
'<(x,y),r>'
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `<(x,y),r>` | Center point and radius. |
| Storage | 24 bytes (16 for center point + 8 for radius). |

**Syntax rules:**

- The radius must be a non-negative number.
- The center is stored as a `point`.

**Constraints and limitations:**

- Circles are perfect circles on a flat plane; they do not account for Earth's curvature.
- Circles cannot be combined with polygons without approximation.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Creating and querying circles

```sql
-- Step 1: Create a table with a circle column.
CREATE TABLE coverage (
    id     int PRIMARY KEY,
    name   text,
    region circle
);

-- Step 2: Insert circles.
INSERT INTO coverage VALUES
    (1, 'small', '<(0,0),1>'),
    (2, 'large', '<(5,5),3>');

-- Step 3: Find circles that contain a given point.
SELECT name FROM coverage WHERE region @> point '(1,0)';
```

**Expected output:**

```text
  name
--------
 small
(1 row)
```

**Why this output appears**: The small circle centred at `(0,0)` with radius 1 contains the point `(1,0)` because the distance from the centre to the point is exactly 1, which is within the radius. The large circle centred at `(5,5)` with radius 3 does not contain `(1,0)`.

#### Real-World Cases

- **Delivery zones**: A `coverage_areas` table stores circular delivery zones as circles, and queries check whether a customer's location falls within a zone.
- **Proximity searches**: A `hotspots` table stores circular regions of interest, and queries find hotspots within a given distance of a point.
- **Signal coverage**: Antenna coverage patterns are approximated as circles for simple planning calculations.

---

### 8. Geometric Operations

#### Definitions

**Core Definition**
Geometric operations are the operators and functions that compare, transform, measure, and combine geometric values—including translation, scaling, rotation, distance, overlap, containment, and intersection.

**Technical Definition**
PostgreSQL provides a large set of native operators and functions for geometric types. Operators include `+` and `-` for translation, `*` and `/` for scaling and rotation, `#` for intersection, `@-@` for length or circumference, `@@` for center, `##` for closest point, `<->` for distance, `&&` for overlap, and a family of positional operators (`<<`, `>>`, `&<`, `&>`, `<<|`, `|>>`, `&<|`, `|&>`, `<^`, `>^`). Functions include `area()`, `center()`, `diameter()`, `height()`, `width()`, `radius()`, `isclosed()`, `isopen()`, `npoints()`, `pclose()`, `popen()`, and `polygon()`.

**Beginner-Friendly Explanation**
Geometric operations are the verbs of the geometric toolbox. You can move a shape (`+`, `-`), resize it (`*`, `/`), find where two shapes intersect (`#`), measure a shape's area or length (`area`, `@-@`), find its centre (`@@`), compute the distance between two objects (`<->`), or ask whether they overlap (`&&`) or are positioned to the left or right of each other.

#### Purposes

- To perform spatial analysis directly in SQL without application-level geometry libraries.
- To filter rows based on spatial relationships (overlap, containment, distance).
- To compute derived geometric properties (area, centre, length) for reporting and analysis.
- To transform shapes for rendering, simulation, or alignment.

#### Syntax Rules and Structure

**Translation and scaling/rotation:**

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Translate shape | `box '((0,0),(1,1))' + point '(2,0)'` |
| `-` | Translate shape | `box '((0,0),(1,1))' - point '(2,0)'` |
| `*` | Scale/rotate | `box '((0,0),(1,1))' * point '(2,0)'` |
| `/` | Scale/rotate | `box '((0,0),(2,2))' / point '(2,0)'` |

**Measurement and properties:**

| Operator / Function | Description | Example |
|---------------------|-------------|---------|
| `#` | Intersection point or box | `'((1,-1),(-1,1))' # '((1,1),(-1,-1))'` |
| `@-@` | Length or circumference | `@-@ path '((0,0),(1,0))'` |
| `@@` | Centre of object | `@@ circle '((0,0),10)'` |
| `##` | Closest point on second object | `point '(0,0)' ## lseg '((2,0),(0,2))'` |
| `<->` | Distance between objects | `circle '((0,0),1)' <-> circle '((5,0),1)'` |
| `area(object)` | Area | `area(box '((0,0),(1,1))')` |
| `center(object)` | Centre point | `center(box '((0,0),(2,2))')` |

**Positional and overlap operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `&&` | Overlaps? | `box '((0,0),(1,1))' && box '((0,0),(2,2))'` |
| `<<` | Strictly left of? | `circle '((0,0),1)' << circle '((5,0),1)'` |
| `>>` | Strictly right of? | `circle '((5,0),1)' >> circle '((0,0),1)'` |
| `&<` | Does not extend to the right of? | `box '((0,0),(1,1))' &< box '((0,0),(2,2))'` |
| `&>` | Does not extend to the left of? | `box '((0,0),(3,3))' &> box '((0,0),(2,2))'` |
| `<<\|` | Strictly below? | `box '((0,0),(3,3))' <<\| box '((3,4),(5,5))'` |
| `\|>>` | Strictly above? | `box '((3,4),(5,5))' \|>> box '((0,0),(3,3))'` |

**Syntax rules:**

- The `~=` operator represents true geometric equality for `point`, `box`, `polygon`, and `circle`; `=` compares areas for `box`, `polygon`, and `circle`.
- The `#` operator returns a point for line-line intersection, a box for box-box intersection, and so on.
- The `<->` operator computes the shortest distance between two geometric objects.

**Constraints and limitations:**

- Area-based comparison (`=`, `<=`, `>=`) can produce surprising results if true geometric equality is intended; use `~=` instead.
- Some operators are type-specific; not all operators apply to all geometric types.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Translation, distance, and overlap

```sql
-- Step 1: Translate a box by a point vector.
SELECT box '((0,0),(1,1))' + point '(2,0)' AS translated;

-- Step 2: Compute the distance between two circles.
SELECT circle '((0,0),1)' <-> circle '((5,0),1)' AS distance;

-- Step 3: Test whether two boxes overlap.
SELECT box '((0,0),(1,1))' && box '((0,0),(2,2))' AS overlaps;

-- Step 4: Compute the area of a polygon.
SELECT area(polygon '((0,0),(4,0),(2,3))') AS triangle_area;

-- Step 5: Find the centre of a box.
SELECT center(box '((0,0),(2,2))') AS box_center;
```

**Expected output:**

```text
 translated
-------------
 (3,1),(2,0)
(1 row)

 distance
----------
        3
(1 row)

 overlaps
----------
 t
(1 row)

 triangle_area
---------------
             6
(1 row)

 box_center
------------
 (1,1)
(1 row)
```

**Why this output appears**: Translating the box by `(2,0)` shifts both corners by 2 units to the right. The distance between the circles is the distance between their centres (5) minus their radii (1 + 1), giving 3. The boxes overlap because the second box contains the first. The triangle area is 6. The centre of the box is `(1,1)`.

#### Real-World Cases

- **Collision detection**: Game engines use `&&` to quickly test whether two bounding boxes overlap before performing precise collision checks.
- **Proximity search**: A `locations` table uses `<->` to find the nearest points of interest to a user's position.
- **Spatial filtering**: A `parcels` table uses `@>` to find all parcels containing a given point.

---

### 9. Geometric Best Practices

#### Definitions

**Core Definition**
Geometric best practices are the guidelines for choosing the right geometric type, indexing correctly with GiST, distinguishing true geometric equality from area equality, and knowing when to use PostGIS instead of the built-in types.

**Technical Definition**
Best practices include: (1) **use the simplest type that models the data**—`point` for locations, `box` for rectangular regions, `polygon` for arbitrary filled shapes; (2) **create GiST indexes on columns used in spatial queries**—GiST indexes accelerate overlap (`&&`), containment (`@>`), and distance (`<->`) operators; (3) **use `~=` for true geometric equality, not `=`**, because `=` compares areas for `box`, `polygon`, and `circle`; (4) **prefer PostGIS for real-world geographic data**—the built-in types are flat-plane and non-geographic, while PostGIS provides SRIDs, projections, and spheroidal calculations; and (5) **use `btree_gist` when combining spatial filters with scalar equality filters**—a composite GiST index on `(spatial_column, scalar_column)` can be dramatically more efficient than separate indexes.

**Beginner-Friendly Explanation**
Best practices for geometric data are about using the right shape, making queries fast, and avoiding subtle bugs. Use `POINT` for simple locations and `POLYGON` for areas. Add a GiST index to any geometric column you query often. Remember that `=` compares area, not shape—use `~=` when you mean “is this the same shape?”. And if you are working with real-world maps, use PostGIS instead of the built-in types.

#### Purposes

- To avoid the performance cliff of unindexed spatial queries.
- To prevent incorrect query results caused by area-based equality semantics.
- To choose the right tool (built-in geometry vs. PostGIS) for the application's needs.
- To combine spatial and scalar filters efficiently with `btree_gist`.

#### Syntax Rules and Structure

**GiST index on a geometric column:**

```sql
CREATE INDEX idx_locations_pos ON locations USING GIST (pos);
```

**Composite GiST index with `btree_gist`:**

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

CREATE INDEX idx_events_location_time
ON scheduled_events
USING GIST (event_location, event_timestamp);
```

**True geometric equality vs. area equality:**

```sql
-- True geometric equality: use ~=
SELECT polygon '((0,0),(1,0),(1,1),(0,1))' ~= polygon '((1,0),(1,1),(0,1),(0,0))';

-- Area equality: = compares areas
SELECT box '((0,0),(1,1))' = box '((0,0),(2,0.5))';
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Simplest type | `point` for locations, `box` for rectangles, `polygon` for areas |
| GiST index | `CREATE INDEX ... USING GIST (geom_column)` |
| True equality | Use `~=` not `=` |
| Scalar + spatial filter | `btree_gist` composite GiST index |
| Real-world GIS | Use PostGIS (`geometry`/`geography`) instead |

**Syntax rules:**

- GiST indexes support `&&`, `@>`, `<@`, `<->`, and the positional operators.
- `btree_gist` provides GiST operator classes for B-tree-comparable types, enabling composite indexes.
- `~=` is the true geometric equality operator for `point`, `box`, `polygon`, and `circle`.

**Constraints and limitations:**

- GiST indexes have higher write overhead than B-tree indexes; create them after bulk loading.
- The built-in geometric types have no SRID or projection support; they are purely Cartesian.
- Composite GiST indexes with non-geometric columns can be slower than pure geometric indexes due to sorting limitations.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: GiST index for spatial queries

```sql
-- Step 1: Create a table with a box column.
CREATE TABLE map_regions (
    id   int PRIMARY KEY,
    name text,
    b    box
);

-- Step 2: Insert sample regions.
INSERT INTO map_regions VALUES
    (1, 'A', '((0,0),(2,2))'),
    (2, 'B', '((1,1),(3,3))'),
    (3, 'C', '((5,5),(6,6))');

-- Step 3: Create a GiST index.
CREATE INDEX idx_map_regions_b ON map_regions USING GIST (b);

-- Step 4: Query overlapping regions.
SELECT id, name FROM map_regions
WHERE b && box '((1.5,1.5),(2.5,2.5))';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
CREATE INDEX
 id | name
----+------
  1 | A
  2 | B
(2 rows)
```

**Why this output appears**: The GiST index on `b` accelerates the `&&` overlap query. Regions A and B overlap the query box `((1.5,1.5),(2.5,2.5))`; region C does not.

#### Real-World Cases

- **Facility management**: A `rooms` table stores room boundaries as `box` values with a GiST index; queries find all rooms overlapping a requested area.
- **Event planning**: A `venues` table stores venue locations as `point` values with a GiST index; nearest-neighbour queries find the closest venues to an attendee.
- **Game development**: A `entities` table stores bounding boxes with a GiST index; collision queries use `&&` to quickly narrow down candidate pairs before precise checks.

---

## References

- PostgreSQL Documentation – Geometric Types – https://www.postgresql.org/docs/16/datatype-geometric.html
- PostgreSQL Documentation – Geometric Functions and Operators – https://www.postgresql.org/docs/current/functions-geometry.html
- PostgreSQL Documentation – Geometric Functions and Operators (8.3) – https://www.postgresql.org/docs/8.3/functions-geometry.html
- PostgreSQL Documentation – GiST Indexes – https://www.postgresql.org/docs/current/gist.html
- Red Gate – MySQL vs PostgreSQL: Spatial Data Types, Functions, and Migration Pitfalls – https://www.red-gate.com/simple-talk/data-analytics/data-science/mysql-vs-postgresql-spatial-data-types-functions-and-migration-pitfalls-explained/
- Neon – The btree_gist Extension – https://github.com/neondatabase/website/blob/6420cdb73bc86b8ab79987abf2a99f8f2aa9543b/content/docs/extensions/btree_gist.md
- Geospatial Types (Core Postgres) – https://github.com/BrewHubPHL/supabase-specialist/blob/master/patterns/geospatial-types.md
- PostGIS Geometry Indexing Best Practices – https://github.com/sempervent/sempervent.github.io/blob/main/docs/tutorials/database-data-engineering/postgis-geometry-indexing.md
- Design PostGIS Tables – https://skillhub.360.com
- PostgreSQL GiST and btree_gist Performance Discussion – https://www.postgresql.org/message-id/