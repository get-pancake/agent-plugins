# Pancake workflow — Codex package

Gives Codex CLI the same Pancake CMO workspace access as the Claude Code plugin: the
[`pancake-cmo-brain`](../shared/pancake-cmo-brain/SKILL.md) skill — the same skill a member can
already download from **Settings → MCP** in the app, symlinked here (not copied) and generated, not
hand-written; see [`../README.md`](../README.md) — plus the workspace-scoped,
API-key-authenticated MCP server.

> **Status:** this package installs a real `SKILL.md` and documents a real MCP server, both
> confirmed against current Codex docs (`SKILL.md` support, `~/.codex/config.toml` MCP
> registration). What is **not yet confirmed** is the exact `config.toml` key for custom auth
> headers on a *remote* (Streamable HTTP) MCP server — see the note in
> [`config/mcp-pancake.toml.example`](config/mcp-pancake.toml.example). Verify and update that file
> against your installed Codex version before treating this as production-ready. There is no
> self-serve public Codex plugin marketplace to publish to yet, so distribution today is "install
> from this repo," same as the Claude Code plugin.

## Install

1. Clone this repo (or download just this `codex/` directory), then copy or symlink the skill into
   Codex's skill directory:

   ```bash
   git clone https://github.com/get-pancake/pancake-agent-plugins
   mkdir -p ~/.codex/skills
   ln -s "$(pwd)/pancake-agent-plugins/codex/skills/pancake-cmo-brain" ~/.codex/skills/pancake-cmo-brain
   ```

2. Register the MCP server — append
   [`config/mcp-pancake.toml.example`](config/mcp-pancake.toml.example) (with the header syntax
   verified for your Codex version) to `~/.codex/config.toml`.

## Authenticate

Same as the Claude Code plugin: get the workspace's MCP API key from **Settings → MCP** in the
Pancake app and export it as `PANCAKE_MCP_API_KEY` in your shell — never in a tracked file or a
prompt.

## Verify the install

Run `/mcp` in Codex CLI and confirm the `pancake` server is listed as connected, then run the
Claude Code package's [`smoke-test.md`](../claude-code/smoke-test.md) steps (same MCP server, same
skill — only the client differs): one `brain_get` read, then one low-risk write (add + remove a
watched keyword).

## Uninstall

Remove the symlink from `~/.codex/skills/` and delete the `[mcp_servers.pancake]` block from
`~/.codex/config.toml`. Revoke the key in Settings → MCP if you no longer want it valid.
