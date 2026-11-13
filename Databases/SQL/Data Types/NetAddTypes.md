# Comprehensive Programming Cheat Sheet: SQL Network Address Types

## Topic Overview

### Definitions

**Core Definition**
Network address types are the PostgreSQL data types—`INET`, `CIDR`, `MACADDR`, and `MACADDR8`—designed to store IPv4 addresses, IPv6 addresses, and MAC addresses with built‑in validation, specialised operators, and functions for subnet arithmetic and address manipulation.

**Technical Definition**
PostgreSQL offers data types to store IPv4, IPv6, and MAC addresses. The `cidr` type holds an IPv4 or IPv6 network specification and follows Classless Internet Domain Routing conventions; the `inet` type holds an IPv4 or IPv6 host address together with an optional subnet (netmask); `macaddr` stores 6‑byte MAC addresses in the traditional EUI‑48 format; and `macaddr8` stores 8‑byte MAC addresses in EUI‑64 format. Using these types instead of plain text types is recommended because they provide input error checking and specialised operators and functions that text types cannot offer. When sorting `inet` or `cidr` data, IPv4 addresses always sort before IPv6 addresses, including IPv4 addresses encapsulated or mapped to IPv6.

**Beginner-Friendly Explanation**
Network address types are the database's specialised containers for IP addresses and hardware addresses. Instead of storing an IP address as a plain string—where typos like `192.168.1.999` would silently be accepted—PostgreSQL validates the address on input, stores it in a compact binary form, and gives you tools to ask questions like “is this address inside that subnet?” or “what is the broadcast address for this network?”. `INET` is for hosts (and optionally their subnet), `CIDR` is strictly for networks, `MACADDR` is for traditional 6‑byte hardware addresses, and `MACADDR8` is for newer 8‑byte EUI‑64 addresses.

### Key Characteristics

- **Validation on input**: Invalid addresses are rejected at insert time, unlike `text` or `varchar` columns that accept any string.
- **Compact binary storage**: `inet` and `cidr` use 7 bytes for IPv4 and 19 bytes for IPv6; `macaddr` uses 6 bytes; `macaddr8` uses 8 bytes.
- **Specialised operators and functions**: Subnet containment (`<<`, `>>`, `&&`), bitwise operations (`&`, `|`, `~`), address arithmetic (`+`, `-`), and functions such as `host()`, `netmask()`, `broadcast()`, `network()`, and `masklen()`.
- **Strict distinction between `inet` and `cidr`**: `inet` accepts values with non‑zero bits to the right of the netmask (i.e., a host address within a subnet), whereas `cidr` rejects such values and accepts only true network addresses.
- **Cast relationships**: Any `cidr` value can be cast to `inet` implicitly; casting an `inet` value to `cidr` silently zeroes any bits to the right of the netmask to produce a valid network address.

### Prerequisites

- A running PostgreSQL instance (network address types have been available since early versions; the documentation shown covers PostgreSQL 15 and 16).
- Familiarity with `CREATE TABLE` syntax and SQL data types.
- Basic understanding of IPv4/IPv6 addressing and CIDR notation (e.g., `192.168.1.0/24`).
- For MAC address types: awareness of the difference between EUI‑48 (6 bytes) and EUI‑64 (8 bytes) formats.

### Related Programming Areas

- Data Definition Language (DDL) – column type declarations.
- Data Manipulation Language (DML) – inserting, querying, and updating network addresses.
- Network and security engineering – IP allowlists, subnet membership checks, and MAC address tracking.
- Query optimisation – B‑tree indexes on `inet` and `cidr` columns support equality and ordering; containment operators (`<<`, `>>`, `&&`) can be indexed with suitable operator classes.
- Application development – mapping PostgreSQL network types to language‑level types such as `InetAddress` or `IpAddress`.

---

## Core Concepts and Features

### 1. `INET`

#### Definitions

