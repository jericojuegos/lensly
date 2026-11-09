# Lensly — Product Specification

> **Goal:** Know what your competitors are doing before your customers tell you — automated, AI-filtered, delivered where you work.
>
> **Tagline:** *Watch your competitors. Not your inbox.*

---

## 1) Product Summary

Lensly monitors competitor websites, pricing pages, and public social/job profiles on a weekly schedule. When something changes, AI scores the significance of the change and — if it matters — delivers a plain-English digest to email, Slack, and the in-app feed.

The core problem: founders and PMs already know they should watch competitors. They don't do it because it's tedious — bookmark the site, check it manually, remember what it looked like last week, figure out what changed. Lensly automates all of that. The AI layer filters signal from noise — a typo fix is not worth your attention; a new pricing tier is.

### What Makes Lensly Different

Most competitor monitoring tools alert on every change. Lensly only alerts on **significant** changes. The AI significance score (1–10) is the core filtering mechanism. Users set their own threshold — a founder might want everything ≥5, a busy PM might want only ≥8.

---

## 2) Credit System

Lensly is free to use. AI analysis of detected changes costs credits.

| Pack | Credits | Price |
|------|---------|-------|
| Free forever | 20 credits on signup | $0 |
| Starter | 100 credits | $9 |
| Growth | 500 credits | $29 |
| Agency | 2000 credits | $79 |

**Credit cost:**
- Website / pricing page change analysis: 1 credit per change detected
- Social post analysis: 1 credit per post detected
- Job posting analysis: 1 credit per new posting detected

**Free tier behaviour:** Scraping and diff detection are always free. AI analysis (significance scoring + digest writing) costs credits. Users with 0 credits still see raw diffs — they just don't get AI significance scores or digest summaries.

---

## 3) Monitor Types

| Monitor Type | What Is Scraped | Change Signal |
|-------------|----------------|---------------|
| **Website** | Full page text content | Any text addition/removal above 50 chars |
| **Pricing page** | Pricing page specifically | Price changes, plan additions/removals, feature list changes |
| **Feature announcements** | Changelog / blog / what's new page | New entries detected |
| **Social — Twitter/X** | Public profile latest posts (no API) | New posts in last 7 days |
| **Social — LinkedIn** | Public company page posts | New posts in last 7 days |
| **Job postings** | Careers page or LinkedIn jobs | New job titles detected |

---

## 4) Core Features by Phase

### Phase 1 — MVP

**Competitor Management**
- Add a competitor: name + website URL (required) + optional: pricing URL, changelog URL, Twitter/X URL, LinkedIn URL, careers URL
- Edit / delete competitors
- Enable/disable individual monitor types per competitor
- Free tier: up to 3 competitors. Credits unlock more (5 credits per additional competitor/month)

**Scraping Engine (Supabase Edge Functions — cron)**
- Weekly cron: every Monday 08:00 UTC
- Per competitor per monitor type: fetch URL → extract text → compare to last snapshot → compute diff
- Diff stored in `change_events` table
- Raw HTML discarded after text extraction
- `robots.txt` respected before every scrape
- Retry logic: 3 attempts with exponential backoff on failure

**AI Change Analysis (Route Handler — triggered by cron result)**
- Input: diff text (not full pages)
- AI scores significance 1–10 with one-line reason
- AI writes plain-English summary: "Acme added a new Enterprise tier at $299/month. Their previous top plan was $149/month."
- Temperature 0.1 — factual, not creative
- Only runs if user has credits ≥ 1
- Credits deducted per change analysed

**In-App Feed**
- Chronological feed of all detected changes across all competitors
- Each item: competitor name, monitor type, AI summary, significance score, timestamp
- Filter by: competitor, monitor type, significance level, date range
- Raw diff toggle — power users can see the actual text diff
- Unread indicator per item

**Notification Delivery**
- Email digest: weekly summary of all significant changes (significance ≥ user threshold)
- Slack: per-change notification to configured webhook (significance ≥ threshold)
- In-app: all changes regardless of threshold — feed is always complete
- User configures significance threshold (default: 6/10)
- User configures email digest day/time

