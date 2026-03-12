---
description: Initialize the agent brain and verify critical constraints before scaffolding
---

# 🚀 Workflow 01 — Init
## Runs: ONCE. When the brain exists but zero code has been written.
## Next: → `02-scaffold.md`

---

## Step 1 — Confirm Brain Is Complete

Read every `.agent/` file. Verify nothing is a placeholder:

- [ ] `AGENT.md` — navigation table + critical constraints filled in
- [ ] `knowledge/spec.md` — product summary, monitor types, credit system, risks
- [ ] `knowledge/architecture.md` — topology, file structure, DB schema, all 5 data flows
- [ ] `knowledge/tech-stack.md` — stack table, Deno imports section, prohibited list
- [ ] `knowledge/decisions.md` — all 7 ADRs present
- [ ] `planning/00-roadmap.md` — Phase 0 + Phase 1 fully detailed
- [ ] `planning/01-active-sprint.md` — Phase 0 tasks as `[ ]`, activity log started
- [ ] `planning/02-backlog.md` — deferred + rejected tables populated
- [ ] `rules/coding-standards.md` — robots.txt pattern + pre-commit checklist
- [ ] `rules/architecture-rules.md` — 8 rules present

**Any file incomplete → stop. Fill it. Then return here.**

---

## Step 2 — Lock the Critical Constraints

Read `AGENT.md` critical constraints. Confirm every one of these is understood:

- [ ] All scraping runs in Edge Functions — never in Next.js Route Handlers (Vercel timeout)
- [ ] `robots.txt` checked before every single URL fetch — zero exceptions
- [ ] Only the text diff is sent to AI — never full page content
- [ ] Diff is truncated to `MAX_DIFF_LENGTH_FOR_AI` (4000 chars) before the API call
- [ ] Credits deducted only after successful AI analysis — not on scrape, not on error
- [ ] Raw HTML discarded immediately after text extraction — never stored
- [ ] Slack webhook URL never returned in any API response
- [ ] `SUPABASE_SERVICE_ROLE_KEY` server-side only — never in `NEXT_PUBLIC_` or mobile

**If you cannot confirm all 8 → reread `AGENT.md` and `rules/architecture-rules.md` before continuing.**

---

## Step 3 — Resolve External Dependencies

Lensly cannot complete certain phases without these. Confirm status for each:

| Dependency | Needed For | Action If Missing |
|-----------|-----------|------------------|
| Supabase project created | Phase 0 — Day 1 | Create at supabase.com now |
| Supabase service role key | Phase 0 — Edge Functions | Copy from Supabase dashboard → Settings → API |
| Google OAuth app | Phase 0 — Auth | Create at console.cloud.google.com |
| Anthropic API key | Phase 1 — AI analyser | Get at console.anthropic.com |
| Resend account + API key | Phase 1 — Email digests | Create at resend.com (free tier fine for V1) |
| Stripe account | Phase 1 — Credit purchase | Create at stripe.com |

- [ ] All Phase 0 dependencies: ✅ resolved
- [ ] Phase 1 dependencies: document any pending ones in `planning/01-active-sprint.md` under 🛑 Blocked

---

## Step 4 — Lock Stack Versions

Run these commands and record the output in `knowledge/tech-stack.md` under a new "Locked Versions" section:

```bash
node --version
pnpm --version
supabase --version   # Supabase CLI
npx expo --version
```

This prevents "works on my machine" problems when resuming the project later.

---

## Step 5 — Update Activity Log

Open `planning/01-active-sprint.md`. Append to the Activity Log:

```
| YYYY-MM-DD | Init workflow complete. Brain verified. Scope locked. External deps documented. Ready to scaffold. |
```

---

## ✅ Done

**→ Next: `02-scaffold.md`**
