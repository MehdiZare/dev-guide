# Environment Management

This guide outlines our organization's standards for managing environment variables, secrets, and configuration across different environments.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start

```bash
# Create environment files
cp .env.example .env.local

# Set environment variables
export API_KEY=your-api-key

# Run with environment variables
# Python
poetry run python -m app.main

# Node.js
npm run start
```

## Environment Types

Our development workflow uses the following environments:

1. **Local** - Individual developer machines
2. **Development** - Shared development environment
3. **Staging** - Pre-production testing environment
4. **Production** - Live production environment

## Environment Files

### Standard Files

- `.env.example` - Template with dummy values (committed to git)
- `.env.local` - Local development values (not committed)
- `.env.development` - Development environment values (not committed)
- `.env.staging` - Staging environment values (not committed)
- `.env.production` - Production environment values (not committed)

### Loading Environment Variables

#### Python Projects

```python
# Install python-dotenv
poetry add python-dotenv

# In your code (app/core/config.py)
import os
from pathlib import Path
from dotenv import load_dotenv
from pydantic_settings import BaseSettings

# Determine environment
ENV = os.getenv("ENV", "development")

# Load the appropriate .env file
env_path = Path(f".env.{ENV}")
load_dotenv(dotenv_path=env_path)

# Fallback to .env.local for local development
if ENV == "development" and not env_path.exists():
    load_dotenv(dotenv_path=Path(".env.local"))

class Settings(BaseSettings):
    API_KEY: str
    DEBUG: bool = False
    DATABASE_URL: str
    SECRET_KEY: str
    
    class Config:
        env_file = f".env.{ENV}"
        env_file_encoding = "utf-8"

settings = Settings()
```

#### Next.js Projects

Next.js has built-in support for environment variables:

- `.env` - Base environment variables (committed to git)
- `.env.local` - Local overrides (not committed)
- `.env.development` - Development environment variables (not committed)
- `.env.production` - Production environment variables (not committed)

Access in code:

```javascript
// Environment variables on the server-side
const apiKey = process.env.API_KEY;

// Environment variables in the browser
// Must be prefixed with NEXT_PUBLIC_
const publicApiUrl = process.env.NEXT_PUBLIC_API_URL;
```

#### Express Projects

```javascript
// Install dotenv
npm install dotenv

// In your code (src/config/index.js)
const dotenv = require('dotenv');
const path = require('path');

// Determine environment
const ENV = process.env.NODE_ENV || 'development';

// Load the appropriate .env file
dotenv.config({
  path: path.resolve(process.cwd(), `.env.${ENV}`),
});

// Fallback to .env.local for local development
if (ENV === 'development') {
  try {
    const localEnvConfig = dotenv.config({
      path: path.resolve(process.cwd(), '.env.local'),
    });
    // Only override if .env.local exists
    if (localEnvConfig.parsed) {
      process.env = {
        ...process.env,
        ...localEnvConfig.parsed,
      };
    }
  } catch (error) {
    console.log('No .env.local file found, using .env.development');
  }
}

module.exports = {
  apiKey: process.env.API_KEY,
  port: parseInt(process.env.PORT || '3000', 10),
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  environment: ENV,
};
```

## Secret Management

### Local Development

For local development, use `.env.local` files:

```
# .env.local
API_KEY=your-local-api-key
DATABASE_URL=postgres://user:password@localhost:5432/db
SECRET_KEY=your-local-secret-key
```

### Cloud Environments

#### AWS Parameter Store

For AWS deployments, use AWS Systems Manager Parameter Store:

```bash
# Store parameters
aws ssm put-parameter --name "/app/production/DATABASE_URL" --value "postgres://user:password@db.example.com:5432/db" --type SecureString

# Get parameters
aws ssm get-parameter --name "/app/production/DATABASE_URL" --with-decryption
```

Access in code:

