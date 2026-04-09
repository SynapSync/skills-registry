---
name: audit
description: >
  Multi-domain codebase audit: security, contracts, patterns, observability, and ops.
  Invoke as `/audit [domain?]` where domain is one of: security | contracts | patterns | observability | ops | all (default).
  Use whenever the user asks to audit or review a codebase for issues, wants a security review,
  asks what's wrong with the code, requests a pre-release or pre-merge quality check, or wants
  a comprehensive inspection across quality dimensions. Trigger proactively on "audit", "security review",
  "code review", "what issues does this codebase have", or "check before we ship".
license: Apache-2.0
metadata:
  author: synapsync
  version: "1.0"
  scope: project
  auto_invoke: false
allowed-tools: Read, Glob, Grep, Bash
---

## Overview

Run a structured, evidence-based audit of a codebase. Two phases always run in order: first detect context, then apply the requested domains.

**Usage:** `/audit [domain?]`

- No domain → run **all** domains (default)
- Available domains: `security` | `contracts` | `patterns` | `observability` | `ops` | `all`

## Safety

Do **not** run destructive operations (DB resets, production deploys, mass deletes, `git push --force`, etc.) unless the user explicitly asks in the same session. Use read-only inspection: read files, Glob, Grep, and read-only git commands (`git log -1`, `git grep`) — no history rewrites.

---

## Phase 1 — Context Detection

Always runs first. Read the project to understand what you're working with. Identify and state explicitly:

- **Language(s):** Go, Python, TypeScript, Rust, etc.
- **Runtime / framework:** gRPC, FastAPI, Next.js, Express, Django, etc.
- **Auth mechanism:** JWT, sessions, OAuth, API keys, none
- **Data layer:** SQL (which DB), NoSQL, ORM, raw queries, none
- **Infra:** Docker, Kubernetes, serverless, bare metal
- **API surface:** REST, gRPC, GraphQL, tRPC, WebSockets
- **Test strategy:** unit, integration, e2e, none / partial
- **CI:** GitHub Actions, GitLab CI, none, unknown

### Minimum reads before Phase 2

Read at least:

- Root directory layout (top-level dirs)
- **One** dependency manifest (`go.mod`, `package.json`, `pyproject.toml`, `Cargo.toml`)
- **One** main entrypoint or app bootstrap (`cmd/`, `main.go`, `src/index.ts`)
- **One** config example (env template, `config.toml`, `docker-compose.yml`, etc.)

### Large / monorepo / polyglot repos

If the repo is large or a monorepo:

1. State **each major component** (`gateway/`, `services/profile/`, `contracts/`) with its own mini-context if stacks differ.
2. **Prioritize:** entrypoints, public API routes, auth boundaries, contract sources (`proto`, OpenAPI), CI workflows, infrastructure that defines secrets and CORS/TLS.
3. **Deep-dive** additional files only when a risk is indicated or the domain requires it. Say what you sampled — don't claim you read the entire tree.

Output findings in a **Context** block (see Output Format). If the project is ambiguous, split by component.

Do **not** proceed to Phase 2 until Phase 1 is complete.

---

## Phase 2 — Audit Domains

