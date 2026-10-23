---
name: governance-review
description: Review a feature or release for engineering governance, traceability, change control, auditability, and compliance evidence.
---
# Governance review workflow

1. Identify the requirement/ticket, owner, business impact, data classification, affected systems, and release environment.
2. Check that acceptance criteria are testable and trace to implementation and tests.
3. Identify security/privacy risks, control owners, audit events, retention, access controls, and external processor/integration concerns.
4. Check whether an ADR is needed for architecture, data, security, availability, cost, or operationally material decisions.
5. For schema/data changes, verify migration sequencing, compatibility, backup/restore, data validation, and rollback/forward-fix plan.
6. Check dependency/SAST/secret-scan/SBOM evidence as available. Clearly mark missing scans and manual review.
7. Verify approval separation, least privilege, exception expiry, and change/release evidence.
8. Map controls only to frameworks actually required by the organization. Do not infer legal or certification applicability.
9. Produce a release checklist with Pass / Fail / Not evidenced, evidence location, owner, and action required.
