# Install the Pancake skills

These are instructions for the coding agent reading this file. Install Pancake's four
skills into the skills directory of the client running this session. This is Markdown, not
a shell script: read it and execute only the steps that apply to your client.

A skill is a directory holding one `SKILL.md`. Nothing here is secret, workspace-specific, or
generated for one user: the same four files serve every Pancake workspace. They are published
from Pancake's source repository on every change, so always fetch them from the URLs below
rather than from a copy you remember.

## The four skills

| Skill                  | Purpose                                                                  | File                                                                                                          |
| ---------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `pancake`              | How to work well with the Pancake MCP tools (read before writing, voice) | https://raw.githubusercontent.com/get-pancake/agent-plugins/main/pancake/skills/pancake/SKILL.md              |
| `pancake-daily-leads`  | Playbook: find N leads today under a credit ceiling                      | https://raw.githubusercontent.com/get-pancake/agent-plugins/main/pancake/skills/pancake-daily-leads/SKILL.md  |
| `pancake-review-leads` | Playbook: judge the leads that arrived since the last check              | https://raw.githubusercontent.com/get-pancake/agent-plugins/main/pancake/skills/pancake-review-leads/SKILL.md |
| `pancake-refresh-icp`  | Playbook: resolve Brain proposals and feedback, then show the Brain diff | https://raw.githubusercontent.com/get-pancake/agent-plugins/main/pancake/skills/pancake-refresh-icp/SKILL.md  |

Each file must be saved as `<skills directory>/<skill name>/SKILL.md`. The directory name must
equal the skill name exactly; the `name` field in each file's frontmatter already matches it.

## Before installing

1. If the client already has a Pancake plugin installed (Claude Code or Codex via
   `get-pancake/agent-plugins`), it ships these same skills. Do not install a second copy;
   stop and say so.
2. Find the skills directory this client reads. Prefer the user-level directory so the skills
   are available in every project; use a project directory only if the user asked for that.
   Known locations:

   | Client                          | User-level skills directory | Project-level                  |
   | ------------------------------- | --------------------------- | ------------------------------ |
   | Claude Code                     | `~/.claude/skills/`         | `.claude/skills/`              |
   | Codex CLI                       | `~/.codex/skills/`          | `.codex/skills/`               |
   | fx                              | `~/.fx/skills/`             | `skills/` or `.agents/skills/` |
   | Factory Droid                   | `~/.factory/skills/`        | `.factory/skills/`             |
   | Cursor                          | `~/.cursor/skills/`         | `.cursor/skills/`              |
   | Any client reading Agent Skills | `~/.agents/skills/`         | `.agents/skills/`              |

   If your client is not listed, use its documentation or `--help` to find where it discovers
   `SKILL.md` files. If it has no skills support at all, say so and stop; do not paste the skill
   text into a configuration file or a system prompt.

3. Preserve every other skill already present. Create only the four directories above and
   write only their `SKILL.md`; never delete or rewrite anything else.

## Install

For each of the four skills, download the file at its URL and save it to
`<skills directory>/<skill name>/SKILL.md`, creating the directory. With a shell that is:

```sh
SKILLS_DIR=~/.agents/skills   # replace with the directory found above
for skill in pancake pancake-daily-leads pancake-review-leads pancake-refresh-icp; do
  mkdir -p "$SKILLS_DIR/$skill"
  curl -fsSL "https://raw.githubusercontent.com/get-pancake/agent-plugins/main/pancake/skills/$skill/SKILL.md" \
    -o "$SKILLS_DIR/$skill/SKILL.md"
done
```

Without a shell, fetch each URL with your file or web tool and write the content unchanged. If a
client offers its own installer (for example fx's `/skills install get-pancake/agent-plugins`),
you may use it instead, provided it installs exactly these four skills.

Re-running this procedure later updates the skills in place; it is the intended way to pick up
a newer version.

## Verify

1. Confirm each of the four `SKILL.md` files exists at its path and begins with
   `---` followed by `name: <skill name>`.
2. If the client lists skills (Claude Code and fx: `/skills`), reload or restart it and check
   that the four appear. Loading is on demand in most clients: seeing them listed is enough.
3. Report which directory received the skills and whether the client needs a restart. The
   skills describe how to use Pancake's tools; they do not connect the client. If Pancake's MCP
   server is not connected yet, follow https://getpancake.ai/install.md next.
