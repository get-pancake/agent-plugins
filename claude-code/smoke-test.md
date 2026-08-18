# Smoke test — Claude Code plugin

Run this after any install/config change to `agent-plugins/claude-code/` or the shared skill,
against a real (non-production-critical) Pancake workspace.

```bash
export PANCAKE_MCP_API_KEY="sk-..."   # from Settings → MCP on the test workspace

claude plugin marketplace add ./agent-plugins/claude-code
claude plugin install pancake-workflow@pancake-cmo
claude plugin details pancake-workflow@pancake-cmo   # expect: Skills (1), MCP servers (1) pancake
```

Then inside a Claude Code session with the plugin enabled:

1. **Read**: ask Claude to call `brain_get`. Confirm the response is the test workspace's actual
   company name and ICP, not an error and not another workspace's data.
2. **Safe write**: ask Claude to call `brain_add_keyword` with a throwaway phrase (e.g.
   `smoke-test-keyword`). Confirm a follow-up `brain_get` shows it, then call `brain_archive_item`
   to remove it.

Clean up:

```bash
claude plugin uninstall pancake-workflow@pancake-cmo
claude plugin marketplace remove pancake-cmo
unset PANCAKE_MCP_API_KEY
```

Last run: 2026-08-18 against Claude Code 2.1.234 — marketplace add, install, and
`plugin details` (1 skill, 1 MCP server) verified locally. The two live MCP calls (`brain_get`,
`brain_add_keyword`/`brain_archive_item`) still need running against a real workspace + API key
before this plugin is called fully validated — track under PAN-139.
