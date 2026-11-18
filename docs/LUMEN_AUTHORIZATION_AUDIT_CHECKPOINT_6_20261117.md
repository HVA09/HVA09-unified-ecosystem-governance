# Lumen Authorization Audit Checkpoint 6 — Authentication, Sessions, CSRF and 2FA

**Date:** 2026-11-17  
**Product source:** `HVA09/lumen-learning-platform`  
**Reviewed ref:** `main` at `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`  
**Method:** source and test-code review only; no runtime tests or live auth probing were performed.

## Executive result

The reviewed authentication implementation uses server-side browser sessions, session-bound access/refresh JWTs, refresh-token rotation, refresh replay revocation, token-version invalidation, CSRF checks for cookie-authenticated mutations, trusted-origin checks on browser login, and encrypted TOTP secrets.

**No confirmed authentication bypass, session ownership flaw, or CSRF bypass was identified in this source review.** This does not certify production cookie/domain/TLS configuration or runtime behavior.

## Controls reviewed

### Login and account lockout

- Login rejects inactive accounts with the same generic credential error as invalid credentials.
- Password failures increment account failure counters and apply a temporary lock.
- When 2FA is enabled, login requires a TOTP code; invalid TOTP attempts count toward the same lockout threshold.
- Successful login clears failure state and creates an independent server-side browser session.
- First-user registration does not automatically grant admin, with tests for first and subsequent accounts.

### Session binding and refresh rotation

- Browser sessions are persisted in `AuthSession` and linked to the authenticated user.
- Access and refresh tokens carry session ID and token version; refresh tokens carry a unique JTI.
- `get_current_user` validates token type, active user, token version, and session ownership/revocation/expiry.
- Refresh selects the session row with `with_for_update()`, validates owner/state/version/JTI, rotates the stored JTI hash, extends expiry, and commits before setting new cookies.
- Reuse of an old rotated refresh token revokes that session.
- Logout revokes the matching device session; a regression test verifies logout of one device leaves another device session active.
- Password change/reset increments `token_version`, invalidating old access/refresh tokens. Password reset locks the user row and is single-use through version advancement.

### CSRF and origin controls

- Cookie-authenticated unsafe requests require a CSRF cookie/header match using constant-time comparison.
- Browser login checks the Origin against configured trusted origins and requires CSRF when Origin is supplied.
- Refresh and cookie logout require CSRF verification.
- Authentication cookies are HttpOnly; cookie security settings enforce HTTPS when SameSite=None is selected.

### Two-factor authentication

- TOTP setup/enable/disable endpoints require an authenticated user.
- An enabled TOTP secret cannot be replaced through setup; disable requires a valid current TOTP code.
- TOTP secrets are encrypted at rest with Fernet and fail closed if encryption configuration is absent or ciphertext cannot be decrypted.
- Login requires TOTP when enabled; invalid codes are rate-limited through account lockout.
- Legacy plaintext secrets are upgraded only after successful verification.
- Tests cover setup, enable/disable, invalid codes, lockout, encrypted storage, migration and fail-closed configuration.

## Remaining verification / release gates

| Area | Verification still needed | Priority |
|---|---|---|
| Browser cookies | Verify production Secure, SameSite, domain/path, HTTPS and trusted CORS origin values against deployed frontend/backend topology | P1 release gate |
| Session lifecycle | Run full auth tests on supported PostgreSQL configuration; verify migration 0022 and expiry cleanup operationally | P1 |
| Refresh concurrency | Add/execute a two-concurrent-refresh test proving row-lock serialization and replay behavior under PostgreSQL | P1 |
| CSRF | Browser E2E coverage for all cookie-authenticated unsafe routes and cross-origin rejection | P1 |
| 2FA recovery | Confirm operational recovery process for lost authenticator and TOTP key rotation/backup; avoid insecure admin bypass | P2 |
| Email flows | Verify real email provider delivery, token expiry and secret redaction in logs in staging | P1 release gate |

## Limitations and boundaries

- Source and test definitions were inspected; tests were not executed.
- No real browser, production cookie, email provider, authenticator device, or deployed origin was tested.
- No secrets, runtime configuration, application code, database, migration, or deployment were changed.
- The separate file-cleanup-before-DB-commit reliability observation remains in Checkpoint 5.
- LUM-010 / S1-02 remains In Progress; content ownership and global leaderboard privacy decisions remain open.

## Source paths reviewed

- `backend/app/auth/routes.py`
- `backend/app/auth/security.py`
- `backend/app/auth/sessions.py`
- `backend/app/auth/cookies.py`
- `backend/app/auth/two_factor.py`
- `backend/app/auth/totp_secrets.py`
- `backend/app/dependencies.py`
- `backend/app/models/auth_session.py`
- `backend/app/schemas/user.py`
- `backend/tests/test_auth.py`
- `backend/tests/test_two_factor.py`
- `backend/tests/test_totp_secrets.py`
- `backend/tests/test_account_security.py`
