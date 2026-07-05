# CargoTrack Docs

Documentation, presentations, and screenshots for the **CargoTrack** logistics platform.

This repo is the documentation hub for the CargoTrack project. It does not contain application code, infrastructure code, or deployment manifests — those live in their own repos.

## Related Repositories

| Repo | Purpose |
|---|---|
| [cargotrack-app](https://github.com/CargoTrack-Org/cargotrack-app) | Application source — frontend + core-service, ai-service, document-service |
| [cargotrack-helm](https://github.com/CargoTrack-Org/cargotrack-helm) | Helm charts for Kubernetes deployment |
| [cargotrack-gitops](https://github.com/CargoTrack-Org/cargotrack-gitops) | ArgoCD Application manifests (GitOps) |
| [cargotrack-infra-repo](https://github.com/CargoTrack-Org/cargotrack-infra-repo) | Terraform — AWS infrastructure (EKS, RDS, networking, etc.) |

## What's in This Repo

| Folder | Contents |
|---|---|
| [architecture/](architecture/) | System overview, service breakdown, architecture diagrams |
| [infrastructure/](infrastructure/) | AWS infrastructure documentation (complements `cargotrack-infra-repo`) |
| [security/](security/) | Security architecture, identity, encryption, network posture |
| [api/](api/) | API reference per microservice |
| [runbooks/](runbooks/) | Operational guides — local dev setup, deployment, incident response |
| [adr/](adr/) | Architecture Decision Records — why key technical decisions were made |
| [presentations/](presentations/) | Slide decks (pitch deck, demo deck, evaluation deck) |
| [screenshots/](screenshots/) | Product screenshots, organized by area |

## Conventions

See [CONTRIBUTING.md](CONTRIBUTING.md) for how documentation, diagrams, and screenshots should be added to this repo.
