# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) — durable documents that capture significant technical decisions, the context behind them, and their consequences.

## When to Write an ADR

Create an ADR when a decision:

- Affects multiple apps or packages
- Is difficult or costly to reverse
- Introduces a new dependency, service, or pattern
- Changes data storage, authentication, or API contracts
- Has trade-offs worth documenting for future contributors

Skip ADRs for routine implementation details, bug fixes, or decisions already covered by an existing ADR.

## Naming Convention

```
NNNN-short-title-in-kebab-case.md
```

Examples:

- `0001-use-monorepo-with-turborepo.md`
- `0002-adopt-jwt-authentication.md`

Use the next sequential number. Do not renumber existing ADRs.

## Status Lifecycle

| Status       | Meaning                                      |
| ------------ | -------------------------------------------- |
| `proposed`   | Under discussion, not yet adopted            |
| `accepted`   | Decision is active                           |
| `deprecated` | Superseded or no longer recommended          |
| `superseded` | Replaced by a newer ADR (link to successor)    |

## Process

1. Copy `template.md` to a new numbered file.
2. Fill in all sections — especially Context, Decision, and Consequences.
3. Open a PR with the ADR for review alongside (or before) the implementation.
4. Update status to `accepted` when the PR merges.

## Index

| ADR  | Title                        | Status     |
| ---- | ---------------------------- | ---------- |
| —    | _No ADRs yet_                | —          |

_Update this table when adding new ADRs._
