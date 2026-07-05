# System Overview

> Status: draft — fill in as the architecture stabilizes.

## Services

| Service | Responsibility | Repo location |
|---|---|---|
| frontend | User-facing web UI | `cargotrack-app/frontend` |
| core-service | Auth, shipments, admin, reports | `cargotrack-app/services/core-service` |
| document-service | Document upload/download/storage | `cargotrack-app/services/document-service` |
| ai-service | Compliance analysis, OCR, copilot | `cargotrack-app/services/ai-service` |

## How they talk to each other

_TODO: describe synchronous (HTTP/internal DNS) vs asynchronous (event-driven) communication between services._

## Diagram

_TODO: add the current architecture diagram to [diagrams/](diagrams/) and reference it here, e.g.:_

```md
![CargoTrack architecture](diagrams/system-architecture.png)
```
