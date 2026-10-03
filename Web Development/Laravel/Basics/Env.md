# Environment Configuration & Variables in Laravel

Environment configuration is the mechanism by which Laravel applications adapt their behaviour across development, staging, and production environments without code changes. At the heart of this system is the `.env` file and Laravel's `env()` helper, which together provide a clean separation between environment-specific values and application code.

---

## 1. Local Environment Files (.env)

### 1.1 The `.env` File and `.env.example` Template

A fresh Laravel installation includes a `.env.example` file in the root directory. When Laravel is installed via Composer, this file is automatically copied to `.env`. If installed manually, you must copy it yourself.

The `.env.example` file serves as a **template** for environment variables. It contains placeholder values that document which variables are needed to run the application. Team members can copy this file to `.env` and fill in their own values, ensuring consistency across development environments.

**Example `.env.example`:**
```ini
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

### 1.2 Version Control Exclusion (.gitignore)

The `.env` file **must not be committed to version control**. Laravel's default `.gitignore` includes `.env` for exactly this reason. Committing it would expose sensitive credentials (database passwords, API keys, encryption keys) and cause conflicts when different developers require different configurations.

**What to commit:**
- `.env.example` — with placeholder values and no secrets
- `.env.testing` — if you need a separate testing configuration (also safe to commit if it contains no real secrets)

**What to exclude:**
- `.env`
- `.env.local`
- `.env.*.local`
- `*.key`, `*.pem` (if stored in the project root)

### 1.3 Variable Types, Parsing Rules, and Casting

All variables in `.env` files are **parsed as strings**. Laravel's DotEnv library handles a few reserved values to allow the `env()` helper to return non-string types.

| `.env` Value | `env()` Return Type | Result |
|---|---|---|
| `true` | Boolean | `true` |
| `(true)` | Boolean | `true` |
| `false` | Boolean | `false` |
| `(false)` | Boolean | `false` |
| `empty` | String | `""` (empty string) |
| `(empty)` | String | `""` (empty string) |
| `null` | Null | `null` |
| `(null)` | Null | `null` |

**Quoting rules:** If a value contains spaces, enclose it in double quotes:
```ini
APP_NAME="My Application"
```

**Casting in config files:** Because `.env` values are strings, you should cast them explicitly when they need to be a specific type:
```php
// config/app.php
'debug' => (bool) env('APP_DEBUG', false),
'timeout' => (int) env('APP_TIMEOUT', 30),
```

**Variable expansion:** Laravel supports referencing other variables within `.env` using `${}` syntax:
```ini
MAIL_FROM_NAME="${APP_NAME}"
```

**Override precedence:** Any variable in `.env` can be overridden by server-level or system-level environment variables. This is useful in containerised deployments where environment variables are injected at runtime.

### 1.4 Environment-Specific Files

Laravel supports additional environment files for specific contexts:

- **`.env.testing`**: Overrides `.env` when running PHPUnit tests or executing Artisan commands with the `--env=testing` option.
- **`.env.local`**: Used by some deployment workflows for local overrides.
- **Layered `.env` files**: Community packages (e.g., `buismaarten/laravel-layered-environment`) allow loading multiple `.env` files in a precedence order, such as `.env` → `.env.override` → `.env.production` → `.env.production.override`.

---

## 2. Core Application Environment Variables

### 2.1 Application Metadata

These variables control the application's identity, environment detection, and security.

| Variable | Purpose | Example | Notes |
|---|---|---|---|
| `APP_NAME` | Application name displayed in UI and notifications | `"My Application"` | |
| `APP_ENV` | Runtime environment | `local`, `staging`, `production` | Determines `App::environment()` |
| `APP_DEBUG` | Enable/disable detailed error pages | `true` (local), `false` (production) | **Must be `false` in production** |
| `APP_URL` | Base URL of the application | `https://example.com` | Used for URL generation |
| `APP_KEY` | Encryption key for cookies, sessions, signed URLs | `base64:...` | Generated via `php artisan key:generate` |
| `LOG_CHANNEL` | Default logging channel | `stack`, `single`, `daily` | |
| `LOG_LEVEL` | Minimum log level | `debug`, `info`, `warning`, `error` | |

**Critical security rules:**
- `APP_DEBUG=true` in production exposes sensitive information. Always set `APP_DEBUG=false` before public traffic.
- `APP_KEY` must be set before deploying. Changing it invalidates all existing encrypted data, sessions, and signed URLs.
- The current environment is determined by `APP_ENV`. You can check it via `App::environment('local')` or `App::environment(['local', 'staging'])`.

