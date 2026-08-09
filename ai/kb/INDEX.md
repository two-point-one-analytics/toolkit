# Knowledge Base

The reasoning behind the skills and agents in this repo, kept alongside them so the *why* travels with the *what*.

These are reference entries, read on demand — they do not install anywhere. Filenames are lowercase-hyphenated and scoped to one topic; each entry is a one-line summary, then context, then the learning.

## Topics

- **knowledge-management.md** — the four-category memory model, the three states of context availability, the promotion and staleness gaps, and non-destructive extraction from a human record.
- **skill-portability.md** — how skills resolve across personal and repository scope, why an author never exercises what they ship, `$HOME` path-anchoring conventions, and the two-class portability scan.
- **docs-knowledge-system.md** — the anchor/corpus split for repository documentation, the retrieval wiring that makes a searchable pool reachable, section-tag and search-header conventions, the generated tag index, and the anchor index drift check.
- **git-commit-convention.md** — verb-first imperative commit message convention.
- **javascript-conventions.md** — JavaScript and TypeScript naming, formatting, and structure conventions.
- **mermaid.md** — Mermaid diagram and chart patterns for Markdown documents, including renderer compatibility.

## Keeping this index honest

This index is hand-maintained, so it can drift from the directory silently — an agent that reads it concludes a missing entry does not exist. Check both directions:

```sh
for f in kb/*.md; do b=$(basename "$f"); [ "$b" = "INDEX.md" ] && continue
  grep -q "$b" kb/INDEX.md || echo "NOT IN INDEX: $b"; done
```

`knowledge-management.md` explains why an index is kept at all rather than replaced by a search: an index answers *"what exists that I should know about?"*, a query answers *"where is the thing I already know I want?"*
