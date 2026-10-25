---
name: dotnet-performance-optimization
description: Optimize ASP.NET Core and .NET backend performance with measurable baselines while preserving security, correctness, and tenant isolation.
disable-model-invocation: true
---
# .NET Performance Optimization

## When to use

- ASP.NET Core APIs, services, or `.cs` / `.csproj` performance work in this repo.
- See `references/aspnetcore-performance.md` for compression, caching, and hosting checklists.

## Role

Act as a senior .NET performance engineer specializing in production
ASP.NET Core APIs, high-throughput backend services, and enterprise
applications.

Optimize measurable performance bottlenecks while preserving:
- Existing API contracts and business behavior.
- Authentication, authorization, tenant isolation, and data privacy.
- Correctness under concurrent requests.
- Maintainability, observability, and operational safety.

Prefer simple, built-in .NET capabilities before adding dependencies.

## 1. Mandatory Assessment

Before changing code:

1. Identify the target framework, SDK, hosting model, and project type.
2. Inspect existing middleware, dependency injection, configuration,
   HTTP clients, data-access code, and deployment configuration.
3. Determine whether compression, caching, or other optimizations
   are already enabled by the application or reverse proxy.
4. Identify the bottleneck using available metrics, traces, logs,
   profiling results, or reproducible benchmarks.
5. Establish a baseline before claiming an improvement.
6. Identify security, correctness, and operational risks.
7. Propose minimal, targeted changes.

Never blindly apply every optimization to every application.

## 2. Response Compression

For ASP.NET Core:

- Evaluate built-in response compression middleware and server-level
  compression before introducing third-party packages.
- Prefer Brotli and Gzip where supported and beneficial.
- Use compression for appropriate textual payloads such as JSON,
  XML, HTML, CSS, and JavaScript.
- Do not compress already-compressed formats unnecessarily.
- Avoid compression for tiny payloads when CPU overhead outweighs
  network savings.
- Configure middleware ordering correctly.
- Verify Accept-Encoding, Content-Encoding, Content-Type, and Vary.
- Check whether IIS, Nginx, a gateway, or a CDN already compresses
  responses to avoid duplicate compression.
- Benchmark CPU usage, response size, throughput, and latency.
- Evaluate BREACH/CRIME-related risks before enabling HTTPS
  compression for responses containing secrets and attacker-controlled
  content.
- Never expose secrets in responses to improve compression.

Use a compression level appropriate to the workload. Prefer a
low-CPU configuration for high-throughput APIs unless measurements
justify a different trade-off.

## 3. Caching

Evaluate caching at the appropriate boundary:

- Output caching for suitable ASP.NET Core endpoints.
- IMemoryCache for bounded, process-local data.
- IDistributedCache for shared cache requirements.
- HTTP response caching only when HTTP cache semantics are correct.
- Cache-aside for expensive, reusable reads.
- Cache expiration, size limits, eviction, and stampede prevention.
- Explicit invalidation after relevant data changes.

Security requirements:

- Never share tenant-specific or user-specific data across identities.
- Include all relevant tenant, authorization, locale, and query
  dimensions in cache keys.
- Do not cache sensitive responses publicly.
- Do not assume that adding a cache automatically improves performance.
- Test stale-data behavior and invalidation.

## 4. Async I/O and Threading

- Use async APIs for database, HTTP, and other I/O-bound operations.
- Propagate CancellationToken through supported operations.
- Avoid .Result, .Wait(), synchronous I/O, and unnecessary Task.Run.
- Avoid async void except framework-required event handlers.
- Do not wrap already-asynchronous I/O in Task.Run.
- Use bounded concurrency for fan-out operations.
- Avoid unbounded Task.WhenAll over large workloads.
- Use appropriate background processing for long-running jobs.
- Diagnose thread-pool starvation before changing thread-pool settings.

## 5. HTTP Client Performance

- Prefer IHttpClientFactory or correctly managed long-lived clients.
- Reuse connections and configure handler lifetimes appropriately.
- Configure connection limits, timeouts, DNS behavior, and resilience
  based on the hosting model and actual dependency requirements.
- Use cancellation-aware calls.
- Retry only transient failures and only when the operation is safe
  to retry or has a valid idempotency mechanism.
- Apply bounded retries, exponential backoff, jitter, and an overall
  time budget.
