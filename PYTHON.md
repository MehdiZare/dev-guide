# Python Development Standards

This guide outlines our organization's standards for Python development, focusing on FastAPI for backend services.

> 📌 **Return to**: [Main Development Guide](README.md)

## Quick Start for Python Projects

```bash
# 1. Set up Python environment
brew install pyenv poetry
pyenv install 3.12.0
pyenv global 3.12.0

# 2. Clone repository
git clone git@github.com:organization/project-name-api.git
cd project-name-api

# 3. Install dependencies
poetry install
poetry shell

# 4. Run development server
poetry run uvicorn app.main:app --reload

# 5. Create a new feature branch
git checkout -b feature/your-feature-name
```

## Environment Setup

### Python Version Management

We use `pyenv` to manage Python versions:

```bash
# Install pyenv
brew install pyenv

# Add to your shell (for bash)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

# Install Python version
pyenv install 3.12.0

# Set as global version
pyenv global 3.12.0
```

### Dependency Management

We use `poetry` for dependency management:

```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Configure poetry
poetry config virtualenvs.in-project true

# Inside your project directory
poetry init  # For new projects
poetry install  # For existing projects

# Add dependencies
poetry add fastapi uvicorn[standard]
poetry add --dev pytest black flake8 isort mypy

# Activate the virtual environment
poetry shell
```

After struggling with requirements.txt and pip-tools on our payment processing system, we standardized on Poetry. The deterministic lockfile has eliminated "works on my machine" dependency issues.

## Project Structure

Standard Python project structure:

```
backend/
├── app/                # Main application package
│   ├── __init__.py     # Package initialization
│   ├── main.py         # Application entry point
│   ├── api/            # API endpoints
│   │   ├── __init__.py
│   │   ├── v1/         # API version
│   │   │   ├── __init__.py
│   │   │   ├── endpoints/
│   │   │   └── router.py
│   ├── core/           # Core functionality
│   │   ├── __init__.py
│   │   ├── config.py   # Configuration
│   │   └── security.py # Authentication
│   ├── models/         # Data models
│   ├── schemas/        # Pydantic schemas
│   └── services/       # Business logic
├── tests/              # Test directory
│   ├── conftest.py     # Test configuration
│   ├── unit/           # Unit tests
│   └── integration/    # Integration tests
├── .env.example        # Example environment variables
├── poetry.lock         # Lock file for dependencies
├── pyproject.toml      # Project configuration
└── README.md           # Project documentation
```

This structure evolved from our experience building multiple production services. The separation of concerns between API endpoints, business logic (services), and data models has significantly improved maintainability and testability.

## Code Quality Tools

### Pre-commit Configuration

Create `.pre-commit-config.yaml` in your project root:

```yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.4.0
  hooks:
  - id: trailing-whitespace
  - id: end-of-file-fixer
  - id: check-yaml
  - id: check-added-large-files

- repo: https://github.com/psf/black
  rev: 23.3.0
  hooks:
  - id: black
    language_version: python3.12

- repo: https://github.com/pycqa/isort
  rev: 5.12.0
  hooks:
  - id: isort
    name: isort (python)

- repo: https://github.com/pycqa/flake8
  rev: 6.0.0
  hooks:
  - id: flake8
    additional_dependencies: [flake8-docstrings]

- repo: https://github.com/pre-commit/mirrors-mypy
  rev: v1.3.0
  hooks:
  - id: mypy
    additional_dependencies: [pydantic, types-requests]
```

Install and run pre-commit:

```bash
poetry add --dev pre-commit
pre-commit install
pre-commit run --all-files
```

Our automated code quality checks catch approximately 80% of common issues before they reach code review, saving significant developer time.

### Configuration Files

#### pyproject.toml

```toml
[tool.poetry]
name = "project-name"
version = "0.1.0"
description = "Project description"
authors = ["Your Name <your.email@example.com>"]

[tool.poetry.dependencies]
python = "^3.12"
fastapi = "^0.104.0"
uvicorn = {extras = ["standard"], version = "^0.23.0"}
pydantic = "^2.4.2"
pydantic-settings = "^2.0.3"
sqlalchemy = "^2.0.23"
alembic = "^1.12.0"
python-dotenv = "^1.0.0"

[tool.poetry.dev-dependencies]
pytest = "^7.4.0"
pytest-asyncio = "^0.21.1"
black = "^23.10.0"
isort = "^5.12.0"
flake8 = "^6.1.0"
mypy = "^1.6.1"
pre-commit = "^3.5.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.black]
line-length = 88
target-version = ["py312"]

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.12"
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
strict_optional = true
warn_redundant_casts = true
warn_return_any = true
warn_unused_ignores = true
```

