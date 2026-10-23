# Stack separation

## Backend
Shared .NET rules cover architecture, API contracts, observability, validation, resilience, and security. EF Core and Dapper/ADO.NET rules are separate because their performance, transaction, and concurrency concerns differ.

## Blazor
Blazor-specific instructions cover render mode, component lifecycle, DI scope/circuit isolation, JS interop, authorization, and client-vs-server trust boundaries.

## Next.js
Next.js-specific instructions cover Server/Client Components, Route Handlers/Server Actions, server-only secrets, caching/revalidation, runtime validation, and browser security.

## Shared controls
Security, governance, testing, and baseline engineering rules apply to all stacks. Avoid putting all guidance into one enormous always-on rule: keep baseline rules concise and invoke detailed skills when doing a task.
