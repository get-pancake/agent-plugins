# Pancake agent plugins

Installable packages that connect a coding agent to your Pancake CMO workspace's MCP server —
reading the GTM Brain, leads, signals, and SEO plan, and starting lead-finding runs.

- [`shared/pancake-cmo-brain/SKILL.md`](shared/pancake-cmo-brain/SKILL.md) — the operating
  conventions for the tools: ground work in the brain first, respect the voice's banned claims,
  patch semantics on every write, revisions as concurrency tokens, and when a lead-finding run
  spends real money. It's the same skill you can download directly from **Settings → MCP** inside
  the Pancake app.
- [`claude-code/`](claude-code/README.md) — an installable Claude Code plugin (this repo's own
  `.claude-plugin/marketplace.json` at the root points at it; `claude-code/.claude-plugin/plugin.json`
  + `claude-code/.mcp.json` describe the plugin itself).
- [`codex/`](codex/README.md) — a Codex CLI package (skill + MCP config example). There is no
  self-serve public Codex plugin marketplace yet, so this documents "install from this repo," and
  one config detail (the exact remote-MCP auth-header key) needs verifying against your installed
  Codex version.

## This repo is a mirror, not the source

This repo is synced automatically from Pancake's product source of truth and is not edited
directly — a pull request against it will be overwritten by the next sync. If something here is
wrong or out of date, contact Pancake support or your workspace admin.

## Authenticate

Get your workspace's MCP API key from **Settings → MCP** inside the Pancake app (each key is
workspace-scoped and carries the access of the member who created it). Set it as
`PANCAKE_MCP_API_KEY` in your shell — never commit it or paste it into a prompt. Full steps are in
each package's README.
