# Knowledge Management

Summary: The four-category memory model, the three states of context availability, the known gaps in both, and how extraction from a human record should work.

## Memory Category Model

Summary: Four categories, cut first by whether an entry terminates and then by size or load discipline.

Context: The destinations accumulated bottom-up, one problem at a time, and were only named as a structure later. Two of the four were explicit; the always-loaded instruction layer was in use but unnamed.

Learning: The primary cut is **whether an entry terminates**.

| | Terminates — work state | Never terminates — world model |
|---|---|---|
| Larger / on-demand | **Plans** — spec, progress, next action; resumable across a cleared context | **KB and repo docs** — definitions, relations, rules, logic; read when relevant |
| Smaller / always-on | **Notes** — one open loop, deleted when acted on; not a historical record | **`AGENTS.md`** — behavioral rules loaded every turn |

The sub-split differs by row, and that asymmetry is real rather than a modeling failure: work memory divides by size, world memory divides by load discipline. Load discipline is the axis that costs context budget on every request, which is why instruction files must stay minimal while `kb/` can grow.

**Termination condition is the test for any proposed new category.** A note leaves when acted on, a plan when the work completes, a KB entry never — it gets revised. If a proposed category's entries would leave for the same reason as an existing category's, it is not a new category. Two candidates were evaluated against this test and rejected:

- **A temporal log of what happened when.** Its appeal is capturing *why* something changed, which is decision provenance rather than chronology. Calendar, email, chat, a daily journal, and git already hold the events, and a plan's Progress Log already holds causality for work worth planning — its high entry cost is exactly what keeps it noise-free. Build query tools against the existing sources instead.
- **A record of work tasks.** Keep these in the purpose-built system and query as needed. The external tool owns status; the repo owns context; neither restates the other's fields. Caveat: querying beats copying only while access is retained, so anything that must outlive that access has to be extracted deliberately beforehand.

Each additional category multiplies retrieval cost — a single question already has to hit plans, notes, KB, repo docs, and the ticketing system — which is an independent argument for keeping the count at four.

## Three States Of Context Availability

Summary: Information is in context, known-but-not-loaded, or invisible — and the third is only reachable via a pointer from the first two.

Context: Derived while designing a searchable docs corpus as a two-axis "anchor vs. corpus" model, then generalized. It refines the load-discipline axis in the category model above, which merges two mechanically different things under "on-demand."

Learning: What an agent can use is not one axis but three discrete states, distinguished by mechanism rather than by degree.

| State | Mechanism | Cost | Failure mode |
|---|---|---|---|
| **1. In context** | System prompt, `AGENTS.md` and personal instruction files — present on every turn | Token budget on every request | Bloat taxes everything, including turns that never need it |
| **2. Aware, not loaded** | An index is in context — name plus a one-line description; content is fetched on request | One line per entry | Index bloat; descriptions drifting from the content they describe |
| **3. No awareness** | Nothing points at it; found only by explicit search | Zero context | Invisible — an agent stops at what it knows about and concludes the material does not exist |

**The load-bearing rule: state 3 is only reachable via a pointer from state 1 or 2.** Content that nothing instructs an agent to search for is functionally absent, however well written. This is why a search-found pool needs an explicit retrieval protocol — a line in the always-loaded instruction file, or a "beyond this index" note in the index itself — and why omitting that wiring silently strands the entire pool.

Corollaries:

- **A slot in state 2 is the scarce resource, not disk space.** An index confers awareness at the cost of weight on the file every agent reads first, so admitting an entry should be a deliberate, high-friction event. This is what keeps an index from degrading into an unmaintainable file list.
- **Choose the state deliberately per class of content, not per document.** Foundational and frequently needed → state 2 with a curated index. High-volume and accretive → state 3 plus a retrieval protocol. Only behavioral rules that must apply on every turn belong in state 1.
- **State 3 scales without bound; states 1 and 2 do not.** Growth pressure should be absorbed by the searchable pool, with promotion into the index reserved for material that has demonstrated it is foundational.

Applications: a repo`s `docs/INDEX.md` over its documentation, and `kb/INDEX.md` over this KB.

## Known Gaps

Summary: Durable knowledge has no path out of work memory, and world-model entries fail silently when they go stale.

Context: Identified by testing concrete facts against the four categories and finding two kinds that landed nowhere. Both are open problems, not solved ones.

Learning: The system covers all four cells but is missing the **promotion edge** between the rows, and a staleness property on the bottom-right cell.

- **Promotion.** Decisions with their rationale, rejected approaches, and constraints discovered mid-work accumulate inside plans and notes. By the termination test they do not belong there — a decision's rationale does not stop being true when the plan completes — but nothing extracts them, so they end up buried in a closed artifact. **Negative knowledge** ("tried X, it failed because Y") has the same problem and no category at all: it is not a definition, relation, rule, or logic, yet it is among the highest-value things to retain, because without it the same ground gets re-litigated.
- **Staleness.** Plans carry `updated` because a stale plan mis-orders discovery. KB entries have no equivalent, so an entry that quietly went false is indistinguishable from one still true — and it does the most damage precisely because the KB is the layer everything else treats as ground truth. A live instance: this very file once routed to a directory that had been deleted earlier the same day, and nothing surfaced it.

The reverse edge needs nothing — world model to work memory is just reading the KB, already covered by `recall`.

## Extraction From A Human Record Is Non-Destructive

Summary: Where a human keeps a journal or task history, treat it as source material, not as files to migrate.

Context: A personal journal, notes app, or task tracker mixes narrative, status, reference history, and reusable lessons in the same entries. Only the last belongs in a KB.

Learning: Do not move, mark processed, or reorganize the source unless explicitly asked. Classify each useful piece by intended use — reusable lesson, historical fact, temporary follow-up, active plan update, or stable operating rule — and write it to the destination that matches, leaving the original intact. Stable operating rules are proposed as instruction-file changes rather than written silently, since those load on every turn.
