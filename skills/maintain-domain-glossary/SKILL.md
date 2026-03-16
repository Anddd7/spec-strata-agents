---
name: maintain-domain-glossary
description: Create and update the canonical domain glossary and enforce ubiquitous language consistency across all domain artifacts. Use when introducing, updating, or deprecating domain terms.
---

# Skill: maintain-domain-glossary

**Version:** 0.1.0

---

## Purpose

Create and maintain the Domain Glossary — the authoritative list of all business terms used in the project. Enforce Ubiquitous Language across all domain artifacts.

---

## Interface

### Input

| Parameter           | Type     | Required | Description                                    |
| ------------------- | -------- | -------- | ---------------------------------------------- |
| `four_color_model`  | `object` | Yes      | Output from `draft-four-color-model`           |
| `context_map`       | `object` | Yes      | Output from `draft-context-map`                |
| `existing_glossary` | `object` | No       | Existing glossary entries for update scenarios |

### Output

```json
{
  "status": "ok" | "error",
  "entries": [
    {
      "term": "<Term>",
      "definition": "<one sentence, plain language>",
      "status": "active" | "deprecated",
      "replaces": "<term or null>",
      "change_reason": "<reason if updated or deprecated, null if new>"
    }
  ],
  "markdown": "<full glossary.md content>",
  "error": "<error message or null>"
}
```

---

## Process

1. **Collect all terms** from Four-Color Model objects and Context Map context names.
2. **For each term:**
   - If it does not exist in the glossary, create a new entry.
   - If it already exists and the definition is unchanged, keep as-is.
   - If it already exists and the definition would change, apply Chesterton's Fence: record the previous definition and the reason for the change in `change_reason`.
3. **Detect synonyms:** if two terms refer to the same concept, flag them. Propose keeping one and deprecating the other. Do not auto-resolve — present to Domain Expert.
4. **Sort entries alphabetically.**
5. **Generate markdown output.**

---

## Markdown Output Format

```markdown
# Domain Glossary

> Last updated: <YYYY-MM-DD>
> All terms in this glossary constitute the Ubiquitous Language of this project.
> No synonyms. No informal aliases.

---

**Customer** — A role played by a Person who has placed at least one Order.

**Order** — A moment-interval representing a customer's request to purchase one or more Products.

**ProductSpec** — A description defining the attributes, pricing rules, and availability of a product offering.

---

## Deprecated Terms

**Buyer** _(deprecated — use Customer)_ — Previously used to refer to the purchasing party. Replaced by Customer for consistency with the Order context ubiquitous language.
```

---

## Validation

| Check                               | Rule                                                                 |
| ----------------------------------- | -------------------------------------------------------------------- |
| No duplicate terms                  | Term names are unique (case-insensitive)                             |
| No empty definitions                | Every active entry has a non-empty definition                        |
| Deprecated terms have a replacement | `replaces` field must be set for deprecated entries                  |
| Changed terms have a reason         | `change_reason` must be non-null for any updated or deprecated entry |
