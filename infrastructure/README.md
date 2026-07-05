# Infrastructure

CargoTrack's infrastructure is entirely defined as code in [cargotrack-infra-repo](https://github.com/CargoTrack-Org/cargotrack-infra-repo) (Terraform) and deployed via [cargotrack-helm](https://github.com/CargoTrack-Org/cargotrack-helm) + [cargotrack-gitops](https://github.com/CargoTrack-Org/cargotrack-gitops) (Helm + ArgoCD). This page explains what's provisioned and why; the Terraform repo remains the source of truth for exact resource configuration.

See [ci-cd-pipeline.md](ci-cd-pipeline.md) for how a code change travels from a `git push` to a running pod.

## AWS Services

| Category | Service | Used for |
|---|---|---|
| Compute | **EKS** | Runs all 4 microservices as containers |
| Compute | **EC2** (EKS managed node group) | The worker nodes EKS schedules pods onto |
| Compute | **ECR** | Stores the 4 Docker images; CI pushes a new tag on every build |
| Compute | **Lambda** | Consumes document-upload events from SQS, writes audit records to DynamoDB |
| Networking | **VPC / Subnets / Route Tables** | Private network, split into public / app / db tiers across 2 AZs |
| Networking | **NAT Gateway / Internet Gateway** | Outbound internet access for private subnets; inbound path for the ALB |
| Networking | **ALB** | Created by the AWS Load Balancer Controller from the frontend's Ingress; routes HTTP to the frontend pod |
| Networking | **VPC Endpoints** | Private routes to S3, Secrets Manager, SSM, KMS — pods reach these without going through the NAT gateway |
| Networking | **Route 53** | DNS for the platform's domain |
| Edge | **CloudFront** | CDN in front of the ALB; terminates HTTPS |
| Edge | **WAF** | Attached to CloudFront — blocks known bad IPs, OWASP Top 10 patterns, SQL injection attempts |
| Edge | **ACM** | TLS certificate for the custom domain |
| Data | **RDS (PostgreSQL)** | Primary application database — users, shipments, documents metadata, compliance reports |
| Data | **S3** | Stores uploaded document files |
| Data | **DynamoDB** | Immutable audit trail of document and compliance events |
| AI | **Bedrock (Nova)** | LLM behind compliance analysis, briefings, and copilot |
| AI | **Textract** | OCR for uploaded documents |
| Messaging | **EventBridge** | Custom event bus; routes domain events to the right SQS queue |
| Messaging | **SQS** | Document-processor and compliance-trigger queues, each with a dead-letter queue |
| Security | **IAM / IRSA** | Every pod assumes its own IAM role via OIDC — no static AWS credentials anywhere |
| Security | **KMS** | Single customer-managed key encrypts RDS, S3, DynamoDB, SQS, and Secrets Manager at rest |
| Security | **Secrets Manager / SSM Parameter Store** | Sensitive secrets (DB password, JWT secret) vs. non-sensitive config (DB host, bucket name) |
| Monitoring | **CloudWatch** | Metrics, logs, alarms, and a dashboard |
| Monitoring | **SNS** | Fans out CloudWatch and GuardDuty alerts to email |
| State | **S3 + DynamoDB** | Terraform remote state and lock table |

## Terraform Modules

`cargotrack-infra-repo/cargotrack-infra/modules/`

| Module | Provisions |
|---|---|
| `networking` | VPC, subnets, internet/NAT gateways, route tables |
| `security` | Security groups for ALBs, EKS nodes, backend services, database |
| `eks` | EKS cluster, managed node group, OIDC provider, CloudWatch observability add-on |
| `compute` | An alternate EC2-based path (auto-scaling groups + ALBs) — not part of the primary EKS deployment |
| `database` | RDS PostgreSQL, Secrets Manager secrets, SSM parameters |
| `storage` | S3 document bucket (versioned, KMS-encrypted, access-logged) |
| `audit` | DynamoDB audit table |
| `eventing` | EventBridge bus + rules, SQS queues + DLQs, the audit Lambda |
| `irsa` | Per-service IAM roles (ai-service, core-service, document-service, ALB controller, cluster autoscaler, External Secrets Operator) |
| `ecr` | ECR repositories + GitHub Actions OIDC role for CI image pushes |
| `monitoring` | CloudWatch dashboard, alarms, SNS topic |
| `endpoints` | VPC interface/gateway endpoints (S3, Secrets Manager, SSM, KMS) |
| `cdn` | CloudFront distribution + WAF |
| `dns` | Route 53 + ACM certificate |

## Kubernetes Deployment

Each microservice has its own Helm chart under `cargotrack-helm/` (`ai-service/`, `core-service/`, `document-service/`, `frontend/`). All four share the same shape:

- A `Deployment` with a rolling update strategy (`maxSurge: 1`, `maxUnavailable: 0` — zero-downtime deploys), pods spread across availability zones, and readiness/liveness probes on `/api/health`
- A `ServiceAccount` annotated for IRSA, so the pod gets its own scoped AWS credentials
- A `Service` (ClusterIP) for internal DNS
- An `HPA`: `minReplicas: 2`, `maxReplicas: 10`, target 70% average CPU

Only `frontend` also has an `Ingress`, which the AWS Load Balancer Controller turns into the ALB.

## Environments

| | Dev | Prod |
|---|---|---|
| Namespace | `cargotrack-dev` | `cargotrack-prod` |
| Helm values file | `values-dev.yaml` | `values-prod.yaml` |
| Tracks branch | `develop` | `main` |
| Image tag updates | on every dev build | on every prod build |

Both environments are managed by the same ArgoCD `root-app` App-of-Apps, defined in `cargotrack-gitops`.
