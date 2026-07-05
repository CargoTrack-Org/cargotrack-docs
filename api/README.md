# API Reference

Endpoint-level documentation for each backend microservice, matching `cargotrack-app/services/`. All endpoints are JSON over HTTP unless noted otherwise.

| File | Service | Port | Reached via |
|---|---|---|---|
| [core-service.md](core-service.md) | Auth, shipments, admin, reports, notifications | 4000 | `frontend` → `/api/*` |
| [document-service.md](document-service.md) | Document upload/download, S3 storage | 4001 | `frontend` → `/api/documents/*` |
| [ai-service.md](ai-service.md) | Compliance analysis, AI briefings, copilot | 4002 | `core-service` only — never called directly by the frontend |

## Auth Conventions

- **JWT** — user-facing endpoints require `Authorization: Bearer <token>`, issued by `POST /api/auth/login`. Admin-only endpoints additionally require the token's `role` claim to be `ADMIN`.
- **Internal secret** — core-service → ai-service calls are authenticated with an `x-internal-secret` header instead of the user's JWT.
- **Public** — a small number of endpoints (health checks, tracking-number lookup, registration/login) require no auth.
