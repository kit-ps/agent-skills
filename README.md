# KIT PS Agent Skills

Agent skills from the KIT Privacy & Security group.

| Skill | Description |
|---|---|
| [`paper-review`](skills/paper-review/) | Pre-submission review of your own paper draft, with security and privacy checks. |
| [`issue-to-merge-request`](skills/issue-to-merge-request/) | Turn a GitLab issue into a tested merge request through a GitLab MCP server only (no git), and work through review rounds. See [server setup](#issue-to-merge-request-server-setup). |

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

## `issue-to-merge-request` server setup

For the person connecting the GitLab MCP server; the agent does not need this.

The community server [zereight/gitlab-mcp](https://github.com/zereight/gitlab-mcp), pinned, with
a project access token (role **Developer**, scope `api`, short expiry). In Hermes, store the
token with `hermes config set GITLAB_TOKEN '...'` and add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  kit-gitlab:
    command: npx
    args: ["-y", "@zereight/mcp-gitlab@2.3.5"]
    env:
      GITLAB_API_URL: https://gitlab.kit.edu/api/v4
      GITLAB_PERSONAL_ACCESS_TOKEN: ${GITLAB_TOKEN}
      GITLAB_ALLOWED_PROJECT_IDS: "PASTE-YOUR-PROJECT-ID-HERE"
      ENABLE_STRICT_PROJECT_SCOPE: "true"
      GITLAB_PERMISSION_MODE: modify
      GITLAB_TOOLSETS: orbit
      GITLAB_TOOLS: "whoami,get_project,get_issue,list_issue_discussions,list_merge_requests,get_merge_request,mr_discussions,get_branch,get_repository_tree,get_file_contents,create_branch,push_files,create_merge_request,create_merge_request_discussion_note"
      GITLAB_DENIED_TOOLS_REGEX: "^(orbit_query|orbit_get_schema|orbit_get_status|orbit_list_tools|discover_tools)$"
```

`GITLAB_TOOLS` only adds tools to the enabled toolsets, so this enables the tiny `orbit` toolset
and denies all of it: the result is exactly the 14 tools listed. `hermes mcp test kit-gitlab` must
report **14 tools**. The server silently ignores a deny pattern it considers invalid, which brings
`discover_tools` (and with it merge and approve) back, so check the count after every change.

The MCP config limits the agent's tools; these settings are what hold if the agent is tricked:

- **Sandbox:** `hermes config set terminal.backend docker`, with no mounts of your home folder,
  `~/.ssh` or `~/.hermes`, and no GitLab variables forwarded. On the local backend the agent's
  shell can read the token and use your own SSH keys. Check from the agent's terminal:
  `env | grep -i gitlab` prints nothing.
- **Protected default branch:** "Allowed to merge: Maintainers", "Allowed to push and merge: No
  one", force push off, and no other rule (wildcards included) giving Developers more.
- **Approvals:** "Prevent approval by author" and "Prevent approvals by users who add commits" on.
- **Merge method:** "Merge commit".
- **CI/CD:** every secret variable **Protected** and Masked; pipelines on the agent's branches run
  its code.

## Licence

MIT. Skills that adapt third-party material include their own `NOTICE.md` and `LICENSES/`.
