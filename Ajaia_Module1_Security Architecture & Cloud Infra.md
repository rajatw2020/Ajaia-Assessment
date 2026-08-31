# Ajaia 90-Day Cloud Security Hardening Plan

## 1. Executive Summary

Ajaia is an AI consultancy operating workloads across GCP and Azure. The environment currently has several high-risk security gaps involving excessive IAM privileges, exposed secrets, publicly reachable databases, weak endpoint and AKS controls, and inadequate backup and software supply-chain protections.

The focus over the next 90 days will be to reduce the overall security exposure, close the most critical gaps around sensitive data and access, strengthen workload and endpoint security, and improve our ability to recover from incidents.

## 2. Five Most Critical Security Risks

| Rank | Risk | Severity | Primary Remediation |
|---|---|---|---|
| 1 | Excessive GCP IAM and Owner-level service accounts | Critical | Replace Owner with least-privilege IAM roles, dedicated service accounts, IAM Conditions/PAM, group-based access, and eliminate service-account keys |
| 2 | Secrets and production credentials exposed through Git, Drive and Sheets | Critical | Rotate exposed credentials; migrate secrets to GCP Secret Manager / Azure Key Vault; replace personal SSH keys with identity-based access |
| 3 | Public Cloud SQL exposure including `0.0.0.0/0` authorized networks | Critical | Remove public IP access; use Cloud SQL Private IP and Private Services Access |
| 4 | Weak Azure/AKS and endpoint security | High/Critical | Entra Conditional Access, Intune, AKS RBAC, namespaces, Pod Security Standards, Azure Policy, network policies, and private AKS |
| 5 | Weak backups and container supply-chain controls | High | Enable PITR and stronger backup/recovery; version GCS objects; migrate GCR to Artifact Registry; scan and pin container images |

## 3. Risk 1 — Excessive GCP IAM

### Current State

- Project-level service accounts have Owner role.
- Developers and CI/CD potentially have excessive control over production resources.
- Service-account credentials create a large blast radius if compromised.

### Target State

Use Google Cloud IAM with least privilege:

- Replace Owner with predefined roles wherever possible.
- Use custom roles only when necessary.
- Use Google Groups for human access.
- Create separate service accounts for each workload and pipeline.
- Use IAM Conditions for context/time-bound access.
- Use Privileged Access Manager for temporary elevation where appropriate.
- Use Policy Simulator before privilege removal.
- Disable service-account key creation where practical.
- Eliminate long-lived service-account JSON keys.

### CI/CD

Cloud Build should use dedicated identities:

```text
Cloud Build
   |
   +-- Build Service Account
   |       |
   |       +-- Artifact Registry Writer
   |
   +-- Deployment Service Account
           |
           +-- Only required deployment permissions
```

Build and deployment identities should be separate.

## 4. Risk 2 — Secrets and Production Access

### Current State

- API keys are stored in `.env` files committed to repositories.
- Developer SSH keys are stored in shared Google Drive.
- Recovery keys are stored in a Google Sheet shared with employees.

Assume exposed credentials are compromised.

### Immediate Actions

1. Identify secrets in current repositories and Git history.
2. Revoke and rotate exposed credentials.
3. Remove secrets from source repositories.
4. Audit use of exposed credentials.
5. Rotate SSH keys.
6. Remove personal SSH keys from shared Drive.
7. Move recovery material into controlled administrative storage.

### GCP

Use Google Secret Manager:

```text
Application
    |
    v
Workload Identity / Service Account
    |
    v
Secret Manager
    |
    +-- Database credentials
    +-- API credentials
    +-- Third-party tokens
```

### Azure

Use Azure Key Vault for Azure workloads and AKS secrets.

### Production SSH

Replace:

```text
Developer -> Personal SSH Key -> Production VM
```

with:

```text
Developer
   |
   v
Entra / Google identity
   |
   v
IAP TCP forwarding / controlled Bastion
   |
   v
Production VM
```

For GCP VMs, Identity-Aware Proxy (IAP) TCP forwarding should be considered to provide identity-based and auditable SSH access without exposing SSH broadly.

## 5. Risk 3 — Public Cloud SQL Exposure

### Current State

Cloud SQL instances have public IP addresses and the authorized networks configuration includes:

```text
0.0.0.0/0
```

This creates unacceptable exposure, especially for the Harmoni database containing PHI.

### Target State

Use:

- Cloud SQL Private IP
- Private Services Access
- No public IP for production databases
- VPC firewall controls
- Restricted application-to-database paths

```text
GCP Shared VPC
      |
      +-- Application Subnet
      |       |
      |       v
      |   Application
      |
      +-- Private Services Access
              |
              v
          Cloud SQL
          Private IP
```

### Harmoni

Because Harmoni contains PHI:

