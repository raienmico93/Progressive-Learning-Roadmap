# Flask Environment Separation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Environment separation is the practice of maintaining distinct configurations for different stages of the software development lifecycle—development, testing, staging, and production—so that each environment has settings appropriate to its purpose without modifying source code.

**Technical Definition:** Flask supports environment separation through its configuration system (`app.config`), which can be populated from multiple sources including Python classes, environment variables, `.env` files, and secrets managers. The application factory pattern (`create_app()`) combined with environment-specific configuration classes (e.g., `DevelopmentConfig`, `TestingConfig`, `StagingConfig`, `ProductionConfig`) enables a single codebase to be deployed across environments with different settings. The active environment is typically selected via an environment variable (e.g., `FLASK_ENV`, `APP_ENV`, or a custom variable), and the corresponding configuration class is loaded using `app.config.from_object()`. The 12-Factor App methodology provides a principled framework for this separation, emphasizing strict configuration via OS environment variables and strict separation of config from code.

**Beginner-Friendly Explanation:** When you build a Flask app, it needs different settings depending on where it's running. On your laptop (development), you want debug mode on and a local database. In testing, you want an in-memory database and no debug. In production, you want debug off, a production database, and secure cookies. Environment separation is how you organize these different settings so you don't accidentally use debug mode in production or a local database in the cloud.

### Key Characteristics

- **Configuration classes:** Environment-specific configuration classes inherit from a base class and override only the settings that differ.
- **Environment variable selection:** The active environment is selected via an OS environment variable, keeping code environment-agnostic.
- **12-Factor alignment:** Configuration is stored in the environment, not in code; the codebase is identical across environments.
- **Precedence and override handling:** OS environment variables take precedence over `.env` files, which take precedence over defaults.
- **Application factory pattern:** The `create_app()` function accepts a configuration name and loads the appropriate settings.
- **Testing isolation:** Testing configurations use in-memory databases, disable CSRF, and set `TESTING=True`.
- **Security hardening:** Production configurations enable `Secure`, `HttpOnly`, and `SameSite` cookie attributes and disable debug mode.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Python classes and inheritance.
- Familiarity with environment variables and the shell.
- Optional: `pip install python-dotenv` for `.env` file support.
- Optional: `pip install pytest` for testing configuration.

### Related Programming Areas

- **Application factory pattern:** The recommended pattern for creating Flask app instances with different configurations.
- **12-Factor App methodology:** A set of best practices for building scalable, maintainable web applications.
- **CI/CD pipelines:** Environment variables are injected by CI/CD systems (GitHub Actions, GitLab CI, Jenkins).
- **Docker/Kubernetes:** Container orchestration platforms use environment variables for configuration.
- **Secrets management:** Production secrets should be stored in dedicated secrets managers (Vault, AWS Secrets Manager, Azure Key Vault).

### Core Concepts / Features

1. Development Environment
2. Testing Environment
3. Staging Environment
4. Production Environment
5. 12-Factor App Methodology Mapping
6. Handling Overlapping Configuration States

---

## 1. Development Environment

### Definitions

**Core Definition:** The development environment is the local setup where developers write, run, and debug the application on their own machines.

**Technical Definition:** The development configuration typically enables `DEBUG=True`, uses a local database (SQLite), disables secure cookie flags (since HTTPS is not used locally), and may enable verbose logging. The configuration is designed for rapid iteration and immediate feedback. The `FLASK_DEBUG` environment variable (or the `--debug` CLI flag) enables the interactive debugger and auto-reloader.

**Beginner-Friendly Explanation:** Your development environment is what you use on your own computer. You want debug mode on so you can see errors, a simple local database, and no HTTPS requirements. It's all about making your life easier while coding.

### Purposes

