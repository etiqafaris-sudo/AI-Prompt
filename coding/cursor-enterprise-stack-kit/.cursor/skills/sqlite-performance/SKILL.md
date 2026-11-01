---
name: sqlite-performance
description: Configure Microsoft.Data.Sqlite for high read throughput using WAL and the repo's per-connection performance PRAGMAs, including sqlite-vec. Follows the personal skills engineering-baseline, dotnet-backend, security-and-privacy, sql-access-review, efcore, efcore-performance, and dotnet-performance-optimization. Use when changing SQLite connections, PRAGMA settings, RagSqliteConnection, SqliteOptions, sqlite-vec loading, EF Core SQLite interceptors, or SQLite latency.
disable-model-invocation: true
---

# High-performance SQLite

## Global skills

Follow the personal skills in `~/.cursor/skills`. When a project copy of the same name exists, follow the personal skill. Do not restate those workflows here.

Always:

- `engineering-baseline` before any edit
- `dotnet-backend` for C#, DI, options, and appsettings
- `security-and-privacy` for SQL, configuration, and file paths
- `sql-access-review` for the PRAGMA command and any other ADO.NET

When the path uses EF Core (`UseSqlite`, interceptor, `DbContext`):

- `efcore`
- `efcore-performance`

When the task is a latency change or a claimed speedup:

- `dotnet-performance-optimization`

When adding or changing tests:

- `testing-and-quality`

When the change is a new backend feature:

- `dotnet-feature`

When changing `journal_mode`, `synchronous`, or `Cache` mode:

- `architecture-decision`

## When to use

- `RagSqliteConnection`, `SqliteVecConnectionInterceptor`, `SqliteOptions`, or the `Sqlite` section in appsettings.
- A new `SqliteConnection` or `UseSqlite` path, sqlite-vec, or SQLite latency work.

## Baseline in this repo

Extend the existing setup. Do not add a second connection factory or a second PRAGMA list.

- `RagSqliteConnection.GetConnectionString` — `ReadWriteCreate`, `Cache = Shared`, `Pooling = true`. Resolve `DataSource` from the content root and create the parent directory.
- `RagSqliteConnection.ApplyPerformancePragmas` — run on every opened connection.
- `SqliteVecConnectionInterceptor` — both `ConnectionOpened` and `ConnectionOpenedAsync` apply the PRAGMAs, then call `sqlite.LoadVector()`.
- `SqliteOptions` / `Sqlite` config — `CacheSizeKib` default `-64000` (a negative `cache_size` is KiB, about 64 MiB). `MmapSizeBytes` default `268435456` (256 MiB).

## PRAGMA set

One parameterized command, in this order:

```sql
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
PRAGMA temp_store = MEMORY;
PRAGMA foreign_keys = ON;
PRAGMA cache_size = $cacheSize;
PRAGMA mmap_size = $mmapSize;
```

Bind `$cacheSize` and `$mmapSize` from `SqliteOptions`. Never concatenate caller input into PRAGMA text.

| PRAGMA | Role | Scope |
| --- | --- | --- |
| `journal_mode=WAL` | Readers do not block the writer. Writes use `-wal` and `-shm` files beside the database. | Stored in the database file. |
| `synchronous=NORMAL` | With WAL, skips a fsync on each commit. A process crash is safe. An OS crash or power loss can drop the last transactions. | Per connection. |
| `temp_store=MEMORY` | Temp tables and sorts stay in memory. | Per connection. |
| `foreign_keys=ON` | Enforcement is off until this connection sets it. Keep it. This is correctness. | Per connection. |
| `cache_size` | Page cache. Negative values are KiB. | Per connection. |
| `mmap_size` | Memory-maps the database file. Size it to the working set and available RAM. `0` disables it. | Per connection. |

## Rules

1. Call `ApplyPerformancePragmas` on every open, including connections returned from the pool. `synchronous`, `cache_size`, `mmap_size`, `temp_store`, and `foreign_keys` do not carry over from another connection.
2. Apply PRAGMAs before queries and before `LoadVector()`.
3. Keep the sync and async interceptor methods on the same path. A `SqliteConnection` opened outside EF Core must call `ApplyPerformancePragmas` too.
4. Leave `synchronous` at `NORMAL`. `OFF` can corrupt the file on a crash. Move to `FULL` only for a stated durability requirement.
5. WAL needs a local disk with working shared-memory locks. Do not put the database on a network share.
6. Leave `journal_mode` at `WAL`. Do not switch to `DELETE`, `TRUNCATE`, or `MEMORY` for speed.
7. Change cache and mmap sizes in `Sqlite` configuration, not by hard-coding a new PRAGMA.
8. Add `PRAGMA busy_timeout` (milliseconds, parameterized) only after `SQLITE_BUSY` appears under concurrent writers. SQLite's default is `0`, which fails immediately. Do not add it before that evidence.
9. Do not change `page_size` on an existing database without a rebuild (`VACUUM`).
10. `Cache=Shared` is the connection string in this repo. It shares one in-process pager cache and changes locking. Switch to `Private` only when a measured concurrency problem requires it.
11. Compare timings before and after. Do not claim a speedup without a measurement.

## Do not add

- `synchronous=OFF`, `locking_mode=EXCLUSIVE`, or `journal_mode=MEMORY` on the RAG database.
- `optimize`, `analysis_limit`, `threads`, or `wal_autocheckpoint` unless a profile shows they hit the bottleneck.
- Per-query cache or mmap settings.

## Check

- Every new SQLite entry point calls `ApplyPerformancePragmas` once per open.
- Both interceptor methods stay aligned.
- Numeric PRAGMAs stay parameterized and come from `SqliteOptions`.
- `foreign_keys` stays `ON`.
- `*.db-wal` and `*.db-shm` next to `data/rag.db` are expected. Do not delete them while the process is running.
