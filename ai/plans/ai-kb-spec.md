# KB System — Architecture, Amendments & Start Plan

**Status:** Draft for review
**Owner:** Curtis Robinson
**Applies to:** `kb-implementation-plan.md` (BASE), `personal-kb-delta-plan.md`
(DELTA), `team-status-workflow-plan.md` (STATUS)
**Supersedes:** `kb-amendments.md`, `kb-amendments-v2.md`

Part A governs — when a plan conflicts with it, the plan needs an edit.
Part B is keyed amendments. Part C is what to do first. Part D is deferred.

**Revision note.** A6.1–A6.5 (temporal storage, the three-layer split, gating by
consequence, ownership and deferral) and C5 are new. B1 gains the topic-note
taper; B2 drops `aliases` and `keywords`; B3 narrows; B4.2 and B5 now reference
the retained event log rather than disposable weekly slices.

Nothing is built.

---

# PART A — ARCHITECTURE

## A1. Governing Principle

> **Scope by audience. Each tier holds the context relevant to its audience,
> reads from the tier above it, and publishes deliberately to the tier below.**

```
┌─ INDIVIDUAL ─────────────────────────────────────────────┐
│  Obsidian vault      capture: meetings, research, 1:1s,  │
│      │               musings                             │
│      ▼                                                    │
│  personal-kb         private entities + private           │
│      │               assertions about team entities       │
│      │               READS team-ops (overlay, not copy)   │
└──────┼───────────────────────────────────────────────────┘
       │ promotion gate — agent proposes, human approves
       ▼
┌─ TEAM ───────────────────────────────────────────────────┐
│  team-ops            AI-agent-built processing system     │
│                                                           │
│  JIRA, GitHub,     ─┬─→ TEMPORAL   internal: status,     │
│  debriefs,          │   debriefs, roll-ups, orchestration │
│  promotions         │                                     │
│                     └─→ DURABLE    canonical entities +   │
│                         assertions                        │
└─────────────────────────────┼────────────────────────────┘
                              │ publish — Phase 2/3
                              ▼
┌─ COMPANY ────────────────────────────────────────────────┐
│  team-context        generated, read-only                 │
│    · wiki (GitHub Pages)   · AI context layer             │
│    · schema / metadata docs, glossary, metric definitions │
└──────────────────────────────────────────────────────────┘
```

## A2. Tiers

| Tier | Purpose | Audience | Authored by |
|---|---|---|---|
| **Vault** | Capture and personal thinking | You | You, in flow |
| **personal-kb** | Your understanding of company, systems, org, project context | You + your agents | Extraction, human-reviewed |
| **team-ops** | Process sources into temporal + durable; orchestrate the team | Team | Extraction + contributions |
| **team-context** | Published artifacts | **Company-wide** | Generated only |

**team-ops is a system, not a document store.** Its job is processing; documents
are outputs.

## A3. Audience Levels

| Level | Meaning |
|---|---|
| `self` | Never leaves the personal repo |
| `team` | Team reads everything; low value outside. **Not secret** — the control that matters is `self` → `team` |
| `company` | Published to team-context. Company-wide, not actually public |

## A4. Overlay, Not Copy

**personal-kb references team knowledge; it never copies it.**

Personal repo holds `local.` entities plus **private assertions whose subject is
a team-canonical entity**. The join happens at **projection time, not ingest
time** — `kb project` reads both stores and renders team definition + your
private assertions together.

**This dissolves the circular-flow problem.** You never ingest team content, so
there is nothing to re-consume and no source-tagging mechanism is needed. The
loop is created by copying.

**One canonical entity registry, in team-ops.** Independent stores, shared
vocabulary.

## A5. Contribution Gradient

| Path | Who | Mechanism |
|---|---|---|
| Personal repo promotion | Anyone running the template | Agent proposes → human approves |
| Debrief `kb_candidates` | Every member, weekly | STATUS interview |
| Direct tracker extraction | Automated | team-ops, post-P3 |

Teammates take the personal repo as a template with a bootstrap you provide.

**The system works if nobody adopts the template** — debriefs plus tracker
extraction carry it. Adoption is an upgrade, not a prerequisite.