- Enable automated backups.
- Enable Point-in-Time Recovery (PITR).
- Establish appropriate backup retention.
- Provide cross-region recovery where required.
- Consider CMEK based on contractual/compliance requirements.
- Enable deletion protection where appropriate.
- Regularly test database restoration.

## 6. Risk 4 — Azure, AKS and Endpoint Security

### Current AKS State

- Everything runs in the default namespace.
- No effective pod security controls.
- Five developers have cluster-admin.
- Weak workload segmentation.

### Target AKS State

```text
AKS
|
+-- ingress
|
+-- namespace: app-prod
+-- namespace: app-staging
+-- namespace: platform
+-- namespace: monitoring
```

Implement:

- Microsoft Entra integration.
- Azure RBAC / Kubernetes RBAC.
- Namespace-level access.
- Remove permanent developer cluster-admin.
- Pod Security Standards.
- Azure Policy for AKS.
- Network policies, preferably Cilium where appropriate.
- Default-deny ingress/egress.
- Workload Identity.
- Azure Key Vault integration.
- Private AKS API endpoint where appropriate.

### Endpoint Security

For company MacBooks:

- Enroll devices in Microsoft Intune.
- Enforce FileVault.
- Enforce OS patch levels.
- Enforce screen lock.
- Enforce firewall/security configuration.
- Use device compliance as an access condition.

For personal Linux devices:

- Do not allow unrestricted privileged production access.
- Prefer managed corporate devices for production administration.
- Alternatively use controlled cloud development/VDI environments or a hardened access gateway.

## 7. Risk 5 — Backup and Supply Chain

### Backup Weaknesses

Current:

- Harmoni: daily backups, 7-day retention, no PITR.
- General client data: weekly backups, 7-day retention.
- GCS deliverables: single-region, no versioning, no backup.
- Overall weekly backup to one unversioned GCS bucket.

### Target

Implement:

```text
Production Data
      |
      +-- Automated Backup
      +-- PITR where supported
      +-- Cross-region recovery
      +-- Versioning
      +-- Protected backup storage
      +-- Regular restore testing
```

For GCS:

- Enable object versioning where appropriate.
- Use appropriate retention policies.
- Separate production and backup projects/buckets.
- Use cross-region/dual-region storage where business requirements justify it.
- Protect backup administration from production compromise.

### Container Supply Chain

Current:

```text
Docker Hub
    |
    v
Cloud Build
    |
    v
GCR
```

Problems:

- `latest` is mutable.
- Docker Hub introduces uncontrolled third-party dependency.
- GCR should be migrated to Artifact Registry.
- Image provenance and vulnerability controls are insufficient.

Target:

```text
Source
  |
  v
Cloud Build
  |
  +-- Dependency scanning
  +-- Container vulnerability scanning
  +-- SBOM
  +-- Provenance / attestations
  |
  v
Artifact Registry
  |
  +-- Immutable version
  +-- Digest
  |
  v
Production
```

Use immutable version tags and preferably image digests rather than `latest`.

## 8. Target-State Network Security Architecture

### GCP Network

Replace the default auto-mode VPC with a custom-mode Shared VPC architecture.

```text
GCP Organization
|
+-- Networking Project
|     |
|     +-- Shared VPC
|            |
|            +-- Production Subnet
|            +-- Non-Production Subnet
|            +-- Management Subnet
|            +-- Security/Tools Subnet
|            +-- Private Services Access Range
|
+-- Production Project
+-- Non-Production Project
+-- Security/Logging Project
+-- CI/CD Project
```

Use non-overlapping CIDR ranges between GCP and Azure.

Example:

```text
10.0.0.0/16      GCP Shared VPC
10.0.0.0/20      Production
10.0.16.0/20     Non-production
10.0.32.0/20     Management
10.0.48.0/20     Security/tools
10.240.0.0/16    Private Services Access
```

These are illustrative ranges and should be finalized after validating existing routes and Azure address space.

### Azure Network

Use hub-and-spoke:

```text
Azure Subscription
|
+-- Hub VNet
|     |
|     +-- Azure Firewall
|     +-- VPN / ExpressRoute Gateway
|     +-- Bastion
|     +-- Private DNS
|
+-- Spoke VNet
      |
      +-- AKS
      +-- Private Endpoints
```

### Cross-Cloud Connectivity

Short term:

```text
GCP Cloud VPN
      ||
      || IPsec
      ||
Azure VPN Gateway
```

Use unique PSKs per tunnel. Do not reuse one PSK across all tunnels.

For future high-throughput/low-latency requirements, evaluate:

```text
GCP Cloud Interconnect
        +
Azure ExpressRoute
```

## 9. Identity Federation

### Target

Use Microsoft Entra ID as the enterprise identity source for cloud access where appropriate.

For human access to GCP:

