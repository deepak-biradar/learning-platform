# Architecture

Source of truth: [`.cursor/rules/06-architecture.mdc`](../rules/06-architecture.mdc)

## Purpose

Define the project's architectural principles, layering, module boundaries, and AI behavior to ensure a **scalable, maintainable, and production-ready** codebase.

Architecture is how we **organize responsibility** — not which framework we pick. Stack choices may change; these boundaries stay constant.

### System Overview

End-to-end request flow (reference diagram):

```
Browser
   │
React App
   │
API Client (Services)
   │
REST API
   │
Controller
   │
Service
   │
Repository
   │
Database
```

## Core Principles

| Principle | Meaning |
| --------- | ------- |
| Separation of concerns | Each layer and module has one clear job |
| High cohesion | Related logic lives together — by feature, not by accident |
| Low coupling | Modules depend on abstractions and public APIs, not internals |
| SOLID where appropriate | Apply when it reduces change cost — not as ceremony |
| Composition over inheritance | Build behavior by combining modules, hooks, and services |
| Feature-first organization | Group by domain capability, not file type alone |
| Modular architecture | Independently understandable, testable, and replaceable units |
| Explicit dependencies | Imports and package boundaries reflect real relationships |
| Scalability and maintainability over shortcuts | Favor clarity today over hacks that compound tomorrow |

### Scope

Applies to all code in `apps/` and `packages/`. Coding style follows `03-coding-standards.mdc`. Security boundaries follow `08-security.mdc`.

Keep this rule **framework-agnostic** where practical — Express, Prisma, React, and TanStack Query are planned choices, not architectural law.

### Monorepo Layout

```
apps/              — deployable applications (web, api)
packages/          — shared libraries consumed by apps
docs/03-architecture/ — human-readable architecture docs for complex systems
docs/04-adr/       — durable records of significant technical decisions
```

- **`apps/`** — deployable boundaries. No duplicated business logic across apps.
- **`packages/`** — reusable libraries, configs, and schemas consumed by apps.
- Do **not** import from `apps/` into `packages/`.

## Rules

### Layered Architecture

Dependencies flow **inward and downward** — UI depends on features; features depend on services; services depend on repositories. Never the reverse.

#### Frontend Layers

| Layer | Responsibility | Depends on |
| ----- | -------------- | ---------- |
| **UI** | Presentational components — layout, styling, rendering props | Features, Shared |
| **Features** | Domain screens and workflows — compose UI, hooks, and services for one capability | Hooks, Services, Shared |
| **Hooks** | Reusable stateful logic — data fetching orchestration, form state, side effects | Services, Shared |
| **Services** | API clients, adapters, and client-side domain orchestration | Shared |
| **Shared** | Cross-cutting UI primitives, types, utils, config — no feature-specific logic | Shared only |

Frontend flow:

```
UI → Features → Hooks → Services → Shared
```

- **UI** — dumb where possible; no direct API calls or business rules.
- **Features** — the home for domain folders (`features/authentication/`, `features/courses/`).
- **Hooks** — extract when logic is reused or a component grows hard to scan.
- **Services** — HTTP calls, response mapping, retry policy — not JSX.
- **Shared** — design tokens, base components, generic helpers — never feature imports.

#### Backend Layers

| Layer | Responsibility | Depends on |
| ----- | -------------- | ---------- |
| **Routes** | HTTP method, path, middleware chain — wire request to controller | Controllers, Shared |
| **Controllers** | Parse request, invoke service, map result to HTTP status and body — thin | Services, Shared |
| **Services** | Business logic, orchestration, transactions, authorization rules | Repositories, Shared |
| **Repositories** | Data access — queries, persistence, mapping to domain types | Database, Shared |
| **Database** | Schema, migrations, constraints — accessed only through repositories | — |
| **Shared** | Config, errors, logging, validation schemas, cross-cutting utilities | Shared only |

Backend flow:

```
Routes → Controllers → Services → Repositories → Database
```

- **Routes** — no business logic; register handlers and middleware only.
- **Controllers** — validate input at boundary, call one service method, return DTO — no SQL.
- **Services** — the only place for business rules and multi-step workflows.
- **Repositories** — encapsulate ORM/query details; services never import the ORM directly.
- **Database** — schema changes via migrations only; never ad-hoc production edits.

### Module Boundaries

