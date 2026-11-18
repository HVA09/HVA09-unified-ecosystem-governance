# Lumen Authorization Audit Checkpoint 3 — UI, E2E and Background Tasks

**Date:** 2026-11-17  
**Product source:** `HVA09/lumen-learning-platform`  
**Reviewed ref:** `main` at `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`  
**Scope:** source and test-code review only. No code execution, live penetration test, deployment inspection, database mutation, or production test was performed.

## Executive result

This pass reviewed the authenticated application shell, teacher-facing content UI, leaderboard and SRS UI, notification client lifecycle, browser smoke-test coverage, curriculum read/progress paths, and scheduled background tasks.

**No new confirmed IDOR or authorization bypass was identified in this pass.** The main remaining issues are incomplete negative-test coverage and unresolved product policy—not evidence that an exploit has been demonstrated.

LUM-010 / S1-02 remains **In Progress**. This checkpoint does not close the content ownership decision or the wider route inventory.

## Findings and evidence

### 1. UI role gating is present, but the real header lacks direct component tests

- `frontend/src/App.jsx` derives `isAdmin` from the authenticated user and `isTeacher` from teacher/admin roles.
- `frontend/src/components/AppHeader.jsx` renders the Admin entry only for admins and Content Manager only for teachers/admins.
- `frontend/src/App.test.jsx` verifies the role flags passed into the header, but mocks AppHeader. The current tree has no `AppHeader.test.jsx`; therefore it does not directly prove that the actual rendered header hides those controls for students.
- This is a **UI regression-test gap**, not an authorization boundary. Server routes must continue to enforce `require_admin` / `require_teacher` independently of hidden controls.

Recommended tests: render the real AppHeader for student, teacher and admin roles; assert Admin is admin-only and Content Manager is teacher/admin-only. Add an integration assertion that a student cannot reach privileged actions even if a UI callback is invoked directly.

### 2. Content Manager flows are tested; cross-teacher ownership remains unresolved

- `ContentManager.jsx` exposes lesson/question create, update, delete and analytics controls.
- `contentApi.js` calls the expected API endpoints; no client-side owner filtering is present.
- The backend `/content` router requires teacher/admin authentication. However, teacher-level resource ownership is not enforced in update/delete handlers, as already documented in Checkpoint 1 and the Content Ownership Decision Brief.
- `ContentManager.test.jsx` covers CRUD interactions and analytics rendering, not cross-teacher authorization.

This is the previously identified LUM-010 product ownership decision. Do not alter handlers or schema until the product owner explicitly chooses the shared-vs-owner-scoped policy. Once decided, add two-teacher allow/deny tests before implementation.

### 3. Leaderboard UI exposes a platform-wide scope by default

- `LeaderboardPanel.jsx` initializes scope to the empty value (platform-wide) and labels it “كل المنصة”; class scope is selected from `listClasses()`.
- It calls the server for the selected scope and displays server errors such as 403.
- `LeaderboardPanel.test.jsx` covers platform display, class selection, and surfaced server denial.
- The backend class leaderboard authorization checks were reviewed in Checkpoint 2. The platform-wide display remains a **product privacy decision**: whether names/scores are globally visible, opt-in, pseudonymous, or limited to a narrower population.

No frontend-only hiding is a substitute for server-side class scope checks. Product owner decision remains open.

### 4. SRS UI is user-oriented; negative cross-user tests remain a backend gate

- `FlashcardReview.jsx` calls the current-user deck/due-card/review APIs without accepting another user's identifier.
- `FlashcardReview.test.jsx` checks that a failed review does not advance the card.
- These UI properties do not establish server authorization. Checkpoint 2 already identified the required backend negative tests: user B cannot review/delete/export user A's cards, and imports are owned by the importer.

### 5. Notifications client lifecycle and reconnect behavior have meaningful coverage

- `NotificationBell.jsx` displays the notification list and delegates read operations through callbacks.
- `useNotifications.js` clears state when the authenticated session is disabled and guards asynchronous state changes against stale sessions.
- `useNotificationsSocket.js` parses messages and reconnects with bounded backoff; frontend hook tests cover retry/unmount behavior.
- `frontend/e2e/smoke_test.py` includes a browser test for transient WebSocket closure and reconnection through the configured Nginx proxy.
- These tests establish client recovery behavior, **not cross-user delivery isolation**. The A/B WebSocket negative test and REST list-isolation assertion remain required as recorded in Checkpoint 2.

### 6. Curriculum read/progress paths and scheduled tasks

- `backend/app/routers/curriculum.py`: learning-path mastery and lesson-progress queries filter by `current_user.id`; lesson completion creates progress for the authenticated user. Public subject/topic metadata is consistent with existing tests that explicitly assert public curriculum discovery.
- `backend/app/tasks.py`: weekly digest generation iterates active users and builds each report using that user's ID; due-card reminders group due cards by `ReviewCard.user_id` and notify only an existing active user for that ID.
- `backend/tests/test_tasks.py` covers activity-based digest dispatch, due-card reminder dispatch, and cleanup behavior. It does not constitute a full cross-user notification isolation test.
- No user-supplied identity parameter was observed in these scheduled task entry points during this review.

## Follow-up test matrix

| Area | Missing/next test | Priority |
|---|---|---|
| AppHeader | Real component role visibility for student/teacher/admin | P1 |
| Content Manager | Cross-teacher update/delete allow-deny tests after ownership policy approval | P1 |
| Global leaderboard | Product decision on visibility/consent; enforce and test selected policy | P1 |
| Notifications REST | Two users; each list excludes the other's notifications | P1 |
| Notifications WebSocket | Connect A and B; emit to A; assert B receives nothing | P1 |
| SRS | B cannot review/delete/export A's cards; importer owns imported cards | P1 |
| Reports | Two-user report isolation assertion | P1 |
| User lifecycle | Password change invalidates prior sessions; stale token rejected after deletion | P1 |

## Boundaries and limitations

- This is source/test-code review, not execution of the backend/frontend suites.
- Browser smoke-test code was inspected but not run in a live browser during this pass.
- No live API probing, penetration test, production data access, or deployment verification occurred.
- No application code, schema, migration, production account, secret, runtime setting, infrastructure, or deployment was changed.
- The content ownership policy and global leaderboard privacy policy remain unapproved.
- Keep LUM-010 / S1-02 In Progress until route/UI inventory, negative tests, and the two product-policy decisions are resolved.

## Source paths reviewed

- `frontend/src/App.jsx`
- `frontend/src/App.test.jsx`
- `frontend/src/components/AppHeader.jsx`
- `frontend/src/components/ContentManager.jsx` and `ContentManager.test.jsx`
- `frontend/src/components/LeaderboardPanel.jsx` and `LeaderboardPanel.test.jsx`
- `frontend/src/components/FlashcardReview.jsx` and `FlashcardReview.test.jsx`
- `frontend/src/features/notifications/components/NotificationBell.jsx`
- `frontend/src/features/notifications/hooks/useNotifications.js`
- `frontend/src/features/notifications/hooks/useNotificationsSocket.js` and tests
- `frontend/e2e/smoke_test.py`
- `backend/app/routers/curriculum.py`
- `backend/app/routers/content.py`
- `backend/app/tasks.py`
- `backend/tests/test_curriculum.py`, `test_content.py`, `test_tasks.py`
