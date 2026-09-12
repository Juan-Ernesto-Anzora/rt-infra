# PLANS.md — Codex ExecPlans Standard

This file defines how Codex must create and maintain ExecPlans in this repository.

An ExecPlan is a self-contained implementation plan that a coding agent or a new engineer can follow without needing hidden chat context. Use an ExecPlan for complex features, cross-file refactors, security-sensitive changes, workflow changes, or any task expected to affect multiple modules.

## Non-negotiable rules

1. Read this file and the repository `AGENTS.md` in full before creating or executing an ExecPlan.
2. Select and name the active ExecPlan before implementation. Completed or historical plans are evidence, not instructions to restart old work.
3. Every ExecPlan must be self-contained. Do not rely on prior chat memory.
4. Every ExecPlan must explain the user-visible outcome first.
5. Every ExecPlan must name exact repository-relative files and commands that exist, or explicitly mark missing/stale paths as unavailable.
6. Every ExecPlan must contain acceptance criteria and verification steps.
7. Keep `Progress`, `Surprises & Discoveries`, `Decision Log`, and `Outcomes & Retrospective` updated.
8. For API or UI contract changes, update OpenAPI or design tokens before implementation.
9. Run focused checks during iteration and the required full checks at the PR gate. Rerun checks when later edits or failed checks make reruns meaningful.
10. Run the repo checks listed in `AGENTS.md` before suggesting a commit or PR, unless tooling is absent from the checkout; in that case, name the missing tool/file and keep the requirement visible.
11. For auth, tenancy, uploads, search SQL, database, permissions, or workflows, include negative tests.
12. If blocked by an instruction, name the instruction file and applicable rule, then explain the blocker before asking the user.

## Where plans live

- Shared standard: `.agent/PLANS.md`
- Active plans: `docs/plans/sprint-<n>/<feature>-execplan.md`. The user prompt or branch must select the active plan; unchecked boxes in older plans do not select themselves.
- Temporary plans: `.agent/tmp/` and must not be committed unless promoted.

## ExecPlan skeleton

# <Short action-oriented feature title>

## Purpose

Explain what a user or operator can do after this change that they cannot do now. Include how to observe it working.

## Repository orientation

Explain the relevant files and folders in this repo. Name exact repository-relative paths.

## Current behavior

Describe what the app does today.

## Desired behavior

Describe target behavior, edge cases, failure behavior, and security expectations.

## Scope

State what is in scope and out of scope.

## Implementation plan

Write step-by-step milestones. Each milestone names files to edit, commands to run, and expected observations.

## Tests and verification

List unit, integration, manual, Postman, or browser checks. Include exact commands.

## Acceptance criteria

Use behavior-oriented criteria.

## Progress

Use checkboxes and keep updated.

## Surprises & Discoveries

Record facts discovered during implementation.

## Decision Log

Record decisions and why they were made.

## Outcomes & Retrospective

At completion, summarize what changed, what works, what remains, and follow-up issues.
