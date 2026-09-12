# code_review.md — Codex Review Standard

Use this checklist when asking Codex to review a branch, PR diff, uncommitted changes, or a commit.

## Review priorities

1. Correctness: Does the change implement the requested behavior?
2. Tenant isolation: Can one tenant access another tenant's data?
3. Security: Are auth, permissions, uploads, SQL, and secrets handled safely?
4. API contract: Does public JSON use clean fields like `request_id`, `flow_id`, `created_at`?
5. Error handling: Are client mistakes returned as 400/401/403/404 instead of 500?
6. Tests: Are relevant tests added or updated?
7. Maintainability: Is logic in the correct service/component layer?
8. UX: Are loading, empty, error, and success states handled?
9. Documentation: Are OpenAPI, README, or demo docs updated?
10. Regression risk: What existing behavior might break?

## Codex review prompt

Review `{head_ref}` against `{base_ref}`. Read `AGENTS.md`, `.agent/PLANS.md`, this file, and `{active_execplan}` first. Confirm the actual refs, working-tree scope, available commands, and whether referenced paths exist before drawing conclusions.

Keep the review read-only. Focus on correctness, tenant isolation, security, public API naming, test coverage, documentation accuracy, regressions, and whether checks are real for this checkout. Distinguish mocked/static evidence from live-service evidence. Return prioritized findings with file paths and suggested fixes, then residual risk and missing verification.