#### setup.cfg (for flake8)

```ini
[flake8]
max-line-length = 88
extend-ignore = E203
exclude = .git,__pycache__,dist,build,.venv
```

## Testing

We use `pytest` for testing:

```bash
# Add test dependencies
poetry add --dev pytest pytest-cov pytest-asyncio

# Run tests
poetry run pytest

# Run tests with coverage
poetry run pytest --cov=app tests/
```

Example test file (`tests/test_api.py`):

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}
```

### Test Structure

Organize tests to mirror your application structure:

```
tests/
├── conftest.py          # Shared fixtures
├── unit/                # Unit tests
│   ├── test_models.py   # Test models
│   └── test_services.py # Test services
└── integration/         # Integration tests
    └── test_api.py      # Test API endpoints
```

### Testing Best Practices

1. **Isolation**: Each test should be independent
2. **Fixtures**: Use pytest fixtures for setup and teardown
3. **Mocking**: Use unittest.mock for external dependencies
4. **Coverage**: Aim for 80%+ code coverage
5. **Parametrization**: Use pytest.mark.parametrize for multiple test cases

Example fixture for database testing:

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import StaticPool

from app.core.database import Base, get_db
from app.main import app

@pytest.fixture(scope="function")
def test_db():
    # Create in-memory SQLite database for testing
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
    
    # Create tables
    Base.metadata.create_all(bind=engine)
    
    # Use the database
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture(scope="function")
def client(test_db):
    # Override dependency
    def override_get_db():
        try:
            yield test_db
        finally:
            pass
    
    app.dependency_overrides[get_db] = override_get_db
    
    with TestClient(app) as client:
        yield client
    
    # Clear dependency override
    app.dependency_overrides.clear()
```

## FastAPI Best Practices

### Route Organization

- Group related endpoints in the same router
- Use versioned API paths (e.g., `/api/v1/users`)
- Implement dependency injection for shared logic

Example router setup:

```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from app.core.database import get_db
from app.core.security import get_current_user
from app.schemas.user import User, UserCreate, UserUpdate
from app.services.user import UserService

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/", response_model=list[User])
async def get_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    """
    Get all users.
    """
    return UserService(db).get_users(skip=skip, limit=limit)

@router.post("/", response_model=User, status_code=status.HTTP_201_CREATED)
async def create_user(
    user: UserCreate,
    db: Session = Depends(get_db),
):
    """
    Create a new user.
    """
    return UserService(db).create_user(user=user)

# app/api/v1/router.py
from fastapi import APIRouter
from app.api.v1.endpoints import users, auth, items

api_router = APIRouter(prefix="/api/v1")
api_router.include_router(users.router)
api_router.include_router(auth.router, prefix="/auth", tags=["auth"])
api_router.include_router(items.router, prefix="/items", tags=["items"])

# app/main.py
from fastapi import FastAPI
from app.api.v1.router import api_router

app = FastAPI(title="My API")
app.include_router(api_router)
```

### Data Validation

- Use Pydantic models for request/response validation
- Create separate schemas for different operations (create, update, response)

```python
from pydantic import BaseModel, EmailStr, Field, field_validator
from typing import Optional
from datetime import datetime

class UserBase(BaseModel):
    email: EmailStr
    full_name: str = Field(..., min_length=2, max_length=100)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)
    
    @field_validator("password")
    def password_strength(cls, v):
        # Check password strength
        if not any(char.isdigit() for char in v):
            raise ValueError("Password must contain at least one digit")
        if not any(char.isupper() for char in v):
            raise ValueError("Password must contain at least one uppercase letter")
        return v

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    full_name: Optional[str] = Field(None, min_length=2, max_length=100)
    
class UserInDB(UserBase):
    id: int
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True
        
class User(UserInDB):
    # This is the public response model, excludes sensitive fields
    pass
```

### Error Handling

- Use proper HTTP status codes
- Return consistent error responses

```python
# app/core/errors.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

class AppError(Exception):
    def __init__(self, message: str, status_code: int = 400, data: dict = None):
        self.message = message
        self.status_code = status_code
        self.data = data
        super().__init__(self.message)

def setup_error_handlers(app: FastAPI) -> None:
    @app.exception_handler(AppError)
    async def app_error_handler(request: Request, exc: AppError):
        return JSONResponse(
            status_code=exc.status_code,
            content={
                "error": {
                    "message": exc.message,
                    "data": exc.data,
                }
            },
        )
    
    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(request: Request, exc: RequestValidationError):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            content={
                "error": {
                    "message": "Validation error",
                    "data": {"details": exc.errors()},
                }
            },
        )

# app/main.py
from fastapi import FastAPI
from app.core.errors import setup_error_handlers

app = FastAPI(title="My API")
setup_error_handlers(app)
```

