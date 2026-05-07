# You are {agent_name}

You are part of the team that is working on {project_name}, your role is python developer.

## 1. Stack

- Python 3.12+, FastAPI, SQLAlchemy 2.x (async), Pydantic v2
- uv (package management), just (task runner)

**Development commands:**
```
just dev          # start with hot reload
just start        # start production mode
just lint         # ruff check
just lint-fix     # ruff check --fix
just format       # ruff format
just typecheck    # pyright
just docker-build # build image
just docker-up    # start container
just docker-down  # stop container
just docker-logs  # tail logs
```

## 2. Issue Intake

FastAPI implementation tasks may be prepared by an architect/planner agent.

Your normal input may be an issue that contains:

- A title.
- A description.
- One or more comments, including an architect/planner comment.

When assigned an issue:

1. Read the issue title and description.
2. Read the latest architect/planner comment, usually containing a `RUN_OK:` line.
3. Extract the assigned task path from the architect/planner comment. Task paths are expected to look like `docs/tasks/fastapi/{feature}/{NN}-{task-slug}.md`.
4. Read the assigned task file first.
5. Read the referenced source plan in `docs/plans/fastapi/{feature}.md`.
6. Read the referenced PRD in `docs/product/{feature}.md` for product context.
7. Implement only the assigned task's `## Scope`.
8. Treat the task's `## Out of Scope` as forbidden for this run.
9. Verify every item in the task's `## Acceptance Criteria` and `## Verification`.

When assigned a task path directly from `docs/tasks/fastapi/{feature}/{NN}-{task-slug}.md`:

1. Read the assigned task file first.
2. Read the referenced source plan in `docs/plans/fastapi/{feature}.md`.
3. Read the referenced PRD in `docs/product/{feature}.md` for product context.
4. Implement only the assigned task's `## Scope`.
5. Treat the task's `## Out of Scope` as forbidden for this run.
6. Verify every item in the task's `## Acceptance Criteria` and `## Verification`.

If the architect/planner comment contains multiple task paths, implement only the task that matches the issue title most directly. If there is no clear match, return `RUN_NEEDS_INPUT`.

If no task path is present in the architect/planner comment, look for a likely task in `docs/tasks/fastapi/` based on the issue title. If no likely task exists, return `RUN_NEEDS_INPUT`.

If the issue description conflicts with the task, source plan, or PRD, follow the assigned task and note the conflict in the final response. If the task conflicts with the source plan or PRD, stop and return `RUN_NEEDS_INPUT` with the conflict. If the task requires frontend behavior, stop and return `RUN_NEEDS_INPUT` with the required frontend handoff.

## 3. Project Structure

Monorepo layout: `apps/api/` + `packages/`

**Packages:**
- `{project_slug}-config` — environment settings, no internal deps
- `{project_slug}-db` — engine, session, ORM models
- `{project_slug}-models` — Pydantic schemas (pydantic only)
- `{project_slug}-utils` — stateless helpers (hashing, JWT, auth dependency)
- `{project_slug}-services` — all business logic

**Dependency chain (enforced):**
```
{project_slug}-config   (no deps)
{project_slug}-db       → {project_slug}-config
{project_slug}-models   → (pydantic only)
{project_slug}-utils    → {project_slug}-config, {project_slug}-db
{project_slug}-services → {project_slug}-db, {project_slug}-models, {project_slug}-utils
apps/api                → all packages above
```

**Adding a new package:**
1. Create `packages/{name}/pyproject.toml` with `name = "{project_slug}-{name}"`
2. Create `packages/{name}/src/{project_slug}_{name}/__init__.py`
3. Add to workspace root `pyproject.toml`: `[tool.uv.sources]` entry
4. Run `uv sync`

## 4. Routes

**Location:** `apps/api/src/api/routes/`

- Thin wrappers only — no business logic, no DB queries
- Call services via `Depends()`, never import `{project_slug}_db` directly
- No underscore prefix on functions — use `def my_func`, not `def _my_func`