**Your contribution weight decays by design.** Anything that makes you a
permanent router of other people's knowledge violates this and should be
rejected on those grounds alone.

## A6. What / Why Division

**Code holds the *what*. The KB holds the *why*.**

Three layers, not two:

| Layer | Source of truth | dbt flows | Raw SQL flows |
|---|---|---|---|
| **Structural** — columns, types, nullability | `information_schema` | scraped | scraped |
| **Declarative description** — what a column means | `schema.yml` | native | **sidecar (below)** |
| **Semantic** — why, exclusions, authority, gotchas | KB | ✓ | ✓ |

**Raw SQL sidecar.** Where flows are SQL scripts rather than dbt models, column
and table descriptions live beside the script, not in the KB:

```
scripts/revenue_daily.sql
scripts/revenue_daily.meta.yml     # table + column descriptions, owner, grain
```

Versioned with the script, reviewed in the same PR, CI-validated against the
scraped schema. A description that lives away from the code producing it drifts
the moment the script changes — which is exactly what this division exists to
prevent.

**Authority when they disagree:** SQL is authoritative for computation, the KB
for intent. A disagreement means the code does not do what the team believes it
does. That is the most valuable output this system can produce, not a nit.

## A6.1 Durable and Temporal Use Different Storage [NEW]

Two knowledge shapes, two techniques. The split follows from **mutability**, not
format preference.

| | Durable knowledge | Temporal |
|---|---|---|
| Store | YAML entities + JSONL assertions | JSONL event log |
| Keyed by | Frozen entity IDs | Event ID + timestamp |
| Mutability | **Revisable** — supersede, retract, merge | **Immutable** — week 31 never changes |
| Queried by | Entity, relationship, traversal | Date range, entity, person, project |
| Needs | Identity resolution, dedup, conflict detection | None of it — a merged PR is a merged PR |
| Index | SQLite, from day one | File scan; SQLite when scans get slow |

Applying entity machinery to temporal data would be actively wrong: supersession
and conflict detection built for records that cannot conflict, and the loss of
the immutability that makes "what was true in Q2" answerable.

### A6.2 Temporal is three layers, not one

Earlier drafts conflated the event log with the weekly debrief. They are
different layers.

| Layer | Shape | Person-partitioned | Mutability |
|---|---|---|---|
| **Event log** | Append-only, continuous, entity-referenced | **No** | Immutable |
| **Weekly slice** | Derived: date range × person | Yes | Disposable, regenerable |
| **Debrief** | MD + frontmatter, human-authored | Yes | Immutable once `complete` |

**The event log is the durable temporal record.** Continuous ingest from JIRA,
GitHub, and other sources: system changes, decisions, actions taken, external
events. Arbitrary timestamps. Some events have no clear owner; some touch several
people; some arrive midweek. A per-person weekly bucket cannot hold any of that.

**The debrief is a projection over a slice of the log, plus human additions** —
which it always was; the layer underneath was simply never named.

**Event record shape:**

```json
{"id":"gh-pr-1234","source":"github","occurred":"2026-08-07","actor":"<handle>",
 "title":"<one line>","entities":["metric.nrr","system.trino"],
 "owner":"curtis-robinson","owner_source":"inferred","url":"<link>"}
```

`entities` is the field that earns the log its keep — it powers the roll-up, the
reverse lookup (which PRs changed this system doc), and B5's drift check. Get it
right; everything else is close to free.

### A6.3 Events Are Retained [OVERRIDES STATUS §6]

STATUS §6 treats `extracts/` as a regenerable intermediate, keeping only the
human-reviewed debrief. Reverse that: **the event log is retained
indefinitely.**

Three reasons:

1. "Regenerable" is not reliably true — API history ages out (B4.2)
2. BASE §11.8 makes re-extraction the escape hatch that keeps prompt and schema
   evolution safe rather than one-way. Discarding events breaks that for
   tracker-derived knowledge specifically
3. Unattributed events have nowhere else to live and would simply be dropped

Cost is an append-only JSONL file that is mostly never read directly. Cheap at
this volume.

## A6.4 Gate by Consequence, Not by Category [NEW]

