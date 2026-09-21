# Laravel Database Configuration & Architecture: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Database Configuration & Architecture is the framework's system for defining, managing, and optimising database connections across multiple database systems, environments, and usage patterns, providing a unified abstraction layer over PDO that supports everything from a single SQLite file to a distributed MySQL cluster with read/write separation.

**Technical Definition:** Laravel's database layer is built on top of PHP's PDO extension and is orchestrated by the `Illuminate\Database\DatabaseManager` class, which manages connection instances, resolves drivers via the `ConnectionFactory`, and provides the `DB` facade for query execution. Configuration is stored in `config/database.php`, with environment-specific values injected via the `.env` file. The framework supports five first-party database drivers (MariaDB 10.3+, MySQL 5.7+, PostgreSQL 10.0+, SQLite 3.26.0+, and SQL Server 2017+) and provides built-in support for read/write connection splitting, multiple named connections, connection testing via Artisan commands, and persistent connections for high-performance environments.

**Beginner-Friendly Explanation:** Every web application needs to store and retrieve data. Laravel makes this easy by providing a consistent way to talk to different databases—whether you're using MySQL on a production server, SQLite for local development, or PostgreSQL for a data-heavy application. You tell Laravel where your database lives and how to connect to it, and Laravel handles the rest. It even lets you use one database for reading and another for writing, which is useful when your application grows large.

### Key Characteristics

- **Multi-Driver Support:** First-party support for MariaDB, MySQL, PostgreSQL, SQLite, and SQL Server, with MongoDB available via an official package.
- **Environment-Driven Configuration:** Database credentials and connection details are stored in the `.env` file and injected into `config/database.php` via the `env()` helper.
- **Read/Write Splitting:** Built-in support for directing SELECT queries to read replicas and INSERT/UPDATE/DELETE queries to a primary write connection.
- **Multiple Named Connections:** Applications can define and switch between multiple database connections at runtime.
- **Connection Testing:** Artisan commands (`db:show`, `db:table`, `db:monitor`) provide insight into database health, schema, and connection counts.
- **Encrypted Environment Variables:** Laravel provides `env:encrypt` and `env:decrypt` commands for securing sensitive database credentials.
- **Persistent Connections:** The `PDO::ATTR_PERSISTENT` option allows connection reuse, with additional pooling support available in Laravel Octane.

### Prerequisites

- **Laravel 10.x, 11.x, or 12.x** (database features are stable across versions; `db:show` and `db:table` since Laravel 9.24; `env:encrypt` since Laravel 9.32).
- **PHP 8.1+** (Laravel 10) or **PHP 8.2+** (Laravel 11/12) with the appropriate PDO extension for your database.
- A Laravel application with the `config/database.php` configuration file.
- Basic understanding of SQL, database connections, and environment variables.

### Related Programming Areas

- **Eloquent ORM:** Eloquent models use the configured database connections for all persistence operations.
- **Query Builder:** The `DB` facade provides a fluent query builder that operates over the configured connections.
- **Migrations & Seeders:** Database schema changes are applied through the configured default connection.
- **Deployment & DevOps:** Database configuration is a critical part of production deployment and environment management.
- **Performance Optimisation:** Connection pooling, read/write splitting, and persistent connections are key performance levers.

### Core Concepts / Features

1. Environment Setup
2. Drivers & Connections
3. Multiple Connections
4. Connection Testing & Pool

---

## 1. Environment Setup

### Definitions

**Core Definition:** Environment setup for Laravel databases is the process of defining database credentials and connection options in the `.env` file and referencing them in `config/database.php`, with optional encryption for sensitive values.

**Technical Definition:** Laravel uses the `Dotenv` library to load environment variables from the `.env` file into the application's environment. The `config/database.php` file references these variables via the `env()` helper, which returns the environment variable's value or a provided default. The `default` key in `config/database.php` determines which connection is used when no connection is explicitly specified. Laravel 9.32+ provides `php artisan env:encrypt` and `php artisan env:decrypt` commands for encrypting the entire `.env` file or specific values, with the decryption key stored outside the repository.

**Beginner-Friendly Explanation:** Your database credentials—host, username, password, database name—are sensitive. You don't want them in your code repository where anyone can see them. Laravel solves this by storing them in a `.env` file that is excluded from version control. The `config/database.php` file then reads these values and uses them to connect. If you want extra security, Laravel can even encrypt the `.env` file itself.

### Purposes

