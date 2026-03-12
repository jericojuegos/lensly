---
description: When something is broken and the cause isn't immediately obvious.
---

# 🐛 Workflow 08 — Debug
## Runs: When something is broken and the cause isn't immediately obvious.
## Next: → `05-task-complete.md` when fixed, OR `06-session-end.md` if unresolved today

---

> Structured debugging prevents spiral debugging — where you change things randomly
> and don't know what fixed it (or broke it more).

---

## Step 1 — Write Down the Bug

Before changing anything, write a one-sentence description of the problem:

> **Bug:** `___________________________________________`
> **Expected:** `___________________________________________`
> **Actual:** `___________________________________________`

Example:
> **Bug:** Scraper Edge Function returns 200 but no `change_events` rows are being inserted.
> **Expected:** When diff > 50 chars, a row should appear in `change_events`.
> **Actual:** Function returns OK but table stays empty. No error in logs.

**If you can't write this clearly → you don't understand the bug yet. Read logs first.**

---

## Step 2 — Gather Evidence

Do not guess. Read the actual error:

```bash
# Edge Function logs
supabase functions logs scraper --tail 50

# Next.js Route Handler errors
# Check browser network tab → look at the failing request's response body

# Supabase DB
# Check Supabase dashboard → Logs → API for RLS rejections

# TypeScript errors
pnpm tsc --noEmit
```

What does the evidence say? Write it down before changing anything.

---

## Step 3 — Identify the Layer

Lensly has 4 distinct layers. Bugs usually live in one:

| Layer | Common Bugs |
|-------|-------------|
| **Edge Function** | Deno import errors, wrong env var names, Supabase service role missing, CORS issues |
| **Next.js Route Handler** | `fs` imported client-side, Zod validation rejecting valid input, Supabase anon key used instead of service role |
| **Database / RLS** | RLS policy blocking write, wrong column name, missing NOT NULL value |
| **Mobile / API client** | Wrong API_BASE_URL, JWT not attached to request, Supabase anon key read blocked by RLS |

Which layer is the bug in?

---

## Step 4 — Lensly-Specific Debug Checklist

Check the most common Lensly mistakes first:

### Edge Function bugs
- [ ] Are imports using Deno ESM style? (`https://esm.sh/...`) not npm style?
- [ ] Is `SUPABASE_SERVICE_ROLE_KEY` set in Edge Function secrets? (`supabase secrets list`)
- [ ] Is the function deployed after the last code change? (`supabase functions deploy [name]`)
- [ ] Is the Supabase client using the service role key — not the anon key?

### Scraper bugs
- [ ] Does robots.txt check happen before the fetch? Or is it being skipped?
- [ ] Is the diff less than `DIFF_MIN_LENGTH` (50 chars)? — would explain no `change_event` insert
- [ ] Is the URL reachable from the Edge Function's server? (not just from your local machine)
- [ ] Is the text extraction stripping too much? Log the extracted text length.

### AI analyser bugs
- [ ] Is `ANTHROPIC_API_KEY` set in Edge Function secrets?
- [ ] Is the diff being truncated to `MAX_DIFF_LENGTH_FOR_AI` before sending?
- [ ] Is the AI response being validated with Zod? Does the actual response match the schema?
- [ ] Is significance coming back as a string ("7") instead of an integer (7)?

### Notification bugs
- [ ] Is `significance` null? (happens when credits = 0 — expected behaviour, not a bug)
- [ ] Is the user's threshold higher than the significance score? (also expected — check settings)
- [ ] Is the Slack webhook URL actually set in `notification_settings`?
- [ ] Is the webhook URL valid? Test it with a manual curl.

### Database bugs
- [ ] Run the failing query in Supabase SQL editor with the service role — does it work there?
- [ ] If it works with service role but not in app → RLS policy is the issue
- [ ] Check column names match exactly — Supabase is case-sensitive

---

## Step 5 — Make One Change at a Time

Identify the most likely cause from Step 4. Change **one thing**. Test. Observe.

Do not:
- Change three things at once
- Delete and rewrite the whole function
- Add `console.log` everywhere simultaneously

Do:
- Add one `console.log` at the exact point of failure
- Change one variable/import/query
- Redeploy the function after the change
- Check if the specific symptom changed

---

## Step 6 — If Stuck After 30 Minutes

If you've checked everything and can't find it:

1. **Simplify.** Strip the function down to the bare minimum — does the simplest version work?
2. **Isolate.** Run each part in isolation — does robots.ts work standalone? Does the DB insert work standalone?
3. **Read the docs.** Supabase Edge Function docs and Deno docs are accurate — if something seems wrong, the docs often explain the correct behaviour.
4. **Mark it blocked.** Add `[!]` to the task in `planning/01-active-sprint.md` with a clear description of what's been tried. Move on.

---

## Step 7 — When Fixed

Write one line in the Activity Log:

```
| YYYY-MM-DD | 🐛 Fixed: [bug description]. Root cause: [what it actually was]. |
```

Example:
```
| 2026-03-17 | 🐛 Fixed: change_events not inserting. Root cause: Deno supabase client was using anon key not service role key — RLS was blocking the insert silently. |
```

If the bug reveals a pattern error → update `rules/coding-standards.md` to prevent it recurring.

---

## ✅ Bug resolved or documented

**→ Fixed: `05-task-complete.md`**
**→ Unresolved today: `06-session-end.md` (mark task `[!]` blocked)**