| What | Gate | Why |
|---|---|---|
| Tracker events — PR merged, ticket moved | **None.** Auto-ingest | Mechanical, verifiable, low consequence |
| Work with no tool trail — meetings, ad-hoc analysis, unblocking | Human **supply** | Unscrapeable by definition |
| Sizing, surfaced items, audience routing | Human **judgment** | No mechanical substitute |
| Durable claims (`kb_candidates`) | Human **approval** | Highest consequence |

The interview's job is to supply what cannot be scraped, judge what needs
judgment, and approve what must not auto-land. Confirming that a PR you merged
was in fact merged is none of the three.

**Payoff:** ungated event ingest shrinks STATUS's "review → correct" stage, which
directly serves the under-ten-minute budget that STATUS §11.1 calls the adoption
cliff.

**Cost, stated plainly:** STATUS §7 uses `origin: extracted | added` to measure
extraction quality — if `added` dominates for someone, Part 1 is not serving
them. Dropping mandatory confirmation weakens that signal. Show extracted items
without requiring per-item confirmation and most of it is recoverable, but this
is a trade, not a free win.

## A6.5 Ownership and Deferral [NEW]

**Owner resolution is a cascade, and its source is recorded:**

```yaml
owner: curtis-robinson
owner_source: default      # explicit | inferred | default
```

explicit assignment → PR/commit author → ticket assignee → entity owner →
**default: Curtis**.

`owner_source` is the field that matters. It makes "how often does this fall to
me" a query rather than a feeling, which is the only way A5's decay property
stays checkable. If `default` fires on most items a year from now, that is
visible rather than merely heavy.

**Scope.** Default-approver covers **unclaimed durable claims and deferred
debrief items only** — never the event log. An unattributed PR merge needs no
approval; it is mechanical and verifiable. Applying the rule to events means
approving thousands of rows a week, and the rule collapses.

**Deferral.** `defer` is a valid debrief response; the item routes to Curtis and
sits in a backlog he owns and may delegate. Delegation mechanics are deliberately
unspecified for now.

Deferral needs friction, or it becomes the default answer — it is the
lowest-effort response under a ten-minute budget. DELTA §11.P4 names this exact
failure mode for the personal KB: *not disclosure, accumulation*. Same shape,
same mitigations:

- **Reason code** — `not-mine` / `needs-context` / `no-time`. Cheap to answer,
  and it separates routing failures from capacity problems, which have different
  fixes
- **Backlog size visible** in the weekly roll-up, alongside missing debriefs
- **A threshold that means something** — past N, reduce inflow rather than let
  the pile grow, mirroring DELTA's rule for daily review slipping

**No new surface.** The deferral backlog is a `deferred` bucket in the existing
`kb review` flow. BASE §4 is explicit that git plus the CLI is the review queue
and building another is the largest available waste of effort.

**Acknowledged exception to A5.** Default-approver-is-Curtis makes you the
router, the pattern A5 says to reject on sight. It is correct at one contributor,
and delegation is the escape valve — but escape valves needing an unspecified
future process tend not to get used. Recording `owner` and `owner_source` as real
fields from day one is the price of the exception: it converts "we will delegate
later" from an intention into a one-line change.

## A7. Publish Boundary Is a Repo Boundary

Git access control does not work at directory granularity — anyone who can clone
gets everything, including history. `team-ops` stays internal; `team-context`
holds generated artifacts with no hand-editors by construction. Likely a repo
plus GitHub Pages wiki; exact shape confirmed at Phase 2/3 and **not a dependency
for starting**.

Every generated file carries `<!-- GENERATED — do not edit. Source: team-ops -->`
and CI fails if regeneration changes the tree.

## A8. Non-Negotiables

1. Plain text files in git are the source of truth; SQLite is derived,
   gitignored, rebuilt, never hand-edited
2. Git diff is the review gate — personal repo: agent/human interaction on main.
   team-ops: pull request
3. **Agents propose; humans dispose.** Never an agent-decided destination
4. Controlled predicate vocabulary; adding a predicate requires review
5. Retract, never delete — tombstones prevent resurrection on re-extraction
6. **No agent writes to the vault, ever.** The single exception is the B3 hygiene
   pass — a one-time migration reviewed as a git diff, not a workflow