- To separate sensitive database credentials from application code, preventing accidental exposure in version control.
- To provide a consistent configuration structure across all supported database systems via `config/database.php`.
- To enable environment-specific database configuration (local, staging, production) without code changes.
- To support encrypted environment variables for production deployments where `.env` files may be exposed.
- To allow database URLs (e.g., `mysql://user:pass@host/db`) as an alternative to individual configuration values.
- To provide sensible defaults for local development via Laravel Sail and Laravel Herd.

### Syntax Rules and Structure

**Complete General Syntax — `.env` File:**

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=secret
```

**Component Breakdown:**

| Variable | Description |
|----------|-------------|
| `DB_CONNECTION` | The driver to use (`mysql`, `pgsql`, `sqlite`, `sqlsrv`, `mariadb`). |
| `DB_HOST` | The database host address. |
| `DB_PORT` | The database port. |
| `DB_DATABASE` | The database name (or absolute path for SQLite). |
| `DB_USERNAME` | The database username. |
| `DB_PASSWORD` | The database password. |

**Complete General Syntax — `config/database.php` Connection Definition:**

```php
'mysql' => [
    'driver' => 'mysql',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
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
],
```

**Component Breakdown:**

| Key | Description |
|-----|-------------|
| `'driver'` | The PDO driver name. |
| `'url'` | Optional database URL (overrides individual values). |
| `'host'` | Database host (from `DB_HOST`). |
| `'port'` | Database port (from `DB_PORT`). |
| `'database'` | Database name (from `DB_DATABASE`). |
| `'username'` | Username (from `DB_USERNAME`). |
| `'password'` | Password (from `DB_PASSWORD`). |
| `'charset'` | Character set. |
| `'strict'` | Enables strict mode (MySQL). |

**Complete General Syntax — Database URL:**

```ini
DB_URL="mysql://root:password@127.0.0.1/forge?charset=UTF-8"
```

**Complete General Syntax — Encrypted Environment File:**

```bash
# Encrypt the .env file
php artisan env:encrypt

# Encrypt only specific values
php artisan env:encrypt --only-values --only="DB_PASSWORD,DB_USERNAME"

# Decrypt the .env file
php artisan env:decrypt
```

**Syntax Rules:**

- The `.env` file must be in the application root and should be excluded from version control via `.gitignore`.
- Environment variables are accessed via the `env()` helper; the second argument provides a default value.
- The `config/database.php` file should not be edited to hardcode credentials—always use `env()`.
- Database URLs follow the schema `driver://username:password@host:port/database?options`.
- The `env:encrypt` command creates a `.env.encrypted` file; the decryption key must be stored securely (e.g., in a password manager or CI/CD secret).
- Encrypted values can be selectively applied using `--only-values` and `--only` flags.

**Constraints and Limitations:**

- **`config:cache` interaction:** Running `php artisan config:cache` writes the resolved configuration (including decrypted credentials if `env:decrypt` was used) to `bootstrap/cache/config.php`. Ensure this file is not exposed.
- **Encryption key management:** The `env:encrypt` decryption key must be kept secret. If lost, the encrypted `.env` cannot be decrypted.
- **Environment file precedence:** Laravel loads `.env` first, then `.env.{APP_ENV}` if it exists. Values in the latter override the former.
- **URL override:** If `DB_URL` is set, it overrides individual `DB_*` variables for connection details.

### Multiple Annotated Complete Code Examples

**Example 1: Basic MySQL Environment Setup**

```ini
# .env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:...

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=laravel_user
DB_PASSWORD=secret_password
```

```php
<?php
// config/database.php

return [
    'default' => env('DB_CONNECTION', 'sqlite'),

    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
        ],
    ],
];
```

**Expected Output:** Laravel connects to the MySQL database at `127.0.0.1:3306` with the credentials from `.env`.

**Why:** The `env()` helper reads the values from `.env` and injects them into the configuration array. The `default` key determines that MySQL is the default connection.

---

**Example 2: Encrypting the `.env` File**

```bash
# Encrypt the .env file with a generated key
php artisan env:encrypt

# Output:
# Environment encrypted successfully.
# Key: base64:abc123...
# Encrypted file: .env.encrypted

# Store the key in a secure location, then delete the plaintext .env
rm .env
```

```bash
# On the production server, decrypt the .env file
php artisan env:decrypt --key=base64:abc123...

# Output:
# Environment decrypted successfully.
```

**Expected Output:** The `.env` file is encrypted to `.env.encrypted`, and can be decrypted on another server using the key.

**Why:** The `env:encrypt` command encrypts the entire `.env` file using a randomly generated key. The key must be stored securely and provided during deployment.

---

**Example 3: Selective Encryption of Database Credentials**

