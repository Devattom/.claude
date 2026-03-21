---
name: step-01-analyze
description: Analyze code and identify Clean Code violations
prev_step: null
next_step: steps/step-02-fix.md
---

# Step 1: ANALYZE

## YOUR MISSION

You are a Clean Code expert (Robert C. Martin). Read the target code and identify each Clean Code violation with surgical precision.

**But first — the prime directive:**

> Before flagging any violation, ask: "Does fixing this make the code genuinely easier to understand?"
> If NO or UNCERTAIN → do not flag it. Note it as "skipped: no clarity gain."

This is not a mechanical audit. It's a judgment call on readability.

---

## EXECUTION SEQUENCE

### 1. Read the reference

**MANDATORY**: Read `references/clean-code-principles.md` — especially the PRIME DIRECTIVE section — before starting.

### 2. Identify the scope

- If `{target}` is a file → analyze that file
- If `{target}` is a feature/folder → explore with Glob + read key files
- If `{target}` is empty → ask the user with AskUserQuestion

### 3. Read the target code

Use Read to read each file in scope. Do not skip this step.

### 4. Produce the violation report

Structure the report as follows:

```markdown
## Clean Code Report — {target}

### 🔴 Critical violations
| File:Line | Rule violated | Description | Clarity impact |
|-----------|--------------|-------------|----------------|
| auth.ts:45 | Flag argument | `sendEmail(user, true)` — boolean signals 2 behaviors | High: caller has no idea what "true" means |
| user.ts:12 | Magic number | `if (age > 65)` — extract as `RETIREMENT_AGE` constant | High: the number has no expressed meaning |

### 🟡 Important violations
| File:Line | Rule violated | Description | Clarity impact |
|-----------|--------------|-------------|----------------|
| order.ts:78 | DRY | Email validation duplicated (also line 102) | Medium: maintenance risk |

### 🟢 Minor improvements
| File:Line | Rule violated | Description | Clarity impact |
|-----------|--------------|-------------|----------------|
| service.ts:23 | Redundant comment | `// increment counter` above `count++` | Low: noise |

### ⏭️ Skipped (rule applies but clarity gain is negative)
| File:Line | Rule | Reason skipped |
|-----------|------|----------------|
| utils.ts:15 | Extract function | 4-line block, called once, reads naturally inline — extracting adds indirection |
| helper.ts:8 | Polymorphism | Switch with 2 stable cases that will never grow — a class hierarchy would be overkill |

### ✅ What's done well
- Consistent `findById` naming across all repositories
- SRP respected in UserRepository
- No null returns in public API
```

**Be SPECIFIC**: cite the exact file:line, the exact principle, and briefly how to fix it.

**Be BALANCED**: note what's done well — the goal is learning, not just criticism.

**Be HONEST about skipped rules**: explicitly document what you chose not to flag and why.

### 5. Confirmation

**If `{auto_mode}` = true:**
→ Load `step-02-fix.md` directly

**If `{auto_mode}` = false:**
→ Display the report, then ask:
```
What do you want to do?
1. Apply all fixes (recommended)
2. Apply critical violations only
3. Report only — no changes
```

---

## GOLDEN RULES FOR ANALYSIS

- **No fixes in this step** — analyze only
- Cite the **exact line number** (format `file.ts:42`)
- Reference the **exact principle** (DRY, SRP, CQS, Law of Demeter, etc.)
- Do not invent violations — base everything on `references/clean-code-principles.md`
- **Always document skipped rules** with a reason
- Distinguish "certain violation" from "suggestion for improvement"
- A 25-line function that reads as natural prose is NOT a violation — it's good code
