# Experiment 001 — What role assignment costs

> **SHELVED on 2026-09-17, unrun.** It measures the cost of assigning roles
> between two agents. That question was dropped when the baseline's aim moved
> from contract assurance to delivering working software
> ([proposal 002](../proposals/002-delivery-and-operation.md)).
>
> Two things in it outlive the question and are worth taking if a cost experiment
> is ever run: the measurement mechanics in §Measurement, which were verified
> before being written down, and the rule that **a close result is the result**.

**Status:** designed, not run.
**Raised:** 2026-09-17
**Gates:** Change 1 of `docs/proposals/001-assurance-and-roles.md`.

---

## The question

Does defining roles by artifact rather than by vendor — and optionally moving
them between agents — change **total cost** and **time blocked on quota**,
compared with the fixed architect/builder split in force today?

## What this cannot answer

Say this before reading any result, not after.

- **It says nothing about safety.** A handful of clean tasks cannot establish the
  rate of rare failures. If the proposed roles catch a class of defect the
  current split misses, this experiment will not see it.
- **It says nothing about assurance quality.** "Accepted" here means the gauntlet
  passed and the operator merged, which is the thing under test, not an
  independent measure of it.
- **It is one operator on one codebase.** No blinding is possible: the operator
  knows which condition is running and has an opinion about which should win.

Change 1 must therefore be decided on the argument. This experiment can only
remove one objection to it — that it costs more — or confirm that objection.

---

## The objective this replaces

The original goal was *roughly comparable token consumption between the two
agents*. That is cut, for a reason worth keeping written down:

> Equal consumption can mean moving useful work to spare capacity. It can equally
> mean making both agents expensive. Raw token equality says nothing about which
> occurred.

There is a second reason. The two accounts meter differently — different window
lengths, different reset points, model-specific caps on one side and not the
other — so equal tokens are not equal quota pressure. A balanced ratio is
therefore compatible with being blocked on both vendors at once.

**Replaced by:** useful work accepted within the operator's quota, time and
spend, at a stated assurance level. Consumption ratio is retained as a
**diagnostic**, never as a success criterion.

---

## Conditions

Four arms. B1 and B2 exist to separate *the cost of changing responsibilities*
from *the cost of moving them*.

| Arm | Roles | Assignment |
|---|---|---|
| **A** | Current: architect / builder (§1) | Fixed. Claude architect, Codex builder |
| **B1** | Proposed: producer / challenger | Fixed. Claude producer, Codex challenger |
| **B2** | Proposed: producer / challenger | Fixed. Codex producer, Claude challenger |
| **C** | Proposed: producer / challenger | Alternating each task |

Quality gates are identical across all four: the same gauntlet rows from
`PROJECT.md`, the same tier rules, the same human gates. Only the role
definitions and the assignment change.

If only three arms are affordable, drop **C** — B1 vs B2 already answers the
question that matters most, which is whether either orientation is cheaper.

---

## Measurement

Both mechanisms below were verified on 2026-09-17 before this document was
written. Do not substitute an unverified one.

### Codex side

`codex exec` prints its total to **stderr**, last lines, as `tokens used` and a
number. Capture it without a pipeline, and check the invocation's own exit status
— a run that died and a run that found nothing look identical (`AGENTS.md` §11.1,
`SETUP.md` §5):

```bash
codex exec -m <model> -c model_reasoning_effort=<effort> -s <sandbox> "$(cat prompt.txt)" > out.txt 2> err.txt
```

Then read `$?`, and `tail -4 err.txt` for the token count. Redirection preserves
the exit status; a pipe does not.

### Claude side

Claude Code reports plan limits and session context through its own session tool,
read in-session rather than from a shell. It returns, for the account:

- each window's label, **percent used**, and reset time (5-hour and weekly)
- extra-usage spend against its monthly cap

and for the session: `tokensUsed` against the context window.

**Record percent-of-window, not only raw tokens.** That is the number that
actually predicts being blocked, and it is the only one comparable across two
vendors that meter differently.

### Protocol per task

Take a reading on both sides immediately before the task starts and immediately
after it is accepted or abandoned. Record the delta. Note any reset that falls
inside the task — a window that rolled over mid-task makes that task's percent
delta meaningless, and the task should be marked and excluded from the quota
figures while still counting for the others.

---

## What to record, per task

