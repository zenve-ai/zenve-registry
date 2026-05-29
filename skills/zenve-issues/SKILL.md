---
name: zenve-issues
description: This skill should be used when an agent needs to read or manage issues in the Zenve runtime — "read my assigned issue", "what issue am I working on", "list open issues", "comment on the issue", "reply to the architect", "close issue #N", "reopen issue", "create an issue", "update / relabel / reassign an issue", or "manage zenve issues". Drives the Zenve runtime issues HTTP API (CRUD + comments + labels) with curl. No authentication required.
version: 1.0.0
---

# Zenve Issues

Read and manage issues by calling the Zenve runtime's issues HTTP API with `curl`. The runtime is a local FastAPI server; the issue routes require **no authentication**. Use this whenever you are assigned an issue, need to read its comments, post a reply, change its state/labels/assignees, or create new issues.

## 1. Setup

Every call is built from two variables. Set them once per session:

```bash
BASE="${ZENVE_RUNTIME_URL:-http://localhost:8001}"   # runtime base URL
WS="$ZENVE_WORKSPACE_ID"                              # injected when you run inside the runtime
```

- When you are invoked by the Zenve runtime (the `claude_code` adapter), `ZENVE_WORKSPACE_ID` is already in your environment along with `ZENVE_AGENT_SLUG`, `ZENVE_PROJECT_SLUG`, and `ZENVE_RUN_ID`. You normally do **not** need to look up the workspace id.
- `ZENVE_RUNTIME_URL` is **not** always injected. The `${ZENVE_RUNTIME_URL:-http://localhost:8001}` default covers the standard local runtime on port `8001`.
- No auth header is needed. Send `-H "Content-Type: application/json"` only on requests with a body.
- Recommended flags: `curl -sS` (quiet but show errors). Add `-f` when you want a non-2xx status to fail the command. Pipe JSON through `jq` for readability.

All issue endpoints live under:

```
$BASE/api/v1/workspaces/$WS/issues
```

## 2. Finding your workspace_id (fallback)

Only needed if `$ZENVE_WORKSPACE_ID` is empty — e.g. you are running this manually outside the runtime. A `workspace_id` is a 12-char hex string (like `aa88c4a62697`), **not** a project name. Resolve it any of these ways:

```bash
zenve workspace ls                              # CLI: lists registered workspaces
curl -sS "$BASE/api/v1/workspaces" | jq         # API: id → path mapping
cat ~/.zenve/workspaces.json | jq               # registry file on disk
```

Match the workspace whose `path` is your project directory, then `WS="<that id>"`.

## 3. Primary workflow — working an assigned issue

This is the loop you run most often: read the issue and its latest comment, do the work, report back, then close it.

```bash
ISSUE=42   # the issue number you were assigned

# 1. Read the issue (title, body, state, labels, assignees)
curl -sS "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" | jq

# 2. Read the discussion — the latest comment is usually the relevant instruction
curl -sS "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments" | jq

# ... do the work the issue describes ...

# 3. Post a comment reporting what you did (e.g. a PR link or a question)
curl -sS -X POST "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments" \
  -H "Content-Type: application/json" \
  -d '{"body":"Opened PR #123 implementing the task."}' | jq

# 4. Close the issue when the work is done
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" \
  -H "Content-Type: application/json" \
  -d '{"state":"closed"}' | jq
```

If you are blocked, post a comment with your question and leave the issue open rather than closing it.

## 4. Reading issues

```bash
# List open issues (default state is "open")
curl -sS "$BASE/api/v1/workspaces/$WS/issues" | jq

# List closed issues, capped at 20
curl -sS "$BASE/api/v1/workspaces/$WS/issues?state=closed&limit=20" | jq

# List all issues regardless of state
curl -sS "$BASE/api/v1/workspaces/$WS/issues?state=all" | jq

# Get a single issue by number
curl -sS "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" | jq

# Just the titles of open issues
curl -sS "$BASE/api/v1/workspaces/$WS/issues" | jq -r '.[] | "#\(.id) \(.title)"'

# List the labels used in this workspace
curl -sS "$BASE/api/v1/workspaces/$WS/issues/labels" | jq
```

