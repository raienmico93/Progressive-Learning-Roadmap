# Installation, Environment Setup, & First Boot

Getting a Laravel application running involves three decisions: **where** you'll develop (local environment), **how** you'll create the project (installer or Composer), and **how** you'll configure it (environment, key, database). This guide covers all three paths.

---

## 1. Local Development Environment Options

Laravel offers three primary approaches to local development, each with different trade-offs in setup time, performance, and isolation.

### 1.1 Laravel Herd (Native macOS/Windows)

**Laravel Herd** is a **blazing-fast, zero-dependency PHP development environment** for macOS and Windows. It bundles everything needed to start Laravel development — PHP, Nginx, Node, and Composer — into a single native application.

**Key characteristics:**

| Feature | Detail |
|---------|--------|
| **Installation** | One-click installer; no Docker, no VMs, no Homebrew required |
| **Performance** | Native execution — significantly faster than containerized alternatives |
| **PHP versions** | Multiple PHP versions switchable per-site |
| **Services** | Built-in support for MySQL, PostgreSQL, Redis, and Meilisearch |
| **Local domains** | Automatic `.test` domain routing (e.g., `myapp.test`) |
| **SSL** | Automatic HTTPS via self-signed certificates |
| **Cost** | Free tier; Herd Pro adds Xdebug, profiling, and advanced tooling |

**When to choose Herd:** You're on macOS or Windows, want the **fastest possible setup**, and prefer native performance over container isolation. It's the recommended starting point for beginners and the fastest path for experienced developers who don't need Docker parity with production.

**Typical workflow:**
```bash
# After installing Herd, create a new Laravel project
laravel new my-app

# Herd automatically serves it at http://my-app.test
```

Herd also includes **Herd Pro** with features like **Xdebug integration**, **dump debugging**, and **performance profiling**.

### 1.2 Laravel Sail (Docker-Powered)

**Laravel Sail** is a **lightweight command-line interface for interacting with Laravel's default Docker development environment**. It provides a containerized setup that mirrors production more closely than native environments.

**Key characteristics:**

| Feature | Detail |
|---------|--------|
| **Isolation** | Full Docker container isolation — no conflicts with host system |
| **Services** | Pre-configured MySQL, PostgreSQL, Redis, Memcached, Meilisearch, MinIO, Mailpit |
| **Parity** | Closer to production Linux environments |
| **Requirements** | Docker Desktop (or equivalent) installed and running |
| **Commands** | All via `./vendor/bin/sail` wrapper (e.g., `sail up`, `sail artisan`) |

**When to choose Sail:** You want **environment isolation**, you're on Linux, you need **services not easily installed natively** (Redis, Meilisearch, MinIO), or you want your local environment to match a containerized production deployment.

**Typical workflow:**
```bash
# Create a new project with Sail
curl -s "https://laravel.build/my-app" | bash

cd my-app
./vendor/bin/sail up -d

# Access at http://localhost
```

**Sail aliasing (recommended):**
```bash
alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'
```

Then use: `sail artisan migrate`, `sail npm run dev`, etc.

### 1.3 Traditional Setup (Composer + PHP + MySQL)

The **traditional setup** installs PHP, Composer, and a database directly on your host system. This is the most flexible but requires the most manual configuration.

**Requirements:**

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **PHP** | 8.2 (Laravel 11/12) | 8.3+ |
| **Composer** | Latest | Latest |
| **Database** | SQLite, MySQL 8.0+, or PostgreSQL 13+ | MySQL 8.0+ or PostgreSQL 15+ |
| **Node.js** | 18+ | 20+ LTS |
| **Web server** | PHP built-in server (`artisan serve`) | Nginx or Apache |

**Required PHP extensions:**
```
BCMath, Ctype, cURL, DOM, Fileinfo, JSON, Mbstring, OpenSSL, PCRE, PDO, Tokenizer, XML
```

**Typical workflow:**
```bash
# Install Composer globally
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Create a new project
composer create-project laravel/laravel my-app

cd my-app
php artisan serve
```

**When to choose traditional:** You're on Linux, you want **full control** over your stack, you're comfortable managing PHP versions and extensions, or you're deploying to a server where you also develop.

### Comparison Summary

| Criteria | Herd | Sail | Traditional |
|----------|------|------|-------------|
| **Setup time** | Minutes | 10–20 min | 30–60 min |
| **Performance** | Fastest | Good | Fast |
| **Isolation** | None | Full | None |
| **Production parity** | Low | High | Medium |
| **Services included** | MySQL, PostgreSQL, Redis | Full suite | Manual install |
| **Best for** | macOS/Windows beginners | Linux, teams, production parity | Full control, Linux servers |

---

## 2. Creating a New Application

### 2.1 Global Laravel Installer vs. Composer Create-Project

Laravel can be installed through **two primary methods**.

**Method 1: Global Laravel Installer**

The installer is a global Composer package that provides a `laravel` command:

```bash
# Install the installer globally
composer global require laravel/installer

# Ensure ~/.composer/vendor/bin is in your PATH
export PATH="$HOME/.composer/vendor/bin:$PATH"

# Create a new project
laravel new my-app
```

