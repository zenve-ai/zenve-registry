# Run Instructions

## Memory

- **Long-term memory** — persisted facts and decisions: `{agent_dir}/memory/long_term.md`
- **Scratch memory** — ephemeral working notes (cleared between runs): `{agent_dir}/memory/scratch.md`

## On Start

1. Load the appropriate rules skill for your stack based on `{project_stack}` (e.g. `react-rules` for React projects, `fastapi-rules` for FastAPI projects) for baseline architecture standards.
2. Read any project-specific architecture or design docs if present.
3. Read `{agent_dir}/memory/long_term.md` — load any persisted context.
4. Read `{agent_dir}/memory/scratch.md` — load working notes from previous runs.

## Executing the Task

- Read the issue title, description, and latest PM completion comment.
- If a PM comment is present, extract the PRD path and read it before planning.
- If no PM comment, treat the issue title and description as the source of truth.
- Inspect relevant source files before planning.
- Do not edit application source code.
- Write the plan to `docs/plans/{feature}.md` using the Feature Plan Template in `AGENTS.md`. Create the `docs/plans/` directory if it does not exist.
- Use `{agent_dir}/memory/scratch.md` for notes you need within this run only.
- Use `{agent_dir}/memory/long_term.md` for facts worth keeping across runs.
- Output a short signal line as your response.

## IMPORTANT: Before ending the session:

1. Update `{agent_dir}/memory/long_term.md` with any durable planning conventions or project facts.
2. Clear `{agent_dir}/memory/scratch.md` or leave a brief summary for the next run.
3. Produce a final response that the gateway will store as the run result.

Your response must be a short signal line referencing the plan file:

```markdown
RUN_OK: wrote plan to docs/plans/{feature}.md
```

## Signalling Outcomes

End your response with one of these signal lines so the gateway can parse the outcome:

| Signal | Meaning |
|--------|---------|
| `RUN_OK: <note>` | Planning completed successfully; brief note of what was created |
| `RUN_FAILED: <reason>` | Planning failed; include a short reason |
| `RUN_NEEDS_INPUT: <question>` | Blocked; waiting for more information |
