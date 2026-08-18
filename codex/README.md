# Pancake workflow — Codex

Gives Codex CLI the same Pancake CMO workspace access as the Claude Code plugin: the
[`pancake-cmo-brain`](skills/pancake-cmo-brain/SKILL.md) skill — the same skill a member can
already download from **Settings → MCP** in the app, generated (not hand-written) here — plus
the workspace-scoped, API-key-authenticated MCP server.

## Install (recommended — verified against codex-cli 0.147.0)

Codex CLI reads the **same** `../claude-code/` plugin directory directly, via this repo's own
`.claude-plugin/marketplace.json`. No separate Codex-specific package is needed for this path:

```bash
codex plugin marketplace add get-pancake/pancake-agent-plugins
codex plugin add pancake-workflow@pancake-cmo
codex mcp list   # expect: pancake  https://.../api/mcp  <your env var>  enabled  Bearer token
```

If `codex mcp list` shows `Auth: Unsupported` instead of `Bearer token`, your Codex version reads
a different `.mcp.json` field than `bearer_token_env_var` — check `codex mcp add --help` for the
current flag name and update `../claude-code/.mcp.json` to match (it already carries both that
field and Claude Code's `headers` field side by side; unrecognized fields are ignored by each
client, which is how one file serves both).

`this/skills/pancake-cmo-brain/` exists as a **real file**, not a symlink to a shared location —
Codex's plugin-install cache step silently drops a symlink that points outside its plugin root,
so a shared symlink installs an empty `skills/` directory even though it looks fine in the
marketplace source tree.

## Install (fallback — no plugin-marketplace support)

If your Codex version predates `codex plugin`, install manually instead:

1. Clone this repo (or download just this `codex/` directory), then copy or symlink the skill
   into Codex's skill directory:

   ```bash
   git clone https://github.com/get-pancake/pancake-agent-plugins
   mkdir -p ~/.codex/skills
   ln -s "$(pwd)/pancake-agent-plugins/codex/skills/pancake-cmo-brain" ~/.codex/skills/pancake-cmo-brain
   ```

2. Register the MCP server — append
   [`config/mcp-pancake.toml.example`](config/mcp-pancake.toml.example) to `~/.codex/config.toml`.
   It uses `--bearer-token-env-var`-equivalent `bearer_token_env_var` syntax, confirmed against
   codex-cli 0.147.0's `codex mcp add --help`.

## Authenticate

Same as the Claude Code plugin: get the workspace's MCP API key from **Settings → MCP** in the
Pancake app and export it as `PANCAKE_MCP_API_KEY` in your shell — never in a tracked file or a
prompt.

## Verify the install

Run the Claude Code package's [`smoke-test.md`](../claude-code/smoke-test.md) steps (same MCP
server, same skill — only the client differs): one `brain_get` read, then one low-risk write
(add + remove a watched keyword).

## Uninstall

Marketplace install: `codex plugin remove pancake-workflow@pancake-cmo` then
`codex plugin marketplace remove pancake-cmo` then `codex mcp remove pancake`.

Manual fallback install: remove the symlink from `~/.codex/skills/` and delete the
`[mcp_servers.pancake]` block from `~/.codex/config.toml`.

Either way, revoke the key in Settings → MCP if you no longer want it valid.
