---
name: parse-document
description: Parse a single PDF, DOCX, XLSX, PPTX, image, or scanned file locally with LiteParse. Use when extracting text or JSON from one document, pulling specific pages out of a file, or running OCR on a scan.
compatibility: Requires Node 20.11+ and `@llamaindex/liteparse`. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Parse Document

Parse a single file with LiteParse.

## Steps

1. **Resolve the file path**. If missing or ambiguous, ask the user.
2. **Check file-type dependencies**:
   - Office files (`.docx` `.xlsx` `.pptx` `.odt` `.rtf` `.csv` `.tsv`): run `which libreoffice`. If absent, stop.
   - Image files (`.png` `.jpg` `.jpeg` `.tif` `.tiff` `.webp` `.gif` `.bmp` `.svg`): run `which magick || which convert`. If absent, stop.
   - PDFs: no extra dependency.
3. **Choose the CLI**: run `which lit`. If it exists, use `lit parse`. Otherwise, `npx -y @llamaindex/liteparse parse`.
4. **Run the parse** with flags: `--format json|text`, `-o <file>`, `--target-pages`, `--no-ocr`, `--ocr-server-url`, `--ocr-language`, `--dpi`, `--password`, `--config`, `-q`.
5. **Post-parse hooks**: if a `liteparse.config.json` exists in the project root and defines `hooks.postParse`, execute each hook command after a successful parse, substituting `{{file}}` with the input path and `{{output}}` with the output path.
6. **Default output placement**: if JSON was requested and no `-o` was given, write next to the source file with a `.liteparse.json` suffix.
7. **Report**: file parsed, flags used, output path or stdout preview, any errors verbatim.

## Examples

```bash
lit parse ./docs/report.pdf --format json
lit parse ./contracts/master.docx --no-ocr
lit parse ./scans/invoice.pdf --ocr-server-url http://localhost:5000/ocr --target-pages "1-2"
lit parse ./slides.pptx --format text -o slides.txt
```
