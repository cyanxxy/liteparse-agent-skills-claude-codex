---
name: convert-format
description: Convert documents between formats using LibreOffice (e.g., DOCX to PDF, PPTX to PDF, ODT to DOCX). Use when you need format conversion without parsing or text extraction.
argument-hint: "<file> --to <format> [-o output]"
allowed-tools: Read Write Bash(which *) Bash(libreoffice *) Bash(mkdir *) Bash(ls *) Bash(mv *)
---

# Convert Format

Convert a document from one format to another using LibreOffice's headless conversion.

## Steps

1. **Resolve `$0`** as the input file path. If no arguments were passed, ask for a file.

2. **Parse `--to <format>`** from `$ARGUMENTS`. Supported target formats:
   - `pdf` — from DOCX, XLSX, PPTX, ODT, ODS, ODP, RTF, CSV, HTML
   - `docx` — from ODT, RTF, PDF (limited), HTML
   - `xlsx` — from ODS, CSV, TSV
   - `pptx` — from ODP
   - `html` — from DOCX, ODT, RTF
   - `txt` — from DOCX, ODT, RTF, PDF
   - `csv` — from XLSX, ODS

   If `--to` was not provided, infer the most likely target:
   - Office files → `pdf`
   - CSV/TSV → `xlsx`
   If the inference is ambiguous, ask the user.

3. **Verify LibreOffice**: run `which libreoffice`. If absent, stop and tell the user LibreOffice is required for format conversion.

4. **Determine output path**:
   - If the user passed `-o <path>`, use it.
   - Otherwise, write to the same directory as the input with the new extension: `<basename>.<target-format>`.

5. **Run the conversion**:
   ```bash
   libreoffice --headless --convert-to <format> --outdir <output-dir> "<input-file>"
   ```
   LibreOffice writes to `--outdir` with the original basename and new extension. If the user specified a custom `-o` filename, rename the output after conversion.

6. **Verify the output** exists and has a non-zero size. If the output file is missing or empty, report the failure.

7. **Post-convert hooks**: if a `liteparse.config.json` exists (passed via `--config` or found at the project root) and contains `hooks.postConvert`, execute each command after success. Substitute `{{file}}` with the input path and `{{output}}` with the output path before running via `bash -c`. Report hook results. If a hook fails, report the error but do not roll back the conversion output.

8. **Report**:
   - Input file and format
   - Output file and format
   - File size of the output
   - Hooks executed and their results
   - Any errors from LibreOffice verbatim

## Examples

```
/liteparse:convert-format ./report.docx --to pdf
/liteparse:convert-format ./data.csv --to xlsx -o spreadsheet.xlsx
/liteparse:convert-format ./slides.pptx --to pdf -o ./exports/slides.pdf
/liteparse:convert-format ./legacy.odt --to docx
/liteparse:convert-format ./invoice.html --to pdf
```

## Notes

- This skill does **not** parse or extract text — it converts the file format directly. For text extraction, use `/liteparse:parse-document`.
- Conversion quality depends on LibreOffice's support for the input format. Complex formatting (macros, embedded objects, advanced layouts) may not convert perfectly.
- PDF to DOCX conversion is supported but limited — LibreOffice produces a best-effort editable document from the PDF layout.
