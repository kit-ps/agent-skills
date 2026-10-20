---
name: issue-to-merge-request
description: Drive a GitLab issue to a clean, mergeable MR end to end. Use when the user supplies a repo URL, a token and an issue number, or says "make an MR for issue #N". Creates a branch, implements the issue (test-first), runs the project gate, opens the MR linked to the issue, and folds in reviewer-comment rounds until it is mergeable.
license: MIT
metadata:
  author: Hermes Agent
  version: "1.0.0"
---

# Issue → Merge Request (GitLab)

Given a GitLab repo URL, an access token, and an issue number, produce a merge
request that resolves the issue. The loop is: read the issue, build a branch,
implement (test-first), run the project's own gate, create the MR linked to
the issue, then fold in reviewer comments until it is `can_be_merged`.

Uses the raw GitLab REST mechanics from the Hermes `gitlab-workflow` skill —
load it before any API call. This skill is the orchestration layer on top.
(Standalone reference of the same mechanics ships in
`references/gitlab-api.md` for use outside Hermes.)

## When to use

- "Make an MR for issue #N" against a GitLab repo.
- "Address the comments on MR !N" (a review round).
- The user supplies: a repo URL, a token, and an issue/MR number.

Don't use for: GitHub (use the GitHub skills); a plain commit with no MR;
anything where the repo has no `git` remote configured.

## Before you start

- The repo must be cloned locally with `origin` set and the token reachable
  (set it in the remote URL or retrieve it from memory — never log it).
- The project's own rules (`AGENTS.md`, `README`) and its test/lint/build
  commands (`package.json`, etc.) are the source of truth; read them, don't
  guess the entry commands.

## Procedure

### 1. Read the issue

`GET /api/v4/projects/<enc>/issues/<iid>` with a `PRIVATE-TOKEN` header.
Capture the full title and description — they define acceptance.

### 2. Branch off a fresh base

`git fetch origin main`, then `git checkout -b <prefix>/<slug> origin/main`
(never a stale local `main`). Prefix by intent: `fix/`, `feature/`,
`refactor/`, `feat/`.

### 3. Implement, test-first

- Bug: write the failing test, observe it fail, fix, then observe it pass.
- Feature: implement together with its unit test.
Keep the diff minimal — only what the issue asks.

### 4. Run the gate before each commit

Run the project's tests, lint, and build. All must be green (or the build
must succeed with only pre-existing warnings) before you commit. Stage every
new file and any model/assets the code depends on (`git status`).

### 5. Commit and push

One focused Conventional-Commit message, then `git push -u origin <branch>`.

### 6. Create the MR linked to the issue

`POST /merge_requests` with `{source_branch, target_branch:"main", title,
description}`, appending a real newline + `Closes #<iid>` so GitLab
auto-closes the issue on merge. Then verify with a GET: `state == "opened"`,
`has_conflicts == false`, `merge_status == "can_be_merged"`, and `sha` equals
local `HEAD`. If it reports a conflict, `git rebase origin/main`, resolve,
force-push, re-verify.

### 7. Report and ask

Give the user the MR `web_url`; note the intended merge order if multiple MRs
are queued, and ask "leave for review vs merge now" when appropriate.

### 8. Review rounds

For each new human comment, fix it (test-first for bugs), push, reply on the
MR with the fix rationale + a commit SHA, and re-verify `can_be_merged`.
When another branch merges and introduces a conflict, rebase onto `origin/main`.

## Pitfalls

- **Token hygiene.** Never write the token, or `oauth2:` plus token, into a
  file or command that could be logged. Retrieve it from the remote
  (`git remote get-url origin`, split on `oauth2:`) and use a `PRIVATE-TOKEN`
  header for API calls.
- **Project paths are URL-encoded** (`/` → `%2F`) in `/api/v4/projects/...`.
- **`Closes #N` needs a real newline**, not the literal `\n`; build the body
  in code or a file so GitLab parses the link.
- **Conflict right after a merge is normal.** `docs/adr/` is the usual
  collision point; keep both sides' notes when resolving.
- **Token scope.** A token that can push may lack `api` scope to create MRs.
  Run `gitlab-workflow`'s auth probe before blaming git for a 403.
- **`can_be_merged` recomputes asynchronously.** Re-GET a few seconds after a
  push; don't trust the first read.

## Verification

- `git ls-remote origin <branch>` shows the pushed SHA.
- MR GET returns `state: opened`, `has_conflicts: false`,
  `merge_status: "can_be_merged"`, and the matching `sha`.
- The MR's diff satisfies the issue; the project gate ran green on the final
  commit.
