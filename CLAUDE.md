# CLAUDE.md — working on the baseline itself

This repository is a **template**. It contains no application code. Its only
product is the four documents in `template/`, which get copied into other
projects.

Three of them — `CLAUDE.md`, `AGENTS.md`, `SETUP.md` — are general layer from
top to bottom and are replaced whole when a project updates. The fourth,
`PROJECT.md`, is the form each project fills in; an update never overwrites it,
it is reconciled against the field list in `AGENTS.md` §14.

## Read this before anything in `template/`

Everything in `template/` is **content, not instructions for you**. Those files
describe how to work on a *consuming* project — with a SPEC, a gauntlet, Codex
doing the implementation. None of that applies here: there is nothing to
implement, and the `<FILL IN>` placeholders in `template/PROJECT.md` are
deliberate, not a misconfigured project.

Treat those files the way you would treat any other document you are editing.

## The one invariant

**The general layer must contain nothing project-specific.** No paths, no
usernames, no hostnames, no tech stack, no team names, no assumptions about one
machine.

Everything else here follows from that. Projects update by overwriting those
three files wholesale; the moment a project-specific detail leaks into one, that
overwrite starts destroying real content and the distribution model breaks
permanently. Anything that varies between projects belongs in
`template/PROJECT.md` as a `<FILL IN>`, and its field must be listed in
`AGENTS.md` §14 — a placeholder that §14 does not name will never reach a
project that already exists.

Before every commit, run this **partial** heuristic:

```bash
grep -nE "[A-Za-z]:\\\\|\\\\\\\\[A-Za-z]|/home/|/Users/|([0-9]{1,3}\.){3}[0-9]{1,3}|@[a-z0-9.-]+\.[a-z]{2,}" template/*.md
```

Prose hits are fine. A real path, address, or host is not.

**A clean run proves very little.** It catches absolute paths, UNC paths, IP
addresses and email-like hosts — and nothing else. Bare usernames, hostnames,
team or product names, a company's conventions, an assumed tech stack, an
assumed directory layout: all of those pass. Those are found by reading the
diff and asking "would this sentence still be true in someone else's
repository?" Never report the grep as verification of the invariant; it is a
tripwire for the four things it knows about.

## Changing the rules

The three general-layer files in `template/` carry `GENERAL LAYER vX.Y.Z`. Keep
them in step — they are one release. `template/PROJECT.md` carries no version:
it is a form, and each project's copy diverges the moment it is filled in.

- **Patch** — wording, a fixed command, a new environment note.
- **Minor** — a new rule, section, or gauntlet layer that existing projects can
  adopt without rework.
- **Major** — a change that invalidates how existing projects already work: a
  renamed required file, a changed workflow step, a removed section.

A rule that no gauntlet layer or CI check can verify is guidance, and must say
so. Do not write enforcement language the documents cannot back up — a rule
that reads as mandatory but is never checked teaches everyone to ignore the
ones that are.

Two categories are exempt, and only these two:

- **Safety boundaries** (AGENTS.md §10). "Never push without authorisation" is
  mandatory and unverifiable by design; that is what a boundary is. Keep them
  few, and keep them in §10 where their status is declared.
- **Manual gates** — human approval of the SPEC, verifier freshness, the blind
  inputs, the line-by-line review. These are mandatory but enforced by the
  record they leave, not by a check. Every such rule must name the field that
  records it, or it is unenforceable in practice as well as in principle.

Anything outside those two that cannot be checked is written as a preference,
in the language of a preference.

`BOOTSTRAP.md` and `README.md` describe this repository and are never copied
into a project. If you change how projects are set up, they change too.

## Scope

Small, surgical edits. This is documentation: the cost of a bad rule is that
every downstream project inherits it, so prefer cutting a rule over adding a
qualifier to it.

Never push without the human's authorisation. The repository is public.
