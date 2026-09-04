# Pancake workflow — Codex

Gives Codex CLI the same Pancake workspace access as the Claude Code plugin: the
[`pancake-cmo-brain`](skills/pancake-cmo-brain/SKILL.md) skill — the same skill a member can
already download from **Settings → MCP** in the app, generated (not hand-written) here — plus
the workspace-scoped MCP server. Authentication is a browser sign-in (OAuth); there is no key
or env var to configure.

## Install (recommended — Codex CLI 0.148.0 or newer)

Codex CLI reads the **same** `../claude-code/` plugin directory directly, via this repo's own
`.claude-plugin/marketplace.json`:

```bash
codex plugin marketplace add get-pancake/agent-plugins
codex plugin add pancake-workflow@pancake-cmo
codex mcp login pancake      # opens the browser sign-in + workspace picker
codex mcp list               # expect: pancake … enabled, OAuth
```

Codex discovers its OAuth client id on its own from **0.148.0** (client-ID metadata documents,
openai/codex#38089). On **0.147.0 or older**, `codex mcp login pancake` fails with
`Dynamic client registration failed: Dynamic client registration not supported` — that version
can only register clients dynamically, which Pancake's sign-in does not implement. Either upgrade
Codex, or keep the plugin's skill and register the server with Pancake's public client id (a
URL, not a credential) before logging in:

```bash
codex mcp add pancake --url https://app.getpancake.ai/api/mcp \
  --oauth-client-id https://app.getpancake.ai/.well-known/mcp-clients/pancake-cli.json
codex mcp login pancake
```

`this/skills/pancake-cmo-brain/` exists as a **real file**, not a symlink to a shared location —
Codex's plugin-install cache step silently drops a symlink that points outside its plugin root,
so a shared symlink installs an empty `skills/` directory even though it looks fine in the
marketplace source tree.

## Install (fallback — no plugin-marketplace support)

If your Codex version predates `codex plugin`, install manually instead:

1. Clone this repo (or download just this `codex/` directory), then copy or symlink the skill
   into Codex's skill directory:

   ```bash
   git clone https://github.com/get-pancake/agent-plugins
   mkdir -p ~/.codex/skills
   ln -s "$(pwd)/agent-plugins/codex/skills/pancake-cmo-brain" ~/.codex/skills/pancake-cmo-brain
   ```

2. Register the MCP server — append
   [`config/mcp-pancake.toml.example`](config/mcp-pancake.toml.example) to `~/.codex/config.toml`
   (its `[mcp_servers.pancake.oauth]` block carries the public client id that Codex 0.147.0 and
   older need), then run `codex mcp login pancake` for the browser sign-in.

## Verify the install

After the browser sign-in, run the Claude Code package's
[`smoke-test.md`](../claude-code/smoke-test.md) steps (same MCP server, same skill — only the
client differs): one `brain_get` read, then one low-risk write (add + remove a watched keyword).

## Uninstall

Marketplace install: `codex plugin remove pancake-workflow@pancake-cmo` then
`codex plugin marketplace remove pancake-cmo` then `codex mcp remove pancake`.

Manual fallback install: remove the symlink from `~/.codex/skills/` and delete the
`[mcp_servers.pancake]` block from `~/.codex/config.toml`.

Either way, disconnect the standing access in the Pancake app under **Settings → MCP →
Connected clients**.
