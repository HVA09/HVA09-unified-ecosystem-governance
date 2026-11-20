# Unified Engineering Toolkit — Reusable Engineering Templates

Status: proposed, documentation-only. Keep product repositories independent; this document defines reusable process templates, not shared runtime code.

## Purpose
Reduce repeated engineering work and review gaps across Lumen, AI Chat App, ERP Global Pro, and future AI Core work. GitHub remains the source of truth for source code and decisions.

## Templates
- Pull Request Review: scope, behavior, tests, risks, rollback, dependencies, evidence.
- Security Review: authentication, authorization, tenant boundaries, input validation, secrets, logging, data exposure, dependency risk.
- CI Failure Playbook: reproduce on exact SHA, classify failure, inspect logs, fix narrowly, rerun relevant and full checks.
- Release Readiness: exact SHA, CI, security checks, migration/backup readiness, monitoring, rollback, owner.
- Rollback: trigger, safe rollback steps, data compatibility, verification, escalation; never destructive rollback without authorization.
- API Review: contract, validation, auth, rate limits, errors, versioning, idempotency, observability.
- Migration Review: backup, preflight, locking, compatibility, dry-run, rollback/forward-fix, verification.
- Dependency Review: necessity, license, maintenance, security advisories, footprint, alternatives.
- Observability Review: actionable logs, metrics, alert thresholds, dashboards, runbooks, privacy.
- ADR: context, options, decision, consequences, status, date.

## Minimum PR checklist
- [ ] Scope and user impact explained
- [ ] Exact base/head reviewed
- [ ] Tests and CI pass on exact head
- [ ] Security/privacy implications reviewed
- [ ] Database/data changes have backup, migration and rollback evidence
- [ ] Observability and operational acceptance addressed
- [ ] Docs updated
- [ ] No secrets or sensitive data in artifacts/logs
- [ ] Rollback or recovery plan documented

## Safe automation boundaries
Automation may run checks, draft documentation, create branches and PRs, and make narrowly scoped reversible fixes backed by evidence. Require explicit authorization for production deployments, destructive data/storage actions, migrations on live databases, secrets, permission/branch-protection changes, financial operations, and irreversible actions.

## Adoption
Apply these templates gradually to existing workflows; do not create a new repository, shared code package, or extra service solely for this toolkit.