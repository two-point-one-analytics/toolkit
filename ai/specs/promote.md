---
description: Spec for an unbuilt `promote` skill — periodic distillation from the searchable corpus into indexed anchor docs
status: unbuilt
---

# `promote` — spec

Distil accumulated corpus material into the indexed anchor docs, periodically and with approval. **Not built.** This is the design, preserved so it does not have to be rederived.

`promote` is the only path from the searchable pool into the map. Without it, everything captured stays search-only forever and the anchor docs never learn anything.

## Behavior

An AI-assisted review pass, not a per-entry action.

- **Reads** `docs/capture.md`, `docs/meetings/`, `docs/guides/`, and **plan files**, clusters by tag, and **proposes distillations into existing anchor docs** — significant value only, existing docs only.
- **Flags graduation candidates** — a tag cluster or a guide that has crossed the threshold to justify a *new* anchor doc, presented with the underlying entries as evidence. This is the only path that adds an `INDEX.md` line, and it is deliberately high-friction.
- **Marks promoted material** (`#promoted` or `promoted: <doc>`) rather than deleting it. Preserves provenance and prevents re-review. Not everything is promoted, and nothing is removed to make room.
- **Regenerates `tags.md`** and normalizes free-form tags in the same pass.
- **Runs the anchor index drift check** (see `kb/docs-knowledge-system.md`) as part of the same sweep.
- **Always proposes before editing** canonical docs. This is the guardrail that makes an automated pass over the knowledge layer acceptable.

## Why plan files are an input

Durable knowledge accumulates inside plans — decisions with rationale, rejected approaches, constraints discovered mid-work — and a plan terminates while that knowledge does not. Nothing else extracts it, so it ends up buried in a closed artifact.

Reading plans alongside the corpus means the knowledge someone produces *while working* reaches the docs without a separate capture ritual. The plan contract already forces the writing; `promote` supplies the missing step.

## Design constraints

- **Propose, never write silently.** Every insertion into an anchor doc is reviewed.
- **Mark, never delete.** Provenance survives; a promoted entry stays where it was.
- **Evidence before graduation.** A new anchor doc requires the cluster that justifies it, shown.
- **Significant value only.** Most corpus material stays in the corpus; that is the design working, not failing.

## Open question it would settle

Whether extraction into a knowledge base is a new skill family or an evolution of this one. `promote` moves corpus → anchor within a repo; a KB extractor moves plans/notes → knowledge base globally. Same shape, same guardrails, different scope. Building one likely settles the other by precedent rather than by decision.