**Hiding sensitive variables from debug pages:** When `APP_DEBUG=true` and an exception occurs, Laravel's debug page shows all environment variables. You can blacklist sensitive variables in `config/app.php`:
```php
'debug_blacklist' => [
    'ENV' => ['APP_KEY', 'DB_PASSWORD'],
    'SERVER' => ['APP_KEY', 'DB_PASSWORD'],
    'POST' => ['password'],
],
```


### 2.2 Database Infrastructure

| Variable | Purpose | Default | Example |
|---|---|---|---|
| `DB_CONNECTION` | Database driver | `mysql` | `mysql`, `pgsql`, `sqlite`, `sqlsrv` |
| `DB_HOST` | Database host | `127.0.0.1` | `localhost`, `postgres`, `db.example.com` |
| `DB_PORT` | Database port | `3306` (MySQL), `5432` (PostgreSQL) | |
| `DB_DATABASE` | Database name | `laravel` | |
| `DB_USERNAME` | Database user | `root` | |
| `DB_PASSWORD` | Database password | (empty) | |
| `DB_CHARSET` | Character set | `utf8mb4` | |
| `DB_COLLATION` | Collation | `utf8mb4_unicode_ci` | |

**Example `.env`:**
```ini
DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=myapp
DB_USERNAME=odoo
DB_PASSWORD=secret
```


**Connection pooling:** Laravel does not natively support connection pooling for traditional PHP-FPM deployments. However, connection pooling can be achieved through:

- **PgBouncer / AWS RDS Proxy** for PostgreSQL: Packages like `dgarbs51/postgres-pooled-mode` provide drop-in support for transaction poolers with direct connections for schema operations.
- **OpenSwoole-based pooling**: The `bardiz12/laravel-connection-pool` package uses OpenSwoole to achieve connection pooling by overriding Laravel's `DatabaseManager` and `ConnectionFactory`. Configure it by changing the driver to `mysql-pool` and setting `pool_count` in `config/database.php`.
- **Serverless environments**: Point database connections at the connection pooler in session mode (e.g., Appwrite's connection pooling on port 6033) for serverless functions.

### 2.3 Caching & Session Storage

| Variable | Purpose | Options | Default |
|---|---|---|---|
| `CACHE_DRIVER` | Cache backend | `file`, `database`, `redis`, `memcached`, `array` | `file` |
| `SESSION_DRIVER` | Session storage | `file`, `cookie`, `database`, `redis`, `memcached` | `file` |
| `SESSION_LIFETIME` | Session TTL (minutes) | Integer | `120` |
| `SESSION_ENCRYPT` | Encrypt session data | `true`/`false` | `false` |
| `SESSION_DOMAIN` | Session cookie domain | Domain name | `null` |
| `REDIS_HOST` | Redis server host | | `127.0.0.1` |
| `REDIS_PORT` | Redis port | | `6379` |
| `REDIS_PASSWORD` | Redis password | | `null` |
| `REDIS_DB` | Redis database index | | `0` |
| `REDIS_CACHE_DB` | Redis cache database | | `1` |
| `MEMCACHED_HOST` | Memcached host | | `127.0.0.1` |

**Example `.env` for Redis cache and sessions:**
```ini
CACHE_DRIVER=redis
SESSION_DRIVER=redis
SESSION_LIFETIME=120
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

**Redis Cluster configuration:** To enable Redis cluster support, set `REDIS_CLUSTER_ENABLED=true` in `.env` and define cluster configurations in `config/database.php` under `redis.clusters`. If you wish to use a single Redis instance, set `REDIS_CLUSTER_ENABLED=false`.

**Additional Redis options:**
```ini
REDIS_CLIENT=phpredis          # or predis
REDIS_PREFIX=myapp_            # Key prefix to avoid collisions
REDIS_CLUSTER=redis            # Cluster option: redis, predis
REDIS_TIMEOUT=0.1              # Connection timeout
```

**Cache prefixing:** Laravel automatically prefixes cache keys with a value derived from `APP_NAME` to avoid collisions when multiple applications share the same Redis/Memcached instance. You can customise this via `REDIS_PREFIX` or `CACHE_PREFIX`:
```php
// config/cache.php
'prefix' => env('CACHE_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_cache'),
```

### 2.4 Message Queue & Async Workers

| Variable | Purpose | Options | Default |
|---|---|---|---|
| `QUEUE_CONNECTION` | Default queue driver | `sync`, `database`, `redis`, `sqs`, `beanstalkd` | `sync` |
| `QUEUE_FAILED_DRIVER` | Failed job storage | `database-uuids` | |
| `DB_QUEUE_TABLE` | Database queue table | | `jobs` |
| `DB_QUEUE` | Default queue name (database) | | `default` |
| `REDIS_QUEUE` | Queue name (Redis) | | `default` |
| `REDIS_QUEUE_CONNECTION` | Redis connection for queues | | `default` |
| `SQS_PREFIX` | SQS queue URL prefix | | |
| `SQS_QUEUE` | SQS queue name | | `default` |
| `SQS_SUFFIX` | SQS queue name suffix | | |
| `AWS_DEFAULT_REGION` | AWS region for SQS | | `us-east-1` |

**Example `.env` for Redis queue:**
```ini
QUEUE_CONNECTION=redis
REDIS_QUEUE=default
REDIS_QUEUE_CONNECTION=default
```


**Example `.env` for SQS:**
```ini
QUEUE_CONNECTION=sqs
AWS_DEFAULT_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
SQS_PREFIX=https://sqs.us-east-1.amazonaws.com/123456789012
SQS_QUEUE=my-app-queue
```


**RabbitMQ (community package):** Laravel does not ship with a native RabbitMQ driver. The `vladimir-yuldashev/laravel-queue-rabbitmq` package adds support. Configuration involves adding a connection to `config/queue.php` and setting:
```ini
QUEUE_CONNECTION=rabbitmq
RABBITMQ_HOST=127.0.0.1
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=default
```


**Kafka (community package):** Kafka integration typically uses the `mateusjunges/laravel-kafka` package. Queue drivers for Kafka are less standardised than Redis or SQS; most implementations use Kafka as a **producer/consumer** rather than a Laravel queue driver.

**Connection credentials:** For SQS, credentials are typically shared with other AWS services (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`). For RabbitMQ, use the `RABBITMQ_*` variables. For Redis queues, the same `REDIS_*` variables apply.

