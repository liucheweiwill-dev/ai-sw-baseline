# Proposal 001 — Assurance boundaries and role portability

> **SUPERSEDED by [002-delivery-and-operation.md](002-delivery-and-operation.md)
> on 2026-09-17.** Kept because its rejected material is recorded here and should
> not be re-proposed.
>
> Changes 2, 3, 5 and part of 4 shipped in v3.0.0, restated in 002. **Change 1
> (role portability) was not adopted**: it solved review independence, which is
> not the binding constraint for delivering working software. Its demotion of
> mutation to Tier 3 was tested and **rejected** — see 002 §8.

**Status:** proposed, not adopted. Nothing here has changed `template/`.
**Raised:** 2026-09-17
**Author:** Claude Code, from two adversarial reviews by Codex (see §Provenance).

This repository had no convention for proposals before this file. `docs/proposals/`
and `docs/experiments/` are new here. They describe this repository and are never
copied into a project.

---

## What this is

Five changes to the general layer, plus one convention that needs no document
edit. Four of the five are minor. The fifth is major and is deliberately held
back, because it should not ship before the experiment in
`docs/experiments/001-role-assignment-cost.md` has run.

**This is not a jointly agreed document.** Codex reviewed the current baseline,
then reviewed a proposed replacement for its role model, and **rejected that
replacement**. What follows is the synthesis: the current design, plus the
findings that survived judgement. The rejected material is recorded in
§Rejected so that it is not re-proposed.

## Version verdict

| Change | Touches | Version |
|---|---|---|
| 2. Degradation by capability, not vendor | §0 | minor |
| 3. Stopping condition admits contract gaps | §11.3 | minor |
| 4. Cross-task invariants | §5, §13, §14, PROJECT.md, SETUP.md §4 | minor |
| 5. Three corrections of record | §5, §7, §10, §14, SETUP.md §4 | minor |
| 1. Role portability | §0, §1, §2, §7, §11, template/CLAUDE.md | **major** |

Recommended sequencing: land 2–5 as one **v2.9.0**; hold 1 for **v3.0.0** pending
the experiment. Every version bump gets a tag (root `CLAUDE.md`).

---

## Change 2 — Degradation is about capability, not about a vendor

**Defect.** §0 drops every `[dual-agent]` rule when one agent is unavailable.
Those rules include the feasibility review (§2 step 2), the double-track review
(§7) and Tier 3 independent verification (§11.3). A Tier 3 task — real funds,
auth, data loss — therefore proceeds with no adversarial check of any kind, and
EVIDENCE merely says so.

The abstraction is wrong. Losing one agent *product* does not make a fresh
reviewer on another model unavailable. §0 discards the whole category when only
one supplier of it went away.

**Change.** Replace vendor availability with a capability requirement: a
different model, a fresh session, read-only. Any product satisfying that supplies
it. Where no second model is available at all, high-consequence work waits, or a
human records a written exception naming what is missing. An EVIDENCE disclosure
field is not that exception.

**Existing projects.** No rework. A project already running single-agent records
the exception once.

**Provenance.** Codex round 1, finding 2. Accepted in full; sharper than the
version this repository's own review had reached.

## Change 3 — A contract gap can block acceptance

**Defect.** §11.3 declares verification satisfied when a round finds no
divergence between the code and the approved contract. Contract-completeness
findings are logged and triaged by a human, and do not block. So a verifier that
discovers replayable refunds produces a *satisfactory* verification result,
because the SPEC never mentioned replay.

**Change.** An unresolved high-consequence finding blocks acceptance whether it
originates in the code or in the contract. Three qualifications, all required:

- A blocker needs a **credible failure path** or an identifiable unmet safety
  obligation. Severity asserted without a path is not a blocker; otherwise the
  challenger holds an unlimited veto.
- Disputes about applicability, severity and acceptable residual risk go to an
  explicit human decision, recorded.
- **Exhausting the review budget means `inconclusive`, not `accepted`.** A human
  may authorise a merge over an inconclusive result; that does not convert failed
  assurance into successful assurance.

§11.3's existing requirement to fix the stopping rule *before* the round runs
stays exactly as it is.

**Existing projects.** No rework. EVIDENCE gains `inconclusive` as a third
outcome alongside satisfied and divergent.

**Provenance.** Raised by this repository's own review; the unbounded-veto flaw
and the `inconclusive` outcome are Codex round 2, finding 5.

## Change 4 — Assurance that outlives a task

**Defect.** The unit of assurance is a task; the expensive failures belong to the
evolving system. Two tasks can each satisfy their own failure model while
introducing an interaction neither modelled. A checkpoint SHA identifies what was
examined; it does not establish that two approved changes compose.

**Change.** One durable file of system-level invariants, and one gauntlet layer
that executes it. A task may add to it.

