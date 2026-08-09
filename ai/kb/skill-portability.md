# Skill Portability

Summary: How skills resolve across scopes, why an author cannot test what they ship, and the path conventions that decide whether an installed copy works on someone else's machine.

## Personal Scope Wins Over Project Scope

Summary: When the same name exists at both levels, the copy closest to the *user* wins — not the more specific one.

Context: The intuitive assumption is that a repo-local skill overrides a personal one by the usual rule of specificity. It is backwards, and it matters whenever the same name exists at both levels — which is exactly what installing skills into a repo creates.

Learning:

- Copilot documents this explicitly for **agents**: a custom agent in the home directory is used instead of a same-named one in the repository.
- For **skills**, Copilot documents no precedence rule across scopes. Until that behavior is known, avoid installing the same skill name at both levels — the resolution is unspecified, not merely undocumented.
- Personal scope: `$HOME/.copilot/skills/` or `$HOME/.agents/skills/` for skills, `$HOME/.copilot/agents/` for agents.
- Repository scope: `.github/skills` or `.agents/skills` for skills; `.github/agents/` for agents.

## Repo-Installed Skills Are Never Exercised By Their Author

Summary: Because personal scope wins, the owner of a repo always runs their own copy; the installed copy only ever runs for someone else.

Context: Installing skills into a repo makes it self-contained for a collaborator who has none of your global configuration. But the person who installs them keeps hitting their personal copy, so their own use never touches what was shipped.

Learning:

- A defect in an installed copy — a bad path, a truncated file, a botched copy — is invisible to the author and surfaces only for the recipient.
- Byte-identical at install time is not the risk; **drift is**. Later fixes to the personal copy silently stop applying to every repo already carrying one, with no signal.
- The cheap guard is a diff, not a test: compare the personal directory against each repo's copy before handing the repo over.
- Genuinely exercising a repo copy requires moving the personal one aside first.

## Path Anchoring

Summary: Three anchors — skill-relative, repo-relative, `$HOME` — and `$HOME` is the required syntax, never bare `~`.

Context: Two path bugs in one session traced to the same cause: anchoring to a user-specific subtree instead of a defined root. The syntax half was settled after finding that an install-time scan matched neither `$HOME` nor a tilde path outside the config directory, so a personal notes path passed clean and was never surfaced as the accepted exception it was believed to be.

Learning:

- **Three tiers.** Skill assets anchor to *the directory the SKILL.md was loaded from* — never a hardcoded home path, since the same skill runs from both a personal directory and a repo-local one, and the repo copy must use its own assets. Repo data anchors to the repo root, unanchored. Personal data anchors to `$HOME`. Never anchor to a user-specific subtree like `dev/personal/<repo>`.
- **Write `$HOME/…`, never `~/…`.** They usually resolve identically, but `~` stays literal inside double quotes while `$HOME` expands, and quoting happens as soon as a path contains a space. Neither expands in a tool call — an agent resolves either form itself — so the decisive reason is not correctness but **greppability**: one consistent form is what makes the install-time scan exact rather than approximate.
- **`$HOME` does not make a path portable to another person.** `$HOME/notes/…` works on every machine its author owns and is still broken for a collaborator, who has a home directory but not that file. Home-anchoring solves cross-machine, not cross-person.

## The Two-Class Portability Scan

Summary: Machine-specific paths are defects; home-anchored paths are a checklist.

Context: A single "find absolute paths" scan conflates two problems with different fixes, and reports legitimate paths as errors until the reader stops reading the output.

Learning: Scan installed files in two passes.

```sh
# Class 1 — machine-specific. Always a defect.
rg -n '/Users/|/home/[a-z]|C:\\Users' <installed-skills-dir>

# Class 2 — home-anchored. Portable across the author's machines, not to another person.
rg -n '\$HOME/|~/' <installed-skills-dir>
```

- **Class 1 is a defect.** A hardcoded machine path breaks for everyone except its author, and the failure is invisible until it happens. Rewrite as relative or repo-relative.
- **Class 2 needs validation, not a rewrite.** These are frequently legitimate — a personal notes file, a memory directory. Report them as items to validate or repoint whenever the files land on a new system, which is exactly what a port or a handoff is.
- **A bare `~/` hit is a convention violation**, since the standard is `$HOME`. Fix on the spot.
- Expect false positives in both classes: prose that *discusses* a path reads the same as a path being used. Report hits for human review rather than judging them.