- To provide a fast, permissive environment for writing and debugging code.
- To enable the interactive debugger and auto-reloader for immediate feedback.
- To use a lightweight local database (SQLite) for rapid iteration.
- To disable production security constraints (HTTPS, secure cookies) that would hinder local development.
- To allow verbose logging and detailed error messages.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# config.py
import os

class DevelopmentConfig:
    DEBUG = True
    TESTING = False
    SECRET_KEY = 'dev-secret-key'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///development.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_SECURE = False
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    EXPLAIN_TEMPLATE_LOADING = True
    TEMPLATES_AUTO_RELOAD = True
```

```python
# app.py
from flask import Flask
from config import DevelopmentConfig

def create_app(config_name='development'):
    app = Flask(__name__)
    app.config.from_object(DevelopmentConfig)
    return app

app = create_app()
```

```bash
# Enable debug mode via CLI
flask --app app run --debug

# Or via environment variable
export FLASK_DEBUG=1
flask --app app run
```

**Component Breakdown:**

| Setting | Development Value | Purpose |
|---------|-------------------|---------|
| `DEBUG` | `True` | Interactive debugger, auto-reload |
| `SQLALCHEMY_DATABASE_URI` | `sqlite:///development.db` | Local lightweight database |
| `SESSION_COOKIE_SECURE` | `False` | No HTTPS required locally |
| `TEMPLATES_AUTO_RELOAD` | `True` | Reload templates on change |
| `EXPLAIN_TEMPLATE_LOADING` | `True` | Log template resolution |

**Syntax Rules:**

- `DEBUG` should be set via the `FLASK_DEBUG` environment variable or `--debug` CLI flag, not hardcoded in code.
- `TEMPLATES_AUTO_RELOAD` defaults to the value of `DEBUG` if not set.
- Local database URIs typically use SQLite for simplicity.
- `SESSION_COOKIE_SECURE` should be `False` in development to allow HTTP.

**Constraints and Limitations:**

- Debug mode should **never** be enabled in production; it exposes sensitive information and allows code execution.
- SQLite is not suitable for production use (limited concurrency, no network access).
- Development configurations should not contain real secrets.

### Annotated Code Examples

**Example 1: Development Configuration with SQLite**

```python
# config.py
import os

class BaseConfig:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-key-change-in-production')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_HTTPONLY = True

class DevelopmentConfig(BaseConfig):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///development.db'
    SESSION_COOKIE_SECURE = False
    TEMPLATES_AUTO_RELOAD = True

# app.py
from flask import Flask
from config import DevelopmentConfig

app = Flask(__name__)
app.config.from_object(DevelopmentConfig)

@app.route('/')
def index():
    return f"Debug mode: {app.config['DEBUG']}"

if __name__ == '__main__':
    app.run(debug=True)
```

**Expected Output:**
- `GET /` → `"Debug mode: True"`
- The interactive debugger is active on errors.
- Templates reload automatically when changed.

**Why this output:** The `DevelopmentConfig` class sets `DEBUG=True` and uses a SQLite database. The `app.config.from_object()` call loads these settings into the application.

### Real-World Cases

- **Local development:** Running the app on `localhost:5000` with debug mode.
- **Rapid prototyping:** Using SQLite for instant database setup without configuration.
- **Debugging:** Using the interactive debugger to inspect errors.

### References

- Flask Debug Mode — https://flask.palletsprojects.com/en/stable/debugging/
- Flask Configuration Handling — https://flask.palletsprojects.com/en/stable/config/
- Compile-N-Run: Flask Environments — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/flask/0-flask-fundamentals/7-flask-environments.mdx

---

## 2. Testing Environment

### Definitions

**Core Definition:** The testing environment is a controlled setup used to run automated tests, ensuring that the application behaves correctly and consistently without external dependencies.

**Technical Definition:** The testing configuration sets `TESTING=True`, which propagates exceptions rather than converting them into 500 responses, making test failures visible. It typically uses an in-memory SQLite database (`sqlite:///:memory:`) for isolation and speed, disables CSRF protection (`WTF_CSRF_ENABLED=False`), and may mock external services. Pytest fixtures create and configure the test application instance.

