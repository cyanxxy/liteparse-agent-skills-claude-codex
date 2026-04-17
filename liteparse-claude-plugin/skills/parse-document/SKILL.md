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
   - Office files (`.doc` `.docx` `.docm` `.odt` `.rtf` `.ppt` `.pptx` `.pptm` `.odp` `.xls` `.xlsx` `.xlsm` `.ods` `.csv` `.tsv`): run `which libreoffice`. If absent, report it and stop.
   - Image files (`.jpg` `.jpeg` `.png` `.gif` `.bmp` `.tiff` `.webp` `.svg`): run `which magick || which convert`. If neither exists, report it and stop.
   - PDFs: no extra dependency.
3. **Choose the CLI**: run `which lit`. If it succeeds, use `lit parse ...`. Otherwise, fall back to `npx -y @llamaindex/liteparse parse ...` (subcommand only — no `lit` prefix under npx).
4. **Run the parse** with the user's flags from `$ARGUMENTS`. Respect any of: `--format json|text`, `-o <file>`, `--target-pages "1-5"`, `--no-ocr`, `--ocr-server-url <url>`, `--ocr-language <lang>`, `--dpi <n>`, `--password <pw>`, `--config <file>`, `-q`.
5. **Default output placement**:
   - If the user did not pass `-o`, the CLI writes to stdout. Show a preview capped at **the first 50 lines or 4,000 characters, whichever comes first**. If the output exceeds that, show the preview, append `… [truncated: showed N of M lines]`, and suggest rerunning with `-o <file>` to write the full output.
   - When JSON was requested and no `-o` was given, **always** write to a file (JSON is not pleasant to read in a preview) — default to the source file's directory with a `.liteparse.json` suffix, then show the first ~40 lines of that file as a preview.
6. **Post-parse hooks**: if a `liteparse.config.json` exists (passed via `--config` or found at the project root) and contains `hooks.postParse`, execute each command in the array after a successful parse. Substitute `{{file}}` with the input path and `{{output}}` with the output path before running via `bash -c`. Report hook results. If a hook fails, report the error but do not roll back the parse output. Show the user what hooks will run before first execution.
7. **Report**:
   - the exact file parsed,
   - key flags used,
   - output path if written to a file, otherwise a preview of stdout,
   - any dependency error or non-zero exit verbatim (do not paraphrase).

## Examples

```
/liteparse:parse-document ./docs/report.pdf --format json
/liteparse:parse-document ./contracts/master.docx --no-ocr
/liteparse:parse-document ./scans/invoice.pdf --ocr-server-url http://localhost:8828/ocr --target-pages "1-2"
/liteparse:parse-document ./slides.pptx --format text -o slides.txt
/liteparse:parse-document /Users/alice/Downloads/lease-agreement.pdf --format json -o ~/parsed/lease.json   # absolute paths work too
```

For details on CLI flags and dependency rules, see the background `liteparse` skill.
