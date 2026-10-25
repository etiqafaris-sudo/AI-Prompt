---
name: security-review
description: Perform a structured, evidence-based security review of code or a feature, prioritizing exploitable risks and safe remediation.
disable-model-invocation: true
---
# Security review workflow

1. Establish scope: identify changed files, entry points, data flows, trust boundaries, auth model, deployment context, and target versions.
2. Map assets and threats: identities, sensitive data, tenant boundaries, secrets, privileged operations, external calls, file uploads, and background jobs.
3. Review against relevant OWASP ASVS / Top 10 themes: access control/IDOR, injection, auth/session, cryptography/secrets, misconfiguration, SSRF, deserialization, file handling, logging, rate limits, and business logic.
4. Trace each input from source to sink. Inspect SQL, shell/process calls, file paths, HTML output, URL fetches, redirects, serialization, and logging.
5. Verify server-side authorization at the resource/data boundary. Test both allowed and denied paths, including cross-user/cross-tenant attempts.
6. Check dependency versions and known advisories using available package tooling; do not invent CVEs or claim a scan ran if it did not.
7. Propose minimal fixes with regression tests. Avoid exploit payloads against live systems; use local unit/integration tests or an authorized staging target.
8. Produce findings with severity, confidence, affected file/line, preconditions, impact, evidence, remediation, regression test, and residual risk.
9. Label findings as Confirmed, Likely, or Needs validation. Distinguish code-review hypotheses from verified exploitability.
10. Finish with prioritized blockers and explicit checks not performed.

## Required output
- Scope and assumptions
- Findings table: ID, severity, confidence, evidence, impact, fix
- Tests added/run and results
- Residual risks / manual verification
- No blanket claim of “secure” or “compliant”
