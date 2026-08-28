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

**Run each of these exactly as written.** If one fails because the command is
not on PATH, *that is the finding* — report it. Do not substitute an absolute
path to make the check pass: the check is asking whether the tool is reachable
the way the rest of the setup will reach it, and a full-path workaround answers
a different question while looking like a green tick.

```bash
ls ~/.claude/skills ~/.agents/skills          # installed skills, no download
git config --global user.name                 # global identity; the per-repo
git config --global user.email                #   check belongs to Step 8
codex doctor                                  # auth, model access, sandbox, search
serena --version                              # must resolve on PATH, not by full path
codex mcp list                                # Serena registered for Codex
```

Expect: `old-coder`, `ponytail-review`, `ponytail-audit` and
`exhaustive-code-slimmer` visible to both agents; `grill-me` and `grilling` for
Claude Code; a configured identity whose email belongs to the account that will
push; `codex doctor` reporting a reachable model and a sandbox policy that
*fails* rather than auto-approves an escalation; and `serena` listed as an MCP
server for Codex.

**Claude Code's MCP registration cannot be checked from here.** A desktop-app
install exposes no `claude` command, so there is no equivalent of `codex mcp
list`. Step 2 writes the project's `.mcp.json`, which is what registers Serena
for Claude Code in this project; confirm it from an interactive session with
`/mcp` after setup, and do not block on it now.

Report every gap to the human with the matching command from `template/SETUP.md`
and stop there. Do not proceed to Step 3 with a half-configured machine — you
will produce a project whose gauntlet cannot run and whose commits are
misattributed.

## Step 1 — Make it a git repository

```bash
git -C <project> rev-parse --git-dir || git -C <project> init -b <main-branch>
git -C <project> rev-parse --verify HEAD 2>/dev/null || echo "no commits yet"
```

Not optional. The Tier 3 verifier is handed a checkpoint SHA, the diff review
needs a baseline, and checkpoint commits are the real safety net whenever the
sandbox is degraded (`AGENTS.md` §12).

**A repository with no commits has no SHA to hand anyone.** Ask which branch
name the human wants as the main branch before initialising, and pass it —
`git init -b <name>` — because `git init` otherwise uses whatever
`init.defaultBranch` happens to be on this machine, which differs between
workstations. That name goes into `PROJECT.md` at Step 3, every task branches
off it, and everything created in Steps 2–7 becomes its first commit at Step 8.

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

Then write `<project>/.mcp.json`, which is what registers Serena for Claude Code
in this project — nothing else in the procedure does it, and Step 0 cannot check
it:

```json
{
  "mcpServers": {
    "serena": {
      "command": "serena",
      "args": ["start-mcp-server", "--context", "claude-code", "--project-from-cwd"]
    }
  }
}
```

The bare command is deliberate. An absolute path here works on exactly one
machine and breaks for everyone who clones the repository — `SETUP.md` §3.2's
`uv tool update-shell` is what makes the bare form resolve.

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

## Step 6 — Make the gauntlet runnable

**The commands in `PROJECT.md` operate on something that does not exist yet.**
Steps 1–5 produced git, documents and configuration; no source tree, no
dependency manifest, no installed tools. Run any gauntlet command now and it
fails for a reason that has nothing to do with the project's quality.

So create the smallest skeleton that makes every command in the table actually
execute. What that means is language-specific — `SETUP.md` §4 names the tools —
but the shape is the same everywhere:

1. **A dependency manifest** listing the gauntlet's own tools, and the install
   command run once. Show the human the command; do not install unasked.
2. **The source packages** the architecture contract names, so the architecture
   check has something to check.
3. **One trivial unit of real behaviour**, plus a test and a property test that
   exercise it. Not the first task's work — scaffolding, named as such, that the
   first task deletes. Without it the Mutation layer has nothing to mutate and
   the Property layer collects nothing.
4. **Tool configuration**: mutation scope, the property-test marker, coverage
   source, the architecture contract.
5. **A `.gitignore`** covering the virtual environment, caches, and every
   artefact the gauntlet writes — coverage reports, mutation caches. Without it
   the first commit swallows the whole toolchain.

Then run every command in the table and fix the configuration until they pass.
**Failures here are toolchain failures and belong to setup**, not to the first
task. Discovering them now is the entire point of this step: a first SPEC that
also has to make eight tools work for the first time cannot tell you which of
its failures are about the code.

One layer may legitimately not pass here: a `CI only` row (`AGENTS.md` §5)
cannot run on this workstation at all. Record it and move on.

## Step 7 — Wire CI

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

## Step 8 — Commit the setup

Everything so far is untracked. **A repository with no commit has no SHA**, and
the workflow hands a SHA to the verifier, to EVIDENCE, and to every checkpoint
reset — so setup is not finished until there is one.

First check the identity that will actually be recorded. Step 0 could only see
the global setting, because the repository did not exist yet; now it does, and a
repository-local override would silently win:

```bash
git -C <project> config user.name     # effective values, not global
git -C <project> config user.email
```

Then show the human what will be committed — `git -C <project> add -A` followed
by `git -C <project> diff --cached --name-only`, so they see the list before it
becomes history rather than after. Nothing from the gauntlet's own output should
appear in it; if caches or coverage reports do, the `.gitignore` from Step 6 is
incomplete. Get their authorisation, then:

```bash
git -C <project> commit -m "Adopt dual-agent baseline <version>"
git -C <project> rev-parse --verify HEAD
```

This is a main-branch commit, so §10's rule applies in full: **the human
authorises it.** Checkpoint commits on task branches come later and are free
(`AGENTS.md` §12); this one is not a checkpoint.

Confirm the branch matches what `PROJECT.md` records as the main branch. `git
init` uses whatever the machine's `init.defaultBranch` says, which is not
necessarily what the human answered at Step 3:

```bash
git -C <project> branch --show-current
```

Rename it now if it differs — `git branch -M <recorded-name>` — while the
repository has exactly one commit and nothing depends on the old name.

## Step 9 — Verify before reporting done

```bash
grep -c "FILL IN" <project>/PROJECT.md    # must be 0
ls <project>/CLAUDE.md <project>/AGENTS.md <project>/SETUP.md \
   <project>/PROJECT.md <project>/ARCHITECTURE.md
ls <project>/docs/development-status.md
git -C <project> rev-parse --verify HEAD  # a SHA, not just a .git directory
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

Comparing sections is enough **because `PROJECT.md` contains no instructions**,
only answers — all guidance lives in `AGENTS.md` §14, which the overwrite above
delivers. If you ever find explanatory prose inside a project's `PROJECT.md`, it
is stale by construction: it froze at whatever release created that project.
Delete it and point at §14.

### 3. Report what changed

Compare the version line before and after and tell the human what changed **in
the rules** — not just that files were updated. If the release added a gauntlet
layer or an architecture check, Step 7's CI wiring needs revisiting too.

If a merge ever looks necessary, something project-specific leaked into a
general-layer file. Move it into `PROJECT.md` instead of merging; a merged
general layer can never be replaced wholesale again, and the whole distribution
model depends on that.
