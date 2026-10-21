# Pancake

Find the right leads, act on buying signals, and turn your go-to-market strategy into action — directly from Claude Code or Cowork, with your Pancake workspace as the source of truth.

Use Pancake to:

- **Find and qualify leads** using your ideal customer profile, company context, and buying signals.
- **Build and manage Plays** that find prospects and organize your LinkedIn outreach.
- **Review results and improve targeting** with lead feedback, run reports, and your GTM Brain.
- **Stay in control** of credits, schedules, sequence activity, and workspace settings.

## What's included

The plugin connects Claude to Pancake's remote MCP server and includes five skills:

- [`pancake`](skills/pancake/SKILL.md): operating guidance for the workspace's tools.
- [`pancake-daily-leads`](skills/pancake-daily-leads/SKILL.md): find leads within a credit budget you approve.
- [`pancake-review-leads`](skills/pancake-review-leads/SKILL.md): review new leads and explain their fit.
- [`pancake-refresh-icp`](skills/pancake-refresh-icp/SKILL.md): review feedback and proposed targeting changes.
- [`pancake-onboarding`](skills/pancake-onboarding/SKILL.md): finish your Pancake welcome here and create your first Play.

Skills are guidance, not a limit on the tools available. The MCP connection discovers the complete tool catalog allowed for the signed-in member and selected workspace. That includes Brain, leads, signals, Plays, lead finding, LinkedIn sequences, public-page research, activity, credits, and workspace settings. Availability follows the workspace's subscription, permissions, and enabled features; operator-only administration tools are not exposed.

The server supplies tool names, descriptions, input schemas, and safety annotations dynamically through MCP. New server tools do not need to be copied into this plugin. Changes to bundled skills or metadata are delivered as plugin updates.

## Install

In Claude Code, add Pancake's marketplace and install **Pancake**:

```bash
claude plugin marketplace add get-pancake/agent-plugins
claude plugin install pancake@pancake
```

The plugin is **Pancake**. The `@pancake` suffix identifies Pancake's marketplace. This GitHub installation route does not imply that the plugin is already published in Anthropic's public directory. Once the directory listing is published, you can also install Pancake there on its supported surfaces, including Cowork.

For an older installation, see the [migration guide](https://github.com/get-pancake/agent-plugins#upgrade-from-an-older-install).

## Connect your workspace

A Pancake account and a workspace with MCP access are required. In Claude Code, run `/mcp`, select **pancake**, and choose **Authenticate**. Sign in to Pancake in the browser, review the consent screen, and select the workspace to connect. Never paste a password or token into a conversation.

If you don't have a workspace yet, complete onboarding in the Pancake app. Until a workspace is ready, the connection exposes only onboarding status; workspace tools become available after it is linked. The connection refreshes automatically while its authorization remains valid.

Start with a read-only request: “Read my Pancake Brain and summarize our ideal customer profile.” The [smoke test](smoke-test.md) explains how to check the connection and tool discovery without changing workspace data.

## Actions and control

Reading workspace information does not start outreach. Lead-finding runs consume credits: review the estimate and explicitly approve the credit budget before starting. Adding people to a LinkedIn sequence starts real outreach and requires your explicit request. Preparing drafts does not enroll recipients or send messages.

Tools can also update the Brain, lead feedback, targeting, schedules, sequence settings, and workspace settings when you ask. Approving access, connecting external accounts, accepting invitations, and paying remain browser actions.

## Data handling

This plugin transmits data to Pancake. It connects over HTTPS to `https://app.pancake.ai/api/mcp` using a workspace-scoped OAuth connection. Tool calls send the requested arguments and authorization to Pancake; the requested workspace results are returned to Claude. Pancake does not receive the full conversation unless the client includes that content in a tool argument.

The plugin contains Markdown guidance and remote MCP configuration. It does not bundle a local executable, start a local MCP process, install dependencies, or register hooks. It does not include credentials or grant access to other workspaces. Requested research, lead finding, and outreach are performed by Pancake and its service providers as described in the [privacy policy](https://pancake.ai/privacy).

OAuth connection metadata is retained while the grant is active; changes made through tools are retained like other workspace changes. Disconnecting stops future access but does not undo prior actions or delete workspace records. See the privacy policy for retention, deletion requests, and subprocessors. Claude's handling of data returned to it is governed by your agreement with Anthropic.

## Disconnect or uninstall

Revoke the connection in the Pancake app under **Settings → Claude & Codex → Connected clients**. To remove the Claude Code plugin:

```bash
claude plugin uninstall pancake@pancake
```

Uninstalling removes the local plugin, but does not revoke the server-side OAuth connection; disconnect it separately.

## Support and policies

- Plugin configuration, skills, and documentation: [MIT license](LICENSE), with [scope and brand exclusions](NOTICE.md). This does not license the Pancake SaaS or its remote MCP service.
- [Pancake support](https://pancake.ai/support) · [support@pancake.ai](mailto:support@pancake.ai)
- [Privacy policy](https://pancake.ai/privacy)
- [Terms of service](https://pancake.ai/terms)
- [Source and installation guide](https://github.com/get-pancake/agent-plugins)
