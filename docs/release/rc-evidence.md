# Infrastructure release-candidate evidence

## Decision and scope

Evidence collected 2026-09-15, America/El_Salvador, against the existing local
nonproduction environment. This is a read-only reconciliation, not a release
approval. Sprint 3 Day 10 completion is supported by API-owned documentation
and source history. Current schema/seed evidence supports important upgrade
postconditions; fresh application, backup/restore and scan-path sign-off remain
incomplete. No SQL scripts, seeds, service restarts or settings changes were
performed. Secret values, user records and audit payloads were not inspected.

## Locations and provenance

| Surface | Inspected location / source revision |
| --- | --- |
| Infra | `C:/dev/rt_project/rt-infra-skeleton`, `docs/codex-astra-adoption`, `a5360fa800d5a164941ea6c3f9eb5762b927e084`; initially clean |
| Infra local main | `0ddcfb085cda3c713acb3f1359fc366a49c55219`; metadata and AGENTS only; remote freshness not checked |
| API | `C:/dev/rt_project/rt-api-skeleton`, `docs/codex-astra-adoption`, `906ee21cf165abdab0a07cb92ad28cee85e1a335`; no reported tracked changes |
| API Sprint 3 history | `89765ee` hardening, merged by `6ce40df` (PR 22), present in inspected ancestry |
| Web | `C:/dev/rt_project/rt-web-skeleton`, `docs/codex-astra-adoption`, `7cb6cb56ccb31a455babc916e4ac5b0a3d23645f`; clean |
| Actual Compose | `C:/dev/softdev-infra/docker-compose.infra.yml`, project `softdev-infra`; Docker project listing and container working-directory labels agree |
| Environment files | `C:/dev/softdev-infra/.env` and API `.env` exist; values not inspected; effective API environment cannot be established while API is unavailable |
| Deployment provenance | softdev-infra has no `.git`; no infrastructure deployment commit or release tag established; checkout commits do not identify a running API/web deployment |

Publication follow-up: fetched `origin/main` is now `90a0189` (planning PR 1
merged). The evidence branch `chore/infra-release-evidence` starts there;
the table above preserves the original inspection context. Astra adoption is
not included in this release-evidence PR.

SQL ownership stays in API `db/`. Canonical instructions are API
`docs/sprint-3-api-verification.md`, `docs/release-readiness.md`,
`docs/sprint-3-known-issues.md`, and
`docs/plans/sprint-3/api-admin-configuration-execplan.md`.
The last plan mentions `docs/sprint-3-api-release-checklist.md`, which is absent;
use the actual verification/release-readiness documents. Do not create a second
Compose tree or copy SQL into infra. No nested infra documentation overrides
were found. API Git status warned that `.pytest_cache` was inaccessible; this
does not establish anything about ignored cache contents.

## Services and persistence

| Service | Actual endpoint / storage | Observation |
| --- | --- | --- |
| SQL Server | `rt_sqlserver` / `46e4d2ff2881`, host port 1433; `softdev-infra_sqlserver_data` at `/var/opt/mssql` | MCP SQL connection succeeds; Docker changed from starting to unhealthy |
| MinIO | `rt_minio`, API `http://localhost:9000`, console port 9001; `softdev-infra_minio_data` at `/data` | Docker healthy; `/minio/health/live` and `/minio/health/ready` HTTP 200 |
| Attachments | `/data/rt-attachments` inside MinIO; API source default bucket `rt-attachments` | Directory exists; authenticated S3 bucket policy, HEAD, PUT and GET not verified |
| Redis | `rt_redis`, host port 6379; `softdev-infra_redis_data` at `/data` | `redis-cli PING` returned PONG; no Docker healthcheck |
| MailHog | `rt_mailhog`, SMTP port 1025, UI `http://localhost:8025`; no persistent mount | UI HTTP 200; current SMTP delivery unverified; message contents not read |
| API | Source default local port 8000 | `/api/health` unavailable with five-second timeout; no API container in the running project |
| Worker / scanner | No worker or scanner Compose service; no Celery dependency/configuration found in API source | Redis availability does not prove task execution |

