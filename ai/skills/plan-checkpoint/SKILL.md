---
name: plan-checkpoint
description: This skill should be used before clearing context, compacting, or switching away from plan work — it audits the plan file for consistency and completeness so nothing needed to resume is lost when volatile context is discarded. Use when the user asks to checkpoint, verify the plan, or wrap up before a reset. Writes only to the plan file.
disable-model-invocation: true
---

# Plan Checkpoint

Audit the plan file so it can be safely resumed after context is gone.

`plan-step` and `plan-run` update the plan as they execute, but those updates are a secondary concern during execution and small ones get missed. This skill exists to do one thing with full attention: confirm the plan matches reality at this point in time, before the conversation that holds the missing pieces is discarded.

**This skill does not resolve plans.** It audits the plan already active in the session — the one being worked. There is no argument, no `description` match, and no frontmatter query. If no plan is active, stop and say so rather than picking one; auditing a plan nobody was working on has nothing to check it against, since the corrections come from the conversation about to be discarded.

Read `plans/README.md` in the current repo for the frontmatter contract this audit checks against.

Clearing context with a stale plan is the failure mode this prevents. Treat it as a safety gate, not a formality.

## Audit Checklist

Work through all of these against the plan file:

1. **Next Action** points at the *next* step, not the one just finished.
2. **Success Criteria** checkboxes match what is actually done.
3. **Progress Log** has an entry covering this session's work.
4. **Decisions made in conversation but never written down.** This is the most commonly missed item and the most costly — those decisions exist only in the context about to be discarded. Scan the session for choices, tradeoffs, and rejected approaches, and record them.
5. **Blockers and open questions** raised this session are captured.
6. **`status`** is still accurate — if every success criterion is met, it should be `complete`, and a `complete` plan carries a final Progress Log entry with outcome, verification, and residual risk.
7. **`updated`** matches the newest Progress Log entry. The field means last *worked*, not last written, so a spec edit that added no log entry correctly leaves it alone — do not "refresh" it to now. Bump it with `TZ=US/Central date +'%Y-%m-%d %H:%M'` only when this checkpoint adds or corrects a log entry; read the value from the shell, never from memory.
8. **No duplicated or contradictory entries** left by incremental updates during execution.

## Rules

- Write only to the plan file. Do not write to the notes file; plans are self-contained and are never mirrored there.
- Do not create or update any index file.
- Read the plan being checkpointed. Do not inventory other plans, and do not run a discovery query — the active plan is the only input.
- Prefer correcting the plan over reporting a problem — this skill fixes, it does not just audit.
- Ask before deleting content that may still be needed. Adding is safe; removing is not.
- If the user explicitly asks to commit afterward, use the `commit` skill. Do not commit merely because a checkpoint ran.

## Workflow

1. Take the plan active in this session. If none is, stop — do not resolve or guess one.
2. Read the plan file in full.
3. Work the audit checklist, correcting the plan as you go.
4. Report what was stale and what you changed. If nothing needed correcting, say so plainly — that is a valid and useful result.
5. State explicitly whether the plan is now safe to resume from cold.

## Smoke Test

Prompt: `plan-checkpoint`.

Expected behavior: read the active plan, correct a stale Next Action, record any decisions made this session that were never written down, bump `updated` to match the newest Progress Log entry, and report whether the plan is resumable from cold.
