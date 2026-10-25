---
name: sql-access-review
description: Review and implement safe Dapper or ADO.NET data access with provider-aware SQL, transaction handling, and concurrency controls.
disable-model-invocation: true
---
# SQL access workflow

## When to use

- Dapper/ADO.NET repositories, query classes, or `.sql` scripts.

1. Confirm database engine/provider and supported driver/version; do not transfer syntax or locking hints between engines.
2. Define query purpose, expected row count, latency target, indexes, transaction boundary, and concurrency invariant.
3. Parameterize every value. For dynamic identifiers/order clauses, map to a fixed allowlist.
4. Select explicit columns and stable aliases; use DTOs/read models.
5. Open late, close promptly, dispose deterministically, and set command timeout/cancellation.
6. For multi-step writes, use a short transaction. Avoid external calls while holding locks.
7. For worker claiming, design atomic claim/update, idempotency, uniqueness constraints, and safe retry behavior.
8. Review deadlock potential, isolation, lock escalation, index use, batch size, and retry policy.
9. Add integration tests for concurrent calls, duplicate requests, rollback, timeout, and provider-specific behavior.
10. Provide query plan/index suggestions only when supported by workload evidence.
