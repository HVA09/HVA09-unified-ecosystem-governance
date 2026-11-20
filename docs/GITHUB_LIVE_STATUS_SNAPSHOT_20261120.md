# GitHub live status snapshot — 2026-11-20

- **Purpose:** reconcile the central governance view with the latest GitHub evidence inspected on 2026-11-20.
- **Scope:** read-only repository and PR inspection. This snapshot does not change product code, databases, deployments, secrets, or branch protection.
- **Authority rule:** live default-branch commit and current PR state outrank older continuity notes. CI evidence applies only to the exact commit tested.

## 1. Repository inventory

| Repository | Current observed main SHA | Role | Finding |
|---|---|---|---|
| `HVA09/ai-chat-app` | `26976f74b73b3f3f4ab3dd7001e945ff85d85b78` | AI Chat App | Independent product repository |
| `HVA09/lumen-learning-platform` | `7240b77f706adc6706b14b0163809e244d46a965` | Lumen | Independent product repository |
| `HVA09/erp-global-pro` | `577c105d81bd7b7c926965c0ed7c94379c1cb695` | ERP Global Pro | Independent product repository |
| `HVA09/HVA09-unified-ecosystem-governance` | `7275d81eeb9490947b6759b28c96c1feb9dbfddf` | Unified governance | Central documentation and coordination repository |

The repository inventory did not show a separate AI Core repository. AI Core remains a documented architecture/workstream, not a verified standalone implementation repository. No separate repositories were found for Research & Discovery, Innovation Studio, Project Forge, Virtual Labs & Simulators, Smart Technology Operations, or Open Innovation. These remain future/deferred workstreams, not missing repositories that should automatically be created.

## 2. AI Chat App

- Observed main: `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`.
- Open work includes the retention/account-deletion/upload safety stack:
  - [PR #733](https://github.com/HVA09/ai-chat-app/pull/733): assistant snapshot and project-retention concurrency; described as a superset replacement for #725.
  - [PR #738](https://github.com/HVA09/ai-chat-app/pull/738): personal upload/account-deletion race; stacked on #733.
  - [PR #739](https://github.com/HVA09/ai-chat-app/pull/739): broader retention/upload transaction safety and durable cleanup outbox; stacked on #738.
- Exact-head evidence for PR #739:
  - HEAD: `c5cd0275d7d9ebf10c97c1597b88f09df2ff69ff`.
  - CI run [#44645616987](https://github.com/HVA09/ai-chat-app/actions/runs/44645616987): completed successfully; backend, frontend, SDK, preview-builder, and production-compose jobs all succeeded.
  - CodeQL run [#44645616881](https://github.com/HVA09/ai-chat-app/actions/runs/44645616881): succeeded.
  - Passing CI does not equal approval or merge readiness. Keep the stack unmerged until base order, review, and all required checks are reconciled.
- PR #738 reports CI run [#44619574353](https://github.com/HVA09/ai-chat-app/actions/runs/44619574353) and CodeQL [#44619574280] succeeded on its tested head; it still requires the #733 dependency and fresh checks after retargeting.
- PR #736 is a synthetic PostgreSQL retention benchmark. Its reports are not production-capacity evidence.
- Other open dependency-update and legacy PRs should be triaged separately; do not let them distract from the security/concurrency stack.
- **Next gate:** verify S3 deletion behavior with focused mocks (correct bucket/key, batching <=1000, error responses, and user-prefix isolation), then rerun CI on the updated exact head. No production or live database testing is implied.

## 3. Lumen

- Observed main: `7240b77f706adc6706b14b0163809e244d46a965`.
- Recent main history includes the leaderboard/privacy UI work associated with PRs #171 and #172.
- [PR #154](https://github.com/HVA09/lumen-learning-platform/pull/154) remains open and is an old LTI/AGS integration stack. Its recorded baseline is behind current main; do not merge the historical #16–#23 chain independently. It needs reconciliation against current main, fresh CI, and Moodle/Canvas sandbox acceptance.
- [PR #166](https://github.com/HVA09/lumen-learning-platform/pull/166) is a documentation-only audit tied to historical baseline `5774c8575bf62a3894a0eb957b77745181ff71e1`; its findings must not be presented as current runtime behavior.
- The Lumen `docs/CONTINUITY.md` still names the 17-Nov main SHA `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`; this is stale relative to the observed 20-Nov main SHA above.
- **Next gates:** refresh continuity documentation; continue route-by-route authorization/IDOR verification; preserve the separate analytics privacy decisions; keep LTI sandbox acceptance distinct from core LMS stability.

## 4. ERP Global Pro

- Observed main: `577c105d81bd7b7c926965c0ed7c94379c1cb695` (documentation synchronization PR #14).
- No open PRs were returned in the current repository search.
- The repository README still names the earlier 17-Nov checkpoint `7bb01d23328a68aae67c659fd6c32b6db033637d`; update this continuity marker to the observed current main.
- Runtime remains Flask + SQLite v8. The Stage25C runbook/preflight exists, but execution against a real restored SQLite v8 copy has not been evidenced.
- Keep `TENANT_DATA_SCOPING_ENABLED=False`. No production multi-company activation, dual-write, PostgreSQL cutover, or live data migration.

## 5. Unified governance and AI Core

- Observed governance main: `7275d81eeb9490947b6759b28c96c1feb9dbfddf`.
- [Governance PR #7](https://github.com/HVA09/HVA09-unified-ecosystem-governance/pull/7) remains open and documents batch throughput/concurrency and branch policy. It does not change repository settings.
- The central task register and AI Core inventory contain historical product SHAs and should be refreshed from the product repositories before treating their task states as live.
- AI Core should first complete its capability inventory and service-boundary review. Reuse proven AI Chat App components where appropriate; do not build duplicate Agent Runtime, MCP, RAG, Workspace, or scheduler services.
- Create a dedicated AI Core repository only after the inventory, API contracts, threat model, and initial implementation scope are accepted. Do not create repositories for every deferred future workstream now.

## 6. Required documentation reconciliation

1. Update Lumen continuity SHA and current PR/decision references.
2. Update ERP README checkpoint SHA and preserve the Stage25C safety gates.
3. Refresh the central `TASK_REGISTER.md`, `UNIFIED_50_TASK_EXECUTION_PLAN.md`, and `AI_CORE_CAPABILITY_INVENTORY.md` with explicit observed SHAs and evidence dates; preserve historical snapshots rather than overwriting them.
4. Keep product code, branches, issues, and databases inside their original repositories.
5. Do not mark any task Done from documentation alone; require the task's stated code, CI, sandbox, restore, or manual-acceptance evidence.

## 7. Safety boundaries

- No production changes, live database access, secrets access, or deployment actions were performed for this snapshot.
- No PR in the product repositories was merged, retargeted, force-pushed, or closed.
- No paid service or GPU resource was provisioned.
- This document is a dated evidence snapshot, not a substitute for a fresh GitHub check before the next code change.
