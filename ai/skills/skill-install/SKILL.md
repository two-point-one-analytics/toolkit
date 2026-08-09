---
name: skill-install
description: This skill should be used to copy one or more user-level skills into the current repo's .github/skills/ so the repo is self-contained and works without the user's global config. Use when preparing a repo for handoff, sharing a repo with a collaborator, or when the user asks to install, replicate, or vendor skills into a repo.
argument-hint: [skill names or glob, e.g. plan-* or capture recall]
---

# Skill Install

Copy user-level skills from `$HOME/.copilot/skills/` into the current repo's `.github/skills/`, so the repo works for someone who does not have the user's global configuration.

This is the handoff mechanism. A repo carrying its own skills plus its own conventions has no dependency on any personal setup.

## Selecting Skills

Resolve the argument against directory names in `$HOME/.copilot/skills/`:

- **Glob** — `plan-*` selects every skill with that prefix.
- **Names** — `capture recall` selects exactly those.
- **No argument** — list what is available and ask. Never install everything by default.

Confirm the resolved list before copying when it came from a glob. State the count.

## Guards

Stop and report rather than proceeding if any of these fail:

- **Must be inside a git repository.** Never install into a non-repo directory.
- **Install only into the current repo.** Never write to another repo's path, even if asked to "also do X" — run there instead.
- **Never copy `skill-install` itself.** It reads from the user's global directory, so a repo-local copy would be broken by design.
- **Every named skill must exist.** If one does not resolve, stop and list what is available rather than installing a partial set.

## Absolute Path Check

After copying, scan every installed file for paths that will not resolve elsewhere. There are **two classes**, and they need different responses — do not collapse them into one list.

```bash
# Class 1 — machine-specific. Always a defect.
rg -n '/Users/|/home/[a-z]|C:\\Users' .github/skills/<installed>/

# Class 2 — home-anchored. Portable across the author's machines, not to another person.
rg -n '\$HOME/|~/' .github/skills/<installed>/
```

**Class 1 is a defect.** A hardcoded machine path breaks for everyone except its author, and the failure is invisible until it happens. Offer to rewrite as relative or repo-relative, but never rewrite silently.

**Class 2 needs validation, not a rewrite.** `$HOME/notes/…` resolves on any of the author's machines and is still wrong for a collaborator, who has a home directory but not that file. These are frequently legitimate — a personal notes file, a memory directory — so report them as items to **validate or repoint whenever the skills land on a new system**, which is exactly what a port or a handoff is.

Skill files should write `$HOME/…`, never bare `~/…`. Both usually resolve the same, but `~` stays literal inside double quotes while `$HOME` expands, and neither expands in a tool call. The decisive reason is this scan: one consistent form is what makes class 2 exact instead of approximate. **A bare `~/` hit is therefore a convention violation** — flag it for correction on the spot.

Expect false positives in both classes — prose that *discusses* a path reads the same as a path being used. Report hits for human review rather than judging them.

**Some skills are inherently personal and should not be installed at all.** Anything whose whole purpose is reading or writing the user's private files — scratchpad skills, personal memory skills — will be broken for a collaborator no matter how the paths are written. If the resolved list includes one, say so and confirm before copying it.

## Companion Files

This skill installs **skills only**. Some skills need supporting files elsewhere in the repo — the plan family needs `plans/README.md` and `plans/plan-template.md`, for example.

Those are the owning skill's responsibility, not this one's. `plan-create` scaffolds its own docs from `seed/` on first use. After installing a family, mention any known first-run step rather than trying to provision it here.

## Overwrite Behavior

- **Overwrite existing skills.** They are mechanism, and re-running should propagate fixes.
- **Report every file that changed**, so an intentional repo-local customization is not silently lost.
- If a repo-local skill differs substantially from the source, say so before overwriting and let the user decide.

## Workflow

1. Confirm the working directory is a git repo. Stop if not.
2. Resolve the skill list from the argument. Confirm it if it came from a glob.
3. Create `.github/skills/` if absent.
4. **Diff before copying.** For each skill already present, capture `diff -r $HOME/.copilot/skills/<name> .github/skills/<name>` and read the hunks. Overwriting destroys the previous content, so this cannot be reconstructed afterward — a report written after the copy can only list filenames.
5. Copy each skill directory recursively, including any `seed/` or reference files it carries.
6. Run the absolute path check across everything installed.
7. Report: skills installed, what actually changed in each overwritten file, both path classes reported separately, and any first-run step the user still needs.

## Report Format

```
Installed to .github/skills/  (N skills)
  plan-create        (new — 3 files)
  plan-retrieve      (unchanged)
  plan-step          (overwritten — 1 of 1 files changed)
      SKILL.md  +12 -3   added `argument-hint`; rewrote the checkpoint-boundary
                         section to require a preview before asking to continue

Machine-specific paths (defects):
  (none)

Home-anchored paths (validate on the new system):
  note/SKILL.md:9   $HOME/notes/ai-notes.md   — personal notes file
  plan-create/SKILL.md:34  $HOME/.copilot/skills/    — prose, describes where the skill runs from

Next: run /plan-create in this repo to scaffold plans/README.md
```

**Summarize the substance of each change, not just the line counts.** `+12 -3` says a file moved; "added `argument-hint`" says whether it moved the way you intended. One clause per changed file is usually enough — enough to recognize an edit you made deliberately, and to catch one you did not.

Report unchanged skills too. "Unchanged" is the result that tells the user a repo is already current, and omitting it makes a no-op run look like nothing happened.

State plainly when there is nothing to report — no absolute paths and no changes is a good result worth saying out loud.

## Smoke Test

Prompt: `/skill-install plan-*`.

Expected behavior: resolve the six plan skills, confirm the list, diff any already present before overwriting them, copy them into `.github/skills/`, scan for absolute paths, and report what was installed — including what changed in each overwritten file — plus the `/plan-create` first-run step, without touching `plans/` directly.
