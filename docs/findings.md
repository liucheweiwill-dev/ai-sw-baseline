# Findings against the baseline

Recorded when they are hit, not batched at the end. A finding here is an
observation from real use, not a decision — a decision becomes a proposal, and
a proposal becomes a release.

Each entry says what was being done, what the baseline failed to account for,
and a candidate change. `Status` is one of `open`, `in proposal NNN`, `closed
in vX.Y.Z`, or `declined`.

---

## F1 — A task has no size, only a consequence

**Hit:** 2026-09-18, before PC-Desk started, from the operator's own experience
of the preceding weeks.
**Against:** v3.0.0, §3, §11, §12.
**Status:** open.

### What happened

Development stops when a subscription's quota window is exhausted. When that
happens part-way through a task, the reasoning the session had accumulated is
gone, and the work is paid for a second time when it resumes.

This is not the same problem as consumption being too fast. Quota can be
enlarged with money; **a task too large to finish inside one window will be
interrupted at any quota level.** The two were being treated as one problem.

Context: OpenAI reset paid Codex limits three times in recent weeks — 8 August,
29 August, and a global reset on 7 September — after fixing bugs that caused
overconsumption in compaction, memory workers, goals, automations, subagents and
MCP handling, said to make a weekly allowance last 10–50% longer. That is real,
and it is not the finding. The finding is that the baseline has nothing to say
about the interruption either way.

### The gap

**§3 sizes a task by consequence and by structural change. Nothing sizes it by
whether it can be finished.** A task that cannot complete inside one quota
window is a task that will be paid for twice, and the Tier table gives no reason
to notice that before starting.

**§12 mandates two checkpoints** — before a deletion pass, and after the
gauntlet. Neither exists so that an interruption costs minutes instead of hours.
The section already says checkpoint commits are free and need no authorisation;
it does not say to use that.

**§11 describes how to invoke Codex and says nothing about surviving a failed
invocation.** Two capabilities of the CLI are unmentioned and directly relevant:

- `codex exec resume --last` continues an interrupted session rather than
  restarting it. Session files persist to disk by default. Resuming re-reads the
  prior turns as input, so it costs input tokens again — but the work is not
  re-derived, which is the expensive half.
- `codex exec -o <FILE>` writes the agent's last message to a file regardless of
  how the run ends, so an ungraceful termination still leaves its conclusion on
  disk.

§11.1 already insists on checking the invocation's own exit status, because a
review that died and a review that found nothing look identical. **An
interrupted build has the same shape and no equivalent instruction.**

### Candidate change

Not yet a proposal. Three parts, and the third is the one that needs an argument
rather than a sentence:

1. **§11 gains the recovery form.** State `resume --last` and `-o`, and that
   persistence is on by default. Small, factual, no rule attached.
2. **§12 gains a preference, written as one.** Commit often enough that an
   interruption loses minutes. Nothing can check this, so it is guidance and
   must say so.
3. **A task acquires a size.** The hard question. Options considered so far: a
   SPEC field estimating the share of a window the task will take; a rule that a
   task expected to exceed some fraction must be decomposed first; or nothing in
   the general layer at all, on the grounds that quota windows are a property of
   a subscription and belong in `PROJECT.md`. The third is the most likely to be
   right and the least satisfying.

### Why this is not urgent

The loss is smaller than it feels. Files written under `-s workspace-write` land
on disk as they go, and the session transcript persists, so an interruption
costs the input tokens to re-read and not the output tokens to re-derive.
Correcting that expectation may be worth more than any rule change here.

---

## F2 — Nothing checks that a proposal's claims match the release

**Hit:** 2026-09-21, while answering an unrelated question about a token-saving
tool.
**Against:** v3.0.0's own release documentation, and the process generally.
**Status:** closed in v3.2.0 for the instance; the gap itself is open.

### What happened

Proposal 002 listed change 5a — defined inputs for the §7 review — among what
shipped in v3.0.0. It did not ship. §7 was untouched by that release. The claim
sat in the repository for four days and was found by accident, while checking
something adjacent.

The two neighbouring claims in the same sentence, 5b and 5c, were both true.
That is what made it survive a reading: the list was mostly right.

### The gap

EVIDENCE has a Spec -> Test mapping precisely so that a claim cannot be made
without something behind it, and §6 warns that a mapping can overclaim. **The
baseline applies none of that to its own release documents.** A proposal says
what a release did; nothing compares that against the diff.

This is cheap to check by hand and was not checked. It is not obviously worth a
rule — the population is one repository and a handful of releases a month — but
it is worth knowing that the document describing a release is unverified.

### Candidate change

Probably none in the general layer; this is about how *this* repository works,
so it belongs in the root `CLAUDE.md` if anywhere. The smallest useful version
is a line in the release ritual: before tagging, read the proposal's list of
changes against `git diff <previous tag>..HEAD` and confirm each one appears.
