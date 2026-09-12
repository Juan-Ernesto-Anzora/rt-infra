# Adopt GPT-6 Astra for the Codex development workflow

## Purpose

Adopt GPT-6 Astra as the checked-in Codex development default for `rt-infra` and reconcile repository instructions, prompts, paths, commands, and handoffs with the current checkout. Sprint 4 is an organizational label for this workflow change. It does not authorize Request Tracker product features, API changes, SQL work, dependency installation, deployment, or secret handling.

The observable result is a small documentation/configuration diff that a new Codex task can follow without reopening completed Sprint 2 or Sprint 3 work, inventing unavailable runners, weakening quality gates, or pausing on routine reversible choices. The local active Codex config already selects `gpt-6-astra` with `medium` reasoning by non-secret field inspection, so no home-directory edit is required.

Source: OpenAI model guidance at `https://developers.openai.com/api/docs/guides/latest-model`, read on 2026-09-12. The guide identifies `gpt-6-astra`, recommends auditing instruction files, preserving supported reasoning effort when possible, and distinguishing model configuration from application/API migration.

## Repository orientation

- `AGENTS.md`: repository-wide architecture, quality gates, Git rules, agent workflow, and Codex scope.
- `.agent/PLANS.md`: ExecPlan standard and active-plan rules.
- `.agent/code_review.md`: read-only review priorities and output contract.
- `docs/codex-prompts.md`: reusable prompts for context, implementation, review, and handoff.
- `.codex/config.toml.example`: checked-in Codex config example; must select `gpt-6-astra` without credentials.
- `docs/plans/sprint-3/infra-ci-demo-data-execplan.md`: historical Sprint 3 plan requiring status reconciliation.
- `docs/plans/sprint-4/codex-astra-adoption-execplan.md`: this active plan.

Current checked-in files are documentation and repository metadata only. There is no `.github/`, `docker-compose.infra.yml`, `.env.infra.example`, package/build file, application source, SQL file, verification doc, known-issues doc, or populated `scripts/` directory in this checkout.

## Current behavior

The infra repository has Codex planning guidance but stale prompts still direct agents to update the Sprint 2 plan and validate with Docker Compose/dev-check scripts that are not present. The Sprint 3 plan contains unchecked historical boxes even though the user reports Sprint 3 Day 10 hardening and infrastructure SQL validation complete.

The checked-in Codex example does not currently set a model. The active home config at `C:/Users/Juan Anzora/.codex/config.toml` was inspected only for non-secret model fields and already contains `model = "gpt-6-astra"` and `model_reasoning_effort = "medium"`. This task does not infer the active model for the running conversation.

No OpenAI SDK, Responses API call, model router, custom agent runtime, API key setting, or application-level OpenAI integration exists in this checkout. `.agent`, `.codex`, `AGENTS.md`, and `docs/codex-prompts.md` control Codex development behavior only.

## Desired behavior

Codex instructions and prompts should:

- Select `gpt-6-astra` in the checked-in Codex example.
- Preserve managed controls, user scope, MCP/auth/permission settings, and existing work.
- Tell agents to finish authorized work, make routine reversible choices, and ask only when material ambiguity or real permission boundaries block progress.
- Require blockers caused by instructions to name the file and applicable rule.
- Reconcile instruction paths with actual checked-in files and clearly mark missing or stale commands.
- Preserve required quality and review gates while distinguishing actual runners from absent tooling.
- State that focused checks run during iteration, full required checks run at the PR gate, and reruns are justified by subsequent edits or failures.
- Record reported Sprint 3 completion while leaving deployment/live SQL evidence unverified from this checkout.

## Scope

In scope:

- Documentation/config updates to `AGENTS.md`, `.agent/PLANS.md`, `.agent/code_review.md`, `docs/codex-prompts.md`, `.codex/config.toml.example`, `docs/plans/sprint-3/infra-ci-demo-data-execplan.md`, and this plan.
- Static validation of references, TOML parsing, and Git diff cleanliness.
- Non-secret inspection of local active Codex model fields.

Out of scope:

- Application OpenAI integration, model API calls, API keys, SQL changes, migrations, dependencies, package or lockfile edits, service deployment, Docker/SQL/MinIO/Redis/Mailhog execution, and home config rewrites when the current fields are already correct.

## Implementation plan

### Milestone 1: Establish the adoption baseline

Read governing instructions, prompt files, checked-in config, Sprint 3 plan, and official OpenAI model guidance. Inspect actual checked-in files and local active config model fields without reading secrets.

