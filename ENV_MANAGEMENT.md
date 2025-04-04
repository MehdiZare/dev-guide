# Environment Management

This guide outlines our organization's standards for managing environment variables, secrets, and configuration across different environments.

> 📌 **Return to**: [Main Development Guide](README.md)

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

After a production incident where debugging was hampered by inconsistent environment variables, we've established strict standards for what goes in each file.

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
    
    # Additional validation
    @validator("DATABASE_URL")
    def validate_database_url(cls, v):
        if not v.startswith("postgresql://") and not v.startswith("postgres://"):
            raise ValueError("DATABASE_URL must be a PostgreSQL connection string")
        return v
    
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

// Validate environment variables at startup
// src/lib/validateEnv.js
function validateEnv() {
  const requiredVars = [
    'DATABASE_URL',
    'NEXT_PUBLIC_API_URL',
    'NEXT_AUTH_SECRET'
  ];
  
  const missingVars = requiredVars.filter(
    (varName) => !process.env[varName]
  );
  
  if (missingVars.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missingVars.join(', ')}`
    );
  }
}

// Call at application startup
// src/server.js or src/pages/api/index.js
validateEnv();
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

// Validate environment variables
const requiredEnvVars = [
  'DATABASE_URL',
  'PORT',
  'JWT_SECRET'
];

const missingEnvVars = requiredEnvVars.filter(
  (varName) => !process.env[varName]
);

if (missingEnvVars.length > 0) {
  throw new Error(
    `Missing required environment variables: ${missingEnvVars.join(', ')}`
  );
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

# Use a cached version for performance
from functools import lru_cache

@lru_cache(maxsize=128)
def get_cached_secret(name):
    """Get a cached secret from AWS Secrets Manager."""
    return get_secret(name)
```

```javascript
// JavaScript
const AWS = require('aws-sdk');
const NodeCache = require('node-cache');

// Cache for secrets to avoid rate limiting
const secretsCache = new NodeCache({ stdTTL: 300 }); // Cache for 5 minutes

async function getSecret(name) {
  // Check cache first
  const cachedSecret = secretsCache.get(name);
  if (cachedSecret) {
    return cachedSecret;
  }
  
  // Get from Secrets Manager
  const client = new AWS.SecretsManager();
  const response = await client.getSecretValue({
    SecretId: name,
  }).promise();
  
  let secret;
  if ('SecretString' in response) {
    secret = JSON.parse(response.SecretString);
  } else {
    secret = response.SecretBinary;
  }
  
  // Store in cache
  secretsCache.set(name, secret);
  
  return secret;
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

### Kubernetes Secrets

For Kubernetes deployments:

```yaml
# kubernetes/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DATABASE_URL: cG9zdGdyZXM6Ly91c2VyOnBhc3N3b3JkQGRiLmV4YW1wbGUuY29tOjU0MzIvZGI=
  API_KEY: eW91ci1hcGkta2V5

# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: your-app:latest
        env:
          - name: DATABASE_URL
            valueFrom:
              secretKeyRef:
                name: app-secrets
                key: DATABASE_URL
          - name: API_KEY
            valueFrom:
              secretKeyRef:
                name: app-secrets
                key: API_KEY
```

## Microservice Configuration Sharing

After resolving several hard-to-debug issues caused by inconsistent configuration across microservices, we've established these patterns for configuration sharing:

### Configuration Service Pattern

For sharing common configuration between microservices:

```
     ┌─────────────────┐
     │                 │
     │  Config Service │
     │                 │
     └───────┬─────────┘
             │
     ┌───────┼─────────┐
     │       │         │
┌────▼───┐ ┌─▼───┐ ┌───▼────┐
│        │ │     │ │        │
│Service1│ │Svc 2│ │Service3│
│        │ │     │ │        │
└────────┘ └─────┘ └────────┘
```

```python
# config_service.py
import requests
from functools import lru_cache
import os

class ConfigService:
    def __init__(self):
        self.base_url = os.environ.get("CONFIG_SERVICE_URL")
        self.service_name = os.environ.get("SERVICE_NAME")
        self.environment = os.environ.get("ENVIRONMENT")
    
    @lru_cache(maxsize=128)
    def get_config(self, key):
        """Get configuration value from the config service."""
        url = f"{self.base_url}/config/{self.service_name}/{self.environment}/{key}"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()["value"]
    
    def refresh_config(self):
        """Clear the config cache to fetch fresh values."""
        self.get_config.cache_clear()

# Usage
config_service = ConfigService()
database_url = config_service.get_config("DATABASE_URL")
```

### Environment Variable Prefixing

For avoiding conflicts in environment variables:

```
# Service 1
SVC1_DATABASE_URL=postgres://user:password@db1.example.com:5432/db1
SVC1_API_KEY=service1-api-key

# Service 2
SVC2_DATABASE_URL=postgres://user:password@db2.example.com:5432/db2
SVC2_API_KEY=service2-api-key
```

```python
# service1/config.py
import os

# Get all environment variables with prefix
prefix = "SVC1_"
config = {
    key[len(prefix):]: value
    for key, value in os.environ.items()
    if key.startswith(prefix)
}

database_url = config.get("DATABASE_URL")
api_key = config.get("API_KEY")
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

Implementation example:

```python
# app/core/config.py
import os
import boto3
import json
from pathlib import Path
from dotenv import load_dotenv
from functools import lru_cache

# 1. Determine environment
ENV = os.getenv("ENV", "development")

# 2. Load environment-specific .env file
env_path = Path(f".env.{ENV}")
if env_path.exists():
    load_dotenv(dotenv_path=env_path)

# 3. Load .env.local for local development
if ENV == "development":
    local_env_path = Path(".env.local")
    if local_env_path.exists():
        load_dotenv(dotenv_path=local_env_path)

# 4. Get secrets from AWS Secrets Manager
@lru_cache(maxsize=128)
def get_secret(name):
    """Get a secret from AWS Secrets Manager."""
    if ENV in ["development", "local"]:
        # Don't use Secrets Manager for local development
        return None
    
    try:
        client = boto3.client('secretsmanager')
        response = client.get_secret_value(SecretId=name)
        
        if 'SecretString' in response:
            return json.loads(response['SecretString'])
        else:
            return response['SecretBinary']
    except Exception as e:
        print(f"Error getting secret {name}: {e}")
        return None

# 5. Configuration class
class Config:
    # Database
    DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///app.db")
    
    # API Keys
    API_KEY = os.getenv("API_KEY")
    if API_KEY is None and ENV not in ["development", "local"]:
        api_keys = get_secret(f"/app/{ENV}/API_KEYS")
        if api_keys and "primary" in api_keys:
            API_KEY = api_keys["primary"]
    
    # Application Settings
    DEBUG = os.getenv("DEBUG", "False").lower() in ["true", "1", "yes"]
    LOG_LEVEL = os.getenv("LOG_LEVEL", "INFO")
    
    # Security
    SECRET_KEY = os.getenv("SECRET_KEY", "development-secret-key")
    if ENV not in ["development", "local"] and SECRET_KEY == "development-secret-key":
        # Force real secret key in non-development environments
        raise ValueError("SECRET_KEY must be set in non-development environments")

# Create configuration instance
config = Config()
```

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

### Secrets Rotation

Implement regular secrets rotation:

```python
# secrets_rotation.py
import boto3
import uuid
import json

def rotate_api_key(secret_name):
    """Rotate API key in Secrets Manager."""
    # Generate new API key
    new_api_key = str(uuid.uuid4())
    
    # Get current secret
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    
    current_secret = json.loads(response['SecretString'])
    
    # Update secret with new primary key and old key as secondary
    updated_secret = {
        "primary": new_api_key,
        "secondary": current_secret["primary"]
    }
    
    # Update secret in Secrets Manager
    client.update_secret(
        SecretId=secret_name,
        SecretString=json.dumps(updated_secret)
    )
    
    return {
        "primary": new_api_key,
        "secondary": current_secret["primary"]
    }

# Usage
rotated_keys = rotate_api_key("/app/production/API_KEYS")
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

### Kubernetes ConfigMaps and Secrets

For Kubernetes, use ConfigMaps for non-sensitive configuration:

```yaml
# kubernetes/config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  NODE_ENV: "production"
  PORT: "3000"
  LOG_LEVEL: "info"

# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: your-app:latest
        envFrom:
          - configMapRef:
              name: app-config
          - secretRef:
              name: app-secrets
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

## Case Studies & Lessons Learned

### Case Study: Production Database Incident

**Problem**: A developer accidentally connected to the production database from their local machine because environment variables weren't properly isolated.

**Solution**:
1. Created dedicated connection strings for each environment
2. Implemented environment-specific prefixes for all services
3. Added validation to prevent production connections from development environments
4. Set up read-only roles for development access when absolutely necessary

**Implementation**:
```python
# app/core/database.py
from app.core.config import config

def get_db_connection():
    """Get database connection with environment safety checks."""
    if config.ENV == "development" and "production" in config.DATABASE_URL:
        raise ValueError(
            "Attempted to connect to production database from development environment!"
        )
        
    # Proceed with connection
    # ...
```

### Case Study: Secret Leakage

**Problem**: API keys were accidentally committed to a public repository.

**Solution**:
1. Immediately rotated all exposed secrets
2. Implemented pre-commit hooks to detect secrets
3. Added automated scanning in CI pipeline
4. Created an incident response plan for leaked credentials

**Implementation**:
```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.15.0
  hooks:
  - id: gitleaks
```

```yaml
# .github/workflows/secret-scan.yml
name: Secrets Scanning

on:
  push:
    branches: [ main, development ]
  pull_request:
    branches: [ main, development ]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - uses: gitleaks/gitleaks-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Case Study: Microservice Configuration Drift

**Problem**: Inconsistent configuration between microservices led to hard-to-debug integration issues.

**Solution**:
1. Implemented a centralized configuration service
2. Standardized configuration naming across services
3. Created automated validation of configuration compatibility
4. Documented all configuration dependencies

**Implementation**:
```python
# app/core/config_validator.py
def validate_service_compatibility():
    """Validate that service configurations are compatible."""
    # Get configuration for dependent services
    auth_service_url = config.get("AUTH_SERVICE_URL")
    user_service_url = config.get("USER_SERVICE_URL")
    
    # Verify services are available
    services = [
        {"name": "Auth Service", "url": f"{auth_service_url}/health"},
        {"name": "User Service", "url": f"{user_service_url}/health"},
    ]
    
    for service in services:
        try:
            response = requests.get(service["url"], timeout=5)
            response.raise_for_status()
            
            # Verify API version compatibility
            api_version = response.headers.get("X-API-Version")
            if not api_version or not semver.match(api_version, ">=2.0.0 <3.0.0"):
                raise ValueError(
                    f"{service['name']} API version {api_version} is not compatible "
                    f"with this service (requires >=2.0.0 <3.0.0)"
                )
                
        except requests.RequestException as e:
            raise ValueError(f"Failed to connect to {service['name']}: {e}")
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

### Environment Validation

Implement validation to catch configuration issues early:

```python
# app/core/validate_env.py
def validate_environment():
    """Validate environment configuration on startup."""
    required_vars = ["DATABASE_URL", "SECRET_KEY"]
    
    missing_vars = [var for var in required_vars if not os.getenv(var)]
    if missing_vars:
        raise ValueError(f"Missing required environment variables: {', '.join(missing_vars)}")
    
    # Validate database URL
    db_url = os.getenv("DATABASE_URL")
    if not db_url.startswith(("postgres://", "postgresql://", "sqlite://")):
        raise ValueError(f"Invalid DATABASE_URL: {db_url}")
    
    # Validate secret key in production
    if os.getenv("ENV") == "production":
        secret_key = os.getenv("SECRET_KEY")
        if not secret_key or len(secret_key) < 32:
            raise ValueError("SECRET_KEY must be at least 32 characters in production")
```

## Environment Documentation

### Environment Documentation Template

Create a dedicated document describing all environment variables:

```markdown
# Environment Configuration

This document describes all environment variables used in the application.

## Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| DATABASE_URL | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/db` |
| SECRET_KEY | Secret key for JWT tokens | `a-very-long-secret-key` |

## Optional Variables

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| DEBUG | Enable debug mode | `False` | `True` |
| LOG_LEVEL | Logging level | `INFO` | `DEBUG` |
| PORT | Server port | `8000` | `3000` |

## Environment-Specific Variables

### Development

| Variable | Description | Default |
|----------|-------------|---------|
| MOCK_SERVICES | Use mock external services | `True` |

### Production

| Variable | Description | Required |
|----------|-------------|----------|
| REDIS_URL | Redis connection string | Yes |
| NEW_RELIC_LICENSE_KEY | New Relic license key | No |
```

### Generate `.env.example`

Automate the creation of `.env.example` to ensure it stays up-to-date:

```python
# scripts/generate_env_example.py
import os
import re
from pathlib import Path

def find_env_vars(directory):
    """Find all environment variables used in the codebase."""
    env_vars = set()
    
    # Regex pattern to match os.getenv and os.environ.get calls
    pattern = r'os\.(?:getenv|environ\.get)\(["\']([A-Za-z0-9_]+)["\']'
    
    # Walk through all .py files
    for path in Path(directory).rglob("*.py"):
        with open(path, "r") as f:
            content = f.read()
            for match in re.finditer(pattern, content):
                env_vars.add(match.group(1))
    
    return sorted(env_vars)

def generate_env_example(env_vars):
    """Generate .env.example file with found variables."""
    with open(".env.example", "w") as f:
        f.write("# Environment Variables\n\n")
        
        for var in env_vars:
            if "SECRET" in var or "PASSWORD" in var or "KEY" in var:
                f.write(f"{var}=your-{var.lower()}\n")
            else:
                f.write(f"{var}=\n")

# Usage
env_vars = find_env_vars("./app")
generate_env_example(env_vars)
```