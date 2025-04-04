# Error Tracking with Sentry

This guide outlines our organization's standards for error tracking and monitoring using Sentry.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start

```bash
# Python/FastAPI
poetry add sentry-sdk

# Next.js
npm install @sentry/nextjs

# Express
npm install @sentry/node @sentry/tracing
```

## Setting Up Sentry

### 1. Create a Sentry Account

If you don't have access to our organization's Sentry account, contact the DevOps team for access.

### 2. Create a New Project

1. Log in to Sentry
2. Navigate to Projects
3. Click "Create Project"
4. Select the platform (Python, React, Node.js, etc.)
5. Give your project a name following our naming convention: `<project-name>-<component>-<environment>`
6. Click "Create Project"

### 3. Get Your DSN

After creating a project, you'll be provided with a DSN (Data Source Name). This is used to authenticate your application with Sentry.

## Sentry Integration

### Python (FastAPI)

```python
# app/core/config.py
import os
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # ... other settings
    SENTRY_DSN: str = ""
    ENVIRONMENT: str = "development"

    class Config:
        env_file = ".env"

settings = Settings()

# app/main.py
import sentry_sdk
from fastapi import FastAPI
from sentry_sdk.integrations.fastapi import FastApiIntegration
from app.core.config import settings

def init_sentry():
    if settings.SENTRY_DSN:
        sentry_sdk.init(
            dsn=settings.SENTRY_DSN,
            environment=settings.ENVIRONMENT,
            traces_sample_rate=1.0,  # Adjust in production (0.0 - 1.0)
            integrations=[FastApiIntegration()]
        )

app = FastAPI()
init_sentry()

# In your .env file
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
ENVIRONMENT=development
```

### Next.js

1. Install Sentry CLI and the Next.js SDK:

```bash
npm install --save @sentry/nextjs
```

2. Initialize Sentry using the wizard:

```bash
npx @sentry/wizard -i nextjs
```

This will:
- Create `sentry.client.config.js` and `sentry.server.config.js`
- Update `next.config.js` to add the Sentry webpack plugin

3. Configure Sentry settings in your `.env.local` file:

```
NEXT_PUBLIC_SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
SENTRY_ENVIRONMENT=development
```

4. Manually configure Sentry if needed:

```javascript
// sentry.client.config.js
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.SENTRY_ENVIRONMENT,
  tracesSampleRate: 1.0, // Adjust in production (0.0 - 1.0)
  
  // Optional: Enrich events with additional data
  beforeSend(event) {
    // Modify event data here
    return event;
  },
});

// sentry.server.config.js
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.SENTRY_ENVIRONMENT,
  tracesSampleRate: 1.0, // Adjust in production (0.0 - 1.0)
});

// next.config.js
const { withSentryConfig } = require('@sentry/nextjs');

const nextConfig = {
  // Your Next.js configuration
};

const sentryWebpackPluginOptions = {
  // Additional configuration options for the Sentry webpack plugin
};

module.exports = withSentryConfig(nextConfig, sentryWebpackPluginOptions);
```

### Express

```javascript
// src/config/index.js
const dotenv = require('dotenv');
dotenv.config();

module.exports = {
  // ... other settings
  sentryDsn: process.env.SENTRY_DSN || '',
  environment: process.env.NODE_ENV || 'development'
};

// src/app.js
const express = require('express');
const Sentry = require('@sentry/node');
const Tracing = require('@sentry/tracing');
const config = require('./config');

const app = express();

// Initialize Sentry (must be the first middleware)
if (config.sentryDsn) {
  Sentry.init({
    dsn: config.sentryDsn,
    environment: config.environment,
    integrations: [
      // Enable HTTP calls tracing
      new Sentry.Integrations.Http({ tracing: true }),
      // Enable Express.js middleware tracing
      new Tracing.Integrations.Express({ app }),
    ],
    tracesSampleRate: 1.0, // Adjust in production (0.0 - 1.0)
  });

  // RequestHandler creates a separate execution context
  app.use(Sentry.Handlers.requestHandler());
  // TracingHandler creates a trace for every incoming request
  app.use(Sentry.Handlers.tracingHandler());
}

// ... your middleware and routes

// The error handler must be before any other error middleware
// and after all controllers
if (config.sentryDsn) {
  app.use(Sentry.Handlers.errorHandler());
}

// ... other error handlers

module.exports = app;

// In your .env file
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
NODE_ENV=development
```

## Error Handling Best Practices

### Structured Error Logging

1. **Use Custom Error Classes**:

   ```python
   # Python
   class AppError(Exception):
       def __init__(self, message, status_code=500, extra=None):
           self.message = message
           self.status_code = status_code
           self.extra = extra or {}
           super().__init__(self.message)

   # When catching errors
   try:
       # Some operation
   except Exception as e:
       sentry_sdk.capture_exception(e)
       raise AppError("Operation failed", 500, {"original_error": str(e)})
   ```

   ```javascript
   // JavaScript
   class AppError extends Error {
     constructor(message, statusCode = 500, extra = {}) {
       super(message);
       this.name = this.constructor.name;
       this.statusCode = statusCode;
       this.extra = extra;
       Error.captureStackTrace(this, this.constructor);
     }
   }

   // When catching errors
   try {
     // Some operation
   } catch (error) {
     Sentry.captureException(error);
     throw new AppError('Operation failed', 500, { originalError: error.message });
   }
   ```

