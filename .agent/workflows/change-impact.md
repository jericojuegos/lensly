---
description: Pre-change blast radius assessment — run before implementing any bug fix, feature addition, or deletion.
---

# 🔍 Workflow: Change Impact Assessment

**Purpose:** Before writing a single line of code, understand what a change touches, what depends on it, and what could break. This workflow prevents the most common cause of regressions: changes that seem isolated but aren't.

**When to run:**
- Before fixing a bug that touches shared logic
- Before adding a new feature to an existing module
- Before deleting any file, function, column, or route
- Before changing a TypeScript type or interface
- Before changing a database schema
- Before modifying an AI prompt

---

## The 5 Questions (Answer All Before Coding)

### Q1: Which layer does this change touch?

| Layer | Examples | Risk Level |
|-------|---------|------------|
| **Route Handler only** | Change a response shape, add a param | 🟡 Low-Medium |
| **Service only** | Fix business logic, tune AI prompt | 🟡 Low-Medium |
| **Repository only** | Optimize a query, add an index | 🟡 Low-Medium |
| **Shared type / interface** | Change a field name or type | 🔴 High — cascades everywhere |
| **Database schema** | Add/remove/rename a column or table | 🔴 High — requires migration |
| **AI abstraction layer** | Change `AIClient` interface | 🔴 High — affects all AI features |
| **Auth middleware** | Change token validation logic | 🔴 Critical — affects every protected route |
| **Constants** | Change a shared constant value | 🔴 Medium-High — affects all consumers |
| **React component in `shared/`** | Change a shared UI component | 🔴 High — affects all features using it |
| **Feature-isolated component** | Change a component in `features/jobs/` | 🟢 Low — isolated by design |

---

### Q2: What directly depends on what I'm changing?

Before touching anything, answer this:

**For backend changes:**
- Which route handlers call the service I'm changing?
- Which services call the repository I'm changing?
- Which other services share the type/interface I'm changing?
- Does any frontend code depend on the API response shape I'm changing?

**For frontend changes:**
- Which pages/components import the component I'm changing?
- Which hooks depend on the API contract I'm changing?
- Which Zustand store slices are affected?

**For database changes:**
- Which repositories query the table/column I'm changing?
- Which Prisma model types will be regenerated?
- Which API responses include this field?
- Are there any existing records that will be affected by this change?

> Run `grep -r "functionName\|ColumnName\|InterfaceName" src/` to find all consumers before making any change.

---

### Q3: What is the blast radius?

Classify the change before proceeding:

```
🟢 ISOLATED    — affects one file, no shared dependencies
               → Proceed directly. Low risk.

🟡 CONTAINED   — affects multiple files in one layer or one feature
               → List all affected files. Verify each after the change.

🔴 CASCADING   — touches a shared type, interface, schema, or middleware
               → STOP. Write an implementation plan first.
               → Get user confirmation before coding.
               → Update all consumers in the same commit.

⛔ BREAKING    — removes or renames a public API endpoint, DB column,
                 or shared interface that has existing consumers
               → STOP. Requires explicit migration strategy.
               → Never delete without a deprecation/migration plan.
```

---

### Q4: Does this touch the database?

If yes, this checklist is mandatory before any code is written:

```
[ ] What is changing? (add column / remove column / rename / new table / drop table)
[ ] Will existing data be affected? (rows that need backfilling, nullability changes)
[ ] Is a Prisma migration file required? (yes, always)
[ ] Will Prisma regenerate types that other files depend on?
[ ] Is this a breaking change for the API? (removing a field that the frontend reads)
[ ] Do any seed scripts or tests need updating?
```

**Migration naming convention:**
```bash
npx prisma migrate dev --name <verb>_<table>_<what>
# Examples:
npx prisma migrate dev --name add_matches_reasoning_column
npx prisma migrate dev --name remove_jobs_legacy_field
npx prisma migrate dev --name create_interest_signals_table
```

---

