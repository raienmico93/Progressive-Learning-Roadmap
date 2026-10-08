# Flask Logging: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Logging in Flask is the practice of recording events, errors, and diagnostic information generated during application execution, using Python's standard `logging` module, which Flask integrates with through `app.logger`.

**Technical Definition:** Flask uses Python's `logging` module and attaches a `Logger` instance to each `Flask` application, accessible via `app.logger`. The logger is named after the application's `import_name` (e.g., `myapp`). By default, Flask configures the logger to log at `DEBUG` level in debug mode and `WARNING` (or the `LOG_LEVEL` config) otherwise. Log records can be formatted with custom `Formatter` classes (including JSON formatters for structured logging), directed to multiple handlers (stream, rotating file, Syslog, etc.), and enriched with context such as request IDs. Flask also provides `app.logger` integration with `@app.before_request` and `@app.after_request` hooks for request logging, and `@app.errorhandler` for error logging.

**Beginner-Friendly Explanation:** Logging is how your Flask app keeps a diary of what it's doing. It records things like "user logged in," "database connection failed," or "page not found." These records help you understand what happened when something goes wrong. Python has a built-in logging system, and Flask hooks into it so you can write logs from anywhere in your app.

### Key Characteristics

- **Standard library integration:** Flask uses Python's `logging` module, so all standard logging features are available.
- **Per-application logger:** `app.logger` is unique to each Flask application instance.
- **Configurable levels:** Log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL) control the verbosity of logging.
- **Multiple handlers:** Logs can be sent to the console, files, Syslog, or external services.
- **Structured logging:** JSON-formatted logs are easier to parse by log aggregation tools (ELK, Datadog, Splunk).
- **Request context:** Request logs capture method, path, status code, and duration.
- **Correlation IDs:** Injecting a unique request ID into all logs enables tracing a request across services.
- **Error tracking:** Integration with Sentry, Rollbar, or APM platforms for exception monitoring.

### Prerequisites

- Python 3.8+ and Flask installed (`pip install flask`).
- Basic understanding of Flask routing and error handling.
- Familiarity with Python's `logging` module (helpful).
- Optional: `pip install python-json-logger` for JSON logging.
- Optional: `pip install sentry-sdk[flask]` for Sentry integration.

### Related Programming Areas

- **Error handling:** Logging errors complements custom error handlers.
- **Observability:** Logs, metrics, and traces form the three pillars of observability.
- **Debugging:** Logs provide a historical record of application behavior.
- **Security:** Logs capture authentication events, suspicious activity, and audit trails.
- **Performance monitoring:** Request logs capture response times and throughput.

### Core Concepts / Features

1. Python Logging (Standard Library Integration)
2. Log Levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
3. Structured Logging (JSON Formatting for Aggregators)
4. Request Logs (Capturing HTTP Request Metadata)
5. Error Logs (Capturing Exceptions and Tracebacks)
6. Production Logging (Standard Error/Output Piping, Rotating File Handlers, Syslog)
7. Correlation IDs / Request IDs (Injecting Unique Identifiers into Logs)
8. Error Tracking Telemetry Integrations (Sentry, Rollbar, APM)

---

## 1. Python Logging (Standard Library Integration)

### Definitions

**Core Definition:** Python logging is the standard library's built-in system for recording messages from a program, and Flask integrates with it by providing a per-application `Logger` instance accessible via `app.logger`.

**Technical Definition:** Flask's `Flask.logger` property returns a `logging.Logger` instance named after the application's `import_name`. Flask configures the logger with a default handler (`logging.StreamHandler`) and a default format that includes the logger name and level. The logger inherits from the root logger, so messages propagate to the root logger's handlers unless `propagate` is set to `False`. Flask's logger is available during application context, and logging calls are thread-safe.

**Beginner-Friendly Explanation:** Python has a built-in logging system, and Flask gives you a logger for your app. You can write messages with `app.logger.info("...")` or `app.logger.error("...")`, and they'll be recorded. You can configure where the logs go (console, file, etc.) and how they're formatted.

### Purposes

- To record application events for debugging and monitoring.
- To provide a standard, thread-safe logging interface.
- To integrate with Python's logging ecosystem (handlers, formatters, filters).
- To distinguish between different severity levels of messages.
- To capture context about the application's state.

### Syntax Rules and Structure

