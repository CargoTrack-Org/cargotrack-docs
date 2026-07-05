# API Reference

Endpoint-level documentation for each backend microservice. One file per service, matching the services in `cargotrack-app/services/`.

If the services expose an OpenAPI/Swagger spec, prefer generating these docs from that spec (or linking to it) over hand-maintaining endpoint lists, to avoid drift between code and docs.

## Services

| File | Service | Port |
|---|---|---|
| [core-service.md](core-service.md) | Auth, shipments, admin, reports | 4000 |
| [document-service.md](document-service.md) | Document upload/download/storage | 4001 |
| [ai-service.md](ai-service.md) | Compliance analysis, OCR, copilot | 4002 |
