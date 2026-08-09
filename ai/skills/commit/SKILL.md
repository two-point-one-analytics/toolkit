---
name: commit
description: This skill should be used when the user asks to commit, save changes in git, or run /commit. It stages and commits relevant git changes, and pushes only when explicitly requested.
---

# Commit

Create small, accurate git commits from relevant changes only.

## Rules

- Commit only when explicitly requested or when a commit/checkpoint workflow is approved.
- Push only when explicitly requested.
- Never amend, reset, rebase, or force push unless explicitly requested.
- Never use `git add .` or `git add -A`.
- Never stage secrets: `.env`, credentials, tokens, private keys, or generated auth files.
- If unrelated changes are present, leave them alone or ask whether to split/include them.
- If there is nothing to commit, say so; do not create an empty commit.

## Workflow

1. Inspect repository state:
   - `git status --short`
   - `git diff`
   - `git diff --cached`
   - `git log --oneline -5`

2. Identify relevant files:
   - Include only files related to the requested work.
   - Preserve unrelated user or agent changes.

3. Draft commit message (verb-first convention):
   - Format: `<verb> <what> <where>` — lowercase, imperative, ~50-char subject, no trailing period.
   - No `type:` prefix; the verb encodes the type.
   - Verbs (use exactly these six):
     - `add` — new thing
     - `fix` — broken thing now works
     - `update` — change existing thing (absorbs CI, dependency bumps, config, housekeeping/meta work)
     - `remove` — delete thing
     - `refactor` — restructure with no behavior change
     - `document` — docs, comments, runbooks
   - `<where>` is optional, written as natural prose (e.g. `… to the ingest pipeline`), not a bracketed scope.
   - Add a body only when the "why" isn't obvious from the diff.
   - Examples: `add dedup logic to the ingest pipeline`, `fix webhook delay timing`, `update deploy runbook`, `remove unused queue dependency`.
   - Full reference: `kb/git-commit-convention.md` in this repo.

4. Stage relevant files explicitly:
   - `git add path/to/file path/to/other-file`

5. Commit:
   - `git commit -m "verb summary"`

6. Verify:
   - `git status --short`
   - Report commit hash, message, branch, and remaining uncommitted changes.

## Hook Failures

- If a hook modifies files, inspect the new diff.
- If the commit failed, fix issues and create a new commit.
- Do not amend unless the user explicitly requests it.

## Smoke Test

Prompt: `/commit docs checkpoint update` in a repo with a harmless docs diff.

Expected behavior: inspect status/diff/log, stage only relevant non-secret files explicitly, commit once, and report hash plus remaining changes.
