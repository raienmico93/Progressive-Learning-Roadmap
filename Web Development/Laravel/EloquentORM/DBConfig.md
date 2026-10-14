# Laravel Database Configuration: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel Database Configuration is the framework's system for defining, managing, and optimizing connections to relational database systems, enabling applications to interact with data through a unified API.

**Technical Definition**

Laravel's database configuration is managed through the `config/database.php` configuration file, with values typically driven by environment variables defined in `.env`. The framework supports five first-party relational databases: MariaDB, MySQL, PostgreSQL, SQLite, and SQL Server. Connections are defined as arrays under the `connections` key, with a `default` connection specified for general use. Laravel's database layer (`Illuminate\Database`) provides a fluent query builder, the Eloquent ORM, a schema builder, and support for advanced features including read/write connection splitting, multiple database connections, connection pooling for PostgreSQL, database transactions with automatic deadlock retry, and connection URL configuration for managed database providers.

**Beginner-Friendly Explanation**

Laravel's database configuration is how you tell your application where to find its database and how to connect to it. Think of it like setting up a phone book: you write down the address (host), the name (database), and the password (credentials) for each database you want to use. Once configured, Laravel handles all the technical details of connecting, querying, and disconnecting. The configuration lives in a file called `config/database.php`, but you usually control it through environment variables in a `.env` file, which keeps sensitive credentials out of your code.

---

### Key Characteristics

- **Five First-Party Drivers**: MariaDB 10.3+, MySQL 5.7+, PostgreSQL 10.0+, SQLite 3.26.0+, and SQL Server 2017+ are supported out of the box.
- **Environment-Driven Configuration**: Most connection settings are configured via `.env` variables (`DB_CONNECTION`, `DB_HOST`, `DB_PORT`, etc.), with sensible defaults in `config/database.php`.
- **Multiple Connection Support**: Unlimited named connections can be defined, each with its own driver and credentials.
- **Read/Write Splitting**: Laravel natively supports separate database hosts for read (SELECT) and write (INSERT/UPDATE/DELETE) operations, with a `sticky` option to ensure read-after-write consistency.
- **Connection URLs**: A single `DB_URL` environment variable can configure an entire connection, simplifying deployment on managed platforms like AWS, Heroku, and Laravel Cloud.
- **Connection Pooling**: PostgreSQL connections support transaction-mode poolers (PgBouncer, AWS RDS Proxy, Neon) via the `pooled` configuration option.
- **Transactions with Deadlock Retry**: `DB::transaction()` accepts an `attempts` parameter that automatically retries transactions on deadlock, with built-in rollback on exception.
- **Connection Management**: Connections are lazily created and can be explicitly disconnected, purged, or reconnected via the `DB` facade.
- **Configuration Caching**: `php artisan config:cache` caches the entire configuration, including database settings, for faster bootstrapping in production.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** installed via Composer.
- **A running database server** (MySQL, PostgreSQL, SQLite, or SQL Server).
- **Required PHP extensions**: `pdo`, `pdo_mysql` (MySQL), `pdo_pgsql` (PostgreSQL), `pdo_sqlite` (SQLite), `sqlsrv` and `pdo_sqlsrv` (SQL Server).
- **Basic understanding of environment variables** and the `.env` file.

---

### Related Programming Areas

- **Eloquent ORM**: Database configuration determines how Eloquent models connect and query.
- **Migrations**: Schema changes are applied through configured connections.
- **Seeders and Factories**: Data population uses the configured database connection.
- **Queue System**: Queue workers interact with the database for job storage and processing.
- **Caching**: Database-backed cache and session drivers use the configured connection.
- **Testing**: Tests can use a separate database connection (e.g., SQLite in-memory) for isolation.
- **Deployment**: Connection URLs and environment variables simplify production configuration.

---

## Core Concepts

---

### 1. Database Drivers

#### Definitions

**Core Definition**

A database driver is the underlying PHP extension and Laravel connector that enables communication between the application and a specific database system.

**Technical Definition**

Laravel supports five first-party database drivers, each mapped to a PHP PDO extension: `mysql` (MySQL/MariaDB), `pgsql` (PostgreSQL), `sqlite` (SQLite), and `sqlsrv` (SQL Server). The driver is specified in the `driver` key of a connection configuration. Each driver has its own set of supported configuration options (e.g., `charset`, `collation`, `sslmode` for PostgreSQL, `encrypt` for SQL Server).

**Beginner-Friendly Explanation**

A database driver is the "translator" between your Laravel application and your database. Different databases (MySQL, PostgreSQL, etc.) speak slightly different languages, and the driver handles the translation. You just tell Laravel which database you are using by setting the `DB_CONNECTION` variable, and Laravel picks the right driver.

#### Purposes