```bash
# Encrypt only DB_PASSWORD and DB_USERNAME
php artisan env:encrypt --only-values --only="DB_PASSWORD,DB_USERNAME"
```

```ini
# .env.encrypted (partial)
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=laravel
DB_USERNAME=ENC:eyJpdiI6...
DB_PASSWORD=ENC:eyJpdiI6...
```

**Expected Output:** Only the username and password values are encrypted; other configuration remains readable.

**Why:** The `--only-values` and `--only` flags allow selective encryption, which is useful when only specific credentials need protection while other configuration can remain in plaintext.

### Real-World Cases

- **Local Development:** Developers use SQLite for zero-configuration local development, switching to MySQL for staging and production.
- **Production Deployment:** Credentials are encrypted and stored in CI/CD secrets, decrypted during deployment.
- **Multi-Environment Applications:** Different `.env` files for local, staging, and production with environment-specific database credentials.
- **Managed Database Providers:** AWS RDS, Heroku, and DigitalOcean provide database URLs that are used directly via `DB_URL`.
- **Team Collaboration:** The `.env.example` file is committed to version control with placeholder values, while `.env` is gitignored.

### References

- Laravel Database: Configuration — https://laravel.com/docs/12.x/database#configuration
- Laravel Configuration: Environment Configuration — https://laravel.com/docs/12.x/configuration#environment-configuration
- Laravel: New Environment Encryption Commands (Laravel News) — https://laravel-news.com/laravel-environment-encryption-commands
- Laravel 11.28 Adds Interactive Env Encryption (Laravel News) — https://laravel-news.com/laravel-11-28
- Managing Encrypted Environment Variables with Veil (Laravel News) — https://laravel-news.com/managing-encrypted-environment-variables

---

## 2. Drivers & Connections

### Definitions

**Core Definition:** Laravel database drivers are the PDO-based adapters that enable communication with specific database systems, each configured as a named "connection" in `config/database.php`.

**Technical Definition:** Laravel supports five first-party database drivers: **MariaDB** (10.3+, via the `mariadb` or `mysql` driver), **MySQL** (5.7+, via `mysql`), **PostgreSQL** (10.0+, via `pgsql`), **SQLite** (3.26.0+, via `sqlite`), and **SQL Server** (2017+, via `sqlsrv`). Each driver requires its corresponding PHP PDO extension (`pdo_mysql`, `pdo_pgsql`, `pdo_sqlite`, `pdo_sqlsrv`). MongoDB is supported via the officially maintained `mongodb/laravel-mongodb` package. The `default` key in `config/database.php` determines which connection is used when no explicit connection is specified.

**Beginner-Friendly Explanation:** Laravel speaks to different databases through "drivers." A driver is like a translator—it knows how to format queries and read responses for a specific database system. You choose the driver that matches your database (MySQL, PostgreSQL, SQLite, etc.), and Laravel handles the translation automatically. SQLite is the simplest—it's just a file on your computer, no server needed. MySQL and PostgreSQL are full database servers for production applications.

### Purposes

- To provide a consistent interface for interacting with multiple database systems via a unified API.
- To enable developers to switch database systems without rewriting application code.
- To configure connection-specific options (host, port, credentials, charset, collation) for each database.
- To support SQLite as a zero-configuration database for local development and testing.
- To enable SQL Server and MariaDB for enterprise environments.
- To allow MongoDB integration for document-oriented data storage.

### Syntax Rules and Structure

**Complete General Syntax — MySQL Connection:**

```php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
    'engine' => null,
],
```

**Complete General Syntax — PostgreSQL Connection:**

```php
'pgsql' => [
    'driver' => 'pgsql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
    'search_path' => 'public',
    'sslmode' => 'prefer',
],
```

**Complete General Syntax — SQLite Connection:**

```php
'sqlite' => [
    'driver' => 'sqlite',
    'url' => env('DB_URL'),
    'database' => env('DB_DATABASE', database_path('database.sqlite')),
    'prefix' => '',
    'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
    'busy_timeout' => null,
    'journal_mode' => null,
    'synchronous' => null,
],
```

**Complete General Syntax — SQL Server Connection:**

```php
'sqlsrv' => [
    'driver' => 'sqlsrv',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', 'localhost'),
    'port' => env('DB_PORT', '1433'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
],
```

**Component Breakdown:**

