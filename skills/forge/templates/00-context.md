# Forge Task: {{task_id}}

**Created:** {{timestamp}}
**Task:** {{task_description}}

---

## Configuration

| Flag | Value |
|------|-------|
| Budget | {{budget}} |
| Team (`-w`) | {{team_mode}} |
| Doc (`--doc`) | {{doc_mode}} |
| Advisor uses remaining | {{advisor_uses_remaining}} |

---

## User Request

```
{{original_input}}
```

## Reference Documents

{{reference_docs}}

---

## Progress

| Phase | Status | Timestamp |
|-------|--------|-----------|
| 00-init | ⏸ Pending | |
| 01-research | ⏸ Pending | |
| 02-plan | ⏸ Pending | |
| 03-execute | ⏸ Pending | |
| 04-test | ⏸ Pending | |
| 05-document | {{doc_status}} | |

---

## State Snapshot

**feature_name:** {{feature_name}}
**next_step:** 01

### Acceptance Criteria

_Defined during phase 01-research_

### Step Context

_Brief summaries added as steps complete_

### Gotchas

_Surprises and deviations discovered during execution_
