---
name: parse-document
description: Parse a single PDF, DOCX, XLSX, PPTX, image, or scanned file locally with LiteParse. Use when extracting text or JSON from one document, pulling specific pages out of a file, or running OCR on a scan.
argument-hint: "<file>"
allowed-tools: Read Write Bash(which *) Bash(lit *) Bash(npx *) Bash(libreoffice *) Bash(magick *) Bash(convert *)
---

# Parse Document

Parse a single file at `$0` with LiteParse, applying any additional flags from `$ARGUMENTS`.

## Steps

1. **Resolve `$0`** relative to the project root. If it is missing or ambiguous, ask the user for the correct path. If the user passed no arguments at all, ask for a file path.
2. **Check file-type dependencies**:
   - Office files (`.docx` `.xlsx` `.pptx` `.odt` `.rtf` `.csv` `.tsv`): run `which libreoffice`. If absent, report it and stop.
   - Image files (`.png` `.jpg` `.jpeg` `.tif` `.tiff` `.webp` `.gif` `.bmp` `.svg`): run `which magick || which convert`. If neither exists, report it and stop.
   - PDFs: no extra dependency.
3. **Choose the CLI**: run `which lit`. If it succeeds, use `lit parse ...`. Otherwise, fall back to `npx -y @llamaindex/liteparse parse ...` (subcommand only — no `lit` prefix under npx).
4. **Run the parse** with the user's flags from `$ARGUMENTS`. Respect any of: `--format json|text`, `-o <file>`, `--target-pages "1-5"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>`, `--dpi <n>`, `--password <pw>`, `--config <file>`, `-q`.
5. **Default output placement**: if the user asked for JSON and did not pass `-o`, write next to the source file with a `.liteparse.json` suffix and report the path.
6. **Report**:
   - the exact file parsed,
   - key flags used,
   - output path if written to a file, otherwise a preview of stdout,
   - any dependency error or non-zero exit verbatim (do not paraphrase).

## Examples

```
/liteparse:parse-document ./docs/report.pdf --format json
/liteparse:parse-document ./contracts/master.docx --no-ocr
/liteparse:parse-document ./scans/invoice.pdf --ocr-server-url http://localhost:5000/ocr --target-pages "1-2"
/liteparse:parse-document ./slides.pptx --format text -o slides.txt
```

For details on CLI flags and dependency rules, see the background `liteparse` skill.
