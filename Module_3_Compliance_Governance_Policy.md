# Module 3: Compliance Governance & Policy

## Part A — Compliance Program Design

### Can Ajaia use one compliance system?

Yes. I would create **one security and compliance program** and map HIPAA, FERPA and SOC 2 requirements to the same set of core controls. I would not run three separate security programs.

### Where they overlap

The main common areas are:

- Access control and least privilege
- Encryption and data protection
- Logging and monitoring
- Risk management
- Incident response
- Vendor management
- Policies, training and documentation

### Where they differ

- **HIPAA:** Protects PHI and has specific privacy, security and breach-notification requirements.
- **FERPA:** Protects student education records and controls access, disclosure and redisclosure.
- **SOC 2 Type I:** Provides assurance that relevant controls are suitably designed at a specific point in time.

### Recommendation

Use one Ajaia control framework with separate compliance mappings:

**One program → common controls → HIPAA / FERPA / SOC 2 mappings.**

This keeps the program manageable while still covering requirements specific to healthcare, education and customer assurance.

---

# Part B — PHI Determination

## 2. Memo to CEO — Harmoni / Claude API

**Subject: HIPAA applicability to Harmoni API data**

The statement that HIPAA does not apply because names and DOBs are removed is **not necessarily correct**.

Symptoms and medication information are health information. Removing names and DOBs does not automatically make the data de-identified.

Under **45 CFR §164.514**, the HIPAA Safe Harbor method requires removal of all specified identifiers and that Ajaia has no actual knowledge that the remaining information could identify the individual. The alternative is Expert Determination.

Therefore, I would not treat the Claude API data as de-identified without a documented assessment.

If the data remains PHI and the AI provider is acting as a business associate, the appropriate BAA and contractual safeguards are required.

**Recommendation:** Complete a formal de-identification assessment, minimize the data sent to the API, confirm the provider's data handling, and ensure the required BAA is in place before sending PHI.

---

## 3. Ethos / FERPA

Sending student performance data to OpenAI does **not automatically violate FERPA**.

As a designated school official, Ajaia must:

- Use the data only for the agreed educational purpose.
- Remain under Ethos's direct control.
- Prevent unauthorized disclosure or redisclosure.
- Limit access to people who need it.
- Have appropriate security controls, including encryption and logging.
- Define retention and deletion requirements.
- Ensure the AI provider's use of the data is consistent with Ethos's FERPA obligations.

This should also be clearly documented in the agreement with Ethos and applicable vendor agreements.

---

# Part C — Policy Gap Analysis

## 4. Incident Response Policy

| Current Policy | Gap |
|---|---|
| All incidents contained within 4 hours | Too rigid; use severity-based response targets. |
| CEO notified within 1 hour | Does not define Security, Privacy/Legal or system-owner escalation. |
| Client notification within 96 hours | Must align with HIPAA and contractual requirements. HIPAA requires a business associate to notify the covered entity without unreasonable delay and no later than 60 days after discovery of a breach — the BAA may require faster notification. |
| Post-incident review within 7 days | Should be based on incident severity and complexity. |
| No evidence preservation | Add evidence collection, preservation and access controls. |
| No forensic procedures | Define investigation and evidence-handling procedures. |
| No breach assessment process | Add a formal HIPAA breach-risk assessment process. |
| No severity levels | Define P1/P2/P3 or equivalent escalation and response requirements. |

## Data Classification Policy

The most immediate concern is:

> **Student education records are classified as Internal.**

I would change this to **Confidential/Restricted**. Treating FERPA-protected records as ordinary Internal data could result in weaker access and sharing controls than required.

Other gaps:

- No handling rules for each classification.
- No retention/deletion requirements.
- No access-review requirements.
- No third-party sharing requirements.
- No data-owner responsibilities.
- No specific controls for highly sensitive data.
- Encryption requirement is too general and should define protection in transit and at rest.

### Recommended classification

**Public → Internal → Confidential → Restricted**

Restricted should include **PHI, student education records, credentials/secrets and other highly sensitive client data**.

### Overall Recommendation

I would keep the governance model simple:

> **One Ajaia security framework, common controls, and separate HIPAA, FERPA and SOC 2 compliance mappings.**