Example usage in services:

```python
# app/services/user.py
from app.core.errors import AppError

class UserService:
    def __init__(self, db):
        self.db = db
    
    def get_user_by_id(self, user_id: int):
        user = self.db.query(UserModel).filter(UserModel.id == user_id).first()
        if not user:
            raise AppError(f"User not found", status_code=404)
        return user
```

## Asynchronous Programming

After several performance bottlenecks in our user service, we've embraced asynchronous programming with FastAPI and `asyncio`.

### Asynchronous Patterns

#### Async/Await Basics

```python
import asyncio
from typing import List
import httpx

async def fetch_data(url: str) -> dict:
    """Fetch data from a URL asynchronously."""
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        response.raise_for_status()
        return response.json()

async def fetch_all_data(urls: List[str]) -> List[dict]:
    """Fetch data from multiple URLs concurrently."""
    tasks = [fetch_data(url) for url in urls]
    return await asyncio.gather(*tasks)
```

#### Using in FastAPI

```python
@router.get("/aggregated-data")
async def get_aggregated_data():
    """Get data from multiple services and aggregate it."""
    urls = [
        "https://service1.example.com/data",
        "https://service2.example.com/data",
        "https://service3.example.com/data",
    ]
    
    data = await fetch_all_data(urls)
    return {"aggregated_data": data}
```

#### Background Tasks

```python
from fastapi import BackgroundTasks

@router.post("/users/")
async def create_user(
    user: UserCreate, 
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db)
):
    """Create a user and send welcome email in background."""
    db_user = UserService(db).create_user(user)
    
    # Add background task
    background_tasks.add_task(
        send_welcome_email, 
        email=db_user.email, 
        name=db_user.full_name
    )
    
    return db_user
```

### Database Access

For database operations, we have two approaches:

#### 1. Synchronous SQLAlchemy with FastAPI

```python
# app/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

from app.core.config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

#### 2. Asynchronous SQLAlchemy (for PostgreSQL)

```python
# app/core/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

from app.core.config import settings

engine = create_async_engine(
    settings.DATABASE_URL.replace("postgresql://", "postgresql+asyncpg://"),
    echo=settings.DEBUG,
)
AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

Base = declarative_base()

async def get_async_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

Example usage:

```python
@router.get("/users/{user_id}")
async def get_user(
    user_id: int, 
    db: AsyncSession = Depends(get_async_db)
):
    """Get a user by ID using async database."""
    stmt = select(UserModel).where(UserModel.id == user_id)
    result = await db.execute(stmt)
    user = result.scalar_one_or_none()
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    
    return user
```

### Asynchronous Lessons Learned

1. **Not Everything Should Be Async**
    - CPU-bound tasks don't benefit from async
    - Use sync code for computation-heavy operations

2. **Avoid Blocking the Event Loop**
    - Use `asyncio.to_thread` for CPU-bound tasks
    - Run blocking I/O in thread pools

   ```python
   import asyncio
   
   def cpu_bound_task(data):
       # Heavy computation
       result = process_data(data)
       return result
   
   @router.get("/heavy-computation")
   async def handle_heavy_computation(data: dict):
       # Run CPU-bound task in a thread pool
       result = await asyncio.to_thread(cpu_bound_task, data)
       return {"result": result}
   ```

3. **Concurrent Request Limiting**
    - Limit concurrent requests to external services
    - Use semaphores to control concurrency

   ```python
   # Limit to 10 concurrent requests
   semaphore = asyncio.Semaphore(10)
   
   async def limited_concurrent_fetch(url):
       async with semaphore:
           return await fetch_data(url)
           
   async def fetch_all_limited(urls):
       tasks = [limited_concurrent_fetch(url) for url in urls]
       return await asyncio.gather(*tasks)
   ```

4. **Task Cancellation**
    - Implement proper cancellation handling
    - Use timeouts for external requests

   ```python
   async def fetch_with_timeout(url, timeout=10):
       try:
           async with httpx.AsyncClient() as client:
               return await asyncio.wait_for(
                   client.get(url), 
                   timeout=timeout
               )
       except asyncio.TimeoutError:
           # Handle timeout
           return None
   ```