**Settings**
- Notification threshold (1–10 slider)
- Email digest preferences (day, time, on/off)
- Slack webhook URL + test button
- Credit balance + purchase link
- `robots.txt` override acknowledgment (for internal/whitelisted URLs)

### Phase 2 — Intelligence Layer
- Historical trend graphs — significance scores over time per competitor
- AI-generated competitive battle cards — "Here's how you compare to [Competitor] right now"
- Manual scan trigger — run a scan now (costs 2 credits, bypasses weekly schedule)
- Change annotation — add your own notes to a change event ("responded with our pricing update")
- Digest customisation — choose which monitor types appear in digest

### Phase 3 — Scale
- Team workspaces — shared competitor lists, shared feed, per-member notification settings
- White-label / agency tier
- Chrome extension — add competitor from any page
- API access — pull change events programmatically
- Screenshot / visual diffing (Playwright-based)

---

## 5) The Significance Score

Every detected change gets an AI significance score from 1–10:

| Score | Meaning | Notify? |
|-------|---------|---------|
| 1–3 | Trivial — typo fix, minor wording, footer update | No (default) |
| 4–5 | Minor — new blog post, small copy change | Digest only |
| 6–7 | Notable — new feature mentioned, pricing page restructure | Digest + Slack |
| 8–9 | Major — new pricing tier, new product launched | Immediate Slack + email |
| 10 | Critical — pivotal change (acquisition, rebrand, free tier added) | Immediate all channels |

The user's threshold setting filters which scores trigger notifications. The in-app feed always shows everything regardless of score.

---

## 6) The Scrape → Analyse → Notify Flow

```
Weekly cron fires (Monday 08:00 UTC)
  → For each active competitor × monitor type:
      → Check robots.txt → skip if disallowed, log reason
      → Fetch URL → extract text (strip nav/footer/boilerplate)
      → Compare to last snapshot → compute diff
      → If diff is substantive (>50 chars changed):
          → Insert change_event (raw diff, timestamp)
          → Trigger AI analysis (if user has credits)
              → Score significance
              → Write plain-English summary
              → Deduct 1 credit
          → Evaluate against user threshold
          → Send Slack notification if score ≥ threshold
  → End of week:
      → Compile all significant changes into email digest
      → Send digest to all users with email enabled
```

---

## 7) robots.txt Policy

Lensly respects `robots.txt` on all domains. Before every scrape:
1. Fetch `[domain]/robots.txt`
2. Parse for `User-agent: *` and `User-agent: Lensly` rules
3. If the target URL path is disallowed → skip scrape, mark monitor as `blocked_by_robots`
4. Notify user: "We couldn't monitor [URL] because this site's robots.txt disallows scraping."
5. User can acknowledge and disable the monitor

This is non-negotiable. Violating `robots.txt` creates legal risk and is ethically wrong.

---

## 8) Success Metrics

| Goal | Target |
|------|--------|
| Signups in first 3 months | X |
| Competitors added per user (avg) | 3+ |
| Free → paid conversion | 15%+ of users who exhaust free credits |
| Weekly active rate | 40%+ (open digest or check feed weekly) |
| False positive rate | <15% of alerts rated "not useful" by users |
| Credit pack repeat purchase | 35%+ |

---

## 9) Known Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Target site blocks scraper | Rotate user-agent strings; respect robots.txt; notify user if blocked |
| AI over-alerts on trivial changes | Significance threshold + user-configurable sensitivity |
| AI under-alerts on important changes | Users can manually adjust threshold; raw diff always visible |
| Scraping JavaScript-rendered pages | Use `playwright` in Edge Function for JS-heavy pages (Phase 1.5) |
| Twitter/X blocks scraping | Public pages only; graceful degradation if blocked; no API dependency |
| Credit cost surprises | Show estimated credits per week in competitor setup; cap analysis at user's balance |
| Email digest goes to spam | SPF/DKIM/DMARC configured; Resend as email provider; plain-text fallback |
| Legal challenge for scraping | robots.txt compliance; rate limiting; no login-gated content; ToS review |
