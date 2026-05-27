---
name: parse-document
description: Use when parsing one document with LiteParse to text or JSON.
---

# Parse Document

Parse one local file to text or JSON.

## Workflow

1. Resolve the file path; ask only if missing or ambiguous.
2. Check dependencies: LibreOffice for Office files, ImageMagick for images, none for PDFs.
3. Use `lit` or `liteparse`; otherwise run `npx -y @llamaindex/liteparse parse`.
4. Preserve user flags such as `--format`, `-o`, pages, OCR, DPI, max pages, password, workers, and quiet mode. Use `TESSDATA_PREFIX` for local OCR data unless help confirms `--tessdata-path`.
5. For JSON without `-o`, write `<source>.liteparse.json`. For text stdout, preview at most 50 lines or 4,000 characters.
6. Report input, flags, output or preview, and exact errors.

More details: [workflow](references/workflow.md), [CLI reference](../liteparse/references/cli-reference.md).
