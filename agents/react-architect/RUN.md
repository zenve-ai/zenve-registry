# Run Instructions

## On Start

1. Load the `react-architect` skill for baseline React architecture standards.
2. Read `react-dev/DESIGN.md` for project-specific visual constraints.

## Executing the Task

- Read the issue title, description, and latest PM completion comment.
- If a PM comment is present, extract the PRD path and read it before planning.
- If no PM comment, treat the issue title and description as the source of truth.
- Inspect relevant frontend files before planning.
- Do not edit source code or write any files.
- Output the plan followed by a signal line.

Your response must follow this format:

```markdown
# React Plan: {Feature Name}

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
