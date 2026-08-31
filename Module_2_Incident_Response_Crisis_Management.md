# Module 2: Incident Response & Crisis Management

## 1. Initial Assessment

I would treat this as **two concurrent security incidents**:

1. **GCP service-account compromise** — Critical, because the credential was exposed publicly and was used to access Harmoni's PHI.
2. **Stolen employee laptop** — High, because the device was stolen while unlocked and had active access to Slack, Google Drive and Harmoni-related information.

I would immediately involve the security, privacy/legal and relevant business teams. For Harmoni, I would treat this as a **potential HIPAA breach** until the formal assessment is completed.

---

## 2. Immediate Containment

### GCP

- Revoke the compromised service-account key immediately.
- Identify other systems and secrets accessible through that account.
- Rotate any related credentials.
- Restrict the suspicious IP where appropriate.
- Preserve Cloud Audit Logs and Cloud SQL logs before making major changes.
- Do not delete the service account itself until its dependencies are understood.

### Stolen Laptop

- Revoke the employee's active Entra and Google sessions.
- Force reauthentication and reset credentials if required.
- Revoke Slack sessions.
- Review Google Drive and cloud activity from the account.
- If MDM were available, remotely lock or wipe the device.

FileVault provides protection for the encrypted disk, but because the laptop was stolen while unlocked, I would not assume the data was protected.

---

## 3. Investigating the GCP Incident

I would build a timeline around the compromised credential and investigate:

- Cloud SQL queries and records accessed.
- Secret Manager and other GCP resources accessed.
- IAM or configuration changes.
- Application/API logs.
- VPC Flow Logs and unusual outbound traffic.
- GCS and BigQuery activity.

The 14 Harmoni queries are particularly important. I would determine **which PHI was accessed, how many records were involved, and whether there is evidence that the results were exfiltrated**.

The lack of a GCS or BigQuery export does not prove that no data was taken, because the attacker may have accessed data through the application layer.

---

## 4. GitHub and Azure Investigation

The service-account key was publicly available for four days, so I would assume it was compromised regardless of whether we can prove it was downloaded.

I would check:

- When the key was committed.
- Whether it appeared in Git history or forks.
- Whether the suspicious activity started after the exposure.

The Azure login attempt from the same IP is also significant. I would correlate the IP and timestamps with Entra sign-in logs to determine whether the GCP and Azure activity could be related.

---

## 5. HIPAA Assessment

I would not immediately tell the client that a HIPAA breach has occurred.

Instead, I would report:

> **We have confirmed unauthorized access to an environment containing PHI and are conducting a formal HIPAA breach assessment.**

The assessment should determine:

- What PHI was accessed.
- How many individuals may be affected.
- Whether the information was actually acquired or exfiltrated.
- What mitigation actions were taken.
- Whether notification requirements are triggered.

All findings and decisions should be documented.

---

## 6. Executive Communication

My initial executive update would be short and factual:

> We are investigating a critical security incident involving a GCP service-account credential that was exposed in a public GitHub repository and subsequently used from an unauthorized location.
>
> We have confirmed 14 queries against the Harmoni database, including tables containing patient symptom descriptions and provider notes. We have not identified a Cloud Storage or BigQuery export, but application-layer exfiltration is still being investigated.
>
> Separately, an employee reported a stolen laptop that was unlocked and had access to Harmoni-related systems. We have started containment of both incidents and are preserving relevant evidence.
>
> Because PHI may have been accessed, we are engaging the appropriate privacy/legal process for a formal HIPAA breach assessment.
