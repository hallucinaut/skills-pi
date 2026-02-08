---
name: backend-api-python
description: "Build secure, production-ready FastAPI backends with Alembic migrations, SQLite for dev, PostgreSQL for prod, and industry-leading security practices."
---

# Backend API Python (FastAPI) Skill

Build secure, production-ready FastAPI backends with industry-leading security practices.

## When to Use

Use this skill when the user wants to:
- Build RESTful APIs with FastAPI framework
- Implement secure authentication and authorization
- Create database migrations with Alembic
- Use SQLite for development, PostgreSQL for production
- Implement rate limiting and input validation
- Set up API documentation (OpenAPI/Swagger)
- Implement comprehensive error handling
- Secure file uploads and sensitive data
- Implement security headers and CORS policies
- Set up environment-based configuration
- Create production-ready database schemas

## Technology Stack

### Core Framework
- **FastAPI**: Modern async Python web framework
  - Automatic OpenAPI documentation
  - Type hints and validation
  - Async/await support
  - Built-in dependency injection
  - WebSocket support

### Database
- **Alembic**: Database migration tool
- **SQLite**: Development database
- **PostgreSQL**: Production database
- **SQLAlchemy**: ORM for database operations

### Security Libraries
- **python-jose/cryptography**: JWT handling and encryption
- **passlib/bcrypt**: Password hashing
- **Pydantic**: Input validation and data modeling
- **python-multipart**: File upload handling
- **slowapi**: Rate limiting
- **trafaret**: Data validation
- **gunicorn/uvicorn**: ASGI server deployment

### Dev Tools
- **pytest**: Testing framework
- **pytest-asyncio**: Async testing
- **black**: Code formatting
- **ruff**: Linting
- **httpx**: HTTP client for testing

## Project Structure

```
backend-api-python/
├── .env                         # Environment variables
├── .env.example                 # Environment template
├── alembic/                     # Alembic migrations
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI app instance
│   ├── config.py                # Configuration settings
│   ├── dependencies.py          # FastAPI dependencies
│   ├── models/                  # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── base.py              # Base model
│   │   └── user.py
│   ├── schemas/                 # Pydantic schemas
│   │   ├── __init__.py
│   │   └── user.py
│   ├── api/                     # API routers
│   │   ├── __init__.py
│   │   ├── deps.py              # API dependencies
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── users.py
│   │       └── auth.py
│   ├── core/                    # Core security modules
│   │   ├── __init__.py
│   │   ├── security.py          # Security utilities
│   │   ├── config.py            # Security config
│   │   └── constants.py
│   ├── crud/                    # CRUD operations
│   │   ├── __init__.py
│   │   └── user.py
│   ├── db/                      # Database utilities
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── session.py
│   ├── services/                # Business logic
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── user.py
│   ├── utils/                   # Helper functions
│   │   ├── __init__.py
│   │   ├── helpers.py
│   │   └── error_handlers.py
│   └── __init__.py
├── requirements.txt             # Production dependencies
├── requirements-dev.txt         # Development dependencies
├── alembic.ini                  # Alembic configuration
├── pyproject.toml               # Project configuration
└── README.md
```

## Security Best Practices

### 1. Authentication & Authorization

**Password Storage:**
- Always use bcrypt or Argon2 for password hashing
- Store only the hash, never the plaintext password
- Use a secure salt value

**JWT Security:**
- Set appropriate expiration times (short-lived tokens)
- Use secure secret keys from environment variables
- Store refresh tokens securely (httpOnly, secure cookies)
- Implement proper token validation and expiration checking

**Authentication Flow:**
```python
# Example: Secure authentication implementation
from passlib.context import CryptContext
from jose import jwt
from datetime import datetime, timedelta
from fastapi import Depends, HTTPException, status

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Verify password against hash."""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """Hash a password."""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: timedelta = None):
    """Create JWT access token with security measures."""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt
```

### 2. Input Validation

**Use Pydantic Models:**
- Always validate all inputs with Pydantic schemas
- Use type hints for better IDE support
- Implement custom validation logic when needed
- Use `EmailStr` for email validation

**Validation Example:**
```python
from pydantic import BaseModel, EmailStr, validator, Field
from typing import Optional

class UserCreate(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    password: str = Field(..., min_length=8, max_length=128)

    @validator('password')
    def password_complexity(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Must contain at least one uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Must contain at least one digit')
        return v
```

