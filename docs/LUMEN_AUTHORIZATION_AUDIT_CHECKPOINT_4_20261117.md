# Lumen Authorization Audit Checkpoint 4 — Quiz and Billing Boundaries

**Date:** 2026-11-17  
**Source:** `HVA09/lumen-learning-platform` `main` at `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`  
**Method:** source and test-code review only; no test suite or live payment provider was executed.

## Executive result

The quiz attempt path consistently scopes attempt reads and writes to the authenticated user. Billing self-service paths scope subscription reads and cancellation to the authenticated user, while checkout binds the provider request to the authenticated user rather than accepting a user ID from the client. Webhook handlers verify provider events before applying them and use provider-scoped event IDs for idempotency.

No confirmed IDOR or authorization bypass was identified in these reviewed paths. This is not a live payment security certification.

## Quiz authorization

- `POST /quiz/topics/{topic_id}/start` requires `get_current_user`; open attempts and retry/cooldown checks filter by both `QuizAttempt.user_id == current_user.id` and topic/mode.
- New attempts are created with `user_id=current_user.id`.
- `POST /quiz/attempts/{attempt_id}/submit` loads the attempt using both attempt ID and authenticated user ID. A foreign attempt is not returned to the caller.
- Mastery updates, result rows, study activity and achievements are written for `current_user.id`.
- `GET /quiz/topics/{topic_id}/history` filters by authenticated user and topic.
- `backend/tests/test_quiz.py` includes `test_cannot_submit_someone_elses_attempt`, duplicate submission denial, client-supplied score ignored, and answers outside the attempt ignored.

**Remaining test improvement:** add a two-user history isolation assertion and a foreign-attempt lookup test that verifies the same denial semantics for malformed/nonexistent IDs (avoid leaking existence). Current source already scopes the query correctly.

## Billing authorization and webhook trust

- `GET /billing/subscription` filters subscriptions by `current_user.id`.
- `POST /billing/checkout` obtains user ID/email from the authenticated principal; the client supplies a plan ID only. Success/cancel redirect URLs come from trusted server configuration.
- `POST /billing/cancel` selects the subscription by `current_user.id`, not by a client-supplied subscription identifier.
- Stripe and PayPal webhook handlers call the configured provider's verification method before processing the event. Invalid verification is rejected.
- Event receipts are uniquely claimed by `(provider, provider_event_id)`; duplicate deliveries are acknowledged without reapplying. Unmatched subscription updates remain retryable rather than permanently consumed.
- Checkout-completed events resolve the user/plan only after provider verification and validate that both records exist.

Tests in `backend/tests/test_billing.py` cover disabled webhook routes, invalid signatures, checkout URL source, client redirect rejection, duplicate event handling, provider-scoped event IDs, missing event IDs and retryable unmatched subscription updates.

**Remaining test improvement:** add two-account subscription read/cancel isolation tests, plus equivalent PayPal verification/error-path tests if the PayPal adapter is enabled for a real deployment. Existing mocked Stripe tests do not prove external provider configuration or production webhook delivery.

## Follow-up matrix

| Area | Next verification | Priority |
|---|---|---|
| Quiz history | Two users; history for A never contains B's attempts | P2 |
| Quiz attempt | Foreign attempt and unknown attempt have consistent non-disclosing denial behavior | P2 |
| Billing self-service | A cannot read/cancel B's subscription; verify both accounts in one test | P1 |
| PayPal | Signature verification and retry/idempotency behavior through adapter tests | P1 when enabled |
| Live payments | Provider dashboard webhook configuration, endpoint secret rotation, TLS and delivery acceptance in staging | Release gate; not performed |

## Limitations and safety

- Code and test definitions were inspected; no backend tests were executed in this pass.
- No real Stripe/PayPal account, provider dashboard, payment event, production subscription, or production database was accessed.
- No application code, schema, migration, secrets, runtime configuration, infrastructure, or deployment was changed.
- LUM-010 / S1-02 remains In Progress because content ownership, leaderboard privacy and remaining cross-user test coverage are still open.
