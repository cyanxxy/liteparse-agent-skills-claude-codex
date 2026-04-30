---
name: screenshot-document
description: Render PDF pages as PNG or JPG images with LiteParse. Use when exporting selected pages as screenshots, generating image previews for visual inspection, or producing page renders for agent reasoning.
compatibility: Requires Node 18+ and either an installed `lit`/`liteparse` binary or `npm` for the `npx` fallback.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Screenshot Document

Generate page screenshots from a PDF.

## Steps

1. **Resolve the PDF path** relative to the project root. If missing, ask for the PDF path. If no arguments were passed, ask for a file.
2. **Verify it is a PDF**. If the extension is not `.pdf`, tell the user `lit screenshot` only supports PDF input and stop. For other formats, suggest running the `convert-format` skill first (e.g. `docx → pdf`, `pptx → pdf`), then screenshot the resulting PDF.
3. **Resolve the output directory**. The upstream CLI default is `./screenshots` (relative to the current working directory). For clarity when running on multiple PDFs, prefer `<pdf-basename>-screenshots` and create it with `mkdir -p` if it does not exist.
4. **Parse extra flags**: `--target-pages "1,3,5"` or `"1-5"`, `--dpi <n>`, `--format png|jpg`, `--password <pw>`, `--config <file>`, `-q`.
5. **Choose the CLI**: run `which lit || which liteparse`. If either exists, use that binary as `<cli>` and run `<cli> screenshot <file> -o <output-dir> <flags>`. Otherwise, fall back to `npx -y @llamaindex/liteparse screenshot <file> -o <output-dir> <flags>` (no `lit` prefix under npx). **Always pass the output directory via `-o`** — it is not a positional argument.
6. **Config files**: if the user passed `--config <file>`, report which config file was used. Do not inspect or execute `hooks.*` entries as part of this workflow.
7. **Report**:
   - source file,
   - page selection (default: all pages),
   - output directory,
   - image format and DPI if specified,
   - number of screenshots written,
   - or the exact CLI error on failure.

## Examples

```bash
lit screenshot ./docs/report.pdf -o ./report-pages
lit screenshot ./docs/report.pdf -o ./report-pages --target-pages "1-3" --dpi 300
lit screenshot ./docs/report.pdf -o ./out --format jpg
lit screenshot ./encrypted.pdf -o ./out --password secret   # caution: password visible in ps/history; prefer --config
```

For CLI flag details, see the background `liteparse` skill.