| Driver | PDO Extension | Default Port | Notes |
|--------|--------------|--------------|-------|
| `mysql` | `pdo_mysql` | 3306 | Most common; supports MariaDB. |
| `pgsql` | `pdo_pgsql` | 5432 | Advanced features; `search_path` configurable. |
| `sqlite` | `pdo_sqlite` | N/A | File-based; zero configuration. |
| `sqlsrv` | `pdo_sqlsrv` | 1433 | Requires Microsoft ODBC driver. |
| `mariadb` | `pdo_mysql` | 3306 | MySQL fork; uses same driver. |

**Syntax Rules:**

- The `driver` key determines which PDO adapter is used.
- The `host`, `port`, `database`, `username`, and `password` keys are common to most drivers.
- SQLite uses `database` as the absolute path to the database file.
- The `charset` and `collation` keys control character encoding.
- The `prefix` key applies a table name prefix to all queries.
- The `strict` key (MySQL) enables strict SQL mode.
- PostgreSQL supports `search_path` and `sslmode` options.

**Constraints and Limitations:**

- **PDO extensions:** Each driver requires its corresponding PDO extension to be installed and enabled in PHP. Missing extensions result in "could not find driver" errors.
- **SQL Server dependencies:** The `sqlsrv` driver requires the Microsoft SQL ODBC driver in addition to the PDO extension.
- **SQLite limitations:** SQLite does not support all features of server-based databases (e.g., some ALTER TABLE operations, concurrent writes).
- **Version requirements:** Each database system has a minimum supported version (e.g., MySQL 5.7+, PostgreSQL 10.0+).

### Multiple Annotated Complete Code Examples

**Example 1: SQLite for Local Development**

```ini
# .env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
DB_FOREIGN_KEYS=true
```

```bash
# Create the SQLite database file
touch database/database.sqlite

# Run migrations
php artisan migrate
```

**Expected Output:** Laravel creates the SQLite database file and runs migrations against it.

**Why:** SQLite is the simplest database option—it requires no server and stores all data in a single file. The `DB_DATABASE` value must be an absolute path.

---

**Example 2: PostgreSQL with SSL**

```ini
# .env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=postgres
DB_PASSWORD=secret
```

```php
<?php
// config/database.php

'pgsql' => [
    'driver' => 'pgsql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'postgres'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
    'search_path' => 'public',
    'sslmode' => 'require',
],
```

**Expected Output:** Laravel connects to PostgreSQL with SSL required.

**Why:** The `sslmode` key enforces SSL for the connection, which is recommended for production deployments.

---

**Example 3: SQL Server Connection**

```ini
# .env
DB_CONNECTION=sqlsrv
DB_HOST=localhost
DB_PORT=1433
DB_DATABASE=laravel
DB_USERNAME=sa
DB_PASSWORD=YourStrongPassword
```

```php
<?php
// config/database.php

'sqlsrv' => [
    'driver' => 'sqlsrv',
    'host' => env('DB_HOST', 'localhost'),
    'port' => env('DB_PORT', '1433'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'sa'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
],
```

**Expected Output:** Laravel connects to SQL Server via the `sqlsrv` driver.

**Why:** SQL Server requires the `sqlsrv` and `pdo_sqlsrv` PHP extensions and the Microsoft SQL ODBC driver to be installed.

### Real-World Cases

- **Local Development:** SQLite for zero-configuration development and automated testing.
- **Production Web Applications:** MySQL or PostgreSQL for typical web application workloads.
- **Enterprise Applications:** SQL Server for organisations with existing Microsoft infrastructure.
- **MariaDB Deployments:** MariaDB as a drop-in replacement for MySQL with enhanced features.
- **Document Databases:** MongoDB for applications requiring flexible, schema-less data storage.

### References

- Laravel Database: Introduction — https://laravel.com/docs/12.x/database#introduction
- Laravel Database: Configuration — https://laravel.com/docs/12.x/database#configuration
- Laravel Database: SQLite Configuration — https://laravel.com/docs/12.x/database#sqlite-configuration
- Laravel Database: Microsoft SQL Server Configuration — https://laravel.com/docs/12.x/database#mssql-configuration
- Laravel MongoDB Documentation — https://www.mongodb.com/docs/drivers/php/laravel-mongodb/

---

## 3. Multiple Connections

### Definitions

**Core Definition:** Multiple connections in Laravel allow an application to define and use several database connections simultaneously, including read/write splitting for load distribution, dynamic connection switching at runtime, and cross-database queries.

**Technical Definition:** The `config/database.php` file can define multiple named connections in the `connections` array. The `DB` facade's `connection()` method returns a specific connection instance by name, allowing queries to be directed to different databases. The `read` and `write` keys within a connection configuration enable automatic query routing: SELECT statements use the `read` hosts, while INSERT/UPDATE/DELETE statements use the `write` host. The `sticky` option ensures that after a write operation, subsequent reads within the same request use the write connection, avoiding replication lag issues.

