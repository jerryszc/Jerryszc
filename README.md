# Jhosbert Osorio

**Backend Developer — Python · FastAPI · PostgreSQL · WebSockets · AWS**

Construyo APIs de backend donde la parte difícil no es el CRUD: es la concurrencia, la
entrega fiable y la seguridad.[this README also in English →](#english)

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)

---

## Sobre mí

Desarrollador backend autodidacta. Construí cuatro sistemas completos de backend en
aproximadamente dos meses: **136 tests automatizados**, MyPy en modo estricto y
integración continua con lint, typecheck, tests y build de Docker en los cuatro.

No es una muestra de velocidad sin coste. Es la consecuencia de una rutina: elegir un problema
real, escribir primero el test que demuestra el fallo, y después la implementación mínima
que lo resuelve.

**Lo que me interesa del backend**

- **Corrección bajo concurrencia.** Qué pasa cuando dos peticiones llegan a la vez.
- **Entrega fiable.** Qué pasa cuando el sistema receptor está caído.
- **Frontera de confianza.** Qué se valida, dónde, y qué se acepta sin verificar.

**Busco:** puesto de Backend Developer junior en una empresa donde el backend sea el
producto, no un accesorio. Abierto a **trabajo remoto en español**, en cualquier país.
Idiomas de trabajo: español (nativo). Inglés A2, en mejora.

---

## Proyectos

Ordenados por la fuerza de la evidencia, no por el orden en que los escribí. Cada entrada
enlaza a su repositorio, donde están las tablas de API, la arquitectura y la salida de los
tests.

