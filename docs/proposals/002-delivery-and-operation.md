# Proposal 002 — Delivery and operation

**Status:** applied to `template/` as **v3.0.0**. Not yet tagged or released.
**Raised:** 2026-09-17
**Supersedes:** `001-assurance-and-roles.md`
**Author:** Claude Code, from three adversarial reviews by Codex (see §Provenance).

---

## Why this replaces 001

Proposal 001 was aimed at the wrong target, and so was most of the baseline it
proposed to fix.

The baseline answers one question well: *does this change match the contract it
was approved against?* The operator's actual goal is a different one: **software
that is genuinely usable, deployable, and diagnosable in production.** Those are
not the same question, and the gap between them was not a gap in the rules — it
was a gap in what the rules were about.

A word search settled it. Across 9,000 words of development process:

| Term | Occurrences before v3.0.0 |
|---|---|
| `deploy` | 2 — both meaning "get authorisation first" |
| `logging`, `observability`, `telemetry` | 0 |
| `production`, `staging`, `rollback` | 0 |
| `incident`, `diagnos*` | 0 |
| `end-to-end`, `e2e`, `smoke` | 0 |

And none of the seven gauntlet layers executed the program under test.

This was already visible in the first review, which reported that "the expensive
failures occur between tasks, across deployed versions, or during recovery —
outside the specific contract each report verified." It was recorded as a rule to
fix. It was a statement that the aim was off.

## What shipped in v3.0.0

Eight changes to the general layer. It is a major release: the workflow gains a
step, the gauntlet's layer set changes, and `PROJECT.md` gains three required
sections that existing projects will receive as blanks through §14's reconcile.

### 1. A project profile, beside the Tier — §3

Two questions, and a project needs both answers. The **Tier** asks how dangerous
*this change* is. The **profile** asks what this service must protect *always*:
customer data, multi-tenancy, payments, uploads, model calls, public
availability. Each declared capability obliges a **standing check** that runs on
every release whatever the change was.

Deriving one from the other breaks both. A project holding customer data does not
make every edit Tier 3 — that inflation teaches people to stop reading the Tier.
And a one-line change to a retention setting can be high stakes in a project
whose profile looks quiet.

The point of a standing check is that **it is not a task's business**. Otherwise
every SPEC must rediscover tenant isolation, and one that forgot is
indistinguishable from one that had no tenants.

### 2. Real execution, as a gauntlet layer — §5

The layer that asks whether the software works. The other seven ask whether the
code is right, which does not imply it.

It must exercise the application as actually built and started. A request to a
health endpoint satisfies the letter of the layer and none of its purpose, so the
rule names a scenario from the SPEC and carries it end to end — and where the
only such scenario needs a person, says so rather than substituting one that
passes.

### 3. Observability as three lists, not one — §4

This corrects a defect in the first draft of this proposal, and it is the most
important item here.

Asking only *what does the operator see* invites the builder to record whatever
makes diagnosis easiest — for the products this baseline is being used on, that
could be body measurements, photographs, prompts or signed URLs. **A test
asserting those fields appear would make the dangerous behaviour contractual.**

So the SPEC writes three lists: **permitted observations** (an allowlist —
stage, error code, elapsed time, release identity, retry outcome, an opaque
reference), **prohibited data** (payloads, credentials, anything identifying a
person, by default), and **diagnostic limits** — what this design gives up.

That third list is the honest one. Some failures are not reproducible without the
input that caused them, and where the input may not be kept, the contract says so
rather than discovering it during an incident.

The check: exercise the failure with synthetic records carrying a recognisable
marker, then search every sink for the marker. Redaction after the data reaches a
collector is too late.

### 4. Release and operation — new §15

A merge changes a branch. Every guarantee in the rest of the file stops at the
edge of the repository.

- **Release identity.** The gauntlet runs against a source checkpoint; what
  reaches users is an artifact, and one tree built twice produces two. Build from
  the tree that passed, give it an immutable identifier, deploy *that*, and have
  the instance report it. Record configuration, schema, model, prompt and
  catalogue revisions beside it — any of those moves behaviour with no source
  change.
- **The traceability claim is bounded.** An identifier gets you to the software
  that was running and a bounded set of places to look. It does **not** identify
  the change that introduced the fault. The earlier draft promised that; it was
  cut.
