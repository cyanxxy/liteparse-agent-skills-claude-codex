---
name: convert-format
description: Convert documents between formats using LibreOffice's headless mode. Use for DOCX→PDF, PPTX→PDF, ODT→DOCX, CSV→XLSX, HTML→PDF, and similar format-only changes — any time the user says "turn X into Y", "export as PDF", "save as DOCX", or wants a format-only conversion without parsing or text extraction.
compatibility: Requires LibreOffice installed and on PATH.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Convert Format

Convert a document from one format to another using LibreOffice's headless conversion.

## Steps

1. **Resolve the input file path**. If no file was provided, ask for one.

2. **Parse `--to <format>`** from the additional flags. Supported target formats:
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
   LibreOffice writes to `--outdir` with the original basename and new extension.

6. **Rename if needed**. If the user specified a custom `-o <path>` that differs from LibreOffice's default output name, rename the output **before** running hooks so `{{output}}` always refers to the final path the user will see:
   ```bash
   mv "<libreoffice-output>" "<user-output>"
   ```

7. **Verify the output** exists and has a non-zero size. If the output file is missing or empty, report the failure and stop.

8. **Config files**: if the user passed `--config <file>`, report which config file was used. Do not inspect or execute `hooks.*` entries as part of this workflow.

9. **Report**:
   - Input file and format
   - Output file and format
   - File size of the output
   - Any errors from LibreOffice verbatim

## Examples

```bash
libreoffice --headless --convert-to pdf --outdir . "./report.docx"
libreoffice --headless --convert-to xlsx --outdir . "./data.csv" && mv data.xlsx spreadsheet.xlsx
libreoffice --headless --convert-to pdf --outdir "./exports" "./slides.pptx"
libreoffice --headless --convert-to docx --outdir . "./legacy.odt"
libreoffice --headless --convert-to pdf --outdir . "./invoice.html"
```

## Notes

- This skill does **not** parse or extract text — it converts the file format directly. For text extraction, use the `parse-document` skill.
- **Concurrent conversions**: LibreOffice's `--headless` mode can fail or produce corrupt output when multiple instances run simultaneously because they share one user profile. Two options:
  - **Serialize** (simplest): wrap each call in `flock` so only one LibreOffice runs at a time:
    ```bash
    flock /tmp/liteparse-libreoffice.lock libreoffice --headless --convert-to pdf --outdir . "./report.docx"
    ```
  - **Isolate profiles** (for parallel workloads): give each invocation its own profile directory via `-env:UserInstallation`:
    ```bash
    libreoffice --headless \
      -env:UserInstallation="file:///tmp/lo-profile-$$" \
      --convert-to pdf --outdir . "./report.docx"
    ```
    Use a unique path per concurrent job (PID, UUID, or job index). Clean up the profile directory when done.
- Conversion quality depends on LibreOffice's support for the input format. Complex formatting (macros, embedded objects, advanced layouts) may not convert perfectly.
- PDF to DOCX conversion is supported but limited — LibreOffice produces a best-effort editable document from the PDF layout.
