# Jhosbert Osorio

**Backend Developer — Python · FastAPI · PostgreSQL · WebSockets · AWS**

Construyo APIs de backend donde lo difícil no es el CRUD: es la concurrencia, la entrega
fiable y la frontera de confianza.
[this README also in English →](#english)

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)

---

## Busco

**Backend Developer junior, remoto, trabajando en español.** Disponible desde octubre de 2026,
que es cuando grado.

Cuatro sistemas de backend completos en unos dos meses, **136 tests automatizados en verde**
(75 + 34 + 16 + 11), MyPy estricto y CI que corre lint, typecheck, tests y build de Docker en
los cuatro repositorios.

Todo lo que afirmo en este README está en el código. Si algo no lo he construido, lo digo
abajo en vez de dejarlo fuera.

---

## Qué hago, y cómo lo compruebo

No me dedico a contar features. Cada proyecto se explica por **el problema que resuelve**, y
cada problema tiene un test con nombre que lo demuestra.

| Proyecto | Tests | El problema que resuelve |
|:---|:---|:---|
| [Inventory Automator](https://github.com/jerryszc/ecommerce-inventory-automator) | 75 | El stock diverge entre marketplaces y nadie sabe cuál es el real. Amazon marca 10, Shopify marca 8, y cada uno penaliza al vendedor por la diferencia |
| [SSO + Webhook Dispatcher](https://github.com/jerryszc/sso-webhook-service) | 16 | Un evento de integración se pierde si el receptor está caído, o se martilea si reintentas sin parar. Aquí hay backoff, cola de muertos e idempotencia obligatoria |
| [E-Commerce Backend API](https://github.com/jerryszc/ecommerce-backend-api) · **[en vivo](https://ecommerce-backend-api-sh6c.onrender.com/health)** | 34 | Dos clientes compran la última unidad a la vez y ambos reciben su pedido. Resuelto con bloqueo de fila por producto |
| [Realtime Task API](https://github.com/jerryszc/Realtime-task-api) | 11 | Dos personas toman la misma tarea porque el tablero solo se actualiza al recargar. Resuelto con push por WebSocket en vez de polling |

Los cuatro están ordenados por fuerza de evidencia, no por cuándo los escribí. Cada
repositorio tiene su sección **Contexto de uso** explicando dónde encaja el servicio en una
empresa real y qué le falta antes de producción.

---

## 1. E-Commerce Multi-Channel Inventory & Operations Automator

[`jerryszc/ecommerce-inventory-automator`](https://github.com/jerryszc/ecommerce-inventory-automator) ·
**75 tests** · Python 3.12 · FastAPI · SQLModel · PostgreSQL 16 · Redis · AWS

Capa de integración entre el ERP del negocio y los marketplaces donde se vende. La usa el
equipo de operaciones, no el usuario final.

| Problema | Solución | Evidencia |
|:---|:---|:---|
| El stock diverge entre canales y cada marketplace penaliza al vendedor | Sincronización *last-write-wins* con `ConflictLog` inmutable que registra cada discrepancia | `app/services/sync.py`, 18 tests de sync e importación |
| Cada proveedor manda CSV con nombres de columna distintos | Parser con tabla de alias en español e inglés, auto-creación de catálogo y deduplicación | `data/samples/messy_sample.csv` procesa filas válidas y reporta `error_rows[]` |
| Una carga pesada bloquea la API | Patrón S3 → SQS → worker | `test_aws.py` cubre S3, SQS y Secrets Manager contra LocalStack |
| Nadie sabe quién cambió qué | Logs JSON con `request_id`, métricas Prometheus, health checks que separan *vivo* de *listo* | 6 tests de observabilidad |
| Cualquiera escribe datos | JWT con rotación de refresh, RBAC `admin`/`operator`, rate limiting distribuido | 9 tests de auth extendida |

**Lo que le falta y lo digo aquí:** el consumidor de la cola no existe. `process_import_job`
está escrito (`app/services/aws_sqs.py`), pero nadie lo invierte todavía: lo que falta es
decidir si lo ejecuta un Lambda o un worker en ECS.

**Cobertura: 81.74%.** CI con PostgreSQL 16, Redis 7 y LocalStack como servicios.

---

## 2. SSO Auth Service & Webhook Dispatcher

[`jerryszc/sso-webhook-service`](https://github.com/jerryszc/sso-webhook-service) ·
**16 tests** · Python 3.12 · FastAPI async · SQLAlchemy async · PostgreSQL 16 · Redis 7 ·
Argon2id · PyJWT

Identidad con OAuth2 y dispatcher de webhooks firmado con HMAC. Los dos van juntos porque
comparten necesidad de auditoría y firma criptográfica.

| Problema | Solución | Evidencia |
|:---|:---|:---|
| Un evento se pierde si el receptor está caído, o se martilea con reintentos infinitos | 5 intentos con backoff exponencial (2s, 4s, 8s, 16s, 32s) y DLQ con reintento manual | `test_backoff_growth` |
| Un reintento tras un timeout cobra dos veces | `Idempotency-Key` obligatoria; sin ella, 422 | `test_webhook_publish_idempotent` |
| El receptor no puede verificar que el mensaje sea auténtico | HMAC-SHA256 sobre JSON canónico con claves ordenadas | `test_hmac_signature` |
| Un refresh token robado sirve para siempre | Rotación en cada uso con `rotated_from_jti`, más lista negra en Redis por `jti` | `test_register_login_me_refresh_logout` |
| No hay forma de saber quién accedió a qué | `audit_log` con retención configurable (90 días por defecto) | 3 tests dedicados a auditoría |

**Lo que le falta:** el rate limiter es *fail-open* a propósito. Si Redis cae, el límite no se
aplica. Es una decisión consciente y su coste es que una caída de Redis elimina la defensa
contra fuerza bruta; se compensa en el balanceador.

**CI contra PostgreSQL 16 y Redis 7 reales**, no SQLite ni mocks, porque la rotación de tokens
y el rate limiting dependen de que Redis se comporte como en producción.

---

## 3. E-Commerce Backend API — Inventory & Orders

[`jerryszc/ecommerce-backend-api`](https://github.com/jerryszc/ecommerce-backend-api) ·
**34 tests** · Python 3.11 · FastAPI · SQLModel · PostgreSQL 15 · Alembic ·
**En producción:** <https://ecommerce-backend-api-sh6c.onrender.com/health>

Backend transaccional: el núcleo de pedidos e inventario de una tienda online. Desplegado en
Render con su propia base de datos, en plan gratuito.

> Comprobado contra la instancia en vivo, no solo en local: un pedido de 2 unidades bajó el
> stock de 4 a 2 y registró el kardex (`-2`, `OUT`, resultante 2). Un pedido de 999 unidades
> devolvió **400** y el stock siguió en 2. Esa es la garantía de atomicidad, medida en
> producción.

| Problema | Solución | Evidencia |
|:---|:---|:---|
| Dos clientes compran la última unidad y ambos reciben su pedido | `SELECT ... FOR UPDATE` por producto: el segundo espera al primero | `test_order_multi_line_atomic_rollback` |
| Un pedido que falla a la mitad deja stock descontado sin pedido | Una sola transacción para todas las líneas, con rollback | `test_order_insufficient_stock_single_line_rolls_back` |
| El stock cambia y nadie sabe por qué | Kardex: cantidad, motivo, stock resultante y el pedido que lo causó | `test_adjust_increases_stock_and_kardex` |
| El dinero en punto flotante produce errores de redondeo | `Decimal` con `max_digits` y `decimal_places` explícitos | Validado en esquema y en los schemas de respuesta |

**Lo que le falta:** no tiene autenticación. Es el másSimple de los cuatro en ese sentido, y
está documentado como limitación en el README en vez de omitido. PostgreSQL es obligatorio por
diseño: el bloqueo de fila es la garantía, y SQLite no lo implementa igual.

**Cobertura mínima: 80%. MyPy strict.**

---

## 4. Realtime Task API — Collaborative Boards

[`jerryszc/Realtime-task-api`](https://github.com/jerryszc/Realtime-task-api) ·
**11 tests** · Python 3.11 · FastAPI · WebSockets · SQLModel · PostgreSQL 16

Tableros colaborativos con push en tiempo real, salas por workspace y RBAC de tres roles.

| Problema | Solución | Evidencia |
|:---|:---|:---|
| Dos personas toman la misma tarea porque el tablero no se actualiza | Push por WebSocket: `task.created`, `task.updated`, `task.moved`, `task.deleted` | `test_manager_board_and_workspace_rooms` |
| Un error de broadcast filtra datos entre clientes de un SaaS | Salas con prefijo explícito, `board:{id}` y `workspace:{id}` | 3 tests de WebSocket |
| Una conexión larga ignora la autorización si solo se valida en el handshake HTTP | Autorización comprobada **también** en el WebSocket, con cierre 4401 (sin credencial) y 4403 (sin permiso) | `test_ws_rejects_missing_token` |
| Una fuga de la base de datos entrega sesiones válidas | Refresh tokens guardados como SHA-256, nunca en texto plano | `test_token_roundtrip` |

**Lo que le falta, y es lo más importante:** el `ConnectionManager` es un diccionario en
memoria del proceso. Con dos réplicas, cada una notifica solo a sus clientes: un usuario
conectado a la réplica A no ve lo que pasa en la B. La solución es Redis Pub/Sub, y es el
primer arreglo que haría.

Es el proyecto con menos tests de los cuatro. Lo digo aquí en lugar de esconderlo.
**Cobertura mínima: 70%.**

---

## Cómo trabajo

Solo lo que está en el código de los repos de arriba.

| Área | En la práctica |
|:---|:---|
| **Concurrencia** | Bloqueo de fila pesimista (`SELECT ... FOR UPDATE`), transacciones explícitas con commit/rollback, salas de WebSocket |
| **Fiabilidad de integraciones** | Idempotencia obligatoria, backoff exponencial, cola de muertos con reintento manual, firmas HMAC-SHA256 |
| **Identidad y acceso** | JWT con rotación y revocación, Argon2id y bcrypt, RBAC de 2 y 3 roles, rate limiting por IP y ámbito, listas negras con TTL |
| **Datos** | PostgreSQL con Alembic, `Decimal` para dinero, kardex de inventario, índices y constraints en el modelo |
| **AWS** | S3, SQS y Secrets Manager vía `boto3`, con LocalStack para desarrollo y tests sin costo |
| **Observabilidad** | Logs JSON con `request_id`, métricas Prometheus, health checks que separan *vivo* de *listo para tráfico* |
| **Calidad** | 136 tests, MyPy `strict`, Ruff, CI de cuatro pasos, build de imagen y smoke test que verifica que la app **responde** |

### Lo que aún no he construido

Lo prefiero dicho:

- ~~Circuit breaker, saga, event sourcing~~ — ninguno implementado
- ~~Distributed tracing~~ — sin OpenTelemetry
- ~~CTEs, window functions, advisory locks~~ — no usadas
- ~~Kubernetes, Terraform, Kafka, RabbitMQ, MySQL~~ — sin experiencia práctica
- Versionado de API: solo el proyecto de SSO usa prefijo `/api/v1`; no es un patrón general

### Estándares de calidad

Los cuatro repos ejecutan en cada push y pull request: `ruff check`, `ruff format --check`,
`mypy` en modo estricto, `pytest` con gate de cobertura (80%, 70% y 80% según el proyecto), y
build de la imagen de Docker con smoke test que levanta el stack y verifica `/health` con
`curl -f`.

El último paso es el que más importa: no basta con que la imagen **construya**, tiene que
**arrancar y responder** en un entorno limpio.

---

## Educación

**Ingeniería Informática (B.Sc. Computer Engineering) — Universidad Nacional Experimental
Politécnica de la Fuerza Armada Nacional Bolivariana (UNEFA), Caracas, Venezuela.**

**2022 – octubre 2026.** Graduación prevista octubre de 2026.

---

<a name="english"></a>

# English

**Backend Developer — Python · FastAPI · PostgreSQL · WebSockets · AWS**

I build backend APIs where the hard part is not the CRUD: it is concurrency, reliable delivery
and the trust boundary.

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)

---

## Looking for

**Junior Backend Developer, remote, working in Spanish.** Available from October 2026, when I
graduate.

Four complete backend systems in about two months, **136 automated tests passing**
(75 + 34 + 16 + 11), MyPy strict, and CI running lint, typecheck, tests and a Docker build on
all four repositories.

Everything I claim here is in the code. Anything I have not built, I list below instead of
leaving it out.

---

## What I do, and how I check it

I do not list features. Each project is explained by **the problem it solves**, and each
problem has a specifically named test that proves it.

| Project | Tests | The problem it solves |
|:---|:---|:---|
| [Inventory Automator](https://github.com/jerryszc/ecommerce-inventory-automator) | 75 | Stock drifts between marketplaces and nobody knows which number is real. Amazon says 10, Shopify says 8, and each penalises the seller for the difference |
| [SSO + Webhook Dispatcher](https://github.com/jerryszc/sso-webhook-service) | 16 | An integration event is lost if the receiver is down, or hammered if you retry forever. Here there is backoff, a dead-letter queue and mandatory idempotency |
| [E-Commerce Backend API](https://github.com/jerryszc/ecommerce-backend-api) · **[live](https://ecommerce-backend-api-sh6c.onrender.com/health)** | 34 | Two customers buy the last unit at the same time and both get their order. Solved with per-product row locking |
| [Realtime Task API](https://github.com/jerryszc/Realtime-task-api) | 11 | Two people pick up the same task because the board only updates on refresh. Solved with WebSocket push instead of polling |

Ordered by strength of evidence, not by when I wrote them. Each repository has a **Use case**
section explaining where the service fits in a real company and what is missing before
production.

---

## 1. E-Commerce Multi-Channel Inventory & Operations Automator

[`jerryszc/ecommerce-inventory-automator`](https://github.com/jerryszc/ecommerce-inventory-automator) ·
**75 tests** · Python 3.12 · FastAPI · SQLModel · PostgreSQL 16 · Redis · AWS

The integration layer between a business's ERP and the marketplaces it sells on. Operations
uses it, not the end customer.

| Problem | Solution | Evidence |
|:---|:---|:---|
| Stock drifts between channels and each marketplace penalises the seller | Last-write-wins sync with an immutable `ConflictLog` of every discrepancy | `app/services/sync.py`, 18 sync and import tests |
| Every supplier sends CSV with different column names | Parser with an alias table in Spanish and English, catalogue auto-creation, deduplication | `data/samples/messy_sample.csv` processes valid rows and reports `error_rows[]` |
| A heavy upload blocks the API | S3 → SQS → worker pattern | `test_aws.py` covers S3, SQS and Secrets Manager against LocalStack |
| Nobody knows who changed what | JSON logs with `request_id`, Prometheus metrics, health checks separating *alive* from *ready* | 6 observability tests |
| Anyone can write data | JWT with refresh rotation, `admin`/`operator` RBAC, distributed rate limiting | 9 extended auth tests |

**What is missing, stated here:** the queue consumer does not exist. `process_import_job` is
written (`app/services/aws_sqs.py`) but nothing calls it yet, so what is left is deciding
whether a Lambda or an ECS worker runs it.

**Coverage: 81.74%.** CI with PostgreSQL 16, Redis 7 and LocalStack as services.

---

## 2. SSO Auth Service & Webhook Dispatcher

[`jerryszc/sso-webhook-service`](https://github.com/jerryszc/sso-webhook-service) ·
**16 tests** · Python 3.12 · FastAPI async · SQLAlchemy async · PostgreSQL 16 · Redis 7 ·
Argon2id · PyJWT

OAuth2 identity plus an HMAC-signed webhook dispatcher. They ship together because they share
a need for auditing and cryptographic signing.

| Problem | Solution | Evidence |
|:---|:---|:---|
| An event is lost if the receiver is down, or hammered by endless retries | 5 attempts with exponential backoff (2s, 4s, 8s, 16s, 32s) and a DLQ with manual retry | `test_backoff_growth` |
| A retry after a timeout charges twice | Mandatory `Idempotency-Key`; without it, 422 | `test_webhook_publish_idempotent` |
| The receiver cannot verify the message is authentic | HMAC-SHA256 over canonical JSON with sorted keys | `test_hmac_signature` |
| A stolen refresh token works forever | Rotation on every use with `rotated_from_jti`, plus a Redis blacklist by `jti` | `test_register_login_me_refresh_logout` |
| No way to know who accessed what | `audit_log` with configurable retention (90 days by default) | 3 dedicated audit tests |

**What is missing:** the rate limiter is deliberately fail-open. If Redis goes down the limit
is not applied. It is a conscious decision and its cost is that a Redis outage removes the
brute-force defence; it is compensated at the load balancer.

**CI against real PostgreSQL 16 and Redis 7**, not SQLite or mocks, because token rotation
and rate limiting depend on Redis behaving as it does in production.

---

## 3. E-Commerce Backend API — Inventory & Orders

[`jerryszc/ecommerce-backend-api`](https://github.com/jerryszc/ecommerce-backend-api) ·
**34 tests** · Python 3.11 · FastAPI · SQLModel · PostgreSQL 15 · Alembic ·
**Live:** <https://ecommerce-backend-api-sh6c.onrender.com/health>

Transactional backend: the order and inventory core of an online store. Deployed on Render
with its own database, on the free tier.

> Checked against the running instance, not only locally: an order of 2 units moved stock
> from 4 to 2 and wrote the kardex entry (`-2`, `OUT`, resulting 2). An order of 999 units
> returned **400** and the stock stayed at 2. That is the atomicity guarantee, measured in
> production.

| Problem | Solution | Evidence |
|:---|:---|:---|
| Two customers buy the last unit and both get their order | `SELECT ... FOR UPDATE` per product: the second waits for the first | `test_order_multi_line_atomic_rollback` |
| An order that fails halfway leaves stock deducted with no order saved | One transaction for every line, with rollback | `test_order_insufficient_stock_single_line_rolls_back` |
| Stock changes and nobody knows why | Kardex: quantity, reason, resulting stock and the order that caused it | `test_adjust_increases_stock_and_kardex` |
| Floating point money produces rounding errors | `Decimal` with explicit `max_digits` and `decimal_places` | Validated in the schema and the response models |

**What is missing:** there is no authentication. It is the simplest of the four in that
respect, and it is documented as a limitation in the README rather than omitted. PostgreSQL is
required by design: row locking is the guarantee, and SQLite does not implement it the same
way.

**Coverage floor: 80%. MyPy strict.**

---

## 4. Realtime Task API — Collaborative Boards

[`jerryszc/Realtime-task-api`](https://github.com/jerryszc/Realtime-task-api) ·
**11 tests** · Python 3.11 · FastAPI · WebSockets · SQLModel · PostgreSQL 16

Collaborative boards with real-time push, workspace-scoped rooms and three-role RBAC.

| Problem | Solution | Evidence |
|:---|:---|:---|
| Two people pick up the same task because the board does not update | WebSocket push: `task.created`, `task.updated`, `task.moved`, `task.deleted` | `test_manager_board_and_workspace_rooms` |
| A broadcast bug leaks data between clients of a multi-tenant SaaS | Explicitly prefixed rooms, `board:{id}` and `workspace:{id}` | 3 WebSocket tests |
| A long-lived connection skips authorisation if it is only checked on the HTTP handshake | Authorisation re-checked **on the WebSocket**, closing 4401 (no credential) and 4403 (no permission) | `test_ws_rejects_missing_token` |
| A database leak hands over valid sessions | Refresh tokens stored as SHA-256, never in plaintext | `test_token_roundtrip` |

**What is missing, and it is the most important thing:** the `ConnectionManager` is an
in-process dictionary. With two replicas each notifies only its own clients, so a user
connected to replica A does not see what happens on B. The fix is Redis Pub/Sub, and it is
the first thing I would change.

It is the project with the fewest tests of the four. I say so here rather than hide it.
**Coverage floor: 70%.**

---

## How I work

Only what is present in the code of the repositories above.

| Area | In practice |
|:---|:---|
| **Concurrency** | Pessimistic row locking (`SELECT ... FOR UPDATE`), explicit commit/rollback transactions, WebSocket rooms |
| **Integration reliability** | Mandatory idempotency keys, exponential backoff, dead-letter queue with manual retry, HMAC-SHA256 signatures |
| **Identity and access** | JWT with rotation and revocation, Argon2id and bcrypt, 2- and 3-role RBAC, per-IP and per-scope rate limiting, TTL blacklists |
| **Data** | PostgreSQL with Alembic, `Decimal` for money, inventory kardex, model-level indexes and constraints |
| **AWS** | S3, SQS and Secrets Manager via `boto3`, with LocalStack for zero-cost development and tests |
| **Observability** | Structured JSON logs with `request_id`, Prometheus metrics, health checks separating *alive* from *ready for traffic* |
| **Quality** | 136 tests, MyPy `strict`, Ruff, four-step CI, image build and a smoke test that verifies the app **responds** |

### What I have not built yet

I would rather say it:

- Circuit breaker, saga, event sourcing — none implemented
- Distributed tracing — no OpenTelemetry
- CTEs, window functions, advisory locks — unused
- Kubernetes, Terraform, Kafka, RabbitMQ, MySQL — no hands-on experience
- API versioning: only the SSO project uses a `/api/v1` prefix; it is not a general pattern

### Quality standards

All four repositories run on every push and pull request: `ruff check`,
`ruff format --check`, `mypy` in strict mode, `pytest` with a coverage gate (80%, 70% and 80%
depending on the project), and a Docker image build with a smoke test that brings the stack
up and checks `/health` with `curl -f`.

The last step matters most: an image building is not enough, it has to **start and respond**
in a clean environment.

---

## Education

**B.Sc. in Computer Engineering — Universidad Nacional Experimental Politécnica de la
Fuerza Armada Nacional Bolivariana (UNEFA), Caracas, Venezuela.**

2022 – October 2026, graduating October 2026.

---

## Contact

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)
