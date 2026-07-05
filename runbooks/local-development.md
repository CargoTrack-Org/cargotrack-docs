# Local Development

## Prerequisites

- Docker + Docker Compose
- Node.js 20+ (only needed if running a service outside Docker)
- AWS credentials (optional — only needed to exercise Bedrock/Textract/S3 for real; everything has a local/mock fallback)

## Running the Full Stack

`cargotrack-app` defines the current microservices architecture in **`docker-compose.v3.yml`** — core-service, document-service, ai-service, frontend, and a local Postgres container, wired together the same way they are in production.

> Note: `docker-compose.yml` + `docker-compose.dev.yml` describe an older two-container (`backend`/`frontend`) layout from before the services were split apart. Use `docker-compose.v3.yml` for anything matching the current architecture.

```bash
cd cargotrack-app
cp .env.example .env.v3   # then adjust values as needed — see below
docker compose -f docker-compose.v3.yml --env-file .env.v3 up --build
```

An admin account is auto-seeded on first startup from `ADMIN_EMAIL` / `ADMIN_PASSWORD` (default: `admin@cargotrack.com` / `admin123`). There's no default regular user — register one through the app.

### Key environment variables (`docker-compose.v3.yml`)

| Variable | Purpose | Local default |
|---|---|---|
| `JWT_SECRET` | Signs auth tokens | dev placeholder — fine locally, must be overridden in any real environment |
| `AI_SERVICE_URL` | How core-service reaches ai-service | `http://ai-service:4002` (container DNS) |
| `INTERNAL_API_SECRET` | Shared secret for core-service → ai-service calls | unset by default (open) — set it if testing that auth path |
| `LLM_PROVIDER` | `bedrock` \| `gemini` \| unset (mock) | unset → deterministic mock responses, no AWS credentials needed |
| `AWS_DEFAULT_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | Only needed if `LLM_PROVIDER=bedrock` or `TEXTRACT_ENABLED=true` | unset |
| `S3_BUCKET`, `EVENT_BUS_NAME` | Only needed to exercise real S3 storage / EventBridge locally | unset — document-service falls back to local disk, event publishing is skipped |

### Verifying it's up

```bash
curl http://localhost:4000/api/health   # core-service
curl http://localhost:4001/api/health   # document-service
curl http://localhost:4002/api/health   # ai-service
curl http://localhost/api/health        # via nginx → core-service
```

## Running a Single Service Outside Docker

Each service under `cargotrack-app/services/<service>` is a standalone Node.js/TypeScript project:

```bash
cd services/core-service
npm install
npx prisma generate      # core-service is the only service that owns migrations
npm run dev
```

Point it at the same Postgres container the compose stack uses (`DATABASE_HOST=localhost`, port from `docker-compose.v3.yml`) if you want to run one service natively while the rest stay in Docker.
