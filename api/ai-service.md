# ai-service API

Port `4002` · Source: `cargotrack-app/services/ai-service`

ai-service is the AI layer: compliance analysis, route briefings, and the admin copilot. It is only ever called from core-service (never directly by the frontend), and every route below is authenticated with an `x-internal-secret` header rather than a user JWT.

## Compliance Analysis

| Method | Path | Description |
|---|---|---|
| POST | `/api/compliance/trigger` | Runs the compliance agent for a shipment. Starts the analysis asynchronously and returns immediately. |

The compliance agent (`agent/runner.ts`) drives a Bedrock Converse loop with tool-use: it pulls the shipment's documents, runs them through Textract OCR, and has the model reason over the extracted content to produce a risk level, an executive summary, and a list of findings (each with severity, evidence, and a confidence score) — written to `ComplianceReport` / `ComplianceFinding` in PostgreSQL.

This same code path also runs continuously in the background: ai-service long-polls the compliance-trigger SQS queue (20s wait, 300s visibility timeout) and processes messages as they arrive — see [../architecture/system-overview.md](../architecture/system-overview.md#background-pipelines).

## AI Briefings

| Method | Path | Description |
|---|---|---|
| POST | `/api/briefing/generate/:shipmentId` | Generates a route briefing (corridor risk, customs complexity, sanctions status, delay probability). Returns `202` immediately; generation happens in the background with an automatic retry on failure. |
| GET | `/api/briefing/:shipmentId` | Returns the briefing once ready, or `{ status: "generating" }` while it's still in progress — by design, this never returns 404 for a shipment that exists. |

## Copilot

| Method | Path | Description |
|---|---|---|
| POST | `/api/copilot/:shipmentId/summary` | Executive summary of the shipment's risk |
| POST | `/api/copilot/:shipmentId/explain-risk` | Plain-language explanation of why a shipment has its current risk score |
| POST | `/api/copilot/:shipmentId/recommendations` | Suggested next actions |
| POST | `/api/copilot/:shipmentId/ask` | Free-form question answering about the shipment |
| GET | `/api/copilot/:shipmentId/similar` | Finds and compares similar past shipments |
| GET | `/api/copilot/:shipmentId/timeline` | Narrative summary of the shipment's tracking history |

All copilot methods fetch the relevant shipment/report data first, then call the configured LLM provider. A request for a shipment that doesn't exist returns 404.

## LLM Provider

The model backing all of the above is selected via `LLM_PROVIDER`: `bedrock` in production, with `gemini` and a `mock` provider (for local development without any AI credentials) also implemented.

## Health

| Method | Path | Auth |
|---|---|---|
| GET | `/api/health` | none |
