# Lensly — Architecture

> Scraping in Edge Functions. Analysis in Route Handlers. AI filters signal from noise. Notifications fan out to email + Slack + in-app.

---

## High-Level Topology

```
Supabase Edge Functions (cron — weekly)
  └── Scraper: fetch URLs → extract text → diff → store change_events
  └── Notifier: evaluate threshold → trigger Slack webhooks
  └── Digester: compile weekly email digest → send via Resend

Mobile App (React Native / Expo)
  └── Reads change feed + competitor list from Supabase directly
  └── All writes go through Lensly Web API

Browser (Next.js Client Components)
  └── Calls Next.js Route Handlers via fetch

Next.js App Router (Web — Vercel)
  ├── Server Components    → direct Supabase reads (feed, competitors)
  ├── Route Handlers       → AI analysis calls, competitor CRUD, settings
  │     └── AIClient       → Vercel AI SDK → Anthropic
  │     └── Supabase Admin → change_events writes, credit management
  └── Middleware           → Supabase Auth session validation

Supabase
  ├── PostgreSQL           → competitors, monitor_configs, snapshots, change_events, credits
  ├── Auth                 → email/password + Google OAuth
  ├── Edge Functions       → scraper cron, notifier, digester
  └── RLS Policies         → users only access their own data

External Services
  ├── Anthropic API        → significance scoring + digest writing
  ├── Resend               → transactional + digest emails
  └── Slack Webhooks       → per-change Slack notifications
```

---

## Web File Structure (`/web`)

```
src/
├── app/
│   ├── (public)/page.tsx                 ← landing page
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (app)/
│   │   ├── dashboard/page.tsx            ← in-app change feed
│   │   ├── competitors/page.tsx          ← competitor list + add
│   │   ├── competitors/[id]/page.tsx     ← competitor detail + monitor config
│   │   └── settings/page.tsx            ← notifications, credits, Slack
│   └── api/
│       ├── competitors/
│       │   ├── route.ts                  ← GET (list), POST (add competitor)
│       │   └── [id]/route.ts             ← GET, PATCH, DELETE
│       ├── monitors/
│       │   └── [competitorId]/route.ts   ← GET, PATCH monitor configs
│       ├── changes/
│       │   ├── route.ts                  ← GET change feed (all competitors)
│       │   └── [id]/analyse/route.ts     ← POST: trigger AI analysis on demand
│       ├── credits/route.ts              ← GET balance, POST webhook
│       ├── settings/
│       │   ├── notifications/route.ts    ← GET, PATCH threshold + digest prefs
│       │   └── slack/route.ts            ← POST webhook URL, POST test
│       └── webhooks/
│           └── payment/route.ts          ← credit purchase webhook
├── components/
│   ├── features/
│   │   ├── feed/
│   │   │   ├── ChangeFeed.tsx            ← chronological list of change events
│   │   │   ├── ChangeCard.tsx            ← competitor + type + summary + score
│   │   │   ├── SignificanceBadge.tsx     ← colour-coded 1–10 score
│   │   │   ├── DiffViewer.tsx            ← raw diff toggle (added/removed lines)
│   │   │   └── FeedFilters.tsx
│   │   ├── competitors/
│   │   │   ├── CompetitorList.tsx
│   │   │   ├── AddCompetitorForm.tsx     ← name + URLs + monitor type toggles
│   │   │   └── MonitorConfig.tsx         ← per-monitor enable/disable
│   │   └── settings/
│   │       ├── ThresholdSlider.tsx       ← significance threshold 1–10
│   │       ├── SlackWebhookForm.tsx
│   │       └── DigestPreferences.tsx
│   └── shared/
├── lib/
│   ├── ai/
│   │   ├── client.ts
│   │   └── prompts/
│   │       ├── analyseChange.ts          ← significance score + summary prompt
│   │       └── compileDigest.ts          ← weekly digest compilation prompt
│   ├── supabase/
│   │   ├── server.ts
│   │   └── browser.ts
│   ├── notifications/
│   │   ├── slack.ts                      ← Slack webhook sender
│   │   └── email.ts                      ← Resend email client
│   └── constants.ts
├── types/
│   ├── competitor.ts                     ← Competitor, MonitorType, MonitorConfig
│   ├── changes.ts                        ← ChangeEvent, Diff, SignificanceScore
│   └── notifications.ts                  ← NotificationSettings, DigestPrefs
└── middleware.ts
```

---

## Edge Functions (`/supabase/functions/`)

```
supabase/functions/
├── scraper/
│   └── index.ts                          ← weekly cron: scrape all active monitors
├── analyser/
│   └── index.ts                          ← triggered by scraper: AI analysis per change
├── notifier/
│   └── index.ts                          ← triggered by analyser: Slack notifications
└── digester/
    └── index.ts                          ← weekly cron: compile + send email digests
```

---

## Mobile File Structure (`/mobile`)

```
app/
├── (public)/index.tsx
├── (auth)/login.tsx + register.tsx
├── (app)/
│   ├── feed.tsx                          ← change feed (read from Supabase directly)
│   ├── competitors.tsx                   ← competitor list
│   ├── competitor/[id].tsx               ← competitor detail
│   └── settings.tsx
lib/
├── api/lensly.ts                         ← typed client → Lensly web API
├── supabase/client.ts                    ← read Supabase: feed + competitors
└── constants.ts
```

---

## Database Schema

