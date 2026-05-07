# Zenve Agents

A curated collection of agents to use with the zenve cli.

## Installation

Install all agents into the current org (`.zenve/agents/`):
```bash
zenve agents add
```

Install a specific agent:
```bash
zenve add --agent architect
```

---

## Agents

| Agent | Mode | Description |
|---|---|---|
| [`pm`](#pm) | `artifact_pr` | Defines product requirements — user stories, acceptance criteria, PRDs |
| [`architect`](#architect) | `artifact_pr` | Plans features from PRDs into ordered, dev-ready implementation tasks |
| [`frontend-dev`](#frontend-dev) | `code_pr` | Builds and maintains frontend UI across React and other frontend stacks |
| [`fullstack-dev`](#fullstack-dev) | `code_pr` | Builds features across React and FastAPI — routes, components, models, migrations |
| [`code-review`](#code-review) | `review_pr` | Reviews open PRs against architecture skills and posts review comments via `gh` |

---

## `pm`

Product Manager agent. Turns ideas and feedback into clear product requirements the architect can plan against.

**Outputs:** user stories · acceptance criteria · PRDs · backlog priorities
**Skills:** `zenve-memory`
**Tools:** Read · Write · Bash · WebSearch · WebFetch

**Install:**
```bash
zenve add --agent pm
```

---

## `architect`

Architect agent. Reads a PRD and produces an ordered, dev-ready task list for the developer agents to execute.

**Outputs:** implementation plan · ordered task list (artifact PR)
**Skills:** `zenve-memory`
**Tools:** Read · Write · Bash

**Install:**
```bash
zenve add --agent architect
```

---

## `frontend-dev`

Frontend Developer agent. Builds and maintains React (and other frontend) features end-to-end — components, state management, design systems, tests.

**Skills:** `zenve-memory` · `react-setup` · `react-rules`
**Tools:** Read · Write · Bash

**Install:**
```bash
zenve add --agent frontend-dev
```

---

## `fullstack-dev`

Fullstack Developer agent. Builds features across React and FastAPI — routes, components, models, migrations, tests.

**Skills:** `zenve-memory` · `react-setup` · `react-rules` · `fastapi-setup` · `fastapi-monorepo-setup` · `fastapi-rules`
**Tools:** Read · Write · Bash

**Install:**
```bash
zenve add --agent fullstack-dev
```

---

## `code-review`

Code Reviewer agent. Reviews open pull requests against the project's architecture skills, posts review comments via `gh`, and signals `approve` or `changes-requested`.

**Tools:** Read · Bash · Grep

**Install:**
```bash
zenve add --agent code-review
```

---

## Agent Format

Each agent is a directory with the 5 standard Zenve files:

```
agents/
└── my-agent/
    ├── manifest.yaml   # name, slug, mode, model, skills, tools
    ├── SOUL.md         # identity, principles, voice
    ├── AGENTS.md       # collaboration rules and handoffs
    ├── RUN.md          # what the agent does on each run
    └── HEARTBEAT.md    # background / scheduled behavior
```

Modes:
- `artifact_pr` — produces documents/artifacts and opens a PR with them
- `code_pr` — writes code and opens a PR
- `review_pr` — reviews an existing PR and posts feedback

## Contributing

PRs welcome. Each agent should:
- Have a clear, single responsibility reflected in `manifest.yaml: description`
- Include all 5 standard files (`manifest.yaml`, `SOUL.md`, `AGENTS.md`, `RUN.md`, `HEARTBEAT.md`)
- Reference shared conventions via `skills:` (e.g. `zenve-memory`) instead of duplicating them
