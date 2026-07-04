# Security

Source of truth: [`.cursor/rules/08-security.mdc`](../rules/08-security.mdc)

## Purpose

Define the project's security philosophy, secure coding practices, AI behavior, and engineering standards to ensure security is considered **throughout the software development lifecycle**.

Security is a **shared responsibility** — not a phase, a ticket, or someone else's job. Every feature (authentication, payments, file uploads, admin panels, APIs) inherits the security posture of the code written today. **Secure-by-default design** is far cheaper than retrofitting after an incident.

## Core Principles

| Principle | Meaning |
| --------- | ------- |
| Security by design | Consider threats and controls when designing features — not after shipping |
| Least privilege | Grant only the access required for the task — users, services, tokens, DB roles |
| Defense in depth | Layer controls — validation, auth, rate limits, logging — so one failure doesn't expose everything |
| Fail securely | When uncertain, deny access; errors must not bypass controls |
| Never trust user input | Treat all external input as untrusted until validated |
| Secure defaults | Safe configuration out of the box — opt-in to loosen, not opt-out to tighten |
| Explicitly opt into risk | Secure by default at every layer — widening access or disabling controls requires deliberate choice |
| Assume breach | Design so a single compromise limits damage — segmentation, least privilege, auditing, token expiration |
| Privacy by default | Collect and expose only data that is necessary; minimize PII exposure |
| Minimize attack surface | Fewer endpoints, permissions, and exposed details means fewer targets |
| Balance with usability | Security must be practical — friction should be proportional to risk |

### Scope

Applies to all code in `apps/`, `packages/`, infrastructure config, and CI/CD. Coding patterns follow `03-coding-standards.mdc`. Dependency practices follow `07-dependencies.mdc`. Authorization placement follows `06-architecture.mdc`.

Keep this rule **technology-agnostic** where practical — specific auth providers, secret managers, and WAF tools are chosen via ADR when introduced.

Align with **OWASP Top 10** principles where applicable; adapt controls to frontend and backend equally.

## Threat Modeling

Before implementing security-sensitive features, consider:

- **What are we protecting?** — data, credentials, user privacy, system integrity.
- **Who are the potential attackers?** — anonymous users, authenticated users, insiders, automated bots.
- **What could go wrong?** — injection, broken access control, data leakage, abuse.
- **What is the impact?** — user harm, compliance exposure, service disruption.
- **How can the risk be reduced?** — validation, auth, rate limits, monitoring, least privilege.

Prefer **lightweight threat modeling during feature design** rather than after implementation. Document significant findings in the PR or ADR when the feature is high-risk.

## Rules

### Authentication

- **Never store plaintext passwords** — hash with an industry-standard algorithm (e.g. argon2, bcrypt, scrypt) and appropriate cost parameters.
- **Use adaptive hashing** — tune work factors as hardware improves; never roll custom password hashing.
- **Support MFA where appropriate** — especially for admin, billing, and sensitive account actions.
- **Secure password reset flows** — time-limited, single-use tokens; never reveal whether an email exists in the system.
- **Session and token security** — httpOnly, secure, and sameSite cookies where applicable; short-lived access tokens with refresh rotation when using JWTs.
- **Proper logout and session invalidation** — server-side session revocation; invalidate refresh tokens on logout and password change.
- **Protect authentication endpoints** — rate limiting, lockout or backoff after repeated failures, CAPTCHA when abuse is detected.

Do **not** prescribe a specific authentication provider — document the chosen approach in an ADR before implementation.

### Authorization

- **Authentication ≠ authorization** — knowing who the user is does not mean they may perform the action.
- **Validate permissions on every protected operation** — at the API and service layer, not only at the route or UI.
- **Default deny** — when access is uncertain, reject the request.
- **Prefer role- and permission-based authorization** — explicit grants over implicit "logged in = allowed".
- **Never rely solely on frontend authorization** — UI hiding is UX, not security; enforce on the server.

Authorization check flow:

```
Request → Authenticate identity → Authorize action → Execute business logic
```

### Input Validation

- **Validate all external input** — request bodies, query strings, path params, headers, cookies, and uploaded files.
- **Validate at system boundaries** — at the API edge before business logic or database access.
- **Reject invalid input early** — fail fast with a consistent error shape; no partial processing of bad data.
- **Sanitize only when appropriate** — validate structure and type first; sanitize HTML/output encoding at render boundaries.
- **Prefer allowlists over blocklists** — define what is permitted, not exhaustive lists of forbidden patterns.
- **Validate uploaded files** — type, size, extension, and content where practical; never trust client-provided MIME types alone.
- **Use schema validation** — structured validators (e.g. Zod) at boundaries for type-safe parsing and rejection.

### Secrets Management

- **Never commit secrets** — API keys, passwords, private keys, connection strings stay out of version control.
- **Never hardcode credentials** — not in source, config files tracked in git, or client bundles.
- **Use environment variables or secret management solutions** — validated at startup; provide `.env.example`, never `.env`.
- **Rotate secrets when compromised** — and periodically for high-value credentials.
- **Keep development and production secrets separate** — never use production credentials locally.

