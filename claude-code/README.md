# Pancake workflow — Claude Code plugin

Gives Claude Code direct, workspace-scoped access to a Pancake CMO account: reading the GTM Brain,
leads, signals, and SEO plan, and starting lead-finding runs, through Pancake's MCP server
(the workspace-scoped, API-key-authenticated MCP server), plus the operating conventions in
[`pancake-cmo-brain`](../shared/pancake-cmo-brain/SKILL.md) — the same skill a member can already
download from **Settings → MCP** in the app, symlinked here (not copied) and generated, not
hand-written; see [`../README.md`](../README.md).

## Install

```bash
claude plugin marketplace add get-pancake/pancake-agent-plugins
claude plugin install pancake-workflow@pancake-cmo   # bundles the pancake-cmo-brain skill
```

The marketplace's own name (declared in `.claude-plugin/marketplace.json`) is `pancake-cmo`, which
is why the install id below is `pancake-workflow@pancake-cmo` even though the repo path is
`pancake-agent-plugins`.

## Authenticate

1. In the Pancake app, go to **Settings → MCP** and reveal (or generate) the workspace's MCP API
   key. Each key is workspace-scoped and carries the access of the member who created it.
2. Export it where Claude Code reads plugin MCP server env vars, e.g. in your shell profile or a
   local (untracked) `.env`:

   ```bash
   export PANCAKE_MCP_API_KEY="sk-..."
   ```

3. Never commit this value or paste it into a prompt. Rotating it in Settings → MCP invalidates the
   old value immediately.

## Verify the install

`claude plugin details pancake-workflow@pancake-cmo` should report 1 skill and 1 MCP server
(`pancake`) — this was confirmed against Claude Code 2.1.234. Run `/mcp` inside Claude Code and
confirm the `pancake` server is connected, then run [`smoke-test.md`](smoke-test.md): one
`brain_get` read, then one low-risk write (add + remove a watched keyword).

## Uninstall

```bash
claude plugin uninstall pancake-workflow@pancake-cmo
```

Uninstalling removes the plugin's skill and MCP server registration only. It does not touch
`PANCAKE_MCP_API_KEY` in your own shell/profile — unset or remove that yourself, and revoke the key
in Settings → MCP if you no longer want it valid.
