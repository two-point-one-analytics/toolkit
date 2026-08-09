---
name: remember
description: This skill should be used to store durable memory from recent conversation or supplied text. It routes current-repo context to repo docs and stable broader-than-current-project knowledge to `$HOME/kb/`.
---

# Remember

Durable memory router for information that should survive beyond the current session.

Use this skill when information should be stored as long-term durable memory. Do not use this for active plan updates or temporary notes; use the plan file itself for active work progress and `note` for short-lived context.

## Routing Rules

- Current repo docs: context that should travel with the repository for future contributors or agents.
- `$HOME/kb/`: stable reusable knowledge that answers "how should we think, build, validate, or debug next time?" from any repo.
- A personal journal or notes app: human-owned record; review as source material but do not move or mark notes processed unless explicitly requested.

If a capture is active work progress, route it to the relevant plan file. If a capture answers both reusable and historical questions, preserve the full operational detail in current repo docs and extract only the reusable lessons into global `kb/`.

## Scope Routing

- Use repo docs when the information explains how to understand, run, maintain, review, or resume work in the current repo.
- Use global KB only for low-churn, stable reusable knowledge that should shape future agent behavior from any repo.
- Use more than one destination only when current-repo detail is canonical but a concise global KB summary should remain discoverable.
- For repo docs, first check root instructions, then `docs/INDEX.md` for the documentation map. Follow existing repo conventions over global defaults.
- If the destination is unclear, ask whether the content should live in the current repo, global KB, or more than one place. Do not write to another repo by default except for the canonical global KB.

## Workflow

1. Identify the source material: recent conversation, changed files, supplied text, or `$ARGUMENTS`.
2. Classify the content using the routing rules.
3. For active or paused multi-step work tracking, use the plan file instead of `remember`.
4. For repo-local documentation, read root instructions and `docs/INDEX.md` when present before choosing a destination.
5. For reusable KB entries, follow the KB write workflow below.
6. For historical or operational reference material, create or update a current-repo doc following local conventions.
7. For personal journal or notes-app sources, extract destination content without modifying the source unless explicitly requested.
8. Ask before reorganizing existing files, merging docs, changing archival structure, or creating a new repo docs layout.
9. Confirm what was written and where.

## KB Write Rules

Use `kb/` for reusable platform behavior, data quirks, analytical methods, workflow patterns, and tooling learnings. Cross-project learnings belong by default. Project-specific learnings belong only when they clarify a reusable pattern; tag them inline.

Do not store task status, half-formed notes, private scratchpad items, or reference history in `kb/`. If the item documents what exists, how a specific tool works, what happened previously, or why prior decisions were made, route it to current repo docs. If it is active work progress, route it to the relevant plan file.

### KB Workflow

1. Read `$HOME/kb/INDEX.md`.
2. Choose an existing lowercase-hyphenated topic file when possible.
3. Read the relevant existing topic file before writing to avoid duplicate or conflicting entries.
4. Search nearby terms across `$HOME/kb/` when the learning could belong in multiple topic files.
5. If a new topic file is needed, propose the filename and ask before creating it or changing `INDEX.md`.
6. Write a finished, scannable entry with a one-line summary, enough context, and the durable learning.
7. Surface intended file changes before writing when the capture is non-trivial or restructures existing content.
8. Re-read a touched file only when confirming placement within a larger file you did not fully see; otherwise rely on the edit result.
9. Confirm the file touched and whether `INDEX.md` changed.

### KB Entry Format

Use this structure for new entries unless the existing topic file has a clearer local convention:

```markdown
## Short Topic Title

Summary: One-line durable takeaway.

Context: Why this matters or when it was observed.

Learning: The reusable fact, behavior, or practice.
```

Keep entries concise. Prefer one sharp entry over a broad note that mixes unrelated learnings.

### KB File Conventions

- Keep the KB flat until `INDEX.md` becomes hard to scan.
- Use lowercase-hyphenated filenames scoped to one topic.
- Prefer appending a clear `##` section over creating many tiny files.
- Keep entries concise and factual; avoid speculative advice unless clearly labeled.

## Reference Doc Format

Use this structure for durable operational reference docs unless the current repo has a clearer local convention:

```markdown
# Reference Title

One-line description.

## Purpose

Why it exists and what problem it solves.

## Architecture

How it works.

## Operations

How to use, deploy, maintain, or troubleshoot it.

## Decisions

Important choices and rationale.

## Links

Repos, docs, and related references.
```

Keep reference docs factual. Put generalized lessons in `kb/`, not only inside historical reference material.

## Smoke Test

Prompt: `/remember Copilot reads personal skills from $HOME/.copilot/skills`.

Expected behavior: route durable reusable tooling knowledge to the KB only after checking the KB index and relevant topic files, or ask if destination is unclear.