**Core Definition**
`INET` is a PostgreSQL type that stores an IPv4 or IPv6 host address together with an optional subnet (netmask).

**Technical Definition**
The `inet` type holds an IPv4 or IPv6 host address and optionally its subnet, all in one field. The subnet is represented by the number of network address bits present in the host address (the “netmask”). If the netmask is 32 and the address is IPv4, the value indicates a single host rather than a subnet; in IPv6, the address length is 128 bits, so 128 bits specify a unique host address. The input format is `address/y`, where `address` is an IPv4 or IPv6 address and `y` is the number of bits in the netmask. If `/y` is omitted, the netmask defaults to 32 for IPv4 or 128 for IPv6, representing a single host. On output, the `/y` portion is suppressed when the netmask specifies a single host. To accept only networks, the `cidr` type should be used instead of `inet`.

**Beginner-Friendly Explanation**
`INET` is the type for an IP address that might also carry a subnet label. You can store just `192.168.1.5` (a single host), or `192.168.1.0/24` (a host address within the `192.168.1.0/24` network). It is the most flexible IP type: it accepts both plain hosts and hosts with a subnet prefix.

#### Purposes

- To store IPv4 and IPv6 host addresses with optional subnet information in a single, validated column.
- To support subnet‑membership queries and address arithmetic directly in SQL.
- To provide a text‑friendly but compact alternative to storing IP addresses as `varchar`.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name INET
```

**Input format:**

```text
address/y
```

where `address` is an IPv4 or IPv6 address and `y` is the netmask length in bits.

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `address` | IPv4 or IPv6 host address. |
| `/y` | Optional netmask length; defaults to 32 (IPv4) or 128 (IPv6). |
| Storage | 7 bytes (IPv4) or 19 bytes (IPv6). |

**Syntax rules:**

- `/y` is optional; omitting it means a single host.
- On output, the `/y` portion is suppressed for single‑host values.
- `inet` accepts values with non‑zero bits to the right of the netmask (unlike `cidr`).

**Constraints and limitations:**

- `inet` is not restricted to network addresses; use `cidr` if only networks should be accepted.
- Sorting places IPv4 before IPv6.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing and querying INET addresses

```sql
-- Step 1: Create a table with an INET column.
CREATE TABLE servers (
    server_id serial PRIMARY KEY,
    hostname  text NOT NULL,
    ip        inet NOT NULL
);

-- Step 2: Insert IPv4 hosts and a subnet.
INSERT INTO servers (hostname, ip) VALUES
    ('web-01', '192.168.1.10'),
    ('web-02', '192.168.1.11'),
    ('db-01',  '10.0.0.5/32'),
    ('net-01', '192.168.1.0/24');

-- Step 3: Query by subnet containment.
SELECT hostname, ip
FROM servers
WHERE ip << inet '192.168.1.0/24';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 4
 hostname |       ip
----------+-----------------
 web-01   | 192.168.1.10
 web-02   | 192.168.1.11
 net-01   | 192.168.1.0/24
(3 rows)
```

**Why this output appears**: The `<<` operator tests whether an address is strictly contained by a subnet. `192.168.1.10` and `192.168.1.11` are hosts inside `192.168.1.0/24`, and `192.168.1.0/24` is equal to the subnet (the `<<=` operator would include equality). `10.0.0.5/32` is outside the subnet and is excluded.

##### Example 2: Using INET functions

```sql
-- Step 1: Extract the host, netmask, and broadcast address.
SELECT
    host(ip)         AS host,
    masklen(ip)      AS netmask_bits,
    broadcast(ip)    AS broadcast,
    network(ip)      AS network
FROM servers
WHERE hostname = 'web-01';
```

**Expected output:**

```text
   host     | netmask_bits |   broadcast   |   network
------------+--------------+---------------+-------------
 192.168.1.10 |           32 | 192.168.1.10/32 | 192.168.1.10/32
