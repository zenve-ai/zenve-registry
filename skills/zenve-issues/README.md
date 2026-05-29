# zenve-issues

Skill that teaches an agent to read and manage issues in the **Zenve runtime** via its HTTP API using `curl` — full CRUD plus comments and labels. The issue routes require **no authentication**.

## Install

```bash
zenve skills add --skill zenve-issues
```

## Usage

Best used by an agent running inside the Zenve runtime, where `ZENVE_WORKSPACE_ID` is already in the environment. Just tell Claude:

> "read my assigned issue #42"
> "what does the latest comment on the issue say"
> "list the open issues"
> "comment on the issue that the PR is ready, then close it"
> "create an issue for the login bug and assign react-dev"
> "relabel issue #42 as priority:high"

## What it does

- **Reads** issues — list (filter by `state`, `limit`), get one, list labels.
- **Comments** — list, add, edit, delete comments on an issue.
- **Writes** issues — create, update (partial PATCH: title/body/state/labels/assignees), close/reopen, delete.
- Frames the **assigned-issue workflow** first: read the issue and its latest comment, do the work, post a reply, close it.
- Builds every call from `${ZENVE_RUNTIME_URL:-http://localhost:8001}` and `$ZENVE_WORKSPACE_ID`, with a fallback for discovering the workspace id (`zenve workspace ls`, `GET /api/v1/workspaces`, or `~/.zenve/workspaces.json`) when running manually.

## Notes

- No auth header is needed; `Content-Type: application/json` is only required on requests with a body.
- `ZENVE_WORKSPACE_ID` is injected by the runtime's `claude_code` adapter; `ZENVE_RUNTIME_URL` is not, which is why the examples default it to the local runtime on port `8001`.
