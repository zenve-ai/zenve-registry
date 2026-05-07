# Zenve Skills

A curated collection of skills to use with zenve cli.

## Installation

Install all skills:
```bash
npx skills add zenve-ai/zenve-skills
```

Install a specific skill:
```bash
npx skills add zenve-ai/zenve-skills --skill fastapi-setup
```

Install globally (available in all projects):
```bash
npx skills add zenve-ai/zenve-skills -g
```

---

## Skills

| Skill | Description |
|---|---|
| [`fastapi-setup`](#fastapi-setup) | Scaffold a production-ready FastAPI project |
| [`fastapi-monorepo-setup`](#fastapi-monorepo-setup) | Scaffold a production-ready FastAPI monorepo with uv workspaces |
| [`fastapi-architect`](#fastapi-architect) | Audit any FastAPI project against architecture rules |
| [`react-setup`](#react-setup) | Scaffold a production-ready React 19 + Vite SPA |
| [`react-architect`](#react-architect) | Audit any React SPA project against architecture rules |
| [`zenve-create-agent`](#zenve-create-agent) | Scaffold a new Zenve agent (manifest + SOUL + AGENTS + RUN + HEARTBEAT) |

---

## `fastapi-setup`

Scaffolds a complete FastAPI project with clean layered architecture, auth, Docker, and architecture rules baked in via `CLAUDE.md`.

**Includes:** SQLAlchemy 2.x · JWT auth · Pydantic v2 · services layer · agents dir · justfile · Dockerfile · CLAUDE.md

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill fastapi-setup
```

**Usage:**
```bash
claude "create a new FastAPI project in dir myapi"
claude "scaffold a fastapi app called myapi"
```

---

## `fastapi-architect`

Audits any FastAPI project against a clean layered architecture — routes, services, models, agents, db access patterns.

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill fastapi-architect
```

**Usage:**
```bash
claude "review my routes"
claude "audit this project"
claude "does this follow the fastapi rules"
```

---

## `react-setup`

Scaffolds a complete React SPA with auth, routing, and state management ready to go.

**Includes:** React 19 · Vite 7 · TypeScript · Tailwind CSS v4 · shadcn/ui (new-york) · Redux Toolkit + RTK Query · React Router v7 · PrivateRoute/PublicRoute · JWT auth with localStorage · Dockerfile + nginx · CLAUDE.md

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill react-setup
```

**Usage:**
```bash
claude "create a new React project called myapp"
claude "scaffold a react spa called dashboard"
claude "new react frontend"
```

---

## `fastapi-monorepo-setup`

Scaffolds a complete FastAPI monorepo using `uv` workspaces with 5 shared packages, JWT auth, SQLAlchemy, and Docker support.

**Includes:** uv workspaces · shared packages (config, db, models, utils, services) · JWT auth · SQLAlchemy 2.x · Pydantic v2 · Docker + docker-compose · CLAUDE.md

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill fastapi-monorepo-setup
```

**Usage:**
```bash
claude "create a new fastapi monorepo called myapp"
claude "scaffold a fastapi monorepo project"
claude "add a new app to a fastapi monorepo"
```

---

## `react-architect`

Audits any React SPA project against a clean component architecture — pages, store, components, lib, and routing patterns.

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill react-architect
```

**Usage:**
```bash
claude "review my components"
claude "audit this react project"
claude "does this follow react rules"
```

---

## `zenve-create-agent`

Scaffolds a new Zenve agent under `.zenve/agents/{slug}/` with the 5 required files: `manifest.yaml`, `SOUL.md`, `AGENTS.md`, `RUN.md`, `HEARTBEAT.md`.

**Includes:** generic SOUL/AGENTS skeletons · `manifest.yaml` template · RUN + HEARTBEAT baselines with `RUN_OK` / `HEARTBEAT_OK` signal conventions · `code_pr` and `artifact_pr` modes · default model `claude-sonnet-4-6`

**Install:**
```bash
npx skills add zenve-ai/zenve-skills --skill zenve-create-agent
```

**Usage:**
```bash
claude "create a new zenve agent called fastapi-dev"
claude "scaffold a zenve agent for product management"
claude "add a new agent in .zenve"
```

---

## Skill Format

Each skill is a directory with a `SKILL.md` file following the Claude Code skill format:

```
skills/
└── my-skill/
    ├── SKILL.md     # skill instructions (required)
    └── README.md    # documentation (optional)
```

## Contributing

PRs welcome. Each skill should:
- Have a clear, focused trigger phrase
- Include a `README.md` with usage examples
- Be self-contained in its own directory
