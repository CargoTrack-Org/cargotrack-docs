# document-service API

Port `4001` · Source: `cargotrack-app/services/document-service`

document-service owns file storage for shipment documents. It stores files in S3 (or a local directory in development), keeps metadata in PostgreSQL, and publishes an event on every upload for the audit pipeline.

## Endpoints

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/api/documents/shipment/:shipmentId` | Upload a document for a shipment (max 10MB). A regular user may only upload to their own shipment; an admin may upload to any shipment. | JWT |
| GET | `/api/documents/shipment/:shipmentId` | List documents for a shipment | JWT |
| GET | `/api/documents/:id/download` | Download a document. Returns 403 unless the caller owns the shipment or is an admin. | JWT |
| GET | `/api/health` | Liveness/readiness probe target | none |

## Storage

Files are written via a pluggable storage provider:

- **S3** (production) — used when `S3_BUCKET` and `AWS_DEFAULT_REGION` are set. Key pattern: `documents/{trackingNumber}/{uuid}{ext}`.
- **Local disk** (development) — falls back to writing into `UPLOAD_DIR` when S3 isn't configured, so the service runs without AWS credentials locally.

## Events

After a successful upload and database write, document-service publishes a `DocumentUploaded` event to EventBridge (`Source: cargotrack.documents`). This feeds the audit-trail pipeline described in [../architecture/system-overview.md](../architecture/system-overview.md) — the event is not required for the upload itself to succeed, and publishing is skipped (with a warning logged) if the event bus isn't configured.
