---
applyTo: "**/*.py,**/*.js,**/*.ts,**/*.rb,**/*.java,**/*.go,**/manifest*.yml"
---

# Cloud.gov Logging Instructions

This document provides guidance for implementing logging and monitoring in cloud.gov applications.

## Overview

Cloud.gov automatically captures logs written to `stdout` and `stderr`. Applications should:
- Write all logs to stdout/stderr (not files)
- Use structured logging (JSON format recommended)
- Include contextual information for debugging
- Never log sensitive data (PII, credentials, tokens)

## Viewing Logs

### Recent Logs

```bash
# View recent logs (last ~100 lines)
cf logs my-app --recent
```

### Stream Live Logs

```bash
# Stream logs in real-time
cf logs my-app

# Press Ctrl+C to stop streaming
```

### Log Types

Cloud.gov provides several log types:

| Type | Source | Description |
|------|--------|-------------|
| `APP` | Application | Your application's stdout/stderr |
| `RTR` | Router | HTTP request logs |
| `STG` | Staging | Build/staging process logs |
| `CELL` | Cell | Container lifecycle events |
| `API` | Cloud Controller | CF CLI operations |

### Filter Logs

```bash
# View only application logs (grep for APP)
cf logs my-app --recent | grep "\[APP"

# View only router logs
cf logs my-app --recent | grep "\[RTR"
```

## Application Logging

### Basic Logging Configuration

#### Python

```python
import logging
import sys

# Configure logging to stdout
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.StreamHandler(sys.stdout)]
)

logger = logging.getLogger(__name__)

# Log messages
logger.info("Application started")
logger.warning("Something might be wrong")
logger.error("An error occurred")
```

#### Node.js

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Log messages
logger.info('Application started');
logger.warn('Something might be wrong');
logger.error('An error occurred');
```

#### Ruby

```ruby
require 'logger'

logger = Logger.new(STDOUT)
logger.level = Logger::INFO

logger.info("Application started")
logger.warn("Something might be wrong")
logger.error("An error occurred")
```

### Structured Logging (Recommended)

Structured JSON logs are easier to parse and analyze:

#### Python with structlog

```python
import structlog
import sys

structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    context_class=dict,
    logger_factory=structlog.PrintLoggerFactory(),
)

logger = structlog.get_logger()

# Structured logging with context
logger.info("user_action", 
    action="login",
    user_id=12345,
    ip_address="192.168.1.1"
)
```

Output:

```json
{"event": "user_action", "action": "login", "user_id": 12345, "ip_address": "192.168.1.1", "level": "info", "timestamp": "2024-01-15T10:30:00Z"}
```

#### Python with python-json-logger

```python
import logging
from pythonjsonlogger import jsonlogger

logger = logging.getLogger()
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter(
    '%(timestamp)s %(level)s %(name)s %(message)s'
)
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("Request processed", extra={
    "request_id": "abc123",
    "method": "POST",
    "path": "/api/users",
    "status": 201,
    "duration_ms": 45
})
```

#### Node.js with Pino

```javascript
const pino = require('pino');

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  timestamp: pino.stdTimeFunctions.isoTime
});

// Structured logging
logger.info({
  event: 'request_processed',
  method: 'POST',
  path: '/api/users',
  status: 201,
  duration_ms: 45
});
```

### Log Levels

Use appropriate log levels:

| Level | When to Use |
|-------|-------------|
| `DEBUG` | Detailed diagnostic information |
| `INFO` | Normal application events |
| `WARNING` | Unexpected but handled situations |
| `ERROR` | Errors that need attention |
| `CRITICAL` | Severe errors, application may crash |

```python
# Control log level via environment variable
import os
log_level = os.environ.get('LOG_LEVEL', 'INFO')
logging.basicConfig(level=getattr(logging, log_level))
```

In `manifest.yml`:

```yaml
applications:
  - name: my-app
    env:
      LOG_LEVEL: INFO  # or DEBUG, WARNING, ERROR
```

## Correlation IDs for Distributed Tracing

Cloud.gov sets an `X-Vcap-Request-Id` header on every request. Capture and propagate this ID across your services to enable end-to-end tracing through log analysis.

**NIST 800-53: AU-3 — Content of Audit Records | AU-12 — Audit Record Generation**

### Python Flask

```python
import uuid
from flask import Flask, request, g

app = Flask(__name__)

@app.before_request
def set_correlation_id():
    # cloud.gov/CF sets X-Vcap-Request-Id automatically
    g.correlation_id = request.headers.get(
        'X-Vcap-Request-Id',
        request.headers.get('X-Request-ID', str(uuid.uuid4()))
    )

@app.after_request
def add_correlation_header(response):
    response.headers['X-Correlation-Id'] = g.correlation_id
    return response
```

Include the correlation ID in every log entry:

```python
@app.after_request
def log_request_with_correlation(response):
    logger.info("request", extra={
        "correlation_id": g.correlation_id,
        "method": request.method,
        "path": request.path,
        "status": response.status_code
    })
    return response
