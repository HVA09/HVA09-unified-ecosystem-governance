# Lumen Authorization & Recovery Audit — Checkpoint 7 (2026-11-17)

## Scope and evidence

Read-only source review of Lumen `main` at `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`. Reviewed backup/restore scripts, Docker Compose volume topology, file routes/tests, Celery tasks/tests, LTI routes/service/network tests, billing webhook tests, frontend file API and LTI callback. No product code, database, secrets, or deployment was changed. No tests were executed in this review; references to tests below mean source-level inspection only.

## Findings

### LUM-011 — Uploaded-file volume is not included in the documented backup/restore chain
**Priority: P1 — recovery/data-integrity release gate**

- `backend/docker-compose.yml` persists uploads in the named `uploads` volume mounted at `/app/uploads`.
- The backup service mounts only `./backups:/backups` and runs `backend/scripts/backup.sh`, which creates a PostgreSQL dump using `pg_dump`.
- The optional offsite Restic service mounts only `./backups:/backups:ro`; it uploads PostgreSQL archives, not the `uploads` volume.
- `restore.sh` restores PostgreSQL only. The integration drill validates a database table only and does not exercise uploaded files.
- Therefore a successful database restore can restore `FileAttachment` metadata while the corresponding physical files remain absent if the uploads volume is lost. This is a recovery gap, not an authorization bypass.

**Required before production recovery is considered complete:** define a consistent backup for the uploads volume (or object storage), encrypt it at rest/offsite, preserve file ownership/permissions, and create a coordinated restore drill that verifies metadata-to-file integrity, including missing/orphan files. Define ordering/consistency around concurrent uploads and deletions. Do not assume database-only backups are full application backups.

### LUM-012 — Restore path accepts symlinks within the backup directory
**Priority: P2 — local operational hardening**

- `restore.sh` checks that the supplied string matches `$BACKUP_DIR/backup_*.sql.gz` and that `-f` succeeds, then validates/decompresses it.
- A symlink located under `BACKUP_DIR` can satisfy these checks while resolving to a file outside the directory. This requires local write access to the backup directory or control of the selected path; it is not remotely reachable through an application route in the reviewed source.
- Harden by rejecting symlinks (e.g. `-L`), resolving/canonicalizing the path and verifying its parent directory, and restricting restore execution to an operator-controlled path. Add tests for symlink-to-outside and symlink-to-directory cases.

### LUM-013 — Local PostgreSQL archives are not encrypted by the backup script
**Priority: P1/P2 depending on host/storage threat model**

- `backup.sh` uses `umask 077`, temporary files, gzip integrity validation, and atomic publication; these are useful integrity/permission controls.
- gzip is compression, not encryption. The optional Restic path is encrypted, but it is a separate profile and does not change the local archive's plaintext-at-rest nature. Host disk/volume encryption and access controls are not established by source review.
- Confirm host-level encrypted storage and backup-directory access controls, or encrypt local archives before they are persisted. Keep Restic credentials outside the application `.env`; the Compose configuration already mounts a dedicated password secret read-only.

### Export/import and alternate application routes

- The backend router inventory contains no generic user-facing database export, account export, backup, or restore endpoint. The reviewed `/files` list/download/delete paths are authenticated and file-ID + current-user scoped via `_get_owned_file`; storage paths are constructed from the authenticated user's ID and stored filename.
- SRS import is an authenticated user-scoped workflow (reviewed in earlier checkpoints); no separate cross-user export/import route was found in the router inventory. This is a source inventory conclusion, not proof that no external operational script or deployment-side process exists.
- The frontend file download uses the authenticated Axios client and retrieves a blob; it does not construct a public static-file URL. No direct public upload-volume mount is defined in the reviewed Compose file.
- File metadata and physical content have separate lifecycle operations. The previously documented account-deletion ordering issue remains separate: deleting physical files before the database commit can leave metadata without files if the commit fails.

### LTI / external callbacks / payment webhooks / background workers

- LTI login initiation is available through GET and POST and is unauthenticated by design; it requires a registered issuer/client, exact configured target URI in production, safe HTTPS platform endpoints, and validated deployment ID.
- LTI launch locks and consumes a hashed, expiring state; verifies the platform ID token; issues a hashed, short-lived one-time exchange code; and places the code in the browser URL fragment. Exchange locks/consumes the code, checks active user and token_version, applies trusted-Origin/CSRF checks when Origin is present, then issues HttpOnly session cookies. Frontend callback removes the fragment from browser history before exchanging the code.
- The LTI service resolves DNS, rejects non-public resolved addresses, pins the selected IP, disables redirects and environment proxy trust, and bounds JWKS response size. Earlier LTI review findings (raw `lti_subject` privacy, same-origin/clock-skew/real Moodle+Canvas acceptance) remain open; this checkpoint does not close them.
- Stripe/PayPal webhook verification and event-id idempotency are covered by source tests reviewed in earlier checkpoints; this pass found no new webhook authorization issue.
- Celery weekly digests query active users and build reports per user ID; due-card reminders derive distinct user IDs from due cards, reload each user, and notify only existing active users. No arbitrary user ID is accepted from a public task endpoint in the reviewed task module. Queue fallback to direct email is a reliability behavior, not an authorization finding.
- No new confirmed IDOR or authorization bypass was identified in this checkpoint.

## Decisions and remaining gates

1. Treat LUM-011 as a P1 recovery gate: add uploads-volume backup/restore and a combined PostgreSQL + uploads recovery drill.
2. Harden restore symlink/path handling (LUM-012) before delegating restore execution to less-trusted operators.
3. Explicitly decide whether local backup storage must be encrypted (LUM-013) based on the production host threat model; Restic offsite encryption alone is not local encryption.
4. Keep the earlier LUM-010 content-ownership and global-leaderboard privacy decisions open; do not change policy/schema until product-owner approval.
5. Run the backup/restore, file isolation, LTI, webhook, and worker tests in CI on the exact current commit; then perform a disposable staging restore drill. This audit did not run tests or access production.
6. Keep product changes out of the governance repository. Implement any approved remediation as small, isolated PRs in the Lumen repository.

## Evidence reviewed

- `backend/docker-compose.yml`
- `backend/scripts/backup.sh`, `offsite-backup.sh`, `restore.sh`
- `backend/scripts/test_backup_restore_integration.py`
- `backend/tests/test_backup_scripts.py`, `test_offsite_backup.py`, `test_files.py`, `test_lti.py`, `test_lti_network.py`, `test_payment_provider_webhooks.py`, `test_tasks.py`
- `backend/app/routers/files.py`, `backend/app/routers/lti.py`, `backend/app/services/lti_service.py`, `backend/app/tasks.py`
- `frontend/src/lib/filesApi.js`, `frontend/src/components/LTICallbackPage.jsx`

**Conclusion:** no new confirmed cross-user authorization bypass was found in the reviewed surfaces. The most material new issue is incomplete disaster-recovery coverage for the persistent uploads volume. This is a source review checkpoint, not a security certification or a production recovery test.
