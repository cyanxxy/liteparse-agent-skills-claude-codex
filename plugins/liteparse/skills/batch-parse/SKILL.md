---
name: batch-parse
description: Use when parsing every supported file in a folder with LiteParse.
---

# Batch Parse

Parse every supported file in a directory.

## Workflow

1. Resolve input and output directories. If output is omitted, use `<input-dir>-liteparse-output`.
2. Ask before reusing a non-empty output directory unless the user supplied it explicitly.
3. Check likely dependencies: LibreOffice for Office files, ImageMagick for images.
4. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse@latest batch-parse`.
5. Run `batch-parse <input-dir> <output-dir>` with requested format, recursion, extension, OCR, DPI, max pages, password, workers, and quiet flags. Use `TESSDATA_PREFIX` for local OCR data unless help confirms `--tessdata-path`.
6. Report paths, filters, success/failure counts, and exact errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
