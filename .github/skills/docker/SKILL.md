# Docker Skill

## Description

Skill to provide Docker commands and recommended practices inside the Copilot agent.

## Objective

Help the user to:

- Write and debug Dockerfiles
- Use docker build / run / compose
- Optimize images and cache
- Troubleshoot common issues

## Docker Compose: specific scope

- Create `docker-compose.yml` for multi-service projects (app, DB, cache, networks)
- Securely configure restarts, volumes, and env vars (`.env`, `env_file`)
- Optimize with `build` + `cache_from`, `depends_on`, and `profiles`
- Convert `docker run` to Compose and vice versa
- Compose diagnostics: `docker compose config`, `docker compose ps`, `logs`, and `events`

## Instructions for the assistant

1. When the user asks for Docker help, respond with:
   - clear, simple steps
   - reproducible commands
   - examples of `Dockerfile`, `docker-compose.yml`, `docker build`, and `docker run`
2. Avoid explaining too much basic concepts unless requested.
3. Always offer a copy-ready command block.
4. Remind the user that images should follow least-privilege principles:
   - `USER` non-root whenever possible.
   - Do not run processes as root inside the container.
   - Avoid privileged capabilities (`--privileged`) unless necessary.
   - Use `chmod`/`chown` in build to ensure proper file access.

## Interaction examples

- User: "I need a Dockerfile for a Node.js app that uses npm ci and caches in one layer"
- Desired response:
  - Suggested Dockerfile
  - Concise caching explanation
  - Command `docker build -t app:latest .`

- User: "I have a web app with Nginx + Python API + Postgres, give me a docker-compose.yml"
- Desired response:
  - 3-service `docker-compose.yml`
  - Use of `depends_on`, `volumes`, and `restart: unless-stopped`
  - Run command `docker compose up -d` and validation `docker compose ps`

## Preferred output format

- Short title (e.g., `docker build`)
- Markdown syntax highlighting
- Command shortcuts
- Validation suggestions (e.g., `docker inspect`, `docker logs`)
