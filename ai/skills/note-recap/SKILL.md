---
name: note-recap
description: This skill should be used to report what is currently open in the personal notes file — follow-ups, open loops, reminders, and unresolved decisions across every repo and workstream. The read counterpart to `note`. Read-only. For plans, use /plan-retrieve; plans are not tracked here.
argument-hint: [optional repo or topic tag to filter by]
---

# Note Recap

Report what is still open. Read-only — do not modify files.

This is the read side of `note`. It covers notes only. Plans are self-contained and discoverable by frontmatter query — use `/plan-retrieve` for those.

## Workflow

1. Read `$HOME/notes/ai-notes.md`.
2. Report every open item, newest first, preserving the date and tag on each.
3. If an argument was given, filter to entries carrying that tag and say how many were hidden.
4. If the file does not exist or has no entries, say so and stop. That is not an error — it means nothing is loose.

When invoked inside a git repo with no argument, still report everything. Mention which entries match the current repo, but do not hide the rest; the cross-repo view is the point of a single file.

## Rules

- List everything. Do not pick an item or infer which one the user means.
- Do not read plan files, repo docs, or the vault beyond `notes.md`.
- Group by tag when there are more than roughly a dozen entries; keep a flat list below that.
- Flag entries that look stale — old date, no clear next step, or superseded by work since done — but do not fix them. Suggest `note-review` for cleanup.
- Do not modify files.

## Smoke Test

Prompt: `/note-recap`.

Expected behavior: list every open note with its date and tag, flag any that look stale, and read nothing else.
