# Cybersecurity Research Portfolio

Independent security research case studies, anonymized for public reference.

## Purpose

This repository presents selected findings from independent security research conducted against production web applications and APIs. Each case study has been thoroughly anonymized to protect the identity of the assessed organizations while preserving the technical substance of the work.

The goal is to demonstrate:

- Practical vulnerability discovery and validation methodology
- Understanding of web, API, and infrastructure security
- Rigorous responsible disclosure practices
- Commitment to safety and minimal impact during testing
- Clear, structured technical reporting

## Research Areas

- **Web Application Security** — Authentication, authorization, session management, input validation
- **API Security** — REST and WebSocket endpoint testing, authentication enforcement, parameter validation
- **Multi-Tenant Security** — Tenant isolation, cross-tenant data leakage, identity boundary enforcement
- **SQL Injection** — Detection, controlled exploitation for impact assessment, privilege analysis
- **Access Control** — IDOR / broken object-level authorization, sequential identifier enumeration
- **Financial System Security** — Payment initiation controls, transaction authorization, mobile money integration security
- **Session Security** — Token predictability, session binding, cross-session exploitation
- **Cryptographic Key Management** — Detection of exposed signing keys, credential storage assessment
- **Client-Side Security** — Credential exposure in JavaScript, client-side filtering bypass

## Case Studies

| # | Case Study | Platform Type | Key Findings |
|---|-----------|---------------|--------------|
| 1 | [Online Gaming Platform](case-studies/gaming-platform-security-research.md) | Multi-tenant gaming / betting platform | SQL injection (CVSS 9.8), predictable session tokens, exposed government signing keys, cross-tenant data leakage, plaintext passwords |
| 2 | [Payment Platform](case-studies/payment-platform-security-assessment.md) | Mobile money payment platform | Unauthenticated payment initiation, multiple IDOR vulnerabilities, KYC data exposure, client-side credential exposure |

## Methodology

Detailed methodology documentation:

- [Web & API Security Testing](methodology/web-api-testing.md) — Endpoint discovery, authentication testing, input validation, session analysis, multi-tenant testing
- [Authorization Testing](methodology/authorization-testing.md) — Authentication presence, IDOR, tenant isolation, session management, financial authorization
- [Responsible Disclosure](methodology/responsible-disclosure.md) — Disclosure principles, timeline standards, public disclosure guidelines

## Safety Principles

Every assessment in this repository was conducted under strict safety constraints:

1. **Minimal footprint** — Use the least invasive technique that confirms the finding.
2. **No destructive operations** — No data modification, deletion, or corruption.
3. **No third-party impact** — Cross-account testing only between researcher-controlled sessions.
4. **No credential harvesting** — Sensitive values are counted, measured, or fingerprinted — never read.
5. **No bulk collection** — Only the minimum records needed for proof.
6. **Complete artifact disclosure** — Every side effect of testing is documented.
7. **Honest scoring** — Claims that cannot be evidenced are withdrawn, not inflated.

## Responsible Disclosure

All research was reported to the affected organizations through coordinated / responsible disclosure:

- Detailed written reports delivered directly to the vendor
- 90-day standard disclosure timelines proposed
- Researcher available for re-testing after remediation
- No public disclosure during active remediation

## Disclaimer

> **Company and client identities are intentionally omitted.** All organization names, domains, IP addresses, infrastructure details, credentials, customer data, and exploit reproduction steps have been removed or replaced with generic labels. This is deliberate and permanent. The purpose of this repository is to demonstrate security research methodology and responsible disclosure practice — not to expose any organization or enable attacks.
>
> **No information in this repository should be used for unauthorized access to any system.** The case studies are published for educational and professional reference purposes only.
>
> **CVSS scores in these case studies are reproduced from the original reports** and were computed from their CVSS 3.1 vectors. They are not independently assigned by any third-party authority and no CVE identifiers exist for these findings.

## Contact

For questions about this research, responsible disclosure inquiries, or content concerns, please see [SECURITY.md](SECURITY.md).

---

*This repository is maintained as a professional reference for independent cybersecurity research.*
