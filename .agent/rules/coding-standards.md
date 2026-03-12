---
trigger: always_on
---

# Lensly — Coding Standards

---

## Style Rules

| Rule | Value |
|------|-------|
| Semicolons | Required |
| Quotes | Single quotes |
| Trailing commas | Always |
| Indent | 2 spaces |
| `any` type | Forbidden |
| `console.log` | Not in committed code |
| Strict mode | `"strict": true` in tsconfig |

---

## robots.txt Check — Always Before Scraping

```typescript
// ✅ Correct — check robots.txt before every fetch
const allowed = await checkRobotsTxt(url, 'Lensly');
if (!allowed) {
  await markMonitorBlocked(monitorConfigId);
  await notifyUserRobotsBlocked(userId, url);
  return; // stop here — never scrape disallowed URLs
}
const html = await fetchWithTimeout(url, SCRAPE_TIMEOUT_MS);

// ❌ Wrong — scraping without robots.txt check
const html = await fetchWithTimeout(url, SCRAPE_TIMEOUT_MS);
```

---

## Diff Before AI — Never Full Pages

```typescript
// ✅ Correct — send only the diff, capped at max length
const diff = computeDiff(previousSnapshot, currentText);
if (diff.length < DIFF_MIN_LENGTH) return; // too small to matter
const truncatedDiff = diff.slice(0, MAX_DIFF_LENGTH_FOR_AI);
const analysis = await analyseChange(truncatedDiff);

// ❌ Wrong — sending full pages
const analysis = await analyseChange(previousSnapshot, currentText);
```

---

## Credit Deduction — After Successful Analysis Only

```typescript
// ✅ Correct
const analysis = await aiClient.analyse(diff);
if (!analysis) return error; // no deduction on failure
await deductCredit(userId, changeEventId);
await updateChangeEvent(id, analysis);

// ❌ Wrong
await deductCredit(userId, changeEventId);
const analysis = await aiClient.analyse(diff); // could fail after deduction
```

---

## Edge Function Imports — Deno Style

```typescript
// ✅ Correct — Deno ESM imports
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';
import Anthropic from 'https://esm.sh/@anthropic-ai/sdk';

// ❌ Wrong — Node.js npm imports
import { createClient } from '@supabase/supabase-js';
```

---

## Empty Digest Guard

```typescript
// ✅ Correct — skip digest if nothing significant
const significantChanges = changes.filter(c => c.significance >= threshold);
if (significantChanges.length === 0) return; // no email sent

// ❌ Wrong — sending empty digest
await sendDigest(userId, []); // spam signal, wastes email reputation
```

---

## Pre-Commit Checklist

- [ ] No scraping logic in Route Handlers — Edge Functions only
- [ ] robots.txt checked before every URL fetch
- [ ] Only diff sent to AI — not full pages
- [ ] Diff truncated to `MAX_DIFF_LENGTH_FOR_AI` before AI call
- [ ] Credits deducted after successful analysis only
- [ ] Raw HTML not stored anywhere — extract text immediately
- [ ] Empty digests skipped — never send empty emails
- [ ] Slack webhook URL never returned raw in API responses
- [ ] All new tables have RLS policies
- [ ] Deno-style imports in all Edge Functions
- [ ] No `any` types
- [ ] No `console.log`
- [ ] All prompts in `src/lib/ai/prompts/`
