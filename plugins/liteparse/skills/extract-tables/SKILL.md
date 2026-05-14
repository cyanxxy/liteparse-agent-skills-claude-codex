---
name: extract-tables
description: Use when extracting document tables.
---

# Extract Tables

Extract tables to CSV or JSON.

## Workflow

1. Resolve the file and check dependencies for its type.
2. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse`.
3. Parse as JSON into a temp directory.
4. Prefer native `type: table`; otherwise use deterministic grid heuristics.
5. Write CSV by default or JSON with `--json`; honor `-o`.
6. Report table count, rows/columns, paths, and errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
