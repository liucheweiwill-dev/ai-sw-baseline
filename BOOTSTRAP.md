# BOOTSTRAP.md — Setting up a project from this baseline

You are an agent that has been pointed at this repository and asked to set up
or update a project. Follow this file in order. It is the whole procedure.

**Shell.** Every command here is POSIX shell. On Windows that means Git Bash,
which ships with Git — not `cmd.exe`, and not PowerShell, where `grep` does not
exist and Windows PowerShell 5.1 rejects `||`. Translate rather than improvise
if you are running somewhere else, and say which shell you used.
`template/SETUP.md` is the file that carries per-platform install commands.

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

Check whether the workstation is ready. **Every command here inspects what is
already installed — none of them downloads or executes anything new.** In
particular, do not reach for `npx skills@latest`: `npx` fetches and runs a
mutable remote package, which is exactly the thing rule 1 forbids, and running
it to check readiness would break the rule before setup has begun.

```bash
ls ~/.claude/skills ~/.agents/skills          # installed skills, no download
git -C <project> config user.name             # effective identity, not global
git -C <project> config user.email
codex doctor                                  # auth, model access, sandbox, search
serena --version                              # symbol navigation
codex mcp list                                # Serena registered for Codex
claude mcp list                               # Serena registered for Claude Code
```

Expect: `old-coder`, `ponytail-review`, `ponytail-audit` and
`exhaustive-code-slimmer` visible to both agents; `grill-me` and `grilling` for
Claude Code; an identity whose email belongs to the account that will push;
`codex doctor` reporting a reachable model and a sandbox policy that *fails*
rather than auto-approves an escalation; and `serena` listed as an MCP server on
both sides.

Report every gap to the human with the matching command from `template/SETUP.md`
and stop there. Do not proceed to Step 3 with a half-configured machine — you
will produce a project whose gauntlet cannot run and whose commits are
misattributed.

## Step 1 — Make it a git repository

```bash
git -C <project> rev-parse --git-dir || git -C <project> init
git -C <project> rev-parse HEAD 2>/dev/null || echo "no commits yet"
```

Not optional. The Tier 3 verifier is handed a checkpoint SHA, the diff review
needs a baseline, and checkpoint commits are the real safety net whenever the
sandbox is degraded (`AGENTS.md` §12).

**A repository with no commits has no SHA to hand anyone.** If `HEAD` does not
resolve, the files you create in Steps 2–5 become the first commit, and the
human authorises it before any task work starts. Ask which branch name they
want as the main branch — it goes into `PROJECT.md` at Step 3, and every task
branches off it.

## Step 2 — Copy the template files

Copy these four, unchanged, into the project root:

```
template/CLAUDE.md    ->  <project>/CLAUDE.md     general layer — never edited
template/AGENTS.md    ->  <project>/AGENTS.md     general layer — never edited
template/SETUP.md     ->  <project>/SETUP.md      general layer — never edited
template/PROJECT.md   ->  <project>/PROJECT.md    the form you fill in at Step 3
```

Copy them verbatim. The first three are general layer from top to bottom and
carry no project content at all; if you feel the urge to adjust one, the thing
you want to change belongs in `PROJECT.md` instead.

Do **not** copy this file, or the baseline's own `README.md` and `CLAUDE.md`.
They describe the baseline repository, not the project.

## Step 3 — Fill in PROJECT.md

`PROJECT.md` is the only file with `<FILL IN>` placeholders, and the only one an
update never overwrites. Its required fields are defined in `AGENTS.md` §14.
Ask the human for each; do not infer them from the directory name or from files
you happen to find.

| Section | What to ask |
|---|---|
| Project | What it does, who uses it, and what it deliberately does not do. |
| Tech stack | Language and version, framework, package manager, version files. |
| Commands | install, build, test, lint, typecheck. |
| Gauntlet commands | One command per layer, plus the architecture check — see below. |
| Branches | The main branch name, and how task branches are named. |
| Agent models | Builder and verifier models, effort per Tier, fallback, and the sandbox/approval policy in force — see below. |
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

**Agent models.** Read the account's actual catalog rather than naming models
from memory — it changes, and it differs per account:

```bash
codex debug models                            # the catalog this account can use
grep -E "^model|reasoning" ~/.codex/config.toml   # the configured defaults
```

The catalog is the set of legal answers; the config line is only today's
default. Then fill the table against the rules in `AGENTS.md` §11:

- The **verifier** takes a *different* model that is **not less capable** than
  the builder's. There is no machine-readable capability ordering, so this is
  the human's judgement, not yours to infer from a name: show them the catalog,
  say which model the builder uses, and ask which of the others is at least its
  equal. Record their answer. Never accept a cheaper mini variant — a weak
  adversary clears whatever it fails to understand, and that reads as assurance.
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

Skills guide, static analysis detects, **CI enforces**. Put into the project's
CI every gauntlet layer that has a real command, plus the architecture check if
one exists. Until that exists, those rules are suggestions a tired human can
skip.

**A layer recorded as `not available` is not wired, and that is the correct
outcome** — do not invent a command to fill the slot. CI runs what exists; the
gaps live in the project layer table and reappear in every EVIDENCE report as
the Structural blind spot. That is the exception mechanism, and it is the only
one: a layer is either a command CI runs, or an explicit `not available` with a
reason. Nothing sits in between.

**CI cannot enforce the manual gates.** Human approval of the SPEC, a fresh
verifier session, the four blind inputs, model independence, and the
line-by-line diff review are not machine-checkable. They are enforced by the
record they leave: the `Human approval` section of the SPEC, and the `Roles`,
`Double-track` and `Independent verification` fields of EVIDENCE. Audit those
fields; do not claim CI covers them.

If the project has no CI yet, say so plainly and record it as a known gap
rather than pretending the baseline is fully in force.

## Step 7 — Verify before reporting done

```bash
grep -c "FILL IN" <project>/PROJECT.md    # must be 0
ls <project>/CLAUDE.md <project>/AGENTS.md <project>/SETUP.md \
   <project>/PROJECT.md <project>/ARCHITECTURE.md
ls <project>/docs/development-status.md
git -C <project> rev-parse --git-dir
diff -q <project>/AGENTS.md <baseline>/template/AGENTS.md   # must be identical
```

The last check matters: an `AGENTS.md` that differs from the baseline's copy
cannot be replaced wholesale on the next update, and whatever was edited into it
will be destroyed the first time someone tries.

Then run every gauntlet command from the filled-in table once, on the current
tree, and report which ones pass, fail, or are `not available`. A table of
commands that has never been executed is a guess.

Report to the human: what you created, what is still missing, and every command
you are waiting on them to run.

---

## Updating a project that already has the baseline

An update is two operations: **overwrite**, then **reconcile**. Never a merge.

```bash
git -C <baseline> pull
```

### 1. Overwrite the three general-layer files

```
template/CLAUDE.md   ->  <project>/CLAUDE.md
template/AGENTS.md   ->  <project>/AGENTS.md
template/SETUP.md    ->  <project>/SETUP.md
```

Whole files, no markers, no splicing. They carry no project content, so there is
nothing in them to preserve. **`PROJECT.md` is not in this list and is never
overwritten.**

### 2. Reconcile PROJECT.md against the new schema

Overwriting cannot deliver a new required field, because the file that holds
the answers is the one file the update does not touch. So compare `PROJECT.md`
against the required-field table in the new `AGENTS.md` §14:

- A field in §14 that `PROJECT.md` lacks: **append it as `<FILL IN>`** and ask
  the human to fill it. Do not guess a value.
- A section in `PROJECT.md` that §14 no longer lists: leave it, and tell the
  human it is now unused. Deleting someone's recorded decision is not an
  update's job.

```bash
grep -c "FILL IN" <project>/PROJECT.md    # must be 0 before you report done
```

This step is why a minor release can add a required field at all. Skip it and
the project silently stays on the old schema while claiming the new version.

### 3. Report what changed

Compare the version line before and after and tell the human what changed **in
the rules** — not just that files were updated. If the release added a gauntlet
layer or an architecture check, Step 6's CI wiring needs revisiting too.

If a merge ever looks necessary, something project-specific leaked into a
general-layer file. Move it into `PROJECT.md` instead of merging; a merged
general layer can never be replaced wholesale again, and the whole distribution
model depends on that.
