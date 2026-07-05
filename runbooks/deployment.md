# Deployment

CargoTrack deploys through GitOps — no one runs `kubectl apply` or `helm upgrade` by hand. See [../infrastructure/ci-cd-pipeline.md](../infrastructure/ci-cd-pipeline.md) for the full pipeline; this page is the short operational version.

## Normal Flow

1. Push to `develop` (dev) or `main` (prod) in `cargotrack-app`.
2. GitHub Actions builds, scans, and pushes a new image to ECR, then bumps the image tag in the corresponding `values-<env>.yaml` in `cargotrack-helm`.
3. ArgoCD, watching `cargotrack-gitops` → `cargotrack-helm`, picks up the commit and rolls it out to the `cargotrack-dev` or `cargotrack-prod` namespace — zero-downtime, since each Deployment is configured with `maxUnavailable: 0`.

Nobody needs cluster access for a routine deploy — only push access to the relevant Git branch.

## Checking a Deployment

```bash
kubectl get pods -n cargotrack-dev        # or cargotrack-prod
kubectl rollout status deployment/core-service -n cargotrack-dev
argocd app get core-service-dev            # if you have ArgoCD CLI access
```

The `deploy.yml` pipeline also runs an automatic smoke test after sync (health-check curls through the ALB) — check the GitHub Actions run if you want that result without touching the cluster directly.

## Rolling Back

A rollback is a Git operation:

- **Revert the tag-bump commit** in `cargotrack-helm` for the affected service/environment, or
- **Use ArgoCD's history** (`argocd app rollback <app> <revision>`) to sync back to a previous known-good state.

Either way, ArgoCD's `selfHeal` will converge the cluster to match Git automatically — there's no separate "undo" step on the cluster itself.

## Environments

| | Dev | Prod |
|---|---|---|
| Branch | `develop` | `main` |
| Namespace | `cargotrack-dev` | `cargotrack-prod` |
| Values file | `values-dev.yaml` | `values-prod.yaml` |

Both are managed by the same ArgoCD `root-app`; there is no separate cluster per environment.
