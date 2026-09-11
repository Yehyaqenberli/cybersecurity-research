# Web & API Security Testing Methodology

## Overview

This document describes the general approach used for web application and API security assessments in the case studies published in this repository. It reflects real methodology applied during independent security research, adapted for public documentation with all target-specific details removed.

## Reconnaissance and Endpoint Discovery

- **Client-side code review:** Examine JavaScript bundles, HTML source, and network requests to identify API base URLs, endpoint patterns, and authentication mechanisms (or their absence).
- **API documentation discovery:** Search for exposed Swagger/OpenAPI documents, developer portals, or debug endpoints that reveal the API surface.
- **Parameter and identifier analysis:** Identify parameter types (sequential integers, UUIDs, opaque tokens) and assess their predictability and enumerability.

## Authentication Testing

- **Presence testing:** For each endpoint, determine whether any authentication mechanism is enforced — tokens, headers, cookies, or sessions.
- **Consistency testing:** Compare authentication enforcement across endpoints in the same application. Inconsistencies (some endpoints protected, others not) indicate systemic gaps rather than intentional design.
- **Credential exposure review:** Check client-side code for embedded API keys, secrets, tokens, or credentials that should be server-side only.

## Input Validation Testing

- **Type confusion:** Submit unexpected data types (strings in numeric fields, special characters, arithmetic expressions) and observe whether the application processes them differently from valid input.
- **Boundary conditions:** Test signed vs. unsigned integers, zero values, negative values, and extreme values to identify missing validation.
- **Error response analysis:** Examine error messages for internal implementation details (database error strings, stack traces, internal hostnames) that confirm backend technology and processing.

## Session Security Analysis

- **Token structure analysis:** Examine session identifiers for predictability, sequential patterns, timestamp components, or insufficient entropy.
- **Binding verification:** Test whether session tokens are bound to the connection or client that created them, or whether any client can use any valid token.
- **Cross-session testing:** Between researcher-controlled sessions only, verify whether one session's token can be used from another session's connection.

## Multi-Tenant Isolation Testing

- **Tenant boundary probing:** Supply alternate tenant identifiers (in headers, parameters, or path segments) to test whether the application enforces tenant isolation.
- **Cross-tenant data leakage:** Verify whether responses include data belonging to tenants other than the authenticated one.
- **Client-side vs. server-side filtering:** Determine whether tenant filtering happens on the server (data never leaves) or on the client (all data is sent, filtered in the browser).

## Proof-of-Concept Validation

- **Minimal impact proof:** Design proof-of-concept demonstrations that confirm the vulnerability with the smallest possible footprint — one record, one transaction, one controlled test.
- **Control tests:** For every positive finding, perform control tests that demonstrate the vulnerability is real rather than a false positive. For example, confirm that invalid identifiers are rejected while valid ones from other tenants are accepted.
- **Negative results:** Document endpoints and operations that were tested and found properly protected. Negative results are as important as positive ones — they define the boundary of the issue and prevent overclaiming.

## Impact Assessment

- **Attack chain mapping:** Document how individual vulnerabilities combine into exploit chains with greater impact than any single finding alone.
- **Scope measurement:** Quantify the affected population (accounts, records, tenants) using safe enumeration techniques, without extracting the underlying data.
- **Regulatory mapping:** Identify relevant regulatory frameworks (data protection, financial services, sector-specific licensing) that the findings may implicate.

## Documentation and Reporting

- **Per-finding structure:** Each finding includes affected component, proof of concept, control tests, CVSS scoring with justification, and specific remediation steps.
- **Artifact inventory:** Every persistent record or side effect created during testing is individually documented with identifiers and locations.
- **Scope limitations:** Untested areas are explicitly listed to prevent assumptions about comprehensive coverage.
- **Scoring integrity:** Scores are computed from vectors, not estimated. When a score was initially assessed higher than the evidence supported, it was tested and downgraded.
