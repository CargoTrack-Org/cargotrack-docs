# Incident Response

## First Checks

1. **CloudWatch dashboard** — `cargotrack-overview`. Covers RDS CPU/storage, EKS node CPU/memory, pod restarts, SQS queue depth.
2. **ArgoCD sync status** — an app stuck `OutOfSync` or `Degraded` usually means the last Git push didn't apply cleanly; check the Application's events.
3. **Pod logs** — `kubectl logs -n cargotrack-<env> deployment/<service>`. Start with the service closest to the symptom (e.g. ai-service for anything AI-related).
4. **SQS queue depth / DLQs** — a growing compliance-trigger or document-processor queue, or messages landing in a dead-letter queue, means a consumer (ai-service or the audit Lambda) is failing repeatedly rather than just being slow.

## Common Failure Modes

| Symptom | Likely cause | Where to check |
|---|---|---|
| Pod stuck `CrashLoopBackOff` | Bad config/secret, or a startup dependency (Postgres, Bedrock) unreachable | `kubectl describe pod`, then logs |
| ArgoCD shows `OutOfSync` and never self-heals | A manifest error in the last Helm values change | ArgoCD Application events, `helm template` locally to validate |
| 5xx from the frontend | ALB healthy but a backend pod isn't ready | `kubectl get pods`, readiness probe status |
| SQS queue depth climbing | Consumer (ai-service / Lambda) erroring on every message | Consumer logs; check the DLQ for the actual failing payload |
| Bedrock/Textract calls failing | IAM permission gap, model access not granted in the region, or throttling | ai-service logs, IAM role policy for `ai_service`, Bedrock model access console |

## Known Issue: Shipment Briefing / Copilot Failures

As of this writing, the shipment briefing endpoint has been reported to return 404, and the copilot's summary/ask endpoints have been reported to fail.

A static trace of the request path — frontend → `core-service` (`admin.ts`, `copilot.ts`) → `ai-service` (`briefing.ts`, `copilot.ts`) — found the routing consistent at every hop: paths, the `x-internal-secret` check, and the nginx proxy rules all line up correctly in the code. `GET /api/briefing/:shipmentId` is written to never return 404 for an existing shipment (it returns `{status:'generating'}` while pending), which means a 404 in practice points to something above the application code, most likely one of:

- `AI_SERVICE_URL` or `INTERNAL_API_SECRET` mismatched between the deployed core-service and ai-service pods
- A NetworkPolicy blocking core-service → ai-service traffic in that environment
- Bedrock model access / IAM permission errors inside ai-service surfacing as generic failures to the copilot

**This has not been root-caused from code alone** — confirming it requires live pod logs and environment variables from the affected environment (dev or prod). Per this project's investigation rules, don't change the request-handling code to "fix" this without first pulling those logs and confirming which of the above (or something else) is actually happening.

## Escalation

_TODO: fill in who/where to escalate to once there's a team beyond a single maintainer._