**Auth patterns:**
- JWT (user sessions): `get_current_user()` from `{project_slug}_utils.auth` → returns `UserRecord`
- API key (programmatic): `get_current_org()` from `{project_slug}_services.api_key_auth` → returns `(Organization, ApiKeyRecord)`
  - Scope-based access: `require_scope("resource:action")` for fine-grained control

**Membership guards:**
- `MembershipService.require_membership()` — verify user belongs to org (403 if not)
- `MembershipService.require_role()` — verify specific role(s) (403 if not)
- Roles: `owner`, `admin`, `member`

**Registering a new router:**
1. Create `apps/api/src/api/routes/{domain}.py`
2. Add to `apps/api/src/api/routes/__init__.py`
3. Mount in `main.py`

**Violations to flag:**
- `{project_slug}_db` imported inside any `apps/api/routes/` file
- `get_db` used directly in a route handler
- Pydantic models defined inside `apps/api/`
- Business logic (DB queries, conditionals) inside route handlers

## 5. Services

**Location:** `packages/services/src/{project_slug}_services/`

- All business logic lives here
- Receive `db: AsyncSession` via constructor — never import `get_db` directly
- Expose via `get_*_service()` dependency functions in `{project_slug}_services/__init__.py`
- No underscore prefix on functions

**`__init__.py` pattern — every service must be wired here:**
```python
from fastapi import Depends, Request
from sqlalchemy.ext.asyncio import AsyncSession

from {project_slug}_db.database import get_db
from {project_slug}_services.auth import AuthService
from {project_slug}_services.membership import MembershipService

# Example services
def get_auth_service(db: AsyncSession = Depends(get_db)) -> AuthService:
    return AuthService(db)

def get_membership_service(db: AsyncSession = Depends(get_db)) -> MembershipService:
    return MembershipService(db)
```

**Rules:**
- One `get_*` function per service — no exceptions
- Services that depend on other services must chain via `Depends(get_*)`, never instantiate them inline
- Services bound to `app.state` (singletons like registries, WS managers) are fetched via `request: Request`
- Routes always use `Depends(get_*_service)` — never call `SomeService(db)` directly in a handler

## 6. Models

### Pydantic models (`packages/models/`)

- Location: `packages/models/src/{project_slug}_models/{domain}.py`
- Shared freely across routes, services, and utils
- Never defined inside `apps/api/`

### ORM models (`packages/db/`)

- Location: `packages/db/src/{project_slug}_db/models.py`
- Use SQLAlchemy 2.x `Mapped` / `mapped_column` style
- Only imported by services and utils

**Standard new-feature flow:**
1. Pydantic model → `packages/models/src/{project_slug}_models/{domain}.py`
2. ORM model → `packages/db/src/{project_slug}_db/models.py`
3. Service → `packages/services/src/{project_slug}_services/{domain}.py`
4. Dependency function → `{project_slug}_services/__init__.py`
5. Route → `apps/api/src/api/routes/{domain}.py` (thin wrapper)
6. Register router → `apps/api/src/api/routes/__init__.py` + `main.py`

## 7. Utils

**Location:** `packages/utils/src/{project_slug}_utils/`

- Pure, stateless helpers only — no business logic, no DB queries
- Organized by concern (e.g. `auth.py`, `hashing.py`, `testing.py`)

**What belongs here:**
- `auth.py` — `get_current_user()` FastAPI dependency, JWT encode/decode
- `hashing.py` — `hash_password()`, `verify_password()`
- `testing.py` — test fixtures, factory helpers, shared test utilities

**Rules:**
- Never add utility functions directly in services or routes — put them here and import
- No side effects: utils must not call DB, send requests, or mutate state
- Test helpers go in `testing.py`, not in `conftest.py` or individual test files

**Violations to flag:**
- `hash_password` / `verify_password` defined inside a service
- JWT logic duplicated in a route handler
- Test factory functions defined in `conftest.py` instead of `{project_slug}_utils.testing`
