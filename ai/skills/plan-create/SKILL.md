---
name: plan-create
description: This skill should be used to create a new plan file in the current repo when none exists yet — for work the user wants to iterate the spec on, work with multiple steps that must be followed in order, or work that could span sessions. Use when the user asks to plan something out, write a plan, or scope a piece of work. Creates only; to look up an existing plan use /plan-retrieve, and to execute one use /plan-step or /plan-run.
argument-hint: [description of the work]
---

# Plan Create

Create a new plan file. This skill creates only — it does not look up, refine, or execute plans.

If a plan for this work may already exist, use `/plan-retrieve` first. Do not create a duplicate.

## When Work Qualifies

Create a plan when any of these is true:

- The user wants to **iterate on the spec** — the shape of the work is still being decided.
- The work has **multiple steps that must be followed** in order.
- There is **enough work that it could span sessions**.

The threshold is deliberately low. When unsure, create the plan — an unnecessary plan costs one file, while work lost to a cleared context costs a session.

Anything smaller — a one-line reminder, a follow-up, a loose open loop — is a `note`, not a plan.

## The Repo Owns The Conventions

Plan conventions live in the repo, not in this skill, so they travel with the repo at handoff.

1. Read `plans/README.md` in the current repo — the frontmatter contract, criteria, and queries.
2. Read `plans/plan-template.md` — the skeleton to copy.

**If either file is missing**, scaffold it from `seed/` in this skill directory before continuing. Copy verbatim; do not improvise. Never overwrite a file that already exists — the repo's copy always wins, even where it diverges from the seed.

Copy from `seed/` **in the directory this SKILL.md was loaded from** — not a hardcoded home path. This skill runs both from `$HOME/.copilot/skills/` and from a repo-local `.github/skills/`, and the repo-local copy must use its own seed.

```bash
mkdir -p plans && cp -n <this-skill-dir>/seed/README.md <this-skill-dir>/seed/plan-template.md plans/
```

`cp -n` will not clobber an existing file. Report which files were created.

The seed is a starting point, not a mirror — once a repo has its own copies, they are authoritative and may diverge. Do not "resync" a repo back to the seed.

Bootstrapping a repo without creating a plan is a valid request on its own. If the user asks only to set up plan conventions, run the scaffold, report what was created, and stop.

If the repo has no `plans/` directory at all, check root instructions and `docs/INDEX.md` for where plans belong before creating one. Some repos do not use plans.

## Workflow

1. Confirm the work qualifies. If it does not, say so and suggest `note` instead.
2. Read `plans/README.md` and `plans/plan-template.md`, scaffolding from `seed/` if absent.
3. Propose a lowercase-hyphenated filename derived from the work, not from the conversation topic.
4. Copy the template verbatim and fill in what is known: `description`, `status: incomplete`, `updated`, Goal, Success Criteria, Next Action.
5. Set `updated` from the shell, never from memory:

   ```bash
   TZ=US/Central date +'%Y-%m-%d %H:%M'
   ```

   This is the field's **initial value only**. `updated` means last *worked*, not last written — from here it is bumped solely alongside a new Progress Log entry, so later edits to the spec leave it alone.

6. Leave the Progress Log empty until real work happens. Creating a plan is not progress.
7. State the filename created and what to do next — `/plan-interview` if the spec has open questions, `/plan-step` or `/plan-run` if it is ready to execute.

## Rules

- Create without asking when invoked with a description. State the filename chosen rather than requesting approval for it.
- Ask first only when the destination repo or the scope is genuinely unclear.
- Do not write plans into another repo by default.
- Do not populate the Progress Log at creation time.
- Do not add priority, size, impact, or maturity fields. Prioritization lives in your ticketing system.
- Do not create or maintain a `plans/INDEX.md`. Discovery is a frontmatter query; an index is a cache needing invalidation.

## Smoke Test

Prompt: `/plan-create migrate the reporting database to dbt Core`.

Expected behavior: confirm the work qualifies, read the repo's plan docs, create `plans/reporting-db-dbt-core-migration.md` from the template with frontmatter and Goal filled in, an empty Progress Log, and report the filename plus the suggested next skill.