```python
from flask import Flask, current_app

app = Flask(__name__)

# Log at different levels
app.logger.debug('Detailed debug information')
app.logger.info('General information')
app.logger.warning('A warning message')
app.logger.error('An error occurred')
app.logger.critical('A critical failure')

# Log with exception information
try:
    risky_operation()
except Exception:
    app.logger.exception('Operation failed')

# Access the logger from anywhere in the app context
current_app.logger.info('Message from a blueprint')
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `app.logger.debug()` | Detailed diagnostic information |
| `app.logger.info()` | General informational messages |
| `app.logger.warning()` | Warning messages for potential issues |
| `app.logger.error()` | Error messages for failures |
| `app.logger.critical()` | Critical failures requiring immediate attention |
| `app.logger.exception()` | Logs an error with the current traceback |

**Syntax Rules:**

- `app.logger` is available during an application context.
- The logger name is the application's `import_name` (e.g., `myapp`).
- `app.logger.exception()` should be called inside an `except` block.
- The logger inherits from the root logger; handlers can be added to either.

**Constraints and Limitations:**

- Flask's default logger only logs to the console (stderr).
- The default log level is `WARNING` outside debug mode.
- Loggers should not be configured inside request handlers; configure them at application startup.

### Annotated Code Examples

**Example 1: Basic Logging in a View**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    username = request.form.get('username')
    app.logger.info(f'Login attempt for user: {username}')
    
    if username == 'admin':
        app.logger.warning('Admin login detected')
        return 'Welcome, admin!'
    
    app.logger.debug(f'Regular user login: {username}')
    return f'Welcome, {username}!'
```

**Expected Output:**
- `POST /login` with `username=alice` → logs `INFO: Login attempt for user: alice` and `DEBUG: Regular user login: alice`.
- `POST /login` with `username=admin` → logs `INFO` and `WARNING: Admin login detected`.

**Why this output:** The `app.logger` records messages at different levels. `INFO` messages are shown by default; `DEBUG` messages require the logger level to be set to `DEBUG`.

### Real-World Cases

- **Authentication:** Logging login attempts and failures.
- **Business events:** Logging order placements or user registrations.
- **Debugging:** Adding debug logs to trace code execution.
- **Auditing:** Recording administrative actions.

### References

- Flask Logging — https://flask.palletsprojects.com/en/stable/logging/
- Python `logging` Documentation — https://docs.python.org/3/library/logging.html

---

## 2. Log Levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)

### Definitions

**Core Definition:** Log levels are numeric values that indicate the severity of a log message, controlling which messages are recorded based on the configured threshold.

**Technical Definition:** Python's `logging` module defines five standard levels: `DEBUG` (10), `INFO` (20), `WARNING` (30), `ERROR` (40), and `CRITICAL` (50). The logger's `level` attribute determines the minimum level that will be processed. Messages below the threshold are ignored. Flask sets the default level to `DEBUG` in debug mode and `WARNING` otherwise. The level can be configured via `app.logger.setLevel()` or the `LOG_LEVEL` config key.

**Beginner-Friendly Explanation:** Log levels tell you how serious a message is. `DEBUG` is for detailed technical info, `INFO` is for normal events, `WARNING` is for potential problems, `ERROR` is for actual failures, and `CRITICAL` is for severe failures. You can set the minimum level you want to see—for example, only `WARNING` and above.

### Purposes

- To filter log messages by severity.
- To control the verbosity of logging in different environments.
- To prioritize important messages over routine ones.
- To integrate with monitoring systems that alert on `ERROR` or `CRITICAL` messages.
- To reduce log noise in production.

### Syntax Rules and Structure

```python
import logging

# Set the logger level
app.logger.setLevel(logging.DEBUG)
app.logger.setLevel(logging.INFO)

# Or via config
app.config['LOG_LEVEL'] = 'INFO'
app.logger.setLevel(app.config['LOG_LEVEL'])

# Log at different levels
app.logger.debug('Debug message')
app.logger.info('Info message')
app.logger.warning('Warning message')
app.logger.error('Error message')
app.logger.critical('Critical message')
```

**Component Breakdown:**

| Level | Numeric Value | Description |
|-------|---------------|-------------|
| `DEBUG` | 10 | Detailed diagnostic information |
| `INFO` | 20 | General informational messages |
| `WARNING` | 30 | Potential issues (default level) |
| `ERROR` | 40 | Errors that don't stop the app |
| `CRITICAL` | 50 | Severe errors requiring immediate attention |

**Syntax Rules:**

- Setting the level to `DEBUG` shows all messages.
- Setting the level to `WARNING` shows only warnings and above.
- The default level is `WARNING` unless debug mode is enabled.
- Use `logging.DEBUG` constants, not integers, for readability.

**Constraints and Limitations:**

- `DEBUG` logging in production can generate excessive log volume.
- `CRITICAL` should be reserved for severe failures.
- The level must be set on both the logger and its handlers for messages to appear.

### Annotated Code Examples

**Example 1: Configuring Log Levels per Environment**

