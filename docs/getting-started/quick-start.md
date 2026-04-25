# Quick Start

Get Flare running locally in under 5 minutes using Docker Compose.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) v2+

## 1. Create your project folder

```bash
mkdir flare && cd flare
```

## 2. Create the environment file

Create a `.env` file with your configuration:

```env
# Admin account (created automatically on first start)
FLARE_ADMIN_USERNAME=admin
FLARE_ADMIN_PASSWORD=SecurePassword123!
FLARE_ADMIN_FULLNAME=System Administrator

# PostgreSQL
POSTGRES_USER=flare
POSTGRES_PASSWORD=StrongPassword123!
POSTGRES_DB=flare

# UI → API connection (external URL, as seen from the browser)
API_BASE_URL=http://localhost:5000/api

# CORS — the origin where flare-ui is served (single URL)
CORS_ALLOWED_ORIGINS=http://localhost:3000

# OpenTelemetry (optional) — remove or leave blank to disable
# FLARE_OTEL_ENDPOINT=http://otel-collector:4318
```

> **Security note:** Use a strong, unique password for both `FLARE_ADMIN_PASSWORD` and `POSTGRES_PASSWORD` before deploying anywhere beyond your local machine.

## 3. Create the docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:15
    container_name: flare-postgres
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - flare-network

  flare-api:
    image: ghcr.io/flrapp/flare-api:latest
    container_name: flare-api
    ports:
      - "5000:5000"
    environment:
      - ConnectionStrings__DefaultConnection=Host=postgres;Database=${POSTGRES_DB};Username=${POSTGRES_USER};Password=${POSTGRES_PASSWORD}
      - FLARE_ADMIN_USERNAME=${FLARE_ADMIN_USERNAME}
      - FLARE_ADMIN_PASSWORD=${FLARE_ADMIN_PASSWORD}
      - FLARE_ADMIN_FULLNAME=${FLARE_ADMIN_FULLNAME}
      - FLARE_CORS_ALLOWED_ORIGINS=${CORS_ALLOWED_ORIGINS}
      - ASPNETCORE_ENVIRONMENT=Production
    depends_on:
      - postgres
    networks:
      - flare-network
    restart: unless-stopped

  flare-ui:
    image: ghcr.io/flrapp/flare-ui:latest
    container_name: flare-ui
    ports:
      - "3000:80"
    environment:
      - API_BASE_URL=${API_BASE_URL}
    depends_on:
      - flare-api
    restart: unless-stopped
    networks:
      - flare-network

volumes:
  postgres_data:

networks:
  flare-network:
    driver: bridge
```

## 4. Start Flare

```bash
docker compose up -d
```

On first start, Flare automatically:
- Runs database migrations
- Creates the admin account from your `.env` values

## 5. Open the UI

Go to [http://localhost:3000](http://localhost:3000) and log in with the credentials you set in `.env`.

You're in. Create your first project and start managing flags.

---

## What's next?

- [Core Concepts](./core-concepts.md) — understand projects, scopes, and flag types
- [OpenTelemetry](../observability/opentelemetry.md) — enable tracing and metrics
- [SDK Overview](../sdks/overview.md) — integrate Flare into your application