See `07-dependencies.mdc` for supply-chain practices; this section covers **application secrets**.

### API Security

- **Always use HTTPS in production** — redirect HTTP to HTTPS; enforce TLS for all external traffic.
- **Validate request payloads** — schema validation before controllers or services process data.
- **Return consistent error responses** — same error shape; no stack traces or internal paths to clients.
- **Avoid exposing internal implementation details** — database errors, file paths, and framework versions stay internal.
- **Rate limiting proportional to risk** — not every endpoint needs the same limits; apply stricter controls where abuse has higher impact:

| Endpoint type | Examples |
| ------------- | -------- |
| Authentication | Login, password reset, OTP verification |
| Public APIs | Unauthenticated or high-volume read/write |
| Expensive operations | Report generation, bulk exports |
| File uploads | Upload initiation and completion |

- **Protect against common API abuse** — brute force, credential stuffing, enumeration, and excessive payload sizes.
- **Protect against SSRF** — when fetching user-supplied URLs (e.g. import from URL), validate targets, block internal/private IP ranges, and prefer allowlists over open fetching.

### Database Security

- **Use parameterized queries or ORM protections** — Prisma and prepared statements prevent injection by default.
- **Never concatenate SQL strings** — no string-built queries with user input.
- **Apply least privilege for database users** — app credentials should not have admin or DDL rights in production.
- **Protect backups** — encrypt at rest, restrict access, test restore procedures.
- **Encrypt sensitive data when appropriate** — at rest and in transit; document key management in an ADR.

### Logging and Monitoring

- **Log security-relevant events** — login success/failure, permission denials, password changes, admin actions.
- **Prefer structured logging over free-form strings** — machine-parseable fields enable search, alerting, and audit trails.
- **Never log passwords, tokens, API keys, or sensitive PII** — mask or omit; treat logs as a data-exposure surface.
- **Include enough context for investigation** — timestamp, request ID, user ID (when safe), action, outcome.
- **Protect log integrity** — restrict write access; consider centralized logging for production.

Safe vs unsafe logging:

| Log | Safe | Unsafe |
| --- | ---- | ------ |
| Auth failure | `{ "event": "login_failure", "userId": "...", "ip": "..." }` | `{ password: '...' }` |
| Auth success | `{ "event": "login_success", "userId": "..." }` | `User logged in` (unstructured) |
| API error | `{ "event": "api_error", "requestId": "...", "errorCode": "..." }` | `{ stack, query, token }` |

### Dependency Security

- **Keep dependencies updated** — stale packages accumulate known CVEs.
- **Monitor security advisories** — Dependabot, audit tooling, CI checks when configured.
- **Remove abandoned packages** — unpatched dependencies are ongoing risk.
- **Review third-party packages before adoption** — follow `07-dependencies.mdc` evaluation checklist.

Full dependency philosophy: `07-dependencies.mdc`.

### Client-Side Security

- **Never trust client validation alone** — duplicate all critical checks on the server.
- **Escape or sanitize user-generated content** — prevent XSS when rendering HTML; use framework-safe patterns by default.
- **Avoid exposing sensitive configuration** — no API secrets, admin keys, or internal URLs in client bundles.
- **Protect against common browser-based attacks** — XSS, CSRF (tokens or sameSite cookies), clickjacking (frame options).
- **Secure local storage usage** — prefer httpOnly cookies for session tokens; localStorage is accessible to XSS.
- **Consider Content Security Policy (CSP)** — restrict script and resource sources in production when feasible.

### File Upload Security

- **Validate file type** — allowlist extensions and MIME types; inspect magic bytes where practical.
- **Never trust the file extension alone** — `virus.exe` renamed to `virus.jpg` is still dangerous; verify content, not just the suffix.
- **Validate file size** — enforce limits at upload and server configuration level.
- **Generate safe filenames** — never use user-supplied names directly; use UUIDs or sanitized identifiers.
- **Scan files where appropriate** — antivirus or content inspection for user-uploaded assets when risk warrants.
- **Store uploads securely** — outside web root; serve via controlled endpoints with auth checks.
- **Never execute uploaded content** — no running user files as scripts or server-side includes.

### Error Handling

- **Do not expose stack traces in production** — generic message to client; full detail in internal logs only.
- **Return user-friendly error messages** — actionable for users, not diagnostic for attackers.
- **Log detailed information internally** — enough for debugging without logging secrets.
- **Avoid leaking sensitive implementation details** — ORM errors, SQL fragments, and path disclosures stay internal.

### Security Reviews

- **Consider security during code reviews** — auth, validation, data access, and secrets in every PR touching sensitive paths.
- **Evaluate new features for potential risks** — STRIDE-lite thinking: spoofing, tampering, escalation, data exposure.
- **Review permissions, data access, and input validation** — the three most common miss areas.
- **Encourage secure coding discussions** — ask "what could go wrong?" before merging.

