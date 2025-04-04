# Local Development with Docker

This guide outlines our organization's standards for containerized local development environments using Docker.

> 📌 **Return to**: [Main Development Guide](README.md)

## Quick Start

```bash
# 1. Install Docker and Docker Compose
brew install docker docker-compose

# 2. Clone project repository
git clone git@github.com:organization/project-name.git
cd project-name

# 3. Start development environment
docker-compose up -d

# 4. View logs
docker-compose logs -f

# 5. Stop environment
docker-compose down
```

## Why Containerized Development?

After years of "it works on my machine" issues and onboarding challenges, we've standardized on Docker for development environments. This approach provides:

- **Consistency**: Identical environments across team members
- **Isolation**: Dependencies don't conflict with other projects
- **Portability**: Works the same on any OS
- **Similarity to production**: Dev environment mirrors production setup

Our team saw a 75% reduction in environment-related issues after the adoption of Docker for local development across projects.

## Development Container Structure

### Basic Structure

Every project should include these Docker configuration files:

```
project/
├── docker-compose.yml      # Local dev environment setup
├── Dockerfile.dev          # Development image definition
├── Dockerfile              # Production image definition
└── .dockerignore           # Files to exclude from Docker context
```

### Docker Compose Configuration

Our standard `docker-compose.yml` structure for a typical web application:

```yaml
version: '3.8'

services:
  # Application service
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app               # Mount code for live reloading
      - /app/node_modules    # Preserve node_modules inside container
    ports:
      - "3000:3000"          # Expose application port
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://postgres:postgres@db:5432/devdb
    depends_on:
      - db
      - redis

  # Database service
  db:
    image: postgres:14-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=devdb
    ports:
      - "5432:5432"          # Expose database port

  # Cache service
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"          # Expose Redis port

volumes:
  postgres_data:
  redis_data:
```

## Language-Specific Configurations

### Python Development Container

```dockerfile
# Dockerfile.dev for Python/FastAPI projects
FROM python:3.12-slim

WORKDIR /app

# Install Poetry
RUN pip install poetry==1.7.0 && \
    poetry config virtualenvs.create false

# Copy poetry configuration files
COPY pyproject.toml poetry.lock* ./

# Install dependencies
RUN poetry install --no-root --no-interaction --no-ansi

# Copy application code
COPY . .

# Run the application with hot reload
CMD ["uvicorn", "app.main:app", "--reload", "--host", "0.0.0.0", "--port", "8000"]
```

### Node.js/Next.js Development Container

```dockerfile
# Dockerfile.dev for Next.js projects
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json* ./
RUN npm ci

# Copy application code
COPY . .

# Run with hot reload
CMD ["npm", "run", "dev"]
```

### Multi-Container Applications

For complex applications with multiple services, we've found these practices to be effective:

1. Use the `depends_on` directive to control startup order
2. Use environment variables for service discovery
3. Create a shared network for inter-service communication
4. Use healthchecks to ensure services are ready before dependent services start

```yaml
# Example with healthchecks
services:
  api:
    # ... config ...
    depends_on:
      db:
        condition: service_healthy
        
  db:
    # ... config ...
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

## Volume Management

### Project Files Mounting

For live-reload during development:

```yaml
volumes:
  - .:/app                  # Mount entire project
  - /app/node_modules       # Exclude node_modules from host mount
```

### Persistent Data

For databases and other stateful services:

```yaml
volumes:
  postgres_data:            # Named volume for database persistence
  
services:
  db:
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

## Environment Variables

### Local Development Variables

For local-only environment variables, create a `.env.docker` file:

```
# .env.docker
DATABASE_URL=postgres://postgres:postgres@db:5432/devdb
REDIS_URL=redis://redis:6379/0
API_KEY=development-api-key
DEBUG=true
```

Then reference it in your `docker-compose.yml`:

```yaml
services:
  app:
    env_file:
      - .env.docker
```

### Secrets Management

For sensitive information even in development:

```yaml
services:
  app:
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/devdb
    secrets:
      - dev_api_key

secrets:
  dev_api_key:
    file: ./secrets/dev_api_key.txt
```

## Developer Workflow

### Common Commands

```bash
# Start all services in background
docker-compose up -d

# View logs for all services
docker-compose logs -f

# View logs for a specific service
docker-compose logs -f app

# Rebuild containers after dependency changes
docker-compose up -d --build

# Run a one-off command in a service
docker-compose exec app npm run lint

# Access a shell in a container
docker-compose exec app sh

# Stop all services
docker-compose down

# Stop all services and remove volumes
docker-compose down -v
```

### Running Tests

```bash
# Run tests
docker-compose exec app npm test

# Run specific test file
docker-compose exec app npm test -- src/components/Button.test.tsx

# Run database migrations
docker-compose exec app npx prisma migrate dev
```

## Development Tools Integration

### VS Code Dev Containers

For projects using VS Code, we recommend the "Dev Containers" extension:

