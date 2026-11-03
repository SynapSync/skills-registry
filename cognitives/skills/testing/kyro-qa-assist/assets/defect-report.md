# Defect Report — Making Findings Reproducible and Traceable

A good report lets a developer reproduce the problem on the first try and understand why it is a problem. Help the tester produce one; the tester owns its content and decides whether to file it.

## Before writing

1. **Reproduce** — can the tester make it happen again? Note how often (always, 3 of 5, once).
2. **Isolate** — reduce to the minimum steps; vary one thing at a time (data, role, browser, environment) to find what triggers it.
3. **Check the previous version** if possible — if it worked before, it is a regression, which usually raises priority.
4. **Find the oracle** — what source says the behavior is wrong (see `oracles.md`)? Without one, report it as a question, not a defect.

## Template

```text
Title:        <what fails> when <condition> — specific, searchable
Environment:  <version/commit or build>, <environment>, <browser/OS/device>, <role/account>
Preconditions:<data and state needed>
Steps:        1. …  2. …  3. …
Expected:     <behavior> — source: <plan/criterion/scenario/previous version> [label]
Actual:       <behavior observed, exact messages>
Frequency:    <always / intermittent n of m>
Regression:   <yes, worked in <version> / no / unknown>
Evidence:     <screenshots, recordings, logs, timestamps>
Severity:     <Critical|Major|Minor|Trivial> — impact on users/business
Priority:     <suggested; the team decides>
Related:      <scope / task / scenario / commit — mark inferred links [Inferred]>
Notes:        <workaround, isolation results, hypotheses [Inferred]>
```

## Severity is not priority

- **Severity** — how bad the impact is (data loss is critical even if rare).
- **Priority** — how soon to fix (a typo on the home page may be trivial severity but high priority).
The tester proposes severity; priority is usually a team decision.

## Linking to the change

When possible, connect the defect to the change that likely introduced it: the commit, the task, the scenario. Mark the link `[Inferred]` unless git proves it. This helps the developer and also tells the tester which neighbors to regression-test after the fix (see `regression.md` → Confirmation vs regression).

## Writing style

Facts over opinions. Neutral tone. One defect per report. Exact text of error messages. No speculation in the Steps / Actual fields — hypotheses go in Notes, labeled.