2. **Add Context to Errors**:

   ```python
   # Python
   def process_user_data(user_id, data):
       with sentry_sdk.configure_scope() as scope:
           scope.set_user({"id": user_id})
           scope.set_tag("feature", "user-data-processing")
           scope.set_extra("data_size", len(data))
           
           try:
               # Process data
           except Exception as e:
               sentry_sdk.capture_exception(e)
               raise
   ```

   ```javascript
   // JavaScript
   function processUserData(userId, data) {
     Sentry.configureScope(scope => {
       scope.setUser({ id: userId });
       scope.setTag('feature', 'user-data-processing');
       scope.setExtra('dataSize', data.length);
     });
     
     try {
       // Process data
     } catch (error) {
       Sentry.captureException(error);
       throw error;
     }
   }
   ```

### Error Grouping

Configure similar errors to be grouped together in Sentry:

```python
# Python
def custom_before_send(event, hint):
    if 'exc_info' in hint:
        exception = hint['exc_info'][1]
        if isinstance(exception, DatabaseError):
            # Group all database errors together
            event['fingerprint'] = ['database-error']
    return event

sentry_sdk.init(
    dsn=settings.SENTRY_DSN,
    before_send=custom_before_send,
)
```

```javascript
// JavaScript
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  beforeSend(event, hint) {
    const error = hint.originalException;
    if (error && error.name === 'DatabaseError') {
      // Group all database errors together
      event.fingerprint = ['database-error'];
    }
    return event;
  },
});
```

### Performance Monitoring

Monitor application performance with Sentry:

```python
# Python
from sentry_sdk import start_transaction

def process_complex_task(task_id):
    with start_transaction(op="task", name=f"process_task_{task_id}") as transaction:
        # Step 1
        with transaction.start_child(op="step", description="step1"):
            step1()
            
        # Step 2
        with transaction.start_child(op="step", description="step2"):
            step2()
```

```javascript
// JavaScript
const transaction = Sentry.startTransaction({
  op: 'task',
  name: `process_task_${taskId}`,
});

// Step 1
const span1 = transaction.startChild({
  op: 'step',
  description: 'step1',
});
await step1();
span1.finish();

// Step 2
const span2 = transaction.startChild({
  op: 'step',
  description: 'step2',
});
await step2();
span2.finish();

transaction.finish();
```

## Sentry Dashboard Usage

### 1. Monitoring Errors

- Check the Issues tab regularly
- Filter by environment (development, staging, production)
- Sort by frequency or users affected
- Assign issues to team members for investigation

### 2. Setting Up Alerts

Configure alerts for:
- New issues in production
- Issues affecting many users
- Regressions (issues that were previously resolved)

### 3. Resolving Issues

When fixing an issue:
1. Reference the Sentry issue ID in your commit message
2. Mark the issue as resolved in Sentry once fixed
3. Include a brief explanation of the fix

## CI/CD Integration

### 1. Release Tracking

Track releases in Sentry to correlate errors with specific deployments:

```yaml
# GitHub Actions workflow
- name: Create Sentry release
  uses: getsentry/action-release@v1
  env:
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    SENTRY_ORG: your-organization
    SENTRY_PROJECT: your-project
  with:
    environment: production
    version: ${{ github.sha }}
```

### 2. Source Maps

For JavaScript applications, upload source maps to Sentry:

```yaml
# GitHub Actions workflow
- name: Upload source maps to Sentry
  env:
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    SENTRY_ORG: your-organization
    SENTRY_PROJECT: your-project
  run: |
    npx @sentry/cli releases files ${{ github.sha }} upload-sourcemaps ./build
```

## Troubleshooting

### Common Issues

1. **Missing Events in Sentry**:
    - Check DSN configuration
    - Verify integration is properly installed
    - Make sure errors are actually being raised

2. **Too Many Events**:
    - Adjust sampling rate in Sentry initialization
    - Set up event filtering to ignore certain errors
    - Review error boundaries and handlers in your code

3. **Missing Context in Events**:
    - Ensure proper scope configuration
    - Add tags and extra data to your events
    - Set up custom context for specific operations

## Security Considerations

1. **Sensitive Data**:
    - Never log PII (Personally Identifiable Information)
    - Configure Sentry to scrub sensitive fields
    - Use data scrubbing in Sentry configuration

2. **DSN Protection**:
    - Store Sentry DSN in environment variables
    - Different DSNs for different environments
    - Rotate DSNs if compromised

3. **Access Control**:
    - Limit access to Sentry dashboard
    - Implement proper role-based access controls
    - Regularly audit user access