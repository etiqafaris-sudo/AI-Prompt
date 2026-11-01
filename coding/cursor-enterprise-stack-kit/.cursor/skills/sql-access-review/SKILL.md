---
name: sql-access-review
description: Review and implement safe Dapper or ADO.NET data access with provider-aware SQL, transaction handling, and concurrency controls. Use when writing or reviewing repositories, query classes, SQL scripts, or raw SQL.
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

## Standards

- Parameterize all values using provider parameters or Dapper parameters. Never concatenate untrusted values into SQL.
- Dynamic table/column names, sort directions, and SQL fragments cannot be parameterized: map user choices to a fixed allowlist.
- Open connections late and dispose deterministically (`using`/`await using` where supported). Let pooling work; do not keep a connection open across user interaction or unrelated work.
- Pass cancellation/timeouts where supported. Set command timeout deliberately for the workload; avoid infinite timeouts by default.
- Use transactions for multi-statement invariants. Keep transactions short and do not make remote HTTP calls while holding DB locks.
- For row claims/queue workers, design atomic claim/update statements, appropriate indexes, idempotency, and concurrency tests. Choose lock hints/isolation based on database semantics; do not copy SQL Server hints to other providers.
- Use explicit column lists; avoid `SELECT *` in stable production queries.
- Dapper mapping should use DTOs/read models and predictable column aliases. Avoid returning raw database rows as public API responses.
- Use `CommandDefinition`/cancellation when compatible with the installed Dapper version.
- ADO.NET provider differences matter (SQL Server, MariaDB/MySQL, Informix, PostgreSQL, etc.). Confirm syntax and parameter naming for the actual provider.
- Batch carefully. Bound batch size, assess lock duration/log growth, and use checkpoints/idempotency for restartability.
- Test duplicate submission, transaction rollback, timeout, deadlock/retry, and concurrent workers where relevant.