7. Generated artifacts are never hand-edited
8. Sizing never compared across people; upward summaries never auto-sent

---

# PART B — AMENDMENTS

## B1. The Vault Is Two Things [NEW]

BASE and DELTA treat the vault as a capture funnel — messy journals, low signal,
aggressive threshold. Accurate for journals, wrong for the rest.

The vault is **also a curated knowledge surface**: atomic topic notes, dense
interlinking, deliberate structure. Authored, maintained, read directly.

**Non-duplication rule:** never hand-author an entity YAML for something that
exists as a vault note — extract it. Never copy a projection back into the vault.
Prose and triples are not the same artifact.

**Primary surface:** through P2, the vault — the KB has nothing to read yet.
After P2 it splits by consumer: vault for you, `projections/` for agents.

**The taper.** Once projections exist, **stop hand-authoring curated topic
notes.** Projections *are* the topic notes — generated from entities and
assertions, carrying typed edges instead of untyped `[[links]]`, regenerated
rather than maintained. The vault reverts to what it is uniquely good at:
journals, transcripts, in-flow capture, raw thinking.

This is not a downgrade. It is what makes the non-duplication rule hold
structurally instead of being something you have to remember.

Expect a transition. Through P1–P2 your topic notes are the only knowledge store
and you will keep writing them. After P2, taper. **If you are maintaining both a
topic note and an entity for the same thing, the taper is overdue.**

**Diagnostic:** if you find yourself reading `projections/` and wanting to edit
them, extraction is not earning its place. Shrink extraction scope; never merge
tiers or relax DELTA §11.P7.

**No sensitivity field in the vault.** The vault crosses no boundary.
Classification happens at extraction (DELTA §4.1).

## B2. Vault Frontmatter [EXTENDS DELTA §6]

Justification is **extraction routing**. Three fields, all **hand-set** — no
agent-generated frontmatter, per A8.6.

```yaml
type: journal        # journal | transcript | topic | reference
updated: 2026-08-09
extract: true        # optional, defaults true
```

| Field | What it buys |
|---|---|
| `type` | Note shape, so extraction thresholds can differ (B2.3) |
| `updated` | Incremental ingest without hashing every file |
| `extract` | Hard opt-out (B2.2) |

**Dropped from earlier drafts, and why:**

- **`aliases`** — moved to the KB, where `entity_aliases` already exists (BASE
  §7.5). Vault aliases were only ever a seed for that table, and generating them
  meant an agent writing to the vault, contradicting A8.6.
- **`keywords`** — dropped entirely. They existed to boost Omnisearch ranking on
  curated topic notes, and topic notes are being tapered (B1). Journal and
  transcript search does not need them.

Quote all string scalars. BASE §11.2 applies: `type: no` silently coerces to
`false` under YAML 1.1.

### B2.2 `extract: false` [NEW — no counterpart in any plan]

DELTA classifies sensitivity *after* extraction. Correct for business content,
wrong for the rest of the vault. Career thinking, assessments of colleagues,
client frustrations should not become assertions **at all** — once a fact carries
a `promotion_candidate` field it is one slip from the boundary.

1. `extract: false` excludes the note from `kb ingest` entirely — no manifest
   entry, no hash, no source record
2. Directory-level globs in `config/vault.yml`
3. **Enforced in code, not prompt** — same standard as DELTA §11.P1
4. The agent cannot set `extract: true`; it may propose, only a human edits
5. `kb ingest --dry-run` reports the excluded set, so over-exclusion is visible

### B2.3 Per-type thresholds [EXTENDS BASE §11.11]

| `type` | Threshold |
|---|---|
| `journal` | Aggressive — BASE §11.11 as written |
| `topic` | Permissive — curated; the author already filtered |
| `transcript` | Aggressive + speaker-aware |
| `reference` | Permissive, low frequency; re-extract on hash change only |

Calibrate in P1. If `type` does not discriminate cleanly, collapse to one
threshold.

## B3. Vault Hygiene Pass [EXTENDS DELTA §8 — precondition to P1]

**Backfill frontmatter before P1 ingestion begins.** DELTA §3 references vault
notes by path and sha256, so backfilling after the manifest exists causes mass
re-extraction, and **renaming a note breaks its manifest reference** — the
vault-side analog of BASE §11.4, written down nowhere.