| Field | Notes |
|---|---|
| Task id, arm, tier | |
| Role assignment | which product held which role |
| Model, effort, sandbox — **actual, per role** | not the `PROJECT.md` default. A reassignment that silently changes review strength is the failure this field exists to catch |
| Session reuse | fresh session or continued; how many |
| Codex tokens | from `tail err.txt`, per invocation, summed |
| Claude tokens | session `tokensUsed` delta |
| Cached vs uncached input | where the surface exposes it; blank where it does not |
| Retries and rounds | review rounds, failed gauntlet runs, re-approvals |
| Quota: % of each window consumed | per side, per window |
| **Quota-blocked minutes** | wall time waiting on a limit, not working |
| Elapsed wall time | start to acceptance |
| **Human minutes** | reading, approving, deciding disputes |
| Outcome | accepted / inconclusive / abandoned |
| **Defects found after acceptance** | filled in later; the field stays open |

Keep it as one row per task in a single file. A spreadsheet is fine. The point is
that it exists before the first task, not that it is elegant.

---

## Metrics

**Primary**

1. Quota-blocked minutes per accepted task.
2. Accepted tasks per calendar week.

**Secondary**

3. Total consumption, both sides, per accepted task.
4. Percent of each account's weekly window per accepted task.
5. Human minutes per accepted task.
6. Defects discovered after acceptance, per accepted task.

**Diagnostic only — never a success criterion**

7. Consumption ratio between the two agents.
8. Attribution: repository reading / reasoning / implementation / review /
   repeated rounds. Record command execution time separately from model
   consumption; a slow test suite is not a token cost.

A result where consumption rises but metric 1 falls substantially is a **win**,
not a loss. More total spend is worth it if it buys throughput. That tradeoff has
to be stated now, before the numbers exist.

---

## Task selection

- **Minimum 5 tasks per arm.** Below that, do not compute a mean; report the
  individual tasks and say the sample cannot support more.
- Comparable groups: similar tier, similar file counts, similar unfamiliarity.
  Do not put all the greenfield work in one arm.
- Interleave arms rather than running each to completion. A block design
  confounds the arm with everything else that changed that week.
- Exclude tasks abandoned for reasons unrelated to the experiment, and say how
  many were excluded and why.

---

## Stopping rule — set now, before any data exists

Fixed in advance so that it is a rule and not a preference chosen after reading
the numbers (`AGENTS.md` §11.3 applies the same discipline to verification):

- Stop at **20 accepted tasks total**, or **4 calendar weeks**, whichever comes
  first.
- Stop early if any arm produces two consecutive tasks that are abandoned for
  process reasons — that arm is unworkable and the reason is the finding.
- Do not extend an arm because the result is close. A close result **is** the
  result: it means the assignment does not matter much, which settles Change 1's
  cost objection just as well as a clear win.

---

## Optional arm — operator comprehension

**Run this only if you want it.** It tests the operator, not the process, and
that is a different kind of exercise.

The concern it addresses: bundled approval can be approved faster and understood
less. Approval time alone cannot tell those apart — faster approval is exactly
what rubber-stamping looks like.

Method: in a small number of review packets, inject one deliberate inconsistency
— a failure marked *covered* in the reconciliation that the SPEC addresses only
partially. Record whether it was caught before approval, and how long approval
took.

Caveats, both real:

- Injecting a defect into a real task risks merging it. Either use retired tasks,
  or have a second party place and track the injection.
- Knowing that injections happen changes how the packets are read. The measure
  degrades after the first few.

---

## Prior evidence

Two data points already exist, from the reviews that produced Proposal 001. Same
model, same effort, same sandbox, same repository:

| Round | Scope | Tokens |
|---|---|---|
| 1 | Six open questions, no proposal supplied | 76,872 |
| 2 | Six named attack points against a stated proposal | 50,543 |

Round 2 received **more** input and cost **34% less**. That is one pair of
observations, not a finding — but it is the direct evidence behind Change 5a, and
it points at something this experiment should watch for: constraining a
reviewer's inputs and question may matter more to cost than which agent holds the
role.

If that holds, the cheapest available change is not reassignment at all. It is
telling the challenger exactly what to look at.

---

## Reporting

Report all four arms including the ones that lost, every excluded task with its
reason, and the metrics that did not move. State the sample size beside every
mean. Where an arm was not run, say so rather than omitting the row.

Then answer one question in one sentence: **does cost still object to Change 1?**
Everything else in that change is decided on the argument, not here.
