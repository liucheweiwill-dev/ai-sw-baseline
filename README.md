# ai-sw-baseline

A reusable baseline for software projects developed by **Claude Code and Codex
working together**. It fixes the roles, the workflow, the risk tiers, and the
evidence a change must produce before it is trusted — so every project starts
from the same rules instead of whatever the last project happened to leave
behind.

## Start here

| I want to… | Read |
|---|---|
| Set up a new project, or update one | **[BOOTSTRAP.md](BOOTSTRAP.md)** |
| Set up a workstation (skills, Serena, tooling) | [template/SETUP.md](template/SETUP.md) |
| Read the rules themselves | [template/AGENTS.md](template/AGENTS.md) |
| Change the baseline | [CLAUDE.md](CLAUDE.md) |

Pointing an agent at this repository is enough: *"read
`BOOTSTRAP.md` in https://github.com/liucheweiwill-dev/ai-sw-baseline and set
this project up."*

## Layout

```
BOOTSTRAP.md     how to set up or update a project from this baseline
CLAUDE.md        how to work on this repository
README.md        this file
template/        the three files that get copied into a project
├─ CLAUDE.md     Claude-specific; points at AGENTS.md
├─ AGENTS.md     the single source of shared rules + the project layer
└─ SETUP.md      what to install on a workstation
```

Nothing outside `template/` is ever copied into a project.

## The model in short

Claude Code is the architecture lead and writes the SPEC. Codex reviews it for
feasibility, implements it, runs a seven-layer gauntlet, and reports EVIDENCE.
**A human approves the SPEC before any code is written** — that approval is the
only step that breaks the correlation of everything being authored by the same
agent, so it is not optional.

Work is tiered. Trivial changes stay cheap; high-stakes changes (money, auth,
data loss, concurrency, public API) add a failure model and an independent
verifier. Tiers ratchet up only — lowering one takes explicit human
instruction.

Every file is split into a **general layer** that is identical across projects
and a **project layer** that each project fills in. Updates replace the general
layer wholesale, which only works because it contains nothing project-specific.

## Assumptions

Both Claude Code and Codex are available. Rules that need both are marked
`[dual-agent]`; the rest still applies with a single agent, and the resulting
EVIDENCE says so.

Commands are written for Windows with macOS and Linux equivalents alongside.
Documents in this repository, and the SPEC and EVIDENCE files it asks projects
to produce, are in English.

## Credit

The evidence-first loop is adapted from
[old-coder](https://github.com/AmazingAng/old-coder); the anti-over-engineering
rules draw on [ponytail](https://github.com/DietrichGebert/ponytail); the
design-convergence step uses
[grill-me](https://github.com/mattpocock/skills). Symbol navigation is
[Serena](https://github.com/oraios/serena).
