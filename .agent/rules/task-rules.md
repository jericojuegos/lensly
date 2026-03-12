---
trigger: always_on
---

# Task Management Protocol

## 1. Task Updating Syntax
- **Never** mark a parent task as `[x]` unless ALL sub-tasks are complete.
- **Skipped/Deferred:** Use `[-]` or append `(Deferred)` to the text. Keep the box unchecked `[ ]`.
- **Sync Timing:** Update `.agent/planning/01-active-sprint.md` *immediately* after verifying a step.

## 2. Partial Completion (The "Half-Done" Rule)
If a task involves full-stack work (Backend + UI) and you only complete one side:
1.  **Do NOT** check the main item.
2.  **Explicitly break it down**:
    - `[x] Backend Logic`
    - `[ ] Frontend UI`

## 3. Verification Requirement (Definition of Done)
A task is considered **complete** `[x]` ONLY when:
1.  The implementation exists in the codebase.
2.  The expected behavior is **verified** (manual test, API test, or unit test).
3.  **Evidence:** If asked, you must be able to state *how* it was verified.
4.  **UAT Checklist:** At end-of-sprint, generate a UAT checklist per `workflows/uat-checklist.md`. The agent handles Levels 1–2 (automated tests + checklist); the user handles Level 3 (judgment).

## 4. Moving & Refinement (The "Drift" Protocol)
If a task cannot be completed in the current Sprint/Phase:
1.  **Create a New Section** in `01-active-sprint.md` (e.g., "## Sprint 2: UI Polish").
2.  **Move the Task:** Cut and paste the task to the new section.
3.  **Leave a Marker:** In the original location, mark it as `[->]` and write `(Moved to Sprint 2)`.
    - *Example:* `[->] Create Wizard UI (Moved to Sprint 2)`

## 5. Scope Creep & Discovery
- **Blocked Tasks:** Mark as `[!]` and add reason.
    - *Example:* `[!] Payment API (Blocked: waiting for credentials)`
- **New Discoveries:** If you discover new work that is not in the plan:
    1.  **Do NOT** secretly do it.
    2.  **Add it** to the `## 🐛 Ad-Hoc / Side Quests` section of `01-active-sprint.md` ONLY if the user explicitly uses the phrase **"AD-HOC TASK:"** or asks for a small UI/config tweak.
    3.  **STOP & ASK** if the task implies **high complexity** (e.g., would take a human >30 mins, or touches >5 files).

## 6. Source of Truth
- **Active Tasks:** `.agent/planning/01-active-sprint.md` is the Living Document.
- **Long Term:** `.agent/planning/00-roadmap.md` is the Vision.
- **Conflict:** If `Active Tasks` contradicts `Roadmap`, `Active Tasks` wins for *today*, but flag it for *tomorrow*.