### 3. Database Security

**Connection Security:**
- Never hardcode database credentials
- Use environment variables for all sensitive data
- Implement SSL connections for PostgreSQL
- Use environment-based configuration for dev/prod

**SQL Injection Prevention:**
- Always use SQLAlchemy ORM (parameterized queries)
- Avoid raw SQL queries unless absolutely necessary
- Use parameterized queries with proper escaping

### 4. API Security

**Rate Limiting:**
- Implement rate limiting to prevent abuse
- Use per-user rate limiting
- Apply stricter limits to sensitive endpoints

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, rate_limit_exceeded_handler)
```

**CORS Configuration:**
- Configure CORS properly for production
- Specify allowed origins, methods, and headers
- Don't use wildcards (*) in production

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],  # Never use "*"
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)
```

**Input Sanitization:**
- Sanitize all user inputs
- Validate and sanitize file uploads
- Use safe HTML escaping for output

### 5. File Upload Security

**Upload Restrictions:**
- Limit file size
- Validate file types
- Store uploads outside web root
- Generate secure random filenames

```python
from fastapi import UploadFile, File, HTTPException

MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB
ALLOWED_EXTENSIONS = {'.jpg', '.jpeg', '.png', '.pdf'}

async def validate_upload(file: UploadFile = File(...)):
    # Check file size
    contents = await file.read()
    if len(contents) > MAX_FILE_SIZE:
        raise HTTPException(400, "File too large")

    # Check extension
    file_ext = os.path.splitext(file.filename)[1].lower()
    if file_ext not in ALLOWED_EXTENSIONS:
        raise HTTPException(400, "Invalid file type")

    # Return validated data
    return {
        "filename": secure_filename(file.filename),
        "size": len(contents),
        "content_type": file.content_type
    }
```

### 6. Error Handling

**Don't Expose Sensitive Errors:**
- Never expose stack traces to clients in production
- Use generic error messages
- Log detailed errors server-side

**Custom Error Responses:**
```python
from fastapi import Request

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"Unexpected error: {exc}", exc_info=True)
    return JSONResponse(
        status_code=500,
        content={"detail": "An unexpected error occurred"}
    )
```

### 7. Security Headers

**Set Security Headers:**
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- X-XSS-Protection: 1; mode=block
- Content-Security-Policy
- Strict-Transport-Security (HSTS)

```python
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from starlette.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(HTTPSRedirectMiddleware)
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["yourdomain.com"]
)
```

### 8. Environment Variables

**Secure Configuration:**
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Database
    DATABASE_URL: str
    DATABASE_URL_DEV: str
    DATABASE_URL_PROD: str

    # JWT
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    # Application
    DEBUG: bool = False
    ENVIRONMENT: str = "development"

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        case_sensitive = True

settings = Settings()
```

### 9. Data Protection

**Sensitive Data:**
- Never log or expose sensitive data
- Use encryption for sensitive fields in database
- Implement proper data masking

**HTTPS Required:**
- Always use HTTPS in production
- Configure redirect from HTTP to HTTPS
- Set HSTS header

### 10. Dependency Injection

**Secure Dependencies:**
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
) -> User:
    """Get current authenticated user."""
    try:
        token = credentials.credentials
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(status_code=401, detail="Invalid token")
        user = crud.get_user(db, user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        return user
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### 11. Audit Logging

**Track Security Events:**
- Log authentication attempts
- Log failed login attempts
- Track sensitive data access
- Keep logs secure and immutable

### 12. Third-Party Dependencies

**Dependency Management:**
- Regularly update dependencies
- Use lock files (requirements.txt, poetry.lock)
- Monitor for security vulnerabilities
- Use pip-audit or similar tools

## Quick Start Commands

```bash
# Create new FastAPI project
pip install fastapi uvicorn alembic sqlalchemy psycopg2-binary python-jose passlib python-multipart

# Initialize database
alembic init alembic

# Create new migration
alembic revision --autogenerate -m "Create initial tables"

# Apply migrations
alembic upgrade head

# Run server
uvicorn app.main:app --reload
```

## Getting Started

1. Clone or set up the project structure
2. Install dependencies from requirements.txt
3. Configure environment variables from .env.example
4. Run database migrations with Alembic
5. Start the development server
6. Access API documentation at `/docs`

## Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
