# core-service API

Port `4000` · Source: `cargotrack-app/services/core-service` · Database: PostgreSQL via Prisma

core-service owns authentication, shipment CRUD, notifications, reports, and all admin operations. It's the only backend service the frontend talks to for anything other than document upload/download, and the only service that calls ai-service.

## Auth

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/api/auth/register` | Create a user, return a JWT | none |
| POST | `/api/auth/login` | Verify credentials, return a JWT | none |
| GET | `/api/auth/me` | Get own profile | JWT |
| PUT | `/api/auth/me` | Update name/email | JWT |
| POST | `/api/auth/me/avatar` | Upload profile picture (max 5MB) | JWT |

JWTs are signed with `{ userId, email, role }`, expiring after 7 days by default. There is no server-side session — every request re-validates the token's signature.

## Shipments

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/api/shipments` | List own shipments (paginated, searchable) | JWT |
| POST | `/api/shipments` | Create a shipment. Also fires off (fire-and-forget) an AI briefing generation call and a compliance-analysis trigger to ai-service | JWT |
| GET | `/api/shipments/:id` | Shipment detail, including tracking events and documents | JWT |
| PUT | `/api/shipments/:id` | Update a shipment (blocked once `DELIVERED` or `CANCELLED`) | JWT |
| DELETE | `/api/shipments/:id` | Cancel a shipment | JWT |
| GET | `/api/tracking/:trackingNumber` | Public tracking lookup by tracking number | none |

## Notifications & Reports

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/api/notifications` | List own notifications | JWT |
| PUT | `/api/notifications/:id/read` | Mark a notification read | JWT |
| GET | `/api/reports/shipment-history` | CSV export of tracking events | JWT |
| GET | `/api/reports/shipment-summary` | CSV export of shipments | JWT |

## Admin

All routes below additionally require `role: ADMIN` on the JWT.

| Method | Path | Description |
|---|---|---|
| GET | `/api/admin/stats` | Platform-wide stats and risk distribution |
| GET | `/api/admin/shipments` | All shipments across all users |
| PUT | `/api/admin/shipments/:id/status` | Update shipment status; publishes `shipment.status_updated` to EventBridge and auto-triggers compliance analysis when the new status is `IN_TRANSIT` or `DELIVERED` |
| GET | `/api/admin/documents` / `/api/admin/documents/:shipmentId` | List all documents, or all documents for one shipment |
| GET | `/api/admin/compliance/:shipmentId` | Read a shipment's compliance report |
| POST | `/api/admin/compliance/trigger/:shipmentId` | Manually re-trigger compliance analysis for a shipment |
| GET / POST | `/api/admin/briefing/:shipmentId`, `/api/admin/briefing/generate/:shipmentId` | Read / (re-)generate a shipment's AI route briefing |
| POST / GET | `/api/admin/copilot/:shipmentId/summary`, `/explain-risk`, `/recommendations`, `/ask`, `/similar`, `/timeline` | Proxies to ai-service's copilot endpoints, authenticated with an internal secret, 110s timeout |

## Health

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/api/health` | Liveness/readiness probe target | none |