```python
# Python
import boto3

def get_ssm_parameter(name, with_decryption=True):
    """Get a parameter from AWS SSM Parameter Store."""
    ssm = boto3.client('ssm')
    response = ssm.get_parameter(
        Name=name,
        WithDecryption=with_decryption
    )
    return response['Parameter']['Value']

# Get database URL
database_url = get_ssm_parameter('/app/production/DATABASE_URL')
```

```javascript
// JavaScript
const AWS = require('aws-sdk');

async function getSSMParameter(name, withDecryption = true) {
  const ssm = new AWS.SSM();
  const response = await ssm.getParameter({
    Name: name,
    WithDecryption: withDecryption,
  }).promise();
  return response.Parameter.Value;
}

// Get database URL
const databaseUrl = await getSSMParameter('/app/production/DATABASE_URL');
```

#### AWS Secrets Manager

For critical secrets, use AWS Secrets Manager:

```bash
# Store secrets
aws secretsmanager create-secret --name "/app/production/API_KEYS" --secret-string '{"primary": "your-api-key", "secondary": "your-backup-key"}'

# Get secrets
aws secretsmanager get-secret-value --secret-id "/app/production/API_KEYS"
```

Access in code:

```python
# Python
import boto3
import json

def get_secret(name):
    """Get a secret from AWS Secrets Manager."""
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=name)
    
    if 'SecretString' in response:
        return json.loads(response['SecretString'])
    else:
        return response['SecretBinary']

# Get API keys
api_keys = get_secret('/app/production/API_KEYS')
primary_key = api_keys['primary']
```

```javascript
// JavaScript
const AWS = require('aws-sdk');

async function getSecret(name) {
  const client = new AWS.SecretsManager();
  const response = await client.getSecretValue({
    SecretId: name,
  }).promise();
  
  if ('SecretString' in response) {
    return JSON.parse(response.SecretString);
  } else {
    return response.SecretBinary;
  }
}

// Get API keys
const apiKeys = await getSecret('/app/production/API_KEYS');
const primaryKey = apiKeys.primary;
```

### GitHub Actions Secrets

For CI/CD workflows, use GitHub repository secrets:

1. Navigate to your GitHub repository
2. Go to Settings > Secrets and variables > Actions
3. Add repository secrets (e.g., `API_KEY`, `DATABASE_URL`)

Reference in workflows:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy application
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "Deploying with API_KEY and DATABASE_URL"
```

## Configuration Hierarchy

Configuration should be loaded in the following priority (highest to lowest):

1. Runtime environment variables
2. Secret management service (AWS Secrets Manager, AWS Parameter Store)
3. Environment-specific `.env` files
4. Default values in code

This ensures that:
- Values can be overridden at runtime
- Secrets are securely managed
- Environment-specific configuration is maintained
- Application has sensible defaults

## Naming Conventions

### Environment Variables

Follow these naming conventions for environment variables:

- Use uppercase with underscores
- Use namespacing for related variables
- Be descriptive but concise

Examples:
```
# Good
DATABASE_URL=postgres://user:password@localhost:5432/db
REDIS_HOST=localhost
REDIS_PORT=6379
AUTH_JWT_SECRET=your-jwt-secret
AUTH_JWT_EXPIRY=3600

# Bad
db=postgres://user:password@localhost:5432/db
redisserver=localhost
redisport=6379
secret=your-jwt-secret
tokenexpiry=3600
```

### Secret Naming in AWS

Follow these naming conventions for AWS Parameter Store and Secrets Manager:

- Use path-like structure (`/app/environment/category/name`)
- Include environment in the path
- Group related secrets

Examples:
```
# Parameter Store
/app/production/database/url
/app/production/redis/host
/app/production/redis/port

# Secrets Manager
/app/production/auth/jwt-secrets
/app/production/api/keys
```

## Sensitive Information

### What Not to Store in Version Control

Never store the following in version control:

- API keys and tokens
- Database credentials
- Private keys
- JWT secrets
- User credentials
- IP addresses of production servers

### Handling Sensitive Data

1. **Use Environment Variables**:
    - Store sensitive data in environment variables
    - Reference environment variables in code

2. **Use Secret Management Services**:
    - Store credentials in AWS Secrets Manager
    - Rotate secrets regularly

3. **Use `.gitignore`**:
    - Add all `.env*` files to `.gitignore` (except `.env.example`)
    - Add credentials files to `.gitignore`

Example `.gitignore` entry:
```
# Environment variables
.env
.env.local
.env.development
.env.staging
.env.production

