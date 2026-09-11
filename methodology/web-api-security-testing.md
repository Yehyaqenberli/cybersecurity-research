# Web & API Security Testing Methodology

## Overview

This document describes the general approach used for web application and API security assessments in the case studies published in this repository. It reflects real methodology applied during independent security research, adapted for public documentation with all target-specific details removed.

## Reconnaissance and Endpoint Discovery

- **Client-side code review:** Examine client-side application code and network requests to identify API structures, endpoint patterns, and authentication mechanisms (or their absence).
- **API documentation discovery:** Search for exposed API documentation or debug endpoints that reveal the application surface.
- **Parameter and identifier analysis:** Identify parameter types and assess their predictability and enumerability.

## Authentication Testing

- **Presence testing:** For each endpoint, determine whether any authentication mechanism is enforced.
- **Consistency testing:** Compare authentication enforcement across endpoints in the same application. Inconsistencies indicate systemic gaps rather than intentional design.
- **Credential exposure review:** Check client-side code for embedded credentials that should be server-side only.

## Input Validation Testing

- **Type confusion:** Submit unexpected data types and observe whether the application processes them differently from valid input, indicating insufficient input validation.
- **Boundary conditions:** Test boundary values including signed vs. unsigned integers, zero values, negative values, and extreme values to identify missing validation.
- **Error response analysis:** Examine error messages for internal implementation details that confirm backend technology and processing.

## Session Security Analysis

- **Token structure analysis:** Examine session identifiers for predictability, sequential patterns, or insufficient entropy.
- **Binding verification:** Test whether session tokens are bound to the connection or client that created them.
- **Cross-session testing:** Between researcher-controlled sessions only, verify whether one session's token can be used from another connection.

## Multi-Tenant Isolation Testing

- **Tenant boundary evaluation:** Test whether the application enforces tenant isolation when client-controlled tenant context is modified.
- **Cross-tenant data leakage:** Verify whether responses include data belonging to tenants other than the authenticated one.
- **Client-side vs. server-side filtering:** Determine whether tenant filtering happens on the server (data never leaves) or on the client (all data is sent, filtered in the browser).

## Proof-of-Concept Validation

- **Minimal impact proof:** Design proof-of-concept demonstrations that confirm the vulnerability with the smallest possible footprint.
- **Control tests:** For every positive finding, perform control tests that demonstrate the vulnerability is real rather than a false positive.
- **Negative results:** Document endpoints and operations that were tested and found properly protected. Negative results define the boundary of the issue and prevent overclaiming.

## Impact Assessment

- **Attack chain mapping:** Document how individual vulnerabilities combine into exploit chains with greater impact than any single finding alone.
- **Scope measurement:** Quantify the affected population using safe techniques, without extracting the underlying data.
- **Regulatory mapping:** Identify relevant regulatory frameworks that the findings may implicate.

## Documentation and Reporting

- **Per-finding structure:** Each finding includes affected component, proof of concept, control tests, severity scoring with justification, and specific remediation steps.
- **Artifact inventory:** Every persistent record or side effect created during testing is individually documented.
- **Scope limitations:** Untested areas are explicitly listed to prevent assumptions about comprehensive coverage.
- **Scoring integrity:** Scores are computed from standardized vectors. When a score was initially assessed higher than the evidence supported, it was tested and downgraded.
