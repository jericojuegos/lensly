---
trigger: always_on
---

# 💰 Rule: Budget Guard & Token Efficiency

**Purpose:** Prevent token waste, runaway loops, and context bloat.

---

## 1. The "Pause & Ask" Protocol

You **MUST** pause and request user approval before proceeding if **ANY** of the following thresholds are met:

- **File Scope:** You intend to edit more than **3 files** in a single turn.
  - *Exception:* A declared Coupled Logic batch in `atomic-rules.md` Section 2 is pre-authorized and does not trigger this threshold. The agent must have explicitly declared the coupling before proceeding.
- **Command Tiering:**
  - **Tier 1 (warn):** More than **2 long-running commands** in one turn (e.g., package installs, builds, migrations). Pause and state what you intend to run.
  - **Tier 2 (hard stop):** Any destructive action (deleting files, `DROP TABLE`, overwriting configs). Always require explicit confirmation, regardless of count.
- *Exemption:* Read-only commands (`ls`, `cat`, `grep`, `pwd`, `git log`, `git status`) never count toward any threshold.

**Trigger phrase:** `"⚠️ BUDGET GUARD TRIGGERED: [reason]. Awaiting approval..."`

### User Override
If the user explicitly says to proceed despite a Budget Guard trigger (e.g., "just do it"), the agent may comply, but must:
1. Acknowledge the override: *"Proceeding on explicit override."*
2. Log it in the sprint Activity Log: `[override] Budget Guard bypassed — [reason]`.

---

## 2. Loop Prevention (The "3-Strike Rule")

If the same error has been attempted **3 times** without success:

1. **STOP** — Do not attempt a 4th fix.
2. **REPORT** — Show the exact error, the 3 approaches tried, and why each failed.
3. **ASK** — Request explicit guidance before continuing.

> This rule exists because repeated blind retries burn tokens without progress and often make the problem worse.

---

## 3. Planning Before Code

For any change touching **more than 1 file**, generate a concise **Implementation Plan** first:
- List each file to be changed and what changes in it.
- Wait for user confirmation before writing code.

*Exception:* If the task is already fully defined in `planning/01-active-sprint.md` with sub-tasks broken down, and it touches only 1 file, you may proceed without a separate plan.

> The relationship between this section and Section 1: the file-scope threshold (Section 1) is the hard stop. This section (3) is the earlier, softer gate — plan before you even reach the threshold.

---

## 4. Context Compression (TOON)

When displaying diagnostic output or large data structures in chat, use **TOON (Token-Oriented Object Notation)**:

- **Skip boilerplate** — Replace standard imports or repeated patterns with `# ...`
- **Focus on changes** — Show only lines relevant to the logic being discussed
- **Abbreviate keys** — Use shorter keys in summaries

**⚠️ TOON Guardrail:** TOON is for *diagnostic output and summaries only*. Never abbreviate implementation code, config files, or anything being written to disk.

#### TOON Example (Python):
```python
# ❌ Verbose (~200 tokens)
{
    "user_identifier": "a1b2c3d4-e5f6-7890-abcd-1234567890ef",
    "email_address": "test.user@example.com",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-02-05T14:22:11Z"
}

# ✅ TOON (~50 tokens, 75% reduction)
{ "id": "a1b2...", "email": "test.u@...", "created_at": "2024-01...", # +1 field }
```
