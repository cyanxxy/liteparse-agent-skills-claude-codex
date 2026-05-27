---
name: convert-format
description: Use when converting Office, CSV, or HTML documents with LibreOffice.
---

# Convert Format

Convert a document without parsing it.

## Workflow

1. Resolve input and `--to <format>`; infer only obvious targets such as Office to PDF or CSV to XLSX.
2. Verify `libreoffice` exists.
3. Use `-o <path>` or write beside the input with the target extension.
4. Run `libreoffice --headless --convert-to <format> --outdir <dir> "<input-file>"`.
5. Rename LibreOffice's output for a custom `-o`, then verify it is non-empty.
6. Report input, output, formats, size, and exact errors.

More details: [workflow](references/workflow.md).