# Credentials
credentials/
*.pem
*.key
```

## Environment Setup Automation

### Environment Setup Scripts

Create scripts for setting up development environments:

```bash
#!/bin/bash
# setup-dev.sh

# Copy example environment file
cp .env.example .env.local

# Generate random secret key
SECRET_KEY=$(openssl rand -hex 32)
echo "SECRET_KEY=$SECRET_KEY" >> .env.local

# Set up local database
docker-compose up -d database
echo "DATABASE_URL=postgres://user:password@localhost:5432/db" >> .env.local

echo "Development environment set up!"
```

### Docker Environment Variables

For containerized applications, pass environment variables to Docker:

```yaml
# docker-compose.yml
version: '3'

services:
  app:
    build: .
    environment:
      - NODE_ENV=production
      - PORT=3000
    env_file:
      - .env.production
```

Or using Docker run:

```bash
docker run -d \
  --name my-app \
  -e NODE_ENV=production \
  -e PORT=3000 \
  --env-file .env.production \
  my-app-image
```

## Testing with Environment Variables

### Unit Tests

Set up test environment variables:

```python
# Python (pytest)
# conftest.py
import os
import pytest

@pytest.fixture(autouse=True)
def env_setup():
    """Set up environment variables for tests."""
    os.environ["DATABASE_URL"] = "sqlite:///:memory:"
    os.environ["SECRET_KEY"] = "test-secret-key"
    os.environ["DEBUG"] = "True"
    
    yield
    
    # Clean up after tests
    os.environ.pop("DATABASE_URL", None)
    os.environ.pop("SECRET_KEY", None)
    os.environ.pop("DEBUG", None)
```

```javascript
// JavaScript (Jest)
// jest.setup.js
process.env.DATABASE_URL = 'sqlite:///:memory:';
process.env.SECRET_KEY = 'test-secret-key';
process.env.DEBUG = 'true';
```

### Integration Tests

Create a separate `.env.test` file:

```
DATABASE_URL=postgresql://test:test@localhost:5432/test_db
SECRET_KEY=test-secret-key
DEBUG=true
```

## Troubleshooting

### Common Environment Issues

1. **Missing Environment Variables**:
    - Check if `.env` file exists
    - Verify variable names (case-sensitive)
    - Ensure environment loader is configured correctly

2. **Wrong Environment Selected**:
    - Check `NODE_ENV` or `ENV` variable
    - Verify environment-specific file is being loaded

3. **Environment Variables Not Available in Deployment**:
    - Check if variables are set in deployment platform
    - Verify secret manager access permissions

### Debugging Environment Variables

Python:
```python
import os

def debug_env():
    """Print all environment variables (excluding sensitive ones)."""
    sensitive_keys = ['SECRET', 'PASSWORD', 'KEY', 'TOKEN']
    
    for key, value in os.environ.items():
        # Skip sensitive variables or truncate them
        if any(s in key.upper() for s in sensitive_keys):
            print(f"{key}=*****")
        else:
            print(f"{key}={value}")

# Call when troubleshooting
debug_env()
```

JavaScript:
```javascript
function debugEnv() {
  // Print all environment variables (excluding sensitive ones)
  const sensitiveKeys = ['SECRET', 'PASSWORD', 'KEY', 'TOKEN'];
  
  Object.keys(process.env).forEach(key => {
    // Skip sensitive variables or truncate them
    if (sensitiveKeys.some(s => key.toUpperCase().includes(s))) {
      console.log(`${key}=*****`);
    } else {
      console.log(`${key}=${process.env[key]}`);
    }
  });
}

// Call when troubleshooting
debugEnv();
```