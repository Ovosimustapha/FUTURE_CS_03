# FUTURE_CS_03
# API Security Risk Analysis — jsonplaceholder.typicode.com/users

Security assessment of a single public, unauthenticated demo API endpoint (`GET /users`), evaluated as if the same request/response pattern appeared in a production system handling real user data.

## Scope

- **Target:** `https://jsonplaceholder.typicode.com/users`
- **Tooling:** Postman, cross-checked with Chrome DevTools Network panel
- **Method:** Passive review of request/response behavior — no exploitation, no destructive or high-volume testing
- **Note:** JSONPlaceholder is an intentionally open mock API with no real user data; findings are framed hypothetically as production risk patterns

## Summary

The endpoint requires no authentication and returns full user records (PII + internal company metadata) for every user in a single unfiltered call. Response headers leak backend framework and hosting details, and rate limiting is IP-based rather than identity-based. This combination maps to OWASP API Security Top 10: **API1 (Broken Object Level Authorization)**, **API2 (Broken Authentication)**, and **API3 (Broken Object Property Level Authorization / Excessive Data Exposure)**.

## Key Findings

| # | Risk | Severity | Fix |
|---|------|----------|-----|
| 1 | No authentication required to fetch all user records | High | Require OAuth2/JWT/API key; reject unauthenticated calls with 401 |
| 2 | Excessive data exposure (full PII, geo-coordinates, internal company fields) | High | Field-level allow-lists per client role; drop unnecessary fields by default |
| 3 | No object-level authorization (BOLA/IDOR pattern) | Medium-High | Scope endpoints to caller's role; enforce per-record ownership checks server-side |
| 4 | Server fingerprinting via `X-Powered-By`, `Server`, NEL/Report-To headers | Low-Medium | Strip infra-identifying headers at the gateway |
| 5 | Coarse, non-identity-aware rate limiting | Low | Tie rate limits to authenticated principal + IP; add anomaly detection |
| 6 | `Access-Control-Allow-Credentials: true` with unverified CORS origin | Medium | Confirm explicit origin allow-list, never `*`, alongside credentialed CORS |

## Business Impact

- Regulatory exposure (NDPR/GDPR) for unprotected PII processing
- Reputational and customer-trust damage from bulk data scraping
- Downstream fraud risk (phishing, SIM-swap, social engineering) from exposed contact data
- Competitive intelligence leakage via exposed company metadata

## Remediation Roadmap

1. **Immediate:** Enforce authentication on all user-data endpoints; default-deny unauthenticated requests
2. **Immediate:** Trim response payloads to caller-relevant fields only
3. **Short-term:** Add object-level authorization checks per record
4. **Short-term:** Lock down CORS configuration
5. **Short-term:** Strip identifying infrastructure headers
6. **Ongoing:** Move rate limiting to per-authenticated-principal basis with bulk-enumeration alerting
7. **Follow-up:** Parameter fuzzing/injection testing once filterable parameters are identified

## Reference

Full findings, evidence screenshots, and detailed remediation guidance are in `API_Security_Risk_Analysis_Report.docx`.
