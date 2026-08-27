# PROJECT.md — this project's values

<!-- ============================================================ -->
<!-- PROJECT LAYER. This file is yours; the baseline never edits   -->
<!-- it and an update never overwrites it.                         -->
<!--                                                               -->
<!-- The required fields are defined in AGENTS.md §14. After a      -->
<!-- baseline update, reconcile this file against that list:        -->
<!-- append any missing field as <FILL IN> and fill it in.          -->
<!--                                                               -->
<!-- Leaving <FILL IN> anywhere is a setup defect.                  -->
<!-- ============================================================ -->

## Project

<FILL IN — what this project is, in two or three sentences: what it does, who
uses it, and what it deliberately does not do. Every project fills this in for
itself; the baseline never ships a value here.>

## Tech stack

<FILL IN — language and version, framework, package manager, anything a
version file pins.>

## Commands

```bash
<FILL IN — install, build, test, lint, typecheck>
```

## Gauntlet commands

One command per layer. Delete no row: if a layer cannot run in this project,
write `not available` and the reason — that row becomes the Structural blind
spot in every EVIDENCE report, and CI wires only the rows that have commands.

| Layer | Command |
|---|---|
| Tests | `<FILL IN>` |
| Types | `<FILL IN>` |
| Lint + format | `<FILL IN>` |
| Changed-line coverage | `<FILL IN — needs a comparison base and a threshold, or `not available`>` |
| Mutation | `<FILL IN>` |
| Property-based | `<FILL IN>` |
| Cleanup | `<FILL IN — must exit non-zero on findings>` |

Architecture check: `<FILL IN, or `not available`>`

See `SETUP.md` §4 for per-language tool suggestions and install commands.

## Branches

- Main branch: `<FILL IN>`
- Task branch naming: `<FILL IN — e.g. task/<NNN-kebab-slug>>`

Checkpoints are commits on the task branch (AGENTS.md §12). Nothing reaches the
main branch without human authorisation.

## Agent models

Model names and effort levels change often and differ per account, so they live
here. The rules they must satisfy are in AGENTS.md §11.

| Role | Model | Reasoning effort |
|---|---|---|
| Builder, Tier 2–3 | `<FILL IN>` | `<FILL IN — enough for implementation, the gauntlet, and the EVIDENCE mapping>` |
| Builder, Tier 1 | same as above | `<FILL IN — lower; overridden per call, not configured>` |
| Verifier, Tier 3 | `<FILL IN — a different model, judged by a human to be at least the builder's equal>` | `<FILL IN>` |

Fallback when the builder model is unavailable: `<FILL IN, or `none`>`.

Codex sandbox and approval policy in force: `<FILL IN — and confirm it fails
rather than auto-approves an escalation>`

## Project-specific safety

<FILL IN — anything beyond the general operational boundaries in AGENTS.md §10:
regulated data, code-level security checks for this stack, licence constraints.
Write `none` if there is nothing.>
