# Infrastructure

Documentation of the AWS infrastructure CargoTrack runs on. This complements — rather than duplicates — the [cargotrack-infra-repo](https://github.com/CargoTrack-Org/cargotrack-infra-repo) Terraform code: the Terraform repo is the source of truth for *what's provisioned*, this folder is for explaining *why it's structured that way* and how the pieces fit together.

## Suggested contents

- `aws-services-inventory.md` — every AWS service in use, grouped by category (compute, networking, data, security, monitoring), and what it's used for
- `networking.md` — VPC layout, subnet tiers, how traffic enters and reaches the app
- `ci-cd-pipeline.md` — how a code change moves from `git push` to a running pod (build → image push → GitOps sync)

Nothing has been written yet — add files here as the infrastructure documentation is authored.
