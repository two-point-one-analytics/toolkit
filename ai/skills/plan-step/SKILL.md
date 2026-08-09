---
name: plan-step
description: Work through an existing plan one step at a time — the slow path. Checkpoint after each step and wait for approval before continuing. For autonomous end-to-end execution use /plan-run; to look up a plan without executing use /plan-retrieve.
argument-hint: [plan-file or description]
disable-model-invocation: true
---

# Plan Step

Resolve the target plan, read it and re-anchor on goal, current state, blockers, and next action, then follow the plan sequentially. Before starting, identify the current step and expected checkpoint boundary without giving a long explanation of every later step. For autonomous end-to-end execution instead, use `/plan-run`.

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

## Approval-Gated Execution

Do not front-load a long explanation of all remaining steps when this skill is activated. Re-anchor on the plan, then focus on the immediate next step.

After each numbered plan step or meaningful sub-step:

- Summarize what changed.
- List files created or modified.
- Call out blockers, decisions, or scope changes.
- Update the plan when meaningful progress, decisions, blockers, or next actions change. Every Progress Log entry bumps frontmatter `updated` in the same edit, using `TZ=US/Central date +'%Y-%m-%d %H:%M'` — read the value from the shell, never from memory.

Before asking for approval to continue, preview the next step:

- **Purpose** — why this step matters to the plan goal.
- **Actions** — the specific work you intend to perform before the next checkpoint.
- **Boundary** — where you will stop and ask again.

Keep the preview brief enough to approve without opening the plan file. Avoid restating unrelated future steps unless they affect the decision. Then wait for explicit user approval before continuing.

## Rules

- Do not skip ahead unless the user asks.
- Stop immediately before destructive actions, commits, external writes, or ambiguous requirements.
- Wait for explicit approval before continuing past a checkpoint.
- Keep the preview short enough to approve without opening the plan file.

## Completion

When all success criteria are met, set `status: complete`, and add a final Progress Log entry with outcome, verification, and residual risk. Ask before moving, deleting, or summarizing the plan elsewhere.

## Smoke Test

Prompt: `/plan-step`.

Expected behavior: resolve to the most recently worked incomplete plan via the rule-3 query, state which plan was selected and why since it was inferred, re-anchor on goal and next action, preview the immediate step's purpose, actions, and boundary — then wait for approval without starting the work.
