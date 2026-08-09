# Git Commit Convention

Verb-first imperative commit messages, used across all repos by both manual commits and agent-generated ones.

## Context

Work spans business management, analysis, documentation, and coding — not classic dev work alone — so the dev-centric Conventional Commits `type:` taxonomy (`feat:`/`fix:`/`docs:`) is a poor fit. Verb-first imperative is also Git's own native guidance (imperative mood, ~50-char subject), so this is not fighting a standard; it only declines the Conventional Commits tooling overlay. No changelog/semver automation is in use, so the prefix would be pure ceremony.

## Convention

Format: `<verb> <what> <where>`

- lowercase, imperative, ~50-char subject, no trailing period
- no `type:` prefix — the verb encodes the type
- `<where>` is optional and written as natural prose (`… to the ingest pipeline`, `deploy runbook`), not a bracketed scope
- body only when the "why" isn't obvious from the diff

### Verbs (exactly 6)

- `add` — new thing
- `fix` — broken thing now works
- `update` — change existing thing (absorbs CI, dependency bumps, config, and housekeeping/meta work)
- `remove` — delete thing
- `refactor` — restructure with no behavior change
- `document` — docs, comments, runbooks

### Examples

- `add dedup logic to the ingest pipeline`
- `fix webhook delay timing`
- `update deploy runbook`
- `remove unused queue dependency`

## Notes

- Verbs map 1:1 to Conventional Commits if changelog tooling is ever needed: `add`→feat, `fix`→fix, `update`→chore/build/ci, `remove`→chore, `refactor`→refactor, `document`→docs.
- Consistency between manual commits and agent-generated commits is the goal; a `commit` skill can encode the same rules.
