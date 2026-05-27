---
name: screenshot-document
description: Use when rendering document pages as PNG images with LiteParse.
---

# Screenshot Document

Render document pages to PNG.

## Workflow

1. Resolve the document path and check LibreOffice/ImageMagick dependencies when needed.
2. Use `<basename>-screenshots` unless the user supplied an output directory.
3. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse screenshot`.
4. Run `screenshot <file> -o <output-dir>` with requested page, DPI, password, and quiet flags.
5. Report source, pages, output directory, PNG output, DPI, count, and exact errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
