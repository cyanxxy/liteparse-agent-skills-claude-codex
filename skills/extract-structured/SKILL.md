---
name: extract-structured
description: Extract user-defined fields (invoice numbers, dates, totals, parties, line items, etc.) from PDFs, Office docs, and images into repeatable structured JSON, JSONL, or CSV. Use whenever specific named fields must be pulled from a document, inline or via a saved schema.
compatibility: Requires Node 18+ and either an installed `lit`/`liteparse` binary or `npm` for the `npx` fallback. LibreOffice for Office files. ImageMagick for images.
license: Apache-2.0
metadata:
  author: Local Workspace
  version: "0.2.0"
---

# Extract Structured

Extract user-defined fields from a document by first parsing it with LiteParse and then normalizing the requested fields into a canonical schema.

## Steps

1. **Resolve the file path** relative to the project root. If it is missing or ambiguous, ask the user for the correct path. If the user passed no arguments at all, ask for a file path.
2. **Check file-type dependencies**:
   - Office files (`.doc` `.docx` `.docm` `.odt` `.rtf` `.ppt` `.pptx` `.pptm` `.odp` `.xls` `.xlsx` `.xlsm` `.ods` `.csv` `.tsv`): run `which libreoffice`. If absent, report it and stop.
   - Image files (`.jpg` `.jpeg` `.png` `.gif` `.bmp` `.tiff` `.webp` `.svg`): run `which magick || which convert`. If neither exists, report it and stop.
   - PDFs: no extra dependency.
3. **Choose the CLI**: run `which lit || which liteparse`. If either succeeds, use that binary as `<cli>` and run `<cli> parse ...`. Otherwise, fall back to `npx -y @llamaindex/liteparse parse ...` (subcommand only — no `lit` prefix under npx).
4. **Decide the extraction schema** (precedence: `--schema` > `--fields` > ask):
   - If the user passed `--schema <file>`, read that file and treat it as the stable contract for this run. If `--fields` was also passed, the schema wins — note the ignored `--fields` in the final report so the user can resolve it.
   - Otherwise, if the user passed `--fields`, normalize the loose request into a canonical schema before extracting. Default inferred fields to optional single-value `string` fields and generate stable `snake_case` names when the user does not provide one.
   - If neither was provided, ask the user which fields they want extracted.
   - Always go through normalization before extracting, even for a single field. That is what lets repeat runs against similar inputs produce the same shape.
5. **Create a per-run temp directory and parse the file as JSON**:
   ```bash
   tmpdir=$(mktemp -d "${TMPDIR:-/tmp}/liteparse-structured.XXXXXX")
   trap 'rm -rf -- "$tmpdir"' EXIT
   <cli> parse <file> --format json -o "$tmpdir/raw.json"
   ```
6. **Read the parsed JSON and extract field values**. Use the parsed pages, text items, OCR output, tables, and bounding boxes to locate the best match for each field. Prefer direct label/value pairs, nearby text on the same page, and repeated section patterns.

   **Single-value fields** (`multiple: false`):
   - Emit as `{"value": <parsed>, "confidence": <0.0-1.0>, "evidence": {"page": <n>, "text": "<matching source text>"}}`.
   - If multiple plausible matches exist, set `value` to the best one and add `"alternatives": [{"value": ..., "evidence": ...}]` listing the other candidates. Mark `"status": "ambiguous"` so downstream consumers can flag it.
   - If no match is found, emit `{"value": null, "status": "missing", "evidence": null}`. Never guess.

   **Multi-value fields** (`multiple: true`, e.g. line items, party names, repeating rows):
   - Walk repeating patterns in the parsed output — table rows, list items, or text blocks that share consistent label/value structure on the same page or across pages.
   - Emit as `{"values": [<entry>, <entry>, ...], "confidence": <0.0-1.0>, "evidence": {"source": "table|list|pattern", "page": <n>}}`.
   - For line-item-style rows, each entry should itself be an object matching the sub-field shape if the schema declares sub-fields; otherwise a flat value per row.
   - If the pattern is partially matched (e.g. 4 of 5 rows parse cleanly), include all successful entries and add `"status": "partial"` with a count of skipped rows in `evidence`.

   **Type coercion**: cast values to the field's declared `type` (string/number/date/boolean) before emitting. On coercion failure, keep the raw string as `value` and set `"status": "type_mismatch"`.
7. **Determine the output format** from the additional flags:
   - `--json` (default)
   - `--jsonl`
   - `--csv`
   JSON is the source of truth; the flat formats are derived views for automation pipelines.
8. **Write output**:
   - If the user passed `-o <path>`, write there.
   - Otherwise write `<basename>-extracted.<ext>` next to the source file.
   - If `--save-schema <file>` was requested, write the normalized schema to that path so the user can reuse it later as a stable automation contract. When the source was already `--schema <src>`, the saved file is a re-serialized canonical copy; if `--save-schema` resolves to the same path as `<src>`, skip the write and say so in the report.
9. **Clean up** the temp directory.
10. **Report**:
    - the exact file parsed,
    - the schema source used (`--fields` or `--schema`),
    - the output path if written to a file, otherwise a preview of stdout,
    - any missing or ambiguous fields, with evidence summaries.

## Schema file format

Schemas loaded via `--schema` and written via `--save-schema` share the shape shown in [`invoice.extract.json`](./invoice.extract.json):

```json
{
  "version": "1.0",
  "name": "<recipe-name>",
  "output": { "format": "json|jsonl|csv", "includeEvidence": true },
  "fields": [
    {
      "name": "snake_case_id",
      "label": "Human Label",
      "type": "string|number|date|boolean",
      "required": true,
      "multiple": false,
      "description": "guidance for matching the right text on the page",
      "examples": ["sample value"],
      "enumValues": ["A", "B"]
    }
  ]
}
```

`examples` and `enumValues` are optional and help disambiguate matches. When normalizing from `--fields`, emit this same shape so saved recipes are drop-in replayable later.

## Examples

```bash
tmpdir=$(mktemp -d "${TMPDIR:-/tmp}/liteparse-structured.XXXXXX") && trap 'rm -rf -- "$tmpdir"' EXIT
lit parse ./invoice.pdf --format json -o "$tmpdir/raw.json"   # then extract: invoice number, invoice date, total amount
lit parse ./invoice.pdf --format json -o "$tmpdir/raw.json"   # then extract using ./examples/invoice.extract.json -> invoice-extracted.json
lit parse ./contracts/master.pdf --format json -o "$tmpdir/raw.json"   # then extract: party name:string, effective date:date, governing law:string -> JSONL
lit parse ./receipts/receipt.pdf --format json -o "$tmpdir/raw.json"   # then extract: merchant name, subtotal, total; save schema to ./schemas/receipt.extract.json
```

For details on CLI flags and dependency rules, see the background `liteparse` skill.
