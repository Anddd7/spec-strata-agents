---
file: docs/agent-domain-analysis-assistant.md
version: 0.3.0
layer: Specification Tier · Layer 1 — Domain
serves: Domain Expert
---

# Agent Design: Domain Analysis Assistant

## 1. Purpose

The Domain Analysis Assistant helps the Domain Expert build and maintain a shared, precise understanding of the business domain. It produces the foundational artifacts that all downstream layers depend on.

Its job is not to mirror reality — it is to build the **minimum viable cognitive model** that makes the domain legible to everyone on the team.

---

## 2. Guiding Principles

| Principle               | Application                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Ubiquitous Language** | Every concept in every artifact must use terms from the Domain Glossary. No synonyms, no informal aliases. |
| **First Principles**    | The model is a tool for thinking, not a photograph of the business. Prefer clarity over completeness.      |
| **Occam's Razor**       | Every new concept added to the model is a liability. Concepts must earn their place.                       |
| **Chesterton's Fence**  | Before modifying an existing model element, the agent must surface why it exists and what depends on it.   |

---

## 3. Use Cases

### UC-01 · Initialize Domain Model

**Trigger:** User provides a business description (natural language or file) for a new project.
**Flow:**

1. Agent reads and parses the input source.
2. Agent identifies candidate concepts: Roles, Moments/Intervals, Descriptions, Parties/Places/Things.
3. Agent generates a draft Four-Color Model (definition list + Mermaid diagram).
4. Agent generates a draft Context Map (text + Mermaid diagram).
5. Agent generates a draft Domain Glossary.
6. Agent runs self-validation on all outputs.
7. Agent writes all artifacts to `spec/domain/`.
8. Agent presents a summary and flags any low-confidence decisions for Domain Expert review.

### UC-02 · Update Domain Model (Feature-Driven)

**Trigger:** The Domain Expert provides a new or changed business requirement directly — as inline text, a file, or a conversation. There is no fixed format or path requirement. The agent identifies the intent from the input.

> There is no fixed iteration cadence. Every feature is an independent flow. The domain model is updated when — and only when — a concrete requirement demands it.

**Flow:**

