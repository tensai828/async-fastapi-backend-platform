# FastAPI + SQLModel + Alembic (Async) Starter

Production‑ready FastAPI backend with async PostgreSQL, Redis, Celery, MinIO object storage, JWT authentication, role‑based permissions, and AI/NLP examples.

---

## 1. Overview

This project is a **batteries‑included template** for building modern backend APIs with **FastAPI** and **SQLModel** on top of **PostgreSQL** using **async SQLAlchemy**.

It demonstrates how to put together:

- **Authentication & authorization** with JWT tokens and role‑based access control.
- **Async database access** with SQLModel, Alembic migrations, and PostgreSQL.
- **Caching & rate limiting** backed by Redis.
- **Background processing** with Celery workers and Celery Beat scheduler.
- **File storage** via MinIO (S3‑compatible object storage).
- **AI/NLP integrations** using HuggingFace Transformers, Celery batch jobs, and OpenAI/LangChain.
- **WebSocket chat** endpoint backed by OpenAI chat models.

You can use it as:

- A **learning project** to see how these pieces fit together.
- A **starter template** for your own production‑grade FastAPI backend.

---

## 2. Main Features

- **User & Role Management**
  - User registration, profile, and admin‑only user operations.
  - Role model (`admin`, `manager`, `user`) with role‑protected endpoints.
  - Social graph: follow/unfollow users, list followers and following.

- **Authentication**
  - Email/password login with JWT access & refresh tokens.
  - Token refresh endpoint and password change flow.
  - Token storage/validation with Redis (session‑like behavior).

- **Data & Domain**
  - Example domain models for **users**, **roles**, **groups**, **teams**, and **heroes**.
  - Initial sample data seeding via `initial_data.py`.

- **Media & File Storage**
  - Image upload endpoints for user avatars.
  - Image resize/processing and storage in MinIO.

- **Caching, Rate Limiting & Pagination**
  - Response caching using `fastapi-cache2` + Redis.
  - Per‑endpoint rate limiting using `fastapi-limiter`.
  - Consistent paginated responses via `fastapi-pagination`.

- **AI / NLP Examples**
  - Sentiment analysis endpoint using Transformers.
  - Celery batch tasks for text generation and delayed execution.
  - WebSocket `/chat/{user_id}` that proxies messages to an OpenAI chat model using LangChain.

- **Operations & Tooling**
  - Async SQLAlchemy/SQLModel engine configuration for different modes (dev/test/prod).
  - Alembic migrations already configured.
  - Docker Compose environment with:
    - FastAPI app
    - Redis
    - Celery worker & Celery Beat
    - MinIO
    - Caddy reverse proxy
  - Makefile helpers for development, migrations, tests, and SonarQube.

---

## 3. Tech Stack

- **Language & Framework**
  - Python 3.10–3.11
  - FastAPI
  - SQLModel + SQLAlchemy 2.x

- **Storage & Messaging**
  - PostgreSQL
  - Redis
  - MinIO (S3‑compatible)

- **Async & Background**
  - `fastapi-async-sqlalchemy`
  - Celery + `celery-sqlalchemy-scheduler`

- **Auth & Security**
  - `pyjwt[crypto]`, `bcrypt`, `cryptography`

- **AI / NLP**
  - `transformers`, `torch` (CPU wheels), `langchain`, `openai`

- **Tooling**
  - Poetry
  - Black, Ruff, MyPy, Pytest
  - Docker Compose

---

## 4. Project Structure (high level)

Key directories:

- `backend/app/app`
  - `main.py` – FastAPI application, lifespan, middleware, WebSocket chat, router inclusion.
  - `api/`
    - `v1/api.py` – API router aggregation.
    - `v1/endpoints/` – Individual route modules: `login`, `user`, `group`, `team`, `hero`, `cache`, `weather`, `natural_language`, `report`, `periodic_tasks`, etc.
    - `celery_task.py` – Celery task definitions (e.g., NLP batch jobs).
  - `core/`
    - `config.py` – Pydantic settings (DB, Redis, MinIO, OpenAI, CORS, etc.).
    - `security.py` – JWT utilities and password hashing.
    - `celery.py` – Celery app configuration.
    - `authz.*` – Authorization rules.
  - `crud/` – Repository layer for users, roles, teams, heroes, groups, media, etc.
  - `db/`
    - `session.py` – Async database session/engine.
    - `init_db.py` – Database seeding (roles, users, groups, teams, heroes).
  - `models/` – SQLModel models (users, roles, groups, teams, heroes, media).
  - `schemas/` – Pydantic/SQLModel schemas for requests & responses.
  - `utils/` – Helpers (exceptions, MinIO client, image resize, UUIDs, tokens, etc.).
  - `initial_data.py` – Entrypoint for populating sample data via CRUD layer.

