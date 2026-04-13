---
name: screenshot-document
description: Render PDF pages as PNG or JPG images with LiteParse. Use when exporting selected pages as screenshots, generating image previews for visual inspection, or producing page renders for agent reasoning.
compatibility: Requires Node 20.11+ and `@llamaindex/liteparse`.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Screenshot Document

Generate page screenshots from a PDF.

## Steps

1. **Resolve file path**. If missing, ask for the PDF path.
2. **Verify it is a PDF**. If not `.pdf`, stop and suggest `/parse-document` to convert first.
3. **Resolve output directory**. If omitted, default to `<pdf-basename>-screenshots`. Create with `mkdir -p`.
4. **Parse flags**: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--format png|jpg`, `--password`, `--config`, `-q`.
5. **Choose the CLI**: `which lit` → `lit screenshot <file> -o <dir>`, otherwise `npx -y @llamaindex/liteparse screenshot <file> -o <dir>`. **Always pass output via `-o`**.
6. **Report**: source file, page selection, output directory, image format/DPI, screenshot count, or errors.

## Examples

```bash
lit screenshot ./docs/report.pdf -o ./report-pages
lit screenshot ./docs/report.pdf -o ./report-pages --target-pages "1-3" --dpi 300
lit screenshot ./docs/report.pdf -o ./out --format jpg
```
