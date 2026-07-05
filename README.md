# CargoTrack Docs

Documentation for **CargoTrack** — a cloud-native logistics platform for tracking international shipments and automatically screening them for customs, sanctions, and compliance risk.

This repo is the documentation hub for the CargoTrack project. It contains no application or infrastructure code — those live in the repos listed below. If you're new to the project, read this page top to bottom; it's designed to give you the full picture on its own.

## What CargoTrack Does

International freight shipments carry legal obligations — correct customs codes, sanctions screening, dangerous goods handling, complete paperwork. Getting any of these wrong can mean a shipment held at customs, fines, or worse.

CargoTrack is a shipment tracking platform that adds an AI layer on top of the basics: when a shipment is created or its status changes, the platform automatically reads its documents, runs a compliance analysis using a large language model, and produces a risk report with specific findings — before the shipment reaches a border.

**Core capabilities:**
- Shipment creation and tracking, with a public tracking-number lookup
- Document upload/storage per shipment (bills of lading, invoices, packing lists, etc.)
- Automated compliance analysis (AI-generated risk score, findings, executive summary)
- AI route briefings (corridor risk, customs complexity, sanctions status, delay probability)
- An admin "copilot" — ask questions about a shipment's risk, get recommendations, compare to similar shipments
- An immutable audit trail of every document and compliance event

## Architecture at a Glance

```
Browser
   │
   ▼
frontend (React SPA, served by nginx)
   │
   ├── /api/*            → core-service   (auth, shipments, admin, reports)
   └── /api/documents/*  → document-service (file upload/download, S3)
                                │
                    core-service │ calls
                                ▼
                          ai-service (compliance analysis, AI briefings, copilot)
                                │
                    ┌───────────┼───────────────┐
                    ▼           ▼               ▼
                Bedrock     Textract          SQS / EventBridge
              (LLM, Nova)   (OCR)          (async event pipelines)
```

| Service | Responsibility | Publicly reachable? |
|---|---|---|
| **frontend** | React SPA; nginx serves the build and reverse-proxies API calls | Yes — the only public entry point |
| **core-service** | Auth, shipments, admin, notifications, reports; orchestrates calls to ai-service | No — internal only |
| **document-service** | Document upload/download, S3 storage, publishes upload events | No — internal only |
| **ai-service** | Compliance analysis (Bedrock + Textract), AI briefings, copilot Q&A | No — internal only |

All backend services run as containers on Kubernetes (EKS) and talk to each other over internal cluster DNS. Two background pipelines run independently of user requests:

1. **Document audit trail** — every document upload publishes an event through EventBridge → SQS → Lambda, which writes an immutable record to DynamoDB.
2. **Compliance analysis** — every shipment creation or status change (`IN_TRANSIT`/`DELIVERED`) publishes an event through EventBridge → SQS, which ai-service polls continuously and processes into a compliance report in PostgreSQL.

See [architecture/system-overview.md](architecture/system-overview.md) for the full breakdown, and [presentations/](presentations/) for visual diagrams.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, TypeScript, Tailwind CSS |
| Backend services | Node.js, TypeScript, Express |
| Database | PostgreSQL (via Prisma ORM) |
| AI | Amazon Bedrock (Nova), Amazon Textract (OCR) |
| Messaging | Amazon EventBridge, Amazon SQS |
| File storage | Amazon S3 |
| Audit log | Amazon DynamoDB |
| Container platform | Amazon EKS (Kubernetes) |
| Deployment | Helm + ArgoCD (GitOps) |
| Infrastructure as code | Terraform |
| CI/CD | GitHub Actions (lint → SAST → dependency scan → image scan → ECR → GitOps) |

## Repositories

| Repo | Purpose |
|---|---|
| [cargotrack-app](https://github.com/CargoTrack-Org/cargotrack-app) | Application source — frontend + core-service, ai-service, document-service |
| [cargotrack-helm](https://github.com/CargoTrack-Org/cargotrack-helm) | Helm charts for Kubernetes deployment |
| [cargotrack-gitops](https://github.com/CargoTrack-Org/cargotrack-gitops) | ArgoCD Application manifests (GitOps) |
| [cargotrack-infra-repo](https://github.com/CargoTrack-Org/cargotrack-infra-repo) | Terraform — AWS infrastructure (EKS, RDS, networking, etc.) |
| **cargotrack-docs** | You are here — documentation, architecture, and presentations |

## Documentation Map

| Folder | Contents |
|---|---|
| [architecture/](architecture/) | Services, data flow, request lifecycle, background pipelines |
| [infrastructure/](infrastructure/) | AWS services inventory, Terraform module breakdown, CI/CD pipeline |
| [security/](security/) | Identity & access, encryption, network boundaries, secrets handling |
| [api/](api/) | Endpoint reference per microservice |
| [runbooks/](runbooks/) | Local dev setup, deployment process, incident response, current known issues |
| [adr/](adr/) | Architecture Decision Records — why key technical decisions were made |
| [presentations/](presentations/) | Slide deck — includes architecture diagrams and product screenshots |

## Conventions

See [CONTRIBUTING.md](CONTRIBUTING.md) for how documentation and presentations should be added to this repo.
