# Module 5: AI-Native Security & Operations

## Part A: My AI Workflow

I use AI mainly as a productivity tool, while keeping the final technical decision and validation with myself.

| Task | Tool / Model | Workflow | What I Don't Trust AI To Do | Time Saving |
|---|---|---|---|---|
| Terraform development | GitHub Copilot / current Copilot model | Give it the infrastructure requirement → generate Terraform → review and test the code → apply through CI/CD | I don't trust it to make security, IAM or production architecture decisions without review. | ~40–50% |
| Troubleshooting / technical analysis | ChatGPT / current GPT model | Provide sanitized error/output → analyze possible causes → compare suggestions with documentation/logs → implement and test | I don't rely on it as the final source of truth. | ~30–40% |
| Documentation | ChatGPT / current GPT model | Provide technical notes → structure and improve the document → review and edit the final version | I don't allow it to invent architecture decisions, controls or compliance statements. | ~50–60% |

**Important:** I never put client PHI, credentials, API keys or other sensitive information into a general-purpose AI tool unless the use has been specifically approved and the appropriate data-processing controls are in place.

---

# Part B: AI Risk & Governance

## 2. AI Risk Register

| Risk | Likelihood | Impact | Current Control | Recommended Mitigation |
|---|---|---|---|---|
| Client code/PHI pasted into unapproved AI tools | H | H | Approved-tool list | Block unapproved tools, DLP/monitoring, and train developers on data handling. |
| AI output contains insecure or incorrect code | M | H | Developer review | Mandatory code review, testing, SAST and security scanning. |
| Sensitive information retained in AI prompts/history | M | H | Limited | Use approved enterprise plans, defined retention, and strict data-classification rules. |
| Hallucinated technical/compliance advice | M | M | Human review | Require validation against official documentation and security review for important decisions. |
| AI-generated code introduces supply-chain vulnerabilities | M | H | CI/CD controls | Dependency scanning, SAST, container scanning and approved libraries. |
| Shadow AI / unauthorized AI vendors | H | M/H | Expense monitoring | Maintain an approved-tool register and periodically review SSO, endpoint and expense data. |
| AI vendor changes data usage or retention terms | M | H | Vendor review | Periodic vendor reassessment and contractual notification requirements. |

### Developer using ChatGPT for client code

The statement *"OpenAI doesn't train on API inputs"* does not by itself make the practice acceptable.

The question is also **whether the developer is using an approved service, what information is being shared, what retention and contractual terms apply, and whether the client has authorized that processing**.

My immediate action would be to stop the practice, determine what data was shared, and review whether any client information needs to be treated as an incident.

---

# 3. AI Tool Governance Framework

## Tool Approval

Before a new AI tool is used with company or client information:

1. Business owner submits the request.
2. Security reviews the tool and use case.
3. Privacy/legal reviews client or regulated data use.
4. Vendor security and contractual terms are checked.
5. Tool is assigned a risk tier.
6. Approved tools are added to the AI register.

## Risk Tiers

| Tier | Example | Approval |
|---|---|---|
| **Low** | Public information / general productivity | Standard approval |
| **Medium** | Internal company information or source code | Security approval |
| **High** | PHI, student records or sensitive client data | Security + Privacy/Legal + contractual review |
| **Prohibited** | Unapproved processing of regulated/client data | Not permitted |

## Data Rules

- **Public:** May be used with approved or public AI tools.
- **Internal:** Use approved tools only.
- **Confidential:** Approved enterprise tools with appropriate controls.
- **Restricted:** PHI, student records, credentials and sensitive client data require explicit approval and contractual safeguards.

## Monitoring

I would monitor for shadow AI through:

- Expense/procurement records.
- SSO and identity logs.
- Endpoint/DNS controls where appropriate.
- DLP/CASB capabilities.
- Periodic review of AI vendors and licenses.

The goal is **visibility rather than employee surveillance**.

## Handling Violations

The response should depend on the severity:

- First-time/low-risk issue → education and correction.
- Repeated violation → formal management action.
- Sensitive client or regulated data exposure → incident response and privacy/legal review.

### Communication to the Team

I would position the policy as a **safe-use framework, not an anti-AI policy**.

The message would be:

> "We want everyone to use AI. The purpose of these rules is to make sure we can use it safely without exposing our clients, our code or regulated data. If you're unsure whether a tool or dataset is allowed, ask Security before using it."

This encourages people to report mistakes early instead of hiding them.
