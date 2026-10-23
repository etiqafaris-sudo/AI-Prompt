# Cursor Enterprise Stack Engineering Kit

A modular, security-first Cursor configuration pack for repositories using .NET / EF Core / Dapper / ADO.NET, Blazor, and Next.js.

## Layout
- `.cursor/rules/*.mdc`: short persistent guardrails, scoped by file globs.
- `.cursor/skills/<skill-name>/SKILL.md`: reusable multi-step workflows.
- `.cursor/agents/*.md`: focused specialist subagents (check your installed Cursor version's custom-agent support and import location).
- `docs/`: governance, secure-development checklist, architecture decision record template.

## Recommended installation
1. Back up any existing `.cursor` folder.
2. Copy `.cursor/` into the repository root. Merge rather than overwrite existing rules.
3. Keep only the rules relevant to the stack in that repository. For a Blazor-only or Next.js-only repository, remove the other frontend rule file or retain it with its scoped globs.
4. Commit the pack so the team gets the same guardrails.
5. Open Cursor and inspect Customize → Rules and Skills to confirm discovery.
6. Invoke workflows explicitly, e.g. `/security-review`, `/dotnet-feature`, `/nextjs-feature`, `/blazor-feature`, `/governance-review`.
7. Adapt commands, target framework, package versions, auth provider, database, deployment model, and regulatory requirements to the actual repository. This kit is a baseline, not a certification.

## Design principles
- Rules state *what must always be true*; skills explain *how to perform a repeatable task*.
- No agent can guarantee security or compliance from prompts alone. Enforce controls with tests, analyzers, CI policies, infrastructure controls, and human review.
- Never let AI run destructive production actions, exfiltrate secrets, execute untrusted scripts, or perform intrusive tests without explicit authorization and a safe target.
- Use the repository's existing architecture and versions before proposing migrations. Do not upgrade frameworks or packages opportunistically.

## Baseline toolchain to consider
- .NET: latest supported LTS chosen by the project, nullable enabled where feasible, analyzers, `dotnet format`, unit/integration tests, dependency vulnerability checks.
- Frontend: locked package manager and lockfile, TypeScript strict mode for Next.js, lint/typecheck/test/build in CI.
- Security: secret scanning, SAST, dependency/SCA scanning, DAST only against authorized non-production targets, threat modeling, SBOM and release provenance where feasible.
- Governance: PR review, ADRs, risk acceptance with expiry, data classification, change/release evidence, incident and vulnerability handling.

## Official references
- Cursor Rules: https://cursor.com/docs/context/rules
- Cursor Skills: https://cursor.com/docs/skills
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- Microsoft .NET security: https://learn.microsoft.com/dotnet/standard/security/
- EF Core performance: https://learn.microsoft.com/ef/core/performance/
- Next.js security: https://nextjs.org/docs/app/building-your-application/authentication
- Blazor security: https://learn.microsoft.com/aspnet/core/blazor/security/
