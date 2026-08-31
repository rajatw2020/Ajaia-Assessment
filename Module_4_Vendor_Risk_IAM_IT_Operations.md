# Module 4: Vendor Risk, IAM & IT Operations

## Part A: Vendor Security Assessment

### 1. Vendor Gaps and Questions

I would not approve the vendor based on the current information. Several items need evidence or clarification.

| Area | Gap / Concern | Follow-up |
|---|---|---|
| SOC 2 Type II | Only a claim; no report provided | Request the latest SOC 2 report, scope, period and exceptions. |
| Encryption | AES-256/TLS 1.2 stated, but implementation is unclear | Ask about key management, rotation and TLS configuration. |
| Model training | "We do not train on customer data" is only a statement | Get this commitment in the contract/BAA and confirm technical enforcement. |
| Data location | AWS us-east-1 only | Confirm data residency, backups, disaster recovery and any subprocessors in other regions. |
| BAA | Vendor says they will sign one | Review and execute the BAA before PHI is processed. |
| Pen test | Last test was in 2023 and performed internally | Request the report and remediation status; require recent independent testing. |
| 90-day logging | Inputs/outputs containing PHI may remain for 90 days | Ask why 90 days is necessary and whether retention can be reduced or disabled. |
| Subprocessors | Not mentioned | Request complete subprocessor list and notification process for changes. |
| Incident response | Not provided | Request IR plan, breach notification commitments and escalation contacts. |
| Access control | Not provided | Ask about RBAC, privileged access, MFA and access reviews. |
| Vulnerability management | Not provided | Ask about scanning, patching and remediation SLAs. |
| Business continuity | Not provided | Request DR strategy, backup approach, RPO/RTO and testing frequency. |
| Data deletion | Not addressed | Confirm deletion process, timelines and deletion from backups. |
| Compliance | Only SOC 2 mentioned | Ask about HIPAA controls, recent audits and relevant certifications/attestations. |

### Questions I would send the vendor

Before approval, I would ask for:

1. Latest SOC 2 Type II report and any exceptions.
2. Independent penetration-test report and remediation status.
3. Current architecture and data-flow diagram.
4. Complete subprocessor list and AWS regions used.
5. BAA and security/privacy terms.
6. Exact retention and deletion process for prompts, outputs and logs.
7. Confirmation that customer data is not used for model training.
8. Encryption and key-management details.
9. IAM, MFA and privileged-access controls.
10. Incident-response process and breach notification timeline.
11. RPO/RTO and disaster-recovery testing results.
12. Vulnerability-management and patching SLAs.

### Engineering says: "We just need your sign-off."

I would not approve it simply because the integration is already built.

My response would be:

> "The integration being complete doesn't change the security review requirement. We need to complete the vendor assessment before PHI is sent to the service."

I would allow the engineering team to continue testing with **synthetic/non-PHI data**, but production access to Harmoni PHI would remain blocked until the required security, privacy/legal and contractual checks are complete.

---

# Part B: Access Control Redesign

## 3. RBAC Model

I would move away from individual admin access and introduce **role-based, least-privilege access** with SSO and MFA.

### Access Tiers

| Tier | Purpose | Example |
|---|---|---|
| **T0 – Basic** | Normal employee access | Email, Slack, approved SaaS |
| **T1 – Developer** | Development environments | Dev GCP, GitHub, CI/CD |
| **T2 – Production Operator** | Controlled production operations | Production deployment/read access |
| **T3 – Privileged Admin** | Infrastructure/security administration | IAM, networking, security |
| **T4 – Break Glass** | Emergency access only | Full administrative recovery |

### RBAC Matrix

| Role | Google Workspace | GCP Dev | GCP Prod | GitHub | AKS | Security |
|---|---|---|---|---|---|---|
| Employee | Standard | None | None | None | None | None |
| Developer | Standard | Editor/Developer | Read-only | Write to assigned repos | Namespace-level | None |
| Senior Developer | Standard | Admin/Developer | Limited deployment | Write + PR approval | Namespace admin | None |
| DevOps | Standard | Admin | Deployment/Operations | Admin for assigned repos | Cluster operations | Limited |
| Security | Standard | Security/IAM audit | Security/audit | Security settings | Audit | Admin |
| Platform Admin | Standard | Platform Admin | Platform Admin | Org/Repo Admin | Cluster Admin | Limited |
| Executive | Standard | None | None | None | None | None |

### Additional Controls

- Enforce **SSO through Entra ID** for GitHub and supported SaaS.
- Enforce MFA, preferably phishing-resistant MFA for privileged roles.
- Eliminate personal GitHub accounts.
- Use company-managed GitHub identities only.
- Remove unnecessary Google Workspace admin privileges.
- Replace individual GCP permissions with groups.
- Review production access regularly.
- Use temporary elevation for privileged operations.
- Maintain a separate break-glass account with strong controls and monitoring.
- Immediately revoke access during employee offboarding.

### Immediate Priority

The three employees using personal GitHub accounts are a **high-priority cleanup item**. Their personal accounts are connected to other organizations, creating an unnecessary ownership and access risk.

I would migrate them to company-managed accounts, remove the personal identities from Ajaia repositories, review their existing permissions, and rotate any credentials or deploy keys they controlled.
