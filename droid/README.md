# Pancake CMO for Factory Droid

Connects Factory's Droid CLI to your Pancake CMO workspace's MCP server, with the same
`pancake-cmo-brain` skill the other packages ship. Authentication is a browser sign-in
(OAuth) — Droid supports Pancake's client-ID-metadata-document flow out of the box, so there
is no key, token, or env var to configure and nothing in this package is secret.

## Install

Droid has no plugin marketplace, so this is a two-step manual install.

**1. Register the MCP server** — either run:

```bash
droid mcp add pancake https://beta.getpancake.ai/api/mcp --type http
```

or append the contents of [`config/mcp-pancake.json.example`](config/mcp-pancake.json.example)
to `~/.factory/mcp.json` (personal) or a trusted project's `.factory/mcp.json` (shared with
your team). Then type `/mcp` inside Droid and authenticate `pancake` — a browser window opens
where you sign in to Pancake the usual way and pick the workspace to connect.

**2. Install the skill** — copy [`skills/pancake-cmo-brain/`](skills/pancake-cmo-brain/) to
`~/.factory/skills/pancake-cmo-brain/` (personal) or `<your-repo>/.factory/skills/` (project),
then restart Droid so it loads.

## Manage the connection

Disconnect or review standing connections anytime in **Settings → MCP → Connected clients**
inside the Pancake app.
