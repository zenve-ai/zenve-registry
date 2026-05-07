# Run Instructions

## Memory

- **Long-term memory** — persisted facts and decisions: `{agent_dir}/memory/long_term.md`
- **Scratch memory** — ephemeral working notes (cleared between runs): `{agent_dir}/memory/scratch.md`

## On Start

1. Read `{agent_dir}/memory/long_term.md` — load any persisted context.
2. Read `{agent_dir}/memory/scratch.md` — load working notes from previous runs.

## Executing the Task

- Complete the task described by the user.
- Stay within the tool permissions you have been given.
- Use `{agent_dir}/memory/scratch.md` for notes you need within this run only.
- Use `{agent_dir}/memory/long_term.md` for facts worth keeping across runs.

## IMPORTANT: Before ending the session:

1. Update `{agent_dir}/memory/long_term.md` with any durable observations.
2. Clear `{agent_dir}/memory/scratch.md` or leave a brief summary for the next run.
3. Produce a final response that the gateway will store as the run result.

Here is an example of a final response:

```markdown
RUN_OK: PR #123 created for issue X
```

## Signalling Outcomes

End your response with one of these signal lines so the gateway can parse the outcome:

| Signal | Meaning |
|--------|---------|
| `RUN_OK: <note>` | Task completed successfully; brief note of what was done |
| `RUN_FAILED: <reason>` | Task failed; include a short reason |
| `RUN_NEEDS_INPUT: <question>` | Blocked; waiting for more information |