**Beginner-Friendly Explanation:** The testing environment is where you run automated tests. You want tests to be fast, isolated, and repeatable. So you use an in-memory database that disappears after each test, disable CSRF (since tests don't go through a browser), and make sure exceptions are visible so you can fix bugs.

### Purposes

- To run automated tests in isolation without affecting development or production data.
- To ensure tests are fast and repeatable using an in-memory database.
- To propagate exceptions for immediate visibility of failures.
- To disable CSRF protection that would block test clients.
- To mock or stub external services (email, payment gateways).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# config.py
class TestingConfig:
    TESTING = True
    DEBUG = False
    SECRET_KEY = 'test-secret-key'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    WTF_CSRF_ENABLED = False
    SESSION_COOKIE_SECURE = False
    SERVER_NAME = 'localhost'
```

```python
# conftest.py
import pytest
from app import create_app
from config import TestingConfig

@pytest.fixture
def app():
    app = create_app('testing')
    app.config.from_object(TestingConfig)
    with app.app_context():
        yield app

@pytest.fixture
def client(app):
    return app.test_client()
```

```python
# test_example.py
def test_index(client):
    response = client.get('/')
    assert response.status_code == 200
```

**Component Breakdown:**

| Setting | Testing Value | Purpose |
|---------|---------------|---------|
| `TESTING` | `True` | Propagate exceptions |
| `SQLALCHEMY_DATABASE_URI` | `sqlite:///:memory:` | In-memory database for isolation |
| `WTF_CSRF_ENABLED` | `False` | Disable CSRF for test clients |
| `SERVER_NAME` | `'localhost'` | Required for URL generation in tests |

**Syntax Rules:**

- `TESTING=True` should be set in the testing configuration.
- The in-memory database is created fresh for each test (or test session).
- `WTF_CSRF_ENABLED=False` is required when using Flask-WTF with test clients.
- Use pytest fixtures (`conftest.py`) to set up the test application and client.

**Constraints and Limitations:**

- In-memory databases are not shared across processes; each test process gets its own.
- `TESTING=True` changes error handling; ensure tests expect exceptions.
- CSRF is disabled in tests; do not rely on CSRF validation being tested unless explicitly enabled.

### Annotated Code Examples

**Example 1: Pytest Configuration with In-Memory Database**

```python
# config.py
class TestingConfig:
    TESTING = True
    DEBUG = False
    SECRET_KEY = 'test-secret'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    WTF_CSRF_ENABLED = False
```

```python
# conftest.py
import pytest
from flask import Flask
from config import TestingConfig

@pytest.fixture
def app():
    app = Flask(__name__)
    app.config.from_object(TestingConfig)
    
    @app.route('/')
    def index():
        return 'Hello, Test!'
    
    with app.app_context():
        yield app

@pytest.fixture
def client(app):
    return app.test_client()

def test_index(client):
    response = client.get('/')
    assert response.status_code == 200
    assert response.data == b'Hello, Test!'
```

**Expected Output:**
- The test passes with `TESTING=True` and the in-memory database.

**Why this output:** The `TestingConfig` sets `TESTING=True` and uses an in-memory SQLite database. The `conftest.py` fixture creates the app with this configuration and provides a test client.

### Real-World Cases

- **Unit tests:** Testing individual view functions and models.
- **Integration tests:** Testing the interaction between components.
- **CI/CD pipelines:** Running tests automatically on every commit.

### References

- Flask Testing — https://flask.palletsprojects.com/en/stable/testing/
- Compile-N-Run: Flask Test Setup — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/flask/8-flask-testing/0-flask-test-setup.mdx
- pytest Documentation — https://docs.pytest.org/

---

## 3. Staging Environment

### Definitions

**Core Definition:** The staging environment is a pre-production environment that mirrors the production setup as closely as possible, used for final testing and validation before deploying to production.

**Technical Definition:** The staging configuration is nearly identical to production but may include additional logging, allow limited access, or point to a staging database. It is used for user acceptance testing (UAT), performance testing, and integration testing with real (but isolated) services. The staging environment should use the same infrastructure (database type, web server, caching) as production to catch environment-specific issues.

**Beginner-Friendly Explanation:** Staging is a "practice run" for production. It looks and behaves exactly like production, but it's not the real thing. You use it to test your app with real data and real infrastructure before you flip the switch to production.

### Purposes

- To validate the application in a production-like environment before deployment.
- To catch environment-specific issues (database performance, proxy configuration, SSL).
- To perform user acceptance testing (UAT) with stakeholders.
- To test deployment procedures and rollback strategies.
- To run integration tests with external services (payment gateways, email providers).

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# config.py
import os

class StagingConfig:
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ.get('SECRET_KEY')
    SQLALCHEMY_DATABASE_URI = os.environ.get('STAGING_DATABASE_URL')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PREFERRED_URL_SCHEME = 'https'
    LOG_LEVEL = 'INFO'
    # May use the same caching/queue backends as production
```

**Component Breakdown:**

| Setting | Staging Value | Purpose |
|---------|---------------|---------|
| `DEBUG` | `False` | No debug mode |
| `SQLALCHEMY_DATABASE_URI` | Staging DB URL | Production-like database |
| `SESSION_COOKIE_SECURE` | `True` | HTTPS-only cookies |
| `PREFERRED_URL_SCHEME` | `'https'` | HTTPS URLs |
| `LOG_LEVEL` | `'INFO'` | Less verbose than development |

**Syntax Rules:**

- Staging should mirror production as closely as possible.
- Use the same database engine (PostgreSQL, MySQL) as production.
- Enable HTTPS and secure cookies.
- Point to staging instances of external services.
- Set `SERVER_NAME` to the staging domain.

**Constraints and Limitations:**

- Staging is not production; do not use real user data without consent.
- Staging environments can drift from production over time; automate deployment to keep them in sync.
- Staging may have limited resources compared to production.

### Annotated Code Examples

**Example 1: Staging Configuration with Environment Variables**

```python
# config.py
import os

class StagingConfig:
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ['SECRET_KEY']
    SQLALCHEMY_DATABASE_URI = os.environ['STAGING_DATABASE_URL']
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PREFERRED_URL_SCHEME = 'https'
    SERVER_NAME = 'staging.example.com'
```

```bash
# Set environment variables for staging
export SECRET_KEY="staging-secret-key"
export STAGING_DATABASE_URL="postgresql://user:pass@staging-db:5432/app"
flask --app app run
```

**Expected Output:**
- The application runs with HTTPS, secure cookies, and a PostgreSQL database.
- No debug mode.

**Why this output:** The `StagingConfig` uses environment variables for secrets and database URLs, and enables production-like security settings. The `SERVER_NAME` is set to the staging domain.

### Real-World Cases

- **UAT:** Stakeholders test new features in a production-like environment.
- **Load testing:** Testing performance under realistic conditions.
- **Integration testing:** Verifying third-party service integrations.

### References

- Flask Configuration — https://flask.palletsprojects.com/en/stable/config/
- 12-Factor App: Build, Release, Run — https://12factor.net/build-release-run

---

## 4. Production Environment

### Definitions

**Core Definition:** The production environment is the live system that serves real users, requiring the highest levels of security, performance, and reliability.

**Technical Definition:** The production configuration disables debug mode, uses a production-grade database (PostgreSQL, MySQL), enables all security cookie flags (`Secure`, `HttpOnly`, `SameSite`), configures caching (Redis, Memcached), and sets up logging and monitoring. The `SECRET_KEY` and other secrets are loaded from environment variables or a secrets manager. `PREFERRED_URL_SCHEME` is set to `'https'`.

**Beginner-Friendly Explanation:** Production is the real deal. It's where your actual users interact with your app. You need to lock down security, use a powerful database, enable caching for speed, and make sure secrets are safe. Debug mode is off, and everything is optimized for performance and reliability.

### Purposes

- To serve real users with maximum security and performance.
- To protect sensitive data with encrypted connections and secure cookies.
- To handle high traffic with caching and load balancing.
- To provide observability through logging and monitoring.
- To comply with security and regulatory requirements.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# config.py
import os

class ProductionConfig:
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ['SECRET_KEY']
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PREFERRED_URL_SCHEME = 'https'
    PERMANENT_SESSION_LIFETIME = 3600  # 1 hour
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16 MB
    CACHE_TYPE = 'redis'
    CACHE_REDIS_URL = os.environ['REDIS_URL']
    LOG_LEVEL = 'WARNING'
```

**Component Breakdown:**

| Setting | Production Value | Purpose |
|---------|------------------|---------|
| `DEBUG` | `False` | No debug mode |
| `SECRET_KEY` | From environment | Secure session signing |
| `SQLALCHEMY_DATABASE_URI` | Production DB URL | Scalable database |
| `SESSION_COOKIE_SECURE` | `True` | HTTPS-only cookies |
| `SESSION_COOKIE_HTTPONLY` | `True` | No JavaScript access |
| `SESSION_COOKIE_SAMESITE` | `'Lax'` | CSRF protection |
| `PREFERRED_URL_SCHEME` | `'https'` | HTTPS URLs |
| `CACHE_TYPE` | `'redis'` | Production caching |
| `MAX_CONTENT_LENGTH` | `16MB` | Limit upload size |

**Syntax Rules:**

- `DEBUG` must be `False`.
- Secrets must be loaded from environment variables or a secrets manager.
- Enable all security cookie flags.
- Use a production-grade database and caching backend.
- Set `PREFERRED_URL_SCHEME='https'` and `SERVER_NAME` to the production domain.

**Constraints and Limitations:**

- Never hardcode secrets in production configuration.
- Debug mode must never be enabled in production.
- Production databases require proper backups, monitoring, and scaling.

### Annotated Code Examples

**Example 1: Production Configuration with Redis Caching**

```python
# config.py
import os

class ProductionConfig:
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ['SECRET_KEY']
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PREFERRED_URL_SCHEME = 'https'
    CACHE_TYPE = 'redis'
    CACHE_REDIS_URL = os.environ['REDIS_URL']
    LOG_LEVEL = 'WARNING'
```

```bash
# Set environment variables in production
export SECRET_KEY="production-secret-from-vault"
export DATABASE_URL="postgresql://user:pass@prod-db:5432/app"
export REDIS_URL="redis://prod-redis:6379/0"
```

**Expected Output:**
- The application runs with debug off, HTTPS, secure cookies, Redis caching, and a production PostgreSQL database.

**Why this output:** The `ProductionConfig` loads secrets from environment variables, enables security flags, and configures production-grade database and caching backends.

### Real-World Cases

- **Live web applications:** Serving millions of users.
- **E-commerce platforms:** Handling transactions securely.
- **SaaS applications:** Multi-tenant production deployments.

### References

- Flask Security Considerations — https://flask.palletsprojects.com/en/stable/web-security/
- 12-Factor App: Config — https://12factor.net/config
- Flask Configuration Best Practices — https://flask.palletsprojects.com/en/stable/config/#configuration-best-practices

---

## 5. 12-Factor App Methodology Mapping

### Definitions

**Core Definition:** The 12-Factor App is a methodology for building software-as-a-service applications that are portable, scalable, and maintainable, with Factor III (Config) specifically addressing configuration management.

**Technical Definition:** Factor III of the 12-Factor App states: "Store config in the environment." Configuration includes everything that varies between deployments (database credentials, API keys, feature flags), while code remains identical across environments. The methodology explicitly warns against storing configuration in code or configuration files that are checked into version control. Flask aligns with this by supporting `app.config.from_prefixed_env()`, which loads environment variables with a `FLASK_` prefix and parses them as JSON, enabling automatic type conversion for booleans, integers, and lists.

**Beginner-Friendly Explanation:** The 12-Factor App says: "Don't put your settings in your code. Put them in the environment." This means your code is the same everywhere, and only the environment variables change. Flask makes this easy with `from_prefixed_env()`, which reads all `FLASK_*` variables and turns them into config settings.

### Purposes

- To ensure the codebase is identical across all environments.
- To keep secrets out of version control.
- To enable deployment to any platform that supports environment variables.
- To simplify scaling and horizontal deployment.
- To support the "build once, deploy many" principle.

### Syntax Rules and Structure

**12-Factor Configuration in Flask:**

```python
# app.py
from flask import Flask

def create_app():
    app = Flask(__name__)
    
    # Load all FLASK_* environment variables into app.config
    app.config.from_prefixed_env()
    
    return app
```

```bash
# Set environment variables (12-Factor style)
export FLASK_SECRET_KEY="5f352379324c22463451387a0aec5d2f"
export FLASK_DATABASE_URL="postgresql://user:pass@localhost/db"
export FLASK_DEBUG=false
export FLASK_ALLOWED_HOSTS='["a.com","b.com"]'
```

```python
# Access in code
app.config['SECRET_KEY']       # "5f352379324c22463451387a0aec5d2f"
app.config['DATABASE_URL']     # "postgresql://user:pass@localhost/db"
app.config['DEBUG']            # False (boolean, parsed from JSON)
app.config['ALLOWED_HOSTS']    # ["a.com", "b.com"] (list, parsed from JSON)
```

**Component Breakdown:**

| 12-Factor Principle | Flask Implementation |
|---------------------|----------------------|
| Store config in the environment | `app.config.from_prefixed_env()` |
| Strict separation of config from code | No hardcoded secrets; use env vars |
| Environment variables are granular controls | Each setting can be overridden independently |
| Config does not vary between deploys | Code is identical; only env vars differ |

**Syntax Rules:**

- Use `from_prefixed_env()` to load all `FLASK_*` environment variables.
- Values are parsed as JSON by default (`json.loads`).
- Use `true`/`false` (lowercase) for JSON booleans.
- Use JSON arrays for lists: `FLASK_ALLOWED_HOSTS='["a.com","b.com"]'`.
- Load configuration very early, before extensions initialize.

**Constraints and Limitations:**

- JSON parsing requires valid JSON; Python-style `True`/`False` are invalid.
- Environment variable names on Windows are case-insensitive.
- `.env` files are not part of the 12-Factor methodology; they are a development convenience.

### Annotated Code Examples

**Example 1: 12-Factor Configuration with Prefixed Environment Variables**

```python
# app.py
from flask import Flask

def create_app():
    app = Flask(__name__)
    
    # 12-Factor: Load all config from environment
    app.config.from_prefixed_env()
    
    return app

app = create_app()

@app.route('/')
def index():
    return f"Debug: {app.config.get('DEBUG', False)}"
```

```bash
# Production environment
export FLASK_DEBUG=false
export FLASK_SECRET_KEY="production-secret"
export FLASK_DATABASE_URL="postgresql://prod-db/app"
flask --app app run

# Development environment
export FLASK_DEBUG=true
export FLASK_SECRET_KEY="dev-secret"
export FLASK_DATABASE_URL="sqlite:///dev.db"
flask --app app run
```

**Expected Output:**
- In production: `Debug: False`
- In development: `Debug: True`

**Why this output:** The same code runs in both environments. Only the environment variables differ, and `from_prefixed_env()` loads them into `app.config`. The `DEBUG` value is parsed as a JSON boolean (`false` → `False`).

### Real-World Cases

- **Docker/Kubernetes:** Environment variables are injected into containers.
- **Heroku/Render:** Platform environment variables configure the app.
- **CI/CD:** Pipelines set environment variables for build, test, and deploy stages.

### References

- The Twelve-Factor App: Config — https://12factor.net/config
- Flask Configuring from Environment Variables — https://flask.palletsprojects.com/en/stable/config/#configuring-from-environment-variables
- Flask `Config.from_prefixed_env` — https://flask.palletsprojects.com/en/stable/api/#flask.Config.from_prefixed_env
- 12-Factor App Methodology in Python — https://notes.kodekloud.com/docs/12-Factor-App-Methodology/

---

## 6. Handling Overlapping Configuration States

### Definitions

**Core Definition:** Overlapping configuration states occur when configuration values are defined in multiple sources (`.env` files, OS environment variables, configuration files) and one source must take precedence over another to avoid unintended overrides.

**Technical Definition:** The precedence order in Flask and `python-dotenv` is: **OS environment variables > `.env` file > `.flaskenv` file > defaults**. `load_dotenv()` does not overwrite existing environment variables by default (using `os.environ.setdefault()`), meaning OS environment variables take precedence over `.env` values. This is critical in CI/CD environments where secrets are injected as OS environment variables and a local `.env` file should not override them. The `override=True` parameter can force `.env` values to win, but this is dangerous in CI/CD because it can override real secrets with local development values.

**Beginner-Friendly Explanation:** When you have settings in multiple places—like a `.env` file and real environment variables—you need to know which one wins. By default, real environment variables (from your shell or CI/CD system) win over `.env` file values. This is good because you don't want your local `.env` to accidentally override the production database URL set by your CI/CD pipeline.

### Purposes

- To ensure that CI/CD-injected secrets are not overridden by local `.env` files.
- To define a clear, predictable precedence order for configuration sources.
- To prevent accidental use of development settings in production.
- To allow local development to use `.env` files without interfering with CI/CD.
- To support key rotation and secret management in production.

### Syntax Rules and Structure

**Default Precedence (Recommended):**

```python
# load_dotenv() does NOT override existing OS environment variables
from dotenv import load_dotenv
load_dotenv()  # OS env vars > .env file

# Precedence:
# 1. OS environment variables (set by shell, CI/CD, Docker)
# 2. .env file
# 3. .flaskenv file
# 4. Application defaults
```

**Forcing `.env` to Override (DANGEROUS in CI/CD):**

```python
load_dotenv(override=True)  # .env file > OS env vars
# WARNING: This will override CI/CD secrets with local .env values!
```

**Flask CLI Precedence:**

```
os.environ > -e path > .env > .flaskenv
```

**Component Breakdown:**

| Source | Precedence | Typical Use |
|--------|------------|-------------|
| OS environment variables | Highest | CI/CD secrets, production config |
| `-e` file (Flask CLI) | High | Explicit config file |
| `.env` file | Medium | Local development secrets |
| `.flaskenv` file | Low | Flask CLI settings (can be committed) |

**Syntax Rules:**

- `load_dotenv()` defaults to `override=False`, preserving OS environment variables.
- Use `override=True` only in local development, never in CI/CD.
- `.env` should be in `.gitignore`; `.flaskenv` can be committed.
- In CI/CD, inject secrets as OS environment variables, not through `.env` files.

**Constraints and Limitations:**

- `override=True` in CI/CD can cause production secrets to be replaced by local development values, a critical security risk.
- Environment variable names on Windows are case-insensitive, which can cause conflicts.
- `.env` files are not encrypted; do not store production secrets in them.

### Annotated Code Examples

**Example 1: Safe Precedence with `override=False`**

```python
# app.py
import os
from dotenv import load_dotenv
from flask import Flask

# Load .env but do NOT override existing OS environment variables
load_dotenv(override=False)

app = Flask(__name__)
app.config.from_prefixed_env()

# In CI/CD: DATABASE_URL is set as an OS environment variable
# In local dev: DATABASE_URL is set in .env
# OS environment variable wins in both cases
print(app.config['DATABASE_URL'])
```

```bash
# CI/CD pipeline (e.g., GitHub Actions)
export FLASK_DATABASE_URL="postgresql://ci-db/app"
python app.py  # Uses CI database, not .env

# Local development
# .env contains: FLASK_DATABASE_URL=sqlite:///dev.db
python app.py  # Uses .env (no OS env var set)
```

**Expected Output:**
- In CI/CD: `postgresql://ci-db/app`
- In local development: `sqlite:///dev.db`

**Why this output:** `load_dotenv(override=False)` sets environment variables only if they are not already set. In CI/CD, the OS environment variable is already set, so the `.env` value is ignored. In local development, the OS environment variable is not set, so the `.env` value is used.

**Example 2: Dangerous Override in CI/CD**

```python
# DANGEROUS: This will override CI/CD secrets with local .env values
load_dotenv(override=True)

# If .env contains FLASK_DATABASE_URL=sqlite:///dev.db,
# it will override the CI/CD-injected production database URL!
```

**Expected Output:**
- The local `.env` value overrides the CI/CD secret, potentially causing the application to use the wrong database.

**Why this output:** `override=True` forces `.env` values to win over existing OS environment variables, which is dangerous in CI/CD where real secrets are injected as environment variables.

### Real-World Cases

- **GitHub Actions:** Secrets are injected as environment variables; `.env` files should not override them.
- **Docker:** `docker run -e DATABASE_URL=...` sets OS environment variables; `.env` files are for local development only.
- **Kubernetes:** Secrets are mounted as environment variables; `.env` files are not used.
- **Local development:** `.env` files provide convenient local settings without affecting CI/CD.

### References

- python-dotenv Documentation: Override — https://pypi.org/project/python-dotenv/
- Flask CLI: Environment Variables from dotenv — https://flask.palletsprojects.com/en/stable/cli/#environment-variables-from-dotenv
- GitHub Issue: dotenv file precedence — https://github.com/pallets/flask/issues/5628
- Python Env Variables: os.environ, dotenv & Pydantic — https://env.dev/guides/python-env-variables

---

## References

- Flask Configuration Handling — https://flask.palletsprojects.com/en/stable/config/
- Flask `Config` API — https://flask.palletsprojects.com/en/stable/api/#flask.Config
- Flask `Config.from_prefixed_env` — https://flask.palletsprojects.com/en/stable/api/#flask.Config.from_prefixed_env
- Flask Debug Mode — https://flask.palletsprojects.com/en/stable/debugging/
- Flask Testing — https://flask.palletsprojects.com/en/stable/testing/
- Flask Security Considerations — https://flask.palletsprojects.com/en/stable/web-security/
- The Twelve-Factor App: Config — https://12factor.net/config
- The Twelve-Factor App: Build, Release, Run — https://12factor.net/build-release-run
- python-dotenv Documentation — https://pypi.org/project/python-dotenv/
- environs Documentation — https://pypi.org/project/environs/
- Dynaconf Flask Integration — https://www.dynaconf.com/flask/
- Compile-N-Run: Flask Environments — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/flask/0-flask-fundamentals/7-flask-environments.mdx
- Compile-N-Run: Flask Test Setup — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/flask/8-flask-testing/0-flask-test-setup.mdx
- GitHub Issue: dotenv file precedence — https://github.com/pallets/flask/issues/5628
- Python Env Variables: os.environ, dotenv & Pydantic — https://env.dev/guides/python-env-variables
- 12-Factor App Methodology in Python — https://notes.kodekloud.com/docs/12-Factor-App-Methodology/
- pytest Documentation — https://docs.pytest.org/