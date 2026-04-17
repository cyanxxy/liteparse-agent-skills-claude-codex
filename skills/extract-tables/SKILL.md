---
name: extract-tables
description: Extract tables from a PDF, DOCX, XLSX, or PPTX file and output them as CSV or structured JSON. Use when pulling tabular data from documents for analysis, import, or downstream processing.
compatibility: Requires Node 18+ and `@llamaindex/liteparse`. LibreOffice for Office files. ImageMagick for images.
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

5. **Extract tables from the JSON output**. Read the parsed JSON and look for table structures. LiteParse JSON output contains page-level items with type and bounding-box metadata. Apply this detection order:

   **(a) Native tables first.** If the JSON contains items with `"type": "table"`, use them directly — these already have rows and cells resolved by LiteParse. Skip heuristic detection for these regions.

   **(b) Heuristic grid detection** (only for pages that had no `type: table` items):
   - **Column detection**: Cluster text items by their `bbox` x-start coordinate. Two items belong to the same column if their x-starts differ by ≤ 5% of page width (or ≤ 10px on 150 DPI renders). A column must have ≥ 3 aligned items to count.
   - **Row detection**: Within a candidate column cluster, sort items by y-coordinate. Two items belong to the same row if their y-centers differ by ≤ 0.5 × median line height on the page.
   - **Minimum table size**: Require ≥ 2 columns and ≥ 2 rows (the header row counts). Discard anything smaller as running text.
   - **Header detection**: Treat the top row as the header if (1) its text items are bold/larger than the body rows, or (2) the row below contains primarily numeric values while the top row is textual.

   **(c) Deterministic ordering**: Emit tables in reading order — page number ascending, then by top-left y-coordinate ascending. This keeps output stable across runs.

   If neither (a) nor (b) produces any result on a page, do not fabricate a table — proceed to the next page.

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
- **Merged cells**: when a cell visually spans multiple columns or rows, emit the value in its top-left cell and leave the spanned cells empty (`""`). Note the merge in the report if you want the consumer to know.
- **Multi-row headers**: if two consecutive rows both look header-like, concatenate them as `"<parent> - <child>"` column names (e.g. `"Q1 - Revenue"`).
