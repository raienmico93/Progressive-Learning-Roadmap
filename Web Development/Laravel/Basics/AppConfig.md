# Advanced Practices & Production Management in Laravel

## 1. Strict Configuration Separation

### 1.1 Twelve-Factor App Methodologies

The Twelve-Factor App methodology, particularly Factor III (Config), establishes a fundamental principle: **store configuration in the environment, not in code**。 Laravel embraces this through its `.env` file and `config()` helper system, but adherence requires discipline.

**The core Laravel rule for strict separation:**

| Context | Helper to Use | Rationale |
|---------|--------------|-----------|
| **Config files** (`config/*.php`) | `env()` | External values enter the system here |
| **Application code** (controllers, services, models) | `config()` | Internal access; survives config caching |

In regular application code, you should **only** use the `config()` helper (e.g., `config('app.timezone')`), never `env()`. Using `env()` outside config files is a violation of strict separation and will return `null` after `php artisan config:cache` is run, because the `.env` file is no longer loaded.

**Correct pattern:**
```php
// config/app.php — env() is acceptable HERE
'name' => env('APP_NAME', 'Laravel'),

// app/Http/Controllers/SomeController.php — use config() ONLY
$appName = config('app.name'); // ✅
$appName = env('APP_NAME');    // ❌ Returns null after config:cache
```

### 1.2 Immutable Infrastructure Principles

Immutable infrastructure means servers and containers are **never modified after deployment**. Changes require building a new image, not patching a running instance.

**Key principles for Laravel:**

- **Package the application as an immutable Docker image.** Use multi-stage builds: install Composer dependencies (`--no-dev --prefer-dist --optimize-autoloader`) and compile frontend assets in builder stages; copy only production output into the runtime stage.
- **Never place credentials, `.env` files, or private keys in the image.** Inject configuration and secrets at runtime through the deployment platform.
- **Keep config immutable per release.** Non-secret configuration flows through ConfigMaps/environment variables; secret configuration flows through Secrets. Changes require a new release, not runtime mutation.
- **Do not deploy `latest` tags.** Use commit SHA or semantic version tags, and promote the exact tested image digest only after all checks pass.

**Docker runtime rules:**
- Inject configuration and secrets at runtime through the deployment platform, not baked into the image.
- Keep containers stateless; persist durable data in managed storage.
- Run migrations as a controlled release step, never on every web-container start.
- Send logs to stdout/stderr without secrets or personal data.

---

## 2. Secret & Credentials Management

### 2.1 Moving Secrets Out of Plaintext `.env` Files

Production environments should prioritize **system environment variables** (Linux `export`, Docker `ENV`, Kubernetes ConfigMap/Secret) over `.env` files for several reasons:

**Security risks of `.env` files:**
- **Permission management difficulty:** `.env` is a plain file requiring manual `chmod 600`. Misconfiguration (e.g., `chmod 644`) can expose contents through web server misconfigurations.
- **Version control accidents:** Despite `.gitignore`, human error (`git add -f .env`) can leak keys to GitHub.
- **Backup/log pollution:** Automated backup scripts may inadvertently include `.env`; error logs may expose its existence.

