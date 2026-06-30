# Planning

| | |
| --- | --- |
| **Purpose** | Identify and prioritize work — roadmap, backlog, sprints |
| **Audience** | Humans and AI assistants |
| **Updated** | 2026-06-27 |
| **Related** | [PROJECT_CONTEXT.md](../../PROJECT_CONTEXT.md) · [09-engineering-workflow](../../.cursor/rules-readme/09-engineering-workflow.md) · [CONTRIBUTING.md](../../CONTRIBUTING.md) |

> **Planning identifies work. Engineering Workflow executes work.**  
> Once a task is selected here, follow [09-engineering-workflow](../../.cursor/rules-readme/09-engineering-workflow.md).

## Workflow

```
Roadmap
    ↓
Backlog
    ↓
Sprint
    ↓
Deliverable
    ↓
Task  ──►  Feature Branch  ──►  Implementation  ──►  PR  ──►  Merge
```

| Document | Purpose | Status |
| -------- | ------- | ------ |
| `roadmap.md` | Long-term direction and milestones | Pending — Sprint 0 planning session |
| `backlog.md` | Prioritized work not yet in a sprint | Pending |
| `sprint-N.md` | Current sprint deliverables and tasks | Pending (`sprint-0.md` first) |

## Task IDs

Use consistent IDs in sprint docs and code comments:

```
S{sprint}-D{deliverable}-T{task}

Example: S0-D2-T1 — Sprint 0, Deliverable 2, Task 1
```

Reference task IDs in PR descriptions and TODOs: `// TODO(S0-D2-T5): Add pagination`

## What belongs here

- Sprint goals, deliverables, and tasks
- Backlog items and priorities
- Roadmap milestones

## What does not belong here

- Current project state → [`PROJECT_CONTEXT.md`](../../PROJECT_CONTEXT.md)
- Why we chose a technology → [`docs/04-adr/`](../04-adr/)
- How to implement code → [Engineering Standards](../../.cursor/rules-readme/)
- Git and PR process → [`CONTRIBUTING.md`](../../CONTRIBUTING.md)