```python
import logging
import os
from flask import Flask

app = Flask(__name__)

# Configure log level based on environment
env = os.environ.get('FLASK_ENV', 'production')
if env == 'development':
    app.logger.setLevel(logging.DEBUG)
else:
    app.logger.setLevel(logging.INFO)

@app.route('/test')
def test_logging():
    app.logger.debug('This is a debug message')
    app.logger.info('This is an info message')
    app.logger.warning('This is a warning message')
    app.logger.error('This is an error message')
    return 'Logged messages'
```

**Expected Output:**
- In development: all five messages appear.
- In production: only `INFO`, `WARNING`, and `ERROR` messages appear (`DEBUG` is filtered).

**Why this output:** The logger's level determines the threshold. In development, `DEBUG` allows all messages. In production, `INFO` filters out `DEBUG` messages.

### Real-World Cases

- **Development:** `DEBUG` level for detailed diagnostics.
- **Staging:** `INFO` level for general events.
- **Production:** `WARNING` or `INFO` to reduce noise.
- **Critical systems:** `ERROR` or `CRITICAL` for alerting.

### References

- Python Logging Levels — https://docs.python.org/3/library/logging.html#logging-levels
- Flask Logging — https://flask.palletsprojects.com/en/stable/logging/

---

## 3. Structured Logging (JSON Formatting for Aggregators)

### Definitions

**Core Definition:** Structured logging is the practice of emitting logs in a machine-readable format (typically JSON) instead of plain text, making it easier for log aggregation tools to parse, search, and analyze logs.

**Technical Definition:** Structured logging uses a custom `logging.Formatter` that serializes log records to JSON. The `python-json-logger` library provides a `JsonFormatter` class that can be used to format logs with fields such as timestamp, level, message, module, and custom attributes. Structured logs are ideal for ELK Stack (Elasticsearch, Logstash, Kibana), Datadog, Splunk, and other log management platforms.

**Beginner-Friendly Explanation:** Instead of writing logs as plain text like `2024-01-15 10:30:00 - ERROR - Something went wrong`, structured logging writes them as JSON: `{"timestamp": "2024-01-15T10:30:00", "level": "ERROR", "message": "Something went wrong"}`. This makes it much easier for tools to search and analyze your logs.

### Purposes

- To enable easy parsing and searching by log aggregation tools.
- To include structured context (user ID, request ID, etc.) in every log.
- To standardize log formats across services.
- To enable powerful filtering and alerting based on log fields.
- To integrate with observability platforms.

### Syntax Rules and Structure

**Using `python-json-logger`:**

```python
import logging
from flask import Flask, request, g
from pythonjsonlogger import jsonlogger

app = Flask(__name__)

# Configure JSON formatter
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter(
    '%(asctime)s %(levelname)s %(name)s %(message)s'
)
handler.setFormatter(formatter)

app.logger.addHandler(handler)
app.logger.setLevel(logging.INFO)

@app.route('/test')
def test():
    app.logger.info('Structured log message', extra={
        'user_id': 42,
        'request_id': getattr(g, 'request_id', None),
        'path': request.path
    })
    return 'Logged'
```

**Custom JSON Formatter:**

```python
import json
import logging
import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': datetime.datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
        }
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id
        return json.dumps(log_data)
```

**Component Breakdown:**

| Field | Description |
|-------|-------------|
| `timestamp` | ISO 8601 timestamp |
| `level` | Log level (DEBUG, INFO, etc.) |
| `logger` | Logger name |
| `message` | Log message |
| `module` | Module where the log was created |
| `exception` | Formatted traceback (if applicable) |
| `request_id` | Correlation ID (custom) |

**Syntax Rules:**

- Use `extra={'key': value}` to add custom fields to log records.
- Configure the formatter on the handler, not the logger.
- Use `jsonlogger.JsonFormatter` or a custom `Formatter` subclass.
- Ensure timestamps are in ISO 8601 format for consistency.

**Constraints and Limitations:**

- JSON logs are larger than plain text logs.
- Custom fields must be JSON-serializable.
- Some log aggregation tools have specific field name expectations.

### Annotated Code Examples

**Example 1: JSON Logging with Custom Fields**

```python
import logging
import uuid
from flask import Flask, request, g
from pythonjsonlogger import jsonlogger

app = Flask(__name__)

# Configure JSON formatter
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter(
    '%(asctime)s %(levelname)s %(name)s %(message)s'
)
handler.setFormatter(formatter)
app.logger.addHandler(handler)
app.logger.setLevel(logging.INFO)

@app.before_request
def set_request_id():
    g.request_id = str(uuid.uuid4())

@app.route('/api/data')
def get_data():
    app.logger.info('Fetching data', extra={
        'request_id': g.request_id,
        'path': request.path,
        'method': request.method
    })
    return {'data': 'value'}
```