(1 row)
```

**Why this output appears**: `host(ip)` returns the address without the netmask. `masklen(ip)` returns 32 because no subnet was specified (single host). `broadcast(ip)` computes the broadcast address for the address's network, which for a `/32` is the address itself. `network(ip)` returns the network part, also the address itself for a `/32`.

#### Real-World Cases

- **Server inventory**: A `servers` table stores each server's IP address as `inet`, enabling subnet‑based queries such as “all servers in the 10.0.0.0/8 range”.
- **Access control lists**: An `acl` table stores allowed client IPs as `inet`; a login check runs `WHERE client_ip <<= inet '10.0.0.0/8'` to test whether the client is inside the internal network.
- **Network monitoring**: A `devices` table stores management IPs and uses `inet_same_family()` to separate IPv4 and IPv6 device groups.

---

### 2. `CIDR`

#### Definitions

**Core Definition**
`CIDR` is a PostgreSQL type that stores an IPv4 or IPv6 network specification in Classless Internet Domain Routing notation, rejecting input that contains host bits to the right of the netmask.

**Technical Definition**
The `cidr` type holds an IPv4 or IPv6 network specification. Input and output formats follow Classless Internet Domain Routing conventions. The format is `address/y`, where `address` is the network's lowest address represented as an IPv4 or IPv6 address, and `y` is the number of bits in the netmask. If `y` is omitted, it is calculated using assumptions from the older classful network numbering system, except it will be at least large enough to include all of the octets written in the input. It is an error to specify a network address that has bits set to the right of the specified netmask. In other words, `cidr` accepts only true network addresses; a host address with a subnet prefix (e.g., `192.168.1.5/24`) would be rejected because bits to the right of the netmask are non‑zero.

**Beginner-Friendly Explanation**
`CIDR` is the strict network type. It accepts only network addresses—the base address of a subnet—not individual hosts. `192.168.1.0/24` is valid; `192.168.1.5/24` is not, because `.5` is a host bit outside the network portion. Use `CIDR` when a column should contain only networks, never hosts.

#### Purposes

- To store network definitions (subnets) with strict validation.
- To prevent accidental storage of host addresses in network columns.
- To support subnet arithmetic and containment queries on networks.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name CIDR
```

**Input examples (from the PostgreSQL documentation):**

| Input | Output | `abbrev()` |
|-------|--------|------------|
| `192.168.100.128/25` | `192.168.100.128/25` | `192.168.100.128/25` |
| `192.168/24` | `192.168.0.0/24` | `192.168.0/24` |
| `192.168/25` | `192.168.0.0/25` | `192.168.0.0/25` |
| `192.168.1` | `192.168.1.0/24` | `192.168.1/24` |
| `192.168` | `192.168.0.0/24` | `192.168.0/24` |
| `128.1` | `128.1.0.0/16` | `128.1/16` |
| `128` | `128.0.0.0/16` | `128.0/16` |

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `address/y` | Network address and netmask length. |
| `y` | Optional; inferred from the number of octets if omitted. |
| Storage | 7 bytes (IPv4) or 19 bytes (IPv6). |

**Syntax rules:**

- Bits to the right of the netmask must be zero; otherwise the input is rejected.
- If `y` is omitted, PostgreSQL infers it from the number of octets provided.
- `cidr` values can be cast implicitly to `inet`, and `inet` values can be cast to `cidr` (with host bits zeroed).

**Constraints and limitations:**

- `cidr` cannot store host addresses with non‑zero bits beyond the netmask.
- When casting `inet` to `cidr`, host bits are silently zeroed rather than raising an error.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing networks and testing containment

```sql
-- Step 1: Create a table with a CIDR column.
CREATE TABLE subnets (
    subnet_id serial PRIMARY KEY,
    label     text NOT NULL,
    network   cidr NOT NULL
);

-- Step 2: Insert valid network addresses.
INSERT INTO subnets (label, network) VALUES
    ('office',  '192.168.1.0/24'),
    ('lab',     '10.0.0.0/16'),
    ('dmz',     '203.0.113.0/24');

-- Step 3: Test whether a given host is inside a network.
SELECT label
FROM subnets
WHERE network >>= inet '192.168.1.50';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 3
 label
--------
 office
(1 row)
```

