# BOOTSTRAP.md — Setting up a project from this baseline

You are an agent that has been pointed at this repository and asked to set up
or update a project. Follow this file in order. It is the whole procedure.

**Two hard rules before you start.**

1. **Install nothing automatically.** Where a step needs software, show the
   command from `template/SETUP.md`, say what it does, and wait for the human
   to confirm that specific command. A workstation is not yours to change.
2. **Invent nothing.** The project layer describes a real project you do not
   know yet. Ask. A `<FILL IN>` you guessed at is worse than one left visible,
   because it looks settled.

---

## Step 0 — Confirm the target and the mode

Establish, from the human, before touching any file:

- Which directory is the project.
- **New project** (no baseline files yet) or **update** (baseline files already
  present). For an update, jump to §Updating.

Check whether the workstation is ready:

```bash
npx skills@latest list -g
git config --global --get-regexp "^user\."
```

Expect `old-coder`, `ponytail-review`, `ponytail-audit`,
`exhaustive-code-slimmer` on both agents, `grill-me` and `grilling` on Claude
Code, and a configured git identity. Anything missing: point the human at
`template/SETUP.md` and let them run it. Do not proceed to Step 3 with a
half-configured machine — you will produce a project whose gauntlet cannot run.

## Step 1 — Make it a git repository

```bash
git -C <project> rev-parse --git-dir || git -C <project> init
```

Not optional. The Tier 3 verifier needs an exact source state, the diff review
needs a baseline, and the git checkpoint is the real safety net whenever the
sandbox is degraded.

## Step 2 — Copy the template files

Copy these three, unchanged, into the project root:

```
template/CLAUDE.md   ->  <project>/CLAUDE.md
template/AGENTS.md   ->  <project>/AGENTS.md
template/SETUP.md    ->  <project>/SETUP.md
```

Copy them verbatim. The general layer is designed to need no edits; if you feel
the urge to adjust one, the thing you want to change belongs in the project
layer instead.

Do **not** copy this file, or the baseline's own `README.md` and `CLAUDE.md`.
They describe the baseline repository, not the project.

## Step 3 — Fill in the project layer

Only `AGENTS.md` has a project layer. It sits below the `END GENERAL LAYER`
marker and contains `<FILL IN>` placeholders. Ask the human for each; do not
infer them from the directory name or from files you happen to find.

| Section | What to ask |
|---|---|
| Project | What it does, who uses it, and what it deliberately does not do. |
| Tech stack | Language and version, framework, package manager, version files. |
| Commands | install, build, test, lint, typecheck. |
| Gauntlet commands | One command per layer — see below. |
| Agent models | Builder and verifier models, and the effort per Tier — see below. |
| Project-specific safety | Regulated data, code-level security checks, licence limits. Write `none` if there is nothing. |

**Gauntlet commands are the step that matters.** `template/SETUP.md` §4
suggests tools per language. Two rules when filling the table:

- A layer must be able to **fail**. A coverage command without a threshold flag
  prints a number and exits 0 — it is decoration, not a layer.
- Delete no row. If a layer cannot run in this project, write `not available`
  and the reason. That row becomes the Structural blind spot in every EVIDENCE
  report, which is the honest outcome — a silently missing row is not.

Any tool named here that is not installed goes back to the human as a command
from `SETUP.md` §4. You do not install it.

**Agent models.** Read the account's actual options rather than naming models
from memory — they change, and they differ per account:

```bash
grep -E "^model|reasoning" ~/.codex/config.toml
```

The configured model and effort are the builder's defaults. Then fill the table
against the rules in `AGENTS.md` §11:

- The **verifier** takes a *different* model that is **not less capable** than
  the builder's. Same generation, different name is the usual answer. Never a
  cheaper mini variant — a weak adversary clears whatever it fails to
  understand, and that reads as assurance.
- **Tier 1** takes a lower effort, applied per call with
  `-c model_reasoning_effort=<lower>`. Do not lower the configured default;
  Tier 2 and 3 need it.

If the human does not know which models are available, that is a question for
them and their provider, not a guess for you to record.

## Step 4 — Write ARCHITECTURE.md

Create `<project>/ARCHITECTURE.md`: the allowed dependency direction and the
forbidden edges. Short — a diagram in text plus a list. For example:

```
UI -> Application -> Domain -> Core
Infrastructure -> Application, Domain

Forbidden:
  Core -> anything
  Domain -> Infrastructure, UI
  Application -> UI
  any cycle
```

For a project with no layering yet, write what is true today and say so. An
`ARCHITECTURE.md` that describes an aspiration is worse than none, because
every future change gets checked against fiction.

Wire the deterministic check for it where one exists — import-linter for
Python, dependency-cruiser for JS/TS, clang-uml diffs for C++. A dependency
rule with no check behind it is a suggestion.

## Step 5 — Create the docs skeleton

```
<project>/docs/development-status.md
```

One line per task once work starts: `<task-id> | Tier | double-track`, plus
cross-task decisions and their reasons. Per-task SPEC and EVIDENCE files live
in `docs/<NNN-kebab-slug>/` and are created by the workflow, not now.

## Step 6 — Wire CI

Skills guide, static analysis detects, **CI enforces**. Put the seven gauntlet
layers and the architecture check into the project's CI. Until that exists,
every rule in `AGENTS.md` is a suggestion that a tired human can skip.

If the project has no CI yet, say so plainly and record it as a known gap
rather than pretending the baseline is fully in force.

## Step 7 — Verify before reporting done

```bash
grep -c "FILL IN" <project>/AGENTS.md     # must be 0
ls <project>/CLAUDE.md <project>/AGENTS.md <project>/SETUP.md <project>/ARCHITECTURE.md
ls <project>/docs/development-status.md
git -C <project> rev-parse --git-dir
```

Then run every gauntlet command from the filled-in table once, on the current
tree, and report which ones pass, fail, or are `not available`. A table of
commands that has never been executed is a guess.

Report to the human: what you created, what is still missing, and every command
you are waiting on them to run.

---

## Updating a project that already has the baseline

The general layer is versioned and carries no project-specific content, so an
update is a **whole-section replacement, never a merge**.

```bash
git -C <baseline> pull
```

For each of `CLAUDE.md`, `AGENTS.md`, `SETUP.md`: replace everything from the
`GENERAL LAYER` marker to the `END GENERAL LAYER` marker with the new version,
leaving the project layer untouched. `CLAUDE.md` and `SETUP.md` are entirely
general layer — replace the whole file.

Compare the version line before and after, and tell the human what changed in
the rules, not just that files were updated.

If a merge ever looks necessary, something project-specific leaked into the
general layer. Move it down into the project layer instead of merging; a merged
general layer can never be replaced wholesale again, and the whole distribution
model depends on that.