**Installer advantages:**
- Shorter, more memorable command
- Prompts for starter kit and testing framework selection
- Can select from Laravel's official starter kits interactively
- Uses the latest version by default

**Method 2: Composer Create-Project**

```bash
composer create-project laravel/laravel my-app
```

**Create-project advantages:**
- Works without installing a global tool
- Allows specifying exact version constraints: `composer create-project laravel/laravel my-app "^11.0"`
- Standard Composer workflow — works in CI/CD without extra installation
- No global PATH configuration needed

**Recommendation:** Use the **installer** for interactive local development (it prompts for starter kit selection). Use **create-project** in CI/CD pipelines, Dockerfiles, and scripted environments where prompts would block execution.

### 2.2 Initial Scaffolding Options

Laravel offers several **starter kits** that provide pre-built authentication and application scaffolding. As of Laravel 12, the landscape has shifted.

| Starter Kit | Type | Includes | Status |
|-------------|------|----------|--------|
| **Breeze** | Minimal | Login, registration, password reset, email verification, profile management | Maintenance-only |
| **Jetstream** | Full-featured | Everything in Breeze + 2FA, teams, API tokens, browser sessions | Maintenance-only |
| **Laravel 12 Starter Kits** | New | React, Vue, or Livewire with Inertia or Blade; modern auth | Active |

**Laravel 12 Starter Kits** are the current recommendation. They offer three flavors:
- **React** (with Inertia)
- **Vue** (with Inertia)
- **Livewire** (with Blade)

Each includes **authentication, registration, password reset, email verification, profile management, and a modern UI** built with Tailwind CSS.

**Breeze** remains a valid choice for **minimal, simple authentication** — especially if you want full control over the UI and don't need teams or 2FA.

**Jetstream** is appropriate when you need **teams, two-factor authentication, browser session management, and API tokens** out of the box. It is more opinionated and heavier than Breeze.

**Interactive selection with the installer:**
```bash
laravel new my-app
# ? Would you like to install a starter kit? [None, React, Vue, Livewire]
# ? Which testing framework? [Pest, PHPUnit]
# ? Would you like to run npm install? [Yes, No]
```

**Non-interactive scaffolding:**
```bash
laravel new my-app --react --pest
laravel new my-app --vue --phpunit
laravel new my-app --livewire --pest
```

**Adding Breeze to an existing project:**
```bash
composer require laravel/breeze --dev
php artisan breeze:install
php artisan migrate
npm install && npm run dev
```

---

## 3. Post-Installation Configuration

### 3.1 Environment Configuration: Crafting `.env` from `.env.example`

When you create a new Laravel project, the installer automatically copies `.env.example` to `.env`. If you create the project manually or clone from a repository, you must do this yourself:

```bash
cp .env.example .env
```

The `.env` file contains **environment-specific values** that should never be committed to version control. It is listed in `.gitignore` by default.

**What to review immediately after install:**

| Variable | Default | Action |
|----------|---------|--------|
| `APP_NAME` | `Laravel` | Set to your project name |
| `APP_ENV` | `local` | Keep `local` for development |
| `APP_DEBUG` | `true` | Keep `true` locally; must be `false` in production |
| `APP_URL` | `http://localhost` | Set to match your local domain (e.g., `http://my-app.test`) |
| `APP_KEY` | (empty) | Generate with `php artisan key:generate` |
| `DB_CONNECTION` | `sqlite` | Change to `mysql` or `pgsql` if needed |
| `MAIL_MAILER` | `log` | Keep `log` locally, or use Mailpit (Sail) |

**Note on Laravel 11+ defaults:** The default `DB_CONNECTION` is now **SQLite**, a change from earlier versions. This makes the framework immediately usable without a database server, but most production applications switch to MySQL or PostgreSQL.

### 3.2 Application Key Generation

```bash
php artisan key:generate
```

This command sets the `APP_KEY` value in your `.env` file. The key is a **random 32-byte string**, base64-encoded and prefixed with `base64:`.

**Why `APP_KEY` is critical:**

| Purpose | What Breaks Without It |
|---------|----------------------|
| **Cookie encryption** | All cookies become invalid; sessions break |
| **Session encryption** | Encrypted session data cannot be decrypted |
| **Signed URLs** | Temporary signed routes and email verification links fail |
| **Encrypted database columns** | Values encrypted with the old key cannot be decrypted |
| **CSRF tokens** | Token validation fails |

**Critical rule:** If you change `APP_KEY` on a running application, **all existing sessions, cookies, and encrypted data become invalid**. Users will be logged out, and any data encrypted with the old key will be unreadable.

**When cloning an existing project:**
```bash
git clone https://github.com/example/my-app.git
cd my-app
composer install
cp .env.example .env
php artisan key:generate  # Generate a NEW key for your environment
php artisan migrate
```

**In production:** Set `APP_KEY` via your deployment platform's environment variables. Generate it once with `php artisan key:generate --show` (which prints the key without writing it), then inject it as a secret.

```bash
php artisan key:generate --show
# base64:abc123...
```

### 3.3 Database Bootstrapping

#### SQLite (New Framework Default)

