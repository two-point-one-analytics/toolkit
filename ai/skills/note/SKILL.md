---
name: note
description: This skill should be used to add a quick, single-item note to the personal notes file — a follow-up, open loop, reminder, or next step that is useful soon but does not belong in a plan, repo docs, or the KB. To read what is open, use `note-recap`; for multi-step work, use `/plan-create` instead.
argument-hint: [the note]
---

# Note

Append one short item to `$HOME/notes/ai-notes.md`.

One file holds everything, across every repo and every workstream. There is no per-repo file and no repo lookup — the note carries its own context via a tag.

## Entry Format

Newest first, directly under the `# Notes` heading:

```markdown
- 2026-07-27 `acme-api` — rate limiter drops bursts above 50/s; retry path unverified
```

- **Date** — today, `YYYY-MM-DD`. Read it from the shell rather than memory: `TZ=US/Central date +%F`.
- **Tag** — the current git repo basename in backticks. Outside a repo, use a short topic label, or omit the tag if none fits. Never invent a repo name.
- **Text** — one line. If it needs more than a line or two, it belongs in a plan.

## What Belongs

- Open loops, follow-ups, reminders, and temporary decisions.
- Preferences or operating rules under trial.
- Promotion candidates for the KB, instructions, or repo docs that are not settled yet.

## What Does Not

- **Multi-step work** — use `/plan-create`. Anything with several steps, a spec worth iterating, or work spanning sessions is a plan, not a note.
- **Plan status** — plans are self-contained. Never mirror a plan's progress, next action, or blockers here.
- **Durable knowledge** — use `remember` for reusable lessons that should shape future reasoning across repos.
- **Repo-specific knowledge worth keeping** — gotchas, rationale, landmines that help someone understand *this* repo — use `capture`, which writes to that repo's `docs/capture.md` and travels with it.
- **Completed work** — this file is not a historical log. Nothing is kept for the record.

## Workflow

1. Read `$HOME/notes/ai-notes.md`. Create it with a `# Notes` heading if absent.
2. If an existing open item covers the same thing, update it rather than appending a duplicate.
3. Otherwise add a new entry at the top of the list.
4. Remove entries the current conversation has clearly resolved.
5. Confirm what was written.

Ask before pruning or rewriting an entry only when its status is ambiguous or the cleanup would require interpreting intent.

## Smoke Test

Prompt: `/note follow up on the agent env permission behavior`.

Expected behavior: append one dated, repo-tagged line to `$HOME/notes/ai-notes.md` and write nothing else.
