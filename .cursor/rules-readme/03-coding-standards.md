# Coding Standards

> Human-readable mirror of [`../rules/03-coding-standards.mdc`](../rules/03-coding-standards.mdc). Cursor reads the `.mdc` file; edit both when updating this rule.


## Purpose

Define how code is written across this monorepo so Cursor generates **readable, consistent, production-quality** TypeScript — not clever one-offs.

This is the most frequently consulted rule. When in doubt, follow these standards over personal preference.

## Core Principles

### Scope

Applies to all TypeScript in `apps/` and `packages/` — frontend (React), backend (Node/Express), shared libraries, and configs.

Security-specific rules: see `08-security.mdc`. Architecture boundaries: see `06-architecture.mdc`.

### General Coding Principles

| Principle | Meaning |
| --------- | ------- |
| Readability over cleverness | Code is read more than written |
| Simplicity over premature optimization | Make it work, then measure |
| Composition over inheritance | Prefer hooks, components, and modules composed together |
| Explicit over implicit | Clear types, names, and control flow |
| Consistency over personal preference | Match existing patterns in the target directory |
| Single Responsibility | One reason to change per module or function |
| DRY (without over-abstraction) | Don't repeat logic; don't abstract a one-off |
| Immutability where practical | Prefer `const`, avoid mutating shared state |

## Rules

### Naming Conventions

| Kind | Convention | Example |
| ---- | ---------- | ------- |
| Files & folders | PascalCase for components; camelCase for utils; kebab-case for route/config files | `UserProfile.tsx`, `formatDate.ts`, `auth-routes.ts` |
| Variables | camelCase | `userCount`, `isLoading` |
| Constants | UPPER_SNAKE_CASE for true constants | `MAX_RETRY_COUNT`, `API_BASE_URL` |
| Functions | camelCase; verb-first | `fetchUser`, `validateInput` |
| React components | PascalCase | `UserProfile`, `CourseCard` |
| Hooks | `use` prefix + PascalCase remainder | `useAuth`, `useCourseProgress` |
| Types & interfaces | PascalCase | `UserProfileProps`, `AuthToken` |
| Enums | PascalCase name; PascalCase or UPPER members per existing file style | `OrderStatus.Pending` |
| Generics | Single uppercase letter or descriptive PascalCase | `T`, `TResponse` |
| Environment variables | UPPER_SNAKE_CASE | `DATABASE_URL`, `JWT_SECRET` |

Boolean prefixes: `is`, `has`, `can`, `should` (e.g. `isValid`, `hasAccess`).

### Project Structure

- **Feature-first organization** — group by domain/feature, not by file type alone.
- **Shared logic in `packages/`** — don't duplicate across `apps/`.
- **Avoid circular dependencies** — especially across package boundaries.
- **Public API through `index.ts`** — export only what consumers need.
- **Keep internal modules private** — no deep imports into another package's internals.
- Colocate components, hooks, types, and tests within a feature folder.

```
packages/auth/
  src/
    index.ts          — public exports
    services/
    types/
    utils/
```

### TypeScript

- TypeScript only — no JavaScript source files.
- Never use `any`. Use `unknown` and narrow with type guards.
- Respect strict mode. Fix type errors; do not suppress with `@ts-ignore`.
- Use explicit null checks when logic depends on presence.

**`interface` vs `type`** (no single winner — choose by use case):

- **`interface`** — object contracts, public APIs, shapes intended to be extended or merged.
- **`type`** — unions, intersections, mapped types, utility compositions, function signatures, tuples, aliases.

A formal team preference may be recorded in an ADR later. Until then, follow existing patterns in the target directory.

Additional practices:

- Use **discriminated unions** for variant state (e.g. `{ status: 'loading' } | { status: 'success'; data: T }`).
- Use **exhaustive `switch`** with `never` in the default arm to catch missing cases.
- Prefer **`satisfies`** when validating object shape while preserving literal types.
- Avoid **`as` type assertions** — narrow with guards or refactor instead.
- Use **utility types** (`Partial`, `Pick`, `Omit`, `Record`) before inventing custom equivalents.
- Apply **generic constraints** (`extends`) when generics need bounds — don't over-genericize.

### React

- Functional components only — no class components.
- Define props as a separate `type` or `interface`; assign to the component.
- Avoid anonymous default exports — name components for debugging and stack traces.
- Keep components focused — extract when JSX or logic grows hard to scan (~150 lines is a soft ceiling).
- Extract reusable logic into custom hooks prefixed with `use`.
- **Memoization** (`useMemo`, `useCallback`, `memo`) only when profiling shows a benefit — not by default.
- **State management** — local state first; lift only when needed; global state via Zustand when introduced (see architecture docs).
- **Accessibility** — semantic HTML, labels, keyboard navigation, ARIA only when semantics aren't enough.
- **Lists** — stable, unique `key` props; never use array index as key when order or identity changes.
- **Error boundaries** — wrap feature sections that can fail independently; don't catch errors silently in UI.
- Feature folder example: `features/authentication/components/LoginForm/LoginForm.tsx`.

