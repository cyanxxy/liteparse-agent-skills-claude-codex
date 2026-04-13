---
name: liteparse
description: Parse PDFs, DOCX, XLSX, PPTX, images, or scans locally with LiteParse. Use when extracting text or JSON, running OCR on scanned documents, batch parsing a folder, or generating PDF page screenshots.
---

# LiteParse

Local document parsing with no cloud dependency. Handles PDFs, Office files, images, and scans.

## Choosing the CLI

1. Check for an installed binary: `which lit`.
2. If `lit` exists, use it: `lit parse <file>`, `lit batch-parse <in> <out>`, `lit screenshot <file> -o <dir>`.
3. If `lit` is missing but `npm` exists, fall back to npx and **drop the `lit` prefix** — the subcommand comes first:
   ```bash
   npx -y @llamaindex/liteparse parse file.pdf --format json
   npx -y @llamaindex/liteparse batch-parse ./in ./out --recursive
   npx -y @llamaindex/liteparse screenshot file.pdf -o ./out --target-pages "1-3"
   ```
4. If `npm` is also missing, tell the user Node.js 20.11+ and npm are required.

## Dependency rules by file type

- **PDFs**: work out of the box.
- **DOCX, XLSX, PPTX, ODT, RTF, CSV, TSV**: need LibreOffice. Verify with `which libreoffice`.
- **PNG, JPG, TIFF, WebP, GIF, BMP, SVG**: need ImageMagick. Verify with `which magick || which convert`.

If a required tool is missing, name the exact missing binary. Do not guess.

## Common commands

### Parse one file

```bash
lit parse document.pdf
lit parse document.pdf --format json -o output.json
lit parse document.pdf --target-pages "1-5" --no-ocr
lit parse document.pdf --ocr-server-url http://localhost:5000/ocr
```

### Batch parse

`lit batch-parse` requires **both** input and output directories.

```bash
lit batch-parse ./input ./output
lit batch-parse ./input ./output --recursive --extension ".pdf"
lit batch-parse ./input ./output --format json --no-ocr
```

### Screenshot pages

Output directory is passed via `-o`, not positionally.

```bash
lit screenshot document.pdf -o ./screenshots
lit screenshot document.pdf -o ./screenshots --target-pages "1,3,5" --dpi 300
lit screenshot document.pdf -o ./screenshots --format jpg
```

## Workflow

1. Confirm the input path, desired output format, and whether OCR is required.
2. Pick `lit` or `npx -y @llamaindex/liteparse` based on what is installed.
3. Verify any file-type dependency (LibreOffice / ImageMagick).
4. Run the smallest command that satisfies the request.
5. For large output or JSON, write to a file and report the path. For short text, print to stdout.
6. Report: the exact file(s) processed, the flags used, the output location, and any blocking error verbatim.

## Failure handling

- Missing `lit` and `npm` → Node.js 20.11+ and npm are required.
- Missing LibreOffice for Office input → ask the user to install LibreOffice.
- Missing ImageMagick for image input → ask the user to install ImageMagick (`magick` or `convert`).
- Unreachable `--ocr-server-url` → surface the URL and suggest dropping the flag to use built-in Tesseract.
- Password-protected PDF without `--password` → surface the CLI error verbatim.

## Config file

A starter config is in `examples/liteparse.config.json` at the plugin root. Use it with any command via `--config <file>` when the user wants consistent defaults.
