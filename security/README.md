# Security

CargoTrack's security model is built around one principle: no long-lived credentials anywhere. Everything else — encryption, network boundaries, secrets — follows from that.

## Identity & Access (IAM / IRSA)

Pods never hold static AWS access keys. Each Kubernetes `ServiceAccount` is annotated with an IAM role ARN; through the cluster's OIDC provider, a pod can call `sts:AssumeRoleWithWebIdentity` and get temporary, auto-rotated credentials scoped to only what that service needs. This is IRSA (IAM Roles for Service Accounts).

Separate IAM roles exist per component:

| Role | Scope |
|---|---|
| `core_service` | RDS access, EventBridge publish, ai-service invocation |
| `document_service` | S3 read/write on the documents bucket, EventBridge publish |
| `ai_service` | Bedrock invoke, Textract, SQS consume, RDS access |
| `alb_controller` | Manage ALBs/target groups on behalf of Ingress resources |
| `cluster_autoscaler` | Scale the EKS managed node group |
| `eso` (External Secrets Operator) | Read Secrets Manager / SSM Parameter Store |

CI/CD uses the same pattern: GitHub Actions authenticates via GitHub's own OIDC provider to assume an IAM role for ECR pushes, rather than storing AWS keys as repo secrets.

## Encryption

A single customer-managed KMS key encrypts everything at rest: RDS, S3 (documents), DynamoDB (audit trail), SQS, and Secrets Manager. Key rotation is enabled. In transit, CloudFront terminates TLS at the edge (ACM-managed certificate); traffic between CloudFront and the ALB, and between services inside the cluster, stays within the private VPC.

## Network Boundaries

Only `frontend` is publicly reachable. The path in is:

```
Internet → Route 53 → CloudFront (TLS + WAF) → ALB → frontend pod
```

`core-service`, `document-service`, and `ai-service` have no public address — they're only resolvable via internal Kubernetes DNS. WAF (attached to CloudFront) blocks known-bad IPs, OWASP Top 10 patterns, and SQL injection attempts before traffic ever reaches the ALB.

## Secrets & Configuration

Two different stores for two different sensitivity levels:

- **Secrets Manager** — DB password, JWT signing secret, admin password. Encrypted with the KMS key.
- **SSM Parameter Store** — non-sensitive config: DB host/port/name, environment name.

The External Secrets Operator (ESO) runs inside the cluster, reads Secrets Manager/SSM hourly, and keeps a Kubernetes `Secret` up to date automatically — nothing is manually copy-pasted into the cluster, and rotation in Secrets Manager propagates without a redeploy.

## Application-Level Auth

- **User authentication**: JWT issued by core-service on login/register, sent as `Authorization: Bearer <token>`. Validated independently by core-service and document-service. Admin-only endpoints additionally check the `role` claim.
- **Service-to-service auth**: calls from core-service to ai-service (briefing, copilot, compliance trigger) are authenticated with a shared `x-internal-secret` header, kept separate from the end-user's JWT so ai-service never has to trust a token it didn't issue.

## Monitoring

GuardDuty continuously analyzes CloudTrail, VPC flow logs, DNS queries, and EKS audit logs for suspicious activity; HIGH/CRITICAL findings route through EventBridge to SNS for email alerting. CloudWatch alarms separately watch resource-level signals (RDS CPU, EKS node CPU/memory, pod restarts, queue depth).