**Beginner-Friendly Explanation:** Sometimes you need more than one database. Maybe you have a main database for user accounts and a separate one for analytics. Or maybe you have one database that handles all the writing and several copies that handle reading, to spread the load. Laravel lets you define multiple connections and switch between them. The read/write splitting feature is particularly useful: when your application grows, you can add read replicas to handle the heavy read traffic while the primary database handles writes.

### Purposes

- To separate read and write operations across different database servers for improved performance and scalability.
- To allow applications to connect to multiple databases (e.g., a main database and a legacy database) simultaneously.
- To enable dynamic connection switching at runtime for multi-tenant applications.
- To support cross-database queries where data from different connections is joined or compared.
- To provide a fallback or secondary connection for high-availability scenarios.
- To isolate different application modules or services into separate databases.

### Syntax Rules and Structure

**Complete General Syntax — Read/Write Connection Configuration:**

```php
'mysql' => [
    'driver' => 'mysql',
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
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
],
```

**Component Breakdown:**

| Key | Description |
|-----|-------------|
| `'read'` | Array of read connection hosts. |
| `'write'` | Array of write connection hosts. |
| `'sticky'` | If `true`, reads after writes in the same request use the write connection. |
| Other keys | Merged from the main connection array. |

**Complete General Syntax — Using Multiple Connections:**

```php
use Illuminate\Support\Facades\DB;

// Use the default connection
$users = DB::table('users')->get();

// Use a specific connection
$legacyUsers = DB::connection('legacy')->table('users')->get();

// Use a specific connection for Eloquent
$legacyUser = LegacyUser::on('legacy')->find(1);
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `DB::connection('name')` | Returns the named connection instance. |
| `Model::on('name')` | Sets the connection for an Eloquent query. |

**Complete General Syntax — Dynamic Connection Switching:**

```php
// Switch the default connection at runtime
config(['database.default' => 'tenant']);

// Or set the connection on the fly
DB::setDefaultConnection('tenant');
```

**Complete General Syntax — Cross-Database Queries:**

```php
// Query across connections (not a true JOIN, but data comparison)
$localUsers = DB::connection('mysql')->table('users')->pluck('email');
$remoteUsers = DB::connection('legacy')->table('users')
    ->whereIn('email', $localUsers)
    ->get();
```

**Syntax Rules:**

- The `read` and `write` keys are optional—if only `read` is provided, both read and write use the same connection.
- The `sticky` option defaults to `true` in Laravel's default configuration.
- Multiple hosts can be provided in the `read` and `write` arrays; Laravel randomly selects one.
- The `DB::connection()` method accepts the connection name as defined in `config/database.php`.
- Eloquent models can specify a connection via the `$connection` property or the `on()` method.

**Constraints and Limitations:**

- **Replication lag:** With read/write splitting, data written to the write connection may not be immediately available on read replicas. The `sticky` option mitigates this within a request but does not eliminate cross-request lag.
- **Cross-database joins:** Laravel does not support true cross-database JOINs between different connection types. Data must be fetched from one connection and used in a query on the other.
- **Transaction scope:** Transactions are scoped to a single connection. Cross-connection transactions are not supported.
- **Connection overhead:** Each additional connection consumes resources. Avoid creating unnecessary connections.

### Multiple Annotated Complete Code Examples

**Example 1: Read/Write Splitting with Multiple Read Replicas**

```php
<?php
// config/database.php

return [
    'default' => 'mysql',

    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'read' => [
                'host' => [
                    '192.168.1.1',  // Read replica 1
                    '192.168.1.2',  // Read replica 2
                ],
            ],
            'write' => [
                'host' => [
                    '192.168.1.3',  // Primary write server
                ],
            ],
            'sticky' => true,
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
        ],
    ],
];
```

```php
<?php
// Application code — reads and writes are automatically routed

// This SELECT goes to one of the read replicas (192.168.1.1 or 192.168.1.2)
$users = DB::table('users')->get();

// This INSERT goes to the write server (192.168.1.3)
DB::table('users')->insert([
    'name' => 'Alice',
    'email' => 'alice@example.com',
]);

// Because sticky is true, this SELECT uses the write connection (192.168.1.3)
$newUser = DB::table('users')->where('email', 'alice@example.com')->first();
```

**Expected Output:** SELECT queries are distributed across the read replicas; INSERT queries go to the write server. After the insert, the subsequent read uses the write connection because `sticky` is enabled.

**Why:** Laravel automatically routes queries based on their type (SELECT vs. INSERT/UPDATE/DELETE). The `sticky` option ensures that data written during the request is immediately readable.

---

**Example 2: Multiple Named Connections**

```php
<?php
// config/database.php

