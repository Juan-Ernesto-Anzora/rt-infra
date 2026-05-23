# Sprint 2 Infra ExecPlan — Local Dev Reliability and Demo Support

## Purpose

This plan supports Sprint 2 by making local development and demo flows repeatable. After the change, a developer can start SQL Server, MinIO, Redis, Mailhog, verify FTS, verify the bucket, seed demo data, and run a full demo without manually rebuilding context.

## Repository orientation

This plan applies to `rt-infra`. Relevant areas: `docker-compose.infra.yml`, `.env.infra.example`, `scripts/`, `AGENTS.md`, and `.agent/PLANS.md`.

## Current behavior

Infra runs services and the MinIO bucket exists manually. SQL Server FTS is enabled. There may not be a single command/doc to validate demo readiness.

## Desired behavior

Provide scripts and documentation to validate local readiness, create or verify the MinIO bucket, check SQL Server FTS, and document demo data.

## Scope

In scope: dev readiness script, MinIO bucket helper, FTS verification SQL, demo startup README, optional seed SQL references. Out of scope: production deployment, Kubernetes, cloud provisioning.

## Implementation plan

### Milestone 1: Readiness checks

Update scripts to check SQL Server, MinIO, Redis, Mailhog, FTS, catalog, and indexes.

### Milestone 2: Bucket check

Add a helper or documented command to create `rt-attachments` if missing.

### Milestone 3: Demo docs

Add `docs/sprint-2-demo-readiness.md`.

## Tests and verification

Run Docker Compose and dev check scripts. Manually verify MinIO, Mailhog, SQL FTS, and the bucket.

## Acceptance criteria

Local readiness can be verified in under five minutes; bucket and FTS checks are documented; demo readiness doc exists.

## Progress

- [ ] Milestone 1 completed.
- [ ] Milestone 2 completed.
- [ ] Milestone 3 completed.

## Surprises & Discoveries

Record findings here.

## Decision Log

Record decisions here.

## Outcomes & Retrospective

Complete after merge.
