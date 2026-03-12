---
description: During a session. Repeat for each task until session ends.
---

# ⚙️ Workflow 04 — Task Loop
## Runs: During a session. Repeat for each task until session ends.
## Next: → `05-task-complete.md` when task is done
## Or: → `06-session-end.md` if stopping mid-task

---

> This is the execution engine. One task at a time, start to finish.

---

## The Loop

```
Pick task (from 03-session-start.md)
  → Build it
  → Self-review it  
  → If passes → 05-task-complete.md
  → If more tasks + time remains → repeat from "Build it" with next task
  → If stopping → 06-session-end.md
```

---

## Phase 1: Build

### Before writing any code, answer these:

1. **What file(s) am I creating or editing?**
   - Write the exact paths before touching anything

2. **Is this server-side or client-side?**
   - Server-side (Edge Function or Route Handler): add `import 'server-only'` if it uses Node/Deno APIs
   - Client-side: confirm it needs no `fs`, `child_process`, or Supabase service role

3. **Does this task touch any of Lensly's critical boundaries?**
   - Scraping → Edge Function only, never Route Handler
   - AI call → check credit balance first, send diff only, deduct after success
   - robots.txt → check before fetch, always
   - Notification → check significance threshold before sending

**If you can't answer all three → reread the relevant brain section before writing.**

---

## Phase 2: Lensly-Specific Build Patterns

Use these patterns. Do not invent alternatives.

### Edge Function structure (Deno)
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );
  // business logic
  return new Response(JSON.stringify({ ok: true }), {
    headers: { 'Content-Type': 'application/json' }
  });
});
```

### robots.txt check (always before fetch)
```typescript
const allowed = await checkRobotsTxt(targetUrl, 'Lensly');
if (!allowed) {
  await markMonitorBlocked(monitorConfigId, supabase);
  return; // stop — do not fetch
}
```

### AI analysis (diff only, credit guard)
```typescript
const { balance } = await getCreditBalance(userId, supabase);
if (balance < CREDIT_COST_PER_ANALYSIS) {
  // no credit — store raw diff only, no AI analysis
  return;
}
const truncatedDiff = diff.slice(0, MAX_DIFF_LENGTH_FOR_AI);
const analysis = await analyseChange(truncatedDiff); // AI call
await deductCredit(userId, changeEventId, supabase);  // deduct AFTER success
```

### Route Handler (Next.js)
```typescript
import 'server-only';
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const InputSchema = z.object({ /* ... */ });

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = InputSchema.safeParse(body);
  if (!parsed.success) return NextResponse.json({ error: 'invalid input' }, { status: 400 });
  // business logic
}
```

---

## Phase 3: Self-Review

Before marking a task done, run through this checklist. Be honest — this is for your own quality, not for show.

### Architecture
- [ ] No scraping logic in a Route Handler
- [ ] No AI calls in a Client Component or mobile code
- [ ] No full page content sent to AI — diff only, truncated
- [ ] robots.txt checked before every URL fetch
- [ ] Credits deducted after successful analysis — not before
- [ ] Raw HTML not stored anywhere
- [ ] Slack webhook URL not in any API response

### Code Quality
- [ ] No `any` TypeScript types
- [ ] No `console.log` statements
- [ ] Input validated with Zod in every Route Handler
- [ ] Deno-style ESM imports in Edge Functions (`https://esm.sh/...`)
- [ ] Node-style imports in Next.js files (npm package names)
- [ ] All prompts in `src/lib/ai/prompts/` — not inline

### Functionality
- [ ] The happy path works
- [ ] What happens when the target URL is unreachable? (scraper timeout)
- [ ] What happens when the AI call fails? (analyser error)
- [ ] What happens when the user has 0 credits? (correct fallback — raw diff shown)
- [ ] What happens when robots.txt blocks the URL? (user notified, monitor marked blocked)

### Edge Cases Specific to This Task
- [ ] *(add task-specific edge cases here before building)*

---

## Phase 4: Decision Log

If you made any decision during the build that wasn't already in the brain — document it.

Ask: *"Did I choose between two approaches?"*

If yes → add an ADR to `knowledge/decisions.md`:
- What was the context?
- What did you decide?
- Why?

Small decisions (variable names, minor patterns) don't need ADRs.
Architectural decisions (which library, which approach, which data shape) always do.

---

## ✅ Task Built and Self-Reviewed

**→ Next: `05-task-complete.md`**
