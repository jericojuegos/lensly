---
description: Every single work session, before writing any code.
---

# ☀️ Workflow 03 — Session Start
## Runs: Every single work session, before writing any code.
## Next: → `04-task-loop.md`

---

> Takes 5 minutes. Saves hours of working on the wrong thing.

---

## Step 0 — Read and Clear the Hibernation Block

Open `planning/01-active-sprint.md`. Look at the very top of the file.

**Is there a `🧊 HIBERNATION STATE` block?**

**Yes →**
1. Read it fully — all 5 fields
2. Note the **Wake-up task** — this is your starting point for today
3. Note any **Open questions** — these need a decision before you can proceed
4. **Delete the entire block** from the file — it has done its job
5. Save the file
6. Continue to Step 1

**No →**
- This is either the very first session, or the last session ended without hibernating
- Continue to Step 1 and orient from the sprint file directly

---

## Step 1 — Orient (read-only, 3 minutes)

Read these files in order. Do not skip:

1. `planning/01-active-sprint.md` — what is the current state?
   - What tasks are `[x]` done?
   - What tasks are `[!]` blocked?
   - What is the single next `[ ]` task?
   - Is there a `[>]` task already in progress?

2. `planning/01-active-sprint.md` Activity Log — what happened last session?

3. If anything feels unclear → read `knowledge/spec.md` section relevant to today's task.

---

## Step 2 — Check for Blockers

Scan the 🛑 Blocked / Waiting section in `planning/01-active-sprint.md`.

For each blocker:
- Is it still blocked? → leave it, pick a different task
- Is it now unblocked? → remove from blocked list, add to `[ ]` tasks, update Activity Log

**Lensly-specific blockers to check:**
- [ ] Resend account — needed before `digester/index.ts` AI + email work
- [ ] Stripe account — needed before `webhooks/payment/route.ts`
- [ ] Anthropic API key in Edge Function secrets — needed before `analyser/index.ts`

---

## Step 3 — Pick the Single Next Task

Look at `planning/01-active-sprint.md` pending tasks. Pick **one** — the next sequential `[ ]` task.

**Rules for picking:**
- Follow the order in the sprint file — tasks are sequenced intentionally
- Never pick a task that depends on an incomplete one above it
- If the next task is blocked → skip it, pick the next unblocked one, note the skip in Activity Log
- If all tasks are blocked → this session is a dependency-resolution session, not a coding session

Write down your chosen task before moving on:
> **Today's task:** `___________________________________________`

---

## Step 4 — Mark the Task Active

In `planning/01-active-sprint.md`, change the chosen task from `[ ]` to `[>]`:

```markdown
- [>] `supabase/functions/scraper/index.ts` — full scrape flow
```

Only one task should be `[>]` at a time. If there's already a `[>]` from a previous session, either complete it first or re-mark it `[ ]` with a note in the Activity Log.

---

## Step 5 — Read the Relevant Brain Section

For today's task, read the matching section in the brain:

| If building... | Read... |
|---------------|---------|
| Scraper Edge Function | `knowledge/architecture.md` — "Core Data Flows" → Flow 1 |
| Analyser Edge Function | `knowledge/architecture.md` — Flow 2 + `rules/coding-standards.md` credit pattern |
| Notifier Edge Function | `knowledge/architecture.md` — Flow 3 |
| Digester Edge Function | `knowledge/architecture.md` — Flow 4 |
| Any Route Handler | `rules/coding-standards.md` — Route Handler pattern |
| Any UI component | `knowledge/architecture.md` — Web File Structure |
| robots.txt logic | `knowledge/spec.md` — "robots.txt Policy" section |
| AI prompt | `knowledge/architecture.md` — "AI Layer" table |

---

## Step 6 — Confirm the Non-Negotiables for Today's Task

Before writing code, confirm which critical constraints apply to today's task:

**If today's task touches scraping:**
- [ ] robots.txt check comes before the fetch — always
- [ ] This runs in Edge Functions — not in a Route Handler

**If today's task touches AI:**
- [ ] Only the diff goes to the model — not full pages
- [ ] Diff is truncated to `MAX_DIFF_LENGTH_FOR_AI` before the call
- [ ] Credit balance is checked before the AI call
- [ ] Credit deducted only after successful response

**If today's task touches the database:**
- [ ] New table has RLS policy
- [ ] `SUPABASE_SERVICE_ROLE_KEY` used only in server-side code

**If today's task touches notifications:**
- [ ] Slack webhook URL not returned in any API response
- [ ] Empty digests are skipped — never send if no significant changes

---

## ✅ Session Start Complete

You know what you're building, you've read the relevant brain section, and the non-negotiables are clear.

**→ Next: `04-task-loop.md`**