1. Install the "Remote - Containers" extension
2. Add `.devcontainer/devcontainer.json` to your project:

```json
{
  "name": "Project Development",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/app",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-python.python"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "python.linting.enabled": true
      }
    }
  }
}
```

### Debugging

#### Python Debugging

Update your `docker-compose.yml` to enable debugging:

```yaml
services:
  app:
    # ... other configuration ...
    ports:
      - "8000:8000"   # Application port
      - "5678:5678"   # Debug port
    command: ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:app", "--reload", "--host", "0.0.0.0", "--port", "8000"]
```

#### Node.js Debugging

Update your `docker-compose.yml` for Node.js debugging:

```yaml
services:
  app:
    # ... other configuration ...
    ports:
      - "3000:3000"   # Application port
      - "9229:9229"   # Debug port
    command: ["npm", "run", "dev:debug"]
```

In `package.json`:
```json
"scripts": {
  "dev:debug": "next dev --inspect=0.0.0.0:9229"
}
```

## Performance Optimization

After encountering sluggish performance in Docker on macOS, we've found these optimization techniques effective:

### Volume Performance

For projects with many files (especially node_modules):

1. Use volume caching:
   ```yaml
   volumes:
     - .:/app:cached        # Use cached mode for better performance
   ```

2. For Node.js projects, use a dedicated volume for node_modules:
   ```yaml
   volumes:
     - .:/app
     - node_modules:/app/node_modules
   
   # Define named volume
   volumes:
     node_modules:
   ```

### Resource Allocation

Ensure Docker has adequate resources:

- At least 4 CPU cores
- At least 8GB of RAM
- At least 2GB of swap space

These settings can be configured in Docker Desktop preferences.

## Multi-Platform Development

For teams with mixed OSes (macOS, Windows, Linux), ensure compatibility by:

1. Using Linux-compatible line endings:
   ```gitattributes
   # .gitattributes
   * text=auto eol=lf
   ```

2. Being mindful of case sensitivity:
    - MacOS and Windows are case-insensitive
    - Linux is case-sensitive

3. Setting explicit file permissions in Dockerfiles:
   ```dockerfile
   COPY --chown=node:node . .
   ```

## Common Issues and Solutions

### Connection Refused Errors

Problem: Services cannot connect to each other.

Solution: Use service names as hostnames, not localhost:
```
# Wrong
DATABASE_URL=postgres://postgres:postgres@localhost:5432/devdb

# Correct
DATABASE_URL=postgres://postgres:postgres@db:5432/devdb
```

### Volume Permission Issues

Problem: Permission denied when writing to volumes.

Solution: Match the user inside and outside the container:
```dockerfile
# Create a user with same UID/GID as the host user
ARG USER_ID=1000
ARG GROUP_ID=1000

RUN addgroup --gid $GROUP_ID user && \
    adduser --disabled-password --gecos '' --uid $USER_ID --gid $GROUP_ID user

USER user
```

### Slow File Syncing

Problem: Changes to files are slow to appear inside the container.

Solution: Use the `:delegated` volume mount option:
```yaml
volumes:
  - .:/app:delegated
```

## Lessons Learned

### Case Study: API Service Development Environment

When we first containerized our API development environment, we encountered several challenges:

1. **Slow file system performance**: File watching for hot reload was unreliable and slow
2. **Dependency management complexity**: Different dependency versions between local and container
3. **Database migration conflicts**: Developers overwriting each other's local database schema

Our solutions:

1. Implemented volume caching and delegated mounts to improve file system performance
2. Created dedicated volumes for language-specific dependency directories
3. Implemented unique database names per developer using environment variables

These changes reduced our setup time for new developers from several hours to less than 15 minutes.

### Case Study: React Native Development

For our mobile application development with React Native, we initially struggled with:

1. Metro bundler performance in Docker
2. Device connection issues
3. Native module compatibility

After experimentation, we settled on a hybrid approach where:
- JavaScript bundling runs in Docker
- Native builds run on the host machine
- We use Docker for the API and backend services

This balanced approach improved team productivity while maintaining consistency.

## Docker Compose Overrides

For developer-specific customizations without modifying the main configuration, use override files:

1. Create a `docker-compose.override.yml` (gitignored):

```yaml
# docker-compose.override.yml
services:
  app:
    environment:
      - DEBUG=true
      - MY_CUSTOM_VAR=value
    volumes:
      - ~/my-custom-path:/app/custom
```

2. Create a `.env` file for personal settings:

```
# .env
COMPOSE_PROJECT_NAME=myproject-jane
POSTGRES_PORT=5433  # Use alternate port to avoid conflicts
```

This approach allows developers to customize their environment without affecting team members.

## Cleanup and Maintenance

Regular maintenance keeps Docker running smoothly:

```bash
# Remove unused containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove everything unused
docker system prune -a --volumes
```

Create a `cleanup.sh` script in your project for easy maintenance.