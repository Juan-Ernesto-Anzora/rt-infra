# Codex prompts — rt-infra

## Context

Read `AGENTS.md`, `.agent/PLANS.md`, `.agent/code_review.md`, `.codex/config.toml.example`, `{active_execplan}`, and the current ticket in full. Inspect Git status, branch/base, actual checked-in files, available commands, verification docs, known issues, and any paths named by the selected plan. Treat completed or historical plans as evidence; do not restart Sprint 2 or Sprint 3 work merely because an old prompt or unchecked box names it.

## Implement

Implement only `{milestone}` from `{active_execplan}`. Preserve existing work and user scope. Keep `Progress`, `Surprises & Discoveries`, `Decision Log`, and `Outcomes & Retrospective` current. Make routine reversible choices that follow repository patterns; ask only when a material ambiguity or real permission boundary changes the outcome. If blocked by an instruction, name the file and applicable rule.

Run focused checks while iterating. Run the required full checks at the PR gate, and rerun checks only when later edits or failures justify it. If required tooling or service files are absent, report the missing runner without inventing a substitute or lowering the requirement.

## Review

Review `{head_ref}` against `{base_ref}` using `.agent/code_review.md`. Read `AGENTS.md`, `.agent/PLANS.md`, and `{active_execplan}` first. Keep the review read-only. Verify instruction hierarchy, actual paths and commands, Sprint evidence reconciliation, Codex configuration, and that no product feature, SQL, dependency, API key, or application-level OpenAI integration change was introduced.

## Handoff

Read `AGENTS.md`, `.agent/PLANS.md`, `.agent/code_review.md`, `{active_execplan}`, verification docs, known issues, checked-in config, build/package files, and CI where present. Do not change files.

Report repository/branch/HEAD/working-tree state; completed behavior with file evidence; documented limitations; deployed status that cannot be established from this checkout; current instruction/config sources; stale paths or commands; exact available checks and whether each is static, mocked, or live; any OpenAI API/custom agent integration as distinct from Codex development configuration; and the next bounded task with acceptance criteria. Never infer the active model from prose or default config; use client-visible status when available, otherwise report it as unknown.
