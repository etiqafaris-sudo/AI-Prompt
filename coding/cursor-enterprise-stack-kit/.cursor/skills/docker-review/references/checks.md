# Docker review checks

Apply a check only when its artifact exists. Pair every hit with the evidence rules in `SKILL.md`.

## Dockerfile

### Base image and stages

- The last `FROM` is the shipping image. Earlier stages may be SDK, Node, or build tools.
- Pin a major (and patch when the repo already pins patches). Flag `latest` and floating tags on the final stage of a production Dockerfile.
- Prefer a digest pin when the rest of the repo pins digests. Do not demand a new digest scheme the repo does not use; report floating tags as Medium unless the tag is `latest` on a production final stage (High).
- Multi-stage production .NET builds use `dotnet/sdk` only before the final stage and `dotnet/aspnet` or `dotnet/runtime` for the final stage. Match the image major to `TargetFramework` (`net8.0` → `8.0`, `net10.0` → `10.0`).
- Windows versus Linux base must match the host the compose or CI file actually targets. Do not mix a Linux final stage with a Windows-only dependency you can see in the project.
- `ADD` of a remote URL, `curl | sh`, and `wget | bash` in any stage are High. Use a pinned copy or a package from the base image's manager, then delete lists in the same `RUN`.

### .NET runtime fit

Verify against the final tag's docs, not from memory:

- .NET 8 and later official Ubuntu images define user `app` (`APP_UID` 1654) and default HTTP to port 8080. A `USER` that names a user the tag does not create will fail at start. Binding port 80 as that user will fail.
- Chiseled and distroless final images have no package manager and no shell. `HEALTHCHECK CMD-SHELL`, `curl`, and `wget` will not run. Use a process that exists in the image, or a Compose/orchestrator probe that does not need a shell.
- Alpine final images use musl. Flag them when the repo has glibc-only native assets. If the app formats cultures, dates, or sorts user text, the image needs ICU or the project must set invariant globalization on purpose. Do not add ICU and invariant mode together without saying which behavior the app needs.
- Self-contained publish (`-r linux-x64 --self-contained true`) does not belong on a framework-dependent aspnet image unless the Dockerfile is the publish vehicle. Framework-dependent output needs the matching runtime image. `UseAppHost=false` is appropriate for framework-dependent Linux images.
- `dotnet publish -c Release` output is what the final stage copies. Debug publish in a production Dockerfile is Medium.
- Final stage sets `ASPNETCORE_ENVIRONMENT` only when the compose or orchestrator does not already set it. Do not bake `Development` into the production image.
- Data-protection keys, logs, and uploads need a writable path or volume if the filesystem is read-only. Call that out when recommending `read_only`.

### Cache and size

- Restore cache: copy project files (and `nuget.config`) , `dotnet restore`, then copy the remaining source. A `COPY . .` before restore busts the restore layer on every source edit.
- One `RUN` per package install, with cache cleanup in that same instruction. A later `RUN rm` does not shrink the previous layer.
- NuGet and apt/apk caches use BuildKit cache mounts when the Dockerfile already uses BuildKit syntax. Do not require a rewrite of a single-stage sample that is not the production image.
- Final stage does not contain the SDK, tests, `.git`, `obj`, or source unless the image is a build image.
- `.dockerignore` exists beside the build context. It excludes `.git`, `**/bin`, `**/obj`, `**/.vs`, `*.user`, `.env`, `appsettings.*.local.json`, `*.pfx`, `*.pem`, `*.key`, and test projects that the Dockerfile does not need. Compare ignore rules to each `COPY`/`ADD`. A dockerignore in the wrong directory does not apply.
- Do not copy the Docker socket, cloud credentials, or SSH agent socket into the image.

### Secrets in layers

- `ARG` and `ENV` values that look like tokens, passwords, or NuGet API keys are Critical if any later instruction can persist them. Unsetting an env var in a later layer leaves it in the earlier layer.
- Private feeds use a BuildKit secret mount (`--mount=type=secret`) or an external credential helper. A copied `nuget.config` with a cleartext password is Critical.
- `COPY` of `.env`, `secrets.json`, user secrets, or certificate files into any stage is Critical even if a later stage does not copy them forward, when that stage is published or cached.

### Privileges, network, shutdown

- Final stage sets `USER` to a non-root user that exists in that base. `chmod 777` and world-writable app directories are High.
- `EXPOSE` documents the port the process listens on. It is not itself exposure. Pair it with Compose `ports`.
- Exec-form `ENTRYPOINT` and `CMD` (`["dotnet", "App.dll"]`). Shell form wraps the process and breaks graceful shutdown. `STOPSIGNAL SIGTERM` unless the app documents another signal.
- A `HEALTHCHECK` uses the listen port and path the app actually serves, with a start period long enough for cold start. Do not flag a missing Dockerfile `HEALTHCHECK` when another inspected surface probes the same endpoint.

## Compose

Grade the production file and the dev file separately. Prefix finding titles with the file's role.

- No `privileged: true`. No `network_mode: host`, `pid: host`, or `ipc: host` for the app unless the file documents a hardware or protocol need. Those are High.
- No bind mount of `/var/run/docker.sock`. That is Critical.
- `cap_drop: ["ALL"]`, then add back only the capabilities the process needs. `cap_add: SYS_ADMIN` or `NET_ADMIN` is High without a stated need.
- `security_opt: ["no-new-privileges:true"]` on production services.
- `user` matches the image's non-root user. Do not set `user: root` to paper over file permissions.
- `read_only: true` plus an explicit tmpfs or volume for paths the app writes. Recommend it only after naming those paths. Missing `read_only` alone is Low.
- Published ports bind to localhost when the service is reached through a local proxy. `0.0.0.0` publication of an admin or database port is High on a production compose file. Application HTTP published on a dev compose file is Info.
- `mem_limit` and `cpus` (or `deploy.resources`) set on production services. .NET honors cgroup memory limits; an unbounded service is Medium because the GC will not see a cap the host did not set.
- `healthcheck` hits the real port and path. `depends_on` uses `condition: service_healthy` when startup order matters.
- `stop_grace_period` is at least the app's shutdown timeout (ASP.NET Core `ShutdownTimeout`, default 30 seconds, is longer than Compose's default 10 seconds).
- `environment` and `env_file` do not commit live passwords, connection strings with credentials, or API keys. Placeholder names are Info. Real secrets in a tracked compose or env file are Critical.
- `restart: always` on a crashing worker hides failures; prefer a policy the operator already uses, and mention log rotation (`logging.options.max-size`) as Low when logs are unbounded.

## CI and supply chain

Inspect the workflow that builds or pushes the image. If you cannot find one, record `NOT FOUND` and stop. Do not claim a scan ran.

- The workflow builds the production Dockerfile, not the dev one, for the pushed tag.
- Image scan (Trivy, Grype, Docker Scout, or the repo's existing scanner) runs on the built image and fails the job on the repo's stated severity threshold. A missing scanner is Medium. `|| true` or `continue-on-error` that hides scanner failure is High.
- Do not invent CVE findings from the scanner's absence.
- Dockerfile lint in CI is Low when missing. Do not block a review on Hadolint style rules that the calibration section rejects.

## .dockerignore

- File is at the context root used by `docker build` / Compose `build.context`.
- Negation rules (`!`) do not re-include `.env`, keys, or `.git`.
- Ignore file itself is not a secret store.
- After listing excluded patterns, name any sensitive path that is still copied. That pairing is the finding; a missing ignore file with a one-folder `COPY` of a known project is Low, not Critical, unless a secret file sits in that folder.
