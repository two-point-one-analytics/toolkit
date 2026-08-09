# Agent Skills

A working set of skills, sub-agents, and instructions for **GitHub Copilot CLI** — built around resumable multi-step work, transient open loops, and keeping durable knowledge somewhere an agent will actually find it.

Clone it, copy what you want into `~/.copilot/`, set two paths, and it works.

## Contents

| Directory | Contents |
|---|---|
| `skills/` | 17 skills — planning, notes, memory, documentation capture, code quality, and git |
| `agents/` | 6 read-only sub-agents for delegating token-heavy work off the main thread |
| `instructions/` | A personal instruction file — communication style, tool-use defaults, editing and git conventions |
| `kb/` | The reasoning behind all of it: memory model, context-availability states, the searchable-docs system, portability and style conventions |
| `plans/` | The plan contract, plus a bootstrap plan for standing up a knowledge system over an existing corpus |
| `specs/` | Designs for two skills that are **not built** — `docify` and `promote` — preserved so they need not be rederived |

## Bootstrapping from scratch

If you are setting up a new machine, the order is:

1. **Install** — the commands below. Delivers a working toolbox.
2. **Set the two paths** and test `disable-model-invocation` (see *After installing*).
3. **Read `plans/durable-knowledge-management.md`** — the bootstrap plan for turning an existing body of notes and documentation into retrievable context. Its first step is deliberately *characterize the corpus before designing for it*, because a structure designed for an imagined corpus rarely fits the real one.

The install delivers a toolbox; the plan delivers a library. Finishing the first and stopping leaves you with working skills and nothing useful to point them at.

### Skills

**Planning** — six skills for one moment each, so a cleared context costs nothing.

| Skill | Purpose |
|---|---|
| `plan-create` | Create a plan file; scaffolds the contract into a repo on first use |
| `plan-retrieve` | Find a plan and report where it stands. Read-only |
| `plan-interview` | Resolve open questions until a plan is execution-ready |
| `plan-step` | Execute one step at a time, with approval gates |
| `plan-run` | Execute autonomously, stopping only for blockers |
| `plan-checkpoint` | Audit a plan for consistency before context is lost |

A plan file holds the spec, the decisions behind it, and the progress — so the next session re-anchors from the file rather than from conversation history. Conventions live **in the target repo**, not in the skill: `plan-create` scaffolds `plans/README.md` and `plans/plan-template.md` from its `seed/` on first use, so a repo carrying those plus the skills is self-contained and works for someone with none of your config.

Discovery is a frontmatter query rather than an index file, on the principle that any index is a cache needing invalidation. The three exclusion guards in those queries are load-bearing — dropping one causes silent misresolution rather than an error.

**Notes** — `note`, `note-recap`, `note-review`. One file holds every open loop across every repo, dated and tagged. Entries are deleted when acted on; it is not a historical log. Review surfaces anything older than 30 days without removing it.

**Memory** — `remember`, `recall`. Read and write a global KB of durable cross-repo knowledge, routing repo-specific facts to the repo instead.

**Documentation** — `capture` writes raw tagged notes to a repo's `docs/capture.md`, found by search rather than by index.

**Craft** — `commit` (verb-first messages, explicit staging, no secrets), `polish` (readability without behavior change), `pair-programmer` (block-by-block work with the user in control), `visualize` (Mermaid diagrams and tables), `skill-install` (vendor skills into a repo so it stands alone).

### Sub-agents

`analyst`, `deep-think`, `explore`, `qa`, `research`, `review` — all restricted to read and search tools, so they investigate and report without editing. The point is context economy: delegate token-heavy work and get back a compact result.

`analyst` expects repo-local SQL runners to exist. The other five are generic.

## Install

```sh
mkdir -p ~/.copilot/skills ~/.copilot/agents
cp -R skills/* ~/.copilot/skills/

# agents use the .agent.md extension
for f in agents/*.md; do
  cp "$f" ~/.copilot/agents/"$(basename "$f" .md)".agent.md
done

cp instructions/personal.md ~/.copilot/copilot-instructions.md
```

Then `/skills reload` in an active session, or `copilot skill list` from the terminal, to confirm registration.

