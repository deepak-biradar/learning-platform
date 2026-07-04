# Project Context

| | |
| --- | --- |
| **Purpose** | Current project state — where we are right now |
| **Audience** | Humans and AI assistants |
| **Updated** | 2026-06-27 |
| **Related** | [README.md](README.md) · [CONTRIBUTING.md](CONTRIBUTING.md) · [docs/planning/](docs/planning/) · [docs/README.md](docs/README.md) |

> If context is lost, this file answers: **where does the project stand today?**  
> For tasks and sprint work, see [`docs/planning/`](docs/planning/). Not a task list.

## Project

- **Name:** TBD (repository: `learning-platform`)
- **Goal:** Build a production-ready AI-powered learning platform
- **First learning path:** Build a Production-Ready SaaS using Modern TypeScript

## Vision

Guide developers through the full product lifecycle — design, build, test, deploy, and maintain production-grade software — by learning through building, not passive tutorials.

## Current Phase

**Foundation** — engineering standards, documentation, repository structure, and tooling before application code.

## Current Milestone

**Engineering Foundation** — completing standards, planning system, and monorepo bootstrap (Sprint 0).

## Current Sprint

**Sprint 0** — see [`docs/planning/sprint-0.md`](docs/planning/sprint-0.md) when published (planning session pending).

## Version

`0.0.0` — pre-release; no tagged release yet.

## Technology Decisions

| Area | Choice | Status |
| ---- | ------ | ------ |
| Runtime | Node.js 24 LTS | Adopted |
| Package manager | pnpm | Adopted |
| Structure | Monorepo | Adopted |
| Build orchestration | Turborepo | Planned |
| Backend | Express.js | Planned |
| Database | PostgreSQL + Prisma | Planned |
| Containers | Docker | Planned |
| CI/CD | GitHub Actions | Planned |
| Repository | Public on GitHub | Adopted |

Significant choices require an ADR in [`docs/04-adr/`](docs/04-adr/) before implementation.

## Engineering Principles

- Documentation-first
- Production-first
- AI-first
- Learn by building
- Free-first (where practical)
- Incremental-first

Full standards: [`.cursor/rules-readme/`](.cursor/rules-readme/) (Engineering Standards v1).

## Completed Milestones

- Environment setup
- Git initialized
- Repository created on GitHub
- Folder structure created
- Engineering Standards v1 (rules 01–09)

## Recent Decisions

- Adopted Git Flow (`main` / `develop`, PR-only merges)
- Established Engineering Standards v1
- Adopted pnpm and monorepo layout
- Sprint tasks live in `docs/planning/` — not in this file
- `PROJECT_CONTEXT.md` is state only — no task lists

## Known Risks

- Monorepo and tooling not yet bootstrapped — application code blocked until Sprint 0 completes
- Several technology choices (Express, Turborepo) planned but not ADR-finalized

## References

| Document | Answers |
| -------- | ------- |
| [README.md](README.md) | What is this project? |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How do I contribute? |
| [docs/planning/](docs/planning/) | What are we building? |
| [docs/04-adr/](docs/04-adr/) | Why did we decide this? |
| [docs/03-architecture/](docs/03-architecture/) | How is the system designed? |
| [09-engineering-workflow](.cursor/rules-readme/09-engineering-workflow.md) | How do we build it? |

Update this file after each completed sprint or major architectural decision. Suggest updates when state changes; do not treat planning docs or standards as substitutes for this file.
