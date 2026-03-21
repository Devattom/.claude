---
name: clean-code
description: Analyze and fix code using Robert C. Martin's Clean Code principles — naming, functions, SOLID, DRY, code smells. Usable in review mode (audit) or guide mode (during code writing).
argument-hint: "[-a] [-e] [-r] <file|feature|folder>"
---

<objective>
Apply Robert C. Martin's Clean Code principles to make code readable, maintainable, and clean.
Two modes: audit existing code (review) or guide code writing (guide).
</objective>

<prime_directive>
**READABILITY AND SIMPLICITY ALWAYS WIN.**

Clean Code rules are tools, not dogma. If strictly applying a rule makes the code harder to understand, less simple, or more complex overall — DO NOT apply it.

Before applying any rule, ask: "Does this change make the code easier to understand for the next developer?"
If the answer is NO or UNCERTAIN → skip the change, explain why.

Examples where NOT to apply Clean Code blindly:
- Extracting a 3-line helper function that will only ever be called once → adds indirection without clarity
- Renaming a well-understood abbreviation (`url`, `id`, `i`) to a verbose name → noise, not clarity
- Splitting a simple 25-line function that reads naturally as a single block → fragmentation hurts comprehension
- Applying polymorphism to a switch with 2 cases that will never grow → over-engineering
- Extracting a boolean condition into a named function when the condition is already obvious

**The goal is a codebase that any developer can read fluently. Rules serve that goal — not the other way around.**
</prime_directive>

<quick_start>

**Audit existing code:**
```bash
/clean-code src/features/auth
/clean-code src/services/user.service.ts
```

**Auto mode (no confirmation):**
```bash
/clean-code -a src/features/auth
```

**Report only (no fixes applied):**
```bash
/clean-code -r src/features/auth
```

**Guide mode (during code writing):**
```bash
/clean-code --guide
```
→ Loads principles into context to guide the current writing session.

</quick_start>

<parameters>

| Flag | Description |
|------|-------------|
| `-a` | Auto mode: apply all fixes without confirmation |
| `-e` | Economy mode: no subagents, direct tools only |
| `-r` | Report only: produce the report without applying fixes |
| `--guide` | Guide mode: load principles to guide code writing |

</parameters>

<modes>

### REVIEW mode (default)
Analyze existing code → identify violations → propose/apply fixes.

Workflow: `step-01-analyze.md` → `step-02-fix.md`

### GUIDE mode (`--guide`)
Load Clean Code principles into context to guide a code writing session.
No multi-step workflow — act as an active reference while the agent writes.

**In guide mode:**
1. Read `references/clean-code-principles.md`
2. Confirm: "Clean Code principles loaded. I'll apply naming, SOLID, DRY, small functions, and error handling rules — but only when they improve readability."
3. While writing: proactively flag violations before committing code

</modes>

<state_variables>

| Variable | Type | Description |
|----------|------|-------------|
| `{target}` | string | File, feature or folder to analyze |
| `{auto_mode}` | boolean | `-a`: apply without confirmation |
| `{economy_mode}` | boolean | `-e`: no subagents |
| `{report_only}` | boolean | `-r`: report only, no fixes |
| `{guide_mode}` | boolean | `--guide`: writing mode |

</state_variables>

<reference_files>

| File | When to load |
|------|--------------|
| `references/clean-code-principles.md` | Always — before any analysis or writing |

</reference_files>

<entry_point>

**If `--guide`:**
→ Load `references/clean-code-principles.md` + confirm guide mode. Done.

**Otherwise:**
→ Load `steps/step-01-analyze.md`

</entry_point>

<step_files>

| Step | File | Purpose |
|------|------|---------|
| 01 | `steps/step-01-analyze.md` | Read code, identify violations |
| 02 | `steps/step-02-fix.md` | Apply corrections |

</step_files>

<when_to_use>

Trigger this skill when:
- An agent just wrote new code → `/clean-code --guide` before writing or review after
- Code reviewing a PR → `/clean-code src/features/xxx`
- Refactoring an existing module → `/clean-code -a src/module`
- Doubt about naming, function structure, or SOLID compliance

**Principles covered:**
- Intent-revealing naming (no magic numbers, consistent verb choices)
- Small functions, single responsibility, CQS, no hidden side effects
- Comments: when to write, when to delete
- DRY — duplicate elimination
- SOLID: SRP, OCP, LSP, ISP, DIP
- Law of Demeter — no train wrecks
- Error handling: exceptions vs return codes, never null
- Code smells: rigidity, fragility, needless complexity, Feature Envy, etc.

</when_to_use>

<execution_rules>
- Always read `references/clean-code-principles.md` before analyzing or fixing
- **PRIME DIRECTIVE FIRST**: if a fix reduces overall readability or simplicity, skip it and explain why
- Never modify functional behavior — pure refactoring only
- Cite exact file:line for each violation
- Reference the exact principle (DRY, SRP, CQS, etc.) for each violation
- Do not invent patterns not present in the reference
- In review mode: propose fixes before applying (unless `-a`)
- In guide mode: proactively flag violations during writing
</execution_rules>

<success_criteria>
- All violations identified with file:line + principle
- Code more readable, names that reveal intent
- Small functions, no hidden side effects
- SOLID respected where it genuinely improves the code
- No functional regression
- Clean TypeScript build after fixes
- Every skipped rule has an explicit reason ("skipped: would add indirection without clarity gain")
</success_criteria>
