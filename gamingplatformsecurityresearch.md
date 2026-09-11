# Security Research Case Study: Online Gaming Platform

## Overview

An independent security assessment was conducted against a multi-tenant online gaming platform serving dozens of licensed operators across multiple jurisdictions. The platform comprised a REST API backend, a real-time WebSocket game server, a separate chat service, and a public-facing data API — all sharing a common infrastructure. Twelve distinct vulnerabilities were identified across eight finding groups, including a critical unauthenticated SQL injection yielding database administrator privileges on hundreds of production databases.

## Researcher Role

Independent security researcher. Coordinated / responsible disclosure.

## Disclosure

- **Timeline:** Coordinated disclosure initiated approximately mid-2026.
- **Format:** Detailed written report delivered directly to the platform vendor.
- **Standard:** 90-day responsible disclosure timeline proposed.
- **Status:** Report delivered. Remediation status not confirmed at time of writing.

## Findings Summary

| # | Finding | Severity |
|---|---------|----------|
| 1 | **Unauthenticated SQL Injection** — A POST parameter in a ticket-lookup endpoint was concatenated directly into a numeric SQL context with no parameterization. Combined with tenant selection derived solely from an HTTP header, this gave unauthenticated access to hundreds of production databases with database-owner privileges. | Critical |
| 2 | **Predictable, Unbound Session Identifiers (WebSocket)** — The real-time game server issued session tokens that were globally sequential and not bound to the WebSocket connection that created them. Any client supplying a valid token could read balances and place bets on behalf of the token's owner. | Critical |
| 3 | **Exposed Cryptographic Signing Keys** — Unencrypted private keys used for a government regulatory integration were stored in plaintext in the database and readable through the SQL injection path without any authentication. Multiple distinct licensed operator identities were affected. | High |
| 4 | **Plaintext Password Storage** — Thousands of user passwords were stored in plaintext across application tables, alongside a correctly hashed control table in the same database — demonstrating the platform had the capability but did not apply it consistently. | High |
| 5 | **Unvalidated Input Leading to Balance Manipulation** — A game feature accepted a signed integer quantity parameter without validation, allowing a negative multiplier to convert a debit into a credit, creating real server-side balance. | High |
| 6 | **Chat Server Identity Spoofing** — The chat service accepted client-supplied tenant and player identity fields without server-side validation, enabling cross-tenant message injection into rooms serving independently licensed operators. | High |
| 7 | **Cross-Tenant Data Leakage (Multiple Vectors)** — Several endpoints and WebSocket events returned data belonging to other tenants, including live betting tickets with usernames, phone numbers, and player identifiers from operators in multiple countries. Five separate information-disclosure vectors were identified. | Medium |
| 8 | **Unauthenticated Sensitive Endpoints** — Financial endpoints (credit lookup, payout requests) required only a username parameter with no session token or authentication, enabling username enumeration via distinct error messages. | Medium |
| 9 | **Internal API Exposed Over Plaintext HTTP** — A backend game API was directly internet-reachable over unencrypted HTTP with no TLS listener, exposing it to on-path interception. | Medium |

## Impact

- **Financial:** Potential for unauthorized balance manipulation, fraudulent betting on behalf of other users, and unauthorized access to financial records across the entire operator network.
- **Regulatory:** Exposure of government-issued cryptographic signing keys for a national gaming regulator, potentially enabling forgery of regulatory compliance records across multiple licensed operators.
- **Privacy:** Cross-tenant leakage of player identifiers, usernames, phone numbers, and betting activity across jurisdictional boundaries, with data-protection implications for multiple national regulators.
- **Infrastructure:** Database-administrator-level access to hundreds of production databases from an unauthenticated internet position, with dangerous server features enabled but not currently exploitable only due to a single missing permission.

## Methodology

