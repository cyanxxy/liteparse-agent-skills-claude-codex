---
name: convert-format
description: Convert documents between formats using LibreOffice (e.g., DOCX to PDF, PPTX to PDF, ODT to DOCX). Use when you need format conversion without parsing or text extraction.
compatibility: Requires LibreOffice installed and on PATH.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Convert Format

Convert a document from one format to another using LibreOffice headless conversion.

## Steps

1. **Resolve file path**. If missing, ask.
2. **Parse `--to <format>`**. Supported targets:
   - `pdf` — from DOCX, XLSX, PPTX, ODT, ODS, ODP, RTF, CSV, HTML
   - `docx` — from ODT, RTF, PDF (limited), HTML
   - `xlsx` — from ODS, CSV, TSV
   - `pptx` — from ODP
   - `html` — from DOCX, ODT, RTF
   - `txt` — from DOCX, ODT, RTF, PDF
   - `csv` — from XLSX, ODS

   If `--to` not provided, infer: Office files → `pdf`, CSV/TSV → `xlsx`. If ambiguous, ask.
3. **Verify LibreOffice**: `which libreoffice`. If absent, stop.
4. **Determine output path**: `-o <path>` if given, otherwise `<basename>.<target-format>` in the same directory.
5. **Run conversion**:
   ```bash
   libreoffice --headless --convert-to <format> --outdir <output-dir> "<input-file>"
   ```
   If user specified a custom filename via `-o`, rename after conversion.
6. **Verify output** exists and is non-zero size.
7. **Report**: input file/format, output file/format, file size, errors verbatim.

## Examples

```bash
/convert-format ./report.docx --to pdf
/convert-format ./data.csv --to xlsx -o spreadsheet.xlsx
/convert-format ./slides.pptx --to pdf -o ./exports/slides.pdf
/convert-format ./legacy.odt --to docx
```

## Notes

- This does **not** parse or extract text. For that, use `/parse-document`.
- PDF to DOCX is supported but limited — LibreOffice produces a best-effort editable document.
- Complex formatting (macros, embedded objects) may not convert perfectly.
