# Oracles — Deciding What "Correct" Means

The hardest part of testing is knowing what the right behavior is. An **oracle** is a source that lets the tester recognize a problem. No single oracle is authoritative: they can disagree, and the disagreement itself is a finding.

This asset adapts the FEW HICCUPPS heuristic (Bach, Bolton) to the sources a Kyro project actually has.

## Oracles mapped to sources

| Oracle | Question | Source in this project | Typical label |
|--------|----------|------------------------|---------------|
| **History** | Is it consistent with how it behaved before? | `git show <old-rev>:<path>`, the diff, old release notes | `[Verified in code/git]` |
| **Claims** | Is it consistent with what we said it does? | Acceptance criteria, spec scenarios (S*), requirements (R*), docs, release notes | `[Declared in plan/docs]` |
| **Purpose** | Is it consistent with why it was built? | Matured ideas (`plan/`), Work briefs, scope objective | `[Declared in plan/docs]` |
| **Product** | Is it consistent with similar behavior elsewhere in this product? | Other code paths, other screens with the same pattern | `[Verified in code/git]` / `[Inferred]` |
| **Image** | Is it consistent with how the organization wants to look? | Design docs, copy guidelines, if present | `[Declared in plan/docs]` / `[Unknown]` |
| **Comparable products** | Is it consistent with how comparable software behaves? | General knowledge — weakest | `[Inferred]` |
| **User expectations** | Would a reasonable user expect this? | Tester's judgment, user flows | Tester decides |
| **Statutes / standards** | Is it consistent with laws, regulations, accessibility standards? | Requirements, WCAG, domain rules | `[Declared in plan/docs]` / `[Inferred]` |
| **Explainability** | Can we explain the behavior at all? | If nobody can, that is a finding | — |

## Procedure for "is this a bug?"

1. Restate what the tester observed, precisely: action, data, environment, result.
2. Gather at least **History**, **Claims**, and **Purpose** when the tier allows. In T3, Claims come only from in-repo docs, CHANGELOG, and commit bodies, and Purpose is usually unknown — say so.
3. Present a comparison:

   ```text
   Before (History):   <behavior> [label] (source)
   Plan says (Claims): <behavior> [label] (source)
   Intent (Purpose):   <behavior> [label] (source)
   Observed:           <what the tester saw>
   ```

4. Classify the situation — as a hypothesis for the tester to decide:
   - **Probable defect** — observed differs from claims and purpose.
   - **Probable intended change** — observed matches claims, differs from history. Still worth confirming the change was communicated.
   - **Plan gap or plan error** — claims are silent, contradictory, or contradict purpose. Recommend raising it with the team; this is a valid QA finding.
   - **Non-goal** — the area is in `spec.nonGoals`. Mention it, but the tester decides whether it still matters to users.
   - **Undetermined** — sources are missing; say what would settle it.
5. Suggest one or two quick checks that would confirm or rule out the hypothesis (different data, different role, previous version, repeat steps).

Never conclude for the tester. Phrase as "this looks like…, because…; you can confirm by…".

## When oracles disagree

A plan that contradicts the original intent, or code that contradicts the plan, is exactly what the tester should surface. Present the contradiction neutrally with both sources; do not pick a winner.