Apply each requested domain using the context from Phase 1. **Adapt every check** to what actually exists — skip checks for components that are not present (e.g. don't require a GraphQL schema if there is no GraphQL).

### security

Core questions:

- Is authentication enforced where it should be? Are public routes **explicit**, not accidentally open?
- Are secrets (keys, passwords, tokens) absent from **code**, committed **config**, and **history**? Flag tracked `.env` with real values, hardcoded production keys, etc.
- Is user input validated at the **boundary** before business logic / persistence?
- Are **security headers** present on HTTP responses where applicable?
- Is session / token invalidation possible and enforced on logout / revoke (if applicable)?
- **Known CVEs:** only report as a finding if you **ran** a supported scanner in this session (`govulncheck`, `osv-scanner`, `npm audit`). Otherwise state: `NOTE: Dependency CVE posture not verified in this run (no scanner executed).` — this is an explicit gap, not a PASS.

Optional when relevant: CORS correctness (credentials + allowlist), admin/docs exposure in production, idempotency of sensitive operations.

### contracts

Core questions:

- Does the declared API surface (spec, schema, types, proto) **match** the actual implementation?
- Are breaking changes guarded, versioned, or documented?
- Is documentation (OpenAPI, GraphQL schema, exported types) **generated from a single source of truth** vs manually maintained (drift risk)?
- Do internal consumers depend on the **declared** contract, not leaked implementation details?

### patterns

Core questions:

- Is error handling explicit / typed where the language encourages it? Avoid leaking stack traces or internal messages to untrusted clients.
- Dependency direction: avoid disallowed cycles / layer violations for this codebase's stated architecture.
- Configuration: loaded and validated at startup, not scattered ad-hoc `getenv` calls in domain logic.
- Abstractions: justified by actual use, not speculative.
- Idioms: respect language / framework conventions.
- State: mutation boundaries are clear (who owns mutable state, concurrency).

### observability

Core questions:

- Are errors logged with enough context to diagnose without a debugger (request id, route, safe fields)?
- Does logging **avoid PII** at routine levels (emails, tokens, full bodies, secrets)? Flag high-risk patterns.
- Health checks: do they reflect **dependencies** (DB, Redis) or only process liveness?
- Metrics / tracing at boundaries (HTTP, gRPC, DB, external APIs) if the stack claims production readiness.

### ops

Core questions:

- Build / images: reproducible enough for the team (lockfiles, pinned bases)? No secrets baked into images without a documented rotation story?
- CI: does the pipeline run what it claims (lint, types, unit tests, contract checks)? Gaps between "we care about this" and "what actually runs" are findings.
- Dependencies: lockfiles + verification (`go mod verify`, lockfile install) where applicable.
- Local / onboarding: can a new engineer run the stack from **documented** steps (README, compose, `make`), or is it tribal knowledge?
- Migrations / deploys: documented rollback or "safe retry" story where relevant.

---

## Behavior Rules

1. **Evidence first.** Read or search the repo before claiming a finding. If you infer, label it `INFERRED (needs confirmation)` and state what file would confirm it.
2. **Findings only.** Report items that are **WRONG**, **INCOMPLETE**, or create **REAL RISK**. Do not summarize what looks good. Do not list compliments.
3. **Empty domain.** If a domain has no findings, output `[DOMAIN] PASS` and continue.
4. **Location.** Prefer `path/to/file:line`. If the issue is process-wide (missing CI job, no policy), use `Location: (process / CI / policy)` and name the artifact that should exist.
5. **Fix.** Must be **actionable**. Use a short code or config block when it fits. For architectural fixes, describe steps and interfaces; a patch is optional.
6. **Severity** (every finding):
   - **BLOCKER** — must fix before merge
   - **HIGH** — must fix before production
   - **MEDIUM** — fix this sprint
   - **LOW** — document and defer
7. End with a **verdict:** `AUDIT PASSED` or `AUDIT FAILED`, with counts.

**Verdict rule:** `AUDIT FAILED` if **any** BLOCKER exists. Optionally fail on HIGH if the user instructed stricter gates in the chat.

---

## Output Format

```markdown
## Context
Date: YYYY-MM-DD
Domains audited: ...

Language(s): ...
Runtime / framework: ...
Auth: ...
Data layer: ...
Infra: ...
API surface: ...
Tests: ...
CI: ...
Sampling notes (if large repo): ...

## Audit Report — [domains] — YYYY-MM-DD

### SECURITY
[BLOCKER|HIGH|MEDIUM|LOW] Short title
Location: path:line OR (process / CI / policy)
Impact: one concrete sentence (what breaks, what leaks, what drifts)
Fix: actionable steps; optional code block if helpful

### CONTRACTS
(same pattern)

### PATTERNS
(same pattern)

### OBSERVABILITY
(same pattern)

### OPS
(same pattern)

---
## Verdict: AUDIT PASSED | AUDIT FAILED
Blockers: N | High: M | Medium: K | Low: J
```

---

## Quick Domain Map

| Domain | Typical artifacts |
|--------|-------------------|
| security | auth middleware, routes, secrets, CORS, headers, dependency scan (if run) |
| contracts | proto, OpenAPI, GraphQL schema, generated clients, CI `*-check` jobs |
| patterns | layers, errors, config loading, module boundaries |
| observability | logging, metrics, health checks, tracing |
| ops | Dockerfile, compose, CI YAML, migration docs, lockfiles |
