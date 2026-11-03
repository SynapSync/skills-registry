# Regression and Risk — Choosing What to Re-test

Testers never have time to re-test everything. The goal is to spend effort where breakage is most likely and most costly, and to be explicit about what is left untested.

## Risk = impact × likelihood

Rate each candidate area **High / Medium / Low** on both axes and explain the rating in one line.

**Impact** — how bad is failure here?

- Money, payments, billing, data loss or corruption, security and permissions, legal or compliance, the main user journey → High.
- Secondary features used daily → Medium.
- Cosmetic, rarely used, easy workaround → Low.

**Likelihood** — how probable is breakage?

- Code changed directly, large or complex diff, shared code with many callers, changed contracts (APIs, schemas, data models, config, dependencies), removed behavior → higher.
- Area with open debt, a history of repeated fixes, or no automated evidence → higher.
- Untouched code with stable history → lower.

Order the test plan: High×High first, then High impact, then High likelihood, then the rest. Start any session with a short **smoke check** of the main journey.

## RCRCRC — what belongs in a regression set

Heuristic by Karen N. Johnson, mapped to project sources:

| Letter | Meaning | How to find it here |
|--------|---------|---------------------|
| **R**ecent | New or changed in this range | `git log`/`git diff` of the range |
| **C**ore | Essential functions that must always work | Scope objective, `must` requirements, main flows |
| **R**isky | Complex or historically fragile | Open debt, large diffs, shared modules, ADR trade-offs |
| **C**onfiguration-sensitive | Depends on environment, flags, settings, versions | Changed config/env files, feature flags, dependency bumps |
| **R**epaired | Recently fixed | `fix:` commits, resolved debt, tasks that fixed defects |
| **C**hronic | Breaks repeatedly | Files touched across many sprints or many `fix` commits (`git log --oneline -- <path>`) |

## Finding indirect impact

For sprint/scope questions in T1/T2, resolve the boundaries through `sources.md` → *Kyro sprint and scope ranges (6.5.0+)* before selecting the **Recent** set. Preserve per-sprint ranges when they cannot be composed exactly. A dirty close, missing/unreachable endpoint, skipped sprint, or active sprint leaves coverage uncertain: include the affected areas as investigation candidates and disclose the gap. An empty committed diff is not evidence that nothing changed outside git. T3 and fallback ranges keep the same risk/regression procedure.

A change to one place can break another. For each changed area, look for:

- **Shared code** — functions, components, or modules called from other flows (`git grep` the changed symbol).
- **Shared data** — tables, fields, files, caches, or stored formats written by the change and read elsewhere.
- **Contracts** — API request/response shapes, events, CLI flags, file formats, error messages consumers may rely on.
- **Removed behavior** — anything deleted: who relied on it?
- **Ordering and state** — new required steps, changed defaults, migrations that must run first.

Label every indirect impact `[Inferred]` and state the path: "X changed → Y calls X → test Y".

## Confirmation vs regression after a fix

These are two different tests; recommend both.

1. **Confirmation (re-test)** — repeat the exact original reproduction steps; the symptom must be gone. Also try close variants (other data, other role, other platform) — fixes are often narrower than the defect.
2. **Regression** — test what surrounds the fix: other callers of the changed code, the happy path of the same feature, and the cases the fix might have over-corrected.

## Where manual testing adds most value

Automated evidence recorded by Kyro usually covers logic and contracts. Point the tester at what it rarely covers:

- visual layout, responsiveness, copy and translations;
- accessibility (keyboard, screen reader, contrast);
- per-role permissions with real accounts;
- real data volumes and messy data;
- cross-browser, OS, or device compatibility;
- interruptions: network loss, refresh, back button, double submit, timeouts, concurrent users;
- the end-to-end journey across modules.

## Answering "what else should I test?"

1. List the changes and risks identified in this session.
2. Mark which the tester reported as tested, and their outcome.
3. Recommend the highest remaining risk first, with the reason.
4. State explicitly what will remain untested if they stop now, so the decision to stop is theirs and informed.
