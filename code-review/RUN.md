# Run Instructions

## Memory

- **Long-term memory** — persisted facts and decisions: `{agent_dir}/memory/long_term.md`
- **Scratch memory** — ephemeral working notes (cleared between runs): `{agent_dir}/memory/scratch.md`

## On Start

1. Read `{agent_dir}/memory/long_term.md` if it exists.
2. Read `{agent_dir}/memory/scratch.md` if it exists.
3. Do **not** preload an architect skill — the stack is unknown until the PR is inspected. The right skill is loaded in step 3 of *Executing the Task*.

## Executing the Task

1. **Read the PR.** Extract the PR number from the run prompt. Run `gh pr view <N> --json title,body,headRefName,baseRefName,labels,files,author` and `gh pr diff <N>`. Read any linked issue and any source task/plan path referenced in the PR body.
2. **Detect the stack** per `AGENTS.md §3` (React, FastAPI, mixed, or unknown).
3. **Load the matching architect skill** — `react-architect`, `fastapi-architect`, or both for mixed PRs. If the stack is unknown, signal `RUN_NEEDS_INPUT` and stop.
4. **Walk the review checklist** in `AGENTS.md §5` against the diff.
5. **Post the review.**
   - Optional: one or more `gh pr comment <N> --body "..."` for individual observations.
   - Required: attempt exactly one `gh pr review <N> --approve|--request-changes|--comment --body "..."` as the final review action.
   - **If `gh pr review` fails** (e.g. GitHub blocks self-review because both identities are the same user), do not signal `RUN_FAILED` and do not retry. Instead, include the full review content directly in your final response before the signal line — the orchestrator will post it as a comment.
6. **Use `{agent_dir}/memory/scratch.md`** for notes needed within this run only (e.g. running list of blockers as you read the diff).
7. **Use `{agent_dir}/memory/long_term.md`** for durable observations — recurring blocker patterns, conventions you noticed, project-specific exceptions to skill rules.
8. Never edit files in the worktree. Never run state-mutating `git` or `gh` commands other than comment/review.

## IMPORTANT: Before ending the session

1. Update `{agent_dir}/memory/long_term.md` with durable observations if any new patterns or project facts emerged.
2. Clear `{agent_dir}/memory/scratch.md` or leave a brief summary for the next run.
3. Confirm exactly one `gh pr review <N>` call was made — or that it failed due to self-review restrictions and you posted the review as a plain `gh pr comment` instead — or that you are signaling `RUN_NEEDS_INPUT` / `RUN_FAILED` and have not posted a review.
4. Produce a final response that the gateway will store as the run result.

Examples of final responses:

```markdown
RUN_OK: approved PR #142 — small frontend change, scope matches task 03-list-view, render states all present
```

```markdown
RUN_CHANGES_REQUESTED: PR #143 — service logic in route handler, missing alembic migration for new column
```

When `gh pr review` failed and the review content must be in the final response:

```markdown
- `server/routes/users.py:42` — business logic in route handler, move to service layer
- `packages/db/models.py` — new column added without an Alembic migration

RUN_CHANGES_REQUESTED: PR #143 — service logic in route handler, missing alembic migration for new column
```

## Signalling Outcomes

End your response with one of these signal lines so the gateway can parse the outcome:

| Signal | Meaning |
|--------|---------|
| `RUN_OK: <note>` | PR approved; brief note of what was reviewed |
| `RUN_CHANGES_REQUESTED: <note>` | Blockers found; review posted with `--request-changes` (or as a comment if self-review was blocked) |
| `RUN_NEEDS_INPUT: <question>` | Cannot review — missing context, unrecognized stack, or unreadable source task |
| `RUN_FAILED: <reason>` | Review tooling failed (e.g. `gh` error) — not "the PR is bad" |
