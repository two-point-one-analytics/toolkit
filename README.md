# Toolkit

Shared development and productivity resources — AI instructions, skills, and sub-agents, plus knowledge documents, technical playbooks, and config files.

Everything here is general-purpose and domain-agnostic: artifacts meant to be copied, adapted, and reused across projects rather than run in place.

**No confidential information.** This repo contains no client data, credentials, or business-sensitive material — only general knowledge and reusable artifacts.

## Contents

| Directory | Contents |
|---|---|
| `ai/` | Agent skills, sub-agents, instruction files, and the knowledge base behind them. See [`ai/README.md`](ai/README.md) |
| `configs/` | Tool and editor configuration files |
| `docs/` | Knowledge documents and technical playbooks |

## Using it

Nothing here installs itself. Clone the repo, copy what you want into place, and adjust any paths for your machine.

`ai/` is the most developed section — a working set of skills and sub-agents for GitHub Copilot CLI, with install steps and the two paths that need repointing documented in [`ai/README.md`](ai/README.md).

## Conventions

- Markdown files use lowercase-and-dashes names, except directory overview files named `README.md` or `INDEX.md`.
- Paths outside a repo are written `$HOME/...` rather than `~`, so they expand correctly when quoted.
- Home-anchored paths are portable across machines, not across people. Anything shipped to someone else needs those repointed or removed.

## License

`ai/` is MIT — see [`ai/LICENSE`](ai/LICENSE).
