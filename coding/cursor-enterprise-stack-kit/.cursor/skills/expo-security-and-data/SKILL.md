---
name: expo-security-and-data
description: Apply Expo and React Native security and data-governance rules for secrets, tokens, local storage, personal data, network calls, permissions, analytics, and OTA updates. Use when building or reviewing an Expo or React Native app that handles authentication, SecureStore, AsyncStorage, MMKV, SQLite, EXPO_PUBLIC env vars, analytics, crash reports, permissions, or EAS Update.
---

# Expo security and data governance

Official Expo skills do not cover this. Apply it together with `expo-data-fetching` for network code and `security-and-privacy` for the server side.

The client bundle, logs, and plaintext storage are readable. Treat the app as an untrusted environment.

## Classify before storing

| Data | Store | Never |
| --- | --- | --- |
| Access tokens, refresh tokens, session secrets | `expo-secure-store` | AsyncStorage, plaintext MMKV, SQLite, Redux persist, logs, `EXPO_PUBLIC_*` |
| Personal data that must stay on device | Encrypted store; key in SecureStore | Analytics, crash payloads, screenshots, test fixtures, git |
| Preferences, flags, non-sensitive cache | AsyncStorage or MMKV | Auth material in the same record or persist whitelist |

SecureStore values are small (about 2KB on iOS). Keep a key there and encrypt larger blobs separately. Delete every sensitive key on logout and account deletion.

## Secrets and config

- `EXPO_PUBLIC_*` is inlined into the app binary. Use it only for public config such as an API base URL.
- Keep service credentials, database passwords, and private API keys on the server or in EAS environment secrets without the `EXPO_PUBLIC_` prefix.
- Do not commit `.env` files that contain secrets. Do not print tokens, authorization headers, or personal data.

## Network and session

- HTTPS only. Leave certificate checks on. Do not enable cleartext traffic.
- Read the access token from SecureStore at request time. Send it in the `Authorization` header.
- Bound timeouts, retries, and cached personal data. Cached responses must not outlive the session when they contain personal data.

## Collection and retention

- Collect the minimum personal data the feature needs, for a stated purpose.
- Define where it lives, who can read it, and when it is deleted.
- On logout: clear SecureStore secrets, in-memory session state, and personal query caches.
- Keep store privacy labels and permission prompts aligned with what the app actually collects.
- Request a permission in context, and only the permission the feature uses.

## Supply chain and updates

- Add Expo packages with `npx expo install` so versions match the SDK.
- Review native modules before adding them. Pin versions.
- Keep EAS Update runtime-version checks. Do not disable update signature verification when the project uses it.
