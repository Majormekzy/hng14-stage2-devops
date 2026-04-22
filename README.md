# HNG Stage 2 DevOps — Containerized Job Processing System

A multi-service job processing application containerized with Docker and deployed with Nginx.

## Services

| Service  | Description                        | Port |
|----------|------------------------------------|------|
| frontend | Node.js/Express job dashboard      | 3000 |
| api      | Python/FastAPI job management API  | 8000 |
| worker   | Python job processor               | —    |
| redis    | Job queue and state store          | 6379 |

## Prerequisites

- Docker >= 24.0
- Docker Compose >= 2.0
- Git

## Run Locally

```bash
# Clone the repo
git clone https://github.com/Majormekzy/hng14-stage2-devops
cd hng14-stage2-devops

# Set up environment variables
cp .env.example .env
# Edit .env and set a strong REDIS_PASSWORD

# Build and start all services
docker compose up --build

# Visit http://localhost:3000
```

## Endpoints

### Frontend (port 3000)
- `GET /` — Job dashboard UI
- `POST /submit` — Submit a new job
- `GET /status/:id` — Check job status
- `GET /health` — Health check

### API (port 8000)
- `POST /jobs` — Create a job
- `GET /jobs/:id` — Get job status
- `GET /health` — Health check

## What a Successful Startup Looks Like
redis     | Ready to accept connections
api       | Uvicorn running on http://0.0.0.0:8000
worker    | Worker started, waiting for jobs...
frontend  | Frontend running on port 3000

## Stopping

```bash
docker compose down
```
