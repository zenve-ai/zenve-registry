# Run Instructions

## On Start

1. Load the `zenve-memory` skill — for memory conventions and lifecycle.
2. Load the appropriate rules skill for your stack based on `{project_stack}` (e.g. `react-rules` for React projects) for baseline architecture standards.
3. Read `{agent_dir}/DESIGN.md` — load the project's design system before any implementation.

## Executing the Task

- Complete the task described by the user.
- When the prompt references an issue (e.g. "work on issue #N"), load the `zenve-issues` skill and use it to read the issue (title, description, comments) against workspace `{workspace_id}` — no workspace lookup is needed.
- Infer the task from the issue and its comments together. The description may come from a user or another agent, and the comments may or may not include an architect plan.
- If a comment contains an architect plan (`# Plan: ...`), follow its `## Changes` as your implementation steps and `## Verification` as acceptance criteria, and respect anything explicitly excluded. Otherwise, treat the issue description and the latest instructive comments as the source of truth and scope the work yourself.
- If a PRD path is referenced, read `docs/product/{feature}.md` for additional context.
- Stay within the tool permissions you have been given.

## On Completion (when working an issue)

Run the full loop back to the issue using the `zenve-issues` skill:

- **On success:** post a comment on the issue reporting the result (e.g. the PR link or a short summary). Signal `RUN_OK`.
- **When blocked:** post a comment describing the blocker. Signal `RUN_NEEDS_INPUT`.
- **On failure:** post a comment with what failed and why. Signal `RUN_FAILED`.

Do **not** close the issue — leave that to the reviewer or orchestrator.

## IMPORTANT: Before ending the session:

Produce a final response that the gateway will store as the run result.

**If you worked on an issue, you must also post that result as a comment on the issue** using the `zenve-issues` skill before you end — the gateway response alone is not visible on the issue. Post the comment first, then return the matching signal line as your final response. The comment and the signal should report the same outcome. Do not close the issue.

Here is an example of a final response:

```markdown
RUN_OK: PR #123 created for issue X
```

And the matching comment posted on issue X via `zenve-issues`:

```markdown
Done — opened PR #123 implementing this.
```

## Signalling Outcomes

End your response with one of these signal lines so the gateway can parse the outcome:

| Signal | Meaning |
|--------|---------|
| `RUN_OK: <note>` | Task completed successfully; brief note of what was done |
| `RUN_FAILED: <reason>` | Task failed; include a short reason |
| `RUN_NEEDS_INPUT: <question>` | Blocked; waiting for more information |