1. **Reconnaissance and Endpoint Discovery** — Identified an unauthenticated API documentation endpoint that referenced internal services and revealed the API structure, including the absence of authentication headers in client code.
2. **SQL Injection Identification** — Detected type-confusion behavior (non-numeric input returned column-name errors; arithmetic expressions were evaluated server-side), confirming direct SQL context injection. Used error-based extraction through framework debug pages, operating at concurrency 1.
3. **Privilege and Reach Assessment** — Verified database-owner privileges, server version, server topology, and database count through read-only SQL functions. Confirmed the scope across multiple physical database servers.
4. **Credential and Key Inventory** — Enumerated sensitive columns using metadata queries. Classified password storage as plaintext vs. hashed using statistical analysis (length distribution, character-set predicates) without reading any actual password values. Identified cryptographic keys using format markers and fingerprint comparison without extracting key material.
5. **Session Security Analysis** — Reverse-engineered the structure of WebSocket session identifiers, confirming they were globally sequential and timestamp-based. Demonstrated cross-session operations between two researcher-controlled sessions only.
6. **Cross-Tenant Isolation Testing** — Tested tenant boundary enforcement across REST and WebSocket interfaces by supplying alternate tenant identifiers. Identified five independent vectors where data from foreign tenants was returned.
7. **Input Validation Testing** — Tested numeric parameters for boundary conditions, discovering that signed integers were accepted without server-side validation, enabling balance manipulation.
8. **Chat Server Authentication Testing** — Tested whether client-supplied identity fields were validated against the authenticated session, confirming they were not, and demonstrating cross-tenant message delivery.
9. **Negative and Control Testing** — Systematically tested endpoints that appeared vulnerable but were properly protected, documenting each negative result. Tested and withdrew an initially inflated CVSS scope assessment after measurements did not support it.

## Safety and Scope Discipline

The following constraints were explicitly maintained throughout testing, as documented in the original report:

- **No destructive database operations.** No `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, or `TRUNCATE` was executed at any point. Database-owner privilege was evidenced solely by reading a system function.
- **No dangerous server procedures were executed.** Dangerous server features were confirmed as enabled by reading configuration values only — they were never invoked.
- **No queries through linked servers.** Internal linked-server definitions were read but no query was executed through any of them.
- **No third-party session takeover.** Cross-session exploitation was demonstrated exclusively between two researcher-controlled sessions. No guessed session identifiers were ever sent.
- **No credential harvesting.** Private key values were never read or extracted — only counted, measured by length, and compared via truncated fingerprints. No password value was ever read.
- **No bulk data collection.** One customer balance was read as minimal proof of injection reaching production data and immediately discarded. Only a handful of foreign ticket records were accessed for cross-tenant proof, each documented individually.
- **No contact with affected individuals.** Phone numbers discovered in cross-tenant leaks were masked in the report; no individual was contacted.
- **No requests to government regulatory endpoints.** Despite discovering signing keys, no request was made to any government integration endpoint, no signature was generated, and no authentication was attempted.
- **All persistent artifacts disclosed.** Every record created during testing (chat messages, demo bets) was individually documented with identifiers, locations, and content, with deletion requested where appropriate.
- **Concurrency 1 throughout.** All testing was sequential with no availability impact.

## Remediation Recommendations

Based on the findings documented in the report:

1. **Parameterize all SQL queries** and eliminate string concatenation of user input into SQL contexts.
2. **Replace header-based tenant selection** with an authenticated, server-side tenant binding mechanism.
3. **Apply least-privilege database access** — remove database-owner roles from application service accounts.
4. **Rotate all exposed cryptographic keys** on a per-licensed-identity basis and migrate key storage to hardware security modules or encrypted secret stores.
5. **Hash all passwords** with a modern key derivation function and purge any plaintext password logs.
6. **Bind session identifiers to their originating connections** and generate them from cryptographically secure random sources.
7. **Validate all input parameters server-side**, including sign and range checks on numeric fields.
8. **Derive all identity fields server-side** in the chat service rather than accepting client-supplied values.
9. **Move data filtering from client-side to server-side** so that cross-tenant data never leaves the backend.
10. **Require authentication on all sensitive endpoints** including financial operations and data lookups.
11. **Enforce TLS on all internet-facing services** and restrict internal APIs to private network segments.
12. **Suppress framework debug error pages in production** — they served as the data exfiltration channel.

## Security Lessons

1. **A single unsanitized parameter can compromise an entire platform.** One injectable field, combined with header-based tenant selection, yielded administrator access to hundreds of production databases. Defense in depth means every layer must hold independently.

2. **Client-side filtering is not a security control.** When the server sends all tenants' data and relies on browser JavaScript to filter, every client receives every tenant's records. Filtering must happen at the data layer.

3. **Session tokens must be cryptographically random and connection-bound.** Sequential, predictable tokens that any socket can use turn a session identifier into a universal access key.

4. **Sensitive material in the database is only as secure as the weakest path to that database.** Government signing keys stored as plaintext columns inherit every vulnerability in every application that touches that database.

5. **Measure your own claims rigorously.** During this research, an initially assessed CVSS 10.0 was systematically tested with five independent measurements and downgraded to 9.8 when the evidence did not support the scope-change claim. Honest scoring builds credibility.

6. **Document what you did NOT do.** Comprehensive disclosure of testing boundaries, artifacts created, and data accessed — including negative results — distinguishes responsible research from unauthorized access.
