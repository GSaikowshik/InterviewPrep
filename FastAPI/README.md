# FastAPI Master Reference & Production Engineering Guide

---

## 1. Architecture & Core Philosophy

FastAPI is a modern, high-performance web framework for building APIs with Python 3.8+ based on standard Python type hints. It is built on top of **Starlette** (for high-performance async web routing) and **Pydantic** (for data validation and schema enforcement).

```mermaid
sequenceDiagram
    autonumber
    actor Client as Web Browser / Mobile Client
    participant Uvicorn as Uvicorn (ASGI Web Server)
    participant Middleware as Middleware Stack (CORS, Auth, Timing)
    participant Router as APIRouter & Dependencies (Depends)
    participant Pydantic as Pydantic v2 (Validation Engine)
    participant Endpoint as Path Operation Function (async def)
    participant DB as Async Database (PostgreSQL / SQLAlchemy 2.0)

    Client->>Uvicorn: HTTP Request (Headers + JSON Payload)
    Uvicorn->>Middleware: ASGI Scope, Receive, Send
    Middleware->>Router: Route Matching
    Router->>Router: Execute Dependencies (DB Session, Auth Check)
    Router->>Pydantic: Validate & Deserialize Request Body
    alt Validation Error
        Pydantic-->>Client: 422 Unprocessable Entity (JSON Errors)
    else Validation Succeeded
        Pydantic->>Endpoint: Inject Validated Python Objects
        Endpoint->>DB: Non-blocking I/O (await session.execute)
        DB-->>Endpoint: SQL Result Set
        Endpoint->>Pydantic: Filter & Serialize via response_model
        Pydantic-->>Middleware: Serialized JSON Response
        Middleware-->>Client: HTTP Response (Status 200/201 + Body)
    end
```

### WSGI vs. ASGI Architecture

| Dimension | WSGI (Flask, Django Classic) | ASGI (FastAPI, Starlette) |
|---|---|---|
| **Paradigm** | Synchronous, blocking request-response | Asynchronous, event-loop driven (`async`/`await`) |
| **Concurrency** | One thread per concurrent request | Single thread handles thousands of idle I/O sockets |
| **WebSockets & SSE**| Requires separate daemons / gevent / Celery | Natively supported within the same process |
| **Standard Servers**| Gunicorn (sync workers), uWSGI | Uvicorn, Hypercorn, Daphne |
| **Throughput** | Moderate; bottlenecks quickly on slow DB/API I/O | Extremely high; on par with NodeJS and Go |

---

## 2. Route Definition & Parameter Extraction

FastAPI extracts parameters based on type annotations and default helper instances:

```python
from fastapi import FastAPI, Path, Query, Header, Cookie, Body, status
from pydantic import BaseModel, Field, EmailStr
from typing import Optional, List
from enum import Enum

app = FastAPI(title="FastAPI Master Reference", version="1.0.0")

class UserRole(str, Enum):
    ADMIN = "admin"
    EDITOR = "editor"
    VIEWER = "viewer"

class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50, examples=["kowshik"])
    email: EmailStr
    age: Optional[int] = Field(None, ge=18, le=120)

@app.post(
    "/users/{user_id}",
    status_code=status.HTTP_201_CREATED,
    tags=["Users"],
    summary="Create or update user profile"
)
async def create_user(
    # 1. Path Parameter: Extracted from URL path with constraint validations
    user_id: int = Path(..., ge=1, description="Primary unique user identifier"),
    
    # 2. Query Parameters: Extracted from URL query string ?role=admin&notify=true
    role: UserRole = Query(UserRole.VIEWER, description="Assigned role"),
    notify: bool = Query(True, description="Send onboarding alert"),
    
    # 3. Request Body: Auto-deserialized and validated via Pydantic
    payload: UserCreate = Body(..., description="User creation payload"),
    
    # 4. Header & Cookie Parameters
    user_agent: Optional[str] = Header(None),
    session_id: Optional[str] = Cookie(None)
):
    return {
        "user_id": user_id,
        "role": role,
        "notify": notify,
        "payload": payload.model_dump(),
        "user_agent": user_agent,
        "session_id": session_id
    }
```

---

## 3. Pydantic v2 Validation & Custom Validators

### Pydantic Validation & Serialization Pipeline

```mermaid
flowchart LR
    RawJSON["Raw Client JSON Payload"] --> Parse["Pydantic Type Coercion & Schema Check"]
    Parse --> Check{"Validation Passes?"}
    Check -- No --> Err["Generate 422 Unprocessable Entity with Field Error Details"]
    Check -- Yes --> Val["Execute @field_validator & @model_validator"]
    Val --> Success["Inject Cleaned Python Object into async def Endpoint"]
```


Pydantic v2 is rewritten in Rust, providing 5x-20x faster serialization and validation compared to v1.

