---
name: docker
description: "Containerize applications with Docker and write Dockerfiles. Use when creating container images, setting up Docker Compose, or managing container orchestration."
---

# Docker Skill

Build and manage Docker containers, images, and Compose files for consistent deployment.

## When to Use

Use this skill when the user wants to:
- Write Dockerfiles for containerization
- Create docker-compose.yml files
- Optimize Docker image size
- Multi-stage builds
- Docker networking and volumes
- Docker secrets and configs
- Container orchestration basics

## Dockerfile Best Practices

- **Use official base images**: Alpinized, distroless when possible
- **Multi-stage builds**: Separate build and runtime stages
- **Leverage caching**: Order COPY/ADD instructions carefully
- **Minimize layers**: Combine commands where possible
- **Clean up**: Remove unnecessary files and packages
- **Non-root user**: Run as non-root user in container

## Docker Compose

Define services, networks, and volumes:

```yaml
version: '3.8'
services:
  app:
    build: .
    environment:
      - DATABASE_URL=${DATABASE_URL}
    volumes:
      - ./data:/app/data
```

## Multi-Stage Build Example

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

## Deliverables

- Optimized Dockerfile
- docker-compose.yml (if needed)
- .dockerignore file
- Container runtime documentation

## Quality Checklist

- Image size is minimized
- Security best practices followed
- Ports and volumes are properly mapped
- Environment variables are used
- Network isolation is configured
- No secrets baked into image