- **Features should be self-contained** — components, hooks, types, and tests colocated under the feature folder.
- **Shared code belongs in shared packages** — if two apps or two features need it, promote to `packages/`.
- **Avoid circular dependencies** — especially across package boundaries; refactor or extract a lower-level module.
- **Prefer dependency direction toward lower layers** — higher layers may import lower; never the reverse.
- **Public APIs through `index.ts`** — export only what consumers need; keep internals private. Internal modules should never be imported directly by other packages unless explicitly exposed through the public API.
- **No deep imports** — consumers import from package root (`@repo/auth`), not `@repo/auth/src/internal/foo`.

### Feature Extraction

When a feature becomes difficult to understand — many files, unclear ownership, or unrelated concerns mixed together — **split it into smaller sub-features** rather than introducing generic shared folders.

```
features/authentication/          — grows large over time
  login/
  registration/
  password-reset/
  shared/                         — auth-only helpers, not app-wide shared
```

Prefer domain splits (`login/`, `registration/`) over dumping growth into `components/`, `utils/`, or `misc/` at the app root.

Feature folder example:

```
features/authentication/
  components/
  hooks/
  services/
  types/
  index.ts          — public feature exports (if consumed outside the feature)
```

### Shared Packages

Promote logic to shared packages when it is **stable, reused, and domain-agnostic** (or cross-domain by design).

| Package | Belongs here | Does not belong here |
| ------- | ------------ | -------------------- |
| **`shared/ui`** | Buttons, inputs, layout primitives, design-system components | Feature-specific screens or business copy |
| **`shared/types`** | DTOs, API contracts, enums used across apps | Feature-internal types never exported |
| **`shared/utils`** | Pure helpers — formatting, parsing, id generation | Stateful logic, API clients, React hooks |
| **`shared/config`** | Validated environment config, constants, feature flags | Secrets in source; scattered `process.env` reads |

When unsure: **start in the feature**; extract to `packages/` when a second consumer appears or an ADR defines a shared contract.

### API Design Principles

- **REST-first** — resource-oriented URLs, standard HTTP verbs, predictable status codes.
- **Consistent naming** — plural nouns for collections (`/courses`, `/users`); kebab-case paths.
- **Consistent error responses** — one error shape across endpoints (code, message, details); map domain errors in controllers.
- **Validation at boundaries** — validate and parse all external input before business logic runs.
- **Versioning strategy** — prefix breaking changes (`/v1/`, `/v2/`) or negotiate via headers; document in `docs/06-api/` when established.
- **Avoid breaking contracts without a plan** — never ship breaking API changes without versioning or an explicit migration strategy.

Request lifecycle:

```
HTTP Request → Route → Validate → Controller → Service → Repository → Response DTO
```

### Database Principles

- **Normalize by default** — reduce duplication and update anomalies; denormalize only with measured need (ADR when significant).
- **Migrations only** — every schema change is a versioned migration; reversible where practical.
- **Never modify production schema manually** — no hotfix DDL in production consoles.
- **Repository abstraction where appropriate** — services depend on repository interfaces or modules, not raw query strings scattered across the codebase.
- **Transactions for multi-step writes** — wrap related inserts/updates in a single transaction at the service layer.
- **Soft deletes by requirement, not default** — prefer hard deletes unless business rules (audit, recovery, compliance) justify soft deletion.

Data access flow:

```
Service → Repository → ORM / query layer → Database
```

### Scalability

- **Design for change** — isolate what varies (payment provider, auth strategy) behind interfaces or modules.
- **Avoid over-engineering** — don't build plugin systems, event buses, or microservices before a second use case exists.
- **Optimize after measurement** — profile before caching, sharding, or premature async complexity.
- **Keep modules independently testable** — inject dependencies; services testable without HTTP or database when boundaries are respected.

### Decision Records (ADR)

Create an ADR in `docs/04-adr/` when a decision:

- Affects multiple apps or packages
- Is costly to reverse
- Introduces a new service, dependency, or architectural pattern
- Changes auth, data storage, or API contracts

Use `docs/04-adr/template.md`. Significant architectural decisions must be **accepted before implementation**. Do not renumber existing ADRs.

## Examples

### ✅ Good frontend layering

