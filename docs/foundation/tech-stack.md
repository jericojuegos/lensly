# Lensly — Tech Stack

---

## Stack Overview

| Layer | Web (Next.js) | Mobile (Expo) | Edge Functions |
|-------|--------------|--------------|---------------|
| **Framework** | Next.js 14+ (App Router) | React Native + Expo SDK 51+ | Supabase Edge Functions (Deno) |
| **Language** | TypeScript (strict) | TypeScript (strict) | TypeScript (Deno) |
| **Styling** | Tailwind CSS | NativeWind | — |
| **Package manager** | pnpm | pnpm | — (Deno imports) |
| **Database + Auth** | Supabase | Supabase (read) + Lensly API | Supabase (service role) |
| **AI Orchestration** | Vercel AI SDK | — | Anthropic SDK (Deno) |
| **AI Provider** | Anthropic Claude | — | Anthropic Claude |
| **Email** | Resend (`resend` npm) | — | Resend (Deno fetch) |
| **Validation** | Zod | Zod | Zod |
| **Deployment** | Vercel | Expo EAS | Supabase (auto) |

---

## Key Constants (`src/lib/constants.ts`)

```typescript
export const FREE_CREDITS_ON_SIGNUP = 20;
export const CREDIT_COST_PER_ANALYSIS = 1;
export const LOW_CREDIT_THRESHOLD = 5;
export const FREE_TIER_MAX_COMPETITORS = 3;
export const DIFF_MIN_LENGTH = 50;              // chars — ignore smaller diffs
export const SCRAPE_TIMEOUT_MS = 10_000;        // 10s per URL
export const SCRAPE_RETRY_ATTEMPTS = 3;
export const SIGNIFICANCE_DEFAULT_THRESHOLD = 6;
export const SIGNIFICANCE_MIN = 1;
export const SIGNIFICANCE_MAX = 10;
export const MONITOR_TYPES = [
  'website', 'pricing', 'changelog', 'twitter', 'linkedin', 'careers'
] as const;
export const CRON_SCRAPER   = '0 8 * * 1';     // Monday 08:00 UTC
export const CRON_DIGESTER  = '0 7 * * 1';     // Monday 07:00 UTC (before scraper)
export const MAX_DIFF_LENGTH_FOR_AI = 4000;     // chars — truncate before sending to AI
export const DIGEST_MAX_CHANGES = 20;           // max changes per digest
```

---

## Deno-Specific Edge Function Imports

Edge Functions run on Deno — not Node.js. Import patterns are different:

```typescript
// ✅ Correct Deno imports in Edge Functions
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';
import Anthropic from 'https://esm.sh/@anthropic-ai/sdk';

// ❌ Wrong — Node.js npm imports don't work in Deno Edge Functions
import { createClient } from '@supabase/supabase-js';
```

---

## Prohibited List

| Prohibited | Reason | Use Instead |
|-----------|--------|-------------|
| Scraping in Route Handlers | Timeout risk (Vercel 10s limit) | Supabase Edge Functions |
| AI calls in Client Components | Security | Route Handlers + Edge Functions |
| Storing raw HTML | Storage bloat | Extract text, discard HTML immediately |
| Sending full page to AI | Token cost | Send diff only (max 4000 chars) |
| Skipping robots.txt check | Legal + ethical risk | Always check before scraping |
| Slack webhook URL in client response | Security | Server-side only, never returned raw |
| Polling Supabase from mobile for writes | Bypasses business logic | Lensly web API for all writes |
| `any` TypeScript | Type safety | `unknown` + Zod |
| `console.log` in production | Noise | Remove before commit |
| Empty digest emails | Spam signal | Skip digest if no significant changes |