- To provide a unified interface for interacting with different database systems.
- To abstract away the differences between database engines.
- To enable switching databases without rewriting application code.
- To support the specific features and optimizations of each database system.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/database.php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => env('DB_CHARSET', 'utf8mb4'),
        'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
        'prefix' => '',
        'strict' => true,
        'engine' => null,
    ],
    'pgsql' => [
        'driver' => 'pgsql',
        'host' => env('DB_HOST', '127.0.0.1'),
        'port' => env('DB_PORT', '5432'),
        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => env('DB_CHARSET', 'utf8'),
        'prefix' => '',
        'search_path' => 'public',
        'sslmode' => 'prefer',
    ],
    'sqlite' => [
        'driver' => 'sqlite',
        'url' => env('DB_URL'),
        'database' => env('DB_DATABASE', database_path('database.sqlite')),
        'prefix' => '',
        'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
    ],
    'sqlsrv' => [
        'driver' => 'sqlsrv',
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', '1433'),
        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        'charset' => env('DB_CHARSET', 'utf8'),
        'prefix' => '',
    ],
],
```

**Component Breakdown**

| Key | Description | Common to All Drivers |
|-----|-------------|----------------------|
| `driver` | The database driver (`mysql`, `pgsql`, `sqlite`, `sqlsrv`). | Yes |
| `host` | Database server hostname or IP. | Yes (except SQLite) |
| `port` | Database server port. | Yes (except SQLite) |
| `database` | Database name (or file path for SQLite). | Yes |
| `username` | Database username. | Yes (except SQLite) |
| `password` | Database password. | Yes (except SQLite) |
| `charset` | Character set. | MySQL, PostgreSQL, SQL Server |
| `collation` | Collation. | MySQL |
| `prefix` | Table prefix. | All |
| `sslmode` | SSL mode (PostgreSQL). | PostgreSQL |
| `foreign_key_constraints` | Enable foreign keys (SQLite). | SQLite |

**Syntax Rules**

- The `driver` key is required and must match a supported driver.
- Connection options are merged with driver-specific defaults.
- SQLite does not use `host`, `port`, `username`, or `password`.
- SQL Server requires the `sqlsrv` and `pdo_sqlsrv` PHP extensions.

**Constraints and Limitations**

- Not all configuration options apply to all drivers.
- SQLite is a file-based database and does not support concurrent writes at scale.
- SQL Server support requires additional PHP extensions and ODBC drivers.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: MySQL Configuration**

**`.env`**

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=app_user
DB_PASSWORD=secret_password
```

**Expected Output**

```
// Laravel connects to MySQL at 127.0.0.1:3306 using the laravel_app database
```

**Why This Output Occurs**

The `DB_CONNECTION=mysql` selects the MySQL driver, and the other environment variables provide the connection credentials. The `config/database.php` file reads these values via the `env()` helper.

---

**Example 2: SQLite Configuration**

**Setup**

```bash
touch database/database.sqlite
```

**`.env`**

```env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database/database.sqlite
```

**Expected Output**

```
// Laravel connects to the SQLite database file
```

**Why This Output Occurs**

SQLite databases are single files. The `DB_DATABASE` variable points to the absolute path of the file. Laravel connects to it without host, port, or credentials.

---

**Example 3: PostgreSQL Configuration**

**`.env`**

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_app
DB_USERNAME=postgres
DB_PASSWORD=secret
```

**Expected Output**

```
// Laravel connects to PostgreSQL at 127.0.0.1:5432
```

**Why This Output Occurs**

The `pgsql` driver uses port 5432 by default and supports PostgreSQL-specific options like `sslmode` and `search_path`.

---

**Example 4: SQL Server Configuration**

**`.env`**

```env
DB_CONNECTION=sqlsrv
DB_HOST=localhost
DB_PORT=1433
DB_DATABASE=laravel_app
DB_USERNAME=sa
DB_PASSWORD=YourStrongPassword
```

**Expected Output**

```
// Laravel connects to SQL Server using the sqlsrv driver
```

**Why This Output Occurs**

The `sqlsrv` driver communicates with SQL Server through the `pdo_sqlsrv` PHP extension. The default port is 1433.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Environment Database Setup**

A development team uses SQLite for local development (zero configuration), MySQL for staging, and PostgreSQL for production. The `.env` file is environment-specific, and the `DB_CONNECTION` variable switches the driver without code changes.

**Explanation**: Driver abstraction enables seamless transitions between database systems across environments.

**Case 2: Legacy System Integration**

An application must connect to a legacy SQL Server database for reporting while using MySQL for its main application data. Two connections (`mysql` and `sqlsrv`) are defined, and Eloquent models specify which connection to use via the `$connection` property.

**Explanation**: Multiple drivers can coexist within a single Laravel application.

---

### 2. Database Configuration

#### Definitions

**Core Definition**

Database configuration is the set of settings in `config/database.php` and `.env` that define how Laravel connects to and interacts with database systems.

**Technical Definition**

Laravel's database configuration is stored in `config/database.php`, which returns an array with three top-level keys: `default` (the default connection name), `connections` (an array of named connection configurations), and `migrations` (migration repository settings). Most connection options use the `env()` helper to read values from the `.env` file, with sensible defaults provided for development. The configuration file is cached in production via `php artisan config:cache`.

**Beginner-Friendly Explanation**

The database configuration is where you tell Laravel everything it needs to know about your database: what type it is, where it lives, and how to log in. You write most of this information in a `.env` file, which keeps your passwords and other sensitive details out of your code repository.

#### Purposes

- To define how Laravel connects to each database system.
- To specify which connection is used by default.
- To keep sensitive credentials out of version control via environment variables.
- To enable environment-specific configuration (local, staging, production).
- To support multiple connections for different databases or services.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/database.php
return [
    'default' => env('DB_CONNECTION', 'sqlite'),
    'connections' => [
        // Connection definitions...
    ],
    'migrations' => [
        'table' => 'migrations',
        'update_date_on_publish' => true,
    ],
    'redis' => [
        // Redis configuration...
    ],
];
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `default` | The default connection name (defaults to `DB_CONNECTION` or `sqlite`). |
| `connections` | Array of named connection configurations. |
| `migrations` | Migration repository settings. |
| `redis` | Redis connection settings (for cache/queue). |

**Syntax Rules**

- The `default` key determines which connection is used when none is specified.
- Each connection in `connections` must have a `driver` key.
- The `env()` helper reads values from `.env`, falling back to the second argument.
- Configuration is cached in production; changes require `config:cache` to be re-run.

**Constraints and Limitations**

- The `config/database.php` file should not contain hardcoded credentials.
- Configuration caching in production means `.env` changes require a cache refresh.
- The `default` connection must exist in the `connections` array.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Default Connection Configuration**

**`.env`**

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=root
DB_PASSWORD=
```

