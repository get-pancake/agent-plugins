# Smoke test — Pancake

Run this after an install or plugin update, using a test workspace. Record the installation source, plugin version, source commit, and the surface tested. A local or GitHub installation is not proof of a fresh installation from Anthropic's public directory.

## Installation

```bash
claude plugin marketplace add get-pancake/agent-plugins
claude plugin install pancake@pancake
claude plugin details pancake@pancake
```

Expect the name **Pancake**, five skills, and one remote MCP server named `pancake`. For a directory publication check, start with a fresh isolated profile and install from the public directory instead. Check Claude Code and Cowork separately once the listing is published.

## Read-only checks

1. **Connection:** use an existing authorization for the test workspace. If there isn't one, get the workspace owner's confirmation before approving a new standing OAuth connection. In Claude Code, use `/mcp` → **pancake** → Authenticate, then complete Pancake sign-in and workspace selection in the browser.
2. **Tool discovery:** inspect the tools returned by the remote server. The plugin must not apply a fixed allowlist. Compare the catalog for the actual connection, accounting for workspace permissions, subscription, and enabled features. Before onboarding is complete, only `onboarding_status` is expected. Operator-only tools must remain unavailable.
3. **Read:** call `workspace_get` or `brain_get`. Confirm the result belongs to the selected test workspace and contains no other workspace's data. Do not publish customer data, tokens, or private tool output in test evidence.
4. **No side effects:** do not run lead finding, enroll recipients, send messages, invite members, or change schedules to verify discovery. Tool availability is not permission to execute it.

## Optional write check

Only if the test workspace owner explicitly authorizes it, add a disposable watched keyword with `brain_add_keyword`, read it back with `brain_get`, and remove that exact keyword with `brain_archive_item`. Record the persisted result and cleanup separately from the read-only checks.

## Cleanup

```bash
claude plugin uninstall pancake@pancake
claude plugin marketplace remove pancake
```

Remove only the isolated test installation. If you created a test OAuth connection, disconnect that specific connection in **Settings → Claude & Codex → Connected clients**; do not revoke an existing connection still used elsewhere.
