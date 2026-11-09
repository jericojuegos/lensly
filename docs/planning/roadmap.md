# 🗺️ Lensly — Roadmap

> **Vision:** Know what your competitors are doing before your customers tell you.
> **North Star:** Every significant competitor change lands in your feed, Slack, and email — filtered by AI so only what matters gets through.

## 📌 Legend
- `[ ]` Pending · `[>]` Current Focus · `[x]` Completed · `[-]` Deferred · `[!]` Blocked

---

## [>] Phase 0: Foundation
*Goal: Web + mobile scaffold, Supabase schema, Edge Functions wired up, no scraping logic yet.*

### Web Setup
- [ ] `pnpm create next-app web --typescript --tailwind --app`
- [ ] Install: `@supabase/ssr`, `ai`, `@anthropic-ai/sdk`, `zod`, `resend`
- [ ] Supabase project + env vars
- [ ] Supabase Auth: email/password + Google OAuth
- [ ] DB tables: `competitors`, `monitor_configs`, `change_events`, `credits`, `credit_transactions`, `notification_settings`
- [ ] RLS policies on all tables
- [ ] `lib/ai/client.ts` + Anthropic adapter
- [ ] `lib/constants.ts` — all constants defined
- [ ] All Route Handlers scaffolded (501)
- [ ] All pages scaffolded (placeholder UI)

### Mobile Setup
- [ ] `npx create-expo-app mobile --template blank-typescript`
- [ ] Install: `@supabase/supabase-js`, `nativewind`, `zod`
- [ ] `lib/supabase/client.ts` — read feed + competitors
- [ ] `lib/api/lensly.ts` — typed client → web API
- [ ] Expo Router navigation scaffold
- [ ] Login + register screens

### Edge Functions Scaffold
- [ ] `supabase/functions/scraper/index.ts` — stub (returns "not implemented")
- [ ] `supabase/functions/analyser/index.ts` — stub
- [ ] `supabase/functions/notifier/index.ts` — stub
- [ ] `supabase/functions/digester/index.ts` — stub
- [ ] Cron schedule registered for scraper + digester
- [ ] Verify Edge Functions deploy + cron fires via Supabase dashboard

---

## Phase 1: MVP — The Full Intelligence Loop

### Scraper (Edge Function)
- [ ] `lib/robots.ts` — parse + check `robots.txt` per URL
- [ ] `lib/scraper.ts` — fetch URL with timeout + retry logic
- [ ] `lib/extractor.ts` — strip HTML → extract clean text (remove nav, footer, scripts)
- [ ] `lib/differ.ts` — compute text diff (added/removed lines)
- [ ] Full scraper flow: robots check → fetch → extract → diff → store change_event
- [ ] UPDATE `monitor_configs.last_snapshot` + `last_scraped_at` after each scrape
- [ ] Mark `robots_blocked = true` + notify user on disallowed URLs

### AI Analyser (Edge Function)
- [ ] `prompts/analyseChange.ts` — significance score + summary prompt
- [ ] Full analyser flow: fetch change_event → credit check → AI analyse → deduct → update
- [ ] Zod validation on AI output (significance must be integer 1–10)

### Notifier (Edge Function)
- [ ] `lib/slack.ts` — send to Slack webhook with formatted message
- [ ] Full notifier flow: check significance >= threshold → send Slack → mark notified

### Digester (Edge Function)
- [ ] `prompts/compileDigest.ts` — weekly digest narrative prompt
- [ ] `lib/email.ts` — Resend client + digest email template (React Email)
- [ ] Full digester flow: aggregate week's changes → skip if none → compile → send
- [ ] Skip empty digests — never send "nothing changed this week"

### Web UI
- [ ] Competitor CRUD — add, edit, delete, enable/disable
- [ ] Monitor type toggles per competitor
- [ ] `ChangeFeed.tsx` — in-app feed with `SignificanceBadge.tsx` + `DiffViewer.tsx`
- [ ] `FeedFilters.tsx` — filter by competitor, type, significance, date
- [ ] `ThresholdSlider.tsx` + Slack webhook form in settings
- [ ] Credit balance display + low credit warning
- [ ] Mark change as read

### Mobile UI
- [ ] Feed screen — reads `change_events` directly from Supabase
- [ ] Competitor list screen
- [ ] Settings screen — threshold + Slack (writes via web API)

---

## Phase 2: Intelligence Layer
- [ ] Historical trend graphs — significance over time per competitor
- [ ] Manual scan trigger — run now (costs 2 credits)
- [ ] AI-generated competitive battle cards
- [ ] Change annotation — add notes to a change event
- [ ] Digest customisation — choose monitor types per digest

---

## Phase 3: Scale
- [ ] Team workspaces — shared competitor list + feed
- [ ] White-label / agency tier
- [ ] Chrome extension — add competitor from any page
- [ ] API access — pull change events programmatically
- [ ] Screenshot / visual diffing (Playwright)

---

## 🧊 Icebox
- RSS feed monitoring (alternative to scraping for changelog detection)
- Wayback Machine fallback for historical snapshots
- Competitor mention monitoring (mentions of competitor name on HN, Reddit)
