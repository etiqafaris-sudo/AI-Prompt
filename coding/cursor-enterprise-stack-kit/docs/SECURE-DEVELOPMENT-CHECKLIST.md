# Secure Development and Release Checklist

Use this as a living checklist; tailor it to system risk and required policy.

## Design
- [ ] Data classification, trust boundaries, threat model, tenant model, and authorization matrix documented.
- [ ] Public endpoints and privileged workflows identified.
- [ ] Authentication/session/CSRF/CORS design reviewed.
- [ ] Abuse controls: rate limits, request size, pagination, timeouts, concurrency, quotas.
- [ ] Secrets, key rotation, encryption, retention, deletion, and audit needs defined.
- [ ] Architecture decisions recorded when material.

## Implementation
- [ ] Server-side authorization for every operation and resource.
- [ ] Cross-tenant/object-level authorization tests.
- [ ] Runtime input validation and output encoding.
- [ ] Parameterized SQL and allowlisted dynamic identifiers.
- [ ] Safe upload, URL-fetch, redirect, serialization, and file-path handling.
- [ ] No secrets or sensitive payloads in logs.
- [ ] Safe error messages and correlation IDs.
- [ ] Idempotency/concurrency guarantees for duplicate-prone workflows.
- [ ] Correct cache isolation for user/tenant data.
- [ ] Dependency versions supported and vulnerability findings triaged.

## Verification
- [ ] Unit and integration tests.
- [ ] Negative security tests and regression tests.
- [ ] Build, lint, format/analyzers, and typecheck as applicable.
- [ ] SAST, secret scanning, SCA, and authorized DAST evidence reviewed.
- [ ] Critical queries/transactions reviewed for plan, indexes, locks, and timeouts.
- [ ] Accessibility checks for critical user journeys.
- [ ] No tests against production or third-party systems without explicit scope/approval.

## Release and operations
- [ ] Required peer/security/privacy reviews completed.
- [ ] Migration plan, backup/restore, compatibility, rollback/forward-fix documented.
- [ ] Configuration and secret injection verified per environment.
- [ ] Monitoring, alerts, dashboards, runbooks, and ownership set.
- [ ] Audit log access/retention protected.
- [ ] Release notes, evidence, approvals, and rollback decision owner recorded.
- [ ] Exceptions have approver, compensating controls, expiry, and remediation ticket.