return [
    'default' => 'mysql',

    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
        ],

        'legacy' => [
            'driver' => 'mysql',
            'host' => env('LEGACY_DB_HOST', '192.168.1.100'),
            'database' => env('LEGACY_DB_DATABASE', 'legacy_db'),
            'username' => env('LEGACY_DB_USERNAME', 'legacy_user'),
            'password' => env('LEGACY_DB_PASSWORD', ''),
        ],
    ],
];
```

```php
<?php
// Using the legacy connection

use Illuminate\Support\Facades\DB;

// Query the legacy database
$legacyOrders = DB::connection('legacy')->table('orders')->get();

// Use Eloquent with a specific connection
class LegacyOrder extends Model
{
    protected $connection = 'legacy';
    protected $table = 'orders';
}

$orders = LegacyOrder::all();
```

**Expected Output:** Queries against the `legacy` connection hit the legacy database server, while queries against the default connection hit the main database.

**Why:** Each named connection has its own configuration. The `DB::connection()` method and the `$connection` property on Eloquent models allow precise control over which database is queried.

---

**Example 3: Dynamic Connection Switching for Multi-Tenancy**

```php
<?php
// app/Http/Middleware/TenantDatabase.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

class TenantDatabase
{
    public function handle(Request $request, Closure $next)
    {
        // Resolve the tenant from the subdomain
        $tenant = $request->route('tenant');

        // Configure a dynamic connection for this tenant
        config([
            'database.connections.tenant' => [
                'driver' => 'mysql',
                'host' => '127.0.0.1',
                'database' => 'tenant_' . $tenant,
                'username' => env('DB_USERNAME'),
                'password' => env('DB_PASSWORD'),
            ],
        ]);

        // Set the tenant connection as default
        DB::setDefaultConnection('tenant');

        return $next($request);
    }
}
```

**Expected Output:** Each tenant request is routed to a tenant-specific database. All subsequent queries use the tenant connection.

**Why:** The middleware dynamically configures a new connection based on the tenant identifier and sets it as the default. This allows a single application instance to serve multiple tenants with isolated databases.

### Real-World Cases

- **High-Traffic Applications:** Read/write splitting with multiple read replicas for load distribution.
- **Multi-Tenant SaaS:** Each tenant has a separate database, with dynamic connection switching per request.
- **Legacy System Integration:** A modern Laravel application connects to a legacy database alongside its main database.
- **Analytics Workloads:** A separate read-only connection for analytics queries to avoid impacting the primary database.
- **Geographic Distribution:** Different connections for different geographic regions to reduce latency.

### References

- Laravel Database: Read and Write Connections — https://laravel.com/docs/12.x/database#read-and-write-connections
- Laravel Database: Using Multiple Database Connections — https://laravel.com/docs/12.x/database#using-multiple-database-connections
- Laravel Database: The Sticky Option — https://laravel.com/docs/12.x/database#the-sticky-option
- Laravel Eloquent: Connection Configuration — https://laravel.com/docs/12.x/eloquent#connection-configuration

---

## 4. Connection Testing & Pool

### Definitions

**Core Definition:** Connection testing and pooling in Laravel encompasses the Artisan commands for inspecting database health (`db:show`, `db:table`, `db:monitor`) and the mechanisms for connection reuse and pooling (`PDO::ATTR_PERSISTENT`, Laravel Octane connection pooling).

**Technical Definition:** Laravel provides three Artisan commands for database inspection: `db:show` provides an overview of the database (type, connection details, open connections, table summary), `db:table` displays details about a specific table (columns, types, keys, indexes), and `db:monitor` monitors the number of open connections and dispatches a `DatabaseBusy` event when a threshold is exceeded. For connection pooling, `PDO::ATTR_PERSISTENT` enables persistent connections that are reused across requests in traditional PHP-FPM setups. In Laravel Octane (Swoole/RoadRunner/FrankenPHP), connection pooling is achieved through coroutine-based connection reuse, significantly reducing connection overhead under high concurrency.

**Beginner-Friendly Explanation:** Laravel gives you tools to check if your database is healthy and performing well. `db:show` tells you what database you're connected to and how many connections are open. `db:table` shows you the structure of a specific table. `db:monitor` warns you when too many connections are open. For performance, you can enable persistent connections so that PHP doesn't have to open a new database connection for every request—it reuses the same one. In Laravel Octane, connection pooling goes even further by sharing connections across concurrent requests.

### Purposes

- To verify database connectivity and configuration using `php artisan db:show`.
- To inspect table structure and schema details using `php artisan db:table`.
- To monitor the number of open database connections and receive alerts when thresholds are exceeded via `php artisan db:monitor`.
- To reduce connection overhead by enabling persistent connections via `PDO::ATTR_PERSISTENT`.
- To achieve high-performance connection reuse in Laravel Octane via coroutine-based pooling.
- To diagnose connection-related issues (timeouts, max connections) in production environments.

### Syntax Rules and Structure

**Complete General Syntax — `db:show`:**

```bash
php artisan db:show
php artisan db:show --database=pgsql
php artisan db:show --counts --views
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `--database=name` | Inspect a specific connection. |
| `--counts` | Include table row counts. |
| `--views` | Include database view details. |

