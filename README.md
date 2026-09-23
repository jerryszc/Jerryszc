I solve: data integrity loss, fragile manual processes, and backends that crash in production.

I build Python APIs that protect business logic, automate critical operations, and are production-ready with zero regressions.

## How I Work

* **API Design & Modeling:** Business logic comes first. I prioritize transactional consistency and clear contracts over improvised endpoints.
* **Error Mitigation & Validation:** I build active defenses against anomalous data, invalid states, and concurrency issues.
* **Quality & QA:** Nothing ships without automated coverage. Zero regressions in production (`pytest`).

## Core Stack — Tools with Purpose

| Stack | What I Use It For | Business Value |
| :--- | :--- | :--- |
| **Python + FastAPI** | Fast, typed, production-ready APIs | Faster time-to-market, lower maintenance cost |
| **SQLModel / Pydantic** | Strict validation and domain modeling | Trusted data, fewer critical errors |
| **PostgreSQL / MySQL / SQLite** | Transactional persistence at the right scale | End-to-end consistency guaranteed |
| **Docker / Docker Compose** | Containerization and production-parity | Predictable deployments and friction-free isolated environments |
| **Pytest** | Automated tests for business logic and edge cases | Secure deployments, zero regressions |
| **Git** | Clean and traceable version control | Frictionless collaboration and auditability |

## 🚀 Portfolio Projects

All three ship with green CI: `ruff` + `mypy strict` + `pytest` + Docker build & smoke test.

* **[E-Commerce Backend API](https://github.com/jerryszc/ecommerce-backend-api)** — 34 pytest passing, CI green
  * **Problem it solves:** Automates commercial management, preventing inventory inconsistencies, order processing errors, and production crashes through a robust, containerized architecture.
  * **Stack:** FastAPI, PostgreSQL, SQLModel, Alembic, Docker, and Pytest.
  * **Enfoque:** Clean architecture, automated database migrations, strict schema validation, and environment parity with Docker Compose.

* **[SSO Webhook Service](https://github.com/jerryszc/sso-webhook-service)** — 16 pytest passing, CI green
  * **Problem it solves:** Centralizes authentication (SSO) and guarantees event delivery through asynchronous webhooks with HMAC signatures, resilient retries, and a dead-letter queue.
  * **Stack:** FastAPI, PostgreSQL, Redis, SQLModel, Alembic, Docker, and Pytest.
  * **Enfoque:** Zero-trust security, strict typing (`mypy strict`), JWT + rate limiting, and containerized smoke tests.

* **[Realtime Task API](https://github.com/jerryszc/Realtime-task-api)** — 11 pytest passing, 76% coverage, CI green
  * **Problem it solves:** Enables real-time collaborative task management with strict access control across workspaces, boards, and tasks.
  * **Stack:** FastAPI, PostgreSQL, SQLModel, WebSockets, RBAC, Docker, and Pytest.
  * **Enfoque:** JWT authentication, owner/admin/member RBAC, native WebSocket broadcasting, and automated coverage.

## 📈 Let's Connect
* GitHub: [@jerryszc](https://github.com/jerryszc)
* LinkedIn: [Jhosbert Osorio](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
* Email: [Jhosbertosorio@gmail.com](mailto:Jhosbertosorio@gmail.com)
