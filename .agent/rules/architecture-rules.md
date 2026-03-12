---
trigger: always_on
---

# Lensly — Architecture Rules

---

## 1. Scraping Is Edge Functions Only
No scraping in Route Handlers. Vercel timeout limit makes this unreliable. All URL fetching, text extraction, and diffing happens in Supabase Edge Functions.

## 2. Diff Only Goes to AI — Never Full Pages
`MAX_DIFF_LENGTH_FOR_AI` is a hard cap. Truncate before sending. The AI scores significance of changes — it doesn't need full page context.

## 3. robots.txt Is Checked Before Every Scrape
No exceptions. Not for internal URLs, not for "trusted" domains. Check every time. Log and notify on block.

## 4. Raw HTML Is Never Stored
Extract text immediately after fetch. Discard HTML. Store only extracted text in `monitor_configs.last_snapshot`.

## 5. Credits Deducted After Successful Analysis
Scraping: free. Diffing: free. AI analysis: 1 credit, deducted after success only. Never on error or empty diff.

## 6. Slack Webhook URL Is Server-Side Only
Never returned in any API response to the client. Read from `notification_settings` in Route Handlers, never exposed.

## 7. Mobile Reads Supabase, Writes via API
Mobile reads `change_events` and `competitors` directly from Supabase. All writes (add competitor, update settings) go through Lensly web API with JWT auth.

## 8. The Change Impact Rule

| Change | Risk | Reason |
|--------|------|--------|
| robots.txt check logic | 🔴 Critical | Legal + ethical — never bypass |
| Credit deduction timing | 🔴 Critical | User trust + revenue |
| Significance threshold defaults | 🟡 Medium | Affects all existing users' notifications |
| Diff algorithm | 🟡 Medium | Test with 20 real before/after pairs |
| Cron schedule | 🟡 Medium | Changing affects all user expectations |
| AI analysis prompt | 🟡 Medium | Test with 20 real diffs before deploying |
| `DIFF_MIN_LENGTH` constant | 🟡 Medium | Too low = noise; too high = missed changes |