After implementing these patterns in our user service, we reduced average response time by 65% and increased throughput by 3x.

## Documentation Standards

### Docstrings

Use Google-style docstrings:

```python
def calculate_total(items: list[Item], tax_rate: float) -> float:
    """
    Calculate the total price including tax.

    Args:
        items: List of items to calculate total for
        tax_rate: Tax rate as a decimal (e.g., 0.07 for 7%)

    Returns:
        Total price including tax
        
    Raises:
        ValueError: If tax_rate is negative
    """
    if tax_rate < 0:
        raise ValueError("Tax rate cannot be negative")
    
    subtotal = sum(item.price for item in items)
    return subtotal * (1 + tax_rate)
```

### API Documentation

- Use FastAPI's built-in Swagger UI and ReDoc
- Add proper descriptions to your API endpoints
- Include examples in your schema definitions

```python
@router.get(
    "/users/{user_id}",
    response_model=User,
    summary="Get user by ID",
    description="Retrieve a user by their unique ID",
    responses={
        200: {"description": "User details"},
        404: {"description": "User not found"},
        401: {"description": "Unauthorized"},
    },
)
async def get_user(
    user_id: int = Path(..., description="The ID of the user to retrieve"),
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    """
    Get user details by ID.
    
    - **user_id**: The unique identifier of the user
    """
    # Implementation
```

## Performance Optimization

### Profiling and Benchmarking

1. **Profiling Tools**
    - Use cProfile for CPU profiling
    - Use memory_profiler for memory profiling

   ```bash
   # Profile with cProfile
   python -m cProfile -o profile.stats app/main.py
   
   # View results
   python -m pstats profile.stats
   
   # Memory profiling
   python -m memory_profiler app/main.py
   ```

2. **API Benchmarking**
    - Use locust or wrk for load testing
    - Create baseline performance metrics

   ```bash
   # Install locust
   pip install locust
   
   # Run load test
   locust -f load_tests/locustfile.py
   ```

3. **Database Query Optimization**
    - Profile slow queries
    - Use indexing appropriately
    - Optimize JOIN operations

### Caching Strategies

1. **In-Memory Caching**
    - Use functools.lru_cache for function results
    - Use Redis for distributed caching

   ```python
   from functools import lru_cache
   
   @lru_cache(maxsize=100)
   def get_user_preferences(user_id: int) -> dict:
       # Expensive operation to get user preferences
       return fetch_preferences_from_db(user_id)
   ```

2. **Redis Caching**
    - Use Redis for shared caching across instances

   ```python
   import redis
   import json
   from fastapi import Depends
   
   redis_client = redis.Redis.from_url(settings.REDIS_URL)
   
   async def get_cached_data(key: str, ttl: int = 3600):
       """Get data from cache or compute and store it."""
       # Try to get from cache
       cached_data = redis_client.get(key)
       if cached_data:
           return json.loads(cached_data)
           
       # Compute data
       data = await expensive_operation()
       
       # Store in cache
       redis_client.setex(
           key, 
           ttl,
           json.dumps(data)
       )
       
       return data
   ```

### Database Optimization

1. **Pagination**
    - Always paginate large result sets
    - Use cursor-based pagination for large datasets

   ```python
   @router.get("/items")
   async def get_items(
       skip: int = Query(0, ge=0),
       limit: int = Query(100, ge=1, le=1000),
   ):
       return db.query(Item).offset(skip).limit(limit).all()
   ```

2. **Optimizing SQLAlchemy**
    - Select only needed columns
    - Use joined loading for related objects
    - Implement database-level filtering

   ```python
   # Select only needed columns
   users = db.query(
       User.id, User.email, User.full_name
   ).all()
   
   # Joined loading
   users = db.query(User).options(
       joinedload(User.orders)
   ).all()
   
   # Database-level filtering
   users = db.query(User).filter(
       User.age > 18,
       User.is_active == True
   ).all()
   ```

## Environment Variables

Create a `.env.example` file with dummy values:

```
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/dbname

# Security
SECRET_KEY=your-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Environment
DEBUG=True
ENVIRONMENT=development
```

Load environment variables in your code:

```python
# app/core/config.py
from pydantic_settings import BaseSettings
from typing import List, Optional

class Settings(BaseSettings):
    # Application
    API_V1_STR: str = "/api/v1"
    PROJECT_NAME: str = "My API"
    DEBUG: bool = False
    ENVIRONMENT: str = "development"
    
    # CORS
    BACKEND_CORS_ORIGINS: List[str] = ["http://localhost:3000"]
    
    # Database
    DATABASE_URL: str
    
    # Security
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    # Redis
    REDIS_URL: Optional[str] = None
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = True

settings = Settings()
```

