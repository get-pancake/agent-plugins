# Pancake agent plugins

Installable packages that connect a coding agent to your Pancake CMO workspace's MCP server —
reading the GTM Brain, leads, signals, and SEO plan, and starting lead-finding runs.
Authentication is a **browser sign-in** (OAuth): your tool opens Pancake's login, you pick the
workspace to connect, and you're done — there is no API key to copy, and nothing in these
packages is secret.

- **`pancake-cmo-brain`** — the operating conventions for the tools: ground work in the brain
  first, respect the voice's banned claims, patch semantics on every write, revisions as
  concurrency tokens, and when a lead-finding run spends real money. It's the same skill you can
  download directly from **Settings → MCP** inside the Pancake app, generated identically into
  `claude-code/skills/pancake-cmo-brain/SKILL.md`, `codex/skills/pancake-cmo-brain/SKILL.md`,
  and `droid/skills/pancake-cmo-brain/SKILL.md`
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
- [`droid/`](droid/README.md) — Factory Droid CLI, as a manual two-step install (Droid has no
  plugin marketplace): register the MCP server (`droid mcp add` or `.factory/mcp.json`), then
  copy the skill into a `.factory/skills/` directory. Droid supports Pancake's
  client-ID-metadata-document OAuth flow natively.

CLIs that support the OAuth flow but ship no client metadata document of their own (Gemini CLI,
Amp, Mastra Code, Pi, Mistral Vibe, fx) can use Pancake's hosted client id instead — set their
static OAuth client id / `client_metadata_url` to
`https://app.getpancake.ai/.well-known/mcp-clients/pancake-cli.json`; the in-app
**Settings → MCP** guide carries per-tool snippets. opencode, GitHub Copilot CLI, Kimi Code, and
goose remain absent: they only register OAuth clients dynamically (DCR), which Pancake's sign-in
does not implement — they land once their upstream client-ID-metadata (CIMD) support ships.

## This repo is a mirror, not the source

This repo is synced automatically from Pancake's product source of truth and is not edited
directly — a pull request against it will be overwritten by the next sync. If something here is
wrong or out of date, contact Pancake support or your workspace admin.

## Authenticate

No setup needed: the first time your tool connects it opens a browser window on
`app.getpancake.ai` where you sign in the usual way (Google or email link) and pick the
workspace this tool may access. Manage or disconnect standing connections anytime in
**Settings → MCP → Connected clients**.
