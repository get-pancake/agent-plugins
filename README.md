# Pancake agent plugins

Installable packages that connect a coding agent to your Pancake workspace's MCP server —
working with your GTM Brain, leads, signals, SEO articles, campaigns, and workspace settings.
Authentication is a **browser sign-in** (OAuth): your tool opens Pancake's login, you pick the
workspace to connect, and you're done — there is no API key to copy, and nothing in these
packages is secret.

- **`pancake`** — the operating conventions for the tools: ground work in the Brain
  first, respect the voice's banned claims, read before writing, use revisions as concurrency
  tokens, and preview credit costs before asking for confirmation to start lead-finding runs.
  It's the same skill you can
  download directly from **Settings → MCP** inside the Pancake app, generated identically into
  `claude-code/skills/pancake/SKILL.md`,
  `pancake/skills/pancake/SKILL.md`,
  `codex/skills/pancake/SKILL.md`, and `droid/skills/pancake/SKILL.md`
  (real files, not symlinks — a symlinked skill silently installs empty under Codex's plugin
  cache).
- **Playbooks** — three ready-made routines shipped as their own skills beside the tool skill,
  each stating what it will spend before spending and ending with what the human should look at:
  [`pancake-daily-leads`](pancake/skills/pancake-daily-leads/SKILL.md) (find N leads today under
  X credits), [`pancake-review-leads`](pancake/skills/pancake-review-leads/SKILL.md) (judge what
  came in since your last check, from a cursor the agent keeps), and
  [`pancake-refresh-icp`](pancake/skills/pancake-refresh-icp/SKILL.md) (resolve Brain proposals
  and feedback, then show the Brain diff). They are authored under `pancake/skills/` and copied
  verbatim into the other packages, so every install gets the same four skills.
- [`claude-code/`](claude-code/README.md) — an installable Claude Code plugin (this repo's own
  `.claude-plugin/marketplace.json` at the root points at it; `claude-code/.claude-plugin/plugin.json`
  and `claude-code/.mcp.json` describe the plugin itself).
- [`pancake/`](pancake/README.md) — the universal OpenAI plugin package for the shared
  ChatGPT and Codex directory, with its `.codex-plugin/plugin.json`, MCP connection, listing
  metadata, brand asset, and generated skill.
- [`codex/`](codex/README.md) — **the same `claude-code/` plugin also installs directly into
  Codex CLI** via `codex plugin marketplace add` / `codex plugin add`. The zero-config browser
  sign-in needs Codex CLI **0.148.0 or newer** (the first release that discovers Pancake's
  client-ID metadata document); 0.147.0 and older must register the server with the hosted
  client id below, and `codex/README.md` documents both plus a manual fallback for Codex
  versions without plugin-marketplace support. `pancake/` is the public universal directory
  package.
- [`droid/`](droid/README.md) — Factory Droid CLI, as a manual two-step install (Droid has no
  plugin marketplace): register the MCP server (`droid mcp add` or `.factory/mcp.json`), then
  copy the skill into a `.factory/skills/` directory. Droid supports Pancake's
  client-ID-metadata-document OAuth flow natively.

## Supported clients

Pancake's sign-in is a Client ID Metadata Document (CIMD) OAuth flow — no dynamic client
registration (DCR), no API key. A client is listed as supported only once its browser sign-in
has completed against production (`app.getpancake.ai`), through the `/connect` workspace picker,
and a `tools/list` came back. "Static id" means the client cannot discover a client id on its
own and must be handed Pancake's hosted one — a public URL, not a credential:

```
https://app.getpancake.ai/.well-known/mcp-clients/pancake-cli.json
```

