# Responsible Disclosure Approach

## Principles

All security research published in this repository was conducted under the following responsible disclosure principles:

### 1. Minimize Harm

- Use the least invasive technique that confirms the vulnerability.
- Prefer read-only operations over writes.
- Use invalid, non-existent, or researcher-controlled identifiers for proof-of-concept demonstrations whenever possible.
- Never perform destructive database operations.
- Never mass-collect or exfiltrate data beyond the minimum needed for proof.
- Rate-limit all requests to avoid availability impact.
- Operate at concurrency 1 — no parallel scanning, no stress testing.

### 2. No Third-Party Impact

- Never take over, access, or modify real users' accounts or sessions.
- Cross-session or cross-account testing is performed only between researcher-controlled sessions.
- If phone numbers, email addresses, or other PII are discovered, do not contact the individuals.
- Mask all PII in reports.

### 3. Complete Disclosure of Side Effects

- Document every persistent artifact created during testing — messages, records, transactions — with identifiers and locations.
- Request deletion of artifacts where appropriate.
- Disclose the exact number of records accessed, not an approximation.

### 4. Honest Scoring and Scope

- Compute CVSS scores from vectors, not from subjective estimation.
- When an initial assessment is not supported by evidence, test it and adjust. Document the reasoning.
- Do not claim scope beyond what was directly demonstrated.
- Document negative results (tested and not vulnerable) alongside positive findings.
- Explicitly list areas that were not tested to prevent assumptions about comprehensive coverage.

### 5. Direct Vendor Communication

- Deliver detailed reports directly to the affected organization.
- Propose a standard 90-day disclosure timeline.
- Offer to re-test after remediation.
- Offer to provide additional technical detail on request.
- Do not publish findings while the vendor is actively remediating.

### 6. Public Disclosure (This Repository)

When publishing case studies:

- **Never** include company names, domains, IP addresses, or other identifying information.
- **Never** include working exploit payloads, commands, or reproduction steps.
- **Never** include credentials, keys, tokens, or secrets.
- **Never** include customer PII or financial data.
- Present findings at a level of abstraction that demonstrates methodology and technical depth without enabling replication against the original target.
- Do not claim outcomes (remediation, bug bounty payments, employment) that are not confirmed.

## Timeline Standard

The disclosure timeline used for the research in this repository follows industry conventions:

| Event | Timing |
|-------|--------|
| Vulnerability discovered | Day 0 |
| Vendor notified with full report | Day 0–1 |
| Vendor response expected | Within 14 days |
| Remediation expected | Within 90 days |
| Public disclosure (anonymized) | After 90 days, or after confirmed remediation |
| Escalation (if no response) | After 14 days of no contact |

Urgent findings (exposed credentials, live financial impact) are flagged for immediate action in the report, with separate priority timelines recommended.