**Expected Output:**
```json
{"asctime": "2024-01-15 10:30:00,123", "levelname": "INFO", "name": "app", "message": "Fetching data", "request_id": "a1b2c3d4-...", "path": "/api/data", "method": "GET"}
```

**Why this output:** The `JsonFormatter` serializes the log record and any `extra` fields into a JSON object. Log aggregation tools can parse this and index the fields for searching.

### Real-World Cases

- **ELK Stack:** Sending JSON logs to Elasticsearch via Logstash or Filebeat.
- **Datadog/Splunk:** Structured logs enable powerful queries and dashboards.
- **Microservices:** Standardized JSON logs across services.
- **Compliance:** Structured logs with audit fields.

### References

- python-json-logger — https://github.com/madzak/python-json-logger
- Python Logging Cookbook: Structured Logging — https://docs.python.org/3/howto/logging-cookbook.html

---

## 4. Request Logs (Capturing HTTP Request Metadata)

### Definitions

**Core Definition:** Request logs are log entries that capture metadata about each HTTP request, including the method, path, status code, response time, client IP, and user agent.

**Technical Definition:** Request logging is typically implemented using `@app.before_request` and `@app.after_request` hooks. The `before_request` hook records the start time and request metadata, while the `after_request` hook calculates the duration and logs the complete request information. The `request` object provides access to `method`, `path`, `url`, `remote_addr`, `headers`, and other attributes. The response object provides `status_code` and `content_length`.

**Beginner-Friendly Explanation:** Request logs record every time someone visits your app—what URL they visited, what method they used, whether it succeeded, and how long it took. This is useful for monitoring traffic, diagnosing performance issues, and detecting suspicious activity.

### Purposes

- To monitor traffic patterns and usage.
- To diagnose performance issues (slow endpoints).
- To detect suspicious or malicious activity.
- To provide audit trails for compliance.
- To calculate metrics like requests per second and error rates.

### Syntax Rules and Structure

```python
import time
import logging
from flask import Flask, request, g

app = Flask(__name__)
app.logger.setLevel(logging.INFO)

@app.before_request
def start_timer():
    g.start_time = time.time()

@app.after_request
def log_request(response):
    duration = time.time() - g.start_time
    app.logger.info(
        f'{request.method} {request.path} {response.status_code} '
        f'{duration:.3f}s {request.remote_addr}',
        extra={
            'method': request.method,
            'path': request.path,
            'status_code': response.status_code,
            'duration': duration,
            'remote_addr': request.remote_addr,
            'user_agent': request.headers.get('User-Agent')
        }
    )
    return response
```

**Component Breakdown:**

| Field | Description |
|-------|-------------|
| `method` | HTTP method (GET, POST, etc.) |
| `path` | Request path |
| `status_code` | HTTP response status code |
| `duration` | Request processing time in seconds |
| `remote_addr` | Client IP address |
| `user_agent` | Client's User-Agent header |

**Syntax Rules:**

- Use `before_request` to capture the start time.
- Use `after_request` to calculate duration and log.
- Store request-scoped data in `g`.
- Use `extra={}` to include structured fields.

**Constraints and Limitations:**

- Logging every request can generate high log volume.
- Sensitive data (tokens, passwords) must be redacted.
- `after_request` does not run if an unhandled exception occurs (use `teardown_request`).

### Annotated Code Examples

**Example 1: Comprehensive Request Logging**

```python
import time
import logging
from flask import Flask, request, g

app = Flask(__name__)
app.logger.setLevel(logging.INFO)

@app.before_request
def start_timer():
    g.start_time = time.time()

@app.after_request
def log_request(response):
    duration = time.time() - g.start_time
    app.logger.info(
        '%s %s %s %.3fs',
        request.method,
        request.path,
        response.status_code,
        duration,
        extra={
            'method': request.method,
            'path': request.path,
            'status_code': response.status_code,
            'duration_ms': round(duration * 1000, 2),
            'remote_addr': request.remote_addr,
            'user_agent': request.headers.get('User-Agent', '')[:100]
        }
    )
    return response

@app.route('/api/users')
def get_users():
    return {'users': ['Alice', 'Bob']}
```

**Expected Output:**
- `GET /api/users` → log entry: `INFO: GET /api/users 200 0.001s` with extra fields.

**Why this output:** The `before_request` hook records the start time, and the `after_request` hook calculates the duration and logs the request metadata. The `extra` dictionary adds structured fields for JSON logging.

### Real-World Cases

