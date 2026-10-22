---
name: pancake-onboarding
description: "Playbook: finish a paid founder's Pancake welcome here — read the welcome and Brain, ask only what is missing, confirm one first Play, create it once, show real progress, offer a tour."
metadata:
  version: "1.1.1"
---

# Create the founder's first Play in their AI app

Use this playbook when a founder who just paid for Pancake pastes Pancake's onboarding prompt
("Help me get started with Pancake by following https://pancake.ai/onboarding.md.", or a local
test server's own guide), or asks you to set up their first Play after choosing **In Agent** on
Pancake's welcome page. The prompt is that one sentence: the guide it names connects Pancake and
brings you here, and a guide served by a local test server names the only server to use.
Pancake researched their company before checkout: the workspace is paid and its Brain exists. You
continue that welcome here: read what Pancake knows, ask only what is missing, agree ONE first
Play with the founder, create it exactly once, and show what it really does. It needs a connected
workspace, and the `pancake` skill's conventions apply.

The welcome's state lives in Pancake, never in this chat: a new chat, a re-pasted prompt or another
tab resumes from what the `postboarding_*` tools answer. Never start a new research onboarding,
create a workspace, or ask the founder to pay.

## 1. Read the welcome — free

First check the connection: `workspace_get` must succeed through it; tell the founder which
workspace it reached and let them confirm it is the one their welcome page names. Then
`postboarding_get_context`. Your connection resolves their welcome: pass `{"reference": "wel_…"}`
only when the founder's message carries one (older prompts did), never an invented one. It
answers:

- `workspace` — the workspace this connection acts on. If the founder expected another one, or the
  call says the reference is not this connection's, stop and help them reconnect Pancake in their
  app (https://pancake.ai/install.md) and approve the right workspace. Change nothing before
  that.
- `session` — `revision` (every `postboarding_advance` names it), `step`, `handoff`,
  `briefRevision` and `completion`; `allowedActions` — the moves possible now. Make only those.
- `capabilities.firstPlayLifecycle` — `coded` (Pancake writes the Play's search from the
  founder's words) or `unavailable` (wait; never fall back to a legacy Play); `capabilities.credits` — the
  credit mode and what is available.
- `brief`, `firstPlay`, `uiFirstPlay`, `walkthrough`, `links` and `nextStep`.
- `playbook` — this guidance with its version. When your installed copy has another version,
  follow the one Pancake sends.

Then resume where the welcome stands:

| The context says                         | Do                                                                                                                                                              |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `session.completion` is `first_play`     | Step 6, then offer the walkthrough (step 7). Never create another Play.                                                                                         |
| `session.completion` is `deferred`       | They chose to set the Play up later and the app is open. If they want one now, use the ordinary Play tools of the `pancake` skill, with the same confirmations. |
| `firstPlay.operationStatus` is `claimed` | Another chat started creating it: call `postboarding_confirm_first_play` with the same `briefRevision` to finish it; Pancake kept its plan.                     |
| `uiFirstPlay` is set                     | They already started a first Play in the app: step 5 makes it this welcome's Play and creates nothing.                                                          |
| `session.handoff` is `null`              | Step 2.                                                                                                                                                         |
| a `brief` is saved                       | Read it back to them and continue at step 3 or 4.                                                                                                               |

## 2. Acknowledge the handoff

`postboarding_advance` with `{"expectedRevision": <session.revision>, "move": {"kind":
"acknowledge_handoff", "playbookVersion": "<playbook.version>", "client": "claude_code" | "codex" |
"other"}}` (`other` for Claude Chat, ChatGPT and any other app). The founder's browser then says the
welcome continues here and follows its progress. Repeating it in a later chat is harmless.

## 3. Learn what is missing — one question at a time

Read before you ask:

1. `brain_get` — the company, ICP, personas and competitors. Tell the founder in two sentences
   what Pancake already understands about their business and target customer, so they can correct
   it.
2. `plays_list` — the Plays that exist (Main always does). The first Play repeats none of them.
3. When `brain.stillResearching` lists sections, say Pancake is still filling them in and carry
   on with what is there.

Then ask only what is still missing, usually two to four questions, one at a time, fewer when the
Brain or their words already answer them:

- who exactly to find, and the immediate goal;
- people or companies;
- the places, company sizes, technologies or signals that matter, and what must be excluded;
- how many new leads one search keeps, only when they care (otherwise Pancake's default of 10);
- how often Pancake searches: once, every day, or every week on a day they name. Never choose it
  for them, and never default to daily.

Keep their words. Their request is the `ask`, verbatim; each question you asked and their answer
is a `turn`, verbatim; a number of leads they named is `target` (an employee count is never this
number) with their words in `targetWords`; what they excluded goes in `exclusions`, verbatim.

Save the brief after each answer that changes it. It is the checkpoint a new chat resumes:
`postboarding_advance` with `{"expectedRevision", "move": {"kind": "save_brief", "brief":
{"playName", "ask", "turns", "outputEntity", "target", "targetWords", "exclusions", "launch"}}}`.
`launch` stays `null` until they answered how often and whether the first search runs now:
`{"firstSearch": true | false, "recurrence": {"frequency": "once"} | {"frequency": "daily"} |
{"frequency": "weekly", "weekday": 0–6 (0 is Sunday)}, "recurrenceWords": <their words or null>}`.
The answer's `session.briefRevision` is the revision they will confirm.

When a criterion cannot be searched, say so plainly and offer the closest narrower Play; never
drop it silently.

## 4. Propose one Play and get an explicit yes

Propose ONE Play, named after who it finds, and say why it is the useful first step. Follow
`capabilities.firstPlayLifecycle`:

- `coded` — Pancake writes, prepares and checks the search itself from the founder's words. No
  sample waits for anyone's approval: the first search runs once the checks pass, then the Play
  searches as they chose.
- `unavailable` — keep the saved brief and wait for search creation to become available. Never fall back to `plays_plan` or legacy creation. The founder may defer setup to enter the app now.

Before anything is created, summarize in one message and ask them to confirm: who it finds, what it
excludes, how many new leads per search, how often it searches and whether the first search runs
now, and the credits it may use (read `capabilities.credits` and `lead_finding_get_spend`; speak
credits, never dollars, and call nothing free unless a tool says so). A change to any of these
after their yes needs a new brief revision and a new yes.

A no or a "not now" creates nothing. When they want the app now and the Play later,
`postboarding_advance` with the move `{"kind": "defer"}` opens the app without a Play.

## 5. Create it — once

`postboarding_confirm_first_play` with `{"briefRevision": <the revision they confirmed>}`. New Plays use the coded lifecycle and its ordinary credit authorization; do not send a legacy `plan` or its old `credits` parameter.

It creates the welcome's one first Play through the workspace's own lifecycle, or answers the one
that already exists: a retry, a new chat or another tab never makes a second Play. `outcome` says
which: `created`, `resumed`, `already_saved` or `adopted` (the Play they started in the app). When
it answers that the brief changed, read the welcome again and confirm the current brief with them.
When it refuses, it says why; tell them, and offer to continue in the app.

## 6. Show what really happens

Read `firstPlay` in `postboarding_get_context` (`progress`, `attention`, `newLeads`, `toReview`,
`link`), and `plays_get` with its `playId`:

- `preparing` — Pancake is writing and checking the search; nothing is wrong. Look again in a few
  minutes.
- `searching` — the first search runs.
- `results` — show its leads with `leads_list` (`playId`) and why each one fits; `no_results` —
  say so, and offer to adjust the brief with them. Never widen the targeting yourself.
- `needs_attention` — tell them what `attention` names (credits, setup) and how to fix it.

Never invent a lead, a count or a result. Give them `firstPlay.link`: it opens the same Play in
Pancake.

## 7. Offer the walkthrough — optional

Ask whether they want a short tour of what Pancake can do for this Play. Show one topic at a time
and record each with `postboarding_advance` and the move `{"kind": "walkthrough", "topic":
<topic>, "outcome": "shown" | "skipped"}`; close the tour with the move `{"kind":
"finish_walkthrough", "outcome": "completed" | "skipped"}`. They can stop, or come back to it in
another chat.

| `topic`        | Show                                                                                                                                | Never                                                                             |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `brain`        | `brain_get`: the ICP and personas, and how their corrections improve every later search.                                            | Edit the Brain unless they ask; then follow the `pancake` skill's revision rules. |
| `plays`        | This Play's audience, search and cadence (`plays_get`), and how to pause or resume it.                                              | Create a demo Play or a schedule they did not choose.                             |
| `leads`        | Its leads and why they fit (`leads_list`, `leads_get`); `lead_feedback_submit` records a verdict they give.                         | Record a verdict they did not give.                                               |
| `research`     | What Pancake already knows about a lead, and `research_read_public_page` for a public page they name.                               | Start a paid lookup without stating its cost and hearing yes.                     |
| `sequences`    | How an approved lead joins this Play's LinkedIn sequence (`campaign_get_overview`).                                                 | Add anyone to a sequence or send anything.                                        |
| `integrations` | Where results are delivered (`workspace_get`), the credits left (`credits_get_balance`); Slack and LinkedIn connect in the browser. | Promise an integration this workspace does not have.                              |

## What the human should look at

The first Play in Pancake (its link), what its first search found once it finished, and the
credits it uses. Connecting LinkedIn or Slack and changing the plan stay theirs, in the browser.
Outreach starts only when they add people to a sequence themselves.
