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
**Status:** closed. The instance was fixed in v3.2.0; the gap was closed on
2026-09-29 by a pre-tag check in the root `CLAUDE.md` — no release, since that
file is not copied into projects.

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

---

## F3 — Nothing says whose rendering of the output EVIDENCE records

**Hit:** 2026-09-21, while assessing two token-compression tools against the
workstation's existing setup.
**Against:** v3.2.0, §5, §6, §11.1, §15.4, and `SETUP.md` §5.
**Status:** open.

### What happened

The workstation runs a shell proxy that compresses command output before the
agent sees it. The global Codex configuration imports its instructions, and
those say to prefix every shell command with it. Measured on this machine, it
removes 24% of output overall and 93% on the commands it handles best.

So the builder runs the gauntlet through a filter, and then writes EVIDENCE from
what it saw.

This is not an argument against the tool. Filtering what an agent *reads* is
sensible and it is why the tool exists. The problem is narrower: **the filtered
version can become the record.**

### The gap

§6 requires EVIDENCE to record, for each layer, "the command, where it ran, and
its output". **It does not say whose rendering of that output.** A compressed
retelling satisfies the sentence.

§11.1 already worries about something one step away: never wrap a `codex exec`
in a pipeline, because the pipeline replaces the invocation's exit status with
the last command's. **A filter between a command and the agent is the same shape
of problem one level up** — it does not replace the status, it replaces the
content, and the document has a rule for the first and nothing for the second.

§15.4 resolves it, and only where it applies: CI builds, runs the layers, and
writes the record; the agent reads that record and speaks about exceptions. Where
that pipeline exists, what the agent read through a filter never becomes the
evidence. **Before it exists — throughout `BOOTSTRAP.md` Steps 1 to 8, and in any
project that never wires CI — the gap is open**, and those are exactly the
moments when the toolchain is least trustworthy.

Whether this particular filter drops anything a reader of EVIDENCE would want is
untested. That is the point: nothing in the baseline asks.

### Candidate change

Not a rule about any named tool — the general layer names no tools, and this
applies equally to anything that compresses, proxies or summarises on the way to
the agent.

1. **§6 gains a clause.** The output EVIDENCE records is the command's own. Where
   something sits between the command and the agent, either the gauntlet is
   exempted from it or EVIDENCE says it was in force. Cheap, and it makes the
   existing sentence mean what it was always taken to mean.
2. **`SETUP.md` §5 gains an environment note.** That section already carries "a
   missing CLI can report success" and "a model at capacity reports it at the
   end of the run" — this belongs beside them, as a thing about the world rather
   than a rule.
3. **Nothing else.** The practical fix is a project decision and belongs in
   `PROJECT.md`'s gauntlet rows: run those commands unfiltered. Proxies of this
   kind generally offer a pass-through mode for exactly this.

### Why this is worth recording rather than fixing now

The tension is real but currently harmless here: PC-Desk has no CI yet and no
gauntlet has run through the filter. It becomes live the first time a layer's
output is compressed on the way into an evidence report — which will be soon.

---

## F4 — A wrapper can turn a failing layer into a passing one

**Hit:** 2026-09-21, while testing whether a shell proxy loses information — a
different question, which it answered by raising this one.
**Against:** v3.2.0 §5, §11.1; `BOOTSTRAP.md` Step 6; and the workstation's
global agent instructions.
**Status:** open, and live on this workstation now.

### What happened

The workstation's shell proxy is imported by the global Codex instructions,
which say to prefix every shell command with it. Four invocations, measured:

| Invocation | Raw exit | Through the proxy |
|---|---|---|
| `git log <nonexistent-ref>` | 128 | **128** — preserved |
| `proxy bash -c 'echo boom; exit 1'` | 1 | **1** — preserved |
| `test bash -c 'echo "1 failed"; exit 1'` | 1 | **0** — lost |
| `err bash -c 'cat f; exit 1'`, where `f` contains `ERROR: something broke` | 1 | **0** — lost |

The last one does not merely lose the status. It prints:

```
[ok] Command completed successfully (no errors)
```

Both failing forms are the tool's documented usage — its help gives
`test [COMMAND]...` and `err [COMMAND]...` with exactly this shape.

One caveat, stated because it is not resolved: the `test` form produced no
stdout at all, so it is unclear whether it ran the command or failed to
recognise it. The exit status was 0 for a failing command either way, which is
the part that matters.

### The gap

**§5 says a layer must be able to fail.** A wrapper that returns 0 on failure
makes the layer unable to fail, and nothing in §5 contemplates a wrapper
existing. The layer's command is recorded in `PROJECT.md`; whether something
sits around it when it actually runs is not.

**§11.1 warns about exactly this hazard in a different shape.** It says never to
wrap a `codex exec` in a pipeline, because the pipeline replaces the
invocation's exit status with the last command's. A wrapper does the same thing
from the other side. The document has a rule for one and nothing for the other,
and the reasoning it already gives — *a review that died and a review that found
nothing look identical* — transfers without modification.

**The hazard is default-on wherever an instruction says "always prefix".** It
does not require anyone to make a mistake.

