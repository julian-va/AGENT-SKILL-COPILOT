---
name: docker
version: 0.1.0
author: Your Name
contact: your.email@example.com
tags: [docker, compose, containers, security, best-practices]
---

# Docker Skill

## Description

Skill to provide Docker and Docker Compose guidance with production-ready best practices.

## Objective

Help the user to:

- Design secure, maintainable `Dockerfile`s
- Build optimized images and use build cache correctly
- Create Compose manifests for multi-container applications
- Debug container startup, networking, volumes, and env configuration
- Apply Docker security and operational best practices

## Scope

This skill covers:

- Dockerfile authoring and optimization
- Multi-stage builds and minimal runtime images
- Build cache, layering, and image size reduction
- `docker build`, `docker run`, `docker compose` workflows
- Compose patterns for services, networks, volumes, healthchecks, and profiles
- Secure configuration via non-root users, env vars, and file permissions

## Dockerfile best practices

- Start from a small, official base image and pin major versions.
- Use multi-stage builds to separate build dependencies from runtime artifacts.
- Leverage layer caching by copying dependency manifests first, installing deps, then adding source code.
- Add a `.dockerignore` to exclude node_modules, logs, git metadata, build artifacts, and local IDE files.
- Use `COPY --chown=<user>:<group>` where supported to avoid extra `chown` layers.
- Set a non-root `USER` in the runtime stage whenever possible.
- Prefer runtime images without package managers or compilers.
- Use explicit healthchecks for long-lived services.
- Reduce image size by removing package manager caches and temporary files in the same layer as install commands.
- Add metadata labels (`org.opencontainers.image.*`) when useful.

### Example pattern

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --production=false
COPY . ./
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
RUN npm prune --production
USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

## Docker Compose best practices

- Use version `3.9` or later and prefer `docker compose` CLI.
- Keep configuration declarative: services, networks, volumes, and configs separated clearly.
- Use `.env` and `env_file` for environment variables, but do not store secrets in source control.
- Define named volumes for persisted data and explicit host paths only when required.
- Use `restart: unless-stopped` or `restart: on-failure` rather than `always` for most dev/prod cases.
- Use `depends_on` for startup ordering; add service-level healthchecks to enforce readiness.
- Use `profiles` to enable optional services in local vs production setups.
- Avoid `container_name` unless needed; let Compose manage names.
- Prefer `networks: default` and explicitly define custom networks when services must be isolated.
- Keep production and development Compose files separate, using overrides like `docker-compose.override.yml` or `docker compose -f ...`.

### Compose example

```yaml
version: "3.9"
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
    env_file:
      - .env
    depends_on:
      api:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8080/healthz || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  api:
    image: myapp/api:latest
    networks:
      - backend

  postgres:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:

networks:
  backend:
    driver: bridge
```

## Security and operations

- Do not run containers as root unless absolutely required.
- Avoid `--privileged` and unnecessary Linux capabilities.
- Use `--read-only` for services that do not need writable filesystem state.
- Mount only the directories that are needed; avoid sharing the entire host file system.
- Use `docker compose config` to validate Compose files before starting.
- Prefer `docker compose up -d --remove-orphans` for predictable local startup.
- Use `docker compose logs --tail 100 -f` to inspect startup failures.
- Scan images regularly with tools like `docker scan`, `trivy`, or `grype`.
- Avoid storing secrets in `.env` or source control; use Docker Secrets, Vault, or a secrets manager.
- Use seccomp/default profiles and AppArmor when available for runtime isolation.
- Keep base images updated and rebuild images after patching dependencies.
- Prefer explicit resource limits with Compose: `deploy.resources.limits` or CLI flags in Docker run.
- Use immutable tags for production images and avoid `latest` in deployment manifests.

## Security checklist for Docker responses

- Non-root runtime user
- Minimal runtime image
- Secrets out of source control
- Healthchecks and restart policy
- Image scanning and base image patching
- Compose config validation

## Common command patterns

- Build image:
  `docker build -t myapp:latest .`
- Run container:
  `docker run --rm -p 8080:8080 myapp:latest`
- Compose up:
  `docker compose up -d`
- Compose down:
  `docker compose down --volumes`
- Inspect runtime:
  `docker compose ps`
  `docker compose logs --tail 50`
  `docker inspect <container>`

## Instructions for the assistant

1. When the user asks for Docker or Compose help, answer with:
   - concrete examples
   - copy-ready `Dockerfile` / `docker-compose.yml`
   - exact CLI commands
   - validation and troubleshooting commands
2. Prefer **best practices first**: security, caching, minimal images, and explicit service health.
3. Do not over-explain basic concepts unless the user asks for them.
4. When reviewing user-provided Dockerfiles or Compose files, point out:
   - cache inefficiencies
   - security issues (`USER`, capabilities, secret handling)
   - portability issues (`container_name`, hardcoded paths)
   - compose ordering and healthcheck gaps
5. Use plain Spanish for the main response, but keep code and commands in English.

## Interaction examples

- User: "Necesito un Dockerfile para una app Node.js con npm ci y cache de dependencias"
  - Respuesta ideal: Dockerfile multi-stage, `npm ci`, `.dockerignore`, `USER node`, `docker build`.
- User: "Dame un docker-compose.yml para una app con Nginx, API Python y Postgres"
  - Respuesta ideal: 3 servicios, `depends_on` + healthchecks, volúmenes, `restart: unless-stopped`, `docker compose up -d`.
- User: "¿Cómo traduzco este `docker run` a Compose?"
  - Respuesta ideal: Compose service definition con `ports`, `volumes`, `environment` y `command`.

## Preferred output format

- short heading
- code blocks with syntax highlighting
- commands in separate blocks
- brief verification steps