| Client                                              | Setup                                               | Status               | Verified (version · date)                                                                                                                                                                                                 |
| --------------------------------------------------- | --------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Code                                         | plugin, no config                                   | Pending verification | 2.1.267 · 2026-09-09 — the CLI's authorize step reached the consent page; consent + `tools/list` still to be completed in a browser                                                                                       |
| Codex CLI ≥ 0.148                                   | plugin, no config                                   | Pending verification | 0.153.4 · 2026-09-09 — the CLI's authorize step reached the consent page; consent + `tools/list` still to be completed in a browser                                                                                       |
| Codex CLI ≤ 0.147                                   | static id (`[mcp_servers.pancake.oauth] client_id`) | Pending verification | 0.147.0 · 2026-09-09 — login reached the consent page only with the static id; production rejected its random `/callback/<token>` path until the fix shipping with this change                                            |
| GitHub Copilot CLI ≥ 1.0.83                         | `~/.copilot/mcp-config.json`, no config             | Pending verification | CIMD shipped in [v1.0.83](https://github.com/github/copilot-cli/releases/tag/v1.0.83) (2026-09-04); older releases: `oauthClientId` static id                                                                             |
| VS Code (Copilot Chat)                              | `.vscode/mcp.json`, no config                       | Pending verification | CIMD since 1.106 ([vscode#271403](https://github.com/microsoft/vscode/pull/271403))                                                                                                                                       |
| Factory Droid                                       | `droid mcp add`, no config                          | Pending verification | CIMD native per [Factory docs](https://docs.factory.ai/harness/mcp)                                                                                                                                                       |
| Gemini CLI                                          | static id (`oauth.clientId` in `settings.json`)     | Pending verification | No CIMD ([gemini-cli#25724](https://github.com/google-gemini/gemini-cli/issues/25724) closed without implementation); random-port `/oauth/callback` redirect is covered by the hosted id                                  |
| Amp                                                 | static id (`amp mcp oauth login --client-id`)       | Pending verification | No CIMD evidence; fixed `localhost:8976/oauth/callback` redirect is covered by the hosted id                                                                                                                              |
| Cursor                                              | `~/.cursor/mcp.json`                                | Pending verification | No CIMD upstream ([forum thread](https://forum.cursor.com/t/mcp-oauth-cimd-support-plans-and-timelines/148096), staff: "no timeline"); DCR or `auth.CLIENT_ID` only — expected to FAIL without CIMD                       |
| Zed                                                 | `settings.json` `context_servers`                   | Pending verification | Ships a CIMD document, but [zed#56769](https://github.com/zed-industries/zed/issues/56769) (CIMD not working) and [zed#62637](https://github.com/zed-industries/zed/issues/62637) (public client without secret) are open |
| goose ≥ 1.32                                        | no config                                           | Pending verification | CIMD native since 1.32.0 ([goose#8550](https://github.com/aaif-goose/goose/pull/8550))                                                                                                                                    |
| opencode                                            | —                                                   | Unsupported          | DCR only; [opencode#25961](https://github.com/anomalyco/opencode/issues/25961) open                                                                                                                                       |
| Kimi Code CLI                                       | —                                                   | Unsupported          | FastMCP OAuth, no client-id configuration documented ([kimi-cli#2172](https://github.com/MoonshotAI/kimi-cli/issues/2172))                                                                                                |
| Mistral Vibe                                        | —                                                   | Unsupported          | [Vibe docs](https://docs.mistral.ai/vibe/code/cli/mcp-servers): OAuth-protected MCP servers not supported (API key/headers only)                                                                                          |
| Mastra Code                                         | —                                                   | Unsupported          | CIMD PR [mastra#22934](https://github.com/mastra-ai/mastra/pull/22934) open                                                                                                                                               |
| Cline                                               | —                                                   | Unsupported          | static-id PR [cline#13679](https://github.com/cline/cline/pull/13679) open, no CIMD                                                                                                                                       |
| Pi (`pi-mcp-adapter`), fx, Windsurf, Warp, Grok CLI | —                                                   | Unverified           | no CIMD or static-id evidence in docs or changelogs                                                                                                                                                                       |
| Claude.ai / Claude Desktop connectors               | —                                                   | Coming               | blocked on the HTTP-logging issue (ADR 0055)                                                                                                                                                                              |

Rows marked "Pending verification" are installed and signed in by hand on a real machine
(PAN-849); a row moves to "Supported" only with a version and date. The in-app **Settings → MCP**
guide mirrors this table and carries the per-client snippets.

### Static-id snippets

**Gemini CLI** — `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "pancake": {
      "httpUrl": "https://app.getpancake.ai/api/mcp",
      "oauth": {
        "enabled": true,
        "clientId": "https://app.getpancake.ai/.well-known/mcp-clients/pancake-cli.json"
      }
    }
  }
}
```

then `/mcp auth pancake` inside Gemini CLI.

**Amp** — after adding the server to Amp's `amp.mcpServers` settings:

```bash
amp mcp oauth login pancake \
  --server-url https://app.getpancake.ai/api/mcp \
  --client-id https://app.getpancake.ai/.well-known/mcp-clients/pancake-cli.json
```

**Codex CLI ≤ 0.147** — see [`codex/README.md`](codex/README.md).

## This repo is a mirror, not the source

This repo is synced automatically from Pancake's product source of truth and is not edited
directly — a pull request against it will be overwritten by the next sync. If something here is
wrong or out of date, contact Pancake support or your workspace admin.

## Upgrade from an older install

Pancake replaces the old `pancake-workflow@pancake-cmo` plugin and `pancake-cmo-brain` skill.
This is a one-time identity change: a marketplace refresh does not rename an installed plugin.
Remove the old plugin and marketplace, then install Pancake:

```bash
# Claude Code
claude plugin uninstall pancake-workflow@pancake-cmo
claude plugin marketplace remove pancake-cmo
claude plugin marketplace add get-pancake/agent-plugins
claude plugin install pancake@pancake

# Codex
codex plugin remove pancake-workflow@pancake-cmo
codex plugin marketplace remove pancake-cmo
codex plugin marketplace add get-pancake/agent-plugins
codex plugin add pancake@pancake
```

In Cowork, remove the old plugin and marketplace in plugin settings, add the same GitHub
repository again, and install **Pancake**. Restart the client after migrating. If you manually
installed the old skill, replace that copy with `skills/pancake/` instead of keeping both.

The MCP server name (`pancake`), URL, and workspace permissions are unchanged. Removing a
plugin does not revoke its standing OAuth grant. The client may ask you to sign in again;
review the requested access before approving it. This repository is Pancake's direct
marketplace, not evidence of acceptance into OpenAI's or Anthropic's public Directory.

## Authenticate

No setup needed: the first time your tool connects it opens a browser window on
`app.getpancake.ai` where you sign in the usual way (Google or email link) and pick the
workspace this tool may access. Manage or disconnect standing connections anytime in
**Settings → MCP → Connected clients**.
