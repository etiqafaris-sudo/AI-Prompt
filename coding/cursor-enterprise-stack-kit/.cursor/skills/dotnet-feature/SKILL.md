---
name: dotnet-feature
description: Design and implement a production-quality .NET backend feature with architecture, security, persistence, tests, observability, and rollout checks.
disable-model-invocation: true
---
# .NET feature workflow

## When to use

- New or changed .NET backend features (`.cs`, `.csproj`, APIs, handlers, persistence).

1. Inspect target framework, solution structure, dependency versions, architecture, DI registration, auth policies, database provider, and test conventions.
2. Write acceptance criteria and identify input/output contracts, authorization, tenant ownership, data classification, concurrency, idempotency, and failure modes.
3. Propose the smallest architecture-consistent plan. Keep HTTP concerns out of domain logic and do not introduce a repository abstraction automatically if EF Core already fits.
4. Choose persistence deliberately:
   - EF Core: aggregate writes, change tracking, relational mapping, migrations.
   - Dapper: explicit high-performance/read-heavy SQL or carefully controlled bulk/query paths.
   - ADO.NET: provider-specific control or low-level scenarios where its extra complexity is justified.
   Do not mix approaches without documenting transaction ownership and mapping boundaries.
5. Implement DTO validation, server-side authorization, tenant filters/ownership checks, safe errors, cancellation, structured logs, and metrics where appropriate.
6. SQL must be parameterized; define indexes based on expected query patterns. Avoid N+1 and unbounded reads.
7. Handle concurrency and retries deliberately. Use idempotency and database constraints for duplicate-prone operations.
8. Add unit tests plus integration tests for persistence/auth/API behavior. Include negative and boundary tests.
9. Run relevant format/analyzer/build/test/dependency checks using the repo's actual commands.
10. Review config, secrets, health checks, metrics, tracing, deployment compatibility, and migration rollback/forward-fix plan.
11. Report files changed, design choices, commands/results, assumptions, residual risks, and recommended review.

## Stop and ask before
- Breaking a public contract
- Destructive schema/data changes
- Disabling an existing security control
- Changing identity/tenant/payment/commission semantics
- Running a production migration or deployment
