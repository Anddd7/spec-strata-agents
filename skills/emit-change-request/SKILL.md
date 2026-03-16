---
name: emit-change-request
description: Generate structured internal change requests under spec/change-requests when downstream layers detect actionable specification conflicts or ambiguities.
---

# Skill: emit-change-request

**Version:** 0.3.0

---

## Purpose

Generate a structured internal Change Request file and write it to `spec/change-requests/`. This skill is the standard inter-layer feedback mechanism for **Spec Infrastructure conflicts** discovered by downstream agents.

> This skill handles **internal CRs only** — conflicts that originate within the agent system and flow bottom-up to a human decision point.
> Feature input from humans requires no special format and is handled directly by `read-input-source`.

---

## Interface

### Input

| Parameter            | Type     | Required | Description                                                                  |
| -------------------- | -------- | -------- | ---------------------------------------------------------------------------- |
| `type`               | `string` | Yes      | Always `"internal"`                                                          |
| `from_layer`         | `string` | Yes      | Originating layer name (e.g. `"Architecture"`)                               |
| `to_layer`           | `string` | Yes      | Target layer name (e.g. `"Domain"`)                                          |
| `trigger`            | `string` | Yes      | The specific task or requirement that surfaced the conflict                  |
| `conflict`           | `string` | Yes      | Description of the specific conflict or gap in the Spec                      |
| `chesterton_context` | `string` | Yes      | Why the existing element was designed this way; what currently depends on it |
| `impact`             | `string` | Yes      | Effect on the originating layer if left unresolved                           |
| `options`            | `array`  | Yes      | Two or more resolution options, each with `label` and `trade_offs`           |
| `recommendation`     | `string` | Yes      | The agent's recommended option with rationale                                |

### Output

```json
{
  "status": "ok" | "error",
  "cr_id": "CR-<YYYYMMDD>-<NNN>-<layer>",
  "file_path": "spec/change-requests/<cr_id>.md",
  "error": "<error message or null>"
}
```

---

## Process

1. **Validate `type`:** If `type` is not `"internal"`, return `status: error` with `ERR_WRONG_TYPE`.
2. **Generate CR ID:** `CR-<YYYYMMDD>-<NNN>-<layer_abbreviation>`
   - `NNN` is a zero-padded sequence number, incremented by reading existing CR files in `spec/change-requests/`.
   - Layer abbreviations: `domain`, `arch`, `process`, `dev`, `validator`
3. **Write the CR file** to `spec/change-requests/<cr_id>.md`.
4. **Return the output object.**

---

## Change Request File Format

```markdown
# Change Request <cr_id>

**Date:** <YYYY-MM-DD>
**Type:** Internal
**From:** <from_layer>
**To:** <to_layer>
**Status:** Open
**Trigger:** <trigger>

---

## Conflict

<conflict>

## Chesterton's Fence

<chesterton_context>

## Impact

<impact>

## Options

### Option A — <label>

<trade_offs>

### Option B — <label>

<trade_offs>

## Recommendation

<recommendation>
```

---

## Status Lifecycle

| Status     | Meaning                                                          |
| ---------- | ---------------------------------------------------------------- |
| `Open`     | CR emitted; downstream layer is paused; awaiting human decision  |
| `Accepted` | Human has approved a resolution option                           |
| `Rejected` | Human has decided no change is needed; downstream layer resumes  |
| `Resolved` | Approved changes applied and validated; downstream layer resumes |

Status is updated by the receiving agent or human — not by the emitting agent.

---

## Validation

| Check                       | Rule                                                        |
| --------------------------- | ----------------------------------------------------------- |
| `type` is `"internal"`      | Reject if any other value                                   |
| All required fields present | No field may be null or empty                               |
| At least two options        | `options` array must contain ≥ 2 entries                    |
| CR ID is unique             | No existing file in `spec/change-requests/` has the same ID |

---

## Error Codes

| Code                | Meaning                                                   |
| ------------------- | --------------------------------------------------------- |
| `ERR_WRONG_TYPE`    | `type` is not `"internal"`                                |
| `ERR_MISSING_FIELD` | A required field is null or empty                         |
| `ERR_DUPLICATE_ID`  | Generated CR ID already exists in `spec/change-requests/` |
