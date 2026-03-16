---
name: draft-implementation-plan
description: Produce a story-scoped implementation plan with technical decisions, AC-mapped steps, unit test coverage, and blocker tracking before coding starts.
---

# Skill: draft-implementation-plan

**Version:** 0.1.0

---

## Purpose

Create a structured implementation plan from a Story and its constraints. The plan is the required bridge between Story confirmation and code generation.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                        |
| ------------------------ | -------- | -------- | -------------------------------------------------- |
| `story`                  | `object` | Yes      | Parsed Story artifact with ACs and scope           |
| `architecture_artifacts` | `object` | Yes      | Referenced architecture constraints and contracts  |
| `procedures`             | `array`  | Yes      | Referenced procedures for module interaction order |
| `existing_plan`          | `object` | No       | Existing plan for update flows                     |
| `mode`                   | `string` | Yes      | `"initialize"` or `"update"`                       |

### Output

```json
{
  "status": "ok" | "error",
  "plan_id": "PLAN-<story-id>",
  "artifact": "<markdown string>",
  "ac_coverage": {
    "steps": "complete" | "incomplete",
    "tests": "complete" | "incomplete"
  },
  "error": "<error message or null>"
}
```

---

## Artifact Structure

```markdown
# PLAN-<story-id>: <Story Title>

**Story:** STORY-<id>
**Version:** <semver>
**Status:** Draft | Confirmed | Superseded

## Technical Decisions

<Decision, rationale, and alternatives>

## Implementation Steps

<Ordered steps mapped to ACs>

## Unit Test Plan

<Test cases mapped to ACs>

## Open Blockers

<CR blockers or "None">
```

---

## Process

1. Read Story ACs, architecture constraints, and procedure rules.
2. Define technical decisions within allowed architecture/process boundaries.
3. Draft implementation steps and map each step to one or more ACs.
4. Draft unit test coverage and map each AC to one or more test cases.
5. Add `Open Blockers` for ACs blocked by active CRs.
6. Validate complete bidirectional AC traceability (AC -> step/test and step/test -> AC).

---

## Rules

- Every AC must map to at least one implementation step.
- Every AC must map to at least one unit test case.
- No step may exist without AC mapping.
- `Open Blockers` section is mandatory (`None` allowed).

---

## Validation

| Check                | Rule                                  |
| -------------------- | ------------------------------------- |
| AC-to-step coverage  | All ACs are covered by steps          |
| AC-to-test coverage  | All ACs are covered by tests          |
| No out-of-scope step | Every step maps back to Story ACs     |
| Blocker declaration  | Blocked ACs appear in `Open Blockers` |

---

## Error Codes

| Code                | Meaning                                    |
| ------------------- | ------------------------------------------ |
| `ERR_NO_STORY`      | Story input is missing or invalid          |
| `ERR_NO_ARCH_INPUT` | Architecture artifacts are missing         |
| `ERR_NO_PROCEDURES` | Referenced procedures are missing          |
| `ERR_MODE_INVALID`  | `mode` is not `"initialize"` or `"update"` |
