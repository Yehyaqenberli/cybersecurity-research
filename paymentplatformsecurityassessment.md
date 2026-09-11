# Security Research Case Study: Payment Platform

## Overview

An independent security assessment was conducted against a production payment platform serving thousands of registered merchants. The platform provided mobile money payment initiation, wallet management, merchant onboarding (KYC), invoicing, and reporting capabilities through a REST API. Seven distinct vulnerabilities were identified, the most critical being an unauthenticated endpoint capable of initiating real financial transactions through a national mobile money network on behalf of any registered merchant.

## Researcher Role

Independent security researcher. Coordinated / responsible disclosure.

## Disclosure

- **Timeline:** Coordinated disclosure initiated approximately late 2026.
- **Format:** Detailed written report delivered directly to the platform operator, with a meeting scheduled for technical discussion.
- **Standard:** 90-day responsible disclosure timeline proposed.
- **Status:** Initial contact made and response received. Remediation status not confirmed at time of writing.

## Findings Summary

| # | Finding | Severity |
|---|---------|----------|
| 1 | **Unauthenticated Payment Initiation** — A POST endpoint accepted payment requests with no authentication header, token, or session. The endpoint triggered real transactions through a national mobile money API, capable of impersonating any of thousands of registered merchants. Sequential integer account identifiers enabled enumeration of the full merchant base. | Critical |
| 2 | **Unauthenticated Wallet Top-Up Initiation** — A separate POST endpoint allowed initiating wallet funding requests with no authentication, using the same sequential account identifier pattern. | Critical |
| 3 | **KYC Data IDOR** — A GET endpoint exposed merchant profile and identity verification data, including third-party payment service credentials, with no authentication. Sequential account identifiers enabled full enumeration of all registered merchants. | High |
| 4 | **Payment Channel IDOR** — A GET endpoint exposed payment channel configurations including mobile money short codes and bank account numbers, with no authentication. This endpoint also provided the technical identifiers required to exploit the payment initiation vulnerability. | High |
| 5 | **Invoice IDOR / Customer PII Exposure** — A GET endpoint exposed invoice records including customer names, email addresses, phone numbers, and transaction descriptions, with no authentication. | High |
| 6 | **Unauthenticated Reputation Abuse** — A POST endpoint accepted merchant reports with no authentication or rate limiting, enabling mass false-flagging of merchants as fraudulent. | Medium |
| 7 | **Client-Side Credential Exposure** — API keys and secrets were embedded in client-side JavaScript source code, visible to any visitor. | Medium |

## Impact

- **Financial:** An unauthenticated attacker could initiate real mobile money transactions impersonating any registered merchant, reaching a live national payment network. The sequential nature of account identifiers meant the entire merchant base was addressable.
- **Privacy:** Merchant KYC data, third-party payment credentials, bank account numbers, and customer PII (names, emails, phone numbers) were accessible without authentication. This exposure potentially triggered obligations under national data protection legislation.
- **Merchant Trust:** The ability to mass-report merchants as fraudulent without authentication could have been used to damage merchant reputations at scale.
- **Credential Compromise:** API keys and secrets embedded in client-side code were exposed to any web visitor.

## Methodology

1. **Endpoint Discovery** — Identified the API structure and endpoint patterns through examination of the web application's client-side code and network requests.
2. **Authentication Testing** — Systematically tested each API endpoint for authentication requirements, discovering that critical financial and data endpoints required no authentication tokens, headers, or sessions.
3. **Authorization and Object-Level Access Testing** — Tested sequential integer identifiers across multiple endpoint groups, confirming that account boundaries were not enforced and that the full range of registered accounts was enumerable.
4. **Payment Flow Validation** — Confirmed that the payment initiation endpoint reached a live national mobile money API by examining the response format and identifiers returned, which matched the production transaction identifier format of the payment network.
5. **Control Testing** — Identified endpoints that were properly protected (wallet queries, transaction history, invoice modification, API key creation), documenting that authentication mechanisms existed in the application but were not applied to the vulnerable endpoints. Also confirmed that cross-account channel validation existed for certain operations.
6. **Error Response Analysis** — Observed that invalid parameters returned internal database error messages, confirming direct database queries against production data.
7. **Client-Side Code Review** — Identified hardcoded API credentials in the application's JavaScript source code.
8. **Attack Chain Mapping** — Documented how the IDOR vulnerabilities provided the exact technical identifiers needed to exploit the payment initiation vulnerability, forming a complete attack chain from reconnaissance to financial impact.

## Safety and Scope Discipline

The following constraints were explicitly maintained throughout testing, as documented in the original report:

- **No real financial transactions completed.** Payment testing used an invalid phone number to prevent actual fund movement while still confirming that the endpoint reached the live payment network.
- **No customer data harvested.** Only sample records were examined for verification purposes; no mass data collection was performed.
- **Read-only operations where possible.** Data enumeration was conducted through read-only GET requests where feasible.
- **Rate-limited requests.** Requests were rate-limited to avoid service disruption.
- **No persistent changes.** No persistent modifications were made to any system.
- **Scope gaps documented.** The report explicitly identified untested areas (mobile app endpoints, WebSocket connections, internal admin panels, rate limiting effectiveness) to prevent overclaiming.

## Remediation Recommendations

Based on the findings documented in the report:

1. **Add authentication middleware to all payment and financial endpoints.** No financial operation should be callable without a verified session.
2. **Implement object-level authorization.** After authentication, verify that the authenticated user owns the account being operated on.
3. **Validate resource relationships server-side.** Ensure that payment channels belong to the requesting account before processing transactions.
4. **Add rate limiting** per account and per IP address on all endpoints, especially financial operations.
5. **Implement audit logging** for all payment initiation attempts, successful or not.
6. **Rotate all exposed client-side credentials immediately** and move secrets to server-side environment variables.
7. **Never embed API keys or secrets in client-side JavaScript.**
8. **Add authentication and rate limiting to reporting endpoints** to prevent mass abuse.
9. **Suppress internal error details.** Database error messages should never be returned to API consumers.

## Security Lessons

1. **Authentication is not optional for financial endpoints.** An unauthenticated payment initiation endpoint on a production payment network is an immediate critical risk. Every endpoint that triggers real-world financial effects must require verified identity.

2. **Sequential identifiers make enumeration trivial.** When account identifiers are sequential integers, a single endpoint without authentication exposes the entire user base. Use non-sequential, non-guessable identifiers for all external-facing references.

3. **IDOR vulnerabilities chain together.** Individually, each IDOR exposed different data. Together, they formed a complete attack chain: enumerate merchants → discover payment channels → initiate payments. Defense must address the pattern, not just individual endpoints.

4. **Negative testing reveals the inconsistency.** Several endpoints in the same application correctly enforced authentication, proving the platform had the capability. The vulnerability was not a missing feature — it was inconsistent application of an existing control.

5. **Client-side secrets are public secrets.** Any credential embedded in JavaScript delivered to the browser is compromised the moment the page loads. There is no client-side secret.

6. **Controlled proof-of-concept protects everyone.** Using an invalid phone number for payment testing confirmed the vulnerability reached the live payment network without causing financial harm — demonstrating that responsible validation is possible even for critical financial flaws.
