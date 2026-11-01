---
name: docker-review
description: >-
  Review Dockerfiles, Compose files, and .dockerignore files for security,
  image size, layer caching, secrets in build context, privileges, health
  checks, graceful shutdown, .NET runtime and OS compatibility, and missing
  image scanning in CI. Use when the user asks for a container, Docker,
  Dockerfile, Compose, image-size, or container best-practices review, or
  when container build or runtime files change.
---

# Docker review

Read-only review of container build and runtime files. Do not edit files, build or push images, start containers, or scan registries during the initial review. Deliver findings, then a prioritized remediation plan. Implement fixes only after the user asks.

Read [references/checks.md](references/checks.md) for the check catalog. Apply only sections whose artifacts exist.

## Scope

In scope:

- Dockerfiles and Containerfiles, including multi-stage files and variants (`Dockerfile.dev`, `Dockerfile.ci`)
- Compose files (`compose.yaml`, `docker-compose*.yml`) and env files they reference by name
- `.dockerignore` and the `COPY`/`ADD` sources they fail to exclude
- Project files needed to judge runtime fit: `TargetFramework`, `nuget.config`, container CI workflows, and Docker-related docs

Out of scope unless the user expands the review: application vulnerability review, Kubernetes manifests, host firewall audit, and live registry scanning. Name those as not reviewed.

If no container files exist, say so and stop. Do not draft a Dockerfile as a substitute for the review.

## Workflow

1. Inventory every in-scope file. Record which compose file is dev-only versus production, and which Dockerfile stage is the final image.
2. Read the final base image reference and, for .NET, the `TargetFramework` before judging compatibility. Do not assume a tag's user, port, shell, or libc.
3. Walk [references/checks.md](references/checks.md). For each failed check, capture the evidence quote and the control you looked for and did not find.
4. Drop candidates that the calibration rules reject.
5. Write the report. Self-check against the exit gate before sending it.

## Evidence rules

- Every finding cites `path:line` or states the search that returned no match (`NOT FOUND: HEALTHCHECK in Dockerfile and compose`).
- A missing control is a finding only when no equivalent control exists in the files you inspected. A Compose `healthcheck` covers a Dockerfile without `HEALTHCHECK`. A dev override running as root is not a production-root finding.
- Build-stage root, package caches deleted in a later layer of the same stage, and `latest` on a local-only dev service are not production findings. Say which stage and which compose file.
- Do not invent CVEs, digests, support dates, or scanner results. If you did not run a scanner, label that check `NOT RUN`.
- Confirm current Microsoft support status from official docs when an EOL claim matters. If you did not check, confidence is `Needs validation`.
- Tag behavior (default user, port, shell, globalization) must come from that tag's Dockerfile or current Microsoft container docs. `USER app` is valid only where that image defines `app`.

## Severity and confidence

Severity is impact in the intended runtime, not how many guides mention the topic.

| Severity | Use when |
| --- | --- |
| Critical | A secret, key, or credential can enter an image layer or the committed build context; the production service mounts the Docker socket; or the production service is `privileged` and reachable beyond localhost |
| High | The final production image runs as root, ships the SDK, uses host networking or `SYS_ADMIN`, or targets a runtime major that does not match the app |
| Medium | Cache-busting or context bloat with a concrete size or rebuild cost; no health check on any inspected surface; no memory limit where the process can be OOM-killed; shell-form `CMD`/`ENTRYPOINT` on the final stage; scanner job absent from the production pipeline |
| Low | Hygiene with limited blast radius: missing image labels, unbounded log files, restart policy that hides crashes |
| Info | Documented tradeoff that is safe for the stated environment |

Confidence:

- **Confirmed** — the file content directly shows the issue
- **Likely** — the issue follows from a tag or orchestrator behavior you verified in docs
- **Needs validation** — the outcome depends on a host, registry, cluster, or pipeline you did not inspect

Do not mark `Confirmed` for anything you only suspect about the deployment environment.

## Calibration

Report these:

- Final stage `FROM` a `dotnet/sdk` image, or a published app copied into an SDK image
- `ARG`/`ENV` or a copied `nuget.config` carrying a credential that remains in a layer
- `COPY . .` while `.dockerignore` is missing or does not exclude `.git`, `**/bin`, `**/obj`, `.env`, `*.pfx`, `*.pem`, and private keys
- Production `user: root`, missing `USER` on a final image whose base is root, `privileged: true`, `docker.sock` mounted, or `cap_add: [SYS_ADMIN, NET_ADMIN]`
- Ports, health checks, or `ASPNETCORE_URLS` still aimed at port 80 while the final .NET 8+ image listens on 8080 as a non-root user
- `HEALTHCHECK` or Compose health check that calls a shell or `curl` on a chiseled or distroless final image
- Alpine final image with a native dependency that requires glibc, or culture-sensitive .NET without ICU and without an explicit invariant-globalization decision
- `CMD`/`ENTRYPOINT` in shell form in the final stage, so PID 1 is a shell and `SIGTERM` does not reach the app

Reject these:

- Port 8080 called "non-standard" for a current .NET aspnet image
- Alpine or chiseled called wrong solely because it is not Debian
- Root inside a build stage that is not the final image
- A missing Dockerfile `HEALTHCHECK` when Compose or the orchestrator probe already targets the same endpoint
- A missing scanner reported as an exploitable vulnerability. Absence of CI scanning is an operational gap
- Style nits with no behavior, size, or security consequence

## Report

```markdown
# Docker review — [repo or service]

## Scope
- Files reviewed:
- Runtime assumed: [local Compose / CI-built production image / not stated]
- Not reviewed:

## Findings
### D1 — [title]
- Severity:
- Confidence:
- Evidence:
- Impact:
- Fix:
- Verify:

## Checks not run
- [scanner, registry, host firewall, support-lifecycle lookup]

## Remediation plan
1. [finding ids] — [change] — [what could break] — [how to verify]
```

Order findings by severity, then by confidence. If there are no findings, say what you inspected and which checks were `NOT RUN`. Do not call the image secure or compliant.

## Remediation plan

Order work as: secrets and credentials, then privilege and exposure, then runtime compatibility, then health and shutdown, then layer size and cache, then CI scanning. Each item names the finding ids, the smallest change, the regression risk (port bind failure, culture changes, read-only filesystem breaking writes), and the verify step. Do not apply the plan in the same turn as the review.

## Exit gate

Send the report only after all of these are true:

- [ ] Every finding has a file location or an explicit `NOT FOUND`
- [ ] No finding is contradicted by another inspected file
- [ ] Final-stage base and, for .NET, `TargetFramework` were compared
- [ ] `.dockerignore` was judged against actual `COPY`/`ADD` sources
- [ ] Dev-only files were not graded as production
- [ ] Scanner and support claims you did not verify are marked `NOT RUN` or `Needs validation`
- [ ] No file was modified
