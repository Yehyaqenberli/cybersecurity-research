# Security Research Case Study: Multi-Tenant Application Platform

## Overview

An independent security assessment was conducted against a multi-tenant application platform serving multiple licensed operators. The platform comprised several interconnected backend services — including REST APIs, a real-time WebSocket service, and supporting infrastructure — sharing a common backend. Multiple critical and high-severity vulnerabilities were identified, including an unauthenticated SQL injection yielding database administrator privileges across numerous production databases.

## Researcher Role

Independent security researcher. Coordinated / responsible disclosure.

## Disclosure

- **Timeline:** Coordinated disclosure initiated in 2026.
- **Format:** Detailed written report delivered directly to the platform vendor.
- **Standard:** 90-day responsible disclosure timeline proposed.
- **Status:** Report delivered. Remediation status not confirmed at time of writing.

## Findings Summary

| # | Finding | Severity |
|---|---------|----------|
| 1 | **Unauthenticated SQL Injection** — Attacker-controlled input reached a backend database query without parameterization. Combined with client-controlled tenant selection, this provided unauthenticated database administrator access across numerous production databases. | Critical |
| 2 | **Predictable, Unbound Session Identifiers** — The real-time service issued session tokens that were predictable and not bound to their originating connections. A valid token could be used from any connection to perform operations on behalf of the token's owner. | Critical |
| 3 | **Exposed Cryptographic Signing Keys** — Unencrypted private keys used for a regulated-sector integration were stored in plaintext in the database and readable through the SQL injection path without authentication. Multiple distinct operator identities were affected. | High |
| 4 | **Insecure Credential Storage** — User passwords were stored in plaintext across application tables, alongside a correctly hashed control table in the same database — demonstrating the platform had the capability but did not apply it consistently. | High |
| 5 | **Unvalidated Input Leading to Balance Manipulation** — A business feature accepted a signed integer parameter without validation, allowing a negative value to invert the intended operation and create unauthorized balance. | High |
| 6 | **Identity Spoofing Across Tenant Boundaries** — A supporting service accepted client-supplied tenant and user identity fields without server-side validation, enabling cross-tenant message injection. | High |
| 7 | **Cross-Tenant Data Leakage (Multiple Vectors)** — Several endpoints and events returned data belonging to other tenants, including user identifiers and activity records. Multiple independent information-disclosure vectors were identified. | Medium |
| 8 | **Unauthenticated Sensitive Endpoints** — Financial endpoints required only a username parameter with no session or authentication, enabling username enumeration through distinct error responses. | Medium |
| 9 | **Internal API Exposed Over Plaintext HTTP** — A backend service was directly internet-reachable over unencrypted HTTP, exposing it to on-path interception. | Medium |

## Impact

- **Financial:** Potential for unauthorized balance manipulation, fraudulent operations on behalf of other users, and unauthorized access to financial records across the operator network.
- **Regulatory:** Exposure of cryptographic signing keys used for a regulated-sector integration, potentially enabling forgery of compliance records across multiple operators.
- **Privacy:** Cross-tenant leakage of user identifiers and activity data across tenant boundaries, with data-protection implications.
- **Infrastructure:** Database-administrator-level access to numerous production databases from an unauthenticated internet position, with dangerous server features enabled but not currently exploitable due to a permission gap.

## Methodology

1. **Reconnaissance and Endpoint Discovery** — Identified an unauthenticated API documentation endpoint that revealed the API structure, including the absence of authentication in client code.
2. **SQL Injection Identification** — Controlled input validation testing demonstrated that attacker-controlled data reached a backend database query without proper parameterization. Used error-based techniques to confirm impact, operating at concurrency 1.
3. **Privilege and Reach Assessment** — Verified database administrator privileges and production database scope through read-only system functions.
4. **Credential and Key Inventory** — Enumerated sensitive columns using metadata queries. Classified credential storage using statistical analysis (length distribution, character-set patterns) without reading any actual values. Identified cryptographic keys using format markers without extracting key material.
5. **Session Security Analysis** — Analyzed the structure of session identifiers, confirming they were predictable. Demonstrated cross-session operations between two researcher-controlled sessions only.
6. **Cross-Tenant Isolation Testing** — Tested tenant boundary enforcement across multiple interfaces by supplying alternate tenant identifiers. Identified multiple independent vectors where data from foreign tenants was returned.
7. **Input Validation Testing** — Tested numeric parameters for boundary conditions, discovering that signed integers were accepted without server-side validation, enabling balance manipulation.
8. **Supporting Service Authentication Testing** — Tested whether client-supplied identity fields were validated against the authenticated session, confirming they were not, and demonstrating cross-tenant message delivery.
9. **Negative and Control Testing** — Systematically tested endpoints that appeared vulnerable but were properly protected, documenting each negative result. Tested and withdrew an initially assessed higher severity rating after measurements did not support the broader scope claim.