- **API monitoring:** Tracking endpoint usage and response times.
- **Security:** Detecting brute-force attacks or unusual patterns.
- **Performance:** Identifying slow endpoints for optimization.
- **Compliance:** Maintaining access logs for audits.

### References

- Flask Request Logging (Stack Overflow) — https://stackoverflow.com/questions/32331929/how-to-log-every-request-in-flask
- Flask `before_request` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.before_request
- Flask `after_request` — https://flask.palletsprojects.com/en/stable/api/#flask.Flask.after_request

---

## 5. Error Logs (Capturing Exceptions and Tracebacks)

### Definitions

**Core Definition:** Error logs are log entries that capture exceptions, tracebacks, and error conditions that occur during application execution, providing diagnostic information for debugging and monitoring.

**Technical Definition:** Error logging is implemented using `app.logger.error()` or `app.logger.exception()`. The `exception()` method automatically includes the current traceback (`exc_info=True`). Error logs are typically generated in `@app.errorhandler` functions or in `try`/`except` blocks. For unhandled exceptions, a global error handler captures the exception and logs it with full context.

**Beginner-Friendly Explanation:** Error logs record when something goes wrong—like a database failure or a bug in your code. They include the traceback (the list of function calls that led to the error), which helps you figure out what happened.

### Purposes

- To capture exceptions and tracebacks for debugging.
- To monitor error rates and patterns.
- To provide context for error tracking services.
- To alert on critical errors.
- To maintain an audit trail of failures.

### Syntax Rules and Structure

```python
import logging
from flask import Flask, request

app = Flask(__name__)
app.logger.setLevel(logging.ERROR)

@app.errorhandler(Exception)
def handle_exception(error):
    app.logger.exception(
        'Unhandled exception: %s',
        error,
        extra={
            'path': request.path,
            'method': request.method,
            'remote_addr': request.remote_addr
        }
    )
    return 'Internal Server Error', 500

@app.route('/crash')
def crash():
    raise ValueError('Simulated crash')
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `app.logger.error()` | Logs an error without traceback |
| `app.logger.exception()` | Logs an error with traceback |
| `exc_info=True` | Includes traceback in the log |

**Syntax Rules:**

- Use `exception()` inside `except` blocks or error handlers.
- Use `error()` for errors without exceptions (e.g., validation failures).
- Include request context in `extra` for better diagnostics.
- Never log sensitive data (passwords, tokens).

**Constraints and Limitations:**

- `logger.exception()` only works inside an exception handler.
- Excessive error logging can fill up log storage.
- Error logs should be monitored and alerted on.

### Annotated Code Examples

**Example 1: Logging Unhandled Exceptions**

```python
import logging
from flask import Flask, request

app = Flask(__name__)
app.logger.setLevel(logging.ERROR)

@app.errorhandler(Exception)
def handle_exception(error):
    app.logger.exception(
        'Unhandled exception on %s %s',
        request.method,
        request.path,
        extra={
            'path': request.path,
            'method': request.method,
            'remote_addr': request.remote_addr,
            'user_agent': request.headers.get('User-Agent')
        }
    )
    return {'error': 'Internal Server Error'}, 500

@app.route('/crash')
def crash():
    raise ValueError('Simulated crash')
```

**Expected Output:**
- `GET /crash` → `{"error": "Internal Server Error"}` with status `500`.
- The log contains the full traceback, the error message, and the request context.

**Why this output:** `app.logger.exception()` logs the exception with `exc_info=True`, capturing the traceback. The `extra` fields add request context.

### Real-World Cases

- **Production monitoring:** Capturing errors for alerting and analysis.
- **Error tracking:** Sending errors to Sentry or Rollbar.
- **Debugging:** Using tracebacks to identify root causes.
- **Compliance:** Maintaining error logs for audits.

### References

- Python `logging.exception()` — https://docs.python.org/3/library/logging.html#logging.Logger.exception
- Flask Error Handling — https://flask.palletsprojects.com/en/stable/errorhandling/

---

## 6. Production Logging (Standard Error/Output Piping, Rotating File Handlers, Syslog)

### Definitions

**Core Definition:** Production logging refers to the configuration of logging for a live application, using handlers that direct logs to appropriate destinations such as standard output/error, rotating files, or Syslog.

**Technical Definition:** Python's `logging` module provides several handlers for production: `StreamHandler` writes to `sys.stderr` or `sys.stdout` (ideal for containerized environments where logs are collected by the platform), `RotatingFileHandler` writes to files with automatic rotation based on size, `TimedRotatingFileHandler` rotates based on time intervals, and `SysLogHandler` sends logs to a Syslog server. Flask applications in production typically use `StreamHandler` (for Docker/Kubernetes) or `RotatingFileHandler` (for traditional servers).

**Beginner-Friendly Explanation:** In production, you need to send your logs somewhere useful—not just the console. You can write them to files that rotate automatically when they get too big, send them to a central logging server (Syslog), or pipe them to standard output so your container platform can collect them.

### Purposes

- To capture logs in production for monitoring and debugging.
- To prevent log files from growing indefinitely (rotation).
- To centralize logs from multiple servers (Syslog).
- To integrate with container platforms (stdout/stderr).
- To comply with log retention policies.

### Syntax Rules and Structure

**StreamHandler (stdout/stderr):**

```python
import logging
import sys

handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(logging.Formatter(
    '%(asctime)s %(levelname)s %(name)s %(message)s'
))
app.logger.addHandler(handler)
```

**RotatingFileHandler (size-based):**

```python
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    'app.log',
    maxBytes=10 * 1024 * 1024,  # 10 MB
    backupCount=5
)
handler.setFormatter(logging.Formatter(
    '%(asctime)s %(levelname)s %(name)s %(message)s'
))
app.logger.addHandler(handler)
```

**TimedRotatingFileHandler (time-based):**

```python
from logging.handlers import TimedRotatingFileHandler

handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',
    interval=1,
    backupCount=30
)
app.logger.addHandler(handler)
```

**SysLogHandler:**

```python
from logging.handlers import SysLogHandler

handler = SysLogHandler(address=('logs.example.com', 514))
app.logger.addHandler(handler)
```

**Component Breakdown:**

| Handler | Destination | Use Case |
|---------|-------------|----------|
| `StreamHandler` | stdout/stderr | Docker, Kubernetes |
| `RotatingFileHandler` | File (size-based) | Traditional servers |
| `TimedRotatingFileHandler` | File (time-based) | Traditional servers |
| `SysLogHandler` | Syslog server | Centralized logging |

**Syntax Rules:**

- Add handlers to `app.logger` in the application factory.
- Set the formatter on each handler.
- Use `StreamHandler(sys.stdout)` for container platforms.
- Configure rotation parameters to prevent disk exhaustion.

**Constraints and Limitations:**

- File handlers require write permissions.
- Syslog may not be available in all environments.
- Multiple handlers can lead to duplicate logs if not configured carefully.

### Annotated Code Examples

**Example 1: Production Logging with Rotation**

```python
import logging
from logging.handlers import RotatingFileHandler
from flask import Flask

app = Flask(__name__)

if not app.debug:
    # Configure production logging
    handler = RotatingFileHandler(
        'logs/app.log',
        maxBytes=10 * 1024 * 1024,  # 10 MB
        backupCount=10
    )
    handler.setLevel(logging.INFO)
    formatter = logging.Formatter(
        '%(asctime)s %(levelname)s %(name)s %(message)s'
    )
    handler.setFormatter(formatter)
    app.logger.addHandler(handler)

@app.route('/')
def index():
    app.logger.info('Index page accessed')
    return 'Hello, World!'
```

**Expected Output:**
- `GET /` → logs `INFO: Index page accessed` to `logs/app.log`.
- When the file reaches 10 MB, it's rotated, and a new file is created.

**Why this output:** The `RotatingFileHandler` writes logs to a file and rotates it when the size limit is reached, keeping up to 10 backup files.

### Real-World Cases

- **Docker/Kubernetes:** Using `StreamHandler` to send logs to stdout.
- **Traditional servers:** Using `RotatingFileHandler` to manage log files.
- **Centralized logging:** Using `SysLogHandler` to send logs to a central server.
- **Cloud platforms:** Using platform-specific log collectors.

### References

- Python `logging.handlers` — https://docs.python.org/3/library/logging.handlers.html
- Flask Logging — https://flask.palletsprojects.com/en/stable/logging/

---

## 7. Correlation IDs / Request IDs (Injecting Unique Identifiers into Logs)

### Definitions

**Core Definition:** A correlation ID (or request ID) is a unique identifier assigned to each incoming request and included in all log messages generated during that request, enabling tracing of a single request's execution path across multiple log entries and services.

**Technical Definition:** A correlation ID is typically a UUID generated in a `@app.before_request` hook and stored in `g` (or a `contextvars.ContextVar`). A custom `logging.Filter` or `logging.LoggerAdapter` injects the ID into every log record. The ID can also be propagated to downstream services via HTTP headers (e.g., `X-Request-ID`). For distributed tracing, correlation IDs are combined with span IDs in systems like OpenTelemetry.

**Beginner-Friendly Explanation:** A correlation ID is like a tracking number for a request. When a user visits your app, you give their request a unique ID. Every log message related to that request includes the ID. If something goes wrong, you can search your logs for that ID and see the entire story of what happened.

### Purposes

- To trace a single request's execution across multiple log entries.
- To correlate logs across microservices.
- To debug complex issues by following a request's path.
- To enable distributed tracing.
- To improve observability and reduce MTTR.

### Syntax Rules and Structure

**Using `g` and a Custom Filter:**

```python
import uuid
import logging
from flask import Flask, g, request

