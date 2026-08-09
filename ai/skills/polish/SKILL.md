---
name: polish
description: This skill should be used to clean up code for readability without functional changes — formatting, comments, naming, and small dead-code issues — applying the relevant language convention docs (loaded on demand) while preserving behavior and existing style. Defer reuse, efficiency, and altitude cleanups to a separate pass.
---

# Polish

Behavior-preserving code cleanup for work that is functionally complete.

Polish should make code clearer and easier to navigate without changing runtime behavior, public APIs, data semantics, or test expectations.

## Conventions (load on demand)

Before editing, identify the language(s) of the target and load the matching convention doc if it exists, then apply it:

- JavaScript / TypeScript -> `$HOME/kb/javascript-conventions.md`
- SQL -> `$HOME/kb/sql-conventions.md`

Repo-local conventions take precedence when more specific (a repo's `AGENTS.md`, formatter config, or runtime/environment constraints). If no convention doc exists for the language, apply general behavior-preserving cleanup.

## Relationship to `simplify`

`polish` owns **readability**: formatting, comments, naming, scannability, and small dead-code removal, aligned to the loaded conventions. Defer **reuse, efficiency, and altitude** work (deduplication, performance rewrites, structural refactors) to the native `simplify` skill. When both are wanted, run `polish` for convention-aligned readability and `simplify` for structural cleanup.

## Allowed Changes

- Simplify verbose local logic only when it improves readability and the behavior is obviously identical (defer dedup/reuse to `simplify`).
- Improve names for variables, functions, comments, or nearby private helpers.
- Improve formatting, ordering, and scannability while preserving project style.
- Add, remove, or refine comments so they explain why non-obvious code exists.
- Remove obvious dead code, redundant temporaries, or unnecessary branches.

## Avoid

- Functional changes, bug fixes, feature work, or behavior-preserving claims that are not obvious.
- Broad refactors, architecture changes, public API changes, or data/model semantic changes.
- Reuse, deduplication, and performance rewrites — those belong to `simplify`.
- New abstractions unless they clearly reduce local complexity.
- Test expectation changes unless the user explicitly requests them.

## Workflow

1. Inspect the target diff or files.
2. Identify the language(s) and load the matching convention doc (repo-local conventions first); see Conventions.
3. Identify behavior-preserving clarity improvements only, aligned to those conventions.
4. Improve names, comments, formatting, and scannability; remove obvious dead code.
5. Defer reuse, efficiency, and structural cleanups to `simplify`; do not perform them here.
6. Preserve existing style, structure, public APIs, data semantics, tests, and runtime behavior.
7. Review the diff for accidental behavior changes.
8. Run focused formatting, lint, or tests when appropriate, or report why they were not run.
9. Report what was polished, which conventions were applied, and any issue intentionally left unchanged.

Polish should reduce friction, not broaden scope.

## Smoke Test

Prompt: `/polish src/example.js`.

Expected behavior: load `kb/javascript-conventions.md`, make only behavior-preserving readability edits aligned to it, defer structural cleanup to `simplify`, review the diff, and report what was polished plus verification or why it was skipped.
