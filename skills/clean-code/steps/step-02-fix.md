---
name: step-02-fix
description: Apply Clean Code fixes identified in step-01
prev_step: steps/step-01-analyze.md
next_step: null
---

# Step 2: FIX

## YOUR MISSION

Apply the Clean Code fixes identified in step-01, following patterns from `references/clean-code-principles.md`.

**Before every single fix, ask: "Does this change make the code easier to understand?"**
If NO → skip it, add it to the "skipped" list with a reason.

---

## EXECUTION SEQUENCE

### 1. Apply fixes

**Process by priority: 🔴 → 🟡 → 🟢**

For each violation:
1. Re-read the corresponding principle in `references/clean-code-principles.md`
2. Apply the BAD → GOOD pattern from the reference
3. **Verify the result is simpler and clearer than before** — if not, revert and skip
4. Never invent a pattern not in the reference

**If 4+ files to modify and `{economy_mode}` = false:**
→ Use parallel Snipper agents, one per file

**If few files or `{economy_mode}` = true:**
→ Apply with Edit directly

### 2. Rules during modifications

- **DO NOT change functional behavior** — pure refactoring
- **DO NOT add new features** — scope limited to identified violations
- **DO NOT reformat untargeted code** — only touch listed violations
- **DO NOT apply a fix that introduces more abstraction layers without clear readability gain**
- If a fix reveals another issue: note it but DO NOT apply without confirmation

### 3. Track progress

```markdown
## Fixes applied

| Violation | File:Line | Status | Result |
|-----------|-----------|--------|--------|
| Flag argument | auth.ts:45 | ✅ | Split into sendWelcomeEmail() and sendAdminAlert() |
| Magic number | user.ts:12 | ✅ | Extracted as RETIREMENT_AGE = 65 |
| DRY | order.ts:78 | ✅ | Extracted shared validateEmail() |
| Extract function | utils.ts:15 | ⏭️ Skipped | 4-line block read naturally inline — extracting adds indirection |
```

### 4. Build check

```bash
npx tsc --noEmit
```

If TypeScript errors → fix them before continuing.

### 5. Final summary

```markdown
## Clean Code — Summary

### Fixes applied: {n}
- 🔴 Critical: {n}
- 🟡 Important: {n}
- 🟢 Minor: {n}
- ⏭️ Skipped (no clarity gain): {n}

### Impact
- Functions extracted: {n}
- Named constants: {n}
- Duplications eliminated: {n}

### TypeScript
- ✅ Compiles without errors
```

---

## GOLDEN RULES FOR FIXES

- **Refactoring = same behavior, better code** — never break anything
- **Simpler result is the only acceptable outcome** — if unsure, skip
- Always refer to the checklist in the reference before each modification
- One fix per violation — don't use it as an excuse to "rewrite everything"
- If a refactoring feels risky (public signature change, interface modification) → flag to the user before applying

---

## END OF WORKFLOW

Present the final summary to the user.

<critical>
Never change functional behavior — pure refactoring only.
Never apply a fix that makes the code harder to understand than before.
</critical>
