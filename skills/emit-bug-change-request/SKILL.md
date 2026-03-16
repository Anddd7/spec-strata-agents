---
name: emit-bug-change-request
description: Generate bug-focused internal change requests from QA to Dev for failed black-box test cases, including reproducible evidence and severity.
---

# Skill: emit-bug-change-request

## Purpose

Create a structured bug CR for failed QA test cases and write it to `spec/change-requests/`.
This skill is dedicated to QA-to-Dev bug feedback.

## Input

| Parameter            | Type     | Required | Description                            |
| -------------------- | -------- | -------- | -------------------------------------- |
| `type`               | `string` | Yes      | Must be `"bug"`                        |
| `from_layer`         | `string` | Yes      | Must be `"qa"`                         |
| `to_layer`           | `string` | Yes      | Must be `"dev"`                        |
| `test_plan_id`       | `string` | Yes      | Related test plan ID                   |
| `test_case_ids`      | `array`  | Yes      | One or more failing test case IDs      |
| `story_or_release`   | `string` | Yes      | Story ID or release identifier         |
| `observed_behavior`  | `string` | Yes      | Actual observed behavior               |
| `expected_behavior`  | `string` | Yes      | Expected behavior                      |
| `reproduction_steps` | `array`  | Yes      | Ordered reproduction steps             |
| `severity`           | `string` | Yes      | `critical`, `high`, `medium`, or `low` |

## Output

```json
{
  "status": "ok" | "error",
  "cr_id": "CR-<YYYYMMDD>-<NNN>-qa-bug",
  "file_path": "spec/change-requests/<cr_id>.md",
  "error": "<error message or null>"
}
```

## Rules

- Reject input if `type` is not `bug`.
- Reject input if `from_layer` is not `qa` or `to_layer` is not `dev`.
- Create one CR per distinct root cause; include multiple `test_case_ids` when failures share the same bug.
- Include reproducible evidence and clear expected vs observed behavior.
