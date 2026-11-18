# Lumen — Authorization and IDOR Audit Checkpoint

- **Date:** 17 November 2026
- **Task:** LUM-010 / S1-02
- **Status:** Partial audit checkpoint; not a security certification and not closure evidence.
- **Application source:** HVA09/lumen-learning-platform, main as inspected on 17 November 2026. Relevant source blob SHAs are recorded below.
- **Change boundary:** Documentation only in the governance repository. No Lumen code, schema, database, production data, secrets, runtime configuration or deployment changed.

## 1. Executive result

The inspected resource domains generally apply explicit per-user, per-teacher, or creator-based access checks. The previously documented curriculum-content ownership gap remains the main unresolved authorization-policy issue in this checkpoint.

| Domain | Observed server-side boundary | Test evidence | Assessment |
|---|---|---|---|
| Uploaded files | List is filtered by FileAttachment.user_id; download and delete use _get_owned_file() and return 404 when the file is not owned by the caller. | backend/tests/test_files.py::test_cannot_access_other_users_file checks cross-user download and delete denial. | Positive evidence for per-user isolation in inspected routes. |
| Classes | Teacher list is scoped to teacher_user_id; class detail/update/delete/student removal use _get_owned_class_or_404(); admin has global management. | backend/tests/test_classes.py includes role-scoped listing, cross-teacher class/assignment boundaries, and student membership cases. | Positive evidence for per-class teacher ownership in inspected routes. |
| Class enrollment | /classes/join rejects teacher/admin before rate-limit reservation or enrollment creation. | Regression test verifies 403 for teacher/admin, no enrollment side effects, and successful student join. | Confirmed in current main after PR #162. |
| Assignments | _can_manage() allows creator or admin; class-scoped visibility checks enrollment; submission route rejects non-student roles. | test_assignments.py and test_classes.py cover creator/other-teacher and enrolled/removed-student boundaries; PR #163 adds student-only submission coverage. | Positive evidence for inspected assignment read/manage/submit paths. |
| Quiz attempts | Start/history queries scope attempts to the authenticated user; submit lookup includes both attempt ID and user_id, returning 404 for another user's attempt. | backend/tests/test_quiz.py::test_cannot_submit_someone_elses_attempt and history ownership filtering. | Positive evidence for attempt ownership in inspected paths. |
| Curriculum content | /content teacher CRUD remains topic-wide; question creator is recorded but not checked for update/delete; lessons have no creator field. Student curriculum reads all published lessons for a topic. | Existing tests cover same-teacher lifecycle but not cross-teacher read/update/delete policy. | Open policy and authorization gap; see LUMEN_CONTENT_OWNERSHIP_DECISION_BRIEF.md. |

## 2. Source evidence inspected

All paths below were read from Lumen main on 17 November 2026:

- backend/app/routers/files.py — blob 98fb01210e45d07badcbc1c1d8f48a7426b5c373
- backend/app/models/file_attachment.py — blob 4d5dc3a365e5d590c12def3e583a61d3b48c7d0f
- backend/app/routers/classes.py — blob c770c0a41b117019bfbb94f135e945ff5ab8874b
- backend/app/models/class_section.py — blob e2e415dbe5c88fd8a5352461fba49c22400ee143
- backend/app/routers/assignments.py — blob bae588076a3d2bc4fa003ec195f58997005679cc
- backend/app/routers/quiz.py — blob f0c677a7a42ea318cbac9e736df0fc4d09341869
- backend/tests/test_files.py — blob 9ead44f67d61da82f7eff83905575e17cc38a0f7
- backend/tests/test_classes.py — blob 649afba9956728fbbfe04f5885f0bfd4aa64cd7e
- backend/tests/test_assignments.py — blob 2cd38650bda1f08b7cd55c0c90ae88d7bc536f2a
- backend/tests/test_quiz.py — blob 8398811fb544209511807483ffbf268db88fcba8

## 3. Findings and limits

1. **No confirmed IDOR was established in the inspected file, class, assignment, or quiz-attempt routes.** This is a source-and-test review, not a live exploit test or proof that every route is safe.
2. **Curriculum content remains unresolved.** The existing API allows teacher-wide topic CRUD without cross-teacher ownership checks. Do not silently change this behavior until the product owner selects policy A, B, or C in the ownership decision brief.
3. **UI authorization coverage remains incomplete.** The inspected frontend Content Manager tests focus on request payloads and list/CRUD behavior; they do not establish cross-teacher authorization behavior in the browser. The UI should handle server-side 403/404 responses, but client-side hiding must never substitute for API authorization.
4. **This checkpoint does not claim a complete endpoint inventory.** Other routers, background jobs, alternate entry points, and every UI action still require systematic review.
5. **No production test was performed.** No real account, production database, or deployment was touched.

## 4. Remaining LUM-010 work

- Complete a route-by-route inventory for every router and sensitive resource, including alternate IDs, nested resources, exports, reports, and background-worker entry points.
- Finish cross-role and cross-owner negative tests for all identified read, write, delete, export, and membership operations.
- Review UI flows and browser E2E behavior for denied access and stale permissions.
- Obtain an explicit curriculum-content ownership decision before implementing any ownership migration or CRUD behavior change.
- Re-run required CI on the exact resulting Lumen PR head and record the source SHA and workflow run IDs.

**Disposition:** Keep LUM-010 / S1-02 **In Progress**. This checkpoint records positive evidence and remaining gaps; it does not close the task.