SQL healthcheck references `/opt/mssql-tools/bin/sqlcmd`, which is absent.
The installed executable is `/opt/mssql-tools18/bin/sqlcmd`. Five retained
healthcheck results exit 1 with missing-file errors. This explains the observed
health signal without treating the database as offline. Raw health commands and
logs were suppressed to avoid exposing credential arguments.

Running local image IDs (immutable local identifiers, not registry digests):

```text
SQL     sha256:6249e946034c0b72e98cbbdd790deede2bb94fe5c73335ecdbb77176af4d267c
MinIO   sha256:d249d1fb6966de4d8ad26c04754b545205ff15a62e4fd19ebd0f26fa5baacbc0
Redis   sha256:bb186d083732f669da90be8b0f975a37812b15e913465bb14d845db72a4e3e08
MailHog sha256:8d76a3d4ffa32a3661311944007a415332c4bb855657f4f6c57996405c009bea
```

API `apps/rt/services/notification_service.py::_send_notification` calls Django
`send_mail` synchronously with `fail_silently=True` and exception logging.
`apps/rt/views.py` finalizes attachments with `scanstatus="pending"` and builds
storage URLs; `apps/rt/serializers.py` exposes storageurl/scanstatus. No scan
consumer was found. AGENTS section 5 requires scanning and blocking serving
until clean; that requirement is not demonstrated by current source/runtime.
This task does not claim that an object was publicly downloadable.

## Database identity and evidence boundaries

Before metadata inspection, `rt_sqlserver.test_connection({})` returned server
`46e4d2ff2881`, database `rt`, SQL Server Developer Edition 2022
`16.0.4210.1` on Linux. Server name matches the local SQL container ID.
`list_connections` exposes the default connection and no named connections.
Login identity is unverified: the identity SELECT was rejected because
`execute_query` is disabled. That setting was preserved. The historical MCP
timeout is resolved for this session's successful metadata calls, not a claim
about future sessions.

Permitted `list_tables`, `describe_table`, `get_relationships`, `list_indexes`
and nonsecret lookup samples provide postcondition evidence. These do not
establish which exact script bytes were applied, who applied them or when.
No stored deployment ledger, application transcript or retained backup was
established. The API's approved SELECT-only sqlcmd fallback is documented;
this task did not load credentials or change MCP controls to invoke it.

### Confirmed schema and indexes

All 18 tables named by API `db/verify-sprint3-release.sql` are present:
Tenant, User, Membership, MembershipRole, Role, Permission, RolePermission,
Flow, Status, Transition, Request, Comment, Attachment, Activity, SlaPolicy,
TenantSetting, FeatureFlag and NotificationTemplate.

SlaPolicy has all 11 required columns: PolicyId, TenantId, Name, AppliesTo,
Targets, CreatedAt, Priority, ResponseMinutes, ResolutionMinutes, IsActive,
UpdatedAt. Priority/response/resolution/active are nonnullable; UpdatedAt is
nullable. Four enabled, trusted checks restrict priorities, positive durations
and response <= resolution. Unique `(TenantId, Name)` and
`IX_SlaPolicy_TenantActivePriority(TenantId, IsActive, Priority)` are present.

Activity has ActivityId, TenantId, nullable RequestId, nullable ActorId, Type,
Payload and CreatedAt. `IX_Activity_TenantCreated(TenantId, CreatedAt DESC)` is
present, supporting admin events without a request. API
`apps/rt/services/admin_audit.py` writes tenant, actor, type, JSON payload and
timestamp; entity identifiers are inferred into payload. Row presence (31 by
metadata) does not prove complete audit coverage or inspect sensitive payloads.

TenantSetting has TenantSettingId, TenantId, Key, Value, ValueType, IsSensitive,
UpdatedAt and UpdatedById; ValueType check is enabled/trusted. Values were not
sampled. Attachment includes GroupId, CommentId, StorageUrl and ScanStatus;
nullable ScanStatus has no database check constraint.

