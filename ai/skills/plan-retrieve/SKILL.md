---
name: plan-retrieve
description: This skill should be used to find an existing plan and report where it stands — goal, what is done, blockers, and the next action — without changing or executing anything. Use when the user asks what the plan for something is, where a plan stands, to summarize or review a plan, or which plan they were last working on. Read-only. To create a new plan use /plan-create; to act on one use /plan-interview, /plan-step, or /plan-run.
argument-hint: [plan-file or description]
---

# Plan Retrieve

Find a plan and report its current state. Read-only — do not modify any file, and do not begin the work the plan describes.

This is the safe way to ask about a plan. Executing is a separate, explicit action.

## Resolving The Plan

Query frontmatter — **never read plan bodies to search**; that is what `description` exists for.

Take the first rule that matches:

1. **Explicit path or filename** — use it directly.
2. **Description** — match the argument against `description`.
3. **No argument** — take the most recently worked incomplete plan, by `updated`.

```bash
# Rule 3 — most recently worked incomplete plans
rg -l -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^status: incomplete$' plans/ | xargs rg -N --no-heading '^updated:' | sort -t: -k3 -r | head -5

# Rule 2 — match a description
rg -N --no-heading -i -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^description:.*<term>' plans/

# Full index — every plan, for a survey
rg --no-heading -N -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^(description|status|updated):' plans/ | sort
```

All three exclusions are load-bearing. `README.md` and `plan-template.md` carry example frontmatter that matches these patterns, and the template's `YYYY-MM-DD` placeholder sorts *above* real dates — dropping that guard silently resolves to the template. `INDEX.md` excludes plan-cluster entry points, which are navigation rather than resumable work.

`plans/README.md` holds the full frontmatter contract. Read it only if resolution fails or the repo's conventions look non-standard.

**Ask rather than guess when the result is ambiguous:**

- More than one plan matches the description.
- The top two entries share the same `updated` value.

List the candidates and let the user choose. Never break a tie silently.

If the repo has no `plans/` directory, say so rather than searching elsewhere.

## Report

Read the resolved plan and report:

- **Goal** — one line.
- **Status** — `incomplete` or `complete`, and how many success criteria are met.
- **Next Action** — verbatim from the plan.
- **Blockers and open questions** — only if present.
- **Last worked** — the `updated` timestamp.

Keep it short enough to act on without opening the file. Do not restate the whole progress log; summarize only what bears on resuming.

Flag it plainly if the plan looks stale — `updated` far in the past, a Next Action that appears already done, or success criteria inconsistent with the log — and suggest `/plan-checkpoint` to reconcile it.

## Handoff

End by naming the next explicit action, without taking it:

- `/plan-interview` — the spec still has open questions.
- `/plan-step` — ready to execute with approval gates.
- `/plan-run` — ready to execute autonomously.
- `/plan-checkpoint` — the plan looks out of date and should be reconciled first.

## Rules

- Do not modify files — not the plan, not any repo doc.
- Do not start the work, even if the Next Action looks trivial.
- Do not read other plans' bodies. The full-index query above is the survey; reading past the resolved plan is not.
- For loose open loops and follow-ups, use `note-recap`. Those are notes, and plans are never mirrored there.

## Smoke Test

Prompt: `/plan-retrieve secret manager`.

Expected behavior: match `description` against the frontmatter query, resolve to the GCP Secret Manager plan, report goal, status, next action, and last-worked timestamp, then name the next skill without executing anything.