**`config/database.php`**

```php
'default' => env('DB_CONNECTION', 'sqlite'),
```

**Expected Output**

```
// The 'mysql' connection is used by default
```

**Why This Output Occurs**

The `default` key reads `DB_CONNECTION` from `.env`, which is set to `mysql`. When `DB::table('users')` is called without specifying a connection, MySQL is used.

---

**Example 2: Configuration Caching**

**Command**

```bash
php artisan config:cache
```

**Expected Output (Terminal)**

```
Configuration cached successfully.
```

**Why This Output Occurs**

Laravel compiles all configuration files (including `config/database.php`) into a single cached file (`bootstrap/cache/config.php`), which is loaded instead of the individual files on each request. This improves performance in production.

---

#### Real-World Cases with Explanation

**Case 1: Laravel Cloud Deployment**

Laravel Cloud injects `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD`, and `DB_DATABASE` environment variables into the application. The `config/database.php` file reads these values automatically, requiring no manual configuration.

**Explanation**: Environment-driven configuration simplifies deployment on managed platforms.

---

### 3. `.env` Database Variables

#### Definitions

**Core Definition**

`.env` database variables are environment-specific configuration values stored in the `.env` file, used to define database connection settings without hardcoding them in source code.

**Technical Definition**

Laravel uses the `vlucas/phpdotenv` library to load variables from `.env` into PHP's `$_ENV` and `$_SERVER` superglobals, accessible via the `env()` helper. Database-related variables include `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_CHARSET`, `DB_COLLATION`, `DB_SOCKET`, `DB_URL`, and `DB_FOREIGN_KEYS`. The `.env` file should never be committed to version control; a `.env.example` file with placeholder values is committed instead.

**Beginner-Friendly Explanation**

The `.env` file is a simple text file that holds your database passwords and other settings. It is like a secret diary that only your application can read. You keep it out of your code repository so that your passwords do not end up on the internet.

#### Purposes

- To keep sensitive credentials out of version control.
- To allow different configurations for different environments (local, staging, production).
- To simplify deployment by changing environment variables instead of code.
- To provide a single source of truth for database connection settings.

#### Syntax Rules and Structure

**Complete General Syntax**

```env
# Database connection driver
DB_CONNECTION=mysql

# Connection host and port
DB_HOST=127.0.0.1
DB_PORT=3306

# Database name and credentials
DB_DATABASE=laravel_app
DB_USERNAME=root
DB_PASSWORD=

# Optional: character set and collation
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci

# Optional: Unix socket (MySQL)
DB_SOCKET=

# Optional: full connection URL
# DB_URL=mysql://user:pass@host:3306/database

# Optional: SQLite foreign key enforcement
DB_FOREIGN_KEYS=true
```

**Component Breakdown**

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_CONNECTION` | Database driver | `sqlite` |
| `DB_HOST` | Database host | `127.0.0.1` |
| `DB_PORT` | Database port | Driver-specific |
| `DB_DATABASE` | Database name (or file path for SQLite) | `laravel` |
| `DB_USERNAME` | Database username | `root` |
| `DB_PASSWORD` | Database password | (empty) |
| `DB_CHARSET` | Character set | `utf8mb4` |
| `DB_COLLATION` | Collation | `utf8mb4_unicode_ci` |
| `DB_SOCKET` | Unix socket path (MySQL) | (empty) |
| `DB_URL` | Full connection URL | (empty) |
| `DB_FOREIGN_KEYS` | Enable foreign keys (SQLite) | `true` |

**Syntax Rules**

- Variables are written in `KEY=VALUE` format.
- Values with spaces or special characters should be wrapped in double quotes.
- Comments start with `#`.
- The `.env` file should be listed in `.gitignore`.
- The `.env.example` file should contain placeholder values.

**Constraints and Limitations**

- `.env` variables are not available in production if configuration caching is enabled; the cached values are used instead.
- Boolean values should be written as `true` or `false` (lowercase).
- The `DB_URL` variable, when set, overrides individual connection variables.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Standard MySQL `.env` Configuration**

