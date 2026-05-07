# Run Instructions

## On Start

1. Load the appropriate rules skill for your stack based on `{project_stack}` (e.g. `react-rules` for React projects, `fastapi-rules` for FastAPI projects) for baseline architecture standards.
2. Read any project-specific architecture or design docs if present.

## Executing the Task

- Read the issue title, description, and latest PM completion comment.
- If a PM comment is present, extract the PRD path and read it before planning.
- If no PM comment, treat the issue title and description as the source of truth.
- Inspect relevant source files before planning.
- Do not edit source code or write any files.
- Output the plan followed by a signal line.

Your response must follow this format:

```markdown
# Plan: {Feature Name}

...plan content...

RUN_OK: {short one-line description of what was planned}
```

## Signalling Outcomes

End your response with one of these signal lines so the gateway can parse the outcome:

| Signal | Meaning |
|--------|---------|
| `RUN_OK: <note>` | Planning completed successfully; brief note of what was created |
| `RUN_FAILED: <reason>` | Planning failed; include a short reason |
| `RUN_NEEDS_INPUT: <question>` | Blocked; waiting for more information |