```text
Employee
   |
   v
Microsoft Entra ID
   |
   | OIDC/SAML Federation
   v
Google Cloud Workforce Identity Federation
   |
   v
GCP IAM
```

For workloads accessing GCP from Azure, use Workload Identity Federation rather than long-lived GCP service-account keys.

Benefits:

- Centralized MFA.
- Centralized joiner/mover/leaver process.
- Group-based authorization.
- No long-lived cloud passwords.
- Reduced service-account key exposure.
- Centralized auditing.

## 10. Conditional Access

Implement Microsoft Entra Conditional Access:

```text
All Users
   |
   +-- Phishing-resistant MFA
   +-- Block legacy authentication
   +-- Require compliant device where appropriate
   +-- Risk-based controls
   +-- Separate administrator policies
```

Use Intune device compliance as an access signal for managed endpoints.

## 11. Secrets Migration Plan

### Phase 1 — Discover and Rotate

- Scan repositories and Git history.
- Inventory all secrets.
- Revoke/rotate exposed credentials.
- Identify applications and owners.

### Phase 2 — Centralize

GCP:

```text
Application -> Service Account -> Secret Manager
```

Azure:

```text
AKS/Application -> Workload Identity -> Key Vault
```

### Phase 3 — Remove Embedded Secrets

- Delete `.env` credentials from repositories.
- Remove credentials from CI/CD variables where possible.
- Remove shared Drive SSH keys.
- Remove recovery secrets from shared spreadsheets.
- Replace personal credentials with federated identities.

### Phase 4 — Prevent Recurrence

Implement:

- Secret scanning in CI/CD.
- Pre-commit secret detection.
- Artifact/image scanning.
- Least-privilege secret access.
- Secret rotation.
- Audit logging and alerting.

## 12. 90-Day Execution Plan

### Days 0–15 — Stop the Bleeding

- Remove Cloud SQL `0.0.0.0/0`.
- Remove unnecessary Cloud SQL public access.
- Rotate all exposed API keys.
- Rotate SSH credentials.
- Remove secrets from repositories.
- Remove developer cluster-admin.
- Inventory GCP Owner permissions.
- Disable unused identities.
- Audit relevant cloud logs.

### Days 15–30 — Identity

- Entra MFA.
- Conditional Access.
- Intune enrollment.
- GCP Workforce Identity Federation.
- IAM group model.
- Least-privilege service accounts.
- Eliminate service-account keys.
- Establish privileged administrator access.

### Days 30–60 — Network and Workloads

- Custom-mode Shared VPC.
- Production/non-production segmentation.
- GCP firewall controls.
- Azure hub/spoke.
- Unique VPN credentials.
- Private AKS.
- AKS namespaces and RBAC.
- Network policies.
- Azure Policy.
- Secret Manager and Key Vault migration.

### Days 60–90 — Resilience and Supply Chain

- Cloud SQL PITR.
- Improved backup retention.
- Cross-region recovery.
- GCS versioning/protection.
- Restore testing.
- Artifact Registry migration.
- Container vulnerability scanning.
- SBOM/provenance.
- CI/CD hardening.
- Centralized security monitoring.

## 13. Target Architecture Summary

```text
                           +-----------------------+
                           |   Microsoft Entra ID   |
                           | MFA + Conditional      |
                           | Access + Device Trust |
                           +-----------+-----------+
                                       |
                              Identity Federation
                                       |
                    +------------------+------------------+
                    |                                     |
                 GCP IAM                              Azure RBAC
                    |                                     |
       +------------+-----------+                 +-------+------+
       |                        |                 |              |
 GCP Shared VPC             CI/CD             Hub VNet         AKS
       |                        |                 |              |
 +-----+------+           Cloud Build            |        Namespaces
 |            |                |                 |        RBAC
Prod        Nonprod       Artifact Registry      |        Cilium
 |                             |                 |        Policy
 |                             |                 |
 +-- App ----------------------+            Azure Firewall
 |
 +-- Cloud SQL Private IP
 |
 +-- Secret Manager

       GCP Cloud VPN / Interconnect
                    ||
                    ||
             Azure VPN Gateway /
             ExpressRoute

Azure workloads
       |
       +-- Key Vault
       +-- Private Endpoints
       +-- Managed Identity / Workload Identity
```

## 14. Guiding Principle

The 90-day transformation should move Ajaia from:

```text
Broad IAM
+ Public databases
+ Shared credentials
+ Unmanaged endpoints
+ Flat Kubernetes
+ Weak backups
+ Mutable container images
```

to:

```text
Least privilege
+ Private data paths
+ Federated identity
+ Managed endpoints
+ Segmented workloads
+ Resilient backups
+ Trusted/immutable software supply chain
```

The architectural principle is:

> **Security should be based on explicit identity, least privilege and controlled trust paths—not on network location or shared credentials.**
