# FUTURE_CS_03
API Security Risk Analysis — jsonplaceholder.typicode.com/users

Security assessment of a single public, unauthenticated demo API endpoint (GET /users), evaluated as if the same request/response pattern appeared in a production system handling real user data.

Scope


Target: https://jsonplaceholder.typicode.com/users
Tooling: Postman, cross-checked with Chrome DevTools Network panel
Method: Passive review of request/response behavior — no exploitation, no destructive or high-volume testing
Note: JSONPlaceholder is an intentionally open mock API with no real user data; findings are framed hypothetically as production risk patterns


Summary

The endpoint requires no authentication and returns full user records (PII + internal company metadata) for every user in a single unfiltered call. Response headers leak backend framework and hosting details, and rate limiting is IP-based rather than identity-based. This combination maps to OWASP API Security Top 10: API1 (Broken Object Level Authorization), API2 (Broken Authentication), and API3 (Broken Object Property Level Authorization / Excessive Data Exposure).

Key Findings

#RiskSeverityFix1No authentication required to fetch all user recordsHighRequire OAuth2/JWT/API key; reject unauthenticated calls with 4012Excessive data exposure (full PII, geo-coordinates, internal company fields)HighField-level allow-lists per client role; drop unnecessary fields by default3No object-level authorization (BOLA/IDOR pattern)Medium-HighScope endpoints to caller's role; enforce per-record ownership checks server-side4Server fingerprinting via X-Powered-By, Server, NEL/Report-To headersLow-MediumStrip infra-identifying headers at the gateway5Coarse, non-identity-aware rate limitingLowTie rate limits to authenticated principal + IP; add anomaly detection6Access-Control-Allow-Credentials: true with unverified CORS originMediumConfirm explicit origin allow-list, never *, alongside credentialed CORS

Business Impact


Regulatory exposure (NDPR/GDPR) for unprotected PII processing
Reputational and customer-trust damage from bulk data scraping
Downstream fraud risk (phishing, SIM-swap, social engineering) from exposed contact data
Competitive intelligence leakage via exposed company metadata


Remediation Roadmap


Immediate: Enforce authentication on all user-data endpoints; default-deny unauthenticated requests
Immediate: Trim response payloads to caller-relevant fields only
Short-term: Add object-level authorization checks per record
Short-term: Lock down CORS configuration
Short-term: Strip identifying infrastructure headers
Ongoing: Move rate limiting to per-authenticated-principal basis with bulk-enumeration alerting
Follow-up: Parameter fuzzing/injection testing once filterable parameters are identified


Reference

Full findings, evidence screenshots, and detailed remediation guidance are in API_Security_Risk_Analysis_Report.docx.