Sequence: git init the vault → agent pass proposing `type` values and flagging
`extract: false` candidates → review as diff → settle naming and do renames
**now** → first `kb ingest`.

**This is the only time an agent touches the vault** (A8.6). It is a one-time
migration, human-merged, not a recurring workflow. No alias or keyword generation
— those fields no longer exist (B2).

After P1, treat vault paths as stable. Build `kb reconcile-manifest` in P4 only
if renames prove unavoidable; never hand-edit the manifest.

## B4. Tracker Extraction Is Team-Side [RESOLVES BASE §9 Phase 4, DELTA §10]

Tracker-derived facts about systems and datasets are **inherently team
knowledge**. Routing them through the personal repo would make you a permanent
router, violating A5.

| Output | Destination | Owner |
|---|---|---|
| **Temporal** — what happened, when, who | **Event log** (A6.2), retained (A6.3) → weekly slices → debriefs → roll-ups | STATUS Part 1 |
| **Durable** — entity and relationship facts | team-ops entities + assertions | team-ops, post-P3 |

**STATUS Part 1 is the only system calling the JIRA and GitHub APIs.** One token
setup, one rate-limit budget, one cursor. Durable extraction consumes
`extracts/` as a file source.

**Reconciling the v1 lock:** BASE §5 and §12 forbid team-side extraction. That
lock was justified by *sequencing* — prove extraction in the personal repo first
— not by principle. Post-P3, team-side extraction is permitted, gated by PR.

### B4.1 Consequent edits

| Location | Change |
|---|---|
| BASE §9 Phase 4 item 1 (vault ingestion) | Strike — personal repo owns it |
| BASE §9 Phase 4 item 2 (GitHub/JIRA) | Rewrite per B4 |
| BASE §9 Phase 4 heading | Label **post-v1**; as written it contradicts §5 and §12 |
| BASE §6 — `kb/state/watermarks.yml` | Remove. Cursors live in STATUS `state/watermarks.yml` |
| BASE §13 Open Q2 | Append pointer to B4 so the durable half is not lost with the temporal |
| DELTA §10 bullet 4 | Correct as written. No change |

### B4.2 The event log is the contract [OVERRIDES STATUS §6]

STATUS §6 describes `extracts/` as a regenerable intermediate. Per A6.2–A6.3
that role splits:

| Artifact | Status |
|---|---|
| **Event log** (JSONL, append-only) | **Retained indefinitely.** The published interface — version its schema, export JSON Schema alongside `debrief.schema.json`, treat shape changes as breaking |
| **Weekly slices** (`extracts/<week>/<person>.json`) | Genuinely disposable — derived from the log by date range and person, regenerable at any time without API access |

The distinction earlier drafts missed: slices are regenerable **because the log
is retained**. Without the log, regeneration depends on API history that ages
out.

### B4.3 Ownership on events [EXTENDS A6.5]

Every event carries `owner` and `owner_source` (A6.5), resolved by cascade at
ingest. Unclaimed events default to Curtis — but **carry no approval
requirement**, per A6.4. They land in the log, feed the roll-up and durable
extraction, and surface for human attention only through the B5 drift report.

## B5. Activity-Without-Knowledge Drift [EXTENDS BASE §9 Phase 4 item 4]

`kb drift` catches stale entities. It misses the more dangerous case:

> **An entity has recent tracker activity but no new assertions.**

The system changed and the knowledge did not — documented behavior is now *wrong*
rather than old, and reads as current.

```
entities referenced in the EVENT LOG within N weeks
  whose newest active assertion predates that activity
```

Report only, never autonomous rewriting.

**Correction from earlier drafts:** this check previously read `extracts/`, which
are person-partitioned and explicitly disposable. It must read the **event log**
(A6.2) — a change to a system is not attached to a person in any way this check
can use, and the retention guarantee in A6.3 is what makes the comparison
possible at all.

## B6. Multi-Contributor Consequences [NEW]

- **Entity ID ownership.** BASE §11.4 forbids renames because IDs are externally
  referenced. With N contributors this needs a named owner and a documented merge
  path, not just a prohibition.
