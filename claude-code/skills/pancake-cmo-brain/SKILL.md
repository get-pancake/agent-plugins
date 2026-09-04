---
name: pancake-cmo-brain
description: "Use Pancake over MCP: ground work in the GTM brain, read leads, SEO plans, and lead-finding runs, and manage signals and feedback."
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
Pancake improve; it does not disqualify the lead or change its stage.

## Signal settings

Always call `signal_settings_get` before `signal_settings_update`. The update is a partial patch:
unmentioned signals and omitted fields stay unchanged. Preserve a signal's existing `config` when
changing only `enabled` or `weight`.

`position_change` is marked `disabled_until_implemented` and cannot be enabled. `hiring` and
`stack` are implemented but opt-in. Change signal settings only when the user asks.

## SEO publication plan

Use `seo_list_calendar` to read a bounded chronological page of publication appointments and
their execution-aware status. Use `seo_get_article` with a publication id for its brief, content
revision pointers, active appointment, and current status. These tools are planning reads: they do
not draft, approve, schedule, or publish content.

## Lead-finding runs

Pancake's scheduler runs the nightly waterfall on its own. This surface can also start work on
demand, and the unit is CREDITS. Call `lead_finding_get_spend` first: it says how many credits the
workspace's spend ceiling still allows today and this month, and how many agent-started runs remain
allowed. Then `lead_finding_preview_plan` with a credit envelope (and optionally a lead target,
a scope — the full waterfall or one pipeline — or an explicit split) to see how the credits would
be spread across post-engagement, company-signal, and persona-sweep, the leads each stage is
expected to find, and what the ceiling would actually let through; it is free. Confirm the credits
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
