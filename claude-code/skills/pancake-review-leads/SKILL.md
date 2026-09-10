---
name: pancake-review-leads
description: "Playbook: review the leads Pancake found since your last check — replay activity_since from a stored cursor, judge each new lead (feedback, promote, disqualify), and report what changed."
---

# Review last night's leads

Use this playbook when the user asks what came in overnight, or to go through new leads — "review
last night's leads", "what did Pancake find since yesterday". It reads the workspace trail from
where you last left off, walks each new lead with the user, records the verdicts, and ends with a
summary of what changed. It needs the `pancake` skill's conventions and a connected workspace.
Nothing here spends credits; feedback and promotions only change what Pancake learns and delivers.

## 0. Your cursor is yours to keep

Pancake stores no per-agent cursor. Keep the last `nextCursor` from `activity_since` in **your own
memory, keyed by workspace id** (for example a note `pancake activity cursor <workspaceId> = <cursor>`
in whatever persistent memory your client gives you). Read `workspace_get` once to learn the id
and name you are keying on.

- **Cursor found**: pass it as `cursor`.
- **No cursor (first run, or memory lost)**: do NOT read from the start of the trail. Tell the user
  you have no cursor and will look at the last 24 hours instead, then call `activity_since` without
  a cursor and keep only events whose `occurredAt` is within the last 24 hours (the trail is
  oldest-first; page with `limit: 200` while `hasMore` is true, and pass every page's `nextCursor`
  forward — a long-lived workspace has thousands of events before last night).

Save the new `nextCursor` as soon as the call returns — it is returned even when nothing happened —
so a crash mid-review never makes you replay the same night twice.

## 1. Pull what happened

`activity_since` with `{"cursor": <stored>, "contexts": ["leads", "credits"], "limit": 200}`.
Keep paging while `hasMore` is true. Collect:

- `leads.lead_finding.completed` / `.failed` / `.cancelled` — each run's lead and rejected counts
  from `payload`; its `correlationId` is the `runId` to read (the `meaning` line says which tool).
- `credits.*` — what those runs were charged, or a refusal to relay.
- Anything else the user should hear about (a ceiling change, a paused connection).

If there were no completed runs, say so, quote the `credits` events if any, store the cursor, and
stop. Do not go fishing in `leads_list` for something to review.

## 2. Read the new leads

For each completed run, `lead_finding_get_run` with its `runId`. The `leads` page is the night's
haul; `report.rejections` and `report.stopped` explain what did not make it. Runs in a waterfall
share a `chainId` — present them as one night, not three runs.

Then `leads_list` (first page, newest first) to pick up the stage each lead is in now and its
`feedback.mine`. Review only leads that are new since the cursor and not yet judged by this member;
say how many that is before starting.

## 3. Judge each lead with the user

Present one lead at a time — name, title, company, `fit` and its reason, `warmness`,
`originSignal`, the signal or post that surfaced them — and ask for one of:

- **Good lead** → `lead_feedback_submit` `{"personId", "verdict": "up"}`, with the user's reason
  as `comment` when they gave one.
- **Not a fit** → `lead_feedback_submit` `{"personId", "verdict": "down", "comment": <why>}`. Ask
  for the reason: a criterion-naming comment ("consultancies are out", "too senior") is what
  the improvement loop turns into an ICP proposal. A down verdict does not remove the lead.
- **Promote** (only for a lead whose `stage` is `needs_review`) → `lead_promote_from_review`
  `{"leadId"}` — it becomes `qualified`, counted, and deliverable.
- **Remove** → `leads_get` first for the exact `version`, then `lead_disqualify`
  `{"personId", "expectedVersion"}`. Confirm explicitly before this one: there is no undo on this
  surface, and it stops live outreach at that lead.
- **Skip** → leave it; it stays unjudged for next time.

Apply each verdict as it is given, not in a batch at the end, so an interrupted session loses
nothing. A `lead_feedback_withdraw` reverses a verdict the user changes their mind about.

If the user says "they all look fine" or "mark them all down", confirm the count once and apply
the same verdict to each lead, naming every one in the summary.

## 4. Report what changed

End with:

1. **The night**: runs completed, leads found against target, credits charged, stop reason, and
   the dominant rejection reason with its `meaning`.
2. **Verdicts**: a table of lead → what you did (up / down + comment / promoted / disqualified /
   skipped).
3. **Signal for the Brain**: the criterion-naming comments you recorded, because these pull the
   next improvement analysis forward — mention that the `pancake-refresh-icp` playbook reviews the
   proposals it writes.
4. **Cursor**: confirm you stored the new cursor for this workspace id.

## What the human should look at

Anything you skipped, any lead the user asked to disqualify (double-check the name), any run that
`failed` or stopped on `budget` (the `pancake-daily-leads` playbook covers the next envelope), and
any `credits` refusal or pause that means tonight's run may not happen.
