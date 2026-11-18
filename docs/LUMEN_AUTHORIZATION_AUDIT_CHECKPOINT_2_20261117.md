# Lumen — Authorization Audit Checkpoint 2

- **Date:** 17 November 2026
- **Scope:** Reports, notifications/WebSocket, gamification, spaced repetition, user self-service, admin analytics/user management, and LTI platform routes.
- **Status:** Partial source-and-test review. Not a penetration test, security certification, or complete endpoint inventory.
- **Source branch:** HVA09/lumen-learning-platform main as inspected 17 November 2026.
- **Change boundary:** Documentation only in the governance repository. No Lumen product code, schema, production data, secrets, configuration, or deployment changed.

## 1. Findings

| Area | Observed controls | Evidence and remaining test gaps | Assessment |
|---|---|---|---|
| Progress reports | Both endpoints derive the report recipient and report user ID from the authenticated user; service queries filter mastery, achievements, cards, submissions, quiz attempts and lesson progress by that user ID. | test_reports.py covers authentication and report content, but has no explicit two-user negative isolation test. | Strong direct self-scope in inspected implementation; add cross-user regression test. |
| Notifications REST | List filters by current user; mark-read lookup requires both notification ID and current user ID; read-all filters by current user. | test_notifications.py has a cross-user mark-read denial test. It does not explicitly assert that user B's list excludes user A's notification. | Positive isolation evidence; add list-isolation test. |
| Notifications WebSocket | Cookie-only access token; validates token type, active user, token version, server-side session ownership/revocation/expiry, and trusted Origin. ConnectionManager indexes sockets by user ID and sends only to that ID. | Tests cover invalid token, untrusted Origin, and multiple sessions for one user. No explicit cross-user non-delivery test found in inspected tests. | Positive design evidence; add cross-user delivery regression test. Manager is process-local; horizontal scaling requires shared pub/sub as documented in code. This is delivery reliability, not an observed authorization bypass. |
| Gamification | Personal stats and achievements use current user ID. Class leaderboard verifies admin, owning teacher, or enrolled student before returning class-scoped results. Global leaderboard intentionally returns ranked active students to any authenticated caller in the inspected route. | Tests cover authentication, verified score computation, hiding emails, and excluding staff. No explicit unauthorized class-leaderboard test was found in test_gamification.py. | Class gate present; document/confirm product privacy policy for global leaderboard and add negative class-access tests. |
| Spaced repetition | Cards list/due/export are scoped to current user; review/delete use `_get_owned_card()` with both card ID and user ID; import assigns current user ID and bounds file bytes/rows/field lengths. | test_srs.py covers CSV bounds, invalid topic rejection and valid import; no explicit cross-user review/delete/export isolation test found in the inspected test file. | Strong ownership predicates in inspected routes; add cross-user regression tests. |
| User self-service | Profile, profile update, password change, and account deletion operate on current_user, not a client-supplied user ID. Password change increments token_version; deletion reloads and checks account state/version before deleting. | Admin tests cover administrative account controls; no dedicated user self-delete race/session regression test was found in inspected paths. | No arbitrary-user target exposed by inspected self-service routes; add dedicated lifecycle tests. |
| Admin | Analytics, CSV export, user listing/mutation/deletion, and audit logs require admin. Role/active-state mutations and deletion use shared active-admin lock and fresh actor/target reads to protect the last active administrator. | test_admin.py covers role denial, roster race cases, audit logs, analytics, export and bounded pagination. | Strong evidence in inspected routes and tests. |
| LTI | Platform registration/listing require admin. Launch binds state to platform, validates signed claims/target link and creates one-time hashed exchange codes. Exchange checks trusted Origin when present, code expiry/consumption, user active state and token_version. | test_lti.py covers admin-only platform setup, signature/state/target mismatch, code invalidation after password change, endpoint SSRF validation, and browser Origin/CSRF. | Strong evidence in inspected flow. Full Moodle/Canvas interoperability and remaining LTI acceptance gates remain separate from this authorization checkpoint. |

## 2. Source evidence inspected

- `backend/app/routers/reports.py` — 1fad80cbeea8adf0444b942c9945945bf249738b
- `backend/app/services/reports.py` — 255037f47f896f1cb4dc11ae0c071a27907e6d70
- `backend/app/routers/notifications.py` — 14f1a7b5c29d7d8bb0de0b98f22e9c1265e524d9
- `backend/app/notifications.py` — 5c3d5308d0d60f6a8f3a5497f4e7300d26755af2
- `backend/app/routers/gamification.py` — 3034eeeaae9c71207d1b2d3327932c5f33a482d6
- `backend/app/routers/srs.py` — 2603f3c396fb18197114602ed914ca248beddbf2
- `backend/app/routers/users.py` — a698af942f7e7486304720b486a58dabe4019f89
- `backend/app/routers/admin.py` — 4fe164da9df4ec9ba69c8036e3eeed2e76312ed2
- `backend/app/routers/lti.py` — 0e23a0c1088f6c68a0a8af622aa02cb45794d655
- `backend/tests/test_reports.py` — 7bf3f164158d524d441d81564117a649ef9797ed
- `backend/tests/test_notifications.py` — 4bdf457afda23048fcf7c521451441b1e55d4456
- `backend/tests/test_gamification.py` — 0a542fc3d7f020fba3aca1cbec3a20c2e06865da
- `backend/tests/test_srs.py` — e5e0ec285582b4d920c79ffd6739a5499e3a875d
- `backend/tests/test_admin.py` — b117c46996844e3f3fbd0352605f2ee90c626258
- `backend/tests/test_lti.py` — 84731b730f3c62ffa6f913c3ba450b639efc334d
- `frontend/src/features/notifications/hooks/useNotificationsSocket.test.jsx` — 457d2b907474bfd1381bf7de5e85f50b791885f0

## 3. Follow-up tests recommended

1. Reports: create two users with different mastery/submission/activity records and assert each report contains only the caller's data.
2. Notifications REST: create notifications for A and B; assert each list is isolated and B cannot mark A's notification read.
3. Notifications WebSocket: connect A and B simultaneously, emit an event for A, assert A receives it and B receives nothing.
4. Gamification: verify a student outside a class and a non-owning teacher cannot read its class leaderboard; verify owner/admin/enrolled student can.
5. Spaced repetition: verify B cannot review/delete A's card and cannot export A's cards; verify imports are always owned by the authenticated importer.
6. User lifecycle: verify password change invalidates prior sessions and account deletion rejects stale token/session state.
7. Global leaderboard: obtain explicit product-owner decision on whether platform-wide names/scores are public to all authenticated accounts or should be role/consent/scoped.

## 4. Disposition

- No confirmed IDOR was established in these inspected routes.
- Several explicit two-user negative tests remain absent; absence of a test is a coverage gap, not proof of an exploit.
- The global leaderboard's visibility is a product privacy decision, not automatically a vulnerability.
- Keep LUM-010 / S1-02 **In Progress**. Curriculum content ownership remains unresolved in `LUMEN_CONTENT_OWNERSHIP_DECISION_BRIEF.md`; the earlier route-by-route inventory and UI review also remain open.
- No live production testing, deployment, migration, or data mutation was performed.