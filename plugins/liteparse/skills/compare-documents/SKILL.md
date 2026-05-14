---
name: compare-documents
description: Use when comparing two documents with LiteParse.
---

# Compare Documents

Parse two files to text and diff them.

## Workflow

1. Resolve both paths; ask only when fewer than two were provided.
2. Check dependencies for each input type.
3. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse`.
4. Parse both files with `parse --format text` into a temp directory.
5. Run `diff -u`; write to `-o <path>` if supplied.
6. Report files, identical/different status, diff or output path, concise change summary, and exact parse errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
