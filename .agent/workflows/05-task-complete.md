---
description: Immediately after finishing a task and passing self-review.
---

# ✅ Workflow 05 — Task Complete
## Runs: Immediately after finishing a task and passing self-review.
## Next: → `04-task-loop.md` (next task) OR `06-session-end.md` (stopping)

---

> 3 minutes. Do not skip. A task is not done until the brain reflects it.

---

## Step 1 — Update the Task Marker

In `planning/01-active-sprint.md`, change `[>]` to `[x]`:

```markdown
- [x] `supabase/functions/scraper/index.ts` — full scrape flow
```

---

## Step 2 — Check for Newly Unblocked Tasks

Completing this task might unblock something else. Scan the 🛑 Blocked section:

- Does completing today's task remove any blocker?
- Example: completing `lib/robots.ts` unblocks `lib/scraper.ts`

If yes → move the task from Blocked to `[ ]` in the sprint. Add a note in Activity Log.

---

## Step 3 — Check for Newly Discovered Blockers

Did building this task reveal something unexpected?

- A dependency that doesn't exist yet?
- An API that behaves differently than expected?
- An external service that needs setup first?

If yes → add a new entry to 🛑 Blocked / Waiting with a clear description.

---

## Step 4 — Update Activity Log

In `planning/01-active-sprint.md`, append to Activity Log:

```
| YYYY-MM-DD | ✅ [task name]. [One sentence about what was actually built or any notable decision made.] |
```

Examples:
```
| 2026-03-15 | ✅ lib/robots.ts. Parses User-agent: * and User-agent: Lensly rules. Returns false for disallowed paths. |
| 2026-03-15 | ✅ lib/scraper.ts. 10s timeout, 3 retries with exponential backoff. HEIC not relevant here — text only. |
| 2026-03-16 | ✅ lib/extractor.ts. Uses cheerio to strip nav/footer/scripts. Blocked: need to test against JS-heavy SPAs. |
```

---

## Step 5 — Sync the Roadmap

Open `planning/00-roadmap.md`. Find this task in the roadmap and mark it `[x]` there too.

**The sprint file and roadmap must always agree on completion status.**

---

## Step 6 — Is the Sprint Done?

Look at `planning/01-active-sprint.md`. Are all tasks `[x]` (or `[-]` deferred)?

- **No — tasks remain** → continue to next task
  - → **`04-task-loop.md`** if still in this session
  - → **`06-session-end.md`** if stopping for the day

- **Yes — all tasks done** → the phase is complete
  - → **`07-phase-complete.md`** before anything else

---

## ✅ Done

Brain is updated. Task is recorded. Ready to continue.

**→ Next task: `04-task-loop.md`**
**→ Stopping for today: `06-session-end.md`**
**→ Sprint complete: `07-phase-complete.md`**
