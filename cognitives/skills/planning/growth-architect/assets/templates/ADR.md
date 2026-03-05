# ADR Template and Lifecycle

Architecture Decision Records document significant architectural decisions with full context, alternatives, and consequences.

**Location:** `{output_dir}/adr/` (create if doesn't exist)
**Naming:** `NNNN-[decision-title].md` (e.g., `0001-use-postgresql-for-database.md`)
**Format:** Always use 4-digit zero-padded sequential numbering

---

## When to Create an ADR

Create an ADR when you make or recommend a **significant architectural decision**, such as:

- Technology stack choices (database, framework, language, infrastructure)
- System architecture patterns (microservices vs monolith, event-driven, etc.)
- Major technical trade-offs (performance vs maintainability, cost vs scalability)
- Integration patterns and external service choices
- Data architecture and storage strategies
- Security and authentication approaches
- API design patterns and versioning strategies
- Deployment and infrastructure decisions

**Do NOT create ADRs for:**
- Minor implementation details
- Routine bug fixes
- UI/UX tweaks without architectural impact
- Simple feature additions that follow existing patterns

---

## Template

```markdown
---
adr_id: NNNN
title: "[Decision Title]"
date: YYYY-MM-DD
status: [proposed|accepted|rejected|deprecated|superseded]
deciders: [growth-architect, user, team]
consulted: [list of people/roles consulted]
informed: [list of people/roles informed]
supersedes: [previous ADR number if applicable]
superseded_by: [future ADR number if superseded]
related_adrs: [list of related ADR numbers]
tags: [architecture, database, api, security, etc.]
---

# ADR NNNN: [Decision Title]

## Status

**[Proposed | Accepted | Rejected | Deprecated | Superseded]**

Date: YYYY-MM-DD

## Context

[Describe the architectural issue or decision that needs to be made. Include:]
- What is the technical or business problem?
- What constraints exist? (time, resources, skills, existing systems)
- What is the scope and impact of this decision?
- What triggered the need for this decision?

## Decision

[State the architectural decision clearly and concisely]

We will [decision statement].

## Consequences

### Positive

- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

### Negative

- [Trade-off 1]
- [Trade-off 2]
- [Risk or limitation]

### Neutral

- [Changes required]
- [Migration considerations]
- [Learning curve]

## Alternatives Considered

### Alternative 1: [Name]

**Description:** [Brief description]

**Pros:**
- [Pro 1]
- [Pro 2]

**Cons:**
- [Con 1]
- [Con 2]

**Why not chosen:** [Clear reasoning]

### Alternative 2: [Name]

[Same structure as above]

## Implementation Notes

- [Key implementation considerations]
- [Migration strategy if applicable]
- [Timeline or phases]
- [Who is responsible]

## References

- [Link to relevant analysis: `{output_dir}/analysis/NNNN-*.md`]
- [External documentation, articles, or resources]
- [Team discussions or meeting notes]

## Revision History

| Date | Author | Change | Reason |
|------|--------|--------|--------|
| YYYY-MM-DD | growth-architect | Created | Initial decision |
```

---

## Status Workflow

```
[Proposed] --> [Accepted] --> [Deprecated/Superseded]
                  |
               [Rejected]
```

- **Proposed:** Decision is being considered, not yet final
- **Accepted:** Decision has been approved and should be implemented
- **Rejected:** Decision was considered but not chosen (keep for historical context)
- **Deprecated:** Decision is no longer recommended but not yet replaced
- **Superseded:** Decision has been replaced by a new ADR (reference the new one)

---

## Integration with Analysis Files

When creating an analysis that includes significant architectural decisions:

1. **In the analysis file** (`{output_dir}/analysis/NNNN-*.md`):
   - Summarize the architectural decision in the Core Findings section
   - Reference the ADR number in Related Artifacts

2. **Create a separate ADR** (`{output_dir}/adr/NNNN-*.md`):
   - Document the full decision with context and alternatives
   - Link back to the analysis file