**Why this output appears**: The `>>=` operator tests whether the left subnet contains or equals the right address. `192.168.1.0/24` contains `192.168.1.50`, so the `office` subnet matches. The other subnets do not contain that address.

##### Example 2: Demonstrating the strictness of CIDR

```sql
-- Step 1: Attempt to insert a host address into a CIDR column.
INSERT INTO subnets (label, network) VALUES ('invalid', '192.168.1.5/24');
```

**Expected output:**

```text
ERROR:  invalid cidr value: "192.168.1.5/24"
DETAIL:  Value has bits set to right of mask.
```

**Why this output appears**: `cidr` rejects network addresses with bits set to the right of the netmask. `192.168.1.5` has host bits (`.5`) outside the `/24` network portion, so PostgreSQL raises an error rather than silently zeroing the bits.

#### Real-World Cases

- **Network inventory**: A `subnets` table stores each network as `cidr`, guaranteeing that only valid network addresses are recorded.
- **Firewall rules**: A `firewall_rules` table stores source and destination networks as `cidr`; queries use `>>=` to test whether a packet's address falls within a rule's network.
- **IP address management (IPAM)**: A `vlans` table associates each VLAN with a `cidr` network and uses `masklen()` and `broadcast()` to compute usable host ranges.

---

### 3. `MACADDR`

#### Definitions

**Core Definition**
`MACADDR` is a PostgreSQL type that stores 6‑byte MAC addresses in the traditional EUI‑48 format, as used by Ethernet hardware.

**Technical Definition**
The `macaddr` type stores MAC addresses, known for example from Ethernet card hardware addresses (although MAC addresses are used for other purposes as well). It occupies 6 bytes and accepts addresses in several formats, including `08:00:2b:01:02:03`, `08-00-2b-01-02-03`, and `08002b:010203`. The type supports the usual comparison operators as well as bitwise logical operators `~`, `&`, and `|`, and the `trunc()` function, which sets the last 3 bytes to zero so that the remaining prefix can be associated with a particular manufacturer.

**Beginner-Friendly Explanation**
`MACADDR` is the container for traditional 6‑byte hardware addresses—the kind you see on Ethernet cards, written like `08:00:2b:01:02:03`. PostgreSQL validates the format, stores it compactly, and lets you truncate the address to its manufacturer prefix.

#### Purposes

- To store MAC addresses with validation instead of plain text.
- To support network device inventory and hardware address tracking.
- To enable manufacturer‑prefix lookups using `trunc()`.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name MACADDR
```

**Accepted input formats:**

```text
08:00:2b:01:02:03
08-00-2b-01-02-03
08002b:010203
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `macaddr` | 6‑byte EUI‑48 MAC address. |
| Storage | 6 bytes. |
| Functions | `trunc()`, bitwise `~`, `&`, `|`. |

**Syntax rules:**

- Multiple separator styles are accepted on input; output is normalised to a canonical form.
- Comparison operators follow the usual ordering rules for the type.
- `trunc(macaddr)` sets the last 3 bytes to zero, leaving the 3‑byte manufacturer prefix.

**Constraints and limitations:**

- `macaddr` stores only 6‑byte EUI‑48 addresses; use `macaddr8` for 8‑byte EUI‑64 addresses.
- MAC addresses are not guaranteed to be globally unique in all contexts (e.g., locally administered addresses).

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing and truncating MAC addresses

