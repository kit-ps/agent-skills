# KIT PS Agent Skills

Agent skills from the KIT Privacy & Security group.

| Skill | Description |
|---|---|
| [`paper-review`](skills/paper-review/) | Pre-submission review of your own paper draft, with security and privacy checks. |
| [`issue-to-merge-request`](skills/issue-to-merge-request/) | Turn a GitLab issue into a tested merge request and work through review rounds. Needs the [zereight/gitlab-mcp](https://github.com/zereight/gitlab-mcp) server. |

## Install

**Any agent** with the [`skills`](https://github.com/vercel-labs/skills) CLI (pick `-a claude-code`, `codex`, `opencode` or `hermes-agent`; add `-g` to install for your user instead of the project):

```sh
npx skills add https://github.com/kit-ps/agent-skills --skill paper-review -a claude-code
```

**Claude Code** plugin:

```
/plugin marketplace add kit-ps/agent-skills
/plugin install paper-review@kit-ps-agent-skills
```

**Hermes:**

```sh
hermes skills install kit-ps/agent-skills/skills/paper-review
```

## Licence

MIT. Skills that adapt third-party material include their own `NOTICE.md` and `LICENSES/`.
