---
name: governance-review
description: Review a feature or release for engineering governance, traceability, change control, auditability, and compliance evidence. Use when assessing release readiness, migrations, audit trails, ADRs, or compliance evidence.
---
# Governance review workflow

Evaluate traceability, ADRs, data classification, auditability, change approvals, migration safety, dependency/security evidence, exception expiry, rollback, and operational readiness. Mark each item Pass, Fail, or Not evidenced. Do not claim regulatory compliance from code conventions alone; ask which framework and jurisdiction actually apply. Identify the accountable owner and the evidence required for each gap.

## Standards

- Before a material feature, identify owner, business purpose, data classification, trust boundaries, authorization model, audit requirements, and acceptance criteria.
- Record architecture choices that affect security, cost, availability, data, integrations, or long-term maintainability as ADRs with context, decision, alternatives, consequences, and date.
- Changes to auth, roles, tenant isolation, personal data, cryptography, payment/commission calculations, retention, or schema require explicit tests and human review.
- Maintain traceability: requirement/ticket → code change → tests → review → deployment/release record. Do not invent ticket IDs or approval evidence.
- Database migrations require peer review, backup/restore considerations, deployment sequencing, data validation, rollback or forward-fix plan, and compatibility assessment.
- Security exceptions must document risk, compensating controls, accountable approver, expiration date, and remediation ticket. Never silently waive a control.
- Apply least privilege and separation of duties for production access and release approval. AI-generated changes do not bypass normal review.
- Logs/audit events should capture who/what/when/outcome and correlation ID for relevant business/security actions, while minimizing personal data and excluding secrets. Define retention/access controls.
- Map controls to the actual required framework (for example organizational policy, ISO 27001, SOC 2, PCI DSS, PDPA, contractual obligations) only after confirming applicability. Do not claim compliance merely because a rules pack is installed.
- Release gate: tests pass; security/dependency findings triaged; migration plan reviewed; configuration/secrets verified; observability and rollback prepared; required approvals recorded.

## Workflow

1. Identify the requirement/ticket, owner, business impact, data classification, affected systems, and release environment.
2. Check that acceptance criteria are testable and trace to implementation and tests.
3. Identify security/privacy risks, control owners, audit events, retention, access controls, and external processor/integration concerns.
4. Check whether an ADR is needed for architecture, data, security, availability, cost, or operationally material decisions.
5. For schema/data changes, verify migration sequencing, compatibility, backup/restore, data validation, and rollback/forward-fix plan.
6. Check dependency/SAST/secret-scan/SBOM evidence as available. Clearly mark missing scans and manual review.
7. Verify approval separation, least privilege, exception expiry, and change/release evidence.
8. Map controls only to frameworks actually required by the organization. Do not infer legal or certification applicability.
9. Produce a release checklist with Pass / Fail / Not evidenced, evidence location, owner, and action required.