### Backend (Node / Express)

Layer responsibilities:

| Layer | Role |
| ----- | ---- |
| Controllers / routes | HTTP parsing, status codes, call services — thin |
| Services | Business logic, orchestration |
| Repositories | Data access via Prisma — no HTTP concerns |
| Validation | Zod schemas at boundaries — validate before business logic |
| DTOs | Typed request/response shapes — separate from Prisma models when needed |

Additional rules:

- Use async/await — handle errors in middleware or service boundaries.
- **Logging** — structured, contextual; never log secrets or PII.
- **Transactions** — wrap multi-step DB writes in Prisma transactions.
- **Configuration** — read from environment via validated config module; no scattered `process.env` access.
- **Dependency injection** — pass dependencies into services (constructor or factory params) for testability.

### Error Handling

- Never swallow errors — empty `catch` blocks are forbidden.
- Throw **domain-specific errors** with clear messages; map to HTTP status in controllers.
- Don't expose stack traces or internal details to clients in production.
- Include **contextual logging** (request ID, user ID, operation) when logging failures.
- Fail fast at validation boundaries — reject bad input before side effects.

### Performance

- Don't optimize prematurely — readability first, measure second.
- Profile before memoizing React components or caching aggressively.
- Avoid unnecessary object creation, array copying, or repeated computations inside performance-critical code after profiling confirms a bottleneck.
- Prefer **streaming** for large file or response payloads where appropriate.

### Security (Coding Practices)

Full rules: `08-security.mdc`. At minimum when writing code:

- Validate all external input with Zod at boundaries.
- Escape or sanitize outputs rendered as HTML.
- Never log secrets, tokens, or credentials.
- Use Prisma parameterized queries — no string-concatenated SQL.
- Apply **principle of least privilege** — minimal scopes for tokens, roles, and DB access.

### Documentation in Code

- **Public APIs in `packages/`** — JSDoc on exported functions and types when behavior isn't obvious.
- **Complex algorithms** — brief comment explaining approach and trade-offs.
- **TODO** — planned work with context; **FIXME** — known defect needing fix.
- Don't comment what the code already says — comment *why* and non-obvious constraints.

## Examples

### ✅ Good naming

```
UserProfile.tsx
useAuth.ts
MAX_PAGE_SIZE
fetchCourseById()
type CourseProgress = { ... }
interface AuthService { signIn(): Promise<Session> }
```

### ❌ Avoid naming

```
data.ts
tempHandler()
flag
IUserProfileProps
useData()
```

### ✅ Good TypeScript

```typescript
type LoadState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string };

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}
```

### ❌ Avoid TypeScript

```typescript
const user = response as User;           // unchecked assertion
catch (e) {}                             // swallowed error
const data: any = fetchSomething();      // any
```

### ✅ Good React

```typescript
type LoginFormProps = {
  onSuccess: (session: Session) => void;
};

const LoginForm = ({ onSuccess }: LoginFormProps) => {
  // ...
};
```

### ❌ Avoid React

```typescript
export default () => <div />;            // anonymous component
items.map((item, i) => <Row key={i} />); // index as key when list mutates
```

### ✅ Good backend layering

```
Route → validate(Zod) → Service → Repository (Prisma) → Response DTO
```

### ❌ Avoid backend layering

```
Route handler with 200 lines of SQL, business logic, and HTTP formatting mixed together
```

## AI Behavior

When generating or editing code:

- **Reuse existing code** before creating new abstractions.
- **Prefer consistency** with surrounding files over novelty.
- **Explain trade-offs** when multiple valid approaches exist.
- **Suggest refactoring** only when complexity clearly warrants it.
- **Don't introduce new patterns** (state libraries, folder layouts, error frameworks) unless asked or documented in an ADR.
- Match naming, structure, and `interface`/`type` usage in the target directory.

## Anti-patterns

- **Clever one-liners** that sacrifice readability.
- **Premature abstraction** — generic utilities used once.
- **God components / god services** — files that do everything.
- **Circular imports** across features or packages.
- **Deep relative imports** — `../../../../utils` instead of package exports.
- **`any` and `@ts-ignore`** — type safety bypass.
- **Index keys on dynamic lists** — broken UI state on reorder/filter.
- **Scattered `process.env`** — use validated config.
- **Logging and returning raw errors** to clients in production.

## Exceptions

- **`interface` vs `type`** — team-wide preference deferred to a future ADR; until then, match local conventions.
- **Express as backend framework** — planned but not finalized; don't hardcode Express-specific patterns beyond thin-controller guidance until the backend ADR is accepted (see `01-project-context.mdc`).