Laravel 11+ defaults to SQLite, which requires **no server process** — the database is a single file.

```ini
DB_CONNECTION=sqlite
# DB_DATABASE=/absolute/path/to/database.sqlite  # Optional; defaults to database/database.sqlite
```

**Creating the SQLite file:**
```bash
touch database/database.sqlite
php artisan migrate
```

**When to use SQLite:** Local development, small applications, prototypes, testing. Not recommended for high-concurrency production workloads.

#### MySQL

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_app
DB_USERNAME=root
DB_PASSWORD=secret
```

**Setup steps:**
```bash
# Create the database
mysql -u root -p -e "CREATE DATABASE my_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Run migrations
php artisan migrate
```

**For Herd:** MySQL is available as a built-in service. Start it from the Herd menu, and it listens on `127.0.0.1:3306`.

**For Sail:** MySQL runs in a container automatically. Use `DB_HOST=mysql` (the service name) instead of `127.0.0.1`.

#### PostgreSQL

```ini
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=my_app
DB_USERNAME=postgres
DB_PASSWORD=secret
```

**Setup steps:**
```bash
# Create the database
createdb my_app
# Or: psql -U postgres -c "CREATE DATABASE my_app;"

php artisan migrate
```

**For Herd:** PostgreSQL is available as a built-in service.

**For Sail:** PostgreSQL runs in a container. Use `DB_HOST=pgsql` (the service name).

#### Connection Pooling (Advanced)

For production deployments with high concurrency, consider **PgBouncer** (PostgreSQL) or **ProxySQL** (MySQL). Laravel does not natively pool connections, but external poolers sit between the application and the database. Configure the pooler's host and port in `.env`:

```ini
DB_HOST=pgbouncer.internal
DB_PORT=6432
```

#### Migration Best Practices

```bash
# Run migrations
php artisan migrate

# Rollback the last batch
php artisan migrate:rollback

# Rollback all migrations and re-run
php artisan migrate:fresh

# Rollback all, re-run, and seed
php artisan migrate:fresh --seed

# Check migration status
php artisan migrate:status
```

**In production:** Use `php artisan migrate --force` to bypass the confirmation prompt in automated deployments.

### 3.4 Running the Application

#### `php artisan serve`

The simplest way to start the application locally:

```bash
php artisan serve
# Laravel development server started: http://127.0.0.1:8000
```

**Options:**
```bash
php artisan serve --host=0.0.0.0 --port=8080
```

**Limitation:** The built-in server is **single-threaded** and not suitable for production. It's fine for development but will block on concurrent requests.

#### Herd Local Domains

Herd automatically serves projects from `~/Herd` (or a configured directory) at `http://project-name.test`. No `artisan serve` needed.

**Linking an existing project:**
```bash
# Herd automatically detects projects in the Herd directory
# For projects elsewhere, use the Herd UI to "Link" a directory
```

**Benefits:**
- Real `.test` domains
- Automatic HTTPS
- No port management
- Multiple projects simultaneously

#### Sail

```bash
./vendor/bin/sail up -d
# Access at http://localhost
```

**Sail commands:**
```bash
sail artisan migrate
sail npm run dev
sail composer require laravel/horizon
sail test
sail down
```

#### Manual Web Server (Nginx/Apache)

For traditional setups, configure your web server to point the document root at the `public/` directory:

**Nginx:**
```nginx
server {
    listen 80;
    server_name my-app.test;
    root /path/to/my-app/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

**Critical:** The document root **must** point to `public/`, never the project root. Pointing to the project root exposes `.env`, `composer.json`, and other sensitive files.

---

## Quick Start Checklist

```bash
# 1. Create the project
laravel new my-app
cd my-app

# 2. Verify .env exists and is configured
cat .env

# 3. Generate application key
php artisan key:generate

# 4. Create the database (SQLite default)
touch database/database.sqlite

# 5. Run migrations
php artisan migrate

# 6. Start the development server
php artisan serve
# Or: open http://my-app.test (Herd)
# Or: sail up -d (Sail)

# 7. (If using a starter kit) Install frontend dependencies
npm install && npm run dev
```

---

## Key Takeaways

1. **Choose your environment based on OS and needs:** Herd for macOS/Windows speed, Sail for Linux and production parity, traditional for full control.
2. **The Laravel installer is more interactive; Composer create-project is more scriptable.** Use the installer locally and create-project in CI/CD.
3. **Starter kits have shifted.** Breeze and Jetstream are maintenance-only; Laravel 12 offers React, Vue, and Livewire starter kits as the active recommendation.
4. **`APP_KEY` is non-negotiable.** Generate it immediately after cloning or creating a project. Changing it invalidates all sessions and encrypted data.
5. **SQLite is the new default** in Laravel 11+, requiring zero configuration. Switch to MySQL or PostgreSQL for production.
6. **The document root must point to `public/`.** Never expose the project root to the web.
7. **`php artisan serve` is for development only.** Use Herd's `.test` domains, Sail, or a real web server for anything beyond quick testing.

---

Would you like me to expand any section — for example, with a full Docker Compose configuration for a production-like local stack, or a step-by-step guide to setting up Xdebug with Herd or Sail?