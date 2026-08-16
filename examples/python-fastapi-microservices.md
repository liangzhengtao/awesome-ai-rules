# Example: Python FastAPI + Microservices

This example shows how to combine rules for a Python microservices project.

## Setup

```bash
# Copy framework rules
cat rules/frameworks/python-fastapi.md > CLAUDE.md

# Append scenario rules
cat rules/scenarios/microservices.md >> CLAUDE.md
cat rules/scenarios/api-design.md >> CLAUDE.md
```

## Combined Rules Preview

```markdown
# Python FastAPI + Microservices AI Rules

## Core Principles
- Use Python 3.10+ with type hints
- Follow async-first approach
- Use Pydantic for data validation
- Implement proper error handling
- Follow RESTful API design

## Project Structure
```
services/
├── user-service/
│   ├── cmd/
│   ├── internal/
│   │   ├── handler/
│   │   ├── service/
│   │   ├── repository/
│   │   └── model/
│   ├── api/
│   └── Dockerfile
└── api-gateway/
```

## API Design
- Use URL versioning: /api/v1/users
- Implement proper pagination
- Use consistent error response format
- Add rate limiting headers
- Support field filtering

## Service Communication
- Use gRPC for synchronous calls
- Use message queues for async events
- Implement circuit breakers
- Add distributed tracing

## Testing
- Unit tests for business logic
- Integration tests for API endpoints
- Use pytest + httpx
- Mock external dependencies
```

## Why This Works

The combined rules ensure your AI assistant:
1. Generates **FastAPI** code with proper async patterns
2. Follows **microservices** architecture principles
3. Implements **RESTful API** best practices
4. Handles **error cases** properly
5. Writes **testable** code
