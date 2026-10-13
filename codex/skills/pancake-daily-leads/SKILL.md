---
name: pancake-daily-leads
description: "Playbook: find N leads today in a Pancake workspace under a stated credit limit — read the credits, preview the plan, confirm, start it, poll, and report what was found and rejected."
---

# Find N leads a day under X credits

Use this playbook when the user asks for leads today under a budget — "find me 10 leads for
under 3,000 credits", "run lead finding, keep it under 2k". It runs the same waterfall Pancake's
scheduler runs overnight, on demand, and never spends more than the number the user agreed to.
It needs the `pancake` skill's conventions and a connected workspace; every step is an MCP tool
call and nothing here needs the Pancake app.

Inputs: **N** (leads to aim for, 1–50; default 10) and **X** (the credit envelope). Only a new
lead in stage `qualified` (a strong match) counts toward N: a person who
was already a lead, and a lead that waits for review (`needs_review`), do not count.
If the user gave only one, ask for the other before spending anything.

## 1. Read what the workspace can spend — free

1. `credits_get_balance` — `availableCredits`, `heldCredits` (reserved by searches still running,
   not spent), the current `cycle` (trial, monthly package or operator plan, with its allowance and
   end), the fixed `prices` (a work email found), and `enforcement`. A search has no fixed price:
   it reserves credits when it starts and is charged its actual research cost, 1,000 credits per
   $1 rounded down, whatever it finds.
2. `lead_finding_get_spend` — `balance`, `startable` (under `enforce`, the credits a start may
   reserve right now), and `limits.startsRemainingToday` / `concurrentRemaining`.
3. `lead_finding_list_runs` with `{"limit": 5}` — if any run is `pending` or `running`, stop here:
   never start a second plan while one is in flight. Tell the user which run is running and offer
   to poll it instead.

Decide the credits you will ask for, `E = min(X, startable.credits)` when `startable` is set, else
`E = X`. E is the most the plan may reserve and the most it can be charged; it is charged only the
research it actually does. In `enforce` the balance binds: a run whose research reaches its cap
stops and publishes what it found. If `enforcement` is `off`, credits are recorded but not charged
on this deployment: say so, and treat X as the only rail.

If `startsRemainingToday` is 0, report it and stop. If the balance cannot cover the least a search
reserves, say so and point the user at Settings › Billing (a new monthly cycle or a larger
package); paying stays with a person.

## 2. Preview the plan — free

`lead_finding_preview_plan` with `{"credits": E, "target": N}` (add `geographies` or a `scope`
only if the user asked for them). Read back:

- `stages` — for post-watchlist, post-keyword, company-stack, company-hiring, and persona-sweep: the
  credits `planned` (what the stage reserves: the most it may cost), the `expectedLeads`
  (p25/p50/p80, an estimate, never a promise), and a `basis` saying whether the estimate comes from
  this workspace's own runs or a fleet aggregate (a basis labeled `seededFrom` borrowed a retired
  merged pipeline's history while the split pipelines are young); a stage with
  `skipped: "target_met"` gets nothing. `expectedLeadsTotal` and `unallocatedCredits` sum it up.
- `runnable.credits` — what the balance lets through (with a `reason` — `balance` or `floor` —
  when it is less than you asked). Use it, not the raw balance.
- `nextStep`, and anything else it warns about (an unusable Brain, missing signals).

If the expected leads fall clearly short of N, say so and offer either a larger E (never above X)
or a smaller N. Do not silently raise the envelope.

## 3. State the spend and get a yes

Before starting, tell the user in one message:

> I will start a `<scope>` plan aiming for **N leads** with an envelope of **E credits**
> (planned: post-watchlist A / post-keyword B / company-stack C / company-hiring D /
> persona-sweep F). Expected leads ≈ L (p50).
> Enforcement is `<mode>`; the workspace has V credits available this cycle. The plan reserves at
> most E and is charged only the research it actually does, rounded down. Nothing has been spent
> yet. Start it?

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

1. **Found**: how many leads the run put in stage `qualified` (a strong match) against N, and
   the first page of people — name, title, company, `fit`, and the signal that surfaced them.
2. **Spent**: `report.credits` — what the ledger charged the run and its hops for their research,
   against E (`documentSpend` is what the run's own record says its research cost). Say plainly
   whether it stayed inside E (it must; if the ledger shows more, report that as a discrepancy
   rather than explaining it away).
3. **Rejected and why**: `report.rejections` — the counts per reason with their `meaning` and a
   few named examples. A wall of hard vetoes means the sources are off; a wall of low ICP scores
   means the bar is high; enrichment or judge failures are a provider issue.
4. **Stopped**: `report.stopped` — `budget` (its research reached its cap), `deadline`,
   `lead_limit`, `sources_dry`, `error`, or `cancelled`, and what that says about the next run.
5. **Advice**: `report.advice` verbatim. An empty `advice` means the run met its target.

Then `credits_get_balance` once more and quote the new `availableCredits`, and `leads_list`
(`limit: 1`) for its `leadCounts`: say each Play's To review count with the Play's name, as the
member will find it on that Play's Leads page. Never add the Plays together.

## What the human should look at

End with a short list: the leads worth a look first (highest `fit`, warmest), any `needs_review`
leads the run parked (`leads_list` shows them; the `pancake-review-leads` playbook judges them),
the rejection reason that dominated and the Brain section it points at, and whether the envelope
was the limit (suggest tomorrow's X) or the sources were (suggest keywords or signal settings).
Never propose a second run in the same message as the result; let the user decide.