app = Flask(__name__)

class RequestIDFilter(logging.Filter):
    def filter(self, record):
        record.request_id = getattr(g, 'request_id', 'N/A')
        return True

# Add filter to the logger
app.logger.addFilter(RequestIDFilter())

# Update formatter to include request_id
handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    '%(asctime)s %(levelname)s [%(request_id)s] %(message)s'
))
app.logger.addHandler(handler)

@app.before_request
def set_request_id():
    g.request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))

@app.route('/test')
def test():
    app.logger.info('Processing request')
    return 'OK'
```

**Using `contextvars` (for async and threads):**

```python
import uuid
import logging
from contextvars import ContextVar
from flask import Flask, request

request_id_var = ContextVar('request_id', default='N/A')

class RequestIDFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id_var.get()
        return True

app.logger.addFilter(RequestIDFilter())

@app.before_request
def set_request_id():
    request_id_var.set(request.headers.get('X-Request-ID', str(uuid.uuid4())))
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `g.request_id` | Request-scoped storage for the ID |
| `RequestIDFilter` | Injects the ID into log records |
| `%(request_id)s` | Formatter placeholder |
| `X-Request-ID` | HTTP header for propagation |

**Syntax Rules:**

- Generate the ID in `before_request`.
- Use a `logging.Filter` to inject the ID into all records.
- Include `%(request_id)s` in the formatter.
- Propagate the ID to downstream services via headers.

**Constraints and Limitations:**