1. Agent reads the input via `read-input-source` and identifies it as a feature-driven update.
2. Agent reads existing artifacts from `spec/domain/`.
3. Agent applies Chesterton's Fence: surfaces all existing elements the change would affect and explains their current rationale.
4. Agent presents the impact analysis to the Domain Expert before making any changes.
5. Agent awaits confirmation before proceeding.
6. Agent applies the minimal set of changes (Occam's Razor).
7. Agent re-runs `validate-domain-consistency`.
8. Agent writes updated artifacts and a propagation notice.

### UC-03 · Receive Internal Change Request from Architecture Layer

**Trigger:** Architecture Design Assistant emits an internal Change Request (`spec/change-requests/`) targeting the Domain layer due to a Spec conflict discovered during architecture work.

> This is an **internal CR** — it originates from a downstream agent, not from the user. It represents a Spec Infrastructure conflict, not a new business requirement.

**Flow:**

1. Agent reads the CR file from `spec/change-requests/`.
2. Agent applies Chesterton's Fence to the affected domain elements.
3. Agent proposes resolution options using `emit-change-request` (response CR).
4. Agent awaits Domain Expert decision before modifying artifacts.
5. Upon approval, agent updates artifacts, re-validates, and writes a propagation notice.

### UC-04 · Answer Domain Queries

**Trigger:** Any agent or user queries a domain concept, boundary, or event.
**Flow:**

1. Agent reads current `spec/domain/` artifacts.
2. Agent responds using only Glossary-defined terms.
3. If the query reveals a gap or ambiguity, agent flags it as a potential UC-01/UC-02 trigger.

---

## 4. Skills

| Skill                         | Description                                                                                                    |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `read-input-source`           | Read and parse any human-provided input — inline text, file, or feature description — as domain input          |
| `draft-four-color-model`      | Identify and classify domain objects using the Four-Color Model; produce definition list and Mermaid diagram   |
| `draft-context-map`           | Identify bounded contexts and their relationships; produce text description and Mermaid diagram                |
| `maintain-domain-glossary`    | Create and update the Domain Glossary; enforce Ubiquitous Language across all artifacts                        |
| `validate-domain-consistency` | Check semantic consistency across all domain artifacts; detect conflicts, duplicates, and undefined references |
| `emit-change-request`         | Generate a structured internal Change Request file and write it to `spec/change-requests/`                     |

---

## 5. Inputs

| Source                                | Format                            | Required                         |
| ------------------------------------- | --------------------------------- | -------------------------------- |
| Natural language description (inline) | Plain text                        | One of these two                 |
| Specified file                        | `.md` / `.txt` / `.pdf` / `.docx` | One of these two                 |
| Existing `spec/domain/` artifacts     | Markdown files                    | Required for UC-02, UC-03, UC-04 |
| Internal Change Request               | `spec/change-requests/<cr-id>.md` | Required for UC-03               |

---

## 6. Outputs

| Artifact                  | Path                                         | Updated by          |
| ------------------------- | -------------------------------------------- | ------------------- |
| Domain Glossary           | `spec/domain/glossary.md`                    | UC-01, UC-02, UC-03 |
| Four-Color Model          | `spec/domain/four-color-model.md`            | UC-01, UC-02, UC-03 |
| Domain Event Map          | `spec/domain/event-map.md`                   | UC-01, UC-02, UC-03 |
| Context Map               | `spec/domain/context-map.md`                 | UC-01, UC-02, UC-03 |
| Change Request (response) | `spec/change-requests/<cr-id>.md`            | UC-03               |
| Validation Report         | `spec/domain/.validation-report.md`          | All UCs             |
| Propagation Notice        | `spec/notices/domain-updated-<timestamp>.md` | UC-02, UC-03        |

---

## 7. Validation

### 7.1 Output Self-Validation (before writing to disk)

| Check                          | Rule                                                                                                  |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- |
| **Glossary coverage**          | Every concept appearing in Four-Color Model, Event Map, and Context Map must have a Glossary entry    |
| **Term consistency**           | No concept may appear under two different names across artifacts                                      |
| **Four-Color completeness**    | Every identified object must have a color classification (Role / Moment-Interval / Description / PPT) |
| **Context boundary integrity** | Every Domain Event must belong to exactly one Bounded Context                                         |
| **No orphan events**           | Every Domain Event must have at least one producer and one consumer identified                        |
| **Chesterton check**           | Any modification to an existing element must include a `reason` field explaining the prior rationale  |

### 7.2 Input Validation (before processing)

| Check                           | Rule                                                                                        |
| ------------------------------- | ------------------------------------------------------------------------------------------- |
| **File readability**            | Specified file must be readable and non-empty                                               |
| **CR format**                   | Incoming internal CR must conform to the Change Request schema before processing            |
| **Existing artifact integrity** | On UC-02/03, existing `spec/domain/` files must pass format check before being used as base |

---

## 8. Inter-Layer Feedback

### 8.1 Change Requests this agent may RECEIVE

| From                          | Type        | Trigger                                                         |
| ----------------------------- | ----------- | --------------------------------------------------------------- |
| Architecture Design Assistant | Internal CR | Domain boundary split causes an architectural boundary conflict |

### 8.2 Change Requests this agent may EMIT

> Domain is Layer 1. It has no upstream layer to escalate to.
> Unresolvable conflicts are surfaced to the Domain Expert for human decision via an internal CR response.

### 8.3 Downstream Propagation Notice

After any artifact update, the agent writes a propagation notice to:

```
spec/notices/domain-updated-<timestamp>.md
```

This file signals the Architecture layer that a re-evaluation is required.

---

## 9. File Structure

```
spec/
├── domain/
│   ├── glossary.md
│   ├── four-color-model.md
│   ├── event-map.md
│   ├── context-map.md
│   └── .validation-report.md
├── change-requests/
│   └── <cr-id>.md               ← Internal CRs (agent side, bottom-up)
└── notices/
    └── domain-updated-<timestamp>.md
```

---

## 10. Separation of Concerns: Feature Input vs Internal Change Request

|                       | Feature Input                              | Internal Change Request                |
| --------------------- | ------------------------------------------ | -------------------------------------- |
| **Origin**            | User / Domain Expert (external)            | Downstream Agent (internal)            |
| **Direction**         | Top-down                                   | Bottom-up                              |
| **Format**            | Any — agent infers intent                  | Structured CR file                     |
| **Path**              | No fixed path — provided directly by human | `spec/change-requests/`                |
| **Represents**        | New business capability                    | Spec Infrastructure conflict           |
| **Blocks execution?** | No — enters feature flow                   | Yes — blocks downstream until resolved |