### 2.5 Mail & Communication

| Variable | Purpose | Options | Default |
|---|---|---|---|
| `MAIL_MAILER` | Mail transport | `smtp`, `sendmail`, `mailgun`, `ses`, `postmark`, `log`, `array` | `smtp` |
| `MAIL_HOST` | SMTP host | | `smtp.mailgun.org` |
| `MAIL_PORT` | SMTP port | | `587` |
| `MAIL_USERNAME` | SMTP username | | |
| `MAIL_PASSWORD` | SMTP password | | |
| `MAIL_ENCRYPTION` | Encryption | `tls`, `ssl` | `null` |
| `MAIL_FROM_ADDRESS` | Default "from" address | | `hello@example.com` |
| `MAIL_FROM_NAME` | Default "from" name | | `${APP_NAME}` |
| `MAILGUN_DOMAIN` | Mailgun domain | | |
| `MAILGUN_SECRET` | Mailgun private API key | | |
| `MAILGUN_ENDPOINT` | Mailgun API endpoint | | `api.mailgun.net` |

**Example `.env` for SMTP (Mailtrap in development):**
```ini
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your-username
MAIL_PASSWORD=your-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=hello@yourapp.com
MAIL_FROM_NAME="${APP_NAME}"
```


**Example `.env` for Mailgun:**
```ini
MAIL_MAILER=mailgun
MAILGUN_DOMAIN=mg.yourdomain.com
MAILGUN_SECRET=key-xxxxxxxxxxxxxxxxxxxxxxxx
MAILGUN_ENDPOINT=api.mailgun.net
MAIL_FROM_ADDRESS=noreply@yourdomain.com
```


**Example `.env` for Amazon SES:**
```ini
MAIL_MAILER=ses
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_DEFAULT_REGION=us-east-1
MAIL_FROM_ADDRESS=noreply@yourdomain.com
```

**Example `.env` for Postmark:**
```ini
MAIL_MAILER=postmark
POSTMARK_TOKEN=your-postmark-token
MAIL_FROM_ADDRESS=noreply@yourdomain.com
```

---

## 3. External Integrations & Third-Party Services