**`.env`**

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=app_user
DB_PASSWORD=secret_password
```

**Expected Output**

```
// Laravel connects to MySQL using the specified credentials
```

**Why This Output Occurs**

Each `DB_*` variable is read by `config/database.php` via the `env()` helper. The values are used to construct the connection.

---

**Example 2: SQLite `.env` Configuration**

```env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database/database.sqlite
DB_FOREIGN_KEYS=true
```

**Expected Output**

```
// Laravel connects to the SQLite file with foreign key constraints enabled
```

**Why This Output Occurs**

SQLite uses a file path instead of a database name. The `DB_FOREIGN_KEYS` variable enables foreign key enforcement.

---

**Example 3: Connection URL Configuration**

```env
DB_URL=mysql://app_user:secret_password@127.0.0.1:3306/laravel_app
```

**Expected Output**

```
// Laravel parses the URL and configures the connection automatically
```

**Why This Output Occurs**

The `DB_URL` variable provides all connection details in a single string, which is parsed by Laravel's connection factory. This is useful for managed database providers that supply a single connection URL.

---

#### Real-World Cases with Explanation

**Case 1: Heroku Deployment**

Heroku provides a `DATABASE_URL` environment variable containing the full connection string for the provisioned PostgreSQL database. The application sets `DB_URL` to this value (or uses `DATABASE_URL` directly if configured), and Laravel parses it automatically.

**Explanation**: Connection URLs simplify deployment on platforms that provide a single database URL.

---

### 4. Multiple Database Connections

#### Definitions

**Core Definition**

Multiple database connections allow a Laravel application to connect to more than one database simultaneously, each defined as a named connection in `config/database.php`.

**Technical Definition**

Additional connections are added as new keys under the `connections` array. The `DB::connection('name')` method retrieves a specific connection. Eloquent models can specify their connection via the `protected $connection` property. The `on()` method allows runtime connection switching for individual queries. The `default` connection is used when no connection is specified.

**Beginner-Friendly Explanation**

Sometimes you need to talk to more than one database at the same time. For example, your main application data might be in MySQL, but you also need to read from a legacy SQL Server database. Multiple connections let you define each database once and switch between them as needed.

#### Purposes

- To connect to multiple databases with different drivers.
- To support multi-tenant applications with separate databases per tenant.
- To integrate with legacy systems or external data sources.
- To separate read and write operations across different database servers.
- To isolate data for security, performance, or organizational reasons.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/database.php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', '127.0.0.1'),
        // ...
    ],
    'mysql_legacy' => [
        'driver' => 'mysql',
        'host' => env('DB_LEGACY_HOST', '127.0.0.1'),
        'database' => env('DB_LEGACY_DATABASE', 'legacy_db'),
        // ...
    ],
    'pgsql_reporting' => [
        'driver' => 'pgsql',
        'host' => env('DB_REPORTING_HOST', '127.0.0.1'),
        // ...
    ],
],
```

```php
// Using a specific connection
$users = DB::connection('mysql_legacy')->select('select * from users');

// Model connection
class LegacyUser extends Model
{
    protected $connection = 'mysql_legacy';
}

// Runtime connection
$users = User::on('mysql_legacy')->get();
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `DB::connection('name')` | Returns the specified connection instance. |
| `$connection` property | Sets the default connection for a model. |
| `on('name')` | Switches connection for a single query. |
| `DB::purge('name')` | Disconnects and reconnects the connection. |

**Syntax Rules**

- Each connection must have a unique name and a `driver` key.
- The `default` connection is used when no connection is specified.
- Models can override the connection via `$connection`.
- `on()` is chainable and applies to the next query.

**Constraints and Limitations**

- Cross-database joins are not supported natively; use separate queries and merge results.
- Transactions across connections are not supported.
- Connection configuration is cached in production.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Defining Multiple Connections**

**`config/database.php`**

```php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', '127.0.0.1'),
        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
    ],
    'mysql_analytics' => [
        'driver' => 'mysql',
        'host' => env('DB_ANALYTICS_HOST', '127.0.0.1'),
        'database' => env('DB_ANALYTICS_DATABASE', 'analytics'),
        'username' => env('DB_ANALYTICS_USERNAME', 'analytics_user'),
        'password' => env('DB_ANALYTICS_PASSWORD', ''),
    ],
],
```

**Usage**

```php
// Default connection
$users = DB::table('users')->get();

// Analytics connection
$pageViews = DB::connection('mysql_analytics')->table('page_views')->get();

