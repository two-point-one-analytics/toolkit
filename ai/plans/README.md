# Plans

Plan files are **agent instructions for resumable work**. Each one holds everything an agent needs to pick up where the last session stopped and to keep iterating on the spec and the decisions behind it.

They are not a work-management system. Prioritization, deadlines, assignees, and status reporting live in your ticketing system. Nothing here ranks or schedules work.

## When To Create A Plan

Create one when any of these is true:

- You want to **iterate on the spec** — the shape of the work is still being decided.
- The work has **multiple steps that must be followed** in order.
- There is **enough work that it could span sessions**.

The threshold is deliberately low. If you are unsure, create the plan. A plan that turns out to be unnecessary costs one file; work lost to a cleared context costs a session.

Anything smaller — a one-line reminder, a follow-up, a loose open loop — is a `note`, not a plan.

## Frontmatter

Three required fields, in this order:

```yaml
---
description: One line. This is the search key used to find the plan later.
status: incomplete
updated: 2026-07-27 16:41
---
```

- **`status`** — `incomplete` or `complete`. Nothing else. Paused is `incomplete`. Blocked is `incomplete` with the blocker in the Progress Log. Superseded is `complete` with `superseded_by` set.
- **`updated`** — when the plan was last **worked**, not last written: the timestamp of the newest Progress Log entry, `YYYY-MM-DD HH:MM` in US/Central, read from the shell rather than guessed:

  ```bash
  TZ=US/Central date +'%Y-%m-%d %H:%M'
  ```

  Set once at creation as the initial value, then bumped only alongside a new Progress Log entry. **Editing the spec does not bump it** — refining the Goal, Success Criteria, or Open Questions is writing, not working. Rule 3 resolution is this field's only consumer, and it has to point at genuinely active work rather than at whichever file was touched most recently; a newly authored spec must not outrank the plan actually being executed. Plans predating this convention carry a date only; both formats sort correctly together and do not need back-populating.

Optional relational fields, used only when the relationship exists:

- `depends_on: <filename>` — cannot proceed until that plan completes.
- `supersedes:` / `superseded_by: <filename>` — replacement lineage.

Do not add priority, size, impact, or maturity fields.

## Open Questions

An optional section holding **decisions the work is blocked on**. Number them `Q1`, `Q2`, … so a Progress Log entry can close one by name: `Q3 resolved — chose X because Y`.

A question earns a `Q` only if answering it is a *decision*. The test is how it closes:

- **Closes by deciding** → it is a question. *"Should the report table stay aggregate-only, or add a detailed flattened table?"*
- **Closes by doing the work** → it is not. That belongs in Success Criteria, or in Next Action if it is the immediate step.

This distinction is the one that erodes. A work item phrased as *"When does X happen?"* reads like a question, but it closes when X is done, and it will duplicate a success criterion tracking the same thing — two records of one status, which the Boundaries below forbid.

Keep resolved questions; do not delete or renumber them, since Progress Log entries refer to them by number. Mark the outcome in the heading (`— RESOLVED: <the decision>`) and move them under a `## Resolved Questions` heading once answered, so the open section stays scannable.

## Finding A Plan

Query frontmatter. Never read plan bodies to search, and do not maintain an index file — the query *is* the index, and it runs in well under a second.

```bash
# Full index
rg --no-heading -N -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^(description|status|updated):' plans/ | sort

# Most recently worked incomplete plans
rg -l -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^status: incomplete$' plans/ | xargs rg -N --no-heading '^updated:' | sort -t: -k3 -r | head -5

# Match a description
rg -N --no-heading -i -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^description:.*<term>' plans/
```

## Skills

| Skill | Purpose |
|---|---|
| `/plan-create` | Create a new plan from `plan-template.md`. Creation only. |
| `/plan-retrieve` | Find a plan and report where it stands. Read-only; does not execute. |
| `/plan-interview` | Refine a plan that has core content — resolve open questions until it is execution-ready. |
| `/plan-step` | Execute the plan one step at a time, waiting for approval at each checkpoint. |
| `/plan-run` | Execute the plan autonomously, stopping only for blockers or unsafe actions. |
| `/plan-checkpoint` | Audit the plan for consistency and completeness before clearing context. |

Every skill that needs to locate a plan uses the **same** resolution rules — pass a path, pass a description, or pass nothing and get the most recently worked incomplete plan.

All of them confirm before acting whenever the plan was inferred rather than named, and all stop to ask when the result is ambiguous — several plans matching a description, or the top two sharing the same `updated` value. Ties are never broken silently.

The skills live in `.github/skills/` and read their conventions from this directory, so a repo carrying `plans/README.md` and `plans/plan-template.md` is self-contained.

## Workflow

1. **Create** — `/plan-create <description>`. Copies `plan-template.md` and fills in what is known. The Progress Log stays empty; creating a plan is not progress.
2. **Retrieve** — `/plan-retrieve` to find a plan and see where it stands without touching it. This is the safe way to ask about a plan.
3. **Refine** *(optional)* — `/plan-interview` when the spec needs decisions resolved before execution is safe.
4. **Execute** — `/plan-step` for approval-gated work, `/plan-run` for autonomous. Both update the plan as they go, appending Progress Log entries and bumping `updated`.
5. **Checkpoint** — `/plan-checkpoint` before clearing context, compacting, or switching away. Execution updates are a secondary concern while executing and small ones get missed; this pass exists to catch them with full attention. Decisions made in conversation but never written down are the most common and most costly miss.
6. **Complete** — set `status: complete`, check off the success criteria, and add a final log entry with outcome and residual risk.

## Boundaries

- **Plans are self-contained.** Everything needed to resume lives in the plan file. No parallel status record anywhere.
- **Personal scratchpads never track plans.** Loose open loops belong in a personal notes file, wherever you keep one. That file is not part of this repo and nothing here depends on it.
- **No index file.** Discovery is a frontmatter query.
- **Prioritization belongs to your ticketing system**, not this directory.
- **The conventions live here, not in global config.** This README and `plan-template.md` are what the skills read, so they travel with the repo.

## Directory Layout

- `README.md` — this file. The contract the plan skills read.
- `plan-template.md` — the skeleton `/plan-create` copies.
- `plans/` — active and paused plans, one file each.

Optional, only if this repo uses them:

- `plans/backlog/` — ideas not yet shaped into executable plans. These participate in discovery by design; their old `updated` values keep them from winning the sort, which is why there is no exclusion rule for them.
- `plans/completed/` — retained as operational history.
- Subdirectories — multi-file plan clusters with their own `INDEX.md` entry point.