- **Recovery.** "Redeploy the previous artifact" is not a plan once a release has
  changed anything that outlives a process. The check is a round trip: old
  writes, new reads and writes, old operates on the result. Exercise the
  mechanism at setup and when the mechanism changes — **not on every deployment**,
  which the earlier draft required and which buys little at constant cost. Where
  a release cannot be reversed at all, write a forward-recovery plan and do not
  label it `rollback tested`.
- **Checks that outlive the session.** Everything else in the baseline happens
  while an agent is working; the failures that reach users happen later. A
  user-shaped request that succeeds, deadlines on async work, recurring jobs that
  report when they did not run, consumption bounded by something that **stops**,
  and an alert channel tested end to end.
- **One pipeline.** The task and its release are one automated run, not two
  conversations and not two reports. An agent reads the record and speaks about
  exceptions; re-narrating a green run costs money on every task and adds nothing.
- **Recovery must not require an agent.** The human needs to deploy, roll back,
  disable a feature and read logs with neither subscription available — the day
  that matters is the day one is exhausted.

### 5. Components without a stable output — new §16

Three claims, kept apart, and never allowed to stand in for one another:
**deterministic guarantees** (validation of inputs *and of model output*,
authorisation, permitted identifiers, bounds, timeouts, retry ceilings — all
enforced outside the model), **statistical quality** (a versioned evaluation set,
declared threshold, sample counts, subgroup results), and **live integration**
(one bounded call, which establishes connectivity and nothing else).

Nondeterminism is not a reason to relax the deterministic half. Mutating a prompt
is an experiment, not a mutation test: there is no oracle.

Evaluation has provenance or it has nothing — and the repetition policy is fixed
before the run, because **retrying until it passes is not evaluation**.

The laundering this exists to stop: mapping a quality claim onto a schema check
plus one successful call turns *"we did not test this"* into a passing line.

One warning sits in the general layer rather than the project layer, because it
is not project-specific: **a reference set the agents wrote cannot establish that
the product serves its users.** Where a system makes claims about people — what
fits them, what suits them — the reference data must come from those people.
Agreement between two models is not evidence about anyone.

### 6. Degradation is about capability, not vendor — §0

What a `[dual-agent]` rule needs is a challenger: a different model, fresh
session, read-only. Losing one product does not remove the capability while
another model can supply it.

And the degradation no longer reaches Tier 3. Real funds, auth or data loss
proceed without an adversary only on a written human exception. A disclosure
field says what happened; an exception is someone deciding it may.

### 7. A contract gap can block acceptance — §11.3

Verification was satisfied when the code matched the contract; gaps in the
contract were logged and did not block. A verifier finding that an operation can
be replayed for real money therefore produced a *satisfactory* result.

Now such a finding blocks — with three qualifications that stop it becoming an
unlimited veto: a blocker needs a credible failure path, not an asserted
severity; disputes go to a recorded human decision; and **running out of review
budget yields `inconclusive`, not `satisfied`.** A human may merge over an
inconclusive result. That authorises the merge; it does not convert failed
assurance into successful assurance.

### 8. Cleanup demoted; mutation kept — §5, §9, §12

The earlier draft demoted both to Tier 3 on the grounds that both were expensive
and neither served the purpose. The review tested that and it was wrong in both
directions.

**Mutation stays.** It detects assertions that cannot tell correct behaviour from
plausible incorrect behaviour — "a recommendation test asserting a non-empty
result may survive a broken exclusion filter, and a successful browser journey
will pass too." There is no equally cheap substitute. Two corrections to the
earlier reasoning: its cost is mostly CPU and wall-clock rather than agent
tokens, and the baseline **already** scoped it to changed files, so re-proposing
that was not a saving. Real savings come from selected operators, sampled
mutants, and incremental runs.

**Cleanup goes further than proposed.** The cheap static half — unused imports,
exports, dead files — stays as a layer, renamed `Unused code`. The broad
deletion pass leaves the gate entirely, including Tier 3: it is expensive, it
changes behaviour, and the riskiest changes are the worst place to add unrelated
deletion. The obligation to remove superseded code survives as a design rule
(§9.6, §9.7), carried out during implementation and checked in review. §12's
mandatory checkpoint moves with it.

