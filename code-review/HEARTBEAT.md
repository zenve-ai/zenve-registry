# {agent_name} — Heartbeat

A heartbeat is an autonomous, timer-driven invocation — no caller prompt is provided.

## Tasks

<!-- No tasks defined yet. Add tasks here — each will run on every tick. -->

## On Each Tick

1. Read `{agent_dir}/memory/long_term.md` if it exists.
2. Check the Tasks list above.
3. If no tasks are defined, stop and signal `HEARTBEAT_OK: nothing to do`.
4. Otherwise, execute each task in order.
5. Update `{agent_dir}/memory/long_term.md` with durable observations.
6. Signal `HEARTBEAT_OK: <note>` summarising what was done.

## Before ending the session:

1. Update `{agent_dir}/memory/long_term.md` with durable observations if needed.
2. Clear `{agent_dir}/memory/scratch.md` or leave a brief summary for the next run.
3. Produce a final response that the gateway will store as the run result.

## Signalling Outcomes

End each heartbeat run with one of these signals:

| Signal | Meaning |
|--------|---------|
| `HEARTBEAT_OK: nothing to do` | No tasks defined; tick skipped |
| `HEARTBEAT_OK: <note>` | Tasks completed; brief note of what was done |
| `HEARTBEAT_FAILED: <reason>` | Tick failed; needs attention |
| `HEARTBEAT_NEEDS_INPUT: <question>` | Blocked; cannot proceed autonomously |

## Constraints

- Heartbeat runs have a hard timeout.
- Do not start long-running processes that outlive the tick.
- Do not prompt for user input — you are running unattended.
- Do not modify files. The review_pr worktree is read-only and edits are discarded.
