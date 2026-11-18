# Lumen Authorization Audit Checkpoint 5 — Classes, Assignments, Files and Account Administration

**Date:** 2026-11-17  
**Product source:** `HVA09/lumen-learning-platform`  
**Reviewed ref:** `main` at `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`  
**Method:** source and test-code review only; no tests were executed and no live environment was probed.

## Executive result

This pass reviewed class ownership and enrollment, assignment visibility/submission/grading, file attachment ownership and upload controls, admin user management, profile self-service, and authentication dependencies.

**No new confirmed IDOR or authorization bypass was identified in these reviewed paths.** Several strong negative tests already exist. A separate account-deletion data-integrity concern is recorded below; it is not an authorization finding.

## Findings by area

### 1. Classes and enrollments

- Teacher class detail/update/delete/invite-code regeneration/student removal all call `_get_owned_class_or_404`, which permits the class owner or admin.
- Class list is scoped by role: admins see all; teachers see their own; students see classes they joined.
- Student roster and mastery data are returned only through teacher-protected class detail.
- `POST /classes/join` rejects teacher/admin roles before enrollment side effects, rate-limits invite attempts, and binds the enrollment to `current_user.id`.
- Tests cover owner-only class detail/modification, role-scoped listing, invite-code secrecy, cross-teacher assignment visibility, outsider access denial, removed-student access revocation, rate limits and student-only joining.

No new authorization defect found.

### 2. Assignments and grading

- Assignment creation validates that the selected class belongs to the teacher unless the actor is admin; creator identity is set from the authenticated teacher.
- Student assignment visibility is based on class enrollment; teacher-created assignments are visible to their creator and admins.
- Submission routes enforce student role and class membership, and persist `student_user_id=current_user.id`.
- Submission listing and grading both call `_get_manageable_assignment`, which restricts management to the creator or admin. Grading additionally scopes the submission by both submission ID and assignment ID.
- Tests cover outsider denial, non-owner teacher denial, creator/admin management, student-only submission, grading role restrictions, and submission lifecycle.

No new authorization defect found. The earlier LUM-010 content ownership decision concerns curriculum lessons/questions and is separate from the assignment creator policy.

### 3. Files and uploads

- File list is filtered by `FileAttachment.user_id == current_user.id`.
- Download and delete use `_get_owned_file(file_id, current_user, db)`, which filters by both file ID and authenticated owner ID.
- Upload storage paths derive from the authenticated user's ID, not a client-provided owner field.
- Upload handling includes size/quota checks, signature/type validation, malware scanning, fail-closed scanner behavior, and cleanup of partial files.
- Tests explicitly verify that one user cannot access another user's file, plus malformed type/signature, malware, scanner failure, quota and partial-write behavior.

No new authorization defect found.

### 4. Admin operations and profile self-service

- Admin analytics, user listing/mutation/deletion and audit-log routes require `require_admin`.
- Role/activation changes and user deletion re-check the actor and target under the shared active-admin roster lock; the last active administrator is protected.
- Admin activation changes increment `token_version`, preventing old tokens from reviving after reactivation.
- Self-service profile endpoints use `/users/me` and the authenticated principal. `ProfileUpdate` permits only `full_name` and `avatar_url`; it does not accept role or active-state mutation.
- Password change verifies the current password and increments `token_version`.
- Account deletion reloads the account after the roster lock, checks active state/token version, re-verifies password, and protects the last active admin.

Tests cover admin-only access, self-role/deactivation restrictions, last-admin concurrency, self-delete, password change, account deletion and session invalidation.

No new authorization defect found.

## Separate data-integrity observation

In `backend/app/routers/users.py`, account deletion removes the user's upload directory from disk before committing the database deletion. If the subsequent database commit fails, the database account/attachment rows may remain while the physical files have already been removed. This is a potential failure-atomicity/data-consistency issue, not an IDOR or permission bypass.

Recommended future hardening: make file cleanup an idempotent post-commit operation (or durable cleanup job/outbox) and add a regression test simulating DB commit failure. Do not change this within the authorization-only review without a separate scoped PR.

## Follow-up matrix

| Area | Remaining verification | Priority |
|---|---|---|
| Classes | Existing cross-teacher/outsider/removed-member coverage appears substantive; execute full suite in CI | P1 release verification |
| Assignments | Existing creator/admin and class-membership negative tests; execute full suite | P1 release verification |
| Files | Existing cross-user denial and fail-closed upload tests; execute full suite | P1 release verification |
| Admin/account | Existing race/session tests; execute full suite against supported PostgreSQL path | P1 release verification |
| Account deletion | Move physical-file cleanup after successful DB commit using durable/idempotent cleanup; add commit-failure test | P2 reliability |
| Content ownership | Still requires explicit product decision before handler/schema change | P1 policy gate |
| Global leaderboard | Still requires explicit privacy/visibility decision | P1 policy gate |

## Limitations and boundaries

- This is source/test-code review; test suites were not executed in this pass.
- No live API probing, database access, production inspection, deployment change, or penetration test occurred.
- No Lumen application code, schema, migration, secrets, infrastructure, or production settings were changed.
- LUM-010 / S1-02 remains In Progress. This checkpoint does not close the broader route inventory or unresolved product-policy decisions.

## Source paths reviewed

- `backend/app/routers/classes.py`, `assignments.py`, `files.py`, `admin.py`, `users.py`
- `backend/app/dependencies.py`
- `backend/app/schemas/user.py`, `admin.py`
- `backend/app/models/user.py`
- `backend/tests/test_classes.py`, `test_assignments.py`, `test_files.py`, `test_admin.py`, `test_profile.py`, `test_account_security.py`
