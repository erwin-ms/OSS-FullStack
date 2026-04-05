# podman-app-deploy
A guide for deploy basic front end and backend app by podman ( daemonless &amp; Rootless) 

A containerised web application with a REST API backed by PostgreSQL, deployed
entirely with **Podman** (no Docker required).

## Architecture

```
┌────────────┐        ┌────────────────┐
│  PostgreSQL │◄──────►│  FastAPI (API)  │──► :8000
│  :5432      │        └────────────────┘
└────────────┘
```

| Component | Description |
|-----------|-------------|
| **api**   | Python 3.12 / FastAPI application that exposes REST endpoints for managing *items* and *feature flags*. |
| **db**    | PostgreSQL 16 (Alpine) with an `init.sql` seed script that creates tables and default data. |

Both services are orchestrated via `podman-compose`.

## Prerequisites

* [Podman](https://podman.io/) ≥ 4.x
* [podman-compose](https://github.com/containers/podman-compose) ≥ 1.0

Install on Fedora / RHEL:

```bash
sudo dnf install -y podman podman-compose
```

Install on Ubuntu / Debian:

```bash
sudo apt-get install -y podman
pip install podman-compose
```

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/erwin-ms/Open_Source-Tech.git
cd Open_Source-Tech

# 2. (Optional) Create a .env file with your own password
cp .env.example .env
# Edit .env and set POSTGRES_PASSWORD

# 3. Build and start all containers
podman-compose up -d --build

# 4. Verify the services are running
podman-compose ps
curl http://localhost:8000/health
```

The API documentation (Swagger UI) is available at **<http://localhost:8000/docs>**.

## API Reference

### Health

| Method | Path      | Description       |
|--------|-----------|-------------------|
| GET    | `/`       | Service info      |
| GET    | `/health` | Health-check      |

### Items

Provision and manage items stored in the database.

| Method | Path              | Description              |
|--------|-------------------|--------------------------|
| GET    | `/api/items/`     | List items (paginated)   |
| POST   | `/api/items/`     | Create a new item        |
| GET    | `/api/items/{id}` | Get item by ID           |
| PATCH  | `/api/items/{id}` | Update item              |
| DELETE | `/api/items/{id}` | Delete item              |

**Example – create an item:**

```bash
curl -X POST http://localhost:8000/api/items/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Widget", "description": "A sample widget", "metadata": {"colour": "blue"}}'
```

### Features

Provision and manage feature flags / toggles.

| Method | Path                  | Description                   |
|--------|-----------------------|-------------------------------|
| GET    | `/api/features/`      | List feature flags (paginated)|
| POST   | `/api/features/`      | Create a new feature flag     |
| GET    | `/api/features/{id}`  | Get feature flag by ID        |
| PATCH  | `/api/features/{id}`  | Update (enable/disable/config)|
| DELETE | `/api/features/{id}`  | Delete feature flag           |

**Example – enable a feature:**

```bash
curl -X PATCH http://localhost:8000/api/features/1 \
  -H "Content-Type: application/json" \
  -d '{"enabled": true}'
```

## Stopping the Services

```bash
podman-compose down          # stop and remove containers
podman-compose down -v       # also remove the database volume
```

## Project Structure

```
.
├── LICENSE
├── README.md
├── .env.example
├── podman-compose.yml       # Podman Compose orchestration
├── api/
│   ├── Containerfile        # OCI image build (Podman-native naming)
│   ├── requirements.txt
│   └── app/
│       ├── __init__.py
│       ├── main.py          # FastAPI entry-point
│       ├── config.py        # Env-based configuration
│       ├── database.py      # Async SQLAlchemy engine
│       ├── models.py        # ORM models
│       ├── schemas.py       # Pydantic request/response schemas
│       └── routers/
│           ├── __init__.py
│           ├── items.py     # /api/items endpoints
│           └── features.py  # /api/features endpoints
└── db/
    └── init.sql             # Database seed script
```

## License

[MIT](LICENSE)