```

### Node.js Express

```javascript
app.use((req, res, next) => {
  // cloud.gov/CF sets x-vcap-request-id automatically
  req.correlationId = req.headers['x-vcap-request-id']
    || req.headers['x-request-id']
    || uuid.v4();
  res.setHeader('X-Correlation-Id', req.correlationId);
  next();
});
```

When calling downstream services, propagate the correlation ID:

```python
import requests

def call_downstream_service(url, correlation_id):
    return requests.get(url, headers={
        'X-Correlation-Id': correlation_id
    })
```

This enables tracing a single user request across multiple services in your log drain.

## Request Logging

### Python Flask

```python
import logging
import time
from flask import Flask, request, g

app = Flask(__name__)
logger = logging.getLogger(__name__)

@app.before_request
def before_request():
    g.start_time = time.time()
    g.request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))

@app.after_request
def after_request(response):
    duration = time.time() - g.start_time
    logger.info("request", extra={
        "request_id": g.request_id,
        "method": request.method,
        "path": request.path,
        "status": response.status_code,
        "duration_ms": round(duration * 1000, 2),
        "remote_addr": request.headers.get('X-Forwarded-For', request.remote_addr)
    })
    return response
```

### Node.js Express

```javascript
const morgan = require('morgan');
const uuid = require('uuid');

// Add request ID
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || uuid.v4();
  next();
});

// JSON request logging
app.use(morgan((tokens, req, res) => {
  return JSON.stringify({
    event: 'request',
    request_id: req.id,
    method: tokens.method(req, res),
    path: tokens.url(req, res),
    status: parseInt(tokens.status(req, res)),
    duration_ms: parseFloat(tokens['response-time'](req, res)),
    remote_addr: req.headers['x-forwarded-for'] || req.ip
  });
}));
```

## Error Logging

### Python Exception Logging

```python
import logging
import traceback

logger = logging.getLogger(__name__)

def process_request():
    try:
        # Your code here
        result = risky_operation()
    except ValueError as e:
        # Log expected errors at WARNING
        logger.warning("validation_error", extra={
            "error_type": "ValueError",
            "message": str(e)
        })
        raise
    except Exception as e:
        # Log unexpected errors at ERROR with traceback
        logger.error("unexpected_error", extra={
            "error_type": type(e).__name__,
            "message": str(e),
            "traceback": traceback.format_exc()
        })
        raise
```

### Node.js Error Logging

```javascript
app.use((err, req, res, next) => {
  logger.error({
    event: 'error',
    request_id: req.id,
    error_type: err.name,
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });
  
  res.status(500).json({ error: 'Internal server error' });
});
```

## Health Check Logging

Implement a health endpoint with minimal logging:

```python
@app.route('/health')
def health():
    # Don't log every health check (too noisy)
    return jsonify({"status": "healthy"}), 200

# Only log health checks at DEBUG level
@app.after_request
def conditional_logging(response):
    if request.path == '/health':
        logger.debug("health_check", extra={"status": response.status_code})
    else:
        # Normal request logging
        pass
    return response
```

## Log Drains (External Logging)

To send logs to an external logging service:

### Create User-Provided Drain

```bash
cf cups my-log-drain -l syslog-tls://logs.example.com:6514
cf bind-service my-app my-log-drain
cf restage my-app
```

### Common Log Drain Services

- **Papertrail**: `syslog-tls://logsN.papertrailapp.com:XXXXX`
- **Splunk**: `syslog-tls://input.splunk.example.com:6514`
- **Sumo Logic**: `syslog-tls://endpoint.sumologic.com:6514`

## Security Considerations

### Never Log Sensitive Data

```python
# BAD - Never do this!
logger.info(f"User login: email={email}, password={password}")
logger.info(f"API call with token: {api_token}")
logger.info(f"Processing SSN: {ssn}")

# GOOD - Log safely
logger.info("user_login", extra={"user_id": user.id})
logger.info("api_call", extra={"endpoint": "/users", "status": 200})
logger.info("record_processed", extra={"record_type": "user_data"})
```

### Automatic Log Redaction

In addition to field-level masking, apply regex-based redaction to catch credentials that may appear in unexpected log output. This is especially important for cloud.gov apps that handle VCAP_SERVICES credentials, CF tokens, and AWS keys from S3/RDS bindings.

**NIST 800-53: AU-9 — Protection of Audit Information | SI-4 — System Monitoring**

#### Python

