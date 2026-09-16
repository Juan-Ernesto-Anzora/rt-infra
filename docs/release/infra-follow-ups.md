# Separate infrastructure release follow-ups

These are reviewable task specifications, not authorization to execute changes.
SQL ownership remains in `C:/dev/rt_project/rt-api-skeleton/db`. References to
API files below are relative to that repository; infra files are explicitly
prefixed. Re-read each owner's AGENTS/PLANS before implementation.

## F1: Repair the existing SQL health probe

- Precondition: schedule any required container recreation with the local
  operator; preserve `softdev-infra_sqlserver_data` and the current image.
- Exact edit: `C:/dev/softdev-infra/docker-compose.infra.yml`, SQL healthcheck
  executable from `/opt/mssql-tools/bin/sqlcmd` to the installed
  `/opt/mssql-tools18/bin/sqlcmd`. Review ODBC18 certificate/auth behavior using
  existing nonproduction settings; do not expose or replace credentials.
- Expected: Compose validation succeeds, SQL identity remains rt on the intended
  server, and Docker health becomes healthy with successful read-only probes.
  Record before/after in infra `docs/release/rc-evidence.md`.
- Rollback: retain the previous Compose definition and image ID; restore only
  the probe definition if necessary. No volume deletion or SQL mutation.

## F2: Complete current read-only release evidence

- Preconditions: operator-provided approved SELECT-only connection with known
  principal, running API at an identified commit, existing authenticated test
  identity and S3 access. Do not enable disabled MCP custom-query controls
  without permission. Existing approved sqlcmd can be used with operator-managed
  credentials; never retrieve values for the evidence document.
- Exact inputs: API `db/verify-sprint3-release.sql`,
  `docs/sprint-3-api-verification.md`,
  `postman/RT-Sprint-3.postman_collection.json`,
  `postman/RT-Sprint-2-Regression.postman_collection.json`,
  `postman/RT-Local.postman_environment.json`.
- Exact evidence edit: infra `docs/release/rc-evidence.md`; reconcile historical
  MCP-timeout wording in API `docs/sprint-3-known-issues.md` and
  `docs/release-readiness.md` in a separate API documentation review.
- Expected: record server/database/login first, then all verification outputs,
  zero duplicate/mismatch violations, natural-key membership resolution and exact
  admin.read plus route permissions. Preserve customized values. Use guarded
  read-only collection mode, authenticated bucket metadata/known-object HEAD and
  current API search/permission-negative checks. No new tenant or test object.
- Rollback: no data changes; revoke/expire any temporary session credentials.
  Mutation/cross-tenant fixtures require a separate disposable-environment task;
  retain historical Day 10 evidence until a new authorized run exists.

## F3: Prove recovery using the real environment

- Preconditions: explicit backup/restore authorization, recovery owner/retention
  policy, storage capacity and a unique disposable database/object destination.
- Exact inputs: `C:/dev/softdev-infra/docker-compose.infra.yml`, API
  `db/verify-sprint3-release.sql`. Evidence edit: infra
  `docs/release/rc-evidence.md`; proposed new runbook:
  infra `docs/release/local-recovery-runbook.md` (not created in this task).
- Expected: follow the evidence document's COPY_ONLY/checksum/restore sequence,
  retain off-volume backup identity, restore into distinct files, CHECKDB and
  read-only SQL checks pass, record recovery time. Verify isolated MinIO object
  recovery too. Never count VERIFYONLY or a Docker volume alone as restore proof.
- Rollback: original rt and bucket remain untouched; remove only the drill's
  explicitly identified disposable resources. Preserve successful backup media.

## F4: Establish attachment scanning and serving enforcement

- Preconditions: security/product owner selects scanner, execution owner and
  release treatment; this is application/security work beyond the evidence task.
- Exact existing review surfaces: API `apps/rt/views.py`,
  `apps/rt/serializers.py`, `apps/rt/models.py`, `tests/test_uploads.py`,
  `rt_api/settings.py`, `pyproject.toml`; deployment surface is the existing
  `C:/dev/softdev-infra/docker-compose.infra.yml`. A dedicated API ExecPlan must
  name any new worker/task files before implementation.
- Expected: identify an existing external scanner or implement an approved
  worker path, prove pending/error/infected objects cannot be served and clean
  objects can, enforce tenant authorization, test retries/failures, and document
  bucket access policy. Redis PONG is insufficient. Do not silently equate
  synchronous notification delivery with Celery execution.
- Rollback: preserve quarantine/deny-by-default behavior; stop the new consumer
  without making unscanned objects available. Two approvals for sensitive upload
  changes and full applicable API/security gates remain required by AGENTS.

## F5: Capture deployment provenance and enforce release gates

- Preconditions: identify release owner, actual API/web deployment process,
  upstream merge state, required CI runs and source-of-truth for local Compose.
- Exact files: infra `docs/release/rc-evidence.md`; API
  `.github/workflows/ci-api.yml`, `docs/release-readiness.md`,
  `docs/sprint-3-api-verification.md`. Any Compose versioning decision must refer
  to `C:/dev/softdev-infra/docker-compose.infra.yml`, without copying `.env`.
- Expected: link actual deployed commits/images, applied-script receipts and
  backup identity, release tag and CI URLs. Review docs/SQL path-filter gaps,
  coverage >=85% and type-check runner availability explicitly; do not claim
  absent gates pass or invent a runner command. Reconcile the stale missing
  release-checklist reference and preserve completed Day 10 results.
- Rollback: documentation/CI edits are reversible; no deployment in this task.
  Keep the candidate unreleased until required gates or explicit owner decisions
  have evidence. Do not rerun old implementation merely to clear checkboxes.