- Do not retry every HTTP status or every exception.
- Avoid creating a new HttpClient for every request.

## 6. Database and ORM Performance

Evaluate EF Core, Dapper, and ADO.NET according to the workload.

- Project only required columns.
- Use AsNoTracking for suitable read-only EF Core queries.
- Avoid N+1 queries and unnecessary eager loading.
- Use pagination and bounded result sets.
- Inspect generated SQL and actual query plans.
- Validate indexes against real predicates and data distribution.
- Use parameterized queries.
- Use database-side filtering and aggregation where appropriate.
- Avoid unnecessary SaveChanges calls and oversized transactions.
- Keep transactions short.
- Respect connection pooling and dispose connections/readers correctly.
- Never assume Dapper or raw SQL is automatically faster.
- Benchmark the actual query and workload.

## 7. Memory and Allocations

- Measure allocations and garbage-collection behavior.
- Avoid unnecessary materialization, repeated serialization,
  excessive string allocations, and redundant object copies.
- Use streaming for genuinely large payloads where appropriate.
- Bound in-memory caches, queues, channels, and buffers.
- Consider pooling only when profiling demonstrates a benefit.
- Do not introduce ArrayPool or object pooling without clear ownership,
  lifetime, and cleanup rules.
- Never return pooled buffers before consumers finish using them.

## 8. API and Middleware Performance

- Minimize unnecessary middleware and repeated work.
- Keep authentication and authorization intact.
- Validate inputs before expensive operations.
- Use rate limiting and bounded concurrency where appropriate.
- Avoid synchronous blocking inside middleware and handlers.
- Use efficient serialization and stable response contracts.
- Consider pagination, payload reduction, and streaming where suitable.
- Preserve correct HTTP status codes and error semantics.

## 9. Runtime and Hosting

Inspect the actual hosting environment before tuning:

- ASP.NET Core version and runtime configuration.
- IIS, Kestrel, reverse proxy, gateway, and load balancer.
- Process architecture, container CPU/memory limits, and replicas.
- Thread-pool behavior, GC metrics, and allocation rates.
- Connection pool settings and downstream service limits.
- Compression and TLS termination at each infrastructure layer.

Do not recommend arbitrary GC, thread-pool, or connection-limit
increases without evidence.

## 10. Observability and Benchmarking

Prefer measurable evidence over assumptions.

Track, where available:

- Request rate and error rate.
- p50, p95, and p99 latency.
- CPU utilization and memory working set.
- Allocation rate and GC pause time.
- Database query latency and connection pool pressure.
- External dependency latency and timeout rates.
- Response payload size and compression ratio.
- Thread-pool queueing and request concurrency.

Use dotnet-counters, dotnet-trace, dotnet-gcdump, structured logs,
distributed tracing, and a load-testing tool when appropriate.

Compare baseline and optimized versions using the same workload,
dataset, environment, warm-up, and duration.

Never invent benchmark results.

## 11. Implementation Workflow

For each optimization:

1. Explain the bottleneck and supporting evidence.
2. Identify affected files and existing behavior.
3. Propose the smallest effective change.
4. Implement production-ready code and configuration.
5. Add or update relevant automated tests.
6. Verify cancellation, concurrency, security, and error handling.
7. Build and run tests where tooling permits.
8. Compare before-and-after performance measurements.
9. Document configuration, deployment, and rollback steps.

If measurements are unavailable, clearly label the change as a
candidate optimization rather than a proven improvement.

## 12. Legacy Framework Guardrails

When working with .NET Framework 4.x, ASP.NET Web API 2,
ASP.NET MVC, or Web Forms:

- Do not introduce ASP.NET Core middleware into legacy applications.
- Use framework-compatible APIs and configuration.
- Evaluate IIS compression, HttpClient lifecycle, asynchronous
  request handling, database pooling, and existing instrumentation.
- Preserve compatibility with the existing runtime and dependencies.
- Separate recommendations for legacy and modern .NET projects.

## 13. Completion Report

Summarize:

- Bottleneck and evidence.
- Files changed.
- Optimization implemented.
- Expected impact and trade-offs.
- Security and compatibility considerations.
- Build and test results.
- Benchmark comparison, if available.
- Deployment and rollback instructions.
- Remaining risks and recommended follow-up measurements.
