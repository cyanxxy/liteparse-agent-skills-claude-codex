---
name: liteparse
description: Background knowledge for running LiteParse: choosing between installed lit and the npx fallback, verifying LibreOffice or ImageMagick dependencies, CLI flag reference, and failure handling across parsing workflows.
user-invocable: false
allowed-tools: Bash(which *) Bash(lit *) Bash(npx *) Bash(npm *) Bash(libreoffice *) Bash(magick *) Bash(convert *)
---

# LiteParse Reference

LiteParse is a local, no-cloud document parser. This skill is background knowledge: it tells Claude how to run LiteParse correctly. For user-triggered actions, prefer the specific skills `/liteparse:parse-document`, `/liteparse:batch-parse`, `/liteparse:screenshot-document`.

## Choosing the CLI

1. Check for an installed binary first:
   ```bash
   which lit
   ```
2. If `lit` exists, use it directly: `lit parse <file>`, `lit batch-parse <in> <out>`, `lit screenshot <file> -o <dir>`.
3. If `lit` is missing but `npm` is available, fall back to npx. **Drop the `lit` prefix** — the subcommand comes first:
   ```bash
   npx -y @llamaindex/liteparse parse file.pdf --format json
   npx -y @llamaindex/liteparse batch-parse ./in ./out --recursive
   npx -y @llamaindex/liteparse screenshot file.pdf -o ./out --target-pages "1-3"
   ```
4. If `npm` is also missing, tell the user Node.js 18+ and npm are required.

## Dependency rules by file type

- **PDFs** work out of the box. Nothing else needed.
- **DOCX, XLSX, PPTX, ODT, RTF, CSV, TSV, Pages, Numbers, Key**: LiteParse converts them via LibreOffice. Verify `libreoffice` is on PATH before running: `which libreoffice`.
- **PNG, JPG, TIFF, WebP, GIF, BMP, SVG**: converted via ImageMagick. Verify `magick` or `convert` is on PATH: `which magick || which convert`.
- If a required tool is missing, report the exact missing binary name. Do not guess.

## Supported flags (upstream CLI, v1.4.x)

**`parse` / `batch-parse`:**
- `--format json|text` (default `text`)
- `-o, --output <file>` (parse only; batch-parse takes positional `<output-dir>`)
- `--no-ocr` to disable OCR
- `--ocr-server-url <url>` for external OCR
- `--ocr-language <lang>` (default `en`)
- `--num-workers <n>` OCR parallelism (defaults to CPU count − 1)
- `--max-pages <n>` (default 10000)
- `--target-pages "1-5,10"` (parse only)
- `--dpi <n>` (default 150)
- `--no-precise-bbox`
- `--preserve-small-text`
- `--password <pw>` for encrypted PDFs
- `--config <file>` JSON config
- `--recursive` (batch-parse)
- `--extension ".pdf"` (batch-parse)
- `-q, --quiet`

**`screenshot`:**
- `-o, --output-dir <dir>` (default `./screenshots`) — **always pass the output via `-o`**
- `--target-pages "1,3,5"` or `"1-5"`
- `--dpi <n>`
- `--format png|jpg` (default `png`)
- `--password <pw>`
- `--config <file>`
- `-q, --quiet`

## Config file

A sample config ships with this plugin at [`examples/liteparse.config.json`](../../examples/liteparse.config.json). Point any of the three commands at it with `--config <file>` when the user wants consistent defaults (OCR language, DPI, max pages, etc.).

## Failure handling

- Missing `lit` and missing `npm` → Node.js 18+ and npm are required.
- Missing `libreoffice` for Office input → install LibreOffice.
- Missing `magick`/`convert` for image input → install ImageMagick.
- Unreachable `--ocr-server-url` → surface the failing URL and suggest dropping the flag to fall back to built-in Tesseract.
- Password-protected PDF without `--password` → surface the CLI's error verbatim.
