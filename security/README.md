# Security

Documentation of CargoTrack's security architecture and posture — separate from day-to-day infrastructure docs because it has a different audience (security review, compliance, audits) and a different lifecycle (should be revisited whenever identity, encryption, or network boundaries change).

## Suggested contents

- **Identity & access** — IAM roles, IRSA (per-pod AWS credentials), who/what can do what
- **Encryption** — what's encrypted at rest and in transit, key management
- **Network boundaries** — what's public vs. private, WAF rules, security groups
- **Secrets handling** — where secrets live, how they reach running pods, rotation policy
- **Threat monitoring** — what's watching for suspicious activity and where alerts go

Nothing has been written yet — add files here as the security documentation is authored.
