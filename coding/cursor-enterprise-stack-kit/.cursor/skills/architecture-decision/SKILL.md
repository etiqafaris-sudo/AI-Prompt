---
name: architecture-decision
description: Compare architectural options and write an evidence-based Architecture Decision Record before consequential design changes.
---
# Architecture decision workflow

1. Restate the decision and constraints: business need, current architecture, runtime, scale, security, team skills, operational support, and cost.
2. List realistic options, including keeping the current design.
3. Compare options against explicit criteria: correctness, security, performance, operability, testability, maintainability, migration cost, and vendor/provider constraints.
4. Separate facts from assumptions; identify what must be prototyped or measured.
5. Recommend one option with rationale and tradeoffs, not just a generic best practice.
6. Create/update an ADR using `docs/adr/ADR-TEMPLATE.md`.
7. Include migration/rollback strategy, security review needs, observability, and ownership.
