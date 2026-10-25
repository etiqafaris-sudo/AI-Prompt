---
name: blazor-feature
description: Build a secure, accessible, maintainable Blazor feature for the repository's actual render mode and authentication architecture.
disable-model-invocation: true
---
# Blazor feature workflow

## When to use

- Blazor UI work (`.razor`, Blazor-related `.cs` files).

1. Identify Blazor Server, WebAssembly, or Blazor Web App render mode; inspect auth, DI lifetimes, component library, routing, and test setup.
2. Define user journeys, roles/resource permissions, loading/empty/error states, validation, accessibility, and mobile behavior.
3. Put business logic in application services, not markup. Keep components focused and avoid premature abstraction.
4. Enforce authorization in server-side use cases/data access. UI-level `AuthorizeView` only controls presentation.
5. Never expose secrets or privileged server services to WebAssembly. Treat browser state as untrusted.
6. Use correct scoped/circuit-safe lifetimes; prevent per-user data from leaking through singleton state.
7. Use async event handlers, cancellation where appropriate, prevent double submissions, and dispose event/timer/JS interop subscriptions.
8. Validate on both client and server. Provide accessible labels, keyboard navigation, focus behavior, and validation messages.
9. Add component tests for critical UI states and integration tests for protected operations.
10. Run build/tests and review auth boundaries, data leakage, render-mode behavior, and error handling.
