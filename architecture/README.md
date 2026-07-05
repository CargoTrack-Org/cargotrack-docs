# Architecture

Documentation of how CargoTrack's application is put together — the services, how they talk to each other, and how a request or event flows end to end.

For infrastructure-level detail (VPC, EKS, RDS, IAM, etc.) see [../infrastructure/](../infrastructure/) instead. This folder is about the *application* architecture; `infrastructure/` is about the *platform* it runs on.

## Contents

- [system-overview.md](system-overview.md) — the microservices, data model, request flow, and background pipelines

Visual architecture diagrams are kept in the slide deck under [../presentations/](../presentations/) rather than as separate image files here, to avoid maintaining the same diagram in two places.
