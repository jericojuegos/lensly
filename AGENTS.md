# 🧠 Lensly — Agent Brain (`AGENTS.md`)

**Project:** Lensly — AI Competitor Intelligence Monitor
**Context:** Personal / Standalone
**Web Stack:** Next.js 14+ App Router · TypeScript · Supabase · Vercel AI SDK (Anthropic Claude)
**Mobile Stack:** React Native · Expo · TypeScript · Supabase
**Background Jobs:** Supabase Edge Functions (Deno) · pg_cron
**Architecture Pattern:** Pragmatic Feature-Driven Architecture (FDA)
**Master Brain:** `nextjs-supabase`
**Last synced:** `nextjs-supabase-v1.0` — 2026-11-09

---

## 🚦 Project State

**Stage:** `initialized`
**Stage last updated:** `2026-11-09`

| Stage | Meaning | Next Action / Skill |
|-------|---------|---------------------|
| `initialized` | Brain & specs ready, Phase 0 not built yet | Run **`/next`** or **`/spec`** / **`/build`** on `docs/planning/roadmap.md` |
| `scaffolded` | Base codebase & DB deployed | Run **`/spec`** or **`/build`** on Wave 1 features in `docs/planning/roadmap.md` |
| `active` | Feature wave development in progress | **`/spec`** → **`/build`** → **`/verify`** → **`/ship`** |

---

## ⚡ Deterministic Verification Commands

Run these commands after every task before marking `[x]`:
- **Typecheck (Every task):** `pnpm tsc --noEmit`
- **Full Build (Feature completion / `/verify`):** `pnpm build`
- **Lint:** `pnpm lint`

---

## 🗺️ Context Map (`docs/INDEX.md`) & Skills (`.agents/skills/`)

> **Progressive Disclosure Rule:** Do not read all docs at once. Read [`docs/INDEX.md`](docs/INDEX.md) or use this quick table:

| Question / Action | File or Skill |
|-------------------|---------------|
| What should I work on next? | **`/next`** ([`.agents/skills/next/SKILL.md`](.agents/skills/next/SKILL.md)) |
| What is the phased roadmap & feature dependency wave matrix? | [`docs/planning/roadmap.md`](docs/planning/roadmap.md) |
| Create or expand a feature spec (`requirements`, `design`, `tasks`, `uat`) | **`/spec`** ([`.agents/skills/spec/SKILL.md`](.agents/skills/spec/SKILL.md)) |
| Build Phase 0 scaffold or execute tasks/waves for a feature | **`/build`** ([`.agents/skills/build/SKILL.md`](.agents/skills/build/SKILL.md)) |
| Verify build, EARS acceptance criteria, and stack Hard Failures | **`/verify`** ([`.agents/skills/verify/SKILL.md`](.agents/skills/verify/SKILL.md)) |
| Ship completed feature, update `roadmap.md`, and generate PR summary | **`/ship`** ([`.agents/skills/ship/SKILL.md`](.agents/skills/ship/SKILL.md)) |
| What are we building overall & why? | [`docs/foundation/prd.md`](docs/foundation/prd.md) |
| Where do files live & what is the DB schema? | [`docs/foundation/architecture.md`](docs/foundation/architecture.md) |
| What packages, constants & env vars exist? | [`docs/foundation/tech-stack.md`](docs/foundation/tech-stack.md) |
| Why was X decided this way? | [`docs/foundation/decisions.md`](docs/foundation/decisions.md) |
| How do we execute tasks, waves & cross-model reviews? | [`.agents/rules/execution-protocol.md`](.agents/rules/execution-protocol.md) |
| What are the FDA layer & security rules? | [`.agents/rules/stack-rules.md`](.agents/rules/stack-rules.md) |

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

## 🛡️ Code Review & Security Audit Gate (All AI Models)

When asked to review code, audit security, or evaluate another AI model's findings (even without invoking `/verify`):
1. **Contract First:** Read `docs/foundation/decisions.md`, `.agents/rules/stack-rules.md`, and the active feature's `docs/specs/<NN-feature-slug>/{requirements,design}.md` before flagging issues. Never contradict settled decisions or Non-Goals.
2. **Evidence-Only Severity Tiers (No Ping-Pong):** Only report **`[P0 — Blocker]`** (provable bug, auth/RLS/security exploit, broken EARS requirement, or `stack-rules.md` Hard Failure) and **`[P1 — Defect]`** (unhandled `design.md` Error Matrix case or race condition) with exact `file:line` and a concrete trigger path. Suppress **`[P2 — Style/Nitpick]`** opinions.
3. **Arbiter Mode for External Reviews:** When given findings from another developer's AI review, classify each item against `design.md` + `stack-rules.md` into **`ACCEPT (P0/P1)`** vs. **`REJECT (P2 / Settled Tradeoff / Guarded Upstream)`** before modifying any code.

---

## 🚫 Anti-Hallucination List (Out of Scope for Current Phase)

Do NOT build in MVP / Current Phase:
- No real-time scraping on demand (scraping is scheduled cron only)
- No screenshot / visual diffing (text diff only in V1)
- No API monitoring (website scraping only in V1)
- No white-label / agency tier (Phase 3)
- No team workspaces with shared competitor lists (Phase 3)
- No historical trend graphs (Phase 2)
- No AI-generated competitive battle cards (Phase 2)
- No Chrome extension (Phase 3)
- No Twitter/X API (use public page scraping only — API costs too much for V1)
