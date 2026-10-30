---
name: pancake-daily-leads
description: "Playbook: find N leads today in a Pancake workspace under a stated credit limit — read the credits, choose the Play, confirm, run it, poll, and report what was found and rejected."
---

# Find N leads a day under X credits

Use this playbook when the user asks for leads today under a budget — "find me 10 leads for
under 3,000 credits", "run lead finding, keep it under 2k". It runs one of the workspace's saved
Plays now with `plays_run`, and never spends more than the number the user agreed to.
It needs the `pancake` skill's conventions and a connected workspace; every step is an MCP tool
call and nothing here needs the Pancake app.

Inputs: **N** (new leads to aim for; default 10) and **X** (the credit limit). Only a new
lead in stage `qualified` (a strong match) counts toward N: a person who
was already a lead, and a lead that waits for review (`needs_review`), do not count.
If the user gave only one, ask for the other before spending anything.

## 1. Read what the workspace can spend — free

1. `credits_get_balance` — `availableCredits`, `heldCredits` (reserved by searches still running,
   not spent), the current `cycle` (trial, monthly package or operator plan, with its allowance and
   end), the fixed `prices` (a work email found), and `enforcement`. A search has no fixed price:
   it reserves credits when it starts and is charged its actual research cost, 1,000 credits per
   $1 rounded down, whatever it finds.
2. `lead_finding_get_spend` — `balance` and `startable` (under `enforce`, the credits a start may
   reserve right now).

Decide the credits you will ask for, `E = min(X, startable.credits)` when `startable` is set, else
`E = X`. E is the most the search may reserve and the most it can be charged; it is charged only
the research it actually does. In `enforce` the balance binds: a run whose research reaches its cap
stops and publishes what it found. If `enforcement` is `off`, credits are recorded but not charged
on this deployment: say so, and treat X as the only rail.

If the balance cannot cover the least a search reserves, say so and point the user at Settings ›
Billing (a new monthly cycle or a larger package); paying stays with a person.

## 2. Choose the Play — free

1. `plays_list` — the workspace's Plays. Use the Play the user named. If there are several and
   they named none, ask which one. Never create a Play here: `plays_create_from_ask` is for when
   the user asks for a new Play.
2. `plays_get` with its `playId`, and check it before anything is spent:
   - `search` — Pancake writes the Play's search. A Play whose `search` is `null` uses the older
     kind of search, which Pancake no longer runs: say so and offer to create a new Play. When
     `search.progress` says the search is not ready yet, tell the user where it stands and stop.
   - `readiness` — when it is `blocked`, quote `readiness.memberRemedy` and stop.
   - `activeRunKnown` must be `true` and `activeRun` must be `null`. If a run is queued or running,
     never start a second one: tell the user which run is running and offer to poll it with
     `lead_finding_get_run` instead.
   - `search.freshness` — while `nextRefreshAt` is ahead, tell the user when the search was last
     refreshed and that a new search may return mostly the same people. Send `refresh: true` only if
     they ask to refresh anyway.

## 3. State the spend and get a yes

Before starting, tell the user in one message:

> I will run the Play **<name>**, looking for **N new leads**, with a limit of **E credits**.
> Enforcement is `<mode>`; the workspace has V credits available this cycle. The search reserves at
> most E and is charged only the research it actually does, rounded down. Nothing has been spent
> yet. Start it?

Wait for an explicit yes. A previous confirmation for a different limit does not carry over.

## 4. Run and poll

`plays_run` with `{"playId": "<id>", "credits": E, "target": N}`, exactly as the user confirmed.
It returns the `runId` at once. Then `lead_finding_get_run` with that `runId` every 1–2 minutes
until `status` is `published` or `failed` — `pending` and `running` both mean wait, never that
something is stuck. The search can take more than one run to find N: `plays_get`'s
`search.request` says how far it is ("21 of 50 new leads found — still searching"), and
`lead_finding_list_runs` with the `playId` lists its runs. Do not start anything else meanwhile. If
the user asks to stop, `plays_cancel_search` stops the search and keeps the leads found so far.

## 5. Report the result

From the final `lead_finding_get_run` (add `include: "rejected"` for the retired people):

1. **Found**: the run's `leadCount` (the new leads it gave the Play) against N, and the Play's
   newest leads from `leads_list` with its `playId` — name, title, company, `fit`, and why they fit.
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