// Model with custom connection
class PageView extends Model
{
    protected $connection = 'mysql_analytics';
    protected $table = 'page_views';
}
```

**Expected Output**

```
// Users are retrieved from the default (mysql) database
// Page views are retrieved from the analytics database
```

**Why This Output Occurs**

The `DB::connection('mysql_analytics')` method returns a connection instance configured for the analytics database. Models with the `$connection` property always use that connection.

---

**Example 2: Runtime Connection Switching**

```php
$users = User::on('mysql_analytics')->where('active', true)->get();
```

**Expected Output**

```
// The User query is executed against the analytics database connection
```

**Why This Output Occurs**

The `on()` method switches the connection for the duration of the query chain.

---

#### Real-World Cases with Explanation

**Case 1: Multi-Tenant SaaS Application**

A multi-tenant SaaS application maintains a central database for tenant metadata and a separate database for each tenant's data. Middleware resolves the tenant from the request and switches the connection using `DB::setDefaultConnection()` or model-specific connections.

**Explanation**: Multiple connections enable data isolation between tenants.

**Case 2: Legacy System Integration**

An application uses MySQL for its main data and SQL Server for a legacy reporting system. Models for legacy data specify the `sqlsrv` connection, while the rest of the application uses the default MySQL connection.

**Explanation**: Multiple connections allow integration with external systems without disrupting the primary application.

---

### 5. Read/Write Connections

#### Definitions

**Core Definition**

Read/write connections allow Laravel to use separate database hosts for read (SELECT) and write (INSERT, UPDATE, DELETE) operations, improving performance by distributing load across multiple database servers.

**Technical Definition**

The `read` and `write` keys are added to a connection configuration, each containing an array of host overrides. Laravel merges the main connection configuration with the `read` or `write` overrides, so only the overridden values (typically `host`) need to be specified. The `sticky` option ensures that after a write operation during a request, all subsequent reads use the write connection, guaranteeing read-after-write consistency.

**Beginner-Friendly Explanation**

Read/write splitting is like having two checkout lanes at a supermarket: one for people buying things (writes) and one for people just browsing (reads). This keeps the queues shorter and the store running faster. Laravel automatically sends SELECT queries to the read database and INSERT/UPDATE/DELETE queries to the write database.

#### Purposes

- To distribute database load across multiple servers.
- To improve performance for read-heavy applications.
- To enable database replication architectures (primary-replica).
- To ensure data consistency with the `sticky` option.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
'mysql' => [
    'read' => [
        'host' => [
            '192.168.1.1',
            '192.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,
    'driver' => 'mysql',
    'database' => 'database',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `read` | Array of read connection overrides (host, etc.). |
| `write` | Array of write connection overrides (host, etc.). |
| `sticky` | If `true`, reads after a write in the same request use the write connection. |
| Multiple hosts | An array of hosts for load balancing. |

**Syntax Rules**

- Only the overridden values (usually `host`) need to be specified in `read` and `write`.
- The rest of the configuration is inherited from the main connection array.
- Multiple hosts in an array are randomly selected per request.
- The `sticky` option defaults to `false`.

**Constraints and Limitations**

- Read replicas may have replication lag, causing stale data if `sticky` is not enabled.
- Schema operations (migrations) always use the write connection.
- Transactions always use the write connection.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Read/Write Configuration**

```php
'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
    ],
    'write' => [
        'host' => '192.168.1.2',
    ],
    'driver' => 'mysql',
    'database' => 'laravel_app',
    'username' => 'app_user',
    'password' => 'secret',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
],
```

**Expected Output**

```
// SELECT queries use 192.168.1.1 (read replica)
// INSERT/UPDATE/DELETE queries use 192.168.1.2 (primary)
```

**Why This Output Occurs**

Laravel inspects the query type and routes it to the appropriate connection. Reads go to the read host, writes to the write host.

---

**Example 2: Sticky Read/Write Configuration**

```php
'mysql' => [
    'read' => ['host' => '192.168.1.1'],
    'write' => ['host' => '192.168.1.2'],
    'sticky' => true,
    // ...
],
```

**Expected Output**

```
// After a write, all subsequent reads in the same request use the write connection
// This ensures that data written is immediately readable
```

**Why This Output Occurs**

The `sticky` option tracks whether a write has been performed during the request. If so, all subsequent reads use the write connection, bypassing the potentially stale read replica.

---

**Example 3: Multiple Read Hosts with Load Balancing**

```php
'read' => [
    'host' => [
        '192.168.1.1',
        '192.168.1.2',
        '192.168.1.3',
    ],
],
```

**Expected Output**

```
// A random read host is selected for each request
```

**Why This Output Occurs**

When an array of hosts is provided, Laravel randomly selects one for each request, distributing read load across multiple replicas.

---

#### Real-World Cases with Explanation

**Case 1: High-Traffic E-commerce Site**

An e-commerce platform receives thousands of read requests for product listings but fewer write requests for orders. Read/write splitting directs product queries to read replicas and order creation to the primary database, improving performance and scalability.

**Explanation**: Read/write splitting is essential for scaling read-heavy applications.

**Case 2: Database Replication with Sticky Reads**

A SaaS application uses a primary database for writes and two read replicas for SELECT queries. The `sticky` option is enabled to ensure that users see their own updates immediately after saving.

**Explanation**: The `sticky` option guarantees read-after-write consistency in replicated environments.

---

### 6. Connection Pooling

#### Definitions

**Core Definition**

Connection pooling is a technique for reusing a small set of database connections across many client requests, reducing the overhead of establishing new connections.

**Technical Definition**

PHP's traditional request lifecycle creates a new database connection for each request and closes it at the end. Connection poolers like PgBouncer, AWS RDS Proxy, and Neon sit between the application and the database, maintaining a pool of persistent connections that are reused across requests. Laravel 13.17+ supports PostgreSQL transaction-mode poolers via the `pooled` configuration option, which handles the incompatibility between transaction pooling and server-side prepared statements.

**Beginner-Friendly Explanation**

Connection pooling is like a carpool for database connections. Instead of every request creating its own connection (which takes time), a pooler keeps a few connections open and shares them. This is especially important for serverless and high-concurrency environments where many requests arrive at once.

#### Purposes

- To reduce the overhead of establishing new database connections.
- To limit the number of concurrent connections to the database server.
- To improve performance in serverless and high-concurrency environments.
- To enable connection reuse across PHP requests.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// config/database.php (Laravel 13.17+)
'pgsql' => [
    'driver' => 'pgsql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'postgres'),
    'password' => env('DB_PASSWORD', ''),
    'pooled' => true,
    // ...
],
```

**Component Breakdown**

| Key | Description |
|-----|-------------|
| `pooled` | Enables transaction-mode pooler compatibility (PostgreSQL). |
| PgBouncer port | Typically 6432 (instead of 5432). |
| `PDO::ATTR_EMULATE_PREPARES` | Set to `true` for transaction pooling mode. |

**Syntax Rules**

- The `pooled` option is available in Laravel 13.17+ for PostgreSQL connections.
- The connection should point to the pooler's port (e.g., 6432 for PgBouncer).
- Server-side prepared statements must be disabled for transaction pooling.

**Constraints and Limitations**

- Connection pooling is not supported for MySQL out of the box (requires ProxySQL or similar).
- Transaction pooling mode does not support session-level features (e.g., `SET` statements, advisory locks).
- The `pooled` option is version-specific (Laravel 13.17+).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: PgBouncer Configuration**