```python
from pydantic import BaseModel, Field, field_validator, model_validator
from datetime import datetime

class ProductSchema(BaseModel):
    name: str = Field(..., min_length=2)
    price: float = Field(..., gt=0.0, description="Price must be strictly positive")
    discount_price: Optional[float] = None
    created_at: datetime = Field(default_factory=datetime.utcnow)

    # Field-level validator
    @field_validator("name")
    @classmethod
    def name_must_be_capitalized(cls, v: str) -> str:
        if not v[0].isupper():
            raise ValueError("Product name must start with a capital letter.")
        return v

    # Model-level (cross-field) validator
    @model_validator(mode="after")
    def check_discount_less_than_price(self):
        if self.discount_price is not None and self.discount_price >= self.price:
            raise ValueError("Discount price must be strictly less than normal price.")
        return self
```

---

## 4. Dependency Injection (`Depends`) & Database Sessions

FastAPI’s Dependency Injection system manages database connection lifecycles, authentication, and cross-cutting concerns cleanly without global state.

```mermaid
graph TD
    Request["Incoming HTTP Request"] --> D1["Depends(get_db) -> Yields SQLAlchemy AsyncSession"]
    Request --> D2["Depends(get_current_user) -> Validates JWT & returns User object"]
    D1 --> Endpoint["Path Operation Function (async def)"]
    D2 --> Endpoint
    Endpoint --> Commit["Database Commit / Rollback"]
    Commit --> Close["Session Closed via finally block"]
```

```python
from fastapi import Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from typing import AsyncGenerator

DATABASE_URL = "postgresql+asyncpg://user:password@localhost:5432/appdb"

engine = create_async_engine(DATABASE_URL, pool_size=20, max_overflow=10, echo=False)
AsyncSessionLocal = async_sessionmaker(bind=engine, class_=AsyncSession, expire_on_commit=False)

# Yield dependency manages transaction lifecycle and guaranteed cleanup
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

---

## 5. Security: OAuth2, Password Hashing & JWT Authentication

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from passlib.context import CryptContext
from datetime import datetime, timedelta
import jwt

SECRET_KEY = "SUPER_SECRET_PRODUCTION_KEY_NEVER_HARDCODE"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/token")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(token: str = Depends(oauth2_scheme)) -> dict:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        return {"username": username, "role": payload.get("role", "user")}
    except jwt.PyJWTError:
        raise credentials_exception
```

---

## 6. Middleware: CORS, Execution Timing & Custom Headers

```python
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
import time

# 1. CORS Configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://myapp.vercel.app", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],
    allow_headers=["*"],
)

# 2. Custom Timing Middleware
class ProcessTimeMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.perf_counter()
        response = await call_next(request)
        process_time = time.perf_counter() - start_time
        response.headers["X-Process-Time"] = f"{process_time:.4f}s"
        return response

app.add_middleware(ProcessTimeMiddleware)
```

---

## 7. Asynchronous Streaming & Background Tasks (AI/LLM Integration)

```python
from fastapi import BackgroundTasks
from fastapi.responses import StreamingResponse
import asyncio

# Background Task (Fire-and-forget email/logging)
def send_email_notification(email: str, message: str):
    time.sleep(2)  # Simulates background task
    print(f"Dispatched email to {email}")

@app.post("/order")
async def place_order(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email_notification, email, "Your order is confirmed!")
    return {"message": "Order accepted and processing."}

# Real-time Server-Sent Events / Token Streaming for LLMs
async def fake_llm_stream():
    tokens = ["Hello", " ", "this", " ", "is", " ", "FastAPI", " ", "streaming!"]
    for token in tokens:
        yield token
        await asyncio.sleep(0.1)

@app.get("/stream-llm")
async def stream_tokens():
    return StreamingResponse(fake_llm_stream(), media_type="text/event-stream")
```

---

## 8. Master FastAPI Production Dockerfile

```dockerfile
FROM python:3.12-slim-bookworm AS builder

WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.12-slim-bookworm AS runner
WORKDIR /app

COPY --from=builder /root/.local /root/.local
COPY . /app

ENV PATH=/root/.local/bin:$PATH     PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

---

## 9. Top 20 High-Frequency FastAPI Interview Questions

1. **What is the difference between `def` and `async def` in FastAPI?**
   * *Answer:* When an endpoint is declared with `async def`, FastAPI runs it directly on the main event loop thread; it must never execute blocking synchronous calls (like `time.sleep` or synchronous DB drivers) or the entire server halts. When declared with standard `def`, FastAPI offloads the request to an external AnyIO worker thread pool, preventing event loop starvation.
2. **How does FastAPI generate automatic OpenAPI and Swagger UI documentation?**
   * *Answer:* FastAPI inspects Python type hints and Pydantic schemas defined on path operations, serializing them into an OpenAPI 3.0 specification JSON schema at `/openapi.json`. It embeds Swagger UI at `/docs` and ReDoc at `/redoc`.
3. **What is the N+1 query problem and how do you prevent it in FastAPI with SQLAlchemy?**
   * *Answer:* The N+1 problem occurs when fetching a parent record requires 1 query, and accessing each related child triggers N additional queries. In SQLAlchemy 2.0 async, prevent this using eager loading via `.options(selectinload(Parent.children))` or `joinedload()`.
4. **Why should you separate Request schemas from Response schemas?**
   * *Answer:* Request schemas accept input data (e.g., plaintext password), while Response schemas exclude sensitive fields (e.g., hashed password, internal DB flags) and may include computed metadata (e.g., `id`, `created_at`).