```
LoginPage (Feature)
  → useLogin (Hook)
    → authService.login (Service)
      → POST /auth/login (API)
LoginForm (UI) — receives onSubmit and errors as props only
```

### ❌ Avoid frontend layering

```
LoginForm fetches /auth/login directly, validates email format,
stores token in localStorage, and navigates on success — all in one component
```

### ✅ Good backend layering

```
POST /auth/login
  → authRoutes
    → authController.login (validate + map response)
      → authService.login (business rules)
        → userRepository.findByEmail (data access)
```

### ❌ Avoid backend layering

```
Route handler with 200 lines of SQL, password hashing, JWT creation,
and HTTP formatting mixed together
```

### ✅ Good module boundary

```
packages/shared/types exports CourseSummary
apps/web/features/courses imports from @repo/types
apps/api/services/courseService imports the same type
```

### ❌ Avoid module boundary

```
apps/web/features/courses imports from apps/api/src/repositories/userRepository
packages/auth imports from apps/web/features/dashboard
```

### ✅ Good shared package usage

```
shared/ui: Button, TextInput, Modal
shared/utils: formatCurrency, slugify
shared/config: getEnvConfig() with Zod validation
```

### ❌ Avoid shared package usage

```
shared/utils: 800-line file with auth, dates, API clients, and React hooks
shared/ui: CourseCheckoutWizard (belongs in features/courses)
```

### ✅ Good API error shape

```
{ "error": { "code": "VALIDATION_FAILED", "message": "Invalid email", "details": [...] } }
```

### ❌ Avoid API error shape

```
{ "msg": "bad" } on one endpoint and { "errorMessage": "..." } on another
```

## AI Behavior

When generating or modifying code:

- **Respect existing architecture** — match layer placement and folder layout in the target app or package.
- **Do not introduce new patterns without justification** — new state libraries, folder schemes, or error frameworks require an ADR or explicit user request.
- **Reuse existing modules before creating new ones** — search for similar services, hooks, or components first.
- **When multiple architectural approaches are equally valid, explain the trade-offs instead of choosing one silently** — present options (feature vs shared package, service vs hook) and recommend with reasoning.
- Place **business logic in services** (backend) or **hooks/services** (frontend) — never in UI or route handlers.
- Prefer **extending an existing feature folder** over creating parallel structures.
- When a decision affects multiple packages, **suggest an ADR** before implementing.

**Never:**

- Create a new `packages/` module for one-off logic used in a single feature.
- Import across layer boundaries in the wrong direction (repository calling service, UI calling repository).
- Duplicate business logic in both `apps/web` and `apps/api` — extract to `packages/` when shared.
- Hardcode framework-specific architecture as immutable when the stack is still planned (see `01-project-context.mdc`).

## Anti-patterns

- **God components** — screens that fetch, validate, mutate state, and render entire workflows alone.
- **God services** — single files owning unrelated domains (auth + billing + notifications).
- **Circular dependencies** — `featureA` imports `featureB` imports `featureA`; or packages importing apps.
- **Feature envy** — one service reaching into another's internals (e.g. `UserService` calling `OrderService` private methods) instead of a public API or orchestration layer.
- **Business logic inside UI** — validation rules, pricing logic, or authorization checks in JSX components.
- **Fat controllers** — HTTP handlers containing SQL, business rules, and email sending.
- **Massive utility files** — catch-all `utils.ts` hundreds of lines deep; split by domain or promote to packages.
- **Copy-paste architecture** — same folder structure recreated per feature with duplicated patterns instead of shared conventions.
- **Leaky abstractions** — exposing ORM models or raw DB rows directly as API responses when DTOs are needed.
- **Deep relative imports** — `../../../../` chains across features instead of package exports.
- **Premature microservices** — splitting deployables before monolith boundaries are understood.

## Exceptions

- **Planned stack** (Express, Prisma, React, TanStack Query) — guidance applies to layering; specific framework patterns follow project config and ADRs once accepted.
- **Small Foundation-phase changes** — before apps exist, prefer documenting structure in rules and ADRs over building unused abstractions.
- **Performance-driven denormalization** — acceptable with measurement and ADR when query cost justifies duplicated data.
- **GraphQL or tRPC** — out of scope unless adopted via ADR; REST-first remains the default until then.
- **Repository interfaces** — whether every service needs an explicit repository interface is deferred to a persistence ADR when data access is implemented.
