# Module 2: Incident Response & Crisis Management

## 1. First 60 Minutes

I would treat the GCP compromise as the **priority incident**, because there is confirmed unauthorized access to the Harmoni database and potentially PHI.

### 0–15 minutes

- Declare a critical security incident.
- Revoke the compromised service-account key.
- Preserve Cloud Audit and Cloud SQL logs.
- Check where the service account is being used before disabling it.
- Notify security and privacy/legal teams.
- Start an incident timeline.

### 15–30 minutes

- Review the 14 Harmoni queries and identify the data accessed.
- Check GCS, BigQuery, Secret Manager and IAM activity.
- Review the suspicious IP and correlate it with the Azure login attempt.
- Check application and network logs for possible data exfiltration.

### 30–60 minutes

- Establish the initial scope and timeline.
- Rotate any related credentials.
- Preserve the GitHub evidence showing the exposed key.
- Continue investigating whether the queried PHI left the environment.
- Prepare an initial executive update.

The objective in the first hour is **containment first, followed by establishing what was accessed and whether there is evidence of data loss.**

---

## 2. Stolen Laptop — 3:30 AM

I would handle this as a **second high-priority incident**, but it should not delay containment of the active GCP compromise.

Immediately:

- Revoke the employee's Entra ID, Google and Slack sessions.
- Force reauthentication/reset credentials where necessary.
- Review Google Drive and cloud activity.
- Determine what Harmoni data was available through the laptop.
- If MDM were available, remotely lock or wipe the device.

Although FileVault was enabled, the laptop was stolen while **unlocked and authenticated**. I would therefore not assume that encryption eliminates the risk.

---

## 3. HIPAA Assessment

Under **45 CFR §164.402**, unauthorized access, acquisition, use or disclosure of PHI is presumed to be a breach unless the organization can demonstrate a **low probability that the PHI was compromised**, based on a documented risk assessment.

### GCP Service Account

**Recommendation: Treat as a likely reportable breach.**

We have confirmed:

- Unauthorized use of the credential.
- 14 queries against Harmoni.
- Access to tables containing symptom descriptions and provider notes.
- Potential application-layer exfiltration has not been ruled out.

The fact that we cannot prove the data was downloaded does not remove the need for a breach assessment. I would notify Harmoni according to the BAA and continue the investigation.

Under **45 CFR §164.410**, a business associate must notify the covered entity without unreasonable delay and no later than 60 days after discovery of a breach, subject to any shorter contractual requirement in the BAA.

### Stolen Laptop

**Recommendation: Keep under investigation; do not immediately classify as a reportable breach.**

FileVault is relevant because properly implemented encryption can render PHI unreadable and therefore outside the definition of unsecured PHI.

However, because the laptop was stolen while unlocked, I would first establish whether the attacker accessed Harmoni data through the active session.

---

## 4. CEO Question — 9:00 AM

> *"Can we just not tell the healthcare client? We don't know for sure data was taken, and the laptop was encrypted."*

I would not recommend that.

For the GCP incident, we already know an unauthorized party used the compromised credential and accessed the Harmoni database, including tables containing PHI. HIPAA does not require us to prove that the attacker definitely downloaded the data before we perform the breach assessment. We need to be able to demonstrate a low probability that the PHI was compromised if we decide notification is not required.

The laptop is a separate matter. FileVault is helpful, but the device was stolen while it was unlocked and authenticated, so we need to establish whether PHI was accessible through the active session.

My recommendation is to notify Harmoni about the GCP incident in accordance with the BAA, provide the facts we have confirmed, and continue the forensic investigation. We can provide further updates as we establish the full scope.

I would rather give the client an early, factual notification than delay and later have to explain why we knew about unauthorized PHI access but chose not to tell them.

---

## Bottom Line

**GCP incident:** likely reportable → notify Harmoni and continue investigation.

**Laptop:** investigate first → encryption may prevent notification, but the unlocked session needs to be assessed.

**Priority:** contain the GCP compromise immediately, revoke laptop access in parallel, preserve evidence, and involve privacy/legal early.
