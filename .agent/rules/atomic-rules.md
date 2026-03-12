---
trigger: always_on
---

# ⚛️ Rule: Atomic Execution

**Purpose:** Prevent error cascades, protect API budget (token limits), and ensure code is "Green" (passing) before adding complexity.

---

## 0. Session Startup (Prime Directive)
At the start of every session, in this order:

1. **DISCOVER** — Read `README.md` to orient to the project.
2. **ALIGN** — Read `planning/01-active-sprint.md`. Check for **AD-HOC TASK** overrides.
3. **VALIDATE** — Skim `rules/` to confirm constraints are fresh in context.
4. **LEARN** — Read `knowledge/` files before proposing any architectural change.
5. **PLAN** — Check `workflows/` for an applicable SOP before free-forming a solution.
6. **EXECUTE** — Follow the Standard Protocol below.

> ⚠️ Steps 1–5 are read-only orientation. Do not write any code until step 6.

---

## 1. The Standard Protocol
Execute tasks in this strict, isolated loop:

1. **READ** — Identify the *next* unchecked item in `planning/01-active-sprint.md` (or the Ad-Hoc section if triggered).

2. **ISOLATE** — Can this task be implemented and verified without requiring another incomplete task to exist first?
   - This covers both **compile-time coupling** (imports, type references) and **runtime coupling** (a DB migration that a new API handler depends on).
   - **YES → proceed to step 3.**
   - **NO → switch to Section 2 (Coupled Logic).**

3. **EXECUTE** — Implement the task.

4. **VERIFY** — Confirm the task works. Verification must cover *all layers the task touches*:
   - Backend change → API test or unit test must pass.
   - Frontend change → UI renders without console errors.
   - Full-stack change → both of the above.
   - Pipeline/worker change → relevant unit or integration test passes.
   - If verification fails: do NOT proceed. Rollback or stash, mark `[!]` in the sprint board, and pause for user input.

5. **UPDATE TRACKER** — Immediately mark the task as `[x]` in `planning/01-active-sprint.md`. Do not skip or defer this step.

6. **COMMIT** — `git commit -m "type(scope): task name"`
   - Use `feat`, `fix`, `refactor`, `docs`, or `chore` as appropriate.

7. **REPEAT** — Return to step 1 and pick up the next unchecked task. Do not re-run the Session Startup (Section 0) between tasks — that runs once per session only.

---

## 2. Exception: Coupled Logic (The "Batch" Rule)
*Use this ONLY when tasks are physically dependent on each other.*

If Task A cannot be compiled or run correctly without Task B existing (e.g., an API route that imports a model that doesn't exist yet):

1. **DECLARE IT** — State explicitly: *"Tasks [A] and [B] are coupled. Executing as an Atomic Group."*
2. **NOTIFY BUDGET GUARD** — A coupled batch may touch more than 3 files. This is an authorized exception to the Budget Guard file-scope threshold. No pause required.
3. **BATCH EXECUTE** — Implement all coupled tasks in the same turn.
4. **JOINT VERIFY** — Verify the group together as a single unit.
5. **SYNC & COMMIT** — Mark all tasks `[x]`, then commit as one unit.

---

## ⛔ 3. Strict Prohibitions

- **YAGNI:** Never implement something that isn't required for the current atomic step to function. If it's not needed right now, it doesn't get written.
- **No silent batching:** Never group unrelated tasks without declaring it. Unrelated batching wastes tokens and increases hallucination risk.
- **No skipping VERIFY:** A task is not done until it is verified. "It looks right" is not verification.
- **No skipping UPDATE TRACKER:** Marking `[x]` is not optional. It is part of the definition of done.

---

## 4. Recovery Protocol
After any hard stop (failed verify, Budget Guard trigger, or 3-Strike rule):

1. Do NOT resume from memory.
2. Re-read `planning/01-active-sprint.md`.
3. Find the last `[x]` item — that is your baseline.
4. The next unchecked `[ ]` item is your restart point.
5. Report your restart point to the user before writing any code.
