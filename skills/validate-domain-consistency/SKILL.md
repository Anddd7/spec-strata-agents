---
name: validate-domain-consistency
description: Run blocking semantic consistency checks across domain artifacts before write operations. Use as the quality gate for initialization and update workflows in the domain layer.
---

# Skill: validate-domain-consistency

**Version:** 0.1.0

---

## Purpose

Perform semantic consistency validation across all domain artifacts before they are written to disk. This skill is the quality gate for the Domain layer.

---

## Interface

### Input

| Parameter          | Type      | Required | Description                                       |
| ------------------ | --------- | -------- | ------------------------------------------------- |
| `glossary`         | `object`  | Yes      | Output from `maintain-domain-glossary`            |
| `four_color_model` | `object`  | Yes      | Output from `draft-four-color-model`              |
| `context_map`      | `object`  | Yes      | Output from `draft-context-map`                   |
| `event_map`        | `object`  | Yes      | Domain Event Map (producer/consumer pairs)        |
| `is_update`        | `boolean` | Yes      | `true` if this is an update to existing artifacts |

### Output

```json
{
  "status": "pass" | "fail",
  "checks": [
    {
      "name": "<check name>",
      "result": "pass" | "fail",
      "detail": "<description of failure or null>"
    }
  ],
  "blocking": true | false,
  "markdown_report": "<full validation report markdown>"
}
```

`blocking: true` means artifacts must NOT be written until failures are resolved.
All failures are blocking by default. There are no warnings — only pass or fail.

---

## Checks

| #   | Check Name                   | Rule                                                                                                |
| --- | ---------------------------- | --------------------------------------------------------------------------------------------------- |
| 1   | `glossary-coverage`          | Every concept name in the Four-Color Model and Context Map must have a corresponding Glossary entry |
| 2   | `term-consistency`           | No concept may appear under two different names across any artifact                                 |
| 3   | `four-color-completeness`    | Every object in the Four-Color Model has exactly one color classification                           |
| 4   | `context-boundary-integrity` | Every Domain Event belongs to exactly one Bounded Context                                           |
| 5   | `no-orphan-events`           | Every Domain Event has at least one identified producer and one identified consumer                 |
| 6   | `chesterton-check`           | If `is_update: true`, every modified or removed element must have a non-null `change_reason`        |
| 7   | `no-cross-context-ownership` | No domain object is claimed as `owned_concept` by more than one context                             |

---

## Markdown Report Format

```markdown
## Domain Validation Report

**Timestamp:** <ISO 8601>
**Mode:** initialization | update

### ✅ PASS

- glossary-coverage
- four-color-completeness

### ❌ FAIL

- **term-consistency:** "Buyer" and "Customer" appear to refer to the same concept across artifacts. Glossary has "Customer"; Four-Color Model uses "Buyer" in two places.
- **no-orphan-events:** "PaymentReceived" has no identified consumer.

### Action Required

The above failures are blocking. Artifacts will not be written until resolved.
Please clarify:

1. Should "Buyer" be replaced with "Customer" everywhere?
2. Which context consumes the "PaymentReceived" event?
```
