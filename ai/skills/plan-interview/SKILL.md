---
name: plan-interview
description: Refine an existing plan that already has core content — interview through open questions, stress-test assumptions, and resolve decisions until it is execution-ready. To create a plan first use /plan-create; to drive it after use /plan-step or /plan-run.
argument-hint: [plan-file or description]
disable-model-invocation: true
---

# Plan Interview

Refine an existing current-repo plan into an execution-ready state by surfacing unknowns, resolving decisions, and stress-testing assumptions. The plan should already exist with core content (goal, scope, initial steps). If the referenced plan is missing or essentially empty, stop and recommend `/plan-create` to create it first. Do not make implementation changes.

Identify the plan file from the provided argument or the current conversation, resolving by explicit path, `description` match, or most recently worked incomplete plan. Query frontmatter — never read plan bodies to search. Confirm the selection when it was inferred, and ask rather than guess when several plans match or the top two share the same `updated` value. The plan file is the durable source of truth for the question backlog, answers, and decisions.

```bash
# Most recently worked incomplete plans
rg -l -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^status: incomplete$' plans/ | xargs rg -N --no-heading '^updated:' | sort -t: -k3 -r | head -5

# Match a description
rg -N --no-heading -i -g '!README.md' -g '!plan-template.md' -g '!INDEX.md' '^description:.*<term>' plans/
```

All three exclusions are load-bearing. `README.md` and `plan-template.md` carry example frontmatter that matches these patterns, and the template's `YYYY-MM-DD` placeholder sorts *above* real dates — dropping that guard silently resolves to the template. `INDEX.md` excludes plan-cluster entry points, which are navigation rather than resumable work.

`plans/README.md` holds the full frontmatter contract. Read it only if resolution fails or the repo's conventions look non-standard.

## Rules

- Operate on the existing plan; do not start a new plan from scratch (use `/plan-create` for creation).
- Do not make implementation changes unless the user explicitly switches out of planning.
- Prefer reading/searching the repo over asking questions when the answer can be discovered.
- Ask the user only for requirements, preferences, tradeoffs, priorities, permissions, or decisions that cannot be inferred safely.
- Build the full question backlog first so the decision surface is visible, then work through one active question at a time.
- For each question, include a recommended answer and why.
- Allow back-and-forth on the active question until it is resolved before moving on.
- Update the question backlog as answers resolve, change, add, or remove later questions.
- Do not rely on an in-session todo list, session memory, or compaction for long question-and-answer retention.
- Do not proceed to implementation. The output is an execution-ready plan.

## Interview Areas

Cover the areas relevant to the work:

- Goal: desired outcome and why it matters.
- Scope: included work, excluded work, acceptable shortcuts.
- Context: repo, files, systems, users, stakeholders, external constraints.
- Requirements: functional behavior, data rules, UX expectations, compatibility needs.
- Constraints: deadlines, safety, permissions, dependencies, environments.
- Decisions: architecture, naming, storage location, interfaces, migration strategy.
- Validation: tests, manual checks, QA commands, success criteria.
- Rollback: failure modes, reversibility, cleanup, monitoring.
- Handoff: what future agents or collaborators need to know.

## Workflow

1. Restate the goal and known context from the existing plan.
2. Explore repo/docs if needed to avoid asking answerable questions.
3. Confirm the plan file has core content; if it does not, recommend `/plan-create` to create it first and stop.
4. Identify open decisions and dependencies between them.
5. Draft the question backlog — for each, what is at stake and a recommended answer with its rationale. Apply the closes-by-deciding test before numbering.
6. Write the backlog into the plan's `## Open Questions` section before starting the interview, so it survives a context reset.
7. Ask only the first unresolved question.
8. After each resolved question or meaningful batch of answers, mark the outcome in its heading, move it to `## Resolved Questions`, and adjust later questions if needed.
9. Continue until all implementation-blocking ambiguity is resolved.
10. Finalize the execution-ready plan suitable for `/plan-step` or `/plan-run`.

## Open Questions Format

Questions live in the plan's existing `## Open Questions` section, in the shape `plans/README.md` defines. Do not create a separate `## Questions` section or invent a parallel format — a second section splits the decision surface the interview exists to make visible.

**A question earns a `Q` only if it closes by deciding.** Apply this before numbering anything:

- Closes by **deciding** → it is a question. *"Should the report table stay aggregate-only, or add a detailed flattened table?"*
- Closes by **doing the work** → it is not. That belongs in Success Criteria, or in Next Action if it is the immediate step. An item phrased as *"When does X happen?"* reads like a question but closes when X is done, and it will duplicate whatever criterion tracks the same thing.

Reclassify backlog items that fail the test rather than numbering them.

```markdown
### Q1. <the decision to be made>

<What is at stake, the options, and a recommended answer with its rationale.>
```

Resolved questions keep their numbers — Progress Log entries refer to them by number. Mark the outcome in the heading (`— RESOLVED: <the decision>`) and move them under a `## Resolved Questions` heading so the open section stays scannable. Never delete or renumber a question.

If the plan has no `## Open Questions` section, add one in the position `plans/plan-template.md` gives it.

## Final Output

End with:

- Goal
- Non-goals
- Requirements
- Decisions made
- Open questions, if any
- Implementation steps
- Validation plan
- Risks and rollback notes
- Recommended `/plan-step` or `/plan-run` prompt

## Smoke Test

Prompt: `/plan-interview dbt core migration`.

Expected behavior: match `description` against the frontmatter query, confirm the inferred selection, verify the plan has core content, then write a question backlog into its `## Open Questions` section — reclassifying any item that closes by doing the work rather than by deciding — and ask only the first question before waiting.
