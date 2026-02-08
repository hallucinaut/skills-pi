---
name: backend-api
description: "Build RESTful APIs, GraphQL endpoints, and database-driven backend services. Use when designing APIs, implementing CRUD operations, handling authentication, or working with backend logic."
---

# Backend API Skill

Build robust, well-documented APIs with clear contracts, proper error handling, and scalable architecture.

## When to Use

Use this skill when the user wants to:
- Design or implement RESTful API endpoints
- Create GraphQL schemas and resolvers
- Handle authentication and authorization
- Work with database models and migrations
- Implement CRUD operations and data validation
- Create API documentation (OpenAPI/Swagger)

## API Design Principles

- **Clear contract**: Define request/response formats explicitly
- **Consistent naming**: Use RESTful conventions (GET, POST, PUT, DELETE) or GraphQL query structure
- **Versioning**: Include API version in path (e.g., /api/v1/...)
- **Error handling**: Return meaningful error codes and messages
- **Documentation**: Include examples and describe fields

## Code Structure

If using a framework (Express, NestJS, Fastify, etc.):

```
backend-api/
├── controllers/      # Request handlers
├── services/         # Business logic
├── models/           # Data models
├── routes/           # Route definitions
└── utils/            # Helper functions
```

## Deliverables

- Complete API implementation
- Request/response examples
- Error handling strategy
- Validation schema
- API documentation (if applicable)

## Quality Checklist

- Clear endpoint definitions
- Proper HTTP status codes
- Input validation
- Error responses are descriptive
- Code is documented
