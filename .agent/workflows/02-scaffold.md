---
description: Sets up both codebases (web and mobile) and Edge Functions from zero.
---

# 🏗️ Workflow 02 — Scaffold
## Runs: ONCE. After `01-init.md`. Sets up both codebases + Edge Functions from zero.
## Next: → `03-session-start.md` (for all future sessions after this)

---

> This is the longest one-time workflow. Work through it in order.
> Each section can be a separate agent session — use `06-session-end.md` between sessions.

---

## Part A — Web (Next.js)

### A1. Create the project
```bash
pnpm create next-app web --typescript --tailwind --app --no-src-dir
cd web
```

### A2. Install dependencies
```bash
pnpm add @supabase/ssr @supabase/supabase-js
pnpm add ai @anthropic-ai/sdk
pnpm add zod
pnpm add resend
pnpm add server-only
```

### A3. Scaffold folder structure
Create these empty directories (add `.gitkeep` if needed):
```
src/app/(public)/
src/app/(auth)/login/
src/app/(auth)/register/
src/app/(app)/dashboard/
src/app/(app)/competitors/
src/app/(app)/settings/
src/app/api/competitors/
src/app/api/monitors/
src/app/api/changes/
src/app/api/credits/
src/app/api/settings/notifications/
src/app/api/settings/slack/
src/app/api/webhooks/payment/
src/components/features/feed/
src/components/features/competitors/
src/components/features/settings/
src/components/shared/
src/lib/ai/prompts/
src/lib/supabase/
src/lib/notifications/
src/types/
```

### A4. Create `src/lib/constants.ts`
Copy all constants from `knowledge/tech-stack.md` "Key Constants" section exactly.

### A5. Create Supabase clients
- `src/lib/supabase/server.ts` — SSR server client
- `src/lib/supabase/browser.ts` — browser client

### A6. Create AI client stub
- `src/lib/ai/client.ts` — Anthropic client factory (stub, no logic yet)

### A7. Create type files
- `src/types/competitor.ts` — `Competitor`, `MonitorType`, `MonitorConfig`
- `src/types/changes.ts` — `ChangeEvent`, `Diff`, `SignificanceScore`
- `src/types/notifications.ts` — `NotificationSettings`, `DigestPrefs`

### A8. Stub all Route Handlers
Each file returns `Response.json({ error: 'not implemented' }, { status: 501 })`:
- `src/app/api/competitors/route.ts`
- `src/app/api/competitors/[id]/route.ts`
- `src/app/api/monitors/[competitorId]/route.ts`
- `src/app/api/changes/route.ts`
- `src/app/api/changes/[id]/analyse/route.ts`
- `src/app/api/credits/route.ts`
- `src/app/api/settings/notifications/route.ts`
- `src/app/api/settings/slack/route.ts`
- `src/app/api/webhooks/payment/route.ts`

### A9. Scaffold pages (placeholder UI)
Each page renders a single `<h1>` with the page name — no real UI yet:
- `src/app/(public)/page.tsx` — landing
- `src/app/(auth)/login/page.tsx`
- `src/app/(auth)/register/page.tsx`
- `src/app/(app)/dashboard/page.tsx`
- `src/app/(app)/competitors/page.tsx`
- `src/app/(app)/competitors/[id]/page.tsx`
- `src/app/(app)/settings/page.tsx`

### A10. Add middleware
`src/middleware.ts` — Supabase Auth session refresh on every request.

### A11. Verify web scaffold
```bash
pnpm dev
```
- [ ] `localhost:3000` loads without errors
- [ ] All `/api/*` routes return 501 (not 404 or 500)

---

## Part B — Supabase Database

### B1. Create all 6 tables
Run the SQL from `knowledge/architecture.md` "Database Schema" section in Supabase SQL editor — in this order:
1. `competitors`
2. `monitor_configs`
3. `change_events`
4. `credits`
5. `credit_transactions`
6. `notification_settings`

### B2. Apply RLS policies
Run all RLS policies from the same section. Verify in Supabase dashboard that each table shows "RLS enabled".

### B3. Create Supabase Storage bucket
- Bucket name: `receipts` — wait, wrong project. Lensly has no storage bucket.
- ✅ No storage bucket needed for Lensly — skip this step.

### B4. Enable Auth providers
In Supabase dashboard → Authentication → Providers:
- [ ] Email provider: enabled
- [ ] Google provider: enabled (paste Google OAuth client ID + secret)

### B5. Create signup trigger for free credits
In Supabase SQL editor:
```sql
-- Auto-insert credit row on new user signup
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.credits (user_id, balance)
  VALUES (new.id, 20); -- FREE_CREDITS_ON_SIGNUP = 20
  INSERT INTO public.notification_settings (user_id)
  VALUES (new.id);
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE PROCEDURE handle_new_user();
```