- **Dedup becomes load-bearing.** Two people will promote the same fact worded
  differently. `kb resolve` moves from convenience to required, and its thresholds
  cannot be calibrated with one operator.

Neither blocks v1. Both belong on the Phase 2 onboarding checklist (BASE §10).

---

# PART C — START PLAN

## C1. Two Parallel Tracks

**Correction to earlier sequencing advice:** personal and team are strictly
sequential *for the KB*. The **status workflow is not** — STATUS Phases 0–1
depend on nothing from the KB and can start immediately.

```
Track A — personal-kb          Track B — team-ops (STATUS)
─────────────────────          ──────────────────────────
vault git init                 repo scaffold
vault hygiene pass (B3)        config: team / form / projects
   ▼                           debrief.schema.json
P0 scaffold                       ▼
   ▼                           Phase 1: interview skill
P1 journal loop  ── 2 weeks ─→    run on yourself 2×
   ▼                              ▼
P2 projections                 Phase 2: JIRA/GitHub extraction
   ▼                              ▼
P3 promotion gate ──────────→  Phase 3: roll-ups
                                  │
                               team KB durable ← gated on P3
```

**Start Track B first.** Three reasons: it removes your least-desirable recurring
task immediately (STATUS §3.2); it has no dependencies; and `config/projects.yml`
is shared vocabulary with the KB, so building it seeds the entity registry Track
A will need. Track A's vault hygiene pass can run in parallel — it is manual,
reviewable, and blocks nothing.

## C2. Track B — Decisions Needed Before Code

| # | Decision | Notes |
|---|---|---|
| B-1 | Repo location and name | `team-ops/resource-management/` per STATUS §5, or standalone |
| B-2 | `team.yml` roster | slug, name, JIRA account, GH handle, domains — per person |
| B-3 | `projects.yml` vocabulary | **Also the KB's entity seed.** Project slugs become `project.*` entities. Include `proj.adhoc` |
| B-4 | `form.yml` question set v1 | Must fit under 10 min (STATUS §5, §11.1). Cut before exceeding |
| B-5 | Week-ending convention + timezone | STATUS §11.11. Pick Friday or Sunday and be explicit |
| B-6 | `type` taxonomy confirmation | STATUS §7: delivery / support / analysis / enablement / admin / learning. Confirm `enablement` and `learning` are prompted explicitly |
| B-7 | Manual drop mechanism | STATUS §6 lists `manual/` as TBD |
| B-8 | Org-level token provisioning | STATUS §11.8 — lead-run, read-only. Start the access request now; it has lead time |

**Phase 0 acceptance:** `debrief.schema.json` exists, `validate_debrief.py` passes
on a hand-written fixture and rejects a malformed one.

**Phase 1 acceptance:** you complete the interview on yourself twice, under ten
minutes both times. If it runs long, cut questions — the budget is the hard
constraint, not the question set.

## C3. Track A — Decisions Needed Before Code

### Vault hygiene (do first, manual)

| # | Decision |
|---|---|
| A-1 | **Is the vault a git repo?** If not, `git init` is step zero |
| A-2 | Final `type` value list — start with `journal` / `transcript` / `topic` / `reference` |
| A-3 | Which directories get `extract: false` — personal musings, career, 1:1 notes |
| A-4 | Note-naming convention; do all renames now |
| A-5 | Journal glob and date format for `config/vault.yml` |

### P0 scaffold

| # | Decision | Notes |
|---|---|---|
| A-6 | Python 3.12 + `uv`; deps per BASE §14 | pydantic, typer, ruamel.yaml, jinja2, numpy, orjson |
| A-7 | Repo location — sibling of team-ops | DELTA §3 assumes adjacency for entity reads |
| A-8 | Entity types starter set | BASE §7.4: metric, source, lob, project, team, person, system, process, concept, client, meeting |
| A-9 | Auto-`restricted` trigger list | DELTA §4.1: compensation, performance, personnel, health, legal, interpersonal conflict |
| A-10 | Embeddings in P1? | **Recommend deferring.** Alias matching plus FTS may carry P1. Adds a model download and a calibration burden before you know you need it |

