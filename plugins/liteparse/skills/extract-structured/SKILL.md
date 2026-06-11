---
name: extract-structured
description: Use when extracting named fields from documents into JSON, JSONL, or CSV.
---

# Extract Structured

Extract requested fields to JSON, JSONL, or CSV.

## Workflow

1. Resolve the file and check dependencies for its type.
2. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse@latest`.
3. Resolve schema in order: `--schema`, `--fields`, then ask. Normalize loose fields to stable `snake_case`.
4. Parse as JSON into a temp directory, extract values with evidence, and never guess missing values.
5. Write JSON by default; honor JSONL/CSV, `-o`, and `--save-schema`.
6. Report schema source, output, missing/ambiguous fields, and errors.

More details: [workflow](references/workflow.md), [sample schema](../../examples/invoice.extract.json), [CLI reference](../liteparse/references/cli-reference.md).