**Advantages of system environment variables:**
- **Process isolation:** Environment variables are visible only to the current process and its children, not accessible via web paths.
- **System-level permission control:** Managed by the OS (e.g., systemd's `EnvironmentFile`).
- **Seamless integration with secret managers** (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).

**Core principle:** *"Sensitive information should not exist in plaintext files within the application directory."*

### 2.2 Integration with Secret Vaults

Several Laravel packages bridge the gap between the framework and enterprise secret stores:

**`yamut/laravel-redacted`** provides a single `redacted()` helper that works exactly like `env()`, resolving values from AWS SSM, Secrets Manager, Azure Key Vault, GCP Secret Manager, HashiCorp Vault, Infisical, and Doppler directly into config files.

```php
// config/database.php
'password' => redacted('asm://prod/myapp/db#password', env('DB_PASSWORD')),
```

The package hooks into Laravel's config loading phase. When `php artisan config:cache` runs, `redacted()` resolves the secret and bakes it into `bootstrap/cache/config.php` — zero network calls at runtime, zero API credentials needed on the server.

**`eznix86/laravel-secrets-loader`** resolves environment variables from secret files using the `_FILE` convention (Docker secrets, systemd credentials, Nomad secrets). The resolution order is:

1. Process environment (`DB_PASSWORD=s3cret`)
2. `DB_PASSWORD_FILE` path (e.g., `/run/secrets/db`)
3. `DB_PASSWORD_PATH` path
4. `$CREDENTIALS_DIRECTORY/DB_PASSWORD` (systemd)
5. `$NOMAD_SECRETS_DIR/DB_PASSWORD` (Nomad)
6. `/run/secrets/DB_PASSWORD` (Docker, Swarm, Podman)

A real environment variable always wins, preserving local overrides and `.env` behavior. If a secret file cannot be read, the package throws rather than allowing the application to boot with an empty credential.

**Other supported managers** include HashiCorp Vault (via `vault-cli` + env injection), AWS Secrets Manager (via `aws-sdk-php`), Doppler (`doppler run --` wrapper), 1Password Connect, and Bitwarden Secrets Manager.

**Security note on config caching:** `php artisan config:cache` evaluates `env()` once and writes results into `bootstrap/cache/config.php`. Secrets land in that file **in plaintext**, and a rotated secret is not seen until the cache is rebuilt. Either skip config caching in high-security environments, or rebuild it whenever secrets change.

### 2.3 Short-Lived Tokens and Dynamic Secret Rotation

**Dynamic secret rotation** requires awareness of the config cache lifecycle:

1. **AWS Secrets Manager** supports automatic rotation via Lambda functions. After rotation, you must rebuild the config cache (`php artisan config:cache`) and restart queue workers/scheduler processes.
2. **HashiCorp Vault** supports dynamic secrets with TTLs. For Laravel, this is best implemented at the database driver level (e.g., Vault database secrets engine issuing short-lived database credentials) rather than at the config layer, since config caching bakes values at build time.
3. **Kubernetes Secrets** can be rotated externally, but environment-variable-injected secrets require pod restarts to pick up changes. Use volume-mounted secrets for hot-reload capability where supported.

**Recommendation:** For secrets requiring frequent rotation, avoid `config:cache` or use a secret manager that supports runtime resolution (like `redacted()` with caching disabled for dynamic values).

---

## 3. Environment-Specific Behaviors

### 3.1 Feature Flagging

Feature flags allow toggling functionality based on environment or runtime state **without deploying new code** — a critical capability for safe production management.

**Laravel Pennant** is the first-party feature flag package, part of the Laravel ecosystem since Laravel 10. It provides:

- **Gradual rollouts:** Percentage-based activation using `Lottery::odds(1, 10)` for ~10% of users.
- **Per-user or per-tenant toggling:** Closures receive the scope (User model) and return a boolean.
- **A/B testing:** Persistent storage in a `features` table ensures users stay on the same side of a flag across requests.
- **Environment-based gating:** Static flags can read from config/environment variables.

```php
use Laravel\Pennant\Feature;

// Define in AppServiceProvider
Feature::define('new-checkout', fn (User $user): bool =>
    $user->created_at->isAfter(now()->subDays(30))
);

// Check in application code
if (Feature::active('new-checkout')) {
    return redirect()->route('checkout.v2');
}
```

**Blade directive:**
```blade
@feature('new-checkout')
    <x-checkout-v2 />
@else
    <x-checkout-legacy />
@endfeature
```

**Route middleware:**
```php
Route::middleware(['auth', EnsureFeaturesAreActive::using('new-checkout')])
    ->group(function () {
        Route::get('/checkout', CheckoutV2Controller::class);
    });
```



**Laravel Toggle** (community package) is a simpler alternative focused on global on/off switches. Flags live in `config/toggle.php` and can be backed by environment variables:

```php
'flags' => [
    'comments' => env('TOGGLE_COMMENTS', true),
    'related-articles' => env('TOGGLE_RELATED_ARTICLES', false),
],
```

Two storage drivers are available: `config` (read-only, sourced from environment variables) and `database` (read-write, falls back to config).

### 3.2 Mocking External Service Credentials in Testing/Staging

**Testing environment:**
- Use `.env.testing` to override `.env` when running PHPUnit or Artisan with `--env=testing`.
- Set `MAIL_MAILER=log` or `MAIL_MAILER=array` to prevent real email delivery.
- Use `QUEUE_CONNECTION=sync` for synchronous job execution in tests.
- Mock HTTP clients with Laravel's `Http::fake()` rather than injecting fake credentials.

**Staging environment:**
- Use **separate OAuth applications** (different client IDs/secrets) for each environment.
- Use **test API keys** for payment gateways (e.g., Stripe's `pk_test_`/`sk_test_` keys).
- Configure storage to use a staging-specific bucket or prefix.
- Set `APP_ENV=staging` to enable environment-aware code paths via `App::environment('staging')`.

**Critical rule:** Never share credentials between environments. Compromise in staging should not affect production.

---

## 4. Validation & Safety Nets

### 4.1 Schema-Based Configuration Validation

**`ashallendesign/laravel-config-validator`** allows you to validate config values against Laravel's native validation rules. It supports:

- **Generator command:** `php artisan make:config-validation app` creates a ruleset file in `config-validation/app.php`.
- **Default rulesets:** Publish via `php artisan vendor:publish --tag=config-validator-defaults`.
- **Environment-specific validation:** Rules can be limited to specific app environments.
- **Exception handling:** Throws `InvalidConfigValueException` on failure, or returns a boolean if `throwExceptionOnFailure` is set to `false`.



**`laramicstudio/env-guard`** defines a schema for your `.env` file as a contract. When the app boots, it checks the actual `.env` against that contract and **fails loudly and early** if something is wrong.

```php
// config/env-guard.php
'rules' => [
    'APP_KEY' => 'required|string',
    'APP_ENV' => 'required|in:local,staging,production',
    'DB_PASSWORD' => 'required|string|min:8',
    'STRIPE_SECRET' => 'required|starts_with:sk_',
    'CACHE_TTL' => 'required|integer|min:1',
    'MAIL_PORT' => 'nullable|integer',
],
```

**`teun/laravel-environment-config-validator`** adds a `php artisan env:validate --strict-example` command that fails CI when `.env.example` is missing required keys. Use this in your deployment pipeline to **fail early on invalid environment config**.

### 4.2 Fail-Fast Mechanisms

Fail-fast validation should occur **before the application boots** to prevent running with misconfigured credentials.

**Pattern 1: Service provider validation**
```php
// In a service provider's boot() method
public function boot(): void
{
    if (app()->environment('production')) {
        $required = ['APP_KEY', 'DB_PASSWORD', 'STRIPE_SECRET', 'AWS_SECRET_ACCESS_KEY'];
        foreach ($required as $key) {
            if (empty(config("services.{$key}"))) {
                throw new \RuntimeException("Missing required config: {$key}");
            }
        }
    }
}
```

**Pattern 2: CI/CD pipeline gate**
Add a validation step to your deployment pipeline:
```bash
php artisan config:validate --env=production
php artisan env:validate --strict-example
```

If validation fails, the pipeline aborts before deploying the new release. This catches misconfigured environments **before traffic hits the new release**.

**Pattern 3: Pre-deploy drift detection**
**`fr3on/laravel-drift`** compares your `.env` against `.env.example` and runs safety checks **without booting the Laravel application container** for maximum safety and speed.

---

## 5. CI/CD and Deployment Pipeline Configuration

### 5.1 Injecting Environment Variables During CI/CD Steps

**CI/CD platforms** provide native secret management:

| Platform | Secret Storage | Injection Method |
|----------|---------------|-----------------|
| **GitHub Actions** | Repository/Environment Secrets | `${{ secrets.APP_KEY }}` in workflow |
| **GitLab CI** | CI/CD Variables (masked, protected) | `variables:` block or `$VAR` |
| **Bitbucket Pipelines** | Repository/Deployment Variables | `$VAR` in `bitbucket-pipelines.yml` |
| **Jenkins** | Credentials Plugin | `withCredentials` binding |

**Example GitHub Actions deployment step:**
```yaml
- name: Deploy to production
  env:
    APP_KEY: ${{ secrets.APP_KEY }}
    DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    STRIPE_SECRET: ${{ secrets.STRIPE_SECRET }}
  run: |
    php artisan config:cache
    php artisan migrate --force
```

**Key principle:** Configuration should be injected as part of the deployment pipeline, not manually edited on servers. This ensures all instances receive identical configuration.

### 5.2 Containerized Configuration Strategies

**Docker Compose (development/small deployments):**
```yaml
services:
  app:
    image: registry.example.com/app:${COMMIT_SHA}
    environment:
      APP_ENV: production
      APP_DEBUG: "false"
    env_file:
      - .env.production
    secrets:
      - db_password
    read_only: true
    tmpfs:
      - /tmp

secrets:
  db_password:
    file: ./secrets/db_password.txt
```



**Docker Swarm secrets:**
Laravel packages like `corbosman/laravel-docker-secrets` provide helper methods usable in `env()` calls within config files:
```php
// config/database.php
'password' => env('DB_PASSWORD', docker_secret('db_password')),
```

Note: Due to the way Laravel parses config files, these helpers can only be used in `env()` calls.

**Kubernetes ConfigMaps and Secrets:**

**ConfigMap (non-sensitive configuration):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: laravel-config
data:
  APP_ENV: "production"
  APP_DEBUG: "false"
  APP_URL: "https://example.com"
  CACHE_DRIVER: "redis"
  QUEUE_CONNECTION: "redis"
```

**Secret (sensitive configuration):**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: laravel-secrets
type: Opaque
stringData:
  APP_KEY: "base64:..."
  DB_PASSWORD: "your-db-password"
  STRIPE_SECRET: "sk_live_..."
```

**Deployment referencing both:**
```yaml
spec:
  containers:
    - name: app
      image: registry.example.com/app:COMMIT_SHA
      envFrom:
        - configMapRef:
            name: laravel-config
        - secretRef:
            name: laravel-secrets
```



**Best practices for Kubernetes:**
- Store **non-sensitive** config in ConfigMaps; **sensitive** data in Secrets.
- Use `envFrom` to inject all keys as environment variables, or `valueFrom.secretKeyRef` for individual keys.
- For secrets that require volume mounting (e.g., service account JSON files), use `volumeMounts` with `secret` volumes.
- **Do not** mount `.env` files as ConfigMaps in production; inject environment variables directly.

**Build-time vs. runtime injection:**
- **Build-time:** Use BuildKit secret mounts for build-time credentials (e.g., private Composer repositories). Never bake secrets into image layers.
- **Runtime:** Inject all application configuration and secrets at container start via environment variables or mounted secret files.

**Deployment pipeline summary:**
```
Build → Scan → Publish (immutable image)
  ↓
Deploy to Staging → Smoke Tests → Approval Gate
  ↓
Promote exact image digest to Production
  ↓
Inject ConfigMap + Secret → Boot container → Health check
```

---

## Key Takeaways

1. **Strict separation is non-negotiable:** Use `env()` only in config files; use `config()` everywhere else. This is the single most important rule for production reliability.
2. **Immutable infrastructure** means immutable images + runtime injection of configuration and secrets. Never patch running containers.
3. **Move secrets out of `.env`** and into system environment variables or dedicated secret vaults (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, GCP Secret Manager) for production.
4. **Feature flags** (Laravel Pennant for granular control, Laravel Toggle for simple on/off) enable safe production changes without deployment.
5. **Fail-fast validation** via schema-based config validation (`env-guard`, `config-validator`) catches misconfigurations before the application boots.
6. **CI/CD pipelines** should inject environment variables and secrets at deployment time, never manually edit servers.
7. **Kubernetes ConfigMaps and Secrets** provide the standard mechanism for containerized configuration injection, with clear separation of sensitive and non-sensitive values.
8. **Config caching writes secrets in plaintext** to `bootstrap/cache/config.php` — either skip config caching for high-security environments or rebuild it whenever secrets rotate.

---

Would you like me to expand any section — for example, with a complete CI/CD pipeline example for GitHub Actions or GitLab CI, or a Kubernetes deployment manifest template?