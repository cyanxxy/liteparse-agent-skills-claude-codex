---
name: extract-tables
description: Extract tables from a PDF, DOCX, XLSX, or PPTX file and output them as CSV or structured JSON. Use when pulling tabular data from documents for analysis, import, or downstream processing.
compatibility: Requires Node 20.11+ and `@llamaindex/liteparse`. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Extract Tables

Parse a document with LiteParse in JSON mode and extract tabular data into CSV or structured JSON.

## Steps

1. **Resolve the input file path**. If no file was provided, ask for one.

2. **Check file-type dependencies**:
   - Office files: verify `which libreoffice`.
   - Image files: verify `which magick || which convert`.
   - PDFs: no extra dependency.

3. **Choose the CLI**: run `which lit`. If it exists, use `lit parse`. Otherwise, fall back to `npx -y @llamaindex/liteparse parse`.

4. **Parse the file as JSON**. Create a unique temp file to avoid collisions with concurrent runs:
   ```bash
   TMPFILE="$(mktemp /tmp/liteparse-tables-XXXXXX.json)"
   <cli> parse <file> --format json -o "$TMPFILE"
   ```

5. **Extract tables from the JSON output**. Read the parsed JSON and look for table structures. LiteParse JSON output contains page-level items with type and bounding-box metadata. Identify items that represent tables by:
   - Looking for `"type": "table"` items
   - Detecting grid-like structures with consistent row/column patterns in text blocks
   - Grouping adjacent text items that share aligned x-coordinates (columns) and sequential y-coordinates (rows)

6. **Determine output format** from the additional flags:
   - `--csv` (default if not specified): write each table as a separate CSV file
   - `--json`: write all tables as a JSON array of objects with headers as keys

7. **Write output**:
   - If the user passed `-o <path>`:
     - For CSV with multiple tables: treat `<path>` as a directory and write `table-1.csv`, `table-2.csv`, etc.
     - For JSON or single-table CSV: write directly to `<path>`
   - Otherwise:
     - CSV: write `<basename>-table-1.csv`, `<basename>-table-2.csv` next to the source file
     - JSON: write `<basename>-tables.json` next to the source file

8. **Clean up** the temp file:
   ```bash
   rm -f "$TMPFILE"
   ```

9. **Report**:
   - Number of tables found
   - Rows and columns per table
   - Output path(s)
   - If no tables were detected, say so clearly and suggest the document may not contain tabular data, or that OCR quality may be insufficient

## Examples

```bash
# Extract tables as CSV (default) — agent creates temp file, extracts, then cleans up
TMPFILE="$(mktemp /tmp/liteparse-tables-XXXXXX.json)"
lit parse ./report.pdf --format json -o "$TMPFILE"   # then extract tables, then rm -f "$TMPFILE"

# Extract from XLSX into a directory
lit parse ./financials.xlsx --format json -o "$TMPFILE"   # then extract -> ./tables/

# Extract as JSON
lit parse ./invoice.pdf --format json -o "$TMPFILE"   # then extract as JSON -> invoice-tables.json

# Extract from specific pages
lit parse ./scan.pdf --format json --target-pages "1-3" -o "$TMPFILE"   # then extract tables as CSV
```

## Notes

- Table extraction quality depends on the document structure. Well-formatted PDFs with explicit table borders yield the best results.
- For scanned documents, ensure OCR is enabled (it is by default).
- XLSX files already contain structured data — for these, LiteParse extracts sheet content directly, which typically produces cleaner table output.
