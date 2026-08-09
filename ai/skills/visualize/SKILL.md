---
name: visualize
description: This skill should be used to add visual structure to a markdown document — Mermaid diagrams, GFM tables, annotated code — or to produce those artifacts from a description when no document exists yet. Use when asked to visualize, diagram, or chart something structural, to make a dense document scannable, or to export diagrams to a shareable file. Not for plotting charts from a dataset.
argument-hint: [document path, or what to diagram]
---

# Visualize

Replace prose that is carrying structure with the form that shows it: a diagram, a table, or annotated code. Works two ways — improving an existing document, or producing artifacts from a description.

This is about **structural** visuals: architecture, flow, relationships, state, contracts — things whose shape is known from the document, not computed from data. Charts built from a dataset (series, axes, palettes, scales) are a different problem and out of scope; this skill has no plotting guidance to offer there.

## Two Modes

**Improve a document** — the argument is a path. Read it first, find what is being described in prose that would read better as structure, and convert only those parts. Preserve the author's voice, ordering, and meaning; this is not a rewrite. Leave prose that is genuinely explanatory alone.

**Produce from a description** — no source document. Generate the diagram or table directly and say where it was written.

## Where Output Goes

- Improving a document: edit it in place. Put images in `img/` beside the document and link them relatively.
- From a description: write to the path given, or the current directory, and state the path.
- **Never write into `plans/`.** That directory is governed by its own frontmatter contract (`plans/README.md`); a stray `plan.md` there is picked up by plan discovery and fails the contract.

## Choosing The Form

Only add a visual where it earns its place. A diagram that restates a sentence is worse than the sentence.

| Content | Form |
|---|---|
| Anything enumerable — files, options, fields, cases | GFM table |
| Architecture, module boundaries, data flow | `flowchart TD` / `LR` |
| Request, interaction, or async flow | `sequenceDiagram` |
| DB tables, warehouse schema, relationships | `erDiagram` |
| Status transitions, lifecycle | `stateDiagram-v2` |
| Type or model structure | `classDiagram` |
| A specific code change | annotated code block |
| UI or layout | a real image — never ASCII-art a mockup |

Common table shapes: `| path | change | why |`, `| option | tradeoff | choice |`, `| field | type | req | notes |`.

**Annotated code** — fence only the relevant or changed lines, then a `| line | note |` table beneath, or inline `# <-- note` comments. Do not paste whole files.

## Diagram Hygiene

- One idea per diagram. Split rather than cram.
- Label edges. Keep node text short.
- **Check for balanced brackets and quotes before writing** — malformed Mermaid fails silently, rendering as nothing rather than as an error.
- Fence as ` ```mermaid `.

## Viewing

Renders with no build step in any Mermaid-aware viewer: VS Code or Cursor Markdown preview (with the Mermaid extension), GitHub, GitLab, or Obsidian.

## Optional: Export A Self-Contained File

Only when asked to share with someone who has no Mermaid viewer. Markdown stays the source of truth.

- **Browserless (preferred, lighter):** render Mermaid → SVG/PNG with no Chromium.
  - Python: `pip install mmdc` then `mmdc -i <file>.md` (bundled engine, offline)
  - or Rust: `mmdr` (SVG/PNG, very fast)
  - Newer tools — verify output before relying on them.
- **Official (needs Chromium via Puppeteer — heavier on WSL):**
  - `mmdc -i <file>.md -o <file>.rendered.md` (replaces ```mermaid blocks with image refs)
  - then `pandoc <file>.rendered.md -o <file>.html --standalone --embed-resources`

State which path was used and flag the output as a generated artifact, not the source.

## Smoke Test

Prompt: `/visualize docs/architecture.md`.

Expected behavior: read the document, convert the parts carrying structure into Mermaid diagrams and GFM tables while leaving explanatory prose intact, edit the file in place, and report what changed — without touching `plans/` or exporting anything.
