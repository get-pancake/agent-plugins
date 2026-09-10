---
name: pancake-refresh-icp
description: "Playbook: refresh the ICP from feedback — read pending Brain proposals and recent lead verdicts, resolve proposals or record market feedback with the user, and show the resulting Brain diff."
---

# Refresh the ICP from feedback

Use this playbook when the user wants the Brain to catch up with what the leads taught them —
"refresh the ICP", "review the Brain proposals", "the last batch was off, fix the targeting". The
Brain changes only through accepted proposals or the user's own edits; this playbook walks the
pending inbox and the recent feedback with the user, applies their decisions, and prints the diff
itself. It needs the `pancake` skill's conventions and a connected workspace. Nothing here spends
credits; answering a question or recording market feedback can pull the next improvement analysis
forward, which uses a small budgeted model call.

## 1. Snapshot the Brain before touching it

`brain_get` with `{"sections": ["icp", "personas", "objections", "market_references", "keywords"]}`.
Keep the whole response as **BEFORE**: every item's `id`, `revision`, and content. The diff at the
end is computed from this snapshot — there is no diff tool, you print it.

## 2. Read the evidence

1. `brain_list_proposals` (default `pending`; page with `nextOffset`). Each item has a
   `proposalId`, a `change` (`create` / `revise` / `archive` with the proposed `content`) or a
   `question`, the engine's `rationale`, and the run that wrote it.
2. `activity_since` with `{"contexts": ["leads", "strategy"], "limit": 200}` and the cursor you
   keep for this workspace (the `pancake-review-leads` playbook explains the cursor; without one,
   read the last 7 days by `occurredAt`, paging while `hasMore`). Collect `leads.lead.feedback_given`
   events — verdict, comment, and the machine's fit score in `payload` — and
   `strategy.proposal.created` events, which point at the inbox above.
3. `leads_list` (first page) for the current `feedback` counts, so you can tell the user how many
   ups and downs the period produced.

Summarize before deciding anything: how many proposals are pending and what they target, how many
down verdicts named a criterion (and which criteria repeat), and whether the machine fit was high
on the leads people rejected — that pattern means the ICP is mis-specified, not the run.

## 3. Resolve with the user, one item at a time

For each pending proposal, show the target item's current content from BEFORE, the proposed
content, and the rationale, and ask:

- **Accept** → `brain_resolve_proposal` `{"proposalId", "decision": "accept"}`. If the target moved
  since the proposal was written the call fails and the proposal stays pending: `brain_get` that
  section again, show the user what changed, and decide again.
- **Reject** → `brain_resolve_proposal` `{"proposalId", "decision": "reject"}` — the loop will not
  re-raise it without new evidence.
- **A `question`** → `brain_answer_proposal_question` `{"proposalId", "answer"}` in the user's own
  words, or reject to dismiss. Questions cannot be accepted.

Then the feedback that has no proposal yet. When the user tells you what the market said — "most
of these are agencies, we do not sell to agencies", "too junior", a customer call — do NOT edit the
Brain yourself: `brain_record_market_feedback` `{"note": <verbatim>, "source": "review"}`. The next
analysis digests it and proposes the change for review.

Only when the user explicitly dictates the new wording ("set the ICP summary to …") use the direct
patch tools from the `pancake` skill (`brain_update_icp`, `brain_update_persona`, …) with the
`id` and `revision` from BEFORE — and say that this bypasses the proposal loop.

Resolving is single-shot; a replay of the same decision answers the current state. Apply each
decision as it is made.

## 4. Print the diff

`brain_get` again with the same sections as **AFTER**, then print the changed fields yourself:

- For each item present in both: list every field whose value differs, as `field: before → after`,
  with the item's `revision` before and after. Unchanged items are not listed.
- Items in AFTER but not BEFORE: `created`. Items in BEFORE but not AFTER: `archived`.
- Nested objects (`icpStructure`, `companyIdentity`): diff their sub-fields, not the blob.

If nothing changed (every proposal rejected, feedback only recorded), say so and list what was
recorded for the next analysis.

## 5. Report

1. **Decisions**: proposal → accept / reject / answered, one line each.
2. **Recorded**: the market-feedback notes you forwarded, verbatim.
3. **Diff**: the field-level changes from step 4.
4. **Cursor**: the new `activity_since` cursor stored for this workspace id.

## What the human should look at

The accepted revisions (they are live for tonight's run), any proposal you rejected on their
behalf, any question left unanswered, and — if downs kept naming a criterion the Brain still does
not state — the section to edit by hand. Mention that the `pancake-daily-leads` playbook is how
to test the refreshed ICP under a small envelope.