Query params on the list endpoint: `state` (default `open`) and `limit` (optional integer).

## 5. Comments

```bash
# List all comments on an issue (oldest → newest)
curl -sS "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments" | jq

# Add a comment (body is required)
curl -sS -X POST "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments" \
  -H "Content-Type: application/json" \
  -d '{"body":"Working on this now."}' | jq

# Edit a comment you made
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments/7" \
  -H "Content-Type: application/json" \
  -d '{"body":"Updated: PR is now merged."}' | jq

# Delete a comment (returns 204, empty body)
curl -sS -X DELETE "$BASE/api/v1/workspaces/$WS/issues/$ISSUE/comments/7"
```

## 6. Writing issues

```bash
# Create an issue — only "title" is required
curl -sS -X POST "$BASE/api/v1/workspaces/$WS/issues" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Login button is misaligned",
    "body": "On mobile the login button overflows the card.",
    "labels": ["bug", "ui"],
    "assignees": ["react-dev"]
  }' | jq

# Update an issue — PATCH is partial, send only the fields that change
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" \
  -H "Content-Type: application/json" \
  -d '{"title":"Login button overflows card on mobile"}' | jq

# Close / reopen
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" \
  -H "Content-Type: application/json" -d '{"state":"closed"}' | jq
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" \
  -H "Content-Type: application/json" -d '{"state":"open"}' | jq

# Replace labels / assignees (the arrays are replaced wholesale, not merged)
curl -sS -X PATCH "$BASE/api/v1/workspaces/$WS/issues/$ISSUE" \
  -H "Content-Type: application/json" \
  -d '{"labels":["bug","priority:high"],"assignees":["fastapi-dev"]}' | jq

# Delete an issue (returns 204, empty body) — destructive, confirm intent first
curl -sS -X DELETE "$BASE/api/v1/workspaces/$WS/issues/$ISSUE"
```

Notes:
- PATCH is partial: omitted fields are left unchanged. To clear a field, send an explicit empty value (e.g. `"labels": []`).
- `labels` and `assignees` are set, not appended — to add one label, send the full intended list.
- DELETE responds `204 No Content`; there is no JSON body, so don't pipe it to `jq`.

## 7. Schema reference

**Issue (response):**

| Field | Type | Notes |
|---|---|---|
| `id` | int | Issue number (read-only) |
| `title` | string | |
| `body` | string | Description, default `""` |
| `state` | string | `open` or `closed` |
| `labels` | string[] | |
| `assignees` | string[] | Usernames / agent slugs |
| `created_at` | string | ISO 8601 |
| `updated_at` | string | ISO 8601 |
| `url` | string \| null | External URL (e.g. GitHub) when applicable |

**Create issue body** — `title` (required); `body`, `labels`, `assignees` (optional).
**Update issue body** — all optional: `title`, `body`, `state`, `labels`, `assignees`.

**Comment (response):** `id` (int), `issue_id` (int), `body` (string), `author` (string), `created_at`, `updated_at`.
**Create / update comment body** — `body` (required on create, optional on update).

The only enumerated value is `state`: `open` | `closed`. Unknown fields in a request body are ignored.

## 8. Errors & tips

- Errors return JSON shaped `{"detail": "..."}` with a matching HTTP status:
  - `404` — issue or comment not found (e.g. `{"detail":"Issue #999 not found"}`)
  - `422` — invalid input (missing required field, wrong type)
  - `502` — external adapter error (e.g. workspace uses the GitHub backend but no token is configured)
- To branch on success/failure, use `-f` and check the exit code, or capture the status: `curl -sS -o /tmp/out -w '%{http_code}' ...`.
- A `204` (DELETE) has no body — checking the status code is the only confirmation.
- Quick check that the runtime is up: `curl -sf "$BASE/healthz"`.
