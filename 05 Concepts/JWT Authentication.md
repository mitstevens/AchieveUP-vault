---
tags: [concept, security]
---

# JWT Authentication

**JSON Web Token** — a signed (not encrypted) string proving "the server vouches for these claims." Three base64 parts: `header.payload.signature`. The payload typically holds the user id, role, and an expiry (`exp`). The signature is an HMAC over the first two parts using a server secret — anyone can *read* a JWT, but nobody can *forge or alter* one without the secret.

## Why use it
Statelessness: the server doesn't store sessions. Every request carries the token; the server just verifies the signature. Scales easily, works across services.

## In this project
- **Issued:** `POST /auth/login` / `/auth/signup` → `achieveup_auth_service.py` signs a JWT with `ACHIEVEUP_JWT_SECRET`.
- **Stored:** frontend `localStorage` under `token` ([[Frontend Auth Flow]]).
- **Sent:** axios request interceptor adds `Authorization: Bearer <token>` ([[Frontend API Layer]]).
- **Verified:** protected backend routes decode + verify before handling.
- **Expiry/refresh:** `/auth/refresh-token` exists; a 401 anywhere logs the user out client-side.

## Security notes for the team
- ⚠️ `config.py` has a **hardcoded fallback secret** (`"achieveup-secret-key-change-in-production"`). If the env var is ever unset in prod, tokens become forgeable. Verify Heroku config.
- `localStorage` storage is XSS-readable; httpOnly cookies are the safer pattern (known trade-off, fine for now — just don't introduce untrusted HTML rendering).
- The JWT is for **AchieveUp accounts only**; the extension uses raw Canvas tokens instead — see [[Flow - Authentication]] for the two-system split.