### Q5: Does this touch the AI layer?

If yes, additional checks apply:

```
[ ] Am I changing a prompt? → Test with BOTH OpenAI and Anthropic providers
[ ] Am I changing the AIClient interface? → All adapters (openai.ts, anthropic.ts) must be updated
[ ] Am I changing a prompt's expected output shape? → Update the Zod schema that parses the AI response
[ ] Could this change cause the AI to return a different structure? → Add/update output validation
[ ] Am I adding a new AI feature? → Create a new file in lib/ai/prompts/ — do NOT inline the prompt
```

---

## Execution Steps

### Step 1: State the change
Write one sentence describing exactly what you are changing and why:
> *"I am adding a `reasoning` column to the `matches` table and returning it in the GET /matches/:jobId response."*

### Step 2: Run the 5 questions
Answer each question above explicitly before writing any code.

### Step 3: Classify blast radius
Assign one of: 🟢 ISOLATED / 🟡 CONTAINED / 🔴 CASCADING / ⛔ BREAKING

### Step 4: Write an implementation plan (if 🔴 or ⛔)
List every file that will change and what changes in it. Example:

```
Files affected:
1. prisma/schema.prisma         → add `reasoning String?` to Match model
2. prisma/migrations/...        → new migration file (auto-generated)
3. src/repositories/matches.ts  → include reasoning in save() and findByJob()
4. src/services/match.service.ts → pass reasoning from AI response to repository
5. src/types/matches.ts         → add `reasoning?: string` to MatchResult type
6. src/lib/ai/prompts/matchScore.ts → update prompt to request reasoning in output
7. Frontend: hooks/useMatches.ts → type update only, no logic change
8. Frontend: features/matches/MatchCard.tsx → render reasoning field
```

Get user confirmation before proceeding.

### Step 5: Execute atomically
Follow `rules/atomic-rules.md`. Each file in the plan is one atomic step. Verify after each.

### Step 6: Verify the full blast radius
After all changes are made, verify every layer that was touched:

```
[ ] Backend: relevant unit/integration tests pass
[ ] Database: migration applied cleanly, no data loss
[ ] API: response shape matches what frontend expects
[ ] Frontend: UI renders correctly, no TypeScript errors
[ ] AI: output still parses correctly with updated Zod schema
[ ] Both AI providers tested (if AI layer was touched)
```

---

## Quick Reference: Change Type → Required Actions

| Change Type | Migration? | Plan Required? | Test Both AI Providers? |
|-------------|-----------|----------------|------------------------|
| Fix a bug in one service | No | No (🟢) | Only if AI layer touched |
| Add a field to an API response | No | If shared type changes | No |
| Add a DB column | **Yes** | Yes (🔴) | No |
| Remove a DB column | **Yes** | Yes (⛔) | No |
| Change a prompt | No | No (🟡) | **Yes** |
| Change `AIClient` interface | No | Yes (🔴) | **Yes** |
| Add a new feature | Maybe | Yes (🟡+) | If AI feature |
| Delete a feature | Maybe | Yes (⛔) | No |
| Change a shared type | No | Yes (🔴) | No |
| Change auth middleware | No | Yes (⛔) | No |
| Add a new API endpoint | No | No (🟢) | No |
| Change a constant value | No | 🟡 — find all consumers | No |

---

## The Deletion Protocol (⛔ Special Case)

Deleting anything — a file, function, column, route, or feature — is the highest-risk operation. Always follow this sequence:

1. **Find all consumers** — `grep -r "thingBeingDeleted" src/`
2. **Confirm no active consumers** — if any exist, they must be updated first
3. **Deprecate before deleting** (for API endpoints) — mark as deprecated in one commit, delete in the next
4. **For DB columns** — write a migration; never drop manually
5. **For feature folders** — confirm with user before deleting; log in activity log
6. **Commit message must say "remove"** — `refactor(jobs): remove legacy status field`

> The rule: **nothing is deleted in the same commit it is still being used.**
