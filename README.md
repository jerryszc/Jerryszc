# Jhosbert Osorio

**Backend Developer — Python · FastAPI · PostgreSQL · WebSockets · AWS**

Construyo APIs de backend donde lo difícil no es el CRUD: es la concurrencia, la entrega
fiable y la frontera de confianza.

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github)](https://github.com/jerryszc)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin)](https://www.linkedin.com/in/jhosbert-osorio-2680913b5)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail)](mailto:Jhosbertosorio@gmail.com)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-25D366?logo=whatsapp)](https://wa.me/584125993061)

---

## Busco

**Backend Developer remoto, trabajando en español.** Disponible para **tiempo completo** o
medio tiempo (~25-30 horas: tardes y noches entre semana). Busca también contratos por
proyecto o consultoría puntual, que es la vía más rápida para entrar a un equipo y aprender en
producción.

Cuatro sistemas de backend completos, **136 tests automatizados en verde** (75 + 34 + 16 + 11),
MyPy estricto y CI que corre lint, typecheck, tests y build de Docker en los cuatro
repositorios.

**Estudio Ingeniería Informática** (inicio septiembre 2026, grado previsto agosto de 2030) y
trabajo en paralelo. Puedo dedicarle jornada completa o media según el equipo; si la oferta es
de tiempo completo, puedo hacerlo. Busco un equipo donde el código que escribo llegue a
producción y pueda aprender de quien lleva más años haciéndolo.

Cuando el proyecto lo permita, también trabajo por proyecto:

| Servicio | Qué incluye |
|:---|:---|
| API REST desde cero | FastAPI o Django, PostgreSQL, Alembic, Docker, documentación OpenAPI |
| Auditoría de concurrencia | Detectar sobreventa, condición de carrera y transacciones que no son atómicas |
| Integración con servicios de terceros | Webhooks firmados, reintentos con backoff, idempotencia obligatoria |
| Autenticación y roles | JWT con rotación, RBAC, rate limiting, hash de contraseñas |
| Migración a FastAPI | Convertir una API existente sin romper el contrato de los clientes |
| Auditoría de un proyecto | Tests que faltan, deuda técnica priorizada, plan por fases |

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

**Ingeniería Informática — Universidad Nacional Experimental Politécnica de la Fuerza Armada
Nacional Bolivariana (UNEFA), Caracas, Venezuela.**

**Septiembre 2026 – agosto 2030.** Grado previsto agosto de 2030. Estudio mientras trabajo, y por
eso mi cronología profesional empieza en paralelo y no antes.

Puedo dedicarle jornada completa o media según el equipo: si la oferta es de tiempo completo,
puedo hacerlo.

**Formación previa:** Bachillerato general, 2020 – 2025.

---

## Contact

**Caracas, Venezuela · remoto · trabajando en español**

| | |
|:---|:---|
| **Email** | [Jhosbertosorio@gmail.com](mailto:Jhosbertosorio@gmail.com) |
| **WhatsApp** | [+58 412-5993061](https://wa.me/584125993061) |
| **LinkedIn** | [linkedin.com/in/jhosbert-osorio-2680913b5](https://www.linkedin.com/in/jhosbert-osorio-2680913b5) |
| **GitHub** | [github.com/jerryszc](https://github.com/jerryszc) |

Si tienes un backend que necesita concurrencia correcta, tests o un despliegue, escríbeme con
el problema — no con la lista de technologies. Te respondo en español.
