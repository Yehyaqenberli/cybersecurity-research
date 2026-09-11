# Authorization and Access Control Testing Methodology

## Overview

Broken authentication and authorization consistently rank among the most impactful web application vulnerabilities. This document describes the methodology used for authentication and authorization testing across the case studies in this repository.

## Authentication Presence Testing

The first step is determining whether authentication is enforced at all:

1. **Strip all auth headers** — Remove `Authorization`, `Cookie`, `X-API-Key`, and any custom authentication headers from requests. If the endpoint still returns data, authentication is not enforced.
2. **Compare across endpoints** — Test every endpoint in the application for authentication requirements. A common pattern is partial enforcement: some endpoints require authentication while others sharing the same backend do not.
3. **Document the control group** — When some endpoints are properly protected, document them explicitly. This proves the application has authentication capabilities that are not consistently applied, strengthening the finding.

## Object-Level Authorization (IDOR)

Insecure Direct Object References occur when an application uses client-supplied identifiers to access resources without verifying the requester's ownership:

1. **Identifier predictability** — Assess whether object identifiers are sequential integers, short numeric ranges, or otherwise predictable. Sequential IDs make enumeration trivial.
2. **Cross-account access** — Using a legitimate session, attempt to access resources belonging to other accounts by modifying the identifier parameter.
3. **Unauthenticated access** — When authentication is absent entirely, IDOR testing merges with authentication testing: any identifier retrieves any resource.
4. **Enumeration scope** — Determine the range of valid identifiers to quantify the affected population. Test boundary values to find the upper limit without extracting all records.

## Tenant Isolation Testing (Multi-Tenant Systems)

Multi-tenant platforms require an additional layer of authorization — tenant boundary enforcement:

1. **Tenant identifier sources** — Identify how the application determines which tenant a request belongs to. Common methods include subdomain, path segment, HTTP header, session attribute, or explicit parameter.
2. **Client-controlled tenant selection** — If the tenant identifier comes from a client-controlled source (HTTP header, request parameter, path segment), test whether supplying a different tenant's identifier grants access to their data.
3. **Server-side vs. client-side isolation** — Verify whether tenant filtering occurs on the server or whether the server sends all tenants' data and relies on client-side code to filter. Inspect response payloads for data belonging to other tenants.
4. **Cross-tenant write testing** — When safe to do so (using researcher-controlled tenants or demo environments only), test whether writes (messages, configuration changes) cross tenant boundaries.

## Session Management Testing

1. **Token entropy** — Analyze session token format for randomness. Tokens containing timestamps, sequential counters, or predictable components are vulnerable to prediction or enumeration.
2. **Token binding** — Verify whether session tokens are bound to the connection, IP, or client that created them. Unbound tokens can be used from any connection.
3. **Token scope** — Determine whether tokens are scoped to a tenant or globally valid across all tenants.
4. **Controlled cross-session testing** — Demonstrate session hijacking between two researcher-controlled sessions only. Never test with sessions belonging to real users.

## Payment and Financial Authorization

Financial endpoints require the strictest authorization controls:

1. **Transaction initiation** — Verify that transaction endpoints require authenticated sessions with verified account ownership.
2. **Resource relationship validation** — Verify that the application checks relationships between resources (e.g., a payment channel belongs to the requesting account) before processing operations.
3. **Safe proof-of-concept design** — Use invalid recipient identifiers, minimum amounts, or sandbox parameters to confirm vulnerabilities reach production systems without completing real transactions.

## Documenting Negative Results

Negative results are critical for a credible assessment:

- **Properly protected endpoints** — List endpoints that correctly enforce authentication and authorization, with the specific error codes returned.
- **Working validation** — Document cases where cross-resource validation correctly rejected invalid combinations.
- **Withdrawn claims** — If an initially assessed severity is not supported by evidence, document the testing that led to the downgrade. This builds trust.
