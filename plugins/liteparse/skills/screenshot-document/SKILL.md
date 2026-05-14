---
name: screenshot-document
description: Use when rendering PDF pages as images.
---

# Screenshot Document

Render PDF pages to PNG or JPG.

## Workflow

1. Resolve the PDF path and confirm it is a PDF.
2. Use `<pdf-basename>-screenshots` unless the user supplied an output directory.
3. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse screenshot`.
4. Run `screenshot <file> -o <output-dir>` with requested page, DPI, format, password, config, and quiet flags.
5. Report source, pages, output directory, format, DPI, count, and exact errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
