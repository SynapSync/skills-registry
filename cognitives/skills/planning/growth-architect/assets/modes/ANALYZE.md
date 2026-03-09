# ANALYZE Mode

The 10-step strategic analysis methodology. Follow each step sequentially. Skip steps only when explicitly not applicable (e.g., Step 6 for non-technical projects).

---

## Step 0 — Context Loading (MANDATORY FIRST STEP)

Before any analysis, check for existing context:

```bash
# Check for previous analyses
ls {output_dir}/analysis/*.md 2>/dev/null | sort -r | head -5

# Check for architecture decisions
ls {output_dir}/adr/*.md 2>/dev/null | sort -r | head -5

# Check for project conventions
ls CLAUDE.md README.md 2>/dev/null
```

**Then state:**
- What existing context was found
- What key insights from previous work inform this analysis
- What project conventions or patterns are already established

**If no context exists:** Proceed with fresh analysis but note this is a greenfield assessment.

---

## Step 1 — Project Understanding

- Rephrase what the user is building or deciding
- Identify:
  - Explicit goals
  - Implicit goals
  - Unvalidated assumptions
- Determine project state:
  - Idea
  - MVP
  - Growing product
  - Mature product
- Cross-reference with past analyses if available

---

## Step 2 — Critical Diagnosis

- What is well designed
- What is weak or incomplete
- **Technical alignment:** Does current implementation support strategic goals?
- Risks:
  - Technical
  - Product
  - Market
  - Execution
- Decisions that could become costly if not corrected early
- **Conflicts with previous recommendations:** Have past suggestions been implemented? Why or why not?

> Ask questions **only** if strictly necessary to move forward.

---

## Step 3 — Strategic Value Contribution (Mandatory)

The agent must **ALWAYS** contribute at least one of these:

- Ideas for new features
- Architecture or design improvements
- MVP simplification
- New growth paths
- Automations
- Strategic AI use
- Potential integrations
- New user segments
- UX or flow improvements

**Never** limit yourself to answering what is asked: always push the project forward.

---

## Step 4 — Project Evolutionary Vision

### Short Term (0-3 months)
- What to validate first
- What to build and what to avoid
- Early signs of success or failure

### Medium Term (3-12 months)
- Scalability
- Product refinement
- Feature expansion
- Monetization (if applicable)
- Technical evolution needed

### Long Term (1-3+ years)
- Platform / ecosystem vision
- Competitive differentiation
- Potential pivots
- Growth or consolidation scenarios

---

## Step 5 — MVP Proposal or Next Step

- MVP or iteration objective
- Minimum necessary features
- What hypotheses are validated
- Key metrics (KPIs)
- Technical requirements: What system changes are needed?

---

## Step 6 — System Design (if applicable)

- High-level architecture
- Main components
- Suggested stack (agnostic, with criteria)
- Integration with existing codebase
- Considerations for:
  - Scalability
  - Security
  - Performance
  - Maintainability

May include:
- Mermaid diagrams
- Data flows
- Conceptual diagrams

**If significant architectural changes are proposed:**
- Create an ADR — read [templates/ADR.md](../templates/ADR.md) for the template and lifecycle
- Provide clear rationale for technical choices

---

## Step 7 — UX / Product (Conceptual)

- Main user flows
- Key screens
- Probable friction points
- Recommended UX principles

---

## Step 8 — Viability and Market Context

- What problem is actually solved
- For whom
- Existing alternatives
- Differential advantage
- Adoption risks

---

## Step 9 — Strategic Handoff

Produce clear, actionable output:

- Clear next steps with priorities
- Priority decisions the user must make
- What to validate before scaling
- Common mistakes to avoid

**Handoff to sprint-forge:**

If the analysis identifies concrete work to execute, suggest:

> "The strategic analysis is complete. To convert these recommendations into an execution plan with sprints, use sprint-forge in INIT mode — it can consume this analysis as context input."

Provide a summary block for sprint-forge consumption:

```markdown
## Strategic Brief for sprint-forge

**Project phase:** [Idea|MVP|Growth|Mature]
**Key findings:** [3-5 bullet points]
**Recommended first phase:** [description]
**Critical constraints:** [blockers, risks, dependencies]
**ADRs created:** [list or "none"]
```

**Output file generation:**

When generating the analysis file, read [templates/ANALYSIS.md](../templates/ANALYSIS.md) for the output template.
When creating ADRs, read [templates/ADR.md](../templates/ADR.md) for the ADR template.
