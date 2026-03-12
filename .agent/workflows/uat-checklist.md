---
description: Generates a UAT (User Acceptance Testing) checklist at the end of every sprint or before go-live.
---

# ✅ Workflow: UAT Checklist Generation

**Trigger:** At the end of every sprint (after all tasks in `01-active-sprint.md` are `[x]`), or when the user requests a UAT review.

---

## The 3 Levels of Acceptance Testing

```
Level 3: USER JUDGMENT     → Does this feel right? Is UX sensible?
Level 2: UAT CHECKLIST     → Agent-generated, user executes manually
Level 1: AUTOMATED TESTS   → Agent writes and runs these itself
```

> ⚠️ The agent is responsible for Levels 1 and 2. Never leave Level 1 for the user to do manually.

---

## 1. Generate the UAT Checklist

After completing all sprint tasks, generate a checklist covering:

1. **Happy path scenarios** — Step-by-step, as if the user is performing the action.
2. **Edge cases and error states** — What happens on bad input, network failure, double-clicks, etc.
3. **Security scenarios** — Relevant to the module (e.g., injection, auth bypass, token handling).
4. **Pass condition** — What "pass" looks like for each item.
5. **Environment setup** — Any prerequisites needed to test (e.g., test API keys, seed data).

### Format

```markdown
## UAT CHECKLIST — [Module Name]

### [Feature Group]
- [ ] [Action the user takes]
  - Steps: [1. Do X → 2. See Y]
  - Pass: [Expected outcome]
- [ ] [Edge case]
  - Steps: [1. Do X with bad input]
  - Pass: [Error message shown, no crash]
```

### Example (Auth Module)

```markdown
## UAT CHECKLIST — Auth Module

### REGISTER
- [ ] User can register with valid email + password
  - Pass: Account created, confirmation shown
- [ ] Duplicate email shows clear error message
  - Pass: "Email already in use" displayed
- [ ] Weak password is rejected with explanation
  - Pass: Validation message lists requirements
- [ ] Empty fields are rejected before hitting the server
  - Pass: Client-side validation fires

### LOGIN
- [ ] User can log in with correct credentials
  - Pass: Redirected to dashboard
- [ ] Wrong password shows error (not which field is wrong)
  - Pass: Generic "Invalid credentials" message
- [ ] Protected routes are inaccessible before login
  - Pass: Redirect to login page

### EDGE CASES
- [ ] Double-click on register button
  - Pass: Only one account created
- [ ] SQL injection attempt in email field
  - Pass: Input sanitized, no server error
```

---

## 2. Where to Store

Store completed UAT checklists in `planning/uat/` with one file per sprint or module:

```
planning/
└── uat/
    ├── sprint-1-foundation.md
    ├── sprint-2-pipeline.md
    └── auth-module.md
```

This builds a **reusable UAT template library** over time — similar modules (auth, billing, uploads) will share patterns across projects.

---

## 3. End-of-Project: Full E2E Checklist

When all modules are complete and before go-live, generate a comprehensive end-to-end checklist covering:

- Complete user journeys from signup to active usage
- Cross-module interactions (e.g., credits affecting feature access, roles affecting visibility)
- Admin vs regular user differences
- Mobile/responsive considerations if applicable

Store this as `planning/uat/go-live-checklist.md`.

---

## 4. Reuse & Compounding

When generating a UAT checklist for a module type that has been tested before (in this or other projects):

1. Reference the existing checklist as a baseline.
2. Only add module-specific items on top.
3. Flag any items that were previously problematic (e.g., "This edge case failed in Sprint 1 — retest").