- The filter must be added to the logger or handler.
- In async views, use `contextvars` instead of `g` (though `g` works in Flask's async support).
- The ID must be propagated explicitly to external services.

### Annotated Code Examples

**Example 1: Request ID in Logs**

```python
import uuid
import logging
from flask import Flask, g, request

app = Flask(__name__)

class RequestIDFilter(logging.Filter):
    def filter(self, record):
        record.request_id = getattr(g, 'request_id', 'N/A')
        return True

app.logger.addFilter(RequestIDFilter())
handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter(
    '%(asctime)s %(levelname)s [%(request_id)s] %(message)s'
))
app.logger.addHandler(handler)

@app.before_request
def set_request_id():
    g.request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))

@app.route('/api/data')
def get_data():
    app.logger.info('Fetching data')
    app.logger.info('Data fetched successfully')
    return {'data': 'value'}
```

**Expected Output:**
```
2024-01-15 10:30:00,123 INFO [a1b2c3d4-...] Fetching data
2024-01-15 10:30:00,124 INFO [a1b2c3d4-...] Data fetched successfully
```

**Why this output:** The `RequestIDFilter` injects the request ID from `g` into every log record. The formatter includes `%(request_id)s`, so all logs for a request share the same ID.

### Real-World Cases

- **Microservices:** Tracing requests across multiple services.
- **Debugging:** Following a single request's execution path.
- **Observability:** Correlating logs, metrics, and traces.
- **Compliance:** Auditing request lifecycles.

### References

- Flask Request ID Logging (Stack Overflow) — https://stackoverflow.com/questions/50856917/how-to-add-request-id-to-flask-logs
- Python `logging.Filter` — https://docs.python.org/3/library/logging.html#filter-objects
- OpenTelemetry Python — https://opentelemetry.io/docs/languages/python/

---

## 8. Error Tracking Telemetry Integrations (Sentry, Rollbar, APM)

### Definitions

**Core Definition:** Error tracking telemetry integration is the practice of sending application errors, exceptions, and performance data to an external monitoring platform (e.g., Sentry, Rollbar, Datadog APM) for aggregation, alerting, and analysis.

**Technical Definition:** Error tracking platforms provide SDKs that integrate with Flask via `app.errorhandler(Exception)` or dedicated middleware. Sentry's `sentry-sdk[flask]` automatically captures unhandled exceptions, adds request context, and sends events to the Sentry server. Rollbar's `pyrollbar` provides similar functionality via `rollbar.init()` and `Rollbar(app)`. APM platforms (Datadog, New Relic) provide deeper performance monitoring, including transaction traces, database query analysis, and distributed tracing.

**Beginner-Friendly Explanation:** Error tracking tools like Sentry automatically capture errors from your app and send them to a dashboard. You can see all your errors in one place, get alerts when new errors occur, and see exactly what the user was doing when the error happened.

### Purposes

- To automatically capture and aggregate exceptions.
- To receive alerts when new errors occur.
- To see the full context of errors (request data, user, stack trace).
- To track error trends and prioritize fixes.
- To monitor application performance (APM).

### Syntax Rules and Structure

**Sentry Integration:**

```python
import sentry_sdk
from sentry_sdk.integrations.flask import FlaskIntegration
from flask import Flask

sentry_sdk.init(
    dsn="https://your-dsn@sentry.io/project-id",
    integrations=[FlaskIntegration()],
    traces_sample_rate=0.1,  # 10% of transactions for performance
    environment="production"
)

app = Flask(__name__)
```

**Rollbar Integration:**

```python
import rollbar
import rollbar.contrib.flask
from flask import Flask, got_request_exception

app = Flask(__name__)

rollbar.init(
    'your-access-token',
    environment='production',
    handler='blocking'
)

@app.before_first_request
def init_rollbar():
    rollbar.contrib.flask.report_exception(app)

# Or use the Flask extension:
# Rollbar(app)
```

**Datadog APM (ddtrace):**

```python
from ddtrace import patch_all
patch_all()  # Automatically patches Flask, SQLAlchemy, etc.

from ddtrace import tracer
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    with tracer.trace('custom.operation') as span:
        span.set_tag('user.id', 42)
        return 'Hello'
```

**Component Breakdown:**

| Platform | SDK | Integration |
|----------|-----|-------------|
| Sentry | `sentry-sdk[flask]` | `FlaskIntegration` |
| Rollbar | `rollbar` | `rollbar.contrib.flask` |
| Datadog APM | `ddtrace` | `patch_all()` |
| New Relic | `newrelic` | `newrelic.agent` |

**Syntax Rules:**

- Initialize the SDK before the Flask app is created or at startup.
- Configure the DSN/token from environment variables.
- Use `traces_sample_rate` to control performance data volume.
- Set the environment (production, staging, development).

**Constraints and Limitations:**

- Sending data to external services requires network access.
- Sampling rates reduce data volume but may miss errors.
- SDKs add overhead to request processing.
- Sensitive data must be scrubbed before sending.

### Annotated Code Examples

**Example 1: Sentry Integration**

```python
import os
import sentry_sdk
from sentry_sdk.integrations.flask import FlaskIntegration
from flask import Flask

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[FlaskIntegration()],
    traces_sample_rate=0.2,
    environment=os.environ.get('FLASK_ENV', 'production'),
    release='myapp@1.0.0'
)

app = Flask(__name__)

@app.route('/crash')
def crash():
    raise ValueError('Simulated crash')
```

**Expected Output:**
- `GET /crash` → the error is captured and sent to Sentry.
- The Sentry dashboard shows the `ValueError`, stack trace, request context, and environment.

**Why this output:** The Sentry Flask integration automatically captures unhandled exceptions and sends them to the Sentry server with full context.

### Real-World Cases

- **Production monitoring:** Capturing all errors in one dashboard.
- **Alerting:** Getting notified of new errors via Slack, email, or PagerDuty.
- **Performance:** Identifying slow endpoints and database queries.
- **Release tracking:** Associating errors with specific deployments.

### References

- Sentry Flask Integration — https://docs.sentry.io/platforms/python/guides/flask/
- Rollbar Flask Integration — https://docs.rollbar.com/docs/python
- Datadog APM for Python — https://docs.datadoghq.com/tracing/trace_collection/dd_libraries/python/
- New Relic Python Agent — https://docs.newrelic.com/docs/apm/agents/python-agent/

---

## References

- Flask Logging — https://flask.palletsprojects.com/en/stable/logging/
- Python `logging` Documentation — https://docs.python.org/3/library/logging.html
- Python Logging Cookbook — https://docs.python.org/3/howto/logging-cookbook.html
- Python `logging.handlers` — https://docs.python.org/3/library/logging.handlers.html
- python-json-logger — https://github.com/madzak/python-json-logger
- Flask Request Logging (Stack Overflow) — https://stackoverflow.com/questions/32331929/how-to-log-every-request-in-flask
- Flask Request ID Logging (Stack Overflow) — https://stackoverflow.com/questions/50856917/how-to-add-request-id-to-flask-logs
- Sentry Flask Integration — https://docs.sentry.io/platforms/python/guides/flask/
- Rollbar Flask Integration — https://docs.rollbar.com/docs/python
- Datadog APM for Python — https://docs.datadoghq.com/tracing/trace_collection/dd_libraries/python/
- New Relic Python Agent — https://docs.newrelic.com/docs/apm/agents/python-agent/
- OpenTelemetry Python — https://opentelemetry.io/docs/languages/python/