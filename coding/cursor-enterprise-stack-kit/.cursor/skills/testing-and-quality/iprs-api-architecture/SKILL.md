---
name: iprs-api-architecture
description: Follow the iPRS API architecture for .NET 10 Minimal APIs, Clean Architecture, vertical slices, CQRS with Mediator, FluentValidation, EF Core, and MariaDB. Use only when the workspace is iPRSApiV2 or the user asks to follow this project's architecture rules.
---
# iPRS API architecture

This project is a .NET 10 REST API built with Clean Architecture and vertical slices. It uses CQRS, Minimal APIs, FluentValidation, and Entity Framework Core with MariaDB.

## Tech stack

- Runtime: .NET 10, ASP.NET Core Minimal APIs
- Database: MariaDB, Entity Framework Core
- Validation: FluentValidation
- Architecture: Clean Architecture, vertical slices, CQRS via https://github.com/martinothamar/Mediator
- Tools: Scalar OpenAPI UI, health checks, logging decorators, EF Core audit interceptor

## Layers

WebApi → Infrastructure → Application → Domain

- Domain contains entities and business rules and must not depend on any other layer.
- Application contains features (vertical slices) and depends only on Domain.
- Infrastructure implements persistence and external services and depends on Application and Domain.
- WebApi is the entry point and composes dependencies and middleware.

## Vertical slices

Each feature is self-contained. Example:

`Application/Features/Book/CreateBook/`

- `CreateBookHandler.cs`
- `CreateBookValidator.cs`
- `CreateBookEndpoint.cs`

Each feature owns its handler, validator, and endpoint. Business logic stays in handlers. Endpoints only map HTTP.

## CQRS

Use https://github.com/martinothamar/Mediator.

- Commands modify state: Create, Update, Delete.
- Queries read state: GetAll, GetById.
- Handlers implement `IHandler<TRequest, TResponse>`.
- Handlers return `Result<T>`.

## DTOs and validation

- Request and response models are C# records.
- Validate every request with FluentValidation.
- Place validators in the feature folder.

## Entities and errors

- Entities inherit `AuditableEntity` (`CreatedOn`, `UpdatedOn`), set by `AuditInterceptor`.
- Success: `return Result.Success(response);`
- Failure: `return Result.Failure<TResponse>(error);`

## API, database, and docs

- Endpoints use Minimal APIs, stay thin, delegate to handlers, and return correct status codes.
- MariaDB connection string lives in `appsettings.json`.
- Health: `/health`
- OpenAPI: `/openapi/v1.json`
- Scalar UI: `/scalar/v1`

Migration commands, when a migration is explicitly requested:

```text
dotnet ef migrations add MigrationName -p src/backend/Infrastructure -s src/WebApi
dotnet ef database update -p src/backend/Infrastructure -s src/WebApi
```

## Adding a feature

1. Create `Application/Features/NewFeature`.
2. Add handler, validator, and endpoint.
3. Define request and response records.
4. Implement business logic in the handler.
5. Add published-event handlers when the feature publishes events.

Auto-discovery registers handlers, validators, and endpoints.

## Generation rules

- Follow Clean Architecture boundaries.
- Use vertical slices, CQRS, FluentValidation, record DTOs, async database calls, and `Result<T>`.

## Do not modify

- `bin/`
- `obj/`
- `Migrations/`
- `*.csproj`
- `appsettings.Production.json`

Only modify source under Domain, Application, Infrastructure, and WebApi.

## Workflow

1. Analyze the feature.
2. Propose a plan.
3. Identify files to change.
4. Implement the solution.
5. Check these architecture rules.
