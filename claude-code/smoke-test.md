# Smoke test — Claude Code plugin

Run this after any install/config change to `agent-plugins/claude-code/` or the shared skill,
against a real (non-production-critical) Pancake workspace.

```bash
claude plugin marketplace add ./agent-plugins   # or get-pancake/pancake-agent-plugins
claude plugin install pancake-workflow@pancake-cmo
claude plugin details pancake-workflow@pancake-cmo   # expect: Skills (1), MCP servers (1) pancake
```

Then inside a Claude Code session with the plugin enabled:

1. **Authenticate**: `/mcp` → **pancake** → Authenticate. The browser opens Pancake's sign-in +
   workspace picker; after approving, the server shows ✔ Connected. No key or env var exists.
2. **Read**: ask Claude to call `brain_get`. Confirm the response is the test workspace's actual
   company name and ICP, not an error and not another workspace's data.
3. **Safe write**: ask Claude to call `brain_add_keyword` with a throwaway phrase (e.g.
   `smoke-test-keyword`). Confirm a follow-up `brain_get` shows it, then call `brain_archive_item`
   to remove it.

Clean up:

```bash
claude plugin uninstall pancake-workflow@pancake-cmo
claude plugin marketplace remove pancake-cmo
```

Then disconnect the grant in the Pancake app: Settings → MCP → Connected clients → Disconnect.

Last run: 2026-08-19 — the browser OAuth flow (ADR 0055) verified end to end against
beta.getpancake.ai with a real Claude Code client: challenge → discovery → consent → tokens →
`ping`/tools succeed; grant visible and revocable in Settings.

Repeat the full flow against `app.getpancake.ai` after the domain cutover; the beta result is not
evidence that the canonical-host OAuth and provider registrations work.