### What already covers this, and why that matters

v3.1.0's headline finding was that a gauntlet layer can be configured,
installed, invoked, and completely inert — a mutation runner activating no
mutant still reports a score. Its remedy: **see each layer fail on purpose once
before recording it as working.**

That remedy catches F4. It is the same failure with a different mechanism, and
the fix already in the document is the right one. This finding is therefore less
about a missing rule than about a rule whose scope needs saying out loud.

### Candidate change

1. **§11.1's pipeline warning generalises.** Anything standing between a command
   and its reported status — a pipeline, a wrapper, a proxy, a runner — can
   replace the signal. Say it as the general case rather than as one instance,
   and keep the reasoning already there.
2. **The "see it fail once" check applies to the invocation, not the tool.**
   Watching `pytest` fail proves nothing about `<wrapper> pytest`. What must be
   seen failing is the exact string `PROJECT.md` records, wrapper included. This
   is the sharp half of this finding and belongs beside the existing remedy in
   `BOOTSTRAP.md` Step 6 and `SETUP.md` §4.
3. **Nothing about any named tool.** The general layer names no tools, and this
   applies to any wrapper. Where a project uses one, `PROJECT.md`'s gauntlet rows
   record the full invocation, and the project decides which commands are
   exempt — most such proxies offer a pass-through mode, and the one here does.

### Relationship to F3

F3 is about fidelity: the output EVIDENCE records may be a compressed rendering
of the real one. F4 is about the gate: the layer can report success on failure.
Different mechanisms, different severity. F3 degrades a record; F4 removes a
check. They share a cause — something sits between a command and the agent — and
a project that exempts its gauntlet from the wrapper closes both.

---

## F5 — Step 2 does two jobs with different cost structures

**Hit:** 2026-09-29, while reading two AI-written workflow proposals against the
baseline. Both proposed skipping the plan review for low-risk tasks; neither had
the baseline in front of it.
**Against:** v3.2.0, §2 step 2, §3, §11.2.
**Status:** open. **Priority undetermined** — see §What decides the priority.

### What the text says

§2 conditions two of its steps on the Tier: step 0 (`Tier 3 or on request`) and
step 7 (`Tier 3`). **Step 2 carries no condition at all**, only `[dual-agent]`.
Read literally, every task gets a feasibility review — a Tier 1 typo included.

§11.2 then sets that review's effort *independently of the Tier*, "at or above
the highest effort any builder row uses". So the literal reading is that every
task, trivial ones included, receives a read-only review at the highest effort
the project configures, from the builder's vendor.

### The gap

**Step 2 fuses two jobs whose costs have nothing in common.**

- **Feasibility review.** Can this SPEC be built as written; what does it miss;
  is the design sound. Worth the highest effort — but only where there is a
  design to examine.
- **Tier confirmation.** Is the proposed Tier right. Every task needs this, and
  it is cheap: it asks one question about consequence, not many about design.

§11.2 explains why the review's effort is not derived from the Tier, and the
reason is entirely about the second job: "raising the Tier is one of the things
the review exists to do". It then applies the effort that justification
requires to both jobs at once, on every task.

### The fix that must not be made

**"Skip step 2 for Tier 1" removes the only check on a Tier 1 that was wrongly
proposed** — and a wrongly-proposed Tier 1 is exactly where the risk hides. A
"config value" change is one of Tier 1's own listed examples, and the third
review of this repository put it plainly: *a one-line storage-policy or logging
change can be high stakes.* The profile's standing checks (§3.1) catch some of
this on release; they do not catch it before the SPEC is approved.

The ratchet does not cover the gap either. §3 lets Codex raise a Tier at any
point, including during implementation — but by then the human has approved a
SPEC written at the wrong Tier, with the wrong layer set and no failure model,
and raising it means reopening that approval. The review exists to catch it
before that happens.

This is recorded because the naive fix was just proposed twice, independently,
and the argument against it existed only in one conversation.

### Candidate change

Split the step rather than skip it.

1. **Every task: a tier confirmation.** One question — is the proposed Tier
   right — at low effort. It keeps §11.2's reason for existing, and costs a
   fraction of a review.
2. **Tier 2 and 3: the full feasibility review**, at the effort §11.2 already
   sets.
3. **A confirmation that raises the Tier promotes the task** into the full
   review. The cheap check is a gate into the expensive one, not a replacement
   for it.

Minor if adopted: a step gains a condition and a lighter variant, and
`PROJECT.md` gains one effort row for the confirmation. No existing project needs
rework.

### What decides the priority

Nothing here is measured. It is not known whether feasibility reviews are 5% of
the builder's spend on this workstation or 30%, and the answer decides whether
this is worth a release or merely worth knowing.

The measurement is cheap and already available: record `tokens used` from each
`codex exec` separately by step — feasibility review, implementation, EVIDENCE —
across a handful of real tasks, alongside each task's Tier. If reviews of Tier 1
tasks are a material share, this moves to a proposal. If they are not, it stays
recorded here for the next time someone suggests skipping the review.
