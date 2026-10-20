# GitLab REST / token reference

The MR flow needs these GitLab specifics. This reference makes the skill
self-contained; the Hermes `gitlab-workflow` skill carries richer per-project
diagnostics.

## Token

- Pull the token from the configured remote so it is never logged:
  `git remote get-url origin`, split on `oauth2:` up to `@`.
- API calls authenticate with a `PRIVATE-TOKEN: <token>` header.
- Project paths in API URLs are URL-encoded (`/` → `%2F`):
  `api/v4/projects/<group>%2F<project>`.
- Access roles · 10=Guest (no push) · 20=Reporter (no push) · 30=Developer
  (push) · 40=Maintainer (push + merge) · 50=Owner. A push-capable token may
  still lack `api` scope to create MRs.

## Endpoints

- Read an issue:
  `GET /api/v4/projects/<enc>/issues/<iid>`
- Create an MR:
  `POST /api/v4/projects/<enc>/merge_requests`
  body `{"source_branch","target_branch":"main","title","description"}`.
- Read back / verify:
  `GET /api/v4/projects/<enc>/merge_requests/<iid>`
  → check `state == "opened"`, `has_conflicts == false`,
  `merge_status == "can_be_merged"`, `sha` equals local HEAD.
- MR notes (review comments):
  `GET /api/v4/projects/<enc>/merge_requests/<iid>/notes`
  and
  `POST /api/v4/projects/<enc>/merge_requests/<iid>/notes`
  with `{"body": "..."}`.

## Linking the MR to the issue

Append a real newline then `Closes #<iid>` to the MR description. GitLab
auto-closes the referenced issue when the MR merges. Build the body in code or
a file so the newline is real (a literal `\n` string is not parsed).

## Pitfalls

- MR `merge_status` recomputes asynchronously after a push — re-GET after a
  few seconds before trusting `can_be_merged`.
- Conflicts right after another MR merges are common; the usual collision is
  `docs/adr/`. Rebase onto `origin/main`, keep both sides' ADR notes, resolve,
  force-push, re-verify.
