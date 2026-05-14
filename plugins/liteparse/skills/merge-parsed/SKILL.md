---
name: merge-parsed
description: Use when merging LiteParse outputs.
---

# Merge Parsed

Combine parsed text or JSON files.

## Workflow

1. Resolve files, glob, or batch output directory; ask only if no inputs were provided.
2. Require all inputs to be text or all JSON.
3. Sort naturally by filename.
4. Concatenate text with filename separators, or wrap JSON in a versioned document envelope.
5. Write `-o <path>` or default to `merged-output.txt`/`.json`.
6. Report file count, size, output path, and skipped files.

More details: [workflow](references/workflow.md).
