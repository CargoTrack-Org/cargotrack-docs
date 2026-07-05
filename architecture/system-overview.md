# System Overview

## The Services

CargoTrack is split into four containers, each with a single responsibility. All four are built from `cargotrack-app`, packaged as Docker images, and deployed as separate Kubernetes Deployments (see [cargotrack-helm](https://github.com/CargoTrack-Org/cargotrack-helm)).

| Service | Port | Responsibility |
|---|---|---|
| **frontend** | 8080 | React SPA. nginx serves the static build and reverse-proxies API calls to the backend services. |
| **core-service** | 4000 | Authentication, shipments, notifications, reports, admin operations. Orchestrates calls to ai-service. |
| **document-service** | 4001 | Document upload/download, S3 storage, publishes upload events. |
| **ai-service** | 4002 | Compliance analysis (Bedrock + Textract), AI route briefings, admin copilot Q&A. |

Only `frontend` is reachable from outside the cluster. The other three are addressed internally by Kubernetes DNS (e.g. `core-service:4000`) and have no public endpoint.

nginx in the frontend container splits incoming requests:

```
/api/documents/*  →  document-service
/api/*            →  core-service
/*                →  React SPA (static files)
```

core-service is the only service that calls ai-service directly — ai-service is never called from the frontend or from document-service.

## Data Model (core-service, PostgreSQL via Prisma)

| Model | Purpose |
|---|---|
| `User` | Account with `email`, hashed `password`, `role` (`USER` \| `ADMIN`) |
| `Shipment` | Unique `trackingNumber`, status, and vNext fields used by the AI layer: `commodityType`, `hsCodeHint`, `isDangerousGoods`, `incoterms`, `declaredValue`, `aiRiskScore`, `aiRiskLevel` |
| `TrackingEvent` | Timeline of status changes for a shipment |
| `ShipmentDocument` | Metadata for an uploaded document (type, S3 key), linked to a shipment |
| `Notification` | Per-user notifications |
| `ShipmentAIBriefing` | One-to-one with `Shipment` — corridor, risk summary, customs complexity, sanctions status, delay probability, key risks |
| `ComplianceReport` | One-to-one with `Shipment` — risk level, overall risk score, executive summary |
| `ComplianceFinding` | Individual findings on a report — severity, finding type, evidence, reasoning, confidence score |

## Authentication

core-service issues a JWT on login/register (`{ userId, email, role }`, 7-day expiry by default). The frontend sends it as `Authorization: Bearer <token>` on every request; core-service and document-service validate it independently. Admin-only routes additionally check `role === ADMIN`.

Internal calls from core-service to ai-service (briefing, copilot, compliance trigger) are authenticated separately with an `x-internal-secret` header, so ai-service never has to trust the end-user's JWT directly.

## Request Flow — User-Facing Example

A user creating a shipment:

```
Browser → frontend (nginx) → core-service: POST /api/shipments
                                   │
                                   ├── writes Shipment row (PostgreSQL)
                                   ├── fire-and-forget → ai-service: POST /api/briefing/generate/:id
                                   └── fire-and-forget → ai-service: POST /api/compliance/trigger
```

The shipment is created and the response returns immediately — the AI briefing and compliance analysis happen asynchronously, so the user isn't blocked waiting on an LLM call.

## Background Pipelines

These run independently of any single user request — they're event-driven, using EventBridge and SQS.

### 1. Document Audit Trail

```
User uploads a document
  → document-service stores the file in S3
  → document-service publishes "DocumentUploaded" to EventBridge
  → EventBridge routes it to the document-processor SQS queue
  → a Lambda function consumes the message
  → Lambda writes an immutable audit record to DynamoDB
```

### 2. Compliance Analysis

```
Shipment is created, or its status changes to IN_TRANSIT / DELIVERED
  → core-service publishes a status/compliance event to EventBridge
  → EventBridge routes it to the compliance-trigger SQS queue
  → ai-service long-polls this queue continuously (20s wait, 300s visibility timeout)
  → on receiving a message: fetches the shipment's documents, runs Textract OCR,
    sends the extracted content + shipment data to Bedrock for analysis
  → ai-service writes a ComplianceReport + ComplianceFindings to PostgreSQL
  → the SQS message is deleted only on success; failures are retried automatically
    via the queue's visibility timeout, with a dead-letter queue as a backstop
```

Compliance analysis can also be triggered manually via `POST /api/admin/compliance/trigger/:shipmentId` (see [../api/core-service.md](../api/core-service.md)), which calls the same code path directly instead of going through the queue.

## AI Layer

- **Bedrock (Amazon Nova)** — the LLM behind compliance analysis, route briefings, and the copilot. Called through the Converse API with tool-use for the compliance agent.
- **Textract** — OCR for uploaded PDFs/images, so Bedrock can read document contents that aren't plain text.
- **Copilot** — a set of admin-facing endpoints (summary, risk explanation, recommendations, free-form Q&A, similar-shipment comparison, timeline narrative) that call the same underlying LLM provider with different prompts, scoped to one shipment at a time.

The LLM provider is pluggable (`LLM_PROVIDER` env var) — Bedrock in production, with a mock provider available for local development without AWS credentials.
