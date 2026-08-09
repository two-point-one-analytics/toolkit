---
description: Spec for an unbuilt `docify` skill — dictation plus screenshots into a polished guide
status: unbuilt
---

# `docify` — spec

Author a polished topic or tutorial doc into `docs/guides/` from dictation and screenshots. **Not built.** This is the design, preserved so it does not have to be rederived.

Built to replace linear video walkthroughs, which are unsearchable and expensive to update. The target reader is an agent that greps, so a searchable document beats a recording even when the recording is easier to make.

A **skill, not a sub-agent** — it needs interactive main-thread context while the author works.

## Capture loop — deterministic, no AI

The ordering guarantee comes from cursor position, not from a model inferring where an image belongs.

1. Configure the markdown editor to copy pasted images into a per-file assets folder using relative paths.
2. Dictate a paragraph; stop.
3. Capture a screenshot to the clipboard.
4. Paste at the cursor — the editor files the image and inserts a relative link inline.
5. Resume dictating.

Because the author pauses and pastes at the cursor, images land in the right place with zero inference.

**Markup is a separate later pass** so it never interrupts dictation. Use a screenshot tool that **overwrites the original file on save** rather than exporting elsewhere — crop, arrows, callouts, numbered steps, blur. In-place saving flattens permanently, which is acceptable for finished documentation; keep raw captures in an `originals/` folder if re-editing is expected.

## Rewrite pass

1. **Read the images** for accurate captions and prose references — do not describe an image you have not looked at.
2. **Add the search header** — `description`, `keywords`, `last_updated`.
3. **Write a BLUF opening** — most important thing first, then scope.
4. **Tighten each section**, matching the repo's existing doc style. Preserve technical specifics verbatim.
5. **Preserve image placement.** Never move or drop an image. Add an italic caption line where useful, since alt text is not rendered as a visible caption in most editors.
6. **File into `docs/guides/`** and update `tags.md`.

## Constraints

- **Does not add an `INDEX.md` line.** Guides are found by search. An index entry happens only on graduation to anchor status, via `promote`.
- Do not invent facts, identifiers, or paths; flag uncertainty rather than filling it in.
- Ask before removing load-bearing content.
- Re-verify each image still matches its section after rewriting.

## Verified tool behavior

Confirmed at design time; re-verify before relying on any of it.

- Markdown editors with paste-to-folder support will file a pasted image and insert a relative link at the cursor. Frontmatter typically renders as a collapsed metadata block.
- Screenshot tools differ on the critical point: some **overwrite the original path** on save, others **export to a configured location**. Only the first is suitable for a markup pass over already-linked images.
- Most editors have no native "open image in editor" action — keep the assets folder open in a file browser and open images from there.
- Agents can read screenshots directly (PNG/JPG) but cannot ingest video; feed frames if a recording must be mined.