**P0 acceptance (DELTA §8):** fixtures validate; a fixture missing `sensitivity`
is rejected; a fixture with `aliases: [no, NRR]` is caught by yamllint's truthy
rule; team entities resolve from `../team-ops`; a note with `extract: false` is
excluded from `kb ingest --dry-run`.

### P1 — the real gate

Run daily for two weeks on real journals. **If daily review exceeds ten minutes,
stop and fix before proceeding** (DELTA §8 P1). Instrument DELTA §1 questions
1–3 with actual numbers.

### C3.1 Deferring Embeddings [DEVIATES FROM BASE §9 Phase 2]

BASE assumes embeddings from Phase 2. A-10 defers them. This subsection records
why that is safe and when to reverse it — without it, a future reader sees a
deviation from the plan with no rationale.

**What is deferred.** Embeddings serve one job in this system: candidate
generation inside `kb resolve` (BASE §4). Not retrieval. The resolve cascade is
exact `content_hash` → FTS5 over `claim` → cosine. Deferring removes only the
third stage; the first two still run and still bucket.

**Why P1 does not need it.**

- B2 `aliases` in vault frontmatter hand-feeds `entity_aliases`, which is the
  resolver's weakest step. Embedding-based alias matching is largely redundant
  with it.
- BASE §9 Phase 2 marks its thresholds placeholders to be calibrated in Phase 3
  against real volume. P1 has no volume, so cosine would run uncalibrated and
  produce noise that cannot be interpreted.
- P1's purpose is the ten-minute review gate. If review runs long with cosine in
  the loop, the cause is ambiguous between extraction quality, resolver noise,
  and bad thresholds. Deferring isolates the variable being measured.
- `sentence-transformers` pulls torch (~2 GB) plus a model download and an
  embeddings cache, during the phase where the loop should be shortest.

**The cost of deferring.** P1 includes the registry bootstrap — batch extraction
across many journal files at once. That is where near-duplicate entity detection
matters most: fifty journals yielding `Net Revenue Retention`, `NRR`, and
`net rev retention` as three entities. Duplicate entities are expensive later
(`kb merge` plus repointed assertions).

Accepted because the bootstrap is agent-interactive and human-reviewed, run once.
At ~40 proposed entities, reading the list catches those three more reliably than
any cosine threshold. At ~400 it does not — which gives the trigger below.

**Trigger — add embeddings when any of these is true:**

1. Registry review stops working by eye — entity count outgrows a single
   readable list at bootstrap or drift review
2. P4 sources land (sent email, repo documentation) and multiply inflow
3. `kb drift` near-duplicate entity pairs become a recurring finding rather than
   an occasional one
4. Any second contributor begins promoting (B6 — dedup becomes load-bearing and
   cannot be calibrated with one operator)

**Why reversal is cheap.** The `embeddings` table already exists in BASE §7.5,
keyed by `(content_hash, model)`. Turning embeddings on is a **backfill, not a
migration**:

- No change to `entities` or `assertions`; source files are untouched
- No re-extraction
- Backfill iterates existing assertions, embeds, inserts — one resumable pass
- Changing models later does not invalidate anything; new rows land under a new
  model name and coexist with old ones

Work required at reversal: add the dependency, write `embed.py`, add the cosine
stage to the resolve cascade, run the backfill, then calibrate per BASE §9
Phase 2.

**Carry forward to calibration:** consider separate thresholds for claim dedup
versus entity resolution. BASE runs one set across both. A bad entity merge costs
more to undo than a missed duplicate assertion, so entity matching likely wants
the more conservative value. A Phase 3 calibration finding, not a decision now.

## C4. First Commands

```bash
# Track B
mkdir -p team-ops/resource-management && cd $_
# config/team.yml, config/projects.yml, config/form.yml
# schemas/debrief.schema.json, scripts/validate_debrief.py

# Track A — vault, manual
cd <vault> && git init && git add -A && git commit -m "baseline"
# then the frontmatter backfill pass, reviewed as a diff

# Track A — repo, after hygiene
uv init personal-kb && cd personal-kb
uv add pydantic typer ruamel.yaml jinja2 numpy orjson
uv add --dev ruff yamllint check-jsonschema pre-commit
pre-commit install
```