```python
import re

# Patterns that match common credential formats in cloud.gov environments
REDACTION_PATTERNS = [
    (re.compile(r'(?:password|secret|token|key)["\s:=]+["\']?([a-zA-Z0-9+/=_-]{20,})["\']?', re.IGNORECASE), '[REDACTED]'),
    (re.compile(r'AKIA[0-9A-Z]{16}'), '[AWS-KEY-REDACTED]'),                              # AWS Access Key ID
    (re.compile(r'(?:ghp|gho|ghu|ghs|ghr)_[A-Za-z0-9_]{36}'), '[GITHUB-TOKEN-REDACTED]'), # GitHub tokens
    (re.compile(r'eyJ[A-Za-z0-9-_]+\.eyJ[A-Za-z0-9-_]+'), '[JWT-REDACTED]'),              # JWT tokens
    (re.compile(r'cf\s+auth\s+\S+\s+\S+'), 'cf auth [REDACTED] [REDACTED]'),           # CF auth commands
]

def redact_sensitive(text: str) -> str:
    """Apply pattern-based redaction to log output."""
    for pattern, replacement in REDACTION_PATTERNS:
        text = pattern.sub(replacement, text)
    return text
```

#### Node.js

```javascript
const REDACTION_PATTERNS = [
  [/(?:password|secret|token|key)["\s:=]+["']?([a-zA-Z0-9+\/=_-]{20,})["']?/gi, '[REDACTED]'],
  [/AKIA[0-9A-Z]{16}/g, '[AWS-KEY-REDACTED]'],
  [/(?:ghp|gho|ghu|ghs|ghr)_[A-Za-z0-9_]{36}/g, '[GITHUB-TOKEN-REDACTED]'],
  [/eyJ[A-Za-z0-9\-_]+\.eyJ[A-Za-z0-9\-_]+/g, '[JWT-REDACTED]'],
];

function redactSensitive(text) {
  let redacted = text;
  for (const [pattern, replacement] of REDACTION_PATTERNS) {
    redacted = redacted.replace(pattern, replacement);
  }
  return redacted;
}
```

Apply redaction as a logging filter so it runs automatically on all output, not just fields you explicitly mask.

### Mask Sensitive Fields

```python
SENSITIVE_FIELDS = ['password', 'token', 'api_key', 'secret', 'ssn', 'credit_card']

def sanitize_log_data(data):
    """Remove or mask sensitive fields from log data"""
    if isinstance(data, dict):
        sanitized = {}
        for key, value in data.items():
            if any(field in key.lower() for field in SENSITIVE_FIELDS):
                sanitized[key] = '***REDACTED***'
            else:
                sanitized[key] = sanitize_log_data(value)
        return sanitized
    return data
```

### PII Handling

```python
import hashlib

def anonymize_user_id(user_id):
    """Create a consistent but anonymous user identifier for logs"""
    return hashlib.sha256(f"{user_id}-{SALT}".encode()).hexdigest()[:16]

logger.info("user_action", extra={
    "anonymous_user": anonymize_user_id(user.id),
    "action": "profile_update"
})
```

## Monitoring and Alerts

### Application Metrics

Log key metrics for monitoring:

```python
import time

def track_operation(operation_name):
    def decorator(func):
        def wrapper(*args, **kwargs):
            start = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start
                logger.info("operation_complete", extra={
                    "operation": operation_name,
                    "duration_ms": round(duration * 1000, 2),
                    "success": True
                })
                return result
            except Exception as e:
                duration = time.time() - start
                logger.error("operation_failed", extra={
                    "operation": operation_name,
                    "duration_ms": round(duration * 1000, 2),
                    "success": False,
                    "error": str(e)
                })
                raise
        return wrapper
    return decorator

@track_operation("database_query")
def fetch_user(user_id):
    return db.query(User).get(user_id)
```

### Alerting Patterns

Log events that should trigger alerts:

```python
# High-severity events that should trigger alerts
logger.critical("database_connection_lost", extra={
    "database": "primary",
    "retry_count": 5,
    "alert": True
})

logger.error("payment_processing_failed", extra={
    "transaction_id": tx_id,
    "error_code": "GATEWAY_TIMEOUT",
    "alert": True
})
```

## Troubleshooting

### No Logs Appearing

1. Ensure logging to stdout/stderr (not files)
2. Check log level isn't too restrictive
3. Verify application is running: `cf app my-app`

### Logs Truncated

- Keep individual log lines under 16KB
- Break large payloads into multiple log entries

### High Log Volume

- Reduce health check logging frequency
- Use appropriate log levels (DEBUG only when needed)
- Sample high-frequency events

## Best Practices Summary

1. **Always log to stdout/stderr** - Never write to files
2. **Use structured JSON format** - Easier to parse and analyze
3. **Include request context** - Request ID, user ID, timestamps
4. **Never log sensitive data** - Mask PII, credentials, tokens
5. **Use appropriate log levels** - INFO for normal, ERROR for problems
6. **Log actionable information** - Include enough context to debug
7. **Configure via environment** - LOG_LEVEL in manifest.yml
8. **Consider log drains** - External services for long-term retention

## References

- [Cloud Foundry Logging](https://docs.cloudfoundry.org/devguide/deploy-apps/streaming-logs.html)
- [Twelve-Factor App - Logs](https://12factor.net/logs)
- [Structured Logging](https://www.structlog.org/)