- `backend/app/pyproject.toml`
  - Poetry dependencies and tooling configuration.

- Top‑level
  - `docker-compose.yml` – Production‑oriented stack.
  - `docker-compose-dev.yml` – Development stack (used by Makefile).
  - `docker-compose-test.yml` – Test stack.
  - `caddy/` – Reverse proxy configuration.
  - `pgadmin.yml`, `pgadmin/` – Optional pgAdmin 4 environment.
  - `static/` – Diagrams and documentation assets.

---

## 5. Getting Started (with Docker)

### 5.1. Prerequisites

- Docker and Docker Compose installed.
- (Optional) `make` available (for running Makefile targets).

### 5.2. Configure Environment

1. Copy the example environment file:

   ```bash
   cp .env.example .env
   ```

2. Edit `.env` as needed:
   - Database credentials (`DATABASE_*`).
   - Redis host/port.
   - MinIO credentials.
   - `OPENAI_API_KEY` for OpenAI/LangChain integrations.

### 5.3. Start Development Stack

Using the Makefile (recommended):

```bash
make run-dev-build   # first time, builds images
# or
make run-dev         # subsequent runs
```

Or directly with Docker Compose:

```bash
docker compose -f docker-compose-dev.yml up --build
```

Services started:

- FastAPI app (behind Caddy)
- Redis
- PostgreSQL (if enabled in dev compose)
- Celery worker
- Celery Beat
- MinIO
- Caddy reverse proxy

### 5.4. Apply Migrations & Seed Data

If you use the provided images/compose files, migrations and initial data are run automatically in the production stack. For the dev stack you can manually seed:

```bash
make init-db
```

This:

- Applies latest Alembic migrations.
- Seeds roles (`admin`, `manager`, `user`), groups, teams, heroes.
- Creates default superuser (`FIRST_SUPERUSER_EMAIL`/`FIRST_SUPERUSER_PASSWORD`).

---

## 6. Running the Application

- **API base URL** (dev): usually `http://localhost` via Caddy.
- **Interactive documentation**:
  - Swagger UI: `http://localhost/docs`
  - ReDoc: `http://localhost/redoc`

Key API areas:

- `POST /api/v1/login` – Login and obtain JWT tokens.
- `POST /api/v1/login/new_access_token` – Refresh access token.
- `GET /api/v1/user` – Get current user profile.
- `GET /api/v1/user/list` – Paginated user list (admin/manager).
- `POST /api/v1/natural_language/sentiment_analysis` – Sentiment analysis demo.
- `POST /api/v1/natural_language/text_generation_prediction_batch_task` – Launch Celery NLP job.
- `GET /api/v1/natural_language/get_result_from_batch_task` – Retrieve Celery job result.
- `GET /api/v1/weather/...` – Weather wrapper (using `WHEATER_URL`).

WebSocket:

- `GET /chat/{user_id}` – Chat endpoint using OpenAI chat model.  
  Connect with a WebSocket client, send JSON messages, and receive streaming bot responses.

---

## 7. Background Jobs & Celery

- Celery is configured in `app.core.celery` and uses:
  - Redis as broker.
  - PostgreSQL as result backend (via `SYNC_CELERY_DATABASE_URI`).
  - `celery-sqlalchemy-scheduler` for scheduled and periodic tasks.
- Example tasks:
  - Text generation via Transformers in `app.api.celery_task`.
  - Delayed execution via Celery ETA (see `natural_language` endpoints).

In the dev stack:

- `celery_worker` service runs workers.
- `celery_beat` service runs the scheduler.

---

## 8. Development & Quality

- **Install dependencies locally (optional, outside Docker):**

  ```bash
  cd backend/app
  poetry install
  ```

- **Code formatting & linting:**

  ```bash
  make formatter   # Black
  make lint        # Ruff + Black check
  make mypy        # Type checking
  ```

- **Tests (via Docker):**

  ```bash
  make run-test    # start test stack and run tests
  make pytest      # run pytest inside test container
  ```

- **Static analysis:**
  - `make run-sonarqube` and `make run-sonar-scanner` for SonarQube analysis.

---

## 9. Notes & Customization

- The project uses **environment‑based settings** via Pydantic `BaseSettings`. See `app.core.config.Settings` for all available options.
- `MODE` controls behavior in testing vs other environments (e.g., NullPool in tests).
- You can safely replace or extend:
  - Domain models and CRUD logic in `models/` and `crud/`.
  - API routers in `api/v1/endpoints/`.
  - Celery tasks in `api/celery_task.py`.
- MinIO is optional; you can swap it for AWS S3 or another object store by updating `utils.minio_client` and related configs.

---

## 10. License

This project is distributed under the terms of the **MIT License** (see `LICENSE` file).

