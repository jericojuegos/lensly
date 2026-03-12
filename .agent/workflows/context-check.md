---
description: Context Health Check — validates consistency across all .agent/ files
---

# 🩺 Workflow: Context Health Check

**Purpose:** Catch rule conflicts, planning drift, stale knowledge, and structural issues before they cause AI agents to hallucinate or ignore constraints.

**When to run:**
- Start or end of every sprint
- After a prolonged break from the codebase
- Whenever an AI agent starts behaving unexpectedly or ignoring documented rules

---

## Checklist

### 1. Rule Consolidation
- [ ] Read all files in `rules/`
- [ ] Are there any contradictions between files? (e.g., `task-rules.md` requires an action that `budget-guard.md` prohibits)
- [ ] Is any single rule file getting too long to fit in one prompt window? (>150 lines is a warning sign)
- [ ] Are there rules duplicated across multiple files that should be consolidated?

### 2. Planning Alignment
- [ ] Read `planning/00-roadmap.md` and `planning/01-active-sprint.md`
- [ ] Does the sprint's current task match the active phase in the roadmap?
- [ ] Are there tasks marked `[x]` in the sprint board but still `[ ]` in the roadmap? (sync drift)
- [ ] Are there tasks marked `[x]` in the sprint board that should have been migrated to `planning/03-history.md`?
- [ ] Does the sprint board have a `## 🧊 HIBERNATION STATE` block still present that needs to be cleared?

### 3. Knowledge Base Verification
- [ ] Read `knowledge/architecture.md` and `knowledge/tech-stack.md`
- [ ] Do the documented technologies match actual dependencies in `package.json` / `requirements.txt`?
- [ ] Has any service boundary changed in the codebase that isn't reflected in `architecture.md`?
- [ ] Does `knowledge/spec.md` contain any content already covered in `architecture.md` or `tech-stack.md`? (duplication check)

### 4. Decisions Log
- [ ] Read `knowledge/decisions.md`
- [ ] Have any major technical decisions been made recently that aren't logged?
- [ ] Are any logged decisions now outdated or reversed?

---

## Output Format

After completing the checklist, produce a report in this exact structure:

```markdown
## 🩺 Context Health Report — [DATE]

### CRITICAL (fix before next coding session)
- [issue description] → [proposed fix]

### WARNING (fix before end of sprint)
- [issue description] → [proposed fix]

### INFO (low priority / optional improvements)
- [issue description] → [proposed fix]

### ✅ Healthy
- [list of areas with no issues found]
```

**Severity guide:**
- `CRITICAL` — Rule conflicts, broken cross-references, agent will get contradictory instructions
- `WARNING` — Stale state, planning drift, roadmap/sprint out of sync
- `INFO` — Duplication, verbosity, structural improvements

After delivering the report, ask:
> *"Would you like me to fix any of the WARNING or CRITICAL items now?"*

Do not auto-fix anything without explicit user approval.
