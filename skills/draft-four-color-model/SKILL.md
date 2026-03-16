---
name: draft-four-color-model
description: Classify domain concepts using the Four-Color Model and output structured definitions plus Mermaid class diagrams. Use for initial domain modeling and iterative domain updates.
---

# Skill: draft-four-color-model

**Version:** 0.1.0

---

## Purpose

Analyze a normalized domain description and classify all identified domain objects using the Four-Color Model. Produce a structured definition list and a Mermaid class diagram.

---

## Interface

### Input

| Parameter        | Type     | Required | Description                                            |
| ---------------- | -------- | -------- | ------------------------------------------------------ |
| `domain_text`    | `string` | Yes      | Normalized domain description from `read-input-source` |
| `existing_model` | `object` | No       | Existing Four-Color Model for update scenarios         |

### Output

```json
{
  "status": "ok" | "error",
  "objects": [
    {
      "name": "<PascalCase term>",
      "color": "MI" | "Role" | "Desc" | "PPT",
      "definition": "<one sentence>",
      "confidence": "high" | "medium" | "low",
      "confidence_note": "<reason if medium or low>"
    }
  ],
  "mermaid_diagram": "<mermaid classDiagram string>",
  "error": "<error message or null>"
}
```

---

## Four-Color Classification Reference

| Color     | Label                       | Symbol     | Identifies                                                                        |
| --------- | --------------------------- | ---------- | --------------------------------------------------------------------------------- |
| 🔴 Red    | Moment-Interval (MI)        | `<<MI>>`   | Events, transactions, process steps occurring at a point in time or over a period |
| 🟡 Yellow | Role                        | `<<Role>>` | How a party/place/thing participates in a moment-interval                         |
| 🔵 Blue   | Description (Desc)          | `<<Desc>>` | Catalog entries, specifications, types, or rules that describe other objects      |
| 🟢 Green  | Party / Place / Thing (PPT) | `<<PPT>>`  | Real-world entities that play roles                                               |

---

## Process

1. **Extract candidate concepts** from the domain text: nouns, noun phrases, and named processes.
2. **Classify each concept** using the Four-Color reference above.
   - If a concept could belong to two colors, choose the primary one and note the ambiguity in `confidence_note`.
   - Apply Occam's Razor: if two candidate concepts represent the same thing, unify them into one entry.
3. **Identify relationships** between objects:
   - MI → Role (a moment-interval involves roles)
   - Role → PPT (a PPT plays a role)
   - MI → Desc (a moment-interval follows a description/rule)
4. **Generate Mermaid diagram** using `classDiagram` syntax. Use stereotypes to indicate color: `<<MI>>`, `<<Role>>`, `<<Desc>>`, `<<PPT>>`.
5. **Assign confidence** to each object:
   - `high`: clearly identifiable from the text
   - `medium`: inferred from context, should be reviewed
   - `low`: assumed to exist but not mentioned — flag for Domain Expert

---

## Mermaid Output Format

```
classDiagram
  class Order {
    <<MI>>
  }
  class Customer {
    <<Role>>
  }
  class Person {
    <<PPT>>
  }
  class ProductSpec {
    <<Desc>>
  }
  Order --> Customer : involves
  Customer --> Person : played by
  Order --> ProductSpec : follows
```

---

## Validation

| Check                      | Rule                                                                      |
| -------------------------- | ------------------------------------------------------------------------- |
| No unclassified objects    | Every extracted concept must have exactly one color                       |
| No duplicate names         | Object names must be unique (case-insensitive)                            |
| Relationship targets exist | Every relationship must reference objects present in the `objects` list   |
| Low-confidence flagged     | Any object with `confidence: low` must have a non-empty `confidence_note` |
