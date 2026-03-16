---
name: read-input-source
description: Read and normalize human-provided input from inline text or files for any layer workflow. Use when processing requirements, stories, notices, release scopes, or change requests.
---

# Skill: read-input-source

**Version:** 0.2.0

---

## Purpose

Read and normalize human-provided input across all layers — requirements, stories, notices, release scopes, and change requests — producing a consistent plain-text representation for downstream skills.

No special format is required from the human. The agent infers input type and intent from the content itself.

---

## Interface

### Input

| Parameter              | Type     | Required    | Description                                                                                    |
| ---------------------- | -------- | ----------- | ---------------------------------------------------------------------------------------------- |
| `inline_text`          | `string` | Conditional | Natural language input provided directly by the user                                           |
| `file_path`            | `string` | Conditional | Path to a `.md`, `.txt`, `.pdf`, or `.docx` file                                               |
| `context_layer`        | `string` | No          | Layer hint (`domain`, `architecture`, `process`, `dev`, `qa`) used to improve intent inference |
| `expected_input_types` | `array`  | No          | Optional allowed types (`requirement`, `story`, `notice`, `change-request`, `release-scope`)   |

At least one of `inline_text` or `file_path` must be provided. If both are provided, merge them with `file_path` content taking precedence, and note the merge in the output metadata.

### Output

```json
{
  "status": "ok" | "error",
  "source": "inline" | "file" | "merged",
  "file_path": "<path or null>",
   "input_type": "requirement" | "story" | "notice" | "change-request" | "release-scope" | "unknown",
   "inferred_intent": "initialization" | "update" | "validation" | "query" | "unknown",
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

3. **Infer input type and intent:**
   - Infer `input_type` from structure and keywords (Story ID, CR format, propagation notice marker, release scope wording).
   - Infer `inferred_intent` from requested operation (`initialization`, `update`, `validation`, `query`).
   - If `expected_input_types` is provided and inferred type is outside the allowed set, set `status: error`.
   - If type or intent cannot be determined confidently, set corresponding value to `unknown` and ask the user to clarify.

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

| Code                        | Meaning                                              |
| --------------------------- | ---------------------------------------------------- |
| `ERR_NO_INPUT`              | Neither inline text nor file path was provided       |
| `ERR_FILE_NOT_FOUND`        | Specified file path does not exist                   |
| `ERR_FILE_EMPTY`            | File exists but contains no readable content         |
| `ERR_FORMAT_UNSUPPORTED`    | File format is not `.md`, `.txt`, `.pdf`, or `.docx` |
| `ERR_UNEXPECTED_INPUT_TYPE` | Inferred input type is not in `expected_input_types` |
