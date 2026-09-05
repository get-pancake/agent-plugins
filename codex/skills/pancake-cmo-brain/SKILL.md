---
name: pancake-cmo-brain
description: "Use Pancake over MCP: ground work in the GTM brain, review its improvement proposals, read and judge leads, manage signals and tracked profiles, read SEO plans, runs, and what changed."
---

# Pancake

You have access to a Pancake workspace over MCP: its go-to-market brain, qualified leads, signal
settings, SEO publication plan, and lead-finding runs.

## Ground every deliverable in the brain first

Before writing **any** marketing copy, outbound message, positioning statement, or
ICP-dependent analysis, call `brain_get`. Pull only the sections you need, for example
`{"sections":["company","icp","voice"]}` for copy, `["icp","personas"]` for targeting.

Never invent a value proposition, a competitor claim, or an ICP attribute the brain does not
contain. If something is missing, say so and offer to add it.

## Respect the voice

`voice` carries `preferred`, `avoided`, and `bannedClaims`. `bannedClaims` are hard constraints.
never write them, even as a paraphrase. `channelVariants` override the base guidelines for
LinkedIn and blog content.

## Updating the brain

Every update tool is a patch: **fields you omit are left untouched**, `null` clears a field, a
value sets it. So `brain_update_icp` with only `icpSummary` will not disturb `industries`.

Nested objects (`companyIdentity`, `icpStructure`) are replaced whole, not merged. To change one
sub-field, read the current object with `brain_get` and send it back complete.

- Company description and identity → `brain_update_company`
- Ideal customer profile → `brain_update_icp`
- Tone of voice → `brain_update_voice`
- Personas, message pillars, objections, competitors/influencers → the `brain_update_*` tools.
  Omit `id` to create; to revise, pass the `id` **and** the `revision` you got from `brain_get`.
- Keywords → `brain_add_keyword` (the phrase is permanent and unique) / `brain_recategorize_keyword`

**Always read before you write.** Every item's `revision` is a concurrency token: if a human edited
it since your `brain_get`, the tool fails and tells you the current revision. When that happens,
call `brain_get` again, re-check that your change still makes sense, and retry. Never guess a
revision.

Only write to the brain when the user has asked you to, or has confirmed a change you proposed.
The brain is shared, versioned, human-owned knowledge, not your scratchpad.

`brain_archive_item` removes an item and needs its `id` and `revision`. Always confirm with the
user first. It returns `{id, kind, revision}`; pass those to `brain_restore_item` to undo. Keep
them, because archived items no longer appear in `brain_get`.

## Reviewing the improvement loop

Pancake's improvement loop reads lead feedback, run outcomes, and market feedback and writes
**proposals** — it never changes the brain by itself. Reviewing them is the loop an agent-run
workspace closes:

1. `brain_list_proposals` lists the pending inbox (`activity_since` also announces new
   proposals as `strategy.proposal.created` events since your last check) (add `{"status":"all"}` for history). A
   `change` of `create` / `revise` / `archive` is a concrete edit with the proposed `content`
   and the engine's `rationale`; a `question` proposes nothing and asks the user something.
2. `brain_resolve_proposal` with `accept` applies a concrete change as a new revision, or
   `reject` dismisses it (the loop will not re-raise it without new evidence). Questions are
   answered with `brain_answer_proposal_question` in the user's own words, which pulls the next
   analysis forward so the concrete proposal follows within minutes.
3. `brain_get` then shows the accepted change at its new revision.

Resolving is single-shot: replaying the same decision answers the proposal's current state. If a
targeted item moved since the proposal was written, the accept fails and the proposal stays
pending — re-read with `brain_get` and decide again. Resolve proposals only when the user asked
you to review the inbox, and put the decision in front of them when the rationale is thin.

When the user reports what the market said — a customer call, a Slack thread, "most of these
people are outside our ICP" — forward it verbatim with `brain_record_market_feedback` rather
than editing the brain yourself. It writes nothing to the brain; the next analysis digests it and
proposes the change for review.

## Reading leads and submitting feedback

Use `leads_list` for a bounded, newest-first page and follow its `nextOffset` to continue. Use
`leads_get` with a returned lead id when you need the full detail. A lead reports two different
signals of quality:

- `fit` is the ICP judgment made when the lead was qualified.
- `warmness` is a current, time-decaying measure of observed engagement.

`originSignal` says what first surfaced the person. `feedback.mine` is this key owner's latest
judgment; the counts summarize all members.

