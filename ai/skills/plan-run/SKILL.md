---
name: plan-run
description: Drive an existing plan to completion autonomously — the fast path. Proceed without routine confirmation, stopping only for blockers or unsafe actions. For step-by-step approval use /plan-step; to look up a plan without executing use /plan-retrieve.
argument-hint: [plan-file or description]
disable-model-invocation: true
---

# Plan Run

Resolve the target plan, read it and re-anchor on goal, current state, blockers, and next action, then follow the plan sequentially and proceed without routine confirmation. Keep the user informed at major milestones, but do not pause after every step. For step-by-step, approval-gated execution instead, use `/plan-step`.

## Resolving The Plan

Query frontmatter — never read plan bodies to search.

Take the first rule that matches:

1. **Explicit path or filename** — use it directly.
2. **Description** — match the argument against `description`.
3. **No argument** — take the most recently worked incomplete plan, by `updated`.

```bash
# Rule 3 — most recently worked incomplete plans
rg -l -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^status: incomplete$' plans/ | xargs rg -N --no-heading '^updated:' | sort -t: -k3 -r | head -5

# Rule 2 — match a description
rg -N --no-heading -i -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^description:.*<term>' plans/
```

All three exclusions are load-bearing. `README.md` and `plan-template.md` carry example frontmatter that matches these patterns, and the template's `YYYY-MM-DD` placeholder sorts *above* real dates — dropping that guard silently resolves to the template. `INDEX.md` excludes plan-cluster entry points, which are navigation rather than resumable work.

`plans/README.md` holds the full frontmatter contract. Read it only if resolution fails or the repo's conventions look non-standard.

**Confirm before executing whenever the plan was inferred rather than named explicitly.** State which plan you selected and why.

**Ask rather than guess when the result is ambiguous:** more than one plan matches the description, or the top two entries share the same `updated` value. List the candidates and let the user choose. Never break a tie silently.

This path runs autonomously, so resolving to the wrong plan is the one genuinely costly failure — when in doubt, ask.

## Execution

Stop and ask before:

- Destructive or irreversible actions.
- Git commits or pushes unless explicitly requested.
- External service changes, credential use, or production-impacting actions.
- Requirements conflicts or unclear success criteria.

Keep the plan updated as meaningful milestones, decisions, blockers, or next actions change. Every Progress Log entry bumps frontmatter `updated` in the same edit, using `TZ=US/Central date +'%Y-%m-%d %H:%M'` — read the value from the shell, never from memory.

When complete, report what changed, commands run, verification results, and any residual risk.

## Rules

- Proceed without routine confirmation — this is the fast path. Do not pause after every step.
- Stop and ask in every case listed under Execution above. The autonomy applies to routine steps only, never to that list.
- Confirm the plan selection whenever it was inferred rather than named explicitly.
- Never break a resolution tie silently; list the candidates and ask.
- Keep the user informed at major milestones.

## Completion

When all success criteria are met, set `status: complete`, and add a final Progress Log entry with outcome, verification, and residual risk. Ask before moving, deleting, or summarizing the plan elsewhere.

## Smoke Test

Prompt: `/plan-run secret manager`.

Expected behavior: match `description` against the frontmatter query, confirm the inferred selection before executing, then work the plan end to end — appending Progress Log entries and bumping `updated` as milestones land, and pausing only for the Execution stop list or a genuine ambiguity.