The protected object is **the obligation's meaning and the check that enforces
it** — not the sentence. Human authorisation is required to delete, narrow, skip,
or stop collecting an invariant. "No removal" alone protects nothing: the line
can stay while its fixture excludes the case that mattered.

Align with what already exists rather than duplicating it. `ARCHITECTURE.md` plus
its deterministic check already carry durable *structural* constraints. This
layer is for durable *behavioural* invariants, and must not restate structural
ones under a competing definition.

**Existing projects.** Adds a `PROJECT.md` row, which §14's reconcile delivers as
a `<FILL IN>`. A project with no such invariants yet writes `none` and a reason.

**Provenance.** Codex round 1, finding 3; the meaning-not-presence correction and
the no-duplication warning are Codex round 2, finding 8.

## Change 5 — Three corrections of record

Each is small and independent.

### 5a. §7's diff review gets defined inputs

§11.3 hands the Tier 3 verifier four inputs, assembled verbatim, and the reason
is independence: exclude the builder's persuasive account, supply the actual
contract and a checkable source state. §7's ordinary double-track review has no
input rule at all, so the reviewer re-reads whatever it likes and anchors on
whatever it reads first.

Give §7 the same treatment, for the same reason. A measured side effect is that
it is cheaper — see the experiment's §Prior evidence.

This also revisits §7's mandatory EVIDENCE-first order. The stated reason is
sound: the mapping says where to look. The cost is that the reviewer reads the
builder's account before inspecting the work. Proposed: read the diff against the
SPEC first, then EVIDENCE, then return to the gaps EVIDENCE names. If that is not
adopted, the order should become a preference rather than a mandate.

### 5b. The Property layer's definition

§5 requires every layer to be an executable check with a machine-evaluable
result. `SETUP.md` §4 then defines the Property layer, in all six languages, as
"the suite contains properties for the SPEC's invariants" — a human judgement.

The correct diagnosis is narrower than "this layer cannot fail". Property tests
*are* executable and *do* fail. What is missing is an executable check that the
required properties **exist and ran**. Define the layer as that check. Whether a
property adequately expresses a SPEC invariant remains a review judgement, and
should say so.

Note also that `BOOTSTRAP.md` Step 6 creates a scaffolding property and tells the
first task to delete it. That proves the toolchain can run a property once. It
establishes nothing about continuing coverage.

### 5c. `PROJECT.md` has no authority, yet carries safety rules

§14 requires a `Project-specific safety` field. §10 lists three artifacts that
carry authority — `AGENTS.md`, `CLAUDE.md`, an approved SPEC — and `PROJECT.md`
is not among them. A project therefore writes a safety rule into a file that §10
classifies as data.

Add `PROJECT.md` to §10's list, scoped to its safety section.

The related claim that `PROJECT.md` carries "no instructions, only answers"
survives for the rest of the file. But the automatic rejection of explanatory
prose in `BOOTSTRAP.md` §Updating step 2 over-reaches: a project-specific
*rationale* can stay correct across many releases, and its presence is not
evidence of drift. Narrow that instruction to baseline guidance, which is what
actually freezes.

**Existing projects.** 5a and 5b change how a layer and a review are run; no
rework. 5c is a one-line addition to an authority list.

**Provenance.** 5a: this repository's own review, plus Codex round 1 finding 1 on
anchoring. 5b: Codex round 1 finding 5, correcting this repository's own
diagnosis. 5c: Codex round 1, finding 8.

---

## Change 1 — Role portability (major; held)

**Defect.** §1 assigns roles by vendor, and `template/CLAUDE.md` forbids the
architect from writing feature code. The result is that one agent defines the
solution, commissions the review of it, disposes of the findings, and judges the
implementation. §11.1 already identifies half of this and mitigates it with an
honest prompt and a disclosure. The mitigation does not remove the selection
effect: a prompt that says *attack what I am least sure of* leaves the author's
confident mistakes least examined.

"Never writes feature code" does not create independence. It creates a handoff.

**Change.** Define the two roles by the artifacts they own rather than by vendor:

| | Producer | Challenger |
|---|---|---|
| Before approval | The SPEC and the solution | An independent derivation of failure cases; review of the SPEC, its failure mappings and its declines |
| After approval | Implementation, ordinary tests, gauntlet runs | Adversarial tests and probes |
| At acceptance | Reproducible results | The acceptance judgement; unresolved findings reported to the human alongside the producer's dispositions |

Either agent may hold either role. Three constraints:

- **No mandatory alternation.** Assignment is per task, capacity-aware, and
  recorded. Round-robin has no demonstrated benefit and may cost more than it
  saves.
- **The feature-code prohibition becomes task-scoped, not vendor-scoped.** A
  challenger may write executable code. It may not repair the implementation it
  judges: doing so makes it a contributor to the solution it accepts.