Security review checklist (sensitive changes):

```
[ ] All external input validated at boundary
[ ] Authorization enforced at service layer
[ ] No secrets in code, logs, or client bundle
[ ] Errors fail securely — no information leakage
[ ] Dependencies reviewed if new packages added
```

## Examples

### ✅ Secure password handling

```
Hash password with argon2/bcrypt before storage
Compare using constant-time verify function
Never log or return password in any response
```

### ❌ Insecure password handling

```
Store password in plaintext or reversible encryption
Log failed login with submitted password
Return "wrong password" vs "user not found" differently (user enumeration)
```

### ✅ Good secret management

```
DATABASE_URL from environment, validated at startup
.env.example documents required keys without values
Secrets injected via CI/CD or secret manager in production
```

### ❌ Poor secret management

```
const API_KEY = "sk-live-abc123" in source code
Commit .env with real credentials
Share production DB URL in Slack or PR comments
```

### ✅ Proper authorization check

```
// Service layer — every protected operation
async function deleteCourse(userId: string, courseId: string) {
  const course = await courseRepository.findById(courseId);
  if (!course) throw new NotFoundError();
  if (!canManageCourse(userId, course)) throw new ForbiddenError();
  await courseRepository.delete(courseId);
}
```

### ❌ Missing authorization

```
// UI hides delete button — but API allows any authenticated user
router.delete('/courses/:id', authenticate, deleteCourseHandler);
// Handler deletes without checking ownership or role
```

### ✅ Safe API error response (production)

```
{ "error": { "code": "FORBIDDEN", "message": "You do not have access to this resource" } }
```

### ❌ Unsafe API error response

```
{ "error": "Prisma error: column users.ssn does not exist at /app/src/repos/user.ts:42" }
```

### ✅ Input validation at boundary

```
const body = createCourseSchema.parse(req.body);  // reject before service
await courseService.create(userId, body);
```

### ❌ Trusting input

```
await courseService.create(req.body);  // unvalidated, any shape accepted
```

## AI Behavior

When generating or modifying code:

- **Prefer secure defaults** — deny by default, validate at boundaries, least privilege scopes.
- **Explain security trade-offs** — when a choice affects exposure, performance, or UX.
- **Warn when security implications exist** — flag missing auth, validation, or secret handling before implementing.
- **Avoid insecure examples** unless explicitly requested for educational purposes — and label them clearly as anti-patterns.
- **Reuse existing security patterns** within the project — middleware, validators, error mappers, auth guards.
- **Recommend validation and authorization** wherever user input or protected resources are involved.
- **When multiple approaches are equally valid, explain trade-offs instead of choosing silently.**
- **If generated code weakens security, explicitly explain why** — e.g. when disabling CSRF, bypassing auth, or loosening validation, state the risk and safer alternatives.

**Never:**

- Generate code with hardcoded secrets, API keys, or default admin passwords.
- Skip server-side validation because "the frontend already checks".
- Expose stack traces, SQL errors, or internal paths in API responses.
- Log tokens, passwords, or full credit card numbers.
- Disable security checks (CORS `*`, auth bypass, `skipValidation`) for convenience without explicit user request and documented risk.
- Recommend deprecated crypto (MD5/SHA1 for passwords, ECB mode, custom ciphers).

## Anti-patterns

- **Hardcoded secrets** — credentials in source, config committed to git, keys in client bundles.
- **Plaintext passwords** — storing or transmitting passwords without hashing.
- **Trusting frontend validation** — "the React form validates email format" is not security.
- **String-built SQL queries** — concatenating user input into SQL strings.
- **Logging sensitive data** — passwords, tokens, session IDs, full PII in application logs.
- **Overly permissive authorization** — `if (isLoggedIn) allow everything`.
- **Disabling security checks for convenience** — `auth: false` in dev that ships to prod, open CORS everywhere.
- **Ignoring security warnings** — audit tool CVEs, linter security rules, or review comments dismissed without action.
- **Security through obscurity** — hiding admin URLs instead of enforcing auth.
- **User enumeration** — different error messages revealing whether an account exists.
- **Insecure direct object references** — accessing `/users/123/data` without verifying the requester owns resource 123.

## Exceptions

- **Auth provider and secret manager selection** — deferred until implementation; document in ADR before building auth flows.
- **WAF, SIEM, and advanced threat detection** — out of scope for Foundation phase; revisit for production hardening.
- **Penetration testing and formal security audits** — planned before production launch; not required for early development.
- **CSP and HSTS strict policies** — may be phased in; document rollout in ADR or deployment docs when introduced.
- **Educational insecure examples** — permitted in docs and learning content when explicitly labeled and never used in production code paths.
- **Development-only shortcuts** — fake auth, disabled CSRF, or bypassed validation may exist locally for developer velocity — never merged into production branches.
