# SKILL: Python Backend 🐍🔧

**Propósito:** Reunir las mejores prácticas para desarrollar servicios backend en Python (API, microservicios, MCPs), con foco en producción, mantenibilidad y seguridad.

---

## ✅ Principios generales

- Preferir **simplicidad** y claridad sobre cleverness.
- Programar con **tipado estático** donde sea práctico (mypy) para mejorar mantenibilidad.
- APIs **stateless** siempre que sea posible; externalizar estado en DB/caches.
- Seguir el principio de responsabilidad única (SRP) y diseño modular.

---

## 🧭 Arquitectura y diseño

- Diseñar **contratos** (OpenAPI, JSON Schema) desde el inicio y versionarlos.
- Separar **transporte** (HTTP, gRPC, WebSocket) de la lógica de dominio y de almacenamiento.
- Usar patrones como **ports & adapters** (hexagonal) para facilitar testing y reemplazo de infra.
- Documentar endpoints, errores y expectativas de idempotencia.

---

## 🛠 Frameworks y librerías recomendadas

- **FastAPI** para APIs modernas y asíncronas (OpenAPI automático, Pydantic para validación). ⚡
- **Pydantic** / **pydantic v2** para modelos, validación y parsing seguro.
- **SQLAlchemy** (Core + ORM) o **Tortoise** (async) según necesidad.
- **Alembic** para migraciones de DB relacional.
- **HTTPX** para clientes HTTP asincrónicos en tests e integración.

---

## ⚙️ Configuración y despliegue

- Gestionar configuración con variables de entorno y librerías tipo `pydantic-settings` o `python-decouple`.
- Usar containers ligeros (Docker) y construir imágenes reproducibles.
- Variables sensibles en secreto manager (Vault, AWS Secrets Manager) o en GitHub Secrets para CI.
- Tener `health` y `ready` endpoints y soporte para _graceful shutdown_.

---

## 🔒 Seguridad

- Validar entradas estrictamente y sanear datos al persistir o renderizar.
- Autenticación y autorización firmes (JWT con verificación de scopes/claims, OAuth2 si aplica).
- CSRF, CORS y cabeceras seguro (HSTS, X-Frame-Options) cuando aplique.
- Rate limiting y protección contra DoS en borde (APIs Gateway, Cloudflare, NGINX).
- Escaneos automáticos de dependencias (dependabot, Snyk) y revisión de vulnerabilidades.

---

## 🧪 Testing y calidad

- Tests unitarios con `pytest` y `pytest-asyncio` para rutas y lógica asíncrona.
- Tests de integración que corran contra DB/Redis reproducible (usar bases en Docker o fixtures in-memory).
- Contratos: contract tests para consumidores/proveedores de APIs.
- Coverage razonable pero enfocado en comportamiento crítico.

---

## 🧹 Linting, formateo y tipos

- Formatear con **black**, ordenar imports con **isort**, lint con **ruff** o **flake8**.
- Tipado con **mypy** + `pyproject.toml` para configuración estricta.
- Revisiones automáticas en PRs (pre-commit hooks).

---

## 📈 Observabilidad y métricas

- Logging estructurado (JSON) y correlación con trazas (trace ids).
- Exponer métricas Prometheus y logs injestionables (ELK, Grafana Loki).
- Tracing distribuido (OpenTelemetry) para requests que atraviesan servicios.
- Alertas por errores, latencia y falta de recursos.

---

## 🚀 Rendimiento y escalabilidad

- Prefiere código asíncrono (async/await) para IO-bound; evitar bloqueo en Uvicorn workers.
- Pooling de conexiones para DB y caches; monitorear conexiones usadas.
- Caché (Redis) para datos de lectura frecuente y cache de respuestas donde aplique.
- Compensar con backoff exponencial y circuit breakers en llamadas externas.

---

## 🧾 Operaciones y mantenimiento

- CI que ejecute lint, tests y checks de tipo antes de merges.
- Despliegues con rollbacks automáticos y canary/blue-green cuando sea posible.
- Documentación operativa (runbooks) para incidentes comunes.
- Backup y políticas de retención para datos críticos.

---

## 🗂 MCPs (Model Context Protocol) — recomendaciones específicas

- Definir esquemas Pydantic para los contextos y versionarlos.
- Endpoints idempotentes para registrar y actualizar contextos.
- Soporte para pub/sub (Redis/Kafka) para notificar consumidores; separar la cola de la lógica de contexto.
- Autorización por contexto y scopes; auditoría de cambios.

---

## ✅ Checklist rápida (PRs)

- [ ] Tests agregados o actualizados
- [ ] Tipos y mypy pasan
- [ ] Linters y formateo OK
- [ ] OpenAPI/Docs actualizados si cambia contrato
- [ ] Cambios de infra documentados (migrations, variables env)

---

## 📚 Recursos y links útiles

- FastAPI: https://fastapi.tiangolo.com
- Pydantic: https://pydantic-docs.helpmanual.io
- SQLAlchemy: https://www.sqlalchemy.org
- OpenTelemetry: https://opentelemetry.io

---

¿Deseas que añada ejemplos concretos (plantilla FastAPI con JWT, Redis, y CI) dentro de este repo para que coincida con la `askill` que creamos antes? 🔧
