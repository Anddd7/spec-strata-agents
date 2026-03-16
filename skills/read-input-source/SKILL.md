---
name: read-input-source
description: Read and normalize human-provided domain input from inline text or files for downstream modeling tasks. Use when domain analysis starts from free-form requirements or documents.
---

# Skill: read-input-source

**Version:** 0.2.0

---

## Purpose

Read and normalize any human-provided domain input — whether an initial business description, a new feature requirement, or a business change — producing a consistent plain-text representation for downstream skills.

No special format is required from the human. The agent infers the intent (initialization vs. feature-driven update) from the content itself.

---

## Interface

### Input

| Parameter     | Type     | Required    | Description                                                                                                           |
| ------------- | -------- | ----------- | --------------------------------------------------------------------------------------------------------------------- |
| `inline_text` | `string` | Conditional | Natural language provided directly by the user — may be a domain description, a feature request, or a business change |
| `file_path`   | `string` | Conditional | Path to a `.md`, `.txt`, `.pdf`, or `.docx` file containing domain or feature content                                 |

At least one of `inline_text` or `file_path` must be provided. If both are provided, merge them with `file_path` content taking precedence, and note the merge in the output metadata.

### Output

```json
{
  "status": "ok" | "error",
  "source": "inline" | "file" | "merged",
  "file_path": "<path or null>",
  "inferred_intent": "initialization" | "feature-update" | "unknown",
  "content": "<normalized plain text>",
  "error": "<error message or null>"
}
```

---

## Process

1. **Determine source:**
   - If `file_path` is provided, attempt to read the file.
   - If the file cannot be read (not found, unsupported format, empty), set `status: error` and return immediately.
   - If `inline_text` is also provided alongside a valid file, merge both and set `source: merged`.

2. **Normalize content:**
   - Strip formatting artifacts (e.g. PDF layout characters, DOCX XML tags).
   - Preserve paragraph structure using blank lines.
   - Do not summarize or interpret — output the raw content faithfully.

3. **Infer intent:**
   - If no existing `spec/domain/` artifacts are present, set `inferred_intent: initialization`.
   - If existing artifacts are present and the input describes a new capability, change, or addition, set `inferred_intent: feature-update`.
   - If intent cannot be determined from content alone, set `inferred_intent: unknown` — the agent will ask the user to clarify before proceeding.

4. **Return output object.**

---

## Validation

| Check                       | Rule                                                                   |
| --------------------------- | ---------------------------------------------------------------------- |
| At least one input provided | If both `inline_text` and `file_path` are null, return `status: error` |
| File is readable            | File exists, is non-empty, and is a supported format                   |
| Content is non-empty        | Normalized content must contain at least one non-whitespace character  |

---

## Error Codes

| Code                     | Meaning                                              |
| ------------------------ | ---------------------------------------------------- |
| `ERR_NO_INPUT`           | Neither inline text nor file path was provided       |
| `ERR_FILE_NOT_FOUND`     | Specified file path does not exist                   |
| `ERR_FILE_EMPTY`         | File exists but contains no readable content         |
| `ERR_FORMAT_UNSUPPORTED` | File format is not `.md`, `.txt`, `.pdf`, or `.docx` |