Other inspected API column contracts are present: Request has tenant/human ID,
title/description, flow/status/priority, requester/assignee, CustomFields, DueAt,
CreatedAt/UpdatedAt and RowVer; Comment has tenant/request/author/group,
MessageMd, Visibility and CreatedAt; Status has tenant/flow/name/category and
IsTerminal; Transition has flow/from/to, GuardRolesJson, GuardPermsJson and
AutoRules. Membership has tenant/user/IsDefaultTenant. FeatureFlag has Key,
Enabled, Description, UpdatedAt/UpdatedById; NotificationTemplate has EventType,
SubjectTemplate, BodyTemplate, IsActive, UpdatedAt/UpdatedById. Both configuration
tables also have their own IDs and TenantId. No full ORM-to-schema parity check
or disabled-FK/trust audit is claimed by this inventory.

All 15 named verification indexes exist: IX_Request_Tenant/Status/Assignee/
Updated; IX_Comment_Request/Created; IX_Attachment_Request;
IX_Activity_Request/TenantCreated; IX_Membership_Tenant; UQ_Role_TenantName;
IX_SlaPolicy_TenantActivePriority; UQ_TenantSetting_TenantKey;
UQ_FeatureFlag_TenantKey; UQ_NotificationTemplate_TenantEvent.
Composite primary keys exist on MembershipRole(MembershipId, RoleId) and
RolePermission(RoleId, PermissionCode).

FK metadata confirms request-to-tenant/flow/status/requester/assignee,
status-to-tenant/flow, transition-to-flow/from-status/to-status,
membership-to-tenant/user, role-to-tenant, membership-role-to-membership/role,
role-permission-to-role/Permission.Code, SLA-to-tenant,
settings/flags/templates-to-tenant/updated-by-user, and
activity-to-tenant/request/actor. Comment and attachment request/tenant links
also exist; attachment links to Comment. These single-column relationships
alone do not prove tenant consistency. Fresh duplicate/mismatch queries from
the verification SQL remain distinct from historical zero-violation results.

### Full-text search

FTS inspected through `sample_data` on `sys.fulltext_indexes`,
`sys.fulltext_index_columns` and `sys.tables`: Attachment object 66099276
(Filename column 6), Request 1861581670 (Title/Description columns 4/5),
Comment 2069582411 (MessageMd column 6). All three indexes are enabled, AUTO,
with completed crawls; all four indexed columns use language 1033 (English).
This confirms metadata, not current API search behavior or Spanish relevance.

### Natural keys and seeds

Lookup samples requested up to 100 rows and returned 1 tenant, 5 roles,
18 permissions, 38 role-permission links and 4 SLA policies. Resolve again by
natural key for any future operation; these observed IDs are evidence only.

| Natural key (ACME) | Observed ID | Permissions |
| --- | --- | --- |
| Tenant.Code=ACME | `5C631843-9BB5-49E0-8C6C-E2B6CC33F818` | n/a |
| Role.Name=RT Admin | `CE1B6131-FB46-4FB8-A363-7EBD0F3E8A87` | 18 |
| Role.Name=RT Manager | `891E679D-3105-44EF-B969-23401CC400FD` | 9 |
| Role.Name=RT Agent | `A0F68F18-1CBA-4A04-B5CC-259E2C05DADE` | 5 |
| Role.Name=RT Requester | `3FA62132-35E8-4EEE-9DA5-3E67C5142149` | 4 |
| Role.Name=RT Viewer | `BD80074D-6131-4070-936F-4546C0725DF0` | 2 |

Exact live permission codes: admin.audit.read, admin.permissions, admin.read,
admin.roles, admin.settings, admin.users, admin.workflows, attachments.write,
comments.write, featureflags.manage, notifications.manage, reports.export,
reports.read, requests.read, requests.transition, requests.write, sla.manage,
tenant.settings.manage. Admin and Manager contain admin.read and
admin.audit.read. API `admin_permissions.py::require_admin_permissions` requires
admin.read plus the route-specific code; do not substitute admin.access or
invent permission GUIDs. Effective signed-in-user membership remains unverified.

ACME policy natural keys Low/Normal/High/Urgent have respectively 240/4320,
120/1440, 30/480 and 15/120 response/resolution minutes, all active. No policy or
administrator setting was changed. These are configuration policies, not proof
of SLA timers/compliance (documented as deferred).

