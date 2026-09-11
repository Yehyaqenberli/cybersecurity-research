# Security Research Case Study: Online Payment Platform

## Overview

An independent security assessment was conducted against a production payment platform serving a large number of registered merchants. The platform provided payment initiation, wallet management, merchant onboarding, invoicing, and reporting capabilities through a REST API. Multiple critical and high-severity vulnerabilities were identified, the most critical being an unauthenticated endpoint capable of initiating real financial transactions on behalf of any registered merchant.

## Researcher Role

Independent security researcher. Coordinated / responsible disclosure.

## Disclosure

- **Timeline:** Coordinated disclosure initiated in 2026.
- **Format:** Detailed written report delivered directly to the platform operator, with a meeting scheduled for technical discussion.
- **Standard:** 90-day responsible disclosure timeline proposed.
- **Status:** Initial contact made and response received. Remediation status not confirmed at time of writing.

## Findings Summary

| # | Finding | Severity |
|---|---------|----------|
| 1 | **Unauthenticated Payment Initiation** — A POST endpoint accepted payment requests with no authentication. The endpoint triggered real financial transactions, capable of impersonating any registered merchant. Predictable account identifiers enabled enumeration of the full merchant base. | Critical |
| 2 | **Unauthenticated Wallet Top-Up Initiation** — A separate POST endpoint allowed initiating wallet funding requests with no authentication, using the same predictable identifier pattern. | Critical |
| 3 | **KYC Data IDOR** — A GET endpoint exposed merchant profile and identity verification data, including third-party payment credentials, with no authentication. Predictable identifiers enabled full enumeration of all registered merchants. | High |
| 4 | **Payment Channel IDOR** — A GET endpoint exposed payment channel configurations including financial routing details, with no authentication. This endpoint also provided the technical identifiers required to exploit the payment initiation vulnerability. | High |
| 5 | **Invoice IDOR / Customer PII Exposure** — A GET endpoint exposed invoice records including customer names, contact information, and transaction descriptions, with no authentication. | High |
| 6 | **Unauthenticated Reputation Abuse** — A POST endpoint accepted merchant reports with no authentication or rate limiting, enabling mass false-flagging of merchants. | Medium |
| 7 | **Client-Side Credential Exposure** — API keys and secrets were embedded in client-side JavaScript source code, visible to any visitor. | Medium |

## Impact

- **Financial:** An unauthenticated attacker could initiate real financial transactions impersonating any registered merchant, reaching a live production payment system. Predictable account identifiers meant the entire merchant base was addressable.
- **Privacy:** Merchant identity verification data, third-party payment credentials, financial routing details, and customer PII were accessible without authentication. This exposure potentially triggered obligations under applicable data protection regulation.
- **Merchant Trust:** The ability to mass-report merchants without authentication could have been used to damage merchant reputations at scale.
- **Credential Compromise:** API keys and secrets embedded in client-side code were exposed to any web visitor.

## Methodology

1. **Endpoint Discovery** — Identified the API structure and endpoint patterns through examination of the web application's client-side code and network requests.
2. **Authentication Testing** — Evaluated authentication enforcement across each API endpoint, discovering that critical financial and data endpoints required no authentication.
3. **Authorization and Object-Level Access Testing** — Tested predictable identifiers across multiple endpoint groups, confirming that account boundaries were not enforced and that the full range of registered accounts was enumerable.
4. **Payment Flow Validation** — Confirmed that the payment initiation endpoint reached a live production payment system by examining the response format, which matched production transaction identifiers.
5. **Control Testing** — Identified endpoints that were properly protected (wallet queries, transaction history, invoice modification, API key creation), documenting that authentication mechanisms existed in the application but were not applied to the vulnerable endpoints. Also confirmed that cross-account resource validation existed for certain operations.
6. **Error Response Analysis** — Observed that invalid parameters returned internal error details, confirming direct database queries against production data.
7. **Client-Side Code Review** — Identified hardcoded API credentials in the application's JavaScript source code.
8. **Attack Chain Mapping** — Documented how the IDOR vulnerabilities provided the exact technical identifiers needed to exploit the payment initiation vulnerability, forming a complete attack chain from reconnaissance to financial impact.

## Safety and Scope Discipline

The following constraints were explicitly maintained throughout testing, as documented in the original report:

- **No real financial transactions completed.** Payment testing used safe test parameters to prevent actual fund movement while still confirming that the endpoint reached the live payment system.
- **No customer data harvested.** Only sample records were examined for verification purposes; no mass data collection was performed.
- **Read-only operations where possible.** Data enumeration was conducted through read-only requests where feasible.
- **Rate-limited requests.** Requests were rate-limited to avoid service disruption.
- **No persistent changes.** No persistent modifications were made to any system.
- **Scope gaps documented.** The report explicitly identified untested areas to prevent overclaiming.

## Remediation Recommendations

1. **Add authentication to all payment and financial endpoints.** No financial operation should be callable without a verified session.
2. **Implement object-level authorization.** After authentication, verify that the authenticated user owns the account being operated on.
3. **Validate resource relationships server-side.** Ensure that payment resources belong to the requesting account before processing operations.
4. **Add rate limiting** per account and per IP address on all endpoints, especially financial operations.
5. **Implement audit logging** for all payment initiation attempts, successful or not.
6. **Rotate all exposed client-side credentials immediately** and move secrets to server-side environment variables.
7. **Never embed API keys or secrets in client-side JavaScript.**
8. **Add authentication and rate limiting to reporting endpoints** to prevent mass abuse.
9. **Suppress internal error details.** Internal error messages should never be returned to API consumers.

## Security Lessons

1. **Authentication is not optional for financial endpoints.** An unauthenticated payment initiation endpoint on a production payment system is an immediate critical risk. Every endpoint that triggers real-world financial effects must require verified identity.

2. **Predictable identifiers make enumeration trivial.** When account identifiers are predictable, a single endpoint without authentication exposes the entire user base. Use non-predictable identifiers for all external-facing references.

3. **IDOR vulnerabilities chain together.** Individually, each IDOR exposed different data. Together, they formed a complete attack chain: enumerate merchants, discover payment configurations, initiate payments. Defense must address the pattern, not just individual endpoints.

4. **Negative testing reveals the inconsistency.** Several endpoints in the same application correctly enforced authentication, proving the platform had the capability. The vulnerability was not a missing feature — it was inconsistent application of an existing control.

5. **Client-side secrets are public secrets.** Any credential embedded in JavaScript delivered to the browser is compromised the moment the page loads. There is no client-side secret.

6. **Controlled proof-of-concept protects everyone.** Using safe test parameters confirmed the vulnerability reached a live production system without causing financial harm — demonstrating that responsible validation is possible even for critical financial flaws.