**`.env`**

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=6432
DB_DATABASE=laravel_app
DB_USERNAME=app_user
DB_PASSWORD=secret
```

**`config/database.php`**

```php
'pgsql' => [
    'driver' => 'pgsql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '6432'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'postgres'),
    'password' => env('DB_PASSWORD', ''),
    'pooled' => true,
    'charset' => 'utf8',
    'prefix' => '',
    'search_path' => 'public',
    'sslmode' => 'prefer',
],
```

**Expected Output**

```
// Laravel connects to PgBouncer on port 6432, which pools connections to PostgreSQL
// Server-side prepared statements are handled correctly
```

**Why This Output Occurs**

The `pooled => true` option instructs Laravel to configure the PDO connection for transaction pooling mode, disabling server-side prepared statements and handling value bindings correctly.

---

#### Real-World Cases with Explanation

**Case 1: Serverless PostgreSQL on Neon**

A serverless application uses Neon PostgreSQL with a connection pooler. The `pooled` option ensures that Laravel works correctly with Neon's transaction pooling mode, which does not support server-side prepared statements.

**Explanation**: Connection pooling is essential for serverless databases that limit concurrent connections.

**Case 2: AWS RDS Proxy**

An application hosted on AWS uses RDS Proxy to pool connections to an RDS PostgreSQL instance. The `pooled` option ensures compatibility with the proxy's transaction pooling mode.

**Explanation**: Connection pooling reduces the connection overhead and improves scalability.

---

### 7. Database Transactions

#### Definitions

**Core Definition**

A database transaction is a sequence of database operations that are executed as a single atomic unit, ensuring that either all operations succeed or none do.

**Technical Definition**

Laravel provides the `DB::transaction()` method, which accepts a closure containing the database operations. The closure is executed within a transaction; if an exception is thrown, the transaction is rolled back automatically. The `DB::beginTransaction()`, `DB::commit()`, and `DB::rollBack()` methods provide manual control. The `attempts` parameter in `DB::transaction()` specifies the number of times to retry the transaction on deadlock.

**Beginner-Friendly Explanation**

A transaction is like a bank transfer: either the money leaves one account and arrives in the other, or neither happens. You never end up with money missing. Laravel's `DB::transaction()` method wraps a group of database operations so that they all succeed or all fail together.

#### Purposes

- To ensure data consistency across multiple database operations.
- To automatically roll back changes if an error occurs.
- To handle deadlocks gracefully with automatic retries.
- To support manual transaction control for complex scenarios.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
// Automatic transaction with retry
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);
    DB::table('posts')->delete();
}, attempts: 5);

// Manual transaction
DB::beginTransaction();
try {
    DB::table('users')->update(['votes' => 1]);
    DB::table('posts')->delete();
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `DB::transaction(Closure, $attempts)` | Executes a closure within a transaction, with optional retry. |
| `DB::beginTransaction()` | Starts a manual transaction. |
| `DB::commit()` | Commits the transaction. |
| `DB::rollBack()` | Rolls back the transaction. |
| `attempts` | Number of times to attempt the transaction (including the initial attempt). |

**Syntax Rules**

- `DB::transaction()` automatically rolls back on exception.
- The `attempts` parameter must be an integer (default: 1).
- Manual transactions require explicit `commit()` or `rollBack()`.
- Nested transactions use savepoints.

**Constraints and Limitations**

- Transactions lock rows, which can cause contention under high concurrency.
- Long-running transactions increase the risk of deadlocks.
- The `attempts` parameter retries on deadlock, but not on all exception types.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Basic Transaction**

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);
    DB::table('posts')->delete();
});
```

**Expected Output**

```
// Both the update and delete are committed as a single unit
// If either fails, both are rolled back
```

**Why This Output Occurs**

The closure is wrapped in a transaction. If any statement throws an exception, the entire transaction is rolled back.

---

**Example 2: Transaction with Deadlock Retry**

```php
DB::transaction(function () {
    DB::table('users')->where('id', 1)->update(['votes' => 1]);
    DB::table('users')->where('id', 2)->update(['votes' => 1]);
}, attempts: 5);
```

**Expected Output**

```
// If a deadlock occurs, the transaction is retried up to 5 times (1 initial + 4 retries)
// If all attempts fail, the exception is re-thrown
```

**Why This Output Occurs**

The `attempts` parameter instructs Laravel to retry the transaction if a deadlock exception is caught. The transaction is rolled back and restarted from the beginning on each retry.

---

**Example 3: Manual Transaction**

```php
DB::beginTransaction();

try {
    DB::table('users')->update(['votes' => 1]);
    DB::table('posts')->delete();
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}
```

**Expected Output**

```
// The transaction is committed if both operations succeed
// If an exception is thrown, the transaction is rolled back and the exception is re-thrown
```

**Why This Output Occurs**

Manual transactions provide explicit control over commit and rollback. The `try/catch` block ensures that the transaction is rolled back on any exception.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Order Processing**

An order creation process involves inserting an order, inserting order items, and updating inventory. All three operations must succeed or fail together. `DB::transaction()` ensures that a failure in any step rolls back the entire order.

**Explanation**: Transactions guarantee data consistency in multi-step operations.

**Case 2: High-Concurrency Job Processing**

A queue worker processes jobs that update shared counters. Under high concurrency, deadlocks may occur. The `attempts: 3` parameter automatically retries the transaction, reducing failed jobs.

**Explanation**: Transaction retry improves resilience in concurrent environments.

---