**Complete General Syntax — `db:table`:**

```bash
php artisan db:table users
php artisan db:table users --database=pgsql
```

**Component Breakdown:**

| Argument/Option | Description |
|-----------------|-------------|
| `users` | The table name to inspect. |
| `--database=name` | Inspect a table on a specific connection. |

**Complete General Syntax — `db:monitor`:**

```bash
php artisan db:monitor --databases=mysql,pgsql --max=100
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `--databases=mysql,pgsql` | Comma-separated list of connections to monitor. |
| `--max=100` | Maximum open connections before dispatching an event. |

**Complete General Syntax — Persistent Connections:**

```php
// config/database.php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'options' => [
        \PDO::ATTR_PERSISTENT => true,
    ],
],
```

**Complete General Syntax — Octane Connection Pooling:**

```php
// config/database.php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'pool' => [
        'enabled' => true,
        'min_connections' => 4,
        'max_connections' => 32,
    ],
],
```

**Syntax Rules:**

- `db:show` and `db:table` are available from Laravel 9.24+.
- `db:monitor` should be scheduled to run every minute for continuous monitoring.
- The `DatabaseBusy` event must be listened for in a service provider to trigger notifications.
- `PDO::ATTR_PERSISTENT` is set in the `options` array of the connection configuration.
- Octane connection pooling requires Laravel Octane with Swoole, RoadRunner, or FrankenPHP.
- The `pool` configuration is only parsed in Octane environments.

**Constraints and Limitations:**

- **Persistent connections:** `PDO::ATTR_PERSISTENT` can cause issues with transaction residue, temporary tables, and user variables in traditional PHP-FPM. Use with caution and test thoroughly.
- **Octane pooling:** Connection pooling is only available in long-running server environments (Octane). Standard PHP-FPM does not benefit from pooling.
- **`db:monitor` limitations:** The command only dispatches an event—it does not automatically send notifications. You must listen for the `DatabaseBusy` event and implement notification logic.
- **External poolers:** For PostgreSQL, tools like PgBouncer are recommended over persistent connections for production environments.

### Multiple Annotated Complete Code Examples

**Example 1: Inspecting Database Health with `db:show`**

```bash
php artisan db:show --counts --views
```

**Expected Output:**

```
Database: mysql
Host: 127.0.0.1
Port: 3306
Database: laravel
Open Connections: 5
Tables: 12
Views: 2

+------------------+--------+---------+
| Table            | Rows   | Size    |
+------------------+--------+---------+
| users            | 1,245  | 512 KB  |
| posts            | 8,932  | 4.2 MB  |
| comments         | 24,567 | 12.1 MB |
+------------------+--------+---------+
```

**Why:** `db:show` provides a comprehensive overview of the database, including connection details, open connection count, and table statistics. The `--counts` and `--views` options include additional details.

---

**Example 2: Monitoring Connections with `db:monitor`**

```php
<?php
// app/Providers/AppServiceProvider.php

namespace App\Providers;

use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Event::listen(function (DatabaseBusy $event) {
            Notification::route('mail', 'devops@example.com')
                ->notify(new DatabaseApproachingMaxConnections(
                    $event->connectionName,
                    $event->connections
                ));
        });
    }
}
```

```php
<?php
// app/Console/Kernel.php (Laravel 10) or routes/console.php (Laravel 11+)

use Illuminate\Support\Facades\Schedule;

Schedule::command('db:monitor --databases=mysql,pgsql --max=100')
    ->everyMinute();
