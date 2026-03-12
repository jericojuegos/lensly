---
description: When every task in the active sprint is [x].
---

# 🏁 Workflow 07 — Phase Complete
## Runs: When every task in the active sprint is [x].
## Next: → `03-session-start.md` for Phase 1 (or whichever phase is next)

---

> A phase is complete when every task in `planning/01-active-sprint.md` is `[x]` or `[-]`.
> This workflow archives the completed phase and sets up the next one.

---

## Step 1 — Verify Phase Is Actually Done

Read every task in `planning/01-active-sprint.md`. For each:

- `[x]` complete → ✅
- `[-]` deferred → ✅ (intentionally skipped — reason documented)
- `[ ]` pending → ❌ stop. Finish this task before running this workflow.
- `[!]` blocked → ❌ stop. Resolve the blocker or explicitly defer with `[-]` and a reason.
- `[>]` in progress → ❌ stop. Complete this task first.

**All tasks must be `[x]` or `[-]` before continuing.**

---

## Step 2 — Write Phase History Entry

Open `planning/03-history.md`. Append a new phase entry:

```markdown
## Phase N — [Phase Name]
**Completed:** YYYY-MM-DD
**Goal:** [Copy the phase goal from 00-roadmap.md]

### What Was Built
[Group by area — e.g. Web, Mobile, Edge Functions, Database]
[List what was actually built — be specific, include file names]

### Key Decisions Made This Phase
[Any ADRs added during this phase]

### Blockers Encountered
[What got blocked, how it was resolved or deferred]

### What Was Deferred (and Why)
[Any [-] tasks, with reason]

### Activity Log
[Copy the Activity Log from 01-active-sprint.md for this phase]
```

**History is permanent. Never delete or edit it after writing.**

---

## Step 3 — Update the Roadmap

Open `planning/00-roadmap.md`:

1. Mark the completed phase header: change `[>]` to `[x]`
2. Mark all completed tasks in that phase: `[x]`
3. Mark any deferred tasks: `[-]`
4. Mark the next phase header: change `[ ]` to `[>]`

---

## Step 4 — Set Up the Next Sprint

Open `planning/01-active-sprint.md`:

1. **Clear** all completed tasks (they're now in `03-history.md`)
2. **Update** the Sprint Focus line to the new phase
3. **Copy** the new phase tasks from `planning/00-roadmap.md` as `[ ]` items
4. **Explode** each high-level task into specific subtasks with file paths

For Lensly Phase 0 → Phase 1, the new sprint should open with:

```markdown
# 🏃 Active Sprint — Lensly

> Sprint Focus: Phase 1 MVP — Full intelligence loop: scrape → diff → AI analyse → notify.

## 🔄 Current Tasks — Phase 1

### Scraper (Edge Function)
- [ ] `supabase/functions/scraper/lib/robots.ts` — parse + check robots.txt per URL
- [ ] `supabase/functions/scraper/lib/scraper.ts` — fetch with 10s timeout + 3-retry backoff
- [ ] `supabase/functions/scraper/lib/extractor.ts` — strip HTML → extract clean text
- [ ] `supabase/functions/scraper/lib/differ.ts` — compute text diff (added/removed lines)
- [ ] `supabase/functions/scraper/index.ts` — full flow: robots → fetch → extract → diff → store
...
```

5. **Set Blocked / Waiting** — copy any unresolved blockers from previous sprint
6. **Clear the Hibernation Note** — remove previous phase's hibernation note if present

---

## Step 5 — Backlog Audit

Open `planning/02-backlog.md`. Quick scan:

- Are there any deferred items that should now move to active (based on new phase)?
- Are there any rejected features being inadvertently built in the new sprint?

Adjust if needed.

---

## Step 6 — Update Activity Log

In the new `planning/01-active-sprint.md`, start a fresh Activity Log:

```markdown
## 📝 Activity Log

| Date | Action |
|------|--------|
| YYYY-MM-DD | Phase N complete. Phase N+1 sprint initialized. [N] tasks queued. |
```

---

## Step 7 — Commit Everything

```bash
git add .agent/
git commit -m "chore(brain): phase N complete — archive to history, Phase N+1 sprint ready"
```

---

## Step 8 — Lensly Phase-Specific Checks

### After Phase 0 → Phase 1
Before starting Phase 1 scraper work, verify:
- [ ] Supabase service role key is set in Edge Function secrets
- [ ] Anthropic API key is set in Edge Function secrets
- [ ] All 4 Edge Functions are deployed and responding (stubs)
- [ ] DB tables and RLS verified in Supabase dashboard
- [ ] Resend account setup — email sending will be needed in Phase 1 digester

### After Phase 1 → Phase 2
Before starting Phase 2:
- [ ] At least one full scrape → analyse → notify cycle tested end-to-end with real competitor
- [ ] Email digest sent and received successfully
- [ ] Credit system working — balance tracked, deducted correctly, free credits on signup

---

## ✅ Phase Complete

**Old phase is archived. New phase is ready.**
**→ Next session: `03-session-start.md`**