```sql
-- Competitors being monitored
CREATE TABLE competitors (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID REFERENCES auth.users NOT NULL,
  name          TEXT NOT NULL,
  website_url   TEXT NOT NULL,
  pricing_url   TEXT,
  changelog_url TEXT,
  twitter_url   TEXT,
  linkedin_url  TEXT,
  careers_url   TEXT,
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT now()
);

-- Per-competitor per-monitor-type configuration
CREATE TABLE monitor_configs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  competitor_id   UUID REFERENCES competitors NOT NULL,
  user_id         UUID REFERENCES auth.users NOT NULL,
  monitor_type    TEXT NOT NULL CHECK (monitor_type IN (
                    'website', 'pricing', 'changelog', 'twitter', 'linkedin', 'careers'
                  )),
  is_enabled      BOOLEAN DEFAULT true,
  last_scraped_at TIMESTAMPTZ,
  last_snapshot   TEXT,                   -- extracted text from last scrape
  robots_blocked  BOOLEAN DEFAULT false,  -- true if robots.txt disallows
  UNIQUE(competitor_id, monitor_type)
);

-- Every detected change
CREATE TABLE change_events (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES auth.users NOT NULL,
  competitor_id   UUID REFERENCES competitors NOT NULL,
  monitor_type    TEXT NOT NULL,
  diff_text       TEXT NOT NULL,          -- text diff only (added/removed lines)
  ai_summary      TEXT,                   -- AI plain-English summary (null until analysed)
  significance    INTEGER,                -- 1–10 (null until analysed)
  significance_reason TEXT,              -- one-line reason from AI
  is_read         BOOLEAN DEFAULT false,
  notified_slack  BOOLEAN DEFAULT false,
  notified_email  BOOLEAN DEFAULT false,
  detected_at     TIMESTAMPTZ DEFAULT now()
);

-- Credit balance + transactions
CREATE TABLE credits (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES auth.users NOT NULL UNIQUE,
  balance     INTEGER NOT NULL DEFAULT 0,
  updated_at  TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE credit_transactions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES auth.users NOT NULL,
  amount      INTEGER NOT NULL,
  reason      TEXT,
  change_id   UUID REFERENCES change_events,
  created_at  TIMESTAMPTZ DEFAULT now()
);

-- Notification settings per user
CREATE TABLE notification_settings (
  user_id             UUID PRIMARY KEY REFERENCES auth.users,
  significance_threshold INTEGER DEFAULT 6 CHECK (significance_threshold BETWEEN 1 AND 10),
  email_digest_enabled  BOOLEAN DEFAULT true,
  email_digest_day      TEXT DEFAULT 'monday',
  slack_webhook_url     TEXT,
  slack_enabled         BOOLEAN DEFAULT false,
  updated_at            TIMESTAMPTZ DEFAULT now()
);
```

### RLS Policies

```sql
-- All tables: users only access their own data
ALTER TABLE competitors ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users manage own competitors"
  ON competitors FOR ALL USING (auth.uid() = user_id);

ALTER TABLE change_events ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users see own change events"
  ON change_events FOR ALL USING (auth.uid() = user_id);

-- Slack webhook URL is user-owned but treated as sensitive
-- Never returned in list queries — only in settings endpoint
```

---

## Core Data Flows

### 1. Weekly Scrape Flow (Edge Function)
```
Cron fires Monday 08:00 UTC
  → scraper/index.ts
  → SELECT all active monitor_configs
  → For each config:
      → fetch robots.txt → check path → skip if disallowed
      → fetch URL with timeout (10s)
      → extract text (strip nav, footer, scripts, styles)
      → diff against monitor_configs.last_snapshot
      → if diff > 50 chars:
          → INSERT change_events { diff_text, detected_at }
          → UPDATE monitor_configs.last_snapshot + last_scraped_at
          → invoke analyser Edge Function (async)
      → else: UPDATE last_scraped_at only
```

### 2. AI Analysis Flow (Edge Function — triggered async)
```
analyser/index.ts receives { change_event_id }
  → SELECT change_event + user credits
  → if credits < 1:
      → UPDATE change_events SET significance = null (no analysis)
      → return (user sees raw diff only)
  → prompts/analyseChange.ts → AIClient (temp 0.1)
  → parse { significance, reason, summary }
  → UPDATE change_events SET ai_summary, significance, significance_reason
  → DEDUCT 1 credit
  → invoke notifier Edge Function
```

### 3. Notification Flow (Edge Function)
```
notifier/index.ts receives { change_event_id }
  → SELECT change_event + notification_settings for user
  → if significance >= threshold AND slack_enabled:
      → POST to slack_webhook_url
      → UPDATE change_events SET notified_slack = true
  → in-app feed updates automatically (client polls or realtime subscription)
```

### 4. Weekly Digest Flow (Edge Function)
```
digester/index.ts — cron: every Monday 07:00 UTC (before scraper)
  → SELECT all change_events from last 7 days per user
  → filter: significance >= user threshold
  → if no significant changes: skip (no empty digests)
  → prompts/compileDigest.ts → AIClient (temp 0.2)
  → Resend.send() → email digest
  → UPDATE change_events SET notified_email = true
```

---

## AI Layer

| Feature | Prompt File | Temp | Notes |
|---------|------------|------|-------|
| Change significance + summary | `prompts/analyseChange.ts` | 0.1 | Factual — score and summarise the diff only |
| Weekly digest compilation | `prompts/compileDigest.ts` | 0.2 | Narrative — synthesise week's changes into readable digest |

---

## Key Architecture Boundaries

| Boundary | Rule |
|----------|------|
| Scraping | Edge Functions only — never in Route Handlers (timeout risk) |
| AI calls | Route Handlers + Edge Functions — never in client components or mobile |
| Mobile writes | Always via Lensly web API |
| Raw HTML | Never stored — extract text immediately, discard HTML |
| Diffs sent to AI | Text diff only — never full page content (cost control) |
| Slack webhook URL | Server-side only — never returned raw in API responses |
| robots.txt | Checked before every scrape — non-negotiable |
| Credits | Deducted after successful AI analysis only |
