---
name: docker-deploy
description: Write Dockerfiles, configure Docker Compose, and manage containerized deployments. Use when building Docker images, optimizing Dockerfiles, setting up multi-stage builds, configuring health checks, or deploying with containers. Trigger on "Docker", "Dockerfile", "docker-compose", "container", "image optimization", "multi-stage build", "health check", or "container deployment".
---

# Docker & Container Deployment

Build small, secure, reproducible containers.

## Dockerfile Best Practices

### Multi-Stage Build (Always)
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage — only what's needed
FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

### Layer Caching Optimization
- `COPY package*.json` → `RUN npm ci` → `COPY . .` (dependency layer cached separately)
- Order layers by frequency of change (dependencies rarely change, source changes often)
- Use `.dockerignore` to exclude node_modules, .git, tests

### .dockerignore
```
node_modules
.git
.env
*.md
tests
coverage
.dockerignore
Dockerfile
```

## Image Size Rules

- Start from `alpine` or `distroless` (not full OS images)
- Multi-stage builds to exclude build tools
- No package managers in final image unless needed
- One process per container
- Target: < 150MB for Node.js, < 20MB for Go/Rust

## Security

- Run as non-root: `USER node` (or `USER 1000`)
- Scan images: `docker scan myapp` or Trivy
- No secrets baked into images — use runtime env vars or secrets manager
- Pin base image versions (don't use `latest`)
- Read-only filesystem where possible: `--read-only --tmpfs /tmp`

## Docker Compose

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

## Health Checks

Every container needs a health check endpoint:
- Returns 200 when healthy, 503 when not
- Checks critical dependencies (DB connection, cache)
- Separate liveness (`/health/live`) from readiness (`/health/ready`)

## Image Tagging

- Never use `latest` in production
- Use Git SHA: `myapp:a1b2c3d`
- Use semantic versions for releases: `myapp:v1.2.3`
- Tag both: `myapp:v1.2.3` + `myapp:a1b2c3d`

## Common Commands

```bash
# Build with cache busting
docker build -t myapp:$(git rev-parse --short HEAD) .

# Multi-arch build
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Prune everything (disk cleanup)
docker system prune -af --volumes
```

## Checklist

- [ ] Multi-stage build (build tools not in production image)
- [ ] Non-root user in final image
- [ ] Base image pinned to specific version
- [ ] .dockerignore excludes unnecessary files
- [ ] Health check endpoint defined
- [ ] Image scanned for vulnerabilities
- [ ] Image size < 150MB (Node.js) or < 50MB (compiled)
- [ ] No secrets in image layers
