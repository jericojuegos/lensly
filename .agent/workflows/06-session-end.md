---
description: At the end of every work session, whether you finished a task or not.
---

# 🧊 Workflow 06 — Session End
## Runs: At the end of every work session, whether you finished a task or not.
## Next: → Nothing. Brain is frozen. Resume with `03-session-start.md` next session.

---

> 5 minutes. The brain must be accurate when you leave.
> If you skip this, the next session starts with stale context — agents hallucinate.

---

## Step 1 — Honest Task Status

Look at the task you were working on today. What is its actual state?

| State | Action |
|-------|--------|
| Fully complete + self-reviewed | Mark `[x]` — run `05-task-complete.md` first |
| Mostly done but not verified | Keep as `[>]` — note exact stopping point in Activity Log |
| Abandoned / approach changed | Mark `[ ]` — add a note explaining what happened |
| Discovered it's blocked | Mark `[!]` — add blocker to 🛑 section with clear description |

**Do not mark `[x]` unless the self-review in `04-task-loop.md` passed.**

---

## Step 2 — Insert the Hibernation Block

Open `planning/01-active-sprint.md`. Insert this block at the **very top of the file**, above everything else. Fill it out honestly — this is what the next session reads first.

```markdown
## 🧊 HIBERNATION STATE (YYYY-MM-DD)

- **Last action:** [Exact last thing done — file name, function name, line if relevant]
- **Current state:** [Honest status — working / broken / partially complete / untested]
- **Files in flight:** [Files actively being edited that are not yet committed]
- **Wake-up task:** [The single first thing to do next session — be specific]
- **Open questions:** [Anything unresolved that needs a decision before continuing]
```

**Filled example:**
```markdown
## 🧊 HIBERNATION STATE (2026-03-15)

- **Last action:** Built `supabase/functions/scraper/lib/robots.ts` — parses User-agent: * and User-agent: Lensly rules.
- **Current state:** Working for standard robots.txt. Not yet tested against `Disallow: /` (full block) or `Allow:` overrides.
- **Files in flight:** `supabase/functions/scraper/lib/robots.ts`
- **Wake-up task:** Write tests for robots.ts edge cases — full block, Allow override, missing robots.txt (treat as allow-all).
- **Open questions:** Should a missing robots.txt be treated as allow-all or block-all? Convention is allow-all — confirm before implementing.
```

**Rules:**
- This block lives at the top of the file — above the sprint title if needed
- One block only — remove the previous one before adding a new one
- `03-session-start.md` will read this block and then delete it

---

## Step 3 — Update Activity Log

Append to Activity Log in `planning/01-active-sprint.md`:

```
| YYYY-MM-DD | Session ended. [One line: what was accomplished, what was left mid-task if applicable.] |
```

---

## Step 4 — Check for New Decisions

Did today's session produce any architectural decision that isn't already in `knowledge/decisions.md`?

Ask: *"Did I choose between two approaches today?"*

- Yes → add ADR to `knowledge/decisions.md` now, before you forget the reasoning
- No → continue

---

## Step 5 — Commit the Brain

```bash
git add .agent/
git commit -m "chore(brain): session end YYYY-MM-DD — [one line summary]"
```

Examples:
```bash
git commit -m "chore(brain): session end 2026-03-15 — robots.ts complete, scraper.ts in progress"
git commit -m "chore(brain): session end 2026-03-16 — Phase 0 scaffold complete, Phase 1 starting"
```

**Commit the brain separately from code commits.** Brain changes and code changes should not be mixed in one commit.

---

## Step 6 — Final Brain Health Check

Before closing the editor, confirm:

- [ ] `planning/01-active-sprint.md` accurately reflects the current state of every task
- [ ] The Hibernation Note tells a stranger exactly where to pick up tomorrow
- [ ] No tasks marked `[x]` that aren't actually complete
- [ ] Activity Log entry added for today

---

## ✅ Brain is frozen. Project can be safely resumed.

**→ Resume next session: `03-session-start.md`**
