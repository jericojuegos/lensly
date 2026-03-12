# 🧠 Lensly — Agent Brain

**Project:** Lensly — AI Competitor Intelligence Monitor
**Web:** Next.js 14+ App Router · TypeScript · Supabase · Vercel AI SDK
**Mobile:** React Native · Expo · TypeScript · Supabase
**Background Jobs:** Supabase Edge Functions (cron) · pg_cron

---

## Navigation

| Question | File |
|----------|------|
| What are we building? | `knowledge/spec.md` |
| How is it structured? | `knowledge/architecture.md` |
| What tools do we use? | `knowledge/tech-stack.md` |
| Why was X built this way? | `knowledge/decisions.md` |
| What are we doing right now? | `planning/01-active-sprint.md` |
| What's the full plan? | `planning/00-roadmap.md` |
| What are the coding rules? | `rules/coding-standards.md` |
| What are the structural rules? | `rules/architecture-rules.md` |
| How do I assess a risky change? | `workflows/change-impact.md` |

---

## Development Workflows (Run These — Don't Think)

| When | Workflow |
|------|---------|
| Brand new project, zero code | `workflows/01-init.md` |
| First time writing code | `workflows/02-scaffold.md` |
| Starting any work session | `workflows/03-session-start.md` |
| During a session, executing a task | `workflows/04-task-loop.md` |
| Just finished a task | `workflows/05-task-complete.md` |
| Ending a session | `workflows/06-session-end.md` |
| All sprint tasks are `[x]` | `workflows/07-phase-complete.md` |
| Something is broken | `workflows/08-debug.md` |

---

## Agent Startup Sequence

Every session, in this exact order — steps 1–4 are read-only:

1. `planning/01-active-sprint.md` — what is the current task?
2. `rules/` — what are the constraints?
3. `knowledge/` — what does the system look like?
4. `workflows/` — does a SOP apply to this task?
5. Execute.

---

## ⚠️ Critical Constraints — Never Violate

- **All AI calls are server-side only.** No AI SDK imports in Client Components, React Native components, or any client-side code. Web: Route Handlers only. Mobile: calls Lensly's own API endpoints.
- **All scraping happens in Supabase Edge Functions — never in Route Handlers.** Route Handlers are synchronous and will timeout on slow pages. Edge Functions run as background jobs on a cron schedule.
- **Raw scraped HTML is never stored long-term.** Store only the extracted text snapshot and the diff. Raw HTML is discarded after processing.
- **Diff is computed server-side before AI analysis.** Never send two full page snapshots to the AI. Send only the diff — reduces token cost by 90%+.
- **AI significance scoring is the gatekeeper for notifications.** Not every change triggers an alert. The AI scores each change (1–10) and only changes above the user's threshold trigger notifications.
- **`SUPABASE_SERVICE_ROLE_KEY` is server-only.** Never in `NEXT_PUBLIC_` vars or mobile app.
- **Slack tokens and email API keys are server-side only.** Never sent to client or mobile.
- **Mobile never calls Supabase directly for mutations.** Reads competitor feed + change history from Supabase directly. All writes go through Lensly's web API.
- **All prompts in `src/lib/ai/prompts/`.** Never inline prompt strings in Route Handlers or Edge Functions.
- **Respect `robots.txt` on all scraped domains.** Check and honour `robots.txt` before scraping any URL. Log disallowed URLs and notify the user instead of scraping.

---

## Anti-Hallucination List

Do NOT build in MVP:

- No real-time scraping on demand (scraping is scheduled cron only)
- No screenshot / visual diffing (text diff only in V1)
- No API monitoring (website scraping only in V1)
- No white-label / agency tier (Phase 3)
- No team workspaces with shared competitor lists (Phase 3)
- No historical trend graphs (Phase 2)
- No AI-generated competitive battle cards (Phase 2)
- No Chrome extension (Phase 3)
- No Twitter/X API (use public page scraping only — API costs too much for V1)
