---
name: generate-code
description: Generate executable business logic and co-located unit tests from a confirmed implementation plan while respecting architecture boundaries and open blockers.
---

# Skill: generate-code

**Version:** 0.1.0

---

## Purpose

Generate production code and unit tests from a confirmed implementation plan with strict scope control.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                  |
| ------------------------ | -------- | -------- | -------------------------------------------- |
| `confirmed_plan`         | `object` | Yes      | Confirmed `PLAN-<story-id>` artifact         |
| `architecture_artifacts` | `object` | Yes      | Component boundaries and interface contracts |
| `target_paths`           | `array`  | Yes      | File/module targets for each plan step       |
| `open_blockers`          | `array`  | No       | ACs blocked by open CRs                      |

### Output

```json
{
  "status": "ok" | "error",
  "generated_files": ["<path>", "..."],
  "skipped_ac": ["AC-<n>", "..."],
  "error": "<error message or null>"
}
```

---

## Process

1. Read confirmed plan and verify status allows code generation.
2. Generate business logic files according to implementation steps.
3. Generate corresponding unit tests as co-located `<module>.test.<ext>` files.
4. Skip any AC listed as blocked and record it in `skipped_ac`.
5. Validate boundary compliance against architecture contracts.

---

## Rules

- Follow the confirmed plan exactly; do not add new scope.
- Generate business logic before unit tests.
- Keep tests co-located with target modules.
- Do not generate code for blocked ACs.
- Do not bypass defined interfaces across component boundaries.

---

## Validation

| Check               | Rule                                                  |
| ------------------- | ----------------------------------------------------- |
| Plan status         | Plan must be confirmed                                |
| Scope control       | Every generated change maps to at least one plan step |
| Blocker enforcement | No generated code for blocked ACs                     |
| Boundary compliance | Cross-component calls use defined contracts           |

---

## Error Codes

| Code                     | Meaning                                             |
| ------------------------ | --------------------------------------------------- |
| `ERR_PLAN_NOT_CONFIRMED` | Plan status is not confirmed                        |
| `ERR_NO_TARGET_PATH`     | Target paths for one or more steps are missing      |
| `ERR_BOUNDARY_VIOLATION` | Planned generation would violate component boundary |