Column-scoped distribution checks (no setting values/template bodies) found
exactly four rows and four distinct natural keys in each table:
TenantSetting: default_page_size, default_timezone, email_from, web_base_url;
FeatureFlag: adminConsole, exportsEnabled, notificationTemplates, slaEnabled;
NotificationTemplate: comment.added, request.assigned, request.closed,
request.created. Per-row configured values/flags/template activation were not
read or reset. Metadata confirms each has TenantId and UpdatedById FKs.

## Upgrade evidence

Apply order below is owned by the API verification document; nothing was applied
here. SHA256 identifies the inspected file bytes, not a deployment receipt.

| API db file, in upgrade order | SHA256 |
| --- | --- |
| upgrade-sprint3-admin-workflows.sql | `86BCE400D4DC167FDE9FF263020EEF0618EC8271A44E15B4C71BD8E2A0B6E364` |
| upgrade-sprint3-admin-users-roles.sql | `2611C7168A962FBCE82FE0A9813645ED7AD6BFEEDE5441B509F250FD08FD9BAB` |
| upgrade-sprint3-sla-reports.sql | `9DDC16AD2209CC30E1BE9675E14C3042FD466C960E4C1268E31984C1B55BC1CD` |
| upgrade-sprint3-admin-settings.sql | `EE7118ED33701595769D8E584EF579CAA81DFD541DF5429E3D31B6697A783797` |
| upgrade-sprint3-api-polish.sql | `81A45CA8D587AEABE032C4F0E855B28BCFF95F46772205C2E954BC714537513C` |
| verify-sprint3-release.sql (SELECT-only verification) | `59A68BDB6DB1B6134CF1F0E50EDC608C3BB3A725933DCE58BC88F77EB93CF276` |

`create-rt-database.sql` is a clean-build script, never an existing-DB upgrade.
Source uses missing-row inserts; do not rerun seeds as a substitute for evidence.
The SLA upgrade explicitly stops on unmapped legacy rows. Preserve customized
policies, permission assignments and administrator settings.

| Evidence stage | Reconciliation |
| --- | --- |
| Syntax | Historical Sprint 3 notes report all five scripts passed PARSEONLY; no parse run in this task |
| Application | Later Day 10 notes state SLA upgrade/audit index applied; exact execution receipt/checksum remains unavailable |
| Postconditions | Current live columns, FKs, indexes and sampled permission/SLA data independently confirm the inspected requirements above |
| Functional | Historical Day 10: 189 pytest; 27 focused; checks/lint/OpenAPI pass; guarded Sprint 3 32 requests/92 assertions; Sprint 2 8/17; disposable mutation 69/178 with no failures/skips |
| Current functional | API unavailable; no fresh JWT, tenant-negative, FTS search, email, attachment or admin audit exercise performed |

PARSEONLY success does not imply script application. Earlier legacy-SLA blocker
notes are superseded by later Day 10 evidence and current columns/defaults.
The reported disposable rt_day10/BETA validation and cleanup are historical;
do not recreate tenants or restart completed Sprint 3 work from old checkboxes.

## Checks and remaining work

Executed from the local Windows host unless noted:

```powershell
docker compose ls --format json
docker compose -f C:\dev\softdev-infra\docker-compose.infra.yml config --services
docker compose -f C:\dev\softdev-infra\docker-compose.infra.yml config --volumes
docker ps --format '{{.ID}}|{{.Names}}|{{.Image}}|{{.Status}}|{{.Ports}}'
docker exec rt_redis redis-cli PING
Invoke-WebRequest http://localhost:9000/minio/health/live -TimeoutSec 5
Invoke-WebRequest http://localhost:9000/minio/health/ready -TimeoutSec 5
Invoke-WebRequest http://localhost:8025 -TimeoutSec 5
Invoke-WebRequest http://localhost:8000/api/health -TimeoutSec 5
Get-FileHash ..\rt-api-skeleton\db\*.sql -Algorithm SHA256
git diff --check
```

Docker access needed the approved host permission boundary; the first sandbox
attempt was denied, then the read-only host commands succeeded. Compose reports
an obsolete `version` attribute, but service/volume validation succeeds. The
API health request failed; the other listed live probes succeeded. Restricted
Docker inspect projections collected mounts/image IDs/health without env values.

