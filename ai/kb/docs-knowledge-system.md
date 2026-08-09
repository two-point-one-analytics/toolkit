# Searchable Docs System

Summary: A two-tier structure for repository documentation — a small curated index plus an unbounded searchable pool — with the retrieval wiring that makes the second tier reachable.

Context: A hand-maintained index does not scale to high-volume mixed-topic content. Every addition needs an index edit, so it drifts; and descriptions bloat as the list grows. But removing the index entirely leaves an agent with no awareness of what exists. This resolves the tension by splitting docs on two independent axes and indexing only one quadrant.

Assumes the **primary reader is an agent that greps and reads**, so the content is the index: retrieval is full-text search plus section tags, not navigation.

## Content model — anchor and corpus

Two properties are usually conflated. Separating them says where every doc type lives.

- **Axis 1 — in the map vs. found by search.** Being linked from `docs/INDEX.md` confers *awareness*, at the cost of weight on the one file every agent reads first.
- **Axis 2 — fixed/foundational vs. accretive.** A small stable set vs. a growing pile.

The **anchor** is the single quadrant that is both in-map and fixed. Everything else is the **corpus** — search-found and accretive.

| Type | In the map | Lifecycle | Home |
|---|---|---|---|
| **Anchor docs** | yes | fixed, foundational | `docs/*.md` + `INDEX.md` |
| **Guides** | no | accretive, polished | `docs/guides/` |
| **Meeting notes** | no | accretive, semi-structured | `docs/meetings/` |
| **Capture** | no | accretive, raw | `docs/capture.md` |

```
docs/
  INDEX.md          # the map: anchor docs + a "Beyond this index" pointer
  <anchor>.md       # foundational, in the map
  guides/           # polished topic docs, one per subject
  meetings/         # dated summaries, YYYY-MM-DD-slug.md
  capture.md        # raw, tagged notes — permanent, not an inbox
  tags.md           # generated tag → location map over the corpus
```

**A slot in `INDEX.md` signals foundational importance**, so adding one should be a deliberate, high-friction event. Growth pressure belongs in the corpus.

## Retrieval protocol — mandatory

The corpus is invisible until searched, so **something in the always-loaded layer must instruct an agent to search it.** Without this wiring an agent stops at the indexed anchor docs and concludes nothing else exists. Two edits per repo.

**1. Instruction file** (`AGENTS.md` or equivalent) — keep it to a pointer:

```
- Beyond the `docs/INDEX.md` map, search `docs/guides/`, `docs/meetings/`, and
  `docs/capture.md` by tag/keyword (see `docs/tags.md`) for anything the indexed docs don't cover.
```

**2. `docs/INDEX.md`** — the detail lives here, since it is read first. Add after the anchor-doc list:

```markdown
## Beyond this index

This index lists only the foundational docs. Additional knowledge lives in searchable
pools that are NOT listed here — search them by tag/keyword before concluding something
isn't documented:

- guides/   — polished topic docs, one per subject
- meetings/ — dated meeting summaries
- capture.md — raw, tagged notes and insights

Each section carries a **Tags:** line; tags.md is the generated tag → location map.
```

Effective retrieval order: instruction file → `INDEX.md` → indexed anchor docs → if insufficient, search the corpus by tag/keyword using `tags.md` for vocabulary.

## Document conventions

**Search header** — additive. New docs get it; backfilling existing ones is optional.

```yaml
---
description: <1–2 sentence BLUF>
keywords: [entities, tools, tasks, identifiers]
last_updated: YYYY-MM-DD
---
```

**Section tags** — a `**Tags:** foo, bar` line under each `##`/`###` in capture, meetings, and any multi-topic doc. This makes **the section, not the file, the retrieval unit**, which is what lets one file hold many unrelated topics without a bloated file-level description. Tags are free-form on write and normalized during a promotion pass.

## tags.md — generated, never hand-edited

A tag → location index over the corpus, rebuilt periodically rather than on every write. Agents can also grep `**Tags:**` lines directly, so this is a convenience and vocabulary aid.

```markdown
---
description: Generated tag → location index for this repo's searchable docs corpus. Do not hand-edit.
last_updated: YYYY-MM-DD
---

# Tags

- **ingest** — capture.md#raw-events-rename, guides/pipeline-setup.md
- **auth** — capture.md#token-rotation, meetings/2026-07-10-sync.md
```

Reference extraction — an unambiguous definition of what a rebuild must collect:

```sh
awk '
  /^#{1,6} / { h=$0; sub(/^#+ +/,"",h) }
  /^\*\*Tags:\*\*/ { t=$0; sub(/^\*\*Tags:\*\* */,"",t)
    n=split(t,a,/, */); for(i=1;i<=n;i++) printf "%s\t%s\t%s\n", a[i], FILENAME, h }
' docs/capture.md docs/meetings/*.md docs/guides/*.md 2>/dev/null | sort
```

## Anchor index drift check

`docs/INDEX.md` is hand-curated, so unlike `tags.md` it can drift from what is on disk — and the drift is **silent**, because an agent that reads the index concludes the missing doc does not exist.

The answer is verification, not removal. An index answers *"what exists that I should know about?"*; a query answers *"where is the thing I already know I want?"* Only the second can be replaced by search.

```sh
# 1. Anchor docs on disk but absent from the index
for f in docs/*.md; do
  b=$(basename "$f"); [ "$b" = "INDEX.md" ] && continue
  grep -q "$b" docs/INDEX.md || echo "NOT IN INDEX: $b"
done

# 2. Index entries pointing at files that no longer exist
rg -o -N '\(([A-Za-z0-9._/-]+\.md)\)' -r '$1' docs/INDEX.md | sort -u | while read -r ref; do
  [ -e "docs/$ref" ] || [ -e "$ref" ] || echo "NO FILE: $ref"
done
```

**Scope it to top-level `docs/*.md` only.** The corpus is deliberately absent from the index, so recursing would flag every corpus file as drift and train the reader to ignore the output.

**A hit is a question, not a defect.** Either the doc belongs in the index and was missed, or it is corpus material sitting in the anchor directory and belongs in `guides/`. Adding an entry to silence the check is the wrong reflex — a slot signals foundational importance.

Measured across three real repos on first run: two of three had drifted, one of them missing a source-of-truth document.

## Install checklist

1. Add the instruction-file pointer (§ Retrieval protocol #1).
2. Add the `## Beyond this index` section to `docs/INDEX.md` (#2).
3. Create `docs/capture.md` on first use; `docs/guides/` and `docs/meetings/` as content arrives.
4. Generate `docs/tags.md` once corpus content exists.
5. Run the anchor index drift check and resolve any hit — index it, or move it to `guides/`.
6. Verify: a fresh agent given only the instruction file and `INDEX.md` follows the protocol and retrieves a corpus item by tag/keyword.
