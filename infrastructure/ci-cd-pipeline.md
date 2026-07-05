# CI/CD Pipeline

CargoTrack uses a GitOps deployment model split across three repos: `cargotrack-app` (code + CI), `cargotrack-helm` (desired state), and `cargotrack-gitops` (ArgoCD's entry point). No pipeline ever touches the Kubernetes cluster directly — everything goes through a Git commit that ArgoCD then applies.

## 1. Build (`cargotrack-app/.github/workflows/build.yml`)

Triggered on push to `main`/`develop` and on pull requests.

1. **Detect changes** — figures out which of the 4 services changed (all of them on `main`, only the changed ones on `develop`), and whether this push targets the `prod` or `dev` environment.
2. **Lint** — `tsc --noEmit` per changed service.
3. **SAST** — SonarQube static analysis (skipped on pull requests).
4. **Dependency scan** — Snyk, per service (non-blocking).
5. **Build, scan, push** (per changed service, in parallel):
   - Authenticate to AWS via GitHub OIDC (no stored AWS keys)
   - Build the Docker image
   - Scan it with Trivy — results uploaded as SARIF, and a **hard gate fails the build on any CRITICAL vulnerability**
   - Push the image to ECR, tagged with both the commit SHA and `latest`
6. **Trigger deploy** — dispatches `deploy.yml` with the image SHA, target environment, and list of changed services.

## 2. Deploy (`cargotrack-app/.github/workflows/deploy.yml`)

1. **Update Helm tags** — authenticates to AWS via OIDC, checks out `cargotrack-helm` (branch `main` for prod, `develop` for dev), and uses `yq` to bump `image.tag` in the relevant `values-<env>.yaml` for each changed service. Commits and pushes the change.
2. **Smoke test** — waits for ArgoCD to sync, checks `kubectl rollout status` for each service, discovers the ALB's DNS name, and curls the health endpoints to confirm the new version is actually serving traffic (non-blocking — reports pass/fail rather than failing the pipeline).

## 3. GitOps Sync (ArgoCD, watching `cargotrack-gitops` → `cargotrack-helm`)

```
root-app (App-of-Apps, bootstrapped once by Terraform)
  └── watches cargotrack-gitops/apps/
        └── one Application per service per environment (e.g. core-service-dev)
              └── watches cargotrack-helm/<service>/, values-<env>.yaml
                    └── applies the rendered manifests to the target namespace
```

Every Application has `syncPolicy.automated: { prune: true, selfHeal: true }` — ArgoCD applies changes automatically and reverts any manual `kubectl` drift back to what's in Git. Failed syncs retry automatically with exponential backoff.

## End-to-End

```
Developer pushes code to cargotrack-app
  → build.yml: lint, SAST, dependency scan, Docker build, Trivy scan, push to ECR
  → deploy.yml: bump image tag in cargotrack-helm, smoke test
  → ArgoCD detects the new commit in cargotrack-helm
  → ArgoCD applies the change to EKS (rolling update, zero downtime)
```

A rollback is a Git operation, not a cluster operation: reverting the tag-bump commit in `cargotrack-helm` (or using ArgoCD's own rollback-to-previous-sync) causes ArgoCD to converge the cluster back to the previous image.
