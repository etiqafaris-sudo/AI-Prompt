---
name: api-contracts
description: Design API contracts with explicit authorization, validation, error semantics, idempotency, and resilience. Use when adding or changing controllers, minimal endpoints, Program/Startup, or OpenAPI documents.
---
# API contract rules

- Define explicit schemas, constraints, examples, status codes, authentication requirements, and error semantics.
- Validate authorization for each operation and each resource. Tenant IDs and user IDs from clients are claims to verify, not trusted facts.
- Use request DTOs, avoid mass assignment, and never bind protected fields (role, owner, tenant, balance, approval status) directly from client input.
- Use consistent problem responses without internal exception details; include a correlation/trace ID.
- For retries, payments, submissions, and job triggers, define idempotency behavior and uniqueness constraints.
- Enforce request size, pagination, rate/concurrency limits, timeouts, and content-type expectations.
- Protect webhook endpoints with signature verification, timestamp/replay checks, and idempotent processing when applicable.
- Document deprecation/versioning and backward compatibility. Add contract tests for public or cross-team APIs.
