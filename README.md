# Tatsat

A high-performance web framework with elegant syntax and powerful validation, built on Starlette and using Satya for data validation. Tatsat provides the developer-friendly interface of FastAPI with the validation performance of Satya.

## Features

- **Modern**: Built on top of Starlette, a lightweight ASGI framework
- **High Performance**: Utilizes Satya's validation engine, which is significantly faster than Pydantic
- **Developer-friendly**: Intuitive, FastAPI-like syntax for rapid development
- **Type-safe**: Leverages Python type hints with Satya models for validation
- **Automatic API documentation**: Built-in Swagger UI and ReDoc integration
- **Dependency Injection**: Powerful dependency injection system 
- **Path Operations**: Intuitive route decorators for all HTTP methods
- **Middleware Support**: Built-in middleware system for cross-cutting concerns
- **Exception Handling**: Comprehensive exception handling mechanisms
- **APIRouter**: Support for route grouping and organization

## Benchmarks

Tatsat significantly outperforms FastAPI in validation and serialization operations. See the [benchmarks](/benchmarks/README.md) for detailed performance comparisons with FastAPI, Starlette, and Flask.

## Installation

```bash
# Install from PyPI
pip install tatsat

# Or install directly from the repository
pip install -e .

# Make sure you have satya installed
pip install satya
```

## Quick Start

Here's a minimal example to get you started:

```python
from tatsat import Tatsat
from satya import Model, Field
from typing import List, Optional

app = Tatsat(title="Tatsat Demo")

# Define your data models with Satya
class Item(Model):
    name: str = Field()
    description: Optional[str] = Field(required=False)
    price: float = Field(gt=0)
    tax: Optional[float] = Field(required=False)
    tags: List[str] = Field(default=[])

# Create API endpoints with typed parameters
@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None):
    return {"item_id": item_id, "q": q}

@app.post("/items/", response_model=Item)
def create_item(item: Item):
    return item

# Run the application with Uvicorn
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Core Concepts

### Application

The `Tatsat` class is the main entry point for creating web applications:

```python
from tatsat import Tatsat

app = Tatsat(
    title="Tatsat Example API",
    description="A sample API showing Tatsat features",
    version="0.1.0",
    debug=False
)
```

### Path Operations

Tatsat provides decorators for all standard HTTP methods:

```python
@app.get("/")
@app.post("/items/")
@app.put("/items/{item_id}")
@app.delete("/items/{item_id}")
@app.patch("/items/{item_id}")
@app.options("/items/")
@app.head("/items/")
```

### Path Parameters

Path parameters are part of the URL path and are used to identify a specific resource:

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

### Query Parameters

Query parameters are optional parameters appended to the URL:

```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

### Request Body

Request bodies are parsed and validated using Satya models:

```python
@app.post("/items/")
def create_item(item: Item):
    return item
```

### Dependency Injection

Tatsat includes a powerful dependency injection system:

```python
def get_db():
    db = Database()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
def read_items(db = Depends(get_db)):
    return db.get_items()
```

### Response Models

Specify response models for automatic serialization and documentation:

```python
@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int):
    return get_item_from_db(item_id)
```

## Advanced Features

### API Routers

Organize your routes using the `APIRouter`:

```python
from tatsat import APIRouter

router = APIRouter(prefix="/api/v1")

@router.get("/items/")
def read_items():
    return {"items": []}

app.include_router(router)
```

### Middleware

Add middleware for cross-cutting concerns:

```python
@app.middleware("http")
async def add_process_time_header(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### Exception Handlers

Custom exception handlers:

```python
@app.exception_handler(404)
async def not_found_exception_handler(request, exc):
    return JSONResponse(
        status_code=404,
        content={"detail": "The requested resource was not found"},
    )
```

### Event Handlers

Handle startup and shutdown events:

```python
@app.on_event("startup")
async def startup_event():
    print("Application is starting up")

@app.on_event("shutdown")
async def shutdown_event():
    print("Application is shutting down")
```

## Documentation

Tatsat automatically generates interactive documentation:

- Swagger UI: Available at `/docs`
- ReDoc: Available at `/redoc`

## Example Applications

Tatsat comes with several example applications in the `examples/` directory:

- **basic_app.py**: Simple CRUD operations with dependency injection
- **advanced_app.py**: Advanced features including middleware, nested models, and authentication
- **comprehensive_benchmark.py**: Performance comparison with FastAPI, Starlette, and Flask

## Route Documentation

For a comprehensive guide to all available route patterns and their usage, see the [ROUTES.md](ROUTES.md) file.

## Dependencies

Tatsat depends on the following packages:

- starlette >= 0.28.0
- uvicorn >= 0.23.0
- satya (for data validation)

## License

MIT License
