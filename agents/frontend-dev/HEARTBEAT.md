# {agent_name} — Heartbeat

A heartbeat is an autonomous, timer-driven invocation — no caller prompt is provided.

## Tasks

On each tick, execute the following tasks in order:

<!-- No tasks defined yet. Add tasks here — each will run on every tick. -->

> The `zenve-issues` skill is available if a defined tick task needs to read or list issues (e.g. polling for assigned open issues). No issue task is defined by default.
>
> **Whenever a tick works on an issue, post a result comment on that issue via `zenve-issues` before the tick ends** — reporting what was done, the blocker, or the failure. Do **not** close the issue. This is the same convention as a normal run (see `RUN.md`).

## On Each Tick

1. Read `{agent_dir}/memory/long_term.md` — load persisted context.
2. Check the Tasks list above.
3. If no tasks are defined, stop and signal `HEARTBEAT_OK: nothing to do`.
4. Otherwise, execute each task in order.
5. Update `{agent_dir}/memory/long_term.md` with results and observations.
6. Signal `HEARTBEAT_OK: <note>` summarising what was done.

## Before ending the session:

1. If this tick worked on any issue, post a result comment on it via `zenve-issues` (do not close it).
2. Update `{agent_dir}/memory/long_term.md` with any durable observations.
3. Clear `{agent_dir}/memory/scratch.md` or leave a brief summary for the next run.
4. Produce a final response that the gateway will store as the run result.

Here is an example of a final response:

```markdown
I have sent an email to nane@acme.com with all 
the sales leads in the last month.

HEARTBEAT_OK: Sales email send successfully to nane@acme.com.
```

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
