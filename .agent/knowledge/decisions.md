# 📋 Architecture Decision Records — Lensly

---

## ADR-001: Scraping in Edge Functions — Not Route Handlers
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Where should the scraping logic run — in Next.js Route Handlers or Supabase Edge Functions?

**Decision:** All scraping runs in Supabase Edge Functions on a cron schedule. Route Handlers are never used for scraping.

**Reasoning:** Vercel Route Handlers have a 10-second execution limit on the free/pro plan. Scraping a slow page, extracting text, computing a diff, and triggering AI analysis can easily take 20–60 seconds across multiple competitors. Edge Functions have a 150-second limit and run independently of any HTTP request. They're designed for exactly this kind of background processing.

---

## ADR-002: Diff Only Sent to AI — Never Full Pages
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Should the AI receive the full before/after page content, or just the diff?

**Decision:** Only the text diff is sent to the AI (capped at 4000 chars). Full page snapshots are never sent.

**Reasoning:** A typical webpage is 5,000–50,000 words of text. Sending two full pages to the AI for comparison would cost 10–50x more in tokens than sending just the diff. The diff is also more useful — it shows exactly what changed, not the entire context. The AI's job is to score the significance of the change and summarise it, not to re-read the whole page.

---

## ADR-003: robots.txt Compliance Is Non-Negotiable
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Should Lensly respect robots.txt, or scrape regardless since it's for monitoring?

**Decision:** Lensly always checks and respects robots.txt before every scrape. Disallowed URLs are never scraped. Users are notified when a URL is blocked.

**Reasoning:** Violating robots.txt creates legal exposure (Computer Fraud and Abuse Act, GDPR in some interpretations) and is ethically wrong. It also risks IP bans that would break monitoring for all users on shared infrastructure. The correct approach is to inform the user and let them decide — perhaps the competitor has a public changelog RSS feed we can use instead.

---

## ADR-004: Significance Score as the Notification Gatekeeper
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Should every detected change trigger a notification, or should there be filtering?

**Decision:** AI scores each change 1–10. Only changes at or above the user's configured threshold trigger Slack and email notifications. The in-app feed shows everything.

**Reasoning:** A competitor's website changes dozens of times per week — cookie banners, A/B tests, minor copy tweaks, footer links. Alerting on every change is notification spam that trains users to ignore Lensly. The significance score is the core product value — it answers "does this matter?" so the user doesn't have to.

---

## ADR-005: Raw HTML Discarded Immediately After Text Extraction
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Should raw scraped HTML be stored for later reprocessing?

**Decision:** Raw HTML is never stored. Text is extracted immediately after fetch, and only the extracted text snapshot is stored in `monitor_configs.last_snapshot`.

**Reasoning:** Raw HTML of a typical page is 100KB–2MB. Storing it for hundreds of monitors × weekly snapshots would create massive storage costs with almost no benefit — we only need the text to compute diffs. If reprocessing is ever needed, we can re-scrape. Text snapshots are 5–20KB and contain everything needed for diffing.

---

## ADR-006: No Twitter/X API — Public Page Scraping Only
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Twitter/X has a public API for reading tweets. Should Lensly use it?

**Decision:** No Twitter/X API in V1. Lensly scrapes public Twitter/X profile pages directly.

**Reasoning:** Twitter/X API access now costs $100+/month at the Basic tier, with severe rate limits. For a free-tier product launching to indie hackers, this cost is not justifiable. Public page scraping gives us the same data (recent posts) at zero API cost. The scraping approach may break if Twitter/X changes their page structure — but that's an acceptable V1 risk. API integration can be added in Phase 2 if scraping proves unreliable.

---

## ADR-007: Resend for Email — Not SendGrid or Mailgun
**Date:** 2026-03-11 | **Status:** Accepted

**Context:** Which email provider for digest delivery?

**Decision:** Resend.

**Reasoning:** Resend has a generous free tier (3,000 emails/month), excellent deliverability, a simple API, React Email integration for templating, and is developer-first. For a bootstrapped product sending weekly digests, it's the obvious choice. SendGrid and Mailgun are enterprise-oriented with more complex setup and worse free tiers.