### 8. Deadlock Handling

#### Definitions

**Core Definition**

A deadlock is a situation in which two or more database transactions are each waiting for locks held by the other, resulting in a cycle that prevents any of them from proceeding.

**Technical Definition**

Deadlocks occur when transactions acquire locks in different orders. For example, Transaction A locks row 1 and waits for row 2, while Transaction B locks row 2 and waits for row 1. The database engine (e.g., MySQL InnoDB) detects the deadlock, rolls back one transaction, and returns a deadlock error (`SQLSTATE[40001]`). Laravel's `DB::transaction()` method can automatically retry the rolled-back transaction when the `attempts` parameter is specified.

**Beginner-Friendly Explanation**

A deadlock is like two people trying to walk through a doorway at the same time, each waiting for the other to go first. Neither can move. The database solves this by making one person step back and try again later. Laravel can automatically retry the transaction so the user never sees the error.

#### Purposes

- To detect and resolve deadlock situations automatically.
- To improve application resilience under high concurrency.
- To reduce the number of failed transactions in production.
- To maintain data consistency despite lock contention.

#### Syntax Rules and Structure

**Complete General Syntax**

```php
DB::transaction(function () {
    // Transaction logic
}, attempts: 5);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `attempts` | Number of times to attempt the transaction (1 initial + N-1 retries). |
| `SQLSTATE[40001]` | Deadlock error code. |
| `SQLSTATE[1213]` | MySQL deadlock error code. |

**Syntax Rules**

- The `attempts` parameter must be a positive integer.
- The retry occurs only on deadlock exceptions (Laravel checks the error code).
- Each retry starts a fresh transaction with a new lock acquisition order.

**Constraints and Limitations**

- Retrying does not guarantee success; if the deadlock persists, the transaction will eventually fail.
- Retrying a transaction that has side effects outside the database (e.g., sending emails) can cause duplicate operations.
- Deadlocks are more common with long transactions; keep transactions short.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Transaction with Deadlock Retry**

```php
DB::transaction(function () {
    DB::table('accounts')->where('id', 1)->decrement('balance', 100);
    DB::table('accounts')->where('id', 2)->increment('balance', 100);
}, attempts: 3);
```

**Expected Output**

```
// If a deadlock occurs, the transaction is retried up to 3 times
// If all attempts fail, the deadlock exception is re-thrown
```

**Why This Output Occurs**

The `attempts: 3` parameter instructs Laravel to catch deadlock exceptions and retry the transaction. The transaction is rolled back and restarted with the same closure.

---

**Example 2: Manual Deadlock Handling**

```php
use Illuminate\Database\QueryException;

$attempts = 0;
$maxAttempts = 3;

while ($attempts < $maxAttempts) {
    try {
        DB::beginTransaction();
        DB::table('accounts')->where('id', 1)->decrement('balance', 100);
        DB::table('accounts')->where('id', 2)->increment('balance', 100);
        DB::commit();
        break;
    } catch (QueryException $e) {
        DB::rollBack();
        if ($e->getCode() === '40001' && $attempts < $maxAttempts - 1) {
            $attempts++;
            continue;
        }
        throw $e;
    }
}
```

**Expected Output**

```
// The transaction is retried on deadlock until the maximum attempts are reached
```

**Why This Output Occurs**

The `try/catch` block catches the `QueryException`, checks the error code for a deadlock, and retries the transaction if attempts remain.

---

#### Real-World Cases with Explanation

**Case 1: Banking Transaction System**

A banking application processes transfers between accounts. Under high load, concurrent transfers can cause deadlocks. The application uses `DB::transaction()` with `attempts: 5` to automatically retry failed transactions, ensuring transfers are processed reliably.

**Explanation**: Automatic deadlock retry improves reliability in high-concurrency financial systems.

**Case 2: Inventory Management**

An inventory system updates stock levels for multiple products in a single transaction. Deadlocks occur when concurrent orders lock products in different orders. The application uses transaction retry and orders lock acquisition consistently to reduce deadlocks.

**Explanation**: Consistent lock ordering and retry logic minimise the impact of deadlocks.

---

### 9. Database Best Practices

#### Definitions

**Core Definition**

Database best practices are the recommended guidelines for configuring, optimising, and maintaining Laravel database connections to maximise performance, security, and reliability.

**Technical Definition**

Best practices encompass connection configuration, indexing, query optimisation, transaction management, connection pooling, environment-specific settings, monitoring, and security. They aim to balance performance, cost, and maintainability across development, staging, and production environments.

**Beginner-Friendly Explanation**

Best practices are the tips experienced developers follow to keep their database fast, secure, and reliable. They cover everything from how to configure your connections to how to write efficient queries.

#### Key Best Practices

1. **Use Environment Variables for All Credentials**: Never hardcode database credentials in `config/database.php`. Use `.env` variables and keep the `.env` file out of version control.

2. **Enable Configuration Caching in Production**: Run `php artisan config:cache` during deployment to cache the database configuration, reducing bootstrapping time.

3. **Use Read/Write Splitting for Read-Heavy Applications**: Configure `read` and `write` connections to distribute load across database replicas. Enable `sticky` to ensure read-after-write consistency.

4. **Set Appropriate Connection Timeouts**: Configure `PDO::ATTR_TIMEOUT` to prevent requests from hanging indefinitely when the database is unresponsive.

5. **Use Connection Pooling for Serverless and High-Concurrency Environments**: Enable the `pooled` option for PostgreSQL connections behind PgBouncer, AWS RDS Proxy, or Neon.

6. **Keep Transactions Short and Narrow**: Long transactions hold locks longer, increasing the risk of deadlocks and blocking other operations. Move non-database work outside transactions.

7. **Use Automatic Deadlock Retry**: Specify the `attempts` parameter in `DB::transaction()` to automatically retry on deadlock, improving resilience.

8. **Index Frequently Queried Columns**: Add indexes to columns used in `WHERE`, `JOIN`, and `ORDER BY` clauses. Use composite indexes for multi-column queries.

9. **Avoid the N+1 Problem**: Eager load relationships with `with()` to prevent executing one query per related record.

10. **Use `select()` to Limit Retrieved Columns**: Avoid `SELECT *` by specifying only the columns you need, reducing data transfer and memory usage.

11. **Monitor Slow Queries**: Enable slow query logging on the database server and use Laravel Telescope or Debugbar to identify performance bottlenecks.

12. **Use the Right Driver for the Job**: Choose SQLite for development and testing, MySQL or PostgreSQL for production, and SQL Server only when required by existing infrastructure.

13. **Configure `max_connections` Appropriately**: Ensure the database server's `max_connections` setting can accommodate the application's connection pool plus other clients.

14. **Tune MySQL/PostgreSQL Server Configuration**: Adjust `innodb_buffer_pool_size` (MySQL), `shared_buffers` (PostgreSQL), and other server-level settings based on available RAM and workload.

15. **Test with a Separate Database**: Use a dedicated test database (e.g., SQLite in-memory) to isolate tests from development and production data.

#### Multiple Annotated Complete Step by Step Code Example and Expected Output

**Example 1: Production-Ready Database Configuration**

**`.env`**

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_production
DB_USERNAME=app_user
DB_PASSWORD=strong_password_here
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
```