```sql
-- Step 1: Create a table with a MACADDR column.
CREATE TABLE devices (
    device_id serial PRIMARY KEY,
    name      text NOT NULL,
    mac       macaddr NOT NULL
);

-- Step 2: Insert MAC addresses in different formats.
INSERT INTO devices (name, mac) VALUES
    ('printer', '08:00:2b:01:02:03'),
    ('switch',  '08-00-2b:aa:bb:cc');

-- Step 3: Truncate to the manufacturer prefix.
SELECT name, mac, trunc(mac) AS manufacturer_prefix
FROM devices;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
  name   |        mac        | manufacturer_prefix
---------+-------------------+---------------------
 printer | 08:00:2b:01:02:03 | 08:00:2b:00:00:00
 switch  | 08:00:2b:aa:bb:cc | 08:00:2b:00:00:00
(2 rows)
```

**Why this output appears**: `trunc(macaddr)` sets the last 3 bytes to zero. Both devices share the same manufacturer prefix `08:00:2b`, which can be looked up in an external OUI (Organizationally Unique Identifier) database to identify the manufacturer.

#### Real-World Cases

- **Device inventory**: A `devices` table stores each device's MAC address as `macaddr`, enabling validation and manufacturer‑prefix queries.
- **DHCP reservations**: A `dhcp_reservations` table maps MAC addresses to fixed IP addresses, using `macaddr` for validated storage.
- **Network access control**: A `allowed_devices` table stores MAC addresses of authorised devices; a login check compares the client's MAC against the table.

---

### 4. `MACADDR8`

#### Definitions

**Core Definition**
`MACADDR8` is a PostgreSQL type that stores 8‑byte MAC addresses in EUI‑64 format.

**Technical Definition**
The `macaddr8` type stores MAC addresses in EUI‑64 format, which is 8 bytes long and is used, for example, in IPv6 link‑local addresses and some modern hardware. It occupies 8 bytes and supports the same comparison and bitwise operators as `macaddr`. The function `trunc(macaddr8)` sets the last 5 bytes to zero, leaving the 3‑byte manufacturer prefix, and `macaddr8_set7bit()` sets the 7th bit to one, creating a modified EUI‑64 address for inclusion in an IPv6 address.

**Beginner-Friendly Explanation**
`MACADDR8` is the container for newer, 8‑byte hardware addresses—the EUI‑64 format. These are longer than traditional MAC addresses and are used in IPv6 and some modern devices. `MACADDR8` gives you the same validation and tools as `MACADDR`, but for the 8‑byte format.

#### Purposes

- To store EUI‑64 MAC addresses with validation.
- To support IPv6 link‑local address generation via `macaddr8_set7bit()`.
- To maintain a device inventory that includes modern 8‑byte hardware addresses.

#### Syntax Rules and Structure

**Column declaration:**

```sql
column_name MACADDR8
```

**Example input:**

```text
08:00:2b:ff:fe:01:02:03
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `macaddr8` | 8‑byte EUI‑64 MAC address. |
| Storage | 8 bytes. |
| Functions | `trunc()`, `macaddr8_set7bit()`, bitwise `~`, `&`, `|`. |

**Syntax rules:**

- `macaddr8` accepts 8‑byte addresses in colon‑separated hex notation.
- `trunc(macaddr8)` zeroes the last 5 bytes, leaving the 3‑byte manufacturer prefix.
- `macaddr8_set7bit()` sets the 7th bit to one for modified EUI‑64 compatibility.

**Constraints and limitations:**

- `macaddr8` is not interchangeable with `macaddr`; the two store different lengths.
- Not all devices use EUI‑64; traditional Ethernet still predominantly uses EUI‑48.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Storing EUI‑64 addresses and generating a modified EUI‑64

```sql
-- Step 1: Create a table with a MACADDR8 column.
CREATE TABLE ipv6_devices (
    device_id serial PRIMARY KEY,
    name      text NOT NULL,
    mac       macaddr8 NOT NULL
);

-- Step 2: Insert an EUI-64 MAC address.
INSERT INTO ipv6_devices (name, mac)
VALUES ('sensor-01', '08:00:2b:ff:fe:01:02:03');

