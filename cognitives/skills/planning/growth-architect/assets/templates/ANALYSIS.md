# Analysis Output Template

Use this template when generating a strategic analysis file. Replace all `[bracketed]` placeholders.

**Location:** `{output_dir}/analysis/` (create if doesn't exist)
**Naming:** `NNNN-[topic]-analysis.md` — 4-digit sequential numbering (check existing files for next number)

---

## Template

```markdown
---
analysis_id: NNNN
title: "[Descriptive Title]"
date: YYYY-MM-DD
author: growth-architect
type: [strategic|architectural|product|growth|technical]
status: [active|archived|superseded]
related_files:
  - {output_dir}/analysis/[previous-analysis].md
  - {output_dir}/adr/NNNN-[relevant-adr].md
  - README.md or CLAUDE.md
tags: [mvp, scalability, architecture, etc.]
---

# [Title] - Strategic Analysis

**Analysis Date:** YYYY-MM-DD
**Project Phase:** [Idea|MVP|Growth|Mature]
**Strategic Focus:** [Primary area of concern]

---

## Executive Summary

[2-3 sentence high-level summary for quick scanning]

---

## Context and Background

[What was analyzed, why, and what existing context informed this analysis]

**Referenced Inputs:**
- `{output_dir}/analysis/[file]` - [Previous analysis connections]
- `README.md` or `CLAUDE.md` - [Project conventions]

---

## Core Findings

[Main body of analysis following the 10-step methodology]

---

## Action Items and Handoffs

### For Immediate Execution
- [ ] [Action 1]
- [ ] [Action 2]

### Strategic Brief for sprint-forge
- [ ] Create sprint plan for: [Feature/Phase Name]
- [ ] Prioritize: [Specific recommendations]
- [ ] Dependencies: [What must be in place first]

### For User/Team
- [ ] Validate assumption: [Hypothesis to test]
- [ ] Gather data on: [Metric/insight needed]

---

## Success Metrics

- [KPI 1]: [Target/Baseline]
- [KPI 2]: [Target/Baseline]

---

## Related Artifacts

- Previous Analysis: `{output_dir}/analysis/[filename].md`
- ADRs: `{output_dir}/adr/[filename].md`
- Project Docs: `README.md`, `CLAUDE.md`, etc.

---

## Notes and Assumptions

[Important context, caveats, or assumptions made during analysis]
```

---

## Post-Generation Actions

After creating the analysis file:

1. **Inform the user** of the file location
2. **Suggest next steps:**
   - "Should I invoke sprint-forge to convert these recommendations into an execution plan?"
   - "Should I create ADRs for the architectural decisions identified?"
3. **Update references:** If this supersedes a previous analysis, note it in the old file's frontmatter
4. **Create ADR if needed:** If significant architectural decisions were made, use the [ADR template](ADR.md)
