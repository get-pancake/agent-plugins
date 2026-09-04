# Pancake — ChatGPT and Codex plugin

Connects ChatGPT and Codex to a Pancake workspace through Pancake's hosted MCP server. The
plugin includes the `pancake-cmo-brain` skill so the agent reads the GTM Brain before creating
marketing work, respects banned claims and revision tokens, and uses write tools only when asked.

## What it can access

- The approved GTM Brain: company, ICP, voice, personas, messages, objections, competitors, and
  watched keywords.
- Qualified leads, lead details, member feedback, signal settings, and lead-finding run history.
- The SEO publication calendar and article plans.
- Requested Brain and signal-setting updates, lead feedback, and reversible Brain item management.

The tenant connector cannot start paid lead-finding runs or publish SEO content.

## Authenticate

The first connection opens Pancake's browser sign-in. Sign in with Google or an email link, then
choose the single workspace the connection may access. No API key or environment variable is
needed. Disconnect at any time under **Pancake → Settings → MCP → Connected clients**.

## Source and support

This directory is the source package for Pancake's universal OpenAI plugin listing. See the
[public repository](https://github.com/get-pancake/agent-plugins) to review its contents,
or follow the repository's [support guide](../SUPPORT.md).