### 1. E-Commerce Multi-Channel Inventory & Operations Automator
[`jerryszc/ecommerce-inventory-automator`](https://github.com/jerryszc/ecommerce-inventory-automator) · **75 tests** · AWS

Sincronización de inventario entre canales de venta con procesamiento asíncrono en AWS,
RBAC y observabilidad de producción.

| Problema de negocio | Solución técnica | Evidencia |
| :--- | :--- | :--- |
| El stock diverge entre marketplaces: Amazon dice 10, Shopify dice 8, y ninguno es fiable | Sincronización *last-write-wins* por `updated_at` con `ConflictLog` inmutable que registra cada discrepancia | 18 tests de sync e importación, incluido el caso de conflicto |
| Los CSV de cada proveedor traen las columnas con nombres distintos y un error bloquea el lote entero | Parser tolerante a alias en español e inglés, auto-creación del catálogo, deduplicación | `data/samples/messy_sample.csv` procesa las filas válidas y reporta `error_rows[]` de las inválidas |
| La API se bloquea mientras procesa una carga pesada | Patrón S3 → SQS → worker: la petición encola y retorna | `test_aws.py` cubre encolado y recepción en SQS, y subida y descarga en S3 |
| No se puede ver qué está pasando en ejecución | Logs JSON con `request_id` y latencia, métricas Prometheus, health checks con estado de DB y Redis | 6 tests de observabilidad, incluido `test_security_headers_present` |
| Cualquiera puede escribir datos | JWT HS256 con rotación de refresh, RBAC `admin`/`operator`, secrets fuera del código | 9 tests de auth extendida; AWS Secrets Manager con creación bajo demanda |

**Stack:** Python 3.12 · FastAPI · SQLModel · PostgreSQL 16 · Redis · AWS (S3, SQS, Secrets Manager) · LocalStack · structlog · Prometheus · Docker · GitHub Actions

---

### 2. SSO Auth Service & Webhook Dispatcher
[`jerryszc/sso-webhook-service`](https://github.com/jerryszc/sso-webhook-service) · **16 tests** · Argon2id

Identidad con OAuth2 y dispatcher de webhooks firmado con HMAC, con reintentos, cola de
muertos e idempotencia obligatoria.

| Problema de negocio | Solución técnica | Evidencia |
| :--- | :--- | :--- |
| Un evento de integración se pierde si el receptor está caído, o se martilea con reintentos infinitos si no se descarta | 5 intentos con backoff exponencial (2s, 4s, 8s, 16s, 32s) y DLQ con reintento manual por API | `test_backoff_growth`; `dispatch_worker.py` marca `status = "dlq"` al agotar intentos |
| Un reintento tras un timeout produce un cobro doble en el receptor | `Idempotency-Key` obligatoria al publicar; sin ella, 422 | `test_webhook_publish_idempotent` |
| El receptor no puede verificar que el mensaje sea auténtico | HMAC-SHA256 sobre JSON canónico, con claves ordenadas para que la firma sea estable | `test_hmac_signature` |
| Un refresh token robado sirve para acceder a la cuenta indefinidamente | Rotación en cada uso, con `rotated_from_jti` que deja rastro de la cadena, más lista negra en Redis por `jti` | `test_register_login_me_refresh_logout` |
| No hay forma de saber quién accedió a qué | Tabla `audit_log` con retención configurable | **3 tests dedicados** a auditoría: login fallido, token M2M y webhook creado |

**Stack:** Python 3.12 · FastAPI async · SQLAlchemy async · psycopg 3 · PostgreSQL 16 · Redis 7 · **Argon2id** · PyJWT (HS256) · Alembic · Docker

> La CI de este proyecto ejecuta los tests contra **PostgreSQL 16 y Redis 7 reales** con
> contenedores de servicio, porque la rotación de tokens y el rate limiting dependen de que
> Redis se comporte como en producción.

---

### 3. E-Commerce Backend API — Inventory & Orders
[`jerryszc/ecommerce-backend-api`](https://github.com/jerryszc/ecommerce-backend-api) · **34 tests** · Transactions

Backend transaccional de e-commerce con bloqueo de fila, kardex de inventario y atomicidad
en la creación de pedidos.

| Problema de negocio | Solución técnica | Evidencia |
| :--- | :--- | :--- |
| Dos clientes compran la última unidad a la vez y ambos reciben su pedido | `SELECT ... FOR UPDATE` por producto: el segundo pedido espera al primero | `test_order_multi_line_atomic_rollback` |
| Un pedido que falla a la mitad deja stock descontado sin pedido guardado | Una sola transacción para todas las líneas, con `rollback()` en el error | `test_order_insufficient_stock_single_line_rolls_back` |
| El stock cambia y nadie sabe por qué | Kardex: cada movimiento registra cantidad, motivo, stock resultante y el pedido que lo causó | `test_adjust_in_increases_stock_and_kardex` |
| El dinero en punto flotante produce errores de redondeo | `Decimal` con `max_digits` y `decimal_places` explícitos en toda columna monetaria | Validado en esquema y en los schemas de respuesta |
| Duplicados y paginación sin control devuelven datos inconsistentes | 409 para duplicados, 422 para esquema inválido, techo de 100 en `limit` | 4 tests de duplicados, 9 de validación |

**Stack:** Python 3.11 · FastAPI · SQLModel · SQLAlchemy · PostgreSQL 15 · Alembic · Pydantic · Pytest · Ruff · MyPy strict · Docker · Render

---

### 4. Realtime Task API — Collaborative Boards
[`jerryszc/Realtime-task-api`](https://github.com/jerryszc/Realtime-task-api) · **11 tests** · WebSockets

Tableros colaborativos tipo Kanban con actualizaciones en tiempo real, aislamiento por sala
y refresh tokens almacenados como hash.

| Problema de negocio | Solución técnica | Evidencia |
| :--- | :--- | :--- |
| Dos personas toman la misma tarea porque el tablero solo se actualiza al recargar | Push por WebSocket en vez de polling: 4 eventos (`task.created`, `updated`, `moved`, `deleted`) | `test_manager_board_and_workspace_rooms` |
| Un error de broadcast filtra datos entre clientes de un SaaS multi-tenant | Salas con prefijo explícito: `board:{id}` y `workspace:{id}` nunca colisionan | 3 tests de WebSocket |
| Una conexión de larga duración ignora la autorización si solo se valida en el handshake de otra ruta | Autorización comprobada **también en el handshake**, con cierre 4401 (sin credencial) y 4403 (sin permiso) | `test_ws_rejects_missing_token`, `test_ws_rejects_invalid_token` |
| Una fuga de la base de datos entrega sesiones válidas | Refresh tokens guardados como SHA-256, nunca en texto plano | `test_token_roundtrip` |

**Stack:** Python 3.11 · FastAPI · WebSockets · SQLModel · PostgreSQL 16 · Alembic · bcrypt · python-jose (HS256) · Docker

> Este es el proyecto con menos tests de los cuatro. Lo digo aquí en lugar de esconderlo:
> es el primero que ampliaría.

---

## Lo que trabajo con

Solo lo que está en el código de los repositorios de arriba.

| Área | En la práctica |
| :--- | :--- |
| **Concurrencia** | Bloqueo de fila pesimista (`SELECT ... FOR UPDATE`), transacciones explícitas con commit/rollback, WebSockets con salas |
| **Fiabilidad de integraciones** | Idempotencia obligatoria por clave, reintentos con backoff exponencial, cola de muertos con reintento manual, firmas HMAC-SHA256 |
| **Identidad y acceso** | JWT con rotación y revocación, Argon2id y bcrypt, RBAC de 2 y 3 roles, rate limiting por IP y por ámbito, listas negras con TTL |
| **Datos** | PostgreSQL con Alembic, `Decimal` para dinero, kardex de inventario, índices y constraints en el modelo, SQLModel/Pydantic |
| **AWS** | S3, SQS y Secrets Manager vía `boto3`, con LocalStack para desarrollo y tests sin costo |
| **Observabilidad** | Logs estructurados JSON con `request_id`, métricas Prometheus, health checks que separan *vivo* de *listo para tráfico* |
| **Calidad** | Pytest (136 tests en total), MyPy `strict = true`, Ruff, CI con cuatro jobs, build de imagen y smoke test que verifica que la app **responde** |

**Lo que aún no he construido** (y prefiero decirlo antes que declararlo):

- ~~Circuit breaker, saga, event sourcing~~ — ninguno implementado
- ~~Distributed tracing~~ — sin OpenTelemetry
- ~~CTEs, window functions, advisory locks~~ — no usadas en estos proyectos
- ~~Kubernetes, Terraform, Kafka, RabbitMQ, MySQL~~ — sin experiencia práctica
- Versionado de API: solo el proyecto de SSO usa prefijo `/api/v1`; no es un patrón general

---

## Estándares de calidad

Los cuatro repositorios ejecutan en cada push y pull request:

1. `ruff check` y `ruff format --check`
2. `mypy` en modo estricto
3. `pytest` con reporte de cobertura y gate por proyecto (80% en e-commerce, 70% en tiempo real)
4. Build de la imagen Docker y smoke test que levanta el compose y verifica `/health` con `curl -f`

El punto 4 es el que más importa: no basta con que la imagen **construya**, tiene que
**arrancar y responder** en un entorno limpio.

---

## Educación

**Ingeniería Informática (B.Sc. Computer Engineering) — Universidad Nacional
Experimental Politécnica de la Fuerza Armada Nacional Bolivariana (UNEFA),
Caracas, Venezuela.**

**2022 – octubre 2026.** Graduación prevista octubre de 2026.

---

<a name="english"></a>

# English

**Backend Developer — Python · FastAPI · PostgreSQL · WebSockets · AWS**

I build backend APIs where the hard part is not the CRUD: it is concurrency, reliable
delivery and security.

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)

---

## About

Self-taught backend developer. I built four complete backend systems in roughly two
months: **136 automated tests**, MyPy in strict mode, and CI running lint, typecheck, tests
and a Docker build on all four.

That is not speed without cost. It is the result of a routine: pick a real problem, write
the test that demonstrates the failure first, then write the smallest implementation that
fixes it.

**What I care about in backend work**

- **Correctness under concurrency.** What happens when two requests arrive at once.
- **Reliable delivery.** What happens when the receiving system is down.
- **The trust boundary.** What gets validated, where, and what is accepted unchecked.

**Looking for:** a junior Backend Developer role at a company where the backend is the
product rather than an accessory. Open to **remote work in Spanish**, anywhere. Working
languages: Spanish (native). English A2, in progress.

---

## Projects

Ordered by strength of evidence, not by when I wrote them.

### 1. E-Commerce Multi-Channel Inventory & Operations Automator
[`jerryszc/ecommerce-inventory-automator`](https://github.com/jerryszc/ecommerce-inventory-automator) · **75 tests** · AWS

Multi-channel inventory sync with asynchronous AWS processing, RBAC and production
observability. Solves stock divergence across marketplaces, supplier CSV imports with
mismatched column names, and API blocking during heavy uploads.

**Stack:** Python 3.12 · FastAPI · SQLModel · PostgreSQL 16 · Redis · AWS (S3, SQS, Secrets Manager) · LocalStack · structlog · Prometheus · Docker

### 2. SSO Auth Service & Webhook Dispatcher
[`jerryszc/sso-webhook-service`](https://github.com/jerryszc/sso-webhook-service) · **16 tests** · Argon2id

OAuth2 identity plus a webhook dispatcher with HMAC signing, exponential backoff retries, a
dead-letter queue and mandatory idempotency. CI runs the tests against **real PostgreSQL 16
and Redis 7** service containers, because token rotation and rate limiting depend on Redis
behaving as it does in production.

**Stack:** Python 3.12 · FastAPI async · SQLAlchemy async · PostgreSQL 16 · Redis 7 · **Argon2id** · PyJWT (HS256) · Alembic · Docker

### 3. E-Commerce Backend API — Inventory & Orders
[`jerryszc/ecommerce-backend-api`](https://github.com/jerryszc/ecommerce-backend-api) · **34 tests** · Transactions

Transactional e-commerce backend with row-level locking, an inventory kardex and atomicity
on order creation. Prevents overselling through `SELECT ... FOR UPDATE` and keeps a full
audit trail of every stock movement.

**Stack:** Python 3.11 · FastAPI · SQLModel · PostgreSQL 15 · Alembic · Pydantic · Pytest · MyPy strict · Docker · Render

### 4. Realtime Task API — Collaborative Boards
[`jerryszc/Realtime-task-api`](https://github.com/jerryszc/Realtime-task-api) · **11 tests** · WebSockets

Kanban-style boards with real-time push updates, room-scoped tenant isolation and
three-role RBAC. Authorisation is re-checked at the WebSocket handshake, not only on the
HTTP path, with close codes 4401 and 4403.

**Stack:** Python 3.11 · FastAPI · WebSockets · SQLModel · PostgreSQL 16 · Alembic · bcrypt · python-jose (HS256) · Docker

---

## What I work with

Only what is present in the code of the repositories above.

| Area | In practice |
| :--- | :--- |
| **Concurrency** | Pessimistic row locking (`SELECT ... FOR UPDATE`), explicit commit/rollback transactions, WebSocket rooms |
| **Integration reliability** | Mandatory idempotency keys, exponential backoff retries, dead-letter queue with manual retry, HMAC-SHA256 signatures |
| **Identity and access** | JWT with rotation and revocation, Argon2id and bcrypt, 2- and 3-role RBAC, per-IP and per-scope rate limiting, TTL blacklists |
| **Data** | PostgreSQL with Alembic, `Decimal` for money, inventory kardex, model-level indexes and constraints, SQLModel/Pydantic |
| **AWS** | S3, SQS and Secrets Manager via `boto3`, with LocalStack for zero-cost local development and tests |
| **Observability** | Structured JSON logs with `request_id`, Prometheus metrics, health checks separating *alive* from *ready for traffic* |
| **Quality** | Pytest (136 tests total), MyPy `strict = true`, Ruff, four-job CI, image build and a smoke test that verifies the app **responds** |

**What I have not built yet** (I would rather say it than claim it):

- Circuit breaker, saga, event sourcing — none implemented
- Distributed tracing — no OpenTelemetry
- CTEs, window functions, advisory locks — unused in these projects
- Kubernetes, Terraform, Kafka, RabbitMQ, MySQL — no hands-on experience

---

## Quality standards

All four repositories run on every push and pull request: `ruff check` + `ruff format
--check`, `mypy` in strict mode, `pytest` with a per-project coverage gate, and a Docker
build followed by a smoke test that brings the stack up and checks `/health` with `curl -f`.

---

## Education

**B.Sc. in Computer Engineering — Universidad Nacional Experimental Politécnica
de la Fuerza Armada Nacional Bolivariana (UNEFA), Caracas, Venezuela.**
2022 – October 2026, graduating October 2026.

---

## Contact

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)