**`config/database.php`**

```php
'mysql' => [
    'read' => [
        'host' => [
            env('DB_READ_HOST_1', '127.0.0.1'),
            env('DB_READ_HOST_2', '127.0.0.1'),
        ],
    ],
    'write' => [
        'host' => [
            env('DB_WRITE_HOST', '127.0.0.1'),
        ],
    ],
    'sticky' => true,
    'driver' => 'mysql',
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

**Deployment Command**

```bash
php artisan config:cache
```

**Expected Output**

```
// Configuration is cached and ready for production
// Read queries are distributed across two replicas
// Write queries go to the primary database
// Sticky mode ensures read-after-write consistency
```

**Why This Output Occurs**

The configuration follows all best practices: environment variables for credentials, read/write splitting for load distribution, sticky reads for consistency, SSL for security, and configuration caching for performance.

---

#### Real-World Cases with Explanation

**Case 1: High-Traffic SaaS Platform**

A SaaS platform handles millions of requests per day. The database configuration uses read/write splitting with three read replicas, connection pooling via PgBouncer, and automatic deadlock retry. Queries are optimised with indexes, and slow queries are monitored with Telescope.

**Explanation**: Following database best practices ensures the platform scales efficiently and remains reliable under load.

**Case 2: Multi-Tenant Application**

A multi-tenant application uses separate database connections for each tenant, with connection credentials stored in a central database. The configuration uses environment variables for the central connection and dynamically configures tenant connections at runtime.

**Explanation**: Best practices enable flexible, secure multi-tenant architectures.

---

## References

- Database: Getting Started – Laravel 13.x Documentation — https://laravel.com/docs/13.x/database
- Database: Getting Started – Laravel 12.x Documentation — https://laravel.com/docs/12.x/database
- Database: Getting Started – Laravel 11.x Documentation — https://laravel.com/docs/11.x/database
- Database: Getting Started – Laravel 10.x Documentation — https://laravel.com/docs/10.x/database
- Laravel Deadlocks: Causes and Fixes – Laravel News — https://laravel-news.com/laravel-deadlocks-causes-and-fixes
- Optimizing Laravel Eloquent and DB Speed: All You Need to Know – Laravel Daily — https://laraveldaily.com/post/optimizing-laravel-eloquent-db-speed
- The Hidden Cost of MySQL Defaults in Laravel Apps – Laravel News — https://laravel-news.com/mysql-defaults
- Elevating SQL Efficiency – Laravel In Practice EP7 – Laravel News — https://laravel-news.com/elevating-sql-efficiency
- Fixing Queries & Adding Indexes: From 400 Queries to 4 – Laravel News — https://laravel-news.com/fixing-queries-adding-indexes
- Laravel 13.17 Supports PostgreSQL Transaction-Mode Poolers – Laravel News — https://laravel-news.com/postgres-transaction-pooler-support
- PgBouncer Laravel Extension – Packagist — https://packagist.org/packages/vermaysha/pgbouncer-laravel-extension
- Laravel DB Transaction Retry – Packagist — https://packagist.org/packages/ahed92wakim/laravel-db-transaction-retry
- Illuminate\Database Package – Packagist — https://packagist.org/packages/illuminate/database
- Laravel Database Configuration – Laravel Cloud Documentation — https://laravel.com/docs/cloud/databases

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation. Key version-specific features include: the `pooled` option for PostgreSQL transaction-mode poolers (Laravel 13.17+), the `attempts` parameter for `DB::transaction()` (available since early Laravel versions), and the `sticky` option for read/write connections. The `DB_URL` environment variable and connection URL configuration are available in Laravel 11+. SQL Server support requires the `sqlsrv` and `pdo_sqlsrv` PHP extensions. Developers should always consult the documentation for their specific Laravel version.