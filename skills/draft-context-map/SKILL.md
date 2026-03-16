---
name: draft-context-map
description: Identify bounded contexts and DDD relationship patterns from domain artifacts and generate context-map text plus Mermaid graph output. Use when defining or refining context boundaries.
---

# Skill: draft-context-map

**Version:** 0.1.0

---

## Purpose

Identify bounded contexts from the domain model and map their relationships using standard DDD context mapping patterns. Produce a text description and a Mermaid diagram.

---

## Interface

### Input

| Parameter              | Type     | Required | Description                               |
| ---------------------- | -------- | -------- | ----------------------------------------- |
| `four_color_model`     | `object` | Yes      | Output from `draft-four-color-model`      |
| `domain_text`          | `string` | Yes      | Original normalized domain description    |
| `existing_context_map` | `object` | No       | Existing Context Map for update scenarios |

### Output

```json
{
  "status": "ok" | "error",
  "contexts": [
    {
      "name": "<PascalCase context name>",
      "responsibility": "<one sentence>",
      "owned_concepts": ["<term>", "..."],
      "owned_events": ["<event>", "..."]
    }
  ],
  "relationships": [
    {
      "from": "<context name>",
      "to": "<context name>",
      "pattern": "U/D" | "SK" | "CS" | "CF" | "ACL" | "OHS" | "PL" | "SW",
      "description": "<one sentence explaining the relationship>"
    }
  ],
  "mermaid_diagram": "<mermaid graph string>",
  "error": "<error message or null>"
}
```

---

## Context Mapping Patterns Reference

| Pattern               | Symbol | Meaning                                                 |
| --------------------- | ------ | ------------------------------------------------------- |
| Upstream / Downstream | `U/D`  | One context produces, the other consumes                |
| Shared Kernel         | `SK`   | Two contexts share a subset of the model                |
| Customer / Supplier   | `CS`   | Downstream negotiates requirements with upstream        |
| Conformist            | `CF`   | Downstream adopts upstream model as-is                  |
| Anti-Corruption Layer | `ACL`  | Downstream translates upstream model to protect its own |
| Open Host Service     | `OHS`  | Upstream publishes a formal protocol for integration    |
| Published Language    | `PL`   | A shared, well-documented exchange format               |
| Separate Ways         | `SW`   | Contexts have no integration                            |

---

## Process

1. **Group domain objects** from the Four-Color Model into candidate bounded contexts based on:
   - Linguistic cohesion (objects that share the same Ubiquitous Language cluster)
   - Responsibility cohesion (objects that change together for the same business reason)
   - Team/organizational boundaries (if mentioned in domain text)
2. **Apply Occam's Razor:** prefer fewer, larger contexts over many small ones unless there is a clear reason to split.
3. **Identify relationships** between contexts by tracing Domain Events that cross context boundaries.
4. **Assign a pattern** to each relationship using the reference above.
5. **Generate Mermaid diagram** using `graph LR` syntax.

---

## Mermaid Output Format

```
graph LR
  OrderCtx["Order Context\n(upstream)"]
  InventoryCtx["Inventory Context\n(downstream)"]
  PaymentCtx["Payment Context"]

  OrderCtx -->|"U/D · OHS"| InventoryCtx
  OrderCtx -->|"CS"| PaymentCtx
```

---

## Validation

| Check                                                 | Rule                                                   |
| ----------------------------------------------------- | ------------------------------------------------------ |
| Every context has a name and responsibility           | No empty fields                                        |
| Every owned concept exists in the Glossary            | Cross-reference with `maintain-domain-glossary` output |
| Every Domain Event is assigned to exactly one context | No orphan events, no duplicate ownership               |
| Every relationship has a valid pattern                | Must be one of the eight standard patterns             |
| No self-referencing relationships                     | `from` ≠ `to`                                          |
