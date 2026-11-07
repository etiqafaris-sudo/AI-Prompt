# Blazor WASM static nginx reference

## Cache map pattern

```nginx
map $sent_http_content_type $static_cache_control {
    default                    "public, max-age=3600";
    "application/wasm"         "public, max-age=31536000, immutable";
    "application/octet-stream" "public, max-age=31536000, immutable";
}

location /_framework/ {
    add_header Cache-Control $static_cache_control always;
    try_files $uri =404;
}
```

## Runtime config

```nginx
location = /appsettings.json {
    add_header Cache-Control "no-store, no-cache, must-revalidate" always;
    try_files $uri =404;
}
```

## Entrypoint-generated config

When `docker-entrypoint.sh` writes `appsettings.json` at start:

- Keep generation idempotent and fail fast on missing env vars.
- Do not log secret values.
- Ensure file ownership matches `nginx` user.

## MIME

Official `mime.types` includes `application/wasm`. If serving custom extensions, extend types explicitly.
