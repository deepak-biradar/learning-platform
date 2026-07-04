# Engineering Standards — Human-Readable

**Engineering Standards v1** — foundational handbook for this project. Cursor consumes `.mdc` files; humans read this folder. Both stay in sync.

| Folder | Format | Audience |
| ------ | ------ | -------- |
| [`.cursor/rules/`](../rules/) | `.mdc` | Cursor and AI assistants (executable) |
| `.cursor/rules-readme/` | `.md` | Humans (GitHub, editor preview) |

When updating a standard, edit **both** files with the **same numbered basename** (e.g. `01-project-context`).

Every standard follows this structure:

```
Purpose → Core Principles → Rules → Examples → AI Behavior → Anti-patterns → Exceptions (optional)
```

## Three Layers

| Layer | Standards | Lifespan |
| ----- | --------- | -------- |
| **Engineering principles** | Coding, architecture, testing, security | Timeless |
| **Project standards** | Git, documentation, dependencies | Project-specific |
| **Workflow** | Engineering workflow (09) | Humans + AI |

## Standard Index

| # | Standard | `.mdc` | `.md` | Status |
| - | -------- | ------ | ----- | ------ |
| 01 | Project context | [`01-project-context.mdc`](../rules/01-project-context.mdc) | [`01-project-context.md`](01-project-context.md) | ✅ |
| 02 | Git workflow | [`02-git-workflow.mdc`](../rules/02-git-workflow.mdc) | [`02-git-workflow.md`](02-git-workflow.md) | ✅ |
| 03 | Coding standards | [`03-coding-standards.mdc`](../rules/03-coding-standards.mdc) | [`03-coding-standards.md`](03-coding-standards.md) | ✅ |
| 04 | Documentation | [`04-documentation.mdc`](../rules/04-documentation.mdc) | `04-documentation.md` | Pending |
| 05 | Testing | [`05-testing.mdc`](../rules/05-testing.mdc) | [`05-testing.md`](05-testing.md) | ✅ |
| 06 | Architecture | [`06-architecture.mdc`](../rules/06-architecture.mdc) | [`06-architecture.md`](06-architecture.md) | ✅ |
| 07 | Dependencies | [`07-dependencies.mdc`](../rules/07-dependencies.mdc) | [`07-dependencies.md`](07-dependencies.md) | ✅ |
| 08 | Security | [`08-security.mdc`](../rules/08-security.mdc) | [`08-security.md`](08-security.md) | ✅ |
| 09 | Engineering workflow | [`09-engineering-workflow.mdc`](../rules/09-engineering-workflow.mdc) | [`09-engineering-workflow.md`](09-engineering-workflow.md) | ✅ |

## v1 Milestone

Tag **`engineering-standards-v1`** when the foundation PR merges. After v1, changes to standards are intentional and traceable — new files require a documented need.

## Related Docs

| Document | Question |
| -------- | -------- |
| [README.md](../../README.md) | What is this project? |
| [PROJECT_CONTEXT.md](../../PROJECT_CONTEXT.md) | Where are we? |
| [docs/planning/](../../docs/planning/) | What are we building? |
| [docs/04-adr/](../../docs/04-adr/) | Why did we decide this? |
| [CONTRIBUTING.md](../../CONTRIBUTING.md) | How do I contribute? |
| [docs/README.md](../../docs/README.md) | Documentation index |

[`CONTRIBUTING.md`](../../CONTRIBUTING.md) is the human onboarding summary — will expand to mirror workflow over time. Planning ownership: [`docs/planning/README.md`](../../docs/planning/README.md).