**The operator's condition was "as long as it does not hurt reliability". For
mutation, that condition was not met, and the demotion was dropped.**

## Carried over from 001

Shipped in v3.0.0: the Property layer is redefined as an executable check that
the required properties exist and ran (5b); `PROJECT.md`'s safety section joins
§10's authority list (5c); and cross-task invariants (4) are folded into §15
rather than standing alone.

**Correction, 2026-09-21.** This section originally also listed 5a — defined
inputs for the §7 diff review — among the changes carried over. **It was not in
v3.0.0.** §7 was untouched by that release, and the claim stood here unnoticed
for four days. It shipped in **v3.2.0**, and not in the form described: the §7
reviewer audits the builder's account, where §11.3's verifier attacks the work
without it, so the two take different input sets rather than the same one. The
shared part is the discipline — named, bounded, handed over — not the list.

The error is left visible rather than edited away. A release document that
claims a change it did not make is exactly what §6 asks EVIDENCE not to do, and
this one did it.

## Cut from 001, and not returning

| Was proposed | Why it is gone |
|---|---|
| **Role portability** (producer/challenger, assignable per task) | It solved review independence, which is not the binding constraint for delivering working software. The first version was rejected outright for dropping independent review of the SPEC before human approval. Shelved, not adopted. |
| **The four-arm experiment** (20 tasks, measuring assignment cost) | It measured a question that no longer matters. `docs/experiments/001-role-assignment-cost.md` is marked shelved. |
| **Comparable token consumption between the two agents** | The ratio can improve while the situation worsens, and the two accounts meter differently enough that equal tokens are not equal pressure. |
| **A traceability chain reaching "the change that introduced it"** | Overclaimed. Identification is not causation. |
| **A live rollback on every deployment** | Constant cost, little purchase. Exercise the mechanism at setup and when it changes. |
| **"The skeleton is never deleted"** | What must survive is the tested journey and the deploy capability, not the original implementation. And a skeleton is the first *approved* increment, not a route around the approval gate. |

## What this still does not do

Written here because a reader will otherwise assume otherwise.

- **It cannot establish that the software is usable.** Real execution proves the
  thing runs; standing checks prove it protects what it claimed. Neither
  establishes that a person in the target market can accomplish what they came
  for. That needs those people. Two agents agreeing on a screenshot is not
  evidence about anyone.
- **The general layer names categories; it supplies no checks.** Tenant
  isolation, retention, provider-event handling and consumption ceilings are
  written against a project's own architecture. `PROJECT.md` carries them, and a
  declared capability with no check is a standing blind spot that EVIDENCE
  repeats every time.
- **Nothing here has been run.** v3.0.0 is a documentation change. Every claim
  about what it costs or catches is untested until a project uses it.

## Before this is tagged

1. **`BOOTSTRAP.md` Step 6 is now in tension with itself** and deserves a read:
   it builds a slice for the toolchain to chew on, while the walking skeleton
   proper is a product increment needing a SPEC. The current wording says both;
   it may want splitting.
2. **§5's Tier 1 set is still two layers.** That is deliberate — a typo does not
   need the application booted, and the profile's availability check covers the
   deployed surface — but it is worth a second look now that Real execution
   exists.
3. **Existing projects get three new `PROJECT.md` sections** as blanks. That is
   §14's reconcile working as designed, but it is real work for whoever owns
   those projects.

## Provenance

Three reviews, all
`codex exec -m gpt-6-astra -c model_reasoning_effort=xhigh -s read-only`, run
from the repository root, exit status checked, never piped:

| Round | Scope | Tokens | Exit |
|---|---|---|---|
| 1 | Six open questions about the baseline, no proposal supplied | 76,872 | 0 |
| 2 | Six named attack points against a proposed role model — **rejected** | 50,543 | 0 |
| 3 | Six named attack points, with the operator's real products supplied | 110,350 | 0 |

Round 3 cost the most and was worth the most. The difference was not the
question; it was that the reviewer finally knew what the software was for.

**Honest note.** Claude Code wrote the critique, wrote all three prompts, judged
which findings to accept, applied the changes, and wrote this document. That is
the arrangement §11.1 describes, applied to the baseline itself, and the same
selection effect applies: the findings most likely to be missing are the ones
this author was confident about. No human approved any prompt before it ran. No
third model has seen any of it.