### B6. Verify database
- [ ] All 6 tables visible in Supabase table editor
- [ ] RLS enabled on all 6 tables
- [ ] Test signup trigger: create a test user → verify `credits` row created with balance 20

---

## Part C — Mobile (React Native / Expo)

### C1. Create the project
```bash
# From lensly root (not inside /web)
npx create-expo-app mobile --template blank-typescript
cd mobile
```

### C2. Install dependencies
```bash
npx expo install @supabase/supabase-js
pnpm add nativewind zod
npx expo install expo-router expo-linking expo-constants expo-status-bar
```

### C3. Scaffold structure
```
app/
  (public)/index.tsx
  (auth)/login.tsx
  (auth)/register.tsx
  (app)/feed.tsx
  (app)/competitors.tsx
  (app)/competitor/[id].tsx
  (app)/settings.tsx
lib/
  api/lensly.ts        ← typed fetch client → Lensly web API
  supabase/client.ts   ← Supabase read-only client
  constants.ts         ← API_BASE_URL + other constants
```

### C4. Create constants
`lib/constants.ts`:
```typescript
export const API_BASE_URL = __DEV__
  ? 'http://localhost:3000'
  : 'https://your-lensly-domain.com'; // update before production
```

### C5. Scaffold screens (placeholder)
Each screen renders `<Text>Screen name</Text>` — no real UI yet.

### C6. Verify mobile scaffold
```bash
npx expo start
```
- [ ] Expo Go app loads on simulator or device without errors
- [ ] All screens navigable

---

## Part D — Edge Functions

### D1. Initialise Supabase locally
```bash
# From lensly root
supabase init
supabase link --project-ref YOUR_PROJECT_REF
```

### D2. Create all 4 Edge Function stubs

`supabase/functions/scraper/index.ts`:
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';

serve(async (_req) => {
  return new Response(
    JSON.stringify({ message: 'scraper not implemented' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
});
```

Repeat for: `analyser/index.ts`, `notifier/index.ts`, `digester/index.ts` — same stub pattern.

### D3. Register cron schedules
In `supabase/config.toml` or via Supabase dashboard → Edge Functions → Schedule:
```
scraper:  0 8 * * 1   (Monday 08:00 UTC)
digester: 0 7 * * 1   (Monday 07:00 UTC — runs before scraper)
```

### D4. Deploy stubs
```bash
supabase functions deploy scraper
supabase functions deploy analyser
supabase functions deploy notifier
supabase functions deploy digester
```

### D5. Verify Edge Functions
- [ ] All 4 functions visible in Supabase dashboard → Edge Functions
- [ ] Cron schedules registered for scraper + digester
- [ ] Test invoke: `supabase functions invoke scraper` → returns stub response

---

## Part E — Environment Variables

### E1. Web `.env.local`
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
ANTHROPIC_API_KEY=
RESEND_API_KEY=
STRIPE_WEBHOOK_SECRET=
```

### E2. Edge Function secrets
```bash
supabase secrets set ANTHROPIC_API_KEY=your_key
supabase secrets set RESEND_API_KEY=your_key
supabase secrets set SUPABASE_SERVICE_ROLE_KEY=your_key
```

### E3. Mobile `.env`
```
EXPO_PUBLIC_SUPABASE_URL=
EXPO_PUBLIC_SUPABASE_ANON_KEY=
EXPO_PUBLIC_API_BASE_URL=http://localhost:3000
```

### E4. Add to `.gitignore`
Confirm `.env.local`, `.env`, and any secrets files are in `.gitignore`.

---

## Part F — Final Scaffold Verification

- [ ] `pnpm dev` in `/web` — loads, no errors, all `/api/*` return 501
- [ ] `npx expo start` in `/mobile` — loads, all screens navigable
- [ ] `supabase functions invoke scraper` — returns stub JSON
- [ ] Supabase dashboard: 6 tables, RLS on all, signup trigger active
- [ ] Test user signup → `credits` row created with balance 20

---

## Update Brain

In `planning/01-active-sprint.md`, mark these tasks complete `[x]`:
- `pnpm create next-app web --typescript --tailwind --app`
- `npx create-expo-app mobile --template blank-typescript`
- All 4 Edge Function stubs created
- Cron schedule registered
- All 6 DB tables + RLS policies
- All Route Handlers scaffolded (501)
- All pages scaffolded (placeholder UI)

Append to Activity Log:
```
| YYYY-MM-DD | Scaffold complete. Web + mobile + Edge Functions running. DB schema deployed. All stubs verified. |
```

---

## ✅ Done

**The full foundation is running. Zero real features exist yet — that's correct.**
**→ Next: `03-session-start.md` for every session from here on.**