Third-party service credentials are stored in `.env` and referenced in `config/services.php` (for services consumed via Laravel's built-in integrations) or in package-specific config files.

### 3.1 Payment Gateways

**Stripe:**
```ini
STRIPE_KEY=pk_live_your_publishable_key_here
STRIPE_SECRET=sk_live_your_secret_key_here
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret
```
Referenced in `config/services.php`:
```php
'stripe' => [
    'model' => App\Models\User::class,
    'key' => env('STRIPE_KEY'),
    'secret' => env('STRIPE_SECRET'),
    'webhook' => [
        'secret' => env('STRIPE_WEBHOOK_SECRET'),
        'tolerance' => env('STRIPE_WEBHOOK_TOLERANCE', 300),
    ],
],
```


**PayPal:**
```ini
PAYPAL_MODE=live
PAYPAL_CLIENT_ID=your_paypal_client_id
PAYPAL_CLIENT_SECRET=your_paypal_client_secret
```


**Security note:** Never hardcode API keys in source code. Always use environment variables. For enhanced security, consider encrypting sensitive values with packages like `laravel-configrypt`, which auto-decrypts `ENC:` prefixed values at runtime.

### 3.2 Storage Systems

**AWS S3:**
```ini
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-bucket-name
AWS_URL=https://your-bucket.s3.amazonaws.com
AWS_ENDPOINT=                    # For S3-compatible services (MinIO, DigitalOcean Spaces)
AWS_USE_PATH_STYLE_ENDPOINT=false
```
Referenced in `config/filesystems.php`:
```php
's3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'url' => env('AWS_URL'),
    'endpoint' => env('AWS_ENDPOINT'),
    'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
],
```


**Google Cloud Storage:** Laravel uses the `google/cloud-storage` package via the `superbalist/laravel-google-cloud-storage` or `spatie/laravel-google-cloud-storage` adapters:
```ini
GOOGLE_CLOUD_PROJECT_ID=your-project-id
GOOGLE_CLOUD_KEY_FILE=path/to/service-account.json
GOOGLE_CLOUD_STORAGE_BUCKET=your-bucket
GOOGLE_CLOUD_STORAGE_PATH_PREFIX=
```

**Local / Public storage:**
```ini
FILESYSTEM_DISK=local
```

### 3.3 Auth Providers (OAuth)

OAuth credentials are configured per provider in `config/services.php` and referenced via environment variables.

**Google (Socialite):**
```ini
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/callback
```
Config:
```php
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URI', '/auth/callback'),
],
```


**GitHub:**
```ini
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_REDIRECT_URI=https://example.com/auth/github/callback
```

**Facebook:**
```ini
FACEBOOK_CLIENT_ID=your-facebook-app-id
FACEBOOK_CLIENT_SECRET=your-facebook-app-secret
FACEBOOK_REDIRECT_URI=https://example.com/auth/facebook/callback
```

**Auth0:**
```ini
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_CLIENT_ID=your-client-id
AUTH0_CLIENT_SECRET=your-client-secret
AUTH0_REDIRECT_URI=https://example.com/auth/callback
```


**PropelAuth:**
```ini
PROPELAUTH_CLIENT_ID=tbc
PROPELAUTH_CLIENT_SECRET=tbc
PROPELAUTH_CALLBACK_URL=https://localhost:8000/auth/callback
PROPELAUTH_AUTH_URL=https://0000000000.propelauthtest.com
PROPELAUTH_API_KEY=tbc
```


**Important:** OAuth **client IDs** are public and can be exposed; **client secrets** must never be committed or exposed in client-side code. Use separate OAuth applications for development, staging, and production, each with their own credentials.

---

## Key Takeaways

1. **`.env` is for local secrets; `.env.example` is for team templates.** Never commit `.env`; always commit `.env.example` with placeholders.
2. **All `.env` values are strings.** Use Laravel's reserved values (`true`, `false`, `null`, `empty`) and explicit casting in config files.
3. **`APP_KEY` and `APP_DEBUG=false` are non-negotiable for production.** Failure to set them correctly exposes sensitive data and breaks encryption.
4. **Core infrastructure variables** follow a consistent naming convention: `DB_*`, `CACHE_*`, `SESSION_*`, `QUEUE_*`, `MAIL_*`, `REDIS_*`, `AWS_*`.
5. **Third-party credentials belong in `.env`, referenced via `config/services.php`.** Never hardcode API keys.
6. **OAuth client secrets are high-value secrets.** Rotate them periodically and use separate applications per environment.
7. **Connection pooling requires external tooling** (PgBouncer, OpenSwoole packages) or serverless platform features.
8. **Redis Cluster requires `REDIS_CLUSTER_ENABLED=true`** and cluster configurations in `config/database.php`.

---

Would you like me to expand any section — for example, with a complete `.env.example` template, or a deployment checklist for environment variables in CI/CD pipelines?