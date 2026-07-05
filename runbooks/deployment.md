# Deployment

> Status: stub.

## Flow

CargoTrack deploys via GitOps. Document the concrete steps and any manual gates here, e.g.:

1. Push to `main` in `cargotrack-app`
2. CI builds, scans, and pushes an image to ECR
3. CI updates the image tag in `cargotrack-helm`
4. ArgoCD (watching `cargotrack-gitops` → `cargotrack-helm`) syncs the change to EKS

## Rollback

_TODO: how to roll back a bad deploy — revert the tag commit in `cargotrack-helm`? ArgoCD history rollback?_

## Environments

_TODO: list environments (dev/prod), what differs between them, and where their config lives._
