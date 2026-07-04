# Contributing

| | |
| --- | --- |
| **Purpose** | How to contribute — workflow, branches, commits, and PRs |
| **Audience** | Human contributors |
| **Updated** | 2026-06-27 |
| **Related** | [README.md](README.md) · [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) · [docs/planning/](docs/planning/) · [Engineering Standards](.cursor/rules-readme/) |

Thank you for contributing. This document covers **collaboration workflow only**. Engineering standards live in [`.cursor/rules-readme/`](.cursor/rules-readme/) (humans) and [`.cursor/rules/`](.cursor/rules/) (Cursor).

Full Git Flow details: [`.cursor/rules-readme/02-git-workflow.md`](.cursor/rules-readme/02-git-workflow.md)

## Repository Setup

1. Clone the repository.
2. Checkout your feature branch (or `develop` after the first PR is merged).
3. Read [`README.md`](README.md) and [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md).
4. Open the project in your preferred IDE. If using Cursor, engineering standards under `.cursor/rules/` are applied automatically.

> Tooling setup (Node.js, pnpm, Docker) will be documented here once the monorepo is initialized.

## Branching

| Branch    | Role                                                         |
| --------- | ------------------------------------------------------------ |
| `main`    | Production-ready. **Never commit or push directly.**         |
| `develop` | Integration branch. **Never commit or push directly.**       |

Both branches receive code **only through Pull Requests**.

Create short-lived branches from `develop` (once it has commits):

| Prefix      | Example                          |
| ----------- | -------------------------------- |
| `feature/`  | `feature/engineering-foundation` |
| `bugfix/`   | `bugfix/login-validation`        |
| `docs/`     | `docs/project-context`           |
| `refactor/` | `refactor/api-validation`        |
| `chore/`    | `chore/github-actions`           |
| `release/`  | `release/v0.1.0`                 |
| `hotfix/`   | `hotfix/production-login` (from `main`) |

No sprint numbers in branch names — track sprints in [`docs/planning/`](docs/planning/).

### Bootstrap

While `main` and `develop` are empty, all work lives on a feature branch (e.g. `feature/engineering-foundation`). The first PR merges that branch into `develop`.

## Commit Convention

Commits are made on **short-lived branches only** — never on `main` or `develop`.

```
<type>(<scope>): <Capitalized meaningful sentence>

[body — required for major changes]
```

```
feat(auth): Implement JWT-based authentication with refresh tokens
docs(project): Establish the initial engineering foundation and project documentation
build(monorepo): Initialize the Turborepo workspace with pnpm
```

## Pull Request Process

1. Pick a task from the active sprint in [`docs/planning/`](docs/planning/).
2. Branch from `develop` (or work on a bootstrap feature branch while `develop` is empty).
3. Make focused, reviewable changes — one logical unit of work per PR.
4. Open a PR targeting **`develop`** (or **`main`** for release/hotfix only).
5. Use **squash merge** into `develop`; **merge commit** for releases into `main`.
6. Delete the branch after merge.

## Engineering Standards

| Audience | Location |
| -------- | -------- |
| Humans   | [`.cursor/rules-readme/`](.cursor/rules-readme/) (`.md`) |
| Cursor   | [`.cursor/rules/`](.cursor/rules/) (`.mdc`) |

Edit both formats when updating a standard. Execution workflow: [`09-engineering-workflow.md`](.cursor/rules-readme/09-engineering-workflow.md).

Architectural decisions require an ADR — see [`docs/04-adr/README.md`](docs/04-adr/README.md).

## Documentation Guide

| Document | Purpose |
| -------- | ------- |
| [`README.md`](README.md) | Project overview and onboarding — *what is this?* |
| [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) | Current phase, milestone, and decisions — *where are we?* |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Collaboration workflow — *how do I contribute?* |
| [`docs/planning/`](docs/planning/) | Roadmap, backlog, sprint tasks — *what are we building?* |
| [`docs/04-adr/`](docs/04-adr/) | Architecture Decision Records — *why this way?* |
| [`docs/README.md`](docs/README.md) | Documentation index |
| [`.cursor/rules-readme/`](.cursor/rules-readme/) | Engineering standards — *how do we work?* |

Each document answers one question. Link between them; don't duplicate content.

## Questions

| Topic | Where to go |
| ----- | ----------- |
| Development work, sprint tasks, roadmap | [`docs/planning/`](docs/planning/) |
| Public bugs, security reports | GitHub Issues |
| PR feedback, code review | Pull Request discussion |