SQL operations: `test_connection({})`, `list_tables({})`,
`get_relationships({})`, `list_indexes({includeUsageStats:false})`,
`describe_table({tableName:...})` for domain schema, and
`sample_data({tableName:...,limit:100})` on Tenant, Role, Permission,
RolePermission and SlaPolicy. FTS system samples use `schema:"sys"`.
`analyze_data_distribution` was restricted to TenantSetting.Key,
FeatureFlag.Key and NotificationTemplate.EventType. No mock services were used
for these probes. Custom `execute_query` was rejected, not passed.

Applicable documentation checks: `git diff --check` and PowerShell reference
checks. Infra has no `.github` workflow, package/build manifest, application
test suite or configured Markdown linter. AGENTS quality requirements remain
visible; do not label missing tools as passing. API CI at
`.github/workflows/ci-api.yml` runs Ruff/isort/Black/pytest, has no live-service
jobs, and its PR path filter excludes docs and SQL. Representative upload/audit
tests monkeypatch ORM/S3/context calls; unit success cannot prove live SQL/S3.

Existing API commands (not rerun for this documentation-only reconciliation):

```powershell
poetry run python manage.py check
poetry run pytest -q
poetry run ruff check .
poetry run black . --check
poetry run isort . --check --diff
poetry run python manage.py spectacular --file .agent/tmp/rt-openapi.yaml --validate
```

Live Postman commands/guards are in API `docs/sprint-3-api-verification.md`.
Do not use `npx` to install absent tooling in this task. Current coverage >=85%,
type checking, frontend gates and CI status were not established. Focused checks
belong to iteration; required full applicable checks belong to the PR gate,
rerun when changes/failures justify it.

## Backup and restore verification

Observed SQL persistence is Docker volume `softdev-infra_sqlserver_data` mounted
at `/var/opt/mssql`; `/var/opt/mssql/backup` does not currently exist. No backup
mount is configured. Only `.env` and Compose exist in softdev-infra. Do not
interpret a persistent volume as a backup. Historical Day 10 documents a
COPY_ONLY clone/restore and subsequent backup deletion, not a retained recovery
artifact. Off-machine backups and current msdb backup history are unverified.

A separately authorized drill must use the existing SQL instance and installed
`/opt/mssql-tools18/bin/sqlcmd`, with credentials supplied by the existing
approved connection, never in documentation or command output. First verify
server/database/login, free space, target path and recovery objective. Select
and record a unique backup filename in an approved backup destination; create
a COPY_ONLY, CHECKSUM full backup without overwriting prior media, copy it to
protected storage outside the live volume and retain its checksum. Use RESTORE
HEADERONLY/FILELISTONLY and VERIFYONLY WITH CHECKSUM; derive logical file names
from the backup, never guess them. Restore with MOVE into distinct files in a
new disposable database (no REPLACE; never restore over rt). Run DBCC CHECKDB,
API `db/verify-sprint3-release.sql` and approved isolated functional/negative
checks. Record restore duration, result, source revision and artifact identity.
VERIFYONLY alone is not a successful restore drill. Remove only newly created
disposable resources after verification; retain backup evidence per policy.

MinIO needs a separately approved object backup/recovery test using the real
`rt-attachments` bucket and isolated destination; SQL backup does not preserve
attachment bytes. Redis persistence/recovery requirements need an owner decision
because no current task consumer was found; MailHog has no persistent mount.
No backup, restore, object copy or cleanup was executed in this task.

## Release unknowns and follow-ups

See [independent follow-up specifications](infra-follow-ups.md): F1 SQL health
probe, F2 complete authorized read-only evidence, F3 recovery drill, F4 scan
enforcement/worker ownership, F5 release provenance and CI gates. Each has
preconditions, exact owned files, expected evidence and rollback considerations.
Do not invent a schema repair: inspected upgrade postconditions already exist.
Release still lacks current API functional evidence, effective login/membership,
full duplicate/tenant-consistency checks, authenticated S3 access, a retained
restore proof and deployed application/CI/tag provenance.
