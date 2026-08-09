---
name: note-review
description: This skill should be used to review and prune the personal notes file — remove resolved items, consolidate duplicates, and flag entries that have outgrown a note. Use when the user asks to clean up notes, prune, or when the list has grown noisy. Writes only to the notes file.
---

# Note Review

Review `$HOME/notes/ai-notes.md` and keep it short enough to read in one pass.

The file holds only what is still open. It is not a historical log — nothing is retained for the record.

## Goals

- Every entry is an open loop, follow-up, reminder, or unresolved decision with a clear next step.
- Duplicates and near-duplicates are consolidated into one line.
- Resolved, obsolete, or superseded entries are removed.
- Entries that should have been plans are flagged for promotion, not silently deleted.
- Entries older than 30 days are surfaced in the report. **Age alone is never a reason to remove one** — an old entry is often the latest word on a topic nobody has revisited.
- Ambiguous items are raised for a decision rather than resolved by assumption.

## Boundaries

- Read and write only `$HOME/notes/ai-notes.md`. Nothing else in the vault.
- Do not modify `plans/`, repo docs, `kb/`, or any project repo.
- Do not commit anything.
- Read a referenced plan or doc only when needed to judge whether an entry is still current.

## Cleanup Rules

Safe to remove or merge without asking, when the evidence is in the file itself or the current conversation:

- A duplicate entry repeating the same open loop, next step, or question.
- An entry whose next step is explicitly completed or replaced by a later entry.
- An open question answered by a later entry.
- An entry describing finished work with no unresolved next step, blocker, or promotion decision.
- A pointer to a plan file that no longer exists or whose frontmatter reads `status: complete`.
- Plan status mirrored here — plans are self-contained. If the detail is not already in the plan file, move it there rather than deleting it.

Ask before removing when any of these are true:

- It is old but still appears to be the latest word on that topic.
- It contains an unresolved decision, blocker, approval gate, or promotion candidate.
- It may map to an external commitment, stakeholder request, ticket, or repo not inspected.
- The work may be deferred or intentionally parked rather than dead.
- Judging it would require interpreting intent rather than applying explicit evidence.

## Promotion

Some entries have outgrown the file. Flag rather than delete:

- **Multi-step work** → should be a plan. Recommend `/plan-create`.
- **Durable lessons** → should be reusable cross-repo memory. Recommend `remember`.
- **Repo-specific facts** → belong in that repo. Recommend `capture`, which writes to its `docs/capture.md`.

Recommend the move and let the user decide. Do not create plans or KB entries from this skill.

## Workflow

1. Read `$HOME/notes/ai-notes.md`.
2. Identify duplicates, resolved items, stale entries, promotion candidates, and anything older than 30 days. Get today's date from the shell — `TZ=US/Central date +%F` — rather than assuming it.
3. Read directly referenced plans or docs only where needed to judge an item.
4. Apply the safe cleanups. Preserve each surviving entry's date and tag.
5. Collect everything uncertain under **Needs Decision** rather than acting on it.
6. Report.

## Output Contract

Report in this order:

- **Removed** — entries deleted, and why.
- **Consolidated** — entries merged, and into what.
- **Promotion candidates** — entries that should become plans, memory, or repo docs.
- **Aged** — surviving entries older than 30 days, with their dates and age in days. Listed for visibility only; removing one still requires the same evidence as any other entry.
- **Needs Decision** — uncertain items, with a recommendation for each.
- **Remaining** — the open list after cleanup, with a count.

If nothing is safe to change without input, make no edits and return only the findings and questions.

## Smoke Test

Prompt: `/note-review`.

Expected behavior: prune clearly resolved and duplicate entries from `notes.md`, flag anything multi-step as a plan candidate, and report ambiguous items without deleting them.
