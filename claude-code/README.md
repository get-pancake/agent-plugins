# Pancake — Claude Code plugin

Gives Claude Code direct, workspace-scoped access to a Pancake account: managing the GTM Brain,
leads, signals, SEO articles, campaigns, lead-finding runs, and workspace settings through Pancake's MCP server, plus
the operating conventions in [`pancake`](skills/pancake/SKILL.md) — the same
skill a member can already download from **Settings → MCP** in the app, generated (not
hand-written) here; see [`../README.md`](../README.md) — and three playbook skills:
[`pancake-daily-leads`](skills/pancake-daily-leads/SKILL.md) (find N leads today under X
credits), [`pancake-review-leads`](skills/pancake-review-leads/SKILL.md) (judge what came in
since your last check), and [`pancake-refresh-icp`](skills/pancake-refresh-icp/SKILL.md)
(resolve Brain proposals and feedback, then show the diff).

Lead-finding runs consume credits: preview the cost and get explicit confirmation before starting.
Enrolling a lead starts real LinkedIn outreach and requires an explicit request.
The tools can edit, approve, and schedule SEO articles, but cannot directly publish to a CMS.

This same plugin directory also installs directly into **Codex CLI** — see
[`../codex/README.md`](../codex/README.md).

## Install

```bash
claude plugin marketplace add get-pancake/agent-plugins
claude plugin install pancake@pancake   # bundles the pancake skill + the three playbooks
```

The plugin is **Pancake**. The `@pancake` suffix selects Pancake's official marketplace.

Upgrading from the old name? Follow the [one-time migration](../README.md#upgrade-from-an-older-install).

## Authenticate

No key, no env var: in a Claude Code session run `/mcp`, select **pancake**, and choose
**Authenticate**. Your browser opens Pancake's ordinary sign-in (Google or email link), you pick
the workspace to connect on the consent screen, and the server flips to ✔ Connected. The
connection refreshes itself afterwards; re-authenticate only if you revoke it or it expires.

## Verify the install

`claude plugin details pancake@pancake` should report 4 skills and 1 MCP server
(`pancake`). After authenticating, run [`smoke-test.md`](smoke-test.md): one `brain_get` read,
then one low-risk write (add + remove a watched keyword).

## Uninstall

```bash
claude plugin uninstall pancake@pancake
```

Then disconnect the standing access in the Pancake app under **Settings → MCP → Connected
clients** (uninstalling the plugin removes the local registration, not the server-side grant).
