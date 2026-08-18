# Pancake agent plugins

Installable packages that connect a coding agent to your Pancake CMO workspace's MCP server —
reading the GTM Brain, leads, signals, and SEO plan, and starting lead-finding runs.

- **`pancake-cmo-brain`** — the operating conventions for the tools: ground work in the brain
  first, respect the voice's banned claims, patch semantics on every write, revisions as
  concurrency tokens, and when a lead-finding run spends real money. It's the same skill you can
  download directly from **Settings → MCP** inside the Pancake app, generated identically into
  both `claude-code/skills/pancake-cmo-brain/SKILL.md` and `codex/skills/pancake-cmo-brain/SKILL.md`
  (real files, not symlinks — a symlinked skill silently installs empty under Codex's plugin
  cache).
- [`claude-code/`](claude-code/README.md) — an installable Claude Code plugin (this repo's own
  `.claude-plugin/marketplace.json` at the root points at it; `claude-code/.claude-plugin/plugin.json`
  and `claude-code/.mcp.json` describe the plugin itself).
- [`codex/`](codex/README.md) — **the same `claude-code/` plugin also installs directly into
  Codex CLI** via `codex plugin marketplace add` / `codex plugin add` (verified against
  codex-cli 0.147.0) — no separate Codex-specific package is needed for that path.
  `codex/README.md` documents that verified flow plus a manual fallback for older Codex
  versions without plugin-marketplace support.

## This repo is a mirror, not the source

This repo is synced automatically from Pancake's product source of truth and is not edited
directly — a pull request against it will be overwritten by the next sync. If something here is
wrong or out of date, contact Pancake support or your workspace admin.

## Authenticate

Get your workspace's MCP API key from **Settings → MCP** inside the Pancake app (each key is
workspace-scoped and carries the access of the member who created it). Set it as
`PANCAKE_MCP_API_KEY` in your shell — never commit it or paste it into a prompt. Full steps are in
each package's README.
