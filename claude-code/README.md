# Pancake workflow — Claude Code plugin

Gives Claude Code direct, workspace-scoped access to a Pancake CMO account: reading the GTM Brain,
leads, signals, SEO plan, and lead-finding run history through Pancake's MCP server, plus
the operating conventions in [`pancake-cmo-brain`](skills/pancake-cmo-brain/SKILL.md) — the same
skill a member can already download from **Settings → MCP** in the app, generated (not
hand-written) here; see [`../README.md`](../README.md).

This same plugin directory also installs directly into **Codex CLI** — see
[`../codex/README.md`](../codex/README.md).

## Install

```bash
claude plugin marketplace add get-pancake/pancake-agent-plugins
claude plugin install pancake-workflow@pancake-cmo   # bundles the pancake-cmo-brain skill
```

The marketplace's own name (declared in `.claude-plugin/marketplace.json`) is `pancake-cmo`, which
is why the install id is `pancake-workflow@pancake-cmo` even though the repo path is
`pancake-agent-plugins`.

## Authenticate

No key, no env var: in a Claude Code session run `/mcp`, select **pancake**, and choose
**Authenticate**. Your browser opens Pancake's ordinary sign-in (Google or email link), you pick
the workspace to connect on the consent screen, and the server flips to ✔ Connected. The
connection refreshes itself afterwards; re-authenticate only if you revoke it or it expires.

## Verify the install

`claude plugin details pancake-workflow@pancake-cmo` should report 1 skill and 1 MCP server
(`pancake`). After authenticating, run [`smoke-test.md`](smoke-test.md): one `brain_get` read,
then one low-risk write (add + remove a watched keyword).

## Uninstall

```bash
claude plugin uninstall pancake-workflow@pancake-cmo
```

Then disconnect the standing access in the Pancake app under **Settings → MCP → Connected
clients** (uninstalling the plugin removes the local registration, not the server-side grant).
