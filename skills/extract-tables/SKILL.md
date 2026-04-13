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

Parse a document in JSON mode and extract tabular data into CSV or JSON.

## Steps

1. **Resolve file path**. If missing, ask.
2. **Check file-type dependencies** (LibreOffice for Office, ImageMagick for images).
3. **Choose the CLI**: `which lit` → `lit parse`, otherwise `npx -y @llamaindex/liteparse parse`.
4. **Parse as JSON**: `<cli> parse <file> --format json -o /tmp/liteparse-tables-raw.json`.
5. **Extract tables** from the JSON output. Identify tables by:
   - `"type": "table"` items
   - Grid-like structures with consistent row/column patterns
   - Adjacent text items sharing aligned x-coordinates (columns) and sequential y-coordinates (rows)
6. **Output format** (`--csv` default, or `--json`):
   - CSV: one file per table (`table-1.csv`, `table-2.csv`)
   - JSON: all tables as an array of objects with headers as keys
7. **Write output**: to `-o <path>` if given, otherwise next to source file as `<basename>-table-N.csv` or `<basename>-tables.json`.
8. **Report**: tables found, rows/columns per table, output paths. If none found, say so.

## Examples

```bash
/extract-tables ./report.pdf
/extract-tables ./financials.xlsx --csv -o ./tables/
/extract-tables ./invoice.pdf --json -o invoice-tables.json
```

## Notes

- Well-formatted PDFs with explicit table borders yield the best results.
- XLSX files already contain structured data and produce cleaner output.
- For scanned documents, ensure OCR is enabled (default).
