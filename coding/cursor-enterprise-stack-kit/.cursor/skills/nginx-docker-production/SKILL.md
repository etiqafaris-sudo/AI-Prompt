---
name: nginx-docker-production
description: >-
  Production nginx.conf tuning inside Docker containers: non-root images,
  unprivileged ports, worker/event limits for cgroup CPU, gzip and cache headers
  for static SPAs and Blazor WASM, reverse-proxy upstream blocks, health checks,
  graceful shutdown, logging, and Compose/Kubernetes wiring. Use when editing
  nginx.conf copied into a Dockerfile, reviewing SmartEdu-style WASM nginx
  images, or proxying to Kestrel/YARP backends in Docker.
---

# Nginx in Docker (production)

Complements the **nginx** skill (general nginx). Pair with **yarp** when the edge is ASP.NET Core and static assets sit behind nginx.

## Workflow

1. Identify role: **static origin** (SPA/WASM), **TLS edge**, or **reverse proxy** to app containers — do not mix patterns without explicit `location` splits.
2. Read the final Docker stage: base image tag, `USER`, exposed port, config path, volume mounts, healthcheck.
3. Validate nginx for container constraints (non-root, writable `pid`/`client_body_temp`, no `listen 80` without capability drop review).
4. Tune workers and gzip against measured container CPU/memory limits.
5. Cross-check Compose service names, upstream URLs, and `proxy_set_header` forwarding when TLS terminates in front of the gateway.

## Container invariants

| Concern | Production default |
| --- | --- |
| User | Run as image `nginx` user or custom non-root; never rely on root in final stage |
| Listen | Unprivileged port (e.g. `8080`); terminate TLS at ingress/load balancer or dedicated sidecar |
| PID | `pid /tmp/nginx.pid` (or `/var/run/nginx.pid` only if volume-owned) |
| Config | `COPY` into image or read-only bind mount; run `nginx -t` in CI |
| Logs | Stream to stdout/stderr or ship via logging driver; set `error_log warn` unless debugging |
| Workers | `worker_processes auto` is fine when CPU limit ≈ host; under tight k8s limits prefer explicit count matching CPU quota |
| Connections | Size `worker_connections` against `ulimit -n` and upstream keepalive pools |
| Secrets | No TLS keys in layers; mount certs at runtime |

## Dockerfile checklist

- Pin image tag (`nginx:1.27-alpine`), not bare `latest` in production pipelines.
- `chown` cache/log/temp paths used by non-root nginx.
- `EXPOSE` matches `listen` in config.
- `STOPSIGNAL SIGQUIT` (nginx default) for graceful worker drain; align Compose `stop_grace_period`.
- Add `HEALTHCHECK` hitting a lightweight path (e.g. `/` or dedicated `/healthz`).

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/ || exit 1
```

## Static SPA / Blazor WASM

- `server_tokens off`; security headers on all responses (`nosniff`, frame policy, referrer).
- `gzip` + `gzip_static on` for `.js`, `.wasm`, `.css`; **do not gzip** already-compressed assets twice without reason.
- Long-cache immutable hashes under `/_framework/`; short or no-cache for `appsettings.json` and boot config.
- `try_files $uri $uri/ /index.html` only for client-router paths; explicit `location =` for sensitive JSON.
- See [references/blazor-wasm-static.md](references/blazor-wasm-static.md).

## Reverse proxy to .NET (Kestrel / YARP)

Place nginx in front only when you need TLS, rate limits, or static/asset split — otherwise YARP may be sufficient internally.

Minimum forward headers:

```nginx
proxy_http_version 1.1;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header Connection "";
```

- Set `proxy_read_timeout` / `proxy_send_timeout` to match longest API operation (file uploads, reports).
- Use `upstream` with `keepalive` for high RPS to Kestrel.
- WebSockets: `proxy_set_header Upgrade $http_upgrade;` and `proxy_set_header Connection "upgrade";`.
- See **yarp** skill for gateway rate limits, route `Order`, and active health checks on clusters.

## Performance tuning (container-aware)

- `sendfile on`, `tcp_nopush on`, `tcp_nodelay on` for static.
- Cap `gzip_comp_level` at 5–6; WASM is often pre-compressed — measure before aggressive gzip.
- Avoid oversized `client_max_body_size` on static-only containers.
- If CPU throttled, reduce `worker_connections` before adding workers.

## Security

- No `add_header` duplication bugs: use `always` where overrides must apply on errors.
- Restrict admin/stub_status; never expose on public listeners.
- Rate limiting (`limit_req`) at edge for anonymous endpoints when gateway does not cover them.
- Read-only root filesystem where orchestrator allows; writable only for cache/pid paths.

## Validation

```bash
docker run --rm -v "$(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro" nginx:1.27-alpine nginx -t
```

## Related skills

- **nginx** — general config, TLS, upstreams, caching (`references/`).
- **yarp** — .NET reverse proxy gateway, routes/clusters, health checks, metrics.
- **docker-review** — Dockerfile/Compose security and production hygiene.

## Provenance

Authored for Cursor global install; patterns align with Microsoft YARP docs and nginx official Docker image behavior.