Only call `lead_feedback_submit` when the user asks to judge a lead or clearly confirms the
verdict. It takes the lead's `personId`, `up` or `down`, and an optional comment. Feedback helps
Pancake improve; it does not disqualify the lead or change its stage. `lead_feedback_withdraw`
takes that verdict back (only the connecting member's own) — withdrawing where none exists is a
no-op.

A lead whose `stage` is `needs_review` is a weak match a run parked for a human: it is not
counted, delivered, or enrollable until someone decides. When the user has looked at it and wants
it in, `lead_promote_from_review` with its lead id moves it to `qualified`; any other stage is
refused with the current stage named. `lead_disqualify` is the explicit removal (it takes the
lead's `personId` and its exact `version` from `leads_get`, and stops live outreach at that lead)
— always confirm first; there is no undo here.

## Signal settings

Always call `signal_settings_get` before `signal_settings_update`. The update is a partial patch:
unmentioned signals and omitted fields stay unchanged. Preserve a signal's existing `config` when
changing only `enabled` or `weight`.

`position_change` is marked `disabled_until_implemented` and cannot be enabled. `hiring` and
`stack` are implemented but opt-in. Change signal settings only when the user asks.

The competitor, influencer, and own-brand signals read the workspace's **tracked profiles** — the
LinkedIn people and company pages whose engagers get collected. `tracked_profiles_list` shows
them with their ids; `tracked_profile_add` tracks a URL with a label and a `kind` (idempotent on
the URL — re-adding updates label and kind); `tracked_profile_remove` takes an id. Changing the
watchlist changes what the next lead-finding run collects, so do it only when the user asks.

## SEO publication plan

Use `seo_list_calendar` to read a bounded chronological page of publication appointments and
their execution-aware status. Use `seo_get_article` with a publication id for its brief, content
revision pointers, active appointment, and current status. Use `seo_get_article_content` with the
same id to pull the article text itself: every immutable content revision with its full markdown
`body`, title, excerpt, and SEO metadata, plus the approval history — `approvedContent` is the
revision a member approved (null until one is), the text to publish through your own site build.
These tools are planning and content reads: they do not draft, approve, schedule, or publish
content.

## Lead-finding runs

Pancake's scheduler runs the nightly waterfall on its own. This surface can also start work on
demand, and the unit is CREDITS. Call `lead_finding_get_spend` first: it says how many credits the
workspace's spend ceiling still allows today and this month, how many agent-started runs remain
allowed, and — under `connection` — this connection's own allowance and whether it (or every agent
start, `agentStartsPaused`) is paused. In enforce mode, the smaller ceiling bounds a run;
in shadow mode, these figures are observational. Members may lower allowances or pause/unpause;
only operators may raise allowances. Then `lead_finding_preview_plan` with a credit envelope (and optionally a lead target,
a scope — the full waterfall or one pipeline — or an explicit split) to see how the credits would
be spread across post-engagement, company-signal, and persona-sweep, the leads each stage is
expected to find, and the runnable budget for the current enforcement mode; it is free. Confirm the credits
with the user, then `lead_finding_start_plan` with the same arguments; it returns the head run id
at once. Poll `lead_finding_get_run` every minute or two until status is `published` or
`failed` — `pending` and `running` both mean wait, never that something is stuck — and never
start a second plan while one is in flight. `lead_finding_cancel_run` stops a run and keeps the
leads it already found. `lead_finding_list_runs` reviews recent runs (a waterfall's stages share a
`chainId`) with each run's lead count, stop reason, credits charged, and origin;
`lead_finding_get_run` reads counts, spend and drop ledgers, and a bounded page of people.

Every `lead_finding_get_run` answer also carries a `report` written for you: `credits` (what the
ledger charged the run and its waterfall hops — `null` when the ledger never saw it, never a
made-up zero — plus the document's spend in credits and the chain envelope), `rejections` (counts
per reason with a plain-language `meaning`, and up to ten named examples), `stopped` (why it
ended: `budget`, `deadline`, `lead_limit`, `sources_dry`, `error`, or `cancelled`, and where
that came from), `chain` (the waterfall's hops and the stage's decision), `origin`, and `advice`
— deterministic next steps: a wall of hard vetoes means the sources are off (review signal
settings, keywords, competitors); a wall of low ICP scores means the bar is high (review the ICP
in the Brain or accept `needs_review` leads); enrichment or judge failures mean a provider issue
(retry later); a budget stop says the spend per lead and what the remainder would cost; dry
sources name the signal with the best recent feedback. An empty `advice` means the run met its
target. Read `advice` before proposing another run. `include: 'decisions'` returns the run's
per-candidate verdicts, vetoes, and drops from its trail (page with `afterSeq`; size with
`decisionsLimit`, up to 200 — `limit` is the people page, up to 50) — also for a
failed run, which has no result document.

## Credit rollout posture

Spend and preview responses include `enforcementMode`. In `shadow`, credit usage is
recorded and balances may go negative; workspace and connection ceilings do not reduce or refuse
runs. Use the preview's `runnable` result, not the raw remaining ceiling, to decide the plan's
budget. Explicit pauses and the normal run limits still apply. In `enforce`, ceilings bind.
The customer credit UI is separately gated by PostHog; MCP tools remain available.

## When lead finding runs

The unattended schedule is the member's choice. `lead_finding_schedule_get` reports its `mode`:
`daily`, `weekdays` (Monday–Friday in the workspace's timezone), `weekly` (with a `weekday`,
0 = Sunday), `off`, or `agent`. `lead_finding_schedule_set` changes it — only when the user
asks. `agent` means Pancake's scheduler stands down and you decide when to look for leads by
calling `lead_finding_start_plan` yourself; no morning digest is sent on days without a run. Switching back to a cadence resumes from the next occurrence and never
backfills missed days. Pass `localTime` (`HH:MM`) to move the start; omit it to keep the current
time, or, for a workspace with no schedule yet, Pancake's overnight slot so results are ready for
the 08:30 digest. These tools never change budgets, lead targets, or tuning.

## What happened since your last check

Do not poll `lead_finding_list_runs` or `leads_list` and diff pages to learn what changed. Call
`activity_since` instead: it returns the workspace trail in order — runs started, completed,
failed, or cancelled; credits held, refused, or settled and ceiling changes; campaign connections
and replies; sender disconnects; Brain revisions and proposals; SEO publication events — from an
opaque cursor. Store the `nextCursor` it returns (it is returned even when nothing happened) and
pass it back as `cursor` on your next check; omit it only the first time. Every event carries a
`meaning` line naming the follow-up call — a run completed points at `lead_finding_get_run`,
credits refused or a ceiling change at `lead_finding_get_spend`, a reply at
`campaign_get_lead_activity`, a sender disconnect at `campaign_get_sender_status`, a Brain
change at `brain_get`. Narrow with `kinds` (exact event kinds) or `contexts` (leads, credits,
campaigns, strategy, seo, mcp, onboarding, slack); a filtered page is still a full page.
