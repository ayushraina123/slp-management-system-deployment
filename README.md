# SLP Management System Deployment

This folder contains the Docker Compose setup for running the complete **SLP Management System** stack locally.

The stack includes PostgreSQL, Eureka Server, Devotee Service, API Gateway, and the React frontend.

---

## Services

| Service | Container | Image | Host Port | Purpose |
|---------|-----------|-------|-----------|---------|
| PostgreSQL | `slp-postgres` | `postgres:16-alpine` | `5432` | Application database |
| Eureka Server | `slp-eureka-server` | `ayushraina1208/slp-eureka-server:1.0` | `8761` | Service discovery |
| Devotee Service | `slp-devotee-service` | `ayushraina1208/slp-devotee-service:1.0` | `8081` | Core backend APIs |
| API Gateway | `slp-api-gateway` | `ayushraina1208/slp-api-gateway:1.0` | `8080` | Public backend entry point |
| Frontend | `slp-frontend` | `ayushraina1208/slp-frontend:1.0` | `3000` | React UI served by Nginx |

---

## Prerequisites

- Docker
- Docker Compose
- Ports `3000`, `5432`, `8080`, `8081`, and `8761` available on your machine

---

## Start the Stack

From this folder, run:

```bash
docker compose up -d
```

Check container status:

```bash
docker compose ps
```

Follow logs:

```bash
docker compose logs -f
```

Follow one service:

```bash
docker compose logs -f api-gateway
```

---

## Access URLs

| Component | URL |
|-----------|-----|
| Frontend | `http://localhost:3000` |
| API Gateway | `http://localhost:8080` |
| Devotee Service direct | `http://localhost:8081` |
| Eureka Dashboard | `http://localhost:8761` |
| PostgreSQL | `localhost:5432` |

Use the frontend at `http://localhost:3000` for normal application usage.

---

## Docker Network Hostnames

Services communicate inside the Compose network using service names:

| Hostname | Used By | Resolves To |
|----------|---------|-------------|
| `postgres` | Devotee Service | PostgreSQL |
| `eureka-server` | Devotee Service, API Gateway | Eureka Server |
| `devotee-service` | Internal callers if needed | Devotee Service |
| `api-gateway` | Frontend Nginx proxy | API Gateway |

Inside containers, do not use `localhost` to reach another container. Use the service hostname instead.

---

## Database

PostgreSQL is configured with:

| Variable | Value |
|----------|-------|
| `POSTGRES_DB` | `project_db` |
| `POSTGRES_USER` | `postgres` |
| `POSTGRES_PASSWORD` | `password` |

Data is persisted in the Docker volume:

```text
postgres-data
```

The compose file also mounts:

```text
./postgres/init.sql -> /docker-entrypoint-initdb.d/init.sql
```

This script runs when the database volume is initialized for the first time.

---

## Export Files

The Devotee Service mounts the local exports folder:

```text
./exports:/app/exports
```

Exported files written by the backend container should be available under:

```text
exports/
```

---

## Stop the Stack

```bash
docker compose down
```

Stop and remove the database volume:

```bash
docker compose down -v
```

Use `-v` only when you intentionally want to reset persisted PostgreSQL data.

---

## Recreate Containers

After pulling or rebuilding images:

```bash
docker compose up -d --force-recreate
```

---

## Startup Order

The compose file waits for:

1. PostgreSQL health check
2. Eureka Server health check
3. Devotee Service startup
4. API Gateway startup
5. Frontend startup

The Devotee Service registers with Eureka, and the API Gateway routes traffic to it through service discovery.

---

## Troubleshooting

### Eureka does not show registered services

Check logs:

```bash
docker compose logs -f eureka-server devotee-service api-gateway
```

Confirm the service environment variables point to:

```text
http://eureka-server:8761/eureka
```

### Backend cannot connect to PostgreSQL

Check PostgreSQL health and logs:

```bash
docker compose ps postgres
docker compose logs -f postgres
```

Confirm the Devotee Service JDBC URL is:

```text
jdbc:postgresql://postgres:5432/project_db
```

### Frontend API calls fail

Confirm the gateway is running:

```bash
docker compose ps api-gateway
```

The frontend Nginx config proxies `/api/`, `/login`, `/logout`, and `/refresh-token` to:

```text
http://api-gateway:8080
```