```

**Expected Output:** Every minute, Laravel checks the open connection count for MySQL and PostgreSQL. If the count exceeds 100, a `DatabaseBusy` event is dispatched, triggering an email notification to the DevOps team.

**Why:** The `db:monitor` command checks the number of open connections and dispatches an event when the threshold is exceeded. The listener sends a notification, alerting the team to potential connection issues.

---

**Example 3: Enabling Persistent Connections (Traditional PHP-FPM)**

```php
<?php
// config/database.php

'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
    'options' => [
        \PDO::ATTR_PERSISTENT => true,
    ],
],
```

**Expected Output:** Database connections are reused across requests instead of being closed and reopened.

**Why:** The `PDO::ATTR_PERSISTENT => true` option tells PDO to use persistent connections. This reduces the overhead of establishing a new connection for each request.

**Warning:** Persistent connections can cause issues with transaction residue, temporary tables, and user variables. Use with caution and test thoroughly.

---

**Example 4: Octane Connection Pooling**

```php
<?php
// config/database.php

'mysql' => [
    'driver' => 'mysql',
    'host' => '127.0.0.1',
    'port' => '3306',
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'pool' => [
        'enabled' => true,
        'min_connections' => 4,
        'max_connections' => 32,
    ],
],
```

```bash
# Start Octane with Swoole
php artisan octane:start --server=swoole
```

**Expected Output:** Octane maintains a pool of 4 to 32 MySQL connections that are reused across concurrent requests, significantly reducing connection overhead.

**Why:** In Octane, the application runs in memory and can maintain persistent connections across requests. The `pool` configuration defines the minimum and maximum number of connections in the pool.

### Real-World Cases

- **Production Monitoring:** `db:show` and `db:monitor` are used to monitor database health and connection counts in production.
- **Performance Tuning:** Persistent connections and Octane pooling reduce connection overhead in high-traffic applications.
- **Troubleshooting:** `db:show` helps diagnose connection issues, such as incorrect credentials or unreachable hosts.
- **Capacity Planning:** `db:monitor` provides data on connection usage, helping teams plan for scaling.
- **High-Concurrency Applications:** Octane connection pooling is essential for applications handling thousands of concurrent requests.

### References

- Laravel Database: Inspecting Your Databases — https://laravel.com/docs/12.x/database#inspecting-your-databases
- Laravel Database: Monitoring Your Databases — https://laravel.com/docs/12.x/database#monitoring-your-databases
- Laravel: New DB Commands (Laravel News) — https://laravel-news.com/laravel-new-db-commands
- Laravel Octane Documentation — https://laravel.com/docs/12.x/octane
- Laravel Octane MySQL Pooling Demo (GitHub) — https://github.com/boiar/laravel-octane-mysql-pooling-connection-demo
- Optimizing Database Connection Handling in Laravel Octane (GitHub Discussion) — https://github.com/laravel/framework/discussions/53475

---

## References

- Laravel Database: Getting Started — https://laravel.com/docs/12.x/database
- Laravel Database: Configuration — https://laravel.com/docs/12.x/database#configuration
- Laravel Database: Read and Write Connections — https://laravel.com/docs/12.x/database#read-and-write-connections
- Laravel Database: Using Multiple Database Connections — https://laravel.com/docs/12.x/database#using-multiple-database-connections
- Laravel Database: Inspecting Your Databases — https://laravel.com/docs/12.x/database#inspecting-your-databases
- Laravel Database: Monitoring Your Databases — https://laravel.com/docs/12.x/database#monitoring-your-databases
- Laravel Configuration: Environment Configuration — https://laravel.com/docs/12.x/configuration#environment-configuration
- Laravel Configuration: Environment Encryption — https://laravel.com/docs/12.x/configuration#encrypting-environment-files
- Laravel: New Environment Encryption Commands (Laravel News) — https://laravel-news.com/laravel-environment-encryption-commands
- Laravel: New DB Commands (Laravel News) — https://laravel-news.com/laravel-new-db-commands
- Laravel Octane Documentation — https://laravel.com/docs/12.x/octane
- Laravel Octane MySQL Pooling Demo (GitHub) — https://github.com/boiar/laravel-octane-mysql-pooling-connection-demo
- Optimizing Database Connection Handling in Laravel Octane (GitHub Discussion) — https://github.com/laravel/framework/discussions/53475
- Laravel MongoDB Documentation — https://www.mongodb.com/docs/drivers/php/laravel-mongodb/
- Managing Encrypted Environment Variables with Veil (Laravel News) — https://laravel-news.com/managing-encrypted-environment-variables
- Laravel 11.28 Adds Interactive Env Encryption (Laravel News) — https://laravel-news.com/laravel-11-28