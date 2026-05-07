# Zenve Registry

The official registry of agents and skills for the [zenve](https://github.com/zenve-ai) cli.

## Contents

- [**Agents**](./agents/README.md) — autonomous agents (pm, architect, frontend-dev, fullstack-dev, code-review) with `manifest.yaml` + SOUL/AGENTS/RUN/HEARTBEAT
- [**Skills**](./skills/README.md) — reusable skills (project scaffolders, architecture auditors, agent conventions) usable from any Claude Code project

## Installation

Install all agents into the current org:
```bash
zenve agents add
```

Install all skills:
```bash
zenve skills add
```

For installing a specific agent or skill, see the per-directory READMEs:
- [agents/README.md](./agents/README.md)
- [skills/README.md](./skills/README.md)

## Layout

```
zenve-registry/
├── agents/      # zenve agents (manifest + SOUL/AGENTS/RUN/HEARTBEAT)
└── skills/      # claude code skills (SKILL.md per skill)
```

## Contributing

PRs welcome. See the contributing notes in each subdirectory's README for the conventions an agent or skill must follow before it can be merged.
