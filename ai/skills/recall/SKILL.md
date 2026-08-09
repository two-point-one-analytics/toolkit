---
name: recall
description: This skill should be used to retrieve durable memory from current repo docs and `$HOME/kb/` when the user asks to recall, look up, check memory, check prior context, or when a task likely depends on captured experience, platform behavior, internal tool history, prior decisions, or durable context.
---

# Recall

Search durable memory before answering from general knowledge when the user asks for, or the task likely depends on, remembered context, captured experience, prior decisions, internal tool history, or platform behavior.

## Scope

- Current repo docs: context that should travel with the active repository; use root instructions for repo guidance and `docs/INDEX.md` as the documentation map.
- `$HOME/kb/`: reusable captured knowledge, platform behavior, data quirks, analytics methods, workflow conventions, and tooling lessons.
- `plans/`: check only when the user asks about active, paused, pending, or in-progress work. Query frontmatter rather than reading plan bodies; the repo's `plans/README.md` holds the canonical queries. For one plan's current state, use `/plan-retrieve`.
- `$HOME/notes/ai-notes.md`: check only when the user asks about temporary notes, recent status, or current-session reminders. One file covers all repos; entries are tagged with the repo basename.

## Workflow

1. For current-repo questions, check root instructions, then `docs/INDEX.md`, then relevant repo docs following the repo map and existing conventions.
2. For reusable platform, analytics, or tooling knowledge, read `$HOME/kb/INDEX.md`, then likely topic files.
3. Search focused keywords across the relevant repo docs and `kb/` when the topic is ambiguous or could span files.
4. Read only relevant files or sections.
5. If the user asks about active work, read `$HOME/notes/ai-notes.md` first, filtering to entries tagged with the current repo, then read only referenced plans/docs. If the notes file does not exist, say so before inspecting plans/docs.
6. Answer with source transparency: distinguish recalled memory from repo docs, global KB, and general knowledge.
7. If useful durable memory is missing, offer to capture it with `remember`.

## Answer Contract

- Cite source files for recalled claims.
- Say `Nothing found in durable memory on <topic>` when no relevant entry exists.
- Keep answers focused on retrieved memory unless the user asks for broader advice.
- Label any supplemental general knowledge clearly when platform, tool, or internal-history behavior matters.

## Smoke Test

Prompt: `/recall warehouse migration decisions`.

Expected behavior: search current repo docs/plans/KB as relevant, cite files, and distinguish recalled memory from general knowledge.
