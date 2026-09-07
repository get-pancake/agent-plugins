# Pancake — ChatGPT and Codex plugin

Connects ChatGPT and Codex to a Pancake workspace through Pancake's hosted MCP server. The
plugin includes the `pancake` skill so the agent reads the GTM Brain before creating
marketing work, respects banned claims and revision tokens, and uses write tools only when asked.

## What it can access

- The approved GTM Brain: company, ICP, voice, personas, messages, objections, competitors, and
  watched keywords.
- Leads, feedback, signal settings, tracked profiles, and lead-finding runs and schedules.
- SEO article briefs, content revisions, approvals, scheduling, and history.
- Requested Brain and signal-setting updates, proposal review, and reversible Brain item management.
- Workspace settings, member invitations, Slack delivery settings, and billing information.
- Campaign status and explicitly requested changes, including real LinkedIn outreach on enrollment.

Lead-finding runs consume credits: preview the cost and get explicit confirmation before starting.
Tools cannot directly publish to a CMS, purchase a plan, approve OAuth grants, or connect Slack.
Destructive changes and access-related actions require the user's explicit instruction or confirmation;
see the bundled skill for each tool's safeguards.

## Authenticate

The first connection opens Pancake's browser sign-in. Sign in with Google or an email link, then
choose the single workspace the connection may access. No API key or environment variable is
needed. Disconnect at any time under **Pancake → Settings → MCP → Connected clients**.

## Source and support

This directory is the source package for Pancake's universal OpenAI plugin listing. See the
[public repository](https://github.com/get-pancake/agent-plugins) to review its contents,
or follow the repository's [support guide](../SUPPORT.md).
