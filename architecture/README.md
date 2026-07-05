# Architecture

High-level documentation of how CargoTrack is put together — the services, how they talk to each other, and how a request flows end to end.

For infrastructure-level detail (VPC, EKS, RDS, IAM, etc.), see [../infrastructure/](../infrastructure/) instead. This folder is about the *application* architecture; `infrastructure/` is about the *platform* it runs on.

## Contents

- [system-overview.md](system-overview.md) — the microservices, what each one does, and how they communicate
- [diagrams/](diagrams/) — exported architecture diagrams (images) and their editable sources

## Suggested diagrams to maintain here

- **Application architecture** — the 4 services and how they call each other
- **Request flow** — a single user request from browser to database and back
- **Event-driven pipelines** — the async flows (e.g. document upload → audit trail, shipment status change → compliance analysis)

Keep diagrams and `system-overview.md` in sync — if you change one, update the other.
