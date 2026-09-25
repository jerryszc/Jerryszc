# Jhosbert Osorio

**Backend Engineer** — Python • FastAPI • PostgreSQL • AWS • Distributed Systems

---

## About Me

I build backend systems that are reliable, observable, and simple to operate. I care about correct abstractions, operational ownership, and code that survives production.

Currently focused on: **distributed systems correctness**, **observability-driven development**, and **developer experience**.

---

## Selected Work

### E-Commerce Inventory Automator
[`jerryszc/ecommerce-inventory-automator`](https://github.com/jerryszc/ecommerce-inventory-automator) — Multi-channel inventory synchronization (Amazon/Shopify) with async processing, full observability, and zero-downtime deployments.

**Problems solved:**

| Business Problem | Technical Solution | Outcome |
|------------------|-------------------|---------|
| **Stock drift across channels** (Amazon says 10, Shopify says 8) | Last-write-wins sync with immutable ConflictLog; per-variant thresholds | Single source of truth; audit trail for every discrepancy |
| **Manual CSV imports from suppliers** (hours, error-prone, blocks team) | S3 → SQS → Lambda worker; alias-tolerant parser; auto-creates catalog; deduplication | API responds in ~50ms; heavy lifting offline; error rows reported per row |
| **No visibility into low stock until sale fails** | Per-variant thresholds; `/alerts/low-stock` with channel filter | Proactive replenishment; zero stock-outs from visibility gaps |
| **Unauthorized changes / no audit trail** | JWT (HS256) + access/refresh rotation; RBAC (admin/operator); immutable ConflictLog | Least-privilege access; full audit trail for compliance |
| **Blocking imports freeze the API** | S3 → SQS → Lambda worker pattern; API returns job ID instantly | API responds in <100ms; heavy lifting scales independently |
| **No visibility into production behavior** | Structured JSON logs (structlog) → CloudWatch; Prometheus metrics; `/health/detailed` | Debugging in seconds, not hours; RED metrics at a glance |

**Stack:** Python 3.12 • FastAPI • SQLModel • PostgreSQL 16 • Redis • Docker • AWS • GitHub Actions

---

### E-Commerce Backend API
[`jerryszc/ecommerce-backend-api`](https://github.com/jerryszc/ecommerce-backend-api) — Commercial management backend preventing inventory inconsistencies and order processing errors.

| Aspect | Implementation |
|--------|----------------|
| **Core Domain** | Product catalog, order management, inventory tracking with transactional guarantees |
| **Architecture** | Clean architecture, automated migrations (Alembic), strict schema validation |
| **Reliability** | Transactional consistency, strict schema validation (Pydantic/SQLModel), 34 tests passing |
| **Infrastructure** | Docker Compose for environment parity, containerized deployment ready |

---

### SSO Webhook Service
[`jerryszc/sso-webhook-service`](https://github.com/jerryszc/sso-webhook-service) — Centralized authentication with guaranteed event delivery.

| Aspect | Implementation |
|--------|----------------|
| **Core Domain** | SSO authentication + async webhook delivery with HMAC signatures |
| **Reliability** | Resilient retries with exponential backoff, dead-letter queue for failed deliveries |
| **Security** | Zero-trust architecture, HMAC signatures, JWT + rate limiting, strict typing (mypy strict) |
| **Testing** | 16 tests passing, containerized smoke tests, CI green |

---

### Realtime Task API
[`jerryszc/Realtime-task-api`](https://github.com/jerryszc/Realtime-task-api) — Real-time collaborative task management with strict access control.

| Aspect | Implementation |
|--------|----------------|
| **Core Domain** | Workspaces, boards, tasks with real-time WebSocket updates |
| **Access Control** | JWT authentication, RBAC (owner/admin/member) across workspaces/boards/tasks |
| **Real-time** | Native WebSocket broadcasting with connection management |
| **Testing** | 11 tests passing, 76% coverage, CI green |

---

## Technical Focus Areas

| Area | What I Focus On |
|------|-----------------|
| **API Design** | RESTful contracts, OpenAPI-first, Pydantic validation, versioning strategy |
| **Database** | PostgreSQL advanced (CTEs, window functions, advisory locks); SQLModel/SQLAlchemy; Alembic migrations |
| **Distributed Systems** | Event-driven patterns, idempotency keys, saga basics, eventual consistency |
| **Observability** | Structured logging (request IDs, structured context), RED metrics, health endpoints, distributed tracing basics |
| **Reliability** | Idempotency keys, retries with backoff, circuit breaker patterns, graceful degradation |
| **Security** | JWT rotation, bcrypt, rate limiting, OWASP headers, secret management (no plaintext in code) |
| **Developer Experience** | LocalStack for zero-cost AWS parity, pre-commit hooks, type-safe code (MyPy strict) |

---

## Engineering Philosophy

- **Correctness over cleverness** — boring code that works beats clever code that surprises
- **Observability first** — if you can't see it, you can't fix it
- **Operational ownership** — you build it, you run it, you instrument it
- **Iterate on feedback** — ship small, measure, learn, repeat

---

## Core Stack — Tools with Purpose

| Stack | What I Use It For | Business Value |
| :--- | :--- | :--- |
| **Python + FastAPI** | Fast, typed, production-ready APIs | Faster time-to-market, lower maintenance cost |
| **SQLModel / Pydantic** | Strict validation and domain modeling | Trusted data, fewer critical errors |
| **PostgreSQL / MySQL / SQLite** | Transactional persistence at the right scale | End-to-end consistency guaranteed |
| **Docker / Docker Compose** | Containerization and production-parity | Predictable deployments and friction-free isolated environments |
| **Pytest** | Automated tests for business logic and edge cases | Secure deployments, zero regressions |
| **Git** | Clean and traceable version control | Frictionless collaboration and auditability |

---

## Quality Standards

All projects ship with green CI: `ruff` + `mypy strict` + `pytest` + Docker build & smoke test.

---

## Contact

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)

---

> *"Simplicity is the soul of efficiency."* — Austin Freeman