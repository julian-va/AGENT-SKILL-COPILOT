# SKILL: Python Backend 🐍🔧

**Purpose:** Gather best practices for developing backend services in Python (APIs, microservices, MCPs), with a focus on production readiness, maintainability, and security.

---

## ✅ Core principles

- Prefer **simplicity** and clarity over cleverness.
- Use **static typing** where practical (mypy) to improve maintainability.
- Design **stateless** APIs whenever possible; externalize state to databases/caches.
- Apply the Single Responsibility Principle (SRP) and keep code modular.

---

## 🧭 Architecture and design

- Define **contracts** (OpenAPI, JSON Schema) from the start and version them.
- Separate **transport** concerns (HTTP, gRPC, WebSocket) from domain logic and storage.
- Use patterns like **ports & adapters** (hexagonal) to make testing and infra replacement easier.
- Document endpoints, error models and idempotency expectations.

---

## 🛠 Recommended frameworks and libraries

- **FastAPI** for modern async APIs (automatic OpenAPI, Pydantic for validation). ⚡
- **Pydantic** / **Pydantic v2** for models, validation and safe parsing.
- **SQLAlchemy** (Core + ORM) or **Tortoise ORM** (async) depending on needs.
- **Alembic** for relational DB migrations.
- **HTTPX** for async HTTP clients in tests and integrations.

---

## ⚙️ Configuration and deployment

- Manage configuration with environment variables and libraries like `pydantic-settings` or `python-decouple`.
- Use lightweight containers (Docker) and build reproducible images.
- Store sensitive variables in a secrets manager (Vault, AWS Secrets Manager) or GitHub Secrets for CI.
- Provide `health` and `ready` endpoints and support for graceful shutdown.

---

## 🔒 Security

- Validate inputs strictly and sanitize data at persistence or rendering boundaries.
- Strong authentication and authorization (JWT with scopes/claims verification, OAuth2 when applicable).
- Address CSRF, CORS and secure headers (HSTS, X-Frame-Options) where relevant.
- Apply rate limiting and edge protections against DoS (API Gateway, Cloudflare, NGINX).
- Run automated dependency scans (Dependabot, Snyk) and vulnerability reviews.

---

## 🧪 Testing and quality

- Unit tests with `pytest` and `pytest-asyncio` for routes and async logic.
- Integration tests that run against reproducible DB/Redis instances (use Docker or in-memory fixtures).
- Contract tests for API consumers/providers.
- Reasonable coverage focused on critical behavior.

---

## 🧹 Linting, formatting and typing

- Format with **black**, sort imports with **isort**, and lint with **ruff** or **flake8**.
- Add typing checks with **mypy** and configure via `pyproject.toml`.
- Run pre-commit hooks and automated checks on PRs.

---

## 📈 Observability and metrics

- Structured logging (JSON) and trace correlation (trace ids).
- Expose Prometheus metrics and shipable logs (ELK, Grafana Loki).
- Distributed tracing (OpenTelemetry) for requests crossing service boundaries.
- Set up alerts for errors, latency and resource issues.

---

## 🚀 Performance and scalability

- Prefer async code (`async/await`) for IO-bound workloads; avoid blocking Uvicorn workers.
- Use connection pooling for DBs and caches and monitor connection usage.
- Cache read-heavy data (Redis) and consider response caching where appropriate.
- Employ exponential backoff and circuit breakers for external calls.

---

## 🧾 Operations and maintenance

- CI should run linters, tests and type checks before merges.
- Support rollbacks and canary/blue-green deployments when feasible.
- Maintain operational runbooks for common incidents.
- Implement backups and retention policies for critical data.

---

## 🗂 MCPs (Model Context Protocol) — specific recommendations

- Define Pydantic schemas for contexts and version them.
- Provide idempotent endpoints to register and update contexts.
- Support pub/sub (Redis/Kafka) to notify consumers; separate the queue from context logic.
- Enforce context-level authorization and audit changes.

---

## ✅ Quick PR checklist

- [ ] Tests added or updated
- [ ] Types and mypy pass
- [ ] Linters and formatting OK
- [ ] OpenAPI/Docs updated if the contract changed
- [ ] Infra changes documented (migrations, env vars)

---

## 📚 Useful resources

- FastAPI: https://fastapi.tiangolo.com
- Pydantic: https://pydantic-docs.helpmanual.io
- SQLAlchemy: https://www.sqlalchemy.org
- OpenTelemetry: https://opentelemetry.io

---

Would you like me to add concrete examples (a FastAPI template with JWT, Redis, and CI) inside this repo to match the askill we created earlier? 🔧
