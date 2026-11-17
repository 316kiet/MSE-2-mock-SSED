# CampusHub Docker deployment

## Quick start
Copy `.env.example` to `.env` and replace all placeholder passwords with local secrets. Then run:

```bash
docker compose up -d --build --wait
```

Open http://localhost:8080, test http://localhost:8080/api/health, and view test email at http://127.0.0.1:8025.

To inspect status: `docker compose ps`; to inspect API migration: `docker compose logs api`.
To stop without deleting data: `docker compose down`.

## Architecture
Browser -> Nginx gateway (8080) -> React frontend (8080), Node API (3000), MinIO storage (9000).
API -> PostgreSQL (persistent database), Redis (cache), RabbitMQ (registration queue), MinIO (persistent images).
Worker -> RabbitMQ -> MailHog (test SMTP).
Only gateway is published on all host interfaces; admin UIs are bound to localhost.

## Design decisions
1. Multi-stage frontend build keeps Node and build dependencies out of the runtime image.
2. Three isolated Compose networks limit service communication to required paths.
3. Named volumes preserve PostgreSQL and MinIO data across `docker compose down` and restarts.

## Verification
`docker compose ps`; `curl http://localhost:8080/api/health`; `curl http://localhost:8080/api/events`.
The grading script is written for Bash: `DC='docker compose' ENV_FILE=.env bash ./grade.sh` from a Bash-compatible environment. Warning: grade.sh runs `docker compose down -v` and deletes named volume data.
