---
name: nextjs-feature
description: Build a secure, performant, accessible Next.js feature with correct Server/Client Component boundaries and authorization.
paths:
  - "**/*.tsx"
  - "**/*.ts"
---
# Next.js feature workflow

1. Inspect Next.js version, router model, auth/session provider, package manager, styling system, test tooling, and deployment runtime.
2. Prefer Server Components. Add Client Components only for browser APIs or interaction; keep the client boundary narrow.
3. Define route/data contracts, validation, auth, resource ownership, tenant scope, caching, loading/error/empty states, and accessibility.
4. Treat Route Handlers and Server Actions as public endpoints. Re-authenticate/authorize and validate every mutation; do not rely on layout or middleware alone.
5. Keep secrets and privileged data access server-only. Validate environment variables; never expose secrets through `NEXT_PUBLIC_` or serialized props.
6. Use runtime schemas for untrusted form/query/cookie/API data. Typescript types are not runtime validation.
7. Set caching/revalidation deliberately. Avoid shared caching of personalized or tenant-scoped data unless isolation is demonstrably correct.
8. Avoid unsafe HTML, open redirects, SSRF, permissive CORS, and missing CSRF defenses for cookie-authenticated mutations.
9. Use semantic HTML, keyboard support, labels, focus management, responsive layouts, and reduced-motion considerations.
10. Run lint, typecheck, tests, and production build. Add tests for denied access, cross-tenant access, invalid payloads, and stale/cached data leakage.