## Safety and Scope Discipline

The following constraints were explicitly maintained throughout testing, as documented in the original report:

- **No destructive database operations.** No data modification statements were executed at any point. Administrator privilege was evidenced solely by reading a system function.
- **No dangerous server procedures were executed.** Dangerous server features were confirmed as enabled by reading configuration values only — they were never invoked.
- **No queries through internal infrastructure links.** Internal infrastructure definitions were read but no query was executed through any of them.
- **No third-party session takeover.** Cross-session exploitation was demonstrated exclusively between two researcher-controlled sessions. No guessed session identifiers were ever sent.
- **No credential harvesting.** Private key values were never read or extracted — only counted, measured by length, and compared via truncated fingerprints. No password value was ever read.
- **No bulk data collection.** One record was read as minimal proof of injection reaching production data and immediately discarded. Only a handful of foreign records were accessed for cross-tenant proof, each documented individually.
- **No contact with affected individuals.** Personal identifiers discovered in cross-tenant leaks were masked in the report; no individual was contacted.
- **No requests to regulated-sector endpoints.** Despite discovering signing keys, no request was made to any external integration endpoint, no signature was generated, and no authentication was attempted.
- **All persistent artifacts disclosed.** Every record created during testing was individually documented with identifiers, locations, and content, with deletion requested where appropriate.
- **Concurrency 1 throughout.** All testing was sequential with no availability impact.

## Remediation Recommendations

1. **Parameterize all database queries** and eliminate string concatenation of user input into query contexts.
2. **Replace client-controlled tenant selection** with an authenticated, server-side tenant binding mechanism.
3. **Apply least-privilege database access** — remove administrator roles from application service accounts.
4. **Rotate all exposed cryptographic keys** on a per-identity basis and migrate key storage to secure key management infrastructure.
5. **Hash all passwords** with a modern key derivation function and purge any plaintext credential logs.
6. **Bind session identifiers to their originating connections** and generate them from cryptographically secure random sources.
7. **Validate all input parameters server-side**, including sign and range checks on numeric fields.
8. **Derive all identity fields server-side** rather than accepting client-supplied values.
9. **Move data filtering from client-side to server-side** so that cross-tenant data never leaves the backend.
10. **Require authentication on all sensitive endpoints** including financial operations and data lookups.
11. **Enforce TLS on all internet-facing services** and restrict internal APIs to private network segments.
12. **Suppress framework debug error pages in production** — they served as a data exfiltration channel.

## Security Lessons

1. **A single unsanitized parameter can compromise an entire platform.** One injectable field, combined with client-controlled tenant selection, yielded administrator access to numerous production databases. Defense in depth means every layer must hold independently.

2. **Client-side filtering is not a security control.** When the server sends all tenants' data and relies on client-side code to filter, every client receives every tenant's records. Filtering must happen at the data layer.

3. **Session tokens must be cryptographically random and connection-bound.** Predictable tokens that any connection can use turn a session identifier into a universal access key.

4. **Sensitive material in the database is only as secure as the weakest path to that database.** Cryptographic keys stored as plaintext columns inherit every vulnerability in every application that touches that database.

5. **Measure your own claims rigorously.** During this research, an initially assessed higher severity was systematically tested with multiple independent measurements and downgraded when the evidence did not support the broader scope claim. Honest scoring builds credibility.

6. **Document what you did NOT do.** Comprehensive disclosure of testing boundaries, artifacts created, and data accessed — including negative results — distinguishes responsible research from unauthorized access.