-- Step 3: Generate the modified EUI-64 for IPv6 use.
SELECT name, mac, macaddr8_set7bit(mac) AS modified_eui64
FROM ipv6_devices;
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 1
   name    |          mac          |     modified_eui64
-----------+-----------------------+-------------------------
 sensor-01 | 08:00:2b:ff:fe:01:02:03 | 0a:00:2b:ff:fe:01:02:03
(1 row)
```

**Why this output appears**: `macaddr8_set7bit()` flips the 7th bit (the universal/local bit) from 0 to 1, converting the EUI‑64 address into the modified EUI‑64 form used in IPv6 link‑local address generation.

#### Real-World Cases

- **IPv6 device tracking**: An `ipv6_devices` table stores EUI‑64 MAC addresses for devices that generate IPv6 link‑local addresses from their hardware addresses.
- **Modern hardware inventory**: A `assets` table uses `macaddr8` for newer network interface cards that report 8‑byte addresses.
- **Network automation**: A provisioning system derives IPv6 interface identifiers from `macaddr8` values using `macaddr8_set7bit()`.

---

### 5. Network Address Best Practices

#### Definitions

**Core Definition**
Network address best practices are the guidelines for choosing between `inet` and `cidr`, using the appropriate MAC address type, indexing network columns, and leveraging PostgreSQL's specialised operators and functions for validation and subnet arithmetic.

**Technical Definition**
Best practices include: (1) **use `inet` for host addresses and `cidr` for networks**, because `inet` accepts host bits while `cidr` rejects them, preventing accidental storage of hosts in network columns; (2) **always use the specialised network types instead of `text` or `varchar`**, because the built‑in types provide input validation and specialised operators that text types lack; (3) **use `macaddr` for EUI‑48 and `macaddr8` for EUI‑64**, matching the type to the hardware address format; (4) **index `inet` and `cidr` columns** when equality or ordering queries are common, and use appropriate operator classes for containment queries; (5) **prefer containment operators (`<<`, `>>`, `&&`, `<<=`, `>>=`) over string `LIKE` patterns** for subnet membership tests, because they operate on the binary network structure rather than text; and (6) **use `host()`, `network()`, `broadcast()`, `netmask()`, and `masklen()`** to extract and compute network properties instead of parsing addresses manually.

**Beginner-Friendly Explanation**
Network address best practices are the habits that keep IP and MAC data correct and queryable. Use `INET` for hosts and `CIDR` for networks so the database enforces the distinction. Never store IP addresses as plain text—PostgreSQL's network types validate them and give you powerful query tools. Match `MACADDR` to 6‑byte addresses and `MACADDR8` to 8‑byte addresses. Index your network columns if you search them often, and use the built‑in subnet operators instead of string matching.

#### Purposes

- To prevent invalid or semantically wrong network data from entering the database.
- To enable efficient subnet‑membership queries using binary operators rather than string manipulation.
- To choose the right type for each address family and format.
- To extract network properties reliably with built‑in functions.

#### Syntax Rules and Structure

**Best‑practice schema pattern:**

```sql
-- Use INET for hosts, CIDR for networks, and MACADDR/MACADDR8 for MACs.
CREATE TABLE network_assets (
    asset_id   serial PRIMARY KEY,
    hostname   text NOT NULL,
    host_ip    inet NOT NULL,          -- host address
    network    cidr NOT NULL,          -- network address
    mac        macaddr,                 -- EUI-48
    mac64      macaddr8                 -- EUI-64 (if applicable)
);