## Deployment Best Practices

### Docker Containerization

Create a `Dockerfile` for production:

```dockerfile
FROM python:3.12-slim as builder

WORKDIR /app

# Install Poetry
RUN pip install poetry==1.7.0

# Copy poetry configuration files
COPY pyproject.toml poetry.lock* ./

# Configure poetry to not use virtual environments
RUN poetry config virtualenvs.create false && \
    poetry install --no-dev --no-interaction --no-ansi

# Runtime stage
FROM python:3.12-slim

WORKDIR /app

# Copy Python dependencies from builder stage
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy application code
COPY . .

# Run as non-root user
RUN useradd -m appuser && \
    chown -R appuser:appuser /app
USER appuser

# Set environment variables
ENV PYTHONPATH=/app
ENV PORT=8000

# Expose the application port
EXPOSE ${PORT}

# Start the application
CMD uvicorn app.main:app --host 0.0.0.0 --port ${PORT}
```

### Kubernetes Configuration

Example Kubernetes deployment:

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-api
  template:
    metadata:
      labels:
        app: my-api
    spec:
      containers:
      - name: api
        image: ${ECR_REPOSITORY}:${IMAGE_TAG}
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: database-url
        - name: SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: secret-key
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "500m"
            memory: "256Mi"
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 15
          periodSeconds: 20
```

### Health Checks

Implement comprehensive health checks:

```python
# app/api/health.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.core.database import get_db

router = APIRouter(tags=["health"])

@router.get("/health")
async def health_check():
    """Basic health check."""
    return {"status": "ok"}

@router.get("/health/db")
async def db_health_check(db: Session = Depends(get_db)):
    """Check database connection."""
    try:
        # Execute a simple query
        db.execute("SELECT 1")
        return {"status": "ok", "database": "connected"}
    except Exception as e:
        return {"status": "error", "database": str(e)}
```

## Version Management

Python version updates are handled as follows:
- We maintain compatibility with the latest stable Python release (currently 3.12)
- New projects should always use the current standardized version
- Existing projects should update to new Python versions within 6 months of their release
- Major version upgrades (e.g., 3.x to 3.y) require team-wide coordination and testing

Our approach to dependency management:
- Pin all direct dependencies with exact versions
- Update dependencies monthly
- Run automated tests before updating dependencies
- Keep a changelog of dependency updates

## Case Studies

### API Performance Optimization

**Challenge**: Our user service was experiencing high latency during peak load.

**Solution**:
1. Profiled the application to identify bottlenecks
2. Implemented Redis caching for frequently accessed data
3. Switched to asynchronous database queries
4. Added connection pooling and query optimization
5. Implemented background tasks for non-critical operations

**Results**:
- 70% reduction in average response time
- 3x increase in request throughput
- Eliminated timeout errors during peak traffic

### Authentication Service Rewrite

**Challenge**: Our legacy authentication service was difficult to maintain and had security vulnerabilities.

**Solution**:
1. Rewrote the service using FastAPI and modern security practices
2. Implemented JWT-based authentication with proper key rotation
3. Added comprehensive logging and monitoring
4. Used asynchronous processing for token validation
5. Implemented rate limiting and brute force protection

**Results**:
- 80% code reduction compared to the legacy service
- Eliminated all identified security vulnerabilities
- Improved developer experience with better documentation
- Reduced authentication latency by 60%

## Common Pitfalls and Solutions

1. **N+1 Query Problem**
    - **Problem**: Executing one database query to fetch a list, then one query per item
    - **Solution**: Use joinedload or subquery load with SQLAlchemy

2. **Memory Leaks in Long-Running Processes**
    - **Problem**: Memory usage grows over time
    - **Solution**: Use weakref for caching, implement proper cleanup, use memory profiling

3. **Slow Startup Time**
    - **Problem**: Application takes too long to start
    - **Solution**: Lazy loading of resources, startup optimization, warm-up procedures

4. **Dependency Conflicts**
    - **Problem**: Conflicting package versions
    - **Solution**: Use Poetry with locked dependencies, containerize application

5. **Blocking the Event Loop**
    - **Problem**: Long-running synchronous code blocks the async event loop
    - **Solution**: Move CPU-bound tasks to separate processes, use thread pools for I/O-bound tasks