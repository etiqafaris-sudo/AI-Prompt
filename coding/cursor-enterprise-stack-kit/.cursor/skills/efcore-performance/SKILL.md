---
name: efcore-performance
description: Diagnose and improve EF Core query and persistence performance without sacrificing correctness or tenant isolation.
disable-model-invocation: true
---
# EF Core performance workflow

## When to use

- DbContext, repositories, migrations, handlers, or other EF Core persistence paths.

1. Identify provider/version, query frequency, cardinality, expected data volume, indexes, and observed latency. Ask for a query plan/metrics if needed.
2. Reproduce with representative data; do not optimize solely by intuition.
3. Inspect generated SQL and query plan. Check N+1, cartesian explosion, missing indexes, excessive columns, client evaluation, tracking overhead, and pagination.
4. Prefer projection to DTOs; use no-tracking for read-only work; use explicit loading strategy and deterministic pagination.
5. Verify connection-pool usage and ensure each DbContext is used sequentially and within one unit of work.
6. Review transaction isolation, concurrency tokens, and lock duration for writes.
7. Add indexes only with evidence and assess write cost/storage and migration lock impact.
8. Add regression/performance tests or a reproducible benchmark where practical.
9. Compare before/after metrics and explain tradeoffs. Never promise a numeric speedup without measurement.
