# Reconcile infrastructure release readiness

## Purpose

Give operators a source-backed release handoff after completed Sprint 3 work,
without confusing historical test results, SQL parsing, applied upgrades, live
postconditions, and current functional verification.

## Repository orientation

Read `AGENTS.md`, `.agent/PLANS.md`, the completed adoption handoff at
`a5360fa:docs/plans/sprint-4/codex-astra-adoption-execplan.md` (source branch;
not yet present on the publication base), and the historical
`docs/plans/sprint-3/infra-ci-demo-data-execplan.md`.
The deliverable is `docs/release/rc-evidence.md`.
Actual infrastructure lives at `C:/dev/softdev-infra/docker-compose.infra.yml`
with an adjacent `.env`. SQL remains owned by
`C:/dev/rt_project/rt-api-skeleton/db`; upgrade order and historical results
remain in that repository's `docs/sprint-3-api-verification.md`.

## Current behavior

Sprint 3 Day 10 is reported complete and documented in the API repository.
Infra main contains metadata/instructions, not runnable infrastructure.
The evidence task started on clean `docs/codex-astra-adoption` at `a5360fa`.
The user subsequently authorized commit, push and PR on
`chore/infra-release-evidence`, created from fetched `origin/main` at `90a0189`.
Deployment, dependency installation and database mutation remain out of scope.

## Desired behavior and scope

Record actual environment paths, source commits, safe service probes, database
metadata and seed evidence, ownership, unknowns and separately actionable
follow-ups. Preserve administrator choices and managed controls. Never read
secret values, run upgrade scripts, reseed data or infer GUIDs. No application
or SQL changes. Historical unchecked boxes do not authorize implementation.

## Implementation plan

1. Read the governing instructions and source handoffs; locate the real Compose
   workspace from Docker labels and `docker compose ls --format json`.
2. Establish SQL instance/database identity with `rt_sqlserver.test_connection`
   before metadata calls. Inspect allowed metadata and nonsecret lookup rows.
3. Compare API-owned upgrade requirements with live postconditions; probe
   existing services and inspect the implemented notification/scan path.
4. Write `docs/release/rc-evidence.md`, reconcile the Sprint 3 plan, and prepare
   separate follow-ups with exact file ownership, preconditions and rollback.
5. Validate references and `git diff --check`; inspect the full documentation
   diff and preserve missing quality gates as explicit limitations.

## Tests and verification

Use the exact read-only commands and MCP operations recorded in the evidence
document. Run `git diff --check` and PowerShell `Test-Path` reference checks.
Do not run application mutation collections or create databases for this task.
Frontend/backend gates in AGENTS remain mandatory for applicable release work;
this infra checkout has no package, application, CI or linter runner.

## Acceptance criteria

- Real Compose/environment paths and SQL ownership are explicit.
- Current identity, metadata and health evidence are separated from historical
  application results; unavailable checks are never marked passed.
- IDs derive from natural keys and permission codes, never fabricated values.
- Backup/restore procedure and missing prerequisites have reviewable follow-ups.
- Only documentation changes, with no secrets or schema/data mutations.

## Progress

- [x] Read instructions, adoption handoff and Sprint 3 source evidence.
- [x] Located the real Compose workspace and verified SQL instance/database.
- [x] Inspected initial schema/index evidence and safe service health probes.
- [x] Finish seed/FTS evidence and environment reconciliation.
- [x] Complete release evidence and separate follow-ups.
- [x] Validate references and review the documentation diff.

## Surprises & Discoveries

- 2026-09-15: MCP identity succeeds for server `46e4d2ff2881`, database `rt`;
  API known-issues documentation still describes an older timeout.
- Custom MCP SELECT is disabled. Use permitted metadata operations without
  weakening the setting; document remaining query evidence separately.
- Four Compose services run; no worker service appears. API port 8000 is
  unavailable, while MinIO health and MailHog HTTP return 200 and Redis PONG.
- SQL's healthcheck uses missing `/opt/mssql-tools/bin/sqlcmd`; installed
  `/opt/mssql-tools18/bin/sqlcmd` explains the unhealthy probe despite live SQL.
- Live evidence confirms the upgraded SLA/audit requirements, exact 18
  permissions, canonical role counts and three enabled AUTO FTS indexes.
- Historical API Day 10 source records the upgrade as applied after earlier
  parse-only notes. It records completed disposable verification and cleanup.

## Decision Log

- 2026-09-15: Reuse `C:/dev/softdev-infra`; do not create guessed infrastructure.
- Preserve SQL and application ownership in rt-api; only reconcile infra docs.
- Keep work on the supplied clean branch; no new publication requested.
- Publication follow-up: the user authorized one PR on
  `chore/infra-release-evidence`. Carry only the four evidence documents to
  `origin/main` at `90a0189`; keep unrelated Astra adoption changes separate.
  Retain these related documentation artifacts in the requested single PR.
- Use metadata-only MCP paths and nonsecret lookup samples; never sample
  TenantSetting values, authentication tables or audit payloads.

## Outcomes & Retrospective

Evidence and five independently reviewable follow-ups are written in
`docs/release/rc-evidence.md` and `docs/release/infra-follow-ups.md`. Sprint 3's
historical plan now points to this reconciliation. Seventeen required paths
were verified. Tracked and new-file whitespace checks, conflict-marker checks
and Markdown fence checks passed; all four changed documents were reviewed.
No application, infrastructure configuration, SQL,
data or secret values were changed. Current application functionality, login
identity, full relational checks, authenticated S3 access, recovery proof and
deployment provenance remain explicit release unknowns, not failed Sprint 3
implementation milestones.
