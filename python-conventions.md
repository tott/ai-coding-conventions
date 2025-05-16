# Python Application Conventions

## Introduction

This document outlines the coding conventions and best practices for Python applications in our organization, with special consideration for AI-assisted development. Following these guidelines will ensure consistency, maintainability, and optimal collaboration between human developers and AI assistants.

## Table of Contents

- [Project Structure](#project-structure)
- [Type Hints](#type-hints)
- [API Development with FastAPI](#api-development-with-fastapi)
- [HTTP Client: HTTPX](#http-client-httpx)
- [Web Interface Development](#web-interface-development)
- [Environment Variables Management](#environment-variables-management)
- [Logging](#logging)
- [Scalability Considerations](#scalability-considerations)
- [Testing](#testing)
- [Documentation](#documentation)


## Project Structure

### Recommended Directory Structure

```
project-name/
├── alembic/                  # Database migrations
├── src/
│   ├── domain1/
│   │   ├── router.py         # FastAPI router
│   │   ├── schemas.py        # Pydantic models
│   │   ├── models.py         # DB models
│   │   ├── dependencies.py   # Router dependencies
│   │   ├── config.py         # Domain-specific configs
│   │   ├── constants.py      # Domain-specific constants
│   │   ├── exceptions.py     # Custom exceptions
│   │   ├── service.py        # Business logic
│   │   └── utils.py          # Helper functions
│   ├── domain2/
│   │   └── ...
│   ├── config.py             # Global configs
│   ├── models.py             # Shared models
│   ├── exceptions.py         # Global exceptions
│   ├── logging.py            # Logging configuration
│   ├── database.py           # DB connection setup
│   └── main.py               # Application entry point
├── tests/
│   ├── domain1/
│   ├── domain2/
│   └── conftest.py
├── templates/                # HTML templates
│   └── ...
├── static/
│   ├── css/
│   └── js/
├── tailwindcss/              # TailwindCSS configuration
├── .env                      # Environment variables (not in git)
├── .env.example              # Example environment variables
├── requirements.txt          # Production dependencies
├── requirements-dev.txt      # Development dependencies
├── Dockerfile                # Container definition
└── README.md
```

When importing from other packages, use explicit module names:

```python
from src.auth import constants as auth_constants
from src.notifications import service as notification_service
```


## Type Hints

### General Guidelines

Always use type hints to improve code clarity, enable better IDE support, and facilitate AI code understanding and generation.

```python
# Variables
age: int = 1
names: list[str] = ["Alice", "Bob"]
user_data: dict[str, Any] = {"name": "Alice", "age": 30}

# Functions
def calculate_area(length: float, width: float) -> float:
    return length * width
```


### Best Practices

1. Use `TypeAlias` for type aliases:
```python
from typing import TypeAlias

IntList: TypeAlias = list[int]
```

2. Use the appropriate collection type hints:
```python
# For Python 3.9+
values: list[int] = [1, 2, 3]
mappings: dict[str, float] = {"field": 2.0}
fixed_tuple: tuple[int, str, float] = (3, "yes", 7.5)
variable_tuple: tuple[int, ...] = (1, 2, 3)
```

3. Use `Any` when a type cannot be expressed appropriately with the current type system:
```python
from typing import Any

def process_unknown_data(data: Any) -> str:
    return str(data)
```

4. Use `object` instead of `Any` when a function accepts any possible object but doesn't need specific operations:
```python
def log_value(value: object) -> None:
    print(f"Value: {value}")
```

5. Prefer protocols and abstract types for arguments, concrete types for return values:
```python
from typing import Sequence, Iterable, Mapping

def process_items(items: Sequence[int]) -> list[str]:
    return [str(item) for item in items]
```

6. Use mypy for static type checking:
```bash
python -m pip install mypy
mypy src/
```


## API Development with FastAPI

### Basic Setup

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="My API", version="1.0.0")

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Adjust in production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```


### Async Route Handling

Properly handle async operations to prevent blocking:

```python
# Good - Non-blocking async route
@app.get("/perfect-endpoint")
async def perfect_endpoint():
    await asyncio.sleep(1)  # Non-blocking I/O operation
    return {"status": "success"}

# Good - Sync route for blocking operations
@app.get("/good-endpoint")
def good_endpoint():
    time.sleep(1)  # Blocking operation, but FastAPI runs it in a thread
    return {"status": "success"}

# BAD - Blocking operation in async route
@app.get("/bad-endpoint")
async def bad_endpoint():
    time.sleep(1)  # This blocks the event loop!
    return {"status": "success"}
```


### Structured Error Handling

```python
from fastapi import HTTPException
from pydantic import BaseModel

class ErrorResponse(BaseModel):
    detail: str
    code: str

@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail, "code": "HTTP_ERROR"}
    )
```


### Deployment with Multiple Workers

Use Uvicorn with multiple workers for production:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

For CPU-bound applications, match workers to the number of CPU cores. For I/O-bound applications, you can use more workers than cores.

## HTTP Client: HTTPX

Always prefer HTTPX over requests for making HTTP requests, especially for modern Python applications.

### Synchronous Usage

```python
import httpx

def fetch_data(url: str) -> dict:
    with httpx.Client(timeout=10.0) as client:
        response = client.get(url)
        response.raise_for_status()
        return response.json()
```


### Asynchronous Usage

```python
import httpx
import asyncio

async def fetch_data_async(url: str) -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:
        response = await client.get(url)
        response.raise_for_status()
        return response.json()
```


### Best Practices

1. Use a single client instance for connection pooling:
```python
# Global client for reuse
http_client = httpx.Client()

# Close on application shutdown
@app.on_event("shutdown")
def shutdown_event():
    http_client.close()
```

2. Always set appropriate timeouts:
```python
client = httpx.Client(
    timeout=httpx.Timeout(5.0, connect=3.0)
)
```

3. Use structured error handling:
```python
try:
    response = client.get(url)
    response.raise_for_status()
except httpx.RequestError as exc:
    logger.error(f"Request failed: {exc}")
except httpx.HTTPStatusError as exc:
    logger.error(f"HTTP error: {exc}")
```


## Web Interface Development

### TailwindCSS Setup with FastAPI

1. Install TailwindCSS:
```bash
npm install tailwindcss
```

2. Create a tailwind configuration file:
```js
// tailwind.config.js
module.exports = {
  content: ["../templates/**/*.html"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

3. Set up the FastAPI template system:
```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")
templates = Jinja2Templates(directory="templates")

@app.get("/")
async def index(request: Request):
    return templates.TemplateResponse("base.html", {"request": request})
```

4. Create HTML templates with TailwindCSS classes:
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App</title>
    <link href="/static/css/tailwind.css" rel="stylesheet">
</head>
<body class="bg-gray-100">
    <div class="container mx-auto p-4">
        <h1 class="text-2xl font-bold text-blue-600">Hello, FastAPI with Tailwind!</h1>
    </div>
</body>
</html>
```


## Environment Variables Management

Use python-dotenv for environment variable management.

```python
import os
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Access environment variables
DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./test.db")
DEBUG = os.getenv("DEBUG", "False").lower() == "true"
```


### Best Practices

1. Keep `.env` out of version control:

```
# .gitignore
.env
```

2. Provide `.env.example` with dummy values:

```
# .env.example
DATABASE_URL=postgresql://user:password@localhost/dbname
DEBUG=False
SECRET_KEY=replace_with_secure_key
```

3. Validate required environment variables on startup:

```python
def validate_env_vars():
    required_vars = ["SECRET_KEY", "DATABASE_URL"]
    missing = [var for var in required_vars if not os.getenv(var)]
    if missing:
        raise RuntimeError(f"Missing required environment variables: {', '.join(missing)}")
```


## Logging

### Basic Setup

```python
import logging
from logging.config import dictConfig

LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "default": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        },
        "json": {
            "format": '{"timestamp": "%(asctime)s", "logger": "%(name)s", "level": "%(levelname)s", "message": "%(message)s"}'
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "default"
        },
        "file": {
            "class": "logging.handlers.RotatingFileHandler",
            "filename": "app.log",
            "maxBytes": 10485760,  # 10MB
            "backupCount": 5,
            "formatter": "json"
        },
    },
    "loggers": {
        "app": {
            "handlers": ["console", "file"],
            "level": "INFO",
            "propagate": False
        },
    },
}

dictConfig(LOGGING_CONFIG)
logger = logging.getLogger("app")
```


### FastAPI Integration

```python
@app.middleware("http")
async def log_requests(request: Request, call_next):
    import uuid
    request_id = str(uuid.uuid4())
    logger.info(f"Request {request_id}: {request.method} {request.url}")
    
    response = await call_next(request)
    
    logger.info(f"Response {request_id}: {response.status_code}")
    return response
```


### Best Practices

1. Use appropriate log levels:

```python
logger.debug("Detailed information for debugging")
logger.info("Confirmation of expected events")
logger.warning("Something unexpected but the application still works")
logger.error("An error that prevents a function from working")
logger.critical("An error that prevents the application from working")
```

2. Mask sensitive information:

```python
def mask_email(email: str) -> str:
    username, domain = email.split('@')
    return f"{username[:2]}{'*' * (len(username) - 2)}@{domain}"

logger.info(f"Processing request for user: {mask_email(user.email)}")
```

3. Log structured data for easier parsing:

```python
import json

def log_event(event_type: str, data: dict) -> None:
    logger.info(f"{event_type}: {json.dumps(data)}")
```


## Scalability Considerations

### Statelessness

1. Avoid storing session state in the application memory:

```python
# BAD - In-memory state
user_sessions = {}

# GOOD - Use external session store
from fastapi_sessions.backends.redis import RedisBackend
```

2. Use external storage for shared state:

```python
import redis

redis_client = redis.Redis.from_url(os.getenv("REDIS_URL"))

def increment_counter(key: str) -> int:
    return redis_client.incr(key)
```

3. Design for horizontal scaling:

```python
# Configure connection pooling appropriately
from sqlalchemy.pool import QueuePool

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=5,
    max_overflow=10
)
```


### Multiple Workers

1. Configure Uvicorn with multiple workers:

```python
# In a deployment script
import multiprocessing

workers = multiprocessing.cpu_count() * 2 + 1

# For Gunicorn with Uvicorn workers
# gunicorn -w {workers} -k uvicorn.workers.UvicornWorker main:app
```

2. Ensure workers don't interfere with each other:

```python
# Use atomic operations for shared resources
async def increment_counter(key: str) -> int:
    # Using Redis INCR which is atomic
    return await redis.incr(key)
```


## Testing

### Structure

```
tests/
├── conftest.py           # Shared fixtures
├── test_main.py          # Application-level tests
└── domain1/
    ├── test_api.py       # API tests
    ├── test_models.py    # Model tests
    └── test_services.py  # Service tests
```


### Example Test with Type Hints

```python
import pytest
from httpx import AsyncClient
from typing import AsyncGenerator

@pytest.fixture
async def client() -> AsyncGenerator[AsyncClient, None]:
    from src.main import app
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.mark.asyncio
async def test_read_main(client: AsyncClient) -> None:
    response = await client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}
```


## Documentation

### Docstring Format

Use Google-style docstrings for better AI understanding:

```python
def calculate_area(length: float, width: float) -> float:
    """Calculate the area of a rectangle.
    
    Args:
        length: The length of the rectangle.
        width: The width of the rectangle.
        
    Returns:
        The area of the rectangle.
        
    Raises:
        ValueError: If length or width is negative.
    """
    if length < 0 or width < 0:
        raise ValueError("Length and width must be positive")
    return length * width
```


### README Structure

Ensure your README.md includes:

1. Project description and purpose
2. Installation instructions
3. Usage examples
4. Configuration options
5. Development setup
6. Testing instructions
7. Contribution guidelines
8. License information

---

By following these conventions, you'll create Python applications that are maintainable, scalable, and optimally suited for AI-assisted development.

[^1]: https://typing.python.org/en/latest/reference/best_practices.html

[^2]: https://github.com/chadrik/doc484

[^3]: https://mypy.readthedocs.io/en/stable/existing_code.html

[^4]: https://google.github.io/styleguide/docguide/style.html

[^5]: https://github.com/zhanymkanov/fastapi-best-practices

[^6]: https://victorsejas.com/blog/fastapi/how-to-setup-fastapi-with-tailwindcss/

[^7]: https://www.python-httpx.org/advanced/clients/

[^8]: https://configu.com/blog/using-py-dotenv-python-dotenv-package-to-manage-env-variables/

[^9]: https://signoz.io/guides/python-logging-best-practices/

[^10]: https://www.linkedin.com/pulse/best-practices-logging-fastapi-applications-manikandan-parasuraman-96n2c

[^11]: https://www.linkedin.com/pulse/understanding-uvicorn-workers-md-ataullha-saim-netlc

[^12]: https://joshdimella.com/blog/python-typing-best-practices

[^13]: https://docs.gruntwork.io/guides/style/markdown-style-guide/

[^14]: https://fastapi.tiangolo.com/tutorial/bigger-applications/

[^15]: https://www.python-httpx.org/async/

[^16]: https://dagster.io/blog/python-type-hinting

[^17]: https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html

[^18]: https://docs.python.org/3/library/typing.html

[^19]: https://realpython.com/lessons/type-hinting/

[^20]: https://codefinity.com/blog/A-Comprehensive-Guide-to-Python-Type-Hints

[^21]: https://experienceleague.adobe.com/en/docs/contributor/contributor-guide/writing-essentials/markdown

[^22]: https://confluence.atlassian.com/bitbucketserver/markdown-syntax-guide-776639995.html

[^23]: https://dev.to/mohammad222pr/structuring-a-fastapi-project-best-practices-53l6

[^24]: https://dev.to/timo_reusch/how-i-structure-big-fastapi-projects-260e

[^25]: https://github.com/encode/httpx/issues/3215

[^26]: https://highon.coffee/blog/httpx-cheat-sheet/

[^27]: http://docs.sqlalchemy.org/en/latest/orm/extensions/mypy.html

[^28]: https://www.ramotion.com/blog/stateless-vs-stateful-web-app/

[^29]: https://pypi.org/project/python-dotenv/

[^30]: https://typing.readthedocs.io/en/latest/source/best_practices.html

[^31]: https://www.markdownguide.org/basic-syntax/

[^32]: https://www.markdownguide.org

[^33]: https://handbook.ebrains.eu/docs/about-handbook/markdown-guide/

[^34]: https://israelmitolu.hashnode.dev/markdown-for-technical-writers-tips-tricks-and-best-practices

[^35]: https://www.codecademy.com/resources/docs/markdown/headings

[^36]: https://www.codecademy.com/resources/docs/markdown/code-blocks

[^37]: https://dev.to/devasservice/fastapi-best-practices-a-condensed-guide-with-examples-3pa5

[^38]: https://www.reddit.com/r/Python/comments/wrt7om/fastapi_best_practices/

[^39]: https://technostacks.com/blog/mastering-fastapi-a-comprehensive-guide-and-best-practices/

[^40]: https://www.reddit.com/r/FastAPI/comments/1aijy4d/beginner_question_for_organizing_fastapi_project/

[^41]: https://docs.railway.com/guides/fastapi

[^42]: https://betterstack.com/community/guides/scaling-python/httpx-explained/

[^43]: https://apidog.com/blog/what-is-httpx/

[^44]: https://github.com/projectdiscovery/httpx

[^45]: https://www.confessionsofadataguy.com/httpx-vs-requests-in-python-performance-and-other-musings/

[^46]: https://opensource.com/article/22/3/python-httpx

[^47]: https://www.twilio.com/en-us/blog/asynchronous-http-requests-in-python-with-httpx-and-asyncio

[^48]: https://joshdimella.com/blog/python-typing-best-practices

[^49]: https://dagster.io/blog/python-type-hinting

[^50]: https://docs.python.org/3/library/typing.html

[^51]: https://github.com/chadrik/doc484

[^52]: https://media.readthedocs.org/pdf/mypy/stable/mypy.pdf

[^53]: https://dev.to/emma_donery/python-dotenv-keep-your-secrets-safe-4ocn

[^54]: https://stackoverflow.com/questions/76783330/for-python-dotenv-is-it-better-practice-to-use-dotenv-load-dotenv-or-dotenv-d

[^55]: https://www.reddit.com/r/Python/comments/yk5bc2/what_is_the_best_flow_for_accessing_environment/

[^56]: https://configu.com/blog/dotenv-managing-environment-variables-in-node-python-php-and-more/