- **A challenger's own passing tests do not authenticate its own failure model.**
  The adversarial tests and their oracles are themselves subject to scrutiny.

**What this must not discard.** The replacement Codex rejected dropped several
controls. Any adopted version keeps all of these:

- **Independent review of the SPEC before human approval** (§2 step 2). Attacks
  that run after implementation arrive after the human has already approved a
  design nobody independently examined. This was the worst fault of the rejected
  draft.
- **Tier 1's abbreviated path** (§5 for its layer set, §6 for its report). A
  ten-step ceremony for a typo is the over-tiering this proposal elsewhere
  tries to cure.
- **Review effort set independently of the proposed Tier** (§11.2). Deriving it
  from the Tier lets a low initial estimate weaken the review whose job is to
  correct it.
- **§11.3's actual blindness design**: exclude the builder's *account*, supply
  the *evidence*. Withholding system context does not produce independence; it
  produces a generic failure list.

**Existing projects.** Rework. §1, §2, §7, §11 and `template/CLAUDE.md` all
change; `PROJECT.md`'s Agent models rows are re-labelled. This is why it is major
and why it is held.

**Provenance.** Codex round 1 finding 4 proposed the producer/challenger split.
Codex round 2 findings 1, 2, 4 and 7 cut the rotation mandate, the solution-blind
derivation, and the unscoped removal of the code prohibition.

---

## Convention change — no document edit

`PROJECT.md`'s Agent models table already records model and effort **per role and
per tier** (feasibility reviewer, builder tiers, verifier). The vendor coupling
lives in §1 and §11's prose, not in the form. An earlier draft of this proposal
claimed otherwise and was wrong.

What is missing is per-task recording of what was **actually** used: product,
model, effort, sandbox, fallback. Without it a reassignment can silently change
the strength or the execution conditions of a review while the record still says
`challenger`. This is a `development-status.md` convention, not a rule the
general layer needs.

**Provenance.** Codex round 2, finding 10.

---

## Rejected

Recorded so that they are not re-proposed.

| Proposed | Why cut |
|---|---|
| Mandatory role alternation every task | Redistributes cost without reducing it; forces context reconstruction, which is the expensive operation it was meant to avoid; alternating *task counts* does not balance *work*, since one task can consume a whole quota window |
| Failure derivation from the need alone | Confuses independence with deprivation of evidence. Excluding trust boundaries, topology and past incidents yields a generic list. And an agent that produced yesterday's design does not become ignorant of it by changing role |
| Reasoning effort set by estimated difficulty | Backwards for assurance. A one-line authorisation change is easy to implement and needs heavy scrutiny. Difficulty may govern the producer's effort; the challenger's must track consequence |
| Consequence-only tiering, structural triggers removed | A "display-only" task can still touch a shared serialisation path. Structural facts stop *automatically raising* the tier but remain mandatory prompts to reassess consequence, and the upward ratchet stays |
| Deleting §5's "all seven layers" mandate | Removes the forcing function and reopens per-project negotiation about what to skip. §5's three states already record honest gaps. Fix the Property definition instead (5b) |
| Deleting §10's "never modify a test to make it pass" | The rule is too absolute — a test can encode a mistaken expectation — but §10 is a safety boundary. Scope it instead: not within the task that made it fail; changing an expectation is a SPEC revision |
| Approving need, failure list, SPEC and declines as one object | Four artifacts do not become one comprehensible decision. The dangerous case is a failure marked *covered* that the SPEC addresses incompletely. Keep §4's revision binding across the whole bundle: a changed decline invalidates approval even when the SPEC text did not change |
| Comparable token consumption as a success criterion | The ratio can improve while the situation worsens — equally consistent with moving work to spare capacity and with making both agents expensive. Demoted to a diagnostic; the experiment says what replaces it |

---

## Open decisions

1. **Change 1 is yours, not a review's.** It is major, and the experiment measures
   cost and throughput only — it cannot tell you whether the assurance is better.
   That judgement has to be made on the argument.
2. **Whether 5a changes §7's EVIDENCE-first order, or only softens it.**
3. **Whether the comprehension arm of the experiment runs at all** — it tests the
   operator, not the process.

---

## Provenance

Two reviews, both
`codex exec -m gpt-6-astra -c model_reasoning_effort=xhigh -s read-only`, run
from the repository root, exit status checked, not piped:

| Round | Scope | Tokens | Exit |
|---|---|---|---|
| 1 | Six open questions, no proposal supplied | 76,872 | 0 |
| 2 | Six named attack points against a stated proposal | 50,543 | 0 |

**Honest note.** Claude Code authored this repository's own critique, wrote both
review prompts, judged which findings to accept, and wrote this proposal. That is
the arrangement §11.1 describes, applied to the baseline itself, and the same
selection effect applies: the findings most likely to be missing are the ones
this author is confident about. No human approved either prompt before it ran. No
third model has seen any of it.