### Milestone 2: Reconcile instructions and prompts

Update `AGENTS.md`, `.agent/PLANS.md`, `.agent/code_review.md`, and `docs/codex-prompts.md` so active plans are selected explicitly, completed plans are historical evidence, blockers name their source rule, and prompts cover context, implementation, review, and handoff.

### Milestone 3: Set the Codex example and Sprint 3 handoff

Add `model = "gpt-6-astra"` to `.codex/config.toml.example`. Update `docs/plans/sprint-3/infra-ci-demo-data-execplan.md` to record reported completion and explicitly mark deployment/live evidence unverified from this checkout.

### Milestone 4: Validate and prepare PR handoff

Validate checked-in TOML, validate referenced repository paths, run `git diff --check`, inspect `git diff --stat`, and prepare a PR description. Do not run live service checks because this checkout contains no runnable service artifacts.

## Tests and verification

- `& 'C:\Users\Juan Anzora\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe' -c "import tomllib, pathlib; tomllib.loads(pathlib.Path('.codex/config.toml.example').read_text(encoding='utf-8')); print('TOML OK')"`: static TOML parse for checked-in config.
- `Test-Path` checks for expected documentation/config paths and intentionally missing stale runtime paths: static reference validation.
- `git diff --check`: static whitespace/conflict-marker check.
- `git status --short --branch`: static working-tree check.

No live SQL, Docker, MinIO, Redis, Mailhog, frontend, backend, or CI checks are available in this checkout.

## Acceptance criteria

- `.codex/config.toml.example` parses as TOML and contains `model = "gpt-6-astra"`.
- Instructions and prompts select active plans explicitly and do not restart Sprint 2 or Sprint 3 from stale unchecked boxes.
- Required quality gates remain visible; absent runners are identified rather than replaced.
- Sprint 3 reported completion is recorded, and unverified deployment/live evidence stays unverified.
- No product source, SQL, dependencies, API keys, or application-level OpenAI integration changes are introduced.

## Progress

- [x] Read governing instructions, prompts, checked-in config, Sprint 3 plan, and official OpenAI model guidance.
- [x] Inspected actual checked-in files and local active config model fields without reading secrets.
- [x] Created `docs/plans/sprint-4/codex-astra-adoption-execplan.md`.
- [x] Reconciled instructions and prompts.
- [x] Added `model = "gpt-6-astra"` to the checked-in Codex example.
- [x] Recorded Sprint 3 reported completion and unverified deployment evidence.
- [x] Run static validation checks.
- [x] Prepare final PR handoff.

## Surprises & Discoveries

- 2026-09-12: `scripts/` exists but is empty; `rg --files` shows no checked-in compose, CI, SQL, verification, known-issue, package, API, or web source files.
- 2026-09-12: The active home Codex config already contains `model = "gpt-6-astra"` and `model_reasoning_effort = "medium"` by non-secret field inspection, so no home config patch is needed.
- 2026-09-12: The initial infra commit message mentions compose and scripts, but the current tree evidence contains only repository metadata and documentation.
- 2026-09-12: `python`, `python3`, and `py` are not on PATH in this shell; the bundled workspace Python was used for TOML validation.

## Decision Log

- 2026-09-12: Use `docs/codex-astra-adoption` branched from `docs/codex-planning` because `main` lacks the ExecPlan files this task must update.
- 2026-09-12: Keep this as a documentation/configuration change only; application-level OpenAI integration is a separate migration surface if introduced later.
- 2026-09-12: Do not edit the active home config because its non-secret model fields already match the target and editing outside the repository would risk unrelated MCP/auth/permission settings.
- 2026-09-12: Preserve Sprint 3 reported completion as handoff evidence while marking deployed/live SQL validation unverified from this checkout.

## Outcomes & Retrospective

The bounded Codex workflow adoption is complete in the working tree. The checked-in config example selects `gpt-6-astra`, active-plan and blocker guidance is clearer, stale Sprint 2 prompt entry points were replaced, Sprint 3 reported completion is recorded with deployment evidence explicitly unverified, and no product source, SQL, dependency, API key, or application-level OpenAI integration was added.

Validation completed with static checks only because this checkout has no runnable service, package, CI, or SQL artifacts. `git diff --check` passed, `.codex/config.toml.example` parsed as TOML with the bundled Python runtime, expected documentation/config paths exist, and stale runtime paths are confirmed absent.