-- Index for subnet queries.
CREATE INDEX idx_assets_network ON network_assets USING gist (network inet_ops);
```

**Component breakdown:**

| Practice | Implementation |
|----------|----------------|
| Host vs. network | `inet` for hosts; `cidr` for networks |
| MAC format | `macaddr` for EUI‑48; `macaddr8` for EUI‑64 |
| Validation | Use network types, not `text`/`varchar` |
| Containment queries | `<<`, `>>`, `&&`, `<<=`, `>>=` |
| Property extraction | `host()`, `network()`, `broadcast()`, `netmask()`, `masklen()` |
| Indexing | B‑tree for equality/order; GiST with `inet_ops` for containment |

**Syntax rules:**

- `cidr` values can be cast to `inet` implicitly; `inet` to `cidr` casts zero host bits.
- Containment operators consider only the network parts of the addresses (ignoring bits to the right of the netmask).
- `macaddr` and `macaddr8` support bitwise logical operators `~`, `&`, `|`.

**Constraints and limitations:**

- `cidr` rejects host bits; use `inet` if host addresses with subnets must be stored.
- `macaddr` and `macaddr8` are distinct types; choose based on the hardware address length.
- Containment operators do not directly index with B‑tree; GiST operator classes may be needed for large tables.

#### Multiple Annotated Complete Step-by-Step Code Examples

##### Example 1: Best‑practice network asset table with indexing and containment queries

```sql
-- Step 1: Create a table using the appropriate network types.
CREATE TABLE network_assets (
    asset_id   serial PRIMARY KEY,
    hostname   text NOT NULL,
    host_ip    inet NOT NULL,
    network    cidr NOT NULL,
    mac        macaddr
);

-- Step 2: Insert hosts and their networks.
INSERT INTO network_assets (hostname, host_ip, network, mac) VALUES
    ('web-01', '192.168.1.10', '192.168.1.0/24', '08:00:2b:01:02:03'),
    ('db-01',  '10.0.0.5',    '10.0.0.0/16',    '08:00:2b:aa:bb:cc');

-- Step 3: Query hosts in a given network using containment.
SELECT hostname, host_ip
FROM network_assets
WHERE network >>= inet '192.168.1.50';

-- Step 4: Extract the network and broadcast address.
SELECT hostname,
       network(host_ip) AS network_part,
       broadcast(host_ip) AS broadcast_addr
FROM network_assets
WHERE hostname = 'web-01';
```

**Expected output:**

```text
CREATE TABLE
INSERT 0 2
 hostname |   host_ip
----------+--------------
 web-01   | 192.168.1.10
(1 row)

 hostname | network_part | broadcast_addr
----------+--------------+----------------
 web-01   | 192.168.1.10/32 | 192.168.1.10/32
(1 row)
```

**Why this output appears**: The `>>=` operator tests whether the stored network contains the queried address; `192.168.1.0/24` contains `192.168.1.50`, so `web-01` is returned. For `web-01`, `host_ip` was inserted without a subnet, so it is treated as a `/32` host; `network()` and `broadcast()` return the address itself.

#### Real-World Cases

- **IPAM (IP Address Management)**: A `network_assets` table stores hosts as `inet`, networks as `cidr`, and MAC addresses as `macaddr`, with containment queries for subnet allocation and device tracking.
- **Access control**: A `login_attempts` table stores client IPs as `inet`; a security query tests `WHERE client_ip <<= inet '10.0.0.0/8'` to flag internal versus external logins.
- **Compliance auditing**: A `hardware_inventory` table stores `macaddr8` values for modern devices and uses `trunc()` to group devices by manufacturer prefix.

---

## References

- PostgreSQL Documentation – Network Address Types – https://www.postgresql.org/docs/15/datatype-net-types.html
- PostgreSQL Documentation – Network Address Functions and Operators – https://www.postgresql.org/docs/16/functions-net.html
- PostgreSQL Documentation – Data Types (overview) – https://www.postgresql.org/docs/current/datatype.html
- PostgreSQL Documentation – `inet` vs. `cidr` – https://www.postgresql.org/docs/current/datatype-net-types.html#DATATYPE-INET-VS-CIDR
- Compile‑N‑Run – PostgreSQL Network Address Types – https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/database/postgresql/10-postgresql-data-types/8-postgresql-network-address-types.mdx