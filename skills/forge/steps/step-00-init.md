---
name: step-00-init
description: Initialize Forge workflow — parse flags, setup output, initialize state
next_step: ./step-01-research.md
---

# Phase 0: Initialization

## RULES:

- 🛑 NEVER skip flag parsing
- ✅ ALWAYS parse ALL flags before any other action
- 🚫 FORBIDDEN to load step-01 until init is complete

## EXECUTION SEQUENCE:

### 1. Parse Flags and Input

**Step 1a: Load defaults**

```yaml
budget: mid
team_mode: false
doc_mode: false
reference_files: ""
```

**Step 1b: Parse user input and override defaults:**

```
--budget high    → {budget} = high
-w or --team     → {team_mode} = true
--doc            → {doc_mode} = true
```

Remaining input after flags → `{task_description}`

**Step 1c: Detect reference files in input:**

```
Scan {task_description} for file path tokens:
1. A token is a file path if it contains '/' AND ends with .md, .txt, .json, .yaml, .yml
2. If the file exists: {reference_files} = path, remove from {task_description}
3. If {task_description} is now empty: derive description from filename
4. If no file paths: {reference_files} = "" (normal mode)
```

**Step 1d: Generate feature_name and task_id:**

```
{feature_name} = kebab-case of {task_description}
Example: "add user authentication" → "add-user-authentication"
```

```bash
bash {skill_dir}/scripts/generate-task-id.sh "{feature_name}"
```

**Step 1e: Set advisor budget:**

```
{advisor_uses_remaining} = 2   (if budget=mid)
{advisor_uses_remaining} = 4   (if budget=high)
```

### 2. Pre-flight Check

```bash
if [[ -z "{task_description}" ]]; then
  echo "Error: No task description provided"
  exit 1
fi
```

### 3. Create Output Structure

```bash
bash {skill_dir}/scripts/setup-templates.sh \
  "{task_id}" \
  "{task_description}" \
  "{budget}" \
  "{team_mode}" \
  "{doc_mode}" \
  "{original_input}" \
  "{reference_files}"
```

### 4. Mark Init Complete and Proceed

```bash
bash {skill_dir}/scripts/update-progress.sh "{task_id}" "00" "init" "complete"
```

Show compact summary:

```
✓ FORGE: {task_description}

| Variable | Value |
|----------|-------|
| task_id  | {task_id} |
| budget   | {budget} |
| team     | {team_mode} |
| doc      | {doc_mode} |
| advisor  | {advisor_uses_remaining} uses |

→ Researching...
```

**Then immediately load step-01-research.md.**