---
## C5. Reading the Graph Without an Agent [NEW]

YAML plus SQLite does not mean every question goes through an AI. Four access
paths, in the order I would reach for them:

**1. SQL against `kb.sqlite`.** The strongest option and the easiest to
overlook. DataGrip is already open; point it at the index and the graph is four
tables — `entities`, `edges`, `assertions`, `assertions_fts`. Recursive CTEs for
traversal, `edges` joined to `entities` for a neighborhood, FTS5 for text. No
agent, no projection lag, arbitrary questions. For a data engineer this beats any
generated document.

**2. `projections/` — generated markdown.** Already specified in BASE §9
Phase 3: Jinja2 templates rendering from index queries — metric glossary,
per-entity pages, per-domain briefs. `kb project` regenerates all of it; every
file carries `<!-- GENERATED — do not edit. Source: kb/ -->`. Byproduct, not
source of truth, regenerated on a schedule.

**3. The YAML files.** One file per entity, prose definition inline, curated
relationships as a list. Grep-able, readable, diffable. Fine for spot lookups.

**4. A second, read-only Obsidian vault pointed at `projections/`.** A partial
departure from DELTA §11.P7, taken deliberately. The rule keeps projections out
of the *capture* vault because Obsidian makes casual editing frictionless. A
separate vault preserves that intent while giving you graph view, backlinks, and
search over generated entity pages with typed edges.

It reduces the edit temptation rather than eliminating it. The saving grace:
`kb project` overwrites, so any edit is silently lost — which teaches the habit
quickly. Adopt knowing that, or use VS Code's markdown preview instead.

**Consequence for earlier vault-search work:** evaluating Omnisearch's API versus
a custom index versus an in-vault RAG plugin was framed around the vault holding
your knowledge. Post-taper it will not. That question now applies only to
searching journals and transcripts, where plain text search is adequate. KB
retrieval is an entirely separate path.

# PART D — DEFERRED

Noted, not scheduled. All sit at the end of the chain.

| Item | Blocked on | Note |
|---|---|---|
| **team-context publish target** | Phase 2/3 | Repo + GitHub Pages assumed. If the destination is Confluence or SharePoint, publish becomes an API integration with auth, idempotency, and conflict handling — a project, not a build step |
| **Schema scrape jobs** | Team KB content | Post-deploy scrape reflects **deployed** state. A PR-time check must parse the changed SQL, not query the warehouse, or it always compares against pre-merge state and appears to pass |
| **Structural drift check** | Scrape jobs | Deterministic diff, no agent. Can block a merge |
| **Semantic conflict hook** | Team KB content | **Advisory only, never blocks.** Never writes to the KB — an agent filing assertions from PR context bypasses A8.3. Comment must name the specific assertion it believes is contradicted, with a link |
| **Raw SQL `.meta.yml` sidecars** | Scrape jobs | A6 |
| **Coverage / sole-expert map** | ~1 quarter of debrief history | STATUS Phase 4 |

**Worth revisiting later:** the semantic conflict hook is a stronger answer to
BASE §13 Open Q4 ("first load-bearing use case") than the coverage map — it reads
from the KB on every PR, is visible to the whole team, and attaches to work
people already do. The coverage map needs a quarter of history before it says
anything.

Cheap early test: run it against your own PRs from the personal repo during P2.
If your assertions are not specific enough to be checkable, that is worth
learning in P2 rather than P4.

---

# PART E — OPEN ITEMS

1. **Is the vault under git?** (A-1) Blocks B3. Also raises DELTA §11.P5's policy
   question — a git repo of work-derived notes on a corporate device — one layer
   earlier than expected
2. **Does `type` discriminate cleanly on real notes?** If most are ambiguous,
   per-type thresholds do not earn their complexity
3. **Should `topic` notes extract at all?** A curated atomic note is already close
   to an entity record; extraction may produce assertions that restate notes you
   wrote deliberately. Journals and transcripts are unambiguously raw inflow.
   Measure in P1
4. **Who owns the canonical registry** once contributors exceed one (B6)
5. **Scaling factors for `size`** — STATUS §7 marks TBD; not needed until Phase 3
