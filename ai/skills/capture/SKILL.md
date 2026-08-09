---
name: capture
description: This skill should be used ONLY when the user explicitly runs /capture or clearly asks to capture a note, insight, or heads-up — it appends cleaned-up, tagged stream-of-thought notes to the current repo's docs/capture.md for future search-based retrieval by a future reader or your future self. Do NOT invoke on your own inference that a discussion, brainstorm, or exchange is worth saving; you may propose capturing but must wait for explicit consent.
argument-hint: "[note or topic to capture]"
---

# Capture

Append cleaned-up, tagged stream-of-thought notes to the current repo's `docs/capture.md` — a permanent, first-class, search-found corpus of raw knowledge. For durable cross-repo personal memory use `remember`; for ephemeral resume state use `note`.

## Invocation policy — explicit only

Run this skill **only on explicit invocation** — the user types `/capture` or clearly asks to capture something. **Never fire it from your own inference** that a discussion, brainstorm, or exchange seems worth saving. You **may proactively propose** it ("this seems worth capturing — want me to `/capture` it?") but must **wait for explicit consent** before invoking. This guardrail is what keeps back-and-forth from ever being documented without the user's say-so.

## Purpose

Every capture exists to make a **future reader's answer richer** when they later search this repo for a topic — whether that reader is a teammate, a successor, or your future self. `docs/capture.md` is raw, accretive, mixed-topic knowledge. It is **not** a changelog, not a historic record, and not an inbox to be emptied — captures are permanent and first-class. Significant material is later distilled into the anchor KB docs by `promote`; capture is where it starts.

The input is usually freeform speech-to-text — stream-of-consciousness that jumps between topics. The job is to **probe for what's missing, clean it up, tag it, and append it** as a new dated section.

> **`docify` and `promote` are not built.** They are referenced below because they define where capture sits in the pipeline — `docify` produces polished guides, `promote` distills captures into anchor docs. Their designs live in this repo's `specs/`. Until they exist, do that work by hand; do not tell the user to invoke them.

## Retrieval model

`docs/capture.md` is **found by search, not by the index** — it is deliberately NOT linked from `docs/INDEX.md`. Retrieval works because:

- Each capture is a `##` section carrying a grep-able `**Tags:**` line — the **section, not the file, is the retrieval unit**, so one file holds many mixed topics without a bloated file-level description.
- Tags are **free-form** at capture time; `promote` normalizes them later.

(The repo-wide retrieval protocol that points agents at `capture.md` — the `INDEX.md` "beyond this index" note and the `AGENTS.md` search line — is established once per repo as a separate convention step, not by this skill.)

## Scope

Use to preserve raw, repo-scoped context that helps a future reader understand *this repo*, captured quickly without polishing.

- **Use for:** gotchas, rationale ("why it's this way"), stream-of-thought insights, recent changes not yet in docs, tips, landmines, recommendations, half-formed ideas, pointers to external references — anything worth keeping but not yet a polished doc.
- **Don't use for — route elsewhere:**
  - A polished topic/tutorial doc (often with screenshots) → `docify` (writes `docs/guides/`).
  - A truly canonical fact that belongs *in* a reference doc → correct that doc (or flag it), don't bury it in capture.
  - Ephemeral open loops, reminders, next steps → `note` (personal notes file, outside the repo). Resumable multi-step work → `plan-create`.
  - A reusable cross-repo lesson → `remember` (global `kb/`).
- **Litmus:** "Does this help someone understand THIS repo, and should it travel with the repo?" → `capture`. "Is it ready to be a polished guide?" → `docify`. "Is it ephemeral / just-for-now?" → `note`. "Reusable across repos?" → `remember`. When something is both repo-specific and a reusable lesson, capture the repo detail here and extract the reusable lesson via `remember`.

## Safety / hygiene

- Write only to `<repo>/docs/capture.md` (append, or create on first use). Do **not** add an `INDEX.md` or `AGENTS.md` line — capture is search-found, not indexed.
- Never edit primary reference-doc *content*. If a capture implies a doc is now wrong, **surface it as a flagged follow-up** — don't silently edit docs beyond capture.
- Entries must be **self-contained for anyone who gets this repo**: no secrets, tokens, private keys, or external/personal absolute paths (`$HOME/...`, machine-specific paths). Keep it portable with the repo.
- Confirm the cwd is the intended git repo before writing. If outside a git repo, ask which repo should own the capture.

## Workflow

1. Confirm explicit invocation/consent (see Invocation policy). If you inferred this without the user asking, propose it instead and stop.
2. Identify the **topic/summary** (for the heading) and the teachable point. Note the type informally (gotcha / rationale / change / tip / idea) if it aids skimming.
3. **Probe before filing.** Don't file a vague capture. Ask a few targeted follow-ups to fill gaps — which specific table / model / pipeline / file it touches, why it matters (what breaks or confuses without it), what the reader should do or avoid, any linked doc / dashboard / ticket / external reference. Stop once a stranger could act on it; don't over-interrogate a clear note.
4. Locate `<repo>/docs/capture.md`. If it doesn't exist, create it from the template below (file-level search header + first section).
5. **Append a new `##` section at the end** (chronological — do not merge into an earlier section, and do not reorder existing sections). Structure:
   - `## <short topic summary>` heading
   - a date line (current date, US/Central) and a `**Tags:**` line
   - the cleaned-up content (tight prose or bullets; preserve technical specifics verbatim; don't invent identifiers/paths).
6. Choose **free-form tags** — entities, tools, tasks, and table/model names a future searcher would grep for. Don't force a controlled vocabulary; `promote` normalizes later.
7. Bump the file header's `last_updated` to the current date.
8. If the capture implies a real doc correction (e.g. a renamed identifier), report it as a flagged follow-up for the user to confirm separately — don't edit the doc.
9. Report: the section heading added, its tags, the file path, and whether the file was newly created.

## Capture File Template

When creating `<repo>/docs/capture.md` for the first time, stamp this header, then add the first section:

```markdown
---
description: Raw, tagged stream-of-thought capture notes for this repo — found by search, not linked from INDEX.
last_updated: YYYY-MM-DD
---

# Capture

Raw, accretive, mixed-topic knowledge for this repo: gotchas, rationale, changes, tips,
ideas, and pointers. Permanent and first-class — not a changelog, not an inbox to empty.
Its purpose is to enrich a future reader's answer when they SEARCH this file by tag/keyword.
Self-contained and travels with this repo; significant material is later distilled into the
anchor KB docs via `promote`.

How to read/append: chronological `##` sections, newest at the bottom. Each section carries a
`**Tags:**` line — the section, not the file, is the retrieval unit. Append via the `capture`
skill, or by hand following the same shape.

## <short topic summary>
YYYY-MM-DD
**Tags:** entity, tool, table/model, task

<cleaned-up note — tight prose or bullets; technical specifics preserved verbatim.>
```

The file header carries only `description` + `last_updated` (kept light on purpose); per-section `**Tags:**` lines do the keyword retrieval, so the file description never has to bloat.

## Smoke Test

Prompt: `/capture heads up — the raw events ingest table got renamed last week, whoever picks this up will look for the old name`.

Expected behavior: confirm it's an explicit invocation, probe for the exact old → new name and why it matters, then (creating `docs/capture.md` with its search header if absent) append a new dated `##` section with a `**Tags:**` line (e.g. `ingest, raw-events, rename`) and the cleaned note — with **no** `INDEX.md` / `AGENTS.md` wiring — and separately flag that `docs/ingest.md` / `docs/schema/raw.md` may need the identifier corrected.
