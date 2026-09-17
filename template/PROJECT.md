# PROJECT.md — this project's values

<!-- ============================================================ -->
<!-- This file holds answers, not instructions. How to fill each   -->
<!-- field, and what "reconcile after an update" means, are in     -->
<!-- AGENTS.md §14 — which is overwritten on update, so it cannot  -->
<!-- go stale the way guidance written here would.                 -->
<!--                                                               -->
<!-- One exception, declared in AGENTS.md §10: the project-specific -->
<!-- safety section carries authority. Write rules there.           -->
<!--                                                               -->
<!-- The baseline never edits this file and an update never        -->
<!-- overwrites it. Any placeholder left unfilled is a setup       -->
<!-- defect; this comment block avoids writing the placeholder     -->
<!-- token so that counting it here counts only real blanks.       -->
<!-- ============================================================ -->

## Project

<FILL IN>

## Tech stack

<FILL IN>

## Commands

```bash
<FILL IN>
```

## Profile

One row per capability in AGENTS.md §3.1. The check runs on every release,
whatever the task's Tier was.

| Capability | Present? | Standing check |
|---|---|---|
| Customer data | `<FILL IN>` | `<FILL IN>` |
| Multi-tenant | `<FILL IN>` | `<FILL IN>` |
| Payments | `<FILL IN>` | `<FILL IN>` |
| Uploads | `<FILL IN>` | `<FILL IN>` |
| Model calls | `<FILL IN>` | `<FILL IN>` |
| Public availability | `<FILL IN>` | `<FILL IN>` |

## Gauntlet commands

| Layer | Command |
|---|---|
| Tests | `<FILL IN>` |
| Types | `<FILL IN>` |
| Lint + format | `<FILL IN>` |
| Real execution | `<FILL IN>` |
| Changed-line coverage | `<FILL IN>` |
| Mutation | `<FILL IN>` |
| Property-based | `<FILL IN>` |
| Unused code | `<FILL IN>` |

Architecture check: `<FILL IN>`

## Release and operation

- Artifact identity: `<FILL IN>`
- Deploy target: `<FILL IN>`
- Deploy authorisation standing for that target: `<FILL IN>`
- Recovery mechanism: `<FILL IN>`
- Recovery last exercised: `<FILL IN>`
- Backup restore last verified: `<FILL IN>`
- Continuing checks and their intervals: `<FILL IN>`
- Alert channel: `<FILL IN>`
- Alert channel last tested end to end: `<FILL IN>`
- Consumption ceilings: `<FILL IN>`
- Behaviour when a ceiling is reached: `<FILL IN>`
- Procedures that work with no agent available: `<FILL IN>`

## Model components

| Component | Evaluation set and its provenance | Scoring and threshold | Re-evaluation triggers | Spend ceiling and its stop |
|---|---|---|---|---|
| `<FILL IN>` | `<FILL IN>` | `<FILL IN>` | `<FILL IN>` | `<FILL IN>` |

Repetition policy, fixed before any run: `<FILL IN>`

## Branches

- Main branch: `<FILL IN>`
- Task branch naming: `<FILL IN>`

## Agent models

| Role | Model | Reasoning effort |
|---|---|---|
| Feasibility review | `<FILL IN>` | `<FILL IN>` |
| Builder, Tier 1 | `<FILL IN>` | `<FILL IN>` |
| Builder, Tier 2 | `<FILL IN>` | `<FILL IN>` |
| Builder, Tier 3 | `<FILL IN>` | `<FILL IN>` |
| Verifier, Tier 3 | `<FILL IN>` | `<FILL IN>` |

Configured default effort: `<FILL IN>`

Capability gap between builder and verifier: `<FILL IN>`

Fallback when the builder model is unavailable: `<FILL IN>`

Codex sandbox and approval policy in force: `<FILL IN>`

## Project-specific safety

<FILL IN>
