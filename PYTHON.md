# Python Development Standards

This guide outlines our organization's standards for Python development, focusing on FastAPI for backend services.

> 📌 **Return to**: [Main Development Guide](../README.md)

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
├── .env.example        # Example environment variables
├── poetry.lock         # Lock file for dependencies
├── pyproject.toml      # Project configuration
└── README.md           # Project documentation
```

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
poetry add --dev pytest pytest-cov

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

## FastAPI Best Practices

### Route Organization

- Group related endpoints in the same router
- Use versioned API paths (e.g., `/api/v1/users`)
- Implement dependency injection for shared logic

Example router setup:

```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/")
async def get_users():
    return {"users": []}

# app/api/v1/router.py
from fastapi import APIRouter
from app.api.v1.endpoints import users

api_router = APIRouter(prefix="/api/v1")
api_router.include_router(users.router)

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
from pydantic import BaseModel, EmailStr

class UserBase(BaseModel):
    email: EmailStr
    full_name: str

class UserCreate(UserBase):
    password: str

class UserUpdate(BaseModel):
    email: EmailStr | None = None
    full_name: str | None = None
    
class UserInDB(UserBase):
    id: int
    is_active: bool
    
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
from fastapi import FastAPI, HTTPException, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={"detail": exc.errors()},
    )

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id not in items:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found",
        )
    return {"item": items[item_id]}
```

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
        404: {"description": "User not found"},
        200: {"description": "User details"},
    },
)
async def get_user(user_id: int):
    """
    Get user details by ID.
    
    - **user_id**: The unique identifier of the user
    """
    # Implementation
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

class Settings(BaseSettings):
    DATABASE_URL: str
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    DEBUG: bool = False
    ENVIRONMENT: str = "development"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

## Version Management

Python version updates are handled as follows:
- We maintain compatibility with the latest stable Python release (currently 3.12)
- New projects should always use the current standardized version
- Existing projects should update to new Python versions within 6 months of their release
- Major version upgrades (e.g., 3.x to 3.y) require team-wide coordination and testing