Personal skills are also read from `~/.agents/skills/` if you prefer that location. Repository-scoped equivalents are `.github/skills/` and `.github/agents/`, with `AGENTS.md` for repo instructions.

### Invoking a skill

Two paths, and the docs are specific about the second:

- **Automatic** — "Copilot will decide when to use your skills based on your prompt and the skill's description."
- **Explicit** — "include the skill name in your prompt, preceded by a forward slash," as in `Use the /plan-step skill to continue the migration plan.`

The documented explicit form is a **slash name inside a sentence**, not a bare command line. These docs write `/plan-step` and `/capture` as shorthand for the skill; a bare `/plan-step` alone is undocumented and may or may not be accepted. If one does not fire, phrase it as a sentence.

`/skills list`, `/skills info`, `/skills add`, `/skills reload`, and `/skills remove` are management subcommands, not a way to run a skill.

## After installing

### 1. Set the two paths

Paths outside a repo are written `$HOME/...`, never bare `~` — `~` stays literal inside double quotes while `$HOME` expands, and quoting happens as soon as a path contains a space. Find them:

```sh
rg -n '\$HOME/' skills/ agents/ instructions/
```

Two need a decision:

- **`$HOME/notes/ai-notes.md`** — where the `note` family keeps open loops. Appears across three skills; change them together.
- **`$HOME/kb/`** — the global KB that `remember`, `recall`, and `polish` read and write. Also referenced in `instructions/personal.md`.

> **`kb/` here and `$HOME/kb/` are different things.** This repo's `kb/` is documentation *about* the system — read it once, it installs nowhere. `$HOME/kb/` is your live knowledge base, which starts empty and accumulates as `remember` writes to it. Seeding it from this repo is reasonable if you want the conventions available at runtime:
>
> ```sh
> mkdir -p ~/kb && cp kb/*.md ~/kb/
> ```
>
> `polish` looks for language convention docs there by name — `sql-conventions.md`, `javascript-conventions.md` — and skips any that do not exist.

Home-anchoring makes a path portable across *your* machines, never across people — so a repo shipped to a collaborator still needs these repointed or removed.

### 2. Test `disable-model-invocation` before trusting autonomous execution

`plan-run`, `plan-step`, `plan-interview`, and `plan-checkpoint` carry `disable-model-invocation: true`. It is what stops the model from firing them on its own inference rather than on explicit invocation — which matters most for `plan-run`, since that one executes a whole plan without pausing.

Copilot documents this key for **agents**, not for **skills**. The concept exists in the product; whether it is honored at skill scope is untested.

**Test it first:** describe multi-step plan work in conversation and confirm `plan-run` does not trigger by itself. If it does, treat the plan executors as manual-only until you have another guard.

### 3. Know what the frontmatter does and does not carry

Copilot documents `name`, `description`, `license`, and `allowed-tools` for skills. These skills also use `argument-hint`, which is undocumented and cosmetic — it will be ignored harmlessly if unsupported.

Agents use canonical tool names — `read`, `search`, `execute`, `web`, `edit`, `agent`, `todo`. All six here are restricted to non-editing tools deliberately; if you add `edit`, you change what the agent is.

## Conventions these assume

- **`updated` means last *worked***, not last written: the timestamp of the newest Progress Log entry, with creation time as the initial value. Editing a spec does not bump it, so a newly authored plan never outranks the one actually being executed.
- **A question earns a `Q`** in a plan only if it closes by *deciding*. If it closes by doing the work, it is a success criterion or the next action.
- **Notes never track plans.** Plans are self-contained; mirroring their status into notes creates two records of one thing.
- **Personal scope beats repository scope**, which is the opposite of the intuitive reading. A skill you install into a repo never runs for you — only for someone who lacks it personally. So you cannot exercise what you ship, and a defect stays invisible until it reaches its recipient.

`kb/knowledge-management.md` holds the model these rest on: four memory categories split by whether an entry terminates, and three states of context availability — in context, aware-but-not-loaded, and invisible-without-search. The load-bearing rule is that the third is only reachable via a pointer from the first two, which is why a searchable pool without a retrieval instruction is functionally absent.

## Sources

[Skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills) · [Custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions) · [Custom agents](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli) · [Agent configuration reference](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
