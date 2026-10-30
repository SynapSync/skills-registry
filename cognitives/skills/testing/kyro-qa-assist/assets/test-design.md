# Test Design — Turning Changes into Concrete Tests

## From Kyro artifacts to test ideas

- A **spec scenario** (given/when/then) is already an acceptance test outline. Make it concrete: real data, real screen or command, observable result.
- An **acceptance criterion** usually hides several tests: the positive case, each boundary, and each way it can fail.
- A **non-goal** tells you what is intentionally not supported — test that the product fails gracefully there, not that it works.
- An **open question** means the expected behavior is undecided: flag any test in that area as needing a decision.
- **Evidence** lists what automation checked. Design manual tests for what it did not.

## Test case format

```text
TC-<n> · <short title>                         Priority: <High|Medium|Low>
Why:          <change or risk it covers> [label] (source)
Oracle:       <where the expected result comes from> [label]
Preconditions:<account/role, data, configuration, flags, previous steps>
Steps:        1. …  2. …  3. …
Expected:     <observable result> [label]
Also check:   <boundaries / negative variants worth running in the same session>
```

Keep steps in the tester's vocabulary: screens, buttons, commands, messages — not function names.

## Products without a screen

Not every product is a web or mobile app. Translate the same ideas to the product's real surface:

- **CLI tools** — the "screen" is the command, its flags, exit code, printed output, and the files it creates or changes. Run state-changing commands in a disposable folder.
- **APIs and libraries** — requests/calls, response shapes, status codes, error messages, and compatibility for existing consumers.
- **Agent instructions, skills, prompts, or configuration** — the behavior is what an agent does when following the text. Test by giving an agent a realistic request in a disposable project and observing what it asks, which files it reads or writes, and whether it follows the documented rules. Contradictions between instructions are high-value findings.
- **Data pipelines and jobs** — inputs, outputs, schedules, idempotency on re-run, and partial-failure recovery.

## Techniques — apply them to the concrete change

- **Equivalence partitioning** — group inputs that should behave the same; test one per group (valid, each kind of invalid).
- **Boundary values** — for any limit, test just below, at, and just above (for 1–100: 0, 1, 100, 101). Include empty, zero, maximum length, and the maximum count of items.
- **State transitions** — for anything with a lifecycle (order, payment, task, session), list the states and test valid transitions, forbidden transitions, and repeated transitions (cancel twice, pay after cancel).
- **Decision tables** — when the outcome depends on several conditions (role × plan × flag), list the combinations that matter and test each rule once.
- **Pairwise** — when combinations explode (browser × language × role), cover every pair of values instead of every combination.
- **Error guessing** — use known weak spots: special characters, copy-paste, time zones and dates, slow network, double clicks, very large files, concurrent edits.

## Negative and edge cases worth always considering

Invalid and missing input · unauthorized role · expired session · duplicate submission · interruption mid-operation · back/refresh · empty lists and first-time use · very large data · localization (long translations, right-to-left, number and date formats) · undo/cancel paths.

## Hidden preconditions — check the diff for them

Developers rarely mention these, and testers lose hours to them. Look for and surface:

- new or changed **environment variables**, config keys, or feature flags;
- **database migrations** or data backfills that must run first;
- new **roles or permissions** required to see the feature;
- required **test data** (accounts in a given state, seeded records);
- **dependency or version** changes (runtime, browser support, OS);
- deployment **order** between services.

## Exploratory charters

When the tester wants to explore rather than follow scripts, propose charters (Elisabeth Hendrickson, *Explore It!*):

```text
Explore <area or feature>
with <resources: data, roles, tools, conditions>
to discover <information: risks, failure modes, inconsistencies>
Timebox: <e.g. 45 min>
```

Ground each charter in a concrete change or risk from this session, and ask the tester to report back what they found so you can suggest the next charter.
