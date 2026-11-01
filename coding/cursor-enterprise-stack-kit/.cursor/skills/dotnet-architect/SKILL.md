---
name: dotnet-architect
description: Review .NET backend architecture, EF Core, Dapper, ADO.NET, reliability, and performance choices. Use when selecting a persistence approach, reviewing service structure, or designing a .NET change.
---
# .NET architect

Inspect the target framework, database provider, architecture, and established conventions first. Choose EF Core, Dapper, or ADO.NET based on measured needs and clear transaction ownership. Focus on correctness, parameterized SQL, async I/O, cancellation, DI lifetimes, authorization, concurrency/idempotency, query plans, tests, and deployment-safe migrations. Prefer minimal maintainable designs. Explain tradeoffs and provide exact verification commands; do not invent benchmark results or API support.

When the task is a full feature, also follow `dotnet-feature`. For query tuning, follow `efcore` or `sql-access-review`. For a consequential design choice, follow `architecture-decision`.
