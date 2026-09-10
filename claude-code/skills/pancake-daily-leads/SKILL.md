---
name: pancake-daily-leads
description: "Playbook: find N leads today in a Pancake workspace under a stated credit ceiling — read the balance, preview the plan, confirm, start it, poll, and report what was found and rejected."
---

# Find N leads a day under X credits

Use this playbook when the user asks for leads today under a budget — "find me 10 leads for
under 3,000 credits", "run lead finding, keep it under 2k". It runs the same waterfall Pancake's
scheduler runs overnight, on demand, and never spends more than the number the user agreed to.
It needs the `pancake` skill's conventions and a connected workspace; every step is an MCP tool
call and nothing here needs the Pancake app.

Inputs: **N** (qualified leads to aim for, 1–50; default 10) and **X** (the credit envelope).
If the user gave only one, ask for the other before spending anything.

## 1. Read what the workspace can spend — free

1. `credits_get_balance` — the billing period's `allowanceCredits`, `heldCredits`, `settledCredits`,
   `availableCredits`, and `enforcementMode`.
2. `lead_finding_get_spend` — today's and this month's ceiling (`remainingCredits`, which window
   is `binding`, when it resets), this connection's own allowance under `connection`, whether
   agent starts are paused, and `limits.startsRemainingToday` / `concurrentRemaining`.
3. `lead_finding_list_runs` with `{"limit": 5}` — if any run is `pending` or `running`, stop here:
   never start a second plan while one is in flight. Tell the user which run is running and offer
   to poll it instead.

Decide the envelope you will ask for, `E = min(X, spend.remainingCredits, the connection's
remaining allowance if one is set)`. In `enforce` mode the ceilings bind, so `E` is what could
actually run. In `shadow` mode nothing is refused and balances may go negative — `E` still stays
at or below X because X is the user's number, not the ceiling's. If `enforcementMode` is `off`,
credits are not tracked on this deployment: say so, and treat X as the only rail.

If `agentStartsPaused` is true or `startsRemainingToday` is 0, report it and stop; only a member
can unpause in Settings.

## 2. Preview the plan — free

`lead_finding_preview_plan` with `{"credits": E, "target": N}` (add `geographies` or a `scope`
only if the user asked for them). Read back:

- `stages` — for post-engagement, company-signal, and persona-sweep: the credits `planned`, the
  `expectedLeads` (p25/p50/p80), `creditsPerLead`, and a `basis` saying whether the estimate comes
  from this workspace's own runs or a fleet aggregate; a stage with `skipped: "target_met"` gets
  nothing. `expectedLeadsTotal` and `unallocatedCredits` sum it up.
- `runnable.credits` — what the current enforcement mode would let through (with a `reason` and
  `resumesAt` when it is less than you asked). Use it, not the raw ceiling.
- `nextStep`, and anything else it warns about (an unusable Brain, a paused connection).

If the expected leads fall clearly short of N, say so and offer either a larger E (never above X)
or a smaller N. Do not silently raise the envelope.

## 3. State the spend and get a yes

Before starting, tell the user in one message:

> I will start a `<scope>` plan aiming for **N leads** with an envelope of **E credits**
> (planned: post-engagement A / company-signal B / persona-sweep C). Expected leads ≈ L (p50).
> Enforcement is `<mode>`; the workspace has V credits available this period and the ceiling
> allows R more today. Nothing has been spent yet. Start it?

Wait for an explicit yes. A previous confirmation for a different envelope does not carry over.

## 4. Start and poll

`lead_finding_start_plan` with exactly the arguments the user confirmed. It returns the head
`runId` at once. Then `lead_finding_get_run` with that `runId` every 1–2 minutes until `status`
is `published` or `failed` — `pending` and `running` both mean wait, never that something is
stuck. A waterfall runs its stages one after another under the same envelope; the head run's
`report.chain` names the hops. Do not start anything else meanwhile. If the user asks to stop,
`lead_finding_cancel_run` keeps the leads found so far.

## 5. Report the result

From the final `lead_finding_get_run` (add `include: "rejected"` for the retired people):

1. **Found**: how many leads qualified against N, and the first page of people — name, title,
   company, `fit`, and the signal that surfaced them.
2. **Spent**: `report.credits` — what the ledger charged the run and its hops against E. Say
   plainly whether it stayed inside E (it must; if the ledger shows more, report that as a
   discrepancy rather than explaining it away).
3. **Rejected and why**: `report.rejections` — the counts per reason with their `meaning` and a
   few named examples. A wall of hard vetoes means the sources are off; a wall of low ICP scores
   means the bar is high; enrichment or judge failures are a provider issue.
4. **Stopped**: `report.stopped` — `budget`, `deadline`, `lead_limit`, `sources_dry`, `error`, or
   `cancelled`, and what that says about the next run.
5. **Advice**: `report.advice` verbatim. An empty `advice` means the run met its target.

Then `credits_get_balance` once more and quote the new `availableCredits`.

## What the human should look at

End with a short list: the leads worth a look first (highest `fit`, warmest), any `needs_review`
leads the run parked (`leads_list` shows them; the `pancake-review-leads` playbook judges them),
the rejection reason that dominated and the Brain section it points at, and whether the envelope
was the limit (suggest tomorrow's X) or the sources were (suggest keywords or signal settings).
Never propose a second run in the same message as the result; let the user decide.
